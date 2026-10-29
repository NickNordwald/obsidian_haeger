---
tags:
  - documentation
---
# Jira

Jira ist die Projekt- und Vorgangsverwaltungssoftware (Issue-Tracking) von Atlassian. Ursprünglich für Bug-Tracking in der Softwareentwicklung entstanden, ist Jira heute das Standardwerkzeug für **agiles Projektmanagement** (Scrum, Kanban) und wird längst nicht mehr nur von Entwickler:innen genutzt, sondern von ganzen Projektteams zur Planung, Priorisierung und Nachverfolgung von Arbeit.

Bereitgestellt von: **Atlassian Corporation** (Australien/USA) — dieselbe Firma wie bei [[Documentation/Services/Confluence.md|Confluence]], mit dem Jira eng verzahnt ist (gemeinsamer Anbieter, gemeinsame Cloud-Instanz, gegenseitige Verlinkung von Inhalten).

Genutzt bei [[Documentation/Haeger Consulting.md|Haeger Consulting]] zur Planung und Nachverfolgung von Projekt- und Entwicklungsarbeit.

> [!tip] Grundprinzip
> In Jira dreht sich alles um **Issues** (Vorgänge) — einzelne Arbeitseinheiten, die einen **Status** durchlaufen (z. B. "To Do" → "In Progress" → "Done"). Projekte, Boards, Sprints und Berichte sind letztlich nur verschiedene Sichten und Organisationsformen auf genau diese Issues.

## Grundkonzepte

### Projekte
Ein **Projekt** ist der oberste Container, meist einem Team, Produkt oder Kunden zugeordnet (vergleichbar mit einem Space in Confluence). Jedes Projekt hat einen eigenen Issue-Nummernkreis (z. B. `DE-123`) und eigene Konfiguration für Workflows, Berechtigungen und Boards.

### Issues (Vorgänge) & ihre Hierarchie
Issues gibt es in unterschiedlichen Typen, die eine natürliche Hierarchie bilden:

| Typ | Zweck |
|---|---|
| **Epic** | Großes, übergeordnetes Thema/Feature, das sich über mehrere Sprints erstreckt |
| **Story** | Eine fachliche Anforderung aus Nutzersicht ("Als X möchte ich Y, damit Z") |
| **Task** | Eine konkrete, in sich abgeschlossene Arbeitsaufgabe |
| **Subtask** | Ein Teilschritt innerhalb einer Story/eines Tasks |
| **Bug** | Ein zu behebender Fehler |

Epics lassen sich in mehrere Stories/Tasks aufbrechen, wodurch große Vorhaben greifbar und einzeln abarbeitbar werden.

### Workflows & Status
Jeder Issue-Typ durchläuft einen **Workflow** — eine definierte Abfolge von Status (z. B. Backlog → To Do → In Progress → In Review → Done) mit klar geregelten Übergängen (**Transitions**). Workflows lassen sich pro Projekt individuell anpassen, z. B. um einen zusätzlichen QA-Status einzufügen.

### Boards: Scrum vs. Kanban
- **Scrum-Board**: Arbeit wird in **Sprints** (meist 1–4 Wochen) organisiert. Am Sprintstart wird eine feste Menge an Issues aus dem Backlog übernommen, am Ende steht eine Auswertung (Sprint Review/Retro).
- **Kanban-Board**: Kontinuierlicher Fluss ohne feste Sprints, oft mit **WIP-Limits** (maximale Anzahl Issues pro Spalte), um Überlastung einzelner Phasen zu vermeiden.

Beide Boards zeigen Issues als Karten in Spalten entsprechend ihrem Status — der visuelle Fortschritt ist auf einen Blick erkennbar.

### Backlog
Das **Backlog** ist die priorisierte Liste aller noch nicht in Arbeit befindlichen Issues eines Projekts. Priorisierung (Reihenfolge von oben nach unten) bestimmt, was als Nächstes in einen Sprint wandert.

## Wichtige Funktionen im Detail

### JQL (Jira Query Language)
Eine SQL-ähnliche Abfragesprache, mit der sich Issues nach beliebigen Kriterien filtern lassen (z. B. `project = DE AND status = "In Progress" AND assignee = currentUser()`). Damit lassen sich eigene **Filter** speichern und als Basis für Boards, Dashboards oder Benachrichtigungen wiederverwenden.

### Dashboards & Berichte
Konfigurierbare **Dashboards** bündeln Widgets (eigene Issues, Filter-Ergebnisse, Diagramme) auf einer Startseite. Für die Projektsteuerung besonders relevant:
- **Burndown-Chart**: verbleibender Arbeitsaufwand über den Sprint-Verlauf — zeigt, ob ein Sprint im Plan liegt.
- **Velocity-Chart**: abgeschlossener Aufwand pro Sprint über Zeit — Grundlage für realistische Sprint-Planung.
- **Cumulative-Flow-Diagram**: zeigt, wie viele Issues sich in welchem Status befinden — deckt Engpässe (Bottlenecks) im Workflow auf.

### Automatisierung (Jira Automation)
Regelbasierte **Wenn-Dann-Automatisierungen** (z. B. "Wenn ein Issue auf 'Done' gesetzt wird, benachrichtige den Reporter" oder "Weise neue Bugs automatisch dem Komponenten-Verantwortlichen zu"), reduzieren manuellen Pflegeaufwand.

### Confluence-Integration
Da Jira und Confluence vom selben Anbieter stammen, lassen sich Confluence-Seiten (z. B. Spezifikationen) direkt mit Issues verknüpfen, und umgekehrt Jira-Issues/-Roadmaps live in Confluence-Seiten einbetten (siehe [[Documentation/Services/Confluence.md#Jira-Integration|Confluence-Notiz]]).

### Components & Labels
**Components** gliedern ein Projekt in technische/fachliche Teilbereiche (z. B. "Backend", "Frontend", "Infrastruktur") mit eigenem Standard-Verantwortlichen. **Labels** sind freie Schlagworte zur zusätzlichen, projektübergreifenden Kategorisierung — analog zu Labels in Confluence.

### Berechtigungen & Rollen
Zugriff wird über **Projektrollen** (z. B. Administrator, Entwickler, Betrachter) und dahinterliegende **Permission Schemes** gesteuert, die festlegen, wer Issues anlegen, bearbeiten, workflow-transitionieren oder löschen darf.

### Zeiterfassung (Worklogs)
Auf Issue-Ebene lässt sich geleistete Arbeitszeit als **Worklog** erfassen sowie eine Schätzung (Story Points oder Zeitschätzung) hinterlegen — Grundlage für Aufwandsvergleiche (geschätzt vs. tatsächlich).

## Typische Arbeitsweisen

- **Sprint-Planung**: Aus dem priorisierten Backlog werden zu Sprintbeginn Issues ins Sprint-Board übernommen, basierend auf Team-Kapazität (Velocity der letzten Sprints).
- **Tägliche Arbeit / Daily Stand-up**: Das Scrum-/Kanban-Board dient als visuelle Grundlage für kurze Status-Abstimmungen im Team.
- **Bug-Tracking**: Neue Fehler werden als Bug-Issue erfasst, mit Priorität versehen und einem Verantwortlichen/einer Komponente zugeordnet.
- **Anforderungen aus Confluence überführen**: Fachliche Spezifikationen werden in Confluence dokumentiert, die daraus resultierenden Arbeitspakete als Jira-Issues angelegt und zurückverlinkt.
- **Retrospektiven vorbereiten**: Burndown- und Velocity-Daten aus abgelaufenen Sprints liefern die Faktenbasis für die Retro-Diskussion.

## Kurzanleitung
1. Gewünschtes **Projekt** öffnen (linke Seitenleiste) und das passende **Board** (Scrum/Kanban) auswählen.
2. Eigene offene Issues über **"Nur meine Issues"**-Filter oder die JQL-Suche (`assignee = currentUser()`) finden.
3. Einen Issue öffnen, Status per Drag-and-drop auf dem Board oder über den **Transition**-Button ändern.
4. Bei neuer Arbeit: **Issue erstellen**, passenden Typ (Story/Task/Bug/Subtask) wählen, kurz und präzise beschreiben, ggf. Component/Label setzen.
5. Fortschritt und Kommentare direkt im Issue dokumentieren, damit der Verlauf für alle nachvollziehbar bleibt.

## Tipps für den Einstieg
- Erst das **Board des eigenen Projekts** eine Weile beobachten, um den vereinbarten Workflow (Status-Reihenfolge) zu verstehen, bevor man selbst Issues verschiebt.
- Issues **klein und konkret** halten — große, vage Tickets sind ein typisches Anfängerproblem und erschweren die Nachverfolgung.
- Bei wiederkehrenden Suchanfragen einen **eigenen Filter speichern**, statt die JQL-Abfrage jedes Mal neu zu tippen.

---
Übersicht: [[Documentation/Websites und Services.md]]
