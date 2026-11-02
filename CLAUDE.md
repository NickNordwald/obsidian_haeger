# Agent-Anweisungen

Dieses Dokument beschreibt Struktur und Konventionen dieses Obsidian Vaults für Agents.

## Ziel des Vaults

1. Dokumentation von Arbeitsprojekten als Data Engineer Trainee bei [[Haeger Consulting]], z. B. bei der Deutz AG
2. Lernen von Themen aus dem Traineeship und neuer Techniken, die für die Arbeit an den Projekten wichtig sind
3. Erklären von bestehenden Codebases für neue Nutzer

## Deine Aufgaben

- Erstelle Dokumentation.
- Halte den Vault aufgeräumt und gut strukturiert.
- Hilf dabei, Themen in der Tiefe zu verstehen.

## Struktur

- `Calendar` enthält tägliche bzw. wöchentliche Notizen in den Unterordnern `Daily` und `Weekly` im Format `JJJJ-MM-TT` bzw. `JJJJ-WWW`, z. B. `2026-W27`.
- `ToDo` enthält das Task-Management. Der Zustand eines Tasks ergibt sich aus dem Unterordner: `Current`, `Simmering`, `Sleeping` oder `Done`.
- `Documentation` enthält Dokumentation zu Technologien, Personen, Services, zu Haeger Consulting und zum Traineeship. Jedes Thema bekommt einen eigenen Unterordner.
- `Projects` enthält die Arbeitsprojekte. Jedes Projekt bekommt einen eigenen Unterordner, z. B. `Deutz`.
- `Resources` enthält Templates, Bases, Canvases sowie Anhänge.
- `Zarchive` ist das Archiv für nicht mehr aktive Inhalte.

## Konventionen

- Vermeide doppelte Notizen. Jedes Thema hat eine Notiz, in der es ausführlich beschrieben ist.
- Schreibe in jede Notiz, was man an dieser Stelle zum Verstehen braucht, auch wenn es anderswo schon steht. Wiederhole dabei höchstens eins bis fünf Sätze. Braucht es mehr, verlinke auf die ausführliche Notiz und sag in einem Satz, was dort steht. Beispiel: „Nutzt [[Kafka]] als Message Queue zwischen den Services“ statt nur „Nutzt [[Kafka]]“.
- Verwende in Dateinamen keine Sonderzeichen. Leerzeichen sind erlaubt.

## Stil

- Schreibe Notizen auf Deutsch. Behalte englische Fachbegriffe bei, wenn sie üblich sind, z. B. Commit oder Library.
- Schreibe sachlich, knapp und präzise. Verzichte auf Füllsätze und Einleitungen wie „In dieser Notiz geht es um …“.
- Nutze Überschriften und Listen, wenn sie das Nachschlagen erleichtern. Schreibe Zusammenhänge und Begründungen als Fließtext.
- Verwende für einen Begriff im ganzen Vault dieselbe Bezeichnung. Ist ein Begriff schon etabliert, übernimm ihn.
- Verwende keine Emojis.

## Formatierung

Mechanische Regeln wie Leerzeilen, Überschriften-Ebenen, Listenzeichen, Codeblöcke und URLs legt `.markdownlint-cli2.jsonc` fest. Sie werden mit dem Check unter „Überprüfung“ geprüft. Hier stehen die Regeln, die der Linter nicht prüft.

### Überschriften und Absätze

- Die Überschrift erster Ebene entspricht dem Dateinamen.
- Nutze höchstens Ebene 4. Beginne Überschriften groß und nummeriere sie nicht.
- Schreibe jeden Absatz in eine Zeile, ohne manuelle Zeilenumbrüche.

### Listen

- Nutze nummerierte Listen nur, wenn die Reihenfolge zählt.
- Beginne jeden Eintrag groß. Beende vollständige Sätze mit einem Punkt, Stichworte ohne.
- Rücke verschachtelte Einträge mit einem Tab ein.

### Auszeichnung und Links

- Setze Dateinamen, Pfade, Befehle, Property-Namen und Code-Bezeichner in Inline-Code.
- Nutze Fettdruck sparsam, nur für Begriffe, die man beim Überfliegen finden soll.
- Nutze deutsche Anführungszeichen („…“) im Fließtext.
- Nutze Tabellen nur, um mehrere Dinge anhand derselben Merkmale zu vergleichen.
- Nutze Mermaid für Diagramme.
- Verlinke Notizen im Vault mit Wikilinks und externe Seiten mit `[Text](URL)`.
- Nutze nur die Callouts `note` für Hinweise, `tip` für Empfehlungen, `warning` für Fallstricke und `question` für offene Fragen. Schreibe sie immer einklappbar, ausgeklappt und mit Titel: `> [!tip]+ Titel`.

### Properties und Tags

- Gib jeder neuen Notiz die Properties `tags` und `erstellt`. Ordne Properties so: `tags`, `erstellt`, dann alle weiteren.
- Füge neue Properties nur hinzu, wenn keine passende existiert. Wähle Namen so spezifisch, dass sie nur in einem Kontext vorkommen, z. B. `lernphase` statt `status`.
- Schreibe Namen von Properties und Tags klein und trenne Wörter mit Bindestrich, z. B. `basis-commit`.
- Vergib Tags nur in den Properties und ohne Hierarchie.

### Anhänge

- Lege Bilder in `Resources/Images/` ab und gib ihnen sprechende Dateinamen, z. B. `Architektur Motorsteuerung.png` statt `Pasted image 20261028.png`.
- Lege Quelldokumente wie PDFs in `Resources/Startdokumente/` ab, dupliziere ihren Inhalt nicht und verlinke sie aus der passenden Notiz.

### Schreibweisen

- Schreibe Abkürzungen mit Leerzeichen: „z. B.“, „d. h.“, „u. a.“.
- Schreibe Zahlen von null bis zwölf im Fließtext aus, ab 13 als Ziffern. Schreibe Zahlen mit Einheit immer als Ziffern.
- Setze ein Leerzeichen zwischen Zahl und Einheit: „4 h“, „16 GB“, „50 %“.
- Trenne Tausender mit Punkt: „1.500“.
- Schreibe Daten als `JJJJ-MM-TT` und Uhrzeiten als `HH:MM`.

## Regeln

- Frage nach, bevor du bestehende Notizen umbenennst oder verschiebst.
- Lösche Notizen nie. Soll eine Notiz entfernt werden, frage nach und verschiebe sie danach nach `Zarchive`.
- Verändere die Leitfäden im Traineeship (`X.Y <Thema> - Leitfaden.md`) inhaltlich nicht. Sie sind vom Unternehmen vorgegeben.
- Fasse deine Änderungen kurz zusammen.
- Rate nie. Recherchiere bei Unsicherheit aktiv oder frage nach.
- Schreibe niemals erfundene Informationen in den Vault.

## Git

- Committe regelmäßig in logischen Einheiten.
- Halte die Commit-Historie sauber.
- Pushe niemals.
- Befolge für Commit-Nachrichten diese Regeln:
	1. Trenne Subject und Body durch eine Leerzeile.
	2. Begrenze die Subject-Zeile auf 50 Zeichen.
	3. Beginne die Subject-Zeile mit einem Großbuchstaben.
	4. Beende die Subject-Zeile nicht mit einem Punkt.
	5. Formuliere die Subject-Zeile im Imperativ.
	6. Brich den Body nach 72 Zeichen um.
	7. Erkläre im Body, was und warum geändert wurde, nicht wie.

## Überprüfung

Nutze die folgenden Checks, um deine Änderungen zu überprüfen. Wähle die Checks, die zur Aufgabe passen. Schlägt ein Check fehl, behebe das Problem oder weise in deiner Zusammenfassung darauf hin.

### Umfang der Änderungen

- Prüfe mit `git status` und `git diff --stat`, welche Dateien du geändert hast. Es sollten nur Dateien betroffen sein, die zur Aufgabe gehören.
- Prüfe mit `git diff`, ob du nur das geändert hast, was die Aufgabe verlangt.
- Prüfe mit `git diff --name-status`, ob Dateien gelöscht (`D`) oder verschoben (`R`) wurden. `D` darf nie vorkommen, `R` nur nach Rückfrage.

### Links

- Liste die Wikilinks einer geänderten Notiz mit `rg -o "\[\[[^\]|#]+" "<Datei>"` und prüfe, ob zu jedem Ziel eine Notiz existiert.
- Suche nach dem Umbenennen oder Verschieben mit `rg "\[\[<Alter Name>"` nach Links auf den alten Namen. Es dürfen keine übrig bleiben.

### Properties und Tags

- Prüfe, ob jede neue Notiz die Properties `tags` und `erstellt` hat.
- Prüfe vor dem Vergeben eines Tags, ob es schon in ähnlicher Form existiert: `rg -i "<tag>"`.
- Prüfe vor dem Hinzufügen einer Property, ob es sie schon gibt und ob sie dort dieselbe Bedeutung hat: `rg "^<property>:"`.

### Duplikate

- Suche vor dem Anlegen einer Notiz nach bestehenden Notizen zum Thema: `rg -il "<Begriff>"`.
- Prüfe, ob Dateinamen eindeutig sind. Die Ausgabe muss leer sein: `find . -name "*.md" -not -path "./.*" | sed 's|.*/||' | sort | uniq -d`

### Idempotenz

- Committe bei Aufräum- oder Normalisierungsaufgaben nach dem ersten Durchlauf und wende die Regel ein zweites Mal an. `git status` muss danach leer bleiben.

### Formatierung

- Prüfe geänderte Notizen mit `npx markdownlint-cli2 "<Datei>"`. Viele Fehler behebt `--fix` automatisch.
- Läufst du in der Flatpak-Sandbox von Obsidian, gibt es dort kein `npx`. Führe den Check dann über den Host aus: `flatpak-spawn --host bash -lic 'cd "<Vault-Pfad>" && npx markdownlint-cli2 "<Datei>"'`. Die Warnungen zu „job control“ kannst du ignorieren.
- Prüfe die Regeln, die markdownlint nicht abdeckt, selbst: Groß- und Kleinschreibung in Listen, Punkt bei Sätzen, Callout-Format, Schreibweisen.

## Workflows

Jeder Workflow liegt in einer eigenen Datei, ist in sich vollständig und verweist nicht auf Regeln anderer Workflows. Eine Ausnahme ist [[Code dokumentieren]]: Folge diesem Workflow immer, wenn du Code dokumentierst, auch innerhalb eines anderen Workflows.

1. [[Thema lernen]]
2. [[Codebase verstehen]]
3. [[Code dokumentieren]]
4. [[Anki-Karten erstellen]]
