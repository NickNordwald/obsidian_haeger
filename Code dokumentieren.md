---
tags:
  - workflow
erstellt: 2026-10-28
---

# Code dokumentieren

Dieser Workflow beschreibt, wie Code im Vault dokumentiert wird. Er gilt immer, wenn Code dokumentiert wird, auch innerhalb anderer Workflows. Andere Workflows dürfen die Notizen um eigene Abschnitte und Properties ergänzen.

Die Doku richtet sich an mich und an Nachfolger. Sie dient vor allem dazu, Detailfragen nachzuschlagen und Reviews vorzubereiten.

## Grundsätze

- Dokumentiere, was man aus dem Code nicht schnell selbst sieht: Zweck, Zusammenspiel, Begründungen, Fallstricke und Begriffe.
- Wiederhole Inhalte aus dem Code nur, wenn man eine Stelle sonst nicht versteht, ohne den Code zu öffnen, z. B. die zwei Konfigurationswerte, die ein Verhalten steuern. Schreibe keine API-Referenz und beschreibe nicht jede Funktion.
- Beschreibe den Code, bewerte ihn nicht. Halte auch Review-Ergebnisse nicht in der Doku fest.
- Dokumentiere alles in derselben Tiefe. Vertiefe nur, was der Anlass verlangt.
- Beschreibe immer den aktuellen Stand. Die Änderungshistorie steht in Git.

## Auswahl

Dokumentiere zuerst, was am wichtigsten ist, in dieser Reihenfolge:

1. Was der Anlass verlangt
2. Einstiegspunkte wie `main`, API-Endpunkte, Jobs und Event-Handler
3. Bereiche, die laut Commit-Historie häufig geändert werden
4. Stellen, die ohne Erklärung schwer zu verstehen sind

## Belege

- Verweise auf Code mit Pfad und Symbol: `src/config/loader.py` → `parse_config()`. Nenne Zeilennummern nur, wenn es auf die Zeile ankommt.
- Zeige Code-Ausschnitte nur, wenn ein Muster ohne sie nicht verständlich ist. Ein Ausschnitt hat höchstens zehn Zeilen.
- Nenne Begründungen nur mit Beleg: Commit-Nachricht, Kommentar im Code, Doku oder Aussage von mir. Gib die Quelle an, z. B. den Commit-Hash.
- Schreibe Vermutungen und Begründungen ohne Beleg in einen Callout: `> [!question]+ Vermutung`.

## Notizen

Lege die Doku in `Documentation/<Projekt>/` an. Bei vielen Notizen gruppiere sie in Unterordnern, z. B. nach Bereich. Die Überblick-Notiz bleibt im Projektordner. Jede Notiz hat diese Properties:

```yaml
tags:
  - <projekt>
erstellt:
basis-commit:
```

`basis-commit` ist der Commit, dem der Inhalt der Notiz entspricht. Die Überblick-Notiz hat zusätzlich `repo-pfad`.

Gib jeder Notiz die Abschnitte ihres Typs in der angegebenen Reihenfolge. Lass leere Abschnitte weg. Überschreitest du eine Längengrenze, kürze die Notiz oder begründe die Länge in deiner Zusammenfassung.

### `<Projekt> - Überblick`

Höchstens 400 Wörter.

- `Zweck`: was das Projekt tut und für wen, in zwei bis drei Sätzen
- `Funktionalitäten`: Link auf jede Funktionalitäts-Notiz mit einem Satz
- `Bauen und Ausführen`: die nötigen Befehle
- `Einstieg`: wo man mit dem Lesen des Codes beginnt

### `<Projekt> - Architektur`

Höchstens 800 Wörter und ein Diagramm.

- `Module`: jedes Modul mit Pfad und einem Satz zu seiner Aufgabe
- `Zusammenspiel`: wie die Module zusammenarbeiten, als Fließtext, optional mit einem Mermaid-Diagramm
- `Entscheidungen`: Architekturentscheidungen mit Beleg

### `<Projekt> - <Funktionalität>`

Eine Notiz pro Funktionalität, höchstens 600 Wörter. Wird eine Notiz länger, prüfe, ob sie zwei Funktionalitäten beschreibt.

- `Zweck`
- `Einstiegspunkt`
- `Beteiligte Module`
- `Ablauf`: höchstens sieben Schritte
- `Fallstricke`

### `<Projekt> - Technologien und Libraries`

- Fasse die direkten Abhängigkeiten in einer Tabelle mit den Spalten Name, Version und Zweck im Projekt zusammen.
- Nimm nur Abhängigkeiten auf, die für das Verständnis relevant sind.
- Beschreibe nur, wie das Projekt eine Technologie nutzt. Allgemeines gehört in eine eigene Notiz unter `Documentation/<Technologie>/`, auf die du verlinkst.

### `<Projekt> - Glossar`

- Erkläre projektspezifische Begriffe und Fachbegriffe der Domäne in je einem Satz.
- Sortiere die Begriffe alphabetisch.

## Schritte

1. **Umfang:** Kläre, welche Notizen neu entstehen oder aktualisiert werden.
2. **Lesen:** Lies den Code und sammle Belege. Ändere den Code nie.
3. **Schreiben:** Schreibe die Notizen nach den Abschnitten oben. Setze `basis-commit` auf die Ausgabe von `git -C "<repo-pfad>" rev-parse HEAD`.
4. **Prüfen:**
	- Prüfe, ob jeder genannte Pfad existiert und jedes genannte Symbol dort vorkommt: `rg -n "<Symbol>" "<repo-pfad>/<Pfad>"`.
	- Prüfe die Längengrenzen mit `wc -w "<Datei>"`.
	- Prüfe, ob der Überblick jede Funktionalitäts-Notiz verlinkt.
	- Prüfe, ob jede Begründung eine Quelle hat oder als Vermutung markiert ist.

## Aktualisieren

- Ermittle die Änderungen seit `basis-commit` mit `git -C "<repo-pfad>" diff --stat <basis-commit>`.
- Aktualisiere die betroffenen Notizen und setze ihren `basis-commit` neu.
- Prüfe, ob neue Funktionalitäten eine eigene Notiz brauchen.
