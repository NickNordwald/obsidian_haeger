---
tags:
  - documentation
---

Syntax-Referenz zum Nachschlagen für den **FastAPI + Uvicorn + Pydantic**-Stack — analog zum [[SQLAlchemy Tutorial|SQLAlchemy Tutorial]] aus Kapitel 1.2. Die **Konzepte und Zusammenhänge** (was die drei Bibliotheken sind, WSGI vs. ASGI, wo sie in der Schichtenarchitektur liegen) stehen in der [[1.5 REST APIs & Layered Architecture - Vertiefung#FastAPI + Uvicorn + Pydantic in Python|Vertiefung]]; hier geht es nur um **wie schreibt man das**.

> [!info] Wie diese Notiz gedacht ist
> Zum **Nachschlagen**, nicht zum Durchlesen. Die Reihenfolge folgt dem Weg einer Anfrage: erst die App und der Server, dann Endpunkte und Parameter, dann Modelle, dann alles, was drumherum gehört. Die Beispiele sind **Syntax-Muster**, keine fertige Lösung — die konkrete Umsetzung schreibst du selbst (vgl. [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]]).
>
> Alles hier ist **Pydantic v2** und **FastAPI ≥ 0.100**. Ältere Beispiele im Netz (`@validator`, `.dict()`, `class Config`) sind v1 und laufen so nicht.

# Inhalt

- [[#0. Installation & Start]]
- [[#1. Die App und der erste Endpunkt]]
- [[#2. Parameter - Pfad, Query, Body, Header]]
- [[#3. Pydantic-Modelle]]
- [[#4. Response-Modelle und Statuscodes]]
- [[#5. Validierung - Field, Constraints, eigene Validatoren]]
- [[#6. Router - mehrere Controller]]
- [[#7. Dependencies (`Depends`)]]
- [[#8. Fehlerbehandlung]]
- [[#9. Lifespan, Middleware, CORS]]
- [[#10. Konfiguration mit pydantic-settings]]
- [[#11. Testen]]
- [[#12. Spickzettel]]

---

# 0. Installation & Start

```bash
uv add fastapi uvicorn[standard] pydantic-settings   # bzw. pip install
```

`uvicorn[standard]` zieht `uvloop` und `httptools` mit (die schnellen C-Implementierungen). `pydantic` kommt als Abhängigkeit von FastAPI automatisch mit.

```bash
uvicorn pricing.api.main:app --reload
#       └────────┬────────┘ └┬┘  └──┬──┘
#            Modulpfad     Objekt  nur Entwicklung
```

| Option | Wofür |
|---|---|
| `--reload` | Neustart bei Dateiänderung — **nur Entwicklung** |
| `--host 0.0.0.0` | von außen erreichbar (Default: nur `127.0.0.1`) |
| `--port 8000` | Port (Default 8000) |
| `--workers 4` | mehrere Prozesse — **nicht** zusammen mit `--reload` |
| `--log-level debug` | Uvicorns eigenes Logging |

Alternativ aus Python heraus starten (praktisch für `python -m`-Aufrufe):

```python
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("pricing.api.main:app", reload=True)
```

---

# 1. Die App und der erste Endpunkt

```python
from fastapi import FastAPI

app = FastAPI(
    title="Pricing API",
    description="REST-Schnittstelle zur Preisempfehlung",
    version="0.1.0",
)

@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

- Der **Dekorator** bestimmt Methode und Pfad: `@app.get`, `@app.post`, `@app.put`, `@app.patch`, `@app.delete`.
- Die Funktion heißt **Path Operation Function**; ihr Name ist frei (er landet aber als `operationId` in der OpenAPI-Doku).
- Rückgabewerte werden automatisch zu JSON serialisiert (`dict`, `list`, Pydantic-Modelle, `dataclass`).
- `title`/`version` erscheinen in der Swagger UI unter `/docs`.

> [!tip] `def` oder `async def`?
> **`def`** → FastAPI führt die Funktion in einem Threadpool aus; blockierender Code (synchrones SQLAlchemy!) ist unschädlich.
> **`async def`** → läuft direkt im Event Loop; darin **darf nichts blockieren**.
> Für dieses Projekt: **`def`**, solange die Adapter synchron sind.

---

# 2. Parameter — Pfad, Query, Body, Header

FastAPI entscheidet **anhand der Typannotation und des Pfads**, woher ein Parameter kommt:

| Parameter …                                          | wird gelesen aus |
| ---------------------------------------------------- | ---------------- |
| taucht im Pfad-String auf                            | **Pfad**         |
| ist ein einfacher Typ (`int`, `str`, `bool`, `date`) | **Query**        |
| ist ein Pydantic-Modell                              | **Body**         |
| ist explizit annotiert (`Header`, `Cookie`, `Form`)  | entsprechend     |
|                                                      |                  |

```python
from datetime import date
from typing import Annotated
from fastapi import FastAPI, Path, Query, Header

@app.get("/bookings/{booking_id}")
def get_booking(
    booking_id: Annotated[int, Path(ge=1, description="ID der Buchung")],
    verbose: bool = False,                                    # Query, optional
    product_id: Annotated[str | None, Query(max_length=20)] = None,
    user_agent: Annotated[str | None, Header()] = None,
):
    ...
```

- **Ohne Defaultwert = Pflichtparameter**, mit Defaultwert = optional.
- `Annotated[T, Path(...) | Query(...)]` ist der **moderne Stil** (seit FastAPI 0.95). Ältere Beispiele schreiben `q: str = Query(None)` — funktioniert noch, ist aber überholt.
- Validierungsregeln direkt am Parameter: `ge`, `gt`, `le`, `lt`, `min_length`, `max_length`, `pattern`.
- Typkonvertierung passiert automatisch: `/bookings/42` → `booking_id` ist ein echtes `int`, `?from=2026-08-01` → `date`.

**Mehrfach-Query-Parameter** (`?tag=a&tag=b`):

```python
def list_items(tag: Annotated[list[str], Query()] = []): ...
```

**Body** ist einfach ein Pydantic-Modell als Parameter:

```python
@app.post("/bookings")
def create_booking(payload: BookingCreate): ...
```

---

# 3. Pydantic-Modelle

Die zentrale Bauform. Eine Klasse, die von `BaseModel` erbt, beschreibt eine Datenstruktur samt Validierung:

```python
from datetime import date
from decimal import Decimal
from pydantic import BaseModel

class BookingCreate(BaseModel):
    product_id: str
    start_date: date
    nights: int
    status: str = "open"          # Defaultwert → optionales Feld
```

| Schreibweise | Bedeutung |
|---|---|
| `x: int` | Pflichtfeld |
| `x: int = 3` | optional mit Default |
| `x: int \| None = None` | optional, darf `null` sein |
| `x: list[str] = []` | Liste (Pydantic kopiert den Default korrekt) |
| `x: Literal["open", "cancelled"]` | nur diese Werte erlaubt |
| `x: SubModel` | verschachteltes Modell |

**Instanziieren und umwandeln:**

```python
b = BookingCreate(product_id="A1", start_date="2026-08-01", nights=3)
b = BookingCreate.model_validate(some_dict)      # aus dict/JSON-kompatiblen Daten
b = BookingCreate.model_validate_json(raw_json)  # direkt aus einem JSON-String

b.model_dump()                 # → dict
b.model_dump(mode="json")      # → dict mit JSON-tauglichen Werten (date → str)
b.model_dump_json()            # → JSON-String
b.model_dump(exclude_none=True, exclude={"status"})
b.model_copy(update={"nights": 5})
```

> [!warning] Pydantic v1 → v2
> `.dict()` → `.model_dump()` · `.json()` → `.model_dump_json()` · `parse_obj()` → `model_validate()` · `@validator` → `@field_validator` · `class Config` → `model_config`.

**Aus ORM-Objekten lesen** (Attribute statt dict) — wichtig für den Rückweg aus SQLAlchemy:

```python
from pydantic import BaseModel, ConfigDict

class BookingRead(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    product_id: str
    start_date: date
    nights: int

BookingRead.model_validate(orm_obj)   # liest orm_obj.id, orm_obj.product_id, …
```

---

# 4. Response-Modelle und Statuscodes

```python
@app.post("/bookings", response_model=BookingRead, status_code=201)
def create_booking(payload: BookingCreate) -> BookingRead:
    ...
```

- **`response_model`** filtert die Antwort auf die dort definierten Felder — das ist auch ein **Sicherheitsmerkmal**: Felder, die im Modell nicht stehen, können nicht versehentlich nach außen gelangen.
- Der **Rückgabetyp in der Signatur** (`-> BookingRead`) tut inzwischen dasselbe; `response_model` braucht man nur noch, wenn beides abweichen soll.
- `status_code=201` setzt den Erfolgs-Statuscode. Lesbarer über die Konstanten:

```python
from fastapi import status
@app.delete("/bookings/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_booking(id: int) -> None: ...
```

**Listen** als Rückgabetyp: `-> list[BookingRead]`.

**Header setzen oder Statuscode zur Laufzeit ändern:**

```python
from fastapi import Response

@app.post("/bookings", status_code=201)
def create_booking(payload: BookingCreate, response: Response) -> BookingRead:
    created = ...
    response.headers["Location"] = f"/bookings/{created.id}"
    return created
```

---

# 5. Validierung — Field, Constraints, eigene Validatoren

```python
from pydantic import BaseModel, Field

class BookingCreate(BaseModel):
    product_id: str = Field(min_length=1, max_length=20)
    nights: int = Field(gt=0, le=365, description="Anzahl Nächte")
    price: Decimal = Field(ge=0, decimal_places=2)
```

Häufige Constraints: `gt` `ge` `lt` `le` (Zahlen), `min_length` `max_length` `pattern` (Strings), `min_length`/`max_length` (Listen), `decimal_places`/`max_digits`.

**Eigene Feldvalidierung:**

```python
from pydantic import field_validator

class BookingCreate(BaseModel):
    product_id: str

    @field_validator("product_id")
    @classmethod
    def upper(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError("product_id muss alphanumerisch sein")
        return v.upper()
```

**Validierung über mehrere Felder hinweg:**

```python
from pydantic import model_validator

class DateRange(BaseModel):
    start: date
    end: date

    @model_validator(mode="after")
    def check_order(self):
        if self.end < self.start:
            raise ValueError("end muss nach start liegen")
        return self
```

- Ein `ValueError` oder `AssertionError` im Validator wird von FastAPI automatisch zu einem **`422`** mit strukturierter Fehlermeldung (Feldpfad + Grund).
- `mode="before"` läuft **vor** der Typkonvertierung (Rohwert), `mode="after"` danach (typisiertes Objekt).

**Unbekannte Felder ablehnen** (sinnvoll für Request-Modelle — schützt vor Tippfehlern im Client):

```python
model_config = ConfigDict(extra="forbid")   # Default ist "ignore"
```

---

# 6. Router — mehrere Controller

```python
# api/routers/bookings.py
from fastapi import APIRouter

router = APIRouter(prefix="/bookings", tags=["bookings"])

@router.get("")                    # → GET /bookings
def list_bookings(): ...

@router.get("/{booking_id}")       # → GET /bookings/42
def get_booking(booking_id: int): ...
```

```python
# api/main.py
from fastapi import FastAPI
from .routers import bookings, prices, recommendations

app = FastAPI(title="Pricing API")
app.include_router(bookings.router)
app.include_router(prices.router)
app.include_router(recommendations.router)
```

- `prefix` darf **nicht** auf `/` enden.
- `tags` steuert die Gruppierung in der Swagger UI.
- `APIRouter(dependencies=[Depends(...)])` hängt eine Abhängigkeit an **alle** Endpunkte des Routers (z. B. Authentifizierung).
- Verschachtelung ist möglich: Ein Router kann per `include_router` weitere Router einbinden (typisch für `/v1`).

---

# 7. Dependencies (`Depends`)

FastAPIs Dependency Injection. Eine Funktion wird zur wiederverwendbaren Abhängigkeit, die FastAPI selbst auflöst:

```python
from typing import Annotated
from fastapi import Depends
from sqlalchemy.orm import Session

def get_session() -> Iterator[Session]:
    with SessionLocal() as session:      # yield → Aufräumen nach dem Request
        yield session

SessionDep = Annotated[Session, Depends(get_session)]

@router.get("/{booking_id}")
def get_booking(booking_id: int, session: SessionDep): ...
```

- Alles **vor** dem `yield` läuft vor dem Handler, alles **danach** nach der Antwort — das ist die Stelle für Sessions, Transaktionen, Aufräumarbeiten.
- Dependencies können selbst Dependencies haben (Verkettung) und werden pro Request **gecacht** (`use_cache=False` schaltet das ab).
- Der `Annotated`-Alias (`SessionDep`) erspart Wiederholung in jedem Handler — der empfohlene Stil.

**In Tests ersetzen** (der eigentliche Gewinn, direkter Bezug zu [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]]):

```python
app.dependency_overrides[get_session] = lambda: fake_session
```

---

# 8. Fehlerbehandlung

**Punktuell im Handler:**

```python
from fastapi import HTTPException, status

if booking is None:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail=f"Buchung {booking_id} nicht gefunden",
    )
```

**Zentral für eigene Exceptions** — der richtige Ort für die Zuordnung aus [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]]:

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from pricing.exceptions import NotFoundError, PricingError

@app.exception_handler(NotFoundError)
def handle_not_found(request: Request, exc: NotFoundError):
    return JSONResponse(status_code=404, content={"detail": str(exc)})

@app.exception_handler(PricingError)
def handle_pricing_error(request: Request, exc: PricingError):
    logger.exception("Unerwarteter Fehler bei %s", request.url.path)
    return JSONResponse(status_code=500, content={"detail": "Interner Fehler"})
```

So bleibt die Zuordnung an **einer** Stelle, und die Handler beschreiben nur den Erfolgsfall.

---

# 9. Lifespan, Middleware, CORS

**Lifespan** — Code beim Start und beim Herunterfahren (Engine anlegen, Verbindungen schließen):

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.engine = create_engine(settings.database_url)   # Startup
    yield
    app.state.engine.dispose()                                # Shutdown

app = FastAPI(lifespan=lifespan)
```

(Die älteren Dekoratoren `@app.on_event("startup")` sind veraltet.)

**Middleware** — läuft um *jede* Anfrage herum; typisch für Timing, Request-IDs, Logging:

```python
import time, uuid

@app.middleware("http")
async def add_request_id(request: Request, call_next):
    request_id = str(uuid.uuid4())
    start = time.perf_counter()
    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id
    response.headers["X-Process-Time"] = f"{time.perf_counter() - start:.4f}"
    return response
```

**CORS** — nötig, sobald ein Browser-Frontend von einer anderen Herkunft zugreift:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

# 10. Konfiguration mit pydantic-settings

Ersetzt handgeschriebenes `os.getenv` und prüft beim **Start** statt beim ersten Zugriff:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_prefix="PRICING_")

    database_url: str
    log_level: str = "INFO"
    default_page_size: int = 50

settings = Settings()      # liest PRICING_DATABASE_URL usw.
```

Fehlt eine Pflichtvariable, scheitert der Prozess **sofort beim Start** mit klarer Meldung — statt mitten in der ersten Anfrage.

---

# 11. Testen

```python
from fastapi.testclient import TestClient
from pricing.api.main import app

client = TestClient(app)

def test_health():
    r = client.get("/health")
    assert r.status_code == 200
    assert r.json() == {"status": "ok"}

def test_create_booking():
    r = client.post("/bookings", json={"product_id": "A1", "nights": 3,
                                       "start_date": "2026-08-01"})
    assert r.status_code == 201
    assert r.headers["location"].startswith("/bookings/")
```

- `TestClient` startet **keinen** Server — er ruft die App direkt über ASGI auf (intern `httpx`). Deshalb schnell genug für Unit-Test-Geschwindigkeit.
- Zusammen mit `dependency_overrides` (siehe [[#7. Dependencies (`Depends`)]]) laufen API-Tests komplett ohne Datenbank.
- Als pytest-Fixture ist `client` ein natürlicher Kandidat für `conftest.py`.

---

# 12. Spickzettel

| Ich will … | Syntax |
|---|---|
| Endpunkt registrieren | `@app.get("/pfad")` / `@router.post("")` |
| Pfadparameter | `/{id}` + `id: int` im Handler |
| Query-Parameter | einfacher Typ als Funktionsparameter |
| Body | Pydantic-Modell als Parameter |
| Statuscode setzen | `@app.post(..., status_code=201)` |
| Antwort filtern | `response_model=` oder `-> Modell` |
| Fehler werfen | `raise HTTPException(404, "…")` |
| Eigene Exception abbilden | `@app.exception_handler(MeinFehler)` |
| Abhängigkeit einhängen | `Annotated[T, Depends(func)]` |
| Router einbinden | `app.include_router(r)` |
| Start/Shutdown | `lifespan=` |
| Modell aus ORM-Objekt | `ConfigDict(from_attributes=True)` |
| Modell → dict | `.model_dump()` |
| Feld einschränken | `Field(gt=0, max_length=20)` |
| Eigene Prüfung | `@field_validator` / `@model_validator` |
| Konfiguration | `BaseSettings` |
| Testen | `TestClient(app)` |
| Doku ansehen | `/docs` (Swagger) · `/redoc` · `/openapi.json` |

# Weiterführend

- [FastAPI Tutorial / User Guide](https://fastapi.tiangolo.com/tutorial/) — die offizielle Doku ist gleichzeitig das beste Tutorial
- [Pydantic Concepts](https://docs.pydantic.dev/latest/concepts/models/) — Modelle, Validatoren, Serialisierung
- [Uvicorn Deployment](https://www.uvicorn.org/deployment/) — Worker, Reverse Proxy, Produktion
- Weitere Quellen: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]], Abschnitt *FastAPI, Pydantic & OpenAPI*
