---
tags:
  - documentation
type: notiz
date: 2026-08-18
---

Abgeschlossener Umsetzungsstand der **Containerisierung** der Pricing-Anwendung (Aufgabe: [[1.7 Docker - Leitfaden#Aufgabe|1.7 Leitfaden – Aufgabe]]). Baut auf [[Umsetzungsstand (Datenbank Migrations Tools)|Kapitel 1.6]] auf.

Theorie: [[1.7 Docker - Vertiefung|Vertiefung]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.7 Docker/Anki Karteikarten|Anki Karteikarten]] · Aufgekommene Fragen: [[Fragen aus der Umsetzung|Q&A – Fragen aus der Umsetzung]] · Netzwerk-Grundlagen: [[IP-Adressen, Ports und Sockets - Grundlagen|IP-Adressen, Ports und Sockets]] · Plan: [[Docker Plan & Syntax-Tutorial|Docker Plan & Syntax-Tutorial]] (15 Blöcke, alle abgearbeitet) · Referenz: [[Musterlösung (Docker)|Musterlösung]]

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Docker/`

## Endstand

Die gesamte Anwendung — API, Datenbank, Migrationen — startet mit **einem** Befehl auf einem frischen Klon:

```bash
docker compose up -d --build --wait
curl http://127.0.0.1:8000/v1/health     # → {"status":"ok"}
```

**Dockerfile** — Multi-Stage (`builder` → `runtime`) auf `python:3.13-slim`:

- `uv` per `COPY --from=ghcr.io/astral-sh/uv:latest`, zwei getrennte `uv sync`-Läufe (erst Abhängigkeiten, dann das eigene Paket) für wirksames Layer-Caching
- `--mount=type=cache,target=/root/.cache/uv` — Download-Cache überlebt Builds, landet aber nicht im Image
- `--no-dev` (pytest/ruff/mypy fliegen raus) und `--no-editable` (Code liegt echt in `site-packages`, nicht als Verweis auf `/app/src`)
- Runtime-Stage übernimmt nur `.venv`, `alembic.ini` und `alembic/` — per `COPY --chown=1000:1000`
- `USER 1000:1000` am Ende, `PYTHONUNBUFFERED=1`, `HEALTHCHECK` per Python-Einzeiler gegen `/v1/health`
- `CMD` in exec-Form mit explizitem `--host 0.0.0.0 --port 8000`

**compose.yml** — vier Dienste unter dem Projektnamen `pricing`:

| Dienst | Rolle | Besonderheit |
|---|---|---|
| `db` | `postgres:18` | Healthcheck `pg_isready`, Volume `pricing_data`, Port an `127.0.0.1` gebunden |
| `migrate` | `alembic upgrade head` | dasselbe Image wie `api`, nur anderes `command`; `restart: "no"` |
| `api` | Uvicorn | wartet auf `migrate` per `condition: service_completed_successfully`, Bind Mount `./daily_data:/app/daily_data:ro` |
| `adminer` | Web-UI | im Profil `dev`, startet nur bei `--profile dev` |

Gemeinsame Umgebung per YAML-Anker (`x-app-env: &app-env`), Passwort per `${DB_PRICING_PW:?…}` aus der `.env`. `.dockerignore` schließt `.git`, `.venv`, Caches, `logs/`, `daily_data/`, `.env`, `tests/` und `scripts/` aus.

## Wichtigste Entscheidungen

**`python:3.13-slim` statt Alpine.** Alpine nutzt musl statt glibc — dadurch fallen für viele Pakete die vorkompilierten Wheels weg und der Build kompiliert aus dem Quellcode. Der Größenvorteil wird durch Build-Zeit und Fehleranfälligkeit mehr als aufgewogen.

**Multi-Stage-Build.** Die Build-Werkzeuge (`uv`, Compiler-Reste, Caches) bleiben in der `builder`-Stage; ins Laufzeit-Image wandert nur die fertige `.venv`. Kleineres Image *und* kleinere Angriffsfläche.

**Zwei `uv sync`-Läufe statt einem.** Abhängigkeiten ändern sich selten, der eigene Code ständig. Getrennt bedeutet: Eine Code-Änderung invalidiert nur die letzte, billige Schicht. Der Beweis dafür ist nicht die gemessene Zeit, sondern das Wort **`CACHED`** in der Build-Ausgabe beim teuren `uv sync`.

**`--no-editable` im zweiten `uv sync`.** Standardmäßig legt uv nur einen Verweis auf `/app/src` an — und `src/` wird nicht in die Runtime-Stage kopiert. Ohne das Flag: `ModuleNotFoundError: No module named 'pricing'` zur Laufzeit, obwohl der Build durchläuft.

**Migrationen als eigener Dienst, nicht im Entrypoint der API.** `migrate` nutzt dasselbe Image mit anderem `command` und läuft **einmal** (`restart: "no"`); `api` startet erst nach `service_completed_successfully`. Würde die API selbst migrieren, würden bei mehreren Instanzen mehrere Migrationsläufe gleichzeitig starten.

**`--chown` am `COPY` statt `RUN chown -R`.** Ein rekursives `chown` über die ~150 MB große `.venv` ändert die Metadaten jeder Datei — und jede geänderte Datei landet **vollständig** in der neuen Schicht. Das verdoppelt den Platzbedarf, ohne dass sich ein Byte Inhalt ändert.

**`USER 1000:1000` numerisch statt `USER app`.** Kubernetes kann mit `runAsNonRoot: true` einen *Namen* nicht auf Nicht-root prüfen und verweigert dann den Start. Außerdem ist die Zahl unabhängig davon, ob `/etc/passwd` in der finalen Stage überhaupt einen Eintrag hat — und sie passt zur UID des Host-Benutzers, was Bind-Mount-Rechte glattzieht.

**Python-Einzeiler statt `curl` im `HEALTHCHECK`.** `python:3.13-slim` enthält weder `curl` noch `wget`; Nachinstallieren kostet ~15 MB und Angriffsfläche für eine einzige HTTP-Anfrage, die Python ohnehin kann. Geprüft wird `127.0.0.1` — hier ist localhost richtig, weil Prüfer und Geprüfter derselbe Container sind.

**`--host 0.0.0.0` explizit im `CMD`.** Im Container muss auf allen Schnittstellen gelauscht werden; ein Server auf `127.0.0.1` nimmt Verkehr über `eth0` gar nicht erst an. Der Port steht ebenfalls explizit da, statt sich auf uvicorns Default zu verlassen.

**Ports nur an `127.0.0.1` gebunden.** `"127.0.0.1:8000:8000"` statt `"8000:8000"` — sonst ist der Dienst im gesamten lokalen Netz erreichbar, an der Firewall vorbei.

**`LOG_FILE` im Container bewusst weggelassen.** Damit legt `setup_logging()` den `RotatingFileHandler` gar nicht erst an; übrig bleibt der stderr-Handler, den `docker logs` abgreift. Ein Container soll sich nicht selbst um Logrotation kümmern — das ist Sache des Logging-Treibers.

**Adminer in ein Profil.** Ein Datenbank-Web-UI gehört nicht in den Standardstart. `profiles: ["dev"]` macht ihn opt-in — mit der Konsequenz, dass `down` dieselben Profile kennen muss wie `up`.

**Passwort per `${DB_PRICING_PW:?meldung}`.** Die `:?`-Form bricht ab, wenn die Variable fehlt — `${VAR}` würde still zum leeren String. Ehrlich dazu: Das hält den Wert aus git heraus, macht ihn aber **nicht geheim** (er steht weiter in `docker inspect`); echte Geheimhaltung bräuchte Docker Secrets.

**`daily_data` als Bind Mount mit `:ro`.** Der Ordner steht in `.dockerignore` und ist damit nicht im Image — die Anwendung lief trotzdem, weil `resolve_ingest_dir()` sauber einen `IngestFolderMissingError` wirft statt abzustürzen. Der Mount macht den Ingest nutzbar; `:ro`, weil lesen genügt.

## Gelernte Lektionen

- **Bei Docker prüft man das Ergebnis, nicht die Absicht.** Alle vier Fehler dieser Iteration waren **stille** Fehler — nichts brach ab, es passierte nur nicht das Gewünschte: `PYTHONUNBUFFERD` (Tippfehler), `start_perios` (Tippfehler), ein übrig gebliebenes `COPY . .` vor den gezielten `COPY`s, und `--no-install-project` in *beiden* `uv sync`. Werkzeuge dagegen: `docker compose config`, das Wort `CACHED`, `docker compose exec api env`, `docker inspect`.
- **Layer-Caching ist eine Kette.** Der Schlüssel jeder Anweisung enthält den der vorherigen — ein `COPY . .` reagiert auf jede Dateiänderung und macht alles Folgende ungültig. Die Reihenfolge im Dockerfile ist eine Aussage darüber, was sich wie oft ändert.
- **Die PID-1-Falle sind zwei Fallen.** Signale (Shell-Form von `CMD` reicht `SIGTERM` nicht weiter → `docker stop` dauert immer 10 s) und Zombies (PID 1 muss verwaiste Kinder einsammeln). Hier war nur die erste relevant, gelöst durch die exec-Form.
- **`HEALTHCHECK` ist nur ein Signal.** Docker startet einen `unhealthy`-Container **nicht** neu. Auswerten müssen es andere: `depends_on: condition`, `up --wait`, Kubernetes.
- **`0.0.0.0` ist keine Adresse, sondern ein Platzhalter beim Lauschen.** Man verbindet sich nie *zu* `0.0.0.0`. Und weil es `127.0.0.1` einschließt, kollidiert ein `-p 8000:8000` mit einem lokal laufenden uvicorn.
- **`down` muss dieselben Profile kennen wie `up`.** Sonst bleibt ein Container stehen, hängt weiter im Projektnetz, und das Netzwerk lässt sich nicht entfernen („Resource is still in use").
- **YAML 1.1 macht aus `no` ein `false`.** `restart: "no"` braucht Anführungszeichen — sonst liest der Parser einen Boolean, wo Compose eine Zeichenkette erwartet.
- **Named Volume und Bind Mount unterscheiden sich um ein Zeichen.** Beginnt der linke Teil mit `.` oder `/`, ist es ein Bind Mount. Ein Tippfehler im Host-Pfad erzeugt stillschweigend ein neues, leeres, root-gehörendes Verzeichnis — die Langform (`type: volume|bind`) verhindert das.
- **`stdout` ist blockgepuffert, wenn es kein TTY ist.** Uvicorns Access-Logs gehen nach stdout und erscheinen ohne `PYTHONUNBUFFERED=1` verspätet — bei einem Absturz gar nicht. `stderr` ist seit Python 3.9 zeilengepuffert und deshalb unauffällig.
- **`/docs` ist ein Client der eigenen API.** FastAPI baut ~25 Zeilen HTML mit zwei CDN-Verweisen; die Substanz kommt aus `/openapi.json`. Folge: Das Internet braucht der Browser, nicht der Server — ein isolierter Container liefert `/docs` trotzdem aus.
