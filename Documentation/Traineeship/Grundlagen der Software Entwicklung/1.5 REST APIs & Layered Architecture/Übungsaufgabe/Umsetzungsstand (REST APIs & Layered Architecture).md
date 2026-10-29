---
tags:
  - documentation
type: notiz
date: 2026-08-13
---

Abgeschlossener Umsetzungsstand der **REST-API-Iteration** der Pricing-Anwendung (Aufgabe: [[1.5 REST APIs & Layered Architecture - Leitfaden#Aufgabe|1.5 Leitfaden – Aufgabe]]). Baut auf [[Umsetzungsstand (Logging & Exceptionhandling)|Kapitel 1.4]] auf.

Theorie: [[1.5 REST APIs & Layered Architecture - Vertiefung|Vertiefung]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/Anki Karteikarten|Anki Karteikarten]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/Q&A/Weiterführende Stichpunkte.md|Q&A-Register]] · Syntax: [[FastAPI, Uvicorn & Pydantic - Syntax|FastAPI/Uvicorn/Pydantic]] · [[SQLAlchemy Schema & Beziehungen - Syntax|SQLAlchemy-Schema]] · Modelle: [[Datenmodelle und Schichtenübergänge|Datenmodelle und Schichtenübergänge]] · Review: [[Code Review (REST API)|Code Review (REST API)]]

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/REST_API/` · Abschluss-Commit `7acbcf0`

Ziel war eine **Presentation-Schicht als REST API**, ohne die bestehenden Schichten zu beschädigen: CRUD für alle Entitäten, die Endpunkte der Bewerberaufgabe, Swagger UI.

## Endstand

| Schicht | Dateien | Zeilen |
|---|---|---|
| `domain/` | 4 | 288 |
| `application/` | 5 | 190 |
| `persistence/` | 11 | 1378 |
| `presentation/` — App, Deps, Fehler, 8 Router, 6 Schemas | 22 | 767 |
| `shared/` | 5 | 195 |
| `main.py` / `bootstrap.py` (Composition Roots) | 2 | 183 |
| `tests/` | 12 | 1188 |

**24 Operationen auf 16 Pfaden** unter `/v1`, alle außer `/health` hinter `X-API-Key`. **123 Tests grün** (0,25 s), `ruff` sauber, `mypy --strict` sauber (63 Dateien). Alles über `uv run …`; Server per `uv run uvicorn pricing.presentation.api.app:app --reload`. Entrypoints: `pricing`, `pricing-ingest`, `pricing-initdb`, `pricing-apikey`.

## Wichtigste Entscheidungen

**Die leitende Regel: Presentation ruft nur Application auf.** Die API-Schicht kennt weder `persistence/` noch SQLAlchemy — einzige Ausnahme ist die Composition Root. Diese eine Regel hat die gesamte Iteration strukturiert und wurde als **Architektur-Test** (`tests/test_architektur.py`) maschinell abgesichert, statt auf Disziplin zu hoffen.

**Vierschichtige klassische Struktur statt Ports & Adapters.** `presentation → application → domain → persistence`, dazu `shared/` als Querschnitt. Das Kapitelthema ist klassische Schichtung, also wurde sie auch gebaut. Konsequenz: Die Verträge wanderten bewusst von `ports.py` nach `persistence/interfaces.py` — eine **Abkehr von der Dependency Inversion** aus 1.2. Die `Protocol`s selbst blieben erhalten; Austauschbarkeit war nie das Problem.

**SQLAlchemy besitzt das Schema.** Statt eines separaten `sql/schema.sql` gibt es eine einzige Quelle der Wahrheit. **Alembic wurde bewusst zurückgestellt** — Migrationen sind Stoff des nächsten Kapitels, und `create_all()` reicht, solange alle Daten aus CSVs reproduzierbar sind. (Genau diese Voraussetzung fiel in 1.6 weg.)

**Wechsel auf `uv` *vor* dem Hinzufügen von FastAPI.** So entstand der Lockfile gleich mit dem vollständigen Stack, statt zweimal. FastAPI und uvicorn wurden **Kern-Abhängigkeiten, kein Extra**: Sie werden auf Modulebene importiert — und ein Extra ist nur ehrlich, wenn das Programm ohne es noch läuft.

**Ein gemeinsamer Composition Root (`bootstrap.py`) für CLI und API.** Zwei getrennte hätten bedeutet, dass jedes neue Konstruktor-Argument an mehreren Stellen nachgezogen werden muss. Die Fabriken geben das **Protocol als Rückgabetyp** zurück, nicht die konkrete Klasse — nur so prüft mypy, ob die Implementierung den Vertrag wirklich erfüllt.

**Läufe als eigene Ressourcen** (`POST /pricing-runs`, `POST /ingestion-runs` → `202` + `Location`) statt synchroner Endpunkte. Der Auslöser war eine Messung: Ein Preislauf dauert 20–40 s, der Ingest über 2 Minuten — deutlich über üblichen Gateway-Timeouts (30–60 s). Damit wurde aus einer Design-Option eine Notwendigkeit. Jeder Statuswechsel läuft in einer **eigenen Transaktion**, weil ein `rollback` sonst auch den Fehlerbericht mitnehmen würde.

**Authentifizierung von Anfang an** (API-Key, SHA-256-gehasht, Header `X-API-Key`). Nachrüsten hätte 26 geschützte Endpunkte und rund 30 Tests gekostet.

**Keine Application-Schicht für reines CRUD.** Trennlinie: *`application/` enthält, wo Regeln sind* — und CRUD hat keine. Router → Repository → SQL direkt spart rund 20 Durchreiche-Funktionen. Die Repositories bleiben trotzdem, aber wegen **Kohäsion**, nicht wegen Austauschbarkeit.

**`capacity` bekommt volles CRUD**, entgegen der ursprünglichen Empfehlung. Es ist die einzige Tabelle mit echten Unique- und FK-Konflikten — dadurch wird `409` real testbar statt theoretisch.

**Geldbeträge als String im JSON.** Pydantics Standardverhalten für `Decimal` wurde bewusst beibehalten: Eine JSON-Zahl ist ein `double` und damit für Geld ungeeignet.

**Pfad-Versionierung `/v1` von Anfang an.** Später nicht mehr nachholbar, ohne jeden Client zu brechen.

**CRUD trotz erkannter fachlicher Grenze gebaut.** `POST /bookings` ändert weder Buchungslage noch Kapazität — die API-Oberfläche folgt hier der Übungsvorgabe, nicht der Domäne. Bewusst dokumentiert, damit die Endpunktliste später nicht für ein fachliches Modell gehalten wird.

**Kunden nicht automatisch anlegen.** `ON CONFLICT DO NOTHING` wurde verworfen: `customers` hat nur eine Spalte und ist ein reines Schlüsselverzeichnis. Auto-Anlage würde Tippfehler in Phantom-Kunden verwandeln, statt sie abzufangen.

**`prices` bekommt keinen eigenen Router.** Die Vorgabe (2× CRUD) war mit `bookings` und `capacity` erfüllt; ein dritter identischer Satz hätte nichts Neues gezeigt.

## Gelernte Lektionen

- **Strings, die kein Werkzeug prüft, sind die teuersten Zeilen im Projekt.** Mehrfach aufgetreten: der Entry Point in `pyproject.toml`, der Pfad in `per-file-ignores`, `[dependency-group]` statt `[dependency-groups]`, das `monkeypatch`-Ziel. Kein Linter, kein Typchecker, kein Test schlägt an — es passiert einfach nichts.
- **Ein stiller Erfolg ist teurer als ein lauter Fehler.** Der Architektur-Test meldete 26-mal grün, ohne die Frage überhaupt zu stellen. **Eine Prüfung muss man einmal absichtlich fehlschlagen sehen, um zu wissen, dass sie prüft.**
- **Ein kopiertes `.venv` ist doppelt kaputt:** Der Editable-Install zeigt weiter aufs Originalverzeichnis, *und* jedes Skript in `.venv/bin/` trägt den fremden Interpreter im Shebang.
- **Grüne Werkzeuge prüfen nicht dasselbe.** mypy arbeitet pfadbasiert und löst Imports nicht auf, pytest schon — beide grün heißt nicht, dass der Import zur Laufzeit funktioniert.
- **Ein Import-Nebeneffekt ist keine Konfiguration.** Umgebungsvariablen zu lesen gehört in die Composition Root, wo sichtbar ist, dass es passiert.
- **Eine Engine ist ein Verbindungspool, keine Verbindung.** Eine pro Request anzulegen wirft den Pool weg, bevor er etwas wiederverwenden kann.
- **Was einen Request überleben muss, darf keine Referenz sein.** Ein Background-Task bekommt eine `run_id`, keine Session — und FastAPI führt `BackgroundTasks` **vor** dem Commit der `yield`-Dependency aus.
- **Reihenfolge kann eine Sicherheitseigenschaft sein.** `resolve()` **vor** `is_relative_to()`, sonst steckt das `..` noch im Pfad.
- **Deklarierte Fehlerfälle sind Teil des API-Vertrags, nicht der Doku.** Ohne `responses=` behauptet `/openapi.json`, jeder Endpunkt liefere nur `200`/`422`.
