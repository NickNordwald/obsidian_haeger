---
tags:
  - documentation
type: tutorial
date: 2026-07-23
---

Geführtes Tutorial für einen **Wegwerf-Spike** (`git`-Branch, wird nie gemergt): SQLAlchemy **besitzt ein schlankes Schema**, das die **Domäne spiegelt** — nur was das Pricing wirklich braucht, landet in der DB (genau wie `CsvDataSource` nur die gebrauchten Spalten liest). Bewusst **enge Kopplung** Code ↔ DB, klassisches ORM-Mapping (Klasse ↔ Tabelle, Objekt ↔ Zeile). Gegenstück (mapping auf ein bestehendes, vollständiges Schema): [[SQLAlchemy (Teil 2) - Plan-Review & Tutorial|SQLAlchemy (Teil 2)]]. Ausgangsstand: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand]].

> [!abstract] Die Idee
> In Teil 2 ist die DB **System of Record** — sie speichert alles aus den Roh-CSVs (Umsatz, Kunde, feature_2 …), auch was das Pricing nie anfasst; die ORM-Modelle *mappen* nur eine Teilmenge davon. Hier drehst du das um: die DB **spiegelt die Domäne**. Was kein Domänen-Objekt braucht, wird **gar nicht erst persistiert** — keine `products`/`customers`-Tabelle, keine Umsatzspalten, kein Fremdschlüssel-Ballast. Die ORM-Modelle sind die **einzige** Schema-Quelle (`create_all`). Ergebnis: enge Kopplung + ein Schema, das exakt so groß ist wie der Bedarf.

> [!warning] Das ist ein Spike
> Wegwerf-Branch, kein Merge. Aggressiv löschen ist ok. Der `main`-Stand (System-of-Record-Variante) bleibt unberührt. **Der CSV-Weg bleibt** erhalten.

---
# Teil A — Was sich konzeptionell ändert

| | Teil 2: **mappen** (main) | Spike: **schlank besitzen** (dieser Branch) |
|---|---|---|
| Schema-Quelle | `sql/schema.sql` (handgeschrieben) | **ORM-Modelle** → `Base.metadata.create_all()` |
| Was steht in der DB | **alles** aus den CSVs (System of Record) | **nur** was die Domäne braucht |
| `products` / `customers` | eigene Tabellen (FK-Ziele) | **gibt es nicht** (Pricing braucht sie nie) |
| Umsatz/Kunde/`feature_2`/… | persistiert | **nie in der DB** |
| `price_recommendations` | `capacity_id` **FK** → Lookup nötig | `product_id`+`calendar_date` **direkt** → **kein** Lookup |
| Ingest | System-of-Record-ETL (volle Zeilen-DTOs) | **schlank**, wiederverwendet `CsvDataSource` |
| psycopg roh (`COPY`, `connect`) | vorhanden (2. Adapter) | **gelöscht** |
| CSV-Pipeline-Adapter | vorhanden | **bleibt** |

**Zentrale Erkenntnis:** Weil die Tabellen die Domänen-Objekte 1:1 spiegeln, **verschwindet das FK-Auflösungs-Problem** aus Teil 1/2. `PriceRecommendation` kennt `product_id`+`calendar_date` — also speichern wir genau das, kein `capacity_id`, kein Lookup-Dict, kein `INSERT … SELECT`. Der Writer wird trivial.

**Was unangetastet bleibt** (der Architektur-Beweis): `domain.py`, `ports.py`, `core/` (builder, recommendations, app), `tests/test_pricing.py`. Du fasst nur `adapters/` + `main.py` + Infra an.

## Designentscheidung: getrennte ORM-Modelle statt „Domänenklassen direkt mappen"

Du hattest angeboten, `Booking` & Co. **direkt** als ORM-Entitäten zu mappen. Ich rate für den Haupt-Refactor **davon ab** und halte die schlanken ORM-Modelle in `adapters/orm.py` von der Domäne getrennt — aus drei Gründen:
1. **Domänen-Reinheit bleibt.** `domain.py` sagt „berührt nie eine DB". Würden die Domänenklassen von `Base` erben / `mapped_column` benutzen, importierte der Kern SQLAlchemy → die Architektur, die du gerade sauber gebaut hast, bricht.
2. **`frozen`-Falle.** `Price` und `CapacityRow` sind `@dataclass(frozen=True)`. Das ORM **setzt** Attribute beim Laden — frozen blockiert das. Direktes Mapping würde dich zwingen, sie zu entfrieren (Design-Rückschritt).
3. **`PriceableUnit` ist keine Zeile.** Das eigentliche Rechenobjekt wird zur Laufzeit *gebaut*, nicht persistiert — „Objekt = Zeile" gilt für die Domäne ohnehin nicht durchgängig.

Weil die ORM-Modelle jetzt **schlank** sind (dieselben Felder wie die Domäne), ist die Konvertierung ORM ↔ Domäne **1:1 und trivial** — der Preis für die Reinheit ist minimal. Wer den „Objekt = Zeile ganz ohne Konvertierung"-Effekt *fühlen* will: siehe [[#Anhang — Domänenklassen direkt mappen (imperative mapping)|Anhang]].

---
# Teil B — Tutorial

> [!tip] Arbeitsmodus
> Boilerplate + je ein Vorbild gebe ich; Wiederholungen schreibst du (mit einklappbaren Hinweisen). Bei Fehlern: Meldung + Schritt nennen.

## Schritt 0 — Branch

Falls noch nicht geschehen: `git switch -c spike/orm-schlank` (vom sauberen `main`). Falls du schon auf deinem Spike-Branch bist und Teile des alten Spike-Tutorials umgesetzt hast: kein Problem, die Schritte unten überschreiben das.

## Schritt 1 — Löschen (explizit)

**Ganze Dateien:**
```bash
git rm sql/schema.sql            # Schema kommt jetzt aus den Modellen
git rm scripts/smoke_test.py     # nutzt den gleich gelöschten psycopg-Weg (falls vorhanden)
```

**Code innerhalb von Dateien:**
- `adapters/db.py`: `get_dsn()` + `connect()` + psycopg-Importe → **weg** (nur `get_engine()` bleibt).
- `adapters/sources.py`: `PostgresqlDataSource` → **weg**. `CsvDataSource` + `SqlAlchemyDataSource` **bleiben**.
- `adapters/writers.py`: `PostgresqlRecommendationWriter` → **weg**. `CsvRecommendationWriter` + `SqlAlchemyRecommendationWriter` **bleiben**.
- `adapters/ingest.py`: **fast alles raus** — die vollen Zeilen-DTOs (`BookingRow`/`PriceRow`), `_parse_*`, `_load_*`, `_derive_products`, `_distinct_customers`, `_bedroom_types`, `_validate_net_revenue`, die `_copy_*`-Loader. Bleibt nur ein **schlanker** ORM-Ingest (Schritt 5). Grund: alles davon existierte nur für die *System-of-Record*-Spalten, die es jetzt nicht mehr gibt.

**`docker-compose.yml`:** die init-dir-Zeile entfernen:
```yaml
    volumes:
      - pricing_data:/var/lib/postgresql
      # ENTFERNEN:  - ./sql:/docker-entrypoint-initdb.d:ro
```

## Schritt 2 — Schlanke ORM-Modelle + `create_schema` (`adapters/orm.py`) ⭐

Die Modelle spiegeln die **Domänen-Objekte** (`CapacityRow`, `Booking`, `Price`, `PriceRecommendation`) — je Feld eine Spalte, plus ein Surrogat-`id` als PK. **Kein** `products`/`customers`, **kein** `capacity_id`-FK, **keine** Umsatz-/Kundenspalten.

**Ich gebe dir Base + `create_schema` + `Capacity` + `PriceRecommendation`** (letztere ist der springende Punkt: `product_id`+`calendar_date` **direkt**, kein FK). `Booking` und `Price` schreibst du.

```python
"""ORM-Modelle als EINZIGE, schlanke Schema-Quelle (Spike).

Die DB spiegelt die Domaene, nicht die Quell-CSVs: nur was das Pricing braucht.
create_all() erzeugt die Tabellen -- es gibt kein sql/schema.sql mehr.
"""

from datetime import date
from decimal import Decimal

from sqlalchemy import Numeric, UniqueConstraint
from sqlalchemy.engine import Engine
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


def create_schema(engine: Engine, *, reset: bool = False) -> None:
    """Erzeugt alle Tabellen aus den Modellen. reset=True droppt vorher alles
    (Voll-Reload-Ersatz fuer TRUNCATE/down -v)."""
    if reset:
        Base.metadata.drop_all(engine)
    Base.metadata.create_all(engine)


class Capacity(Base):
    __tablename__ = "capacity"

    id: Mapped[int] = mapped_column(primary_key=True)
    product_id: Mapped[str]
    calendar_date: Mapped[date]
    bookable_units: Mapped[int]
    optional_units: Mapped[int]

    __table_args__ = (UniqueConstraint("product_id", "calendar_date"),)


# TODO: class Booking(Base) -> "bookings"
#   id (PK), product_id, start_date (date), nights (int), status (str)
#   -> spiegelt domain.Booking; kein cust_id/Umsatz/feature_2!
#
# TODO: class Price(Base) -> "prices"
#   id (PK), product_id, change_date (date), length_of_stay (str),
#   price_sterling (Decimal, mapped_column(Numeric(12, 2)))
#   -> Spalte heisst price_sterling (Domaenen-Feldname), nicht current_price


class PriceRecommendation(Base):
    __tablename__ = "price_recommendations"

    id: Mapped[int] = mapped_column(primary_key=True)
    product_id: Mapped[str]           # <-- direkt, KEIN capacity_id-FK
    calendar_date: Mapped[date]       # <-- damit entfaellt die FK-Aufloesung
    days_until_travel: Mapped[int]
    recommended_adjustment_pct: Mapped[Decimal] = mapped_column(Numeric(10, 6))
    occupancy: Mapped[float]
    total_capacity: Mapped[int]
    booked_units: Mapped[int]
```

`mapped_column(primary_key=True)` auf `Mapped[int]` → SQLAlchemy erzeugt in Postgres automatisch eine IDENTITY-Spalte. `Numeric(...)`/`Mapped[float]` wie in Teil 2. Weil wir das Schema **besitzen**, brauchen wir hier **keine** IDENTITY/GENERATED/CHECK-Handarbeit wie im alten Spike — es gibt schlicht nur die schlanken Spalten.

> [!hint]- Lösungshinweis: `Booking` und `Price`
> ```python
> class Booking(Base):
>     __tablename__ = "bookings"
>
>     id: Mapped[int] = mapped_column(primary_key=True)
>     product_id: Mapped[str]
>     start_date: Mapped[date]
>     nights: Mapped[int]
>     status: Mapped[str]
>
>
> class Price(Base):
>     __tablename__ = "prices"
>
>     id: Mapped[int] = mapped_column(primary_key=True)
>     product_id: Mapped[str]
>     change_date: Mapped[date]
>     length_of_stay: Mapped[str]
>     price_sterling: Mapped[Decimal] = mapped_column(Numeric(12, 2))
> ```

**Checkpoint:**
```bash
python -c "from pricing.adapters import orm; print(sorted(orm.Base.metadata.tables))"
# -> ['bookings', 'capacity', 'price_recommendations', 'prices']   (kein products/customers!)
```

## Schritt 3 — `SqlAlchemyDataSource` (fast unverändert)

Deine `SqlAlchemyDataSource` aus Teil 2 liest schon die schlanken Modelle und baut Domänen-Objekte — die Konvertierung ist 1:1. **Einzige Anpassung:** in `load_prices` heißt das ORM-Attribut jetzt `price_sterling` (nicht mehr `current_price`):
```python
price_sterling=r.price_sterling,   # vorher: r.current_price
```
Sonst nichts. (Genau das ist der Lohn des schlanken Mappings: nichts umzubauen.)

## Schritt 4 — `SqlAlchemyRecommendationWriter` wird trivial ⭐

Kein Fremdschlüssel mehr → **keine** FK-Auflösung, **kein** Lookup-Dict, **kein** Missing-Key-Handling. Nur: alt leeren, neu einfügen.
```python
class SqlAlchemyRecommendationWriter:
    def __init__(self, engine: Engine):
        self._engine = engine

    def write(self, recs: list[PriceRecommendation]) -> None:
        with Session(self._engine) as session, session.begin():
            session.execute(text("TRUNCATE price_recommendations RESTART IDENTITY"))
            if recs:
                session.execute(
                    insert(orm.PriceRecommendation),
                    [
                        {
                            "product_id": r.product_id,
                            "calendar_date": r.calendar_date,
                            "days_until_travel": r.days_until_travel,
                            "recommended_adjustment_pct": r.recommended_adjustment_pct,
                            "occupancy": r.occupancy,
                            "total_capacity": r.total_capacity,
                            "booked_units": r.booked_units,
                        }
                        for r in recs
                    ],
                )
```
Vergleiche das mit dem Teil-1/2-Writer (Bulk-`SELECT` capacity → Dict → pro Zeile nachschlagen → zählen). **Das** ist der konkrete Gewinn eines domänen-gespiegelten Schemas. (`text`/`insert`/`Session` importieren wie in Teil 2.)

## Schritt 5 — `ingest.py`: schlank, wiederverwendet `CsvDataSource`

Weil nur die Domänen-Spalten in die DB gehen, **liest der Ingest exakt dieselben schlanken Objekte wie die CSV-Pipeline** — also einfach `CsvDataSource` benutzen und per ORM einfügen. Der ganze System-of-Record-Apparat (volle DTOs, Produkt-/Kundenableitung, Validierungen) entfällt.

```python
"""ETL (schlank): liest die CSVs ueber CsvDataSource (nur gebrauchte Spalten)
und laedt sie per ORM in die selbst-besessene DB."""

from pathlib import Path

from sqlalchemy import insert
from sqlalchemy.engine import Engine
from sqlalchemy.orm import Session

from . import orm
from .db import get_engine
from .sources import CsvDataSource


def run_ingest(engine: Engine, data_dir: Path) -> None:
    src = CsvDataSource(data_dir)
    capacity = src.load_capacity()
    bookings = src.load_bookings()
    prices = src.load_prices()

    orm.create_schema(engine, reset=True)

    with Session(engine) as session, session.begin():
        if capacity:
            session.execute(
                insert(orm.Capacity),
                [
                    {
                        "product_id": c.product_id,
                        "calendar_date": c.calendar_date,
                        "bookable_units": c.bookable_units,
                        "optional_units": c.optional_units,
                    }
                    for c in capacity
                ],
            )
        # TODO: bookings -> insert(orm.Booking), [ {product_id, start_date, nights, status} ]
        # TODO: prices   -> insert(orm.Price),   [ {product_id, change_date, length_of_stay, price_sterling} ]


def main() -> None:
    engine = get_engine()
    run_ingest(engine, Path("daily_data"))
    engine.dispose()


if __name__ == "__main__":
    main()
```

> [!hint]- Lösungshinweis: die zwei fehlenden Inserts
> ```python
>         if bookings:
>             session.execute(
>                 insert(orm.Booking),
>                 [
>                     {
>                         "product_id": b.product_id, "start_date": b.start_date,
>                         "nights": b.nights, "status": b.status,
>                     }
>                     for b in bookings
>                 ],
>             )
>         if prices:
>             session.execute(
>                 insert(orm.Price),
>                 [
>                     {
>                         "product_id": p.product_id, "change_date": p.change_date,
>                         "length_of_stay": p.length_of_stay,
>                         "price_sterling": p.price_sterling,
>                     }
>                     for p in prices
>                 ],
>             )
> ```

> [!note] Erwarte: langsamer als `COPY`
> ORM-Bulk-Insert (SQLAlchemy „insertmanyvalues") ist deutlich langsamer als das alte `COPY` — bei ~2,4 Mio Zeilen Minuten statt ~110 s. Das ist die Lektion, kein Bug. Zum Iterieren `daily_data` temporär verkleinern.

## Schritt 6 — `db.py` entschlacken & `main.py` verdrahten

**`db.py`:** nur `get_engine()` (aus Teil 2) bleibt, psycopg-Rohzugang (`get_dsn`/`connect`) raus.

**`main.py`:** die ORM-Pipeline gegen die eigene DB (der CSV-Weg bleibt im Code, wird hier nur nicht verdrahtet):
```python
import logging

from .adapters.db import get_engine
from .adapters.sources import SqlAlchemyDataSource
from .adapters.writers import SqlAlchemyRecommendationWriter
from .core.app import PricingApp
from .core.builder import PriceableUnitBuilder
from .core.recommendations import HeuristicRecommendationStrategy, RecommendationEngine


def main() -> None:
    logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
    engine = get_engine()
    app = PricingApp(
        source=SqlAlchemyDataSource(engine),
        builder=PriceableUnitBuilder(),
        engine=RecommendationEngine(HeuristicRecommendationStrategy()),
        writer=SqlAlchemyRecommendationWriter(engine),
    )
    print(f"Höchste Empfehlung: {app.run()}")
    engine.dispose()


if __name__ == "__main__":
    main()
```

## Schritt 7 — Laufen & verifizieren

```bash
docker compose down -v && docker compose up -d
until docker exec pricing-db pg_isready -U pricing -d pricing; do sleep 1; done
python -m pricing.adapters.ingest      # create_schema(reset) + schlanke ORM-Inserts
python -m pricing.main                 # Pricing aus der DB
ruff check . && mypy src tests && pytest -q
```

**Beweis, dass die DB wirklich schlank ist:**
```bash
docker exec -it pricing-db psql -U pricing -d pricing -c "\dt"          # nur 4 Tabellen
docker exec -it pricing-db psql -U pricing -d pricing -c "\d bookings"  # nur 5 Spalten (id + 4)
```
Kein `products`/`customers`, keine Umsatzspalten, kein `capacity_id`.

**Fachliche Gegenprobe:** „Höchste Empfehlung" muss identisch zum `main`-Stand sein — Referenz aus dem Handtest: `RFJA3D` / 2027-04-22 / **10.600000**, `price_recommendations` = **98 232**.

## Definition of Done (Spike)

> [!info] Gedankenexperiment, nicht umgesetzt
> Dieser Spike war eine **Durchdenk-Übung**, kein Umbau: Wie sähe das Projekt aus, wenn das ORM das Schema besäße und man nur das modellierte, was die Business-Logik tatsächlich braucht? Der Stand in `RDB_ORM/` behielt bewusst `sql/schema.sql` und den vollen „system of record"-Ansatz.
>
> Die eigentliche Erkenntnis wurde später trotzdem übernommen: In [[Umsetzungsstand (REST APIs & Layered Architecture)|Kapitel 1.5]] wurde `sql/schema.sql` entfernt und SQLAlchemy zur einzigen Quelle der Wahrheit gemacht — allerdings mit dem *vollständigen* Schema, nicht mit dem hier skizzierten schlanken. Der Spike hat also die Frage geklärt, nicht die Antwort geliefert.

Ursprüngliche Abnahmekriterien (nie abgearbeitet, hier als Dokumentation des Gedankengangs):

- `sql/schema.sql` + psycopg-Weg gelöscht; CSV-Weg bleibt.
- `orm.py`: 4 schlanke Modelle + `create_schema()`; `\d bookings` = id + 4 Spalten; kein `products`/`customers`.
- `price_recommendations` ohne `capacity_id`; Writer ohne FK-Auflösung.
- `ingest.py` schlank über `CsvDataSource`; kein `_copy_*`/DTO-Apparat mehr.
- Ingest + `pricing.main` laufen; „Höchste Empfehlung" == `main`-Referenz.
- `ruff`/`mypy`/`pytest` grün (Tests unverändert = Kern nie angefasst).

---
# Teil C — Diskussion: domänen-gespiegeltes vs. System-of-Record-Schema

**Gewinne (dieser Spike):**
- **Das Schema ist so groß wie der Bedarf.** Keine toten Spalten/Tabellen, weniger Speicher, klarere Absicht.
- **FK-Auflösung verschwindet.** `price_recommendations` ist self-describing (`product_id`+`calendar_date`) → Writer trivial, kein Lookup-Dict/`INSERT … SELECT`.
- **Ingest schrumpft radikal**, weil er dieselben schlanken Objekte wie die CSV-Pipeline lädt.
- **Enge, ehrliche Kopplung:** ändert sich die Domäne, ändert sich das Schema — Modelle sind die eine Wahrheit.

**Verluste (bewusst):**
- **Keine referenzielle Integrität.** `product_id` ist ein freier String (kein `products`-FK); die DB kann eine verwaiste `product_id` nicht verhindern.
- **Kein System of Record.** Umsatz, Kunde, `feature_2` sind für immer weg — die DB kann **nur** die Fragen des Pricings beantworten. Ein zweiter Consumer (Reporting, Analytics) wäre aufgeschmissen.
- **Enge Kopplung schneidet in beide Richtungen:** *jede* Domänenänderung erzwingt eine Migration.

**Wann was:** app-eigene, **single-consumer**, greenfield Service-DB („database per service") → domänen-gespiegelt-schlank ist ideal. Geteilte DB / Analytics / System of Record → das volle Schema (`main`). Du hast jetzt **beide Pole** gebaut und kannst den Trade-off aus Erfahrung begründen.

---
# Anhang — Domänenklassen direkt mappen (imperative mapping)

> Ausführliche Einordnung (die 4 ORM-Varianten + Vor-/Nachteile des direkten Mappings): [[ORM-Anbindungsvarianten & direktes Domänen-Mapping|ORM-Anbindungsvarianten & direktes Domänen-Mapping]].

Wenn du den „Objekt = Zeile ganz ohne Konvertierung"-Effekt willst, ist **imperative (classical) mapping** das klassische SQLAlchemy-Mittel: man definiert die `Table` + das Mapping **im Adapter** und mappt eine **pure Python-Klasse** — die Domänenklasse muss SQLAlchemy also *nicht* importieren:
```python
# in adapters/orm.py
from sqlalchemy import Table, Column, Integer, String, Date, MetaData
from sqlalchemy.orm import registry
from ..domain import Booking

mapper_registry = registry()
metadata = mapper_registry.metadata

bookings = Table(
    "bookings", metadata,
    Column("id", Integer, primary_key=True),
    Column("product_id", String), Column("start_date", Date),
    Column("nights", Integer), Column("status", String),
)
mapper_registry.map_imperatively(Booking, bookings)
```
Danach liefert `session.scalars(select(Booking))` **direkt** `domain.Booking`-Instanzen, `session.add(booking)` persistiert sie — keine Konvertierung.

**Caveats (warum es hier trotzdem nicht der Haupt-Weg ist):**
- **`frozen` muss weg.** `Price`/`CapacityRow` sind frozen → beim Laden kann das ORM keine Attribute setzen. Du müsstest sie entfrieren (Domänen-Rückschritt).
- **`__init__`/`__post_init__` wird beim Laden übersprungen** (SQLAlchemy baut Instanzen über `__new__`) → Validierung läuft nur bei normaler Konstruktion (CSV-Pfad), nicht beim DB-Laden.
- **Surrogat-`id`** braucht ein Attribut auf der Klasse; wo es keinen natürlichen Schlüssel gibt (bookings), müsstest du der Domänenklasse ein `id`-Feld geben (pollution) — oder mit zusammengesetzten natürlichen Schlüsseln arbeiten.

Für den echten Refactor: **getrennte schlanke ORM-Modelle** (oben). Imperative Mapping ist die lehrreiche Kür, wenn du „Domänenklasse *ist* die Zeile" einmal erleben willst.
