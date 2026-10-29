---
tags:
  - documentation
---

# Git von innen — ein Durchlauf

Ein kleines Projekt von `git init` bis zur Zusammenarbeit mit einer Kollegin. Nach jedem Befehl steht genau, **welche Dateien unter `.git/` neu entstehen, sich ändern oder verschwinden**, und warum.

Das ist kein ausgedachtes Beispiel. Alle Ausgaben stammen aus einem echten Durchlauf mit Git 2.53. Nach jedem Schritt wurde `.git/` mit dem Stand davor verglichen. Weil Name und Zeitstempel fest gesetzt waren, bekommst du beim Nachspielen **exakt dieselben Hashes** (Skript am Ende).

Verwendet werden nur die wichtigsten Befehle: `init`, `status`, `add`, `commit`, `switch`, `merge`, `branch -d`, `remote`, `push`, `fetch` und die zweite Hälfte von `pull`. Was jeder davon allgemein tut, steht in der [[Git Befehlsreferenz]].

## Inhalt

1. [[#1. git init — das leere Repo|git init]]
2. [[#2. Dateien anlegen — Git merkt nichts|Dateien anlegen]]
3. [[#3. git add — Blobs und Index|git add]]
4. [[#4. git commit — Trees, Commit, Branch|Der erste Commit]]
5. [[#5. Eine Datei ändern — unstaged, staged, committed|Eine Datei ändern]]
6. [[#6. git switch -c — ein Branch entsteht|Einen Branch anlegen]]
7. [[#7. Ein Commit auf dem Feature-Branch|Commit auf dem Feature-Branch]]
8. [[#8. Zurück auf main — und dort weiterarbeiten|Zurück auf main]]
9. [[#9. git merge — zwei Linien werden eins|git merge]]
10. [[#10. Den Branch löschen|Den Branch löschen]]
11. [[#11. git remote und git push — ein Server kommt dazu|remote und push]]
12. [[#12. Die Kollegin pusht, wir holen mit git fetch|fetch]]
13. [[#13. Fast-Forward — der zweite Teil von git pull|Fast-Forward]]
14. [[#Gesamtbild|Gesamtbild und Zusammenfassung]]

> [!info] Lesehilfe
> - **NEU / GEÄNDERT / GELÖSCHT** sind die echten Unterschiede in `.git/` gegenüber dem Schritt davor. Die Beispiel-Hooks in `.git/hooks/` sind ausgeblendet.
> - Hashes werden im Text auf **7 Zeichen** gekürzt (`03f3189`). Git akzeptiert diese Kurzform überall, solange sie eindeutig ist.
> - Ein Objekt liegt unter `.git/objects/<erste 2 Zeichen>/<restliche 38>`. Aus `03f3189…` wird also `.git/objects/03/f3189…`.

---

## 1. git init — das leere Repo

```bash
git init -b main preisimport && cd preisimport
```

```
NEU      .git/config
NEU      .git/description
NEU      .git/HEAD
NEU      .git/info/exclude
```

Dazu kommen die leeren Ordner `objects/`, `refs/heads/` und `refs/tags/` (leere Ordner tauchen im Vergleich nicht auf).

| Datei | Inhalt | Wozu |
|---|---|---|
| `HEAD` | `ref: refs/heads/main` | „Ich stehe auf Branch `main`.“ |
| `config` | `[core]`-Abschnitt | Repo-Einstellungen. Hier landen später Remotes und Tracking |
| `info/exclude` | nur Kommentare | Wie `.gitignore`, aber lokal und nicht versioniert |
| `description` | Platzhaltertext | Nur für das alte Web-Interface `gitweb`, praktisch bedeutungslos |

**Was es noch *nicht* gibt** — und das ist der interessante Teil:

- **`refs/heads/main` existiert nicht.** `HEAD` zeigt also auf einen Branch, den es noch gar nicht gibt. Git nennt das einen **unborn branch**. Deshalb meldet `git status` „No commits yet“.
- **`index` existiert nicht.** Es gibt noch nichts zu stagen.
- **`objects/` ist leer.**

---

## 2. Dateien anlegen — Git merkt nichts

```
preisimport/
├── README.md
└── src/
    └── loader.py
```

```python
# src/loader.py
def load(rows):
    for row in rows:
        insert(row["sku"], row["price"])
```

```
$ git status
Untracked files:
	README.md
	src/
```

**In `.git/` ändert sich nichts**, auch nicht durch `git status`. Untracked heißt genau das: Die Dateien stehen nicht im Index (den es noch nicht gibt), und Git hat keine Objekte für sie. `status` weiß von ihnen nur, weil es das Arbeitsverzeichnis durchsucht und mit dem Index vergleicht.

Beachte: `status` zeigt `src/` als **Ordner**, nicht `src/loader.py`. Solange in einem Ordner nichts getrackt ist, fasst Git ihn zusammen.

---

## 3. git add — Blobs und Index

```bash
git add .
```

```
NEU      .git/index
NEU      .git/objects/0a/eb68f425750f6e1ceca422e82ad2b92472475a    ← Blob: Inhalt von loader.py
NEU      .git/objects/4b/ed18268698d70efd639c726294a6ed102ae6e2    ← Blob: Inhalt von README.md
```

**Zwei Blobs**, einer pro Datei. Ein Blob enthält nur den *Inhalt*, keinen Namen. Woher der Name kommt, zeigt der frisch angelegte Index:

```
$ git ls-files --stage
100644 4bed18268698d70efd639c726294a6ed102ae6e2 0	README.md
100644 0aeb68f425750f6e1ceca422e82ad2b92472475a 0	src/loader.py
```

Eine **flache** Liste: Modus, Blob-Hash, Stage-Nummer (0 = kein Konflikt), voller Pfad. Für `src/` gibt es **keinen eigenen Eintrag** und **noch keinen Tree**. Ordner existieren hier nur als Teil der Pfade.

```
$ git status
Changes to be committed:
	new file:   README.md
	new file:   src/loader.py
```

Warum „new file“? `status` vergleicht den Index mit dem Tree des letzten Commits. `HEAD` → `refs/heads/main` → existiert nicht → Git vergleicht mit einem **leeren** Tree, also ist jede Datei neu.

> [!tip] Der Inhalt ist schon gesichert
> Ab diesem Moment liegt der Inhalt beider Dateien in `.git/objects`. Löschst du jetzt versehentlich `loader.py`, holt `git restore src/loader.py` sie aus dem Blob zurück, obwohl es noch keinen einzigen Commit gibt.

---

## 4. git commit — Trees, Commit, Branch

```bash
git commit -m "Add loader and README"
```

```
NEU      .git/objects/99/cca54e82c33ffa9d00b332c545da492ecf5778    ← Tree für src/
NEU      .git/objects/00/98e0294448b9e2b1221648e9b01c375e75a9f8    ← Tree für das Wurzelverzeichnis
NEU      .git/objects/03/f31890bef427f0e756bc6a39d15fe112551313    ← Commit
NEU      .git/refs/heads/main                                       ← der Branch wird geboren
NEU      .git/logs/HEAD                                             ← Reflog von HEAD
NEU      .git/logs/refs/heads/main                                  ← Reflog von main
NEU      .git/COMMIT_EDITMSG                                        ← die letzte Message
GEÄNDERT .git/index
```

Der Reihe nach, so wie Git es tut:

**Schritt 1: Trees aus dem Index bauen, von unten nach oben.** Zuerst den tiefsten Ordner, denn der übergeordnete Tree braucht dessen Hash:

```
$ git cat-file -p 99cca54          # Tree für src/
100644 blob 0aeb68f…	loader.py

$ git cat-file -p 0098e02          # Tree für die Wurzel
100644 blob 4bed182…	README.md
040000 tree 99cca54…	src          ← verweist auf den src-Tree
```

Aus dem flachen Index (`src/loader.py`) ist eine Baumstruktur geworden. Hier steht der Name `loader.py`, der im Blob fehlt, und `040000` ist der Modus für „Unterordner“.

**Schritt 2: Das Commit-Objekt schreiben.**

```
$ git cat-file -p 03f3189
tree 0098e0294448b9e2b1221648e9b01c375e75a9f8
author Nick <nick@example.com> 1792656600 +0200
committer Nick <nick@example.com> 1792656600 +0200

Add loader and README
```

Es zeigt auf **einen** Tree, den der Wurzel. **Keine `parent`-Zeile**, das ist der allererste Commit. Kein Diff, kein Branch-Name.

**Schritt 3: Den Branch setzen.**

```
$ cat .git/refs/heads/main
03f31890bef427f0e756bc6a39d15fe112551313

$ cat .git/HEAD
ref: refs/heads/main                    ← unverändert seit git init!
```

`refs/heads/main` wird **angelegt**. `HEAD` wird nicht angefasst. Er zeigte schon die ganze Zeit auf `main`, nur dass es `main` jetzt auch gibt. Die Kette `HEAD → main → 03f3189 → 0098e02 → …` ist zum ersten Mal vollständig auflösbar.

**Schritt 4: Protokollieren.** Beide Reflogs bekommen ihre erste Zeile:

```
0000000000000000000000000000000000000000 03f31890… Nick <nick@example.com> 1792656600 +0200	commit (initial): Add loader and README
```

Alter Wert (Nullen = „gab es nicht“), neuer Wert, wer, wann, warum.

> [!question] Warum ist `index` GEÄNDERT?
> Die **Einträge** sind identisch, `git ls-files --stage` zeigt dieselben zwei Zeilen wie vorher. Der Index enthält aber zusätzlich eine Erweiterung, den *Cache-Tree*: Er merkt sich, welcher Tree-Hash zu welchem Ordner gehört. Beim nächsten Commit muss Git dann nur die Trees der geänderten Ordner neu berechnen. Diese Erweiterung hat der Commit gerade befüllt. Inhaltlich gilt weiter: **Der Index bleibt nach dem Commit, wie er war, und ist jetzt gleich dem `HEAD`-Tree.**

```
$ git status
nothing to commit, working tree clean
```

**Bestand:** 5 Objekte (2 Blobs, 2 Trees, 1 Commit), 1 Branch.

```
main ──► 03f3189 ──► tree 0098e02 ─┬─► blob 4bed182  (README.md)
                                   └─► tree 99cca54 ──► blob 0aeb68f  (loader.py)
```

---

## 5. Eine Datei ändern — unstaged, staged, committed

### 5a. Ändern

```python
# src/loader.py
def load(rows):
    for row in rows:
        if row["price"] is None:
            continue
        insert(row["sku"], row["price"])
```

```
$ git status --short
 M src/loader.py
```

**In `.git/` ändert sich nichts.** Das ist die genaue Bedeutung von *unstaged*: Die Änderung existiert **nur** im Arbeitsverzeichnis. Es gibt keinen Blob für den neuen Inhalt. `status` erkennt die Änderung, weil sich Größe und Änderungszeit der Datei von den Werten im Index unterscheiden. Erst dann hasht es den Inhalt zur Bestätigung.

> [!warning] Das macht `git restore` so endgültig
> `git restore src/loader.py` würde jetzt den Blob `0aeb68f` aus dem Index zurück auf die Platte schreiben. Für die neue Fassung gibt es kein Objekt und keinen Reflog-Eintrag, sie wäre unwiederbringlich weg.

### 5b. Stagen

```bash
git add src/loader.py
```

```
NEU      .git/objects/47/e6d10215348e2b880b8caf6a8ccfbe27d2e909    ← Blob: neue Fassung von loader.py
GEÄNDERT .git/index
```

```
$ git ls-files --stage
100644 4bed18268698d70efd639c726294a6ed102ae6e2 0	README.md        ← unverändert
100644 47e6d10215348e2b880b8caf6a8ccfbe27d2e909 0	src/loader.py    ← zeigt jetzt auf den neuen Blob

$ git status --short
M  src/loader.py
```

Das `M` ist von der zweiten in die erste Spalte gewandert. Die erste Spalte vergleicht `HEAD` ↔ Index, die zweite Index ↔ Arbeitsverzeichnis. Der alte Blob `0aeb68f` bleibt bestehen, denn der Commit `03f3189` braucht ihn weiterhin.

### 5c. Committen

```bash
git commit -m "loader: skip rows without price"
```

```
NEU      .git/objects/f2/88fb4c60f88fb7133aec0655164d8a60d54a98    ← neuer Tree für src/
NEU      .git/objects/b2/802fbe1c738be038fb7fb417d706a3e45e1693    ← neuer Wurzel-Tree
NEU      .git/objects/d8/19102cb2fd79457d42dd993caa55ca781daf61    ← Commit
GEÄNDERT .git/refs/heads/main
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/logs/refs/heads/main
GEÄNDERT .git/COMMIT_EDITMSG
GEÄNDERT .git/index
```

Diesmal entstehen **kein neuer Blob** (den gab es seit 5b schon) und nur die Trees **auf dem Pfad von der geänderten Datei zur Wurzel**:

```
$ git cat-file -p b2802fb          # neuer Wurzel-Tree
100644 blob 4bed182…	README.md    ← DERSELBE Blob wie im ersten Commit
040000 tree f288fb4…	src          ← neuer src-Tree

$ git cat-file -p d819102
tree b2802fbe1c738be038fb7fb417d706a3e45e1693
parent 03f31890bef427f0e756bc6a39d15fe112551313     ← jetzt mit Vorgänger
author Nick <nick@example.com> 1792657200 +0200
committer Nick <nick@example.com> 1792657200 +0200

loader: skip rows without price
```

`README.md` hat sich nicht geändert, also verweist der neue Wurzel-Tree auf **denselben** Blob `4bed182`. Jeder Commit ist ein vollständiger Snapshot, aber Unverändertes wird nie doppelt gespeichert.

`refs/heads/main` wird **überschrieben**, aus `03f3189…` wird `d819102…`. Die Historie bleibt trotzdem erhalten, und zwar über die `parent`-Zeile im neuen Commit, nicht in der Ref. Der Reflog von `main` hat jetzt zwei Zeilen:

```
0000000… 03f3189… … commit (initial): Add loader and README
03f3189… d819102… … commit: loader: skip rows without price
```

**Bestand:** 9 Objekte (3 Blobs, 4 Trees, 2 Commits).

```
main ──► d819102 ──parent──► 03f3189
```

---

## 6. git switch -c — ein Branch entsteht

Die Währungsumrechnung soll auf einem eigenen Branch entstehen:

```bash
git switch -c feature/waehrung
```

```
NEU      .git/refs/heads/feature/waehrung
NEU      .git/logs/refs/heads/feature/waehrung
GEÄNDERT .git/HEAD
GEÄNDERT .git/logs/HEAD
```

```
$ cat .git/refs/heads/feature/waehrung
d819102cb2fd79457d42dd993caa55ca781daf61     ← derselbe Commit wie main

$ cat .git/HEAD
ref: refs/heads/feature/waehrung
```

Das ist alles, was `switch -c` tut:
1. Es legt eine **41-Byte-Datei** mit dem aktuellen Commit-Hash an. Das ist `git branch feature/waehrung`.
2. Es lässt **`HEAD` auf sie zeigen.** Das ist `git switch feature/waehrung`.

**Keine Objekte, kein Index, kein Arbeitsverzeichnis.** Beide Branches zeigen auf denselben Commit, also gibt es nichts umzuschreiben.

Zwei Details:
- Der **Schrägstrich im Namen** wird zum Unterordner: `refs/heads/feature/waehrung`. Deshalb kann es nicht gleichzeitig einen Branch `feature` und einen Branch `feature/waehrung` geben. `feature` müsste eine Datei *und* ein Ordner sein.
- Der Reflog von `HEAD` notiert den Wechsel (`checkout: moving from main to feature/waehrung`). Daher weiß `git switch -`, wohin „zurück“ geht.

---

## 7. Ein Commit auf dem Feature-Branch

```python
# src/currency.py
def to_eur(amount, currency, rates):
    return amount * rates[currency]
```

```bash
git add src/currency.py
git commit -m "currency: add conversion to EUR"
```

`add`:
```
NEU      .git/objects/56/f30a4caa6ac72137d5eaaf2d8192a6fd6aaf73    ← Blob currency.py
GEÄNDERT .git/index
```

`commit`:
```
NEU      .git/objects/25/ff708952de8dab50e8ba98cc85addb71057f40    ← Tree src/ (jetzt mit 2 Dateien)
NEU      .git/objects/da/1a8bf443f604c8d5fa4aac9edf902da4e7525f    ← Wurzel-Tree
NEU      .git/objects/92/21c85359199896a73d871654d4fcb81985ac6e    ← Commit
GEÄNDERT .git/refs/heads/feature/waehrung                           ← NICHT main!
GEÄNDERT .git/logs/refs/heads/feature/waehrung
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/COMMIT_EDITMSG
GEÄNDERT .git/index
```

```
$ git cat-file -p 25ff708          # Tree src/
100644 blob 56f30a4…	currency.py
100644 blob 47e6d10…	loader.py    ← derselbe Blob wie in main
```

**Entscheidend ist, welche Ref sich geändert hat:** `refs/heads/feature/waehrung`. `refs/heads/main` steht unverändert auf `d819102`. Git hat `HEAD` aufgelöst (`ref: refs/heads/feature/waehrung`) und **diesen** Branch weitergesetzt.

```
* 9221c85 currency: add conversion to EUR     ← feature/waehrung
* d819102 loader: skip rows without price     ← main
* 03f3189 Add loader and README
```

Noch ist die Historie linear. `main` ist einfach ein Commit zurück.

---

## 8. Zurück auf main — und dort weiterarbeiten

### 8a. Wechseln

```bash
git switch main
```

```
GEÄNDERT .git/HEAD          ← ref: refs/heads/main
GEÄNDERT .git/index         ← Eintrag für src/currency.py entfernt
GEÄNDERT .git/logs/HEAD
```

**Diesmal ändern sich Index und Arbeitsverzeichnis.** Git vergleicht den Tree von `main` mit dem aktuellen Stand und bringt beide in Einklang:

```
$ ls src
loader.py                   ← currency.py ist von der Platte verschwunden

$ git ls-files --stage
100644 4bed182… 0	README.md
100644 47e6d10… 0	src/loader.py
```

`currency.py` ist nicht verloren. Der Blob `56f30a4` liegt weiter in `objects/` und ist über `feature/waehrung` erreichbar. Er gehört nur nicht zum Snapshot von `main`. Keine Ref hat sich bewegt, kein Objekt ist entstanden.

### 8b. Parallel auf main committen

Während das Feature wartet, wird auf `main` die README ergänzt:

```bash
# README.md: Zeile "Zeilen ohne Preis werden übersprungen." ergänzt
git add README.md
git commit -m "docs: mention skipped rows in README"
```

```
NEU      .git/objects/85/9c2a6d76006c6cc2f02419842ebd56e0a12bda    ← Blob README v2
NEU      .git/objects/a0/f5f8060b4f56f64dae223c62f23933d4045d7f    ← Wurzel-Tree
NEU      .git/objects/99/62756fab5164a992ed2312faf8b935419b1fb9    ← Commit
GEÄNDERT .git/refs/heads/main
…
```

Nur **ein** neuer Tree: `src/` hat sich nicht geändert, also zeigt der neue Wurzel-Tree auf den alten src-Tree `f288fb4`.

**Jetzt laufen die Branches auseinander.** Beide haben einen Commit, den der andere nicht hat:

```
* 9962756 docs: mention skipped rows in README     ← main
| * 9221c85 currency: add conversion to EUR        ← feature/waehrung
|/
* d819102 loader: skip rows without price          ← gemeinsamer Vorfahre (merge-base)
* 03f3189 Add loader and README
```

Allein ist das selten, im Team ist es der Normalfall. Genau dafür gibt es `merge` und `rebase`.

---

## 9. git merge — zwei Linien werden eins

```bash
git merge feature/waehrung
```

```
NEU      .git/objects/52/15efc7d2e2cefccb1d0660847d3e4553f1c3ca    ← zusammengeführter Wurzel-Tree
NEU      .git/objects/4e/b51f9ed732f1854425551010e4b1b66d373a8e    ← Merge-Commit
NEU      .git/ORIG_HEAD                                             ← Stand von main VOR dem Merge
GEÄNDERT .git/refs/heads/main
GEÄNDERT .git/index
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/logs/refs/heads/main
```

**So kommt Git zum Ergebnis:**

1. **Basis finden:** `git merge-base main feature/waehrung` liefert `d819102`.
2. **Drei-Wege-Vergleich pro Datei** zwischen Basis, `main` und `feature`:

   | Datei | Basis `d819102` | `main` `9962756` | `feature` `9221c85` | Ergebnis |
   |---|---|---|---|---|
   | `README.md` | `4bed182` | **`859c2a6`** (geändert) | `4bed182` | nur `main` hat geändert → `859c2a6` |
   | `src/loader.py` | `47e6d10` | `47e6d10` | `47e6d10` | unverändert |
   | `src/currency.py` | — | — | **`56f30a4`** (neu) | nur `feature` hat sie → übernehmen |

   Es hat nie dieselbe Stelle auf beiden Seiten geändert, also gibt es **keinen Konflikt**.
3. **Trees bauen:** Der neue src-Tree wäre identisch mit dem von `feature` (`25ff708`), also wird er **wiederverwendet**. Neu ist nur der Wurzel-Tree:
   ```
   $ git cat-file -p 5215efc
   100644 blob 859c2a6…	README.md    ← von main
   040000 tree 25ff708…	src          ← von feature
   ```
4. **Merge-Commit mit zwei Eltern:**
   ```
   $ git cat-file -p 4eb51f9
   tree 5215efc7d2e2cefccb1d0660847d3e4553f1c3ca
   parent 9962756fab5164a992ed2312faf8b935419b1fb9    ← 1. Parent: main (wo du standest)
   parent 9221c85359199896a73d871654d4fcb81985ac6e    ← 2. Parent: feature (was reinkam)
   …
   Merge branch 'feature/waehrung'
   ```
   Die zwei `parent`-Zeilen **sind** der Merge. Sonst unterscheidet sich das Objekt durch nichts von einem normalen Commit. Die Reihenfolge zählt: `HEAD^1` ist immer die Seite, auf der du standest. Deshalb zeigt `git log --first-parent` die „Hauptlinie“ von `main`.
5. **`main` weitersetzen**, Index und Arbeitsverzeichnis nachziehen (`currency.py` ist wieder da), und **`ORIG_HEAD`** auf `9962756` setzen. `git reset --hard ORIG_HEAD` würde den Merge komplett rückgängig machen.

```
*   4eb51f9 Merge branch 'feature/waehrung'     ← main
|\
| * 9221c85 currency: add conversion to EUR     ← feature/waehrung
* | 9962756 docs: mention skipped rows in README
|/
* d819102 loader: skip rows without price
* 03f3189 Add loader and README
```

> [!note] Was bei einem Konflikt anders wäre
> Hätten beide Seiten dieselbe Zeile in `loader.py` geändert, hielte Git nach Schritt 2 an. Es entstünden `MERGE_HEAD`, `MERGE_MSG` und `AUTO_MERGE`, im Index stünden für `loader.py` **drei Einträge** (Stage 1 = Basis, 2 = ours, 3 = theirs), und in der Datei stünden Konfliktmarker. Details in der [[Git Befehlsreferenz#git merge|Befehlsreferenz]].

---

## 10. Den Branch löschen

```bash
git branch -d feature/waehrung
```

```
GELÖSCHT .git/refs/heads/feature/waehrung
GELÖSCHT .git/logs/refs/heads/feature/waehrung
```

**Nur die Ref und ihr Reflog verschwinden.** Der Commit `9221c85` und seine Objekte bleiben, und zwar dauerhaft: Er ist der **zweite Parent des Merge-Commits** und damit von `main` aus erreichbar. Genau das prüft `-d`, bevor es löscht. Bei einem nicht gemergten Branch würde es sich weigern, nur `-D` würde dann trotzdem löschen.

Der Branch war also nur ein **Name** für einen Commit. Die Arbeit selbst steckt in den Objekten.

---

## 11. git remote und git push — ein Server kommt dazu

Als „Server“ dient ein **Bare-Repository**, also ein Repo ohne Arbeitsverzeichnis. GitLab und GitHub speichern ihre Repos genauso.

```bash
git init --bare -b main /tmp/bsp/server.git
git remote add origin /tmp/bsp/server.git
```

```
GEÄNDERT .git/config
```

```ini
[remote "origin"]
	url = /tmp/bsp/server.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

`remote add` **schreibt nur Konfiguration** und nimmt keinen Kontakt zum Server auf. Die zweite Zeile ist die **Refspec**: „Jeder Branch `refs/heads/X` dort landet bei mir als `refs/remotes/origin/X`.“

```bash
git push -u origin main
```

```
NEU      .git/refs/remotes/origin/main
NEU      .git/logs/refs/remotes/origin/main
GEÄNDERT .git/config
```

**Lokal** entstehen keine Objekte, sie werden ja nur verschickt. Es ändert sich zweierlei:

- **`refs/remotes/origin/main`** wird angelegt, mit `4eb51f9…`. Das ist Gits Notiz: „So stand `main` auf dem Server, als ich zuletzt nachgesehen habe.“
- **`config`** bekommt durch `-u` die **Tracking-Konfiguration**:
  ```ini
  [branch "main"]
  	remote = origin
  	merge = refs/heads/main
  ```
  Ab jetzt wissen `git push`, `git pull` und `git status` ohne Argumente, welcher Remote-Branch zu `main` gehört.

**Auf dem Server** landen alle **18 Objekte** (5 Blobs, 8 Trees, 5 Commits) und `refs/heads/main`. Hier lohnt sich ein zweiter Blick: `9221c85` ist mit dabei, obwohl es den Branch `feature/waehrung` nicht mehr gibt. Er wird über den Merge-Commit erreicht, und Git überträgt **alles, was vom gepushten Commit aus erreichbar ist**. Den Branch-Namen selbst hast du nie gepusht, ihn kennt nur dein Rechner.

> [!note] Loose Objects oder Packfile?
> Übertragen wird immer ein **Packfile**. Kleine Pakete (unter 100 Objekten, `transfer.unpackLimit`) entpackt der Empfänger aber wieder zu einzelnen Dateien. Deshalb liegen hier alle 18 Objekte als einzelne Dateien in `server.git/objects/`.

---

## 12. Die Kollegin pusht, wir holen mit git fetch

Lisa klont das Repo, ergänzt in `loader.py` einen Logger und pusht:

```bash
# in Lisas Klon
git commit -m "loader: add module logger"      # → d56efd7
git push
```

Auf dem Server steht `main` jetzt auf `d56efd7`. **Bei uns ändert sich nichts.** Git hat keine Verbindung zum Server, solange wir nicht selbst nachfragen.

```bash
git fetch
```

```
From /tmp/bsp/server
   4eb51f9..d56efd7  main       -> origin/main
```

```
NEU      .git/objects/ba/f3965…   ← Blob: Lisas loader.py
NEU      .git/objects/e3/df7ae…   ← Tree src/
NEU      .git/objects/f5/41026…   ← Wurzel-Tree
NEU      .git/objects/d5/6efd7…   ← Lisas Commit
GEÄNDERT .git/refs/remotes/origin/main            ← 4eb51f9 → d56efd7
GEÄNDERT .git/logs/refs/remotes/origin/main
NEU      .git/refs/remotes/origin/HEAD             ← "Standard-Branch des Servers ist main"
NEU      .git/logs/refs/remotes/origin/HEAD
NEU      .git/FETCH_HEAD                           ← was gerade geholt wurde
```

Genau **4 Objekte**, nämlich nur die, die uns fehlten. `fetch` und der Server handeln vorher aus, welche Commits auf beiden Seiten schon bekannt sind.

**Und was ist unverändert geblieben?**

```
$ cat .git/refs/remotes/origin/main
d56efd7ccab65e641bbb886a333218851b629d39     ← neu

$ cat .git/refs/heads/main
4eb51f9ed732f1854425551010e4b1b66d373a8e     ← unverändert!
```

`refs/heads/main`, `index`, Arbeitsverzeichnis: alles unberührt. **Deshalb ist `fetch` immer gefahrlos.** Es aktualisiert nur deine Notizen über den Server. `status` kann jetzt vergleichen:

```
$ git status
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
```

```
* d56efd7 loader: add module logger            ← origin/main
*   4eb51f9 Merge branch 'feature/waehrung'    ← main
|\
…
```

> [!tip] Genau jetzt ist der richtige Moment zum Hinschauen
> Zwischen `fetch` und dem Zusammenführen kannst du prüfen, was reinkommt: `git log main..origin/main` (welche Commits) oder `git diff main origin/main` (welche Änderungen). `git pull` überspringt diesen Moment.

---

## 13. Fast-Forward — der zweite Teil von git pull

```bash
git merge origin/main
```

```
Updating 4eb51f9..d56efd7
Fast-forward
 src/loader.py | 4 ++++
```

```
GEÄNDERT .git/refs/heads/main          ← 4eb51f9 → d56efd7
GEÄNDERT .git/ORIG_HEAD                ← 4eb51f9
GEÄNDERT .git/index
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/logs/refs/heads/main
```

**Kein einziges neues Objekt.** `main` (`4eb51f9`) ist ein Vorfahre von `origin/main` (`d56efd7`). Wir haben nichts, was Lisa nicht hat, also gibt es nichts zusammenzuführen. Git **schiebt die Ref einfach vor** und zieht Index und Arbeitsverzeichnis nach. Daher der Name: „vorspulen“. Im Reflog steht `merge origin/main: Fast-forward`, im Vergleich zum echten Merge in Schritt 9 (`Merge made by the 'ort' strategy`).

**`git pull` ist genau Schritt 12 + Schritt 13** in einem Befehl. Hätten wir zwischendurch selbst auf `main` committet, wäre Schritt 13 ein echter Merge wie in Schritt 9 geworden, mit Merge-Commit. Oder mit `pull --rebase` ein Rebase, der unseren Commit auf `d56efd7` neu aufsetzt. Das ist das Thema von Block 2 und 3 der [[Git Leseliste]].

```
$ git status
Your branch is up to date with 'origin/main'.
```

---

## Gesamtbild

### Alle Objekte am Ende (22)

| Commit | Neue Blobs | Neue Trees | Wiederverwendet |
|---|---|---|---|
| `03f3189` Add loader and README | `4bed182` README v1, `0aeb68f` loader v1 | `99cca54` src, `0098e02` Wurzel | — |
| `d819102` loader: skip rows | `47e6d10` loader v2 | `f288fb4` src, `b2802fb` Wurzel | README v1 |
| `9221c85` currency | `56f30a4` currency | `25ff708` src, `da1a8bf` Wurzel | README v1, loader v2 |
| `9962756` docs: README | `859c2a6` README v2 | `a0f5f80` Wurzel | **ganzer src-Tree** `f288fb4` |
| `4eb51f9` Merge | — | `5215efc` Wurzel | README v2, **src-Tree von feature** `25ff708` |
| `d56efd7` Lisa: logger | `baf3965` loader v3 | `e3df7ae` src, `f541026` Wurzel | README v2, currency |

Sechs Commits, jeder ein **vollständiger Snapshot**. Trotzdem nur 6 Blobs, weil jeder Dateiinhalt genau einmal existiert, egal in wie vielen Commits er vorkommt.

### Welcher Befehl fasst was an

| Befehl | `objects/` | `index` | Arbeitsverz. | `HEAD` | eigener Branch | `refs/remotes/` | `config` |
|---|---|---|---|---|---|---|---|
| `init` | (leer) | — | — | **angelegt** | — | — | **angelegt** |
| Datei ändern | — | — | ✏️ | — | — | — | — |
| `add` | **Blobs** | ✏️ | — | — | — | — | — |
| `commit` | **Trees + Commit** | (Cache) | — | — | ✏️ / **angelegt** | — | — |
| `switch -c` | — | — | — | ✏️ | **angelegt** | — | — |
| `switch` | — | ✏️ | ✏️ | ✏️ | — | — | — |
| `merge` (echt) | **Trees + Commit** | ✏️ | ✏️ | — | ✏️ | — | — |
| `merge` (Fast-Forward) | — | ✏️ | ✏️ | — | ✏️ | — | — |
| `branch -d` | — | — | — | — | **gelöscht** | — | — |
| `remote add` | — | — | — | — | — | — | ✏️ |
| `push` | (Server) | — | — | — | — | ✏️ | ✏️ (mit `-u`) |
| `fetch` | **geholte** | — | — | — | — | ✏️ | — |

Fast jeder Befehl, der eine Ref bewegt, schreibt zusätzlich eine Zeile in den passenden Reflog unter `logs/`.

### Die vier Sätze, die man mitnehmen sollte

1. **Objekte entstehen nur bei `add`, `commit`, `merge` (bzw. `rebase`) und `fetch`**, und sie werden nie verändert, nur neu angelegt.
2. **Ein Branch ist eine Datei mit einem Hash.** Anlegen, Wechseln und Löschen kosten fast nichts, weil keine Daten bewegt werden.
3. **`HEAD` zeigt auf einen Branch, der Branch auf einen Commit.** `commit` und `merge` bewegen den Branch, `switch` bewegt `HEAD`.
4. **`refs/remotes/` sind deine Notizen über den Server.** `fetch` aktualisiert sie, deine eigenen Branches ändern sich erst durch `merge` oder `rebase`.

### Self-Check

- Warum ändert sich in Schritt 5a nichts in `.git/`, und was folgt daraus für `git restore`?
- In Schritt 8b entsteht nur *ein* neuer Tree. Warum nicht zwei wie in Schritt 7?
- Woran erkennt man im Objekt `4eb51f9`, dass es ein Merge-Commit ist? Was gibt `git rev-parse 4eb51f9^2` aus?
- Warum durfte `git branch -d` in Schritt 10 löschen? Wann hätte es sich geweigert?
- `fetch` hat 4 Objekte geholt. Welche, und warum nicht alle 22?
- Warum entsteht in Schritt 13 kein Commit, in Schritt 9 aber schon?

> [!example]- Skript zum Nachspielen (identische Hashes)
> Das Skript legt alles unter `/tmp/bsp/` an. `snap .` gibt nach jedem Schritt aus, was sich in `.git/` geändert hat. Die festen Werte für Name und Datum sorgen für dieselben Hashes wie oben, und eine eigene `~/.gitconfig` wird dabei ignoriert.
> ```bash
> #!/bin/bash
> set -e
> rm -rf /tmp/bsp/preisimport /tmp/bsp/server.git /tmp/bsp/lisa /tmp/bsp/snap*
> mkdir -p /tmp/bsp
> export GIT_AUTHOR_NAME="Nick" GIT_AUTHOR_EMAIL="nick@example.com"
> export GIT_COMMITTER_NAME="Nick" GIT_COMMITTER_EMAIL="nick@example.com"
> export GIT_CONFIG_GLOBAL=/dev/null GIT_CONFIG_NOSYSTEM=1
> T=1792656000
> ts(){ T=$((T+600)); export GIT_AUTHOR_DATE="$T +0200" GIT_COMMITTER_DATE="$T +0200"; }
> snap(){ (cd "$1" && find .git -type f -not -path '*/hooks/*' -exec sha1sum {} \; | sort -k2) > /tmp/bsp/snap.new
>   if [ -f /tmp/bsp/snap.old ]; then
>     join -1 2 -2 2 -a1 -a2 -e NONE -o 0,1.1,2.1 /tmp/bsp/snap.old /tmp/bsp/snap.new | awk '$2=="NONE"{print "NEU      "$1} $3=="NONE"{print "GELÖSCHT "$1} $2!="NONE"&&$3!="NONE"&&$2!=$3{print "GEÄNDERT "$1}'
>   else awk '{print "NEU      "$2}' /tmp/bsp/snap.new; fi
>   mv /tmp/bsp/snap.new /tmp/bsp/snap.old; }
> cd /tmp/bsp
> # 1
> git init -q -b main preisimport; cd preisimport; git config gc.auto 0; snap .
> # 2
> mkdir src
> printf '# Preisimport\n\nLädt Preislisten in die Datenbank.\n' > README.md
> printf 'def load(rows):\n    for row in rows:\n        insert(row["sku"], row["price"])\n' > src/loader.py
> git status; snap .
> # 3 + 4
> git add .; snap .
> ts; git commit -q -m "Add loader and README"; snap .
> # 5
> printf 'def load(rows):\n    for row in rows:\n        if row["price"] is None:\n            continue\n        insert(row["sku"], row["price"])\n' > src/loader.py
> git status --short; snap .
> git add src/loader.py; snap .
> ts; git commit -q -m "loader: skip rows without price"; snap .
> # 6 + 7
> git switch -c feature/waehrung; snap .
> printf 'def to_eur(amount, currency, rates):\n    return amount * rates[currency]\n' > src/currency.py
> git add src/currency.py; ts; git commit -q -m "currency: add conversion to EUR"; snap .
> # 8
> git switch main; snap .
> printf '# Preisimport\n\nLädt Preislisten in die Datenbank.\nZeilen ohne Preis werden übersprungen.\n' > README.md
> git add README.md; ts; git commit -q -m "docs: mention skipped rows in README"; snap .
> # 9 + 10
> ts; git merge -q --no-edit feature/waehrung; snap .
> git branch -d feature/waehrung; snap .
> # 11
> git init -q --bare -b main /tmp/bsp/server.git
> git remote add origin /tmp/bsp/server.git; snap .
> git push -u origin main; snap .
> # 12: Lisa
> git clone -q /tmp/bsp/server.git /tmp/bsp/lisa; cd /tmp/bsp/lisa
> export GIT_AUTHOR_NAME=Lisa GIT_AUTHOR_EMAIL=lisa@example.com GIT_COMMITTER_NAME=Lisa GIT_COMMITTER_EMAIL=lisa@example.com
> printf 'import logging\n\nlog = logging.getLogger(__name__)\n\n' | cat - src/loader.py > /tmp/bsp/l && mv /tmp/bsp/l src/loader.py
> git add .; ts; git commit -q -m "loader: add module logger"; git push -q
> cd /tmp/bsp/preisimport
> export GIT_AUTHOR_NAME="Nick" GIT_AUTHOR_EMAIL="nick@example.com" GIT_COMMITTER_NAME="Nick" GIT_COMMITTER_EMAIL="nick@example.com"
> git fetch; snap .
> # 13
> git merge --ff-only origin/main; snap .
> git log --oneline --graph --all
> ```
