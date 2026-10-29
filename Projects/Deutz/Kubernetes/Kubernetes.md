---
tags:
  - documentation
---
# Kubernetes: Lernpfad (Deutz)

Lernpfad für das [[Projects/Deutz/Deutz|Deutz-Projekt]]. Er beginnt bewusst bei null und baut nicht auf dem [[Kubernetes Crashkurs - Überblick|Crashkurs aus der Interview-Vorbereitung]] auf. Den kannst du bei Bedarf nachschlagen.

> [!important] Ziel: was du im Projekt können musst
> 1. **Apps deployen und debuggen** mit `kubectl`: Pods, Deployments, Services, ConfigMaps/Secrets, Probes, Logs und Fehlerbilder
> 2. **CronJobs anlegen** und prüfen, ob sie laufen
> 3. **Helm verstehen**: Charts installieren und konfigurieren, ein eigenes Mini-Chart lesen und anpassen
> 4. **Argo CD verstehen**: wie aus einem Git-Repo ein Deployment wird (GitOps), Sync, Status und Rollback
>
> **Budget:** etwa 10 h, höchstens 2 h pro Tag. Geübt wird fast nur im Browser und kostenlos.

## Arbeitsweise

1. Du arbeitest den Block in der **Online-Quelle** durch, also mit Übungen im echten Terminal.
2. In der **Block-Notiz** stehen vorab Leitfragen. Du schreibst während oder direkt nach dem Üben **roh** mit: was du gemacht hast, woran du hängen geblieben bist, offene Fragen. Stichpunkte reichen.
3. Du beantwortest die Leitfragen **aus dem Kopf**. Danach sagst du Claudian „Block X reviewen“. Claudian gibt dir ein Urteil pro Frage (✅/🟡/❌) und schreibt daraus den fertigen Abschnitt.
4. Befehle, die du wirklich benutzt hast, landen in der [[Kubernetes Befehlsreferenz]].
5. Pro Block entstehen **5–8 Anki-Karten** in [[Projects/Deutz/Kubernetes/Anki Karteikarten|Anki Karteikarten]], Deck `Deutz::Kubernetes`.

## Fortschritt

- [ ] **Block 1: Kubernetes-Grundlagen** (~5 h): [[1. Kubernetes-Grundlagen]]
      Quelle: die offizielle Doku, [Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/) und ausgewählte „Tasks“-Seiten. Die Seiten erklären erst und lassen dich dann mitmachen, im [Kubernetes-Playground auf Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes).
- [ ] **Block 2: Helm** (~1,5 h): [[2. Helm]]
      Quellen: [Helm Quickstart](https://helm.sh/docs/intro/quickstart/) und [Using Helm](https://helm.sh/docs/intro/using_helm/), geübt im [iximiuz-k3s-Playground](https://labs.iximiuz.com/playgrounds/k3s) (Helm vorinstalliert) oder im Killercoda-Playground
- [ ] **Block 3: Argo CD / GitOps** (~2,5 h): [[3. Argo CD]]
      Quelle: [Akuity Academy: Introduction to GitOps and Argo CD](https://academy.akuity.io/), kostenlos, 23 Lektionen, von den Argo-Erfindern
      Alternative mit gehosteten Labs: [GitOps Fundamentals (Octopus, ehemals Codefresh)](https://learning.octopus.com/course/gitops-fundamentals)
- [ ] **Block 4: Abschluss** (~1 h): [[4. Abschlussübung]]
      Eigene App als Helm-Chart plus CronJob, ausgerollt per Argo CD

## Notizen in diesem Ordner

| Notiz | Inhalt |
|---|---|
| [[1. Kubernetes-Grundlagen]] … [[4. Abschlussübung]] | Leitfragen, deine Mitschrift und der fertige Abschnitt pro Block |
| [[Kubernetes Befehlsreferenz]] | `kubectl`, `helm` und `argocd` zum Nachschlagen |
| [[Projects/Deutz/Kubernetes/Anki Karteikarten\|Anki Karteikarten]] | Deck `Deutz::Kubernetes` |

## Verwandt

- [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.7 Docker/1.7 Docker - Vertiefung|1.7 Docker]] als Vorwissen: Images, Container, Compose
- [[Kubernetes Crashkurs - Überblick]] und [[Kubernetes Leseliste]] aus der Interview-Vorbereitung
- [[ToDo/Current/Kubernetes|Task: Kubernetes]] · [[ToDo/Current/Deutz|Task: Vorbereitung auf das Deutz-Projekt]]
