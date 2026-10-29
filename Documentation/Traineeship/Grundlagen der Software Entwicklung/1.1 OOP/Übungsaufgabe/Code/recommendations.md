"""
Price-recommendation logic.

The RecommendationEngine reads the current capacity snapshot from Postgres,
computes a recommended price adjustment (in percent) for every future
(product_id, calendar_date), and writes the results to price_recommendations.
The table is fully replaced on each run (TRUNCATE + INSERT).

Formula (from the README):

    recommended_adjustment_pct = 15 * (a - 0.5) * (0.5 + d)

    a = occupancy fraction         = booked / (booked + bookable + optional)
    d = days_until_travel / 300    (calendar_date - reference_date, in days)

Rows with zero total capacity or negative (bookable + optional) are skipped;
counts of both are returned so callers can track data-quality issues.

get_max() returns the single recommendation with the highest signed
recommended_adjustment_pct, enriched with capacity and price context.
"""

import logging
from dataclasses import dataclass
from datetime import date, datetime, timezone
from decimal import Decimal
from typing import Optional

import polars as pl
from sqlalchemy import Engine, text

from src.database import DATABASE_URL, engine


logger = logging.getLogger(__name__)


@dataclass
class RecommendationResult:
    """Summary of a single recommendation run."""

    rows_written: int
    rows_skipped_zero_capacity: int
    rows_skipped_negative_capacity: int
    calculated_at: datetime


@dataclass
class MaxRecommendation:
    """A single recommendation enriched with capacity and price context."""

    product_id: str
    calendar_date: date
    days_until_travel: int
    recommended_adjustment_pct: Decimal
    calculated_at: datetime
    booked_units: int
    total_capacity: int
    occupancy: float
    current_prices: dict[str, Optional[Decimal]]


class RecommendationEngine:
    """Computes and persists price recommendations."""

    def __init__(
        self,
        db_engine: Engine = engine,
        reference_date: Optional[date] = None,
        connection_url: str = DATABASE_URL,
    ):
        """
        db_engine       Injectable SQLAlchemy engine (tests can pass a test DB).
        reference_date  The date used as "today" when computing days_until_travel.
                        Defaults to the system clock. Pin it in tests for
                        reproducible output.
        connection_url  Connection string used by polars' ADBC write path.
                        Kept alongside the SQLAlchemy engine because ADBC
                        wants a URL string, not an Engine.
        """
        self.engine = db_engine
        self._reference_date = reference_date
        self.connection_url = connection_url

    @property
    def reference_date(self) -> date:
        """Resolve the reference date at call time, not construction time.

        This matters when the engine is long-lived (e.g. held by a FastAPI
        app across requests): without this, every run would use the date
        at which the process started.
        """
        return self._reference_date or date.today()

    # ---------- public entrypoints ----------

    def run(self) -> RecommendationResult:
        """Compute, truncate, and write the full recommendation set."""
        reference_date = self.reference_date
        calculated_at = datetime.now(timezone.utc)

        capacity = self._read_capacity()

        recs, skipped_zero, skipped_negative = self._compute(
            capacity,
            reference_date=reference_date,
            calculated_at=calculated_at,
        )

        self._truncate("price_recommendations")
        if recs.height > 0:
            self._write(recs, "price_recommendations")

        result = RecommendationResult(
            rows_written=recs.height,
            rows_skipped_zero_capacity=skipped_zero,
            rows_skipped_negative_capacity=skipped_negative,
            calculated_at=calculated_at,
        )

        if result.rows_skipped_zero_capacity > 0:
            logger.warning(
                "Recommendation run skipped %d future (product_id, calendar_date) "
                "rows with zero total capacity (occupancy undefined).",
                result.rows_skipped_zero_capacity,
            )
        if result.rows_skipped_negative_capacity > 0:
            logger.warning(
                "Recommendation run skipped %d future (product_id, calendar_date) "
                "rows with negative (bookable + optional) capacity -- upstream data issue.",
                result.rows_skipped_negative_capacity,
            )

        return result

    def get_max(self) -> Optional[MaxRecommendation]:
        """Return the recommendation with the highest signed adjustment.

        Enriched with booked_units, total_capacity, occupancy, and the
        current price for every length_of_stay the product is priced for
        ("current" = most recent change_date <= the travel date).

        Returns None if the price_recommendations table is empty.
        """
        top = self._fetch_top_recommendation()
        if top is None:
            return None

        booked = top["booked_units"]
        total = top["total_capacity"]
        prices = self._fetch_current_prices(
            product_id=top["product_id"],
            as_of=top["calendar_date"],
        )

        return MaxRecommendation(
            product_id=top["product_id"],
            calendar_date=top["calendar_date"],
            days_until_travel=top["days_until_travel"],
            recommended_adjustment_pct=top["recommended_adjustment_pct"],
            calculated_at=top["calculated_at"],
            booked_units=booked,
            total_capacity=total,
            occupancy=(booked / total) if total > 0 else 0.0,
            current_prices=prices,
        )

    # ---------- computation ----------

    @staticmethod
    def _compute(
        capacity: pl.DataFrame,
        reference_date: date,
        calculated_at: datetime,
    ) -> tuple[pl.DataFrame, int, int]:
        """Apply the recommendation formula to every future capacity row.

        Returns the recommendation DataFrame plus counts of the two kinds of
        rows we dropped for data-quality reasons.
        """
        future = capacity.filter(pl.col("calendar_date") > reference_date)

        future = future.with_columns(
            (
                pl.col("booked_units")
                + pl.col("bookable_units")
                + pl.col("optional_units")
            ).alias("total_capacity")
        )

        skipped_zero = future.filter(pl.col("total_capacity") == 0).height
        skipped_negative = future.filter(
            pl.col("bookable_units") + pl.col("optional_units") < 0
        ).height

        usable = future.filter(
            (pl.col("total_capacity") > 0)
            & (pl.col("bookable_units") + pl.col("optional_units") >= 0)
        )

        recs = usable.with_columns(
            (pl.col("booked_units") / pl.col("total_capacity")).alias("a"),
            (
                (pl.col("calendar_date") - reference_date).dt.total_days()
                / 300.0
            ).alias("d"),
            days_until_travel=(
                pl.col("calendar_date") - reference_date
            ).dt.total_days().cast(pl.Int32),
        ).with_columns(
            recommended_adjustment_pct=(
                15.0 * (pl.col("a") - 0.5) * (0.5 + pl.col("d"))
            ).cast(pl.Decimal(precision=12, scale=6)),
        ).with_columns(
            calculated_at=pl.lit(calculated_at),
        ).select(
            "product_id",
            "calendar_date",
            "days_until_travel",
            "recommended_adjustment_pct",
            "calculated_at",
        )

        return recs, skipped_zero, skipped_negative

    # ---------- DB helpers: bulk ingestion path ----------

    def _read_capacity(self) -> pl.DataFrame:
        return pl.read_database_uri(
            query="SELECT product_id, calendar_date, booked_units, "
                "bookable_units, optional_units FROM capacity",
            uri=self.connection_url,
            engine="adbc",
        )

    def _truncate(self, table_name: str) -> None:
        with self.engine.begin() as conn:
            conn.execute(text(f"TRUNCATE TABLE {table_name}"))

    def _write(self, df: pl.DataFrame, table_name: str) -> None:
        """Bulk-insert via ADBC. See ingestion.py for rationale."""
        df.write_database(
            table_name=table_name,
            connection=self.connection_url,
            if_table_exists="append",
            engine="adbc",
        )

    # ---------- DB helpers: get_max path ----------

    def _fetch_top_recommendation(self) -> Optional[dict]:
        """Join price_recommendations with capacity and return the top row.

        Signed-max ordering: the single most positive adjustment wins. Ties
        are broken by Postgres' arbitrary ordering, which the README allows.
        """
        sql = text(
            """
            SELECT
                pr.product_id,
                pr.calendar_date,
                pr.days_until_travel,
                pr.recommended_adjustment_pct,
                pr.calculated_at,
                c.booked_units,
                (c.booked_units + c.bookable_units + c.optional_units)
                    AS total_capacity
            FROM price_recommendations pr
            JOIN capacity c
              ON c.product_id = pr.product_id
             AND c.calendar_date = pr.calendar_date
            ORDER BY pr.recommended_adjustment_pct DESC
            LIMIT 1
            """
        )
        with self.engine.connect() as conn:
            row = conn.execute(sql).mappings().first()
        return dict(row) if row is not None else None

    def _fetch_current_prices(
        self,
        product_id: str,
        as_of: date,
    ) -> dict[str, Optional[Decimal]]:
        """Return the current price per length_of_stay for a product.

        'Current' means: for each length_of_stay that this product is priced
        for in `prices`, the price_sterling from the most recent change_date
        that is <= as_of. If no such row exists (every change_date is after
        as_of), the value is None. Length_of_stay values that the product
        has no price-change row for at all are not included.
        """
        # DISTINCT ON picks one row per length_of_stay, ordered so that the
        # kept row is the most recent change_date <= as_of (NULLs LAST so a
        # length_of_stay with only future changes yields a NULL price).
        sql = text(
            """
            SELECT length_of_stay, change_date, price_sterling
            FROM (
                SELECT DISTINCT ON (length_of_stay)
                    length_of_stay,
                    change_date,
                    price_sterling
                FROM prices
                WHERE product_id = :product_id
                ORDER BY length_of_stay,
                         (change_date <= :as_of) DESC,
                         change_date DESC
            ) latest
            """
        )
        with self.engine.connect() as conn:
            rows = conn.execute(
                sql, {"product_id": product_id, "as_of": as_of}
            ).mappings().all()

        # A row's change_date might still be > as_of if *every* change for
        # that length_of_stay is in the future -- in that case we report null.
        result: dict[str, Optional[Decimal]] = {}
        for r in rows:
            if r["change_date"] <= as_of:
                result[r["length_of_stay"]] = r["price_sterling"]
            else:
                result[r["length_of_stay"]] = None
        return result