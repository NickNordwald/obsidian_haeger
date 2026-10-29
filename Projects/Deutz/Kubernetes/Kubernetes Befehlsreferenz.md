---
tags:
  - documentation
---
# Kubernetes Befehlsreferenz

Nachschlagewerk zum [[Projects/Deutz/Kubernetes/Kubernetes|Kubernetes-Lernpfad]], analog zur [[Git Befehlsreferenz]]. Hier stehen erst einmal die Standardbefehle. Die Referenz wächst mit den Blöcken: Was du beim Üben wirklich gebraucht hast, kommt dazu, und Stolperfallen werden ergänzt.

> [!tip] Drei Gewohnheiten, die viel Zeit sparen
> - `alias k=kubectl`, ist in den Killercoda-Umgebungen meist schon gesetzt
> - YAML **generieren statt tippen**: `k create … --dry-run=client -o yaml > x.yaml`
> - Felder nachschlagen: `k explain cronjob.spec.jobTemplate --recursive | less`

---

## minikube (lokaler Cluster)

minikube startet einen Kubernetes-Cluster mit einem Node **als Docker-Container** auf deinem Rechner. Eingerichtet für Debian 13 (x86_64) mit dem Docker-Treiber.

> [!example]- Installation (einmalig, im eigenen Terminal, nicht in Obsidian)
> ```bash
> # 0. Voraussetzung: Docker läuft OHNE sudo
> docker run --rm hello-world
>
> # 1. minikube
> curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
> sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
>
> # 2. kubectl (aktuelle stabile Version, mit Prüfsumme)
> curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
> curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
> echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check     # muss "kubectl: OK" ausgeben
> sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && rm kubectl kubectl.sha256
>
> # 3. Docker als Standard-Treiber und erster Start
> minikube config set driver docker
> minikube start
> kubectl get nodes        # ein Node "minikube", STATUS Ready
>
> # 4. Komfort (bash): Autovervollständigung und Alias k
> echo 'source <(kubectl completion bash)' >> ~/.bashrc
> echo 'alias k=kubectl' >> ~/.bashrc
> echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
> source ~/.bashrc
> ```

| Befehl | Was er tut |
|---|---|
| `minikube start` / `minikube stop` | Cluster starten / anhalten, Objekte bleiben erhalten |
| `minikube status` | Läuft der Cluster? |
| `minikube delete` | Cluster komplett wegwerfen, für einen sauberen Neuanfang |
| `minikube ip` | IP des Nodes, für NodePort-Services |
| `minikube service <svc> --url` | URL eines NodePort-Services ausgeben |
| `minikube dashboard` | Web-Oberfläche öffnen |
| `minikube image load <image:tag>` | Lokal gebautes Docker-Image in den Cluster laden, sonst kommt es zu `ImagePullBackOff` |
| `minikube addons list` / `minikube addons enable metrics-server` | Zusatzfunktionen, z. B. für `kubectl top` |

---

## kubectl: Überblick & Kontext

| Befehl | Was er tut |
|---|---|
| `kubectl get pods -A` | Pods in allen Namespaces |
| `kubectl get all -n <ns>` | Die wichtigsten Objekte eines Namespaces |
| `kubectl get pod <name> -o yaml` | Vollständiges Objekt, `spec` und `status` |
| `kubectl get pods -o wide --show-labels` | Mit Node, IP und Labels |
| `kubectl describe <typ> <name>` | Details plus **Events** am Ende, die erste Anlaufstelle beim Debuggen |
| `kubectl explain <typ>.<feld>` | Doku zu einem Feld direkt im Terminal |
| `kubectl config get-contexts` | Verfügbare Cluster/Kontexte |
| `kubectl config use-context <ctx>` | Cluster wechseln |
| `kubectl config set-context --current --namespace=<ns>` | Standard-Namespace setzen |

## kubectl: Anlegen & Ändern

| Befehl | Was er tut |
|---|---|
| `kubectl apply -f <datei-oder-ordner>` | Deklarativ anlegen oder aktualisieren |
| `kubectl delete -f <datei>` | Alles aus der Datei löschen |
| `kubectl create deployment web --image=nginx:1.27 --replicas=3` | Deployment anlegen |
| `kubectl expose deployment web --port=80 --target-port=80` | Service (ClusterIP) vor das Deployment setzen |
| `kubectl create configmap app-cfg --from-literal=KEY=wert` | ConfigMap anlegen |
| `kubectl create secret generic app-sec --from-literal=PW=geheim` | Secret anlegen, nur base64, **nicht** verschlüsselt |
| `kubectl run tmp --image=busybox --rm -it --restart=Never -- sh` | Wegwerf-Pod zum Testen von innen |
| `kubectl edit <typ> <name>` | Live im Editor ändern. Unter Argo CD wird das wieder überschrieben! |

## kubectl: Deployments ausrollen

| Befehl | Was er tut |
|---|---|
| `kubectl scale deployment web --replicas=5` | Skalieren |
| `kubectl set image deployment/web nginx=nginx:1.28` | Image wechseln, löst einen Rollout aus |
| `kubectl rollout status deployment/web` | Fortschritt des Rollouts verfolgen |
| `kubectl rollout history deployment/web` | Revisionen |
| `kubectl rollout undo deployment/web` | Auf die vorige Revision zurück |
| `kubectl rollout restart deployment/web` | Alle Pods neu starten, z. B. nach einer ConfigMap-Änderung |

## kubectl: Jobs & CronJobs ⭐

| Befehl | Was er tut |
|---|---|
| `kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" -- date` | CronJob anlegen |
| `kubectl get cronjobs` / `kubectl get jobs` | Zeitplan, letzter Lauf, Job-Historie |
| `kubectl create job hello-manuell --from=cronjob/hello` | CronJob **sofort einmal** auslösen |
| `kubectl logs job/<job-name>` | Ausgabe eines Job-Laufs |
| `kubectl patch cronjob hello -p '{"spec":{"suspend":true}}'` | CronJob pausieren |

Cron-Format: `Minute Stunde Tag Monat Wochentag`. Beispiel: `0 2 * * *` heißt täglich um 02:00 Uhr.

## kubectl: Debuggen

| Befehl | Was er tut |
|---|---|
| `kubectl get pods` | 1. Welcher Status? (`Pending`, `CrashLoopBackOff`, `ImagePullBackOff` …) |
| `kubectl describe pod <name>` | 2. Events lesen: Warum? |
| `kubectl logs <pod> [-c <container>] [-f]` | 3. Was sagt die App? |
| `kubectl logs <pod> --previous` | Logs des **abgestürzten** Vorgänger-Containers |
| `kubectl exec -it <pod> -- sh` | 4. Von innen nachsehen |
| `kubectl get events --sort-by=.metadata.creationTimestamp` | Alle Events chronologisch |
| `kubectl port-forward svc/web 8080:80` | Service lokal unter `localhost:8080` erreichbar machen |
| `kubectl get endpoints web` | Hat der Service überhaupt Pods? Ist die Liste leer, passt der Selector nicht. |

---

## helm

| Befehl | Was er tut |
|---|---|
| `helm repo add <name> <url>` / `helm repo update` | Chart-Repository einbinden und aktualisieren |
| `helm search repo <begriff>` | Charts suchen |
| `helm show values <repo>/<chart>` | Alle konfigurierbaren Werte samt Defaults |
| `helm install <release> <chart> -n <ns> --create-namespace` | Installieren |
| `helm install … -f my-values.yaml --set image.tag=1.2` | Mit eigenen Werten; `--set` gewinnt gegen `-f` |
| `helm upgrade --install <release> <chart> -f my-values.yaml` | Aktualisieren oder neu installieren (idempotent) |
| `helm list -A` | Installierte Releases |
| `helm history <release>` / `helm rollback <release> <rev>` | Revisionen / zurückrollen |
| `helm uninstall <release>` | Entfernen |
| `helm create <name>` | Gerüst für ein eigenes Chart |
| `helm template <release> <chart-pfad> -f values.yaml` | Gerendertes YAML ausgeben, **ohne** zu installieren |
| `helm lint <chart-pfad>` | Chart auf Fehler prüfen |

---

## argocd (CLI)

| Befehl | Was er tut |
|---|---|
| `argocd login <server>` | Am Argo-CD-Server anmelden |
| `argocd app list` | Alle Applications mit Sync- und Health-Status |
| `argocd app get <app>` | Details: Ressourcen, Status, letzter Sync |
| `argocd app create <app> --repo <url> --path <pfad> --dest-server https://kubernetes.default.svc --dest-namespace <ns>` | Application anlegen. Im Projekt passiert das eher deklarativ per YAML im Git. |
| `argocd app diff <app>` | Unterschied Git ↔ Cluster |
| `argocd app sync <app>` | Manuell synchronisieren |
| `argocd app history <app>` / `argocd app rollback <app> <id>` | Historie / zurückrollen |

> [!warning] GitOps-Grundregel
> Unter Argo CD ist **Git die Wahrheit**. `kubectl apply/edit/scale` und `helm upgrade` an Argo CD vorbei werden beim nächsten Sync (mit `selfHeal`) wieder zurückgedreht. Änderungen also immer per Commit.
