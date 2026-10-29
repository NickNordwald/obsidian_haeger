"""
Pydantic models for API request and response bodies.

These are the wire-format models and are intentionally separate from the
SQLAlchemy ORM models in datamodels.py. That separation lets the database
schema and the API schema evolve independently.
"""

from datetime import date, datetime
from decimal import Decimal
from typing import Optional

from pydantic import BaseModel, Field


class StartIngestionRequest(BaseModel):
    """Body for POST /start-ingestion.

    data_dir is a path on the *server's* filesystem, not an upload. This
    matches the README spec. For a file-upload flow see /start-ingestion-upload.
    """

    data_dir: str = Field(
        ...,
        description="Path on the server containing the three daily CSV files.",
        examples=["daily_data"],
    )


class IngestionDataQuality(BaseModel):
    """Data-quality counters surfaced by an ingestion run.

    Both fields are expected to be 0 on a clean data drop. Non-zero
    values indicate upstream issues that didn't stop the ingestion but
    are worth investigating.
    """

    orphan_bookings: int = Field(
        ...,
        description=(
            "Number of Firm booking-nights whose (product_id, calendar_date) "
            "had no matching capacity row and were silently dropped."
        ),
    )
    duplicate_prices_collapsed: int = Field(
        ...,
        description=(
            "Number of duplicate (product_id, change_date, length_of_stay) "
            "rows in prices.csv that were collapsed to one by taking the max."
        ),
    )


class IngestionRowCounts(BaseModel):
    """Rows written to each table by an ingestion run."""

    capacity: int
    prices: int


class IngestionResponse(BaseModel):
    """Response for both ingestion endpoints."""

    status: str = Field(..., examples=["ok"])
    message: str = Field(..., examples=["Ingestion complete."])
    rows_written: IngestionRowCounts
    data_quality: IngestionDataQuality


class RecommendationResponse(BaseModel):
    """Response for POST /price-recommendations."""

    status: str = Field(..., examples=["ok"])
    rows_written: int = Field(
        ...,
        description="Number of (product_id, calendar_date) recommendations written.",
    )
    rows_skipped_zero_capacity: int = Field(
        ...,
        description="Rows where booked + bookable + optional == 0, so occupancy is undefined.",
    )
    rows_skipped_negative_capacity: int = Field(
        ...,
        description="Rows where bookable + optional < 0, indicating an upstream data issue.",
    )
    calculated_at: datetime = Field(
        ...,
        description="UTC timestamp shared by all rows written in this run.",
    )


class MaxRecommendationResponse(BaseModel):
    """Response for GET /max-recommendation.

    Returns the bookable unit with the highest (signed) recommended price
    adjustment, enriched with capacity context and the current price for
    every length_of_stay the product is priced for.

    Interpretation note: "highest" is read as signed maximum (most positive
    adjustment), not largest magnitude. See README for the rationale.
    """

    product_id: str
    calendar_date: date
    days_until_travel: int
    recommended_adjustment_pct: Decimal
    calculated_at: datetime

    booked_units: int
    total_capacity: int = Field(
        ...,
        description="booked + bookable + optional",
    )
    occupancy: float = Field(
        ...,
        description="booked_units / total_capacity, in [0, 1]",
    )

    current_prices: dict[str, Optional[Decimal]] = Field(
        ...,
        description=(
            "Current price per length_of_stay for this product, where "
            '"current" means the most recent change_date <= calendar_date. '
            "Null when no such price-change row exists for that length_of_stay."
        ),
        examples=[{"2": 220.00, "3": 310.00, "4": None, "7": 640.00, "XN": 89.00}],
    )