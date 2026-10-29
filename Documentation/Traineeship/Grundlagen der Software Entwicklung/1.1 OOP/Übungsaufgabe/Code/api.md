"""
FastAPI application exposing the three endpoints required by the README:

    POST /start-ingestion         -- ingest from a server-side folder path (spec)
    POST /start-ingestion-upload  -- same, but files uploaded in the request (bonus)
    POST /price-recommendations   -- compute and persist recommendations
    GET  /max-recommendation      -- return the highest-adjustment recommendation

Routes are intentionally thin: they parse the request, call a function from
ingestion.py / recommendations.py, and return a response. All business logic
lives in those modules.
"""

import shutil
import tempfile
from pathlib import Path

from fastapi import FastAPI, File, HTTPException, UploadFile, status

from src.ingestion import IngestionResult, Ingestor
from src.recommendations import RecommendationEngine
from src.schemas import (
    IngestionDataQuality,
    IngestionResponse,
    IngestionRowCounts,
    MaxRecommendationResponse,
    RecommendationResponse,
    StartIngestionRequest,
)

app = FastAPI(
    title="Tourism Pricing API",
    description="Prototype backend for CSV ingestion and price recommendations.",
    version="0.1.0",
)


# ---------- helpers ----------


def _ingestion_response(result: IngestionResult) -> IngestionResponse:
    """Translate an IngestionResult dataclass into the wire-format response."""
    return IngestionResponse(
        status="ok",
        message="Ingestion complete.",
        rows_written=IngestionRowCounts(
            capacity=result.capacity_rows_written,
            prices=result.prices_rows_written,
        ),
        data_quality=IngestionDataQuality(
            orphan_bookings=result.orphan_bookings,
            duplicate_prices_collapsed=result.duplicate_prices_collapsed,
        ),
    )


# ---------- ingestion ----------


@app.post("/start-ingestion", response_model=IngestionResponse)
def start_ingestion(request: StartIngestionRequest) -> IngestionResponse:
    """Ingest the three daily CSVs from a folder on the server's filesystem.

    Matches the README spec literally. For an upload-based alternative, see
    /start-ingestion-upload.
    """
    data_dir = Path(request.data_dir)
    if not data_dir.is_dir():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"data_dir '{request.data_dir}' is not a directory on the server.",
        )

    try:
        result = Ingestor(data_dir=data_dir).run()
    except FileNotFoundError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(e),
        )

    return _ingestion_response(result)


@app.post("/start-ingestion-upload", response_model=IngestionResponse)
def start_ingestion_upload(
    capacity: UploadFile = File(..., description="capacity CSV"),
    bookings: UploadFile = File(..., description="bookings CSV"),
    prices: UploadFile = File(..., description="prices CSV"),
) -> IngestionResponse:
    """Ingest the three daily CSVs from files uploaded in the request body.

    Convenience endpoint not required by the spec. Writes the uploads to a
    temporary directory, then reuses the same Ingestor as /start-ingestion.
    The temp directory is cleaned up automatically when the request ends.

    The ingestor resolves filenames by glob (*_capacity.csv etc.), so we
    save the uploads under filenames that match -- the date prefix we use
    is arbitrary.
    """
    with tempfile.TemporaryDirectory() as tmp:
        tmp_path = Path(tmp)
        for upload, target_name in [
            (capacity, "upload_capacity.csv"),
            (bookings, "upload_bookings.csv"),
            (prices, "upload_prices.csv"),
        ]:
            with open(tmp_path / target_name, "wb") as f:
                shutil.copyfileobj(upload.file, f)

        result = Ingestor(data_dir=tmp_path).run()

    return _ingestion_response(result)


# ---------- price recommendations ----------


@app.post("/price-recommendations", response_model=RecommendationResponse)
def price_recommendations() -> RecommendationResponse:
    """Compute and persist price recommendations for every future capacity row.

    Uses the system clock as the reference date for "days until travel".
    Fully replaces the price_recommendations table on each run.
    """
    result = RecommendationEngine().run()
    return RecommendationResponse(
        status="ok",
        rows_written=result.rows_written,
        rows_skipped_zero_capacity=result.rows_skipped_zero_capacity,
        rows_skipped_negative_capacity=result.rows_skipped_negative_capacity,
        calculated_at=result.calculated_at,
    )


@app.get("/max-recommendation", response_model=MaxRecommendationResponse)
def max_recommendation() -> MaxRecommendationResponse:
    """Return the recommendation with the highest (signed) adjustment.

    Read-only: does not trigger a recompute. If price_recommendations is
    empty, returns 404 -- call POST /price-recommendations first.
    """
    result = RecommendationEngine().get_max()
    if result is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="No recommendations available. Call POST /price-recommendations first.",
        )

    return MaxRecommendationResponse(
        product_id=result.product_id,
        calendar_date=result.calendar_date,
        days_until_travel=result.days_until_travel,
        recommended_adjustment_pct=result.recommended_adjustment_pct,
        calculated_at=result.calculated_at,
        booked_units=result.booked_units,
        total_capacity=result.total_capacity,
        occupancy=result.occupancy,
        current_prices=result.current_prices,
    )