Fragen, die während der Übungsaufgabe zu [[1.7 Docker - Leitfaden|1.7 Docker]] tatsächlich aufgekommen sind — mit kurzer Antwort und Verweis auf die ausführliche Stelle. Gedacht zum **Nachschlagen**, nicht zum Durchlesen.

Ausführlich: [[1.7 Docker - Vertiefung|1.7 Docker – Vertiefung]] · [[IP-Adressen, Ports und Sockets - Grundlagen|IP-Adressen, Ports und Sockets – Grundlagen]] · [[Musterlösung (Docker)|Musterlösung (Docker)]]

# Inhalt

- [[#A. Arbeiten mit der CLI]]
- [[#B. Netzwerk]]
- [[#C. Images bauen]]
- [[#D. Benutzer und Rechte]]
- [[#E. Compose und YAML]]
- [[#F. Die Anwendung selbst]]

---

# A. Arbeiten mit der CLI

## Warum zeigte `docker build -t x - <<EOF` nur „transferring context: 2B"?

Das `-` als letztes Argument heißt: **der Build-Kontext kommt von stdin**. Es wurde also nur das Dockerfile übertragen und kein einziges Projektfile — die Messung war wertlos.

Richtig ist `-f -` (Rezept von stdin, Kontext bleibt der Punkt) **plus** ein `COPY`, weil BuildKit **faul** überträgt und ohne benutzenden Befehl gar nichts schickt:

```bash
docker build --progress=plain --no-cache -f - . <<'EOF'
FROM alpine
COPY . /ctx
RUN du -sh /ctx && du -sh /ctx/* /ctx/.[!.]* 2>/dev/null | sort -h
EOF
```

## Wie komme ich aus einer Container-Shell wieder raus?

`exit` oder **Strg+D**.

- **Strg+C** schickt `SIGINT` an den Vordergrundprozess — die Shell *ignoriert* das und gibt nur einen neuen Prompt.
- `q` und `/exit` sind keine Shell-Befehle, sondern Pager-/Editor-Konventionen (`less`, `vim`).
- **Strg+D** sendet **EOF auf stdin**, und darauf beendet sich jede Shell.

Derselbe Mechanismus erklärt, warum `docker run debian:13` sofort mit `Exited (0)` zurückkehrt: Ohne `-i` bekommt die Shell sofort EOF.

## Muss ich nach Experimenten etwas aufräumen?

Jeder Build **ohne `-t`** hinterlässt ein namenloses Image (`<none>:<none>`, „dangling"), dazu Build-Cache. Erst schauen, dann gezielt löschen:

```bash
docker system df               # wo liegt der Platz?
docker image prune             # namenlose Images
docker builder prune           # BuildKit-Cache (oft der größte Posten)
```

⚠️ **Nicht** `docker volume prune` oder `docker system prune --volumes` — die räumen auch Volumes ab.

## Warum sollte ich `docker build … | head -3` vermeiden?

`head` schließt die Pipe nach drei Zeilen, Docker bekommt `SIGPIPE` und kann mitten im Build abbrechen. Zum Filtern `grep` nehmen — das liest bis zum Ende.

---

# B. Netzwerk

## Ist `0.0.0.0` dasselbe wie localhost?

**Nein.** `127.0.0.1` ist eine **konkrete Adresse** (die virtuelle Schnittstelle `lo`, die immer existiert). `0.0.0.0` ist **keine Adresse**, sondern ein Platzhalter **beim Lauschen**: „auf allen Schnittstellen annehmen". Man verbindet sich nie *zu* `0.0.0.0`.

Ein Container hat zwei Schnittstellen: `lo` und `eth0`. Verkehr von außen kommt über **`eth0`** — ein Server auf `127.0.0.1` nimmt ihn gar nicht erst an. Daher im Container **immer `--host 0.0.0.0`**, auf dem Laptop dagegen `127.0.0.1`.

→ Grundlagen-Notiz, Abschnitte 3 und 6

## Woher kommt `127.0.0.1`, und warum darf ich einfach darauf lauschen?

Das ganze Netz `127.0.0.0/8` ist per Standard für **Loopback** reserviert. Der Kernel stellt dafür die virtuelle Schnittstelle **`lo`** bereit, die **immer existiert und immer aktiv ist** — ohne Netzwerkkarte, Kabel, WLAN oder DHCP. Deshalb funktioniert es auch offline.

`localhost` ist dabei kein Schlüsselwort, sondern eine Zeile in `/etc/hosts`.

## Warum `-p 8000:8000`, und warum ausgerechnet 8000?

**Links = Host, rechts = Container.** Die linke Zahl ist **frei wählbar**, die rechte muss zu dem passen, worauf der Prozess im Container lauscht. Die 8000 im Container ist bei dir nur uvicorns **Default**, weil im `CMD` kein `--port` stand — deshalb ist es besser, ihn explizit hinzuschreiben.

Ports **unter 1024** darf nur root binden; darum 8000 statt 80.

## `address already in use` — warum?

Zwei Programme wollen dasselbe **(IP, Port)-Paar**. `docker run -p 8000:8000` bindet auf dem Host an `0.0.0.0:8000`, und `0.0.0.0` **schließt `127.0.0.1` ein** — ein dort laufendes uvicorn kollidiert also.

| A | B | Kollision? |
|---|---|---|
| `0.0.0.0:8000` | `127.0.0.1:8000` | ✅ ja |
| `127.0.0.1:8000` | `192.168.178.42:8000` | ❌ nein |

Finden: `ss -tlnp | grep :8000`

## Docker lauscht doch nur — wie kommt die Antwort zurück?

Zwei Gründe:

1. **„Lauschen" betrifft nur den Verbindungsaufbau.** `accept()` liefert einen **neuen, eigenen Socket** für diese eine Unterhaltung, und der ist **beidseitig**. Die Antwort geht durch die bestehende Verbindung, nicht „an einen Port zurück".
2. **NAT ist zustandsbehaftet.** Der Kernel merkt sich jede Übersetzung (*conntrack*) und **dreht sie auf dem Rückweg automatisch um**. Das muss so sein — käme die Antwort sichtbar von `172.18.0.2:8000`, würde der Client sie verwerfen.

Nebenbei lauscht dort tatsächlich ein Prozess: **`docker-proxy`** (`ss -tlnp | grep 8000`).

## Woher weiß ich, welches Netzwerk das richtige ist?

`bridge`, `host` und `none` sind Standard-Netze **jeder** Docker-Installation. Compose legt zusätzlich **`<projektname>_default`** an — der Projektname kommt aus dem Verzeichnisnamen, solange kein `name:` gesetzt ist.

```bash
docker inspect pricing-db --format '{{json .NetworkSettings.Networks}}' | jq
```

## Was genau tut `--network <netz>`?

Vier Schritte:

1. **Kabel gebaut** — ein `veth`-Paar erzeugt
2. **Eingesteckt** — Ende A in die Bridge, Ende B als `eth0` in den Container
3. **Adresse vergeben** — die nächste freie aus dem Subnetz
4. **Namen eingetragen** — in die DNS-Tabelle dieses Netzes

Ohne 1–2 hinge der Container am falschen Switch; ohne 4 wäre `db` ein unbekannter Name.

## Was ist eine Bridge?

Ein **virtueller Switch** im Kernel. Ein Switch schiebt Pakete zwischen Buchsen hin und her, ohne zu routen oder zu übersetzen — er arbeitet **unter** IP, mit MAC-Adressen.

Zwei Folgen: Alles am selben Switch erreicht sich direkt; was an einem **anderen** Switch hängt, nicht. **Jedes Docker-Netzwerk ist ein eigener Switch** — deshalb sehen sich Container verschiedener Netze nicht.

## Was heißt „im DNS des Netzes registriert"?

Container-IPs sind **nicht stabil** — bei jedem Neustart kann eine andere vergeben werden. Deshalb betreibt Docker in jedem Container-Namespace einen kleinen **DNS-Server unter `127.0.0.11`** (`cat /etc/resolv.conf`) mit einer Tabelle *Name → aktuelle IP*.

„Registriert" heißt schlicht: Docker hat den Container beim Anschließen dort eingetragen. Im vorgegebenen `bridge`-Netz gibt es diesen Dienst **nicht** — daher funktioniert Namensauflösung nur in selbst angelegten Netzen.

## Woher kommt `docker_default`?

Die hat **Compose beim ersten `up` angelegt** — obwohl in der Datei kein Wort über Netzwerke steht. Compose legt pro Projekt eines an (`<projekt>_default`), hängt alle Dienste hinein und registriert sie im DNS.

Warum nicht das vorgegebene `bridge`? Weil es dort **keine Namensauflösung** gäbe und keine **Isolation zwischen Projekten**.

Lebensdauer: `stop` lässt es bestehen, **`down` entfernt es**.

## Warum funktioniert `DB_HOST: db` einfach so?

Drei Schritte hintereinander:

```
compose.yml   environment: { DB_HOST: db }
      ↓       Compose setzt die Umgebungsvariable im Container
config.py     pydantic-settings bildet Feld db_host ← DB_HOST ab
      ↓
db.py         URL.create(..., host="db", ...)
      ↓
psycopg       DNS-Anfrage an 127.0.0.11  →  172.18.0.2
```

Der Hostname ist der **Dienstname** aus der Compose-Datei — nicht `container_name`, nicht der Image-Name.

---

# C. Images bauen

## Block 5 war schneller — aber warum eigentlich?

Zunächst gar nicht wegen des Cachings: Ein übrig gebliebenes `COPY . .` **vor** den gezielten `COPY`s machte die Optimierung wirkungslos. Der gemessene Gewinn kam von **`--no-dev`** (pytest, ruff, mypy fielen weg).

**Der Cache ist eine Kette:** Der Schlüssel jeder Anweisung enthält den Schlüssel der vorherigen. `COPY . .` reagiert auf **jede** Dateiänderung — bricht es, sind alle folgenden Schichten ungültig.

**Beweisen statt messen:** Auf das Wort **`CACHED`** in der Build-Ausgabe achten. Es muss beim teuren `uv sync` stehen.

## Was tut `--mount=type=cache,target=…`?

BuildKit hängt für die Dauer *dieses einen* `RUN` ein Verzeichnis ein, das **nicht zur Schicht gehört**, aber **zwischen Builds überlebt**.

**Unterschied zum Layer-Cache** — sie ergänzen sich:

| | Layer-Cache | Cache-Mount |
|---|---|---|
| Bewirkt | Schritt wird **übersprungen** | Schritt **läuft**, aber mit warmem Cache |
| Greift | wenn sich nichts geändert hat | wenn der Layer-Cache **verfehlt** |

Ändert sich `uv.lock`, läuft `uv sync` neu — muss die 40 unveränderten Pakete aber nicht erneut herunterladen. Ohne Mount landete der Download-Cache dauerhaft im Image.

`UV_LINK_MODE=copy` gehört dazu, weil Hardlinks nicht über die Mount-Grenze funktionieren.

## `ModuleNotFoundError: No module named 'pricing'`

Zwei getrennte Ursachen mit identischem Symptom:

1. **`--no-install-project` stand bei *beiden* `uv sync`.** Der Sinn der Zweiteilung ist ja, dass der zweite Lauf das eigene Paket installiert. → Flag im zweiten Aufruf entfernen.
2. **Editable Install.** Standardmäßig legt uv nur einen **Verweis auf `/app/src`** ins `.venv` — und `src/` wird nicht in die Runtime-Stage kopiert. → **`--no-editable`**, dann liegt der Code echt in `site-packages`.

Prüfen: `docker run --rm <image> python -c "import pricing; print(pricing.__file__)"` muss einen Pfad unter `site-packages` zeigen.

## Was ist der Unterschied zwischen `RUN` und `CMD`?

**`RUN` = Fließband, `CMD` = Einschaltknopf.**

| | `RUN` | `CMD` |
|---|---|---|
| Wann | beim **Bauen** | beim **Starten** |
| Ergebnis | eine **neue Schicht** | ein **Eintrag in der Config** |
| Überschreibbar | nein, eingebacken | ja, durch Argumente nach dem Image |
| Anzahl | beliebig viele | nur die **letzte** zählt |

`RUN uvicorn …` würde den Server *während des Builds* starten und ewig hängen. `CMD ["uv","sync"]` würde bei *jedem* Containerstart neu installieren.

Dass `CMD` überschreibbar ist, nutzt der `migrate`-Dienst: dasselbe Image, `command: ["alembic","upgrade","head"]`.

## Wie funktioniert `HEALTHCHECK`?

Docker führt in Abständen einen Befehl **im Container** aus und wertet nur den **Exit-Code** aus (`0` = gesund).

| Parameter | Bedeutung |
|---|---|
| `--interval` | Abstand zwischen zwei Prüfungen |
| `--timeout` | wie lange eine Prüfung dauern darf |
| `--retries` | Fehlschläge in Folge bis `unhealthy` |
| `--start-period` | **Karenzzeit** — Fehlschläge zählen hier **nicht** gegen `retries` |

Zustände: `starting` → `healthy` / `unhealthy`. Der erste Erfolg beendet die Karenzzeit vorzeitig.

> **Docker tut von sich aus nichts damit.** Ein `unhealthy`-Container wird **nicht** neu gestartet. Der Healthcheck ist ein **Signal**, das andere auswerten: `depends_on: condition: service_healthy`, `up --wait`, Kubernetes.

Protokoll der letzten Prüfungen: `docker inspect --format '{{json .State.Health}}' <c> | jq`

## Warum ein Python-Einzeiler statt `curl` im Healthcheck?

`python:3.13-slim` enthält weder `curl` noch `wget`. Nachinstallieren kostet ~15 MB und Angriffsfläche für eine einzige HTTP-Anfrage, die Python ohnehin kann. Geprüft wird `127.0.0.1` — hier ist localhost richtig, weil Prüfer und Geprüfter derselbe Container sind.

---

# D. Benutzer und Rechte

## Was bedeutet `1000:1000`, und muss der Benutzer einen Namen haben?

Format **`UID:GID`**. **Nein, ein Name ist nicht nötig.**

Der Kernel speichert bei jeder Datei nur zwei **Integer** und vergleicht sie mit denen des Prozesses. Namen sind Zeilen in `/etc/passwd` und `/etc/group` — ein **Nachschlagewerk für Menschen**. Ohne Eintrag brechen nur Komfortfunktionen (`whoami`, `$HOME`, lesbares `ls -l`).

**Warum 1000?** Konvention: `0` = root, `1–999` = Systemkonten, **ab 1000** = menschliche Benutzer. Der erste Benutzer eines Debian-Systems hat UID 1000 — dieselbe Zahl im Image lässt bei Bind Mounts die Rechte zusammenpassen.

## Wie weiß der Container, dass er als dieser Benutzer laufen soll?

**Gar nichts wird nachgeschlagen oder automatisch gewählt.** `USER` landet als String in der Image-Config; beim Start ruft `runc` vor dem `execve()` einfach `setgid(1000); setuid(1000)` auf — ein Zahlenfeld im Prozess-Eintrag.

```bash
docker run --rm -u 4711:4711 python:3.13-slim id    # geht, obwohl 4711 nirgends existiert
docker run --rm -u gibtsnicht python:3.13-slim id   # Error: unable to find user
```

Nur bei **Namen** liest Docker `/etc/passwd` im Image — und scheitert ohne Eintrag.

## Warum in der Musterlösung `useradd app`, aber dann `USER 1000:1000`?

Zwei verschiedene Vorgänge: `RUN useradd` **legt den Eintrag an** (Dateisystem), `USER` **wechselt die Identität** (Config). Angelegt wird einmal, benutzt einmal.

Die Zahl statt des Namens, weil:

1. **Kubernetes** mit `runAsNonRoot: true` einen Namen nicht auf Nicht-root prüfen *kann* und den Start verweigert
2. es **unabhängig vom Dateisystem** ist (wichtig bei Multi-Stage)
3. die **Gruppe explizit** dasteht

## Warum `--chown` am `COPY` und nicht `RUN chown -R`?

`chown` ändert die **Metadaten jeder einzelnen Datei** — und eine geänderte Datei landet **vollständig** in der neuen Schicht. Ein rekursives `chown` über ein 150-MB-`.venv` **verdoppelt dessen Platzbedarf im Image**, ohne dass sich ein Byte Inhalt ändert.

Und: **`USER` gilt für alles danach** — gehört ans **Ende** der Runtime-Stage.

## Was war nochmal die PID-1-Falle?

Es sind **zwei**:

| | **A: Signale** | **B: Zombies** |
|---|---|---|
| Ursache | **Shell-Form** von `CMD` macht `/bin/sh` zu PID 1; sie reicht `SIGTERM` **nicht weiter** | PID 1 muss verwaiste Kindprozesse per `wait()` einsammeln — die Anwendung tut das nicht |
| Symptom | `docker stop` dauert **immer 10 s**, hartes Ende, DB-Recovery | Prozesstabelle füllt sich langsam mit Zombies |
| Lösung | **exec-Form**, oder Skript mit **`exec "$@"`** | **`init: true`** / `docker run --init` (tini) |
| Bei uns relevant | **ja** | nein, uvicorn forkt nicht |

Verschärfend: Der Kernel stellt Signale mit Standardverhalten an **PID 1 gar nicht zu**, solange kein eigener Handler registriert ist.

---

# E. Compose und YAML

## Wann Einrückung, wann `-`, wann `[]`?

YAML kennt nur **zwei** Strukturen; der Rest ist Schreibweise.

- **Mapping** (Schlüssel → Wert): eingerückte `schlüssel: wert`-Zeilen
- **Sequence** (Liste): Einträge mit `-`

`[ "a", "b" ]` ist der **Flow-Stil** und für den Parser **identisch** mit der `-`-Variante (Block-Stil).

| Schlüssel | Struktur |
|---|---|
| `services`, `volumes` (oben) | Mapping |
| `ports`, `profiles` | Liste |
| `command`, `entrypoint`, `healthcheck.test` | Liste |
| `environment` | **beides erlaubt** (`KEY: wert` oder `- KEY=wert`) |
| `depends_on` | **Liste** (Kurzform) **oder Mapping** (Langform mit `condition`) |

Deshalb musste `depends_on` in Block 9 von `-` auf Einrückung umgebaut werden: `condition` lässt sich in der Listenform nicht ausdrücken.

## Warum muss `restart: "no"` in Anführungszeichen?

In YAML 1.1 sind **`no`, `yes`, `on`, `off`** Schreibweisen für **Booleans**. Ohne Anführungszeichen liest der Parser `restart: false`, Compose erwartet aber eine Zeichenkette.

| Wert | Verhalten |
|---|---|
| `"no"` | nie neu starten (Default) — **Pflicht für Einmal-Dienste** wie `migrate` |
| `on-failure` | nur bei Exit ≠ 0 |
| `always` | immer, auch nach `docker stop` |
| `unless-stopped` | wie `always`, außer du hast selbst gestoppt — **richtig für `api`** |

## Was ist diese `x-app-env: &app-env`-Syntax?

**YAML-Anker**, kein Compose-Feature:

- **`&name`** — „merk dir diesen Block"
- **`*name`** — „setz ihn hier ein"
- **`<<: *name`** — einsetzen **und** ergänzen/überschreiben

Der Parser löst das auf, *bevor* Compose die Datei sieht. Das **`x-`**-Präfix markiert ein **Extension Field**, das Compose bei der Validierung ignoriert — ohne das gäbe es einen Fehler wegen eines unbekannten Schlüssels.

Kontrolle: `docker compose config` zeigt die aufgelöste Datei.

## Named Volume oder Bind Mount — woran erkennt Docker das?

In der Kurzform **rein am ersten Zeichen links**:

| Beginnt mit | Ergebnis |
|---|---|
| `.` oder `/` | **Bind Mount** |
| alles andere | **Named Volume** |
| *(nur ein Pfad, kein Doppelpunkt)* | **anonymes Volume** |

`pricing_data:/var/lib/postgresql` und `./pricing_data:/var/lib/postgresql` unterscheiden sich um **zwei Zeichen**.

Die **Langform** (`type: volume|bind|tmpfs`) macht es explizit — wichtig, weil ein **Tippfehler im Host-Pfad** in der Kurzform stillschweigend ein neues, leeres, root-gehörendes Verzeichnis erzeugt.

## Wofür der `volumes:`-Block ganz am Ende?

Er **deklariert** Named Volumes, damit Compose sie verwalten kann: anlegen beim `up`, Projekt-Präfix davor, entfernen bei `down -v`. Fehlt er: `refers to undefined volume`.

Bind Mounts gehören dort **nicht** hin — sie sind keine Docker-Objekte.

## `external: true` vs. `internal: true`

Zwei völlig verschiedene Dinge mit ähnlichen Namen:

| | `external: true` | `internal: true` |
|---|---|---|
| Gilt für | Volumes, Netzwerke, Secrets, Configs | **nur Netzwerke** |
| Regelt | **Besitz** | **Erreichbarkeit** |
| Bedeutet | „Compose verwaltet das nicht" — kein Anlegen, kein Präfix, **kein Löschen bei `down -v`** | „kein Gateway nach draußen" — weder Internet noch Host |
| Typisch für | geteilte Volumes, **Produktionsdaten schützen** | `db` nur im `backend`-Netz, `api` in beiden |

Bei `internal` verliert ein `ports:`-Eintrag seinen Sinn; Zugriff dann über `docker compose exec db psql`.

## `down` meldet „Network … Resource is still in use" — warum?

Weil beim `up` ein **Profil** aktiv war und beim `down` nicht:

```bash
docker compose --profile dev up -d     # startet auch adminer
docker compose down -v                 # ← sieht adminer gar nicht
```

Compose bezieht bei **jedem** Befehl nur Dienste ein, deren Profile **aktiv** sind. Der `adminer`-Container lief also weiter, hing weiter im Projektnetz — und ein Netzwerk lässt sich nicht entfernen, solange ein **laufender** Container daran hängt. Container und Volume waren da längst weg, nur das Netz blieb.

(Ob man die Weboberfläche geöffnet hatte, spielt keine Rolle — es geht um den *angeschlossenen Container*, nicht um eine offene Verbindung.)

**Richtig:**

```bash
docker compose --profile dev down -v      # dieselben Profile wie beim up
docker compose --profile "*" down -v      # alle Profile
docker compose down -v --remove-orphans   # räumt auch auf, was nicht ins Bild passt
```

> **Merkregel: `down` muss dieselben Profile kennen wie `up`.** Dauerhaft lösen lässt sich das mit `COMPOSE_PROFILES=dev` in der `.env` — dann gilt es für alle Befehle im Projekt.

## `docker container stop adminer` sagt „No such container" — warum?

Weil `adminer` der **Dienstname** ist und `pricing-adminer` der **Containername**.

| Ebene | Name | Verwendet in |
|---|---|---|
| Dienst | `adminer` | `docker compose <befehl> adminer` |
| Container | `pricing-adminer` (durch `container_name`) | `docker <befehl> pricing-adminer` |

**`docker` kennt keine Dienste, nur Container.** Ohne `container_name` hieße der Container `pricing-adminer-1` (Projekt + Dienst + Nummer) — deshalb ist es meist bequemer, `container_name` wegzulassen: Dann ist der Containername aus dem Dienstnamen ableitbar statt eine zweite, unabhängige Bezeichnung.

## Wie halte ich das DB-Passwort aus der Compose-Datei heraus?

**Variablen-Substitution** aus `.env` (liegt bereits in `.gitignore`):

```yaml
DB_PRICING_PW: ${DB_PRICING_PW:?DB_PRICING_PW fehlt in .env}
```

| Syntax | Wenn die Variable fehlt |
|---|---|
| `${VAR}` | **leerer String** — still und gefährlich |
| `${VAR:-standard}` | Standardwert |
| **`${VAR:?meldung}`** | **Compose bricht ab** ← nehmen |

Dazu eine eingecheckte `.env.example` (mit `!.env.example` in der `.gitignore`).

> ⚠️ Das hält es aus **git** heraus, macht es aber **nicht geheim** — der Wert steht weiter in `docker inspect`. Echte Geheimhaltung: **Docker Secrets** als Datei unter `/run/secrets/…` (offizielle Images unterstützen dafür die `_FILE`-Konvention, z. B. `POSTGRES_PASSWORD_FILE`). Für die eigene Anwendung bräuchte es `SettingsConfigDict(secrets_dir=...)`.

---

# F. Die Anwendung selbst

## Wie löse ich Block 11 (Logs sichtbar machen)?

Zwei Teile, einer war schon erledigt:

1. **`LOG_FILE` nicht setzen** → `setup_logging()` legt den `RotatingFileHandler` gar nicht erst an, übrig bleibt der Konsolen-Handler nach **stderr**, und `docker logs` fängt stdout **und** stderr ab. Das Level passt bereits, weil die API `setup_logging(console_level="INFO", …)` im `lifespan` aufruft.
2. **`PYTHONUNBUFFERED=1`** ins Dockerfile — das ist die eigentliche Arbeit.

**Warum:** `stderr` ist seit Python 3.9 zeilengepuffert und erscheint sofort. **`stdout` ist blockgepuffert (~8 KB), wenn es kein TTY ist** — und uvicorns **Access-Logs gehen nach stdout**. Ohne die Variable tauchen sie verspätet auf, bei einem Absturz gar nicht.

**Grundsätzlich:** Ein Container soll sich nicht selbst um Logrotation kümmern. Er schreibt nach stdout/stderr; was damit passiert, entscheidet der Logging-Treiber des Daemons.

## Wie kommt der Container an `daily_data`?

**Gar nicht.** `daily_data/` steht in `.dockerignore` (also nicht im Image) und ist nirgends als Bind Mount eingebunden.

Folge ist **kein Absturz**, sondern ein sauberer Fachfehler: `resolve_ingest_dir()` prüft `ziel.is_dir()` und wirft `IngestFolderMissingError`. Die API läuft, `/v1/health` ist grün, alle DB-Endpunkte funktionieren — nur der Ingest meldet „Ordner fehlt". Deshalb fiel es nicht auf.

Falls gewünscht:

```yaml
  api:
    volumes:
      - ./daily_data:/app/daily_data:ro
```

`:ro` ist richtig — lesen ja, schreiben nein. Rechte passen, weil Host-Dateien und Container-Prozess beide UID 1000 sind.

## Was schickt `/docs` eigentlich zurück — eine statische HTML-Seite?

HTML **ja**, statische Datei **nein**. FastAPI ruft `get_swagger_ui_html()` auf — eine **Python-Funktion**, die ~25 Zeilen HTML als String baut. Die Seite ist **fast leer**: ein `<div id="swagger-ui">`, zwei **CDN**-Verweise und `SwaggerUIBundle({url: "/openapi.json"})`.

Die Substanz kommt aus `/openapi.json` und ist **JSON**. **`/docs` ist ein Client deiner eigenen API.**

Zwei Folgen: Das **Internet braucht der Browser**, nicht der Server (isolierter Container → `/docs` geht trotzdem; offline → weiße Seite). Und `/docs`/`/openapi.json` sind **standardmäßig öffentlich**, auch wenn alle Router Auth verlangen.

→ ausführlich in [[1.5 REST APIs & Layered Architecture - Vertiefung|1.5]], Stichwort *Swagger & OpenAPI*

---

# Zwei Fehler, die zweimal auftraten

Beide waren **stille** Fehler — nichts brach ab, es passierte nur nicht das Gewünschte:

| Fehler | Wirkung |
|---|---|
| `PYTHONUNBUFFERD=1` (Tippfehler) | eine Variable, die niemand liest → Block 11 wirkungslos |
| `start_perios: 10s` (Tippfehler) | keine Karenzzeit im Healthcheck |
| `COPY . .` vor den gezielten `COPY`s | Layer-Caching wirkungslos |
| `--no-install-project` in **beiden** `uv sync` | eigenes Paket nie installiert |

**Lehre daraus:** Bei Docker prüft man das *Ergebnis*, nicht die Absicht. `docker compose config`, das Wort `CACHED` in der Build-Ausgabe, `docker compose exec api env` und `docker inspect` sind dafür da.
