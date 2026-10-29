"""
Uvicorn entrypoint for local development.

Run with:
    python -m src.main

Or directly with uvicorn (equivalent, and what you'd use in production):
    uvicorn src.api:app --reload
"""

import uvicorn


if __name__ == "__main__":
    uvicorn.run(
        "src.api:app",
        host="127.0.0.1",
        port=8000,
        reload=True,
    )