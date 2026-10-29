---
tags:
  - documentation
type: code-review
projekt: REST_API
pfad: ~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/REST_API
datum: 2026-08-13
commit: 7acbcf0
---

Vollständiges Code Review des Übungsprojekts zu [[1.5 REST APIs & Layered Architecture - Leitfaden|Kapitel 1.5]]. Begleitend zu [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]] und [[REST API Plan & Syntax-Tutorial|REST API Plan & Syntax-Tutorial]]. Theoriebezug: [[1.5 REST APIs & Layered Architecture - Vertiefung|Vertiefung]], [[Datenmodelle und Schichtenübergänge|Datenmodelle und Schichtenübergänge]].

> [!info] Rahmen
> Übungsprojekt ohne Produktivbetrieb. Sicherheitsbefunde sind deshalb bewusst niedrig gewichtet und nur dort genannt, wo sie eine **Inkonsistenz in der eigenen Haltung** aufzeigen. Bewertungsmaßstab ist Architektur, Design und Konsistenz — nicht Betriebsreife.
>
> Alle Aussagen sind am Code und am installierten Framework-Quellcode verifiziert, nicht aus dem Gedächtnis behauptet. Ein zweites Modell (Sonnet) hat die Befunde adversarial gegengelesen; sechs meiner ursprünglichen Thesen sind dabei gefallen und stehen unten unter [[#Geprüft und nicht beanstandet]].

# Inhaltsverzeichnis

- [[#Gesamtbild]]
- [[#Was besonders gelungen ist]]
- [[#Architektur — die eigentlichen Befunde]]
- [[#Bugs und Robustheit]]
- [[#Duplikation und Eleganz]]
- [[#API-Design]]
- [[#Tests]]
- [[#Betrieb und Tooling]]
- [[#Geprüft und nicht beanstandet]]
- [[#Priorisierung — die drei Dinge]]

# Gesamtbild

| Kennzahl | Wert |
|---|---|
| Umfang | ~4.300 Zeilen Python (src, tests, scripts) |
| `ruff check` (E,W,F,I,N,UP,B,SIM,G,LOG,TRY) | grün |
| `ruff format --check` | 63 Dateien sauber |
| `mypy --strict` | grün, 62 Dateien |
| Tests | 123 grün in 0,86 s, ohne laufende Datenbank |
| Line-Coverage | 66 % — mit einem großen Aber, siehe [[#Tests]] |

Das ist handwerklich deutlich über Trainee-Niveau. `mypy --strict` grün über ein Projekt mit SQLAlchemy-ORM, Pydantic und FastAPI ist keine Selbstverständlichkeit — an genau dieser Kombination scheitern regelmäßig Produktivprojekte. Die Kommentare erklären durchgehend das **Warum** statt das **Was**, was in der Praxis selten ist.

Die zentrale Beobachtung des Reviews ist aber eine andere, und sie ist wichtiger als jede Einzelmeldung:

> [!abstract] Die Architektur ist nicht falsch angelegt — sie ist unter Feature-Druck erodiert.
> Der Batch-Pfad (`PricingApp`, `DataSource`, `RecommendationWriter`) ist sauber: domänentypisierte Ports, Dependency Injection von außen, keine Rückgriffe. Er stammt aus den frühen Commits.
>
> Der REST-Pfad hat dieselbe Disziplin nicht durchgehalten. Die Repository-Protocols geben ORM-Entitäten zurück, die Application-Schicht greift auf den Composition Root zu, die Background-Services umgehen ihre eigenen Abstraktionen. Das sind die letzten Feature-Commits (`893a5f3`, `67e4436`).
>
> **Das ist der eigentliche Lerninhalt.** Nicht „Schichten sind schwer", sondern: Schichten halten nicht von allein. Sie brauchen einen Test, der sie erzwingt — und `test_architektur.py` erzwingt genau die Hälfte, die nicht erodiert ist.

# Was besonders gelungen ist

Nicht als Höflichkeitsfloskel vorweg, sondern weil mehrere dieser Punkte über dem liegen, was ich in Produktivcode regelmäßig sehe.

## Die Testsuite denkt in Fallen, nicht in Zeilen

`tests/test_architektur.py` ist eine **Fitness Function**: ein AST-Walk (Z. 27–44), der die Schichtregel aus einer Absichtserklärung in eine prüfbare Zusage verwandelt, parametrisiert pro Datei, mit einer *dokumentierten und begründeten* Ausnahme (`AUSNAHME` Z. 15, `deps.py` darf `sqlalchemy.orm` nennen, weil FastAPI die Annotation zur Laufzeit auflösen muss). Das ist ein Werkzeug, das die meisten Teams nie bauen.

`tests/test_schema_vertrag.py` fängt eine Bug-Klasse, die sonst *nichts* fängt: Ein Read-Feld mit Default `None`, das im ORM gar nicht existiert, läuft still durch ruff, mypy und alle Gates — und bleibt im Response einfach leer. Der Docstring nennt die realen Vorfälle (`refrence_date`, `error_msg`). Genau so schreibt man einen Regressionstest.

Weitere Tests, die eine echte Falle kodieren statt Coverage zu erzeugen:

- `test_max_wird_nicht_als_id_gelesen` — Routenreihenfolge `/max` vor `/{id}`, sonst 422.
- `test_ingest_pfade.py` — parametrisierter Path-Traversal inkl. **Symlink nach draußen**, und der Docstring erklärt, warum `resolve()` *vor* `is_relative_to()` stehen muss.
- `test_fehlend_und_falsch_antworten_gleich` — denkt daran, dass unterschiedliche Antworten auf „Schlüssel fehlt" vs. „Schlüssel falsch" verraten, ob ein Schlüssel existiert.
- `test_unerwarteter_fehler_wird_500_ohne_details` — prüft aktiv, dass kein Geheimnis in die 500 leckt.

## Fortgeschrittenes SQLAlchemy-Verständnis

`persistence/writers.py` ist der anspruchsvollste Code im Projekt und richtig gebaut: Temp-Table mit `postgresql_on_commit="DROP"`, `COPY` über die rohe psycopg-Verbindung, dann `INSERT ... FROM SELECT` mit Join **in der Datenbank** statt 1,33 Mio. Zeilen nach Python zu laden (Z. 100–102 dokumentiert genau das). Der Kommentar Z. 127–129 zu `.rowcount` — warum man dem DBAPI-Rowcount bei `INSERT...SELECT` nicht trauen darf und warum die Zeilenzahl nach dem `TRUNCATE` trotzdem exakt stimmt — zeigt, dass hier verstanden und nicht kopiert wurde.

`repositories.py` koordiniert `joinedload(orm.PriceRecommendation.capacity)` in **allen drei** Methoden (Z. 158, 168, 176) mit dem `AliasPath("capacity", "product_id")` in `schemas/recommendation.py:11-12`. Ohne den `joinedload` wäre die Folge kein Crash, sondern ein stilles N+1 — pro Empfehlung eine zusätzliche SELECT. Das ist die unauffälligere und deshalb schwerer zu findende Falle.

`_flush()` vs. `_flush_after_delete()` (Z. 13–32): dieselbe `ForeignKeyViolation` bedeutet beim Anlegen „du verweist auf etwas, das es nicht gibt" (422) und beim Löschen „auf dich verweist noch etwas" (409). Dass diese Unterscheidung überhaupt gemacht wurde, ist die Sorte Detail, die man normalerweise erst nach einem Produktionsvorfall einbaut.

## Bewusste Entscheidungen mit Begründung

- `bootstrap.engine()` als `@lru_cache`-Singleton, mit dem Kommentar „hatte vorher fuer jede Request eigene" — ein selbst gefundenes und behobenes Antipattern.
- `schema_admin` als eigenes Kommando statt beim App-Start: „eine Anwendung, die beim Hochfahren Tabellen anlegt, ist ein Betriebsrisiko."
- `api_key_admin` als CLI statt Endpunkt, mit Begründung.
- POST nutzt `model_dump(exclude_none=True)`, PUT nutzt `model_dump()` ohne Exclude — korrekte, im Docstring *benannte* POST/PUT-Semantik (`bookings.py:52` vs. `:67-69`).
- `orm.py`: `NAMING_CONVENTION`, `type_annotation_map`, `Computed(persisted=True)` für `net_revenue` — und der Ingest schließt genau diese Spalte vom `COPY` aus.
- `errors.py:26` nutzt `logger.error(..., exc_info=exc)` statt `logger.exception(...)`. Das ist hier **richtig**, weil man in einem FastAPI-Exception-Handler mit explizitem `exc`-Parameter sitzt, nicht in einem `except`-Block.
- Der Stack ist aktuell und modern: PEP-695-`type`-Alias in `responses.py:9`, SQLAlchemy-2.0-`Mapped`, Pydantic v2, `httpx2`.

# Architektur — die eigentlichen Befunde

## A1 · Die Repository-Protocols lecken das ORM in die Presentation

**Der zentrale Bruch.** `persistence/interfaces.py:16` importiert `orm`, und alle CRUD-Protocols sind in ORM-Typen formuliert:

```python
class BookingRepository(Protocol):
    def list_all(self, ...) -> Sequence[orm.Booking]: ...
```

Damit hantiert die Presentation-Schicht faktisch mit ORM-Objekten — `BookingRead.model_validate(booking)` in `routers/bookings.py:24` bekommt eine `orm.Booking`. Zugespitzt in `schemas/recommendation.py:11-12`: `AliasPath("capacity", "product_id")` greift durch eine ORM-**Relationship** durch. Die Presentation kennt damit nicht nur das ORM, sondern dessen Beziehungsgraph.

`test_architektur.py` fängt das nicht, weil es **Importnamen** prüft, nicht Laufzeittypen. Der Test ist nicht schlecht gebaut — Architekturtests auf Importbasis sind prinzipiell laufzeitblind. Aber die Zusage, die er suggeriert („Presentation kennt keine Implementierung"), hält er nur dem Buchstaben nach.

Der Kontrast im selben Modul ist die Pointe: `DataSource` und `RecommendationWriter` (Z. 19–32) sind sauber domänentypisiert (`list[CapacityRow]`, `list[PriceRecommendation]`). **Derselbe Entwickler hat im selben File den Port einmal richtig und einmal falsch geschnitten.** Der Unterschied ist nur, dass der eine Teil älter ist.

> [!tip] Fix
> Repository-Protocols in Domänen- oder DTO-Typen formulieren und das Mapping ORM → DTO in die Persistenz ziehen. Dann verschwindet auch der `AliasPath`-Durchgriff, weil das Repository ein flaches Ergebnisobjekt liefert statt eines Objektgraphen.

## A2 · Application greift auf den Composition Root zu

`application/run_service.py:4` und `application/ingestion_service.py:7` importieren `..bootstrap`:

```python
from ..bootstrap import make_db_pricing_app, session_cm
from ..bootstrap import engine, session_cm
```

Die Application-Schicht **holt** sich ihre Abhängigkeiten aus dem Composition Root, statt sie gereicht zu bekommen. Das ist ein Service-Locator und damit die Umkehrung dessen, was `PricingApp` im selben Ordner vorbildlich macht: dort kommt alles über den Konstruktor (`pricing_app.py:27-37`).

Ungefangen, weil `test_architektur.py` nur Regeln für `presentation/` und `domain/` hat. **Für `application/` existiert keine Regel** — genau in der Schicht, die tatsächlich erodiert ist.

## A3 · Die Background-Services umgehen ihre eigenen Abstraktionen

Beide Services importieren `..persistence.orm` direkt und mutieren ORM-Objekte:

```python
run.status = "running"          # run_service.py:16
run.started_at = datetime.now(UTC)
```

Obwohl `RunRepository` und `IngestionRunRepository` als Protocol existieren. Aber — und das ist die Erklärung, nicht die Entschuldigung — **die Protocols können es gar nicht**: `interfaces.py:104-125` kennt nur `create_queued`, `get`, `list_all`, `active_exists`. Keine Methode zum Statuswechsel.

Die Schnittstelle wurde aus Router-Sicht entworfen (lesen, anlegen) und hat die Worker-Sicht (Fortschritt melden) nie antizipiert. Der Bypass ist also nicht Bequemlichkeit, sondern die einzige verbliebene Option. Das ist ein sehr lehrreicher Fehlermodus: *Eine Abstraktion, die nur einen ihrer Konsumenten kennt, wird vom anderen umgangen.*

## A4 · `create_queued` committet — und das ist kein Schlamperei-Befund

`repositories.py:206` ruft `self._session.commit()` in einem Repository. Auf den ersten Blick ein klarer Bruch der Unit-of-Work-Klammer, die `bootstrap.session_scope` etabliert.

Der naheliegende „Fix" — Commit entfernen — würde den Code aber **kaputt** machen, und der Grund dafür ist im installierten Framework-Quellcode nachprüfbar:

- `fastapi/routing.py:2193-2205`: `await response(scope, receive, send)` steht **innerhalb** von `async with self._solve_dependencies(...)`.
- `starlette/responses.py:169-170`: `Response.__call__` sendet erst den Body und ruft **danach** `await self.background()` auf — alles bevor die `async with`-Klammer verlassen wird.

> [!warning] Konsequenz
> **Background-Tasks laufen garantiert, bevor der Exit-Code von yield-Dependencies ausgeführt wird** — also bevor `session_scope` committet (`bootstrap.py:66-72`).
>
> Ohne das explizite `commit()` in `create_queued` wäre die neue Run-Zeile beim Start des Background-Tasks noch uncommitted. `execute_run` öffnet über `session_cm()` eine **eigene** Session; unter READ COMMITTED sieht die die Zeile nicht. `session.get(...)` liefert `None`, der Service loggt „kein bekannter run" und kehrt still zurück (`run_service.py:13-15`) — der Client hat aber längst 202 + `Location` bekommen, und der Run bleibt für immer auf `queued`.
>
> Das wäre **kein Rennen, sondern deterministisch bei jedem einzelnen Aufruf.**

Der Befund lautet also nicht „Repository darf nicht committen", sondern: **die Commit-Entscheidung steht an der falschen Stelle.** Sie gehört sichtbar an die Aufrufstelle (Router oder Application-Service), wo man sie im Zusammenhang mit `tasks.add_task(...)` lesen kann — nicht versteckt in einer Repository-Methode, wo der nächste Leser sie für einen Fehler hält und „aufräumt".

Nebenbefund aus derselben Zeile: `Session(...)` läuft mit dem Default `expire_on_commit=True`. Nach dem Commit sind alle Attribute von `new_run` expired — der Zugriff auf `run.id` in `pricing_runs.py:35-36` und `RunRead.model_validate(run)` in Z. 37 lösen ein implizites Re-SELECT aus. Ein unsichtbarer Extra-Roundtrip pro Run-Start.

Und: `session_scope` selbst (Z. 66–72) hat laut Coverage **0 % ausgeführte Statements**. Kein Test hätte diese Falle je gefangen — in keine Richtung.

## A5 · `shared/exceptions.py` ist ein Sammelbecken

In einem Modul liegen Domänen-Invarianten (`DomainError`), Persistenz-Belange (`UnknownReferenceError`, `ConflictError` — beide entstehen ausschließlich in `_flush`) und Ingest-Belange (`IngestFolderOutsideBaseError`). `domain/entities.py:14` importiert daraus — die innerste Schicht hängt damit an einem Modul, das Fremdschlüssel und Ordnerpfade kennt.

Der spannendere Teil ist die Vererbung:

```python
class UnknownReferenceError(DomainError):   # FK-Verletzung als Domain-Fehler
class ConflictError(PricingError):          # Unique-Verletzung NICHT als Domain-Fehler
```

Beide entstehen an derselben Stelle aus derselben `IntegrityError`. Der einzige Grund für die unterschiedliche Einordnung ist, dass `errors.py` `DomainError` auf 422 und `ConflictError` auf 409 mappt. **Eine HTTP-Statuscode-Entscheidung bestimmt die Vererbungshierarchie der Exceptions.** Das ist der Schwanz, der mit dem Hund wedelt — die Presentation-Schicht formt die Typen der innersten Schicht.

# Bugs und Robustheit

## B1 · Race Condition beim Start von Läufen

`pricing_runs.py:32-34` und `ingestion_runs.py:37-40`:

```python
if repository.active_exists():
    raise ConflictError(...)
run = repository.create_queued(...)
```

Klassisches Check-then-act ohne DB-Constraint. Zwei parallele POSTs starten zwei Läufe.

Die schwerere, im Code gar nicht adressierte Variante: `active_exists()` prüft jeweils nur die **eigene** Tabelle. Nichts hindert einen Client daran, gleichzeitig einen Pricing-Run *und* einen Ingest zu starten — beide Checks sehen „frei". `execute_run` hält seine Session über die volle Laufzeit offen und liest `capacity`/`bookings`, während `run_ingest` (`ingest.py:230`) genau darauf `TRUNCATE ... CASCADE` mit ACCESS-EXCLUSIVE-Lock macht.

Die Reihenfolge „erst Ingest, dann Preislauf" ist in `app.py:52-54` und im Docstring von `ingestion_runs.py:31-33` als **Konvention** dokumentiert — erzwungen wird sie nirgends.

> [!tip] Fix
> Partieller Unique-Index statt Vorab-Check:
> ```sql
> CREATE UNIQUE INDEX uq_pricing_runs_aktiv ON pricing_runs ((true))
>   WHERE status IN ('queued','running');
> ```
> Dann wird der Konflikt zur `UniqueViolation`, die `_flush` bereits in `ConflictError` → 409 übersetzt. Der Check-then-act verschwindet ersatzlos.

## B2 · Ein Absturz sperrt den Endpunkt dauerhaft

Stirbt der Prozess während eines Laufs, bleibt die Zeile auf `running`. `active_exists()` ist damit für immer `True`, der Endpunkt antwortet dauerhaft 409. Kein Reaper, kein Timeout, kein Heartbeat — im ganzen Repo verifiziert.

## B3 · `BackgroundTasks` kollidiert mit dem Session-Dependency-Muster

Verifiziert: **keine einzige** der 24 Route-Funktionen ist `async def`. Alle laufen über `run_in_threadpool`, und `starlette/background.py:23` nutzt für synchrone Background-Tasks exakt denselben Mechanismus. Beide ziehen aus demselben globalen AnyIO-`CapacityLimiter` (Default 40).

Zusammen mit A4 ergibt das eine Kette:

1. Ein 20–40-s-Pricing-Run oder 2-Minuten-Ingest belegt einen gemeinsamen Worker-Thread.
2. Weil die Dependency-Cleanup der auslösenden Request erst **nach** dem Task läuft, bleibt zusätzlich deren SQLAlchemy-Connection für die gesamte Job-Laufzeit ausgecheckt und untätig — parallel zu der, die der Task aktiv nutzt. **Zwei Connections pro Job.**
3. `db.py:18` ruft `create_engine(url, echo=echo)` ohne `pool_size`/`max_overflow` — es gelten die Defaults 5 + 10 = 15.

Zwei, drei gleichzeitige Läufe plus normaler Verkehr erschöpfen Thread- und Connection-Pool — und zwar für **alle** Endpunkte, nicht nur die Run-Endpunkte. Für ein Übungsprojekt völlig in Ordnung; der Lerninhalt ist, *warum* `BackgroundTasks` hier strukturell das falsche Werkzeug ist. Nicht weil „man eine Queue bräuchte", sondern weil sein dokumentiertes Lifecycle-Verhalten mit dem yield-Dependency-Session-Muster kollidiert.

## B4 · Die Fehlerdisziplin hat eine Hintertür

`errors.py:24-27` gibt bei 500 bewusst keine Details preis, und `test_unerwarteter_fehler_wird_500_ohne_details` prüft das aktiv.

Aber `run_service.py:41` und `ingestion_service.py:68` schreiben `str(exc)` in `error_message`, und `RunRead`/`IngestionRunRead` liefern dieses Feld ungefiltert an jeden Client aus (`schemas/pricing_run.py:16`, `schemas/ingestion_run.py:23`). Ein `OperationalError` von psycopg enthält typischerweise Host, Port und Benutzernamen.

**Dieselbe Leckage, die vorne sorgfältig geschlossen wurde, steht hinten offen.** Das ist der interessantere Befund als das Leck selbst: Eine Sicherheitszusage, die nur an einer Stelle durchgesetzt wird, ist keine.

## B5 · Konfigurationsfehler entkommen in allen vier Entrypoints

`main.py:15-16`, `ingest.py:267-268`, `schema_admin.py:33-34`, `api_key_admin.py:97-98` rufen alle `get_settings()` **vor** dem `try`. Ein `ConfigError` dort läuft ungefangen und ohne konfiguriertes Logging durch — genau der Fall, für den `ConfigError` gebaut wurde. Vier Mal derselbe Copy-Paste-Fehler.

## B6 · Zwei Datumsparser für dieselbe Datei

`sources.py:74` nutzt `strptime(row["date"], "%Y%m%d")`, `ingest.py:50` nutzt `strptime(value.split(".")[0], "%Y%m%d")`.

Verifiziert an den echten Daten: Die `date`-Spalte enthält keine Nachkommastellen (`20240223`), der Fall tritt aktuell also nicht ein. **Latent, nicht akut** — aber beide Parser sind laut Coverage komplett ungetestet (`sources.py` Missing 67–71, `ingest.py` Missing 50/54/58). Die Divergenz ist für CI vollständig unsichtbar, auf beiden Seiten.

# Duplikation und Eleganz

## D1 · Die beiden Run-Services sind Copy-Paste

`run_service.execute_run` und `ingestion_service.execute_ingestion_run` implementieren dieselbe Zustandsmaschine (`queued → running → succeeded | failed`) mit derselben Drei-Transaktionen-Struktur. Sie divergieren nur kosmetisch: einmal `logger.exception`, einmal `logger.error` + `# noqa: TRY400` — beides für dieselbe Aussage.

> [!tip] Fix
> Ein gemeinsamer Context-Manager, dem Modellklasse, `run_id` und die eigentliche Arbeit übergeben werden. Das ist zugleich die natürliche Stelle, um die in A3 fehlenden `mark_running`/`mark_succeeded`/`mark_failed` unterzubringen.

## D2 · Die Repositories sind vierfach dasselbe Skelett

`SqlAlchemyBookingRepository` (Z. 47–97) und `SqlAlchemyCapacityRepository` (Z. 100–146) sind bis auf Modell und Filterfelder identisch; `SqlAlchemyRunRepository` (Z. 198–227) und `SqlAlchemyIngestionRunRepository` (Z. 230–259) ebenso. Eine generische Basis mit `Generic[M]` spart ~60 Zeilen und ist genau der Anlass, bei dem Python-Generics zum ersten Mal Sinn ergeben statt nur Syntax zu sein.

## D3 · `PriceableUnit` ist die einzige Entität ohne Dataclass

`entities.py:17-81` — 65 Zeilen aus `__slots__` und fünf handgeschriebenen Properties, während `CapacityRow`, `Price` und `Booking` im selben File `@dataclass` nutzen.

Der Beweis, dass es nicht nötig war, steht 11 Zeilen darunter: `Booking` (Z. 92–114) ist ein `@dataclass` **mit** `__post_init__`-Validierung **und** einer `@property` **und** einer Methode — exakt die Kombination, die `PriceableUnit` angeblich zum Ausbruch zwang. `@dataclass(frozen=True, slots=True)` mit `__post_init__` hätte gereicht; die drei echten Ableitungen bleiben ohnehin Properties.

## D4 · Magic Numbers in der Kernformel

`recommendations.py:24-29`:

```python
days_until_travel_adjusted = unit.days_until_travel(reference_date) / 300
pct = 15.0 * (occupancy - 0.5) * (days_until_travel_adjusted + 0.5)
```

Vier unbenannte Zahlen in der einzigen fachlichen Formel des Projekts. Was ist 300 — ein Buchungshorizont? Was bedeutet 15.0 — maximale Anpassung in Prozent? Warum 0.5 als Auslastungs-Neutralpunkt? Der Code, der die Domäne trägt, ist der einzige, der sie nicht erklärt.

## D5 · Totes Feature-Fragment rund um die Authentifizierung

Drei Befunde, die zusammengehören:

- `application/auth_service.py` ist ein Zweizeiler-Durchreicher ohne eigene Logik.
- `orm.py:142` definiert `last_used_at` — verifiziert: **nie irgendwo geschrieben.** Eine tote Spalte.
- `deps.py:65` definiert `ApiKeyNameDep` — verifiziert: **nirgends verwendet.** Toter Alias.

Alle drei lösen sich gegenseitig auf: `authenticate()` ist genau die Stelle, an der `last_used_at` gesetzt gehört, und `ping` sollte `ApiKeyNameDep` nutzen (siehe [[#API3 · `ping` verschenkt seinen Zweck]]). Aus einer sinnlosen Indirektion würde eine sinnvolle.

## D6 · Boilerplate in der Verdrahtung

- `app.py` wiederholt `dependencies=[Depends(require_api_key)]` **siebenmal** (Z. 68, 72, 75, 78, 81, 84, 87). Ein geschützter Eltern-Router, in den die Sub-Router eingehängt werden, ersetzt alle sieben.
- `"/v1"` ist an rund zwölf Stellen hartkodiert — auch in `Location`-Headern (`f"/v1/bookings/{booking.id}"`). Ändert sich das Präfix, brechen die Header **still**. `request.url_for("get_booking", booking_id=...)` ist idiomatisch und bruchsicher.
- `BookingCreate` und `BookingUpdate` (`schemas/booking.py:25-54`) sind 13-Feld-Zwillinge, Unterschied nur `base_currency`. Im selben Ordner macht `CapacityRead(CapacityWrite)` es per Vererbung richtig.

## D7 · Sprach- und Umlautmischung

Deutsche Bezeichner (`daten`, `feld`, `wert`, `zeilen`, `treffer`, `basis`, `ziel`, `meldung`, `was`, `kennung`) stehen neben englischen (`product_id`, `list_all`, `find_active_name`) — teils in derselben Signatur:

```python
def update(self, booking_id: int, daten: Mapping[str, object]) -> orm.Booking | None:
```

Dazu die Umlaut-Inkonsistenz, verifiziert per Zählung: „Schluessel" 28×, „Schlüssel" 0×. „loeschen" immer ASCII. Aber „Kapazität" und „Empfehlung" durchgehend mit echtem Umlaut. Die Regel ist nicht „Domäne mit, Rest ohne" — es ist schlicht unsystematisch.

> [!tip] Empfehlung
> Bezeichner durchgehend **englisch** (das Domänenvokabular ist es ohnehin: product, booking, capacity), Kommentare und Docstrings durchgehend **deutsch mit echten Umlauten**. Das ist eine Zehn-Minuten-Entscheidung, die jede künftige Datei betrifft.

# API-Design

## API1 · Der GET→PUT-Round-Trip ist gebrochen

`BookingRead` (`schemas/booking.py:8-22`) liefert `cust_id`, `source_id` und `feature_2` **nicht** aus. `BookingCreate` und `BookingUpdate` verlangen `cust_id` als Pflichtfeld ohne Default.

Ein Client kann eine Buchung also lesen, aber das Gelesene nicht zurückschreiben — er müsste `cust_id` aus einer Quelle kennen, die die API ihm nie gezeigt hat. Für eine CRUD-Ressource ist das der zentrale Vertragsbruch.

## API2 · `get_max_recommendation` lügt im Schema

`routers/recommendations.py:32-39` ist als `-> RecommendationRead | None` annotiert, gibt aber nie `None` zurück — bei fehlendem Treffer wird `NotFoundError` geworfen. Das `| None` erzeugt in `/openapi.json` ein `anyOf` mit `null`, und generierte Clients bauen daraufhin eine Null-Behandlung, die nie eintritt.

## API3 · `ping` verschenkt seinen Zweck

`routers/ping.py:10` gibt `{"ping": "response"}` zurück. Der Endpunkt heißt laut `app.py:18` „Prueft den API-Schluessel" — dann sollte er auch sagen, **welcher** Schlüssel gilt. `ApiKeyNameDep` existiert dafür bereits und ist genau deshalb toter Code (siehe [[#D5 · Totes Feature-Fragment rund um die Authentifizierung]]).

## API4 · Doppelte Pfadauflösung

`ingestion_runs.py:35` ruft `resolve_ingest_dir(daten.folder)` nur für den Seiteneffekt der Validierung und verwirft das Ergebnis. `execute_ingestion_run` löst denselben Pfad in `ingestion_service.py:45` erneut auf. Zwischen beiden Aufrufen kann sich das Dateisystem ändern — und die Arbeit wird doppelt gemacht.

## API5 · Constraint-Namen gehen an den Client

`repositories.py:22` reicht `exc.orig.diag.constraint_name` in die `ConflictError`, `errors.py:22` gibt `str(exc)` roh aus. Der Client sieht `uq_capacity_product_id` — interne Schemanamen, deterministisch erzeugt aus `NAMING_CONVENTION` in `orm.py:27-33`.

Bemerkenswert im Kontrast: `test_meldung_verraet_den_serverpfad_nicht` stellt sicher, dass beim Ingest-Fehler **nie** der aufgelöste Serverpfad zurückgeht, sondern nur der vom Client selbst geschickte String. Zwei entgegengesetzte Haltungen im selben Error-Handling-System.

## API6 · Paginierung ohne Gesamtzahl

`limit`/`offset` liefern eine nackte Liste. Kein `X-Total-Count`, keine `Link`-Header. Ein Client kann nicht wissen, ob er auf der letzten Seite ist, ohne eine leere Seite anzufordern.

# Tests

Die Ideen sind exzellent (siehe [[#Was besonders gelungen ist]]). Die **Verteilung** ist das Problem.

## T1 · Das Einfache ist dicht getestet, das Riskante gar nicht

Von 83 Testfunktionen entfallen 45 auf reine, IO-freie Domänenlogik (`test_domain.py` 23, `test_recommendations.py` 13, `test_builder.py` 9). Das ist der Code, der am wenigsten kaputtgehen kann.

Grep über `tests/` nach den Endpunkten:

| Endpunkt | Treffer in Tests |
|---|---|
| `/v1/recommendations` | 11 |
| `/v1/capacity` | 5 |
| `/v1/bookings` | **0** |
| `/v1/products` | **0** |
| `/v1/pricing-runs` | **0** |
| `/v1/ingestion-runs` | **0** |

Vier von sechs Ressourcen sind nicht „dünn getestet", sondern **nie angefasst**. Es existiert kein Fake-`BookingRepository`.

## T2 · Die 66 % Line-Coverage sind eine potemkinsche Zahl

> [!danger] Wichtig zu verstehen
> `routers/bookings.py` zeigt **56 % Coverage** — und trotzdem läuft kein einziger Handler-Body je. Die mehrzeiligen `@router.post(...)`-Decorator und die Funktionssignaturen werden beim **Modul-Import** einmal ausgeführt und zählen als „covered". Die eigentlichen Bodies (Z. 31–34, 52–54, 69–72, 82–83) stehen in der Missing-Liste.
>
> Dasselbe Muster: `repositories.py` „31 %" sind fast nur Klassen- und `def`-Zeilen. `writers.py` „55 %" stammen komplett aus den modulweiten SQLAlchemy-Core-Table-Definitionen — **beide `.write()`-Methoden laufen nie.**

Was tatsächlich bei ~0 % echter Verhaltensabdeckung liegt:

- `writers.py` — der komplexeste Code im Projekt
- `repositories.py` — alle sechs Klassen
- `ingest.run_ingest` (Z. 212–261)
- `CsvDataSource.load_*` — alle drei Methoden
- `bootstrap.session_scope` — die Transaktionsklammer selbst
- `run_service.execute_run` (18 %), `ingestion_service.execute_ingestion_run` (0 %)
- `api_key_admin.py` — **0 %, 75/75 Statements.** Die CLI, die die Sicherheitsgrenze der ganzen API verwaltet.
- `schema_admin.py` — 0 %, 28/28

## T3 · Die Lücke ist eine Design-Folge, kein Fleiß-Problem

Das ist der wichtigste Punkt dieses Abschnitts. Die Persistenzschicht ist nativ Postgres:

- `repositories.py:4` — `import psycopg.errors as pg`
- `writers.py:113` und `ingest.py:253` — `conn.connection.driver_connection` für `COPY`
- `orm.py:91-94` — `Computed(...)`

Ein SQLite-Test war also nie eine Option. Und weil kein leichtgewichtiger Test möglich war, entstand **gar keiner**. Bezeichnend: `scripts/smoke_orm.py` zeigt, dass die Lücke gespürt und mit einem manuellen Skript kompensiert wurde — inklusive sorgfältiger Vorbedingungsprüfungen. Bei bereits vorhandenem `docker-compose.yml` ist der Weg zu echten Fixtures kurz.

## T4 · Der Schema-Vertragstest deckt nur die halbe Richtung ab

`test_schema_vertrag.py` prüft die **Read**-Modelle gegen das ORM. `BookingCreate`, `BookingUpdate` und `CapacityWrite` sind nicht dabei — dabei landen deren Felder über `orm.Booking(**daten)` (`repositories.py:74`) direkt im Konstruktor.

Fairerweise: SQLAlchemys generiertes `__init__` wirft bei unbekannten Kwargs einen lauten `TypeError`, das wäre also kein stiller Fehler. Aber der Test wäre drei Zeilen Erweiterung und würde die Symmetrie herstellen.

## T5 · Globaler Zustand in den Tests

- `test_exception.py:55` und `:61` rufen `get_settings.cache_clear()` **ohne** `try`/`finally`, mit dem Assert dazwischen. Schlägt der Assert fehl, bleibt der Cache für alle folgenden Tests geleert. Aktuell harmlos, weil die Reihenfolge deterministisch ist — genau ein `pytest-xdist` oder `pytest-randomly` von echter Flakiness entfernt.
- `test_ingest_pfade.py:18` mutiert per `monkeypatch.setattr(get_settings(), ...)` die **gecachte Settings-Instanz**.
- `test_api_fehler.py:5` macht `from conftest import FakeCapacityRepository` — funktioniert nur, weil `tests/__init__.py` fehlt und pytest im Prepend-Modus den Ordner in `sys.path` schiebt. Die Fakes gehören in ein eigenes `tests/fakes.py`.

## T6 · Ein Test zeigt einen Design-Smell an

`test_reference_date_falls_back_today` monkeypatcht das gesamte `date`-Symbol im Modul. Das funktioniert, ist aber Symptom: `RecommendationEngine.reference_date` (`recommendations.py:44-45`) ruft `date.today()` und ist damit unrein. Sauberer wäre, das Datum im Composition Root aufzulösen und hineinzureichen — dann braucht der Test keinen Monkeypatch mehr, und die Engine wird vollständig deterministisch.

# Betrieb und Tooling

- **Kein README.** Verifiziert. Die Docstrings sind gut, aber es gibt keinen Einstiegspunkt, der die Reihenfolge `docker compose up -d db` → `pricing-initdb` → `pricing-apikey create` → `pricing-ingest` → `uvicorn` erklärt. Diese Reihenfolge steht derzeit verstreut in vier Docstrings.
- **Die CLI ignoriert die eigene Konfiguration.** `main.py:25-26` und `ingest.py:275` hartkodieren `Path("daily_data")`, obwohl `config.py:29` `ingest_base_dir` definiert. Nur der API-Pfad liest die Einstellung. Ändert man die Umgebungsvariable, laufen CLI und API auseinander. Zudem akzeptiert `parse_verbosity` außer `-v` nichts — kein `--data-dir`, kein `--output`.
- **Kein `lifespan`** in `app.py`, die Engine wird nie disposed. Geringe Tragweite, weil `engine()` ohnehin ein Prozess-Singleton ist und das OS die Sockets beim Beenden freigibt — aber es ist die idiomatische Stelle dafür.
- **Kein Alembic.** `create_all()` ist in `schema_admin.py:1-6` als bewusste Entscheidung dokumentiert, und für den Kapitelstand ist das korrekt. Der Anschluss ist ohnehin schon geplant: [[1.6 Datenbank Migrations Tools - Leitfaden|Kapitel 1.6 Datenbank Migrations Tools]].
- `main.py:36` druckt bei leerem Ergebnis `Höchste Empfehlung: None`, sonst den rohen Dataclass-`repr`.

# Geprüft und nicht beanstandet

Diese Punkte sahen nach Befunden aus und sind bei genauerem Hinsehen **korrekt** — hier aufgeführt, damit sie nicht bei der nächsten Durchsicht erneut als Fehler markiert werden.

| Vermutung | Befund |
|---|---|
| `httpx2` in den dev-Dependencies sei ein Vertipper für `httpx` | **Falsch.** Starlette 1.6.0 macht `import httpx2 as httpx` (`testclient.py:33`) und stuft `httpx` explizit als deprecated ein. `httpx2` ist die korrekte, aktuelle Wahl. |
| `.env` liege im Repository | **Nein.** `.gitignore:5` (`*.env`) greift, `git check-ignore` bestätigt. Getrackt sind nur fünf Config-Dateien. |
| `.mypy_cache` / `.ruff_cache` / `.pytest_cache` fehlen in `.gitignore` | **Kein Problem.** Alle drei legen eine eigene `.gitignore` mit `*` in ihrem Verzeichnis an. `git status` ist sauber. |
| Bare `except:` in `bootstrap.py:70` sei ein Lint-Verstoß | **Nein, und es ist inhaltlich richtig.** Ruff lässt bare excepts, die re-raisen, bewusst durch (mit `--select E722` explizit geprüft). Für einen Generator-Dependency ist das Fangen von `BaseException`/`GeneratorExit` sogar notwendig, damit der Rollback auch beim Abbruch läuft. |
| Der Docker-Volume-Pfad `/var/lib/postgresql` sei falsch (statt `.../data`) | **Korrekt so.** Das `postgres:18`-Image hat den deklarierten Volume-Pfad genau dorthin verschoben. |
| Die 14 Trivial-Fabriken (7 in `bootstrap`, 7 in `deps`) seien vergessene Zusammenlegung | **Nein, es ist der Preis zweier bewusster Grenzen.** `deps.py:3-4` darf keine konkrete Implementierung nennen, `bootstrap.py:5-6` bewusst kein `fastapi` importieren (CLI-Startzeit). Beide Constraints zusammen erzwingen die zweistufige Indirektion. Anders als bei A2/A3 wird die Grenze hier tatsächlich eingehalten. |
| Die Exception-Handler in `errors.py` könnten sich durch Vererbung überdecken | **Sauber.** Starlette löst über die MRO auf: `UnknownReferenceError` → `DomainError` → 422, `ConflictError` → 409, `NotFoundError` → 404. Reihenfolge stimmt. |

# Priorisierung — die drei Dinge

Sortiert nach **Lernwert pro investierter Stunde**, nicht nach Schweregrad.

> [!success] 1. Den Application-Bypass reparieren — und dabei die Transaktionsfrage aufmachen
> **Betrifft:** A2, A3, A4 · **Aufwand:** ein Nachmittag
>
> `RunRepository` und `IngestionRunRepository` um echte Schreibmethoden erweitern (`mark_running`, `mark_succeeded`, `mark_failed`), dann `bootstrap`- und `orm`-Importe aus `application/` entfernen. Die Abhängigkeiten kommen als Parameter herein, so wie `PricingApp` es schon vormacht.
>
> Der Clou: Man **kann** das nicht tun, ohne zu klären, wo jetzt committet werden muss — weil Background-Tasks vor der Dependency-Cleanup laufen. Diese eine Änderung verbindet Dependency Injection, Protocol-Design und den FastAPI-Lifecycle in einer einzigen konkreten Aufgabe. Danach: `test_architektur.py` um eine Regel für `application/` erweitern, damit es nicht wieder erodiert.

> [!success] 2. Echte Postgres-Integrationstests für die IO-Schicht
> **Betrifft:** T1–T3 · **Aufwand:** ein bis zwei Tage
>
> `docker-compose.yml` steht bereits, `scripts/smoke_orm.py` enthält die halbe Arbeit — es muss in pytest-Fixtures verwandelt werden (Session-scoped Engine, Function-scoped Transaktion mit Rollback).
>
> Lernwert: Das Exception-Mapping in `_flush` (`ForeignKeyViolation` → 422, `UniqueViolation` → 409) lässt sich **prinzipiell nicht** mit einem Fake verifizieren — dafür braucht es eine echte Datenbank, die den Constraint wirklich verletzt. Und die 66-%-Zahl entlarvt sich dabei selbst: Der Unterschied zwischen Line-Coverage und Verhaltensabdeckung wird am eigenen Projekt sichtbar statt im Lehrbuch.

> [!success] 3. Den API-Vertrag aus Konsumentensicht durchspielen
> **Betrifft:** API1, API2, API3 · **Aufwand:** unter einer Stunde
>
> `cust_id`/`source_id`/`feature_2` in `BookingRead` aufnehmen, das `| None` bei `get_max_recommendation` streichen, `ping` den Schlüsselnamen zurückgeben lassen.
>
> Bewusst als Kontrast zu 1. und 2.: klein, schnell, sofort sichtbar. Die Leitfrage dahinter ist die eigentliche Übung — *„Kann ich mit dem, was mein GET liefert, tatsächlich ein PUT bauen?"* Diese Frage einmal an der eigenen API gestellt zu haben, verändert, wie man die nächste entwirft.

**Bewusst niedriger eingestuft**, obwohl real: Magic Numbers (D4) — wichtig, aber mechanisch. Generische Repository-Basis (D2) — guter Generics-Übungsstoff, aber reines Refactoring ohne neues Konzept. Exception-Hierarchie entwirren (A5) — lohnend, aber näher an Geschmack als an Mechanismus. Sprachkonvention (D7) — sollte entschieden werden, bevor mehr Code entsteht, kostet aber keine Denkarbeit.

---

## Fazit

Das Projekt zeigt zwei Dinge gleichzeitig, und beide sind wertvoll: ein sauber geschnittenes Batch-System mit domänentypisierten Ports — und daneben, in denselben Ordnern, die realistische Erosion derselben Prinzipien unter Feature-Druck. Der Abstand zwischen `PricingApp` und `run_service` ist das ehrlichste Lernmaterial im ganzen Repository.

Was hier über dem Erwartbaren liegt, ist nicht die Schichtung an sich, sondern das Werkzeugbewusstsein: ein AST-basierter Architekturtest, ein Vertragstest gegen stille DTO-Fehler, `COPY` mit Temp-Table statt naivem `INSERT`, `joinedload` gegen N+1. Das sind Dinge, die man normalerweise erst nach dem zweiten Produktionsvorfall baut.
