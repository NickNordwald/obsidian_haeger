"""
Ingestion logic for the daily CSV files.

The Ingestor loads the three customer CSVs (capacity, bookings, prices),
preprocesses them according to our data model decisions, and writes them
to Postgres. capacity and prices are fully replaced (TRUNCATE + INSERT);
price_recommendations is left untouched.

Filenames are resolved by glob: the ingestor looks for exactly one file
matching *_capacity.csv, *_bookings.csv, *_prices.csv in the provided
directory. The date prefix is not inspected.

Data-quality accounting:
    _build_capacity counts orphan bookings (Firm booking-nights whose
    (product_id, calendar_date) does not exist in capacity.csv -- these
    are silently dropped by the left join).

    _build_prices counts duplicate composite keys that had to be
    collapsed (the customer's prices data contains rows with the same
    (product_id, change_date, length_of_stay) but different prices; we
    keep the max as a defensive placeholder).

Both counts are returned to the caller via IngestionResult and also
logged at WARNING level if non-zero.
"""

import logging
from dataclasses import dataclass
from pathlib import Path

import polars as pl
from sqlalchemy import Engine, text

from src.database import DATABASE_URL, engine


logger = logging.getLogger(__name__)


@dataclass
class IngestionResult:
    """Summary of a single ingestion run.

    Exposed so the API endpoint can report data-quality counts back to the
    caller without reaching into the DB.
    """

    capacity_rows_written: int
    prices_rows_written: int
    orphan_bookings: int
    duplicate_prices_collapsed: int


class Ingestor:
    """Loads the three daily CSVs into Postgres."""

    def __init__(
        self,
        data_dir: str | Path,
        db_engine: Engine = engine,
        connection_url: str = DATABASE_URL,
    ):
        self.data_dir = Path(data_dir)
        self.engine = db_engine
        # Kept alongside the engine because polars' ADBC bulk-write path
        # wants a connection string, not a SQLAlchemy Engine.
        self.connection_url = connection_url

    # ---------- filename resolution ----------

    def _resolve_csv(self, suffix: str) -> Path:
        """Find the single CSV in data_dir matching ``*_<suffix>.csv``.

        The customer's filenames carry a date prefix (e.g. ``20260108_``)
        that changes from one daily drop to the next. Rather than hardcode
        a date, we glob for the suffix and expect exactly one match.

        Raises FileNotFoundError with a clear message if zero or multiple
        files match -- both are data-drop errors worth surfacing rather
        than silently picking one.
        """
        pattern = f"*_{suffix}.csv"
        matches = sorted(self.data_dir.glob(pattern))
        if len(matches) == 0:
            raise FileNotFoundError(
                f"No file matching '{pattern}' in {self.data_dir}"
            )
        if len(matches) > 1:
            raise FileNotFoundError(
                f"Multiple files matching '{pattern}' in {self.data_dir}: "
                f"{[m.name for m in matches]}"
            )
        return matches[0]

    # ---------- public entrypoint ----------

    def run(self) -> IngestionResult:
        """Run the full ingestion pipeline.

        Returns a structured result so the caller can inspect row counts
        and data-quality counters. Non-zero counters are also logged at
        WARNING level.
        """
        capacity_df, orphan_bookings = self._build_capacity()
        prices_df, duplicate_prices_collapsed = self._build_prices()

        self._truncate("capacity")
        self._truncate("prices")

        self._write(capacity_df, "capacity")
        self._write(prices_df, "prices")

        result = IngestionResult(
            capacity_rows_written=capacity_df.height,
            prices_rows_written=prices_df.height,
            orphan_bookings=orphan_bookings,
            duplicate_prices_collapsed=duplicate_prices_collapsed,
        )

        if result.orphan_bookings > 0:
            logger.warning(
                "Ingestion dropped %d Firm booking-nights with no matching "
                "(product_id, calendar_date) in capacity.csv.",
                result.orphan_bookings,
            )
        if result.duplicate_prices_collapsed > 0:
            logger.warning(
                "Ingestion collapsed %d duplicate (product_id, change_date, "
                "length_of_stay) rows in prices.csv by taking the max price.",
                result.duplicate_prices_collapsed,
            )

        return result

    # ---------- preprocessing ----------

    def _build_capacity(self) -> tuple[pl.DataFrame, int]:
        """Load capacity.csv and merge in booking counts from bookings.csv.

        A booking with bkg_nights=N starting on date D occupies one unit of
        capacity on each of the N calendar dates D, D+1, ..., D+N-1. We expand
        each booking into one row per occupied night before aggregating, so
        booked_units per (product_id, calendar_date) reflects all bookings
        that overlap that date -- not only those that start on it.

        Returns:
            (merged_df, orphan_count)

            merged_df       One row per (product_id, calendar_date) from
                            capacity.csv, with booked_units attached.
            orphan_count    Number of (product_id, calendar_date) pairs in
                            the exploded booking data that had no matching
                            capacity row and were silently dropped by the
                            left join. Expected to be 0 with clean data.
        """
        capacity = pl.read_csv(self._resolve_csv("capacity"))
        bookings = pl.read_csv(self._resolve_csv("bookings"))

        # Drop cancelled bookings, parse the start date, then explode each
        # booking into one row per night of the stay.
        bookings_agg = (
            bookings
            .filter(pl.col("status") == "Firm")
            .with_columns(
                pl.col("date").cast(pl.Utf8).str.strptime(pl.Date, format="%Y%m%d", strict=False)
            )
            .with_columns(
                pl.date_ranges(
                    start=pl.col("date"),
                    end=pl.col("date").dt.offset_by(
                        (pl.col("bkg_nights") - 1).cast(pl.Utf8) + "d"
                    ),
                    interval="1d",
                ).alias("calendar_date")
            )
            .explode("calendar_date")
            .group_by(["product_id", "calendar_date"])
            .agg(pl.len().alias("booked_units"))
        )

        capacity = capacity.with_columns(
            pl.col("calendar_date").cast(pl.Utf8).str.strptime(pl.Date, format="%Y%m%d", strict=False)
        )

        # Count orphans: (product_id, calendar_date) pairs in bookings_agg
        # with no matching row in capacity. An anti-join returns exactly
        # those rows from the left side that have no match on the right.
        orphan_count = bookings_agg.join(
            capacity.select(["product_id", "calendar_date"]),
            on=["product_id", "calendar_date"],
            how="anti",
        ).height

        merged = (
            capacity
            .join(bookings_agg, on=["product_id", "calendar_date"], how="left")
            .with_columns(pl.col("booked_units").fill_null(0).cast(pl.Int32))
            .rename({"is_bookable": "bookable_units", "is_option": "optional_units"})
            .with_columns(
                # Match the Postgres schema (SQLAlchemy Integer = INT4). ADBC's
                # COPY BINARY path is stricter than SQLAlchemy's INSERT and
                # rejects Int64 -> Integer coercion.
                pl.col("bookable_units").cast(pl.Int32),
                pl.col("optional_units").cast(pl.Int32),
            )
            .select([
                "product_id",
                "calendar_date",
                "booked_units",
                "bookable_units",
                "optional_units",
            ])
        )

        return merged, orphan_count

    def _build_prices(self) -> tuple[pl.DataFrame, int]:
        """Load prices.csv, drop the DWH id, rename columns to match the schema.

        The source data contains a small number of duplicate
        (product_id, change_date, length_of_stay) keys with different prices.
        We keep the maximum price for each duplicate group as a defensive
        placeholder. This is a known upstream data-quality issue; ideally
        the customer would disambiguate.

        Returns:
            (prices_df, duplicates_collapsed)

            prices_df              One row per (product_id, change_date,
                                   length_of_stay), ready to write.
            duplicates_collapsed   Number of rows removed by the dedup step.
                                   Equal to (input_rows - output_rows). 0
                                   means no duplicates were found in this
                                   drop.
        """
        prices = pl.read_csv(self._resolve_csv("prices"))
        input_rows = prices.height

        deduped = (
            prices
            .with_columns(
                pl.col("date").cast(pl.Utf8).str.strptime(pl.Date, format="%Y%m%d", strict=False)
            )
            .rename({
                "date": "change_date",
                "current_price": "price_sterling",
            })
            .group_by(["product_id", "change_date", "length_of_stay"])
            .agg(pl.col("price_sterling").max())
            .with_columns(pl.col("price_sterling").cast(pl.Decimal(precision=12, scale=2)))
            .select([
                "product_id",
                "change_date",
                "length_of_stay",
                "price_sterling",
            ])
        )

        duplicates_collapsed = input_rows - deduped.height
        return deduped, duplicates_collapsed

    # ---------- DB helpers ----------

    def _truncate(self, table_name: str) -> None:
        with self.engine.begin() as conn:
            conn.execute(text(f"TRUNCATE TABLE {table_name}"))

    def _write(self, df: pl.DataFrame, table_name: str) -> None:
        """Bulk-insert a DataFrame into Postgres via ADBC.

        We use ADBC (Arrow Database Connectivity) rather than the default
        SQLAlchemy path because SQLAlchemy's INSERT emits one round-trip
        per row. For the capacity table (~1.3M rows) that took ~18 s on
        a local DB; ADBC cuts it to a few seconds by streaming Arrow
        data in bulk.
        """
        df.write_database(
            table_name=table_name,
            connection=self.connection_url,
            if_table_exists="append",
            engine="adbc",
        )


if __name__ == "__main__":
    # Hardcoded path for now; the API endpoint will replace this later.
    logging.basicConfig(level=logging.INFO)
    result = Ingestor(data_dir="daily_data").run()
    print(f"Ingestion complete: {result}")