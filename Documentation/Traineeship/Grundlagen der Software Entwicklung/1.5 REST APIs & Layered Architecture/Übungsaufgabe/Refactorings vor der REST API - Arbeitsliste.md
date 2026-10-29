---
tags:
  - documentation
type: tutorial
date: 2026-08-04
---

Abarbeitbare Liste der Refactorings, die **vor** dem Bau der REST API sinnvoll sind — Ergebnis des Code-Scans vom 2026-08-04. Begleitnotiz zum [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]].

> [!info] Arbeitsmodus für diese Liste
> Ausnahmsweise **mit Kopiervorlagen** (auf Wunsch). Die Blöcke sind so sortiert, dass nach jedem Block `ruff` / `mypy` / `pytest` grün sein müssen — wenn nicht, hör auf und schau nach, bevor du weitermachst.
>
> Nicht enthalten: die `TRUNCATE`-vs.-`computed_at`-Frage (bewusste Entscheidung, bleibt).

**Kontrolle nach jedem Block:**

```bash
./.venv/bin/ruff check . && ./.venv/bin/mypy && ./.venv/bin/python -m pytest -q
```

---

# Block 1 — `slots` auf die Domänen-Objekte

**Warum:** gemessen **344 B → 64 B** pro Objekt. Bei 1,33 Mio. `CapacityRow` + 1,33 Mio. `PriceableUnit` + 940k `Booking` sind das grob **900 MB → 200 MB** Spitzenverbrauch.

In `domain/entities.py` die vier Dataclass-Dekoratoren ergänzen:

```python
@dataclass(frozen=True, slots=True)      # CapacityRow
@dataclass(slots=True)                    # Booking
@dataclass(frozen=True, slots=True)      # Price
@dataclass(order=True, slots=True)       # PriceRecommendation
```

Und in `PriceableUnit` direkt unter die Klassenzeile:

```python
class PriceableUnit:
    """Diese Klasse bildet ein buchbare Einheit."""

    __slots__ = (
        "_bookable_units",
        "_booked_units",
        "_calendar_date",
        "_optional_units",
        "_product_id",
    )
```

> [!tip] Warum das kollisionsfrei ist
> Die Slots sind die `_`-Attribute, die Properties heißen ohne Unterstrich. `asdict()`, `fields()` und `order=True` funktionieren mit Slots unverändert. Nebeneffekt: Ein Tippfehler bei einer Zuweisung wirft jetzt `AttributeError`, statt still ein neues Attribut anzulegen.

- [x] Block 1 erledigt, Checks grün

---

# Block 2 — Builder: 1,33-Mio.-Set für eine Warnung einsparen

**Warum:** `capacity_keys` ist ein zweites Set über alle Kapazitätszeilen, das nur der Diagnose dient. Die Zahl lässt sich beim Bauen mitzählen.

In `domain/builder.py` den Abschnitt von `priceable_units = [` bis `orphan_nights = sum(...)` ersetzen:

```python
        # Zaehlt beim Bauen mit, wie viele Firm-Naechte eine Kapazitaetszeile
        # gefunden haben -- spart ein zweites Set ueber alle Kapazitaetszeilen.
        # Korrekt, weil (product_id, calendar_date) in capacity eindeutig ist.
        priceable_units: list[PriceableUnit] = []
        matched_nights = 0
        for row in capacity_rows:
            booked_units = booked[(row.product_id, row.calendar_date)]
            matched_nights += booked_units
            priceable_units.append(
                PriceableUnit(
                    product_id=row.product_id,
                    calendar_date=row.calendar_date,
                    bookable_units=row.bookable_units,
                    booked_units=booked_units,
                    optional_units=row.optional_units,
                )
            )

        neg = sum(1 for u in priceable_units if u.has_negative_available_capacity)
        if neg:
            logger.warning("%d Einheiten mit negativer Kapazität", neg)

        orphan_nights = sum(booked.values()) - matched_nights
        if orphan_nights:
            logger.warning(
                "%d Firm-Buchungsnächte ohne passende Kapazitätszeile — verworfen.",
                orphan_nights,
            )

        return priceable_units
```

- [x] Block 2 erledigt, Checks grün

---

# Block 3 — `load_prices()` entfernen ⛔ *bewusst übersprungen (2026-08-04)*

> [!info] Entscheidung: bleibt drin
> Nick behält `load_prices()` vorerst — bewusst, nicht aus Versehen. Die API wird Preise voraussichtlich brauchen; dann ist der Lesepfad schon da. **Der Rest dieses Blocks ist damit gegenstandslos**, die Beobachtung bleibt zur Einordnung stehen.
>
> Der einzige verbleibende Nebeneffekt: `CsvDataSource.__init__` löst die Preis-CSV eifrig auf und scheitert, wenn sie fehlt — auch wenn niemand Preise liest. Falls das mal stört, ist die Auflösung faul zu machen (erst in `load_prices()`) die kleinere Änderung.

**Ursprüngliche Begründung:** Der Port deklariert es, beide Adapter implementieren es, **niemand ruft es auf**.

**`persistence/interfaces.py`** — Methode und Import streichen:

```python
from ..domain.entities import Booking, CapacityRow, PriceRecommendation


class DataSource(Protocol):
    "Interface einer Quelle, welche die Eingangsdaten einliest."

    def load_capacity(self) -> list[CapacityRow]: ...

    def load_bookings(self) -> list[Booking]: ...
```

**`persistence/sources.py`:**

- `load_prices()` aus **beiden** Klassen löschen
- in `CsvDataSource.__init__` die Zeile `self._prices_path = resolve_csv(data_dir, "prices")` löschen
- die dann unbenutzten Importe `Decimal` und `Price` entfernen

`domain/entities.py` behält `Price` — die API wird `GET /prices` vermutlich brauchen, dann aber über ein Repository statt über `DataSource`.

- [x] Block 3 erledigt, Checks grün

---

# Block 4 — Docstrings, die inzwischen lügen

**`persistence/interfaces.py`** — der aktuelle Text begründet ausführlich die *entgegengesetzte* Architekturentscheidung. Ersetzen durch:

```python
"""
Vertraege der Persistenzschicht.

Diese Protocols definieren, *was* die Anwendung von der Datenhaltung braucht,
ohne das *Wie* festzulegen. Sie gehoeren bewusst in die Persistenzschicht
(klassische Schichtung): Die Abhaengigkeit zeigt nach unten, die Application-
Schicht importiert diese Vertraege. Die konkreten Implementierungen stehen
in sources.py und writers.py.
"""
```

**`persistence/sources.py`:**

```python
"""
Liest die Eingangsdaten und stellt sie als Domaenen-Objekte bereit.

Zwei Implementierungen des DataSource-Vertrags: aus CSV-Dateien und aus der
Datenbank (SQLAlchemy).
"""
```

**`persistence/writers.py`:**

```python
"""
Schreibt die berechneten Preisempfehlungen weg.

Zwei Implementierungen des RecommendationWriter-Vertrags: in eine CSV-Datei
und in die Datenbank (SQLAlchemy).
"""
```

**`pyproject.toml`:**

```toml
description = "Preisempfehlungs-Anwendung in Schichtenarchitektur"
```

- [x] Block 4 erledigt, Checks grün

---

# Block 5 — `PricingApp` entzerren

**Warum:** Drei überlappende öffentliche Methoden, zwei davon nur von Tests benutzt, jede ruft `_compute()` (42 s). Für die API braucht es den Schnitt zwischen *rechnen* und *schreiben*.

**`application/pricing_app.py`** — Kopf und die vier Methoden ersetzen:

```python
"""
Application-Schicht: orchestriert den Preislauf.
"""

import logging
import time
from dataclasses import dataclass

from ..domain.builder import PriceableUnitBuilder
from ..domain.entities import PriceRecommendation
from ..domain.recommendations import RecommendationEngine
from ..persistence.interfaces import DataSource, RecommendationWriter

logger = logging.getLogger(__name__)


@dataclass(frozen=True, slots=True)
class PricingRunResult:
    """Ergebnis *eines* Laufs -- damit der Aufrufer nicht neu rechnen muss."""

    recommendations: list[PriceRecommendation]
    max_recommendation: PriceRecommendation | None
    duration_seconds: float


class PricingApp:
    def __init__(
        self,
        source: DataSource,
        builder: PriceableUnitBuilder,
        engine: RecommendationEngine,
        writer: RecommendationWriter,
    ):
        self._source = source
        self._builder = builder
        self._engine = engine
        self._writer = writer

    def compute(self) -> list[PriceRecommendation]:
        """Rechnet einen Lauf durch. Schreibt nichts."""
        start = time.perf_counter()
        logger.info("Bookings und Capacity werden geladen")
        capacity = self._source.load_capacity()
        bookings = self._source.load_bookings()

        logger.info(
            "%d Kapazitaetszeilen und %d Buchungen geladen in %.2f s",
            len(capacity),
            len(bookings),
            time.perf_counter() - start,
        )

        units = self._builder.build(capacity, bookings)
        recs = self._engine.calculate_all(units)

        logger.info("%d Empfehlungen berechnet aus %d Einheiten", len(recs), len(units))
        return recs

    def run(self) -> PricingRunResult:
        """Rechnet und schreibt -- der vollstaendige Lauf."""
        start = time.perf_counter()
        logger.info("Preislauf gestartet")

        recs = self.compute()
        self._writer.write(recs)
        duration = time.perf_counter() - start

        logger.info(
            "Preislauf beendet: %d Empfehlungen in %.2f s", len(recs), duration
        )

        return PricingRunResult(
            recommendations=recs,
            max_recommendation=self._engine.get_max(recs),
            duration_seconds=duration,
        )
```

**`main.py`** — die Ausgabe anpassen:

```python
        result = app.run()
    except PricingError:
        ...

    print(f"Höchste Empfehlung: {result.max_recommendation}")
```

**`tests/test_app.py`** — vier Stellen:

Zeile 72/73:
```python
    assert result.max_recommendation is not None
    assert result.max_recommendation.product_id == "P1"
```

Zeile 100/101:
```python
    assert result.max_recommendation is not None
    assert result.max_recommendation.product_id == "P2"
```

Die beiden Tests `test_calculate_all_writes_but_returns_nothing` und `test_get_max_recommendation_does_not_write` komplett ersetzen durch:

```python
def test_compute_does_not_write(harness: AppHarness) -> None:
    """compute() rechnet nur -- Schreiben ist ein getrennter Schritt."""
    recs = harness.app.compute()

    assert len(recs) == 1
    assert recs[0].product_id == "P1"
    harness.writer.write.assert_not_called()


def test_run_result_carries_recs_and_duration(harness: AppHarness) -> None:
    """Ein Lauf beantwortet alle Fragen -- ohne ein zweites Mal zu rechnen."""
    result = harness.app.run()

    assert len(result.recommendations) == 1
    assert result.duration_seconds >= 0
```

**`scripts/smoke_orm.py`** — eigene Zeitmessung entfällt, das Ergebnis bringt sie mit:

```python
result = app.run()
print(f"Höchste Empfehlung (SQLAlchemy): {result.max_recommendation}")
print(f"Gesamtdauer: {result.duration_seconds:.2f} s")
engine.dispose()
```
(die Zeilen `start = time.perf_counter()`, `dauer = ...` und der `import time` können weg)

- [x] Block 5 erledigt, Checks grün

---

# Block 6 — Session statt Engine in den Adaptern ⭐

**Warum — die wichtigste Änderung der Liste:** Aktuell öffnet jeder Loader seine **eigene** Session (drei pro Lauf, plus eine im Writer). Für die API muss die Session **pro Request** existieren und von allen Beteiligten geteilt werden, sonst laufen Lesen und Schreiben eines `POST /pricing-runs` in verschiedenen Transaktionen. Damit beantwortest du zugleich die geparkte Frage *„wer besitzt die Transaktionsgrenze?"* — der Aufrufer, nicht der Adapter.

**`persistence/sources.py`** — Konstruktor und die drei `with`-Blöcke:

```python
from sqlalchemy import select
from sqlalchemy.orm import Session
# Engine-Import kann weg
```

```python
class SqlAlchemyDataSource:
    """Liest die Eingangsdaten per SQLAlchemy aus der Datenbank.

    Bekommt die Session von aussen -- die Lebensdauer gehoert dem Aufrufer
    (CLI: ein Lauf, API: ein Request). Selektiert einzelne Spalten statt
    ganzer ORM-Entitaeten, weil das Schema ein System of Record ist und weit
    mehr enthaelt, als das Pricing braucht.
    """

    def __init__(self, session: Session):
        self._session = session

    def load_capacity(self) -> list[CapacityRow]:
        stmt = select(
            orm.Capacity.product_id,
            orm.Capacity.calendar_date,
            orm.Capacity.bookable_units,
            orm.Capacity.optional_units,
        )
        return [
            CapacityRow(
                product_id=r.product_id,
                calendar_date=r.calendar_date,
                bookable_units=r.bookable_units,
                optional_units=r.optional_units,
            )
            for r in self._session.execute(stmt)
        ]

    def load_bookings(self) -> list[Booking]:
        stmt = select(
            orm.Booking.product_id,
            orm.Booking.start_date,
            orm.Booking.nights,
            orm.Booking.status,
        )
        return [
            Booking(
                product_id=r.product_id,
                start_date=r.start_date,
                nights=r.nights,
                status=r.status,
            )
            for r in self._session.execute(stmt)
        ]
```

**`scripts/smoke_orm.py`** — der Aufrufer besitzt jetzt die Session:

```python
from sqlalchemy.orm import Session

engine = get_engine(echo=False)

with Session(engine) as session:
    app = PricingApp(
        source=SqlAlchemyDataSource(session),
        builder=PriceableUnitBuilder(),
        engine=RecommendationEngine(
            HeuristicRecommendationStrategy(), reference_date=REF
        ),
        writer=SqlAlchemyRecommendationWriter(session),
    )
    result = app.run()
    session.commit()          # <- der Aufrufer committet

print(f"Höchste Empfehlung (SQLAlchemy): {result.max_recommendation}")
print(f"Gesamtdauer: {result.duration_seconds:.2f} s")
engine.dispose()
```

`scripts/smoke_orm_source.py` analog anpassen (falls du es behalten willst).

> [!important] Was das für die API bedeutet
> Später liefert eine `Depends`-Funktion die Session:
> ```python
> def get_session() -> Iterator[Session]:
>     with Session(engine) as s:
>         yield s
>         s.commit()
> ```
> und die Router bauen `PricingApp` pro Request. Das ist billig — es sind vier Objektkonstruktionen.

- [x] Block 6 erledigt, Checks grün

---

# Block 7 — Writer: Join in der Datenbank statt 1,33 Mio. Zeilen im Speicher

**Warum:** Der Writer lädt aktuell die **komplette** `capacity`-Tabelle in ein Python-dict (gemessen ~208 MB), nur um `(product_id, calendar_date) → capacity_id` aufzulösen. Der Join gehört in die Datenbank.

**Ablauf neu:** temporäre Tabelle → `COPY` hinein → `INSERT … SELECT` mit Join → fehlende Zeilen aus der Differenz der Zeilenzahlen ableiten.

Umgesetzt mit **SQLAlchemy Core** statt handgeschriebenem SQL: Die Temp-Tabelle ist ein echtes `Table`-Objekt, damit ist die COPY-Spaltenliste ableitbar (gleiche Drift-Sicherheit wie im Ingest) und die Join-Bedingung referenziert Modell-Attribute statt Strings.

**`persistence/writers.py`** — die SQLAlchemy-Variante vollständig:

```python
import csv
import logging
from dataclasses import asdict, fields
from pathlib import Path
from typing import cast

import psycopg
from psycopg import sql
from sqlalchemy import (
    Column,
    Date,
    Float,
    Integer,
    MetaData,
    Numeric,
    Table,
    Text,
    insert,
    select,
    text,
)
from sqlalchemy.orm import Session

from ..domain.entities import PriceRecommendation
from . import orm

logger = logging.getLogger(__name__)


# Eigene MetaData! Die Staging-Tabelle darf NICHT in Base.metadata landen --
# sonst wuerde pricing-initdb sie als permanente Tabelle mit anlegen.
_staging_metadata = MetaData()

_tmp_recs = Table(
    "tmp_recs",
    _staging_metadata,
    Column("product_id", Text, nullable=False),
    Column("calendar_date", Date, nullable=False),
    Column("days_until_travel", Integer, nullable=False),
    Column("recommended_adjustment_pct", Numeric(10, 6), nullable=False),
    Column("occupancy", Float, nullable=False),
    Column("total_capacity", Integer, nullable=False),
    Column("booked_units", Integer, nullable=False),
    prefixes=["TEMPORARY"],
    postgresql_on_commit="DROP",
)

# Aus dem Table-Objekt abgeleitet -- kann nicht abdriften.
_TMP_COLS = tuple(c.name for c in _tmp_recs.c)

# INSERT ... SELECT: der Join passiert in der Datenbank.
_INSERT_FROM_TMP = insert(orm.PriceRecommendation).from_select(
    [
        "capacity_id",
        "days_until_travel",
        "recommended_adjustment_pct",
        "occupancy",
        "total_capacity",
        "booked_units",
    ],
    select(
        orm.Capacity.id,
        _tmp_recs.c.days_until_travel,
        _tmp_recs.c.recommended_adjustment_pct,
        _tmp_recs.c.occupancy,
        _tmp_recs.c.total_capacity,
        _tmp_recs.c.booked_units,
    ).select_from(
        _tmp_recs.join(
            orm.Capacity,
            (orm.Capacity.product_id == _tmp_recs.c.product_id)
            & (orm.Capacity.calendar_date == _tmp_recs.c.calendar_date),
        )
    ),
)


class SqlAlchemyRecommendationWriter:
    """Schreibt die Empfehlungen in die Datenbank.

    Loest capacity_id per Join *in der Datenbank* auf, statt die gesamte
    capacity-Tabelle (1,33 Mio. Zeilen) in ein Python-dict zu laden.
    """

    def __init__(self, session: Session):
        self._session = session

    def write(self, recs: list[PriceRecommendation]) -> None:
        session = self._session
        conn = session.connection()

        _tmp_recs.create(bind=conn)

        raw = cast(psycopg.Connection, conn.connection.driver_connection)
        copy_stmt = sql.SQL("COPY {tbl} ({cols}) FROM STDIN").format(
            tbl=sql.Identifier(_tmp_recs.name),
            cols=sql.SQL(", ").join(map(sql.Identifier, _TMP_COLS)),
        )
        with raw.cursor() as cur, cur.copy(copy_stmt) as copy:
            for r in recs:
                copy.write_row([getattr(r, c) for c in _TMP_COLS])

        # TRUNCATE hat kein Core-Konstrukt. delete(orm.PriceRecommendation) waere
        # nativ, ist bei einer Volltabelle aber deutlich langsamer und setzt die
        # IDENTITY-Sequenz nicht zurueck.
        conn.execute(text("TRUNCATE price_recommendations RESTART IDENTITY"))
        conn.execute(_INSERT_FROM_TMP)

        # NICHT .rowcount benutzen -- siehe Kasten unten.
        inserted = conn.execute(
            select(func.count()).select_from(orm.PriceRecommendation)
        ).scalar_one()
        logger.info("%d Empfehlungen in price_recommendations geschrieben", inserted)

        missing = len(recs) - inserted
        if missing:
            logger.warning(
                "%d Recommendations ohne passende capacity-Zeile übersprungen.",
                missing,
            )
```

> [!warning] Vier Details, die leicht schiefgehen
> - **Eigene `MetaData`.** Läge `tmp_recs` in `Base.metadata`, würde `pricing-initdb` sie als *permanente* Tabelle anlegen. Das ist der gefährlichste Fehler in diesem Block.
> - **`ON COMMIT DROP`** setzt eine laufende Transaktion voraus — die hat die Session (Block 6). Ein zweites `write()` in **derselben** Transaktion scheitert, weil die Tabelle noch existiert; das ist gewollt, denn wegen des `TRUNCATE` wäre das ohnehin ein Fehler.
> - **`conn.execute(...)` statt `session.execute(...)`** — `Session.execute()` liefert laut Stubs ein `Result` ohne `rowcount`; nur `Connection.execute()` gibt ein `CursorResult`. (Für den Zähl-Aufruf spielt es keine Rolle, für die Einheitlichkeit schon.)

> [!bug] Warum hier **nicht** `.rowcount` — echter Fehler aus dem ersten Lauf
> Die naheliegende Fassung `inserted = conn.execute(_INSERT_FROM_TMP).rowcount` erzeugte die Meldung *„98233 Recommendations ohne passende capacity-Zeile übersprungen"* — bei **98232** berechneten Empfehlungen. Arithmetisch unmöglich, und genau dieses Eins-zu-viel war der Beweis: `rowcount` war **`-1`**.
>
> `-1` ist in PEP 249 das Sentinel für **„Zeilenzahl nicht ermittelbar"** — ausdrücklich *nicht* „null Zeilen". SQLAlchemys Dokumentation dazu:
> > *The primary purpose of 'rowcount' is to report the number of rows matched by the WHERE criterion of an **UPDATE or DELETE** statement […] In order to retrieve `cursor.rowcount` for [INSERT] statements, set the `preserve_rowcount` execution option to True […] the returned value will be `-1`* [sonst].
>
> Der Mechanismus: Ein `INSERT` ohne `RETURNING` liefert keine Zeilen, also schließt SQLAlchemy den Cursor sofort und merkt sich den `rowcount` nur bei UPDATE/DELETE.
>
> **Wichtig:** Das `INSERT` selbst war nie kaputt — nur die Messung. Deshalb zählt der Code jetzt nach, statt abzuleiten. Zulässig, weil unmittelbar davor `TRUNCATE` lief: Die Tabelle enthält exakt das, was dieser Lauf eingefügt hat. Die Logzeile ist damit eine **Beobachtung** statt einer Schlussfolgerung.
>
> *Alternative, die auch ginge:* `conn.execution_options(preserve_rowcount=True).execute(...).rowcount` — idiomatischer, spart die Zusatzabfrage, hängt aber an DBAPI-Eigenheiten.
>
> **Lehre:** Sentinel-Werte (`-1`, `None`) nie ungeprüft in Arithmetik ziehen — sie werfen keine Ausnahme, sondern liefern stille Unsinnszahlen. Und beim Debuggen: Behauptet eine Kennzahl etwas Unmögliches, verdächtige zuerst die Kennzahl.

- [x] Block 7 erledigt, Checks grün · smoke_orm läuft, Ergebnis identisch zu vorher

---

# Block 8 — Konfiguration bündeln + `-v`/`-vv` überall

**Warum:** `db.py` hat Host/DB/User hartkodiert, `logging_setup` liest heimlich `os.getenv`, und `load_dotenv()` steht in drei Entrypoints. Ein Settings-Objekt räumt das zusammen — und scheitert **beim Start** statt bei der ersten Anfrage.

## 8a — Abhängigkeit

`pyproject.toml`:

```toml
dependencies = ["pydantic-settings>=2.0"]
```
(der Kommentar „Kernlösung braucht keine Runtime-Dependencies" stimmt damit nicht mehr — anpassen oder löschen)

**Und im selben Zug das mypy-Plugin aktivieren:**

```toml
[tool.mypy]
...
plugins = ["pydantic.mypy"]
```

> [!warning] Ohne diese Zeile scheitert Block 8 an mypy
> `db_password` hat keinen Default → für mypy ist es ein **Pflichtargument im Konstruktor**, und `Settings()` ohne Argumente wirft:
> ```
> error: Missing named argument "db_password" for "Settings"  [call-arg]
> ```
> Zur Laufzeit ist der Aufruf korrekt — `BaseSettings` holt den Wert aus der Umgebung —, aber das kann mypy nicht wissen. Das Plugin bringt es ihm bei. Die Alternative wäre ein `# type: ignore[call-arg]`, also das Symptom zu unterdrücken.
>
> Nebengewinn: Das Plugin prüft auch normale Pydantic-Modelle strenger (Feldnamen und Typen beim Erzeugen) — das zahlt sich bei den API-DTOs aus.

> [!warning] Pylance meckert trotzdem weiter
> **Pylance ist nicht mypy** — es basiert auf Pyright und liest **keine mypy-Plugins**. Der Eintrag oben beruhigt also nur das Terminal, nicht den Editor.
>
> Kleinste Lösung, eine Zeile in `get_settings()`:
> ```python
> return Settings()  # pyright: ignore[reportCallIssue]
> ```
> **Nicht** `# type: ignore[call-arg]` nehmen: mypy hat den Fehler dank Plugin nicht mehr, und `strict = true` enthält `warn_unused_ignores` — du bekämst prompt *„unused type: ignore comment"*. Ein `# pyright: ignore` ist für mypy dagegen nur ein Kommentar.
>
> **Strukturelle Alternative**, falls die Doppelprüfung öfter stört: In `.vscode/settings.json` `"python.analysis.typeCheckingMode": "off"` setzen und die Erweiterung **Mypy Type Checker** (`ms-python.mypy-type-checker`) installieren. Dann gibt es genau *eine* Prüfinstanz — dieselbe, die in der CI läuft — und Pylance kümmert sich nur noch um Autovervollständigung und Navigation. Beim API-Bau (SQLAlchemy-Konstrukte, FastAPI-Dekoratoren) sind sich die beiden Prüfer regelmäßig uneinig; dann lohnt sich das.

```bash
./.venv/bin/python -m pip install -e ".[dev,db,orm]"
```

## 8b — `shared/config.py` (neu)

```python
"""Konfiguration aus Umgebung/.env -- eine Quelle fuer alle Entrypoints."""

from functools import lru_cache

from pydantic import Field, ValidationError
from pydantic_settings import BaseSettings, SettingsConfigDict

from .exceptions import ConfigError


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    db_host: str = "localhost"
    db_name: str = "pricing"
    db_user: str = "pricing"
    # Optional, obwohl faktisch Pflicht fuer jeden DB-Zugriff -- siehe Kasten unten.
    db_password: str | None = Field(default=None, validation_alias="DB_PRICING_PW")

    log_file: str | None = None
    log_level: str = "INFO"


@lru_cache
def get_settings() -> Settings:
    """Einmal lesen, danach gecacht. Uebersetzt Pydantic-Fehler in ConfigError."""
    try:
        return Settings()
    except ValidationError as e:
        fehlend = ", ".join(str(err["loc"][0]) for err in e.errors())
        raise ConfigError("Konfiguration", fehlend) from e
```

> [!tip] Wie die Umgebungsvariablen gefunden werden
> **Der Feldname ist der Suchschlüssel** — groß-/kleinschreibungsunabhängig. `log_file` sucht also `LOG_FILE`, `log_level` sucht `LOG_LEVEL`. Das `= None` ist nur der **Rückfallwert**, falls nichts gefunden wird; es verhindert das Einlesen nicht, sondern macht das Feld optional.
>
> Beim Passwort passt der Feldname nicht: `db_password` würde `DB_PASSWORD` suchen, die `.env` hat aber `DB_PRICING_PW`. Deshalb der `validation_alias`. Und weil dort **kein Default** steht, ist das Feld **Pflicht** — fehlt es, gibt es eine `ValidationError`, die `get_settings()` in `ConfigError` übersetzt.
>
> **Rangfolge der Quellen** (stark → schwach): Argumente an `Settings(...)` → echte Umgebungsvariablen → `.env`-Datei → Default im Feld. Dass echte Variablen die `.env` überstimmen, ist in Docker/CI praktisch: Dort setzt man Variablen, und eine mitkopierte `.env` stört nicht.

## 8c — `shared/cli.py` (neu)

```python
"""Gemeinsame CLI-Bausteine fuer die Entrypoints."""

import argparse

# Von leise nach laut. -v verschiebt um eine Stufe nach rechts.
_LADDER = ("ERROR", "WARNING", "INFO", "DEBUG")


def parse_verbosity(
    prog: str, argv: list[str] | None, default_level: str = "WARNING"
) -> str:
    """Wertet -v/-vv aus und liefert das Konsolen-Loglevel.

    Die Verbosity wirkt *relativ* zum Default des jeweiligen Kommandos, damit
    -v ueberall etwas bewirkt -- auch dort, wo schon INFO die Voreinstellung ist.
    """
    parser = argparse.ArgumentParser(prog=prog)
    parser.add_argument(
        "-v",
        "--verbose",
        action="count",
        default=0,
        help="mehr Ausgabe; mehrfach angebbar (-vv)",
    )
    args = parser.parse_args(argv)

    idx = _LADDER.index(default_level.upper())
    return _LADDER[min(idx + args.verbose, len(_LADDER) - 1)]
```

> [!tip] Warum relativ statt fest zugeordnet
> Eine feste Tabelle (`0→WARNING, 1→INFO, 2→DEBUG`) macht `-v` bei den Admin-Kommandos wirkungslos, weil die ohnehin auf `INFO` starten. Mit der Leiter ergibt sich:
>
> | Aufruf | `pricing` (Default `WARNING`) | `pricing-ingest` / `-initdb` (Default `INFO`) |
> |---|---|---|
> | ohne Flag | `WARNING` | `INFO` |
> | `-v` | `INFO` | `DEBUG` |
> | `-vv` | `DEBUG` | `DEBUG` |
> | `-vvv` | `DEBUG` | `DEBUG` |
>
> `min(...)` deckelt am oberen Ende — `-vvvv` läuft nicht ins Leere.

## 8d — `shared/logging_setup.py`

Signatur erweitern und die `os.getenv`-Aufrufe entfernen:

```python
def setup_logging(
    console_level: str | None = None,
    file_level: str | None = None,
    log_file: str | None = None,
) -> None:

    console_level = (console_level or "WARNING").upper()
    file_level = (file_level or "INFO").upper()
```

und in der Handler-Erzeugung `if log_file:` statt `if log_file := os.getenv("LOG_FILE"):`. Der `import os` kann weg.

> [!warning] Warum das zwingend zusammengehört
> `pydantic-settings` liest die `.env` in das **Settings-Objekt**, nicht in `os.environ`. Sobald `load_dotenv()` verschwindet, liefert `os.getenv("LOG_FILE")` also `None`. Deshalb bekommt `setup_logging` die Werte jetzt übergeben — was es nebenbei ohne Umgebung testbar macht.

## 8e — `persistence/db.py`

> [!warning] Warum `db_password` optional ist — die Falle, die sonst zuschlägt
> Naheliegend wäre `db_password: str = Field(validation_alias=...)` **ohne** Default: Pflichtfeld, scheitert beim Start. Genau das bricht aber `pricing`.
>
> `main()` ruft `get_settings()` eifrig auf, noch vor dem `try`-Block. Der CSV-Lauf liest aber CSV und schreibt CSV — er **öffnet nie eine Datenbankverbindung**. Er würde also an einem Passwort scheitern, das er gar nicht braucht, und zwar ohne saubere Fehlerbehandlung. (Symptom: `test_abbruch_loggt_mit_traceback` schlägt fehl.)
>
> **Der Grundsatz „beim Start scheitern" gilt für die Werte, die *dieser Prozess* braucht.** Das Passwort ist eine Anforderung der **Datenbankschicht** — also fordert `get_engine()` es ein. Für `pricing-ingest` und `pricing-initdb` ändert das nichts: Beide rufen `get_engine()` als erste Handlung im `try` auf und scheitern weiterhin sofort, nur eben mit sauberem Exit-Code.
>
> *Die Alternative wäre, die Einstellungen aufzuteilen (`Settings` fürs Logging, `DbSettings` für die Datenbank). Konzeptionell sauberer, aber zwei Klassen und zwei Caches für sechs Werte — bei dieser Größe überwiegt die Einfachheit.*

```python
from sqlalchemy import URL, Engine, create_engine

from ..shared.config import get_settings
from ..shared.exceptions import ConfigError


def get_engine(echo: bool = False) -> Engine:
    s = get_settings()
    if s.db_password is None:
        raise ConfigError("Umgebungsvariable", "DB_PRICING_PW")
    url = URL.create(
        "postgresql+psycopg",
        username=s.db_user,
        host=s.db_host,
        database=s.db_name,
        password=s.db_password,
    )
    return create_engine(url, echo=echo)
```
(`_require_password`, `import os` und der `ConfigError`-Import entfallen)

## 8f — Die drei Entrypoints

**`main.py`** (Default `WARNING`):

```python
from .shared.cli import parse_verbosity
from .shared.config import get_settings
# load_dotenv und argparse-Import entfallen


def main(argv: list[str] | None = None) -> int:
    console_level = parse_verbosity("pricing", argv, default_level="WARNING")
    settings = get_settings()
    setup_logging(
        console_level=console_level,
        file_level=settings.log_level,
        log_file=settings.log_file,
    )
    ...
```
(`_CONSOLE_LEVELS` in `main.py` kann weg)

**`persistence/ingest.py`** und **`persistence/schema_admin.py`** (Default `INFO`, weil man sie von Hand aufruft):

```python
def main(argv: list[str] | None = None) -> int:
    console_level = parse_verbosity("pricing-ingest", argv, default_level="INFO")
    settings = get_settings()
    setup_logging(
        console_level=console_level,
        file_level=settings.log_level,
        log_file=settings.log_file,
    )
    try:
        ...
```
(bei `schema_admin` entsprechend `"pricing-initdb"`; `load_dotenv()` in beiden entfernen)

**`scripts/smoke_orm.py`**: `load_dotenv()` entfernen — `get_engine()` holt sich die Settings selbst.

## 8g — `tests/test_exception.py`

Der Test prüft weiterhin `get_engine()` — dort sitzt die Anforderung ja (s. Kasten in 8e) —, muss aber den Settings-Cache berücksichtigen:

```python
from pricing.persistence.db import get_engine
from pricing.shared.config import get_settings


def test_fehlendes_passwort_wird_uebersetzt(
    monkeypatch: pytest.MonkeyPatch, tmp_path: Path
) -> None:
    """Nur wer eine Engine will, braucht das Passwort -- dann aber verbindlich."""
    monkeypatch.delenv("DB_PRICING_PW", raising=False)
    monkeypatch.chdir(tmp_path)   # kein .env in Reichweite
    get_settings.cache_clear()    # lru_cache umgehen

    with pytest.raises(ConfigError) as exc_info:
        get_engine()

    assert "DB_PRICING_PW" in str(exc_info.value)
    get_settings.cache_clear()    # Cache fuer die uebrigen Tests leeren
```

> [!warning] Zwei Fallen im Test
> - **`lru_cache`**: Ohne `cache_clear()` bekommst du ein zuvor erfolgreich geladenes Settings-Objekt zurück und der Test schlägt nicht an.
> - **Die `.env` im Projektordner**: `delenv` allein reicht nicht, weil pydantic-settings die Datei liest. Deshalb das `chdir(tmp_path)`.

- [x] Block 8 erledigt, Checks grün

---

# Abschluss

```bash
./.venv/bin/ruff check . && ./.venv/bin/ruff format . && ./.venv/bin/mypy && ./.venv/bin/python -m pytest -q
pricing-initdb -v
pricing-ingest            # Laufzeit gegen 56,26 s vergleichen
python -m pricing.main -v # Referenzwert vergleichen
python scripts/smoke_orm.py
```

**Referenzwert aus Block 0 des Schema-Umbaus:**
`PriceRecommendation(product_id='RFJA3D', calendar_date=2027-04-22, days_until_travel=261, recommended_adjustment_pct=Decimal('10.275000'), occupancy=1.0, total_capacity=3, booked_units=3)`

- [x] Alles grün, Werte identisch → committen
