---
tags:
  - workflow
erstellt: 2026-11-02
---

# Anki-Karten erstellen

Dieser Workflow beschreibt, wie du aus Notizen Anki-Karten für das Plugin [Flashcards](https://github.com/reuseman/flashcards-obsidian) in Version 2 erstellst. Das Plugin überträgt Karten aus Markdown-Dateien über AnkiConnect nach Anki. Obsidian ist die Quelle für den Inhalt, Anki verwaltet nur den Lernfortschritt. Änderungen an Karten in Anki überschreibt der nächste Abgleich.

## Kartendatei

Lege die Karten in einer eigenen Datei `<Thema> - Karteikarten.md` neben den Notizen zum Thema an. Die Datei hat diesen Aufbau:

```yaml
---
tags:
erstellt:
cards-deck: "<Bereich>::<Thema>"
---
```

- `cards-deck` legt das Zieldeck in Anki fest. `::` trennt die Ebenen. Fehlt die Property, leitet das Plugin das Deck aus dem Ordnerpfad ab.
- Das Plugin übernimmt die Einträge aus `tags` als Tags in Anki und ergänzt das Tag `obsidian`.
- Unter dem Frontmatter steht eine Checkliste der behandelten Abschnitte, darunter folgen die Karten.
- Gliedere die Karten mit Überschriften nach Abschnitten. Das Plugin zeigt die übergeordneten Überschriften in Anki als Kontext über der Frage an.

## Kartenformat

Eine Karte ist ein Absatz:

```markdown
Frage::Antwort
```

- Das Plugin wertet jeden Absatz als Karte, der außerhalb von Code und Formeln `::` enthält. Das erste `::` trennt Frage und Antwort.
- Trenne Karten durch eine Leerzeile. Zeilen ohne Leerzeile dazwischen bilden einen Absatz und damit eine einzige Karte.
- Erzwinge einen Zeilenumbruch in der Antwort mit zwei Leerzeichen am Zeilenende. Ein einfacher Zeilenumbruch wird in Anki zu einem Leerzeichen.
- Schreibe Karten nicht in Listen oder Callouts. Für Listen und Callouts gelten eigene Regeln des Plugins.
- Markdown wie Fettdruck, Inline-Code, Formeln mit `$` und Wikilinks funktioniert. `::` innerhalb von Inline-Code oder Formeln zählt nicht als Trenner.
- Vermeide `==Text==`, `{1:Text}` und `#card`. Das Plugin macht daraus zusätzliche Karten, z. B. Cloze-Karten aus Hervorhebungen.
- Schreibe keine Anker `^q-xxxx` und keine Property `flashcards`. Das Plugin trägt sie beim Abgleich selbst ein. Kopierst du eine bestehende Karte, entferne den Anker am Absatzende.
- Schreibe Beispiele für die Syntax in Notizen immer in Inline-Code oder Codeblöcke, sonst entstehen daraus Karten.

## Kartenstil

- Eine Karte fragt genau eine Sache ab.
- Die Frage ist ohne Kontext eindeutig.
- Die Antwort hat höchstens zwei Zeilen. Braucht sie mehr, teile die Karte oder lass den Rest in der Notiz.
- Nur Basic-Karten mit `::`, keine Reversed-Karten mit `:::` und keine Cloze-Karten
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
	- Keine zwei Zeilen mit `::` folgen ohne Leerzeile aufeinander. Die Ausgabe muss leer sein: `rg -nU "::[^\n]*\n[^\n]*::" "<Datei>"`.
	- Die Datei enthält keine Cloze- oder Hashtag-Syntax und keine Anker. Die Ausgabe muss leer sein: `rg -n "==|\{\d+:|\{\{c\d+::|#card|\^q-" "<Datei>"`.
	- Keine Karte nutzt `:::`.
6. **Übergabe:** Sag mir, dass ich Anki starten und in der Kartendatei den Befehl „Flashcards: Update Anki from current note“ ausführen soll. Danach muss jede Karte am Absatzende einen Anker `^q-xxxx` haben, und die Datei hat eine Property `flashcards`.

## Fallstricke

- „Flashcards: Update Anki from vault“ durchsucht ohne weitere Einstellung den ganzen Vault. Jeder Absatz mit `::` außerhalb von Code wird dabei zur Karte, z. B. auch Dataview-Felder wie `status:: offen`. Gleiche deshalb nur die aktuelle Notiz ab oder schränke den Abgleich unter „Settings → Flashcards → Sync scope“ ein.
- Ist „Settings → Flashcards → Highlight cloze“ aktiv, macht der Abgleich aus jeder Hervorhebung `==Text==` eine Cloze-Karte, auch in Notizen ohne Karteikarten wie den Leitfäden. Das Plugin schreibt dann Anker und die Property `flashcards` in diese Notizen. Lass die Einstellung deshalb ausgeschaltet.
- Anki lehnt eine Karte als Duplikat ab, wenn eine andere Karte desselben Notiztyps dieselbe Frage hat, auch in einem anderen Deck. Das Log meldet dann `cannot create note because it is a duplicate`. Formuliere die Frage um und gleiche erneut ab.
- Entfernst du eine Karte aus der Datei, fragt das Plugin vor dem Löschen in Anki nach. Bestätigst du, gehen die Karte und ihr Lernfortschritt in Anki verloren.
- Die Verknüpfung zu Anki steht in der Datei selbst, in der Property `flashcards` und in den Ankern. Bearbeite oder entferne sie nicht, sonst legt das Plugin die Karten neu an und der Lernfortschritt geht verloren.
- Verschiebst du eine Kartendatei ohne `cards-deck`, wandern die Karten beim nächsten Abgleich in das Deck des neuen Ordners.
