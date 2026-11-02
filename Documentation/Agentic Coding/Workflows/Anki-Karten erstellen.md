---
tags:
  - workflow
erstellt: 2026-11-02
anki-sync: false
---

# Anki-Karten erstellen

Dieser Workflow beschreibt, wie du aus Notizen Anki-Karten für das Plugin [Obsidian Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync) erstellst. Das Plugin überträgt Kartenblöcke aus Markdown-Dateien über AnkiConnect nach Anki.

## Kartendatei

Lege die Karten in einer eigenen Datei `<Thema> - Karteikarten.md` neben den Notizen zum Thema an. Die Datei hat diesen Aufbau:

```yaml
---
tags:
erstellt:
deck: "<Bereich>::<Thema>"
---
```

- `deck` legt das Zieldeck in Anki fest. `::` trennt die Ebenen.
- Das Plugin übernimmt die Einträge aus `tags` als Tags in Anki.
- Unter dem Frontmatter steht eine Checkliste der behandelten Abschnitte, darunter folgen die Karten.

## Kartenformat

Ein neuer Block sieht so aus:

```markdown
<!-- basicblock-start -->
Frage::
Antwort
<!-- basicblock-end -->
```

- Schreibe keine `oid` in neue Blöcke. Das Plugin trägt sie beim ersten Abgleich selbst ein. Kopierst du einen bestehenden Block, entferne die `oid`.
- `::` trennt Frage und Antwort und darf im Block nur einmal vorkommen. Weitere `::` und `}}` zerstören die Karte.
- Schreibe kein HTML wie `<br>`. Das Plugin maskiert HTML, und Anki zeigt es als Text an.
- Erzwinge einen Zeilenumbruch mit zwei Leerzeichen am Zeilenende. Ein einfacher Zeilenumbruch wird in Anki zu einem Leerzeichen.
- Vermeide Listen, Tabellen und Codeblöcke in Antworten. Markdown wie Fettdruck, Inline-Code, Formeln mit `$` und Bilder mit `![[…]]` funktioniert.
- Schreibe Start- und Endmarker immer als Paar, auch in Notizen, die das Format nur erklären.

## Kartenstil

- Eine Karte fragt genau eine Sache ab.
- Die Frage ist ohne Kontext eindeutig.
- Die Antwort hat höchstens zwei Zeilen. Braucht sie mehr, teile die Karte oder lass den Rest in der Notiz.
- Nur Basic-Karten, keine Cloze.
- Fasse lieber zusammen, als Karten auf eine Zielzahl aufzufüllen.

## Schritte

1. **Quellen erkennen:** Prüfe den Ordner und benenne, was du gefunden hast:
	- Nur Notizen
	- Eine Leseliste
	- Eine Lernkontrolle
2. **Discovery:** Frage nach:
	- Ob die erkannten Quellen stimmen und welche du nutzen sollst
	- Anzahl der Karten, als Richtwert pro Abschnitt oder insgesamt
	- Detailgrad: Kernbegriffe, Zusammenhänge oder Details zum Nachschlagen
	- Name des Decks
3. **Inhalt sammeln:** Je nach Quelle:
	- **Nur Notizen:** Nutze den Inhalt der Notizen im Ordner.
	- **Leseliste:** Die Leseliste ist nur das Verzeichnis, nicht die Grundlage der Karten. Schreibe die Karten aus dem Inhalt der Quellen, nicht aus den Beschreibungen in der Leseliste.
		- Die Quellen liegen meist nicht im Vault. Rufe Webseiten, Dokumentation und Artikel über das Web ab und lies genau die Kapitel und Abschnitte, die die Leseliste angibt.
		- Liegt eine Quelle im Vault, z. B. als PDF in `Resources/Startdokumente/`, nutze diese Datei.
		- Ist eine Quelle nicht frei zugänglich, z. B. ein Buch, suche nach einer frei zugänglichen Fassung desselben Inhalts, etwa dem Kapitel auf der Website des Verlags. Findest du keine, sag mir das und lass die Quelle weg.
		- Nimm nur Aussagen auf, die du in der Quelle gefunden hast.
	- **Lernkontrolle:** Decke jede Frage der Lernkontrolle mit mindestens einer Karte ab. Falsch beantwortete Fragen und gesammelte Auszüge haben Vorrang.
4. **Karten schreiben:** Schreibe die Karten nach Kartenformat und Kartenstil, gegliedert nach Abschnitten.
5. **Prüfen:**
	- Jeder Block hat genau ein `::`, keine `oid` und kein `<br>`.
	- Start- und Endmarker sind gleich häufig: `rg -c "basicblock-(start|end)" "<Datei>"`.
6. **Übergabe:** Sag mir, dass ich Anki starten und den Abgleich ausführen soll. Die Abschlussmeldung muss `Failed Created Blocks: 0` zeigen.

## Fallstricke

- Verschiebst, umbenennst oder löschst du eine Kartendatei, löscht das Plugin die Karten beim nächsten Abgleich samt Lernfortschritt. Frage vorher nach und empfiehl einen Export aus Anki mit Lernfortschritt.
- Bleibt nach dem Abgleich `oid="undefined"` in einem Block stehen, wurde er nicht angelegt. Entferne das Attribut und gleiche erneut ab.
