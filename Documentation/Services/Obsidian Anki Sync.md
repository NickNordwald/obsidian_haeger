---
tags:
  - documentation
anki-sync: false
---
# Obsidian Anki Sync

Obsidian-Plugin, das Karteikarten aus Markdown-Notizen nach **Anki** überträgt. Karten werden direkt in den Fachnotizen als HTML-Kommentar-Blöcke geschrieben; das Plugin erkennt sie, legt sie in Anki an und schreibt eine Kennung zurück in die Datei.

Bereitgestellt von: **debanjandhar12** ([GitHub](https://github.com/debanjandhar12/Obsidian-Anki-Sync)) · Version 1.0.0 · installiert unter `.obsidian/plugins/ObsidianAnkiSync/`

Voraussetzung: **Anki muss laufen** und das Add-on **AnkiConnect** installiert haben. Ausgelöst wird der Abgleich über den Befehl **„Start Obsidian Anki Sync"** oder das Anki-Symbol in der Ribbon-Leiste.

Übersicht aller Dienste: [[Documentation/Websites und Services|Websites und Services]]

Im Vault genutzt für die Karteikarten des [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Kapitelabschluss - Q&A-Ernte und blinde Flecken|Traineeships]] — rund 900 Karten über alle Kapitel.

## Das Kartenformat

> [!danger] Die wichtigste Regel
> **Ein neuer Block hat KEIN `oid`-Attribut.** Nicht `oid=""`, nicht `oid="undefined"` — das Attribut fehlt schlicht. Das Plugin trägt die Kennung nach dem ersten erfolgreichen Abgleich selbst ein.

```markdown
<!-- basicblock-start -->
Die Frage, Markdown erlaubt::
Die Antwort in **einer** Zeile, Zeilenumbrüche als `<br>`
<!-- basicblock-end -->
```

- **Trenner ist `::`** am Ende der Fragezeile
- **Die Antwort steht auf einer Zeile** — Umbrüche mit `<br>`, weil ein echter Zeilenumbruch den Block beendet
- **Zieldeck** kommt aus dem Frontmatter der Datei: `deck: "Traineeship::Bereich::Unterkapitel"` (Ankis Deck-Trenner ist ebenfalls `::`)
- Nach dem Abgleich trägt die Startzeile zusätzlich die Kennung, z. B. `oid="ObsB8YxPlS8emdeS0uHlXyCK"` (der Marker selbst wird hier bewusst nicht noch einmal vollständig ausgeschrieben — siehe „Ein offener Startmarker friert Obsidian ein" unten)

Nur **Basic-Karten** verwenden, keine Cloze — siehe Vault-`CLAUDE.md`.

## Was schiefgehen kann

### Ein offener Startmarker friert Obsidian ein

Das gefährlichste Muster, weil es nicht nach einem Plugin-Fehler aussieht: **Obsidian reagiert nicht mehr, sobald man das Anki-Symbol anklickt.**

Ursache war eine Regex mit verschachtelten Quantoren (`(\n| (\n|.)*?)*?`), mit der das Plugin die Kartenblöcke sucht. Findet sie ein `basicblock-start` ohne zugehöriges `basicblock-end`, probiert die Regex-Engine exponentiell viele Zerlegungen des restlichen Dateitexts durch — der Obsidian-Prozess läuft auf 100 % CPU und wirkt wie abgestürzt.

Entscheidend: Der offene Marker musste **nicht** in einer Karteikarten-Datei stehen. Das Plugin liest *jede* Markdown-Datei des Vaults und ignoriert dabei Code-Blöcke. Ein Startmarker, der nur zur Veranschaulichung in einer Doku-Notiz stand, genügte.

Seit Korrektur 4 ist die Regex linear und kann nicht mehr blockieren. Ein offener Marker ergibt dann schlicht keine Karte. Zum Aufspüren unbalancierter Marker:

```bash
grep -rlZ 'basicblock-start' --include='*.md' . | while IFS= read -r -d '' f; do
  s=$(grep -c 'basicblock-start' "$f"); e=$(grep -c 'basicblock-end' "$f")
  [ "$s" = "$e" ] || printf 'UNBALANCIERT start=%s end=%s  %s\n' "$s" "$e" "$f"
done
```

### `oid="undefined"` — Blöcke werden nie angelegt

Der häufigste und tückischste Fehler. Das Plugin fragt Anki nach einer Notiz mit dieser Kennung, **findet die Geisternotiz mit dem Wert „undefined"** und geht in den Update-Zweig statt in den Anlege-Zweig. Ergebnis: „x updated, 0 created", das Deck bleibt leer, und alle betroffenen Blöcke überschreiben nacheinander dieselbe eine Notiz.

Symptom in der Statuszeile: viele Updates, keine Neuanlagen.
Behebung: Attribut ersatzlos entfernen, dann neu abgleichen.

### Das Plugin löscht ungefragt

```js
// Delete anki notes created by app which are no longer in obsidian vault
if (!blockIds.includes(ankiId)) { await deteteNote(ankiId); }
```

Jede Notiz, die das Plugin in Anki findet, aber **nicht mehr im Vault**, wird beim nächsten Abgleich **gelöscht — samt Lernfortschritt**. Betroffen sind alle Notizen mit dem Modell `ObsidianAnkiSyncModel` und dem Vault-Tag.

> [!warning] Vor jeder größeren Umstellung sichern
> Karteikarten-Dateien verschieben, umbenennen, archivieren oder Blöcke auskommentieren löscht die zugehörigen Karten. Vorher in Anki: **Datei → Exportieren → Anki-Deck-Paket**, mit **„Lernfortschritt einschließen"**.
>
> Die automatische Sicherung des Plugins läuft nur, wenn in den Einstellungen `backup` aktiv ist — Standard ist **aus**.

### Karten dauerhaft aus dem Abgleich nehmen

Das Plugin findet seine Notizen über `tag:<Vaultname> note:ObsidianAnkiSyncModel tag:ObsidianAnkiSync`. Entfernt man in **Anki** den Tag `ObsidianAnkiSync` von einer Notiz, ist sie für das Plugin unsichtbar: kein Update, keine Löschung, Lernfortschritt bleibt. **Erst den Tag entfernen, dann die Blöcke im Vault deaktivieren** — in dieser Reihenfolge.

## Angewendete Korrekturen an `main.js`

Fünf Korrekturen. 1–3 betreffen Fehler, die erst ab einigen hundert Karten sichtbar werden; 4–5 kamen nach dem Einfrier-Vorfall dazu.

Sicherungen im Plugin-Ordner:
- `main.js.original-backup` — unveränderter Originalstand
- `main.js.pre-regex-hardening` — Stand mit den Korrekturen 1–3

Wiederherstellung jeweils durch Zurückkopieren und Neuladen von Obsidian.

> [!important] Bei einem Plugin-Update sind die Korrekturen weg
> Nach jedem Update prüfen und erneut anwenden. Symptom, falls vergessen: `TypeError … reading 'frontmatter'` in der Konsole und Karten, die nicht angelegt werden.

**1. Fehlendes `await` in `addNote`**

```js
updateNote(ankiId, deckName, modelName, fields, tags);       // vorher
await updateNote(ankiId, deckName, modelName, fields, tags); // nachher
```

Ohne `await` feuert das Plugin pro neuer Karte einen unbeaufsichtigten Hintergrundvorgang mit je fünf AnkiConnect-Aufrufen ab. Bei vielen Karten überlastet das AnkiConnect, das die Anfragen einzeln abarbeitet.

**2. `clearUnusedTags` aus der Pro-Notiz-Schleife entfernt**

Diese Wartungsoperation läuft über die **gesamte Sammlung** und wurde für **jede einzelne Notiz** aufgerufen — bei 900 Karten 900 Volldurchläufe. Der Abgleich kam dadurch nie zu Ende. Folge des Entfernens: Unbenutzte Tags bleiben in Ankis Tag-Liste stehen; aufräumen mit *Extras → Datenbank überprüfen*.

**3. Ungeprüfte Dereferenzierung in `getDocYAMLProp`** — der eigentliche Fehler

```js
let fileCache = this.metadataCache.getFileCache(this.file).frontmatter;
```

Schreibt das Plugin eine neue `oid` in die Datei, verwirft Obsidian deren Metadaten-Cache. `getFileCache()` liefert dann kurzzeitig `null`, und der nächste Block stürzt beim Lesen des `deck`-Frontmatters ab. Im Log erkennbar als Schwall von `TypeError`s direkt nach jeder angelegten Karte.

Die Korrektur prüft auf `null` **und** merkt sich das Frontmatter pro Datei — sonst landeten Karten während der Neuindizierung im falschen Deck.

**4. Verschachtelte Quantoren in den Block-Regexes entfernt**

```js
/<!--(\t|\n| )*?basicblock-start(\n| (\n|.)*?)*?-->(\n|.)*?.../gi  // vorher
/<!--[\t\n ]*basicblock-start[^>]*?-->[\s\S]*?.../gi               // nachher
```

Betrifft alle drei Block-Arten (`basic`, `cloze`, `replace`), die Startmarker-Regex in `updateOIdinObsidian` und die Kommentar-Regexes. `[^>]*?` kann den öffnenden Kommentar nicht überschreiten, wodurch das Backtracking linear bleibt. Über alle 1018 Blöcke des Vaults gegen die alte Regex geprüft: **identische Treffer, null Abweichungen** — reiner Drop-in-Ersatz.

> [!danger] Die Kommentar-Regex braucht ihre Capture-Gruppe
> `getAttribInCommentLine` liest **`matches[0][1]`**, also Gruppe 1 — den Inhalt zwischen `<!--` und `-->`, aus dem das Attribut geparst wird. Die Klammern sind daher Pflicht:
>
> ```js
> const CommentsRegExp = /<!--([\s\S]*?)-->/gi;   // richtig
> const CommentsRegExp = /<!--[\s\S]*?-->/gi;     // TypeError: … reading 'trim'
> ```
>
> Beim ersten Anlauf der Härtung fehlten die Klammern — der Abgleich brach sofort nach „Recognized Blocks" mit `Cannot read properties of undefined (reading 'trim')` ab. Die anderen vier Fundstellen der Regex nutzen nur `match[0]` bzw. `replaceAll`, dort ist die Gruppe wirkungslos, aber unschädlich.

**5. Opt-out per Frontmatter (`anki-sync: false`)**

Neu eingeführt, kein Bugfix am Original. Dateien mit

```yaml
anki-sync: false
```

werden vom Abgleich ausgenommen. Gedacht für Notizen, die das Kartenformat nur *zeigen* — ohne das Opt-out hätte das Plugin aus den Beispielblöcken dieser Notiz und aus `CLAUDE.md` echte Karten angelegt und beiden Dateien eine `oid` hineingeschrieben. Der Filter greift in **beiden** Schleifen (Einsammeln *und* Löscherkennung).

> [!warning] Nicht nachträglich auf bestehende Karten anwenden
> Bekommt eine Datei mit **bereits synchronisierten** Karten das Opt-out, gelten diese Karten als „nicht mehr im Vault" und werden beim nächsten Abgleich samt Lernfortschritt gelöscht. Vorher der Weg über den Anki-Tag, siehe *Karten dauerhaft aus dem Abgleich nehmen*.

## Fehlersuche

Die Meldung oben rechts zeigt nur die Zusammenfassung. Die echten Fehler stehen in der **Entwicklerkonsole**: `Strg+Shift+I` → Reiter **Console**, nach `TypeError` filtern.

Die Abschlusszeile eines sauberen Laufs:

```
Sync Completed!
Created Blocks: n  Updated Blocks: m  Deleted Blocks: 0
Failed Created Blocks: 0  Failed Updated Blocks: 0
```

Offene Blöcke im Vault zählen:

```bash
grep -rc 'oid="undefined"' --include='Anki Karteikarten.md' Documentation/ | grep -v ':0'
```

Kein Treffer heißt: alles übertragen.

## Eigenheiten

- **Keine Änderungserkennung** — jeder Lauf verarbeitet *alle* Karten neu, mit mehreren AnkiConnect-Aufrufen pro Karte. Die Dauer wächst linear mit der Gesamtzahl.
- **Kein Ordnerfilter, kein Einzeldatei-Abgleich** — es gibt nur „alles abgleichen".
- **Die Blockmarker sind HTML-Kommentare** und damit in der Leseansicht unsichtbar. Wer dort nachschaut, hält sie fälschlich für fehlend — im Quelltextmodus sind sie da.
