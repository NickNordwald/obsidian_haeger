Abhakbarer Plan zur Übungsaufgabe aus [[1.7 Docker - Leitfaden|1.7 Docker (Leitfaden)]]. Theorie dazu: [[1.7 Docker - Vertiefung|1.7 Docker – Vertiefung]]. Laufender Stand: [[Umsetzungsstand (Docker)|Umsetzungsstand (Docker)]].

> [!tip] Wenn du hängst
> Zu jedem Block gibt es eine ausgearbeitete Referenz in [[Musterlösung (Docker)|Musterlösung (Docker)]] — inklusive der fertigen Dateien, der Begründungen und einer Fehlersuch-Tabelle. Faustregel: erst ~20 Minuten selbst versuchen, dann nachschlagen und **die Begründung** mitlesen.

> [!info] Aufgabenstellung
> „Dockerisiere deine Anwendung und mache die gesamte Anwendung inklusive Datenbank über einen einzigen `docker-compose`-Befehl startbar."

**Arbeitsmodus:** Du schreibst, ich reviewe und erkläre. Jeder Block führt **genau eine** neue Technik ein, in der Reihenfolge von einfach nach komplex. Erst abhaken, wenn der jeweilige *Test* am Blockende durchläuft — nicht, wenn der Code „fertig aussieht".

**Projektordner:** `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Docker/`

**Ausgangslage (geprüft):** Compose-Datei mit `db` (postgres:18, Healthcheck, Volume) und `adminer` existiert. Die Anwendung selbst läuft noch auf dem Host. Konfiguration kommt über `pydantic-settings` aus Umgebungsvariablen — `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PRICING_PW`, `LOG_FILE`, `LOG_LEVEL`, `INGEST_BASE_DIR`. Es gibt **keine** `DATABASE_URL`; die URL wird in `persistence/db.py` aus den Einzelteilen zusammengesetzt.

---

# Übersicht

| Phase                | Blöcke | Ziel                                                            |
| -------------------- | ------ | --------------------------------------------------------------- |
| **A — Vorbereitung** | 1–2    | Kontext sauber, Startbefehl bekannt                             |
| **B — Das Image**    | 3–7    | `docker build` erzeugt ein lauffähiges Image                    |
| **C — Der Verbund**  | 8–11   | `docker compose up` startet alles, in der richtigen Reihenfolge |
| **D — Schliff**      | 12–15  | Non-root, Healthcheck, Profile, Doku                            |

---

# Phase A — Vorbereitung

## - [ ] Block 1: `.dockerignore`

**Neue Technik:** Der Build-Kontext und seine Filterung.

**Warum zuerst:** Jeder `docker build` überträgt den gesamten Kontext an BuildKit. Deine `.venv/` ist mehrere hundert MB groß und für das Image nutzlos — ohne Filterung wartest du bei jedem Versuch darauf.

**Syntax:** Wie `.gitignore`, aber eigene Datei. `!` negiert.

```
# Kommentar
verzeichnis/
*.endung
!ausnahme.endung
```

**Zu tun:** Lege `.dockerignore` im Projektstamm an. Geh deine `.gitignore` durch und ergänze, was dort fehlt. Denk an: `.git`, `.venv`, Caches (`.mypy_cache`, `.pytest_cache`, `.ruff_cache`), `__pycache__`, `logs/`, `.env`, `tests/`, `.vscode/`, `daily_data/`, `price_recommendations.csv`, `.coverage`.

**Entscheidungsfragen, die du beantworten solltest:**
- Gehört `alembic/` ins Image? (Denk daran, wer `alembic upgrade head` ausführen soll.)
- Gehört `tests/` ins Image? Was spricht dafür, was dagegen?
- Warum ist `.env` in der Liste besonders wichtig?

**Test:** Misst, was tatsächlich im Kontext ankommt — braucht noch **kein** `Dockerfile` im Projekt.

```bash
docker build --progress=plain --no-cache -f - . <<'EOF'
FROM alpine
COPY . /ctx
RUN du -sh /ctx && du -sh /ctx/* /ctx/.[!.]* 2>/dev/null | sort -h
EOF
```

Zwei Details, ohne die es nicht funktioniert:

- **`-f -`** liest das Rezept von stdin, der Punkt bleibt der Kontext. Schreibt man stattdessen nur `-` (ohne `-f`), kommt *auch der Kontext* von stdin — dann überträgt Docker das Projekt gar nicht und meldet `transferring context: 2B`.
- **`COPY . /ctx` ist Pflicht.** BuildKit überträgt **faul**: Ohne eine Anweisung, die den Kontext benutzt, schickt es nichts, und die Messung wäre wertlos.

**Erwartung:** ~2 MB. Das `du` listet auf, was angekommen ist — geh die Liste durch und frag dich bei jedem Posten, ob er im Image etwas zu suchen hat.

---

## - [ ] Block 2: Den Startbefehl auf dem Host festnageln

**Neue Technik:** keine — Vorbereitung.

**Warum:** Bevor du einen Startbefehl in ein Dockerfile schreibst, musst du wissen, wie er lautet. Rate ihn nicht.

**Zu tun:** Finde heraus und notiere im Umsetzungsstand:
1. Mit welchem exakten Befehl startest du die API heute? (`uvicorn <modulpfad>:app --host … --port …`)
2. Was ist der Import-Pfad der FastAPI-App? (Hinweis: `src/pricing/presentation/api/app.py`)
3. Welche Umgebungsvariablen braucht die Anwendung zum Start — und welche davon *muss* sie haben, damit `get_engine()` nicht mit `ConfigError` abbricht?
4. Warum ist `--host 127.0.0.1` im Container falsch? (Denk an den `net`-Namespace.)

**Test:** Der Befehl startet lokal, `curl http://127.0.0.1:8000/v1/health` liefert `{"status":"ok"}`.

---

# Phase B — Das Image

## - [ ] Block 3: Minimales, einstufiges Dockerfile

**Neue Technik:** `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`.

**Ziel:** Erst *lauffähig*, dann *gut*. Optimiert wird in Block 5 und 6.

**Syntax-Muster:**
```dockerfile
FROM python:3.13-slim

# uv als einzelne Binärdatei aus einem fremden Image holen
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

WORKDIR /app
COPY . .
RUN uv sync --frozen

CMD ["uv", "run", "uvicorn", "...", "--host", "0.0.0.0"]
```

**Merkposten:**
- `--frozen` = installiere **exakt** die `uv.lock`, ohne sie zu aktualisieren. In einem Image immer.
- `CMD` in **exec-Form** (JSON-Array). Warum, steht in der Vertiefung unter *PID-1-Falle*.
- `--host 0.0.0.0`, nicht `127.0.0.1` — sonst lauscht der Server nur auf dem Container-Loopback und ist von außen unerreichbar.

**Test:**
```bash
docker build -t pricing-api:dev .
docker run --rm -it --entrypoint sh pricing-api:dev     # erst umsehen
#   → ls -la /app, ls /app/.venv/bin, uv --version
docker run --rm -p 8000:8000 pricing-api:dev            # dann starten
curl http://127.0.0.1:8000/v1/health
```

> [!tip] Wenn es scheitert
> Das ist der Normalfall beim ersten Versuch. Reihenfolge: `docker logs` bzw. die Ausgabe lesen → `--entrypoint sh` und selbst nachsehen → `docker inspect`. Häufige Ursachen hier: falscher Modulpfad, Paket nicht installiert (src-Layout!), fehlende Umgebungsvariable.

---

## - [ ] Block 4: Konfiguration von außen

**Neue Technik:** `-e` / `--env-file`, und das Zusammenspiel mit `pydantic-settings`.

**Warum:** Das Image darf keine Konfiguration enthalten. Es muss dieselbe Binärgestalt in Entwicklung und Produktion sein — nur die Umgebung unterscheidet sich.

**Zu tun:** Starte den Container gegen deine bereits laufende `db` aus Compose. Zwei Hürden sind eingebaut:
1. Der Container hängt noch **nicht** im Compose-Netzwerk → wie erreicht er die Datenbank? (`--network` mit dem Namen aus `docker network ls`.)
2. `db_host` muss auf den Dienstnamen zeigen, nicht auf `localhost`.

```bash
docker network ls                       # wie heißt das Compose-Netz?
docker run --rm -p 8000:8000 \
  --network <netz> \
  -e DB_HOST=db \
  -e DB_PRICING_PW=pricing \
  pricing-api:dev
```

**Entscheidungsfrage:** Deine `Settings`-Klasse hat `env_file=".env"` konfiguriert. Wenn im Container keine `.env` liegt (siehe Block 1) — woher kommen die Werte dann, und ist das gewollt? Was passiert, wenn doch eine `.env` mitkopiert würde?

**Test:** Ein Endpunkt, der die Datenbank anfasst, antwortet ohne `ConfigError` und ohne `connection refused`.

---

## - [ ] Block 5: Layer-Caching — die Reihenfolge umbauen

**Neue Technik:** Cache-Invalidierung gezielt steuern.

**Warum:** Dein `COPY . .` aus Block 3 wirft bei **jeder** Code-Änderung die Abhängigkeitsinstallation weg. Miss das erst, dann behebe es.

**Zu tun:**
1. Baue neu, ändere eine Zeile in `src/pricing/main.py`, baue nochmal. Miss die Zeit (`time docker build …`).
2. Baue das Dockerfile um: erst `pyproject.toml` und `uv.lock` kopieren, dann installieren, **dann** den Quellcode kopieren.
3. Wiederhole die Messung.

**Syntax-Muster:**
```dockerfile
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-install-project --no-dev
COPY src/ ./src/
COPY alembic/ ./alembic/
COPY alembic.ini ./
RUN uv sync --frozen --no-dev
```

**Merkposten:**
- `--no-install-project` installiert **nur die Abhängigkeiten**, nicht dein eigenes Paket. Genau das trennt den teuren vom billigen Schritt.
- `--no-dev` lässt pytest, ruff und mypy weg — die gehören nicht in ein Laufzeit-Image.

**Test:** Zweiter Build nach einer Code-Änderung dauert **spürbar** kürzer als der erste. Trage beide Zeiten in den Umsetzungsstand ein.

---

## - [ ] Block 6: Cache-Mount für uv

**Neue Technik:** `RUN --mount=type=cache`.

**Warum:** Selbst wenn sich die Lockfile ändert, muss uv die Pakete nicht erneut *herunterladen*. Der Cache soll Builds überleben, aber nicht im Image landen.

**Syntax:**
```dockerfile
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-dev
```

**Merkposten:**
- `UV_LINK_MODE=copy` ist bei Cache-Mounts nötig, weil Hardlinks über die Mount-Grenze hinweg nicht funktionieren.
- `UV_COMPILE_BYTECODE=1` erzeugt `.pyc`-Dateien schon beim Bauen — der erste Request wird dadurch schneller.

**Test:** `uv.lock` künstlich ändern (ein Paket hinzufügen und wieder entfernen), neu bauen — der Download-Schritt ist deutlich kürzer als beim allerersten Mal.

---

## - [ ] Block 7: Multi-Stage-Build

**Neue Technik:** Mehrere `FROM`, `COPY --from=<stage>`, `AS <name>`.

**Warum:** `uv` und der Download-Cache haben zur Laufzeit nichts verloren. Und wegen des Whiteout-Problems (Vertiefung, Stichwort 3) lässt sich das nachträglich **nicht** entfernen.

**Struktur:**
```dockerfile
FROM python:3.13-slim AS builder
# … uv holen, Abhängigkeiten installieren, Projekt installieren …

FROM python:3.13-slim AS runtime
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY --from=builder /app/src   /app/src
ENV PATH="/app/.venv/bin:$PATH"
CMD ["uvicorn", "…"]
```

**Entscheidungsfragen:**
- Was genau muss aus der Builder-Stage herüber? (Denk an `alembic/` und `alembic.ini` — wer braucht die?)
- Warum steht `ENV PATH=…` in der Runtime-Stage, und was ersetzt das? (Vorher stand `uv run` im `CMD`.)
- Braucht die Runtime-Stage noch `uv`?

**Test:**
```bash
docker image ls pricing-api          # Größe vor/nach vergleichen
docker image history pricing-api:dev # welche Schicht kostet wie viel?
docker run --rm pricing-api:dev uv --version   # sollte jetzt FEHLSCHLAGEN
```
Trage die Größen vor und nach Multi-Stage in den Umsetzungsstand ein.

---

# Phase C — Der Verbund

## - [ ] Block 8: Die API als Compose-Dienst

**Neue Technik:** `build:` im Dienst, `environment:`, `depends_on`.

**Zu tun:** Ergänze in der Compose-Datei einen Dienst `api`. Gleichzeitig zwei Aufräumarbeiten:
- `name: pricing` auf oberster Ebene setzen (löst das Volume-Namens-Problem aus 1.6).
- Überlegen, ob die Datei `compose.yaml` heißen soll — dann jetzt umbenennen.

**Syntax-Muster:**
```yaml
name: pricing

services:
  api:
    build:
      context: .
    environment:
      DB_HOST: db
      DB_PRICING_PW: pricing
    ports:
      - "127.0.0.1:8000:8000"
    depends_on:
      - db
```

**Entscheidungsfragen:**
- `container_name` setzen wie bei `db` und `adminer` — oder bewusst weglassen? (Was verhindert ein fester Containername?)
- Passwort direkt in der Compose-Datei, oder über Variablensubstitution aus `.env`? Beides ist für ein Toy-Projekt vertretbar — begründe deine Wahl.

**Test:**
```bash
docker compose up -d --build
docker compose ps
curl http://127.0.0.1:8000/v1/health
```

> [!warning] Erwarteter Fehlschlag
> Es ist gut möglich, dass die API beim ersten Start mit `connection refused` stirbt — genau dafür ist Block 9 da. Wenn es beim ersten Versuch klappt, war es Glück, kein Beweis.

---

## - [ ] Block 9: Auf Bereitschaft warten, nicht auf Start

**Neue Technik:** `depends_on` mit `condition: service_healthy`.

**Warum:** `depends_on: [db]` wartet nur, bis der *Container* läuft — nicht, bis PostgreSQL Verbindungen annimmt.

**Syntax:**
```yaml
    depends_on:
      db:
        condition: service_healthy
```

**Zu tun:** Umbauen. Schau dir dabei den bestehenden `healthcheck` beim `db`-Dienst an und ergänze `start_period`. Überlege, was ein sinnvoller Wert ist.

**Test — der aussagekräftige:**
```bash
docker compose down -v      # Volume weg → DB muss komplett neu initialisieren
docker compose up -d --build
docker compose logs api | head -30
```
Die Neuinitialisierung dauert am längsten. Wenn die API *das* übersteht, hält die Bedingung.

---

## - [ ] Block 10: Migrationen im Verbund

**Neue Technik:** Einmal-Dienst mit `condition: service_completed_successfully` (oder Entrypoint-Skript).

**Warum:** Ein frisches Volume hat eine leere Datenbank. Die Aufgabe verlangt, dass **ein** Befehl alles startbar macht — also muss `alembic upgrade head` Teil davon sein.

**Entscheidung:** Ansatz A (Entrypoint-Skript in der API) oder B (eigener `migrate`-Dienst)? Die Abwägung steht in der Vertiefung unter *Das Migrationsmuster*. Notiere deine Wahl **mit Begründung** im Umsetzungsstand.

**Syntax-Muster für B:**
```yaml
  migrate:
    build:
      context: .
    command: ["alembic", "upgrade", "head"]
    environment: …
    depends_on:
      db:
        condition: service_healthy

  api:
    depends_on:
      migrate:
        condition: service_completed_successfully
```

**Merkposten für A:** Ein Entrypoint-Skript muss mit `exec "$@"` enden, damit die Anwendung PID 1 wird. Sonst kommt kein `SIGTERM` an.

**Entscheidungsfragen:**
- Bei Ansatz B: Braucht der `migrate`-Dienst dieselben Umgebungsvariablen? Wie vermeidest du Doppelung? (Stichwort YAML-Anker `&name` / `*name` oder `extends`.)
- Aus welchem Arbeitsverzeichnis muss Alembic laufen, damit `script_location = %(here)s/alembic` greift?

**Test — der eigentliche Abnahmetest der Aufgabe:**
```bash
docker compose down -v
docker compose up -d --build
docker compose exec db psql -U pricing -d pricing -c '\dt'
# → alle Tabellen da, inkl. alembic_version
curl http://127.0.0.1:8000/v1/health
```

---

## - [ ] Block 11: Logs sichtbar machen

**Neue Technik:** keine — Konfigurationskorrektur.

**Warum:** `docker logs` liest nur stdout/stderr von PID 1. Deine `setup_logging()` legt bei gesetztem `LOG_FILE` einen `RotatingFileHandler` an — dessen Ausgabe siehst du im Container nirgends, und die Datei sichert niemand.

**Zu tun:** Prüfe, was im Container mit `LOG_FILE` passiert, und sorge dafür, dass die Anwendungslogs in `docker compose logs api` auftauchen. Überlege dabei auch, ob `console_level` (Default `WARNING`) für einen Container passend ist — die API ruft `setup_logging()` in ihrem `lifespan` auf.

**Test:** Ein Request auf einen Endpunkt erzeugt eine sichtbare Zeile in `docker compose logs -f api`.

---

# Phase D — Schliff

## - [ ] Block 12: Nicht als root

**Neue Technik:** `USER`, `COPY --chown`.

**Zu tun:** Lege in der Runtime-Stage einen unprivilegierten Benutzer an (oder benutze eine feste UID) und wechsle darauf. Achte auf die Eigentümerrechte der kopierten Dateien.

**Syntax-Muster:**
```dockerfile
COPY --from=builder --chown=1000:1000 /app/.venv /app/.venv
USER 1000:1000
```

**Test:**
```bash
docker compose exec api id        # → uid=1000, nicht 0
docker compose exec api whoami
```
Danach den vollständigen Ablauf aus Block 10 wiederholen — ein häufiger Fehler ist, dass die Migration danach an Schreibrechten scheitert.

---

## - [ ] Block 13: `HEALTHCHECK` für die API

**Neue Technik:** `HEALTHCHECK` im Dockerfile bzw. `healthcheck:` im Dienst.

**Warum:** Damit `docker compose ps` und `--wait` etwas Sinnvolles über die API aussagen — und damit sie sich in dieselbe Systematik einreiht wie `db`.

**Entscheidungsfrage:** Dein Image ist `slim` — ist `curl` überhaupt installiert? Wenn nein: nachinstallieren (kostet Platz) oder mit Bordmitteln prüfen (Python kann HTTP)? Was ist der schlankere Weg?

**Merkposten:** `/v1/health` ist ohne API-Schlüssel erreichbar — genau dafür ist der Endpunkt gedacht.

**Test:**
```bash
docker compose up -d --wait
docker compose ps      # api muss "healthy" zeigen, nicht nur "running"
```

---

## - [ ] Block 14: Adminer in ein Profil

**Neue Technik:** `profiles:`.

**Warum:** Adminer ist ein Entwicklungswerkzeug — eine ungeschützte Datenbank-Oberfläche gehört nicht in den Standardstart.

**Syntax:**
```yaml
  adminer:
    profiles: ["dev"]
```

**Test:**
```bash
docker compose down
docker compose up -d              # kein adminer
docker compose --profile dev up -d # mit adminer
```

---

## - [ ] Block 15: Dokumentation und Abnahme

**Zu tun:**
1. **README-Abschnitt** im Projekt: der eine Startbefehl, die Voraussetzungen, wie man an die Datenbank kommt, wie man Logs liest, wie man aufräumt.
2. **Backup-Befehl** notieren (`pg_dump` über `docker compose exec -T`) — auch wenn dir die Daten egal sind, gehört der Reflex geübt.
3. **Umsetzungsstand** vervollständigen: getroffene Entscheidungen mit Begründung, gelernte Lektionen, offene Punkte.

**Abnahmetest — der Test, den die Aufgabe eigentlich meint:**
```bash
cd /tmp && rm -rf abnahme && git clone <dein-repo> abnahme && cd abnahme
docker compose up -d --build --wait
curl http://127.0.0.1:8000/v1/health
```
Auf einem frischen Klon, ohne Vorwissen, ohne manuelle Schritte. Wenn das läuft, ist die Aufgabe erfüllt.

> [!question] Und wenn es *nicht* läuft?
> Dann ist genau das der Erkenntnisgewinn: Irgendetwas existiert nur auf deinem Rechner. Notier es im Umsetzungsstand — diese Liste ist wertvoller als ein Setup, das auf Anhieb klappt.

---

# Optionale Erweiterungen

Nur, wenn Zeit bleibt — nicht Teil der Aufgabe:

- - [ ] `compose.override.yaml` für die Entwicklung: Quellcode als Bind Mount, `uvicorn --reload`
- - [ ] Test-Stage im Dockerfile (`--target test`), die `pytest` ausführt
- - [ ] Basis-Image per Digest festnageln statt per Tag
- - [ ] `internal: true`-Netz, damit die Datenbank nur für die API erreichbar ist
- - [ ] `deploy.resources.limits` bzw. `mem_limit` setzen und mit `docker stats` beobachten
- - [ ] Multi-Arch-Build mit `docker buildx` ausprobieren
