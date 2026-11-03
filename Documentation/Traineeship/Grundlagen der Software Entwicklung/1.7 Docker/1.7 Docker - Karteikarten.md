---
deck: "Traineeship::Grundlagen der Software Entwicklung::Docker"
---

Karteikarten zu [[1.7 Docker - Vertiefung|1.7 Docker – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::Docker`. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr, bei dünnem weniger — nicht auf eine Zielzahl auffüllen. Details, die nicht auf eine Karte passen, stehen in der Vertiefung.

# Fortschritt

- [x] Warum Containerisierung? (inkl. Kontrollfrage *Container vs. VM*)
- [x] Docker-Architektur: Docker Engine, Docker Daemon, Docker Client
- [x] Docker-Objekte: Images, Container, Volumes, Networks (inkl. Kontrollfrage *Image vs. Container*)
- [x] Docker CLI: Grundlegende Befehle und Anwendung
- [x] Dockerfile: Struktur und Best Practices (inkl. Kontrollfragen *Image-Erstellung* & *Multi-Stage*)
- [x] Docker Volumes und Datenpersistenz (inkl. Kontrollfrage *Datenpersistenz*)
- [x] Docker Compose: Multi-Container-Anwendungen (inkl. Kontrollfrage *Vorteile von Compose*)
- [x] Fragen aus der Umsetzung (siehe [[Fragen aus der Umsetzung|Q&A/Fragen aus der Umsetzung]]) — Karten sind in die Abschnitte oben einsortiert

---

# Warum Containerisierung?

<!-- basicblock-start oid="ObsB8YxPlS8emdeS0uHlXyCK" -->
Was ist ein **Container**? (präzise Definition)::
**Ein ganz normaler Prozess auf dem Host-Kernel**, der mit einer eingeschränkten Sicht auf das System und einem begrenzten Ressourcenbudget gestartet wurde.<br>Kein eigener Kernel, kein Boot, keine Virtualisierung. Der Prozess taucht auf dem Host in `ps aux` mit normaler PID auf — er weiß nur nichts davon.<br>Es gibt **kein** Kernel-Feature namens „Container": Der Begriff bezeichnet die Kombination mehrerer Linux-Bausteine.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGSFqzOchy7qy8JhP7HEAY" -->
Nenne die **vier Bausteine**, aus denen ein Container besteht — und was jeder regelt.::
1. **Namespaces** — *was der Prozess sieht* (`pid`, `mnt`, `net`, `uts`, `ipc`, `user`, `cgroup`)<br>2. **cgroups** — *wie viel er bekommt* (CPU, RAM, I/O, Prozessanzahl)<br>3. **Union-Dateisystem (OverlayFS)** — *woraus sein Dateisystem besteht* (gestapelte read-only-Schichten + dünne beschreibbare Schicht)<br>4. **Capabilities / seccomp / AppArmor bzw. SELinux** — *was er darf* (Syscalls und Sonderrechte)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuIDeOkZcQxGTAb27uPa40" -->
Unterschied **Namespace** vs. **cgroup**?::
**Namespace = Sicht:** „Du siehst nur diesen Ausschnitt der Welt" (eigene Prozessliste, eigener Dateibaum, eigener Netzwerk-Stack).<br>**cgroup = Menge:** „Du darfst höchstens 512 MB RAM und eine halbe CPU verbrauchen."<br>Ohne cgroups könnte ein Container mit Speicherleck den ganzen Host in den OOM-Killer treiben — er *sieht* nichts anderes, aber er *nimmt* sich alles.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZNnzoVly8VQtTdVQIc7WW" -->
Wie unterscheidet sich ein **Container** von einer **VM**? Nenne den strukturellen Kern, nicht nur „leichter".::
Sie virtualisieren auf **verschiedenen Ebenen**:<br>**VM** virtualisiert die **Hardware** — ein Hypervisor stellt CPU/RAM/Platte/NIC nach, darauf bootet ein vollständiges Gast-OS **mit eigenem Kernel**.<br>**Container** virtualisiert das **Betriebssystem-Interface** — es gibt nichts zu booten, weil der Kernel schon läuft; der Kernel spielt dem Prozess nur eine eingeschränkte Weltsicht vor.<br>Der **geteilte Kernel** ist der gesamte Unterschied; alles andere (Größe, Startzeit, Dichte, Isolationsgrad) folgt daraus.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs206GerIJWNK9N4gBvkZ1J" -->
Welche **Konsequenzen** folgen aus dem geteilten Kernel? (3 Vorteile, 2 Nachteile)::
**Vorteile:** Startzeit 50–500 ms statt Minuten (Prozessstart statt Boot) · Größe in MB statt GB · Dichte von Hunderten statt ~10–20 pro Host, weil kein RAM fest reserviert wird.<br>**Nachteile:** **Schwächere Isolation** — eine Kernel-Lücke gefährdet *alle* Container (Ausbruch = Kernel kompromittiert, nicht Hypervisor) · das **Gast-OS muss zum Host-Kernel passen**: Linux-Container brauchen einen Linux-Kernel, Windows-Software geht nur per VM.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseStp2MbHmlr6aUUbaxq2D" -->
Wann nimmt man trotzdem eine **VM** statt eines Containers?::
1. **Anderes Betriebssystem** nötig (Windows auf Linux-Host) — geht mangels eigenem Kernel prinzipiell nicht im Container.<br>2. **Harte Sicherheitsgrenze zwischen fremden Mandanten** — beliebiger fremder Code (Cloud-Anbieter, CI für fremde PRs); Zwischenlösungen: Firecracker, gVisor, Kata Containers.<br>3. **Eigener Kernel / Kernelmodul** nötig (Treiber, andere Kernel-Version, Echtzeit-Patches).<br>4. **Compliance**, die eine Hardware-Isolationsgrenze verlangt.<br>Merke: kein Entweder-oder — in der Praxis laufen Container fast immer *in* VMs (jeder K8s-Knoten ist eine VM).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsH2Air4i9gHwo4QPJZrUCH" -->
Was schreibt ein **Paketmanager** (`uv.lock`) fest, was ein **Container** — welche Abhängigkeitsschichten deckt wer ab?::
**Paketmanager: was dein Programm importiert.** **Container: worauf dein Programm läuft.**<br>Schichten: 1 Anwendungscode (git) · 2 Python-Pakete (`uv.lock`) · 3 Interpreter · **4 Systembibliotheken (glibc, libpq, libssl)** · **5 OS-Userland** · 6 Kernel · 7 CPU-Architektur.<br>Ein Image packt **Schicht 1–5** ein. Schicht 6 wird mit dem Host **geteilt** (daher Größen- und Geschwindigkeitsvorteil). Schicht 7 bleibt ein echtes Problem (amd64 ≠ arm64).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9OUYiMnqzQEsIMWpsofuo" -->
Was ist ein **Union-Dateisystem (OverlayFS)** im Docker-Kontext — und welche zwei Konsequenzen hat es?::
Ein Image besteht aus **gestapelten, schreibgeschützten Schichten**; beim Start legt Docker eine **dünne beschreibbare Schicht** obendrauf, OverlayFS blendet alles zu einem Dateibaum zusammen.<br>**Konsequenz 1:** Schichten werden **geteilt** — zehn Container aus demselben Image belegen die Basis nur *einmal* auf der Platte. Deshalb ist „ein Container pro Dienst" bezahlbar, „eine VM pro Dienst" nicht.<br>**Konsequenz 2:** Änderungen im Container sind **flüchtig** — sie liegen nur in der obersten Schicht und verschwinden mit `docker rm`. Daraus entsteht die Notwendigkeit von **Volumes**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJTDZb1ZBSQHSgYy3kgppg" -->
Warum war **Docker** 2013 erfolgreich, obwohl es technisch nichts Neues enthielt?::
Namespaces, cgroups und Union-Dateisysteme gab es alle schon (LXC seit 2008) — sie waren nur **unbenutzbar**: seitenweise Konfiguration und kein Weg, eine fertige Umgebung *weiterzugeben*.<br>Dockers Beitrag sind **drei Dinge, die zusammen ein Ökosystem ergeben**:<br>1. **Image-Format mit Schichten** → Umgebungen werden kopier- und wiederverwendbar<br>2. **Dockerfile** → der Bauplan ist Text, liegt im Repo, durchläuft Code Review<br>3. **Registry** (Docker Hub) → Umgebungen bekommen eine Adresse<br>Punkt 3 ist der unterschätzte: Erst dadurch wurde `docker run postgres:18` möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrYMJvtvQZ1gdWvi7AnNw7" -->
Was ist **Umgebungs-Drift** und was der **Snowflake-Server**?::
**Umgebungs-Drift:** Maschinen, die dieselbe Umgebung haben *sollten* (lokal, CI, Staging, Prod), haben sie nicht — das Symptom ist der Satz „bei mir läuft's" (technisch korrekt, aber nicht übertragbar). Analog zur **Schema-Drift** aus Kapitel 1.6.<br>**Snowflake-Server:** ein langlebiger Server *mit Geschichte*, der über Jahre nachjustiert statt neu gebaut wurde — einzigartig, gewachsen, nicht reproduzierbar. Der konzeptionelle Restfehler von Konfigurationsmanagement (Puppet/Chef/Ansible).<br>Die Antwort darauf heißt **Immutable Infrastructure**: nicht reparieren, sondern ersetzen („Cattle, not Pets").
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCzSi7xJZjhVqjqUdLAtnl" -->
Nenne die **fünf Hauptvorteile** der Containerisierung.::
1. **Umgebungsparität** — dasselbe *Artefakt* läuft lokal, in CI und in Prod (ein Skript beschreibt einen *Weg* und endet überall anders; ein Image ist ein *Ergebnis*).<br>2. **Isolation zwischen Projekten** — PostgreSQL 15 und 18 nebeneinander, ohne das System zu verschmutzen.<br>3. **Wegwerfbarkeit** — `down && up` stellt einen exakt definierten Zustand in Sekunden wieder her.<br>4. **Onboarding/Reproduzierbarkeit** — ein Befehl statt einer Seite README; der einzige belastbare Test, ob das Projekt wirklich *beschrieben* ist.<br>5. **Voraussetzung für CI/CD, Skalierung, Kubernetes** — sie alle brauchen ein standardisiertes, startbares Artefakt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGBq2r98FxxTKdyuJpHkRu" -->
Warum ist ein Dockerfile mit `FROM python:3.13` **nicht** automatisch reproduzierbar?::
Ein Tag ist ein **beweglicher Zeiger**: Hinter `python:3.13` steht heute ein anderes Image als in vier Wochen (Patches, neue Basis-Distribution). Dasselbe Dockerfile baut damit unterschiedliche Ergebnisse.<br>Reproduzierbarkeit erfordert **Disziplin**: möglichst genaue Tags, besser ein **Digest** (`python:3.13-slim@sha256:…`) — plus ein Lockfile (`uv sync --frozen`) für die Paketebene.<br>Merke: Docker gibt dir die *Möglichkeit* zur Reproduzierbarkeit, nicht die Garantie.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsq3w2zHU9JAE2HB3jejFDG" -->
Drei häufige **Missverständnisse** über Docker — und die Korrektur.::
„Container sind sicher wie VMs" → **Nein**, geteilter Kernel; bei fremdem Code gehört eine VM darunter.<br>„Docker macht Deployments zustandslos" → **Nein**, es verschiebt Zustand nur an eine *explizit benannte* Stelle (Volume, externe DB). Das ist der Fortschritt — das Problem bleibt.<br>„Docker ersetzt Migrations-Tools" → **Nein**. Ein Image kann eine *frische* DB initialisieren; eine *bestehende* fortschreiben kann nur Alembic. Beides zusammen ist der Normalfall.<br>Bonus: „Docker ist die einzige Option" → Das **Format** ist standardisiert (**OCI**); Podman, containerd/nerdctl und Buildah bauen und starten dieselben Images.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHRu4QURQE8gitsPHElNyZ" -->
Warum ist der **Hauptprozess im Container PID 1** ein praktisches Problem?::
Der `pid`-Namespace macht den Hauptprozess zu **PID 1** — und PID 1 hat unter Linux Sonderrechte *und* Sonderpflichten: Sie **ignoriert per Default Signale, für die kein Handler registriert ist**.<br>Folge: Ein falsch gestarteter Container reagiert nicht auf `docker stop`/`SIGTERM` und wird nach 10 Sekunden hart abgeschossen (`SIGKILL`) — ohne sauberes Herunterfahren.<br>Kommt beim Dockerfile wieder: **`exec`-Form vs. Shell-Form** von `CMD`/`ENTRYPOINT`.
<!-- basicblock-end -->

# Docker-Architektur: Engine, Daemon, Client

<!-- basicblock-start oid="ObsdvaSb3oG3mEdxaH7dQ3tl" -->
Was tut der `docker`-Befehl — und was tut er **nicht**?::
Er **startet keine Container**. Er ist ein **HTTP-Client**: Er parst deine Eingabe, formt daraus eine HTTP-Anfrage an einen dauerhaft laufenden Serverprozess (`dockerd`) und stellt dessen Antwort dar. Danach beendet er sich.<br>Das erklärt: Container überleben das Schließen des Terminals (sie gehören dem Daemon, nie deiner Shell) · `docker ps` zeigt auch fremd gestartete Container · `docker version` zeigt **zwei** Versionsblöcke (Client und Server) · man kann einen Daemon auf einem *anderen* Rechner steuern.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs94Xqsbes7RamlHPORHknk" -->
Grenze **Docker Client**, **Docker Daemon** und **Docker Engine** gegeneinander ab.::
**Client:** das CLI-Programm, das du tippst (`/usr/bin/docker`) — kurzlebig, zustandslos.<br>**Daemon:** der langlebige Serverprozess `dockerd`, der den gesamten Zustand hält und verwaltet.<br>**Engine:** **Sammelbegriff für das Produkt** = Daemon + REST API + CLI. **Kein eigener Prozess und keine dritte Komponente!**<br>Typischer Prüfungsfehler: „Engine, Daemon und Client kommunizieren miteinander" — die Engine *ist* das Bündel aus den anderen beiden.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsckx3pRCugnarxghhdhBKy" -->
Inwiefern ist die Docker-API eine **REST API** im Sinne von Kapitel 1.5? Nenne vier Merkmale.::
Wörtlich, nicht als Analogie: `curl --unix-socket /var/run/docker.sock http://localhost/v1.44/containers/json`<br>1. **Collection-Ressource** `/containers/json` mit `GET` — Substantiv im Plural, genau die gelernte Namenskonvention<br>2. **Query-Parameter** zum Filtern (`?all=true`) statt eigenem Endpunkt<br>3. **Versionierung im Pfad** (`/v1.44/`) für Rückwärtskompatibilität<br>4. **OpenAPI-Spezifikation** vorhanden — wie FastAPI sie erzeugt<br>Folge: **Jedes Programm kann Docker steuern** (Compose, Portainer, Testcontainers, CI). Docker brauchte nie eine Plugin-Schnittstelle — die API *ist* die Schnittstelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3Fy85cStb4RyfJC3UJM5r" -->
Worüber kommunizieren Client und Daemon standardmäßig — und was folgt daraus für Berechtigungen?::
Über einen **Unix-Domain-Socket**: `/var/run/docker.sock` (Symlink auf `/run/docker.sock`), nicht über TCP.<br>Ein Unix-Socket ist eine **Datei mit Dateirechten** (`srw-rw---- root:docker`). Deshalb braucht `docker` „sudo": nicht wegen eines Sonderrechts auf Container, sondern wegen **Schreibrecht auf diese eine Datei** — und deshalb hilft `usermod -aG docker $USER`.<br>TCP (`tcp://…:2376`) geht auch, aber **nur mit TLS-Client-Zertifikaten**: ein offener Docker-Port = offener Root-Zugang, und es gibt Bots, die danach scannen.<br>Ziel umstellen: `DOCKER_HOST=…` oder sauberer `docker context use <name>`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsthacsWpY0S1LuLvugMbzM" -->
Nenne die **vier Ebenen unter dem Client** und je eine Aufgabe.::
1. **`dockerd`** — Benutzerfreundlichkeits-/Zustandsschicht: REST API, Netzwerke, benannte Volumes, Registry-Login, Build-Orchestrierung. Entscheidet, *was* passieren soll.<br>2. **`containerd`** (CNCF) — Ausführungsschicht: Image-Pull, Snapshots, Container-Lebenszyklus. Eigene gRPC-API.<br>3. **`containerd-shim-runc-v2`** — einer **pro Container**: Elternprozess, hält stdin/stdout/TTY, meldet den Exit-Code, **entkoppelt den Container vom Daemon**.<br>4. **`runc`** — OCI-Runtime: legt Namespaces + cgroups an, `pivot_root`, `execve()` — und **beendet sich**.<br>Darunter: der Linux-Kernel.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8HdgipNnZgdhuhZ9dEFPy" -->
Welches Problem löst der **containerd-shim** — und warum beendet sich **runc** nach dem Start?::
**Shim:** Er ist Elternprozess des Containers statt containerd/dockerd. Dadurch kann man **`dockerd` neu starten oder aktualisieren, ohne dass alle Container sterben** — ohne Shim wären sie Kinder des Daemons und gingen mit ihm unter. (Ob sie einen Daemon-*Shutdown* überleben, steuert `live-restore` in `daemon.json`.)<br>**runc:** Nach dem `execve()` **ist** der Prozess die Anwendung. Es gibt keinen „Container-Prozess", der ihn einsperrt — die Einschränkungen sind **Eigenschaften, die der Kernel dem Prozess angeheftet hat**. Also gibt es für runc nichts mehr zu tun.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsst9Gzrq9UrPHdVWF363z1" -->
Was tut `runc` konkret, Schritt für Schritt?::
1. `clone()`/`unshare()` mit den **Namespace-Flags** → neue Weltsicht<br>2. **cgroup** anlegen und Limits hineinschreiben → Ressourcenbudget<br>3. Root-Dateisystem mounten und per **`pivot_root`** hineinwechseln<br>4. **Capabilities** fallen lassen, **seccomp**-Profil setzen<br>5. **`execve()`** auf den Entrypoint — der Prozess *wird* die Anwendung<br>6. runc **endet**<br>Grundlage ist eine `config.json` nach **OCI-Runtime-Spec**; Laufzeit: wenige Millisekunden.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsy5z6MqHvSTV6rBuvVu6AM" -->
Wozu dient die **OCI** — und welche alternativen Runtimes gibt es wofür?::
Die **Open Container Initiative** (2015) standardisiert drei Dinge: **Image-Format**, **Runtime-Verhalten** und **Distribution** (Registry-Protokoll). Dadurch ist jede Ebene der Zwiebel **austauschbar**, ohne dass sich oben etwas ändert.<br>**runc** – Referenz, Go · **crun** – in C, schneller/kleiner (Podman-Default) · **gVisor (`runsc`)** – User-Space-Kernel fängt Syscalls ab, für fremden Code ohne volle VM · **Kata Containers** – Mikro-VM pro Container, echte Hardware-Isolation · **youki** – Rust.<br>Damit relativiert sich „Container sind unsicherer als VMs": Das gilt für die *Standard*-Runtime; wer mehr Isolation braucht, tauscht die unterste Ebene aus und behält Image-Format, Registry und CLI.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSsQrXVgV9dUwFGIGR5R0W" -->
Warum ist Mitgliedschaft in der Gruppe **`docker`** gleichbedeutend mit **root**?::
Weil `dockerd` als root läuft (Namespaces, Netzwerkbrücken, `iptables` erfordern Privilegien) und du ihn über den Socket beliebig beauftragen kannst. Der Beweis passt in eine Zeile:<br>`docker run -v /:/host -it alpine chroot /host sh`<br>→ Das gesamte Host-Wurzeldateisystem in einem Container, mit root-Shell darauf. **Kein Exploit — so ist es gebaut.**<br>Die Gruppe zu vergeben ist dasselbe wie passwortloses `sudo` zu vergeben, nur ohne dass es so aussieht. Dasselbe gilt für das Einhängen von `/var/run/docker.sock` in einen Container (*Docker-out-of-Docker*).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsheC74RKVAllRLMUgQfarI" -->
Was ist **Rootless Mode** und was kostet er?::
Seit Docker 20.10: `dockerd` läuft als **normaler Benutzer** innerhalb eines User-Namespace; root *im* Container wird auf deine unprivilegierte UID abgebildet. Einrichtung: `dockerd-rootless-setuptool.sh install`; der Socket liegt dann unter `$XDG_RUNTIME_DIR/docker.sock`.<br>**Preis:** Ports < 1024 nur mit Zusatzkonfiguration · Netzwerk über einen User-Space-Stack (RootlessKit/slirp4netns) und damit langsamer · einige Storage-Treiber und cgroup-Funktionen brauchen aktuelle Kernel.<br>Merke: „Docker braucht root, Podman nicht" stimmt so **nicht mehr**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsukrt18fgNuLfzebgayRXc" -->
Was ist der architektonische Kernunterschied zwischen **Docker** und **Podman**?::
**Podman ist daemonlos.** Das CLI startet `conmon` (Shim-Äquivalent) und `crun` direkt als **Kindprozesse deines Benutzers** — kein zentraler root-Server, Container sind normale Prozesse deiner Sitzung (systemd kann sie sauber verwalten).<br>**Der Preis:** kein zentraler Daemon heißt auch **kein zentraler Zustandshalter** — das wird mit Quadlet/`podman generate systemd` nachgebaut. Anderes Modell, kein reiner Gewinn.<br>Weil beide **OCI-konform** sind, sind die *Images* identisch: Dasselbe Dockerfile baut und läuft unverändert; `alias docker=podman` trägt im Alltag.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDjw4KCJVnSh92j6efqzd1" -->
Beschreibe grob den Ablauf von `docker run -d -p 8000:8000 image` (7 Stationen).::
1. **CLI** parst und baut JSON → `POST /containers/create` über den Socket<br>2. **dockerd**: Image lokal vorhanden? sonst Pull<br>3. **containerd**: Manifest + fehlende Layer von der **Registry**, entpacken, **Snapshot** anlegen (ro-Layer + rw-Schicht)<br>4. **dockerd**: **Netzwerk** einrichten — `veth`-Paar, Bridge `docker0`, IP, `iptables`-Regel für `-p`; Volumes einhängen<br>5. **dockerd → containerd** (gRPC): erzeugen und starten<br>6. **containerd** schreibt die OCI-`config.json`, startet den **Shim**, der **runc** aufruft → Namespaces, cgroups, `pivot_root`, `execve()`<br>7. **CLI** druckt die ID und **beendet sich** — der Container läuft weiter
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshamIFl6pFCMUQxqdVSbKR" -->
Warum läuft `docker build` **nicht** über dockerd → containerd → runc, und was kann **BuildKit** dafür?::
Weil **Bauen** ein anderes Problem ist als **Ausführen**. Seit Engine 23 geht der Build an **BuildKit** (eigener Daemon `buildkitd`, angesprochen über `docker buildx`).<br>BuildKit analysiert das Dockerfile als **Graph** und baut unabhängige Zweige **parallel** (großer Gewinn bei Multi-Stage) · überspringt für das Ziel unnötige Schichten · **Cache-Mounts** (`--mount=type=cache`, z. B. für den uv-Paket-Cache) überleben den Build, ohne im Image zu landen · **Secrets** (`--mount=type=secret`) landen **nicht** in einer Schicht · **Multi-Arch**-Builds (amd64 + arm64).<br>Merksatz: **Ausführen** → dockerd → containerd → runc; **Bauen** → dockerd → BuildKit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsH33XRARAHqwwrHCrDVVa4" -->
Was ist eine **Registry** und was passiert bei einem Pull?::
Ein eigener HTTP-Dienst, der Images speichert und ausliefert (Docker Hub, `ghcr.io`, ECR, GitLab, selbst betrieben) — **nicht Teil der Engine**. Protokoll: **OCI-Distribution-Spec**.<br>Ein Pull holt zuerst das **Manifest** (JSON: welche Layer, welche Architektur, welche Konfiguration), dann nur die **fehlenden Layer** — vorhandene werden übersprungen (darum lädt das zweite Image einer Familie viel schneller).<br>`postgres:18` bedeutet implizit `docker.io/library/postgres:18`.<br>Praxisfalle: Docker Hub hat **Rate Limits** für anonyme Pulls — der klassische „warum ist der CI-Build plötzlich rot"-Moment.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6GwJpy1HDt3TkqVXbsDnH" -->
Was ist **Docker Compose** architektonisch — und woher kommt der **Projektname**?::
Compose ist **kein zusätzlicher Dienst**, sondern ein weiteres **CLI-Plugin**, das dieselbe REST-API aufruft: Es liest die Compose-Datei und schickt dieselben `POST /containers/create`-Requests, nur mehrere in der richtigen Reihenfolge.<br>**Projektname:** Compose stellt jedem Namen ein Präfix voran, das standardmäßig aus dem **Verzeichnisnamen** stammt — das Volume `pricing_data` heißt real `db_migration_tools_pricing_data`. Der Daemon kennt **keine Projekte**, nur global eindeutige Namen; der Projektname ist reine **client-seitige Namensraum-Konvention**.<br>Festnageln mit `name: pricing` auf oberster Ebene der Compose-Datei.<br>Nebenbei: `docker-compose` (V1, Python) ist seit 2023 abgekündigt, `docker compose` (V2, Go-Plugin) ist Standard; bevorzugter Dateiname ist heute `compose.yaml`.
<!-- basicblock-end -->

# Docker-Objekte: Images, Container, Volumes, Networks

<!-- basicblock-start oid="Obs6rXsyniXHRpYzuOpJZdLZ" -->
Welches **einheitliche CLI-Muster** haben alle vier Docker-Objekte — und wie teilen sie sich auf?::
Jedes Objekt hat eine **ID** (SHA-256, meist auf 12 Zeichen gekürzt), optional einen **Namen**, einen **Lebenszyklus** und dieselben vier Verben:<br>`docker <objekt> ls | inspect | rm | prune` — für `image`, `container`, `volume`, `network`.<br>**Aufteilung:** *Vorlage* = **Image** ↔ *Instanz* = **Container**; **Volume** und **Network** gehören zur *Umgebung* und existieren **unabhängig** von beiden — sie überleben Container und werden zwischen ihnen geteilt. Genau diese Unabhängigkeit erzeugt die meisten Missverständnisse.<br>(`docker ps`/`images`/`rmi` sind alte Kurzformen; `ps` heißt so, weil Container Prozesse *sind*.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaoSeae0xr4x8hZp8JaW0W" -->
Was ist ein **Image** und woraus besteht es physisch?::
Eine **unveränderliche, schreibgeschützte, aus Schichten aufgebaute Vorlage** für ein Container-Dateisystem plus Metadaten, wie daraus ein Container zu starten ist.<br>Physisch drei Teile:<br>**Manifest** — JSON-Liste der Layer und der Config, per Digest referenziert<br>**Config** — JSON: `Cmd`, `Entrypoint`, `Env`, `WorkingDir`, `User`, `ExposedPorts`, Architektur<br>**Layers** — je ein Tar-Archiv mit den **Änderungen** dieser Schicht<br>Schlüsselsatz: Ein Layer enthält **nicht** das ganze Dateisystem, sondern nur die **Differenz** zur darunterliegenden — ein *Diff*, wie ein git-Commit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsamNyIGLRMKZ0b6URx82JW" -->
Was bedeutet **Content-Addressing** bei Images und welche drei Folgen hat es?::
Jede Schicht und jede Config wird über den **SHA-256-Hash ihres Inhalts** adressiert — dasselbe Prinzip wie bei git-Commits.<br>1. **Identischer Inhalt = identischer Hash = nur einmal gespeichert.** Zwei Images auf derselben Basis teilen sie sich auf der Platte.<br>2. **Manipulationssicherheit** — ein Layer kann sich nicht unbemerkt ändern, denn die Adresse *ist* der Inhalt.<br>3. **Effizienter Transfer** — beim Pull kommen nur Layer, deren Digest lokal fehlt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnFPZwVD0f9LR8K170diiZ" -->
Zerlege den vollständigen Image-Namen `docker.io/library/postgres:18@sha256:…` — und was ist der Unterschied zwischen **Tag** und **Digest**?::
`[Registry]/[Namespace]/[Repository]:[Tag]@[Digest]` — `docker.io` und `library` sind Defaults, deshalb ist `postgres:18` die Kurzform.<br>**Tag** = *bewegliches Label*, jederzeit auf ein anderes Image umhängbar (`postgres:18` zeigt heute auf 18.1, bald auf 18.2). **Digest** = *unveränderlich*, bezeichnet für immer exakt dieses Image.<br>Daher: Ein Dockerfile mit Tag allein erzeugt **keine** Reproduzierbarkeit.<br>**`latest`** ist kein „neuestes", sondern nur das Tag, das Docker ohne Angabe annimmt — ohne jede eingebaute Bedeutung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7uvHFUN6ApEim9p4QJFNi" -->
Du kopierst in Schicht 3 ein Secret ins Image und löschst es in Schicht 4 per `RUN rm`. Ist es weg?::
**Nein.** Layer sind rein **additive Diffs** — eine höhere Schicht kann nichts wirklich entfernen. Sie legt nur eine **Whiteout-Markierung** an, die die Datei im zusammengesetzten Dateibaum *unsichtbar* macht. **Die Datei bleibt in der unteren Schicht** und damit im Image; mit `docker save` + Tar-Entpacker in einer Minute wiederherstellbar.<br>Dasselbe gilt für Plattenplatz: `apt-get install` in einer Schicht und `rm -rf /var/lib/apt/lists/*` in der nächsten macht das Image **nicht** kleiner.<br>Lösungen: alles in **einem** `RUN`, **Multi-Stage-Builds**, **BuildKit-Secrets**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszKaTMeiqO6zVTEP3MVpnq" -->
Wozu dienen `docker image history` und woher kommen **`<none>:<none>`**-Images?::
**`docker image history <image>`** zeigt jede Schicht mit Größe und erzeugendem Befehl — der erste Befehl beim Optimieren eines Images und der Weg, jedes fremde Image zu verstehen.<br>**`<none>:<none>` (dangling):** Beim Neubau hängt Docker den Tag auf das **neue** Image um; das alte verliert seinen Namen, existiert aber weiter. Nach einer Woche Entwicklung sind das schnell mehrere Gigabyte. Aufräumen mit `docker image prune`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3ViiyAoRnmd5EmJv4wnfv" -->
Was ist ein **Container** — und warum ist „Container = laufender Container" falsch?::
Eine **Instanz eines Images**: dessen ro-Schichten + eine **dünne beschreibbare Schicht** + eine **Laufzeitkonfiguration** (Name, Env, Ports, Mounts, Netzwerke, Limits).<br>Ein Container **existiert auch im gestoppten Zustand** — mit Schicht, Konfiguration und Namen; er belegt Plattenplatz und blockiert seinen Namen.<br>Daher: `docker ps` zeigt nur laufende, **`docker ps -a` alle** — und daher die Fehlermeldung „name is already in use", obwohl `docker ps` leer aussieht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1SoXYC4HtoIFGwGPCNJsA" -->
Was ist `docker run` in Wahrheit — und warum kann man einen Container nicht nachträglich umkonfigurieren?::
`docker run` = **`docker create` + `docker start`** (plus `pull`, falls das Image fehlt). Kein eigenes Konzept.<br>Optionen wie `-e`, `-v`, `-p`, `--name` gehören zur **Erzeugung**, nicht zum Start — deshalb gibt es sie bei `docker start` nicht. Wer etwas ändern will, **löscht den Container und erzeugt einen neuen**.<br>Das ist Absicht und heißt **Immutable Infrastructure**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxY5fEkUmGjyNRsHu7wQW3" -->
Unterschied `docker stop` vs. `docker kill` — und was hat das mit PID 1 zu tun?::
**`stop`** schickt `SIGTERM`, wartet standardmäßig **10 Sekunden**, dann `SIGKILL`. **`kill`** schickt sofort `SIGKILL`.<br>Verbindung zur **PID-1-Falle**: Bekommt dein Prozess das Signal nicht (weil eine Shell dazwischenhängt) oder reagiert nicht darauf, dauert **jedes** Herunterfahren 10 Sekunden und endet hart. Für PostgreSQL heißt hart abgeschossen: **Recovery beim nächsten Start**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6emZtKYBuduM8WZIRG3UY" -->
Warum kehrt `docker run debian:13` sofort und ohne Fehler zurück?::
Weil **ein Container genau so lange lebt wie sein PID-1-Prozess**. Endet der Hauptprozess, endet der Container — mit dessen Exit-Code. Es gibt keinen Container, der „läuft" und dabei nichts tut.<br>Hier: Der Startbefehl ist `bash`; ohne angeschlossenes Terminal liest bash von leerem stdin, bekommt EOF und beendet sich **sauber** (`Exited (0)`). Kein Defekt.<br>**Debugging-Regel:** „Mein Container startet nicht" heißt fast immer „**mein Prozess ist beendet**" → `docker ps -a` (Exit-Code! 137 = SIGKILL, oft OOM) → `docker logs` → `docker inspect`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFbKu59HdjiyaHyMMUdyMu" -->
Wofür brauchst du `docker exec` und `docker inspect`?::
**`docker exec -it <name> <befehl>`** startet einen **zweiten Prozess im laufenden Container** — dein Fenster hinein, um zu prüfen, was der Container *wirklich* sieht: `docker exec api env`, `docker exec db psql -U pricing`. So erreicht man die DB, ohne einen Port nach außen zu öffnen.<br>**`docker inspect`** gibt den vollständigen Zustand jedes Objekts als JSON aus — die Stelle zum *Nachsehen statt Raten*. Mit Go-Templates gezielt:<br>`docker inspect --format '{{.State.ExitCode}}' api`<br>Tipp: `docker run --rm …` für Wegwerf-Container, sonst füllt sich `docker ps -a` mit Leichen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7GPBQR0t0EzdeqDL44rBk" -->
Erkläre **Image vs. Container** mit der OOP-Analogie — und wo versagt sie?::
**Image = Klasse** (Bauplan, liegt einmal vor) · **Container = Objekt** (Instanz mit eigenem Zustand) · **`docker run -e … -p …` = Konstruktorargumente** · viele Container aus einem Image.<br>Ebenso tragfähig: **Programm auf der Festplatte** vs. **Prozess im Speicher**.<br>**Grenzen der Analogie:** (1) Ein Image ist zusätzlich **aus Schichten aufgebaut** und teilt seine Basis mit anderen Images — eher Vererbung als Instanziierung. (2) Ein Container **existiert auch, wenn er nicht läuft** — ein „Objekt im Ruhezustand" kennt OOP nicht. (3) Der Container-Zustand ist **flüchtig** und verschwindet mit `docker rm`.<br>Praktisch: drei `postgres:18`-Container = drei unabhängige Datenbanken, **ein** Image auf der Platte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTSCmgieiK5scIiSJLNhpY" -->
Nenne die **drei Mount-Arten** mit Einsatzzweck und Nachteil.::
**Named Volume** (`pricing_data:/var/lib/postgresql`) — von Docker verwaltet unter `/var/lib/docker/volumes/`, ein **echtes Docker-Objekt**, portabel. Für **Datenbanken/Produktivdaten**.<br>**Bind Mount** (`./src:/app/src`) — konkreter Host-Pfad, **kein** Docker-Objekt. Für **Quellcode in der Entwicklung** (Live-Reload ohne Neubau). Nachteil: hängt am Host-Pfad; Host-UID/GID gelten im Container → „Permission denied" und Dateien, die dem Container-root gehören.<br>**tmpfs** — nur im RAM. Für Secrets und Caches.<br>**Merkregel:** Named Volume, wenn die Daten *wichtig* sind. Bind Mount, wenn du sie *selbst bearbeiten* willst.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSwptnQgR1m8Z6T2lh6Vk3" -->
Was passiert mit einem **Volume**, wenn der Container gelöscht wird — und was sind die zwei typischen Fallen?::
**Nichts.** `docker rm <container>` löscht das Volume **nicht** (Daten sollen nicht versehentlich verschwinden). Bewusst löschen: `docker rm -v`, `docker compose down -v`, `docker volume prune`.<br>**Falle 1 – anonyme Volumes:** Deklariert ein Dockerfile `VOLUME /pfad` und du mountest nichts, legt Docker automatisch ein Volume mit **zufälligem Hash-Namen** an. Persistent, aber niemand weiß wozu.<br>**Falle 2 – wer gewinnt beim Mount:** Beim ersten Mount eines **leeren** Named Volume kopiert Docker den Image-Inhalt des Zielverzeichnisses hinein (Bind Mounts **nicht** — die überdecken kommentarlos). Ist das Volume schon gefüllt, gewinnt **immer das Volume** → neues Compose-Projekt = neues leeres Volume = leere Datenbank.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKsn0Rvj3qbp6s3qT2f8jy" -->
Warum lautet der Volume-Pfad bei `postgres:18` `/var/lib/postgresql` und nicht mehr `/var/lib/postgresql/data`?::
Mit **PostgreSQL 18** hat das offizielle Image sein `VOLUME` auf **`/var/lib/postgresql`** geändert und `PGDATA` **versionsspezifisch** gemacht (`/var/lib/postgresql/18/docker`). Wer beim Upgrade 17 → 18 den alten Pfad stehen lässt, bekommt einen Container, der nicht startet — oder schlimmer: eine **leere Datenbank**, während die alten Daten unberührt im Volume liegen.<br>**Allgemeine Lehre:** Wohin ein Image seine Daten schreibt, ist **Teil seines Vertrags** und kann sich zwischen Major-Versionen ändern. Nachsehen:<br>`docker image inspect postgres:18 --format '{{json .Config.Volumes}}'`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCxcHV07NkU9Haiv7CJdTN" -->
Nenne die fünf **Netzwerktreiber** und wofür sie da sind.::
**bridge** — eigenes virtuelles Netz mit NAT nach außen; **Standard** für Einzelhost<br>**host** — kein eigener `net`-Namespace, teilt den Host-Stack; maximale Performance, **kein** Port-Mapping möglich<br>**none** — nur Loopback; bewusste Totalisolation<br>**overlay** — verbindet Container über **mehrere Hosts**; Swarm/Cluster<br>**macvlan** — eigene MAC/IP im physischen LAN; Legacy-Integration<br>Technisch legt der Daemon je Container ein **`veth`-Paar** an (virtuelles Kabel), hängt das Host-Ende in eine Bridge und schreibt `iptables`-Regeln fürs Port-Publishing.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsP1Uf2lssTSgNMBoVuN4Oz" -->
Warum funktioniert `db` als Hostname in Compose — und in welchem Netzwerk **nicht**?::
**Nur in selbst angelegten Netzwerken** betreibt Docker einen eingebauten **DNS-Server**, in dem der **Containername ein auflösbarer Hostname** ist.<br>Im vorgegebenen `bridge`-Netz (dort landen Container ohne `--network`) erreichen sich Container **nur über IP** — und die ändert sich bei jedem Neustart.<br>**Compose legt für jedes Projekt automatisch ein eigenes Netzwerk an** und hängt alle Dienste hinein — deshalb ist der **Dienstname** ein gültiger Hostname, ohne dass man etwas konfiguriert. Nebeneffekt: zwei Compose-Projekte sind netzwerkseitig voneinander **isoliert**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstzeLW2iGGwjqlFTfuGiiM" -->
Was bedeutet `ports: - "127.0.0.1:5432:5432"` genau — und brauchen zwei Container im selben Netz ein Port-Mapping?::
Schema: **`host-ip : host-port : container-port`**.<br>**Nein** — Container-zu-Container-Verkehr im selben Netzwerk braucht **kein** Port-Mapping. `ports:` ist nur für den Zugriff **von außen** (vom Host) da.<br>Das `127.0.0.1:`-Präfix ist wichtig: **ohne** lauscht der Port auf *allen* Interfaces, und Docker trägt seine `iptables`-Regeln **an der Firewall vorbei** ein — so landen Datenbanken offen im Internet.<br>Und: **`localhost` im Container ist der Container** (Folge des `net`-Namespace). Deshalb muss `db_host` von `localhost` auf den Dienstnamen `db` wechseln, sobald die API selbst im Container läuft.<br>Gegenrichtung (Container → Host): unter Docker Desktop `host.docker.internal`, unter Linux-Engine nur mit `extra_hosts: ["host.docker.internal:host-gateway"]`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsoq144DQ1YVKNHLDwjvgFi" -->
Wie findest du heraus, wo Dockers Plattenplatz hingeht — und welche prune-Befehle sind gefährlich?::
**`docker system df`** (bzw. `-v` für Details pro Objekt) — Übersicht über Images, Container, Volumes und **Build-Cache** (oft der größte Posten).<br>Aufräumen: `docker container prune` (gestoppte) · `docker image prune` (dangling) · `docker image prune -a` (alle ohne laufenden Container) · `docker builder prune` (BuildKit-Cache) · `docker system prune` (die ersten drei).<br>**Gefährlich:** `docker volume prune` und `docker system prune --volumes` löschen **Volumes ohne zugeordneten Container** — also womöglich deine Datenbank, wenn der Container gerade entfernt ist. Keine Nachfrage pro Objekt, kein Rückgängig.<br>Angewöhnen: erst `docker system df -v` lesen, dann gezielt löschen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTo3jS2VO5T7Mg1ltjSKCa" -->
Was bedeutet `USER 1000:1000` — und muss dieser Benutzer einen **Namen** haben?::
Format **`UID:GID`**. **Nein, ein Name ist nicht nötig:** Der Kernel speichert bei jeder Datei nur zwei **Integer** und vergleicht sie mit denen des Prozesses — Namen sind bloß Zeilen in `/etc/passwd` und `/etc/group`, ein **Nachschlagewerk für Menschen**.<br>**Mechanik:** `USER` landet als String in der **Image-Config**; beim Start ruft `runc` vor dem `execve()` einfach `setgid(1000); setuid(1000)` auf — ein Zahlenfeld im Prozess-Eintrag. **Nichts wird nachgeschlagen, nichts automatisch gewählt** (`docker run --rm -u 4711:4711 python:3.13-slim id` funktioniert, obwohl 4711 nirgends existiert). Nur bei **Namen** liest Docker `/etc/passwd` im Image — und scheitert ohne Eintrag.<br>Ohne Eintrag brechen nur Komfortfunktionen: `whoami`/`id -un` scheitern, `$HOME` ist leer, `ls -l` zeigt `1000`, Programme mit `getpwuid()` können abbrechen. Sauberer daher `groupadd --gid 1000 app && useradd --uid 1000 --gid 1000 --no-create-home app`.<br>**Warum 1000?** Distributions-Konvention: `0` = root, `1–999` = Systemkonten, **ab 1000** = menschliche Benutzer. Der erste Benutzer eines Debian-Systems hat UID 1000 — dieselbe Zahl im Image lässt bei Bind Mounts die Rechte zusammenpassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRiRwKtORXi7vDeL4KH2fV" -->
Warum gehört `--chown` ans `COPY` und **nicht** in ein eigenes `RUN chown -R`?::
```dockerfile
COPY --from=builder --chown=1000:1000 /app/.venv /app/.venv   # ✅ eine Schicht
COPY --from=builder /app/.venv /app/.venv                     # ❌ und dann
RUN chown -R 1000:1000 /app/.venv                             #    ~150 MB doppelt
```
`chown` ändert die **Metadaten jeder einzelnen Datei** — und eine geänderte Datei landet **vollständig** in der neuen Schicht. Ein rekursives `chown` über ein großes Verzeichnis **verdoppelt dessen Platzbedarf im Image**, obwohl sich kein Byte Inhalt ändert.<br>Grund: das additive Layer-Modell. Namen statt Zahlen (`--chown=app:app`) gehen auch — aber nur, wenn der Benutzer an **dieser Stelle** im Dockerfile schon existiert.<br>Und: **`USER` gilt für alles danach** (jedes `RUN`, `HEALTHCHECK`, `CMD`, `ENTRYPOINT`) — deshalb ans **Ende** der Runtime-Stage.
<!-- basicblock-end -->

# Docker CLI: Grundlegende Befehle und Anwendung

<!-- basicblock-start oid="ObsaLFS44YxNe7f0j97BbBQK" -->
Wie ist ein Docker-Befehl aufgebaut — und was ist der Fehler, den jeder genau einmal macht?::
`docker [objekt] <verb> [optionen] <ziel> [befehl + argumente]`<br>**Alles nach dem Image-Namen ist nicht mehr für Docker bestimmt**, sondern wird als Befehl *in* den Container gereicht:<br>`docker run pricing-api -p 8000:8000` ❌ — übergibt `-p 8000:8000` an **dein Programm**, ohne Docker-Fehlermeldung!<br>`docker run -p 8000:8000 pricing-api` ✅<br>**Merksatz: Flags gehören vor das Image.**<br>Der Befehl am Ende **ersetzt das `CMD`** des Images — das ist ein Werkzeug, kein Sonderfall: `docker run --rm -it python:3.13-slim bash`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsqu0vj3GOa13Lqw5QvO3ey" -->
Was bewirken `-i` und `-t` einzeln — und welche drei Kombinationen sind sinnvoll?::
**`-i` (`--interactive`)** hält **stdin offen**. Ohne das bekommt der Prozess sofort EOF (genau der Grund, warum `docker run debian` sich sofort beendet).<br>**`-t` (`--tty`)** weist ein **Pseudo-Terminal** zu: Zeilenumbrüche, Farben, Prompt, Signalweitergabe von `Strg+C`.<br>`docker run -it … bash` — Shell zum Arbeiten (beides)<br>`docker run -i … < datei.sql` — Eingabe aus Pipe: **`-i` ja, `-t` nein!**<br>`docker run -d …` — Dienst im Hintergrund (weder noch)<br>`-dit` braucht man nur für „Container künstlich am Leben halten" — meist ein Zeichen, dass etwas anderes falsch konstruiert ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbBLJAaAndekU63d5HnhEH" -->
Wie erkundest du ein **fremdes Image**, bevor du es benutzt?::
Ein Image ist eine **Blackbox**, bis man `inspect` und `history` darauf angewendet hat:<br>`docker image inspect <img> --format '{{json .Config}}' \| jq` → **`Env`** (erwartete Variablen), **`Volumes`** (Datenpfade), **`Cmd`/`Entrypoint`** (Startbefehl), `ExposedPorts`<br>`docker image history <img>` → jede Schicht mit Größe und erzeugendem Befehl<br>`docker run --rm -it <img> bash` → drin umsehen, ohne den eigentlichen Dienst zu starten
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvqbPO4s3YqSXbPki9XSO8" -->
Nenne die wichtigsten **Beobachtungs-Befehle** — und was zeigen `docker stats` und `docker diff`?::
`docker ps -a` (Status + Exit-Code) · `docker logs -f --tail 50 <c>` / `--since 10m` · `docker top` (Prozessliste im Container) · `docker events` (Live-Strom aller Daemon-Ereignisse) · `docker inspect`<br>**`docker stats`** — live CPU/RAM/Netz/IO; die Spalte **`MEM USAGE / LIMIT`** zeigt Verbrauch **und cgroup-Limit** → so findet man Container kurz vor dem OOM-Kill.<br>**`docker diff <c>`** — jede Datei, die sich seit dem Start in der **beschreibbaren Schicht** geändert hat (`A`dded/`C`hanged/`D`eleted). Praktische Demonstration des Layer-Modells und der Weg herauszufinden, ob eine Anwendung **heimlich ins Container-FS schreibt statt in ein Volume**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspOtJ7h2rnu2sRmUTx55gE" -->
Warum zeigt `docker logs` nichts, obwohl die Anwendung nachweislich loggt?::
**`docker logs` liest nur stdout/stderr des PID-1-Prozesses.** Wer in eine **Datei** im Container loggt, sieht hier nichts.<br>Daher die Regel: **Container loggen nach stdout.** Für unser Projekt heißt das: `logging_setup.py` braucht im Container einen **StreamHandler**, keinen FileHandler.<br>(Zweiter möglicher Grund: Es loggt ein *anderer* Prozess als PID 1.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsySYHIUlcmdy8E8cRgAQhH" -->
Wie debuggst du ein Image, dessen Container sofort stirbt?::
1. **`docker logs <c>`** — was sagt er, bevor er stirbt?<br>2. **Entrypoint umgehen:** `docker run --rm -it --entrypoint sh <image>` → drin: `ls -la /app`, `env`, `python -c "import pricing"`<br>3. Mit identischer Konfiguration: `docker compose run --rm --entrypoint sh api`<br>**Warum Schritt 2 so wertvoll ist:** Er startet **das Image mit allen Schichten**, aber **nicht deine Anwendung** — damit trennst du „Image falsch gebaut" von „Anwendung startet nicht".<br>Falls `bash` fehlt: `sh` probieren (Alpine); bei distroless gibt es **gar keine Shell** — das ist Absicht (kleinere Angriffsfläche).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBloIqXZ1hkz2fUDzCDcN5" -->
Was bedeuten die Exit-Codes **0**, **1**, **137** und **143**?::
**0** — Prozess **sauber** beendet. Bei einem *Dienst* trotzdem verdächtig: Dann fehlt der Vordergrundprozess.<br>**1** — Anwendungsfehler → `docker logs`<br>**137** — 128 + 9 = **SIGKILL**: meist **OOM-Killer** oder `docker kill` → `docker stats`, Speicherlimit prüfen<br>**143** — 128 + 15 = **SIGTERM**: normales `docker stop`, in Ordnung<br>**Regel:** Alles über 128 heißt „durch Signal beendet", Signalnummer = Code − 128. `docker ps -a` zeigt den Code immer an — die schnellste Diagnose überhaupt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAjbdH8alP9Nt900TA4CWK" -->
Übersetze: `port is already allocated` · `name is already in use` · `connection refused an db:5432` · `exec format error`::
**`port is already allocated`** — der **Host**-Port ist belegt; oft ein lokal installiertes PostgreSQL → `ss -tlnp \| grep 5432`<br>**`name is already in use`** — ein **gestoppter** Container hält den Namen → `docker rm <name>`<br>**`connection refused` an `db:5432`** — Netzwerk und DNS sind **richtig**, der Dienst ist nur **noch nicht bereit** → Healthcheck + `depends_on: condition: service_healthy`<br>**`exec format error`** — **falsche Architektur** (arm64-Image auf amd64) → `docker image inspect … Architecture`<br>(Zum Vergleich: `no such host` wäre das *echte* DNS-Problem — falsches Netzwerk oder Default-Bridge.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2Z8gAijooGKf7ScmBVtDB" -->
Unterschied `docker compose exec` vs. `docker compose run` — und welches nimmst du für `alembic upgrade head`?::
**`exec`** braucht einen **laufenden** Container und startet einen zweiten Prozess **darin**.<br>**`run`** erzeugt einen **neuen** Container aus dem Dienst — mit dessen Image, Netzwerk und Umgebung, aber anderem Befehl (deshalb **`--rm` nicht vergessen**).<br>Für Migrationen ist **`run`** richtig: `docker compose run --rm api alembic upgrade head` — die Migration soll unabhängig davon laufen, ob die API gerade läuft, und in einem Container, der danach verschwindet.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsi9gNNr4TXozxHByecwoKL" -->
Nenne die vier Compose-Befehle des Alltags — und wozu dient `docker compose config`?::
`docker compose up -d --build` — bauen (falls nötig) und im Hintergrund starten<br>`docker compose logs -f api` — einem **Dienst** zusehen (Dienstname, nicht Containername!)<br>`docker compose down` — alles weg, **Volumes bleiben**<br>`docker compose down -v` — … auch die Volumes. **Vorsicht.**<br>**`docker compose config`** rechnet Variablen-Substitution, `.env`, `extends` und Overrides aus und zeigt die **tatsächlich wirksame** Konfiguration. Bei „warum nimmt er meine Umgebungsvariable nicht?" ist das der **erste** Befehl, nicht der zehnte.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7su4fBJRXGoyM1KpQdokj" -->
Was bewirken `-q`, `--filter` und `--format` — und warum gibt es sie überhaupt?::
`-q` (quiet) gibt **nur IDs** aus — für Pipes gemacht: `docker rm $(docker ps -aq --filter status=exited)`<br>`--filter` schränkt ein: `--filter name=pricing`, `--filter dangling=true`, `--filter status=exited`<br>`--format` formatiert mit Go-Templates: `--format 'table {{.Names}}\t{{.Status}}'`<br>Fast **jeder** Befehl versteht alle drei. Der Grund dafür ist architektonisch: Darunter steckt nur eine **REST-API mit JSON-Antworten** — die CLI formatiert lediglich, was der Daemon liefert.
<!-- basicblock-end -->

# Dockerfile: Struktur und Best Practices

<!-- basicblock-start oid="Obs1OajbluS7ghtS8kR6m7bZ" -->
Was ist der **Build-Kontext** bei `docker build -t name:tag .` — und warum scheitert `COPY ../datei .`?::
Der Punkt am Ende ist **nicht „hier"**, sondern der **Build-Kontext**: das Verzeichnis, dessen Inhalt (abzüglich `.dockerignore`) **an BuildKit übertragen** wird, bevor der Build beginnt.<br>Der Build läuft **nicht in deinem Verzeichnis**, sondern in einem separaten Prozess, der nur sieht, was du ihm geschickt hast. `COPY ../datei .` ist deshalb **kein Rechteproblem**, sondern schlicht außerhalb des Übertragenen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXhI7Tbx0Ul3owGZ778oWN" -->
Beschreibe den **Image-Build** von `docker build` bis zum fertigen Image (Kontrollfrage).::
1. CLI packt den **Build-Kontext** (minus `.dockerignore`) und schickt ihn an **BuildKit**<br>2. Dockerfile wird geparst und als **Abhängigkeitsgraph** analysiert<br>3. Basis-Image aus `FROM` auflösen/holen<br>4. Anweisungen der Reihe nach ausführen, wo möglich aus dem **Cache**<br>5. Jede `RUN`/`COPY`/`ADD` erzeugt eine **neue Schicht** (Diff zum Vorzustand)<br>6. **Config** schreiben (`Cmd`, `Env`, `User`, `ExposedPorts`)<br>7. Manifest + Config bekommen ihren **Digest** — das Image existiert<br>8. **Tag** darauf setzen<br>**Rolle des Dockerfiles:** Es macht die Umgebung zu **Code** — versioniert, reviewbar, wiederholbar, im selben Repo wie die Anwendung. Derselbe Gedanke wie bei Alembic: Was den Systemzustand bestimmt, gehört in die Versionskontrolle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvAP4wbPcKsAoXWLfpDdWU" -->
Nenne drei Gründe, warum eine **`.dockerignore`** bei einem Python-Projekt nicht optional ist.::
1. **Geschwindigkeit** — der Kontext wird bei *jedem* Build vollständig übertragen (`.venv/`, `.git/` sind hunderte MB).<br>2. **Cache** — ein `COPY . .` wird durch **jede** Änderung im Kontext ungültig, auch durch eine neue Log-Datei. Ohne `.dockerignore` baut Docker ständig neu, ohne dass sich Relevantes geändert hat.<br>3. **Sicherheit** — was nicht im Kontext ist, kann nicht versehentlich ins Image kopiert werden (`.env`!).<br>**Faustregel:** mindestens alles aus `.gitignore` — **plus `.git` selbst**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgT0osBf0vk4AA2bRV3qWH" -->
Vier klassische Dockerfile-Fallstricke: `EXPOSE`, `RUN cd`, `ENV` vs. `ARG`, `VOLUME`.::
**`EXPOSE` öffnet keinen Port** — reine Dokumentation im Image (und Hinweis für `docker run -P`). Veröffentlicht wird mit `-p` bzw. `ports:`; beides ist unabhängig voneinander.<br>**`RUN cd /app` wirkt nicht** — jede `RUN`-Anweisung startet eine eigene Shell. Dafür gibt es **`WORKDIR`**.<br>**`ENV` bleibt im Image** (per `docker inspect` lesbar), **`ARG` nicht** — aber `ARG` steht in der **Build-Historie** (`docker image history`). Also **beides kein Ort für Secrets**.<br>**`VOLUME` in eigenen Images ist meist ein Fehler** — es erzwingt ein **anonymes Volume** für jeden Nutzer des Images und lässt sich nicht abschalten. Wo Daten liegen, entscheidet die Compose-Datei.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnJpwpxuiYdWQCrFRi5Pu8" -->
Wie funktioniert **Layer-Caching**, und was ist der Cache-Schlüssel je Anweisung?::
BuildKit prüft je Anweisung, ob das Ergebnis im Cache liegt. **Die eine Regel: Muss eine Schicht neu gebaut werden, werden alle folgenden ebenfalls neu gebaut.**<br>**`COPY`/`ADD`** → **Inhalt** der Dateien (Prüfsumme)<br>**`RUN`** → **nur der Befehlstext** (!)<br>**`FROM`** → Digest des Basis-Images<br>**Berüchtigte Folge:** `RUN apt-get update` hat beim zweiten Build denselben Text und kommt **aus dem Cache** — obwohl die Paketlisten veraltet sind. Deshalb `update` und `install` **immer in einem `RUN`**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9IZrfI8Znv0j2I1rnoFvG" -->
Warum kopiert man `pyproject.toml`/`uv.lock` **vor** dem Quellcode?::
**Reihenfolge nach Änderungshäufigkeit** — die wichtigste Strukturentscheidung in jedem Dockerfile. Selten Geändertes nach oben, häufig Geändertes nach unten.<br>❌ `COPY . .` + `RUN uv sync` → jede Code-Änderung wirft die Abhängigkeiten weg, Minuten pro Build<br>✅ `COPY pyproject.toml uv.lock ./` + `RUN uv sync --frozen --no-dev` + danach `COPY src/ ./src/` → der teure Schritt hängt nur an der **Lockfile**<br>Unterschied im Alltag: zwei Sekunden statt zwei Minuten, bei **jedem** Build.<br>Ergänzend: **Cache-Mounts** (`RUN --mount=type=cache,target=/root/.cache/uv …`) überleben Builds, landen aber **nicht** in einer Schicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsC1FpMR2okW3EfwjgNMNGz" -->
**exec-Form vs. shell-Form** bei `CMD`/`ENTRYPOINT` — und warum ist die shell-Form fast immer falsch?::
`CMD ["uvicorn", "app"]` (**exec**, JSON-Array) → **dein Programm ist PID 1**.<br>`CMD uvicorn app` (**shell**) → `/bin/sh` ist PID 1, dein Programm ist **PID 2**.<br>**Das Problem:** `docker stop` schickt `SIGTERM` an **PID 1**, also an die Shell — die **reicht es nicht weiter**. Deine Anwendung merkt nichts, fährt nicht herunter, nach 10 Sekunden kommt `SIGKILL` für alle.<br>**Symptome:** jedes `docker compose down` dauert exakt 10 Sekunden · Verbindungen brechen ab · die Datenbank macht Recovery — **ohne eine einzige Fehlermeldung**.<br>Die shell-Form kann dafür Variablen expandieren. Wer das braucht: Entrypoint-Skript, das mit **`exec "$@"`** endet — `exec` ersetzt die Shell, das Programm wird PID 1.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ey7DwnRSHSq8S1mCxIyZ" -->
Rollenverteilung `ENTRYPOINT` vs. `CMD` — und warum bei Anwendungs-Images oft nur `CMD`?::
**`ENTRYPOINT` = *was*** ausgeführt wird (das Programm). **`CMD` = *womit*** (die Standard-Argumente).<br>`docker run img` → ENTRYPOINT + CMD · `docker run img --port 9000` → **CMD wird ersetzt** · `docker run --entrypoint bash img` → ENTRYPOINT wird ersetzt.<br>**Nur `CMD`** (ohne ENTRYPOINT) ist flexibler und für Anwendungs-Images meist richtig: Dann funktioniert `docker compose run --rm api alembic upgrade head` einfach, weil der Befehl **komplett** ersetzt wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8kptf2ndA089ULbWCm2Om" -->
Was ist ein **Multi-Stage Build** und warum sollte man ihn nutzen? (Kontrollfrage)::
Mehrere **`FROM`**-Anweisungen in einer Datei. Jedes `FROM` beginnt eine **Stage** mit frischem Dateisystem; **`COPY --from=<stage>`** holt gezielt einzelne Artefakte. **Nur die letzte Stage wird das Image.**<br>**Grund:** Bauen und Laufen haben **verschiedene Abhängigkeiten** — Compiler, Paketmanager, Header, Caches braucht man nur zum Installieren. Einstufig bleibt alles im Image, und wegen des **Whiteout-Problems** hilft nachträgliches `rm` nicht.<br>**Vorteile:** kleineres Image (oft Faktor 2–5) · kleinere Angriffsfläche · **keine Secret-/Quellcode-Reste** aus der Bauphase · schnellere Deployments · BuildKit baut unabhängige Stages **parallel** · **`--target`** erlaubt mehrere Ziele aus einer Datei (z. B. eine Test-Stage für CI).<br>Nebenbei: `COPY --from=` kann auch **fremde Images** referenzieren, z. B. `ghcr.io/astral-sh/uv`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCRxbHDIdCPiccHuxglXWi" -->
Warum ist **Alpine** bei Python oft die falsche Sparsamkeit — und was ist die Standardwahl?::
Alpine nutzt **musl** statt **glibc**. Die PyPI-Wheels sind traditionell gegen glibc gebaut (`manylinux`) und greifen dort nicht → `pip`/`uv` fallen auf den **Quellcode** zurück und **kompilieren**. Dokumentiert sind Builds von **über 20 Minuten statt 30 Sekunden** — und das Ergebnis ist am Ende oft *größer*, weil Compiler und Header mitinstalliert werden mussten.<br>Inzwischen gibt es **`musllinux`-Wheels** (PEP 656), auch für psycopg — aber das ist eine Eigenschaft **jedes einzelnen Pakets**, keine Garantie.<br>**Standardwahl: `python:3.13-slim`** (Debian, ~150 MB). Die ~100 MB Unterschied sind irrelevant gegenüber der Verlässlichkeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsquQtgFCkEnqehpKZi3B4v" -->
Nenne die zentralen **Dockerfile-Best-Practices** (mind. 6).::
1. **Reihenfolge nach Änderungshäufigkeit** — Abhängigkeiten oben, Quellcode unten<br>2. **`.dockerignore`** pflegen (≥ `.gitignore` + `.git`)<br>3. **Nicht als root laufen** — `USER 1000:1000` am Ende, kostet eine Zeile<br>4. **exec-Form** für `CMD`/`ENTRYPOINT` (PID-1-Falle)<br>5. **Ein Prozess pro Container** — kein `supervisord`; zwei Dinge = zwei Container<br>6. **Nach stdout loggen** — sonst zeigt `docker logs` nichts<br>7. **Basis-Image mit Tag, besser Digest** festnageln — nie `latest`<br>8. **apt-Idiom in *einem* `RUN`:** `apt-get update && apt-get install -y --no-install-recommends X && rm -rf /var/lib/apt/lists/*`<br>9. **Keine Secrets** in `ENV`/`ARG`/`COPY .env` — zur Bauzeit **BuildKit-Secrets** (`--mount=type=secret`), zur Laufzeit von außen<br>10. **`HEALTHCHECK`** setzen, wenn andere Dienste auf Bereitschaft warten sollen
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIlYxyWJNvOjw9RXN8ksDe" -->
Ist `0.0.0.0` dasselbe wie `localhost`? Und warum muss ein Server im Container darauf lauschen?::
**Nein.** `127.0.0.1`/`localhost` ist eine **konkrete Adresse** (Loopback-Schnittstelle). **`0.0.0.0` ist keine Adresse**, sondern ein **Platzhalter beim Lauschen**: „auf allen Schnittstellen annehmen". Man verbindet sich nie *zu* `0.0.0.0`.<br>Ein Container hat **zwei** Schnittstellen: `lo` (127.0.0.1) und **`eth0`** (die veth-IP, z. B. 172.17.0.2). Verkehr von außen kommt über **eth0** an — ein Server auf `127.0.0.1` nimmt ihn **gar nicht erst an** und ist trotz Port-Mapping unerreichbar.<br>Deshalb im Container **immer `--host 0.0.0.0`**, auf dem eigenen Rechner dagegen `127.0.0.1`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswX8D5eorP4xosh0uaLEy3" -->
`-p 8001:8000` — welche Zahl ist frei wählbar, und woher kommt die andere?::
Schema **`host : container`**.<br>**Links (Host)** ist **frei wählbar** — das tippst du in den Browser. Kollidiert sie, kommt `port is already allocated`; im Container ist derselbe Port trotzdem frei (`net`-Namespace).<br>**Rechts (Container)** muss zu dem passen, worauf der Prozess **tatsächlich lauscht**. Steht im `CMD` kein `--port`, ist es der **Default des Programms** (uvicorn: 8000) — ein stillschweigender Default, den man besser explizit hinschreibt.<br>Technisch: Der Daemon trägt eine **DNAT-Regel in `iptables`** ein, die Pakete vom Host-Port auf die Container-IP umschreibt und über die Bridge schickt (`sudo iptables -t nat -L DOCKER -n`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDKercBUK6bEpY4wldN3qp" -->
Docker lauscht auf dem Host-Port und leitet nach innen weiter — wie kommt die **Antwort** zurück?::
Zwei Gründe.<br>**1. Lauschen betrifft nur den Verbindungsaufbau.** `bind()`/`listen()` machen erreichbar, **`accept()` liefert einen neuen, eigenen Socket** für *diese eine Unterhaltung* — und der ist **beidseitig**. Der Listening-Socket wartet unverändert weiter. Die Antwort geht also nicht an einen Port zurück, sondern durch die bestehende Verbindung.<br>**2. NAT ist zustandsbehaftet.** Der Kernel merkt sich jede Übersetzung in der **conntrack**-Tabelle und **dreht sie auf dem Rückweg automatisch um** — aus `172.17.0.2:8000` wird wieder `127.0.0.1:8000`. Das *muss* so sein, sonst würde der Client die Antwort verwerfen (sie gehörte zu keiner ihm bekannten Verbindung). Deshalb gibt es nur **eine** DNAT-Regel.<br>Nebenbei: In der Standardkonfiguration lauscht dort wirklich ein Prozess — **`docker-proxy`**, sichtbar in `ss -tlnp`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsn29oNp1NxE5xCBDJu6God" -->
Was ist eine **Bridge** im Docker-Kontext?::
Ein **virtueller Switch** im Linux-Kernel. Ein Switch ist ein Kasten mit Buchsen: Was an Buchse 3 hereinkommt und für Buchse 5 bestimmt ist, wird dorthin weitergeschoben. Er übersetzt nichts und routet nicht — er arbeitet **unter** IP, mit **MAC-Adressen**.<br>Zwei Folgen: **Alles am selben Switch erreicht sich direkt** · **was an einem anderen Switch hängt, nicht** (solange keiner sie verbindet).<br>**Jedes Docker-Netzwerk ist ein eigener Switch** (`docker0`, `br-<netz-id>`). Container in verschiedenen Netzen sehen sich deshalb nicht — nicht weil etwas blockiert, sondern weil **kein Kabel dazwischen liegt**.<br>Der Name ist historisch: Ein Bridge verband ursprünglich zwei Netzsegmente; ein Switch ist ein Bridge mit vielen Anschlüssen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs462UAECkNZy0jyx4FA8rs" -->
Was ist ein **`veth`-Paar** und wie hängt ein Container am Netzwerk?::
Ein Container hat durch den `net`-Namespace einen **eigenen, anfangs leeren** Netzwerk-Stack — nur `lo`. Zum Anschließen braucht es ein Kabel.<br>Ein **`veth`-Paar** (virtual ethernet) entsteht immer als **Paar**: zwei Enden, was in das eine hineingeht, kommt aus dem anderen — wie ein Patchkabel.<br>Docker steckt **Ende A in die Bridge** und **verschiebt Ende B** in den Namespace des Containers, wo es **`eth0`** heißt. Deshalb erscheint auf dem Host pro laufendem Container ein `veth…`-Interface (`ip -brief link \| grep veth`).<br>Adressvergabe: Jedes Netz hat ein **Subnetz** (z. B. `172.18.0.0/16`); `…0.1` ist die **Bridge selbst und zugleich das Gateway**, danach der Reihe nach die Container. Docker spielt dabei DHCP-Server.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHnys1OgoBJFIZ12DEpX79" -->
Was heißt es, dass ein Container **im DNS des Netzes registriert** wird?::
Container-IPs sind **nicht stabil** — bei jedem Neustart kann je nach Startreihenfolge eine andere vergeben werden. Fest hinschreiben kann man sie nirgends.<br>Deshalb betreibt **Docker in jedem Container-Namespace einen winzigen DNS-Server unter `127.0.0.11`** (eine Loopback-Adresse *des Containers*; `cat /etc/resolv.conf` zeigt ihn). Er führt pro Netzwerk eine Tabelle **Name → aktuelle IP**.<br>**„Registriert" heißt: Docker hat den Container beim Anschließen mit seinem Namen in diese Tabelle eingetragen.** Eine Anfrage nach `db` geht an `127.0.0.11` und bekommt die *momentan gültige* Adresse. Unbekannte Namen (`google.com`) reicht Docker an die DNS-Server des Hosts weiter.<br>Im **vorgegebenen `bridge`-Netz gibt es diesen Dienst nicht** — daher die Regel, dass Namensauflösung nur in selbst angelegten Netzen funktioniert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsloLbbtRaLm6W0siOvk0ua" -->
Was löst `--network <netz>` beim Containerstart konkret aus? (4 Schritte)::
1. **Kabel gebaut** — ein `veth`-Paar erzeugt<br>2. **Eingesteckt** — Ende A in die **Bridge** des Netzes, Ende B als **`eth0`** in den Container-Namespace<br>3. **Adresse vergeben** — die nächste freie aus dem **Subnetz** des Netzes<br>4. **Namen eingetragen** — Containername in die **DNS-Tabelle** dieses Netzes (`127.0.0.11`)<br>Ohne 1–2 hinge der Container am **falschen Switch**; ohne 4 wäre der Dienstname ein **unbekannter Hostname**.<br>Wichtig: Container-zu-Container-Verkehr läuft **direkt über die Bridge** — ohne Host-Stack, ohne `iptables`-DNAT, ohne `docker-proxy`. Ein veröffentlichter Port (`ports:`) ist dafür **irrelevant** und existiert nur für den Zugriff vom Host.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsemoK0qyqL0y8aY8qVKi2F" -->
Nenne die **zwei** PID-1-Fallen in Containern — Ursache, Symptom, Lösung.::
**Falle A – Signale:** Die **Shell-Form** von `CMD` macht `/bin/sh -c "…"` zu PID 1, deine Anwendung ist PID 2 (ein Kind davon). `docker stop` schickt `SIGTERM` **nur an PID 1**, die Shell **reicht es nicht weiter** → jedes Herunterfahren dauert 10 s und endet hart. **Lösung: exec-Form** (JSON-Array) — oder ein Skript, das mit **`exec "$@"`** endet (`exec` *ersetzt* die Shell, statt ein Kind zu starten).<br>**Falle B – Zombies:** PID 1 hat die **Pflicht**, verwaiste Kindprozesse per `wait()` einzusammeln. Ist PID 1 deine Anwendung, weiß sie davon nichts → die Prozesstabelle füllt sich langsam mit **Zombies**. Betrifft Programme, die forken (CI-Runner, `subprocess.run()`, Headless-Browser). **Lösung: `docker run --init` bzw. `init: true`** — schiebt **tini** als PID 1 davor, das Signale weiterleitet *und* Zombies einsammelt.<br>Verschärfend: Der Kernel stellt Signale mit Standardverhalten (`SIGTERM`) an **PID 1 gar nicht zu**, solange kein eigener Handler registriert ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHxM72LDmGKFDVCS6Uj45l" -->
Unterscheide `external: true` und `internal: true` in Compose.::
**Zwei völlig verschiedene Dinge** mit ähnlichen Namen:<br>**`external: true`** (Volumes, Netzwerke, Secrets, Configs) regelt den **Besitz**: „Compose verwaltet das nicht." → Es wird **nicht angelegt** (Fehler, falls es fehlt), bekommt **kein Projekt-Präfix** und wird bei **`down -v` nicht gelöscht**. Typisch für: Volume von zwei Projekten geteilt · **Produktionsdaten vor versehentlichem `down -v` schützen** · von außen verwaltet (Terraform, NFS).<br>**`internal: true`** (**nur Netzwerke**) regelt die **Erreichbarkeit**: Docker richtet **kein Gateway nach draußen** ein. Container darin erreichen sich untereinander, aber weder das Internet noch der Host sie. Muster: `db` nur im internen `backend`-Netz, `api` in `backend` **und** `frontend` — nur die Presentation-Schicht ist von außen erreichbar.<br>Preis: Ein `ports:`-Eintrag am internen Dienst widerspricht der Absicht; Zugriff dann über `docker compose exec db psql`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVkgRwzzo9jXXn2OvrviXv" -->
Wie funktioniert **`HEALTHCHECK`** — Parameter, Zustände, Wirkung?::
Docker führt in Abständen einen Befehl **im Container** aus und wertet **nur den Exit-Code** aus (`0` = gesund, alles andere = ungesund).<br>**`--interval`** Abstand zwischen Prüfungen · **`--timeout`** wie lange eine Prüfung dauern darf · **`--retries`** Fehlschläge in Folge bis `unhealthy` · **`--start-period`** **Karenzzeit**, in der Fehlschläge **nicht** gegen `retries` zählen (der erste Erfolg beendet sie vorzeitig).<br>Zustände: **`starting` → `healthy` / `unhealthy`**.<br>**Docker tut von sich aus nichts damit** — ein `unhealthy`-Container wird **nicht** neu gestartet. Der Healthcheck ist ein **Signal**, das andere auswerten: `depends_on: condition: service_healthy`, `up --wait`, Kubernetes.<br>Protokoll der letzten Prüfungen: `docker inspect --format '{{json .State.Health}}' <c> | jq`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsB373opZetYtJfRQ84YhM1" -->
Was tut `RUN --mount=type=cache,target=…` — und wie verhält es sich zum **Layer-Cache**?::
BuildKit hängt für die Dauer *dieses einen* `RUN` ein Verzeichnis ein, das **nicht zur Schicht gehört**, aber **zwischen Builds überlebt**. Ohne das landete z. B. uvs Download-Cache dauerhaft im Image (und wäre wegen Whiteout nicht mehr entfernbar).<br>**Sie ergänzen sich, statt sich zu überlappen:**<br>**Layer-Cache** → der Schritt wird **übersprungen** (`CACHED`), greift, wenn sich nichts geändert hat.<br>**Cache-Mount** → der Schritt **läuft**, aber mit warmem Cache; zahlt sich genau dann aus, wenn der Layer-Cache **verfehlt** (z. B. `uv.lock` geändert → nur das neue Paket wird geladen, nicht alle 40).<br>`UV_LINK_MODE=copy` gehört dazu, weil **Hardlinks nicht über die Mount-Grenze** funktionieren. Aufräumen: `docker builder prune`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsxr7t1a0le2CPr68O3OMFh" -->
Unterschied **`RUN`** vs. **`CMD`**?::
**`RUN` = Fließband, `CMD` = Einschaltknopf.**<br>**`RUN`** — beim **Bauen**; erzeugt eine **neue Schicht**; **nicht überschreibbar** (eingebacken); beliebig viele.<br>**`CMD`** — beim **Starten**; nur ein **Eintrag in der Config**; **überschreibbar** durch Argumente nach dem Image-Namen; nur die **letzte** zählt.<br>`RUN uvicorn …` würde den Server *während des Builds* starten und ewig hängen (und ein Prozess lässt sich nicht in eine Schicht backen). `CMD ["uv","sync"]` würde bei *jedem* Containerstart neu installieren.<br>Dass `CMD` überschreibbar ist, nutzt der `migrate`-Dienst: dasselbe Image, `command: ["alembic","upgrade","head"]`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLyOf5ec2vEgq9VcK20xmw" -->
Warum ist der Layer-Cache eine **Kette** — und wie *beweist* man, dass er greift?::
Der Cache-Schlüssel jeder Anweisung enthält den **Schlüssel der vorherigen**. Bricht ein Glied, sind **alle folgenden** Schichten ungültig.<br>Deshalb ist ein übrig gebliebenes **`COPY . .` weit oben fatal**: Es reagiert auf **jede** Dateiänderung im Kontext und macht jede Reihenfolge-Optimierung darunter wirkungslos.<br>**Beweisen statt messen:** Zeitmessung schwankt mit Netz und Systemlast — der verlässliche Indikator ist das Wort **`CACHED`** in der Build-Ausgabe. Es muss beim **teuren `uv sync`** stehen, nachdem man eine Zeile Quellcode geändert hat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWpqCjhIT6QDXA4K7tWoIM" -->
`ModuleNotFoundError: No module named '<eigenes Paket>'` im Container — die **zwei** Ursachen?::
Identisches Symptom, verschiedene Stellen:<br>**1. `--no-install-project` stand bei *beiden* `uv sync`.** Der Sinn der Zweiteilung ist ja, dass Lauf 1 **nur Abhängigkeiten** installiert und Lauf 2 **das eigene Paket**. → Flag im zweiten Aufruf entfernen.<br>**2. Editable Install** (Standard): uv legt nur einen **Verweis auf `/app/src`** ins `.venv` — und `src/` wird nicht in die Runtime-Stage kopiert, der Verweis zeigt ins Leere. → **`--no-editable`**, dann liegt der Code echt in `site-packages`.<br>Prüfen: `docker run --rm <image> python -c "import pricing; print(pricing.__file__)"` muss einen `site-packages`-Pfad zeigen.
<!-- basicblock-end -->

# Docker Volumes und Datenpersistenz

<!-- basicblock-start oid="Obs47C9AhreOuJIeCroNlOnL" -->
Löst Docker das Zustandsproblem?::
**Nein — es lokalisiert es.** Der Gewinn ist nicht, dass Zustand verschwindet, sondern dass er an genau **einer benannten, sichtbaren Stelle** liegt statt überall verstreut.<br>Ein Container **ohne** Volume ist vollständig wegwerfbar. Ein Container **mit** Volume ist wegwerfbar **bis auf einen Pfad**, den man explizit hingeschrieben hat.<br>Derselbe Satz wie in Kapitel 1.6, eine Ebene tiefer: **Code ist ersetzbar, Daten sind es nicht.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspQhfbKJ4j6IDg6Qk8q130" -->
Nenne **zwei** Gründe, warum eine Datenbank nicht in die beschreibbare Container-Schicht schreiben darf.::
1. **Persistenz** — die Schicht verschwindet mit `docker rm`.<br>2. **Performance** — OverlayFS arbeitet nach **Copy-on-Write**: Beim *ersten* Schreibzugriff wird die Datei **vollständig** in die obere Schicht kopiert, nicht nur der geänderte Block. 4 KB in einer 2-GB-Datei ändern = **2 GB kopieren**. Dazu der generelle Aufwand, dass jeder Zugriff durch die Union-Schicht muss.<br>Ein **Volume umgeht OverlayFS vollständig** — direkter Zugriff aufs Host-Dateisystem, ohne Copy-on-Write. Deshalb steht in der Doku jedes Datenbank-Images „benutze ein Volume": nicht nur für die Persistenz, sondern damit die DB überhaupt brauchbar schnell ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAGJ4GlQZ5qTH2zUui82Wy" -->
Wo liegt ein Named Volume physisch — und warum sollst du das nicht wissen müssen?::
Unter `/var/lib/docker/volumes/<name>/_data`.<br>Aber: Ein Volume ist ein **Objekt mit Namen**; wo es liegt, ist eine **Implementierungsfrage**. `local` ist nur der Standardtreiber — über `driver_opts` kann derselbe Name z. B. auf einen **NFS-Share** zeigen (`type: nfs`, `device: ":/export/daten"`).<br>Das ist der Sinn der Abstraktion: **Der Container weiß nie, was an seinem Pfad hängt.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvwAEdnDL2xoM5tV5rw7oU" -->
**Mount-Semantik:** Wer gewinnt, wenn im Image an der Zielstelle schon Dateien liegen?::
**Named Volume, leer** → Docker **kopiert den Image-Inhalt ins Volume** (einmalig, beim Erzeugen)<br>**Named Volume, gefüllt** → das **Volume gewinnt**, der Image-Inhalt bleibt unsichtbar<br>**Bind Mount** (leer oder voll) → der **Host-Pfad gewinnt immer**, es wird **nie** vorbefüllt<br>**Kernsatz:** Ein Named Volume wird **genau einmal** aus dem Image befüllt. Danach ist es die Quelle der Wahrheit, und Änderungen im Image erreichen es nie mehr.<br>Damit erklärt sich der 1.6-Fall vollständig: neuer Ordner → neuer Projektname → **neues leeres** Volume → einmalige Erstbefüllung → frisch initialisierte, leere Datenbank. Kein Fehler, sondern dokumentiertes Verhalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHuqNMiq1wghJRDkKqDFaH" -->
Warum hängt man ein **anonymes Volume über ein Unterverzeichnis** eines Bind Mounts?::
```yaml
- ./:/app          # Quellcode vom Host
- /app/.venv       # anonymes Volume — schützt dieses Unterverzeichnis
```
Der **spezifischere (längere) Pfad gewinnt**. Ohne die zweite Zeile würde die **Host**-`.venv` (bzw. `node_modules` in der JS-Welt) die im Container installierte überdecken — mit Bibliotheken, die für das **falsche System** kompiliert wurden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPxdQaySnBFPpttzLgeocJ" -->
Warum ist `-v /tippfehler:/daten` gefährlicher als die `--mount`-Syntax?::
**`-v` legt fehlende Host-Pfade stillschweigend an** — leer und **root gehörend**. Statt eines Fehlers bekommst du einen Container, der ins Nichts schaut. Klassisch bei Tippfehlern im Pfad.<br>**`--mount type=bind,source=…,target=…,readonly`** ist wortreicher, aber **explizit und ohne Sonderregeln** — es scheitert sauber, wenn die Quelle fehlt.<br>In Compose entspricht das der **long syntax** mit `type:`/`source:`/`target:` statt der Kurzform.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsyr7I9IzyVKorQbaJ050TO" -->
Warum entstehen im Container **Rechteprobleme**, und wie löst man sie?::
Im Container gibt es **keine Übersetzung von Benutzern** — der Kernel kennt nur **numerische UIDs/GIDs**, und die sind auf Host und Container dieselben Zahlen, auch bei unterschiedlichen Namen.<br>**Symptom 1:** „Permission denied" — Prozess läuft als UID 1000, die Dateien gehören UID 0 (oder umgekehrt). Bei Bind Mounts gelten schlicht die Host-Rechte.<br>**Symptom 2:** Ein Container, der als root schreibt, erzeugt **root-Dateien in deinem Projektverzeichnis** — danach hilft nur `sudo rm`.<br>**Gegenmittel:** `COPY --chown=1000:1000 …` im Dockerfile · `USER 1000:1000` · zur Laufzeit `docker run -u "$(id -u):$(id -g)"`.<br>Bei **Named Volumes** ist es milder: Beim ersten Anlegen übernimmt Docker die Rechte des Zielverzeichnisses aus dem Image. Bei **Bind Mounts** gibt es diese Hilfe nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsng2J5tt4hfrbEW0BnB83I" -->
Wie sicherst du ein **Volume** — und warum ist genau das bei einer laufenden Datenbank falsch?::
**Es gibt kein `docker volume backup`.** Ein Volume ist ein Verzeichnis; sichern musst du selbst. Muster: ein **Wegwerf-Container** hängt Volume und Host-Verzeichnis ein und packt das eine ins andere:<br>`docker run --rm -v pricing_data:/quelle:ro -v "$(pwd)":/backup alpine tar czf /backup/x.tar.gz -C /quelle .`<br>**Bei laufenden Datenbanken unbrauchbar:** PostgreSQL hält Daten im RAM, schreibt asynchron und führt ein WAL — ein `tar` erwischt einen **Zwischenzustand**. Die Sicherung *sieht* vollständig aus und ist es nicht; man merkt es erst beim Wiederherstellen.<br>**Richtig:** das Werkzeug der Datenbank benutzen — `docker compose exec -T db pg_dump -U pricing pricing > backup.sql` (`-T` schaltet die TTY ab, nötig beim Umleiten).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9TaYUER7JnDeQtA0GfRs8" -->
Welche Daten gehören **nicht** in ein Volume?::
**Logs** → nach **stdout**, der Daemon sammelt sie ein (sonst zeigt `docker logs` nichts und niemand sichert die Dateien)<br>**Konfiguration/Secrets** → Umgebungsvariablen bzw. Secret-Mechanismen<br>**Caches/Temporäres** → `tmpfs` oder einfach die Container-Schicht<br>**Quellcode in Produktion** → **ins Image**, er gehört zum Artefakt<br>**Ja, ins Volume gehören:** Datenbank-Datenverzeichnis; hochgeladene Dateien (klein, Einzelhost — in der Cloud besser Objektspeicher/S3).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsxmf8Bknz6C85NY9vRVt8e" -->
Warum ist ein Volume kein tragfähiges Konzept, sobald die Anwendung auf **mehreren Servern** läuft?::
**Ein Volume ist ein Einzelhost-Konzept** — es liegt auf *dieser* Maschine. Auf mehreren Knoten braucht es Netzwerkspeicher, in Kubernetes **`PersistentVolumes`**, in der Cloud einen **verwalteten Datenbankdienst**.<br>Ehrliche Einordnung: **In Produktion betreibt man Datenbanken häufig gar nicht selbst in Containern**, sondern nimmt RDS/Cloud SQL — Backups, Failover und Updates sind schwierig genug.<br>In der **Entwicklung** ist die DB im Container dagegen ideal: exakt reproduzierbar, in Sekunden neu, ohne das System zu verschmutzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvjbiZ8ldAbLNTC42BbdbL" -->
Beantworte die Kontrollfrage: **Wie funktioniert Datenpersistenz in Docker? Erkläre Volumes.**::
Ein Container schreibt standardmäßig in seine **dünne beschreibbare Schicht**, die mit `docker rm` verschwindet. Persistenz entsteht, indem man **einzelne Pfade aus dieser Schicht heraushebt** — per **Named Volume**, **Bind Mount** oder **tmpfs**.<br>Ein **Volume** ist ein von Docker verwaltetes, **eigenständiges Objekt** mit eigenem Namen und Lebenszyklus. Es umgeht das **Union-Dateisystem vollständig** (Persistenz *und* bessere Schreibleistung) und **überlebt seinen Container** — `docker rm` löscht es nicht; dafür braucht es `docker rm -v`, `docker compose down -v` oder `docker volume prune`.<br>**Konzeptioneller Kern:** Docker macht Anwendungen nicht zustandslos, sondern **zwingt dazu, jeden Zustand explizit zu benennen**. Alles, was nicht in einem Volume steht, ist per Definition wegwerfbar — daraus entsteht die Reproduzierbarkeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiyySXCygBMKtUevII9Lzo" -->
Wie unterscheidet Compose in der **Kurzform** zwischen Named Volume und Bind Mount?::
**Rein textuell am ersten Zeichen der linken Seite:**<br>`.` oder `/` → ein **Pfad** → **Bind Mount** (`./daily_data:/app/daily_data:ro`)<br>alles andere → ein **Name** → **Named Volume** (`pricing_data:/var/lib/postgresql`)<br>nur ein Pfad ohne Doppelpunkt → **anonymes Volume** (`/app/.venv`)<br>`pricing_data:…` und `./pricing_data:…` unterscheiden sich um **zwei Zeichen** und bedeuten Verschiedenes.<br>Das Top-Level-`volumes:` deklariert **nur Named Volumes** (sonst `refers to undefined volume`); Bind Mounts gehören dort nicht hin — sie sind keine Docker-Objekte.<br>**Langform** (`type: volume|bind|tmpfs`, `source:`, `target:`, `read_only:`) macht den Typ **explizit** — wichtig, weil ein **Tippfehler im Host-Pfad** in der Kurzform als *anderer* Bind Mount gilt und Docker das Verzeichnis stillschweigend anlegt (leer, root gehörend).<br>Prüfen: `docker inspect <c> --format '{{range .Mounts}}{{.Type}} {{.Source}}{{end}}'` — `docker volume ls` zeigt Bind Mounts gar nicht.
<!-- basicblock-end -->

# Docker Compose: Multi-Container-Anwendungen

<!-- basicblock-start oid="Obsda8u9kGNl1z7gECR7LtBQ" -->
Welche **Symmetrie** besteht zwischen Dockerfile und Compose-Datei?::
Beide sind der Schritt **von imperativ zu deklarativ** — auf zwei verschiedenen Ebenen:<br>**Bauzeit:** eine Kette von Befehlen → **Dockerfile**<br>**Laufzeit:** eine Kette von `docker run` → **`compose.yaml`**<br>Das Dockerfile beschreibt, *woraus ein Image besteht*; Compose beschreibt, *woraus eine Anwendung besteht* (Dienste, Netze, Volumes, Abhängigkeiten). Beides ist Text, liegt in git und durchläuft Code Review.<br>Eine Kette von `docker run`-Befehlen funktioniert zwar, ist aber **Prosa in Befehlsform**: nicht versioniert, in der Reihenfolge fehleranfällig, beim zweiten Mal anders getippt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsw5lsXZ1bOwUBxUfVcyyzX" -->
Unterscheide die **drei Ebenen von Umgebungsvariablen** in Compose.::
**`.env` im Projektverzeichnis** → wirkt auf die **Compose-Datei selbst**: Substitution von `${VAR}`, z. B. `image: postgres:${POSTGRES_VERSION}`<br>**`env_file:`** im Dienst → **alle** Variablen der Datei landen **im Container**<br>**`environment:`** im Dienst → einzeln benannte Variablen **im Container**<br>**Die Falle:** `.env` wird **automatisch** für die Substitution gelesen — ihre Variablen landen deswegen aber **nicht** im Container! Wer das erwartet, sucht lange. Dafür braucht es explizit `env_file: [.env]`.<br>Vorrang (später gewinnt): `environment:` > `env_file:` > `ENV` aus dem Dockerfile.<br>Klärt jede Diskussion: **`docker compose config`**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuEpYJhlqwd4PCoBdHn3xs" -->
Warum reicht `depends_on: [db]` nicht — und wie macht man es richtig?::
`depends_on` allein garantiert nur die **Reihenfolge**, nicht die **Bereitschaft**: Der Daemon meldet „gestartet", sobald der Prozess läuft — PostgreSQL braucht danach noch Sekunden, bis es Verbindungen annimmt. Die API startet in diese Lücke und stirbt mit `connection refused`.<br>**Richtig:** ein `healthcheck` beim Dienst **plus** die Bedingung beim Abhängigen:<br>`depends_on: {db: {condition: service_healthy}}`<br>Ohne `condition` ist ein Healthcheck **reine Anzeige** in `docker ps`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsheHRFf9nZ7fc4jR46H4Oz" -->
Nenne die drei **`condition`-Werte** von `depends_on` und je einen Einsatzzweck.::
**`service_started`** — Container läuft (Default, schwach)<br>**`service_healthy`** — **Healthcheck meldet gesund**; für Datenbanken, auf die die App wartet<br>**`service_completed_successfully`** — Container ist **mit Exit-Code 0 beendet**; genau das Muster für einen **Migrations-Dienst**, der vor der API laufen muss<br>Dazu: **`start_period`** im Healthcheck ist die **Karenzzeit**, in der Fehlschläge noch **nicht** gegen `retries` zählen. Wird oft vergessen — ohne sie gilt ein Dienst, der 30 s zum Hochfahren braucht, vorschnell als ungesund.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsmf7VPaxmF3WODIuQQrPaZ" -->
Warum hilft `depends_on` **nicht**, wenn die Datenbank im laufenden Betrieb neu startet?::
Weil **`depends_on` nur beim Start gilt.** Fällt die DB später aus, greift keine Compose-Einstellung mehr.<br>Die **Anwendung** muss Verbindungsabbrüche selbst überstehen — durch Wiederholversuche und einen Connection Pool, der tote Verbindungen erkennt. SQLAlchemy bietet dafür **`pool_pre_ping=True`**.<br>Merksatz: **Compose löst das Startproblem; die Robustheit im Betrieb bleibt Aufgabe der Anwendung.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsY0OskYz67cXicSEKltkhK" -->
Drei Wege, **Migrationen** vor dem API-Start auszuführen — mit Bewertung.::
**A — im Entrypoint der API:** Skript führt `alembic upgrade head` aus, dann `exec uvicorn …`. Einfach; aber bei **mehreren API-Instanzen migrieren alle gleichzeitig**.<br>**B — eigener Einmal-Dienst:** ein Dienst `migrate`, der nur migriert und endet; die API wartet per **`condition: service_completed_successfully`**. **Sauber und explizit**, ein Dienst mehr — das in echten Projekten häufigere Muster.<br>**C — manuell:** `docker compose run --rm api alembic upgrade head`. Explizit, aber **verletzt die Aufgabenstellung** („ein einziger Befehl").
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszxSuF80MielprxASGUoyh" -->
Wozu dienen **Profile** in Compose?::
Dienste mit `profiles: ["dev"]` starten **nur**, wenn das Profil aktiv ist:<br>`docker compose up -d` (ohne) vs. `docker compose --profile dev up -d` (mit).<br>**Typischer Kandidat: Adminer** — ein Entwicklungswerkzeug, das in Produktion nichts verloren hat (ungeschützte DB-Oberfläche auf Port 8080). Mit einem Profil bleibt es in **derselben Datei**, startet aber nur auf Wunsch.<br>Verwandt: **Override-Dateien.** Compose liest automatisch `compose.yaml` **und** `compose.override.yaml`; für andere Umgebungen `-f compose.yaml -f compose.prod.yaml`. Typischer Override-Inhalt = genau das, was in Produktion falsch wäre: Bind Mount des Quellcodes, `--reload`, Debug-Ports.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBF7Rw51Gq31jJTKc2d96G" -->
Was löscht `docker compose down` — und was ändern `-v` und `--rmi all`?::
**`down`** entfernt **Container und Netzwerk**; **Volumes und Images bleiben**.<br>**`down -v`** entfernt **zusätzlich die Volumes** — also womöglich die Datenbank. **Vorsicht.**<br>**`down --rmi all`** entfernt zusätzlich die **Images**.<br>Zum Vergleich: **`stop`** stoppt nur (Container, Netz, Volumes bleiben alle bestehen).<br>Feinheit: **`up` erzeugt Container neu, wenn sich ihre Konfiguration geändert hat** — deshalb wirkt eine Änderung an `environment:` sofort, eine am **Code** aber nicht: dafür muss das **Image** neu gebaut werden (`up -d --build`). Das ist die häufigste Ursache für „meine Änderung wirkt nicht".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBNJFPOAtHB1XxJOPfq43T" -->
Nenne die **Vorteile von Docker Compose** (Kontrollfrage).::
1. **Deklarative, versionierte Anwendungstopologie** — welche Dienste, wie verbunden, welche Volumes/Netze: eine Textdatei im Repo statt Befehlshistorie.<br>2. **Ein Befehl statt vieler** — reproduzierbar, richtige Reihenfolge, keine Tippfehler.<br>3. **Automatische Vernetzung mit DNS** — eigenes Netz, Dienstname = Hostname.<br>4. **Abhängigkeiten ausdrückbar** — inkl. „warte, bis die DB *bereit* ist" und „migriere vor dem API-Start".<br>5. **Isolation zwischen Projekten** über den Projektnamen (eigenes Netz, eigene Volume-/Containernamen).<br>6. **Ein Modell für mehrere Umgebungen** über Overrides und Profile.<br>7. **Onboarding und CI werden trivial** — der belastbare Test, ob die Anwendung vollständig *beschrieben* ist.<br>Der wichtigste: Compose macht aus „wie läuft dieses System eigentlich?" etwas, das man **lesen** kann, statt es zu rekonstruieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2emRlU9sqX1B7ZcEcflWI" -->
Wo liegen die **Grenzen** von Compose?::
**Einzelhost-Werkzeug** — verteilt nichts über mehrere Server, kein Scheduling, keine Ausfallsicherheit. Dafür Swarm oder **Kubernetes**.<br>**Kein Rolling Update** — `up` stoppt und ersetzt Container; kurze Unterbrechung inklusive.<br>**Kein echtes Secret-Management** — `secrets:` ohne Swarm sind schlicht Dateien im Dateisystem.<br>**Aber:** Für einen einzelnen Server mit überschaubarer Last ist Compose eine völlig legitime **Betriebsform**; viele kleinere Systeme laufen genau so. Der Sprung zu Kubernetes lohnt erst bei mehreren Knoten oder automatischer Skalierung.<br>Nützliche Optionen: **`--no-deps`** (nur diesen Dienst neu starten) und **`--wait`** (kehrt erst zurück, wenn alle Healthchecks gesund sind — gut für CI).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWjz3dpx3d2Tr3uxEs8sn2" -->
Was gehört unter `build:` in einem Compose-Dienst — und was ist mit `version:`?::
```yaml
build:
  context: .          # der Build-Kontext
  dockerfile: Dockerfile
  target: runtime     # Multi-Stage: bis zu dieser Stage
  args: {…}
image: pricing-api:latest   # Name für das ERGEBNIS
```
`context` ist genau der Build-Kontext aus dem Dockerfile-Stichwort, `target` die Multi-Stage-Auswahl. Steht `image` zusammen mit `build`, ist es der Tag für das gebaute Image.<br>**`version: "3.8"` ist obsolet** — Compose V2 warnt davor. Wenn ein Tutorial damit anfängt, ist es alt; das gilt meist auch für den Rest seines Inhalts.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskkFRD5eTxvn3xISeTY0zy" -->
YAML in Compose: wann **Einrückung**, wann **`-`**, wann **`[]`**?::
YAML kennt nur **zwei** Strukturen, der Rest ist Schreibweise:<br>**Mapping** (Schlüssel → Wert): eingerückte `schlüssel: wert`-Zeilen<br>**Sequence** (Liste): Einträge mit `-`<br>**`[ "a", "b" ]` ist der Flow-Stil und für den Parser identisch** mit der `-`-Variante (Block-Stil).<br>**Mapping:** `services`, `volumes` (oben) · **Liste:** `ports`, `profiles`, `command`, `entrypoint`, `healthcheck.test`<br>**Beides erlaubt:** `environment` (`KEY: wert` **oder** `- KEY=wert`) · `depends_on` (**Liste** = Kurzform, **Mapping** = Langform mit `condition`).<br>Deshalb muss `depends_on` von `-` auf Einrückung umgebaut werden, sobald man `condition: service_healthy` braucht — in der Listenform lässt es sich nicht ausdrücken.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7HrZABiZ9TnY1Ozi6hwEH" -->
Die vier `restart`-Werte — und warum muss `"no"` in Anführungszeichen?::
`"no"` (Default, nie) · `on-failure` (nur bei Exit ≠ 0) · `always` (immer, auch nach `docker stop` und Reboot) · **`unless-stopped`** (wie `always`, außer du hast selbst gestoppt).<br>**`api: unless-stopped`** — nach Absturz/Reboot wieder da, aber ein bewusstes Stoppen bleibt respektiert.<br>**`migrate: "no"`** — ein Einmal-Dienst darf sich **nie** neu starten: Er endet *absichtlich* mit 0; `always` ergäbe eine Endlosschleife und `service_completed_successfully` träte nie ein.<br>**Anführungszeichen:** In YAML 1.1 sind **`no`, `yes`, `on`, `off` Booleans** — ohne Quotes liest der Parser `restart: false`, Compose erwartet aber eine Zeichenkette. (Dieselbe Falle bei Ländercode `NO` und `version: 3.10` → 3.1.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsf8FtewMsZMTLprOrWKkER" -->
Was bedeutet die Syntax `x-app-env: &app-env` … `environment: *app-env`?::
**YAML-Anker — kein Compose-Feature.**<br>**`&name`** definiert („merk dir diesen Block") · **`*name`** referenziert („setz ihn hier ein") · **`<<: *name`** ist der **Merge-Key**: einsetzen **und** ergänzen/überschreiben.<br>Der Parser löst das auf, **bevor Compose die Datei überhaupt sieht** — Compose bekommt den Block zweimal ausgeschrieben.<br>Das **`x-`**-Präfix markiert ein **Extension Field**, das Compose bei der Validierung ignoriert; ohne das gäbe es einen Fehler wegen eines unbekannten Top-Level-Schlüssels.<br>Kontrolle: **`docker compose config`** zeigt die aufgelöste Datei ohne Anker.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAljvy1ymloWdegHagkWaz" -->
Woher kommt das Netzwerk `<projekt>_default`, obwohl in der Compose-Datei kein Wort über Netzwerke steht?::
**Compose legt es beim ersten `up` automatisch an.** Der implizite Name ist `default`, davor kommt der **Projektname** (aus `name:` oder sonst dem **Verzeichnisnamen**). Compose hängt danach **jeden Dienst hinein** und **registriert ihn unter seinem Dienstnamen im DNS**.<br>**Warum nicht das vorgegebene `bridge`-Netz?** Weil es dort **keine Namensauflösung über Containernamen** gäbe (dann wäre `db` als Hostname unmöglich) und **keine Isolation zwischen Projekten** — zwei Projekte mit je einem Dienst `db` kämen sich in die Quere.<br>Lebensdauer: **`stop` lässt es bestehen, `down` entfernt es.**<br>Zum Vergleich: `bridge`, `host` und `none` legt der **Daemon beim Start** an; sie existieren auf jeder Installation.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvcOpGlseR2HOw9edXtXcc" -->
`docker compose down` meldet **„Network … Resource is still in use"** — häufigste Ursache?::
Beim `up` war ein **Profil** aktiv, beim `down` nicht:<br>`docker compose --profile dev up -d` → startet auch `adminer`<br>`docker compose down -v` → **sieht `adminer` gar nicht**<br>Compose bezieht bei **jedem** Befehl nur Dienste ein, deren Profile **aktiv** sind. Der übrig gebliebene Container hängt weiter im Projektnetz — und ein Netzwerk lässt sich nicht entfernen, solange ein **laufender** Container daran hängt. (Nicht die offene Weboberfläche ist schuld, sondern der angeschlossene Container.)<br>**Merkregel: `down` muss dieselben Profile kennen wie `up`.**<br>`docker compose --profile dev down -v` · `--profile "*"` · `--remove-orphans` · dauerhaft: `COMPOSE_PROFILES=dev` in der `.env`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9jagnmvLiddFMrjdqFge1" -->
Warum scheitert `docker container stop adminer`, während `docker compose stop adminer` funktioniert?::
**`docker` kennt keine Dienste, nur Container.**<br>**Dienstname** `adminer` → für `docker compose <befehl> adminer`<br>**Containername** `pricing-adminer` → für `docker <befehl> pricing-adminer`<br>Ohne `container_name` in der Compose-Datei hieße der Container **`<projekt>-<dienst>-<nummer>`**, also `pricing-adminer-1`. Genau deshalb lässt man `container_name` meist weg: Dann ist der Containername **aus dem Dienstnamen ableitbar**, statt eine zweite, unabhängige Bezeichnung zu sein (die zudem Skalierung verhindert und projektübergreifend kollidiert).
<!-- basicblock-end -->
