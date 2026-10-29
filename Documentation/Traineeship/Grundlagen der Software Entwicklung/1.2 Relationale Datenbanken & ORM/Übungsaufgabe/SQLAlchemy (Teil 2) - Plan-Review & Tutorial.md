---
tags:
  - documentation
type: tutorial
date: 2026-07-22
---

Plan-Review **und** geführtes Umsetzungs-Tutorial für **Teil 2 (SQLAlchemy)** der DB-Erweiterung. Kontext: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand (DB-Erweiterung)]] · Code-Review Teil 1: [[Code-Review (Stand Teil 1 - rohes SQL)|Code-Review (Stand Teil 1)]] · Theorie: [[1.2 Relationale Datenbanken & ORM - Vertiefung|1.2 Vertiefung]].

> [!abstract] So ist das Dokument gedacht
> **Teil A** ist mein Review deines Plans — Entscheidungen, Begründungen, Antworten auf deine zwei offenen Fragen (Schema anpassen? DB leer/Ingest?). Lies das einmal ganz, bevor du tippst.
> **Teil B** ist das Tutorial in Schritten. Jeder Schritt hat: *Ziel · Datei · Aufgabe · Syntax-Hilfe · Checkpoint*. Ich gebe dir die Boilerplate und je ein Vorbild fertig; die Wiederholungen schreibst **du** (mit einklappbaren Lösungshinweisen zum Nachsehen, falls du hängst). Ziel: du bist zeitnah durch **und** hast die Syntax einmal selbst in den Fingern gehabt.

---
# Teil A — Plan-Review

## Dein Plan, gespiegelt
Du willst dieselbe Datenquelle (`DataSource`) und den Writer (`RecommendationWriter`) ein **drittes Mal** implementieren, jetzt über SQLAlchemy-ORM, und danach Vor-/Nachteile gegen den psycopg-Weg diskutieren. Du hast das Tutorial gelesen, bist aber unsicher bei Syntax und Praxis. Die DB läuft in Docker, ist aber evtl. leer, und `ingest.py` gehört zur psycopg-Lösung.

**Das ist der richtige Zuschnitt.** Teil 2 verlangt genau das: *„dieselbe Quelle mit SQLAlchemy, Vor-/Nachteile diskutieren"*. Du erfindest keine neue Fachlogik — du fügst einen dritten **Adapter** hinter dieselben zwei Protocols. Deine Architektur aus Teil 1 macht das billig: `app.py`/`builder.py`/`recommendations.py` fasst du **nicht** an.

## Die 6 Kernentscheidungen (mit Begründung)

### E1 — Das Schema bleibt, wie es ist. **Keine Anpassung nötig.**
Das ist deine wichtigste Frage, also ausführlich:

Es gibt zwei Welten, wie ORM und Schema zusammenhängen:
- **(a) ORM besitzt das Schema:** Du beschreibst die Tabellen als Python-Klassen und lässt SQLAlchemy sie per `Base.metadata.create_all()` (oder Alembic-Migrationen) erzeugen.
- **(b) ORM bildet ein bestehendes Schema ab:** Die Tabellen existieren schon (hier: aus `sql/schema.sql`, beim Container-Start eingespielt). Deine ORM-Klassen sind nur eine **Sicht** darauf, um typisiert zu lesen/schreiben. SQLAlchemy erzeugt nichts.

**Wir nehmen (b).** Gründe:
1. Dein ganzes Projekt-Motto ist *„erweitern, nicht ersetzen"* — CSV bleibt, psycopg bleibt, ORM kommt als dritter Adapter dazu. `schema.sql` bleibt die **eine** Quelle der DDL-Wahrheit. Zwei konkurrierende Schema-Definitionen (SQL-Datei *und* `create_all`) würden früher oder später auseinanderdriften.
2. „ORM auf bestehende Tabelle mappen" ist der **häufigste Realfall** im Job (die DB ist meist schon da). Genau das lernst du hier.
3. Du umgehst die fummeligen Stellen: `GENERATED ALWAYS AS ... STORED` (`net_revenue`), `GENERATED ALWAYS AS IDENTITY`, `CHECK`-Constraints müsstest du bei (a) alle in SQLAlchemy-Syntax nachbauen (`Computed`, `Identity`, `CheckConstraint`). Bei (b) **ignorierst du sie einfach** — du mappst nur die Spalten, die du liest/schreibst.

> [!warning] Konsequenz-Regel für den ganzen Teil 2
> Du rufst **niemals** `Base.metadata.create_all(engine)` gegen die echte DB. Die Tabellen sind schon da. Deine ORM-Klassen dürfen ruhig eine **Teilmenge** der Spalten abbilden (bei `bookings` brauchst du nur 4 von 15). Das ist gewollt und spart dir das Nachbauen der DB-generierten Spalten.

Einzige Spalte, bei der du kurz nachdenken musst: `price_recommendations.id` ist `GENERATED ALWAYS AS IDENTITY`. Du **schreibst sie nie selbst** (die DB vergibt sie), also lässt du sie beim Insert einfach weg → kein Problem. Genauso `computed_at` (hat `DEFAULT now()`): nicht mappen, DB füllt sie. (Ob `computed_at` überhaupt sinnvoll ist, war ein offener Punkt im Teil-1-Review — für Teil 2 ignorierst du sie schlicht.)

### E2 — DB seeden mit dem bestehenden psycopg-`ingest.py`. Das ist **kein Betrug.**
Deine Sorge „ingest gehört zur nicht-ORM-Lösung, die DB wäre leer" ist berechtigt, aber die Auflösung ist einfach: **Das Befüllen der DB ist Infrastruktur, kein Teil des Vergleichs.** Der Vergleich ORM-vs-psycopg findet in **`DataSource` (lesen)** und **`RecommendationWriter` (schreiben)** statt — nicht im ETL. Dass die Daten per `COPY` reinkamen, ist für den lesenden ORM-Adapter unsichtbar; er sieht nur Tabellen.

Also: einmal `python -m pricing.ingest` laufen lassen, DB ist voll, fertig. Einen ORM-Ingest zu bauen ist ein **optionaler Stretch** (Schritt 6) — er ist sogar *lehrreich*, weil er zeigt, wie ORM-Bulk-Insert gegen `COPY` verliert. Aber für „zeitnah fertig" nicht nötig.

> [!info] Falls die DB gerade leer ist
> `docker compose down -v && docker compose up -d` spielt `schema.sql` automatisch neu ein (init-dir auf leerem Volume) → leere Tabellen mit Schema. Dann `python -m pricing.ingest` → gefüllt. Das hast du in Teil 1 schon einmal versehentlich durchgespielt.

### E3 — Injektionspunkt ist die **`Engine`**, nicht die Connection. (Wichtiger Lern-Unterschied!)
Bei psycopg hast du eine **offene `Connection`** injiziert (`PostgresqlDataSource(conn)`). Der naheliegende Reflex wäre, jetzt eine `Session` zu injizieren. **Mach das nicht.** Der idiomatische SQLAlchemy-Weg:

- Die **`Engine`** ist das langlebige, thread-sichere App-Objekt (ein Verbindungs-Pool). Sie wird einmal gebaut und injiziert.
- Die **`Session`** ist die *Unit of Work* — kurzlebig, für **eine** Arbeitseinheit. Man öffnet sie **innerhalb** einer Methode (`with Session(engine) as session:`) und wirft sie danach weg.

Also: du injizierst die `Engine`, und jede Methode öffnet sich ihre eigene kurze Session. Das ist genau der konzeptionelle Unterschied, den du in der Diskussion anführen kannst: *psycopg gibt dir eine rohe Verbindung; das ORM schiebt eine Sitzungs-/Identity-Map-Schicht dazwischen, die anders lebt.*

### E4 — ORM-Entities in **eigenes Modul**, Domänenschicht bleibt rein.
`domain.py` sagt im Docstring wörtlich: *„Nichts hier sollte je ein file system, Datenbank, etc. berühren!"* ORM-Klassen sind Persistenz. Sie kommen in ein **neues Modul `orm.py`**. Deine `SqlAlchemyDataSource` liest ORM-Objekte und **konvertiert sie in die vorhandenen Domänen-Dataclasses** (`CapacityRow`, `Booking`, `Price`) — exakt die Rolle, die bei psycopg der `Cls(**row)`-Trick spielte. So bleibt die Grenze sauber und die Business-Logik merkt wieder nichts.

### E5 — Treiber `postgresql+psycopg` (v3, schon installiert).
SQLAlchemy braucht eine URL mit Dialekt+Treiber. Du hast psycopg **3** schon drin — der passende Dialekt heißt `postgresql+psycopg` (der v2-Treiber wäre `+psycopg2`, den willst du **nicht**). Kein neuer Treiber nötig, nur das SQLAlchemy-Paket.

### E6 — Scope: nur **DataSource + Writer** neu. Ingest-Vergleich optional.
Konkret neu: `orm.py` (Modelle), `get_engine()` in `db.py`, `SqlAlchemyDataSource` in `sources.py`, `SqlAlchemyRecommendationWriter` in `writers.py`, ein Verdrahtungs-Skript, ein Diskussions-Abschnitt. Das war's.

## Fallen, die dich sonst Zeit kosten
- **mypy `strict`:** SQLAlchemy 2.0 ist voll typisiert (bringt `py.typed` mit) und braucht **keinen** mypy-Plugin mehr, *wenn* du die moderne `Mapped[...]`-Syntax nutzt (nicht die alte 1.x-`Column`-Syntax). Nutze durchgängig `Mapped[]` + `mapped_column()`.
- **`Numeric` → `Decimal`:** SQLAlchemy gibt `NUMERIC`-Spalten als `Decimal` zurück — genau wie psycopg. Deine Domain-Konvertierung bleibt also trivial. Bei `current_price`/`recommended_adjustment_pct` explizit `Numeric(12, 2)` bzw. `Numeric(10, 6)` angeben.
- **`TRUNCATE` ist keine ORM-Operation:** Das machst du mit `session.execute(text("TRUNCATE ..."))`. Völlig normal, ORM und rohes SQL mischen ist erlaubt.
- **Namens-Kollision `Price`:** Es gibt schon `domain.Price`. Nenne die ORM-Klasse **nicht** auch `Price` im selben Namensraum — greif sie modul-qualifiziert ab (`from pricing import orm` → `orm.Price`). Dann kollidiert nichts.
- **`echo=True` ist dein bester Lehrer:** Baue die Engine anfangs mit `echo=True`. Dann druckt SQLAlchemy **jedes** emittierte SQL-Statement in die Konsole. Genau das willst du sehen — es macht die „Magie" sichtbar und ist Gold für die Diskussion.

## Was du bewusst **nicht** tust
- Kein `create_all()` gegen die echte DB (E1).
- Kein ORM-Ingest jetzt (nur optionaler Stretch, E2/E6).
- Keine ORM-Klasse in `domain.py` (E4).
- Kein `Session`-Injektion durch alle Schichten (E3).

---
# Teil B — Tutorial

> [!tip] Arbeitsmodus (wie in Teil 1)
> Du schreibst, ich reviewe. Wo ich unten `# TODO` schreibe, bist **du** dran. Erst selbst versuchen, dann den einklappbaren `Lösungshinweis` aufmachen. Nach jedem Schritt der **Checkpoint** — läuft der nicht, nicht weitermachen.

## Schritt 0 — Vorbereitung (Dependency + DB füllen)

**Ziel:** SQLAlchemy installiert, DB gefüllt, alles grün.

**Aufgabe:**
1. In `pyproject.toml` unter `[project.optional-dependencies]` einen neuen Extra ergänzen:
   ```toml
   orm = ["sqlalchemy>=2.0"]
   ```
2. Neu installieren (SQLAlchemy nutzt den psycopg-Treiber aus dem `db`-Extra, also beide mitnehmen):
   ```bash
   pip install -e ".[dev,db,orm]"
   ```
3. DB hochfahren und füllen:
   ```bash
   docker compose up -d
   python -m pricing.ingest
   ```

**Checkpoint:**
```bash
python -c "import sqlalchemy; print(sqlalchemy.__version__)"   # >= 2.0
```
Und in Adminer (http://localhost:8080) oder per SQL: `SELECT count(*) FROM capacity;` → ~1.327.560. Wenn die Zahl steht, ist die Bühne bereit.

---

## Schritt 1 — Engine bauen (`db.py`)

**Ziel:** Eine `get_engine()`-Funktion analog zu `get_dsn()`/`connect()`, die die SQLAlchemy-Engine liefert. Passwort kommt aus derselben `.env`.

**Aufgabe:** In `db.py` diese Funktion ergänzen (die bestehenden `get_dsn`/`connect` bleiben unangetastet — psycopg-Weg lebt weiter):

```python
from sqlalchemy import create_engine
from sqlalchemy.engine import URL, Engine


def get_engine(echo: bool = False) -> Engine:
    pw = os.environ["DB_PRICING_PW"]
    url = URL.create(
        "postgresql+psycopg",   # Dialekt + psycopg-3-Treiber
        username="pricing",
        host="localhost",
        database="pricing",
        password=pw,
    )
    return create_engine(url, echo=echo)
```

`URL.create()` übernimmt das korrekte Escaping des Passworts — das Pendant zu `make_conninfo` aus dem psycopg-Weg (kein String-Zusammenbauen).

**Checkpoint:**
```bash
python -c "from pricing.db import get_engine; e=get_engine(); \
print(e.connect().execute(__import__('sqlalchemy').text('select 1')).scalar())"
# -> 1
```

---

## Schritt 2 — ORM-Modelle (`orm.py`, neu)

**Ziel:** Ein neues Modul mit einer `Base` und **vier** gemappten Klassen, die die vorhandenen Tabellen abbilden — nur die Spalten, die du brauchst.

**Referenz — welche Spalten du je Tabelle brauchst** (aus `sql/schema.sql`):

| ORM-Klasse            | Tabelle                 | Zu mappende Spalten                                                                                                   |
| --------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `Capacity`            | `capacity`              | `id`, `product_id`, `calendar_date`, `bookable_units`, `optional_units`                                               |
| `Booking`             | `bookings`              | `id`, `product_id`, `start_date`, `nights`, `status`                                                                  |
| `Price`               | `prices`                | `id`, `product_id`, `change_date`, `length_of_stay`, `current_price`                                                  |
| `PriceRecommendation` | `price_recommendations` | `id`, `capacity_id`, `days_until_travel`, `recommended_adjustment_pct`, `occupancy`, `total_capacity`, `booked_units` |

> [!note] Warum überall `id` mitmappen?
> SQLAlchemy braucht auf jeder gemappten Klasse einen **Primärschlüssel**. `capacity.id` brauchst du außerdem im Writer für die FK-Auflösung. Bei den reinen Lese-Modellen (`Booking`, `Price`) mappst du `id` nur, weil das ORM es verlangt — benutzen tust du es nicht.

**Ich gebe dir zwei Klassen fertig** (die Boilerplate + die trickreiche `PriceRecommendation` mit FK und `Numeric`). **Du schreibst `Booking` und `Price` selbst.**

```python
"""
ORM-Modelle (SQLAlchemy 2.0). Bilden die bestehenden Tabellen aus sql/schema.sql
ab — wir erzeugen hier KEIN Schema (kein create_all), sondern lesen/schreiben nur.
Bewusst eine Teilmenge der Spalten: nur, was Source/Writer brauchen.
"""

from datetime import date
from decimal import Decimal

from sqlalchemy import BigInteger, ForeignKey, Numeric
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class Capacity(Base):
    __tablename__ = "capacity"

    id: Mapped[int] = mapped_column(BigInteger, primary_key=True)
    product_id: Mapped[str]
    calendar_date: Mapped[date]
    bookable_units: Mapped[int]
    optional_units: Mapped[int]


# TODO: class Booking(Base) — __tablename__ = "bookings"
#       Spalten: id (BigInteger, PK), product_id, start_date, nights, status
# TODO: class Price(Base) — __tablename__ = "prices"
#       Spalten: id (BigInteger, PK), product_id, change_date, length_of_stay,
#                current_price (Decimal, mapped_column(Numeric(12, 2)))


class PriceRecommendation(Base):
    __tablename__ = "price_recommendations"

    id: Mapped[int] = mapped_column(BigInteger, primary_key=True)
    capacity_id: Mapped[int] = mapped_column(ForeignKey("capacity.id"))
    days_until_travel: Mapped[int]
    recommended_adjustment_pct: Mapped[Decimal] = mapped_column(Numeric(10, 6))
    occupancy: Mapped[float]
    total_capacity: Mapped[int]
    booked_units: Mapped[int]
```

Beachte: einfache Typen (`str`, `int`, `date`, `float`) leitet SQLAlchemy 2.0 aus dem `Mapped[...]` automatisch ab — nur bei `Numeric` (Nachkommastellen!) und den `BigInteger`-IDs bist du explizit. `id` und `computed_at` tauchen bei `PriceRecommendation` als Insert-Werte **nie** auf → DB vergibt/füllt sie.

> [!hint]- Lösungshinweis: `Booking` und `Price`
> ```python
> class Booking(Base):
>     __tablename__ = "bookings"
>
>     id: Mapped[int] = mapped_column(BigInteger, primary_key=True)
>     product_id: Mapped[str]
>     start_date: Mapped[date]
>     nights: Mapped[int]
>     status: Mapped[str]
>
>
> class Price(Base):
>     __tablename__ = "prices"
>
>     id: Mapped[int] = mapped_column(BigInteger, primary_key=True)
>     product_id: Mapped[str]
>     change_date: Mapped[date]
>     length_of_stay: Mapped[str]
>     current_price: Mapped[Decimal] = mapped_column(Numeric(12, 2))
> ```

**Checkpoint:**
```bash
python -c "from pricing import orm; print(orm.Base.metadata.tables.keys())"
# -> dict_keys(['capacity', 'bookings', 'prices', 'price_recommendations'])
```

---

## Schritt 3 — `SqlAlchemyDataSource` (`sources.py`)

**Ziel:** Dritte Implementierung des `DataSource`-Protocols. Liest per ORM, gibt dieselben Domänen-Objekte zurück wie die anderen beiden.

**Ich gebe dir `load_capacity` fertig, du schreibst `load_bookings` und `load_prices`.**

Oben in `sources.py` ergänzen:
```python
from sqlalchemy import select
from sqlalchemy.engine import Engine
from sqlalchemy.orm import Session

from . import orm
```

Neue Klasse (ans Ende von `sources.py`):
```python
class SqlAlchemyDataSource:
    "DataSource-Implementierung über SQLAlchemy-ORM."

    def __init__(self, engine: Engine):
        self._engine = engine

    def load_capacity(self) -> list[CapacityRow]:
        with Session(self._engine) as session:
            rows = session.scalars(select(orm.Capacity)).all()
            return [
                CapacityRow(
                    product_id=r.product_id,
                    calendar_date=r.calendar_date,
                    bookable_units=r.bookable_units,
                    optional_units=r.optional_units,
                )
                for r in rows
            ]

    # TODO: load_bookings(self) -> list[Booking]
    #   select(orm.Booking) -> je r ein domain.Booking(product_id, start_date,
    #   nights, status) bauen.
    # TODO: load_prices(self) -> list[Price]
    #   select(orm.Price) -> domain.Price(...). ACHTUNG Feld-Umbenennung:
    #   das ORM-Attribut heisst .current_price, das Domänenfeld price_sterling.
```

> [!note] Der Lern-Kontrast zu psycopg
> Bei psycopg hast du die Umbenennung im **SQL** gemacht (`current_price AS price_sterling`) und dann `Price(**row)`. Beim ORM machst du sie im **Python-Konstruktoraufruf** (`price_sterling=r.current_price`). Beide Male native Typen (`Decimal`, `date`) — kein Parsing. Das ist ein perfektes Diskussions-Detail: *wo* die Mapping-Arbeit liegt, verschiebt sich, sie verschwindet nicht.

> [!hint]- Lösungshinweis: `load_bookings` / `load_prices`
> ```python
>     def load_bookings(self) -> list[Booking]:
>         with Session(self._engine) as session:
>             rows = session.scalars(select(orm.Booking)).all()
>             return [
>                 Booking(
>                     product_id=r.product_id,
>                     start_date=r.start_date,
>                     nights=r.nights,
>                     status=r.status,
>                 )
>                 for r in rows
>             ]
>
>     def load_prices(self) -> list[Price]:
>         with Session(self._engine) as session:
>             rows = session.scalars(select(orm.Price)).all()
>             return [
>                 Price(
>                     product_id=r.product_id,
>                     change_date=r.change_date,
>                     length_of_stay=r.length_of_stay,
>                     price_sterling=r.current_price,
>                 )
>                 for r in rows
>             ]
> ```

**Checkpoint** (temporäres Skript oder REPL):
```python
from pricing.db import get_engine
from pricing.sources import SqlAlchemyDataSource
src = SqlAlchemyDataSource(get_engine())
print(len(src.load_capacity()), len(src.load_bookings()), len(src.load_prices()))
# grob: 1327560  939559  124528  (= dieselben Zahlen wie psycopg)
```

---

## Schritt 4 — `SqlAlchemyRecommendationWriter` (`writers.py`)

**Ziel:** Dritte Implementierung von `RecommendationWriter`. Gleiche Aufgabe wie der psycopg-Writer: FK `capacity_id` auflösen, alt löschen, neu schreiben — jetzt per ORM.

Das ist der interessanteste Schritt. Hier gebe ich dir das **Skelett mit drei Lücken**, weil er die härteste Stelle ist und du Zeit sparen sollst.

Oben in `writers.py` ergänzen:
```python
from sqlalchemy import insert, select, text
from sqlalchemy.engine import Engine
from sqlalchemy.orm import Session

from . import orm
```

Neue Klasse:
```python
class SqlAlchemyRecommendationWriter:
    "RecommendationWriter-Implementierung über SQLAlchemy-ORM."

    def __init__(self, engine: Engine):
        self._engine = engine

    def write(self, recs: list[PriceRecommendation]) -> None:
        with Session(self._engine) as session, session.begin():
            # (1) FK-Auflösung: {(product_id, calendar_date): capacity_id}
            rows = session.execute(
                select(orm.Capacity.id, orm.Capacity.product_id, orm.Capacity.calendar_date)
            ).all()
            capacity_ids = {(pid, cal): cid for cid, pid, cal in rows}

            # (2) Idempotenz: alte Empfehlungen weg (TRUNCATE ist rohes SQL)
            session.execute(text("TRUNCATE price_recommendations RESTART IDENTITY"))

            # (3) Zeilen als Liste von dicts bauen, fehlende Keys zählen/überspringen
            mappings: list[dict] = []
            missing = 0
            for r in recs:
                key = (r.product_id, r.calendar_date)
                cid = capacity_ids.get(key)
                if cid is None:
                    missing += 1
                    continue
                # TODO: ein dict an mappings.append(...) mit den Ziel-Spalten:
                #   capacity_id, days_until_travel, recommended_adjustment_pct,
                #   occupancy, total_capacity, booked_units
                ...

            # (4) Bulk-Insert (executemany über die ORM-Tabelle)
            if mappings:
                session.execute(insert(orm.PriceRecommendation), mappings)

            if missing:
                logger.warning(
                    "%d Recommendations ohne passende capacity-Zeile übersprungen.",
                    missing,
                )
```

Erklärung der SQLAlchemy-Teile, die neu für dich sind:
- **`with Session(...) as session, session.begin():`** — `session.begin()` öffnet eine Transaktion, die beim fehlerfreien Verlassen des Blocks **committet** (und bei Exception zurückrollt). Das ist das ORM-Pendant zu deinem `conn.transaction()` bei psycopg.
- **`session.execute(insert(Model), list_of_dicts)`** — die 2.0-Bulk-Insert-Form: ein `INSERT` mit `executemany` über alle dicts. Das ist der ORM-Weg, viele Zeilen in einem Rutsch zu schreiben. (Bewusst **nicht** `session.add_all([...])` mit lauter Objekten — das wäre die „reine" Unit-of-Work-Variante, aber langsamer; siehe Diskussion.)

> [!note] Das Aha-Detail für die Diskussion
> Schau genau hin: Die **FK-Auflösung ist exakt dasselbe Lookup-Dict wie bei psycopg** (`writers.py` Teil 1). Das ORM nimmt dir dieses Problem **nicht** ab — es ändert nur, wie du `SELECT`/`INSERT` schreibst. Merk dir das; es ist eines deiner stärksten Diskussions-Argumente („ORM ≠ automatisch weniger Denkarbeit bei Fremdschlüsseln").

> [!hint]- Lösungshinweis: das `mappings.append(...)`-dict
> ```python
>                 mappings.append(
>                     {
>                         "capacity_id": cid,
>                         "days_until_travel": r.days_until_travel,
>                         "recommended_adjustment_pct": r.recommended_adjustment_pct,
>                         "occupancy": r.occupancy,
>                         "total_capacity": r.total_capacity,
>                         "booked_units": r.booked_units,
>                     }
>                 )
> ```
> Die Schlüssel sind die **Spaltennamen** (= ORM-Attributnamen). `id` und `computed_at` lässt du weg → DB vergibt/füllt.

**Checkpoint:** siehe Schritt 5 (End-to-End).

---

## Schritt 5 — Verdrahten & Ende-zu-Ende vergleichen

**Ziel:** `PricingApp` einmal komplett über den ORM-Adapter laufen lassen und die Ausgabe gegen den psycopg- und CSV-Lauf halten.

**Aufgabe:** Ein kleines Skript (z. B. `scratch_orm.py` im Projektwurzel — analog zu `scratch_test.py`, **nicht** committen):
```python
import logging
from datetime import date

from pricing.app import PricingApp
from pricing.builder import PriceableUnitBuilder
from pricing.db import get_engine
from pricing.recommendations import HeuristicRecommendationStrategy, RecommendationEngine
from pricing.sources import SqlAlchemyDataSource
from pricing.writers import SqlAlchemyRecommendationWriter

logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")

REF = date(2026, 7, 22)   # DASSELBE reference_date wie in scratch_test.py!
engine = get_engine(echo=False)   # echo=True, wenn du das SQL sehen willst

app = PricingApp(
    source=SqlAlchemyDataSource(engine),
    builder=PriceableUnitBuilder(),
    engine=RecommendationEngine(HeuristicRecommendationStrategy(), reference_date=REF),
    writer=SqlAlchemyRecommendationWriter(engine),
)
app.calculate_all_price_recommendations()
print(f"Höchste Empfehlung (SQLAlchemy): {app.get_max_recommendation()}")
engine.dispose()
```

**Checkpoint (der eigentliche Beweis):**
```bash
python scratch_orm.py
```
Die „Höchste Empfehlung" muss **identisch** zu deinem psycopg-Lauf (`scratch_test.py`) sein — gleiches `REF`, gleiche DB, gleiche Fachlogik → gleiches Ergebnis. Und `SELECT count(*) FROM price_recommendations;` muss dieselbe Zahl liefern wie nach dem psycopg-Writer. Die `missing`-Warnung sollte **nicht** auftauchen (Quelle und Writer zeigen auf dieselbe DB).

> [!success] Wenn das grün ist, ist Teil 2 fachlich fertig.
> Drei Adapter (CSV, psycopg, SQLAlchemy) hinter zwei Protocols, `app.py` nie angefasst. Der Rest ist Diskussion + Aufräumen.

---

## Schritt 6 — *Optional/Stretch:* ORM-Ingest gegen `COPY` messen

Nur wenn Zeit ist. **Ziel:** am eigenen Leib spüren, warum `COPY` existiert. Schreib einen Mini-ORM-Loader für **eine** Tabelle (z. B. `capacity`, 1,33 Mio Zeilen) via `session.execute(insert(orm.Capacity), list_of_dicts)` und stoppe die Zeit gegen die ~110 s des psycopg-`COPY`-Ingests. Erwartung: ORM-`executemany` ist **deutlich** langsamer, `session.add_all()` mit echten Objekten nochmal langsamer. Das ist kein Bug — es ist die Lektion.

---

## Schritt 7 — Diskussion Vor-/Nachteile (Teil-2-Pflicht)

Das verlangt die Aufgabe explizit. Formulier es selbst aus — hier das Gerüst aus dem, was du in Schritt 1–6 konkret erlebt hast (nicht aus dem Lehrbuch abgeschrieben):

**Pro ORM:**
- Weniger handgeschriebenes SQL für Standard-CRUD; Query in Python-Ausdrücken (`select(orm.Capacity)`), Refactoring-fest.
- Automatisches Typ-Mapping (`Numeric`→`Decimal`, `Date`→`date`) — wie psycopg, aber ohne SQL-Aliasing.
- DB-Dialekt austauschbar (Postgres→SQLite für Tests) ohne SQL umzuschreiben.
- Unit of Work / Identity Map / Beziehungen (`relationship`) — zahlt sich bei komplexen Objektgraphen aus (hier kaum genutzt → schwaches Argument *für dieses Projekt*, ehrlich benennen).
- Migrationen via Alembic, wenn das ORM das Schema besäße.

**Contra ORM:**
- **Kein `COPY`** — Massen-Insert ist prinzipiell langsamer (Schritt 6 belegt es). Für ETL über Millionen Zeilen der falsche Hammer.
- Abstraktionsleck: `TRUNCATE` musstest du trotzdem als rohes SQL (`text()`) machen; die FK-Auflösung blieb dasselbe Lookup-Dict.
- „Magie": ohne `echo=True` siehst du nicht, welches SQL wirklich läuft; N+1-Fallen lauern.
- Lernkurve/zusätzliche Dependency für einen Vorteil, der bei *diesem* schlanken Lese-/Schreib-Fall klein ist.

**Fazit-Aufhänger (dein eigenes Urteil):** Für diesen Anwendungsfall — massives ETL + schlanke, klar definierte Lese-/Schreibpfade — spielt der psycopg/`COPY`-Weg seine Stärken aus, während das ORM seine (Objektgraphen, Migrationen, Dialekt-Portabilität) hier kaum ausspielen kann. Das ist **kein** „ORM ist schlechter", sondern *„Werkzeug zum Problem"*.

---

## Definition of Done (Teil 2)
- [x] `orm.py` mit 4 Modellen; `python -c "from pricing import orm"` läuft.
- [x] `get_engine()` in `db.py`; `select 1`-Checkpoint grün.
- [x] `SqlAlchemyDataSource` liefert dieselben Zeilenzahlen wie psycopg.
- [x] `SqlAlchemyRecommendationWriter` schreibt; „Höchste Empfehlung" == psycopg-Lauf, keine `missing`-Warnung.
- [ ] Diskussion (Schritt 7) ausformuliert. — **nicht erfolgt**, siehe [[Aufgehobene offene Punkte (Kapitel 1)#Die Diskussion „ORM vs. rohes SQL" wurde nie ausformuliert|Aufgehobene offene Punkte]]
- [x] Aufräumen: `sqlalchemy` im `orm`-Extra; `scratch_orm.py` **nicht** committed (vgl. Teil-1-Review, Punkt Repo-Hygiene).
- [x] `ruff check` + `mypy` grün.

> [!question] Wenn du hängst
> Sag mir die Fehlermeldung + welchen Schritt — dann reviewe/korrigiere ich gezielt, statt dass ich dir die Lösung vorwegnehme. Das ist der vereinbarte Arbeitsmodus.

---
# Teil C — SQLAlchemy-Referenz (warum das alles funktioniert)

> [!abstract] Zweck
> Nachschlagewerk zu genau den SQLAlchemy-2.0-Bausteinen, die du in Teil B benutzt hast. Jeder Abschnitt verweist auf die Stelle in *deinem* Code (`orm.py`, `db.py`, `sources.py`, `writers.py`), damit „warum funktioniert das" beim Nachlesen klickt. Schwerpunkt unten auf `execute`/`scalars`/`scalar`, weil das die verwirrendste Stelle ist.

## 1. Die zwei Ebenen: `Engine` vs. `Session`

Das ist das Fundament — die häufigste Anfänger-Verwirrung.

- **`Engine`** (aus `create_engine(url)`, `db.py`): ein langlebiger, thread-sicherer **Verbindungs-Pool** + Dialekt-Wissen. Du baust sie **einmal** und injizierst sie. Sie öffnet selbst keine Transaktion.
- **`Session`** (`with Session(engine) as session:`): die **Unit of Work** — kurzlebig, für *eine* Arbeitseinheit. Sie *leiht* sich eine Verbindung aus dem Pool, hält die **Identity Map** (Cache je Primärschlüssel innerhalb der Session) und gibt die Verbindung beim Verlassen des `with` zurück.

Deshalb injizierst du in `SqlAlchemyDataSource`/`-Writer` die **Engine** und öffnest je Methode eine frische `Session` — nicht umgekehrt. (Kontrast zu psycopg: dort injizierst du eine offene `Connection`; das ORM schiebt mit der Session eine zusätzliche Schicht dazwischen.)

## 2. Die Modelle: `DeclarativeBase` / `Mapped` / `mapped_column` (`orm.py`)

```python
class Base(DeclarativeBase): pass          # Registry-Wurzel; sammelt in Base.metadata

class Capacity(Base):
    __tablename__ = "capacity"             # bindet Klasse an bestehende Tabelle
    id: Mapped[int] = mapped_column(BigInteger, primary_key=True)
    product_id: Mapped[str]                # blosse Annotation reicht
    current_price: Mapped[Decimal] = mapped_column(Numeric(12, 2))  # Detail nötig
```

- **`Mapped[X]`** ist die typisierte Annotation: sie sagt SQLAlchemy *und* mypy den Python-Typ **und** dass die Spalte gemappt ist. Typ-Ableitung: `int`→`Integer`, `str`→`String`, `date`→`Date`, `float`→`Float`, `bool`→`Boolean`.
- **`mapped_column(...)`** brauchst du nur für **Extras**: `primary_key=True`, `ForeignKey("capacity.id")`, oder eine Typ-Präzisierung, die die Annotation nicht ausdrückt — v. a. **`Numeric(p, s)`** (Nachkommastellen!) und `BigInteger`. Ohne Extras genügt die nackte Annotation (`product_id: Mapped[str]`).
- **Kein `create_all()`**: Die Klassen *beschreiben* die schon existierenden Tabellen (fürs Abfragen/Einfügen). Deshalb mappst du nur die Spalten, die du brauchst, und ignorierst `GENERATED`/`IDENTITY`/`CHECK` gefahrlos.

## 3. Abfrage *bauen*: `select()`

`select(orm.Capacity)` erzeugt nur ein **Statement-Objekt** — es läuft noch nichts. Zwei Formen, die du benutzt hast, mit wichtigem Unterschied:
- `select(orm.Capacity)` — wählt eine **Entität** → Ergebnis sind `Capacity`-Objekte.
- `select(orm.Capacity.id, orm.Capacity.product_id, orm.Capacity.calendar_date)` — wählt **einzelne Spalten** → Ergebnis sind Tupel-artige `Row`s.

Diese Unterscheidung entscheidet, ob du gleich `scalars` oder `execute` nimmst (nächster Abschnitt).

## 4. Abfrage *ausführen*: `execute` vs. `scalars` vs. `scalar` ⭐

Ausgeführt wird über die Session. Das Ergebnis ist zunächst ein **`Result`** — ein Iterator über **`Row`**-Objekte (tuple-artig). Ein `Row` hat so viele Elemente, wie das `select` „Dinge" auswählt.

```python
# select EINER Entität -> jede Row ist ein 1-Tupel (Capacity,)
session.execute(select(orm.Capacity)).all()      # -> list[Row], jede Row[0] = Capacity
```
Das ist umständlich, wenn du nur die Objekte willst. Dafür gibt es **`scalars()`**: es schält **das erste Element jeder Row** heraus.

```python
# .scalars() -> ScalarResult, das direkt die Objekte liefert:
session.scalars(select(orm.Capacity)).all()      # -> list[Capacity]   <-- dein load_capacity
```
`session.scalars(stmt)` ist Kurzform für `session.execute(stmt).scalars()`.

**Deshalb die zwei verschiedenen Muster in deinem Code:**

| Stelle | Query | Aufruf | Warum |
|---|---|---|---|
| `load_capacity` (`sources.py`) | `select(orm.Capacity)` — **eine** Entität | `session.scalars(...).all()` | du willst die Objekte direkt, `scalars` schält das 1-Tupel weg |
| Writer FK-Lookup (`writers.py`) | `select(Capacity.id, .product_id, .calendar_date)` — **drei** Spalten | `session.execute(...).all()` | du willst das ganze 3-Tupel: `for cid, pid, cal in rows` |

> [!warning] Die Falle
> Hättest du im Writer `scalars()` genommen, bekämest du **nur die erste Spalte** (`id`) — die anderen zwei wären weg. Faustregel: **`scalars` bei *einer* Sache, `execute` bei *mehreren* Spalten.**

**Die Abschluss-Methoden** (auf `Result` *und* `ScalarResult` verfügbar):

| Methode | liefert | wenn 0 / >1 |
|---|---|---|
| Iteration (`for x in result`) | Zeilen einzeln (streamt, keine Zwischenliste) | — |
| `.all()` | `list[...]` | leere Liste ok |
| `.first()` | erste Zeile oder `None` | >1 ok (Rest verworfen) |
| `.one()` | genau eine Zeile | wirft `NoResultFound` / `MultipleResultsFound` |
| `.one_or_none()` | eine oder `None` | wirft bei >1 |

Und die **`scalar*`-Kurzformen** auf `Result` (Row → direkt der Einzelwert):

| Methode | ≈ | typischer Einsatz |
|---|---|---|
| `.scalar()` | `.scalars().first()` | Einzelwert-Query, z. B. `session.execute(select(func.count()).select_from(orm.Booking)).scalar()` |
| `.scalar_one()` | `.scalars().one()` | „exakt ein Wert erwartet" (streng) |
| `.scalar_one_or_none()` | `.scalars().one_or_none()` | Einzelwert oder nichts |

**Merksatz zu `scalars`/`scalar`:** *`scalars` = „gib mir die erste Spalte **jeder** Zeile" (Plural). `scalar` = „gib mir die erste Spalte der **einen** Zeile" (Singular).*

**`Row`-Zugriff**, falls du mal doch mit Tupeln arbeitest: per Index `row[0]`, per Entpacken `cid, pid, cal = row`, oder per Label `row.id` / `row.product_id`.

## 5. Schreiben: `insert()` als Bulk (`writers.py`)

```python
session.execute(insert(orm.PriceRecommendation), mappings)   # mappings: list[dict]
```
- `insert(Model)` baut ein INSERT-Statement; mit einer **Liste von dicts** als zweitem Argument wird es zu **`executemany`** — ein INSERT, viele Parametersätze. Das ist der 2.0-Weg für „viele Zeilen auf einmal".
- `id`/`computed_at` lässt du in den dicts **weg** → die DB vergibt/füllt sie (IDENTITY / `DEFAULT now()`).
- **Kontrast** zum „reinen" Unit-of-Work-Weg `session.add_all([PriceRecommendation(...), ...])`: der baut echte ORM-Objekte, führt sie durch die Identity Map und flusht — bequemer bei Objektgraphen, aber **langsamer** bei Massen-Insert. Für reine Zeilen ist `insert(), [dicts]` schlanker.
- `insert().from_select(...)` wäre die set-basierte Variante (INSERT direkt aus einem SELECT-JOIN) — siehe [[FK-Auflösung - App-Speicher vs. Datenbank|FK-Auflösung-Notiz]].

## 6. Transaktionen: `Session` + `session.begin()`

```python
with Session(self._engine) as session, session.begin():
    ...   # bei sauberem Verlassen: COMMIT; bei Exception: ROLLBACK
```
- `session.begin()` liefert einen Transaktions-Kontext, der beim fehlerfreien Blockende **committet**, bei Exception **zurückrollt**. Das ist das ORM-Pendant zu psycopgs `conn.transaction()`.
- **Ohne** `begin()`: die Session „autobegint" bei der ersten Nutzung, und du müsstest am Ende selbst `session.commit()` rufen. Das doppelte `with … , session.begin():` macht die Grenze explizit — wie im psycopg-Writer.

## 7. `text()` — die Notluke für rohes SQL

```python
session.execute(text("TRUNCATE price_recommendations RESTART IDENTITY"))
```
`TRUNCATE` ist keine ORM-Operation. `text("…")` führt beliebiges SQL aus. Völlig legitim, ORM und rohes SQL zu mischen — man greift zur Notluke, wenn ein Konstrukt (wie `TRUNCATE`) sich nicht lohnt oder gar nicht als ORM-Ausdruck existiert.

## 8. Mini-Spickzettel

| Ich will… | …das benutze ich |
|---|---|
| Verbindung/App-Objekt | `create_engine(url)` → `Engine` (injizieren) |
| Arbeitseinheit / Transaktion | `with Session(engine) as s, s.begin():` |
| Tabelle beschreiben | Klasse(Base) + `__tablename__` + `Mapped[..]`/`mapped_column(..)` |
| Abfrage bauen | `select(Entity)` oder `select(col1, col2, …)` |
| **Objekte** zurück | `session.scalars(select(Entity)).all()` |
| **Mehrere Spalten** zurück | `session.execute(select(a, b)).all()` → `for a, b in rows` |
| **Ein einzelner Wert** | `session.execute(select(func.count()…)).scalar()` |
| viele Zeilen schreiben | `session.execute(insert(Model), list_of_dicts)` |
| rohes SQL | `session.execute(text("…"))` |
| sehen, was wirklich läuft | `create_engine(url, echo=True)` |
