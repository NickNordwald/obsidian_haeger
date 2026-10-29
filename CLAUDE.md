---
anki-sync: false
---

# CLAUDE.md — Arbeits-Vault von Nick (Haeger Consulting)

Dieses Dokument beschreibt Struktur und Konventionen dieses Obsidian-Vaults für Claude/Claudian. Ziel des Vaults: Nick (Data Engineer Trainee bei [[Documentation/Haeger Consulting.md|Haeger Consulting]]) bei der Strukturierung und Dokumentation seiner Arbeitsprojekte unterstützen — mit sinnvollen Backlinks und einer konsistenten Tag-Taxonomie, damit alles verbunden und auffindbar bleibt.

**Sprache**: Notizinhalte werden auf Deutsch verfasst, auch wenn die Konversation mit dem Nutzer auf einer anderen Sprache läuft.

## Grundprinzip

Bestehende Strukturen respektieren, bevor neue erfunden werden. Vor dem Anlegen einer neuen Notiz immer kurz prüfen (Dateiname/Grep), ob es die betreffende Entität (Person, Service, Projekt, Firma) nicht schon gibt — Duplikate vermeiden, stattdessen verlinken.

Bei Unsicherheit über Fakten (z. B. Firmen-/Projektdetails) nicht raten. Entweder aktiv recherchieren (WebFetch/defuddle für öffentliche Quellen) oder beim Nutzer nachfragen — nichts Erfundenes ins Vault schreiben.

## Ordnerstruktur

| Ordner | Zweck |
|---|---|
| `Calendar/Daily/`, `Calendar/Weekly/` | Tages-/Wochennotizen (Templater-generiert, embedden `ToDo/ToDo.base`) |
| `ToDo/Current/`, `ToDo/Simmering/`, `ToDo/Sleeping/` | Task-Management — Zustand wird über den **Ordner** abgebildet, nicht über einen Tag |
| `Documentation/Services/` | Eine Notiz pro genutztem Tool/Service |
| `Documentation/People/` | Eine Notiz pro relevanter Person |
| `Documentation/` (Root) | Firmenprofile und sonstige Referenzdokumentation |
| `Projects/` | Arbeitsprojekte (siehe unten) — neu eingeführt |
| `Resources/Templater/` | Notiz-Vorlagen |
| `Resources/Bases/`, `Resources/Canvases/`, `Resources/Images/` | Base-Dateien, Canvases, Bilder |
| `Resources/Startdokumente/` | Rohe Quelldokumente (PDFs etc., z. B. vom Arbeitsrechner) — werden **nicht** dupliziert, sondern von einer passenden `Documentation/`-Notiz aus per Wikilink referenziert (siehe z. B. `Documentation/Services/PGP.md` → `Resources/Startdokumente/PGP Quick Guide1.0.pdf`) |
| `Zarchive/` | Archiv für nicht mehr aktive Inhalte |

## Task-System (`ToDo/`)

Frontmatter-Schema (siehe `Resources/Templater/Task.md`):
```yaml
importance:
type:
energy:
effort:
scheduled:
deadline:
creation:
completed:
tags:
  - "#task"
```
Der Zustand eines Tasks wird durch den **Ordner** bestimmt (`Current` = aktiv, `Simmering` = später/köchelt, `Sleeping` = pausiert/geparkt), nicht durch ein Statusfeld. `ToDo/ToDo.base` aggregiert alle drei Ordner plus eine "Today"-Ansicht. Beim Anlegen neuer Tasks dieses Schema und die `"#task"`-Tag-Schreibweise (mit Raute, als String) beibehalten — das ist eine bewusste Ausnahme von der sonstigen Tag-Konvention unten.

## Documentation-System (`Documentation/`)

- **Services** (`Documentation/Services/<Name>.md`): Tag `documentation`, kurze Erklärung was der Service kann + Anbieter-Unternehmen, plus Kurzanleitung. Vorbild: `Documentation/Services/Confluence.md`.
- **People** (`Documentation/People/<Name>.md`): Tag `person`, Frontmatter `role`, `company` (Wikilink zur Firma). Vorbild: `Documentation/People/Martin Stoller.md`.
- **Firmen/Kunden** (`Documentation/<Firma>.md`): Tag `documentation`, Frontmatter `type: company`, öffentlich recherchierbare Fakten (Branche, Standort, Größe, Website) sauber von persönlichem Kontext (eigene Rolle, Team) trennen.

## Traineeship-Kapitel (`Documentation/Traineeship/`)

Der Lernpfad ist in Bereiche (z. B. `Grundlagen der Software Entwicklung/`) und darin in nummerierte Unterkapitel gegliedert (z. B. `Relationale Datenbanken & ORM/`, `Testing mit PyTest/`). Pro Unterkapitel:

- **`X.Y <Thema> - Leitfaden.md`** — vom Unternehmen vorgegeben (Motivation, Stichworte, Kontrollfragen, Übungsaufgabe). **Nicht verändern.**
- **`X.Y <Thema> - Vertiefung.md`** — von Claudian erstellt und **Stichwort für Stichwort** ausgebaut (wächst mit, nicht auf einmal fertigstellen). Vorbild: `1.3 Testing mit PyTest - Vertiefung.md`.
- **`Anki Karteikarten.md`** — parallel zur Vertiefung gepflegt, `deck`-Frontmatter passend zur Ordnerstruktur (`Traineeship::<Bereich>::<Unterkapitel>`), Fortschritts-Checkliste oben, Karten pro behandeltem Stichwort. **Nur Basic-Karten** (Frage/Antwort mit `::`) — keine Cloze-Karten (Lückentexte). Die Kartenzahl richtet sich nach dem Stoff: **~10 Karten pro Stichwort sind der Richtwert**, bei dichten Stichworten (z. B. HTTP-Statuscodes, FastAPI) dürfen es mehr sein, bei dünnen weniger. Kein Auffüllen auf eine Zielzahl: Lieber verwandte Punkte zu einer Karte zusammenfassen; was nicht auf eine Karte passt, bleibt in der Vertiefung.
- **`Übungsaufgabe/`** (Unterordner, falls das Kapitel eine Implementierung verlangt) — `Umsetzungsstand (<Thema>).md` als laufendes Statusdokument.

### Anki-Karteikarten anlegen — verbindliches Format

> [!danger] Ein neuer Kartenblock hat KEIN `oid`-Attribut
> Nicht `oid=""`, nicht `oid="undefined"` — das Attribut fehlt schlicht. Das Sync-Plugin trägt die Kennung nach dem ersten erfolgreichen Abgleich selbst ein.
>
> **Niemals das Blockformat aus einer bestehenden Datei kopieren**, ohne den `oid`-Teil zu entfernen. Genau so ist der Fehler entstanden, der 151 Karten monatelang unbemerkt nicht nach Anki übertragen hat.

Korrekter Block:

```markdown
<!-- basicblock-start -->
Die Frage, Markdown erlaubt::
Die Antwort in **einer** Zeile, Zeilenumbrüche als `<br>`
<!-- basicblock-end -->
```

Regeln:

- Trenner ist **`::`** am Ende der Fragezeile
- Die **Antwort steht auf genau einer Zeile** — jeder echte Zeilenumbruch beendet den Block. Umbrüche, Listen und Tabellen mit `<br>` bauen
- `deck`-Frontmatter setzen: `deck: "Traineeship::<Bereich>::<Unterkapitel>"`
- Nur **Basic-Karten**, keine Cloze

Nach dem Anlegen dem Nutzer sagen: Anki starten, Sync ausführen, und in der Abschlussmeldung auf **`Failed Created Blocks: 0`** achten. Prüfung im Vault:

```bash
grep -rc 'oid="undefined"' --include='Anki Karteikarten.md' Documentation/ | grep -v ':0'
```

Kein Treffer = alles übertragen. Treffer = diese Blöcke sind gescheitert; `oid="undefined"` wieder ersatzlos entfernen und erneut abgleichen.

> [!warning] Das Plugin löscht ungefragt
> Notizen, die es in Anki findet, aber nicht mehr im Vault, werden beim nächsten Abgleich **samt Lernfortschritt gelöscht**. Vor dem Verschieben, Umbenennen oder Archivieren von Karteikarten-Dateien den Nutzer auf einen Anki-Export mit Lernfortschritt hinweisen.

Hintergrund, Plugin-Korrekturen und Fehlersuche: [[Documentation/Services/Obsidian Anki Sync.md|Obsidian Anki Sync]]. **Fünf Korrekturen an `main.js` sind angewendet** und gehen bei einem Plugin-Update verloren. Symptome danach: `TypeError … reading 'frontmatter'` in der Konsole und Karten, die nicht entstehen (Korrekturen 1–3), sowie ein einfrierendes Obsidian beim Klick auf das Anki-Symbol (Korrektur 4).

> [!danger] Ein `basicblock-start` ohne `basicblock-end` ließ Obsidian einfrieren
> Das Plugin liest **jede** Markdown-Datei des Vaults und ignoriert dabei Code-Blöcke — ein unvollständiger Beispiel-Marker in einer Doku-Notiz genügte. Korrektur 4 macht die Regex linear; Marker in Notizen trotzdem immer als vollständiges Paar schreiben.

Notizen, die das Kartenformat nur **beispielhaft zeigen** (wie diese Datei), brauchen `anki-sync: false` im Frontmatter — sonst legt das Plugin aus den Beispielen echte Karten an. Das Feld nur bei Dateien setzen, die noch **keine** synchronisierten Karten haben, sonst werden diese gelöscht.

### Workflow beim Durcharbeiten eines Unterkapitels

Wenn Nick sagt „lass uns Kapitel X.Y durcharbeiten" (oder ähnlich, ohne dass er das jedes Mal neu spezifizieren muss):

1. **Leitfaden gründlich durcharbeiten.** Jedes Stichwort ausführlich erklären (nicht nur stichpunktartig) und in die `Vertiefung`-Notiz schreiben: Hintergrund/Warum, Konzept, konkrete Beispiele (Code, wo passend), Bezug zu vorherigen Kapiteln/bereits Gelerntem, die Kontrollfragen aus dem Leitfaden explizit beantworten, Self-Check-Fragen am Ende jedes Abschnitts. Ein Inhaltsverzeichnis oben verlinkt die Abschnitte.
2. **Parallel dazu die Anki-Karteikarten pflegen** — pro behandeltem Stichwort passende Karten ergänzen (nicht erst am Kapitelende alles auf einmal).
3. **Für die Übungsaufgabe (Implementierung):** eigener Projektordner außerhalb des Vaults (unter `~/HaegerConsulting/Projects/Traineeship/...`, auch wenn git es nicht bräuchte — Nicks bewusste Präferenz). Im Vault dazu `Übungsaufgabe/Umsetzungsstand (<Thema>).md` mit Arbeitsmodus, Zielen, Stand pro Datei/Bereich, offenen Punkten, gelernten Lektionen. **Arbeitsmodus: Nick schreibt den Code selbst, Claudian reviewt/korrigiert/erklärt und leitet an — nicht die Implementierung vorwegschreiben**, außer auf explizite Ansage (z. B. mechanische Wiederholung, oder wenn Nick explizit sagt „das schreib du").
4. **Bei viel neuer Syntax/Technik auf einmal** (z. B. pytest-Fixtures, Parametrisierung, Mocking): zusätzlich einen abhakbaren **Tutorial-Style-Plan** (`Übungsaufgabe/<Thema> Plan & Syntax-Tutorial.md`) — eine Liste aller zu schreibenden Tests/Schritte von einfach nach komplex, jeder Block führt genau eine neue Technik ein (mit Kurz-Erklärung + Code-Muster), `- [ ]`-Kästchen zum Abhaken. Vorbild: `Testing mit PyTest/Übungsaufgabe/Test-Suite Plan & Syntax-Tutorial.md`.

## Projects (`Projects/`) — neu

Für jedes Arbeitsprojekt eine Hub-Notiz unter `Projects/<Projektname>.md`. Wächst ein Projekt (viele Meetings, Entscheidungen, Teilnotizen), daraus einen Unterordner `Projects/<Projektname>/` machen mit der Hub-Notiz `Projects/<Projektname>/<Projektname>.md` plus z. B. `Meetings/` und `Decisions/` als Unterordner — analog zum Muster in `Documentation/` (Services/People als Unterordner).

Frontmatter-Schema für Projekt-Hub-Notizen:
```yaml
---
tags:
  - project
status: active   # active | paused | done
client:
start:
deadline:
team: []
---
```
- `status` bildet den Zustand ab (kein Status-Tag, analog zum ToDo-System, das Zustand ebenfalls nicht über Tags löst).
- `client` verlinkt auf eine Firmen-Notiz unter `Documentation/`, falls es sich um ein Kundenprojekt handelt (client-Notizen bekommen den Tag `client`, siehe Taxonomie unten).
- `team` ist eine Liste von Wikilinks auf `Documentation/People/`-Notizen der Projektbeteiligten.

Eine Projekt-Notiz sollte immer verlinken auf: beteiligte Personen, genutzte Services (falls projektspezifische Tools zum Einsatz kommen), die Kunden-/Auftraggeber-Notiz (falls vorhanden) und relevante ToDo-Items.

## Tag-Taxonomie (verbindlich)

Nur diese Tags verwenden — keine Ad-hoc-Tags erfinden. Falls ein neuer Tag wirklich nötig erscheint: den Nutzer fragen und danach hier ergänzen.

| Tag | Verwendung |
|---|---|
| `project` | Projekt-Hub-Notizen unter `Projects/` |
| `documentation` | Referenz-/Wissensnotizen (Services, Firmenprofile, How-tos) |
| `person` | Personen-Notizen unter `Documentation/People/` |
| `client` | Kunden/externe Auftraggeber-Organisationen (Firmen-Notizen, die kein Arbeitgeber sind) |
| `meeting` | Meeting-Notizen (i. d. R. innerhalb eines Projekts) |
| `decision` | Entscheidungs-Dokumentation (i. d. R. innerhalb eines Projekts) |
| `"#task"` | Einzelne ToDo-Items — Ausnahme, siehe Task-System oben |

Tags gehören ins YAML-Frontmatter unter `tags:` als Liste, ohne führende Raute (Ausnahme: `"#task"` s. o.).

## Verlinkung (Backlinks) — zentrales Prinzip

- Immer Wikilinks `[[...]]` statt Klartext-Erwähnung verwenden, sobald eine Entität (Person, Service, Firma, Projekt) bereits eine Notiz hat oder eine verdient.
- Proaktiv verlinken, nicht nur auf Nachfrage: Beim Anlegen/Bearbeiten einer Notiz aktiv nach passenden bestehenden Notizen suchen (z. B. per Grep über den Vault) und verknüpfen — auch wenn der Nutzer es nicht explizit erwähnt.
- Referenz-Notizen (Person, Service, Client, Projekt) bekommen wo sinnvoll einen Rücklink-Abschnitt zur übergeordneten Übersicht, siehe Vorbild `Documentation/Websites und Services.md` ↔ `Documentation/Services/*.md`.
- Vor dem Anlegen einer neuen Person-/Service-/Firmen-Notiz immer prüfen, ob sie nicht schon existiert (Dateiname-Suche), um Duplikate und getrennte Wissensinseln zu vermeiden.

## Namenskonventionen

- Dateinamen in Title Case, Leerzeichen erlaubt, auf Deutsch (Vorbild: `Websites und Services.md`, `Martin Stoller.md`).
- Ordnernamen ebenso in Title Case.
- Bestehende Frontmatter-Felder nicht grundlos entfernen oder umbenennen.

## Verhalten

- Bei mehrdeutigen Strukturentscheidungen (neuer Tag, neue Top-Level-Struktur, Umbenennung bestehender Notizen) lieber nachfragen als eigenmächtig entscheiden — solche Änderungen sind später schwer rückgängig zu machen.
- Innerhalb der hier definierten Konventionen (Tags aus der Taxonomie, Verlinkung, Frontmatter-Schemata) darf proaktiv und ohne Rückfrage gehandelt werden.
