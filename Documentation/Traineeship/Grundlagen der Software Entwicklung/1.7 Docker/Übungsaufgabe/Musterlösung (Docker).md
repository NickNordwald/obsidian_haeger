Referenzlösung zu [[Docker Plan & Syntax-Tutorial|Docker Plan & Syntax-Tutorial]]. Aufgekommene Fragen: [[Fragen aus der Umsetzung|Q&A/Fragen aus der Umsetzung]]. Theorie: [[1.7 Docker - Vertiefung|1.7 Docker – Vertiefung]]. Stand: [[Umsetzungsstand (Docker)|Umsetzungsstand (Docker)]].

> [!warning] Benutzungshinweis
> Diese Datei ist das **Nachschlagewerk**, nicht der Weg. Der Lerneffekt entsteht beim Selberschreiben und beim Lesen der Fehlermeldungen. Nimm sie, wenn du an einem Block länger als ~20 Minuten festhängst — und lies dann **die Begründung**, nicht nur den Code.
>
> Alles hier ist auf deinen tatsächlichen Projektstand geprüft: `src`-Layout mit `uv`, `pydantic-settings` mit `DB_HOST`/`DB_PRICING_PW`, FastAPI unter `pricing.presentation.api.app:app`, `/v1/health` ohne Schlüssel, Alembic mit `script_location = %(here)s/alembic`.

# Inhalt

- [[#Die drei Dateien am Stück]] — zum schnellen Abgleich
- [[#Block-für-Block-Erläuterung]] — warum es so aussieht
- [[#Antworten auf die Entscheidungsfragen]]
- [[#Fehlersuche]]
- [[#Der Abnahmetest]]

---

# Die drei Dateien am Stück

## `.dockerignore`

```gitignore
# Versionskontrolle
.git
.gitignore

# Virtuelle Umgebung -- wird im Image neu gebaut
.venv

# Caches
__pycache__/
**/__pycache__/
*.pyc
.mypy_cache/
.pytest_cache/
.ruff_cache/
.coverage

# Laufzeit-Artefakte des Hosts
logs/
daily_data/
price_recommendations.csv

# Konfiguration und Geheimnisse
.env
*.env

# Werkzeug- und Editor-Kram
.vscode/
.python-version

# Container-Dateien selbst
Dockerfile
.dockerignore
compose.yaml
docker-compose.yml

# Tests und Skripte gehoeren nicht ins Laufzeit-Image
tests/
scripts/
```

## `Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1

# ============================================================
# Stage 1: builder -- installiert Abhaengigkeiten und Projekt
# ============================================================
FROM python:3.13-slim AS builder

# uv als einzelne Binaerdatei aus dem offiziellen Image holen.
# Kein curl, kein pip, keine zusaetzliche Schicht mit Installer-Kram.
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never

WORKDIR /app

# --- Schritt 1: NUR die Abhaengigkeiten -------------------------
# Teuer, aendert sich selten. Haengt allein an pyproject.toml/uv.lock,
# nicht am Quellcode -> bleibt bei Code-Aenderungen im Cache.
COPY pyproject.toml uv.lock ./
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev --no-install-project --no-editable

# --- Schritt 2: das Projekt selbst ------------------------------
# Billig, aendert sich staendig -> steht deshalb weiter unten.
COPY src/ ./src/
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev --no-editable

# ============================================================
# Stage 2: runtime -- nur das, was zum Laufen noetig ist
# ============================================================
FROM python:3.13-slim AS runtime

# Unprivilegierter Benutzer. UID 1000 ist in Debian-slim frei.
RUN groupadd --gid 1000 app \
 && useradd --uid 1000 --gid 1000 --no-create-home app

# MUSS identisch zur Builder-Stage sein: im .venv stehen absolute Pfade.
WORKDIR /app

# Die fertige virtuelle Umgebung -- dank --no-editable liegt das
# pricing-Paket darin, src/ wird zur Laufzeit NICHT mehr gebraucht.
COPY --from=builder --chown=1000:1000 /app/.venv /app/.venv

# Alembic braucht sein Verzeichnis und seine ini zur Laufzeit.
COPY --chown=1000:1000 alembic.ini ./
COPY --chown=1000:1000 alembic/ ./alembic/

ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

USER 1000:1000

# Reine Dokumentation -- veroeffentlicht wird in compose.yaml.
EXPOSE 8000

# python statt curl: slim hat kein curl, und nachinstallieren
# waere ~15 MB fuer eine HTTP-Anfrage, die Python auch kann.
HEALTHCHECK --interval=10s --timeout=3s --start-period=20s --retries=3 \
  CMD ["python", "-c", \
       "import urllib.request as u, sys; \
        sys.exit(0 if u.urlopen('http://127.0.0.1:8000/v1/health', timeout=2).status == 200 else 1)"]

# exec-Form: uvicorn wird PID 1 und bekommt SIGTERM direkt.
# --host 0.0.0.0, sonst lauscht der Server nur im Container-Loopback.
CMD ["uvicorn", "pricing.presentation.api.app:app", \
     "--host", "0.0.0.0", "--port", "8000"]
```

## `compose.yaml`

```yaml
# Projektname explizit -- sonst leitet Compose ihn vom Verzeichnisnamen ab,
# und ein Ordnerwechsel erzeugt neue, leere Volumes (Erfahrung aus 1.6).
name: pricing

# YAML-Anker: einmal definieren, dreimal verwenden.
x-app-env: &app-env
  DB_HOST: db                # der Dienstname, NICHT localhost
  DB_NAME: pricing
  DB_USER: pricing
  DB_PRICING_PW: pricing
  LOG_LEVEL: INFO
  # LOG_FILE bewusst NICHT gesetzt: ohne Datei-Handler geht alles
  # nach stderr und damit in `docker compose logs`.

services:
  # ---------------------------------------------------------
  db:
    image: postgres:18
    environment:
      POSTGRES_DB: pricing
      POSTGRES_USER: pricing
      POSTGRES_PASSWORD: pricing
    ports:
      - "127.0.0.1:5432:5432"      # nur lokal erreichbar
    volumes:
      - pricing_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U pricing -d pricing"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s            # Karenzzeit fuer die Erstinitialisierung
    restart: unless-stopped

  # ---------------------------------------------------------
  # Einmal-Dienst: migriert und beendet sich. Die API wartet darauf.
  migrate:
    build:
      context: .
    image: pricing-api:latest      # gleiches Image wie api -> zweiter Build ist Cache-Treffer
    command: ["alembic", "upgrade", "head"]
    environment: *app-env
    depends_on:
      db:
        condition: service_healthy
    restart: "no"                  # ein Einmal-Dienst darf sich NICHT neu starten

  # ---------------------------------------------------------
  api:
    build:
      context: .
    image: pricing-api:latest
    environment: *app-env
    ports:
      - "127.0.0.1:8000:8000"
    depends_on:
      migrate:
        condition: service_completed_successfully
    restart: unless-stopped
    # Optional, falls die Ingest-Endpunkte CSVs vom Host lesen sollen:
    # volumes:
    #   - ./daily_data:/app/daily_data:ro

  # ---------------------------------------------------------
  # Entwicklungswerkzeug -- startet nur mit `--profile dev`.
  adminer:
    image: adminer
    profiles: ["dev"]
    environment:
      ADMINER_DEFAULT_SERVER: db
    ports:
      - "127.0.0.1:8080:8080"
    depends_on:
      db:
        condition: service_healthy

volumes:
  pricing_data:
```

> [!note] Dateiname
> Wenn du bei `docker-compose.yml` bleibst, funktioniert alles unverändert — `compose.yaml` ist nur der heute bevorzugte Name. Umbenennen ist ein `git mv`.

---

# Block-für-Block-Erläuterung

## Block 1 — `.dockerignore`

**Warum `.git` ganz oben steht:** Es ist bei einem Projekt mit Historie oft der größte Einzelposten und für das Image komplett nutzlos. `.gitignore` kennt es naturgemäß nicht — deshalb reicht „Inhalt von `.gitignore` kopieren" nicht aus.

**Warum `Dockerfile` und `compose.yaml` selbst drinstehen:** Sie werden nicht ins Image kopiert (`COPY` holt sie nicht), aber jede Änderung an ihnen würde sonst den Kontext-Hash verändern und Cache-Schichten unnötig invalidieren.

**Warum `tests/` und `scripts/` draußen bleiben:** Sie gehören nicht in ein Laufzeit-Image — weniger Angriffsfläche, kleineres Artefakt. Wenn du später eine Test-Stage baust (`--target test`), kopierst du sie dort gezielt hinein; dann müsste die Zeile hier weg oder die Stage andere Pfade nutzen.

**Erwartetes Ergebnis:** Kontextgröße rund **2 MB**. Ohne die Datei wären es **476 MB** — `.venv` 181 MB, `daily_data` 139 MB, `.git` 65 MB, dazu Caches und `price_recommendations.csv`. Die `.dockerignore` filtert also über 99 % weg.

## Block 2 — Startbefehl

```bash
uv run uvicorn pricing.presentation.api.app:app --host 127.0.0.1 --port 8000
```

Der Importpfad ist `pricing.presentation.api.app:app` — Modulpfad, dann Doppelpunkt, dann der Name des `FastAPI`-Objekts (in `app.py` Zeile 88: `app = FastAPI(...)`).

**Pflicht-Umgebungsvariablen:** Nur `DB_PRICING_PW` ist hart erforderlich — ohne sie wirft `get_engine()` einen `ConfigError`. `DB_HOST`, `DB_NAME`, `DB_USER` haben Defaults, aber `DB_HOST` muss im Container zwingend überschrieben werden.

## Block 3 — Minimales Dockerfile

Die Zwischenstufe, die es später nicht mehr gibt. Nur zur Vollständigkeit:

```dockerfile
FROM python:3.13-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
WORKDIR /app
COPY . .
RUN uv sync --frozen
CMD ["uv", "run", "uvicorn", "pricing.presentation.api.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Warum `uv run` hier und später nicht mehr:** `uv run` aktiviert die Umgebung selbst. Sobald `PATH` auf `/app/.venv/bin` zeigt (Block 7), ist das überflüssig — und `uvicorn` direkt aufzurufen spart einen Zwischenprozess, was für die PID-1-Signalweitergabe sauberer ist.

**Kein `apt-get` nötig:** Deine `pyproject.toml` fordert `psycopg[binary]`. Das Binary-Wheel bringt `libpq` mit — genau der Punkt aus Stichwort 1 der Vertiefung. Bei `psycopg` ohne `[binary]` bräuchtest du hier `libpq5` und im Builder `libpq-dev` plus `gcc`.

## Blöcke 5 + 6 — Caching und Cache-Mount

Der entscheidende Trick ist das **zweistufige `uv sync`**:

```dockerfile
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project --no-editable   # nur Fremdpakete
COPY src/ ./src/
RUN uv sync --frozen --no-dev --no-editable                        # jetzt auch pricing
```

`--no-install-project` überspringt dein eigenes Paket. Damit hängt der teure Schritt (SQLAlchemy, FastAPI, psycopg, …) **nur an der Lockfile**. Änderst du `main.py`, wird ab `COPY src/` neu gebaut — und das dauert Sekunden statt Minuten.

Die Flags im Einzelnen:

| Flag | Wirkung |
|---|---|
| `--frozen` | Lockfile exakt befolgen, nicht aktualisieren. In Images immer. |
| `--no-dev` | pytest, ruff, mypy, httpx2 weglassen |
| `--no-install-project` | nur Abhängigkeiten, nicht `pricing` selbst |
| `--no-editable` | `pricing` **kopieren** statt verlinken (dazu unten mehr) |
| `UV_COMPILE_BYTECODE=1` | `.pyc` beim Bauen erzeugen → schnellerer erster Request |
| `UV_LINK_MODE=copy` | nötig bei Cache-Mounts, weil Hardlinks über Mount-Grenzen nicht gehen |
| `UV_PYTHON_DOWNLOADS=never` | uv soll das Python des Images nehmen, kein eigenes laden |

## Block 7 — Multi-Stage

**Was herüberkopiert wird und was nicht:**

| Pfad | Runtime? | Warum |
|---|---|---|
| `/app/.venv` | ✅ | enthält alle Abhängigkeiten **und** `pricing` |
| `alembic/`, `alembic.ini` | ✅ | Migrationen müssen zur Laufzeit ausführbar sein |
| `src/` | ❌ | dank `--no-editable` liegt der Code in `site-packages` |
| `uv`, Cache, `pyproject.toml` | ❌ | reine Bauwerkzeuge |

> [!important] Die zwei Fallen bei Multi-Stage mit venv
> **1. `WORKDIR` muss in beiden Stages identisch sein.** In `.venv/pyvenv.cfg` und in den Shebangs der Skripte stehen **absolute Pfade**. Baust du in `/build` und kopierst nach `/app`, zeigt alles ins Leere.
>
> **2. Das Basis-Image muss in beiden Stages dasselbe sein.** Das `.venv` verweist auf den Interpreter unter `/usr/local/bin/python3.13`. Ein anderes Runtime-Image (etwa `debian:13-slim` ohne Python) ergibt ein `.venv`, dessen Interpreter fehlt.

**Zu `--no-editable`:** Ohne dieses Flag installiert `uv sync` dein Projekt als *editable* — im `.venv` liegt dann nur ein Verweis auf `/app/src`. Das funktioniert auch, **aber dann musst du `src/` mit in die Runtime-Stage kopieren**. `--no-editable` baut stattdessen ein Wheel und kopiert den Code ins `site-packages`. Ein Ding weniger, das man vergessen kann.

**Erwartete Größen:** einstufig ~450–550 MB, Multi-Stage ~230–280 MB. Die genauen Werte gehören in den Umsetzungsstand.

## Blöcke 8–10 — Compose

**Warum beide Dienste `build:` **und** dasselbe `image:` haben:** So baut Compose das Image genau einmal wirklich; der zweite Durchlauf ist ein vollständiger Cache-Treffer und dauert Millisekunden. Die Alternative — nur `api` baut, `migrate` referenziert nur `image:` — funktioniert meistens auch, kann aber beim allerersten `up` scheitern, wenn Compose das noch nicht existierende Image zu ziehen versucht.

**Warum `restart: "no"` bei `migrate`:** Ein Einmal-Dienst, der sich neu startet, ist eine Endlosschleife. Und `service_completed_successfully` würde nie eintreten.

**Warum YAML-Anker:** `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PRICING_PW` braucht sowohl `migrate` als auch `api`. Ohne Anker stünde derselbe Block zweimal da — und beim nächsten Ändern denkt man an eine Stelle. Die Syntax: `&app-env` definiert, `*app-env` verwendet. Der `x-`-Präfix markiert einen Abschnitt, den Compose selbst ignoriert.

**Warum `start_period` beim `db`-Healthcheck:** Beim allerersten Start (leeres Volume) initialisiert PostgreSQL sein Datenverzeichnis, startet zwischendurch neu und lehnt Verbindungen ab. Ohne Karenzzeit verbrauchen diese normalen Fehlschläge die `retries`, und der Dienst gilt als ungesund, obwohl er nur beschäftigt ist.

**Warum kein `container_name` für `api`:** Ein fester Containername verhindert, dass man denselben Dienst mehrfach starten kann (`docker compose up --scale api=3`) und kollidiert mit einem zweiten Compose-Projekt. Bei `db` und `adminer` hattest du es gesetzt — das ist für ein Übungsprojekt in Ordnung, aber ohne ist es sauberer. Die generierten Namen (`pricing-api-1`) sind ohnehin gut lesbar, sobald `name: pricing` gesetzt ist.

## Block 11 — Logs

Hier musst du **nichts ändern**, wenn du `LOG_FILE` einfach nicht setzt:

- `setup_logging()` legt den `RotatingFileHandler` nur an, wenn `log_file` einen Wert hat.
- Der Konsolen-Handler schreibt nach `stderr` — und `docker logs` fängt stdout **und** stderr ab.
- Die API ruft `setup_logging(console_level="INFO", …)` in ihrem `lifespan` auf, das Level passt also bereits.

`PYTHONUNBUFFERED=1` im Dockerfile ist die eigentliche Arbeit dieses Blocks: Ohne die Variable puffert Python seine Ausgabe, wenn sie nicht in ein Terminal geht — und Logzeilen erscheinen dann verspätet oder bei einem Absturz gar nicht.

## Block 12 — Non-root

```dockerfile
RUN groupadd --gid 1000 app && useradd --uid 1000 --gid 1000 --no-create-home app
COPY --from=builder --chown=1000:1000 /app/.venv /app/.venv
USER 1000:1000
```

**Warum einmal `app` und einmal `1000:1000`?** Das sieht redundant aus, ist aber zweierlei: `RUN groupadd/useradd` **legt den Eintrag an** (je eine Zeile in `/etc/group` und `/etc/passwd` — wirkt aufs Dateisystem), `USER` **wechselt die Prozess-Identität** (`setgid`/`setuid` beim Start — wirkt auf die Image-Config). Angelegt wird also einmal, benutzt wird einmal.

Dass in `USER` die **Zahl** steht statt `app`, hat drei Gründe:

1. **Kubernetes kann Namen nicht prüfen.** Mit `runAsNonRoot: true` verifiziert das Kubelet vor dem Start, dass die UID ≠ 0 ist. Steht dort ein Name, verweigert es den Start: `container has runAsNonRoot and image has non-numeric user (app), cannot verify user is non-root`. Das ist der praktisch relevanteste Grund.
2. **Unabhängig vom Dateisystem** — in einer Multi-Stage-Datei ist leicht übersehen, in *welcher* Stage `useradd` lief. `USER app` scheitert dort ohne Eintrag, `USER 1000:1000` nie.
3. **Die Gruppe ist explizit** — bei `USER app` kommt die Primärgruppe implizit aus `/etc/passwd`. Und beim Debuggen von Bind-Mount-Rechten denkt man ohnehin in Zahlen, weil der Host den Namen `app` nicht kennt.

Alle drei Varianten sind zulässig: nur `USER 1000:1000` ohne `RUN` (funktioniert, aber `whoami`/`$HOME`/`ls -l` bleiben kaputt) · `RUN useradd` + `USER app` (am lesbarsten, K8s-Problem) · **`RUN useradd` + `USER 1000:1000`** (Name für den Komfort *im* Container, Zahl für die Robustheit *von außen*).

**Reihenfolge beachten:** `USER` muss **nach** allen `COPY`- und `RUN`-Anweisungen stehen, die Schreibrechte brauchen. Und `--chown` gehört an jedes `COPY`, sonst gehören die Dateien root und der Prozess kann sie zwar lesen, aber nichts daneben anlegen.

**Warum die Migration trotzdem funktioniert:** Alembic schreibt nur in die Datenbank, nicht ins Dateisystem. Wäre das anders (etwa bei `alembic revision --autogenerate`, das Dateien in `alembic/versions/` erzeugt), bräuchte man dafür ein Bind Mount und passende Rechte — deshalb erzeugt man Migrationen auf dem Host und wendet sie im Container nur an.

## Block 13 — Healthcheck ohne curl

`python:3.13-slim` enthält **kein** `curl` und kein `wget`. Drei Wege:

| Weg | Bewertung |
|---|---|
| `apt-get install curl` | ~15 MB und eine Angriffsfläche mehr, für eine HTTP-Anfrage |
| Python-Einzeiler | **empfohlen** — Python ist ohnehin da |
| kein Healthcheck | verschenkt `--wait` und `service_healthy` |

Der Einzeiler aus dem Dockerfile prüft `/v1/health` und gibt Exit-Code 0 oder 1 zurück — genau das, was Docker als Gesundheitssignal erwartet.

## Block 14 — Profil

```yaml
  adminer:
    profiles: ["dev"]
```

Eine Zeile. `docker compose up -d` startet danach db, migrate und api; `docker compose --profile dev up -d` zusätzlich Adminer.

---

# Antworten auf die Entscheidungsfragen

| Block | Frage | Antwort |
|---|---|---|
| 1 | Gehört `alembic/` ins Image? | **Ja.** Der `migrate`-Dienst führt `alembic upgrade head` im Container aus und braucht die Versionsdateien. |
| 1 | Gehört `tests/` ins Image? | **Nein** für das Laufzeit-Image. Ja, wenn du später eine Test-Stage baust — dann aber gezielt dort. |
| 1 | Warum ist `.env` besonders wichtig? | Sie enthält `DB_PRICING_PW`. Käme sie ins Image, läge das Passwort dauerhaft in einer Schicht — und wäre auch durch späteres `rm` nicht mehr zu entfernen (Whiteout-Problem). |
| 2 | Warum `--host 127.0.0.1` im Container falsch? | Wegen des `net`-Namespace ist das der Loopback **des Containers**. Der veröffentlichte Port leitet auf die Container-IP, nicht auf dessen Loopback — der Server wäre unerreichbar. |
| 4 | Woher kommen die Werte ohne `.env` im Image? | Aus den Umgebungsvariablen, die Compose setzt. `pydantic-settings` liest Umgebungsvariablen mit **höherer** Priorität als die `.env`-Datei — läge doch eine im Image, würde Compose sie trotzdem überstimmen. Aber sie hat dort nichts verloren. |
| 7 | Was muss aus der Builder-Stage herüber? | `/app/.venv`, plus `alembic/` und `alembic.ini` aus dem Kontext. `src/` **nicht**, dank `--no-editable`. |
| 7 | Warum `ENV PATH=…` statt `uv run`? | `PATH` auf `/app/.venv/bin` macht `uvicorn` und `alembic` direkt aufrufbar. Ein Zwischenprozess weniger, und die Signalweitergabe an PID 1 bleibt sauber. |
| 7 | Braucht Runtime noch `uv`? | **Nein.** Genau das ist der Gewinn des Multi-Stage-Builds. |
| 8 | `container_name` setzen? | Besser nicht — verhindert Skalierung und kollidiert projektübergreifend. |
| 8 | Passwort in Compose oder `.env`? | Für ein Übungsprojekt direkt in der Compose-Datei, weil sie damit selbsterklärend ist. In echt: Variablensubstitution `${DB_PRICING_PW}` aus einer nicht eingecheckten `.env`. |
| 10 | Ansatz A oder B? | **B** (eigener `migrate`-Dienst). Trennt Migration und Anwendungsstart sauber, skaliert auf mehrere API-Instanzen, und der Fehlschlag einer Migration ist am Exit-Code sofort sichtbar. |
| 10 | Wie Doppelung der Variablen vermeiden? | YAML-Anker `&app-env` / `*app-env`. |
| 10 | Arbeitsverzeichnis für Alembic? | `/app` — dort liegt `alembic.ini`, und `%(here)s` löst relativ zu ihr auf. `WORKDIR /app` erledigt das. |
| 13 | `curl` nachinstallieren? | Nein, Python-Einzeiler. |

---

# Fehlersuche

| Symptom | Ursache | Lösung |
|---|---|---|
| `ModuleNotFoundError: No module named 'pricing'` | Projekt nicht installiert oder `src/` fehlt bei editable Install | Zweites `uv sync` **ohne** `--no-install-project` vorhanden? `--no-editable` gesetzt? |
| `ConfigError: Umgebungsvariable DB_PRICING_PW` | Variable kommt nicht im Container an | `docker compose exec api env \| grep DB_` — Anker richtig referenziert? |
| `connection refused` an `db:5432` | DB noch nicht bereit | `condition: service_healthy` gesetzt? `start_period` groß genug? |
| `could not translate host name "db"` | Container nicht im Compose-Netz oder falscher Dienstname | `docker compose ps`, Dienstname prüfen |
| `address already in use` beim `-p` | auf dem **Host** lauscht schon etwas auf dem Port | `ss -tlnp \| grep :8000` — siehe [[IP-Adressen, Ports und Sockets - Grundlagen|Netzwerk-Grundlagen]] |
| `exec: "uvicorn": not found` | `PATH` zeigt nicht auf `/app/.venv/bin` | `ENV PATH=…` in der **Runtime**-Stage? |
| `/app/.venv/bin/python: bad interpreter` | `WORKDIR` in den Stages unterschiedlich | beide auf `/app` setzen |
| `permission denied` beim Start | `USER` vor den `COPY`s, oder `--chown` vergessen | Reihenfolge prüfen |
| `migrate` läuft endlos neu | `restart` nicht auf `"no"` | `restart: "no"` setzen |
| API startet, aber `docker compose logs` bleibt leer | `LOG_FILE` gesetzt, oder Ausgabe gepuffert | `LOG_FILE` entfernen, `PYTHONUNBUFFERED=1` setzen |
| Code-Änderung wirkt nicht | Image nicht neu gebaut | `docker compose up -d --build` |
| `docker compose ps` zeigt api dauerhaft `starting` | Healthcheck schlägt fehl | `docker inspect --format '{{json .State.Health}}' pricing-api-1 \| jq` |

---

# Der Abnahmetest

```bash
# Kompletter Neuaufbau -- inklusive leerem Volume
docker compose down -v
docker compose up -d --build --wait

# 1. Alle Dienste gesund?
docker compose ps

# 2. Migration wirklich gelaufen?
docker compose exec db psql -U pricing -d pricing -c '\dt'
docker compose exec db psql -U pricing -d pricing -c 'SELECT version_num FROM alembic_version;'

# 3. API antwortet?
curl -s http://127.0.0.1:8000/v1/health
# → {"status":"ok"}

# 4. Logs sichtbar?
docker compose logs api | tail -20
# → "Pricing API gestartet"

# 5. Nicht als root?
docker compose exec api id
# → uid=1000 gid=1000

# 6. Aufräumen
docker compose down
```

## Optional: API-Schlüssel anlegen

Auf einem frischen Volume gibt es noch keinen Schlüssel — außer `/v1/health` ist damit kein Endpunkt nutzbar. Der Einmal-Befehl dafür:

```bash
docker compose run --rm api pricing-apikey create "lokale Entwicklung"
```

Das ist bewusst **kein** Compose-Dienst: Schlüssel gehören nicht automatisiert erzeugt, und der Aufruf zeigt schön den Unterschied zwischen `run` (neuer Wegwerf-Container) und `exec` (in einen laufenden hinein).

## Optional: Test-Stage

```dockerfile
FROM builder AS test
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-editable          # diesmal MIT dev-Abhaengigkeiten
COPY tests/ ./tests/
CMD ["pytest", "-q"]
```

```bash
docker build --target test -t pricing-api:test . && docker run --rm pricing-api:test
```

Setzt voraus, dass `tests/` **nicht** in `.dockerignore` steht — also entweder die Zeile dort entfernen oder die Test-Stage separat bauen.
