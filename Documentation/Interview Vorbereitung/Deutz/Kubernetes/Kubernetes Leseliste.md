---
tags:
  - documentation
---

# Kubernetes Leseliste (Deutz)

Leseaufgaben zu [[Kubernetes Crashkurs - Überblick]]. Straffer als die [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste|React-Leseliste]]: ein **Minimalpfad** (~1 h 45), der auf jeden Fall drin sein sollte, ein **Rest-Core** (~1 h 15) bei etwas mehr Zeit und optionale Vertiefungen.

Ablauf wie gehabt: erst den [[Kubernetes Crashkurs - Überblick|Überblick]] lesen, dann die Quellen mit Leseauftrag. Unklares sofort in [[Kubernetes Crashkurs - Überblick#10. Offene Fragen|Offene Fragen]], am Ende die [[Kubernetes Crashkurs - Überblick#9. Fürs Gespräch: Kontrollfragen|Kontrollfragen]] aus dem Kopf.

> [!info] Eine Quelle trägt alles
> **kubernetes.io/docs**, die offizielle Doku. Die „Concepts"-Seiten sind lang — **lies immer nur die Einleitung und die im Leseauftrag genannten Abschnitte**, der Rest ist Referenz. Alle URLs am 28.09.2026 geprüft. Stand heute: Kubernetes **1.37**, kind **0.33**.

---

## Minimalpfad (~1 h 45) — Pflicht

- [x] **Overview** (~15 min) — <https://kubernetes.io/docs/concepts/overview/>
      *Leseauftrag: „Going back in time" (Deployment-Epochen: Server → VM → Container) und „Why you need Kubernetes". Welche fünf, sechs Fähigkeiten zählt die Seite auf, und welche davon hat Compose nicht? Lies auch „What Kubernetes is not".*
- [x] **Cluster Architecture** (~20 min) — <https://kubernetes.io/docs/concepts/architecture/>
      *Leseauftrag: das Diagramm oben verstehen und jede Komponente in einem Satz erklären können. Vergleiche mit dem Diagramm in Abschnitt 3 des Überblicks.*
- [x] **Controllers** (~10 min) — <https://kubernetes.io/docs/concepts/architecture/controller/>
      **Der Schlüsseltext für das mentale Modell.**
      *Leseauftrag: das Thermostat-Beispiel. Was heißt „desired state" vs. „current state"? Schlag den Bogen zu Reacts Reconciliation.*
- [ ] **Pods** (~15 min) — <https://kubernetes.io/docs/concepts/workloads/pods/>
      Nur bis einschließlich „Workload resources for managing pods".
      *Leseauftrag: Warum legt man Pods fast nie direkt an? Wann gehören zwei Container in einen Pod?*
- [ ] **Deployments** (~25 min) — <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>
      Abschnitte „Creating a Deployment", „Updating a Deployment", „Rolling Back a Deployment", „Scaling a Deployment". Die YAML-Beispiele genau lesen.
      *Leseauftrag: Was ist die Beziehung Deployment → ReplicaSet → Pod? Was passiert beim Image-Wechsel mit den ReplicaSets, und warum macht das Rollback so einfach?*
- [ ] **Service** (~20 min) — <https://kubernetes.io/docs/concepts/services-networking/service/>
      Einleitung, „Defining a Service", „Service type" (nur ClusterIP/NodePort/LoadBalancer).
      *Leseauftrag: Wie findet ein Service „seine" Pods? Was passiert, wenn der Selector nicht passt?*

## Rest-Core (~1 h 15) — wenn Zeit ist

- [ ] **Objects In Kubernetes** (~15 min) — <https://kubernetes.io/docs/concepts/overview/working-with-objects/>
      *Leseauftrag: `spec` vs. `status`. Pflichtfelder eines Manifests. Wer schreibt welche Hälfte?*
- [ ] **ConfigMaps** (~10 min) — <https://kubernetes.io/docs/concepts/configuration/configmap/>
      Einleitung + „ConfigMaps and Pods".
- [ ] **Secrets** (~15 min) — <https://kubernetes.io/docs/concepts/configuration/secret/>
      Nur die Einleitung und den **Warnkasten** oben.
      *Leseauftrag: Warum ist ein Secret ohne weitere Maßnahmen nicht geheim? Welche vier Maßnahmen empfiehlt die Seite?*
- [ ] **Liveness, Readiness and Startup Probes** (~15 min) — <https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/>
      Nur „Define a liveness HTTP request" und „Define readiness probes". Brauchst du direkt für die Übung.
- [ ] **kubectl Quick Reference** (~10 min, überfliegen) — <https://kubernetes.io/docs/reference/kubectl/quick-reference/>
      *Leseauftrag: die Abschnitte „Viewing and finding resources" und „Interacting with running Pods" — das sind deine Debug-Befehle.*
- [ ] **kind Quick Start** (~10 min) — <https://kind.sigs.k8s.io/docs/user/quick-start/>
      Installation, „Creating a Cluster", „Loading an Image Into Your Cluster". Vorbereitung für die Übung.

---

## Vertiefung (optional)

- **PersistentVolumes** (~20 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>
  Einleitung + „Lifecycle of a volume and claim". Für Schritt 2 der Übung hilfreich.
- **StatefulSets** (~15 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/>
  Einleitung + „Using StatefulSets" + „Limitations". Antwort auf „Wie betreibt man eine Datenbank in K8s?".
- **Jobs** (~10 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/workloads/controllers/job/>
  Nur die Einleitung und „Running an example Job". Für den Migrations-Schritt der Übung.
- **Resource Management for Pods and Containers** (~15 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/>
  requests vs. limits.
- **Namespaces** (~10 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/>
- **DNS for Services and Pods** (~10 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/>
  Warum `DB_HOST: db` in der Übung einfach weiter funktioniert.
- **Gateway API** (~15 min, *andere Perspektive*) — <https://kubernetes.io/docs/concepts/services-networking/gateway/>
  plus kurz der Blogpost **Ingress NGINX Retirement** — <https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/>. Aktuelles Gesprächsthema.
- **Horizontal Pod Autoscaling** (~10 min, *weiterführend*) — <https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/>
  Nur die Einleitung.
- **Helm Quickstart** (~15 min, *weiterführend*) — <https://helm.sh/docs/intro/quickstart/>
- **Kubernetes Basics Tutorial** (~1 h, *andere Perspektive*) — <https://kubernetes.io/docs/tutorials/kubernetes-basics/>
  Das offizielle Einsteiger-Tutorial mit minikube: Deploy → Explore → Expose → Scale → Update. Deckt denselben Stoff wie die Übung ab, nur mit einer Beispiel-App. **Nimm das nur, wenn du die Übung nicht machst.**

## Was hier bewusst fehlt

Netzwerk-Interna (CNI, Overlay-Netze) · Scheduling-Details (Affinity, Taints/Tolerations) · RBAC im Detail · Cluster-Aufbau und -Betrieb (kubeadm, Upgrades, etcd-Backups) · Service Mesh · Security (Pod Security Standards, Network Policies). Alles davon steht als Vokabel im [[Kubernetes Crashkurs - Überblick#8. Das Ökosystem: Vokabeln|Überblick, Abschnitt 8]] oder kann im Gespräch als „kenne ich als Begriff, noch nicht praktisch" benannt werden.
