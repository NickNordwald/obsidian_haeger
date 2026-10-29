---
tags:
  - documentation
type: notiz
date: 2026-08-14
---

Abhakbare **Schritt-für-Schritt-Anleitung**, um das bestehende Schema der Pricing-Anwendung unter Alembic zu stellen (Aufgabe: [[1.6 Datenbank Migrations Tools - Leitfaden#Aufgabe|1.6 Leitfaden – Aufgabe]]). Begleitnotiz zum [[Umsetzungsstand (Datenbank Migrations Tools)|Umsetzungsstand]]; Theorie dahinter: [[1.6 Datenbank Migrations Tools - Vertiefung|Vertiefung]], insbesondere [[1.6 Datenbank Migrations Tools - Vertiefung#Alembic|Stichwort Alembic]].

> [!info] So arbeitest du mit dieser Liste
> **Abweichung vom sonstigen Arbeitsmodus, auf deine ausdrückliche Ansage:** Diesmal stehen die konkreten Befehle und der konkrete Code hier drin, statt dass du sie selbst erarbeitest. Damit du trotzdem etwas mitnimmst, hat **jeder Block dieselbe Struktur**:
>
> 1. **Was du tust** — die Befehle, zum Kopieren
> 2. **Warum** — die Entscheidung dahinter
> 3. **Woran du erkennst, dass es geklappt hat** — die Prüfung
>
> Die Kästen **„Halt kurz an"** sind der eigentliche Lernteil: eine Frage, die du beantworten können solltest, bevor du weitergehst. Lies sie nicht über.
>
> Arbeite die Blöcke **der Reihe nach** ab und hake die `- [ ]`-Kästchen ab. Wenn ein Ergebnis von meiner Beschreibung abweicht: **nicht weitermachen, sondern melden** — an drei Stellen (Blöcke 7, 8, 12) kann dein Schema Dinge tun, die ich ohne laufende Datenbank nicht mit Sicherheit vorhersagen kann. Diese Stellen sind als > [!warning] markiert.
>
> **Zeitbedarf:** realistisch 3–5 Stunden, sinnvoll auf zwei Sitzungen aufgeteilt. Ein guter Schnitt ist **nach Block 8** — dort ist die Baseline bewiesen und nichts an der echten Datenbank verändert.
>
> **Die drei dichtesten Blöcke** sind 4 (Umbau von `env.py`), 7 (Lese-Checkliste) und 11 (erste eigene Migrationen). Block 4 hat als Sicherheitsnetz eine vollständige Endfassung der Datei zum Abgleichen (4d) — nutze sie, statt lange zu suchen.

## Wo du am Ende landest

**Der Onboarding-Ablauf heute:**

```bash
docker compose up -d
uv run pricing-initdb              # ← create_all()
uv run pricing-apikey test
uv run uvicorn pricing.presentation.api.app:app --reload
```

**Der Onboarding-Ablauf danach:**

```bash
docker compose up -d
uv run alembic upgrade head        # ← die einzige Änderung
uv run pricing-apikey test
uv run uvicorn pricing.presentation.api.app:app --reload
```

**Genau eine Zeile ändert sich.** Das ist kein Zufall, sondern der Beleg dafür, dass die Entscheidung aus Kapitel 1.5 richtig war: Schemaverwaltung war schon damals ein eigenes Kommando und nicht Teil des Anwendungsstarts. Es wechselt nur der Motor dahinter.

Was du zusätzlich bekommst: eine Datenbank, die ihre eigene Version kennt, einen reproduzierbaren Aufbauweg für jede neue Umgebung, und ab Block 11 die Fähigkeit, das Schema zu **ändern** statt es nur anzulegen.

---

# Block 0 — Ausgangszustand sichern

**Was du tust:**

```bash
cd ~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools

git status                       # muss sauber sein
git switch -c feature/alembic    # eigener Branch

docker compose up -d
docker compose ps                # db muss "healthy" sein
```

Und einmal festhalten, was in der Datenbank gerade steht:

```bash
docker compose exec db psql -U pricing -d pricing -c '\dt'
```

**Warum:** Ab Block 9 fasst du die echte Entwicklungsdatenbank an. Vorher willst du wissen, was drin ist — und auf einem eigenen Branch arbeiten, damit `git switch main` jederzeit ein Rückweg ist.

**Prüfung:** `\dt` listet deine 9 Tabellen (`api_keys`, `bookings`, `capacity`, `customers`, `price_recommendations`, `prices`, `pricing_runs`, `products`, `ingestion_runs`). Falls **keine** Tabellen kommen, ist die Datenbank leer — auch gut, dann überspringst du später Block 9.

- [x] Branch `feature/alembic` angelegt, Arbeitsbaum sauber
- [x] Container läuft, `\dt` zeigt den Ist-Zustand
- [x] Notiert, ob die DB leer ist oder die 9 Tabellen hat: ______

```bash
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ docker compose ps
NAME              IMAGE         COMMAND                  SERVICE   CREATED          STATUS                   PORTS
pricing-adminer   adminer       "entrypoint.sh docke…"   adminer   10 seconds ago   Up 3 seconds             127.0.0.1:8080->8080/tcp
pricing-db        postgres:18   "docker-entrypoint.s…"   db        10 seconds ago   Up 9 seconds (healthy)   127.0.0.1:5432->5432/tcp
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ docker compose exec db psql -U pricing -d pricing -c '\dt'
Did not find any tables.
```


---

# Block 1 — Alembic aufnehmen

**Was du tust:**

```bash
uv add "alembic>=1.19.1"
uv run alembic --version
```

**Warum die Versionsuntergrenze:** Die Erkennung benannter CHECK-Constraints kam in **1.19.0** (04.08.2026), und **1.19.1** hat darin vier Tage später einen Fehler behoben (spaltengebundene Constraints wurden nicht berücksichtigt → falsche Erkennungen). Dein Schema hat fünf benannte `CheckConstraint`s (in vier Tabellen) — du willst die korrigierte Fassung.

**Warum Haupt-Abhängigkeit und nicht `--dev`:** Alembic wird im **Onboarding-Ablauf** gebraucht, also von jedem, der die Anwendung startet — nicht nur beim Entwickeln. Dieselbe Begründung, die in `pyproject.toml` schon bei FastAPI steht („Hauptfunktion der Anwendung, kein Zusatz"). Wäre Alembic in `[dependency-groups] dev`, würde ein Deployment-Image es nicht mitbringen.

**Prüfung:**

```bash
sed -n '/^dependencies = \[/,/^\]/p' pyproject.toml    # alembic muss in der Liste stehen
grep -n 'name = "alembic"' uv.lock                     # 3 Treffer sind richtig, s. u.
uv run alembic --version                               # 1.19.1 oder höher
```

> [!note] Warum drei Treffer in `uv.lock` richtig sind
> Ein Lockfile nennt ein Paket an **drei** Stellen, und jede bedeutet etwas anderes:
>
> | Stelle | Bedeutung |
> |---|---|
> | `[[package]] name = "alembic"` | Der Eintrag des Pakets selbst — mit aufgelöster Version und Prüfsumme |
> | `dependencies = [{ name = "alembic" }]` unter `name = "pricing"` | „Dein Projekt hängt davon ab" — die **aufgelöste** Kante im Abhängigkeitsgraphen |
> | `[package.metadata] requires-dist = [{ name = "alembic", specifier = ">=1.19.1" }]` | Deine **Forderung** aus `pyproject.toml`, wörtlich gespiegelt |
>
> Die letzten beiden Zeilen sind genau die Unterscheidung, die ein Lockfile ausmacht: **was du verlangt hast** (`>=1.19.1`) gegenüber **was daraus wurde** (1.19.1). Meine ursprüngliche Erwartung „muss 1 ausgeben" war schlicht falsch.
>
> *(Und `grep -A8` zeigte acht Zeilen — der `dependencies`-Block ist wegen des Kommentarkastens elf Zeilen lang, `alembic` stand also knapp außerhalb. Auch mein Fehler.)*

> [!question]- Halt kurz an: Warum `uv add` und nicht `pip install alembic`?
> Weil `pip install` die Abhängigkeit **nur in die venv** schreibt, nicht in `pyproject.toml` und nicht in `uv.lock`. Auf deinem Rechner läuft dann alles, bei jedem anderen (und in CI) fehlt Alembic — und niemand erfährt, welche Version du benutzt hast. `uv add` macht beides: Manifest **und** Lockfile. Das ist genau die Lektion aus dem uv-Umbau in 1.5: Eine Umgebung wird **erzeugt**, nicht kopiert.

- [x] `uv add "alembic>=1.19.1"` gelaufen
- [x] `uv run alembic --version` zeigt 1.19.1 oder höher
- [x] Alembic steht in `pyproject.toml` **und** in `uv.lock`


```bash
nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ sed -n '/^dependencies = \[/,/^\]/p' pyproject.toml    # alembic muss in der Liste stehen
grep -n 'name = "alembic"' uv.lock                     # 3 Treffer sind richtig, s. u.
uv run alembic --version                               # 1.19.1 oder höher
dependencies = [
    "pydantic-settings>=2.0",
    "psycopg[binary]>=3.2",
    "sqlalchemy>=2.0",
    # Die REST-API ist eine Hauptfunktion der Anwendung, kein Zusatz -- deshalb
    # harte Abhaengigkeit und kein optionales Extra. Ein Extra haette bedeutet,
    # dass jedes nackte "uv run" es wieder deinstalliert (uv synchronisiert auf
    # den Standardzustand, und Extras gehoeren nicht dazu).
    "fastapi>=0.141.1",
    "uvicorn[standard]>=0.52.1",
    "alembic>=1.19.1",
]
10:name = "alembic"
608:    { name = "alembic" },
627:    { name = "alembic", specifier = ">=1.19.1" },
Using CPython 3.13.5 interpreter at: /usr/bin/python3.13
Removed virtual environment at: .venv
Creating virtual environment at: .venv
Installed 44 packages in 28ms
alembic 1.19.1
```


---

# Block 2 — Migrationsumgebung anlegen

**Was du tust:**

```bash
uv run alembic init alembic
```

**Warum das `generic`-Template (und nicht `pyproject`):** In der [[1.6 Datenbank Migrations Tools - Vertiefung#2. Die Migrationsumgebung — was `alembic init` anlegt|Vertiefung]] steht, das `pyproject`-Template sei für dein Projekt naheliegend — alles an einem Ort. Für diesen ersten Aufbau gewinnt es trotzdem nicht, aus zwei Gründen:

1. Es erzeugt **trotzdem** eine `alembic.ini` (dort liegen weiterhin Verbindung und Logging). Du hättest also zwei Konfigurationsorte statt einem.
2. Wir höhlen `alembic.ini` in Block 3 ohnehin aus — die Verbindung kommt aus `shared/config.py`. Der Hauptvorteil des `pyproject`-Templates (echte Listen statt kommagetrennter Strings) betrifft Optionen, die du gar nicht setzt.

Der Wechsel ist später jederzeit möglich; es ist eine Formatfrage, keine Architekturentscheidung.

**Prüfung:** Es sind entstanden:

```
alembic.ini
alembic/
├── env.py
├── README
├── script.py.mako
└── versions/          (leer)
```

> [!question]- Halt kurz an: Warum liegt `env.py` in deinem Projekt und nicht in der Bibliothek?
> Weil es **kein Konfigurationsfile ist, sondern ein Skript, das bei jedem `alembic`-Aufruf ausgeführt wird**. Es entscheidet, wie die Verbindung zustande kommt und welches Metadata-Objekt verglichen wird. Genau das ist bei jedem Projekt anders — deshalb wird es generiert und gehört dir. Du wirst es gleich umbauen.

- [x] `uv run alembic init alembic` gelaufen
- [x] Die fünf Dateien/Ordner sind da

>[!note]- Ergebnis der Prüfing
>```bash
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ tree
.
├── alembic
│   ├── env.py
│   ├── README
│   ├── script.py.mako
│   └── versions
├── alembic.ini
├── daily_data
│   ├── 20260108_bookings.csv
│   ├── 20260108_capacity.csv
│   └── 20260108_prices.csv
├── docker-compose.yml
├── logs
│   └── pricing.log
├── price_recommendations.csv
├── pyproject.toml
├── scripts
│   ├── __pycache__
│   │   └── smoke_orm.cpython-313.pyc
│   └── smoke_orm.py
├── src
│   └── pricing
│       ├── application
│       │   ├── auth_service.py
│       │   ├── ingestion_service.py
│       │   ├── __init__.py
│       │   ├── pricing_app.py
│       │   ├── __pycache__
│       │   │   ├── auth_service.cpython-313.pyc
│       │   │   ├── ingestion_service.cpython-313.pyc
│       │   │   ├── __init__.cpython-313.pyc
│       │   │   ├── pricing_app.cpython-313.pyc
│       │   │   └── run_service.cpython-313.pyc
│       │   └── run_service.py
│       ├── bootstrap.py
│       ├── domain
│       │   ├── builder.py
│       │   ├── entities.py
│       │   ├── __init__.py
│       │   ├── __pycache__
│       │   │   ├── builder.cpython-313.pyc
│       │   │   ├── entities.cpython-313.pyc
│       │   │   ├── __init__.cpython-313.pyc
│       │   │   └── recommendations.cpython-313.pyc
│       │   └── recommendations.py
│       ├── __init__.py
│       ├── main.py
│       ├── persistence
│       │   ├── api_key_admin.py
│       │   ├── api_keys.py
│       │   ├── db.py
│       │   ├── ingest.py
│       │   ├── __init__.py
│       │   ├── interfaces.py
│       │   ├── orm.py
│       │   ├── __pycache__
│       │   │   ├── api_key_admin.cpython-313.pyc
│       │   │   ├── api_keys.cpython-313.pyc
│       │   │   ├── db.cpython-313.pyc
│       │   │   ├── ingest.cpython-313.pyc
│       │   │   ├── __init__.cpython-313.pyc
│       │   │   ├── interfaces.cpython-313.pyc
│       │   │   ├── orm.cpython-313.pyc
│       │   │   ├── repositories.cpython-313.pyc
│       │   │   ├── schema_admin.cpython-313.pyc
│       │   │   ├── sources.cpython-313.pyc
│       │   │   └── writers.cpython-313.pyc
│       │   ├── repositories.py
│       │   ├── schema_admin.py
│       │   ├── sources.py
│       │   └── writers.py
│       ├── presentation
│       │   ├── api
│       │   │   ├── app.py
│       │   │   ├── deps.py
│       │   │   ├── errors.py
│       │   │   ├── __init__.py
│       │   │   ├── __pycache__
│       │   │   │   ├── app.cpython-313.pyc
│       │   │   │   ├── deps.cpython-313.pyc
│       │   │   │   ├── errors.cpython-313.pyc
│       │   │   │   ├── __init__.cpython-313.pyc
│       │   │   │   └── responses.cpython-313.pyc
│       │   │   ├── responses.py
│       │   │   ├── routers
│       │   │   │   ├── bookings.py
│       │   │   │   ├── capacity.py
│       │   │   │   ├── health.py
│       │   │   │   ├── ingestion_runs.py
│       │   │   │   ├── __init__.py
│       │   │   │   ├── ping.py
│       │   │   │   ├── pricing_runs.py
│       │   │   │   ├── products.py
│       │   │   │   ├── __pycache__
│       │   │   │   │   ├── bookings.cpython-313.pyc
│       │   │   │   │   ├── capacity.cpython-313.pyc
│       │   │   │   │   ├── health.cpython-313.pyc
│       │   │   │   │   ├── ingestion_runs.cpython-313.pyc
│       │   │   │   │   ├── __init__.cpython-313.pyc
│       │   │   │   │   ├── ping.cpython-313.pyc
│       │   │   │   │   ├── pricing_runs.cpython-313.pyc
│       │   │   │   │   ├── product.cpython-313.pyc
│       │   │   │   │   ├── products.cpython-313.pyc
│       │   │   │   │   ├── recommendations.cpython-313.pyc
│       │   │   │   │   └── test.cpython-313.pyc
│       │   │   │   └── recommendations.py
│       │   │   └── schemas
│       │   │       ├── booking.py
│       │   │       ├── capacity.py
│       │   │       ├── ingestion_run.py
│       │   │       ├── __init__.py
│       │   │       ├── pricing_run.py
│       │   │       ├── product.py
│       │   │       ├── __pycache__
│       │   │       │   ├── booking.cpython-313.pyc
│       │   │       │   ├── capacity.cpython-313.pyc
│       │   │       │   ├── ingestion_run.cpython-313.pyc
│       │   │       │   ├── __init__.cpython-313.pyc
│       │   │       │   ├── pricing_run.cpython-313.pyc
│       │   │       │   ├── product.cpython-313.pyc
│       │   │       │   └── recommendation.cpython-313.pyc
│       │   │       └── recommendation.py
│       │   ├── __init__.py
│       │   └── __pycache__
│       │       └── __init__.cpython-313.pyc
│       ├── __pycache__
│       │   ├── app.cpython-313.pyc
│       │   ├── bootstrap.cpython-313.pyc
│       │   ├── builder.cpython-313.pyc
│       │   ├── db.cpython-313.pyc
│       │   ├── domain.cpython-313.pyc
│       │   ├── exceptions.cpython-313.pyc
│       │   ├── ingest.cpython-313.pyc
│       │   ├── __init__.cpython-313.pyc
│       │   ├── logging_setup.cpython-313.pyc
│       │   ├── main.cpython-313.pyc
│       │   ├── orm.cpython-313.pyc
│       │   ├── ports.cpython-313.pyc
│       │   ├── recommendations.cpython-313.pyc
│       │   ├── sources.cpython-313.pyc
│       │   └── writers.cpython-313.pyc
│       ├── py.typed
│       └── shared
│           ├── cli.py
│           ├── config.py
│           ├── exceptions.py
│           ├── __init__.py
│           ├── logging_setup.py
│           └── __pycache__
│               ├── cli.cpython-313.pyc
│               ├── config.cpython-313.pyc
│               ├── exceptions.cpython-313.pyc
│               ├── __init__.cpython-313.pyc
│               └── logging_setup.cpython-313.pyc
├── tests
│   ├── conftest.py
│   ├── __pycache__
│   │   ├── conftest.cpython-313.pyc
│   │   ├── conftest.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_api_auth.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_api_fehler.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_api_routing.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_app.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_architektur.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_builder.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_domain.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_exception.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_ingest_pfade.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_pricing.cpython-313-pytest-9.1.1.pyc
│   │   ├── test_recommendations.cpython-313-pytest-9.1.1.pyc
│   │   └── test_schema_vertrag.cpython-313-pytest-9.1.1.pyc
│   ├── test_api_auth.py
│   ├── test_api_fehler.py
│   ├── test_api_routing.py
│   ├── test_app.py
│   ├── test_architektur.py
│   ├── test_builder.py
│   ├── test_domain.py
│   ├── test_exception.py
│   ├── test_ingest_pfade.py
│   ├── test_recommendations.py
│   └── test_schema_vertrag.py
└── uv.lock
28 directories, 155 files```

---

# Block 3 — `alembic.ini` entschärfen

Öffne `alembic.ini`. Drei Änderungen.

### 3a — Die Datenbank-URL entfernen

Suche die Zeile

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

und **kommentiere sie aus**:

```ini
# URL wird NICHT hier gesetzt: alembic.ini liegt im Repository, ein Passwort
# gehoert nicht hinein. env.py holt die Verbindung aus shared/config.py (.env).
# sqlalchemy.url =
```

**Warum:** Zwei Gründe, beide schon im Projekt entschieden. Erstens wird `alembic.ini` eingecheckt — dein `.gitignore` schließt `*.env` aus, aber nicht `.ini`. Zweitens hättest du **zwei Quellen für dieselbe Information**, und genau das hat der Schema-Umbau in 1.5 beseitigt.

### 3b — Den ruff-Hook aktivieren

Weiter unten steht ein auskommentierter Abschnitt `[post_write_hooks]`. Der Abschnitt enthält **drei** Varianten (black, ruff-als-Modul, ruff-als-Programm) — du aktivierst die **mittlere**. Entferne dort die Rauten vor **vier** Zeilen:

```ini
# lint with attempts to fix using "ruff" - use the module runner, against the "ruff" module
hooks = ruff
ruff.type = module
ruff.module = ruff
ruff.options = check --fix REVISION_SCRIPT_FILENAME
```

> [!warning] Die Zeile `hooks = ruff` wird am leichtesten übersehen
> Sie steht in der Vorlage **eine Zeile über** den drei `ruff.*`-Zeilen und ist die einzige, die den Hook tatsächlich einschaltet. Bleibt sie auskommentiert, sind die anderen drei **wirkungslos** — und zwar lautlos: kein Fehler, keine Warnung, Alembic ignoriert Konfiguration für Hooks, die nicht in `hooks` genannt sind.
>
> Das ist derselbe Fehlertyp wie `[dependency-group]` statt `[dependency-groups]` beim uv-Umbau in 1.5: **ein String, den kein Werkzeug prüft.**
>
> Weiter unten steht ein **zweites** `# hooks = ruff` (die `exec`-Variante). Das bleibt auskommentiert — sonst hättest du den Schlüssel zweimal.
>
> **Prüfen:** `grep -n "^hooks" alembic.ini` muss **genau eine** Zeile ausgeben.

**Warum:** Generierte Migrationsdateien sind Python-Code in deinem Repository — und dein Repository hat eine Prüfkette (`ruff`, `mypy`). Ohne diesen Hook erzeugt Alembic Dateien, die deine eigenen Regeln verletzen (typisch: Importsortierung, überflüssige Leerzeilen). Der Hook lässt ruff direkt nach dem Generieren darüberlaufen. **Das Werkzeug fügt sich in deine Regeln ein, nicht umgekehrt.**

### 3c — Nichts weiter anfassen

Zwei Zeilen, über die du beim Lesen stolpern wirst und die **so bleiben**:

| Zeile | Bedeutung |
|---|---|
| `prepend_sys_path = .` | Legt das Projektwurzelverzeichnis auf `sys.path`. Für dich **nicht** der Grund, warum `import pricing` funktioniert — siehe Block 4 |
| `path_separator = os` | Wie Listen in dieser Datei getrennt werden. Neuere Alembic-Vorgabe, unkritisch |

> [!question]- Halt kurz an: Warum steht in der Vorlage `%%(rev)s_%%(slug)s` mit doppeltem Prozentzeichen?
> Weil `alembic.ini` von Pythons `configparser` gelesen wird, und dort ist `%` das Zeichen für Variableninterpolation (`%(here)s` wird ersetzt). Ein *literales* Prozentzeichen muss deshalb als `%%` geschrieben werden. Praktisch relevant wird das, wenn ein Passwort ein `%` enthält — dann kracht es an einer sehr unerwarteten Stelle. Deins nicht, aber merk es dir.

- [x] `sqlalchemy.url` auskommentiert, mit Begründung als Kommentar
- [x] `[post_write_hooks]` auf ruff gesetzt
- [x] `prepend_sys_path` und `path_separator` verstanden und unangetastet

---

# Block 4 — `env.py` an das Projekt anschließen

Das ist der inhaltlich wichtigste Block. Öffne `alembic/env.py`.

**Was du tust:** Drei Stellen ändern.

### 4a — Imports und `target_metadata`

Ganz oben, nach den bestehenden Imports:

```python
from pricing.persistence.db import get_engine
from pricing.persistence.orm import Base
```

Dann die Zeile `target_metadata = None` ersetzen:

```python
# Die Modellseite des Autogenerate-Vergleichs. Ohne diese Zeile erzeugt
# --autogenerate leere Migrationen (bzw. schlaegt vor, alles zu loeschen).
target_metadata = Base.metadata
```

### 4b — Der Online-Modus benutzt die Engine des Projekts

In `run_migrations_online()` steht:

```python
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
```

Ersetze das durch:

```python
    # Verbindung kommt aus dem Projekt (shared/config.py -> .env), nicht aus
    # alembic.ini. env.py ist eine Composition Root, wie main.py und
    # bootstrap.py -- sie darf alle Schichten kennen.
    connectable = get_engine()
```

Danach sind `engine_from_config` und `pool` unbenutzt — **beide Importzeilen oben löschen**, sonst meckert ruff.

### 4c — Der Offline-Modus auch

In `run_migrations_offline()` steht `url = config.get_main_option("sqlalchemy.url")`. Da die Option jetzt leer ist, ersetze sie:

```python
    url = get_engine().url.render_as_string(hide_password=False)
```

**Warum überhaupt:** Der Offline-Modus (`--sql`) erzeugt SQL zum Weitergeben, statt es auszuführen. **Du brauchst ihn in dieser Übung nirgends** — kein späterer Block ruft ihn auf. Eine halb kaputte Funktion im eigenen Code ist trotzdem schlechter als eine funktionierende, und es ist eine Zeile. Wenn dir Block 4 zu voll wird: Diese Änderung darfst du überspringen und später nachziehen.

*(`create_engine()` baut nur ein Objekt und verbindet sich nicht — den Aufruf hier zu machen, kostet also nichts.)*

### 4d — Abgleich: so sieht `alembic/env.py` am Ende komplett aus

Vergleiche Zeile für Zeile. Die englischen Kommentare aus der Vorlage darfst du behalten oder löschen — inhaltlich zählt der Rest.

```python
from logging.config import fileConfig

from alembic import context

from pricing.persistence.db import get_engine
from pricing.persistence.orm import Base

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# Die Modellseite des Autogenerate-Vergleichs. Ohne diese Zeile erzeugt
# --autogenerate leere Migrationen (bzw. schlaegt vor, alles zu loeschen).
target_metadata = Base.metadata


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode."""
    url = get_engine().url.render_as_string(hide_password=False)
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""
    # Verbindung kommt aus dem Projekt (shared/config.py -> .env), nicht aus
    # alembic.ini. env.py ist eine Composition Root, wie main.py und
    # bootstrap.py -- sie darf alle Schichten kennen.
    connectable = get_engine()

    with connectable.connect() as connection:
        context.configure(connection=connection, target_metadata=target_metadata)

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

**Die drei häufigsten Abweichungen**, wenn etwas nicht stimmt: `engine_from_config`/`pool` noch importiert (ruff meldet es), `target_metadata` steht noch auf `None`, oder die `pricing`-Importe stehen über `from alembic import context` (ruff sortiert das um — `pricing` ist per `known-first-party` die letzte Gruppe).

> [!question]- Halt kurz an: Warum funktioniert `from pricing.persistence.db import get_engine` überhaupt?
> **Nicht** wegen `prepend_sys_path = .` — das legt das Projektwurzelverzeichnis auf den Pfad, und dein Code liegt in `src/`. Es funktioniert, weil dein Paket **editable installiert** ist: In `.venv/lib/python3.13/site-packages/` liegt `_editable_impl_pricing.pth`, das auf `src/` zeigt. `uv run` stellt vor jedem Start sicher, dass die Installation aktuell ist.
>
> Das ist exakt dieselbe Mechanik wie bei `pricing-initdb`: **Entry Points und Imports entstehen beim *Installieren*, nicht beim Importieren** — die Lektion aus dem uv-Umbau in 1.5. Deshalb heißt es hier auch `uv run alembic` und nicht nur `alembic`.

> [!note] Was die Zeile `fileConfig(config.config_file_name)` tut
> Sie konfiguriert Pythons Logging aus dem `[loggers]`-Abschnitt von `alembic.ini` — deshalb siehst du gleich `INFO [alembic.runtime.migration] …`-Zeilen. Dein `shared/logging_setup.py` wird davon **nicht** gestört, weil Alembic in einem **eigenen Prozess** läuft. Innerhalb dieses Prozesses gilt allerdings Alembics Logging-Konfiguration, nicht deine.

**Prüfung:**

```bash
uv run ruff check alembic/env.py
uv run alembic current
```

**Erwartet** — genau diese zwei Zeilen und **keine Versionsangabe** darunter:

```
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
```

Beide Zeilen sind gute Nachrichten, und beide sagen etwas:

| Zeile | Bedeutung |
|---|---|
| `Context impl PostgresqlImpl` | Die Verbindung **steht**. Alembic hat sich verbunden und den PostgreSQL-Dialekt erkannt — also hat `get_engine()` aus deiner `.env` funktioniert |
| `Will assume transactional DDL` | Alembic weiß, dass diese Datenbank **DDL in Transaktionen** kann. Genau die Eigenschaft aus [[1.6 Datenbank Migrations Tools - Vertiefung#5.3 Transaktionalität: der Unterschied zwischen den Datenbanken\|Stichwort 1, Abschnitt 5.3]] — der Grund, warum eine abgebrochene Migration bei dir keinen Halbzustand hinterlässt |

**Dass darunter keine Versionsnummer steht, ist richtig:** Es gibt noch keine `alembic_version`-Tabelle. Bricht der Befehl mit `Traceback`, `ConfigError` oder `OperationalError` ab, siehe *Wenn etwas schiefgeht* ganz unten.

- [x] `target_metadata = Base.metadata` gesetzt
- [x] `run_migrations_online()` benutzt `get_engine()`
- [x] `run_migrations_offline()` benutzt die URL aus der Engine *(optional)*
- [x] Unbenutzte Imports (`engine_from_config`, `pool`) entfernt
- [x] **Datei gegen die Endfassung in 4d abgeglichen**
- [x] `uv run ruff check alembic/env.py` ist sauber
- [x] `uv run alembic current` zeigt die zwei INFO-Zeilen und **keine** Versionsangabe


```bash
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ uv run ruff check alembic/env.py
All checks passed!
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ uv run alembic current
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
```


---

# Block 5 — Eine leere Datenbank besorgen

**Das Problem:** Die Baseline muss das **gesamte** Schema enthalten. `--autogenerate` erzeugt aber nur die *Unterschiede* zwischen Modell und Datenbank. Steht das Schema schon in der Datenbank, käme eine **leere** Migration heraus.

> [!success] Für dich erledigt — deine Datenbank ist bereits leer
> Block 0 hat `Did not find any tables.` ausgegeben. **Damit entfällt dieser ganze Block**, und mit ihm das `DB_NAME=`-Präfix in allen folgenden Befehlen. Du arbeitest durchgehend auf `pricing`.
>
> **Warum sie leer ist, obwohl das Projekt eine Kopie von `REST_API` ist:** Docker Compose leitet den Projektnamen aus dem **Verzeichnisnamen** ab, und Volumes gehören zum Projekt. `DB_Migration_Tools` ist ein anderer Projektname als `REST_API` → es entstand ein **neues, leeres Volume** (`pricing_data`), nicht das alte. Der `rsync`-Kopiertrick aus 1.5 kopiert eben nur Dateien, keine Docker-Volumes.
>
> Das ist ein Glücksfall für dieses Kapitel: Du bekommst die Baseline geschenkt und musst nichts umschalten. **Springe direkt zu Block 6.**

<details><summary><b>Falls die Datenbank doch Tabellen enthält</b> — der Weg über eine zweite Datenbank (auch später als Übung lesenswert)</summary>

**Die schlechte Lösung** wäre, die Entwicklungsdatenbank zu leeren. **Die gute Lösung** ist eine zweite, leere Datenbank im selben Container:

```bash
docker compose exec db createdb -U pricing pricing_baseline
docker compose exec db psql -U pricing -d pricing_baseline -c '\dt'
```

Die zweite Zeile muss `Did not find any relations.` sagen.

**Warum das elegant ist:** Deine Konfiguration liest `db_name` über `pydantic-settings` — und **Umgebungsvariablen haben dort Vorrang vor der `.env`-Datei**. Du kannst die Zieldatenbank also pro Befehl umschalten, ohne eine einzige Datei zu ändern:

```bash
DB_NAME=pricing_baseline uv run alembic current
```

In diesem Fall bekommt **jeder** Alembic-Befehl der Blöcke 6–8 dieses Präfix, und Block 9 (`stamp`) wird gebraucht.

**Warum eine zweite Datenbank besser ist als „Tabellen löschen und neu bauen":** Weil eine gewachsene Entwicklungsdatenbank Dinge enthält, die **nicht aus CSVs reproduzierbar** sind: `api_keys` und die Lauf-Historie in `pricing_runs`/`ingestion_runs`. Genau die Daten, die im [[Umsetzungsstand (Datenbank Migrations Tools)|Umsetzungsstand]] als Grund dafür stehen, dass „wegwerfen und neu befüllen" keine Migrationsstrategie mehr ist.

</details>

- [x] Bestätigt: Datenbank ist leer → Block übersprungen, **kein `DB_NAME`-Präfix nötig**

---

# Block 6 — Die Baseline erzeugen

**Was du tust:**

```bash
uv run alembic revision --autogenerate -m "baseline: bestehendes schema"
```

*(Falls deine Datenbank laut Block 5 doch Tabellen hatte: `DB_NAME=pricing_baseline` davorsetzen — und dann bei jedem Alembic-Befehl bis Block 8.)*

**Prüfung:** Die Ausgabe listet neun erkannte Tabellen:

```
INFO  [alembic.autogenerate.compare] Detected added table 'products'
INFO  [alembic.autogenerate.compare] Detected added table 'customers'
...
Generating .../alembic/versions/<hash>_baseline_bestehendes_schema.py ... done
```

Und in `alembic/versions/` liegt genau **eine** Datei.

> [!warning] Wenn stattdessen „No changes in schema detected" kommt
> Dann hat Alembic gegen eine Datenbank verglichen, die das Schema schon hat. Zwei mögliche Ursachen: Du hast versehentlich schon einmal `upgrade` laufen lassen, oder `target_metadata` in `env.py` steht noch auf `None` (dann hält Alembic *beide* Seiten für leer). Prüfen mit `docker compose exec db psql -U pricing -d pricing -c '\dt'` und einem Blick in `env.py`.

- [x] Baseline-Migration erzeugt
- [x] Alle 9 Tabellen sind in der Ausgabe genannt

---

# Block 7 — Die generierte Datei lesen

**Der wichtigste Block der ganzen Anleitung.** Autogenerate liefert einen *Entwurf*. Jetzt prüfst du ihn — und lernst dabei mehr über dein eigenes Schema als in den fünf Blöcken davor.

Öffne `alembic/versions/<hash>_baseline_bestehendes_schema.py` und arbeite diese Checkliste ab:

### 7a — Die Reihenfolge der Tabellen

`products` und `customers` müssen **vor** `capacity`, `bookings` und `prices` angelegt werden, weil dort Fremdschlüssel darauf zeigen. `price_recommendations` kommt nach `capacity`.

- [x] Reihenfolge stimmt (Alembic sortiert normalerweise korrekt nach FK-Abhängigkeit)

### 7b — `op.f()` um die Constraint-Namen

Du wirst so etwas sehen:

```python
sa.CheckConstraint("nights >= 1", name=op.f("ck_bookings_nights_positive")),
```

**Was `op.f()` bedeutet:** „Dieser Name ist bereits endgültig — wende die Naming Convention **nicht noch einmal** darauf an." Ohne diese Markierung würde aus `ck_bookings_nights_positive` beim Ausführen `ck_bookings_ck_bookings_nights_positive`. Das ist genau die Schnittstelle zwischen deiner `NAMING_CONVENTION` aus `orm.py` und Alembic.

- [x] `op.f(...)` taucht um Constraint- und Index-Namen auf
- [x] Die Namen entsprechen deiner Convention (`ix_`, `uq_`, `ck_`, `fk_`, `pk_`)

### 7c — `Identity(always=True)`

In den **sieben** Tabellen mit Autowert muss stehen:

```python
sa.Column("id", sa.BigInteger(), sa.Identity(always=True, start=1, increment=1), nullable=False)
```

**Worauf es ankommt:** `always=True` muss da sein. Es verbietet das manuelle Setzen von IDs — verschwände es, wäre das eine stille Schemaänderung. (Dass `start=1, increment=1` ergänzt werden, ist normal: Alembic schreibt die Vorgabewerte explizit hin.)

- [x] `always=True` ist in allen sieben Tabellen erhalten

### 7d — Die `Computed`-Spalte

In `bookings`:

```python
sa.Column("net_revenue", sa.Numeric(precision=12, scale=2),
          sa.Computed("gross_revenue - discount_amount", persisted=True), nullable=True)
```

- [x] `sa.Computed(...)` ist da, mit `persisted=True`

### 7e — Die `server_default`-Werte

| Spalte | Erwartet |
|---|---|
| `bookings.base_currency` | `server_default=sa.text("'STG'")` |
| `api_keys.created_at` | `server_default=sa.text("now()")` |
| `price_recommendations.computed_at` | `server_default=sa.text("now()")` |

- [x] Alle drei sind vorhanden

### 7f — Die Typen

Dein `type_annotation_map` bildet `str` auf `Text` ab (nicht `VARCHAR`) und `datetime` auf `DateTime(timezone=True)`.

- [x] Textspalten sind `sa.Text()`, **nicht** `sa.String()` oder `sa.VARCHAR(...)`
- [x] Zeitstempel sind `sa.DateTime(timezone=True)`

### 7g — Das `downgrade()`

Ganz unten muss ein vollständiges `downgrade()` stehen, das alle Tabellen wieder löscht — in **umgekehrter** Reihenfolge.

- [x] `downgrade()` ist vollständig und nicht `pass`

> [!warning] Wenn hier etwas abweicht
> Die Punkte 7c bis 7f sind genau die Stellen, die ich in der [[1.6 Datenbank Migrations Tools - Vertiefung#12. Was das konkret für dieses Projekt heißt|Vertiefung]] als „anspruchsvoller als ein Lehrbuchbeispiel" markiert habe. **Wenn eine davon fehlt oder anders aussieht: nicht selbst reparieren, sondern melden.** Die Korrektur gehört in die Migration, und dann will ich sie mit dir zusammen ansehen — das ist der Moment, in dem „Autogenerate erspart dir das Tippen, nicht das Denken" konkret wird.

- [x] Migrationsdatei vollständig durchgelesen und alle Punkte abgehakt
- [x] `uv run ruff check alembic/` ist sauber (der post-write-Hook sollte das erledigt haben)

---

# Block 8 — Aus dem Nichts aufbauen und beweisen, dass es stimmt

**Was du tust:**

```bash
uv run alembic upgrade head
```

Erwartete Ausgabe: `Running upgrade  -> <hash>, baseline: bestehendes schema`

Dann drei Prüfungen — die dritte ist die eigentliche:

```bash
# 1. Sind die Tabellen da?
docker compose exec db psql -U pricing -d pricing -c '\dt'

# 2. Kennt die Datenbank ihre Version?
docker compose exec db psql -U pricing -d pricing -c 'SELECT * FROM alembic_version;'

# 3. DER BEWEIS: Stimmt das Ergebnis mit den Modellen überein?
uv run alembic check
```

**Prüfung 3 muss sagen:** `No new upgrade operations detected.`

**Warum das der Beweis ist:** `alembic check` vergleicht dein `orm.py` gegen die gerade aus der Migration gebaute Datenbank. Kommt kein Unterschied heraus, dann erzeugt die Migrationskette **dasselbe Schema wie `create_all()`**. Genau das ist die Zusage, die eine Baseline machen muss.

Zusätzlich lohnt ein Blick auf die Tabelle mit den meisten Besonderheiten:

```bash
docker compose exec db psql -U pricing -d pricing -c '\d+ bookings'
```

Darin siehst du auf einen Schlag, ob die drei kritischen Dinge aus Block 7 in der **echten** Datenbank angekommen sind: die generierte Spalte (`net_revenue … generated always as (…) stored`), der Default (`base_currency … 'STG'::text`) und die Check-Constraints mit ihren Convention-Namen (`ck_bookings_status_valid`, `ck_bookings_nights_positive`).

> [!warning] Wenn hier etwas fehlt oder anders aussieht
> Das ist der Punkt, an dem sich ein Fehler in der Baseline zum ersten Mal in der Datenbank zeigt. **Zeig mir die Ausgabe, bevor du weitermachst** — eine falsche Baseline zu korrigieren ist jetzt trivial (Migration löschen, Tabellen droppen, neu erzeugen) und wird mit jedem weiteren Block teurer.

> [!question]- Halt kurz an: Warum ist `alembic check` hier aussagekräftiger als `\dt`?
> `\dt` zeigt nur, **dass** Tabellen existieren. `alembic check` vergleicht **Spalten, Typen, Nullbarkeit, Indizes, Fremdschlüssel und benannte CHECK-Constraints** gegen dein Modell. Es beantwortet die Frage „ist das Schema *richtig*?" statt „ist überhaupt etwas da?".
>
> Und die Grenze aus der Theorie gilt weiter: `check` sieht nur, was Autogenerate sehen kann. Ein geänderter **Ausdruck** in einer CHECK-Constraint bliebe unsichtbar — das probierst du in Block 11 selbst aus.

- [x] `upgrade head` gelaufen
- [x] `\dt` zeigt 9 Tabellen **plus `alembic_version`**
- [x] `alembic_version` enthält genau eine Zeile mit dem Hash deiner Migration
- [x] **`alembic check` sagt „No new upgrade operations detected."**
- [x] `\d+ bookings` zeigt `Computed`, `server_default` und die Check-Constraints

> [!success] Ab hier funktioniert die Anwendung wieder vollständig
> Deine Datenbank hat jetzt dasselbe Schema wie vorher `pricing-initdb` — nur mit `alembic_version` dazu. Wenn du willst, kannst du den Rest des Onboarding-Ablaufs sofort ausprobieren:
> ```bash
> uv run pricing-apikey test
> uv run uvicorn pricing.presentation.api.app:app --reload
> ```
> Und über `/docs` testen wie gewohnt. **Das ist der gute Punkt, um die erste Sitzung zu beenden** — die Baseline steht und ist bewiesen.


```bash
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ uv run alembic upgrade head
Using CPython 3.13.5 interpreter at: /usr/bin/python3.13
Removed virtual environment at: .venv
Creating virtual environment at: .venv
Installed 44 packages in 28ms
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 7f94db1e1511, baseline: bestehendes schema
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ docker compose exec db psql -U pricing -d pricing -c "\dt"
                  List of tables
 Schema |         Name          | Type  |  Owner  
--------+-----------------------+-------+---------
 public | alembic_version       | table | pricing
 public | api_keys              | table | pricing
 public | bookings              | table | pricing
 public | capacity              | table | pricing
 public | customers             | table | pricing
 public | ingestion_runs        | table | pricing
 public | price_recommendations | table | pricing
 public | prices                | table | pricing
 public | pricing_runs          | table | pricing
 public | products              | table | pricing
(10 rows)

(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ docker compose exec db psql -U pricing -d pricing -c 'SELECT * FROM alembic_version;'
 version_num  
--------------
 7f94db1e1511
(1 row)

(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ uv run alembic check
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.schemas
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.tables
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.types
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.constraints
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.defaults
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.comments
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.checkconstraint_byname
No new upgrade operations detected.
(pricing) nick@thinkpad:~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools$ docker compose exec db psql -U pricing -d pricing -c '\d+ bookings'
```

---

# Block 9 — Die echte Datenbank stempeln

> [!success] Für dich entfällt dieser Block
> Deine Datenbank war leer, du hast in Block 8 direkt `upgrade head` gemacht — es gibt nichts zu stempeln. **Lies den Block trotzdem einmal durch**: `stamp` ist der Befehl, mit dem man in der Praxis eine gewachsene Datenbank in ein Migrationssystem holt, und die Kontrollfrage dazu kommt garantiert wieder.
>
> **Wenn du es üben willst** (10 Minuten, keine Auswirkung auf deine Arbeit): Leg eine Wegwerf-Datenbank an, bau darin das Schema mit `create_all()` — solange `pricing-initdb` noch existiert, also **vor Block 10** — und stemple sie:
> ```bash
> docker compose exec db createdb -U pricing pricing_stampuebung
> DB_NAME=pricing_stampuebung uv run pricing-initdb        # Schema ohne alembic_version
> DB_NAME=pricing_stampuebung uv run alembic current       # leer -> DB kennt ihren Stand nicht
> DB_NAME=pricing_stampuebung uv run alembic upgrade head  # SCHEITERT: DuplicateTable
> DB_NAME=pricing_stampuebung uv run alembic stamp head    # so ist es richtig
> DB_NAME=pricing_stampuebung uv run alembic current       # jetzt steht der Hash da
> docker compose exec db dropdb -U pricing pricing_stampuebung
> ```
> Der absichtlich scheiternde vierte Befehl ist der Lernwert: Du siehst **einmal selbst**, warum es `stamp` gibt.

**Was du tust:**

```bash
uv run alembic stamp head          # ohne DB_NAME -> die echte pricing-DB
uv run alembic current
```

**Warum `stamp` und nicht `upgrade`:** Die Tabellen existieren dort bereits (per `create_all()` gebaut). Ein `upgrade` würde versuchen, sie **noch einmal** anzulegen, und mit `DuplicateTable` scheitern. `stamp` schreibt nur die Versionsnummer in `alembic_version` und führt **nichts** aus — die Aussage ist: „Diese Datenbank ist bereits auf dem Stand dieser Migration."

**Prüfung:** `alembic current` zeigt jetzt deinen Baseline-Hash mit dem Zusatz `(head)`.

```bash
uv run alembic check     # sollte ebenfalls sauber sein
```

> [!tip] Rückweg — `stamp` ist die harmloseste Operation von allen
> Falls sich die Baseline später doch als falsch herausstellt: `stamp` schreibt **ausschließlich** in `alembic_version` und fasst deine Tabellen nicht an. Rücknahme:
> ```bash
> uv run alembic stamp base      # Versionseintrag wieder entfernen
> ```
> Danach ist die Datenbank exakt in dem Zustand von vor diesem Block. Du kannst die Baseline korrigieren und erneut stempeln, so oft du willst.

- [x] `alembic stamp head` auf der echten Datenbank gelaufen
- [x] `alembic current` zeigt den Baseline-Hash
- [x] `alembic check` gegen die echte Datenbank ist sauber

---

# Block 10 — `pricing-initdb` ablösen

Jetzt gibt es **zwei** Wege, ein Schema zu erzeugen: `create_all()` und `upgrade head`. Genau das ist der Zustand, den dieses Kapitel beseitigen soll — zwei Wege, die auseinanderlaufen können, ohne dass es jemand merkt.

**Meine Empfehlung: `create_all()` entfernen, das Kommando `pricing-initdb` abschaffen.**

Begründung: Der Docstring in `schema_admin.py` argumentiert, Schemaverwaltung gehöre in ein eigenes Kommando statt in den Anwendungsstart. Dieses Argument bleibt vollständig gültig — nur ist das Kommando ab jetzt `alembic upgrade head`. Ein zweites, schwächeres Kommando daneben stiftet nur Verwirrung: Wer `pricing-initdb` benutzt, bekommt ein Schema **ohne** `alembic_version` und damit eine Datenbank, die ihren Stand nicht kennt.

**Was du tust:**

1. `src/pricing/persistence/schema_admin.py` **löschen**
2. In `pyproject.toml` die Zeile `pricing-initdb = "pricing.persistence.schema_admin:main"` **entfernen**
3. In `scripts/smoke_orm.py` Zeile 50 anpassen — der Hinweistext nennt noch den alten Befehl:
   ```python
   "Schema anlegen:  uv run alembic upgrade head",
   ```
4. `uv sync` (damit der Entry Point aus der venv verschwindet)

**Prüfung:**

```bash
uv sync
uv run pricing-initdb        # muss jetzt "command not found" bzw. einen Fehler geben
uv run ruff check src scripts tests
uv run mypy --strict src scripts tests
uv run pytest
```

> [!question]- Halt kurz an: Warum reicht es nicht, `pricing-initdb` einfach liegen zu lassen?
> Weil es keinen Fehler produziert, sondern etwas **fast Richtiges** tut. Jemand (auch du, in vier Monaten) ruft es auf, bekommt neun Tabellen und keine `alembic_version` — und beim nächsten `alembic upgrade head` scheitert alles mit `DuplicateTable`, ohne dass der Zusammenhang erkennbar wäre.
>
> Dasselbe Muster wie beim `.venv`-Problem aus 1.5: Der gefährliche Zustand ist nicht der kaputte, sondern der **fast funktionierende**.

- [x] `schema_admin.py` gelöscht
- [x] Entry Point aus `pyproject.toml` entfernt
- [x] `smoke_orm.py` nennt den neuen Befehl
- [x] `uv sync` gelaufen, `pricing-initdb` existiert nicht mehr
- [x] ruff, mypy und alle 123 Tests weiterhin grün

---

# Block 11 — Die erste echte Migration (und ein Beweis für den blinden Fleck)

Bis hier hast du nur **abgebildet**, was schon da war. Jetzt änderst du zum ersten Mal etwas.

*(Alle Befehle wie gehabt ohne `DB_NAME`-Präfix — du bist die ganze Zeit auf `pricing`. Nur falls du die optionale `stamp`-Übung aus Block 9 gemacht hast: Achte darauf, dass dort kein Präfix hängengeblieben ist.)*

### 11a — Eine Spalte hinzufügen (der Normalfall)

Ergänze in `src/pricing/persistence/orm.py` in der Klasse `Booking` eine Spalte:

```python
    channel: Mapped[str | None] = mapped_column(Text)
```

Dann:

```bash
uv run alembic revision --autogenerate -m "channel-spalte fuer buchungen"
```

Lies die erzeugte Datei — sie sollte genau zwei Zeilen enthalten (`op.add_column` / `op.drop_column`). Dann:

```bash
uv run alembic upgrade head
docker compose exec db psql -U pricing -d pricing -c '\d bookings' | grep channel
```

### 11b — Hoch, runter, wieder hoch

```bash
uv run alembic downgrade -1
docker compose exec db psql -U pricing -d pricing -c '\d bookings' | grep channel   # jetzt leer
uv run alembic upgrade head
```

**Warum dieser Dreischritt:** Er ist der Test, den Liquibase als `update-testing-rollback` eingebaut hat und den Alembic **nicht** hat. Du machst ihn hier von Hand — und weißt danach, dass dein `downgrade()` nicht nur dasteht, sondern funktioniert.

### 11c — Der blinde Fleck (die eigentliche Übung)

> [!warning] `status_valid` kommt in `orm.py` **dreimal** vor
> Eine Textsuche nach `status_valid` findet drei Treffer, und alle drei heißen gleich:
>
> | Zeile | Klasse |
> |---|---|
> | **73** | **`Booking`** ← **nur diese ändern** |
> | 150 | `PricingRun` |
> | 168 | `IngestionRun` |
>
> Die Naming Convention macht daraus zwar drei verschiedene Namen (`ck_bookings_status_valid`, `ck_pricing_runs_status_valid`, `ck_ingestion_runs_status_valid`) — im Quelltext heißen sie aber identisch. **Orientiere dich an `class Booking`, nicht an der Textsuche.**
>
> Und das Perfide daran, passend zum Thema dieses Blocks: Erwischst du die falsche Klasse, **bleibt `alembic check` genauso grün**. Der blinde Fleck würde deinen Fehler decken.

Ändere in `orm.py` die CHECK-Constraint von `Booking` (Zeile 72–74) — füge einen erlaubten Status hinzu:

```python
        CheckConstraint(
            "status IN ('Firm', 'Cancelled', 'Expired', 'Pending', 'Refunded')",
            name="status_valid",
        ),
```

Und dann:

```bash
uv run alembic check
```

**Erwartung: `No new upgrade operations detected.`** — obwohl du das Schema gerade geändert hast.

**Warum:** Die CHECK-Erkennung ist **rein namensbasiert**. Der Name `status_valid` ist unverändert, also gilt die Constraint für Alembic als gleich, *„regardless of any difference in their expression text"*. Kein Fehler, keine Warnung, grüner Check — und eine Datenbank, die den neuen Status weiterhin ablehnt.

Die Korrektur ist eine **handgeschriebene** Migration:

```bash
uv run alembic revision -m "status_valid um Refunded erweitern"      # OHNE --autogenerate
```

Die erzeugte Datei hat einen Kopfbereich (Docstring, `revision`, `down_revision`, `branch_labels`, `depends_on`, Imports) und darunter zwei leere Funktionen mit `pass`. **Den Kopf lässt du unverändert** — du ersetzt nur die beiden Funktionskörper:

```python
def upgrade() -> None:
    op.drop_constraint(op.f("ck_bookings_status_valid"), "bookings", type_="check")
    op.create_check_constraint(
        "status_valid", "bookings",
        "status IN ('Firm', 'Cancelled', 'Expired', 'Pending', 'Refunded')",
    )


def downgrade() -> None:
    op.drop_constraint(op.f("ck_bookings_status_valid"), "bookings", type_="check")
    op.create_check_constraint(
        "status_valid", "bookings",
        "status IN ('Firm', 'Cancelled', 'Expired', 'Pending')",
    )
```

> [!bug] Das `op.f()` beim Löschen ist **nicht optional** — hier schlägt Block 7b zurück
> Ohne `op.f()` scheitert die Migration mit einer Fehlermeldung, die zunächst unsinnig aussieht:
> ```
> psycopg.errors.UndefinedObject: constraint "ck_bookings_ck_bookings_status_valid"
>   of relation "bookings" does not exist
> [SQL: ALTER TABLE bookings DROP CONSTRAINT ck_bookings_ck_bookings_status_valid]
> ```
> **Der Name wurde verdoppelt.** Genau der Mechanismus aus [[#7b — `op.f()` um die Constraint-Namen|Block 7b]], nur in die andere Richtung: Deine `ck`-Convention lautet `ck_%(table_name)s_%(constraint_name)s`. Weil sie den Platzhalter `constraint_name` enthält, greift sie auf **jeden** Namen, den du übergibst — auch auf einen, der schon vollständig ist. Aus `ck_bookings_status_valid` wird `ck_bookings_` + `ck_bookings_status_valid`.
>
> **Zwei richtige Schreibweisen:**
>
> | Variante | Warum sie funktioniert |
> |---|---|
> | `op.drop_constraint(op.f("ck_bookings_status_valid"), …)` | `op.f()` markiert den Namen als **endgültig** — die Convention lässt ihn in Ruhe. Das ist die Form, die Autogenerate selbst erzeugt, und sie zeigt den echten Namen aus der Datenbank |
> | `op.drop_constraint("status_valid", …)` | Der **kurze** Name; die Convention baut daraus `ck_bookings_status_valid` — symmetrisch zu `create_check_constraint` |
>
> **Falsch ist nur die Mischform**: voller Name *ohne* `op.f()`. Empfehlung: die erste Variante, weil dort in der Migration steht, was tatsächlich in der Datenbank passiert.
>
> **Merksatz:** Sobald eine Naming Convention den Platzhalter `%(constraint_name)s` enthält, gilt für Constraints dieses Typs: **entweder kurzer Name oder `op.f()` — nie der volle Name nackt.** Für `ix`/`uq`/`fk`/`pk` in deinem Schema stellt sich die Frage nicht, weil deren Conventions diesen Platzhalter nicht haben.

> [!note] Warum beim Anlegen der kurze Name steht und beim Löschen `op.f()`
> Beide Aufrufe müssen denselben Namen erzeugen — `ck_bookings_status_valid` —, kommen aber aus verschiedenen Richtungen dorthin:
>
> - **`create_check_constraint("status_valid", …)`** übergibt den kurzen Namen und **lässt die Convention arbeiten**: `ck_` + Tabelle + kurzer Name.
> - **`drop_constraint(op.f("ck_bookings_status_valid"), …)`** übergibt den fertigen Namen und **hält die Convention davon ab**, ihn ein zweites Mal zu bearbeiten.
>
> Wer stattdessen den kurzen Namen auch beim Löschen nimmt, bekommt dasselbe Ergebnis — nur der volle Name **ohne** `op.f()` ist falsch (siehe Kasten oben).

```bash
uv run alembic upgrade head
```

> [!warning] Das `downgrade()` hier ist eine kleine Lüge
> Es setzt die alte Constraint wieder — was **scheitert**, sobald auch nur eine Zeile inzwischen `'Refunded'` enthält. Genau die Asymmetrie aus [[1.6 Datenbank Migrations Tools - Vertiefung#5.2 Warum `downgrade()` oft eine höfliche Lüge ist|Stichwort 1, Abschnitt 5.2]]: Struktur umkehrbar, Daten nicht. Eine ehrliche Fassung würde vorher die betroffenen Zeilen bereinigen — schreib das als Kommentar dazu.

**Zum Schluss aufräumen:** Ob `channel` und `'Refunded'` dauerhaft bleiben sollen, entscheidest du. Sind sie nur Übung, nimmst du sie mit einer weiteren Migration wieder zurück (**nicht** durch Löschen der Migrationsdateien — die sind schon gelaufen).

- [x] `channel`-Spalte per Autogenerate hinzugefügt und angewandt
- [x] `downgrade -1` → `upgrade head` erfolgreich durchgespielt
- [x] **Den blinden Fleck bei CHECK-Constraints selbst nachgewiesen** (`alembic check` blieb grün)
- [x] Handgeschriebene Migration für die Constraint-Änderung geschrieben und angewandt
- [x] Entschieden, was von den Übungsänderungen bleibt

---

# Block 12 — Prüfkette, Aufräumen, Commit

### 12a — `alembic check` in die Prüfkette

Deine bisherige Kette:

```bash
uv run ruff check src scripts tests
uv run mypy --strict src scripts tests
uv run pytest
```

Neu dazu:

```bash
uv run ruff check src scripts tests alembic
uv run alembic check
```

**`alembic` bei ruff dazunehmen: ja.** Migrationsdateien sind Code in deinem Repository, und der post-write-Hook hält sie ohnehin sauber.

**`alembic` bei mypy dazunehmen: probier es aus — es spricht mehr dafür als dagegen.**

```bash
uv run mypy --strict src scripts tests alembic
```

Entgegen dem, was man oft liest, bestehen Alembic-Migrationen `--strict` in aller Regel: Alembic liefert seit einigen Versionen vollständige Typ-Stubs für `op` mit (`alembic/op.pyi`), und `script.py.mako` annotiert `upgrade()`/`downgrade()` bereits mit `-> None`. *(Ein Prüfer hat das für 1.19.1 mit einer Autogenerate- und einer Baseline-artigen Migration nachgestellt — beide sauber.)*

Der eigentliche Gewinn betrifft nicht die generierten Dateien, sondern die **handgeschriebenen** wie deine aus Block 11c — dort tippt man sich leicht bei einem Argumentnamen.

- **Läuft es sauber durch:** `files = ["src", "tests", "alembic"]` in `pyproject.toml` setzen.
- **Meldet es Fehler in generierten Dateien:** bei `files = ["src", "tests"]` bleiben und **einen Kommentar dazuschreiben**, warum — sonst hält es später jemand für ein Versehen. Eine passende Begründung: *Migrationen werden einmal geschrieben und nie wieder angefasst; der Nutzen einer Typprüfung ist dort gering.*

### 12b — Aufräumen

```bash
# Nur nötig, falls du in Block 5 oder 9 Wegwerf-Datenbanken angelegt hast:
docker compose exec db psql -U pricing -d pricing -c '\l' | grep pricing_
# und dann je Fund:  docker compose exec db dropdb -U pricing <name>
```

`.gitignore` braucht **nichts** — `**/__pycache__/` deckt `alembic/versions/__pycache__/` bereits ab.

### 12c — Was committet wird

- [x] `alembic.ini`, `alembic/env.py`, `alembic/script.py.mako`, `alembic/README`
- [x] **alle** Dateien in `alembic/versions/`
- [x] `pyproject.toml` + `uv.lock`
- [x] gelöschte `schema_admin.py`, angepasstes `smoke_orm.py`

```bash
git add -A
git commit -m "alembic eingefuehrt, schema unter versionskontrolle"
```

### 12d — Der Beweis, dass der neue Onboarding-Ablauf funktioniert

*(Vier Befehle, ~5 Minuten, kein neuer Stoff — aber der einzige Punkt, an dem die **komplette** Kette inklusive deiner handgeschriebenen Migration aus Block 11c geprüft wird. Block 8 hat nur die Baseline bewiesen.)*

Zum Abschluss einmal die Sicht eines neuen Kollegen einnehmen:

```bash
docker compose exec db createdb -U pricing pricing_probe
DB_NAME=pricing_probe uv run alembic upgrade head
DB_NAME=pricing_probe uv run alembic check          # muss sauber sein
docker compose exec db dropdb -U pricing pricing_probe
```

**Das ist die Zusage, die du ab heute geben kannst:** Aus einer leeren Datenbank entsteht mit *einem* Befehl das aktuelle Schema. Vorher war das nur für `create_all()` wahr — und dort ohne jede Möglichkeit, später etwas zu ändern.

- [x] Prüfkette erweitert, mypy-Entscheidung kommentiert
- [x] `pricing_baseline` gelöscht
- [x] Commit gesetzt
- [x] Aufbau aus dem Nichts ein letztes Mal verifiziert

---

# Block 13 — Nachziehen der Dokumentation

- [x] [[Umsetzungsstand (Datenbank Migrations Tools)|Umsetzungsstand]]: Abschnitt *Erledigt* füllen, offene Punkte abhaken, Ergebnisse aus Block 7 (was Autogenerate tatsächlich erzeugt hat) eintragen
- [x] Beantwortete Fragen aus [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Q&A/Weiterführende Stichpunkte|Q&A]] dort als beantwortet markieren (`env.py` und Engine, Template-Wahl, Constraint-Namen in der laufenden DB)
- [x] Neuen Onboarding-Ablauf dokumentieren

---

# Wenn etwas schiefgeht

| Symptom | Ursache | Lösung |
|---|---|---|
| `alembic: command not found` | Ohne `uv run` aufgerufen | Immer `uv run alembic …` |
| `ConfigError: Umgebungsvariable DB_PRICING_PW` | `.env` wird nicht gefunden | Aus dem **Projektwurzelverzeichnis** aufrufen, nicht aus `alembic/` |
| `Can't locate revision identified by '…'` | `alembic_version` nennt eine Revision, deren Datei fehlt | Meist nach Branch-Wechsel. Prüfen mit `alembic history`; notfalls `alembic stamp <vorhandener hash>` |
| `Target database is not up to date` | Es gibt ausstehende Migrationen | `alembic upgrade head` |
| `DuplicateTable: relation "products" already exists` | `upgrade` statt `stamp` auf einer bestehenden DB | Siehe Block 9 |
| `No changes in schema detected` bei der Baseline | Gegen die volle DB verglichen | `DB_NAME=pricing_baseline` davorsetzen |
| `Multiple head revisions are present` | Zwei Migrationen mit demselben Vorgänger | `alembic heads`, dann `alembic merge` |
| ruff meckert über `alembic/versions/*.py` | post-write-Hook nicht aktiv | Block 3b prüfen; einmalig `uv run ruff check --fix alembic` |
| `UndefinedObject: constraint "ck_bookings_ck_bookings_..." does not exist` | Voller Constraint-Name **ohne** `op.f()` — die Naming Convention hat ihn ein zweites Mal präfixiert | `op.f("ck_...")` schreiben oder den kurzen Namen; siehe Block 11c |
| Migration bricht mittendrin ab | — | **Kein Aufräumen nötig.** PostgreSQL kann transaktionales DDL: Die Migration wurde vollständig zurückgenommen, `alembic current` steht noch auf der vorherigen Revision. Datei korrigieren, `upgrade head` wiederholen |

**Grundregel bei allem, was mit der Datenbank hakt:** Erst `uv run alembic current` (wo stehe ich?), dann `uv run alembic history -v` (welche Kette gibt es?). Die beiden Befehle beantworten neun von zehn Fragen.
