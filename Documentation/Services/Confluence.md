---
tags:
  - documentation
---
# Confluence

Confluence ist eine Wiki- und Kollaborationssoftware zur gemeinsamen Erstellung, Organisation und Pflege von Dokumentation und internem Wissen. Anders als eine einfache Notizen-App ist Confluence auf **strukturierte, verlinkte und durchsuchbare Team-Dokumentation** ausgelegt — vergleichbar mit einem firmeninternen Wikipedia, das eng mit Jira und dem Rest von Atlassian verzahnt ist.

Bereitgestellt von: **Atlassian Corporation** (Australien/USA)

Genutzt bei [[Haeger Consulting]] zur kollaborativen Wissensteilung.

> [!tip] Warum Confluence "tief" wirkt
> Confluence hat relativ wenige Grundkonzepte (Space → Seite → Inhalt), aber sehr viele **Bausteine**, um Inhalte innerhalb einer Seite reichhaltig zu gestalten (Makros, Vorlagen, Datenbanken, Whiteboards). Die Tiefe entsteht also nicht durch Komplexität der Struktur, sondern durch die Menge an verfügbaren Werkzeugen *innerhalb* einer Seite.

## Grundkonzepte

### Spaces (Bereiche)
Ein **Space** ist der oberste Container für zusammengehörige Inhalte — meist ein Team, eine Abteilung oder ein Projekt (z. B. "Consulting", "HR", "IT-Doku"). Jeder Space hat:
- eine eigene **Seitenhierarchie** (Baumstruktur)
- eigene **Berechtigungen** (wer darf lesen/schreiben/administrieren)
- eine **Space-Übersichtsseite** als Startpunkt/Inhaltsverzeichnis

Es gibt unterschiedliche Space-Typen: **Team-Spaces** (laufende Zusammenarbeit), **Projekt-Spaces** (zeitlich begrenzt), **Knowledge-Base-Spaces** (Nachschlagewerk/FAQ) und **persönliche Spaces**.

### Seiten & Seitenhierarchie
Alles innerhalb eines Spaces sind **Seiten**. Seiten können beliebig tief **verschachtelt** werden (Unterseiten von Unterseiten), was links im Seitenbaum sichtbar ist. Das ist der wichtigste Ordnungsmechanismus in Confluence — vergleichbar mit Ordnern/Unterordnern, nur dass jede "Ordner-Seite" gleichzeitig auch Inhalt tragen kann.

### Der Editor
Confluence nutzt einen WYSIWYG-Editor mit Unterstützung für:
- Überschriften, Listen, Tabellen, Bilder, Dateianhänge
- **Mehrspaltige Layouts** (Seite in Sektionen/Spalten aufteilen)
- **Slash-Befehle** (`/`) zum schnellen Einfügen von Elementen und Makros, ohne die Maus zu benutzen
- **Echtzeit-Zusammenarbeit**: Mehrere Personen können gleichzeitig an derselben Seite arbeiten (ähnlich Google Docs), Cursor der anderen sind sichtbar

## Wichtige Funktionen im Detail

### Makros
Makros sind die eigentliche "Superkraft" von Confluence — kleine, einfügbare Bausteine, die Inhalte dynamisch darstellen oder anreichern. Wichtige Beispiele:

| Makro | Zweck |
|---|---|
| **Inhaltsverzeichnis (TOC)** | Erzeugt automatisch ein Sprungmenü aus den Überschriften der Seite |
| **Info/Tipp/Warnung/Hinweis-Panel** | Farbige Kästen für wichtige Hinweise |
| **Aufklappen (Expand)** | Ein-/ausklappbarer Bereich, um lange Inhalte zu verstecken |
| **Aufgabenliste** | Checkbox-Listen mit Zuweisung an Personen, sichtbar auch in "Meine Aufgaben" |
| **Seiteneigenschaften / -bericht** | Strukturierte Metadaten je Seite, die sich auf einer Übersichtsseite als Tabelle aggregieren lassen (z. B. Status-Dashboards über mehrere Seiten hinweg) |
| **Jira-Issues / Roadmap** | Bindet lebende Jira-Tickets, Filter oder Roadmaps direkt in die Seite ein |
| **Excerpt / Excerpt Include** | Definiert einen Textausschnitt einmal und lässt ihn auf anderen Seiten wiederverwenden (Single Source of Truth) |
| **Kinderseiten anzeigen (Children Display)** | Listet automatisch alle Unterseiten auf — nützlich für Inhaltsverzeichnisse |
| **Kürzlich aktualisiert** | Zeigt zuletzt geänderte Seiten eines Spaces/Labels |

### Vorlagen & Blueprints
Statt bei null anzufangen, bietet Confluence vorgefertigte **Vorlagen** für wiederkehrende Dokumenttypen: Meeting-Notizen, Retrospektiven, Entscheidungsdokumentation, Produktanforderungen (PRD), How-to-Anleitungen, Projektpläne u. v. m. Eigene Team-Vorlagen lassen sich ebenfalls anlegen, um einen einheitlichen Standard im Space zu erzwingen.

### Whiteboards
Visuelle Collaboration-Fläche innerhalb von Confluence (ähnlich Miro) für Brainstorming, Mindmaps, Prozessdiagramme oder Retros mit Sticky Notes — in Echtzeit gemeinsam nutzbar und in Seiten einbettbar.

### Datenbanken (Confluence Databases)
Erlauben, strukturierte, tabellarische Daten (ähnlich einer Mini-Tabellenkalkulation/Notion-Datenbank) direkt in Confluence zu pflegen und in mehreren Ansichten (Tabelle, Board, Kalender) darzustellen — z. B. für Tool-Übersichten, Kontaktlisten oder Entscheidungs-Logs.

### Jira-Integration
Da beide Produkte von Atlassian stammen, ist die Verzahnung besonders tief: Tickets lassen sich direkt aus Confluence heraus erstellen, referenzieren und deren Live-Status anzeigen. Umgekehrt lässt sich aus einem Jira-Ticket heraus eine verlinkte Confluence-Seite (z. B. eine Spezifikation) öffnen.

### Kommentare & Erwähnungen (@Mentions)
Es gibt **Seitenkommentare** (unten auf der Seite, für generelles Feedback) und **Inline-Kommentare** (an markiertem Text, für punktuelles Feedback — vergleichbar mit Kommentaren in Word/Google Docs). Über `@Name` lassen sich Kolleg:innen erwähnen und benachrichtigen.

### Berechtigungen & Einschränkungen
Auf **Space-Ebene** wird geregelt, wer grundsätzlich Zugriff hat (Lesen/Schreiben/Administrieren). Zusätzlich lassen sich einzelne **Seiten einschränken** (view/edit restrictions), z. B. für vertrauliche Inhalte innerhalb eines sonst offenen Spaces.

### Versionierung & Seitenverlauf
Jede Speicherung erzeugt eine neue Version. Über den **Seitenverlauf** lassen sich alle bisherigen Versionen einsehen, Unterschiede zwischen zwei Versionen vergleichen (Diff-Ansicht) und ältere Versionen wiederherstellen.

### Suche & CQL
Neben der einfachen Volltextsuche gibt es die **Confluence Query Language (CQL)** für komplexe, filterbare Suchanfragen (z. B. "alle Seiten mit Label X, geändert in den letzten 30 Tagen, im Space Y") — nützlich für Power-User und zum Bauen dynamischer Reports.

### Labels (Tags)
Seiten lassen sich mit frei definierbaren **Labels** versehen, um sie themenübergreifend zu kategorisieren und über Spaces hinweg wiederzufinden (z. B. alle Seiten mit Label `onboarding`).

### Export & Weitergabe
Seiten oder ganze Spaces lassen sich als **PDF** oder **Word-Dokument** exportieren, z. B. um Dokumentation an externe Personen ohne Confluence-Zugang weiterzugeben.

## Typische Arbeitsweisen

- **Wissensdatenbank aufbauen**: Einen Space als "Single Source of Truth" für ein Thema pflegen, mit klarer Seitenhierarchie und konsistenter Verschlagwortung (Labels).
- **Meeting-Dokumentation**: Für jedes Meeting eine Seite aus der Meeting-Notizen-Vorlage anlegen, Aktionspunkte als Aufgabenliste erfassen und Teilnehmer per @Mention verantwortlich machen.
- **Projektdokumentation**: Ein Projekt-Space mit Unterseiten für Anforderungen, Entscheidungen, Status-Updates und Retrospektiven; Jira-Tickets für Fortschritt live einbetten.
- **Onboarding**: Ein eigener Bereich mit strukturierter Seitenhierarchie ("Woche 1", "Tools", "Kontakte"), damit neue Mitarbeitende sich selbstständig einarbeiten können.
- **Standards & Prozesse**: Wiederkehrende Abläufe (z. B. "Wie richte ich X ein") als How-to-Vorlage dokumentieren, damit Wissen nicht nur in Köpfen einzelner Personen existiert.

## Kurzanleitung
1. Gewünschten **Space** öffnen bzw. über die Suche finden.
2. Über **+ Erstellen** eine neue Seite anlegen (ggf. passende Vorlage wählen).
3. Inhalt im Editor bearbeiten — mit `/` Makros wie Panels, Aufgabenlisten oder Tabellen einfügen.
4. Kolleg:innen per **@-Mention** einbinden, ggf. Inline-Kommentare für Feedback nutzen.
5. Seite **veröffentlichen** und bei Bedarf sinnvolle **Labels** vergeben, damit sie später wiedergefunden wird.

## Tipps für den Einstieg
- Erst die **Seitenhierarchie eines bestehenden Spaces** erkunden, bevor man selbst neue Seiten anlegt — meist gibt es schon eine sinnvolle Struktur.
- Beim Anlegen neuer Dokumentation möglichst eine **Vorlage** nutzen statt bei null zu beginnen.
- Den **Seitenverlauf** nicht scheuen — nichts geht durch Bearbeiten verloren, ältere Versionen lassen sich jederzeit wiederherstellen.

---
Übersicht: [[Documentation/Websites und Services]]
