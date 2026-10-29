---
tags:
  - documentation
---

# Pricing-API auf Kubernetes — Übung

Kleine Übung zu [[Kubernetes Crashkurs - Überblick]]. Zeitrahmen **~2–3 h**. Du bringst deine eigene Pricing-API aus [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.7 Docker/Übungsaufgabe/Umsetzungsstand (Docker)|Kapitel 1.7 Docker]] von `compose.yml` auf einen lokalen Kubernetes-Cluster. Danach hast du im Gespräch eine echte Geschichte: *„Ich habe meine Compose-Anwendung nach Kubernetes übertragen, und die interessanten Stellen waren …"*

> [!important] Arbeitsmodus wie immer
> **Du schreibst die Manifeste selbst**, Claudian reviewt und erklärt. Die Code-Muster unten sind bewusst **generisch** (Platzhalter `<…>`) — die Übersetzung deiner `compose.yml` ist die eigentliche Übung. Nutze die [[Kubernetes Crashkurs - Überblick#5. Compose → Kubernetes: die Übersetzungstabelle|Übersetzungstabelle]] und `kubectl explain`.

| | |
|---|---|
| Projektordner (Vorschlag) | `~/HaegerConsulting/Projects/Interviews/KubernetesBasics/` — darin ein Ordner `k8s/` für die Manifeste |
| Quelle | `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Docker` (`Dockerfile`, `compose.yml`) |
| Cluster | kind 0.33 (Kubernetes 1.37), ein Node |
| Bewusst weggelassen | `adminer` (dev-Profil), das `daily_data`-Volume (Ingest), Ingress/Gateway, Helm |

## Ziel-Architektur

```
Namespace „pricing"
│
├── Secret      pricing-db          (Passwort — imperativ angelegt, nicht im Git)
├── ConfigMap   pricing-config      (DB_HOST, DB_NAME, DB_USER, LOG_LEVEL)
│
├── PVC         pricing-data  ◀──── Deployment db (postgres:18, 1 Replica)
├── Service     db  ─────────────▶  (ClusterIP, Port 5432)
│
├── Job         migrate             (alembic upgrade head, läuft einmal durch)
│
├── Deployment  pricing-api         (2 Replicas, Probes auf /v1/health)
└── Service     pricing-api  ────▶  (ClusterIP, Port 8000)  ◀── kubectl port-forward
```

---

## Schritte

### Schritt 0 — Werkzeuge und Cluster (~20 min)

- [ ] `kubectl` und `kind` installieren (Anleitung: [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)). Docker muss laufen — kind startet die Nodes als Docker-Container.
- [ ] Cluster anlegen und prüfen:
  ```bash
  kind create cluster --name pricing
  kubectl cluster-info --context kind-pricing
  kubectl get nodes          # ein Node, STATUS Ready
  docker ps                  # Überraschung: der „Node" ist ein Container
  ```
- [ ] Image bauen und **in den Cluster laden**:
  ```bash
  docker build -t pricing-api:0.1 ~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Docker
  kind load docker-image pricing-api:0.1 --name pricing
  ```

> [!warning] Zwei Stolperfallen, bevor du anfängst
> 1. **Der Cluster sieht deine lokalen Docker-Images nicht.** Er hat seine eigene containerd-Instanz. Ohne `kind load` endet jeder Pod in `ImagePullBackOff`.
> 2. **Nimm nicht den Tag `:latest`.** Bei `:latest` versucht Kubernetes standardmäßig *immer*, das Image aus einer Registry zu ziehen (`imagePullPolicy: Always`) — und scheitert, weil `pricing-api` in keiner Registry liegt. Mit einem festen Tag wie `0.1` gilt `IfNotPresent`. (In der Praxis sind feste Tags ohnehin Pflicht: Was genau ist „latest" beim Rollback?)

### Schritt 1 — Namespace, Secret, ConfigMap (~15 min)

**Neue Technik:** Konfiguration als eigene Objekte.

- [ ] Namespace `pricing` anlegen und als Standard setzen (Befehle: [[Kubernetes Crashkurs - Überblick#6.4 Die Befehle, die du kennen musst|Überblick 6.4]]).
- [ ] Secret **imperativ** anlegen, damit das Passwort in keiner Datei landet:
  ```bash
  kubectl create secret generic <name> --from-literal=<KEY>=<wert> [--from-literal=…]
  ```
  *Überleg: Welche Variablen braucht der Postgres-Container, welche die API — und welche davon sind geheim? Tipp: Das DB-Passwort brauchen beide, nur unter verschiedenen Namen.*
- [ ] ConfigMap `k8s/config.yaml` für die nicht geheimen Werte. Muster:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: <name>
  data:
    <KEY>: "<wert>"
  ```
- [ ] Prüfen: `kubectl get secret,configmap` und `kubectl get secret <name> -o yaml` — **schau dir an, wie das Passwort dort steht, und dekodiere es mit `base64 -d`.** Genau das ist der Punkt aus [[Kubernetes Crashkurs - Überblick#4.4 ConfigMap und Secret — Konfiguration raus aus dem Image|Abschnitt 4.4]].

### Schritt 2 — Die Datenbank (~30 min)

**Neue Techniken:** PersistentVolumeClaim, Deployment, Service, Umgebungsvariablen aus Secret.

- [ ] `k8s/db.yaml` mit drei Objekten (durch `---` getrennt): **PVC**, **Deployment** (1 Replica, `postgres:18`), **Service** namens `db`.
- Muster PVC (kind bringt eine StorageClass `standard` mit, die den Speicher automatisch anlegt):
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: <name>
  spec:
    accessModes: ["ReadWriteOnce"]
    resources:
      requests:
        storage: 1Gi
  ```
- Muster: Umgebungsvariable aus einem Secret-Key, Volume einhängen (im Container-Teil des Pod-Templates):
  ```yaml
      containers:
        - name: <name>
          image: <image>
          env:
            - name: <VARIABLE_IM_CONTAINER>
              valueFrom:
                secretKeyRef:
                  name: <secret-name>
                  key: <KEY_IM_SECRET>
          volumeMounts:
            - name: data
              mountPath: <pfad-wie-in-compose>
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: <pvc-name>
  ```
- Muster Service:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: <name>             # wird zum DNS-Namen!
  spec:
    selector:
      app: <label-der-pods>
    ports:
      - port: <port>
        targetPort: <containerPort>
  ```
- [ ] Anwenden, Pod beobachten bis `Running`, dann testen:
  ```bash
  kubectl exec -it deploy/db -- psql -U pricing -d pricing -c 'select 1'
  ```
- [ ] **Frage für dein Log:** Warum muss der Service genau `db` heißen, damit die API ohne Code-Änderung funktioniert?

> [!note] Deployment statt StatefulSet — bewusste Vereinfachung
> Für eine Datenbank wäre ein **StatefulSet** das richtige Objekt ([[Kubernetes Crashkurs - Überblick#4.5 Volumes, PersistentVolumeClaim, StatefulSet — wenn Daten bleiben müssen|Abschnitt 4.5]]). Mit einer einzigen Replica funktioniert ein Deployment + PVC aber genauso und ist ein Objekt weniger zu lernen. Umbau ist Bonusaufgabe B1 — und die Begründung der Vereinfachung ist gute Gesprächsmunition.

### Schritt 3 — Migration als Job (~15 min)

**Neue Technik:** Job, Konfiguration per `envFrom`.

- [ ] `k8s/migrate.yaml`: ein **Job**, der dein Image mit dem Befehl `alembic upgrade head` startet. Muster:
  ```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: <name>
  spec:
    backoffLimit: 6
    template:
      spec:
        restartPolicy: Never     # Pflicht bei Jobs: Never oder OnFailure
        containers:
          - name: <name>
            image: <image:tag>
            command: [<wie in compose>]
            envFrom:
              - configMapRef:
                  name: <configmap>
            env:
              - …                # das Passwort wie in Schritt 2
  ```
- [ ] `kubectl get jobs` → `COMPLETIONS 1/1`. `kubectl logs job/<name>` zeigt die Alembic-Ausgabe.
- [ ] **Frage für dein Log:** In Compose sorgte `depends_on: service_healthy` dafür, dass die Migration erst nach der DB startet. Hier gibt es das nicht — warum klappt es trotzdem (oder nach ein paar Fehlversuchen)? Sieh dir `kubectl get pods` an, wenn du Job und DB gleichzeitig anwendest.

### Schritt 4 — Die API (~30 min)

**Neue Techniken:** Replicas, Probes, Ressourcen.

- [ ] `k8s/api.yaml`: **Deployment** `pricing-api` mit **2 Replicas** + **Service** `pricing-api` auf Port 8000. Gerüst erzeugen statt abtippen:
  ```bash
  kubectl create deployment pricing-api --image=pricing-api:0.1 --replicas=2 --port=8000 --dry-run=client -o yaml
  ```
- [ ] Konfiguration wie im Job einbinden.
- [ ] **readinessProbe** und **livenessProbe** auf deinen Health-Endpunkt. Muster:
  ```yaml
          readinessProbe:
            httpGet:
              path: <pfad>
              port: <port>
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: <pfad>
              port: <port>
            initialDelaySeconds: 10
            periodSeconds: 10
  ```
  *Denk dran: Kubernetes ignoriert das `HEALTHCHECK` aus deinem Dockerfile.*
- [ ] **resources** mit kleinen `requests` und `limits` (z. B. 100m CPU / 128Mi Speicher als Request) — Syntax: `kubectl explain deployment.spec.template.spec.containers.resources`.
- [ ] Testen:
  ```bash
  kubectl port-forward svc/pricing-api 8000:8000
  curl http://127.0.0.1:8000/v1/health
  ```
- [ ] Optional: API-Schlüssel direkt im Pod anlegen und einen geschützten Endpunkt aufrufen — `kubectl exec -it deploy/pricing-api -- pricing-apikey create nick`.

### Schritt 5 — Die eigentliche Lektion: kaputt machen und zuschauen (~30 min)

Hier lernst du, was Kubernetes von Compose unterscheidet. **Pro Experiment: erst Vorhersage aufschreiben, dann ausführen.** Ein zweites Terminal mit `kubectl get pods -w` offen lassen.

- [ ] **Selbstheilung:** `kubectl delete pod <einer-der-api-pods>` — was passiert, und welcher Controller war das?
- [ ] **Skalieren:** auf 4 Replicas hochskalieren, dann wieder auf 2 — einmal imperativ (`kubectl scale`), einmal deklarativ (YAML ändern + `apply`). Was ist der Unterschied, wenn du danach das YAML erneut anwendest?
- [ ] **Rolling Update:** Image als `pricing-api:0.2` neu taggen (`docker tag`), in kind laden, im YAML ändern, anwenden. `kubectl rollout status` und `kubectl get rs` beobachten — wie viele ReplicaSets gibt es jetzt?
- [ ] **Kaputtes Update + Rollback:** `kubectl set image deployment/pricing-api <container>=pricing-api:gibtsnicht`. Welcher Status? Laufen die alten Pods weiter — warum? Dann `kubectl rollout undo`.
- [ ] **Datenbank-Neustart:** DB-Pod löschen. Sind die Daten (z. B. die Alembic-Version) nach dem Neustart noch da? Warum?
- [ ] **Label-Fehler:** im API-Service den Selector auf ein falsches Label ändern. Was sagt `curl`, was `kubectl get endpointslices`? Zurückdrehen.

### Schritt 6 — Aufräumen und festhalten (~10 min)

- [ ] `kind delete cluster --name pricing`
- [ ] Unten die **Lessons Learned** ausfüllen — daraus wird deine Gesprächsgeschichte.

---

## Bonus (nur mit Restzeit)

- [ ] **B1 — StatefulSet:** die DB auf ein StatefulSet mit `volumeClaimTemplates` und Headless-Service umbauen.
- [ ] **B2 — Init-Container statt Job:** die Migration als `initContainer` im API-Deployment. Vor- und Nachteile gegenüber dem Job? (Stichwort: Was passiert bei 2 Replicas?)
- [ ] **B3 — Kustomize:** `k8s/kustomization.yaml` anlegen und alles mit `kubectl apply -k k8s/` ausrollen.

---

## Stand und Lessons Learned

*Diese Abschnitte führst du. Claudian ergänzt nur beim Review.*

| Schritt | Status | Notiz |
|---|---|---|
| 0 — Cluster + Image | offen | |
| 1 — Namespace, Secret, ConfigMap | offen | |
| 2 — Datenbank | offen | |
| 3 — Migration als Job | offen | |
| 4 — API | offen | |
| 5 — Experimente | offen | |

**Antworten auf die Log-Fragen:**

-

**Was mich überrascht hat / was schiefging:**

-

**Meine 3-Minuten-Geschichte fürs Gespräch** (Stichpunkte: Ausgangslage → Übersetzung → eine Stolperfalle → was ich daraus über Kubernetes verstanden habe):

-
