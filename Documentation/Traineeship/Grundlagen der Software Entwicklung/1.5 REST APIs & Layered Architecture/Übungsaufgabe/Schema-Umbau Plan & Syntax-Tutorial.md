---
tags:
  - documentation
type: tutorial
date: 2026-08-04
---

Abhakbarer Plan für den Umbau **„SQLAlchemy besitzt das Schema"** — inklusive Umstellung des Ingests. Begleitnotiz zum [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]]. Vorgeschichte: der Spike [[ORM besitzt das Schema (Spike) - Tutorial|„ORM besitzt das Schema"]] aus Kapitel 1.2.

**Erklärung der Konstrukte** (`Identity`, `Computed`, `server_default`, Constraints, `relationship`): [[SQLAlchemy Schema & Beziehungen - Syntax|SQLAlchemy Schema & Beziehungen – Syntax]] · Abfrageseite: [[SQLAlchemy Tutorial|SQLAlchemy Tutorial]] aus Kapitel 1.2.

> [!info] So arbeitest du mit dieser Liste
> Die Blöcke sind **von einfach nach komplex** sortiert; jeder führt genau eine neue Technik ein (mit Kurzerklärung + Code-Muster). Der Reihe nach abarbeiten, `- [ ]` abhaken. **Du schreibst den Code, Claudian reviewt.** Die Schnipsel sind *Muster*, keine fertigen Lösungen.

## Was sich ändert — und was ausdrücklich nicht

Du bewegst dich auf **einer** Achse. Der Schema-*Umfang* bleibt, wie er ist:

| | Schema schlank (Domäne) | Schema voll (System of Record) |
|---|---|---|
| DDL gehört SQL | — | **Ausgangspunkt** |
| DDL gehört SQLAlchemy | Spike aus 1.2 (verworfen) | **Ziel** ⭐ |

Die 1.2-Entscheidung („System of Record, volles Schema") bleibt also bestehen — `products`, `customers`, Umsatzspalten, Fremdschlüssel, alles bleibt. Es ändert sich nur, **wer die DDL schreibt**, und dass es künftig **Migrationen** gibt.

> [!important] Alembic kommt bewusst **nicht** jetzt (Entscheidung 2026-08-04)
> Migrationen sind Stoff des **nächsten Kapitels**. Sie hier vorwegzunehmen bringt derzeit wenig und kostet zweifach:
>
> - **`create_all()` reicht für den aktuellen Bedarf.** Es legt fehlende Tabellen an und überspringt vorhandene — es scheitert nur am *Ändern* bestehender Tabellen. Die anstehende `pricing_runs`-Tabelle für die API ist eine **neue** Tabelle und damit unproblematisch.
> - **Der Preis einer Schemaänderung ist hier niedrig:** keine schützenswerten Daten (alles kommt aus den CSVs), ein Entwickler, keine zweite Umgebung. „Ändern" heißt `dropdb` + `createdb` + `create_all()` + neu ingesten.
> - **Man würde es schlechter lernen.** Nebenbei eingerichtet bleibt es bei `init` + `--autogenerate`; die interessanten Teile (Migrationen lesen, Autogenerate-Fallen wie „Umbenennung = drop + add", `downgrade`, Bestandsdaten) fallen unter den Tisch.
>
> **Es geht nichts verloren:** Alembic später auf ein bestehendes Schema aufzusetzen ist ein Einzeiler (`alembic stamp head` bzw. eine initiale Migration) — und selbst Lernstoff. Umgekehrt startet das Migrationskapitel dann mit einem **echten, nicht-trivialen Schema in SQLAlchemy-Hand** statt mit einem Spielbeispiel.
>
> Die Blöcke 8 und 9 sind deshalb **zurückgestellt** und durch Block 8′ ersetzt.

Der Gewinn dieses Umbaus ist damit: **eine Quelle der Wahrheit** statt zwei unabgeglichener Beschreibungen — und ein bereiteter Boden für das Migrationskapitel.

---

# Block 0 — Vorbereitung

- [x] **Baseline messen.** `pricing-ingest` einmal laufen lassen und die Zeile *„Ingestion erfolgreich durchlaufen in X s"* notieren → hier eintragen: `56,26 s`
- [x] Branch: `git switch -c refactor/sqlalchemy-owns-schema`
- [x] **Referenzwert festhalten:** `python -m pricing.main -v` laufen lassen und die Zeile *„Höchste Empfehlung: …"* notieren → `____________`. Am Ende muss derselbe Wert herauskommen; das ist dein Beweis, dass sich am Schema nichts verschoben hat.
Höchste Empfehlung: PriceRecommendation(product_id='RFJA3D', calendar_date=datetime.date(2027, 4, 22), days_until_travel=261, recommended_adjustment_pct=Decimal('10.275000'), occupancy=1.0, total_capacity=3, booked_units=3)
*(Neue Abhängigkeiten braucht es keine — `alembic` ist zurückgestellt, SQLAlchemy ist bereits da.)*

---

# Block 1 — Naming Convention ⭐ *neue Technik*

**Warum zuerst:** Constraints und Indizes, die SQLAlchemy selbst benennt, bekommen kryptische oder gar keine Namen. Alembic kann eine namenlose Constraint später **nicht zuverlässig löschen** — dann steckst du in einer Migration fest. Das legt man einmal am Anfang fest, nie später.

```python
# persistence/orm.py
from sqlalchemy import MetaData
from sqlalchemy.orm import DeclarativeBase

NAMING_CONVENTION = {
    "ix": "ix_%(table_name)s_%(column_0_name)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}

class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=NAMING_CONVENTION)
```

- [x] Naming Convention in `Base` eintragen

> [!warning] Folge für alle `CheckConstraint`s
> Weil `"ck"` das Muster `%(constraint_name)s` benutzt, **muss jede** `CheckConstraint` ein `name=` bekommen. Ohne Namen bricht das Rendering mit einem unschönen Fehler ab.

---

# Block 2 — Die zwei einfachen Modelle ⭐ *ForeignKey-Ziele*

`products` und `customers` fehlen heute komplett in `orm.py` — sie existieren nur in `schema.sql`. Sie sind der leichteste Einstieg.

```python
from sqlalchemy import Text
from sqlalchemy.orm import Mapped, mapped_column

class Product(Base):
    __tablename__ = "products"
    product_id: Mapped[str] = mapped_column(Text, primary_key=True)
    bedroom_type: Mapped[str | None] = mapped_column(Text)
```

Zwei Dinge, die hier schon zählen:

- **`Mapped[str]` = `NOT NULL`, `Mapped[str | None]` = nullable.** Die Optionalität der Annotation steuert die Spalte — man schreibt kein `nullable=` mehr.
- **`Text` statt Default.** Ohne expliziten Typ macht SQLAlchemy aus `Mapped[str]` ein `VARCHAR`. Dein Schema benutzt durchgängig `TEXT` — das musst du explizit sagen, sonst weicht der Diff in Block 7 ab.

- [x] `Product`
- [x] `Customer` (nur `cust_id`, PK)

---

# Block 3 — `capacity` vollständig ⭐ *Identity, UniqueConstraint, Index*

```python
from sqlalchemy import BigInteger, Date, ForeignKey, Identity, Index, UniqueConstraint

class Capacity(Base):
    __tablename__ = "capacity"
    __table_args__ = (
        UniqueConstraint("product_id", "calendar_date"),
        Index("ix_capacity_calendar_date", "calendar_date"),
    )

    id: Mapped[int] = mapped_column(BigInteger, Identity(always=True), primary_key=True)
    product_id: Mapped[str] = mapped_column(Text, ForeignKey("products.product_id"))
    calendar_date: Mapped[date]
    bookable_units: Mapped[int]
    optional_units: Mapped[int]
```

- **`Identity(always=True)`** ist das Gegenstück zu `BIGINT GENERATED ALWAYS AS IDENTITY`. (Nicht `autoincrement` — das erzeugt in Postgres eine Sequence, also etwas anderes.)
- **`__table_args__`** ist ein **Tupel** — bei genau einem Eintrag das Komma nicht vergessen: `(UniqueConstraint(...),)`.

- [x] `Capacity` mit FK, UNIQUE und Index

---

# Block 4 — `prices` ⭐ *CheckConstraint, Numeric*

```python
from sqlalchemy import CheckConstraint, Numeric

class Price(Base):
    __tablename__ = "prices"
    __table_args__ = (
        CheckConstraint("current_price >= 0", name="current_price_non_negative"),
        Index("ix_prices_product_id", "product_id"),
    )

    id: Mapped[int] = mapped_column(BigInteger, Identity(always=True), primary_key=True)
    source_id: Mapped[str | None] = mapped_column(Text)
    product_id: Mapped[str] = mapped_column(Text, ForeignKey("products.product_id"))
    change_date: Mapped[date]
    current_price: Mapped[Decimal] = mapped_column(Numeric(12, 2))
    length_of_stay: Mapped[str] = mapped_column(Text)
```

Neu gegenüber heute: `source_id` (fehlt im aktuellen Modell) und der Check.

- [x] `Price` vollständig

---

# Block 5 — `bookings` ⭐ *Computed, server_default* — der dickste Brocken

15 Spalten statt heute 5. Zwei Konstrukte sind neu:

```python
from sqlalchemy import Computed, text

class Booking(Base):
    __tablename__ = "bookings"
    __table_args__ = (
        CheckConstraint(
            "status IN ('Firm', 'Cancelled', 'Expired', 'Pending')", name="status_valid"
        ),
        CheckConstraint("nights >= 1", name="nights_positive"),
        Index("ix_bookings_product_id", "product_id"),
        Index("ix_bookings_cust_id", "cust_id"),
    )

    id: Mapped[int] = mapped_column(BigInteger, Identity(always=True), primary_key=True)
    # … source_id, bkg_id, product_id (FK), cust_id (FK), start_date,
    #   booking_creation_date, status, cancellation_date,
    #   gross_revenue, discount_amount …

    net_revenue: Mapped[Decimal | None] = mapped_column(
        Numeric(12, 2),
        Computed("gross_revenue - discount_amount", persisted=True),
    )
    base_currency: Mapped[str] = mapped_column(Text, server_default=text("'STG'"))
    nights: Mapped[int]
    feature_2: Mapped[str | None] = mapped_column(Text)
```

> [!warning] Die `server_default`-Falle
> `server_default="STG"` erzeugt `DEFAULT STG` — Postgres liest das als **Bezeichner**, nicht als Zeichenkette, und die DDL scheitert. Richtig ist `server_default=text("'STG'")` mit inneren Anführungszeichen.

**`Computed(..., persisted=True)`** entspricht `GENERATED ALWAYS AS (…) STORED`. Wichtige Folge: In diese Spalte darf **nie** geschrieben werden — der Ingest lässt sie schon heute korrekt weg.

- [x] `Booking` mit allen 15 Spalten, 2 FKs, 2 Checks, 2 Indizes

---

# Block 6 — `price_recommendations` ⭐ *TIMESTAMPTZ*

```python
from datetime import datetime
from sqlalchemy import DateTime, func

class PriceRecommendation(Base):
    __tablename__ = "price_recommendations"
    __table_args__ = (Index("ix_price_recommendations_capacity_id", "capacity_id"),)

    id: Mapped[int] = mapped_column(BigInteger, Identity(always=True), primary_key=True)
    capacity_id: Mapped[int] = mapped_column(BigInteger, ForeignKey("capacity.id"))
    computed_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now()
    )
    # … days_until_travel, recommended_adjustment_pct, occupancy,
    #   total_capacity, booked_units
```

`DateTime(timezone=True)` ist `TIMESTAMPTZ`. **`computed_at` fehlt heute im Modell** — mit dem Umbau kommt es dazu und steht der API später zur Verfügung.

- [x] `PriceRecommendation` inkl. `computed_at`

---

# Block 7 — Gegenprobe: stimmt das Schema überein? ⭐ *pg_dump-Diff*

Der wichtigste Schritt. Bevor `schema.sql` verschwindet, musst du wissen, dass die Modelle dasselbe beschreiben.

**Vorprüfung ohne Datenbank** — zeigt das SQL, das `create_all()` erzeugen würde:

```bash
./.venv/bin/python -c "
from sqlalchemy.schema import CreateTable, CreateIndex
from sqlalchemy.dialects import postgresql
from pricing.persistence.orm import Base
pg = postgresql.dialect()
for t in Base.metadata.sorted_tables:
    print(str(CreateTable(t).compile(dialect=pg)).strip() + ';')
    for ix in sorted(t.indexes, key=lambda i: i.name):
        print(str(CreateIndex(ix).compile(dialect=pg)).strip() + ';')
    print()
"
```

**Dann der Vergleich gegen die echte Datenbank.**

### 1. Datenbank starten und Vergleichsdatenbank anlegen

```bash
cd ~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/REST_API
docker compose up -d db
docker exec pricing-db createdb -U pricing pricing_check
```

### 2. Schema aus den Modellen in `pricing_check` anlegen

```bash
./.venv/bin/python -c "
from sqlalchemy import URL, create_engine
from pricing.persistence.orm import Base
url = URL.create('postgresql+psycopg', username='pricing', password='pricing',
                 host='localhost', database='pricing_check')
Base.metadata.create_all(create_engine(url))
print('OK: Schema aus den Modellen in pricing_check angelegt')
"
```

### 3. Spalten vergleichen

```bash
SQL_COLS="SELECT table_name, column_name, data_type,
 coalesce(character_maximum_length::text,'-'),
 coalesce(numeric_precision::text,'-')||'/'||coalesce(numeric_scale::text,'-'),
 is_nullable, coalesce(column_default,'-'),
 is_identity, coalesce(identity_generation,'-'),
 is_generated, coalesce(generation_expression,'-')
FROM information_schema.columns WHERE table_schema='public'
ORDER BY table_name, column_name"

for db in pricing pricing_check; do
  docker exec pricing-db psql -U pricing -d $db -At -F'|' -c "$SQL_COLS" > /tmp/cols_$db.txt
done
diff /tmp/cols_pricing.txt /tmp/cols_pricing_check.txt && echo "OK Spalten identisch"
```

### 4. Constraints vergleichen — **ohne Namen**

```bash
SQL_CONS="SELECT conrelid::regclass::text, contype, pg_get_constraintdef(oid)
FROM pg_constraint WHERE connamespace='public'::regnamespace ORDER BY 1,2,3"

for db in pricing pricing_check; do
  docker exec pricing-db psql -U pricing -d $db -At -F'|' -c "$SQL_CONS" > /tmp/cons_$db.txt
done
diff /tmp/cons_pricing.txt /tmp/cons_pricing_check.txt && echo "OK Constraints identisch"
```

### 5. Indizes vergleichen — **Namen herausgefiltert**

```bash
SQL_IDX="SELECT tablename, regexp_replace(indexdef, ' INDEX [^ ]+ ON ', ' INDEX ON ')
FROM pg_indexes WHERE schemaname='public' ORDER BY 1,2"

for db in pricing pricing_check; do
  docker exec pricing-db psql -U pricing -d $db -At -F'|' -c "$SQL_IDX" > /tmp/idx_$db.txt
done
diff /tmp/idx_pricing.txt /tmp/idx_pricing_check.txt && echo "OK Indizes identisch"
```

### 6. Aufräumen

```bash
docker exec pricing-db dropdb -U pricing pricing_check
```

> [!tip] Warum nicht einfach `pg_dump --schema-only` diffen?
> Weil dessen Ausgabe von **Namensunterschieden** überschwemmt wird: `schema.sql` erzeugt `capacity_pkey`, `bookings_status_check`, `idx_capacity_calendar_date`; die Naming Convention erzeugt `pk_capacity`, `ck_bookings_status_valid`, `ix_capacity_calendar_date`. Alles davon ist **egal** — es sind dieselben Constraints mit anderen Etiketten.
>
> Die drei Abfragen oben vergleichen deshalb genau das, was zählt: Spalten samt Typ, Nullability, Default, Identity und Generierungsausdruck; Constraint-**Definitionen** ohne Namen; Index-**Definitionen** ohne Namen. **Ziel: alle drei Diffs sind leer.** Wenn du zusätzlich alles sehen willst: `docker exec pricing-db pg_dump -U pricing --schema-only pricing > /tmp/ist.sql` (und analog für `pricing_check`).

- [x] Alle drei Diffs leer
- [x] `pricing_check` gelöscht

---

# Block 8′ — Schema anlegen per `create_all()`

Statt Alembic (siehe Kasten oben) reicht vorerst eine kleine Funktion — **nicht** beim Import, **nicht** beim Anwendungsstart, sondern als eigenes Kommando:

```python
# persistence/orm.py  (oder ein schlankes persistence/schema_admin.py)
def create_schema(engine: Engine) -> None:
    """Legt fehlende Tabellen an. Aendert bestehende NICHT."""
    Base.metadata.create_all(engine)
```

Dazu ein Entry Point, damit man es bewusst aufruft:

```toml
[project.scripts]
pricing-initdb = "pricing.persistence.schema_admin:main"
```

- **`create_all()` ist idempotent**, aber nur für *fehlende* Tabellen (`checkfirst=True` ist Default). Eine bestehende Tabelle wird **nie** angefasst — kein `ALTER`, keine Warnung, kein Hinweis. Genau hier fängt später Alembic an.
- **Warum nicht beim Anwendungsstart:** Eine API, die beim Hochfahren Tabellen anlegt, ist ein Betriebsrisiko — und in der Composition Root wäre es ein Seiteneffekt, der nicht dorthin gehört.

> [!warning] Die Regel, solange es kein Alembic gibt
> **Schemaänderung an einer bestehenden Tabelle = `dropdb pricing && createdb pricing && pricing-initdb && pricing-ingest`.**
> Das ist hier vertretbar, weil sämtliche Daten aus den CSVs stammen. Sobald das nicht mehr gilt, ist der Zeitpunkt für Migrationen gekommen — spätestens im nächsten Kapitel.

- [x] `create_schema()` + Entry Point `pricing-initdb`
- [x] `pip install -e .` (Entry Points werden beim Installieren geschrieben!)
- [x] `dropdb pricing && createdb pricing && pricing-initdb`
- [x] `psql pricing -c '\d bookings'` — alle 15 Spalten, Checks, Indizes vorhanden?

---

# Block 9 — *(zurückgestellt)* Alembic

Kommt mit dem nächsten Kapitel. Was dann zu tun ist, in Kurzform als Merkposten:

- `alembic init migrations`, in `env.py` `target_metadata = Base.metadata` setzen und die Engine über `get_engine()` beziehen (statt `sqlalchemy.url` in der `alembic.ini` — sonst stehen Zugangsdaten im Repo und die Konfiguration hat zwei Quellen)
- `compare_type=True` und `compare_server_default=True` bei `context.configure(...)` — beide sind **standardmäßig aus** und genau die Fälle, die später weh tun
- Bestehende DB per `alembic stamp head` als „ist schon auf diesem Stand" markieren, oder eine initiale Migration erzeugen
- Die **Naming Convention aus Block 1** ist die Voraussetzung dafür, dass Alembic Constraints später überhaupt zuverlässig löschen kann

---

# Block 10 — Ingest auf SQLAlchemy ⭐ *COPY über die SQLAlchemy-Connection*

Das Parsen, `_validate_net_revenue`, `_derive_products` usw. bleiben **unverändert**. Es ändern sich nur die fünf `_copy_*`-Funktionen und `run_ingest()`/`main()`.

**Signatur umstellen:** `run_ingest(engine: Engine, data_dir: Path)`.

```python
with engine.begin() as conn:                       # commit/rollback automatisch
    conn.execute(text("TRUNCATE products, customers, capacity, bookings, prices "
                      "RESTART IDENTITY CASCADE"))
    raw = conn.connection.driver_connection        # die echte psycopg-Connection
    with raw.cursor() as cur:
        _copy_products(cur, products)
        ...
```

- **`engine.begin()`** ersetzt `conn.transaction()` — der Block committet am Ende bzw. rollt bei einer Exception zurück.
- **`conn.connection.driver_connection`** gibt dir die darunterliegende psycopg-Verbindung. Damit behältst du `COPY`.

> [!important] Warum `COPY` bleibt
> `COPY` schafft grob 100.000–500.000 Zeilen/s, ORM-Inserts eher 5.000–20.000/s. Bei 1,33 Mio. Kapazitätszeilen und 940k Buchungen wären das Minuten statt Sekunden. SQLAlchemy besitzt danach Schema, Engine, Verbindung und Transaktion — der schnellste Schreibweg bleibt trotzdem erhalten. *(Für `products`/`customers` kannst du gerne Core-`insert()` nehmen; dort ist Tempo egal und du siehst beide Varianten.)*

**Spaltenlisten aus den Tabellen ableiten**, damit sie nicht mehr driften können:

```python
def copy_columns(table, exclude: frozenset[str] = frozenset()) -> list[str]:
    return [c.name for c in table.columns if c.name not in exclude]

COLS = copy_columns(Booking.__table__, frozenset({"id", "net_revenue"}))
stmt = f"COPY bookings ({', '.join(COLS)}) FROM STDIN"
```

Dann die Zeilen **in derselben Reihenfolge** schreiben — am robustesten über ein dict:

```python
values = {"source_id": r.source_id, "bkg_id": r.bkg_id, ...}
copy.write_row([values[c] for c in COLS])
```

So kann die Reihenfolge im `COPY`-String nie mehr von der Reihenfolge der Werte abweichen.

- [x] `run_ingest` auf `Engine` umgestellt
- [x] `_copy_*` nutzen den Cursor aus `driver_connection`
- [x] Spaltenlisten aus `__table__` abgeleitet (mindestens für `bookings`)
- [x] `main()` baut die Engine über `get_engine()`
- [x] `pricing-ingest` läuft, Dauer notiert: `____ s` (Baseline aus Block 0 vergleichen)

---

# Block 11 — psycopg-Adapter aufräumen

Jetzt hat nur noch der `COPY` im Ingest direkten psycopg-Bezug. Zur Disposition stehen:

| Was | Anmerkung |
|---|---|
| `db.connect()`, `db.get_dsn()` | nur noch von `scripts/smoke_test.py` und einem Test benutzt |
| `PostgresqlDataSource` | zweite Implementierung neben `SqlAlchemyDataSource` |
| `PostgresqlRecommendationWriter` | dito — enthält dieselbe `capacity_ids`-Auflösung wie die SQLAlchemy-Variante |
| `scripts/smoke_test.py` | hängt am psycopg-Weg |

**Empfehlung: entfernen.** Es sind Duplikate derselben Fachlichkeit, und Duplikate driften. Sie stammen aus Kapitel 1.2 als Lernübung — der Lerneffekt ist eingefahren und in der Vertiefung dokumentiert.

> [!warning] `tests/test_exception.py` hängt daran
> `test_fehlendes_passwort_wird_uebersetzt` ruft `get_dsn()` auf. Wenn `get_dsn()` verschwindet, den Test auf **`get_engine()`** umstellen — die ruft dasselbe `_require_password()` auf, der `ConfigError` wird also weiterhin geprüft.

- [x] Entscheidung getroffen und im Umsetzungsstand notiert
- [x] Falls entfernt: `test_exception.py` auf `get_engine()` umgestellt

---

# Block 12 — Abschluss

- [x] `git rm sql/schema.sql` — **eine** Quelle der Wahrheit, sonst war der Umbau umsonst
- [x] **`docker-compose.yml` anpassen:** Die Zeile `- ./sql:/docker-entrypoint-initdb.d:ro` spielt `schema.sql` beim ersten Start eines frischen Volumes automatisch ein. Ohne die Datei mountet sie ins Leere — und beim nächsten frischen Container fehlt das Schema. Mount entfernen; das Anlegen übernimmt ab jetzt `pricing-initdb`.
- [x] `./.venv/bin/ruff check .` · `./.venv/bin/mypy` · `./.venv/bin/python -m pytest -q` (57 grün)
- [x] `pricing-ingest` und `python -m pricing.main -v` laufen durch
- [x] **„Höchste Empfehlung" mit dem Wert von vor dem Umbau vergleichen** — sie muss identisch sein, sonst hat sich beim Schema etwas verschoben
- [x] Commit

---

# Fallensammlung

1. **`__table_args__` ist ein Tupel** — bei einem Eintrag das Komma nicht vergessen.
2. **Jede `CheckConstraint` braucht `name=`**, sobald die Naming Convention `%(constraint_name)s` benutzt.
3. **`server_default="STG"` ist falsch** — `text("'STG'")` ist richtig.
4. **`Mapped[str]` wird `VARCHAR`, nicht `TEXT`** — Typ explizit angeben.
5. **Die `Computed`-Spalte nie beschreiben.**
6. **`create_all()` ändert nichts an bestehenden Tabellen** — schweigend. Wer eine Spalte ergänzt und sich wundert, warum sie fehlt, hat diese Falle gefunden. Bis Alembic da ist: neu aufbauen.
7. **Entry Points werden beim Installieren geschrieben** — nach dem Ergänzen von `pricing-initdb` einmal `pip install -e .` (dieselbe Falle wie beim Struktur-Umbau).

# Definition of Done

- [x] `sql/schema.sql` ist weg, `Base.metadata` ist die einzige Schema-Quelle
- [x] Alle 6 Tabellen vollständig als Modelle, inkl. Constraints, Indizes und Naming Convention
- [x] `pricing-initdb` legt das Schema an; `pg_dump`-Gegenprobe zeigt keine semantischen Abweichungen zum alten `schema.sql`
- [x] Ingest läuft über SQLAlchemy-Engine und -Transaktion, `COPY` erhalten, Laufzeit vergleichbar zur Baseline
- [x] `ruff`/`mypy`/`pytest` grün, „Höchste Empfehlung" **identisch** zum Referenzwert aus Block 0
