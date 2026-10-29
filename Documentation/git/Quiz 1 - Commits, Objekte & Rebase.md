---
tags:
  - documentation
---

# Quiz 1 — Commits, Objekte & Rebase

Quiz zu [[Git Leseliste#Block 1 — Commits, Objektmodell & Rebase|Block 1 der Git-Leseliste]]: Pro Git „Git Objects", „git rebase in depth", Chris Beams' Commit-Message-Regeln und Scoped Commits.

> [!info] So geht's (~30 Min.)
> 1. **Aus dem Kopf** antworten, direkt unter **Antwort:**. Nicht nachschlagen, denn genau die Lücken sind das Ergebnis. Stichpunkte reichen.
> 2. Erst wenn alle Fragen beantwortet sind, die eingeklappten Lösungen öffnen **oder** Claudian sagen „Quiz 1 reviewen". Dann gibt es ein Urteil pro Frage (✅/🟡/❌) und auf Wunsch Anki-Karten zu den Lücken.
>
> Die Fragen werden schwieriger: **A–D** fragen den Stoff je Artikel ab, **E** verlangt Transfer auf eigene Situationen.

---

## A — Git Objects (Pro Git 10.2)

### A1. Welche drei Objekttypen behandelt das Kapitel, und was speichert jeder davon?

**Antwort:**
blobs: speichert jede art von file
trees: speichert referenzen zu blobs und anderen trees
commits: speichert ein referenz zu einem tree (snapshot), information über Zeitpunkt und Person und den parent commit

> [!success]- Lösung
> - **Blob**: nur den **Inhalt** einer Datei, ohne Namen und ohne Rechte.
> - **Tree**: ein Verzeichnis. Pro Eintrag stehen dort Modus (`100644` normal, `100755` ausführbar, `120000` Symlink), Typ (blob/tree), Hash und **Dateiname**.
> - **Commit**: Zeiger auf **einen** Top-Level-Tree, Eltern-Commit(s), Author, Committer und Message.
>
> Ein vierter Typ, das **annotated Tag**, kommt erst in Kap. 10.3.

### A2. Du hast zwei Dateien `a.txt` und `kopie/b.txt` mit exakt gleichem Inhalt. Wie viele Blobs legt Git an? Was passiert auf Objektebene, wenn du `a.txt` nur umbenennst?

**Antwort:**

Er legt zwei blobs an und dementpsrechend auch zwei ojekte.

> [!success]- Lösung
> **Ein** Blob. Der Hash hängt nur vom Inhalt ab, also ist er bei beiden Dateien identisch. Der Name steht im Tree, nicht im Blob.
> Beim Umbenennen entsteht **kein neuer Blob**, nur ein neuer Tree (anderer Name, gleicher Blob-Hash) und ein neuer Commit. Deshalb muss Git Umbenennungen beim Anzeigen erst *erraten*: Es speichert keine „rename"-Information, sondern vergleicht Snapshots.

### A3. Was steht in einem Commit-Objekt, und was steht ausdrücklich *nicht* darin?

**Antwort:**

Habe ich oben schon beantwortet. zeigt keine diff an

> [!success]- Lösung
> **Drin:** `tree <hash>`, `parent <hash>` (keiner beim ersten Commit, zwei oder mehr beim Merge), `author` und `committer` (jeweils Name, E-Mail, Zeitstempel), eine Leerzeile und dann die Message.
> **Nicht drin:** ein Diff. Git speichert **Snapshots**, keine Änderungen. Was `git show` als Diff anzeigt, wird beim Anzeigen aus zwei Trees berechnet. Auch der Branch-Name steht nicht im Commit.

### A4. Wie entsteht der Hash eines Objekts, und wo landet es auf der Platte?

**Antwort:**

sha1 und er landet im Ordner ...


> [!success]- Lösung
> 1. Git baut einen Header: Typ, Leerzeichen, Größe in Bytes, Null-Byte, also z. B. `blob 16\0`.
> 2. Header und Inhalt werden aneinandergehängt, davon wird der **SHA-1** berechnet (40 Hex-Zeichen).
> 3. Header und Inhalt werden mit **zlib** komprimiert und unter `.git/objects/<erste 2 Zeichen>/<restliche 38>` gespeichert.
>
> *Ausblick:* Git 3.0 legt neue Repos mit SHA-256 an. Das Prinzip bleibt gleich.

### A5. Was tun `git cat-file -t <hash>` und `git cat-file -p <hash>`? Und was zeigt `git cat-file -p main^{tree}`?

**Antwort:**


> [!success]- Lösung
> `-t` gibt den **Typ** des Objekts aus (blob/tree/commit/tag), `-p` gibt den **Inhalt** lesbar aus („pretty-print").
> `main^{tree}` heißt: der Tree, auf den der letzte Commit von `main` zeigt. Die Ausgabe ist also das Wurzelverzeichnis des Projekts mit Modus, Typ, Hash und Namen pro Eintrag.

### A6. Du änderst per Rebase nur die Message eines Commits ganz unten in einer Kette von fünf Commits. Warum haben danach **alle fünf** einen neuen Hash, obwohl sich an den Dateien der oberen vier nichts geändert hat?

**Antwort:**


> [!success]- Lösung
> Die Message ist Teil des Commit-Inhalts, also ändert sich der Hash des unteren Commits. Der nächste Commit enthält `parent <hash>`, und dieser Parent-Hash hat sich geändert. Damit ändert sich sein Inhalt und damit sein Hash, und so weiter bis oben. Die Kette funktioniert wie eine Blockchain.
> **Die Konsequenz fürs Team:** Rebase erzeugt immer **neue** Commits. Wer die alten schon hat, z. B. weil sie gepusht waren, hat danach eine abweichende Historie. Das ist die Grundlage der „Goldenen Regel" in Block 2. Die Trees der oberen Commits bleiben übrigens dieselben Objekte, neu sind nur die Commit-Objekte.

---

## B — git rebase in depth

### B1. Was ist der Unterschied zwischen `pick`, `reword`, `edit`, `squash` und `fixup`?

**Antwort:**


> [!success]- Lösung
> - `pick`: Commit unverändert übernehmen.
> - `reword`: übernehmen, aber die Message neu schreiben.
> - `edit`: übernehmen und **anhalten**, damit du ihn ändern oder aufteilen kannst. Weiter geht es mit `git rebase --continue`.
> - `squash`: in den **vorherigen** Commit einschmelzen. Der Editor öffnet sich mit **beiden** Messages zum Zusammenführen.
> - `fixup`: wie `squash`, aber die Message des eingeschmolzenen Commits wird **verworfen**.

### B2. Du führst `git rebase -i HEAD~3` aus und siehst diesen Plan. Wie bringst du den dritten Commit in den ersten ein, ohne seine Message zu behalten?
```
pick 8d3fc77 Add greeting.txt
pick 2a73a77 Add farewell.txt
pick 0b9d0bb fixup greeting.txt
```

**Antwort:**


> [!success]- Lösung
> Die Zeile **verschieben** (direkt unter den Ziel-Commit) und `pick` durch `fixup` (oder `f`) ersetzen:
> ```
> pick 8d3fc77 Add greeting.txt
> fixup 0b9d0bb fixup greeting.txt
> pick 2a73a77 Add farewell.txt
> ```
> Der Plan wird **von oben nach unten** ausgeführt. Oben steht der **älteste** Commit, also genau umgekehrt wie bei `git log`.

### B3. Wie automatisiert man das aus B2? Nenne beide Befehle und die Config-Option.

**Antwort:**


> [!success]- Lösung
> ```bash
> git commit -a --fixup <ziel-commit>     # erzeugt Message "fixup! <Betreff des Ziels>"
> git rebase -i --autosquash HEAD~3       # sortiert die fixup!-Commits automatisch ein
> ```
> Mit `git config --global rebase.autosquash true` kann man `--autosquash` weglassen. Es gibt auch `git commit --squash`, wenn man die Message noch bearbeiten will.

### B4. Wie teilt man einen zu großen Commit in zwei auf? Beschreibe die Schritte. Warum ist `git reset HEAD^` hier laut Fußnote **kein** „soft reset"?

**Antwort:**


> [!success]- Lösung
> 1. `git rebase -i` und den Commit auf `edit` setzen. Der Rebase hält bei ihm an.
> 2. `git reset HEAD^` nimmt den Commit zurück, die Änderungen bleiben im Working Directory, sind aber **nicht gestaged**.
> 3. `git commit -p` (oder `git add -p`) und mit `s` die Hunks aufteilen, `y` für den ersten Teil wählen und committen.
> 4. Den Rest committen, dann `git rebase --continue`.
>
> Ohne Option ist `reset` ein **mixed** Reset: HEAD wandert zurück, der Index wird zurückgesetzt, das Working Directory bleibt. `--soft` würde alles **gestaged lassen**, dann könnte man nur alles auf einmal committen. Das ist nicht das Ziel beim Aufteilen. Details dazu in Block 5.

### B5. Ausgangslage:
```
A--B--C--D        main
    \--E--F       feature-1
        \--G      feature-2
```
`feature-2` hängt gar nicht von `E` ab. Welcher Befehl setzt `G` direkt auf `main`? Wie sieht der Graph danach aus?

**Antwort:**


> [!success]- Lösung
> ```bash
> git rebase --onto main feature-1 feature-2
> ```
> Das heißt: Nimm die Commits von `feature-2`, die **nicht** in `feature-1` sind (also nur `G`), und spiele sie auf `main` ab.
> ```
> A--B--C--D        main
>    |      \--G'   feature-2
>     \--E--F       feature-1
> ```
> `G'` ist ein **neuer** Commit mit neuem Hash (siehe A6).

### B6. Mitten im Rebase gibt es einen Konflikt. Du hast die Datei repariert und mit `git add` markiert. Wie geht es weiter, und welche Alternative gibt es noch? Was ist bei `--ours` und `--theirs` anders als beim Merge?

**Antwort:**


> [!success]- Lösung
> - **Weiter:** `git rebase --continue`, **nicht** `git commit`.
> - **Alternativen:** `git rebase --skip` lässt diesen Commit ganz weg. Das ist sinnvoll, wenn die Änderung upstream schon in anderer Form drin ist. `git rebase --abort` bricht komplett ab und stellt den Zustand vor dem Rebase wieder her.
> - Welcher Commit gerade scheitert, zeigt `git rebase --show-current-patch` bzw. `git show REBASE_HEAD`.
> - **ours/theirs vertauscht:** Beim Rebase ist `--ours` die Basis, auf die du rebasest (HEAD), und `--theirs` **dein eigener** Commit, der gerade abgespielt wird. Der Grund: Rebase checkt die Basis aus und „merged" deine Commits einzeln hinein.

### B7. Der Rebase ist völlig schiefgelaufen, Commits scheinen weg. Was rettet dich? Und wann darf man laut Artikel Historie überhaupt umschreiben?

**Antwort:**


> [!success]- Lösung
> **`git reflog`**: Es listet jede Änderung an einer Referenz mit dem alten Hash. Den kann man per `git reset --hard <hash>`, `cherry-pick` oder `checkout` zurückholen. Fast nichts in Git ist wirklich weg, solange die Garbage Collection nicht gelaufen ist.
> **Umschreiben:** Unproblematisch bei **noch nicht gepushten** Commits, auf eigenen Feature-Branches und in eigenen Forks. Davon abgeraten wird bei öffentlichen, geteilten oder stabilen Branches wie `main`. Nach dem Umschreiben eines eigenen, schon gepushten Feature-Branches ist ein Force-Push nötig. Der Artikel nennt `git push -f`, besser ist `--force-with-lease` (Block 3).

### B8. Was unterscheidet `git pull` von `git pull --rebase`?

**Antwort:**


> [!success]- Lösung
> - `git pull` = `git fetch` + `git merge origin/<branch>`. Wenn beide Seiten neue Commits haben, entsteht ein **Merge-Commit**.
> - `git pull --rebase` ≈ `git fetch` + `git rebase origin/<branch>`. Deine lokalen Commits werden **oben drauf** gesetzt, die Historie bleibt linear.
> - Als Standard einstellen: `git config --global pull.rebase true`.

---

## C — How to Write a Git Commit Message (Chris Beams)

### C1. Zähle die sieben Regeln auf.

**Antwort:**


> [!success]- Lösung
> 1. Betreff und Body durch eine **Leerzeile** trennen
> 2. Betreff auf **50 Zeichen** begrenzen
> 3. Betreff **großschreiben**
> 4. **Kein Punkt** am Ende des Betreffs
> 5. Betreff im **Imperativ**
> 6. Body bei **72 Zeichen** umbrechen
> 7. Im Body **was und warum** erklären, nicht **wie**

### C2. Mit welchem Satz prüft man den Imperativ? Welche dieser Betreffzeilen bestehen den Test?
a) `Fixed bug with Y` · b) `Refactor subsystem X for readability` · c) `More fixes for broken stuff` · d) `Remove deprecated methods` · e) `Changing behavior of X`

**Antwort:**


> [!success]- Lösung
> Der Test lautet: **„If applied, this commit will _<Betreff>_."**
> Es bestehen nur **b** und **d**. a) ist Vergangenheit, c) ist kein Satz und sagt nichts, e) ist ein Partizip.
> Git selbst schreibt ebenfalls im Imperativ: `Merge branch 'x'`, `Revert "…"`.

### C3. Warum soll der Body *was und warum* erklären, aber nicht *wie*?

**Antwort:**


> [!success]- Lösung
> Das **Wie** zeigt der Diff, und gut geschriebener Code erklärt sich weitgehend selbst. Was der Code **nicht** sagen kann: Was war vorher kaputt oder unbefriedigend? Warum ist diese Lösung die richtige, welche Alternativen wurden verworfen, welche Nebenwirkungen gibt es? Genau dieser Kontext fehlt Monate später beim `git blame`, und dann ist niemand mehr da, der ihn kennt.

### C4. Woher kommen die Grenzen 50 und 72, und wie streng sind sie?

**Antwort:**


> [!success]- Lösung
> - **50** ist eine Faustregel: Sie zwingt zu einem knappen Betreff und passt in Ausgaben wie `git log --oneline` oder `shortlog`. Als **harte** Grenze nennt Beams **72**. Längere Betreffzeilen kürzt GitHub mit „…" ab.
> - **72** für den Body: Git bricht selbst nicht um. Bei 72 Zeichen bleibt in einem 80-Spalten-Terminal Platz für die Einrückung, die `git log` hinzufügt.

### C5. Welche drei Dinge sollte ein Team laut Beams für seine Commit-Messages festlegen?

**Antwort:**


> [!success]- Lösung
> 1. **Stil**: Markup, Umbruchbreiten, Grammatik, Groß- und Kleinschreibung, Zeichensetzung
> 2. **Inhalt**: welche Informationen in den Body gehören und welche nicht
> 3. **Metadaten**: wie Ticket-IDs, Pull-Request-Nummern usw. referenziert werden

---

## D — Scoped Commits

### D1. Wie ist eine Scoped-Commit-Message aufgebaut, und warum gehört der Scope nach vorn? Nenne mindestens zwei der Gruppen, für die das laut der Seite zählt.

**Antwort:**


> [!success]- Lösung
> ```
> <scope>: <description>
>
> [optional body]
>
> [optional trailer(s)]
> ```
> Der Scope ist das Subsystem, Modul oder der Bereich, also **wo** sich etwas geändert hat. Das ist beim Überfliegen des Logs die wichtigste Information.
> - **Contributors** wollen wissen, was sich in „ihrem" Bereich getan hat oder womit es beim Rebase Konflikte geben könnte.
> - **Debugger** suchen Änderungen an der Komponente, in der der Bug auftritt.
> - **Incident Responder** suchen bei einem Ausfall Commits im passenden Bereich, z. B. `auth`, kurz vor dem Fehleranstieg.

### D2. Ein Commit betrifft zwei Scopes. Welche Optionen nennt die Seite? Und wohin gehört eine Jira-Ticketnummer?

**Antwort:**


> [!success]- Lösung
> **Mehrere Scopes:** einen allgemeineren Scope wählen, beide mit Komma trennen (`a, b: …`), bei Änderungen am ganzen Repo `treewide`/`all`/`global` verwenden, oder im Notfall den Commit als „special commit" ohne Scope behandeln.
> **Ticket:** Das ist frei. Vorgeschlagen werden Klammern nach dem Scope (`auth (PROJ-123): fix login bug`) oder ein Trailer im Body (`Jira-Ticket: PROJ-123`). Bezug zu [[Jira]].

### D3. Warum soll man aus Scoped Commits **kein** Changelog generieren?

**Antwort:**


> [!success]- Lösung
> Commit-Log und Changelog haben **verschiedene Leser**. Das Log ist für **Entwickler**, die verstehen wollen, wie der Code entstanden ist. Das Changelog ist für **Nutzer**, die wissen wollen, was sich zwischen zwei Releases für sie ändert. Ein Nutzer interessiert sich nicht für Refactorings, und „fix typo" ist für ihn Rauschen. Genau das ist der Kritikpunkt an Conventional Commits, deren Typen (`feat`, `fix`) vor allem für automatische Changelogs da sind. Mehr dazu in Block 11 (Keep a Changelog).

---

## E — Transfer

### E1. Beams sagt „Betreff großschreiben", die Linux- und Go-Beispiele auf scopedcommits.com schreiben nach dem Doppelpunkt klein weiter. Wer hat recht, und was machst du in einem neuen Team?

**Antwort:**


> [!success]- Lösung
> Keiner von beiden hat „recht". Beides sind Konventionen, und Beams sagt selbst, dass das Team den Stil festlegen soll (C5). Selbst die Beispiele auf scopedcommits.com sind uneinheitlich: FreeBSD schreibt `linuxulator: Return EINVAL …` groß, Linux und Go schreiben klein.
> **Im neuen Team:** erst `git log --oneline -30` lesen und die vorhandene Konvention übernehmen. Gibt es eine Datei wie `CONTRIBUTING.md` oder eine Confluence-Seite dazu, gilt die. **Konsistenz schlägt persönliche Vorliebe.** Die übrigen Regeln (Leerzeile, Imperativ, kein Punkt, was und warum) gelten in beiden Stilen.

### E2. Schreib eine vollständige Commit-Message (Scoped-Stil, Beams-Regeln) für diese Änderung: Im ETL-Loader deiner Pipeline ist der Import abgestürzt, wenn eine Zeile keinen Preis hatte (`NULL`). Du überspringst solche Zeilen jetzt mit einer Warnung im Log und hast einen Test dafür ergänzt. Ticket: `DATA-42`.

**Antwort:**


> [!success]- Lösung
> Ein mögliches Beispiel:
> ```
> loader: skip rows without price instead of crashing
>
> The import aborted with a TypeError as soon as a source row had a NULL
> price, so a single bad row blocked the whole nightly load.
>
> Such rows are now skipped and logged as a warning with their row
> number, so they can be fixed at the source. Rejecting the whole batch
> was considered, but one incomplete product should not block all
> others.
>
> Jira-Ticket: DATA-42
> ```
> **Checkliste:** Scope vorn · Imperativ („skip", nicht „skipped") · ≤ 50 Zeichen, kein Punkt · Leerzeile · Body erklärt das *Problem* und *warum diese Lösung*, nicht welche Zeilen geändert wurden · Ticket als Trailer. Der Test muss nicht erwähnt werden, er ist im Diff sichtbar und gehört zum selben logischen Commit.

### E3. Vor deinem ersten Pull Request sieht dein Feature-Branch so aus (älteste oben). Wie räumst du auf, und was musst du danach beim Push beachten?
```
a1 Add price validation
b2 wip
c3 fix tests
d4 Add currency conversion
e5 typo
f6 fix price validation edge case
```

**Antwort:**


> [!success]- Lösung
> `git fetch` und dann `git rebase -i origin/main`. Es werden nur die Commits angezeigt, die noch nicht auf `main` sind. Im Plan:
> - `b2`, `c3` und `f6` gehören inhaltlich zur Validierung, also direkt unter `a1` verschieben und als `fixup` markieren
> - `e5` als `fixup` unter den Commit, in dem der Tippfehler entstanden ist (z. B. `d4`)
> - wenn nötig, die Messages per `reword` an die Team-Konvention anpassen
>
> Ergebnis sind **zwei** saubere Commits: Validierung und Währungsumrechnung. Beim nächsten Mal gleich `commit --fixup` benutzen (B3).
> **Push:** Falls der Branch schon gepusht war, haben alle Commits neue Hashes (A6). Ein normaler Push wird abgelehnt, du brauchst einen Force-Push, und zwar mit **`git push --force-with-lease`**. Das ist in Ordnung, weil es dein eigener Feature-Branch ist. Auf `main` niemals.

---

## Auswertung

*(füllt Claudian beim Review aus)*

| Teil | ✅ | 🟡 | ❌ |
|---|---|---|---|
| A — Git Objects | | | |
| B — Rebase | | | |
| C — Commit Messages | | | |
| D — Scoped Commits | | | |
| E — Transfer | | | |
