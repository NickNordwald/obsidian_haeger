---
tags:
  - documentation
---

# Kubernetes Crashkurs — Überblick (Deutz)

Einstiegstext für die Kubernetes-Vorbereitung. Erst diesen Text lesen (~30 min), dann die [[Kubernetes Leseliste]], am Ende die [[Pricing-API auf Kubernetes - Übung|Übung]].

> [!note] Fortsetzung im Projekt
> Der eigentliche Lernpfad für das Projekt liegt jetzt unter [[Projects/Deutz/Kubernetes/Kubernetes|Projects/Deutz/Kubernetes]]. Dieser Crashkurs bleibt als Archiv der Interview-Vorbereitung.

Übergeordnet: [[Interview Vorbereitung]] · Schwesterthema: [[React Crashkurs - Übersicht|React]] · Vorwissen: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.7 Docker/1.7 Docker - Vertiefung|1.7 Docker]]

> [!important] Ziel: das Modell erklären und `kubectl` bedienen können
> Wie beim React-Crashkurs ist das Ziel nicht, Kubernetes zu *betreiben* (das ist ein eigener Beruf), sondern:
> 1. in zwei Minuten erklären, **welches Problem** Kubernetes löst und **wie** es das tut (deklarativ + Reconciliation),
> 2. die **fünf, sechs Kernobjekte** (Pod, Deployment, Service, ConfigMap/Secret, Volume) sicher benennen und abgrenzen,
> 3. einen Cluster **mit `kubectl` bedienen und debuggen** — und das an der eigenen Übung belegen können.

## Inhalt

1. [[#1. Das Problem: Compose endet an der Maschinengrenze]]
2. [[#2. Die Kernidee: gewünschter Zustand + Regelkreis]]
3. [[#3. Architektur: Control Plane und Nodes]]
4. [[#4. Die Kernobjekte]]
5. [[#5. Compose → Kubernetes: die Übersetzungstabelle]]
6. [[#6. Wie man es bedient: kubectl und Manifeste]]
7. [[#7. Debuggen: die vier Befehle und die typischen Fehlerbilder]]
8. [[#8. Das Ökosystem: Vokabeln]]
9. [[#9. Fürs Gespräch: Kontrollfragen]]
10. [[#10. Offene Fragen]]

---

## 1. Das Problem: Compose endet an der Maschinengrenze

Aus [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.7 Docker/1.7 Docker - Vertiefung|Kapitel 1.7]] kennst du den Stand: Ein Image ist ein standardisiertes, startbares Artefakt, und `compose.yml` beschreibt, welche Container mit welchen Netzen, Volumes und Umgebungsvariablen zusammenspielen. Genau dort steht aber auch die Grenze: **Compose ist ein Einzelhost-Werkzeug.**

Sobald eine Anwendung ernsthaft betrieben wird, kommen Fragen, die Compose nicht beantwortet:

| Frage | Compose | Kubernetes |
|---|---|---|
| Der Server stirbt — wer startet die Container woanders neu? | niemand | der Scheduler verteilt die Pods auf andere Nodes |
| Last steigt — wie komme ich von 2 auf 10 Instanzen, verteilt auf mehrere Maschinen? | `--scale` auf *einem* Host | `replicas: 10`, automatisch über alle Nodes; mit HPA sogar selbsttätig |
| Neue Version ausrollen, ohne Ausfall, mit Rückweg? | Container stoppen, neu starten | Rolling Update + `kubectl rollout undo` |
| Wie finden sich Dienste, wenn Container ständig woanders landen? | Compose-DNS auf einem Host | Service = stabile IP + DNS-Name clusterweit |
| Ein Container hängt, läuft aber noch? | `restart:` reagiert nur auf Exit | Liveness-Probe erkennt es und startet neu |

Kubernetes ist also ein **Container-Orchestrator**: Es verwaltet Container über einen *Cluster* aus vielen Maschinen hinweg, als wäre er ein großer Rechner.

**Herkunft in drei Sätzen:** Kubernetes (griech. *κυβερνήτης*, „Steuermann"; kurz **K8s** — K, acht Buchstaben, s) entstand bei Google als Open-Source-Nachfolger des internen Systems *Borg*, wurde 2014 veröffentlicht und 2015 in Version 1.0 an die neu gegründete **CNCF** (Cloud Native Computing Foundation) übergeben. Heute ist es der De-facto-Standard; jeder große Cloud-Anbieter hat einen verwalteten Dienst dafür (**EKS** bei AWS, **AKS** bei Azure, **GKE** bei Google). Aktuelle Version: **1.37** (August 2026), drei Minor-Releases pro Jahr.

> [!warning] Die ehrliche Gegenfrage gehört dazu
> Kubernetes ist komplex und für eine einzelne App auf einem Server Overkill — Compose ist dafür eine völlig legitime Betriebsform (so steht es auch in deiner Docker-Vertiefung). Wer im Gespräch sagt „*für kleine Systeme würde ich kein Kubernetes nehmen, der Betriebsaufwand lohnt sich erst ab …*", wirkt reifer als jemand, der K8s für alles empfiehlt.

---

## 2. Die Kernidee: gewünschter Zustand + Regelkreis

Das ist **der** Satz, auf den sich fast jede Konzeptfrage zurückführen lässt:

> **Du sagst Kubernetes nicht, *was es tun* soll, sondern *wie der Zustand aussehen* soll. Kubernetes gleicht den Ist-Zustand dann in einer Endlosschleife an den Soll-Zustand an.**

- **Imperativ** (Docker-CLI-Denke): „Starte drei Container." — Stirbt einer, sind es zwei. Niemand merkt es.
- **Deklarativ** (Kubernetes): „Es sollen drei Instanzen laufen." — Stirbt eine, sieht ein **Controller** die Abweichung (Soll 3, Ist 2) und startet eine neue. Ohne dass du etwas tust.

Diese Schleife heißt **Reconciliation Loop** (Kontrollschleife): *beobachten → mit Soll vergleichen → Differenz beheben → von vorn*. Die Kubernetes-Doku vergleicht das mit einem Thermostat. Jedes Objekt hat deshalb zwei Hälften:

- **`spec`** — was du willst (schreibst du)
- **`status`** — was gerade ist (schreibt Kubernetes)

> [!tip] Der Bogen zu React — das ist Gold fürs Gespräch
> Du hast letzte Woche genau dasselbe Prinzip gelernt: In React beschreibst du, *wie die UI bei diesem State aussehen soll* (`UI = f(state)`), und React **reconciled** den echten DOM dagegen ([[1. React-Grundlagen - Vertiefung#7. Render und Commit — was React eigentlich tut|Render und Commit]]). Kubernetes macht das mit Infrastruktur: Du beschreibst den Soll-Cluster in YAML, Controller gleichen den echten Cluster an. **Beide Systeme sind deklarativ, beide nennen den Abgleich „Reconciliation".** Wer das im Gespräch verbindet, zeigt Verständnis statt Auswendiglernen.

Daraus folgt die zweite wichtige Eigenschaft: **Selbstheilung**. Weil der Controller den Soll-Zustand ständig durchsetzt, ist ein gelöschter oder abgestürzter Pod kein Vorfall, sondern ein Normalfall. Das ist das „Cattle, not Pets"-Prinzip aus deiner Docker-Vertiefung, zu Ende gedacht: Pods sind Wegwerfware, niemand pflegt einen einzelnen.

---

## 3. Architektur: Control Plane und Nodes

Ein Cluster besteht aus zwei Arten von Maschinen:

```
┌──────────────────────── Control Plane (das Gehirn) ───────────────────────┐
│                                                                           │
│   kubectl ──HTTPS──▶  kube-apiserver  ◀──▶  etcd (Key-Value-Speicher)     │
│                        ▲        ▲                                         │
│                        │        │                                         │
│              kube-scheduler   kube-controller-manager                     │
│         („welcher Node?")     („Soll = Ist?" — die Regelkreise)           │
└────────────────────────┼──────────────────────────────────────────────────┘
                         │ (Nodes fragen den API-Server, nicht umgekehrt)
      ┌──────────────────┴────────────┬────────────────────────────┐
      ▼                               ▼                            ▼
┌──── Node 1 ─────┐           ┌──── Node 2 ─────┐           ┌──── Node 3 ─────┐
│ kubelet         │           │ kubelet         │           │ kubelet         │
│ kube-proxy      │           │ kube-proxy      │           │ kube-proxy      │
│ containerd      │           │ containerd      │           │ containerd      │
│ [Pod] [Pod]     │           │ [Pod]           │           │ [Pod] [Pod]     │
└─────────────────┘           └─────────────────┘           └─────────────────┘
```

**Control Plane** — trifft Entscheidungen, speichert den Zustand:

| Komponente | Aufgabe |
|---|---|
| **kube-apiserver** | Die einzige Tür in den Cluster. Eine REST-API — alles (kubectl, Controller, kubelets) spricht nur mit ihr. Einzige Komponente, die direkt auf etcd zugreift |
| **etcd** | Verteilter, konsistenter Key-Value-Speicher. Hier liegt der **gesamte** Cluster-Zustand (alle `spec`s und `status`e). Backup von etcd = Backup des Clusters |
| **kube-scheduler** | Sieht neue Pods ohne Node und entscheidet, wo sie laufen (freie Ressourcen, Regeln wie Affinität). Startet selbst nichts |
| **kube-controller-manager** | Bündelt die eingebauten Controller (Deployment-, ReplicaSet-, Node-, Job-Controller …) — die Regelkreise aus Abschnitt 2 |

**Worker Nodes** — führen die Arbeit aus:

| Komponente | Aufgabe |
|---|---|
| **kubelet** | Agent auf jedem Node. Holt sich vom API-Server „diese Pods gehören zu dir" und sorgt dafür, dass sie laufen; führt die Probes aus, meldet Status zurück |
| **Container-Runtime** | Startet die Container tatsächlich — heute meist **containerd** (oder CRI-O), angesprochen über die Schnittstelle **CRI** |
| **kube-proxy** | Setzt die Service-Netzwerkregeln auf dem Node um (iptables/IPVS), damit die Service-IP beim richtigen Pod ankommt. Wird in manchen Setups durch das Netzwerk-Plugin (z. B. Cilium) ersetzt |

> [!note] Bogen zu 1.7 Docker
> In deiner Docker-Vertiefung steht, dass Kubernetes 2022 `dockershim` entfernt hat und direkt mit **containerd** spricht. Jetzt siehst du, wo das passiert: im kubelet. Das heißt **nicht**, dass Docker-Images nicht mehr gehen — Images sind ein offener Standard (OCI). Du baust weiter mit `docker build`, der Cluster startet sie mit containerd.

**Der Weg eines `kubectl apply`** — die beliebte „Was passiert, wenn …"-Frage:

1. `kubectl` schickt das Manifest per HTTPS an den **API-Server**, der prüft (Authentifizierung, Autorisierung, Validierung) und in **etcd** speichert.
2. Der **Deployment-Controller** sieht ein neues Deployment ohne ReplicaSet → legt eines an. Der **ReplicaSet-Controller** sieht Soll 3/Ist 0 → legt drei Pod-Objekte an (noch ohne Node).
3. Der **Scheduler** sieht Pods ohne Node → trägt jeweils einen Node ein.
4. Das **kubelet** auf diesem Node sieht „neuer Pod für mich" → lässt **containerd** das Image ziehen und die Container starten → meldet den Status zurück.

Beachte: Keine Komponente ruft eine andere direkt auf. Alle **beobachten den API-Server** und reagieren auf Änderungen. Das macht das System robust — fällt ein Teil kurz aus, holt er beim Wiederanlauf einfach nach.

---

## 4. Die Kernobjekte

Alles in Kubernetes ist ein **Objekt** (auch „Ressource") mit derselben Grundform: `apiVersion`, `kind`, `metadata` (Name, Namespace, Labels), `spec`. Die folgenden musst du sicher erklären können.

### 4.1 Pod — die kleinste Einheit

Ein **Pod** ist eine Hülle um **einen oder mehrere Container**, die zusammen auf demselben Node laufen und sich **Netzwerk** (eine IP, sie erreichen sich über `localhost`) und **Volumes** teilen. Meist: ein Container pro Pod. Mehrere nur, wenn sie untrennbar zusammengehören (z. B. App + Log-Shipper als „Sidecar").

- Kubernetes plant **Pods**, nicht Container.
- Pods sind **sterblich und austauschbar**: Ein neuer Pod bekommt eine neue IP und einen neuen Namen. Deshalb legt man Pods fast nie direkt an, sondern über einen Controller (→ Deployment) und spricht sie über einen Service an (→ 4.3).

### 4.2 Deployment (und ReplicaSet) — für zustandslose Apps

Ein **Deployment** sagt: „Von *diesem* Pod-Template sollen *N* Kopien laufen." Es ist das Standardobjekt für zustandslose Dienste wie deine Pricing-API.

- Darunter liegt ein **ReplicaSet**, das die Anzahl hält. Das Deployment verwaltet ReplicaSets — bei jeder Änderung am Pod-Template (z. B. neues Image) legt es ein **neues** ReplicaSet an und fährt das alte herunter.
- **Rolling Update**: schrittweise neue Pods hoch, alte runter (Standard: max. 25 % zusätzlich / max. 25 % gleichzeitig nicht verfügbar). Kein Ausfall, sofern Readiness-Probes stimmen.
- **Rollback**: Die alten ReplicaSets bleiben liegen → `kubectl rollout undo deployment/<name>`.
- **Skalieren**: `replicas` ändern oder `kubectl scale`.

### 4.3 Service — stabile Adresse vor wechselnden Pods

Pods kommen und gehen mit neuen IPs. Ein **Service** gibt einer Gruppe von Pods eine **stabile virtuelle IP und einen DNS-Namen** und verteilt die Last auf sie. *Welche* Pods dazugehören, bestimmt ein **Label-Selector** (→ 4.7).

Innerhalb des Clusters erreichst du einen Service unter seinem Namen: `db` im selben Namespace, voll ausgeschrieben `db.<namespace>.svc.cluster.local`. Das ersetzt die Compose-Service-Namen eins zu eins.

| Typ | Erreichbar von | Wofür |
|---|---|---|
| **ClusterIP** (Standard) | nur innerhalb des Clusters | interne Dienste, z. B. die Datenbank |
| **NodePort** | Port 30000–32767 auf **jedem** Node | einfacher Zugang von außen, v. a. zum Testen |
| **LoadBalancer** | externe IP vom Cloud-Anbieter | Produktion in der Cloud |

Für HTTP-Routing von außen (Hostnamen, Pfade, TLS) kommt davor eine weitere Schicht: früher **Ingress**, heute die **Gateway API** (→ Abschnitt 8).

### 4.4 ConfigMap und Secret — Konfiguration raus aus dem Image

Genau das Prinzip aus deiner [[Documentation/Traineeship/Python Basics/Konfiguration - Umgebungsvariablen, .env und CLI-Flags|Konfigurations-Notiz]]: dasselbe Image, unterschiedliche Konfiguration pro Umgebung.

- **ConfigMap**: nicht geheime Key-Value-Paare (`LOG_LEVEL`, `DB_HOST`) — als Umgebungsvariablen oder als Datei in den Container eingeblendet.
- **Secret**: dasselbe für Passwörter/Tokens.

> [!warning] Klassiker-Falle: Secrets sind nicht verschlüsselt
> Die Werte in einem Secret sind nur **base64-kodiert** — das ist Kodierung, keine Verschlüsselung. Sicherheit kommt erst durch Zugriffsrechte (RBAC), optional Verschlüsselung in etcd („encryption at rest") oder externe Secret-Stores (Vault, Cloud-KMS). Deshalb gehören Secret-Manifeste mit Klartextwerten auch **nicht ins Git**.

### 4.5 Volumes, PersistentVolumeClaim, StatefulSet — wenn Daten bleiben müssen

Ein Container-Dateisystem ist flüchtig (kennst du aus Docker). Für dauerhafte Daten:

- **PersistentVolume (PV)**: ein Stück echter Speicher (Cloud-Disk, NFS, lokaler Pfad).
- **PersistentVolumeClaim (PVC)**: die *Anforderung* eines Pods: „Ich brauche 1 GiB, lesen/schreiben." Kubernetes bindet einen passenden PV daran — meist wird er über eine **StorageClass** automatisch angelegt („dynamic provisioning").
- Die Trennung ist gewollt: Der Entwickler sagt, *was* er braucht (PVC); *woher* es kommt (PV/StorageClass), entscheidet der Cluster-Betreiber.

Für **zustandsbehaftete** Anwendungen wie Datenbanken gibt es statt des Deployments das **StatefulSet**: stabile Pod-Namen (`db-0`, `db-1`), geordnetes Starten/Stoppen und **pro Pod ein eigenes Volume**, das beim Neustart an denselben Pod zurückkehrt.

> [!note] Praxis-Einordnung
> Viele Teams betreiben Datenbanken **nicht** in Kubernetes, sondern nehmen einen verwalteten Dienst (RDS, Azure Database, Cloud SQL) und lassen nur die zustandslosen Apps im Cluster laufen. Eine gute, ehrliche Antwort auf „Würden Sie Postgres in K8s betreiben?".

### 4.6 Job und CronJob — einmalige und wiederkehrende Arbeit

- **Job**: startet Pods, bis eine Aufgabe **erfolgreich abgeschlossen** ist (mit Wiederholungen bei Fehlschlag). Passt genau auf deinen `migrate`-Service aus `compose.yml`.
- **CronJob**: legt nach Zeitplan (Cron-Syntax) Jobs an — z. B. ein nächtlicher ETL-Lauf.

### 4.7 Labels, Selectors, Namespaces — der Klebstoff

- **Labels** sind frei wählbare Key-Value-Etiketten an Objekten (`app: pricing-api`, `tier: backend`).
- **Selectors** wählen Objekte über Labels aus. **So hängt alles zusammen:** Ein Deployment findet „seine" Pods über einen Selector, ein Service findet seine Ziel-Pods über einen Selector. Es gibt keine festen Verweise auf Pod-Namen — nur Label-Abfragen. Stimmt ein Label nicht, hat der Service schlicht keine Ziele (häufiger Anfängerfehler).
- **Namespaces** teilen einen Cluster logisch auf (pro Team, pro Umgebung, pro App). Namen müssen nur innerhalb eines Namespaces eindeutig sein; Rechte und Ressourcen-Quoten lassen sich pro Namespace vergeben.

### 4.8 Probes und Ressourcen — was eine Produktions-Pod-Definition ausmacht

**Probes** (vom kubelet ausgeführt, meist ein HTTP-GET auf einen Health-Endpunkt):

| Probe | Frage | Folge bei Fehlschlag |
|---|---|---|
| **readinessProbe** | „Kann der Pod gerade Anfragen bedienen?" | fliegt aus dem Service (bekommt keinen Traffic), wird **nicht** neu gestartet |
| **livenessProbe** | „Lebt der Prozess noch sinnvoll?" | Container wird **neu gestartet** |
| **startupProbe** | „Ist der langsame Start schon durch?" | hält die anderen Probes zurück, bis der Start geklappt hat |

> [!tip] Wichtiges Detail für deine Übung
> Kubernetes **ignoriert** die `HEALTHCHECK`-Anweisung aus deinem Dockerfile. Den Health-Check `/v1/health` musst du im Manifest als Probe neu angeben.

**Ressourcen** pro Container:
- **requests**: garantierte Menge — danach entscheidet der **Scheduler**, auf welchen Node der Pod passt.
- **limits**: Obergrenze — bei CPU wird gedrosselt, bei Speicher wird der Container **beendet** (`OOMKilled`).

---

## 5. Compose → Kubernetes: die Übersetzungstabelle

Da du `compose.yml` kennst, ist das dein schnellster Weg ins Modell — und gleichzeitig die Vorlage für die [[Pricing-API auf Kubernetes - Übung|Übung]]:

| In deiner `compose.yml` | In Kubernetes |
|---|---|
| `services: api:` (zustandslos) | **Deployment** + **Service** |
| `services: db:` (mit Daten) | **StatefulSet** (oder Deployment mit 1 Replica) + **PVC** + **Service** |
| `services: migrate:` mit `restart: "no"` | **Job** (alternativ: `initContainer` im API-Pod) |
| Service-Name `db` als Hostname | **Service**-Name `db` → gleicher DNS-Name |
| `environment:` | **ConfigMap** (+ **Secret** für Passwörter) |
| `.env` mit `DB_PRICING_PW` | **Secret** |
| `volumes: pricing_data:` | **PersistentVolumeClaim** |
| `ports: "127.0.0.1:8000:8000"` | **Service** (NodePort/LoadBalancer) oder lokal `kubectl port-forward` |
| `healthcheck:` / Dockerfile-`HEALTHCHECK` | **readiness-/livenessProbe** |
| `restart: unless-stopped` | Standardverhalten: Pods werden neu gestartet, Controller ersetzen sie |
| `depends_on: condition: service_healthy` | **gibt es nicht.** Stattdessen: Apps müssen Wiederholungen aushalten; Readiness-Probes, Job-Retries, Init-Container |
| `build:` | **gibt es nicht.** Kubernetes baut nie — Images kommen fertig aus einer Registry |
| `profiles: ["dev"]` | am ehesten eigene Namespaces oder Overlays (Kustomize/Helm-Values) |

> [!important] Die zwei Zeilen „gibt es nicht" sind die interessantesten
> - **Kein `depends_on`**: In einem verteilten System kann jede Abhängigkeit jederzeit kurz weg sein. Kubernetes erzwingt deshalb, dass Anwendungen das aushalten (neu versuchen, bis die DB da ist), statt eine Startreihenfolge vorzutäuschen.
> - **Kein `build`**: Die Trennung zwischen *bauen* (CI-Pipeline → Image in Registry) und *betreiben* (Cluster zieht das Image) ist hart.
>
> Übrigens gibt es mit **Kompose** ein Werkzeug, das `compose.yml` automatisch übersetzt — für die Übung bewusst **nicht** benutzen.

---

## 6. Wie man es bedient: kubectl und Manifeste

### 6.1 Das Werkzeug

**`kubectl`** ist der Kommandozeilen-Client. Er spricht mit dem API-Server — wie die `docker`-CLI mit dem Docker-Daemon (auch dort war es eine REST-API, siehe 1.7). Zu welchem Cluster er spricht, steht in `~/.kube/config` (**Kontext** = Cluster + Benutzer + Namespace).

Zum Üben lokal: **kind** („Kubernetes in Docker" — jeder Node ist ein Docker-Container; Version 0.33), alternativ **minikube** oder **k3d**. Docker Desktop hat auch einen eingebauten Cluster.

### 6.2 Aufbau eines Manifests

```yaml
apiVersion: apps/v1          # API-Gruppe/Version des Objekttyps
kind: Deployment             # Objekttyp
metadata:
  name: pricing-api          # Name (eindeutig im Namespace)
  labels:
    app: pricing-api
spec:                        # SOLL-Zustand
  replicas: 2
  selector:
    matchLabels:
      app: pricing-api       # „meine Pods sind die mit diesem Label" …
  template:                  # … und so sieht jeder dieser Pods aus:
    metadata:
      labels:
        app: pricing-api     # muss zum Selector passen!
    spec:
      containers:
        - name: api
          image: pricing-api:0.1
          ports:
            - containerPort: 8000
```

Mehrere Objekte in einer Datei werden durch `---` getrennt. Bei Syntaxfragen fragst du den Cluster selbst: `kubectl explain deployment.spec.template.spec.containers.readinessProbe`.

### 6.3 Imperativ vs. deklarativ bedienen

- **Imperativ**: `kubectl create deployment …`, `kubectl scale …`, `kubectl delete pod …` — schnell zum Ausprobieren und Debuggen.
- **Deklarativ**: YAML-Dateien im Git, `kubectl apply -f ordner/` — Kubernetes rechnet selbst aus, was sich geändert hat. **So wird in der Praxis gearbeitet** (und mit GitOps sogar automatisiert, → Abschnitt 8).

Trick, um sich ein YAML-Gerüst erzeugen zu lassen statt es abzutippen:

```bash
kubectl create deployment pricing-api --image=pricing-api:0.1 --dry-run=client -o yaml > api-deployment.yaml
```

### 6.4 Die Befehle, die du kennen musst

```bash
# Überblick
kubectl get nodes
kubectl get pods                     # -o wide: mit Node + IP;  -w: live beobachten
kubectl get deploy,svc,pods -n pricing
kubectl get all -n pricing           # (zeigt nicht wirklich „alles", aber das Wichtigste)

# Anwenden / Entfernen
kubectl apply -f k8s/                # ganzen Ordner anwenden
kubectl delete -f k8s/

# Namespace
kubectl create namespace pricing
kubectl config set-context --current --namespace=pricing   # Standard-Namespace setzen

# Skalieren und Ausrollen
kubectl scale deployment/pricing-api --replicas=3
kubectl set image deployment/pricing-api api=pricing-api:0.2
kubectl rollout status deployment/pricing-api
kubectl rollout history deployment/pricing-api
kubectl rollout undo deployment/pricing-api

# Zugriff von deinem Rechner
kubectl port-forward svc/pricing-api 8000:8000
```

---

## 7. Debuggen: die vier Befehle und die typischen Fehlerbilder

Wenn etwas nicht läuft, immer in dieser Reihenfolge:

```bash
kubectl get pods                         # 1. Welcher Status?
kubectl describe pod <name>              # 2. Warum? → ganz unten „Events" lesen!
kubectl logs <name>                      # 3. Was sagt die App?  (--previous: Log des abgestürzten Vorgängers)
kubectl exec -it <name> -- sh            # 4. Reinschauen (falls der Container läuft)
kubectl get events --sort-by=.metadata.creationTimestamp   # Bonus: alles, was zuletzt passiert ist
```

| Status | Bedeutet | Typische Ursache |
|---|---|---|
| `Pending` | Pod hat keinen Node / wartet | zu wenig Ressourcen für die `requests`, PVC nicht gebunden |
| `ErrImagePull` / `ImagePullBackOff` | Image nicht ladbar | Tippfehler im Namen/Tag, private Registry ohne Zugangsdaten, **lokales Image nicht in kind geladen** |
| `CrashLoopBackOff` | Container startet, stirbt, wird neu gestartet — mit wachsender Pause | App-Fehler beim Start (z. B. DB nicht erreichbar, Env-Variable fehlt) → `kubectl logs --previous` |
| `Running`, aber `0/1 READY` | läuft, besteht die Readiness-Probe nicht | falscher Probe-Pfad/-Port, App noch nicht bereit |
| `OOMKilled` (in `describe`) | Speicherlimit überschritten | `limits.memory` zu knapp |
| Service antwortet nicht | keine Ziel-Pods | Selector passt nicht zu den Pod-Labels → `kubectl get endpointslices` prüfen |

---

## 8. Das Ökosystem: Vokabeln

Diese Begriffe sollst du **einordnen**, nicht beherrschen. Je zwei Sätze reichen im Gespräch.

| Begriff | Was es ist |
|---|---|
| **Helm** | „Paketmanager" für Kubernetes: Vorlagen („Charts") für ganze Anwendungen mit Parametern (`values.yaml`). `helm install postgres …` statt zehn YAML-Dateien selbst zu schreiben |
| **Kustomize** | In `kubectl` eingebaut (`kubectl apply -k`): Basis-YAML plus Überlagerungen pro Umgebung (dev/prod), ganz ohne Template-Sprache |
| **Ingress / Gateway API** | HTTP-Routing von außen (Host, Pfad, TLS). Die Ingress-API ist eingefroren; Nachfolger ist die **Gateway API**. Der verbreitete Controller *ingress-nginx* wurde im März 2026 eingestellt — eine gute aktuelle Bemerkung |
| **HPA** (Horizontal Pod Autoscaler) | Passt `replicas` automatisch an Last an (z. B. CPU-Auslastung) |
| **Operator / CRD** | Eigene Objekttypen (Custom Resource Definitions) plus eigener Controller — so bringt man Kubernetes bei, z. B. „eine Postgres-Datenbank mit Backups" als ein Objekt zu verwalten |
| **RBAC** | Rollenbasierte Rechte: wer darf welche Aktion auf welchen Objekten in welchem Namespace |
| **GitOps (Argo CD, Flux)** | Das Git-Repo ist die Wahrheit; ein Controller im Cluster gleicht den Cluster laufend an das Repo an — Reconciliation, eine Ebene höher |
| **Service Mesh (Istio, Linkerd)** | Zusatzschicht für Service-zu-Service-Kommunikation: mTLS, Retries, Metriken, ohne Code-Änderung |
| **EKS / AKS / GKE** | Verwaltetes Kubernetes der großen Clouds: Anbieter betreibt die Control Plane, du die Workloads |

> [!note] Falls es um eine Daten-Rolle geht
> Kubernetes taucht im Data Engineering vor allem als **Ausführungsplattform** auf: Spark kann Executors als Pods starten, Airflow kann jede Task in einem eigenen Pod ausführen (`KubernetesExecutor`/`KubernetesPodOperator`), und nächtliche ETL-Läufe sind klassische **CronJobs**. Das reicht als Einordnung.

---

## 9. Fürs Gespräch: Kontrollfragen

Erst **laut und aus dem Kopf** beantworten, dann nachschlagen — wie beim [[Interview-Fragenkatalog]] für React. Stichpunkte direkt unter die Frage schreiben, Claudian reviewt.

**Grundverständnis**
- [ ] Was ist Kubernetes und welches Problem löst es, das Docker/Compose nicht löst?
- [ ] Was heißt „deklarativ" bei Kubernetes, und was ist die Reconciliation Loop? Wo ist die Parallele zu React?
- [ ] Was passiert, wenn ein Node ausfällt, auf dem zwei Pods eines Deployments liefen?

**Architektur**
- [ ] Nenne die Komponenten der Control Plane und eines Nodes mit je einem Satz.
- [ ] Was passiert zwischen `kubectl apply -f deployment.yaml` und dem laufenden Container?
- [ ] Warum ist etcd die kritischste Komponente?

**Objekte**
- [ ] Pod vs. Container vs. Deployment — wie hängen sie zusammen?
- [ ] Warum spricht man Pods über einen Service an statt direkt über ihre IP?
- [ ] ClusterIP, NodePort, LoadBalancer — wann welches?
- [ ] Deployment vs. StatefulSet — wann welches?
- [ ] ConfigMap vs. Secret — und wie sicher ist ein Secret wirklich?
- [ ] Liveness- vs. Readiness-Probe — was passiert jeweils bei Fehlschlag?
- [ ] requests vs. limits?

**Bedienung**
- [ ] Wie rollst du eine neue Version ohne Ausfall aus, und wie gehst du zurück?
- [ ] Ein Pod steht auf `CrashLoopBackOff` — wie gehst du vor?
- [ ] Wie würdest du eine bestehende `compose.yml` nach Kubernetes bringen? (→ deine Übung)

**Einordnung**
- [ ] Wann würdest du Kubernetes **nicht** einsetzen?
- [ ] Was ist Helm, und warum benutzt man es?

---

## 10. Offene Fragen

*Alles, was beim Lesen unklar bleibt, sofort hier notieren — Claudian beantwortet gesammelt.*

-
