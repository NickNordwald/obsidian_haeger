---
tags:
  - documentation
type: notiz
date: 2026-08-13
---

Übersicht über **alle Datenmodelle** der Pricing-Anwendung, ihre Schicht und die Wege zwischen den Schichten. Stand: Kapitelende 1.5, Commit `7acbcf0`.

**Als Bild:** [[Datenmodelle und Schichten.canvas|Datenmodelle und Schichten (Canvas)]] — dieselben Inhalte visuell, mit den Wegen und Brücken.

Verwandt: [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]] · [[REST API Plan & Syntax-Tutorial|REST API Plan]] · [[1.5 REST APIs & Layered Architecture - Vertiefung|Vertiefung]] · [[Schichtenarchitektur.canvas|Schichtenarchitektur (Canvas, Stand vor der API)]]

## Die Schichten

```
presentation  →  application  →  domain
       ↓              ↓            ↑
       └──────→  persistence  ─────┘

shared/      wird von allen benutzt (Querschnitt, keine Schicht)
bootstrap.py Composition Root, darf alles kennen
main.py      Composition Root, darf alles kennen
```

Tatsächlicher Importgraph, aus dem AST erhoben:

| Schicht | importiert |
|---|---|
| `domain` | `shared` |
| `persistence` | `domain`, `shared` |
| `application` | `domain`, `persistence`, `shared`, `bootstrap` |
| `presentation` | `application`, `persistence`, `shared`, `bootstrap` |
| `bootstrap` | `application`, `domain`, `persistence` |
| `main` | `bootstrap`, `shared` |

`domain` importiert außer `shared` nichts — die einzige Schicht ohne jede Abhängigkeit nach außen. Alle Abweichungen von der reinen Regel stehen unter [[#Schichtverletzungen im Überblick]], die Umbauanleitungen dazu unter [[#Offene Umbauten]].

---

# Alle Datenmodelle

## 1. Domain — `domain/entities.py`

Reines Python, keine Bibliothek. Kennt weder Datenbank noch HTTP.

| Klasse | Art | Felder |
|---|---|---|
| `CapacityRow` | frozen dataclass | `product_id`, `calendar_date`, `bookable_units`, `optional_units` |
| `Booking` | dataclass | `product_id`, `start_date`, `nights`, `status` |
| `Price` | frozen dataclass | `product_id`, `change_date`, `length_of_stay`, `price_sterling` |
| `PriceRecommendation` | dataclass, `order=True` | `product_id`, `calendar_date`, `days_until_travel`, `recommended_adjustment_pct`, `occupancy`, `total_capacity`, `booked_units` |
| `PriceableUnit` | Klasse mit Invarianten | `product_id`, `calendar_date`, `bookable_units`, `booked_units`, `optional_units` |

`PriceRecommendation` vergleicht nur nach `recommended_adjustment_pct` (alle anderen Felder haben `compare=False`) — deshalb funktioniert `max(recs)` direkt.

`PriceableUnit` ist die einzige Klasse mit Verhalten statt nur Daten: Sie prüft ihre Invarianten im Konstruktor und rechnet `occupancy`.

## 2. Persistenz — `persistence/orm.py`

SQLAlchemy-Modelle. **Sie besitzen das Schema**, `create_all()` erzeugt daraus die Tabellen.

| Klasse | Tabelle | Zweck |
|---|---|---|
| `Product` | `products` | buchbare Einheit, PK `product_id` |
| `Customer` | `customers` | PK `cust_id` |
| `Capacity` | `capacity` | Kapazität je Einheit und Tag, `UNIQUE (product_id, calendar_date)` |
| `Booking` | `bookings` | Buchungen, FK auf `products` und `customers` |
| `Price` | `prices` | Preisstand, FK auf `products` |
| `PriceRecommendation` | `price_recommendations` | Ergebnis eines Preislaufs, FK auf `capacity` |
| `ApiKey` | `api_keys` | Name + SHA-256-Hash, `revoked_at` |
| `PricingRun` | `pricing_runs` | Status eines Preislaufs |
| `IngestionRun` | `ingestion_runs` | Status eines Ingest-Laufs |

> **Namensgleichheit beachten:** `orm.Booking` ≠ `domain.entities.Booking`, `orm.PriceRecommendation` ≠ `domain.entities.PriceRecommendation`, `orm.Price` ≠ `domain.entities.Price`. Gleicher Name, andere Schicht, andere Felder. Im Code werden sie über den Modulpräfix unterschieden (`orm.Booking` vs. `Booking`).

Unterschiede am Beispiel `PriceRecommendation`:

| | domain | orm |
|---|---|---|
| Einheit | `product_id` + `calendar_date` direkt | `capacity_id` als FK |
| Zeitstempel | keiner | `computed_at` |
| Identität | keine | `id` |

## 3. Persistenz — CSV-Zeilenmodelle in `persistence/ingest.py`

| Klasse | Zweck |
|---|---|
| `BookingRow` | eine Zeile aus `*_bookings.csv`, 15 Felder, alle schon getypt |
| `PriceRow` | eine Zeile aus `*_prices.csv` |

Diese Modelle spiegeln die **Dateiform**, nicht das Datenmodell. Sie existieren nur zwischen Parsen und `COPY` und verlassen `ingest.py` nie. `CapacityRow` wird aus `domain` wiederverwendet, weil die CSV-Spalten dort zufällig genau passen.

## 4. Application — `application/pricing_app.py`

| Klasse | Zweck |
|---|---|
| `PricingRunResult` | Ergebnis *eines* Laufs: `recommendations`, `max_recommendation`, `duration_seconds` |

Kein Datenmodell im engeren Sinn, sondern ein Rückgabebündel, damit der Aufrufer nicht neu rechnen muss.

## 5. Presentation — `presentation/api/schemas/`

Pydantic-Modelle. Ein Modell **pro Richtung**, nicht pro Entität.

| Datei | Klasse | Richtung |
|---|---|---|
| `product.py` | `ProductRead` | raus |
| `booking.py` | `BookingRead` | raus |
| | `BookingCreate` | rein (`POST`) |
| | `BookingUpdate` | rein (`PUT`) |
| `capacity.py` | `CapacityWrite` | rein (`POST` und `PUT`) |
| | `CapacityRead(CapacityWrite)` | raus, erbt und ergänzt `id` |
| `recommendation.py` | `RecommendationRead` | raus |
| `pricing_run.py` | `RunRead` | raus |
| `ingestion_run.py` | `IngestionRunCreate` | rein |
| | `IngestionRunRead` | raus |

Jedes `…Read` hat `model_config = ConfigDict(from_attributes=True)` — sonst kann `model_validate()` keine ORM-Objekte lesen.

Warum Create und Update getrennt sind: `POST` wird mit `model_dump(exclude_none=True)` ausgelesen (weggelassene Felder überlassen der Datenbank ihren Default), `PUT` mit `model_dump()` (weggelassene Felder werden bewusst `null`, weil `PUT` ersetzt).

Was die DTOs **wegfiltern**: `BookingRead` zeigt kein `cust_id` und kein `feature_2`, `RecommendationRead` kein `capacity_id`, `computed_at`, `total_capacity` oder `booked_units`. Die Spalte existiert, der Client sieht sie nicht.

`RecommendationRead` holt zwei Felder aus der verknüpften Kapazitätszeile:

```python
product_id: str = Field(validation_alias=AliasPath("capacity", "product_id"))
```

`AliasPath` liest `objekt.capacity.product_id` statt `objekt.product_id`. Es ist ein **validation**-Alias, wirkt also nur beim Lesen: Nach außen bleibt die Antwort flach, `/openapi.json` zeigt `product_id` auf oberster Ebene.

---

# Wege durch die Schichten

## A. Ingest: CSV → Datenbank

```
*.csv
  → csv.DictReader                     ingest.py
  → BookingRow / PriceRow / CapacityRow
  → COPY bzw. INSERT                   ingest.py, eigene Transaktion
  → Tabellen products, customers, capacity, bookings, prices
```

Die ORM-Klassen werden hier nur als **Tabellenbeschreibung** benutzt (`_table(orm.Booking)`), nicht als Objekte instanziiert. `COPY` schreibt an der ORM-Schicht vorbei, weil es um Größenordnungen schneller ist.

## B. Preislauf: Datenbank → Empfehlungen

```
capacity, bookings
  → SqlAlchemyDataSource.load_*()      persistence/sources.py
  → domain.CapacityRow, domain.Booking     ← Übersetzung ORM → domain
  → PriceableUnitBuilder.build()       domain/builder.py
  → PriceableUnit
  → RecommendationEngine.calculate_all()
  → domain.PriceRecommendation
  → SqlAlchemyRecommendationWriter     persistence/writers.py
  → orm-Tabelle price_recommendations      ← Übersetzung domain → ORM
```

`sources.py` und `writers.py` sind die Übersetzer. Zwischen ihnen kommt kein ORM-Objekt vor: `PricingApp` und die Domain sehen nur Domain-Objekte. Dieselbe Kette läuft mit `CsvDataSource` und `CsvRecommendationWriter` komplett ohne Datenbank — das ist der Zweck der Verträge `DataSource` und `RecommendationWriter`.

## C. API lesen

```
HTTP GET
  → Router                             routers/*.py
  → Repository (über Dependency)       repositories.py
  → SELECT
  → orm-Objekt                             ← ORM-Objekt erreicht den Router
  → …Read.model_validate(objekt)       Router
  → JSON
```

## D. API schreiben

```
JSON
  → …Create / …Write                   Pydantic validiert
  → .model_dump(exclude_none=True)     Router, ergibt ein dict
  → Repository.create(daten)           repositories.py
  → orm.Capacity(**daten)
  → session.add + _flush
  → orm-Objekt zurück
  → …Read.model_validate(objekt)       Router
  → JSON
```

Der Router übergibt dem Repository ein **`Mapping[str, object]`**, kein DTO. Damit kennt `persistence` kein Pydantic.

## E. Hintergrundlauf

```
POST /v1/pricing-runs
  → Router prüft active_exists()
  → repository.create_queued()  → INSERT + COMMIT      (nicht nur flush!)
  → tasks.add_task(execute_run, run.id)                 nur die Zahl
  → 202 + Location, Antwort geht raus
  → execute_run(run_id)                run_service.py, eigene Session
  → Weg B
  → Status in pricing_runs
```

Über die Grenze zum Hintergrundlauf geht **nur eine Zahl**. Ein ORM-Objekt wäre unbrauchbar, weil die Request-Session dann geschlossen ist. Der `COMMIT` in `create_queued` ist Pflicht, weil FastAPI den Background-Task **vor** dem Commit der Dependency ausführt.

---

# Die Verdrahtung: vom Repository bis in den Handler

Vier Stationen, für jede Entität gleich. Am Beispiel `capacity`:

**1. Vertrag** — `persistence/interfaces.py`

```python
class CapacityRepository(Protocol):
    def get(self, capacity_id: int) -> orm.Capacity | None: ...
```

**2. Implementierung** — `persistence/repositories.py`

```python
class SqlAlchemyCapacityRepository:
    def __init__(self, session: Session) -> None: ...
```

Nennt das Protocol **nicht**. Die Zugehörigkeit entsteht strukturell, nicht durch Vererbung.

**3. Fabrik** — `bootstrap.py`

```python
def make_capacity_repository(session: Session) -> CapacityRepository:
    return SqlAlchemyCapacityRepository(session)
```

Der Rückgabetyp ist das **Protocol**. Hier — und nur hier — prüft `mypy`, ob die Implementierung den Vertrag erfüllt. Stünde die konkrete Klasse als Rückgabetyp, prüfte niemand etwas.

**4. Dependency** — `presentation/api/deps.py`

```python
def get_capacity_repository(session: SessionDep) -> CapacityRepository:
    return make_capacity_repository(session)

CapacityRepositoryDep = Annotated[CapacityRepository, Depends(get_capacity_repository)]
```

**5. Benutzung** — `routers/capacity.py`

```python
def get_capacity(capacity_id: int, repository: CapacityRepositoryDep) -> CapacityRead:
```

Der Router nennt nur `CapacityRepositoryDep`. Den Typ des Rückgabewerts kennt `mypy` über die Protocol-Signatur — **ohne dass der Router `orm` importiert**. Ein Import ist nur nötig, um einen Namen zu *schreiben*, nicht um mit einem Wert zu arbeiten.

Der Nutzen der Kette zeigt sich im Test: `app.dependency_overrides[deps.get_capacity_repository] = lambda: FakeCapacityRepository()` ersetzt die Datenbank, ohne eine Zeile Produktionscode anzufassen.

## Die Session

```
bootstrap.engine()          @lru_cache, eine Engine (= ein Pool) für alles
  → bootstrap.session_scope()   Generator: yield → commit, bei Ausnahme rollback
      → deps.SessionDep         FastAPI ruft ihn pro Request
      → bootstrap.session_cm    dieselbe Logik als with-Block für CLI und Skripte
```

`session_cm = contextmanager(session_scope)` — als Aufruf, nicht als Dekorator, damit beide Gestalten erhalten bleiben.

Die Transaktionsgrenze gehört dem **Aufrufer**: In der API committet die Dependency nach der Antwort, im CLI der `with`-Block. Repositories rufen nur `flush()`. Einzige Ausnahme: `create_queued()` bei beiden Lauf-Ressourcen, siehe Weg E.

---

# Schichtverletzungen im Überblick

Vollständige Liste, aus dem AST erhoben. Zugrunde gelegte Ordnung: `presentation` (1) → `application` (2) → `domain` (3) → `persistence` (4). `shared` ist Querschnitt, `main`/`bootstrap` sind Composition Roots.

| # | Kante | Art | Stellen |
|---|---|---|---|
| 1 | `persistence` → `domain` | **aufwärts** | `interfaces.py`, `sources.py`, `writers.py`, `ingest.py` |
| 2 | `application` → `persistence` | überspringt `domain` | `pricing_app.py`, `auth_service.py`, `run_service.py`, `ingestion_service.py` |
| 3 | `presentation` → `persistence` | überspringt `application` | nur `deps.py` → `interfaces` |
| 4 | `application` → `bootstrap` | zeigt auf die Composition Root | `run_service.py`, `ingestion_service.py` |
| 5 | `presentation` → `bootstrap` | zeigt auf die Composition Root | nur `deps.py` |

**Nummer 1 ist der einzige Aufwärtspfeil im ganzen Projekt.**

Ursachen, Bewertung und **ausführliche Umbauanleitungen** stehen unten unter [[#Offene Umbauten]].

# Die zwei Brücken

Stellen, an denen die reine Schichtregel bewusst durchbrochen ist.

## Brücke 1: ORM-Objekte erreichen die Presentation

**Regel wäre:** `presentation` kennt nur `application`.
**Tatsächlich:** Der Router bekommt ein `orm.Capacity` und wandelt es selbst in ein DTO.

`presentation/api/deps.py` importiert `persistence/interfaces.py` — die einzige Datei unter `presentation/`, die das tut. Die Router selbst importieren nichts aus `persistence`.

**Warum:** Für CRUD gibt es keine Anwendungsschicht. Entscheidung vom 11.08.: **`application/` enthält, wo Regeln sind — CRUD hat keine.** Eine Zwischenschicht hätte rund 20 Durchreiche-Funktionen bedeutet, die Felder von A nach B kopieren.

**Was trotzdem gilt:** Ein ORM-Objekt verlässt den Router **nie**. Die Umwandlung in ein DTO passiert im Handler, spätestens beim `return`.

**Was `presentation` weiterhin nicht darf:** `psycopg`, `persistence.orm`, `persistence.repositories`, `persistence.api_keys`, `persistence.db`. Einzige erlaubte SQLAlchemy-Berührung ist der Name `Session` in `deps.py`, rein als Annotation für `SessionDep` — FastAPI löst sie zur Laufzeit auf, ein `TYPE_CHECKING`-Import genügt nicht. In `presentation/` wird `Session(...)` nirgends **aufgerufen**; das passiert in `bootstrap.py` und in `persistence/api_key_admin.py`, das als eigener CLI-Entrypoint seine eigene Composition Root ist.

Beides prüft `tests/test_architektur.py` per AST über alle Dateien unter `presentation/`.

## Brücke 2: `application` importiert `bootstrap`

**Regel wäre:** Nur Composition Roots kennen alles.
**Tatsächlich:** `run_service.py` und `ingestion_service.py` importieren `bootstrap.session_cm`, `bootstrap.engine` und `make_db_pricing_app`.

**Warum:** Ein Hintergrundlauf braucht eine **eigene** Session, weil die des Requests längst geschlossen ist. Er muss sie sich also selbst beschaffen.

**Der saubere Ausweg wäre**, die Session-Fabrik als Parameter hereinzureichen (`tasks.add_task(execute_run, run.id, session_cm)`). Nicht gemacht, um die Regel „nur eine Zahl überquert die Grenze" nicht aufzuweichen. Diese Brücke ist die schwächer begründete der beiden.

---

# Kurzreferenz: Was darf wen kennen

| Von → Nach | erlaubt? | Bemerkung |
|---|---|---|
| `domain` → alles andere | nein | prüft `test_architektur` |
| `persistence` → `domain` | ja | die Verträge liegen hier und sprechen Domain |
| `persistence` → `application` | nein | |
| `application` → `domain` | ja | |
| `application` → `persistence` | ja | über die Protocols in `interfaces.py` |
| `application` → `presentation` | nein | |
| `presentation` → `application` | ja | `auth_service`, `run_service`, `ingestion_service` |
| `presentation` → `persistence` | **nur `deps.py`, nur `interfaces`** | Brücke 1 |
| `presentation` → `bootstrap` | ja | `deps.py` holt dort die Fabriken |
| alle → `shared` | ja | Querschnitt |
| `bootstrap`, `main` → alles | ja | Composition Roots |

## Wo welche Bibliothek vorkommen darf

| Bibliothek | erlaubt in |
|---|---|
| `pydantic` (BaseModel) | `presentation/api/schemas/` |
| `pydantic-settings` | `shared/config.py` |
| `fastapi` | `presentation/` |
| `sqlalchemy` | `persistence/`, `bootstrap.py`, `deps.py` (nur `Session` als Annotation) |
| `psycopg` | `persistence/` — importiert in `ingest.py`, `writers.py` (beide für `COPY`) und `repositories.py` (für `psycopg.errors`); in `db.py` steht der Name nur im Verbindungsstring |
| keine externe | `domain/` |

---

# Zwiebel statt Torte: warum die Zeichnung nicht passt

## Was die Zwiebelarchitektur ist

Der Begriff stammt von Jeffrey Palermo (2008, *Onion Architecture*). Der Unterschied zur Schichttorte liegt nicht in anderen Regeln, sondern in einer anderen **Zeichnung**: konzentrische Ringe statt eines Stapels.

```
        ┌─────────────────────────────┐
        │   Infrastruktur             │   DB, HTTP, Dateien, Framework
        │  ┌───────────────────────┐  │
        │  │  Anwendungsfälle      │  │   orchestriert
        │  │  ┌─────────────────┐  │  │
        │  │  │   DOMÄNE        │  │  │   Regeln, Entitäten
        │  │  └─────────────────┘  │  │
        │  └───────────────────────┘  │
        └─────────────────────────────┘
             Abhängigkeiten zeigen NUR nach innen
```

Eine einzige Regel: **Ein innerer Ring darf nichts über einen äußeren wissen.**

Der inhaltliche Unterschied zur Torte steckt darin, wo die Datenbank sitzt. In der Schichttorte liegt sie **unten**, und unten heißt „grundlegend": Alles baut darauf auf. In der Zwiebel liegt sie **außen** und ist damit ein austauschbares Randdetail; im Kern steht die Fachlichkeit.

Dieselbe Idee heißt bei Alistair Cockburn *Ports & Adapters* (2005) und bei Robert C. Martin *Clean Architecture* (2012). Der technische Mechanismus, der „nur nach innen" möglich macht, ist die **Dependency Inversion**: Der Kern deklariert die Ports, der äußere Ring implementiert sie. So darf die Anwendung eine Datenbank benutzen, ohne sie zu kennen.

## Warum dieses Projekt faktisch eine Zwiebel ist

| Merkmal | Zwiebel verlangt | dieses Projekt |
|---|---|---|
| Kern ohne Abhängigkeiten | ja | ✅ `domain` importiert außer `shared` nichts |
| Alle kennen den Kern | ja | ✅ `persistence`, `application`, `bootstrap` importieren `domain` |
| Infrastruktur austauschbar | ja | ✅ CSV- und SQLAlchemy-Adapter, derselbe Lauf |
| Ports im Kern | ja | ❌ `interfaces.py` liegt in `persistence` |

Drei von vier Merkmalen sind erfüllt. Es fehlt genau eines: Die Verträge liegen im äußeren Ring statt im Kern. Genau daraus entstehen die Pfeile ① und ②.

Anders gesagt: Die Zeichnung im Kopf ist eine Torte mit `persistence` ganz unten, der Code ist eine Zwiebel mit `domain` im Kern. Die „Verletzungen" ① und ② sind der Abstand zwischen beidem, nicht ein Fehler im Code.

## Warum die Umwandlung nicht in `domain` gehören darf

Naheliegender Gedanke: Wenn `sources.py` das Problem ist, könnte die Umwandlung doch in die Domäne wandern. Der Blick in den Code sagt nein:

```python
# persistence/sources.py, CsvDataSource.load_capacity
CapacityRow(
    product_id=row["product_id"],
    calendar_date=datetime.strptime(row["calendar_date"], "%Y%m%d").date(),
    bookable_units=int(row["is_bookable"]),      # CSV-Spalte heißt anders
    optional_units=int(row["is_option"]),        # als das Domänenfeld
)
```

Die CSV-Spalte heißt `is_bookable`, das Domänenfeld `bookable_units`; Datumsangaben kommen als `%Y%m%d`-Strings. Läge diese Funktion in `domain`, wüsste die Domäne, wie Spalten in einer Kundendatei heißen und in welchem Format der Lieferant Daten schreibt.

**Der Prüfstein für jede Schichtfrage:** Wer muss sich ändern, wenn sich das Datenformat ändert? Die Antwort darf nie „die Domäne" lauten.

Zum Vergleich: Im SQLAlchemy-Zweig derselben Methode ist die Umwandlung eine reine 1:1-Übergabe, weil die ORM-Spalten schon so heißen wie die Domänenfelder. Die ganze Übersetzungsarbeit steckt im CSV-Zweig — und die gehört dorthin, wo CSV gelesen wird.

*(Randbeobachtung: Dass die Domänenklasse `CapacityRow` heißt und nicht etwa `DailyCapacity`, ist ein Symptom derselben Sache. Das Wort „Row" stammt aus der Datei, nicht aus der Fachsprache.)*

---

# Offene Umbauten

Nichts davon ist umgesetzt (Stand 2026-08-13). Reihenfolge nach Nutzen pro Aufwand.

## R1 · `DataSource` und `RecommendationWriter` nach `domain/ports.py`

**Behebt:** ② zur Hälfte (`application → persistence` schrumpft von vier Dateien auf zwei).
**Behebt nicht:** ① und ③.

**Warum es geht:** `interfaces.py` enthält zwei verschiedene Sorten Verträge.

| Protocol | formuliert in | kann umziehen? |
|---|---|---|
| `DataSource` | `domain.CapacityRow`, `domain.Booking`, `domain.Price` | ✅ |
| `RecommendationWriter` | `domain.PriceRecommendation` | ✅ |
| `ApiKeyRepository` | nur `str` | ✅ (nach `application`) |
| `ProductRepository` … `IngestionRunRepository` (6 Stück) | **`orm.X`** | ❌ |

**Die sechs CRUD-Repositories können nicht umziehen.** Ihre Signaturen nennen ORM-Typen (`-> Sequence[orm.Product]`); in `domain` würde daraus ein Import von SQLAlchemy in die einzige Schicht, die heute gar keine Fremdbibliothek kennt. Das wäre schlimmer als das Problem.

**Schritte:**

1. Neu: `src/pricing/domain/ports.py` mit `DataSource` und `RecommendationWriter`, Import der Entities aus `.entities`.
2. Neu: `src/pricing/application/ports.py` mit `ApiKeyRepository`. Begründung: Authentifizierung ist Anwendungspolitik, kein Domänenbegriff, und der Verbraucher (`auth_service`) besitzt den Vertrag.
3. Beide aus `persistence/interfaces.py` entfernen. Dort bleiben die sechs Repositories.
4. Importe umstellen:
   - `application/pricing_app.py`: `from ..persistence.interfaces import …` → `from ..domain.ports import …`
   - `application/auth_service.py`: → `from .ports import ApiKeyRepository`
   - `bootstrap.py`: zwei Importzeilen aufteilen
5. `sources.py`, `writers.py`, `api_keys.py` **bleiben unverändert** — sie nennen die Protocols nirgends, die Zugehörigkeit ist strukturell.

**Kosten:** zwei neue Dateien, vier geänderte Importblöcke, keine Logikänderung.

**Prüfen:** `uv run pytest` (123 grün), `mypy --strict`, `uv run pricing -v` liefert weiterhin 94267 Empfehlungen und `RFJA3D / 2027-04-22`. Der Architektur-Test muss weiter grün sein: `domain` darf danach immer noch nichts aus `persistence` importieren.

## R2 · `session_cm` hereinreichen statt holen

**Behebt:** ④ vollständig (`application → bootstrap`).

**Minimalvariante:**

```python
# application/run_service.py
def execute_run(
    run_id: int,
    session_cm: Callable[[], AbstractContextManager[Session]],
) -> None:
    with session_cm() as session:
        ...

# presentation/api/routers/pricing_runs.py
from ....bootstrap import session_cm
tasks.add_task(execute_run, run.id, session_cm)
```

Dasselbe für `ingestion_service.execute_ingestion_run`, das zusätzlich `engine()` braucht — entweder als zweiter Parameter oder, besser, zusammen mit R3 aufgelöst.

**Ehrlicher Haken:** Die Annotation `AbstractContextManager[Session]` bringt `from sqlalchemy.orm import Session` nach `application`. Faktisch hängt die Schicht schon an SQLAlchemy (sie importiert `orm`), aber der Import wird dadurch sichtbar. Wer das nicht will, braucht R3 dazu.

**Kosten:** zwei Signaturen, zwei Aufrufstellen, zwei Importzeilen.
**Prüfen:** Ein `POST /v1/pricing-runs` muss weiterhin `202` liefern und der Status nach ~25 s auf `succeeded` stehen. Danach darf `grep -rn bootstrap src/pricing/application/` nichts mehr finden.

## R3 · Die Run-Services fassen kein `orm` mehr an

**Behebt:** den Rest von ②, und macht R2 sauber.

**Problem heute:** `run_service.py` und `ingestion_service.py` holen sich `orm.PricingRun` und setzen Felder direkt — Persistenzarbeit in der Anwendungsschicht.

```python
run = session.get(orm.PricingRun, run_id)
run.status = "running"
run.started_at = datetime.now(UTC)
```

**Ziel:** Statuswechsel als Repository-Methoden.

```python
# persistence/interfaces.py — Vertrag erweitern
class RunRepository(Protocol):
    def mark_running(self, run_id: int, jetzt: datetime) -> bool: ...
    def mark_succeeded(self, run_id: int, jetzt: datetime, anzahl: int) -> bool: ...
    def mark_failed(self, run_id: int, jetzt: datetime, meldung: str) -> bool: ...
```

Jede Methode committet selbst (eigene Transaktion je Statuswechsel, wie heute die drei `with`-Blöcke) und liefert `False`, wenn es die Zeile nicht gibt.

Der Service bekommt dann eine **Repository-Fabrik** statt einer Session:

```python
def execute_run(
    run_id: int,
    repo_scope: Callable[[], AbstractContextManager[RunRepository]],
    rechne: Callable[[date], int],          # liefert die Anzahl Empfehlungen
) -> None:
```

`bootstrap` baut beide Callables. Danach importiert `application` weder `bootstrap` noch `orm` noch `sqlalchemy`.

**Kosten:** drei Repository-Methoden je Lauf-Ressource, zwei neue Fabriken in `bootstrap`, beide Services umgeschrieben. Der größte Posten dieser Liste, schätzungsweise zwei Stunden.

**Prüfen:** Beide Läufe end-to-end, inklusive Fehlerfall (Status muss `failed` mit `error_message` werden).

## R4 · Die Zeichnung ehrlich machen

**Behebt:** ① und ② als *Beschreibungsproblem*, ohne eine Zeile Code.

Entweder die Ordnung `presentation → application → persistence → domain` festschreiben, oder gleich als Zwiebel zeichnen und benennen.

**Was zu tun ist:**

1. Im [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]] die Architekturentscheidung vom 04.08. ergänzen (nicht ersetzen): Die damalige Wahl war klassische Schichtung mit Verträgen in der Persistenz; der gebaute Code ist zwiebelförmig, weil `domain` nichts importiert.
2. Die Canvas anpassen: `domain` nach unten oder als Ringe.
3. Diese Notiz und die Kurzreferenz nachziehen.

**Was dagegen spricht:** Das Kapitel heißt *Layered Architecture*, und die Entscheidung für klassische Schichtung ist bewusst dokumentiert. Ein Wechsel des Modells sollte eine sichtbare Entscheidung sein, keine stille Korrektur — sonst widerspricht die Canvas dem, was zwei Notizen weiter als Absicht steht.

## R5 · Vollständige Trennung (dokumentiert, nicht empfohlen)

**Behebt:** ①, ②, ③ vollständig. Alle Pfeile zeigen abwärts.

```
persistence  liefert eigene, neutrale Zeilentypen   (kennt domain nicht)
application  bildet sie auf Domänenobjekte ab       (kennt beides)
domain       bleibt unberührt                        (kennt niemanden)
```

**Kosten:** ein **dritter** Modellsatz. Pro Konzept gäbe es dann CSV-Zeile, neutralen Persistenztyp, ORM-Modell, Domänenobjekt und DTO. Der Übersetzungscode aus `sources.py` verschwindet dabei nicht, er zieht nur eine Schicht höher.

**Warum trotzdem notiert:** Es ist die einzige Variante, die ① wirklich beseitigt. Alle anderen verschieben nur, wer den Vertrag besitzt. Für ein Projekt dieser Größe ist der Preis zu hoch.

## Was welcher Umbau erledigt

| | ① aufwärts | ② Sprung app→pers | ③ Sprung pres→pers | ④ app→bootstrap | ⑤ deps→bootstrap |
|---|---|---|---|---|---|
| R1 Ports teilen | — | halb | — | — | — |
| R2 Fabrik hereinreichen | — | — | — | ✅ | — |
| R3 Repository-Methoden | — | ✅ | — | ✅ | — |
| R4 umzeichnen | ✅ | ✅ | — | — | — |
| R5 volle Trennung | ✅ | ✅ | ✅ | — | — |
| Ausnahme benennen | — | — | — | — | ✅ |

**Empfohlene Reihenfolge, falls es je gemacht wird:** R4 (Entscheidung), dann R1 (billig, klärt die Trennlinie), dann R2+R3 zusammen (R2 allein tauscht nur einen Import gegen einen anderen). R5 nur, wenn das Projekt deutlich wächst.
