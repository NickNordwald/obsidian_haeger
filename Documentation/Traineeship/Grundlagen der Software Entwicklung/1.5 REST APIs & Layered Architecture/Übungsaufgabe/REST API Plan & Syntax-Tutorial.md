---
tags:
  - documentation
type: notiz
date: 2026-08-10
---

Abhakbarer **Plan für die REST API** der Pricing-Anwendung — im Tutorial-Stil, damit die neue Syntax genau dort erklärt wird, wo du sie zum ersten Mal brauchst. Begleitnotiz zum [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]]; Theorie dahinter: [[1.5 REST APIs & Layered Architecture - Vertiefung|1.5 Vertiefung]], Syntax-Referenz: [[FastAPI, Uvicorn & Pydantic - Syntax|FastAPI, Uvicorn & Pydantic]]. Vorbilder aus den letzten Kapiteln: [[Test-Suite Plan & Syntax-Tutorial|Test-Suite Plan]] · [[Logging & Exceptions Plan & Syntax-Tutorial|Logging & Exceptions Plan]].

> [!success] Abgeschlossen am 2026-08-13 — alle 17 Blöcke durch
> Gebaut wurden **24 Operationen auf 16 Pfaden in acht Routern**, nicht die unten geplanten 27 Endpunkte: **Block 10 (`prices`) ist bewusst übersprungen** (die CRUD-Vorgabe war schon erfüllt), dafür kamen die drei Endpunkte je Lauf-Ressource dazu. Gates am Ende: Ruff sauber, `mypy --strict` über 63 Dateien, **123 Tests grün**.
>
> Vier Blöcke hat Claudian auf Ansage übernommen (13, 15, 16, 17), die übrigen hat Nick geschrieben. Fünf Kästchen sind offen geblieben: die drei von Block 10 (⏭️ übersprungen), ein `docker`-Test, der in Claudians Umgebung nicht lief, und der `API_KEY`-Eintrag in der `.env`, der nur für `curl` gedacht ist. Ergebnis und Befunde stehen im [[Umsetzungsstand (REST APIs & Layered Architecture)|Umsetzungsstand]].

> [!info] So arbeitest du mit dieser Liste
> Die Blöcke sind **von einfach nach komplex** sortiert; jeder führt genau **eine** neue Technik ein und baut auf dem vorigen auf. Der Reihe nach abarbeiten, `- [ ]` abhaken, wenn es läuft. **Du schreibst den Code, ich reviewe.** Die Schnipsel sind *Muster*, keine fertigen Lösungen — Namen, Texte und Feinheiten entscheidest du.
>
> In jedem Block sagt ein **→ Was du tust**-Absatz konkret, welche Datei dran ist.
>
> **Zum Code in den Blöcken:** Gezeigt wird nur, was man **nicht raten kann** — neue Syntax, unintuitive Schalter, Fallstricke. Handler-Rümpfe, Abfragen und Schemafelder stehen bewusst *nicht* da; die Kästchen beschreiben, was hinterher wahr sein muss, nicht wie du dahin kommst. Wenn ein Block dir zu wenig Anhalt gibt, frag nach — das ist billiger, als wenn ich dir die Lösung vorwegschreibe.

# Das Design — und warum es so aussieht

## Die Endpunkt-Landkarte

**27 Endpunkte in acht Routern.** Die Leitfaden-Vorgabe (je 2 × `POST`/`PUT`/`GET`/`DELETE`) ist damit deutlich übererfüllt.

| Router | Endpunkte | Zweck |
|---|---|---|
| `health` | `GET /health` | Lebenszeichen — **der einzige offene Endpunkt** |
| `products` | `GET /products` · `GET /products/{product_id}` | nur lesen — werden im Ingest abgeleitet |
| `bookings` | `GET` · `GET /{id}` · `POST` · `PUT /{id}` · `DELETE /{id}` | echtes CRUD |
| `prices` | `GET` · `GET /{id}` · `POST` · `PUT /{id}` · `DELETE /{id}` | echtes CRUD |
| `capacity` | `GET` · `GET /{id}` · `POST` · `PUT /{id}` · `DELETE /{id}` | CRUD **mit Konflikten** — siehe Entscheidung 2 |
| `recommendations` | `GET` · `GET /max` · `GET /{id}` · `GET /by-unit/{product_id}/{date}` | nur lesend + Bewerberaufgabe 3 |
| `pricing-runs` | `POST` · `GET` · `GET /{id}` | Bewerberaufgabe 2, asynchron |
| `ingestion-runs` | `POST` · `GET /{id}` | Bewerberaufgabe 1, asynchron |

**`customers` bekommt bewusst keine Ressource** — die Bewerberaufgabe nennt `cust_id` ausdrücklich irrelevant. Eine Ressource anzubieten, die fachlich niemanden interessiert, wäre genau das, wovor die [[Weiterführende Stichpunkte|Q&A-Notiz]] warnt.

> [!warning] Die drei Endpunkte der Bewerberaufgabe — nicht zwei
> Leitfaden und Vertiefung sprechen von „den 2 Endpunkten". Die [[Ursprüngliche Aufgabe|Ursprüngliche Aufgabe]] verlangt aber **drei**: `/start-ingestion`, `/price-recommendations` und `/max-recommendation`. Vermutlich ist die Ingestion gemeint als „schon durch `pricing-ingest` erledigt". Der Plan baut alle drei — der dritte fällt fast geschenkt ab, sobald das Muster für lang laufende Läufe einmal steht. **Bei Gelegenheit mit den Trainern klären.**

## Die Entscheidungen vom 2026-08-10

**1. Element-URL der Empfehlungen: interne ID kanonisch, fachliches Paar als Alias.**

```
GET /recommendations/{id}                              ← kanonisch, liefert 200
GET /recommendations/by-unit/{product_id}/{date}       ← Alias, liefert 307 → kanonisch
```

Der Alias wird als **Weiterleitung** umgesetzt, nicht als zweiter Handler mit eigener Antwort. Grund: Zwei URLs, die dieselbe Repräsentation mit `200` zurückgeben, sind zwei Ressourcen im Sinne von HTTP — mit getrennten Cache-Einträgen, doppelter Lese-Logik und der Frage, welche der beiden in einem `Location`-Header stehen soll. Ein `307` beantwortet das: Es gibt **eine** Ressource, und der Alias sagt, wo sie wohnt. Der Preis ist ein zusätzlicher Roundtrip, für BI-Clients irrelevant.

*Falls dir der Roundtrip später doch stört: beide mit `200` antworten zu lassen ist vertretbar, aber dann bitte bewusst und dokumentiert.*

**2. `capacity` bekommt volles CRUD.** Meine Empfehlung war „nur lesen", weil `pricing-ingest` die Tabelle bei jedem Lauf truncatet — über die API angelegte Zeilen sind danach weg. Das bleibt wahr und **gehört in die Endpunkt-Beschreibung in Swagger**.

Dafür ist deine Wahl die lehrreichere, und zwar aus einem Grund, der beim Entscheiden noch nicht auf dem Tisch lag: **`capacity` ist die einzige Tabelle, an der Konflikte echt werden.**

| Fall | Was passiert in der DB | Richtige Antwort |
|---|---|---|
| `POST /capacity` mit bereits existierendem `(product_id, calendar_date)` | verletzt `uq_capacity_product_id` | `409 Conflict` |
| `POST`/`PUT` mit unbekannter `product_id` | verletzt `fk_capacity_product_id_products` | `422` (Eingabe unplausibel) |
| `DELETE /capacity/{id}`, auf das Empfehlungen zeigen | verletzt `fk_price_recommendations_capacity_id` | `409 Conflict` |

Bei `bookings` und `prices` gibt es keine Unique-Constraints außer dem Primärschlüssel, den die DB selbst vergibt — dort wäre `409` ein theoretisches Kapitel geblieben. Hier wird es eins mit Code. Das ist auch der Anlass, `ConflictError` in `exceptions.py` zu ergänzen (stand ohnehin auf der Liste).

**3. `POST /pricing-runs` antwortet `202` und rechnet im Hintergrund.** Die [[Umsetzungsstand (REST APIs & Layered Architecture)#Laufzeit-Messung (2026-08-04)|42-Sekunden-Messung]] hat das entschieden. Braucht eine neue Tabelle `pricing_runs`.

**4. Authentifizierung kommt direkt mit: API-Keys, gehasht in der Datenbank.** Header `X-API-Key`, **alles außer `/health`** geschützt. Details in Block 2 (ausstellen) und Block 3 (prüfen) — die deshalb *vorne* stehen: Nachrüsten hieße 26 geschützte Endpunkte und ~30 Tests anfassen.

Warum überhaupt, obwohl der Leitfaden es nicht verlangt: Zwei Endpunkte sind echte Kandidaten. `POST /ingestion-runs` nimmt laut Aufgabe einen **serverseitigen Ordnerpfad vom Client** entgegen (Path Traversal), und `POST /pricing-runs` kostet 16–42 s CPU (ein kostenloser DoS-Knopf). Die CRUD-Endpunkte sind demgegenüber harmlos — geschützt werden sie trotzdem, weil eine Regel ohne Ausnahmen weniger Fehler produziert als eine mit.

**Warum API-Keys und nicht OAuth2/JWT:** Die Anwendung hat **keinen Nutzerbegriff** — die `customers`-Tabelle sind Hotelgäste, keine API-Nutzer. Für ein Passwort-Login müsste man sich Nutzer ausdenken. Die Konsumenten sind BI-Werkzeuge und Skripte, also Maschine-zu-Maschine, und dafür ist ein Schlüssel im Header der passende Mechanismus. JWT bleibt ein Thema für später — mit dem Widerrufsproblem, das im [[Weiterführende Stichpunkte|Q&A]] schon beschrieben ist.

**Cookies wären hier die falsche Wahl** — nicht aus Geschmack: Der Browser hängt Cookies **automatisch** an jede Anfrage an die Domain, woraus überhaupt erst CSRF entsteht. Ein `X-API-Key`- oder `Authorization`-Header wird nie automatisch mitgeschickt, weshalb dieser Angriff hier strukturell nicht existiert.

## Versionierung: Vorschlag `/v1`

Alle Router unter `/v1` einhängen — **eine Zeile** beim `include_router`, und die Entscheidung ist später nicht mehr nachholbar, ohne jeden Client zu brechen. Kosten heute: null. *Wenn du das anders willst, sag es vor Block 4 — danach steht es in 27 Pfaden.*

## Die Schichtregel für diese Iteration

> **Die Presentation-Schicht ruft nur die Application-Schicht auf und kennt weder `persistence/` noch SQLAlchemy.**

**Konkretisiert in Block 4:** ORM-Objekte dürfen bis in die **Application**-Schicht (`application → persistence` zeigt nach unten), aber **nicht** in die Presentation-Schicht. Die Umwandlung in ein Pydantic-DTO passiert im Router — genau an dieser Grenze. Grund: Für CRUD gibt es keine passenden Domänen-Objekte (`orm.Booking` hat 15 Spalten, `domain.Booking` vier), weil die Domänen-Entitäten Ausschnitte für die Preisrechnung sind, keine Abbilder der Tabellen. **Die Asymmetrie ist die Lektion: Domänen-Objekte gibt es, wo es Regeln gibt.**

### Die Sinkhole-Frage — entschieden am 2026-08-11

Eine Application-Schicht, die nur `session.get()` weiterreicht, ist ein **Sinkhole**: eine Schicht, die nichts tut außer durchreichen. Der Plan hatte zunächst ein `crud_service.py` mit rund 20 solchen Funktionen vorgesehen und die Auswertung ans Ende verschoben. **Entscheidung: Es entfällt.** Die Trennlinie lautet:

> **`application/` enthält, wo Regeln sind. CRUD hat keine.**

| Pfad | Application-Schicht | Warum |
|---|---|---|
| Preislauf | `pricing_app.py` | orchestriert laden → bauen → rechnen → schreiben |
| Authentifizierung | `auth_service.py` | hier landen „gilt der Schlüssel noch?", Ablaufzeit, Ratenbegrenzung |
| Läufe | `run_service.py` | Statusübergänge, Fehlerbehandlung, eigene Session |
| **CRUD** | **keine** | Zeilen verschieben ist keine Regel |

Der CRUD-Pfad geht damit **Router → Repository → SQL**. Die Schichtregel bleibt trotzdem prüfbar, weil die Router das Repository nur über den `…RepositoryDep`-Alias aus `deps.py` bekommen — sie nennen **Verträge, keine Implementierungen**, und importieren weder `orm` noch `sqlalchemy`.

*Was die Repositories trotzdem verdienen: **Kohäsion.** Alles SQL einer Entität an einem Ort — spätestens bei den fünf bedingten Filtern in Block 6 und dem JOIN in Block 12 will man das nicht im Handler haben. Das ist ein Argument, das nichts mit Abstraktion zu tun hat.*

## Dateistruktur

`←` markiert Neues, alles Übrige bleibt unangetastet. **`domain/` wird nicht angefasst** — kein einziger Import ändert sich dort.

```
src/pricing/
├── main.py                       Composition Root der CLI (bleibt)
├── bootstrap.py                  ← baut den Objektgraphen; CLI und API rufen ihn
│
├── domain/                       unverändert
│   ├── entities.py
│   ├── builder.py
│   └── recommendations.py
│
├── application/
│   ├── pricing_app.py            (bleibt) compute() / run()
│   ├── auth_service.py           ← Schlüssel prüfen (Block 3)
│   └── run_service.py            ← Läufe starten, Status fortschreiben (Block 14)
│                                   KEIN crud_service — CRUD hat keine Regeln
│
├── persistence/
│   ├── interfaces.py             (+ Repository-Protocols — Verträge bleiben hier)
│   ├── api_keys.py               ← Schlüssel erzeugen, hashen, nachschlagen (Block 3)
│   ├── api_key_admin.py          ← CLI `pricing-apikey` (Block 2)
│   ├── repositories.py           ← Repositories für bookings/prices/capacity (ab Block 7)
│   ├── orm.py                    (+ api_keys · pricing_runs, ingestion_runs ab Block 14)
│   ├── sources.py                unverändert
│   ├── writers.py                unverändert
│   ├── db.py · ingest.py · schema_admin.py    unverändert
│
├── presentation/
│   └── api/
│       ├── app.py                ← FastAPI-Instanz + include_router, sonst nichts
│       ├── deps.py               ← get_session, SessionDep, require_api_key
│       ├── errors.py             ← Exception→Status-Handler (Block 5, erweitert in 11)
│       ├── schemas/              ← ein Modul je Entität
│       │   ├── product.py        ProductRead
│       │   ├── booking.py        BookingCreate / BookingUpdate / BookingRead
│       │   ├── price.py          PriceCreate / PriceUpdate / PriceRead
│       │   ├── capacity.py       CapacityCreate / CapacityUpdate / CapacityRead
│       │   ├── recommendation.py RecommendationRead
│       │   └── run.py            RunRead, StartPricingRun, StartIngestionRun
│       └── routers/              ← ein Modul je Ressource
│           ├── health.py · products.py · bookings.py · prices.py
│           ├── capacity.py · recommendations.py
│           └── pricing_runs.py · ingestion_runs.py
│
└── shared/
    ├── exceptions.py             (+ NotFoundError, ConflictError, AuthenticationError)
    ├── config.py                 (+ ingest_base_dir für Block 15)
    └── cli.py · logging_setup.py unverändert

tests/
├── conftest.py                   ← endlich Inhalt: TestClient-Fixtures (Block 17)
├── test_architecture.py          ← prüft die Schichtregel (Block 17)
├── test_domain.py … test_app.py  (bleiben, 57 Tests)
└── api/                          ← neuer Unterordner
    ├── test_auth.py               (401 ohne / mit falschem / mit widerrufenem Schlüssel)
    ├── test_health.py
    ├── test_bookings.py
    ├── test_capacity_conflicts.py
    ├── test_recommendations.py   (inkl. Routen-Reihenfolge `/max`)
    └── test_pricing_runs.py
```

### Warum genau so — die sechs Entscheidungen dahinter

**1. Die Verträge bleiben in `interfaces.py`, die Implementierungen daneben.** Genau das Muster, das schon existiert: `interfaces.py` besitzt die `Protocol`s, `sources.py` und `writers.py` implementieren sie. Die Repositories fügen sich ein, statt ein neues Muster aufzumachen. Passt zur Architekturentscheidung *„die Persistenzschicht besitzt ihre Verträge"*.

**2. `repositories.py` ist nicht `sources.py`.** `DataSource` heißt „lade **alle** Kapazitätszeilen" — auf die Pipeline zugeschnitten. Ein Repository heißt „hol Buchung 42 / lege an / lösche". Das sind **getrennte Verträge**, keine Erweiterung von `DataSource`. Stand so schon als Absicht im Umsetzungsstand.

**3. `repositories.py` bleibt zunächst eine einzige Datei** — obwohl vier Module symmetrischer aussähen. Die Wiederholung zwischen den Entitäten wird nur sichtbar, wenn sie **untereinander** steht; erst danach weiß man, was wirklich gemeinsam ist. Aufteilen geht später in fünf Minuten, verfrüht aufteilen versteckt die Frage hinter Struktur. Gleiches Argument wie in Block 10.

**4. `schemas/` und `routers/` sind dagegen sofort Pakete.** Hier unterscheiden sich die Entitäten wirklich: Jeder Router bringt fünf Endpunkte mit Dekoratoren, Docstrings und `responses`-Deklarationen mit — in einer Datei wären das schnell 400 Zeilen. Und der 1:1-Spiegel `routers/booking**s**.py` ↔ `schemas/booking.py` macht das Suchen überflüssig.

**5. `errors.py` getrennt von `app.py`.** `app.py` soll in zwanzig Sekunden lesbar bleiben und genau eine Frage beantworten: *welche Router gibt es?* Sechs Exception-Handler samt Zuordnungstabelle würden die Datei verdoppeln und diese Frage zuschütten.

**6. `bootstrap.py` an der Paketwurzel, nicht in `application/`.** Es wählt die **konkreten** Adapter aus (CSV oder SQLAlchemy) — das ist eine Kompositions-, keine Anwendungsentscheidung. Composition Roots sind von der Schichtregel ausgenommen, genau wie `main.py`. Läge es in `application/`, würde man diese Ausnahme stillschweigend zur Normalität erklären.

> [!note] Modulnamen mit Unterstrich, URLs mit Bindestrich
> `routers/pricing_runs.py` bedient `/v1/pricing-runs`. Kein Widerspruch: Python-Module dürfen keine Bindestriche enthalten, URL-Pfade bevorzugen sie (Naming-Konvention aus der Vertiefung). Die Übersetzung passiert im `prefix` des Routers.

> [!tip] Starten in der Entwicklung
> `uv run uvicorn pricing.presentation.api.app:app --reload`. Ein vierter Entry Point `pricing-api` in `pyproject.toml` wäre möglich, bringt aber wenig: `--reload` braucht ohnehin die Import-String-Form, weil der Watcher das Modul neu laden können muss.

---

# Block 1 — Gerüst und `GET /health`

**Neue Technik:** FastAPI-Instanz, `APIRouter`, `include_router`, Uvicorn starten.

**→ Was du tust:** Abhängigkeiten aufnehmen, `presentation/api/` anlegen, einen Router mit einem Endpunkt, App starten, `/docs` im Browser ansehen.

```bash
uv add fastapi "uvicorn[standard]"
```

> [!warning] Korrektur (2026-08-13) — hier stand ursprünglich `uv add --optional api`
> Ich hatte FastAPI als **Extra** vorgeschlagen, mit der Begründung, der CLI-Batch-Lauf brauche es nicht. Das war ein Rückfall in genau den Fehler, den du in der uv-Migration schon behoben hattest: Die Alt-Extras `db`/`orm` sind entfallen, weil `sqlalchemy` **auf Modulebene** importiert wird und damit keine Wahlmöglichkeit ist. Für FastAPI gilt dasselbe — `presentation/api/app.py` und `deps.py` importieren es auf Modulebene.
>
> Ein Extra ist das richtige Werkzeug, wenn eine Abhängigkeit **wahlweise** dazukommt. Bei einer Hauptfunktion der Anwendung ist es nur eine zusätzliche Sache, die man vergessen kann — und `uv run` entfernt sie sogar aktiv wieder (siehe Block 13.0). FastAPI und Uvicorn stehen deshalb in `[project.dependencies]`, und `[project.optional-dependencies]` bleibt entfallen. `uv sync` ohne Flags stellt den vollständigen Zustand her.

```python
# presentation/api/routers/health.py
router = APIRouter(tags=["health"])

@router.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

```python
# presentation/api/app.py
app = FastAPI(title="Pricing API", version="0.1.0")
app.include_router(health.router, prefix="/v1")   # ← hier entsteht die Versionierung
```

- [x] `fastapi` + `uvicorn[standard]` als Extra `api`, `uv.lock` aktualisiert
- [x] `presentation/api/{app.py,deps.py,routers/,schemas/}` mit `__init__.py`
- [x] `GET /v1/health` liefert `200` und `{"status": "ok"}`
- [x] `uv run uvicorn pricing.presentation.api.app:app --reload` läuft
- [x] `/docs` zeigt den Endpunkt, `/openapi.json` liefert das Dokument

> [!tip] `--reload` ist ein Entwicklungsschalter
> Er startet einen Watcher-Prozess, der bei jeder Dateiänderung neu lädt. In Produktion nie benutzen: doppelter Speicher und ein Neustart bei jedem `touch`.

> [!warning] Warum `pricing.presentation.api.app:app` und nicht `app.py`
> Uvicorn bekommt einen **Import-Pfad**, keinen Dateipfad — `modul:objekt`. Das ist derselbe Mechanismus wie bei den Entry Points in `pyproject.toml`, und derselbe Fallstrick: ein String, den kein Werkzeug prüft.

---

# Block 2 — Schlüssel ausstellen: Tabelle und Kommando

**Neue Technik:** bewusst **keine** FastAPI-Technik. Du benutzt nur, was du schon kannst — ein ORM-Modell, eine Session, einen Entry Point nach dem Muster von `schema_admin.py`. Neu ist allein das Sicherheitsdenken.

**→ Was du tust:** Tabelle `api_keys` in `orm.py`, `pricing-initdb` laufen lassen, Kommando `pricing-apikey`.

## Warum SHA-256 — und ausdrücklich **kein** bcrypt

Der reflexhafte Gedanke ist „Zugangsdaten hasht man mit bcrypt". Hier wäre das ein Fehler, und der Grund ist lehrreich:

Passwort-Hashing ist **absichtlich langsam**, weil menschliche Passwörter wenig Entropie haben — jeder Rateversuch soll teuer sein. Ein Schlüssel aus `secrets.token_urlsafe(32)` hat dagegen **256 Bit Entropie**; den errät niemand, egal wie schnell die Hashfunktion läuft. Daraus folgen zwei Dinge:

- **bcrypt kostet hier nur.** Der Hash wird bei **jeder einzelnen Anfrage** berechnet. Mit Kostenfaktor 12 sind das grob 250 ms pro Request — für null Sicherheitsgewinn.
- **Kein Salt nötig.** Salz schützt gegen Rainbow Tables, und die gibt es für 256-Bit-Zufall nicht. Der Verzicht zahlt sich sogar aus: Ein **deterministischer** Hash ist indizierbar, du findest die Zeile also mit einem einzigen `WHERE key_hash = …` statt alle Schlüssel zu laden und einzeln zu vergleichen.

**Keine neue Abhängigkeit** — `secrets` und `hashlib` sind Standardbibliothek.

## Die Tabelle

| Spalte | Zweck |
|---|---|
| `id` | Primärschlüssel |
| `name` | wofür der Schlüssel ist („BI-Tool", „Nick lokal") |
| `key_hash` | SHA-256 des Schlüssels, `UNIQUE` + Index |
| `created_at` | `server_default=func.now()` |
| `revoked_at` | `NULL` = gültig |
| `last_used_at` | optional, siehe Warnung unten |

`name` ist kein Schmuck: Ohne ihn weiß nach drei Monaten niemand mehr, welchen Schlüssel man widerrufen darf. Und `revoked_at` statt Löschen, damit nachvollziehbar bleibt, dass es ihn gab.

## Das Henne-Ei-Problem

Wer legt den **ersten** Schlüssel an? Nicht die API — ein unauthentifizierter Endpunkt, der Schlüssel ausstellt, wäre genau die Lücke, die wir schließen wollen. Also ein viertes Kommando:

```
uv run pricing-apikey create "Nick lokal"
→ pk_8Xr2…                  (nur JETZT sichtbar)
```

Gespeichert wird **nur der Hash**. Dieselbe Eigenschaft haben GitHub-Tokens: Wer die Datenbank liest, hält keinen gültigen Schlüssel in der Hand.

## Was am Ende funktionieren muss

```bash
uv run pricing-apikey create "Nick lokal"          # gibt den Schlüssel aus, genau einmal
uv run pricing-apikey list                         # id, name, angelegt, Status — nie der Hash
uv run pricing-apikey revoke 1                     # setzt revoked_at
uv run pricing-apikey --help                       # dokumentiert die drei Unterbefehle
```

Vier Schritte dahin. Signaturen sind vorgegeben, **die Rümpfe schreibst du.**

---

### Schritt 2.1 — Tabelle `api_keys`

- [x] Modell in `orm.py`, Naming Convention und `Identity(always=True)` wie bei den anderen Tabellen
- [x] `datetime: DateTime(timezone=True)` in die `type_annotation_map` der `Base` — projektweit, damit es niemand mehr vergessen kann
- [x] `UniqueConstraint("key_hash")` — verhindert Duplikate **und** liefert den Index, auf dem später jede Anfrage nachschlägt
- [x] `__table_args__` ist ein **Tupel** (siehe Falle unten)
- [x] Tabelle einmal droppen und von `pricing-initdb` neu anlegen lassen

> [!bug] `__table_args__` muss ein Tupel sein — auch bei nur einem Eintrag
> ```python
> __table_args__ = UniqueConstraint("key_hash")      # ❌ ArgumentError beim Import
> __table_args__ = (UniqueConstraint("key_hash"),)   # ✅ Komma macht das Tupel
> ```
> Ohne das Komma ist `(x)` in Python einfach `x` in Klammern — ein Tupel entsteht erst durch das Komma. SQLAlchemy prüft den Typ und wirft `ArgumentError: __table_args__ value must be a tuple, dict, or None` **beim Import des Moduls**, nicht erst beim Zugriff. Damit ist `pricing-initdb` sofort kaputt.

> [!warning] `create_all()` ändert bestehende Tabellen nicht
> Die Tabelle existiert schon ohne den Unique-Constraint. `pricing-initdb` wird sie **nicht** nachbessern — es legt nur *fehlende* Tabellen an. Einmal `DROP TABLE api_keys;` (sie ist leer, also gefahrlos), dann neu anlegen. Genau diese Lücke schließt das Migrationswerkzeug im nächsten Kapitel.

---

### Schritt 2.2 — `shared/cli.py` zerlegen

**Das Problem:** `parse_verbosity` baut den Parser selbst und ruft `parse_args()`. Bei `pricing-apikey create --name X` bekommst du `error: unrecognized arguments`. Die Funktion vermischt zwei Aufgaben: *das `-v`-Flag definieren* und *die ganze Kommandozeile besitzen*.

**Ziel — zwei Bausteine, der alte Aufruf bleibt bestehen:**

```python
def add_verbosity(parser: argparse.ArgumentParser) -> None:
    """Haengt -v/--verbose an einen Parser, den der Aufrufer besitzt."""

def level_from(verbose: int, default_level: str = "WARNING") -> str:
    """Rechnet die Anzahl der -v auf der Leiter in einen Level-Namen um."""

def parse_verbosity(prog: str, argv: list[str] | None, default_level: str = "WARNING") -> str:
    """Bleibt fuer die drei einfachen Kommandos — benutzt jetzt die beiden oben."""
```

- [x] `_LADDER` und die `min(...)`-Logik wandern nach `level_from`, sonst steht sie zweimal da
- [x] `parse_verbosity` ist danach ein Dreizeiler, der die beiden neuen Funktionen aufruft
- [x] `main.py`, `ingest.py`, `schema_admin.py` **unverändert**, 57 Tests grün

---

### Schritt 2.3 — `persistence/api_key_admin.py`: die drei Operationen

*Von Claudian geschrieben (2026-08-10, auf Ansage) — Durchsprache im Chat.*

Neue Datei, Geschwister von `schema_admin.py`. Fünf kleine Funktionen:

```python
KEY_PREFIX = "pk_"

def generate_key() -> str: ...
def hash_key(raw_key: str) -> str: ...
def create_key(session: Session, name: str) -> str: ...
def revoke_key(session: Session, key_id: int) -> None: ...
def list_keys(session: Session) -> Sequence[ApiKey]: ...
```

Was jede tun soll:

| Funktion | Inhalt | Warum so |
|---|---|---|
| `generate_key` | `KEY_PREFIX` + `secrets.token_urlsafe(32)` | 32 Zufallsbytes → 43 Zeichen, 256 Bit. **`secrets`, nicht `random`** — `random` ist vorhersagbar und für Geheimnisse untauglich. Das Präfix macht den Schlüssel in Logs erkennbar |
| `hash_key` | `hashlib.sha256(raw_key.encode()).hexdigest()` | 64 Hex-Zeichen. **Eigene Funktion**, weil Block 3 sie beim Prüfen wiederbraucht — beide Seiten müssen zwingend identisch hashen |
| `create_key` | Schlüssel erzeugen, hashen, Zeile anlegen, **den Klartext zurückgeben** | Der Klartext existiert nur in diesem Moment. Danach gibt es ihn nirgends mehr |
| `revoke_key` | `revoked_at` auf jetzt setzen | Unbekannte `id` → `NotFoundError`; ein zweiter Widerruf desselben Schlüssels darf ruhig durchgehen |
| `list_keys` | alle Zeilen, nach `id` sortiert | Rückgabe von ORM-Objekten ist hier in Ordnung: Aufrufer und Funktion liegen in derselben Schicht |

- [x] **`create_key` committet nicht.** Die Transaktionsgrenze gehört dem Aufrufer — dieselbe Entscheidung wie bei den Adaptern
- [x] **`create_key` gibt nichts aus.** Kein `print`, kein Logger mit dem Schlüssel darin. Ausgabe ist Sache von `main()`
- [x] `hash_key` bekommt den **vollständigen** String inklusive Präfix — genau das, was der Client später im Header schickt

> [!warning] Warum die Trennung von Erzeugen und Ausgeben wichtig ist
> Eine Funktion, die selbst druckt, ist nicht testbar und nicht wiederverwendbar. `create_key` gibt den Schlüssel **zurück**; wer ihn wohin schreibt, entscheidet die Schicht, die den Menschen kennt.

---

### Schritt 2.4 — `main()` mit Unterbefehlen

`argparse` kann Unterbefehle selbst. Das Gerüst — reine Bauanleitung, die Logik kommt von dir:

```python
def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(prog="pricing-apikey")
    add_verbosity(parser)
    sub = parser.add_subparsers(dest="command", required=True)   # ← required!

    p_create = sub.add_parser("create", help="neuen Schluessel ausstellen")
    p_create.add_argument("name", help="wofuer der Schluessel ist")

    p_revoke = sub.add_parser("revoke", help="Schluessel widerrufen")
    p_revoke.add_argument("key_id", type=int)                    # Positionsargument

    sub.add_parser("list", help="Schluessel auflisten")
    return parser
```

`main(argv)` macht dann der Reihe nach:

1. Parser bauen, `parse_args(argv)` — **`argv` als Parameter**, nicht `sys.argv` lesen (die Lektion aus Kapitel 1.3)
2. `setup_logging(...)` mit `level_from(args.verbose)`, wie in den anderen drei Kommandos
3. `get_engine()` holen
4. `with Session(engine) as session:` — darin nach `args.command` verzweigen, danach `session.commit()`
5. Ergebnis ausgeben (siehe unten)
6. `except PricingError` → `1`, `except Exception` → `2`, sonst `0` — Muster von `schema_admin.py`

- [x] `dest="command", required=True` — ohne `required` ist `args.command` bei nacktem Aufruf `None` und du läufst in einen `AttributeError` statt in eine Hilfemeldung
- [x] Der **Schlüssel geht nach `stdout`**, alles andere über den Logger (der schreibt nach `stderr`)
- [x] `list` gibt `id`, `name`, `created_at` und Status („aktiv" / „widerrufen am …") aus — **nie** `key_hash`
- [x] Entry Point `pricing-apikey = "pricing.persistence.api_key_admin:main"` in `pyproject.toml`

> [!tip] Warum stdout und stderr getrennt gehören
> ```bash
> uv run pricing-apikey create --name "CI" > key.txt
> ```
> Nur wenn der Schlüssel auf **stdout** und die Logzeilen auf **stderr** gehen, steht in `key.txt` hinterher genau der Schlüssel und nichts sonst. Dein `logging_setup` schreibt schon auf `stderr` — du musst also nur `print()` für den Schlüssel benutzen und ihn *nicht* zusätzlich loggen.

> [!warning] Der Schlüssel darf nicht ins Log
> Ein `logger.info("Schluessel %s angelegt", key)` würde ihn in `logs/pricing.log` verewigen — und damit die ganze Übung entwerten. Loggen darfst du Name und `id`, nie den Schlüssel selbst.

---

### Schritt 2.5 — Ausprobieren

- [x] `uv run pricing-apikey --help` zeigt die drei Unterbefehle
- [x] `create` gibt den Schlüssel aus; ein zweiter `create` mit demselben Namen geht durch (zwei Schlüssel, ein Zweck)
- [x] `psql` zeigt in `api_keys` **keinen** Klartext, nur 64 Hex-Zeichen
- [x] `create ... > key.txt` — in der Datei steht nur der Schlüssel
- [x] `list` zeigt beide, `revoke <id>` setzt `revoked_at`, `list` zeigt es an
- [x] Der Schlüssel landet in deiner `.env`, nicht im Repository

> [!warning] `last_used_at` ist nicht gratis
> Diese Spalte bedeutet ein `UPDATE` bei **jeder** Anfrage — aus einem reinen Lesepfad wird ein Schreibpfad. Entweder weglassen oder nur fortschreiben, wenn der letzte Eintrag älter als etwa eine Stunde ist. Für „wann wurde dieser Schlüssel zuletzt benutzt?" reicht das völlig.

> [!warning] Der Schlüssel gehört nicht ins Repository
> Ein Schlüssel im Quelltext ist derselbe Fehler wie ein Passwort im Quelltext. Die `.env` steht seit Kapitel 1.4 nicht ohne Grund in der `.gitignore`.

---

# Block 3 — Schlüssel prüfen: die erste Dependency

**Neue Technik:** `Annotated[…, Depends(…)]`, Dependency mit `yield`, Dependency-Ketten, `APIKeyHeader`, `HTTPException`, `select().where()`.

Der Block mit der meisten neuen Syntax im ganzen Plan. Deshalb **sieben kleine Schritte** und jede Signatur vorgegeben. *Alle Syntax hier ist gegen die installierte FastAPI-Version geprüft, nicht aus dem Gedächtnis geschrieben.*

## Was am Ende funktionieren muss

```bash
KEY=$(grep API_KEY .env | cut -d= -f2)

curl -i localhost:8000/v1/health                            # 200 — offen
curl -i localhost:8000/v1/products                          # 401 + WWW-Authenticate: ApiKey
curl -i -H "X-API-Key: falsch" localhost:8000/v1/products   # 401
curl -i -H "X-API-Key: $KEY"  localhost:8000/v1/products    # 404 (Router kommt erst in Block 4)
```

Und in `/docs` erscheint oben rechts ein **Authorize**-Knopf, über den sich alle geschützten Endpunkte ausprobieren lassen.

## Der Weg einer Anfrage — das Zielbild

```
HTTP-Anfrage mit Header X-API-Key: pk_…
      │
      ├─ APIKeyHeader      liest den Header aus, gibt str | None            [presentation]
      ├─ get_session       öffnet eine Session, schließt sie nach der Antwort [presentation]
      ├─ get_repository    baut SqlAlchemyApiKeyRepository(session)         [presentation]
      │        │
      │        └─ authenticate(repository, raw_key) -> str | None           [application]
      │                 └─ repository.find_active_name(raw_key)            [persistence]
      │                          └─ hash_key() + SELECT … WHERE …
      │
      └─ require_api_key   None → HTTPException(401) · sonst weiter zum Handler
```

**Fünf Funktionen in vier Dateien.** Von unten nach oben gebaut, damit jeder Schritt für sich lauffähig ist.

## Die neue Syntax, gesammelt

**1. `Annotated[T, Depends(f)]` — das zentrale Muster.**

```python
SessionDep = Annotated[Session, Depends(get_session)]

def handler(session: SessionDep) -> ...:      # ← FastAPI ruft get_session() auf
```

`Annotated[T, meta]` ist „Typ `T`, plus Metadaten, die Werkzeuge lesen können". FastAPI liest die Metadaten und erfährt daraus: *woher* kommt der Wert dieses Parameters. Der Typ `T` bleibt für mypy und deinen Editor gültig.

Ohne Alias müsstest du in **jedem** Handler `session: Session = Depends(get_session)` schreiben — der Alias sammelt das an einer Stelle. Das ist die moderne Form; in älteren Tutorials steht noch die Default-Wert-Variante.

**2. Dependency mit `yield`** — alles vor `yield` läuft **vor** dem Handler, alles danach **nach** der Antwort. Wie ein Kontextmanager.

**3. `HTTPException`** — die Art, aus der Presentation-Schicht einen Statuscode zu erzwingen:

```python
raise HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="…",                                  # landet als {"detail": "…"} im Body
    headers={"WWW-Authenticate": "ApiKey"},      # zusätzliche Antwort-Header
)
```

**4. `select(...).where(...)` mit `session.scalar(...)`** — erste gefilterte Abfrage im Projekt.

---

### Schritt 3.1 — `hash_key` umziehen: neue Datei `persistence/api_keys.py`

**Warum überhaupt:** `hash_key` steht in `persistence/api_key_admin.py` — das ist die **CLI**. Gleich braucht auch der Lesepfad der API dieselbe Funktion. Wenn zwei Aufrufer dasselbe brauchen, gehört es in das **tiefere** der beiden Module; eine Datenbankabfrage soll nicht aus einem Kommandozeilenwerkzeug importieren.

**→ Was du tust:** Neue Datei `src/pricing/persistence/api_keys.py`. Verschiebe dorthin aus `api_key_admin.py`:

```python
KEY_PREFIX = "pk_"
_KEY_BYTES = 32
def generate_key() -> str: ...
def hash_key(raw_key: str) -> str: ...
```

- [x] `persistence/api_keys.py` angelegt, die vier Dinge dorthin verschoben
- [x] `api_key_admin.py` importiert sie jetzt: `from .api_keys import generate_key, hash_key`
- [x] `uv run pricing-apikey list` läuft weiter, `ruff`/`mypy` sauber

---

### Schritt 3.2 — Der Vertrag: `persistence/interfaces.py`

**→ Was du tust:** Ein zweites `Protocol` neben `DataSource` und `RecommendationWriter`.

```python
class ApiKeyRepository(Protocol):
    """Nachschlagen eines Schluessels. Kennt die Anwendung, nicht die Tabelle."""

    def find_active_name(self, raw_key: str) -> str | None: ...
```

Drei Entscheidungen darin, die begründet sind:

| Entscheidung | Warum |
|---|---|
| Rückgabe `str \| None`, **nicht** `ApiKey` | Ein ORM-Objekt zurückzugeben würde SQLAlchemy in die Application-Schicht tragen. `PricingApp` kennt aus demselben Grund nur `CapacityRow`, nicht `orm.Capacity`. |
| Nimmt den **Rohschlüssel**, nicht den Hash | Dann bleibt das Hashen vollständig in `persistence/`. Die Anwendungsschicht muss nicht wissen, *dass* gehasht wird. |
| „active" steckt im **Namen** der Methode | Der Widerruf ist Teil der Abfrage (`revoked_at IS NULL`), nicht ein `if` danach — sonst könnte ein Aufrufer ihn vergessen. |

- [x] `ApiKeyRepository` als `Protocol` in `interfaces.py`

---

### Schritt 3.3 — Die Implementierung: `persistence/api_keys.py`

**→ Was du tust:** Eine Klasse, die den Vertrag erfüllt. Muster: genau wie `SqlAlchemyDataSource` — **Session kommt von außen.**

```python
class SqlAlchemyApiKeyRepository:
    def __init__(self, session: Session) -> None: ...

    def find_active_name(self, raw_key: str) -> str | None: ...
```

Die neue Syntax für den Rumpf:

```python
stmt = select(ApiKey.name).where(          # nur die EINE Spalte, nicht die ganze Zeile
    ApiKey.key_hash == hash_key(raw_key),
    ApiKey.revoked_at.is_(None),           # mehrere where-Bedingungen = UND
)
return self._session.scalar(stmt)          # erster Wert der ersten Zeile, oder None
```

| Baustein                           | Bedeutung                                                                                                       |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `select(ApiKey.name)`              | wählt eine **Spalte** statt der ganzen Entität — es kommt ein `str` zurück, kein ORM-Objekt                     |
| mehrere Argumente in `.where(...)` | werden mit **UND** verknüpft                                                                                    |
| `.is_(None)`                       | erzeugt `IS NULL`. **Nicht** `== None` schreiben — Ruff meldet das als `E711`, und in SQL ist `= NULL` nie wahr |
| `session.scalar(stmt)`             | erwartet **höchstens eine** Zeile, gibt deren ersten Wert oder `None`                                           |

- [x] `SqlAlchemyApiKeyRepository` mit `__init__(self, session)` — Session von außen, wie bei `SqlAlchemyDataSource`
- [x] `find_active_name` nutzt `select(ApiKey.name)`, zwei `where`-Bedingungen, `session.scalar`
- [x] `.is_(None)` statt `== None`

> [!tip] `scalar` vs. `scalars` vs. `execute`
> `session.scalar(stmt)` → **ein** Wert oder `None`. `session.scalars(stmt).all()` → **Liste** von Werten (so macht es `list_keys`). `session.execute(stmt)` → Zeilen-Objekte mit mehreren Spalten (so macht es `SqlAlchemyDataSource.load_capacity`).

---

### Schritt 3.4 — Die Anwendungsschicht: `application/auth_service.py`

**→ Was du tust:** Eine Funktion, eine Zeile Rumpf.

```python
def authenticate(repository: ApiKeyRepository, raw_key: str) -> str | None:
    """Gibt den Namen des Schluessels zurueck, oder None."""
```

- [x] `application/auth_service.py` mit `authenticate`
- [x] Importiert **nur** `ApiKeyRepository` aus `persistence.interfaces` — **kein** SQLAlchemy, **kein** `orm`

> [!note] Heute ein Einzeiler — und die einzige Application-Funktion im CRUD-Umfeld
> `authenticate` reicht momentan nur weiter. Sie bleibt trotzdem, weil hier die **Regeln** landen, die noch kommen: Gilt ein Schlüssel nach einer Ablaufzeit noch? Wird `last_used_at` fortgeschrieben — und wie oft? Greift eine Ratenbegrenzung?
>
> Genau das ist die Trennlinie aus [[#Die Sinkhole-Frage — entschieden am 2026-08-11|Die Sinkhole-Frage]]: **`application/` enthält, wo Regeln sind.** Bei der Authentifizierung sind sie absehbar, bei CRUD nicht — deshalb gibt es dort keine Anwendungsschicht.

---

### Schritt 3.5 — Die Dependencies: `presentation/api/deps.py`

Der Kern des Blocks. **Fünf Dinge** in dieser Reihenfolge:

```python
# 1) Session pro Request
def get_session() -> Iterator[Session]:
    with Session(get_engine()) as session:
        yield session                     # ← davor: Aufbau · danach: Abbau

SessionDep = Annotated[Session, Depends(get_session)]

# 2) Repository aus der Session
def get_api_key_repository(session: SessionDep) -> ApiKeyRepository:
    ...

RepositoryDep = Annotated[ApiKeyRepository, Depends(get_api_key_repository)]

# 3) Header-Leser (Modulebene, wird einmal gebaut)
_api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

# 4) Die Prüfung
def require_api_key(
    repository: RepositoryDep,
    raw_key: Annotated[str | None, Depends(_api_key_header)],
) -> str:
    ...                                   # None -> HTTPException(401), sonst Namen zurückgeben

# 5) Für Handler, die wissen wollen, wer aufruft (optional)
ApiKeyNameDep = Annotated[str, Depends(require_api_key)]
```

Beachte: **eine Dependency darf selbst Dependencies haben.** `require_api_key` fordert `RepositoryDep`, das fordert `SessionDep`, das ruft `get_session`. FastAPI löst die Kette von unten auf.

- [x] `Iterator` kommt aus `collections.abc`, nicht aus `typing`
- [x] `get_session` mit `yield` und `with Session(get_engine())`
- [x] `get_api_key_repository` gibt den **Protocol**-Typ zurück, konstruiert aber die konkrete Klasse
- [x] `_api_key_header` mit `auto_error=False` auf **Modulebene**
- [x] `require_api_key` wirft `HTTPException(401, …, headers={"WWW-Authenticate": "ApiKey"})` bei fehlendem *oder* ungültigem Schlüssel
- [x] Fehlender Header und falscher Schlüssel führen zur **gleichen** Antwort

> [!warning] `auto_error=False` ist Pflicht
> Ohne diesen Schalter antwortet FastAPIs `APIKeyHeader` bei fehlendem Header selbst — und zwar mit **`403`**. Das ist semantisch falsch: `401` heißt „ich weiß nicht, wer du bist", `403` heißt „ich weiß es, du darfst aber nicht". Hier gibt es keine Rollen, also ist es immer `401`. Mit `auto_error=False` bekommst du `None` statt einer Antwort und entscheidest selbst.

> [!important] `deps.py` ist die Composition Root der API
> Hier steht ein Widerspruch, den du sonst zu Recht bemerken würdest: Die Schichtregel sagt *„Presentation kennt weder `persistence/` noch SQLAlchemy"* — aber `deps.py` importiert `Session`, `get_engine` **und** die konkrete Repository-Klasse.
>
> Das ist dieselbe Ausnahme wie bei `main.py`: **`deps.py` verdrahtet, es rechnet nicht.** Es ist die eine Datei in `presentation/`, die alle Schichten kennen darf — damit alle **anderen** Dateien dort (Router, Schemas) frei davon bleiben. Kein Handler in Block 4–15 wird SQLAlchemy importieren.

> [!note] Warum das Repository über eine *eigene* Dependency kommt
> `require_api_key` könnte die Klasse auch direkt bauen. Die eigene Funktion zahlt sich in Block 17 aus: `app.dependency_overrides[get_api_key_repository] = lambda: FakeRepo()` ersetzt die Datenbank im Test durch ein Fake — **ohne laufenden Postgres und ohne Monkeypatching.** Was man ersetzen will, braucht einen Namen.

> [!note] Hier darf `HTTPException` geworfen werden
> In Block 5 kommt die Regel „kein `raise HTTPException` in der Service-Schicht". Eine **Dependency ist Presentation-Schicht** — sie darf HTTP sprechen, das ist ihr Job. Deshalb braucht Block 3 keinen zentralen Exception-Handler: `authenticate()` gibt `None` zurück, die Dependency macht daraus ein `401`.

---

### Schritt 3.6 — Einhängen: `presentation/api/app.py`

```python
app.include_router(health.router, prefix="/v1")                                   # offen
app.include_router(products.router, prefix="/v1",
                   dependencies=[Depends(require_api_key)])                       # geschützt
```

- [x] `health` **ohne** `dependencies=`
- [x] Jeder andere Router **mit** `dependencies=[Depends(require_api_key)]`
- [x] `/openapi.json` zeigt `components.securitySchemes.APIKeyHeader` und `security` an den geschützten Pfaden — bei `/v1/health` steht dort **nichts**

> [!tip] Dependencies werden pro Request zwischengespeichert
> Wenn ein Router `dependencies=[Depends(require_api_key)]` hat *und* ein Handler darin zusätzlich `ApiKeyNameDep` benutzt, läuft die Prüfung trotzdem nur **einmal** — FastAPI merkt sich das Ergebnis je Anfrage. Du darfst beides kombinieren, ohne doppelt zu hashen.

---

### Schritt 3.7 — Ausprobieren

- [x] `uv run uvicorn pricing.presentation.api.app:app --reload` startet
- [x] `/v1/health` ohne Schlüssel → `200`
- [x] Ein geschützter Pfad ohne Schlüssel → `401` **und** Header `WWW-Authenticate: ApiKey`
- [x] Mit falschem Schlüssel → `401`
- [x] Mit dem Schlüssel aus der `.env` → kommt durch (noch `404`, solange kein Router existiert)
- [x] Mit einem per `pricing-apikey revoke` **widerrufenen** Schlüssel → `401`
- [x] `/docs`: **Authorize**-Knopf vorhanden, Schlüssel eintragen, Endpunkt ausprobieren funktioniert
- [x] Der Schlüssel steht in `.env` als `API_KEY=pk_…` (nur für dein `curl`, die Anwendung liest ihn nicht)

> [!note] Warum die Authentifizierung **hier** steht und nicht am Ende
> Ab jetzt schreibst du jeden Router und jeden Test mit ihr. Käme sie nach Block 17, müsstest du 26 geschützte Endpunkte und rund 30 Tests nachrüsten — jeder `TestClient`-Aufruf bräuchte plötzlich einen Header. Die Reihenfolge ist an dieser einen Stelle im Plan wirklich zwingend.

---

# Block 4 — Erster Lesepfad: `GET /products`

**Neue Technik:** Pydantic-Modell als DTO, `response_model` über die Rückgabe-Annotation, `from_attributes`, ein Repository für eine Entität.

**Der teuerste Block im Plan** — nicht weil viel Code entsteht, sondern weil er das **Muster** für alle folgenden CRUD-Endpunkte festlegt. Ab Block 6 ist es Wiederholung.

## Was am Ende funktionieren muss

```bash
curl -s -H "X-API-Key: $API_KEY" localhost:8000/v1/products | head -c 200
# [{"product_id":"…","bedroom_type":"…"}, …]

curl -i localhost:8000/v1/products      # 401 — ohne Schlüssel
```

## Der Weg einer Anfrage

```
GET /v1/products
  └─ require_api_key                                         [presentation]
  └─ Handler in routers/products.py                          [presentation]
       └─ repository.list_all()                              [persistence]
            └─ SELECT … FROM products
       └─ list[ProductRead]  ← Pydantic wandelt ORM → JSON    [presentation]
```

> [!important] Die Architekturentscheidung, die dieser Block festlegt
> Für CRUD geben die Repositories **ORM-Objekte** zurück — nicht Domänen-Objekte. Grund: Es gibt für CRUD keine Domänen-Objekte, die passen. Vergleiche:
>
> ```
> orm.Booking     15 Spalten: id, source_id, bkg_id, product_id, cust_id, start_date, …
> domain.Booking   4 Felder:  product_id, start_date, nights, status
> ```
>
> Die Domänen-`Booking` ist ein **Ausschnitt für die Preisrechnung**, kein Abbild der Tabelle. CRUD verwaltet aber die ganze Zeile. Also:
>
> **ORM-Objekte dürfen bis in die Application-Schicht, aber nicht in die Presentation-Schicht.**
>
> Das ist mit der Schichtordnung vereinbar: `application → persistence` zeigt **nach unten**, und die dokumentierte Regel verbietet nur der *Presentation*-Schicht, `persistence` zu kennen. Die Umwandlung passiert genau an dieser Grenze — im Router, durch das Pydantic-Modell. Kein Router importiert `orm` oder `sqlalchemy`.
>
> **Die Asymmetrie ist die eigentliche Lektion:** Domänen-Objekte gibt es dort, wo es Regeln gibt. Wo keine sind, verschiebt man Zeilen.

## Die neue Syntax, gesammelt

**1. Das Pydantic-Modell filtert.** Es ist nicht nur eine Beschreibung, es ist ein **Sieb**: Was nicht im Modell steht, kommt nicht in die Antwort — auch wenn das ORM-Objekt es hat.

```python
class ProductRead(BaseModel):
    model_config = ConfigDict(from_attributes=True)   # liest Attribute statt dict-Keys
    product_id: str
    bedroom_type: str | None
```

**2. `response_model` kommt aus der Rückgabe-Annotation.** Du schreibst kein `response_model=` in den Dekorator — FastAPI liest den Rückgabetyp:

```python
@router.get("/products")
def list_products(...) -> list[ProductRead]: ...
```

**3. Die Umwandlung schreibst du trotzdem selbst hin:**

```python
return [ProductRead.model_validate(p) for p in repository.list_all()]
```

FastAPI *könnte* die ORM-Objekte auch selbst umwandeln — aber dann meldet `mypy --strict` zu Recht:

```
error: Incompatible return value type (got "Sequence[Product]", expected "list[ProductRead]")
```

Und es hätte recht: Die Funktion gäbe etwas anderes zurück, als sie behauptet. `model_validate` macht die Schichtgrenze im Code sichtbar, statt sie einem Framework zu überlassen — **genau hier wird aus einem ORM-Objekt ein DTO.**

---

### Schritt 4.1 — Der Vertrag: `persistence/interfaces.py`

```python
class ProductRepository(Protocol):
    def list_all(self) -> Sequence[orm.Product]: ...
```

- [x] `ProductRepository` als `Protocol`
- [x] `interfaces.py` importiert dafür `orm` — das ist innerhalb derselben Schicht, also unkritisch

> [!note] Warum `Sequence` und nicht `list`
> `Sequence` sagt „ich lese nur, ich verändere nicht". `session.scalars(...).all()` gibt ohnehin eine unveränderliche Sequenz zurück. `list` würde dem Aufrufer versprechen, dass er anhängen darf.

---

### Schritt 4.2 — Die Implementierung: `persistence/repositories.py` *(neu)*

Neue Datei. Hier landen ab jetzt alle CRUD-Repositories (`bookings`, `prices`, `capacity` folgen).

```python
class SqlAlchemyProductRepository:
    def __init__(self, session: Session) -> None: ...
    def list_all(self) -> Sequence[orm.Product]: ...
```

Rumpf: `select(orm.Product).order_by(orm.Product.product_id)`, dann `self._session.scalars(stmt).all()`.

- [x] Session von außen, wie bei `SqlAlchemyDataSource` und `SqlAlchemyApiKeyRepository`
- [x] `select(orm.Product)` — die **ganze Entität**, nicht einzelne Spalten (anders als bei `find_active_name`)
- [x] `order_by` gesetzt, damit die Reihenfolge nicht dem Zufall überlassen ist
- [x] `SELECT count(*) FROM products` einmal ausführen und die Zahl notieren

> [!tip] Warum hier noch keine Paginierung
> `products` ist eine kleine Stammdatentabelle. Wenn `count(*)` über etwa 1.000 liegt, gehört auch hier ein Limit hin — dann zieh es aus Block 6 vor. Bei `bookings` (940.000 Zeilen) ist es Pflicht.

---

### Schritt 4.3 — Das DTO: `presentation/api/schemas/product.py` *(neu)*

```python
class ProductRead(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    product_id: str
    bedroom_type: str | None
```

- [x] `ProductRead` mit genau den zwei Spalten
- [x] `from_attributes=True` gesetzt

> [!warning] Ohne `from_attributes=True` scheitert es
> Pydantic erwartet standardmäßig ein `dict`. Bei einem ORM-Objekt bekommst du dann einen Validierungsfehler. Der Schalter hieß in Pydantic v1 `orm_mode` — in älteren Tutorials steht noch der alte Name.

> [!important] Das Modell ist gleichzeitig ein Sicherheitsnetz
> Getestet: Ein Objekt mit einem zusätzlichen Feld `geheim` wird durch `ProductRead` geschickt — **das Feld erscheint nicht in der Antwort.** Wenn du morgen eine Spalte `internal_note` an die Tabelle hängst, landet sie nicht versehentlich in der API. Genau deshalb ist ein eigenes DTO besser als „einfach das ORM-Objekt zurückgeben".

---

### Schritt 4.4 — Die Dependency: `presentation/api/deps.py`

```python
def get_product_repository(session: SessionDep) -> ProductRepository: ...

ProductRepositoryDep = Annotated[ProductRepository, Depends(get_product_repository)]
```

- [x] Nach dem Muster von `get_api_key_repository` — `SessionDep` als Parameter, **nicht** `Session`
- [x] Alias direkt darunter

---

### Schritt 4.5 — Der Router: `presentation/api/routers/products.py` *(neu)*

```python
router = APIRouter(prefix="/products", tags=["products"])

@router.get("")
def list_products(repository: ProductRepositoryDep) -> list[ProductRead]: ...
```

Rumpf: `repository.list_all()` aufrufen und jedes Objekt per `ProductRead.model_validate(...)` umwandeln (siehe *Die neue Syntax*, Punkt 3).

- [x] `prefix="/products"` am **Router**, nicht am Endpunkt — dann steht der Pfad einmal da
- [x] `tags=["products"]` für die Gruppierung in `/docs`
- [x] Der Router importiert **kein** `sqlalchemy` und **kein** `orm`
- [x] Die Umwandlung per `model_validate` steht im Rumpf — `mypy --strict` erzwingt sie

> [!tip] `@router.get("")` statt `@router.get("/")`
> Mit `prefix="/products"` und Pfad `""` entsteht `/v1/products`. Mit `"/"` entstünde `/v1/products/`, und FastAPI würde von `/v1/products` dorthin **weiterleiten** — ein zusätzlicher Roundtrip und eine Quelle für den `307`-vs-`POST`-Fallstrick aus dem Q&A.

---

### Schritt 4.6 — Einhängen und aufräumen: `presentation/api/app.py`

- [x] `products.router` mit `prefix="/v1"` und `dependencies=[Depends(require_api_key)]`
- [x] **`routers/test.py` löschen** und aus `app.py` entfernen
- [x] `/docs` zeigt eine Gruppe „products" mit dem Antwortschema

> [!warning] Warum `test.py` weg muss
> pytest sammelt Funktionen namens `test*` in Dateien namens `test*`. Aktuell greift das nicht, weil `testpaths = ["tests"]` die Sammlung begrenzt — aber sobald jemand `pytest src/` aufruft, wird dein Endpunkt als Testfall ausgeführt.

---

### Schritt 4.7 — Ausprobieren

- [x] `GET /v1/products` mit Schlüssel → `200` und eine JSON-Liste
- [x] Ohne Schlüssel → `401`
- [x] `/docs` → Authorize, dann „Try it out" am Endpunkt
- [x] Ein Blick in `/openapi.json`: Das Schema `ProductRead` steht unter `components.schemas`

---

# Block 5 — Element-Endpunkt, 404 und der zentrale Exception-Handler

**Neue Technik:** Pfadparameter, `@app.exception_handler`, Übersetzung Exception → Statuscode an einer Stelle.

## Was am Ende funktionieren muss

```bash
curl -s  -H "X-API-Key: $API_KEY" localhost:8000/v1/products/<echte-id>   # 200
curl -si -H "X-API-Key: $API_KEY" localhost:8000/v1/products/GIBTSNICHT   # 404
# {"detail":"Produkt GIBTSNICHT nicht gefunden"}
```

## Das Prinzip

**Kein `try/except` im Handler, und kein `raise HTTPException` in der Service-Schicht.**

Die Application-Schicht wirft `NotFoundError` — sie weiß nichts von HTTP. Die Presentation-Schicht übersetzt **einmal zentral**. Ohne das steht in jedem der 26 Handler dieselbe Fehlerbehandlung.

```python
# presentation/api/errors.py
@app.exception_handler(NotFoundError)
def handle_not_found(request: Request, exc: NotFoundError) -> JSONResponse:
    return JSONResponse(status_code=404, content={"detail": str(exc)})
```

## Die Zuordnungstabelle — das Zielbild

| Exception | Status | Begründung |
|---|---|---|
| `NotFoundError` | `404` | Ressource existiert nicht |
| `ConflictError` | `409` | Unique- oder FK-Verletzung, siehe Block 11 |
| `DomainError` | `422` | Eingabe syntaktisch ok, fachlich unmöglich |
| `DataSourceError` | `503` + `Retry-After` | nachgelagertes System weg, Retry sinnvoll |
| `ConfigError` | `500` | Betriebsfehler, nach außen generisch |
| alles Übrige | `500` | generische Meldung außen, **voller Traceback im Log** |

> [!note] `401` fehlt in dieser Liste — mit Absicht
> Das macht seit Block 3 die Security-Dependency selbst, mit `HTTPException`. Eine Dependency *ist* Presentation-Schicht und darf HTTP sprechen; ein Service darf es nicht. Genau daran lässt sich die Schichtgrenze festmachen.

---

### Schritt 5.1 — `NotFoundError` — schon da

`shared/exceptions.py` hat sie seit Block 2 (für `pricing-apikey revoke <unbekannte-id>`).

- [x] `NotFoundError(was, kennung)` existiert und baut ihre Meldung selbst (hält `TRY003` ruhig)

---

### Schritt 5.2 — Die Handler: `presentation/api/errors.py` *(neu)*

**→ Was du tust:** Eine Funktion, die die Handler an eine `FastAPI`-Instanz hängt — damit `app.py` schlank bleibt.

**Die Importe:**

```python
import logging
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from ...shared.exceptions import NotFoundError      # drei Punkte: api → presentation → pricing
```

**Die Form:** Der Dekorator braucht eine `app`-Instanz — also stehen die Handler **innerhalb** der Registrierungsfunktion:

```python
def register_error_handlers(app: FastAPI) -> None:
    @app.exception_handler(NotFoundError)
    def handle_not_found(request: Request, exc: NotFoundError) -> JSONResponse:
        return JSONResponse(status_code=404, content={"detail": str(exc)})

    @app.exception_handler(Exception)
    def handle_unexpected(request: Request, exc: Exception) -> JSONResponse:
        logger.error("Unerwarteter Fehler", exc_info=exc)     # NICHT .exception()!
        return JSONResponse(status_code=500, content={"detail": "Interner Fehler"})
```

`request` wird nicht gebraucht, muss aber in der Signatur stehen — Starlette ruft jeden Handler mit beiden Argumenten auf.

> [!warning] `app.add_exception_handler(...)` statt des Dekorators geht **nicht** durch `mypy --strict`
> Die naheliegende Alternative — Handler auf Modulebene, Registrierung per `app.add_exception_handler(NotFoundError, handle_not_found)` — scheitert:
> ```
> error: Argument 2 to "add_exception_handler" has incompatible type
>   "Callable[[Request, NotFoundError], JSONResponse]";
>   expected "Callable[[Request, Exception], Response | Awaitable[Response]]"
> ```
> Starlette verlangt einen Handler für **jede** `Exception`; deiner nimmt nur `NotFoundError`. Der Dekorator umgeht die Prüfung, weil er den Typ aus dem Argument ableitet. Geprüft: Dekorator-Variante grün, `add_exception_handler`-Variante rot.

> [!bug] `logger.exception()` funktioniert hier **nicht** — nachgemessen
> Der reflexhafte Griff wäre `logger.exception(...)` wie in `main.py`. In einem FastAPI-Exception-Handler ist das falsch, und zwar nicht formal, sondern wirkungslos:
> ```
> Thread: AnyIO worker thread   sys.exc_info(): None
> A: logger.exception(...)        -> angehängter Traceback: KEINE
> B: logger.error(..., exc_info=exc) -> angehängter Traceback: ValueError
> ```
> `logger.exception()` liest die **aktuelle** Ausnahme aus `sys.exc_info()`. Ein synchroner Handler läuft aber in einem AnyIO-Worker-Thread, und dort ist keine Ausnahme aktiv — im Log stünde `NoneType: None` statt des Stacks.
>
> Du **hast** die Ausnahme ja als Parameter: `logger.error("…", exc_info=exc)` hängt sie explizit an. Ruff meldet den Fehler übrigens auch (`LOG004`).

> [!note] Starlette wirft nach dem `Exception`-Handler erneut
> Nachgemessen: Der Handler erzeugt die `500`-Antwort, danach reicht `ServerErrorMiddleware` die Ausnahme weiter, damit der ASGI-Server sie ebenfalls protokollieren kann. Der Client bekommt trotzdem sauber sein `500`. Das ist auch der Grund für den `TestClient`-Hinweis weiter unten.

**Nur zwei Handler jetzt.** Die übrigen Zeilen der Tabelle baust du, wenn der Fall zum ersten Mal auftreten kann:

- [x] `NotFoundError` → `404` mit `{"detail": str(exc)}`
- [x] `Exception` (Auffangnetz) → `500` mit **generischem** Text, plus `logger.exception(...)`
- [x] `register_error_handlers(app)` wird in `app.py` aufgerufen
- [x] `ConflictError`, `DomainError`, `DataSourceError` kommen später — nicht auf Vorrat bauen

> [!warning] Nach außen wenig, ins Log alles
> Ein Traceback in der HTTP-Antwort verrät Pfade, Bibliotheksversionen und manchmal Zugangsdaten. Nach außen `{"detail": "Interner Fehler"}`, ins Log der vollständige Stack per `logger.exception` — wie in [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]].

> [!tip] Der `Exception`-Handler und Tests
> Ein Handler für `Exception` fängt in der laufenden Anwendung alles ab. Im `TestClient` **wirft** der Fehler standardmäßig trotzdem durch, damit du im Test den echten Traceback siehst. Wer das Verhalten der Produktion testen will, braucht `TestClient(app, raise_server_exceptions=False)`. Merken für Block 17.

---

### Schritt 5.3 — Nachschlagen: Repository, Service, Router

Drei kleine Ergänzungen entlang derselben Kette wie in Block 4:

```python
# interfaces.py
def get(self, product_id: str) -> orm.Product | None: ...

# repositories.py — session.get(orm.Product, product_id)

# routers/products.py
@router.get("/{product_id}")
def get_product(product_id: str, repository: ProductRepositoryDep) -> ProductRead:
    produkt = repository.get(product_id)
    if produkt is None:
        raise NotFoundError("Produkt", product_id)
    return produkt
```

- [x] Das **Repository** gibt `None` zurück (es kennt keine Fachlichkeit)
- [x] Der **Router** macht aus `None` die `NotFoundError` — der zentrale Handler übersetzt sie
- [x] `session.get(orm.Product, product_id)` — Nachschlagen über den Primärschlüssel, wie in Block 2 gelernt

> [!note] Warum das Repository nicht selbst wirft
> „Nicht gefunden" ist für ein `GET /products/{id}` ein Fehler, für eine Suchfunktion ein normales Ergebnis. Das Repository weiß nicht, welcher Fall gilt — es liefert Daten oder `None`. Erst der Aufrufer weiß, ob das schlimm ist. **Ein Repository, das wirft, nimmt dem Aufrufer diese Entscheidung weg.**

> [!note] Warum `NotFoundError` und nicht direkt `HTTPException(404)`
> Der Router *dürfte* `HTTPException` werfen — er ist Presentation-Schicht. Aber dann stünde in 26 Handlern derselbe Statuscode und dieselbe Body-Form. Über `NotFoundError` steht die Übersetzung **einmal** in `errors.py`. Und ab Block 11 kommt `ConflictError` aus dem Repository, wo `HTTPException` verboten wäre — dann brauchst du den Handler ohnehin.

---

### Schritt 5.4 — Ausprobieren

- [x] Echte ID → `200`
- [x] Unbekannte ID → `404` mit lesbarem `detail`
- [x] Kein `try/except` in `routers/products.py`
- [x] Ein absichtlich provozierter Fehler (z. B. DB-Container aus) → `500` mit generischer Meldung nach außen und vollem Traceback in `logs/pricing.log`

---

# Block 6 — `GET /bookings`: Filter, Paginierung, stabile Sortierung

**Neue Technik:** `Query`-Parameter mit `Annotated`, Wertgrenzen, `LIMIT`/`OFFSET`, `ORDER BY` über einen eindeutigen Schlüssel.

Ab hier ist das Muster aus Block 4/5 bekannt — neu ist nur, was oben steht.

## Was am Ende funktionieren muss

```bash
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/bookings?limit=5"
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/bookings?product_id=P1&from=2026-06-01&to=2026-06-30"
curl -i -H "X-API-Key: $API_KEY" "localhost:8000/v1/bookings?limit=9999"   # 422
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/bookings?product_id=GIBTSNICHT"   # 200 []
```

## Die neue Syntax

```python
@router.get("")
def list_bookings(
    repository: BookingRepositoryDep,
    product_id: Annotated[str | None, Query()] = None,
    von: Annotated[date | None, Query(alias="from")] = None,      # 'from' ist Python-Schlüsselwort!
    bis: Annotated[date | None, Query(alias="to")] = None,
    limit: Annotated[int, Query(ge=1, le=500)] = 50,
    offset: Annotated[int, Query(ge=0)] = 0,
) -> list[BookingRead]: ...
```

| Baustein | Wirkung |
|---|---|
| `Query(ge=1, le=500)` | Grenzen werden **erzwungen** — `limit=9999` → `422`, bevor dein Code läuft |
| Default-Wert in der Signatur | macht den Parameter optional |
| `alias="from"` | der URL-Parameter heißt `from`, die Python-Variable `von`. **Nötig**, weil `from` ein Schlüsselwort ist |
| `date` als Typ | FastAPI parst `2026-06-01` automatisch zu `datetime.date`; unparsbar → `422` |

- [x] Default-Limit gesetzt — **nie** unbegrenzt, die Tabelle hat 940.000 Zeilen
- [x] `limit` nach oben gedeckelt (`le=500`)
- [x] `alias="from"`/`alias="to"` benutzt
- [x] `ORDER BY` über einen **eindeutigen** Schlüssel (`id`), notfalls als letztes Kriterium
- [x] Unbekannte `product_id` → `200` mit leerer Liste, **nicht** `404`
- [x] Die Filter werden in der **Abfrage** angewandt, nicht in Python nach dem Laden
- [x] `BookingRead` in `schemas/booking.py` — entscheide bewusst, welche der 15 Spalten nach außen gehen

> [!warning] Ohne stabile Sortierung ist Paginierung kaputt
> `LIMIT/OFFSET` ohne `ORDER BY` gibt Postgres **keine** Reihenfolgegarantie. Dieselbe Zeile kann auf Seite 1 *und* Seite 2 auftauchen, andere fallen ganz durch. Immer nach etwas Eindeutigem sortieren.

> [!warning] Filtern gehört in die Datenbank
> Die Versuchung ist, `list_all()` zu laden und in Python zu filtern. Bei 940.000 Zeilen holst du damit 100 MB über die Leitung, um 5 Zeilen zu behalten. Die `where`-Bedingungen werden **bedingt** an das `select` gehängt — nur die, die der Client geschickt hat.

> [!note] Leere Liste ist kein `404`
> Die *Sammlung* `/bookings` existiert, sie ist nur gerade leer. `404` hieße „diesen Endpunkt gibt es nicht". Eine der Kontrollfragen aus dem Leitfaden.

> [!tip] `cust_id` und `feature_2` gehören nicht in `BookingRead`
> Die Bewerberaufgabe nennt sie ausdrücklich irrelevant. Das DTO ist die Stelle, an der du das durchsetzt — und Block 4 hat gezeigt, dass Pydantic zusätzliche Felder tatsächlich herausfiltert.

---

# Block 7 — `POST /bookings`: erzeugen, `201`, `Location`

**Neue Technik:** Create-DTO, `status_code=201`, `Response`-Objekt für Header.

## Was am Ende funktionieren muss

```bash
curl -si -X POST -H "X-API-Key: $API_KEY" -H "Content-Type: application/json" \
  -d '{"bkg_id":"B1","product_id":"P1","start_date":"2026-07-01","nights":3,"status":"Firm"}' \
  localhost:8000/v1/bookings
# 201 Created · Location: /v1/bookings/12345 · Body = die angelegte Buchung

curl -si -X POST … -d '{"nights":"drei"}' …    # 422, ohne dass dein Code läuft
```

## Die neue Syntax

```python
@router.post("", status_code=status.HTTP_201_CREATED)
def create_booking(
    payload: BookingCreate,                 # ← Body, weil es ein BaseModel ist
    response: Response,                     # ← FastAPI reicht das Antwortobjekt herein
    repository: BookingRepositoryDep,
) -> BookingRead:
    ...
    response.headers["Location"] = f"/v1/bookings/{neu.id}"
    return neu
```

| Baustein                       | Wirkung                                                         |
| ------------------------------ | --------------------------------------------------------------- |
| Parameter mit `BaseModel`-Typ  | FastAPI liest ihn aus dem **Request-Body**, nicht aus der Query |
| `status_code=201` im Dekorator | Erfolgsfall-Status; Fehler kommen weiter über Exceptions        |
| `response: Response`           | Zugriff auf die Antwort-Header, ohne die Rückgabe zu verlieren  |

- [x] `BookingCreate` in `schemas/booking.py` — **ohne** `id` und ohne berechnete Spalten (`net_revenue` ist `Computed`)
- [x] `201` als Erfolgsstatus
- [x] `Location`-Header zeigt auf die Element-URL der neuen Ressource
- [x] Rückgabe ist die angelegte Buchung als `BookingRead`
- [x] Der Service committet **nicht** selbst — siehe Kasten
- [x] Ungültiger Body → `422` (macht FastAPI ohne dein Zutun)

> [!important] Wer committet?
> Bisher galt: Der Aufrufer besitzt die Transaktionsgrenze. In der API ist der Aufrufer der **Handler** — er ist das Ende der Kette und weiß, wann die Arbeit fertig ist. Also `session.commit()` im Handler oder in `get_session` nach dem `yield`, aber **nicht** im Repository und **nicht** im Service.
>
> Entscheide es jetzt und halte es in allen Schreib-Endpunkten durch. *(Variante `get_session` committet am Ende: elegant, aber sie committet auch, wenn der Handler nur gelesen hat — harmlos, aber unklarer. Ich würde im Handler committen.)*

> [!warning] Ein eigenes Create-Modell ist keine Förmelei
> Würdest du `BookingRead` für den Body wiederverwenden, könnte ein Client eine `id` mitschicken — und du müsstest sie von Hand ignorieren. **Ein Modell pro Richtung**, nicht eines für alles. Das ist die Antwort auf die Kontrollfrage „warum definieren wir unsere Objekte jetzt doppelt".

> [!tip] `422` bekommst du geschenkt
> Fehlt ein Feld oder hat es den falschen Typ, lehnt Pydantic ab, **bevor** dein Handler läuft — mit Feldnamen und Grund im Body. Genau das ist gemeint mit „Validierung ist eine Schichtgrenze, kein Feature".

---

# Block 8 — `PUT /bookings/{id}`: vollständig ersetzen

**Neue Technik:** Update-DTO, Semantikunterschied `PUT` vs. `PATCH`, Idempotenz.

## Was am Ende funktionieren muss

```bash
curl -si -X PUT -H "X-API-Key: $API_KEY" -H "Content-Type: application/json" \
  -d '{"bkg_id":"B1","product_id":"P1","start_date":"2026-07-02","nights":4,"status":"Firm"}' \
  localhost:8000/v1/bookings/12345          # 200, danach zweimal identisch

curl -si -X PUT … localhost:8000/v1/bookings/999999                     # 404
```

- [x] `BookingUpdate` in `schemas/booking.py` — dieselben Felder wie `BookingCreate`, ohne `id`
- [x] `PUT` **ersetzt alle** Felder: ein weggelassenes Feld wird geleert, nicht beibehalten
- [x] Unbekannte ID → `404` (dieselbe `NotFoundError` wie in Block 5)
- [x] Zweimal dasselbe `PUT` → identisches Ergebnis
- [x] In der Endpunkt-Beschreibung steht, dass es **kein** Teil-Update ist

> [!warning] Die häufigste Verwechslung bei `PUT`
> Viele Implementierungen setzen nur die mitgeschickten Felder — das ist `PATCH`-Verhalten unter `PUT`-Namen. Folge: Ein Client, der ein Feld absichtlich leeren will, kann es nicht. Entweder `PUT` ehrlich als Ersetzen umsetzen (so hier) oder es `PATCH` nennen.

> [!note] Warum `PUT` idempotent ist, `POST` aber nicht
> Zweimal dasselbe `PUT` führt zum **gleichen Zustand** — die Ressource sieht danach so aus wie im Body, egal wie oft du es schickst. Zweimal `POST` erzeugt **zwei** Buchungen. Deshalb darf ein Client ein `PUT` nach einem Timeout gefahrlos wiederholen, ein `POST` nicht (dafür gibt es `Idempotency-Key`, siehe Q&A).

---

# Block 9 — `DELETE /bookings/{id}`: `204` und Idempotenz

**Neue Technik:** `204 No Content` — eine Antwort ohne Body.

## Was am Ende funktionieren muss

```bash
curl -si -X DELETE -H "X-API-Key: $API_KEY" localhost:8000/v1/bookings/12345   # 204, leerer Body
curl -si -X DELETE -H "X-API-Key: $API_KEY" localhost:8000/v1/bookings/12345   # 404
```

## Die neue Syntax

```python
@router.delete("/{booking_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_booking(booking_id: int, repository: BookingRepositoryDep) -> None: ...
```

- [x] Erfolg → `204`, **kein Body** (verifiziert: der Body ist wirklich leer)
- [x] Rückgabetyp `None` — kein DTO, es gibt nichts zurückzugeben
- [x] Unbekannte ID → `404`
- [x] Zweiter Aufruf → wieder `404`

> [!note] `404` beim zweiten Aufruf, und trotzdem idempotent
> Idempotenz bezieht sich auf den **Zustand**, nicht auf die Antwort. Nach dem ersten `DELETE` ist die Buchung weg; nach dem zweiten ist sie *auch* weg. Der Zustand hat sich beim zweiten Mal nicht geändert — das ist die Definition. Dass der Statuscode ein anderer ist, widerspricht dem nicht.

> [!tip] Warum nicht `200` mit `{"deleted": true}`
> Weil es nichts zu repräsentieren gibt. `204` sagt „erledigt, und ich habe dir nichts zu zeigen" — der Client muss keinen Body parsen. Ein Body mit `{"success": true}` ist die Sorte Höflichkeit, die Clients dazu verleitet, auf Statuscodes zu verzichten.

---

> [!info] Block 10 bewusst übersprungen (11.08.)
> Nicks Entscheidung: „Ich überspringe Block 10 erstmal, weil das nicht so relevant ist." Die Vorgabe des Leitfadens (je zweimal CRUD) ist mit `bookings` und `capacity` erfüllt; ein dritter wortgleicher Satz hätte nichts Neues gezeigt. `prices` hat deshalb bis heute keinen Router.

# Block 10 — `prices` analog

**Neue Technik:** keine — bewusst mechanische Wiederholung. **Dieser Block ist absichtlich kurz.**

Fünf Endpunkte für `prices` nach dem Muster aus Block 4–9. Der Zweck ist nicht, etwas Neues zu lernen, sondern zu **sehen, was sich wiederholt**.

- [~] ⏭️ **bewusst übersprungen** — `PriceRepository` (Protocol + Implementierung), `schemas/price.py`, `routers/prices.py`
- [~] ⏭️ **bewusst übersprungen** — `GET`, `GET /{id}`, `POST`, `PUT /{id}`, `DELETE /{id}`
- [~] ⏭️ **bewusst übersprungen** — **Notiere beim Schreiben, welche Stellen wortgleich sind** — daraus entscheidet sich später, ob `repositories.py` in Module zerfällt

> [!tip] Wiederholung ist hier Absicht
> Erst nach der zweiten Ressource sieht man, was wirklich gemeinsam ist. Wer nach der ersten abstrahiert, baut die falsche Abstraktion. **Wenn dir das zu stumpf wird, sag Bescheid — mechanische Wiederholung darf ich übernehmen.**

> [!warning] Eine Sache ist bei `prices` doch neu: der negative Preis
> Die Tabelle hat `CheckConstraint("current_price >= 0")`. Ein `POST` mit `-5` würde also in der Datenbank scheitern — mit einer `CheckViolation`, die ohne Behandlung als **`500`** herauskommt. Falsch, denn der Fehler liegt beim Client.
>
> **Die richtige Stelle ist das DTO:** `current_price: Decimal = Field(ge=0)` in `PriceCreate`. Dann lehnt Pydantic ab, bevor überhaupt SQL entsteht — mit `422` und einer Meldung, die das Feld nennt.
>
> Und der Constraint bleibt trotzdem in der Datenbank. Das ist kein Widerspruch, sondern **Tiefenverteidigung**: Das DTO schützt vor Tippfehlern der API-Nutzer, der Constraint vor Fehlern *deines* Codes und vor allem, was an der API vorbei schreibt (`pricing-ingest`). Zwei Prüfungen, zwei Bedrohungsmodelle.

---

# Block 11 — `capacity`: CRUD mit echten Konflikten

**Neue Technik:** `session.flush()`, `IntegrityError` abfangen und in `409`/`422` übersetzen, `raise … from exc`.

**Der interessanteste Block im Plan.** Die Datenbank kennt die Regeln schon — deine Aufgabe ist, ihr „nein" in eine brauchbare HTTP-Antwort zu übersetzen. `capacity` ist die einzige Tabelle, an der das echt wird.

## Was am Ende funktionieren muss

```bash
# doppelt anlegen -> 409
curl -si -X POST … -d '{"product_id":"P1","calendar_date":"2026-06-01","bookable_units":3,"optional_units":0}' …
curl -si -X POST … -d '{…dieselbe Kombination…}' …          # 409 Conflict

# unbekanntes Produkt -> 422
curl -si -X POST … -d '{"product_id":"GIBTSNICHT", …}' …     # 422

# löschen, obwohl Empfehlungen darauf zeigen -> 409
curl -si -X DELETE … localhost:8000/v1/capacity/<id-mit-empfehlungen>   # 409
```

## Die drei Fälle und ihre Ursachen in der Datenbank

| Fall | Verletzt | sqlstate | psycopg-Klasse | Antwort |
|---|---|---|---|---|
| `POST` mit vorhandenem `(product_id, calendar_date)` | `uq_capacity_product_id` | `23505` | `UniqueViolation` | `409` |
| `POST`/`PUT` mit unbekannter `product_id` | `fk_capacity_product_id_products` | `23503` | `ForeignKeyViolation` | `422` |
| `DELETE`, auf das Empfehlungen zeigen | `fk_price_recommendations_capacity_id` | `23503` | `ForeignKeyViolation` | `409` |

*Alle drei sind Unterklassen von `psycopg.errors.IntegrityError` — geprüft.*

## Die neue Syntax

```python
from sqlalchemy.exc import IntegrityError
from psycopg import errors as pg

try:
    self._session.add(zeile)
    self._session.flush()              # ← erzwingt das SQL JETZT
except IntegrityError as exc:
    if isinstance(exc.orig, pg.UniqueViolation):
        raise ConflictError(...) from exc
    if isinstance(exc.orig, pg.ForeignKeyViolation):
        raise DomainError(...) from exc
    raise
```

| Baustein | Wirkung |
|---|---|
| `session.flush()` | schickt das `INSERT` sofort zur Datenbank, **ohne** zu committen |
| `exc.orig` | die ursprüngliche psycopg-Exception unter der SQLAlchemy-Hülle |
| `raise … from exc` | verkettet die Ursachen, damit der Traceback im Log beide zeigt |
| `exc.orig.diag.constraint_name` | der Name des verletzten Constraints, z. B. `uq_capacity_product_id` |

> [!important] Warum `flush()` der Kern dieses Blocks ist
> `session.add()` schickt **nichts** zur Datenbank — es merkt das Objekt nur vor. Der Fehler entstünde erst beim `commit()` im Handler, also **außerhalb** des Repositories, wo du nicht mehr weißt, welche Operation ihn ausgelöst hat.
>
> `flush()` holt den Fehler dorthin, wo du den Kontext hast: „ich habe gerade *diese* Kapazitätszeile eingefügt". Erst dadurch kannst du eine Meldung schreiben, die dem Client hilft.

> [!warning] Nach einem `IntegrityError` ist die Session unbrauchbar
> Postgres bricht die ganze Transaktion ab. Jede weitere Anweisung in derselben Session scheitert mit `PendingRollbackError`, bis jemand `rollback()` gerufen hat. Sorge dafür, dass das passiert — am saubersten im `get_session`-Kontextmanager (`except: session.rollback(); raise`), damit es für alle Endpunkte gilt und nicht in jedem Repository steht.

> [!warning] Nicht vorher prüfen, sondern den Fehler fangen
> Die Versuchung ist ein `SELECT` vor dem `INSERT`. Das ist eine **Race Condition**: Zwischen Prüfung und Einfügen kann ein anderer Request dieselbe Zeile anlegen. Nur der Constraint entscheidet zuverlässig — also einfügen, `IntegrityError` fangen, übersetzen.

---

### Schritt 11.1 — `ConflictError`: `shared/exceptions.py`

```python
class ConflictError(PricingError):
    def __init__(self, constraint: str | None) -> None: ...
```

- [x] Nach dem Muster von `UnknownReferenceError` — Meldung **in** der Klasse bauen (`TRY003`), Constraint-Name mitführen

Sie deckt **zwei** Ursachen ab, die beide `409` sind: „gibt es schon" (Unique) und „wird noch gebraucht" (Fremdschlüssel beim Löschen). Der `constraint`-Name im Text sagt dem Client, welcher Fall es war.

---

### Schritt 11.2 — `_flush()` erweitern: `persistence/repositories.py`

Seit Block 7/8 gibt es die Modulfunktion `_flush(session)`, die `ForeignKeyViolation` → `UnknownReferenceError` (`422`) übersetzt. Sie braucht **einen** neuen Zweig:

```python
if isinstance(exc.orig, pg.UniqueViolation):
    raise ConflictError(exc.orig.diag.constraint_name) from exc
```

Damit ist der doppelte `POST` erledigt — für **alle** Repositories auf einmal. Genau dafür wurde die Funktion herausgezogen.

> [!bug] Aber: Beim `DELETE` bedeutet dieselbe Verletzung das Gegenteil
> Eine `ForeignKeyViolation` hat **zwei** völlig verschiedene Bedeutungen, je nachdem welche Anweisung sie auslöst:
>
> | Anweisung | Bedeutung | Antwort |
> |---|---|---|
> | `INSERT`/`UPDATE` | „**du** zeigst auf etwas, das es nicht gibt" (`fk_capacity_product_id_products`) | `422` |
> | `DELETE` | „**etwas anderes** zeigt noch auf dich" (`fk_price_recommendations_capacity_id`) | `409` |
>
> Ein einziges `_flush()` kann das nicht unterscheiden — es sieht nur `IntegrityError`. Also braucht der Löschpfad seine eigene Übersetzung:
>
> ```python
> def _flush_after_delete(session: Session) -> None:
>     try:
>         session.flush()
>     except IntegrityError as exc:
>         if isinstance(exc.orig, pg.ForeignKeyViolation):
>             raise ConflictError(exc.orig.diag.constraint_name) from exc
>         raise
> ```
>
> Zwei Funktionen mit fast gleichem Rumpf, aber unterschiedlicher **Bedeutung** — das ist ehrliche Duplikation, keine faule. Wer sie über einen Schalter (`beim_loeschen=True`) zusammenlegt, versteckt die Fachlichkeit hinter einem Parameter.

*(Nachgeprüft: `price_recommendations.capacity_id -> capacity.id` hat `ondelete=None`, also **NO ACTION** — Postgres lehnt das Löschen tatsächlich ab. Es gibt kein `CASCADE`, das die Empfehlungen mitlöschen würde.)*

- [x] `_flush` um den `UniqueViolation`-Zweig erweitert
- [x] `_flush_after_delete` als zweite Funktion, `ForeignKeyViolation` → `ConflictError`
- [x] `CapacityRepository` mit `list_all`, `get`, `create`, `update`, `delete` — Muster wie `bookings`
- [x] `delete` benutzt `_flush_after_delete`, alle anderen `_flush`
- [x] `raise … from exc` überall, alles Unbekannte **weiterwerfen**

> [!note] Warum die Übersetzung ins Repository gehört und nicht höher
> Nur hier weiß man, **welche** Anweisung gescheitert ist und welcher Constraint welchen fachlichen Fall bedeutet. Eine Schicht darüber, die `IntegrityError` fängt, müsste `sqlstate`-Codes kennen — dann wüsste sie plötzlich, dass Postgres darunter liegt.

---

### Schritt 11.3 — Die neuen Handler: `presentation/api/errors.py`

- [x] `ConflictError` → `409`
- [x] `DomainError` → `422`
- [x] Beide mit `{"detail": str(exc)}`

Damit sind vier der sechs Zeilen der Zuordnungstabelle aus Block 5 gebaut.

---

### Schritt 11.4 — Endpunkte, Swagger-Warnung, Tests

- [x] Fünf Endpunkte wie bei `bookings` (`GET` Liste + Element, `POST`, `PUT`, `DELETE`)
- [x] In der Endpunkt-Beschreibung: **„Achtung: `pricing-ingest` truncatet diese Tabelle — über die API angelegte Zeilen sind nach dem nächsten Import weg."**
- [x] Alle drei Konfliktfälle als Test (Block 17)

> [!note] Die Swagger-Warnung ist kein Schmuck
> Du hast diese Endpunkte bewusst gebaut, obwohl der Ingest die Tabelle leert. Das ist vertretbar — aber nur, wenn es **dort steht, wo der Nutzer es liest**. Eine Einschränkung, die nur im Umsetzungsstand dokumentiert ist, kennt der API-Nutzer nicht.

---

# Block 12 — `recommendations`: nur lesen, `/max`, Alias

**Neue Technik:** `JOIN` in der Abfrage, Routen-Reihenfolge, `RedirectResponse`.

## Was am Ende funktionieren muss

```bash
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/recommendations?limit=5"
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/recommendations/max"        # Bewerberaufgabe 3
curl -s -H "X-API-Key: $API_KEY" "localhost:8000/v1/recommendations/42"
curl -si -H "X-API-Key: $API_KEY" "localhost:8000/v1/recommendations/by-unit/P1/2026-06-01"   # 307
curl -si -X POST -H "X-API-Key: $API_KEY" localhost:8000/v1/recommendations    # 405
```

## Warum dieser Block einen JOIN braucht

`price_recommendations` speichert **`capacity_id`** — eine interne Zahl. Ein BI-Nutzer will `product_id` und `calendar_date` sehen. Die stehen in `capacity`. Also:

```python
select(
    orm.PriceRecommendation.id,
    orm.Capacity.product_id,
    orm.Capacity.calendar_date,
    orm.PriceRecommendation.recommended_adjustment_pct,
    ...
).join(orm.Capacity, orm.PriceRecommendation.capacity_id == orm.Capacity.id)
```

Das ist der Fall, für den es DTOs überhaupt gibt: **Die Ressource ist nicht die Tabelle.** Die Antwort ist eine fachliche Sicht, die aus zwei Tabellen zusammengesetzt wird.

- [x] `RecommendationRead` zeigt `product_id` + `calendar_date`, **nicht** `capacity_id`
- [x] Das Repository macht den JOIN in der **Abfrage** (nicht zwei Abfragen und Zusammenbauen in Python)

---

### Schritt 12.1 — Liste mit Filtern

- [x] `GET /recommendations` mit `product_id`, `from`/`to`, `limit` (Default!), `offset`
- [x] Stabile Sortierung
- [x] Muster wie Block 6

---

### Schritt 12.2 — `/max` — und der Fallstrick

```python
@router.get("/max")                    # ← MUSS vor /{recommendation_id} stehen
def get_max(...) -> RecommendationRead: ...

@router.get("/{recommendation_id}")
def get_one(recommendation_id: int, ...) -> RecommendationRead: ...
```

- [x] `/max` liefert die Einheit mit der höchsten `recommended_adjustment_pct` (`ORDER BY … DESC LIMIT 1`)
- [x] Leere Tabelle → `404`, nicht `500`
- [x] **`/max` steht im Code vor `/{recommendation_id}`**
- [x] Ein Test sichert genau diese Reihenfolge ab

> [!warning] Der Fehler, den man beim Lesen nicht sieht
> FastAPI prüft Routen in **Deklarationsreihenfolge**. Steht `/{recommendation_id}` zuerst, landet `/max` dort, und der Handler versucht `"max"` als `int` zu parsen → `422`, mit einer Meldung, die nach einem Client-Fehler aussieht. Der Code sieht dabei völlig richtig aus.
>
> Deshalb: **Test schreiben**, nicht auf Sorgfalt vertrauen. Ein Kommentar `# muss vor /{id} stehen` hilft dem nächsten Leser, aber nur der Test verhindert das Zurückrutschen.

---

### Schritt 12.3 — Der Alias mit `307`

```python
@router.get("/by-unit/{product_id}/{calendar_date}")
def by_unit(...) -> RedirectResponse:
    return RedirectResponse(url=f"/v1/recommendations/{id}", status_code=307)
```

- [x] Alias findet die Empfehlung über `(product_id, calendar_date)` und leitet auf die kanonische URL weiter
- [x] Nicht gefunden → `404` (nicht Weiterleitung ins Leere)
- [x] `307`, nicht `302` — siehe Kasten
- [x] **Kein** `POST`, `PUT` oder `DELETE` auf dieser Ressource

> [!note] Warum `307` und nicht `302`
> `302` erlaubt Clients historisch, die Methode auf `GET` zu ändern. Bei einem reinen Lese-Alias fällt das nicht auf — aber es ist die falsche Angewohnheit: `307` garantiert, dass Methode **und** Body erhalten bleiben. Wer sich `307` antrainiert, baut später keinen `POST` kaputt.

> [!note] Warum kein `POST` auf `/recommendations`
> Eine Preisempfehlung wird nicht angelegt, sondern **berechnet**. Ein Client, der einen Empfehlungswert mitbringt, ist fachlicher Unsinn. FastAPI antwortet auf ein `POST` an einen nur-`GET`-Pfad automatisch mit `405 Method Not Allowed` — genau richtig, ohne dass du etwas tun musst.

---

# Block 13 — `bootstrap.py`: ein Objektgraph für CLI und API

**Neue Technik:** `contextlib.contextmanager`, `functools.lru_cache` auf einer Fabrik, Protocol-Rückgabetypen als Prüfstelle für `mypy`. Kein neues Framework — dieser Block räumt auf und behebt dabei einen echten Fehler.

## Das Problem in einem Satz

Die Frage „**welche konkrete Klasse erfüllt eigentlich welchen Vertrag?**" wird derzeit an **drei** Stellen beantwortet:

| Datei | Zeilen | was dort mit `new` gebaut wird |
|---|---|---|
| `src/pricing/main.py` | 31–36 | `PricingApp` mit `CsvDataSource` + `CsvRecommendationWriter` |
| `scripts/smoke_orm.py` | 96–104 | `PricingApp` mit `SqlAlchemyDataSource` + `SqlAlchemyRecommendationWriter` |
| `src/pricing/presentation/api/deps.py` | 26–33, 41–42, 69–96 | `Session` + fünf `SqlAlchemy…Repository` |

Diese drei Stellen sind der **Composition Root** deiner Anwendung: der Ort, an dem Verträge auf Implementierungen treffen. Ein Composition Root soll es geben — aber genau *einen*. Verteilt auf drei Dateien passiert Folgendes: Bekommt `PricingApp` ein fünftes Konstruktor-Argument, musst du es an zwei Stellen nachziehen und wirst die dritte vergessen, weil kein Werkzeug dich erinnert.

`bootstrap.py` sammelt ausschließlich diese Auswahl. **Keine Logik zieht dorthin um** — nur die `new`-Aufrufe.

## Zwei Fehler, die dieser Block nebenbei behebt

**Fehler 1 — jeder HTTP-Request baut eine eigene Engine.** In `deps.py` steht `Session(get_engine())`, und `get_engine()` ruft `create_engine(...)` — jedes Mal neu. Eine Engine ist aber nicht bloß ein Verbindungsobjekt, sondern der **Verbindungspool**. Eine neue Engine pro Request heißt: ein neuer Pool pro Request, also eine neue TCP-Verbindung zu Postgres pro Request, die nie wiederverwendet wird. Gemessen (SQLite, Zähler auf dem `connect`-Event des Pools):

```
frische Engine pro Request -> physische connects: 5
geteilte Engine            -> physische connects: 1
Pool nach 5 Requests: size: 1
```

Fünf Requests, fünf echte Verbindungsaufbauten statt einem. Bei Postgres kostet jeder davon TCP-Handshake, Authentifizierung und einen serverseitigen Prozess. Der Pool, für den SQLAlchemy die ganze Maschinerie mitbringt, läuft aktuell **komplett leer** — er wird angelegt und weggeworfen, bevor er etwas wiederverwenden kann.

**Fehler 2 — die `PricingApp`-Verdrahtung ist zweimal von Hand geschrieben.** `main.py` und `smoke_orm.py` unterscheiden sich in genau zwei Argumenten (Quelle und Senke), teilen aber `PriceableUnitBuilder()` und `RecommendationEngine(HeuristicRecommendationStrategy())` wörtlich.

> [!note] Warum `bootstrap.py` an die Paketwurzel gehört (`src/pricing/bootstrap.py`, neben `main.py`)
> Die Datei **wählt konkrete Adapter aus** — CSV oder SQLAlchemy. Das ist eine Kompositions-, keine Anwendungsentscheidung. Composition Roots sind von der Schichtregel ausgenommen, genau wie `main.py`. Läge die Datei in `application/`, würdest du diese Ausnahme stillschweigend zur Normalität erklären: Ab dann darf `application/` konkrete Persistenzklassen importieren, und die Regel ist tot.
>
> Sie sollte trotzdem **kein `fastapi` importieren** — aber nicht mehr aus dem Grund, der hier ursprünglich stand (das `api`-Extra, siehe 13.0: es ist entfallen). Zwei Gründe bleiben, und der zweite ist der eigentliche:
>
> 1. **Startzeit.** `main.py` importiert `bootstrap`, also zahlt jeder CLI-Aufruf, was `bootstrap` importiert. Gemessen: `import pricing.main` kostet ~440 ms (SQLAlchemy und Pydantic), `import fastapi` legt **~155 ms** darauf. Ein Drittel mehr Startzeit für `pricing-apikey list` — spürbar, aber allein kein Argument.
> 2. **`deps.py` muss ohnehin existieren.** Dort steht `require_api_key`, und diese Funktion trifft eine **HTTP**-Entscheidung: fehlgeschlagene Suche → `401` mit `WWW-Authenticate`-Header. Das ist Presentation-Logik, die in einem Composition Root nichts zu suchen hat. Wenn die Datei also bleibt, wäre es schlechter, die `Annotated`-Aliase von ihr zu trennen: Du hättest die FastAPI-Verdrahtung auf zwei Dateien verteilt, statt sie an einem Ort zu haben.
>
> Kurz: `Depends` und `Annotated` bleiben in `deps.py`, weil sie dort **zu ihren Nachbarn gehören** — nicht, weil ein Import verboten wäre.

---

## Schritt 13.0 — das `api`-Extra ist entfallen ✅ (erledigt 2026-08-13)

Der Fund, der dazu geführt hat: `uv run` synchronisiert die venv **vor** jedem Aufruf auf den **Standardzustand** des Projekts. Der umfasst die `[dependency-groups] dev` (deshalb bleiben `pytest`, `ruff`, `mypy` erhalten) — aber **keine Extras**. Jedes nackte `uv run …` deinstallierte also FastAPI wieder:

```
$ uv run python -c "import fastapi"
ModuleNotFoundError: No module named 'fastapi'
```

Es gab drei Wege heraus, und der gewählte ist der einzige, der das Problem wirklich beseitigt statt es zu verwalten:

| Weg | Bewertung |
|---|---|
| Immer `uv run --extra api …` schreiben | funktioniert, und du vergisst es genau dann, wenn es weh tut |
| `[tool.uv] default-extras = ["api"]` | getestet, funktioniert — hält aber ein Extra am Leben, das keins ist |
| **FastAPI + Uvicorn nach `[project.dependencies]`** | ✅ gewählt |

**Die Begründung ist deine eigene aus der uv-Migration.** Dort sind die Extras `db`/`orm` entfallen, weil `sqlalchemy` auf **Modulebene** importiert wird — eine Abhängigkeit, ohne die das Paket nicht importierbar ist, ist kein Extra. Für FastAPI gilt genau dasselbe: `app.py` und `deps.py` importieren es auf Modulebene, und die REST-API ist eine Hauptfunktion, kein Zusatz. `[project.optional-dependencies]` bleibt damit entfallen, so wie nach der Migration vorgesehen.

- [x] `fastapi>=0.141.1` und `uvicorn[standard]>=0.52.1` in `[project.dependencies]`, mit Kommentar zur Begründung
- [x] `[project.optional-dependencies]` entfernt
- [x] `uv sync` (ohne Flags) → `uv run python -c "import fastapi"` läuft durch
- [x] Gates nach der Umstellung: Ruff grün, `mypy --strict` 43 Dateien grün, **57 Tests grün**

---

## Schritt 13.1 — `src/pricing/bootstrap.py`: Engine und Session

Neue Datei. Fang mit einem Docstring an, der die Regel festhält, denn dieser Datei sieht man ihre Sonderrolle sonst nicht an:

```python
"""Composition Root: hier -- und nur hier -- werden konkrete Klassen ausgewaehlt.

Von der Schichtregel ausgenommen, wie main.py. Importiert bewusst kein fastapi:
main.py importiert dieses Modul, und der CLI-Start soll die ~155 ms fuer den
FastAPI-Import nicht zahlen. Die Depends-Verdrahtung bleibt in
presentation/api/deps.py, wo auch require_api_key steht.
"""
```

### Funktion 1: `engine()`

```python
@lru_cache
def engine() -> Engine:
    return get_engine()
```

`from functools import lru_cache`, `from sqlalchemy import Engine`, `from .persistence.db import get_engine`.

**Was `lru_cache` hier tut:** Es ist ein Dekorator, der sich die Rückgabewerte einer Funktion pro Argumentkombination merkt. Beim ersten Aufruf läuft der Körper und das Ergebnis wird gespeichert; bei jedem weiteren Aufruf mit denselben Argumenten kommt das gespeicherte Objekt zurück, ohne dass der Körper läuft. Bei einer Funktion **ohne Parameter** gibt es genau eine Kombination — die Funktion wird also effektiv zu einem Singleton-Zugriff. Du kennst das Muster schon aus `shared/config.py`, wo `get_settings()` genau so gebaut ist.

Dass wir es hier als *neue Funktion in bootstrap* schreiben und nicht einfach `@lru_cache` auf `get_engine` in `db.py` kleben, hat einen Grund: `get_engine(echo=...)` hat einen Parameter, und `get_engine(echo=True)` müsste weiterhin eine frische Engine mit Logging liefern. Die Entscheidung „**für die Anwendung teilen wir eine Engine**" ist eine Kompositionsentscheidung und gehört deshalb hierher, nicht in die Persistenzschicht.

> [!warning] `lru_cache` und Tests
> Ein gecachtes Objekt überlebt den Test, der es erzeugt hat. Wenn du in Block 17 gegen eine andere Datenbank testest, brauchst du `engine.cache_clear()` — dieselbe Methode, die `lru_cache` auch `get_settings` mitgegeben hat.

> [!note] Eine Engine, viele Threads — ist das sicher?
> Ja, und das ist die naheliegende Sorge. FastAPI führt **synchrone** Handler (also alle deine) in einem Threadpool aus, mehrere gleichzeitig. Sicher ist das, weil die geteilte Sache der *Pool* ist und der explizit threadsicher ist: Er gibt jedem Thread eine eigene Verbindung heraus. Nicht threadsicher ist die `Session` — und die bleibt pro Request eine eigene. Die Regel lautet: **Engine teilen, Session niemals.**

### Funktion 2: `session_scope()`

Das ist der heutige `get_session` aus `deps.py`, wörtlich umgezogen, mit `engine()` statt `get_engine()`:

```python
def session_scope() -> Generator[Session]:
    """Eine Session mit Transaktionsklammer. Generatorform -- so will FastAPI es."""
    with Session(engine()) as session:
        try:
            yield session
            session.commit()
        except:
            session.rollback()
            raise
```

Zum nackten `except:` ohne Exception-Typ: Das ist hier **Absicht** und keine Schludrigkeit. Es muss auch `BaseException` fangen — etwa wenn der Server während einer laufenden Anfrage abgebrochen wird. Ein Rollback ist in *jedem* Fall die richtige Reaktion, und das `raise` gibt den Fehler unverändert weiter. Ruff akzeptiert das, weil unmittelbar `raise` folgt.

### Funktion 3: `session_cm`

Eine Zeile, kein `def`:

```python
session_cm = contextmanager(session_scope)
```

`from contextlib import contextmanager`. Das ist der Kern des Blocks, also einmal ausführlich:

> [!warning] `Iterator[Session]` genügt hier nicht — `Generator[Session]` (gefunden 2026-08-13)
> Pylance meldet an dieser Zeile: *„Annotating the return type as `-> Iterator[Foo]` with `@contextmanager` is deprecated. Use `-> Generator[Foo]` instead."* Die Meldung gilt **nicht** dem `contextmanager`, sondern der Annotation von `session_scope`. Sie ist berechtigt:
>
> `Iterator[T]` sagt nur „liefert T". Ein Kontextmanager-Generator wird aber auch von außen **bedient**: `contextlib` ruft `.throw()` darauf, um eine Ausnahme aus dem `with`-Block an der `yield`-Stelle hineinzuwerfen — genau das lässt dein `except: rollback()` überhaupt auslösen. Diese Fähigkeit gehört zum Generator-Protokoll, nicht zum Iterator-Protokoll. `Generator[YieldType, SendType, ReturnType]` drückt sie aus; seit Python 3.13 haben die letzten beiden Parameter Vorgabewerte (`None`), deshalb reicht die Kurzform.
>
> Damit ändert sich in `bootstrap.py`: `from collections.abc import Generator` statt `Iterator`, und `def session_scope() -> Generator[Session]:`. Geprüft — `Generator[Session]` und `Generator[Session, None, None]` sind beide zur Laufzeit subskribierbar und beide `mypy --strict`-grün; nimm die Kurzform, die Pylance selbst vorschlägt.
>
> **Warum `mypy` dazu geschwiegen hat, und das ist die eigentliche Lektion:** Beide Werkzeuge lesen Typstubs aus **typeshed**, aber jedes bringt seine eigene Kopie mit. Im typeshed von `mypy 1.x` in deiner venv steht genau *eine*, nicht markierte Signatur:
> ```python
> def contextmanager(func: Callable[_P, Iterator[_T_co]]) -> Callable[_P, _GeneratorContextManager[_T_co]]: ...
> ```
> Pylance bringt ein neueres typeshed mit, das die `Iterator`-Überladung als deprecated führt. **Zwei Werkzeuge, zwei Stub-Versionen, zwei Wahrheiten** — und eine Deprecation auf Stub-Ebene erzeugt zur Laufzeit *keine* `DeprecationWarning` (mit `-W error::DeprecationWarning` gegengeprüft: nichts). Wenn Editor und CI-Gate sich widersprechen, ist deshalb nicht automatisch der Editor im Unrecht. Hier hatte er recht und `mypy` war bloß älter.

Du brauchst dieselbe Session-Logik in **zwei Gestalten**. FastAPI will eine **Generatorfunktion**: Es ruft sie auf, nimmt den Wert beim `yield` als Dependency, bearbeitet den Request und läuft danach den Rest des Generators (also `commit` bzw. `rollback`). Ein `with`-Block kann mit einer Generatorfunktion aber nichts anfangen — dafür braucht Python ein Objekt mit `__enter__`/`__exit__`.

`contextmanager` ist genau dieser Übersetzer: Er nimmt eine Generatorfunktion und gibt eine Funktion zurück, die beim Aufruf einen Kontextmanager liefert. Der Trick ist, dass du ihn **nicht als Dekorator** benutzt, sondern als normale Funktion aufrufst — dann bleiben *beide* Namen erhalten:

```
contextmanager(session_scope) ->  mit "with" nutzbar
session_scope()               ->  bleibt ein Generator
```

Genau das war gemessen. Als Dekorator (`@contextmanager` über `session_scope`) hättest du nur noch die `with`-Form, und FastAPI könnte die Funktion nicht mehr als Dependency verwenden. Eine Implementierung, zwei Gestalten, kein doppelter `commit`/`rollback`-Code.

- [x] `bootstrap.py` angelegt, mit Docstring
- [x] `engine()` mit `@lru_cache`
- [x] `session_scope()` — inhaltlich unverändert gegenüber `deps.get_session`, nur `engine()` statt `get_engine()`
- [x] Rückgabetyp auf `Generator[Session]` umgestellt, Import `Generator` statt `Iterator` aus `collections.abc` (siehe Callout — Pylance hatte recht)
- [x] `session_cm = contextmanager(session_scope)`

---

## Schritt 13.2 — die zwei `PricingApp`-Fabriken (weiter in `bootstrap.py`)

Zwei Funktionen, weil es zwei echte Varianten gibt — CSV für den CLI-Lauf, Datenbank für API und Smoke-Test.

```python
def make_csv_pricing_app(data_dir: Path, output: Path) -> PricingApp: ...
def make_db_pricing_app(session: Session, reference_date: date | None = None) -> PricingApp: ...
```

Im Körper steht jeweils genau der `PricingApp(...)`-Aufruf, der heute in `main.py` bzw. `smoke_orm.py` steht — mit den Argumenten aus der Signatur statt fest eingetragener Werte. Was du dabei entscheidest:

- **Pfade sind Parameter, keine Konstanten.** Heute steht `Path("daily_data")` in `main.py:32` und `Path("price_recommendations.csv")` in `main.py:35`. Beides wandert in die Signatur, damit `bootstrap` nichts über *dein* Verzeichnislayout weiß. Der Aufrufer entscheidet.
- **`reference_date` durchreichen.** `RecommendationEngine.__init__` nimmt es schon (`reference_date: date | None = None`, `domain/recommendations.py:37`), und `smoke_orm.py` braucht es für seinen festen Vergleichswert `REF = date(2026, 7, 22)`. `None` heißt dort „heute" — das ist bereits so implementiert, du musst nur den Parameter weiterleiten.

- [x] beide Fabriken in `bootstrap.py`, Rückgabetyp `PricingApp`
- [x] kein `Path(...)`-Literal und kein Datum im Körper — alles kommt aus der Signatur

---

## Schritt 13.3 — `src/pricing/main.py` umstellen

In `main(argv)` (der in **`src/pricing/main.py`**, nicht der in `api_key_admin.py` oder `ingest.py`) ersetzt du den `PricingApp(...)`-Block durch einen Aufruf von `make_csv_pricing_app(...)` mit denselben zwei Pfaden.

Danach räumst du die Importe auf: `PricingApp`, `PriceableUnitBuilder`, `RecommendationEngine`, `HeuristicRecommendationStrategy`, `CsvDataSource`, `CsvRecommendationWriter` braucht `main.py` alle nicht mehr — sie werden zu einem Import von `bootstrap`. Sechs Importe werden einer. `Path` bleibt, weil die Pfadliterale jetzt hier stehen.

- [x] `uv run pricing -v` liefert **denselben** Wert für „Höchste Empfehlung" wie vorher
- [x] Ruff meldet keine unbenutzten Importe in `main.py`

> [!tip] Vorher messen
> Lauf `uv run pricing -v` **bevor** du umstellst und notier die Zahl. Ein Refactoring ohne vorher festgehaltenen Referenzwert ist eine Hoffnung, keine Prüfung.

---

## Schritt 13.4 — die fünf Repository-Fabriken (weiter in `bootstrap.py`)

Je Vertrag eine Fabrik, jede genau eine Zeile Körper:

```python
def make_api_key_repository(session: Session) -> ApiKeyRepository: ...
def make_product_repository(session: Session) -> ProductRepository: ...
def make_booking_repository(session: Session) -> BookingRepository: ...
def make_capacity_repository(session: Session) -> CapacityRepository: ...
def make_recommendation_repository(session: Session) -> RecommendationRepository: ...
```

**Der Rückgabetyp ist das Protocol, nicht die Klasse** — und das ist der ganze Gewinn. `mypy` prüft an dieser Stelle, ob `SqlAlchemyBookingRepository` den Vertrag `BookingRepository` wirklich erfüllt: jede Methode, jeder Parametername, jeder Typ. Nennst du als Rückgabetyp die konkrete Klasse, prüft niemand mehr etwas, und eine Abweichung fällt erst dort auf, wo aufgerufen wird.

Das ist auch die Antwort auf deine Frage von Block 6 („warum brauche ich diese Interfaces?"), an ihrer schärfsten Stelle: Der Vertrag ist erst dann ein Vertrag, wenn er an einer Stelle steht, wo ein Werkzeug ihn **gegen** die Implementierung hält. Diese fünf Signaturen sind genau diese Stelle.

> [!warning] Der Test kommt sofort — mach ihn absichtlich
> Ändere zur Probe in `interfaces.py` einen Parameternamen, z. B. `von` in `BookingRepository.list_all` zu `ab`. `mypy --strict` muss jetzt bei `make_booking_repository` meckern, nicht irgendwo im Router. Danach zurückändern. Ein Vertrag, dessen Verletzung du nie gesehen hast, ist unbewiesen.

- [x] fünf Fabriken, Rückgabetyp jeweils das Protocol aus `persistence/interfaces.py`
- [x] `mypy --strict` grün
- [x] die absichtliche Vertragsverletzung einmal gesehen und rückgängig gemacht — **offen, das ist deine Lernübung** (Claudian hat sie nicht für dich gemacht)

---

## Schritt 13.5 — `deps.py` umstellen

In **`src/pricing/presentation/api/deps.py`** bleibt die Struktur (`Annotated`-Aliase, `Depends`), aber die Körper werden zu Weiterleitungen:

```python
def get_session() -> Iterator[Session]:   # loeschen -- ist jetzt bootstrap.session_scope
def get_booking_repository(session: SessionDep) -> BookingRepository:
    return make_booking_repository(session)      # statt SqlAlchemyBookingRepository(session)
```

Und `SessionDep` zeigt auf die Fabrik aus bootstrap:

```python
SessionDep = Annotated[Session, Depends(session_scope)]
```

Damit verschwinden aus `deps.py`: `persistence.db.get_engine`, `persistence.api_keys.SqlAlchemyApiKeyRepository` und alle vier Klassen aus `persistence.repositories`. Es bleiben die Verträge aus `persistence.interfaces`, `application.auth_service.authenticate` und die Fabriken aus `bootstrap`.

> [!warning] Korrektur an meiner früheren Behauptung
> In der ersten Fassung dieses Blocks stand, `deps.py` importiere danach **kein `sqlalchemy`** mehr. Das ist falsch, und zwar nicht knapp: Die Zeile `SessionDep = Annotated[Session, Depends(...)]` **braucht** den Namen `Session` zur Laufzeit, weil FastAPI die Annotationen beim Bauen des Routen-Graphen auflöst. Ein Import unter `if TYPE_CHECKING:` genügt dafür nicht.
>
> Es gibt drei Wege, die Importtabelle trotzdem sauber aussehen zu lassen, und alle drei sind schlechter als der ehrliche Import:
>
> 1. `Annotated[Any, Depends(session_scope)]` — `mypy --strict` schluckt das, FastAPI auch. Du hast dann eine hübsche Importliste und **keine Typprüfung mehr** an der Stelle, wo die Session an fünf Fabriken weitergegeben wird. Ein echtes Werkzeug gegen eine kosmetische Kennzahl getauscht.
> 2. `type DbSession = Session` in `persistence/interfaces.py` re-exportieren und den Alias importieren. Der Importgraph zeigt dann auf `interfaces` statt auf `sqlalchemy` — das Objekt ist immer noch eine SQLAlchemy-Session. Es verschiebt nur, wo die Wahrheit steht.
> 3. `Depends` nach `bootstrap` holen. Bricht den CLI-Lauf ohne `api`-Extra, siehe oben.
>
> Die belastbare Regel lautet deshalb: **Presentation nennt Verträge und Fabriken, keine Implementierungen.** `Session` ist ein Typname für die Annotation, keine Auswahl einer Implementierung — die einzige Stelle, die `Session(...)` *aufruft*, ist `bootstrap`. Genau das prüft der Architektur-Test in Block 17: verboten sind `persistence.orm`, `persistence.repositories`, `persistence.api_keys`, `persistence.db` und `psycopg`; `sqlalchemy.orm.Session` ist in `deps.py` erlaubt.

**Vorher → nachher für `deps.py`:**

| | importiert aus `persistence` | importiert `sqlalchemy` | ruft `SqlAlchemy…()` auf |
|---|---|---|---|
| vorher | `db`, `api_keys`, `repositories`, `interfaces` | `Session` | ja, 5× |
| nachher | nur `interfaces` | nur `Session` (Annotation) | nein |

- [x] `get_session` aus `deps.py` gelöscht, `SessionDep` zeigt auf `bootstrap.session_scope`
- [x] alle sechs Fabrik-Funktionen leiten weiter, kein `SqlAlchemy…` mehr in der Datei
- [x] `uv run uvicorn pricing.presentation.api.app:app --reload` im Browser prüfen — **offen**; Claudian hat stattdessen die ASGI-App direkt getrieben und das OpenAPI-Schema erzeugt (12 Pfade, 18 Operationen), was denselben Routen-Graph baut
- [x] ein falscher Schlüssel liefert weiter `401` (die Auth-Kette geht durch dieselbe Session)

---

## Schritt 13.6 — `scripts/smoke_orm.py` umstellen

Hier zahlt sich `session_cm` aus. Aktuell steht dort `with Session(engine) as session:` und weiter unten ein manuelles `session.commit()` (Zeile 105). Beides erledigt jetzt die Klammer:

```python
with session_cm() as session:
    check_data(session)
    app = make_db_pricing_app(session, reference_date=REF)
    result = app.run()
    # kein session.commit() mehr -- session_scope macht das beim Verlassen
```

Zwei Punkte, die du beim Umstellen bewusst entscheiden musst:

- **`check_schema(engine)` braucht weiter eine Engine.** Nimm `engine()` aus bootstrap. Damit fällt auch das lokale `get_engine(echo=False)` weg — und mit ihm der `try`, der `ConfigError` in eine Meldung übersetzt. Den Fall „Passwort fehlt" musst du also um den *ersten* `engine()`-Aufruf legen, nicht mehr um eine separate Zeile.
- **`engine.dispose()` im `finally`** bezieht sich jetzt auf die geteilte Engine. Das ist in einem Skript, das danach endet, richtig — nur ist der Name jetzt ein Funktionsaufruf: `engine().dispose()`.

- [x] `uv run python scripts/smoke_orm.py` liefert denselben Wert wie vorher
- [x] die drei Vorbedingungs-Meldungen mit gestopptem Container prüfen (`docker compose stop db`) — **offen**, in Claudians Umgebung ist `docker` nicht verfügbar

---

## Schritt 13.7 — Gates und Abschluss

- [x] `uv run ruff check src scripts tests` und `uv run ruff format --check` grün
- [x] `uv run mypy --strict src scripts` grün
- [x] `uv run pytest` — **57 Tests weiter grün**, keiner angepasst (dies ist ein Refactoring: verhaltensneutral per Definition)
- [x] `uv run pricing -v` und `smoke_orm.py` liefern die notierten Referenzwerte
- [x] Im Umsetzungsstand festhalten: der Engine-pro-Request-Fund mit den gemessenen Zahlen, `default-extras`, und die korrigierte Schichtregel

> [!tip] Jetzt ist der Moment für den Architektur-Test
> Erst nach diesem Umbau ist „Presentation nennt keine Implementierungen" tatsächlich wahr — also lässt sich die Regel festnageln. Ein Test, der die `import`-Anweisungen aller Dateien unter `presentation/` per AST prüft, macht aus einer Absicht etwas, das `pytest` durchsetzt. Details in Block 17.

---

# Block 14 — `POST /pricing-runs`: `202` und Hintergrundausführung

**Neue Technik:** neue Tabelle, `BackgroundTasks`, Statusressource, `202 Accepted`.

**Der Block mit der gefährlichsten Falle.** Die [[Umsetzungsstand (REST APIs & Layered Architecture)#Laufzeit-Messung (2026-08-04)|42-Sekunden-Messung]] hat ihn erzwungen: Ein Lauf dauert 16–42 s, übliche Gateway-Timeouts liegen bei 30–60 s.

## Was am Ende funktionieren muss

```bash
curl -si -X POST -H "X-API-Key: $API_KEY" localhost:8000/v1/pricing-runs
# 202 Accepted · Location: /v1/pricing-runs/1 · {"id":1,"status":"queued"}

curl -s -H "X-API-Key: $API_KEY" localhost:8000/v1/pricing-runs/1
# {"id":1,"status":"running",…}   … später "succeeded" mit recommendation_count

curl -si -X POST -H "X-API-Key: $API_KEY" localhost:8000/v1/pricing-runs   # 409, solange einer läuft
```

## Die neue Syntax

```python
@router.post("", status_code=status.HTTP_202_ACCEPTED)
def start_run(tasks: BackgroundTasks, response: Response, ...) -> RunRead:
    ...                                    # Zeile mit status="queued" anlegen und COMMITTEN
    tasks.add_task(execute_run, run_id)    # läuft NACH der Antwort, im Threadpool
    response.headers["Location"] = f"/v1/pricing-runs/{run_id}"
    return ...
```

---

### Schritt 14.1 — Tabelle `pricing_runs`: `persistence/orm.py`

| Spalte | Zweck |
|---|---|
| `id` | Primärschlüssel |
| `status` | `queued` / `running` / `succeeded` / `failed` — als `Text` mit `CheckConstraint`, wie `bookings.status` |
| `reference_date` | mit welchem Referenzdatum gerechnet wurde |
| `started_at`, `finished_at` | `timestamptz`, nullable |
| `recommendation_count` | Ergebnis, nullable |
| `error_message` | bei `failed`, nullable |

- [x] Tabelle in `orm.py`, Naming Convention wie überall
- [x] `uv run pricing-initdb` legt sie an — **neue** Tabelle, also unproblematisch für `create_all()`
- [x] `CheckConstraint` für die vier Statuswerte

---

### Schritt 14.2 — Der Lauf im Hintergrund: `application/run_service.py` *(neu)*

```python
def execute_run(run_id: int) -> None: ...      # bekommt eine ID, KEINE Session, KEIN Objekt
```

Ablauf: eigene Session öffnen → Status auf `running`, committen → `PricingApp.run()` → Ergebnis und `succeeded` schreiben, committen. Bei Fehler: `failed` + `error_message`, committen.

- [x] `execute_run(run_id: int)` — Parameter ist eine **Zahl**
- [x] Öffnet ihre **eigene** Session über `bootstrap.session_scope()`
- [x] Schreibt Statuswechsel **einzeln committend**, nicht am Ende in einem Rutsch
- [x] Fängt `Exception`, schreibt `failed` + Meldung, loggt mit `logger.exception`

> [!bug] Die Falle, die diesen Block gefährlich macht
> **Die Hintergrundaufgabe darf die Session aus `Depends` nicht benutzen.** Die wird geschlossen, sobald die Antwort raus ist — die Aufgabe läuft danach. Wer die Request-Session weiterreicht, bekommt `DetachedInstanceError` oder eine geschlossene Verbindung, und zwar **unzuverlässig**, weil es vom Timing abhängt.
>
> Deshalb bekommt `execute_run` die **`run_id`** als Zahl und öffnet ihre eigene Session. Genau deshalb braucht der Lauf überhaupt eine Datenbank-ID: Sie ist der einzige Wert, der eine Prozessgrenze überlebt.

> [!warning] Fortschritt, der nicht committet wird, ist unsichtbar
> Wenn du Status und Zwischenstände in **einer** Transaktion schreibst, sieht sie niemand von außen, bis der Lauf fertig ist — der Status springt von `queued` direkt auf `succeeded`. Jeder Statuswechsel braucht ein eigenes `commit()`. Dasselbe Prinzip steckt hinter der geparkten Fortschrittsanzeige im Umsetzungsstand.

---

### Schritt 14.3 — Die Endpunkte

- [x] `POST /pricing-runs` → `202` + `Location`, antwortet **sofort**
- [x] `GET /pricing-runs/{id}` → Status; unbekannt → `404`
- [x] `GET /pricing-runs` → Historie, neueste zuerst, mit Default-Limit
- [x] Läuft schon einer (`queued` oder `running`) → **`409`** statt eines zweiten

> [!warning] Zwei parallele Läufe würden sich gegenseitig zerstören
> Der Writer macht `TRUNCATE price_recommendations` und legt eine Temp-Tabelle **ohne** `checkfirst` an (steht als offener Punkt im Umsetzungsstand). Zwei gleichzeitige Läufe = kaputte Daten oder ein Fehler. Die `409`-Regel ist deshalb kein Komfort, sondern der Schutz.
>
> Ehrlich dazu: Die Prüfung „läuft schon einer?" ist selbst eine Race Condition, wenn zwei Requests gleichzeitig kommen. Wirklich dicht wird es erst mit einem Unique-Index auf „höchstens eine Zeile mit Status running" (partieller Index) oder einem Advisory Lock. Für den Prototyp reicht die Prüfung — **aber wissen, dass sie nicht dicht ist.**

---

# Block 15 — `POST /ingestion-runs`: dasselbe Muster, plus Pfad-Absicherung

**Neue Technik:** Eingabevalidierung für Dateipfade.

Struktur identisch zu Block 14 — deshalb kurz. Neu ist nur, dass hier ein **Pfad vom Client** hereinkommt.

## Was am Ende funktionieren muss

```bash
curl -si -X POST -H "X-API-Key: $API_KEY" -H "Content-Type: application/json" \
  -d '{"folder":"daily_data"}' localhost:8000/v1/ingestion-runs      # 202

curl -si -X POST … -d '{"folder":"../../etc"}' …                     # 422
curl -si -X POST … -d '{"folder":"gibtsnicht"}' …                    # 422
```

- [x] Tabelle `ingestion_runs` analog zu `pricing_runs`, plus Spalte `folder`
- [x] `execute_ingestion_run(run_id)` ruft `run_ingest()`, eigene Session — **Abweichung:** nur die `run_id`, kein `folder`. Der Ordner steht in der Zeile; die Hintergrundaufgabe liest ihn und prüft ihn **erneut** durch `resolve_ingest_dir`. Symmetrisch zu Block 14 (nur eine Zahl überquert die Grenze) und Absicherung in der Tiefe.
- [x] Neue Einstellung `ingest_base_dir: Path` in `shared/config.py` (Default `daily_data`)
- [x] Der Pfad aus dem Body wird gegen dieses Basisverzeichnis geprüft (`resolve_ingest_dir` in `application/ingestion_service.py`)
- [x] Außerhalb → `422`, nicht existierend → `422`; eigene Ausnahmeklassen `IngestFolderOutsideBaseError` / `IngestFolderMissingError` (beide `DomainError`), weil `TRY003` freie Meldungen am `raise` verbietet
- [x] `409`, solange ein Ingest läuft — mit **sprechender** Meldung: `ConflictError` nimmt jetzt optional einen Text

> [!warning] Ein Pfad vom Client ist eine Waffe
> Die Bewerberaufgabe verlangt „nimmt einen Ordnerpfad entgegen". Ungeprüft heißt das: Jeder darf dem Server sagen, welches Verzeichnis er lesen soll — `../../` inklusive. Die Absicherung ist ein Fünfzeiler:
>
> ```python
> ziel = (basis / eingabe).resolve()
> if not ziel.is_relative_to(basis.resolve()):
>     raise DomainError(...)
> ```
>
> `resolve()` **muss** vor der Prüfung stehen — sonst enthält der Pfad noch `..` und `is_relative_to` sagt fälschlich ja. Das ist der Unterschied zwischen „Prototyp" und „fahrlässig".

---

# Block 16 — Swagger-Politur

**Neue Technik:** `summary`, `responses=`, Docstrings als Beschreibung.

Kein neuer Code, sondern die Endpunkte selbsterklärend machen. `/docs` ist die Dokumentation, die niemand vergisst zu aktualisieren — **wenn** sie aus dem Code kommt.

- [x] `tags` pro Router (haben alle schon)
- [x] `summary="…"` pro Endpunkt — eine Zeile, erscheint in der Liste
- [x] Docstring pro Handler — wird zur ausführlichen Beschreibung, Markdown erlaubt
- [x] Fehlerfälle deklariert: `responses={404: {"description": "…"}, 409: {"description": "…"}}`
- [x] `title`, `version`, `description` an der `FastAPI`-Instanz
- [x] Bei `capacity`: die Truncate-Warnung aus Block 11 steht in der Beschreibung
- [x] Einmal komplett durch `/docs` klicken und **jeden** Endpunkt ausprobieren

> [!tip] Warum `responses=` mehr ist als Kosmetik
> Ohne die Angabe behauptet `/openapi.json`, jeder Endpunkt könne nur `200` und `422` liefern. Ein Client-Generator baut daraus Code, der `404` und `409` nicht behandelt. Die Deklaration ist Teil des Vertrags, nicht der Dokumentation.

> [!warning] Prüfe, was das Schema verrät
> `/openapi.json` ist öffentlich lesbar, sobald jemand die URL kennt. Sieh nach, dass dort keine internen Feldnamen, keine Beispieldaten mit echten Kundennummern und kein `key_hash` auftauchen.

---

# Block 17 — Tests mit `TestClient`

**Neue Technik:** `TestClient`, `dependency_overrides`, Fakes statt Datenbank.

## Voraussetzung zuerst

> [!warning] `TestClient` braucht ein HTTP-Client-Paket
> `fastapi` allein bringt es nicht mit (nur `fastapi[standard]` täte das). Ohne es wirft bereits der *Import* von `TestClient` einen `RuntimeError`, der das erwartete Paket beim Namen nennt. Es gehört in die **dev-Gruppe**, nicht ins `api`-Extra: Zum *Betreiben* der API braucht man es nicht, nur zum Testen. Beim Ausprobieren am 10.08. war es nicht installiert.

- [x] `uv add --group dev <paket>` — genau das, was die Fehlermeldung nennt

---

### Schritt 17.1 — `tests/conftest.py` füllen

Die seit Kapitel 1.3 leere Datei bekommt endlich Inhalt.

```python
@pytest.fixture
def anon_client() -> Iterator[TestClient]: ...      # ohne Schlüssel
@pytest.fixture
def client() -> Iterator[TestClient]: ...           # mit gültigem Schlüssel im Header
```

Der Kern ist das Ersetzen der Dependencies:

```python
app.dependency_overrides[get_api_key_repository] = lambda: FakeApiKeyRepository()
app.dependency_overrides[get_booking_repository] = lambda: FakeBookingRepository()
...
app.dependency_overrides.clear()        # ← nach jedem Test, sonst leckt es
```

- [x] `client` setzt den Header `X-API-Key` selbst, damit kein Test ihn wiederholt
- [x] `anon_client` schickt keinen Header
- [x] `dependency_overrides` wird nach jedem Test **geleert**
- [x] **Kein laufender Postgres nötig** — das ist die Messlatte für diesen Block

> [!note] Warum Overrides und nicht Monkeypatching
> `dependency_overrides` ist FastAPIs eingebauter Ersatzmechanismus: Du tauschst die **Dependency**, nicht das Modul. Damit entfällt das `monkeypatch.setattr("pricing.main.…")`-Problem aus Kapitel 1.3 — **ein String weniger, den kein Werkzeug prüft.** Und die Fakes werden durch die `Protocol`s aus `interfaces.py` typgeprüft: Driftet ein Fake von der echten Klasse ab, meldet es `mypy`, nicht erst die Produktion.

---

### Schritt 17.2 — Was geprüft wird

- [x] **Auth:** ohne Schlüssel `401`, falscher Schlüssel `401`, widerrufener Schlüssel `401`, `/health` ohne Schlüssel `200`
- [x] **Pro Endpunkt:** Erfolgsfall + der wichtigste Fehlerfall
- [x] **`/max` vor `/{id}`** — der Test, der die Routen-Reihenfolge festnagelt
- [x] **Die drei Konfliktfälle aus Block 11** (`409` doppelt, `422` unbekanntes Produkt, `409` beim Löschen)
- [x] **Paginierung:** `limit=9999` → `422`; Seite 1 und Seite 2 überschneiden sich nicht
- [x] **`204`** liefert wirklich einen leeren Body
- [x] **`Location`** nach `POST` zeigt auf eine URL, die dann `200` liefert
- [x] Das DTO filtert: eine Spalte, die nicht im Schema steht, erscheint nicht in der Antwort

> [!tip] Der `500`-Test braucht einen Schalter
> Ein Handler für `Exception` fängt in der Produktion alles ab — im `TestClient` wirft der Fehler standardmäßig trotzdem durch, damit du im Test den echten Traceback siehst. Wer das Produktionsverhalten prüfen will, braucht `TestClient(app, raise_server_exceptions=False)`.

---

### Schritt 17.3 — Der Architektur-Test

Die Schichtregel steht bisher nur in einer Notiz — also genau die Sorte Zusage, die in drei Monaten niemand mehr kennt. Sie lässt sich prüfen:

- [x] `tests/test_architecture.py`: läuft über alle `.py`-Dateien unter `src/pricing/presentation/`, liest ihre Importe (`ast.parse`, dann `ast.Import`/`ast.ImportFrom`)
- [x] Behauptung: **keine** importiert `psycopg`, `persistence.orm`, `persistence.repositories`, `persistence.api_keys` oder `persistence.db` — ohne Ausnahme, ab Block 13 auch `deps.py` nicht mehr
- [x] Einzige erlaubte `sqlalchemy`-Berührung: `Session` in `deps.py`, ausschließlich als Annotation (siehe Korrektur-Callout in Block 13.5) — der Test führt sie als benannte Ausnahme mit Begründung, nicht stillschweigend
- [x] Behauptung: `domain/` importiert nichts aus `application/`, `persistence/` oder `presentation/`

> [!important] Warum dieser Test mehr wert ist als er kostet
> Er ist ~20 Zeilen und macht aus einer Architekturabsicht eine **prüfbare Zusage**. Dieses Kapitel hat drei Fehler produziert, die kein Werkzeug sah (Entry Point, `per-file-ignores`, `[dependency-group]`). Die Schichtregel wäre der vierte gewesen — jetzt ist sie es nicht.

---

### Schritt 17.4 — Die Schichtung auswerten

Die Sinkhole-Frage ist [[#Die Sinkhole-Frage — entschieden am 2026-08-11|vorab entschieden]] — CRUD bekommt keine Anwendungsschicht. Was am Ende trotzdem in den Umsetzungsstand gehört, ist die **Auswertung mit Zahlen**:

- [x] Zeilen je Schicht neu messen und mit dem Stand vor der API vergleichen
- [x] Zwei Aufrufpfade dokumentieren: Preislauf **tief** (Router → application → domain → persistence), CRUD **flach** (Router → persistence). Beides bewusst, beides begründet
- [x] Was der Architektur-Test aus 17.3 tatsächlich abdeckt — und was er nicht sieht
- [x] Was von der Ist-Analyse des Leitfadens (Aufgabenteil 1) nach der Iteration anders zu beantworten wäre

Das ist der Aufgabenteil, den der Leitfaden mit „Analysiere den aktuellen Stand deiner Anwendung" meint — nur jetzt **nach** dem Umbau.

# Fortschritt (Gesamtüberblick)

- [x] Block 1 — Gerüst und `GET /health` *(2026-08-10)*
- [x] Block 2 — Schlüssel ausstellen: Tabelle und Kommando *(2026-08-10)*
- [x] Block 3 — Schlüssel prüfen: die erste Dependency *(2026-08-11)*
- [x] Block 4 — Erster Lesepfad: `GET /products`
- [x] Block 5 — Element-Endpunkt, 404 und der zentrale Exception-Handler
- [x] Block 6 — `GET /bookings`: Filter, Paginierung, stabile Sortierung
- [x] Block 7 — `POST /bookings`: erzeugen, `201`, `Location`
- [x] Block 8 — `PUT /bookings/{id}`: vollständig ersetzen
- [x] Block 9 — `DELETE /bookings/{id}`: `204` und Idempotenz
- [x] Block 10 — `prices` analog
- [x] Block 11 — `capacity`: CRUD mit echten Konflikten
- [x] Block 12 — `recommendations`: nur lesen, `/max`, Alias
- [x] Block 13 — `bootstrap.py`: ein Objektgraph für CLI und API
- [x] Block 14 — `POST /pricing-runs`: `202` und Hintergrundausführung
- [x] Block 15 — `POST /ingestion-runs`: dasselbe Muster, plus Pfad-Absicherung
- [x] Block 16 — Swagger-Politur
- [x] Block 17 — Tests mit `TestClient`

## Danach

Nach Block 17: die **Fortschrittsanzeige** für die langen Läufe (als Idee im Umsetzungsstand geparkt) und — falls gewünscht — **Ablaufzeiten für API-Schlüssel**. Beides sind eigene kleine Iterationen, keine Blöcke dieses Plans.
