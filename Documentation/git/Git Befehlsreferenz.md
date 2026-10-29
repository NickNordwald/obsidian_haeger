---
tags:
  - documentation
---

# Git Befehlsreferenz

Die Git-Befehle, die man im Arbeitsalltag regelmäßig braucht. **Teil 1** erklärt jeden Befehl in einem Satz. **Teil 2** zeigt per Link (↓), was er intern tut, und zwar auf drei Arten:

- **Plumbing:** wie man denselben Effekt mit den Low-Level-Befehlen erreicht, die Git intern benutzt (aus [[Git Leseliste#Block 1 — Commits, Objektmodell & Rebase|Pro Git 10.2]]).
- **`.git/` vorher → nachher:** welche Dateien unter `.git/` tatsächlich neu entstehen, sich ändern oder verschwinden.
- **Was in den Dateien passiert:** pro Datei, was genau drinsteht oder sich ändert. Jeder Abschnitt ist für sich verständlich.

Ein zusammenhängendes Beispiel mit echten Hashes steht in [[Git von innen - ein Durchlauf]].

Gehört zur [[Git Leseliste]]. Alle Plumbing-Abläufe und Dateiänderungen wurden am 22.10.2026 mit Git 2.53 in Sandbox-Repos nachgeprüft.

> [!tip] Ausprobieren
> Jede Plumbing-Sequenz lässt sich in einer Sandbox nachspielen (`git init /tmp/sandbox`). Danach mit `git log --oneline --graph --all`, `git ls-files --stage` und `cat .git/HEAD` nachsehen, was passiert ist.

---

# Teil 1 — Übersicht

### Einrichten

| Befehl | Was er tut | |
|---|---|---|
| `git init` | Macht den aktuellen Ordner zum Repository | [[#git init\|↓]] |
| `git clone <url>` | Kopiert ein entferntes Repo samt Historie und checkt den Standard-Branch aus | [[#git clone\|↓]] |
| `git config` | Liest und setzt Einstellungen (Name, E-Mail, Verhalten von `pull` usw.) | [[#git config\|↓]] |

### Änderungen vorbereiten und festhalten

| Befehl | Was er tut | |
|---|---|---|
| `git status` | Zeigt, was staged, unstaged und untracked ist | [[#git status\|↓]] |
| `git add <pfad>` | Übernimmt den aktuellen Inhalt einer Datei in den nächsten Commit (stagen) | [[#git add\|↓]] |
| `git add -p` | Stagt nur ausgewählte Teile (Hunks) einer Datei | [[#git add\|↓]] |
| `git restore <pfad>` | Verwirft ungestagte Änderungen an einer Datei | [[#git restore\|↓]] |
| `git restore --staged <pfad>` | Nimmt eine Datei aus dem Staging zurück, die Änderung bleibt erhalten | [[#git restore\|↓]] |
| `git rm` / `git rm --cached` | Löscht eine Datei aus dem Repo (und von der Platte) bzw. hört nur auf, sie zu tracken | [[#git rm und git mv\|↓]] |
| `git mv <alt> <neu>` | Benennt eine getrackte Datei um | [[#git rm und git mv\|↓]] |
| `git commit -m "…"` | Speichert den gestagten Stand als neuen Commit | [[#git commit\|↓]] |
| `git commit --amend` | Ersetzt den letzten Commit (Inhalt und/oder Message) | [[#git commit --amend\|↓]] |
| `git commit --fixup <commit>` | Erzeugt einen Korrektur-Commit, den ein späterer Rebase in `<commit>` einschmilzt | [[#git rebase -i\|↓]] |
| `git diff` | Zeigt Änderungen: ungestagte (`git diff`), gestagte (`--staged`) oder zwischen zwei Commits | [[#git diff\|↓]] |

### Historie ansehen

| Befehl | Was er tut | |
|---|---|---|
| `git log` | Listet Commits auf. Nützlich: `--oneline --graph --all`, `-S <text>`, `-- <pfad>` | [[#git log und git show\|↓]] |
| `git show <commit>` | Zeigt einen Commit mit seinem Diff (oder ein beliebiges anderes Objekt) | [[#git log und git show\|↓]] |
| `git blame <datei>` | Zeigt für jede Zeile, welcher Commit sie zuletzt geändert hat | [[#git blame\|↓]] |
| `git reflog` | Zeigt, wohin `HEAD` und die Branches früher gezeigt haben. Das Sicherheitsnetz | [[#git reflog\|↓]] |

### Branches

| Befehl | Was er tut | |
|---|---|---|
| `git branch` | Listet Branches auf, legt an (`git branch <name>`), löscht (`-d`) oder benennt um (`-m`) | [[#git branch\|↓]] |
| `git switch <branch>` | Wechselt auf einen anderen Branch. `-c` legt ihn vorher an | [[#git switch und git checkout\|↓]] |
| `git checkout` | Der alte Allzweckbefehl: Branch wechseln **und** Dateien wiederherstellen. Heute aufgeteilt in `switch` und `restore` | [[#git switch und git checkout\|↓]] |
| `git merge <branch>` | Führt einen anderen Branch in den aktuellen zusammen (Fast-Forward oder Merge-Commit) | [[#git merge\|↓]] |
| `git rebase <basis>` | Setzt die eigenen Commits auf eine neue Basis um, sodass die Historie linear bleibt | [[#git rebase\|↓]] |
| `git rebase -i <basis>` | Interaktiver Rebase: Commits umsortieren, zusammenfassen, umbenennen, aufteilen, löschen | [[#git rebase -i\|↓]] |
| `git cherry-pick <commit>` | Übernimmt die Änderung eines einzelnen Commits auf den aktuellen Branch | [[#git cherry-pick\|↓]] |

### Zusammenarbeiten

| Befehl | Was er tut | |
|---|---|---|
| `git remote -v` / `git remote add` | Zeigt bzw. registriert entfernte Repos (meist `origin`) | [[#git remote\|↓]] |
| `git fetch` | Holt neue Commits und Branch-Stände vom Remote, **ohne** etwas an deinen Branches zu ändern | [[#git fetch\|↓]] |
| `git pull` | `fetch` + `merge` (oder mit `--rebase`: `fetch` + `rebase`) in einem Schritt | [[#git pull\|↓]] |
| `git push` | Lädt deine Commits hoch und setzt den Branch auf dem Remote weiter | [[#git push\|↓]] |
| `git push --force-with-lease` | Überschreibt einen Remote-Branch nach Rebase, aber nur, wenn ihn niemand zwischendurch verändert hat | [[#git push\|↓]] |

### Rückgängig machen

| Befehl | Was er tut | |
|---|---|---|
| `git reset --soft <commit>` | Setzt den Branch zurück, die Änderungen bleiben **staged** | [[#git reset\|↓]] |
| `git reset [--mixed] <commit>` | Setzt den Branch zurück, die Änderungen bleiben **unstaged** im Arbeitsverzeichnis | [[#git reset\|↓]] |
| `git reset --hard <commit>` | Setzt Branch, Index und Arbeitsverzeichnis zurück. Ungesicherte Änderungen sind **weg** | [[#git reset\|↓]] |
| `git revert <commit>` | Erzeugt einen **neuen** Commit, der die Änderung eines alten umkehrt. Sicher auf geteilten Branches | [[#git revert\|↓]] |
| `git clean -fd` | Löscht alle untracked Dateien und Ordner. Unwiderruflich | [[#git clean\|↓]] |

### Zwischenspeichern und parallel arbeiten

| Befehl | Was er tut | |
|---|---|---|
| `git stash` | Legt laufende Änderungen beiseite und hinterlässt ein sauberes Arbeitsverzeichnis | [[#git stash\|↓]] |
| `git stash pop` / `apply` | Holt die beiseitegelegten Änderungen zurück (`pop` entfernt sie danach aus der Liste) | [[#git stash\|↓]] |
| `git stash list` | Zeigt alle Stashes | [[#git stash\|↓]] |
| `git worktree add <pfad> <branch>` | Checkt einen zweiten Branch in einem zweiten Ordner aus, beide nutzen dasselbe Repo | [[#git worktree\|↓]] |

### Releases und Fehlersuche

| Befehl | Was er tut | |
|---|---|---|
| `git tag` / `git tag -a v1.0 -m "…"` | Setzt eine feste Markierung auf einen Commit (lightweight bzw. annotated) | [[#git tag\|↓]] |
| `git bisect` | Binäre Suche nach dem Commit, der einen Fehler eingeführt hat | [[#git bisect\|↓]] |

---

# Teil 2 — Low Level

## Vorab: der Werkzeugkasten

### Die Plumbing-Befehle

Git unterscheidet **Porcelain** (die bequemen Befehle aus Teil 1) und **Plumbing** (kleine Bausteine, die genau eine Sache tun). Jeder Porcelain-Befehl lässt sich aus diesen Bausteinen zusammensetzen:

| Befehl | Tut genau eins |
|---|---|
| `git hash-object -w <datei>` | Datei als Blob in `objects/` schreiben, Hash ausgeben |
| `git update-index` | Einen Eintrag im Index anlegen, ändern oder entfernen |
| `git ls-files --stage` | Index anzeigen |
| `git write-tree` | Aus dem Index die Trees bauen, Hash des Top-Level-Trees ausgeben |
| `git read-tree <tree>` | Einen Tree in den Index laden. Mit `-u` auch ins Arbeitsverzeichnis, mit `-m` als Merge mit lokalen Änderungen |
| `git checkout-index` | Dateien aus dem Index ins Arbeitsverzeichnis schreiben |
| `git commit-tree <tree> -p <parent>` | Ein Commit-Objekt schreiben, Hash ausgeben |
| `git update-ref <ref> <neu> [<alt>]` | Eine Ref auf einen Hash setzen. Mit `<alt>` nur, wenn sie noch auf `<alt>` zeigt |
| `git symbolic-ref HEAD <ref>` | `HEAD` auf einen Branch zeigen lassen |
| `git cat-file -t/-p <obj>` | Typ bzw. Inhalt eines Objekts anzeigen |
| `git ls-tree <tree>` | Einträge eines Trees anzeigen |
| `git rev-parse <name>` | Einen Namen (`main`, `HEAD~2`, `v1.0`) in einen Hash auflösen |
| `git rev-list <bereich>` | Alle Commits eines Bereichs auflisten, z. B. `main..feature` |
| `git diff-files` / `diff-index` / `diff-tree` | Arbeitsverzeichnis ↔ Index / Index ↔ Commit / Tree ↔ Tree vergleichen |
| `git merge-base A B` | Den letzten gemeinsamen Vorfahren zweier Commits finden |
| `git merge-tree --write-tree A B` | Drei-Wege-Merge **nur im Objektspeicher** durchführen, Hash des Ergebnis-Trees ausgeben |

### Die wichtigsten Dateien unter `.git/`

| Pfad | Inhalt |
|---|---|
| `HEAD` | `ref: refs/heads/<branch>`, im detached-Zustand direkt ein Hash |
| `index` | Die Staging Area: binär, eine Zeile pro getrackter Datei (Modus, Blob-Hash, Stage-Nr., Pfad, Dateisystem-Metadaten) |
| `objects/xx/…` | Einzelne Objekte, zlib-komprimiert (*loose objects*) |
| `objects/pack/` | Viele Objekte gebündelt und delta-komprimiert (*packfiles*). Entstehen bei `fetch`, `clone` und `gc` |
| `refs/heads/` | Lokale Branches, eine Datei pro Branch mit einem Hash |
| `refs/remotes/<remote>/` | Remote-Tracking-Branches (`origin/main`) |
| `refs/tags/` | Tags |
| `refs/stash` | Der neueste Stash |
| `packed-refs` | Viele Refs gesammelt in einer Datei. Eine lose Datei in `refs/` hat Vorrang |
| `logs/` | Der Reflog: pro Ref eine Zeile pro Änderung |
| `config` | Repo-Einstellungen, u. a. Remotes und Tracking-Konfiguration |
| `ORIG_HEAD` | Der Stand von `HEAD` vor einer „gefährlichen“ Operation (`reset`, `merge`, `rebase`) |
| `MERGE_HEAD`, `CHERRY_PICK_HEAD`, `REVERT_HEAD` | Existieren nur während eines unterbrochenen Merges, Cherry-Picks oder Reverts |
| `rebase-merge/` | Zustand eines laufenden Rebase, inkl. des Plans `git-rebase-todo` |
| `FETCH_HEAD` | Was der letzte `fetch` geholt hat |

> [!note] Refs nicht per Hand lesen
> Refs können in `packed-refs` statt als einzelne Datei liegen, und ab Git 3.0 wird das *reftable*-Format Standard. In Skripten deshalb `git rev-parse` und `git update-ref` verwenden statt `cat` und `echo`. Zum Verstehen ist `cat` aber völlig in Ordnung.

---

## So sind die Abschnitte aufgebaut

Jeder Befehl hat drei Teile:

1. **Plumbing**: derselbe Effekt aus Low-Level-Bausteinen.
2. **`.git/` vorher → nachher**: die Liste der Dateien, die tatsächlich **NEU** entstehen, **GEÄNDERT** oder **GELÖSCHT** werden. So in einer Sandbox mit Git 2.53 beobachtet, die Beispiel-Hooks sind ausgeblendet.
3. **Was in den Dateien passiert**: pro Datei, was genau drinsteht oder sich ändert. Jeder Abschnitt erklärt das für sich, ohne dass du nach oben scrollen musst.

Zwei Formate tauchen dabei immer wieder auf:

- **Ref-Datei** (`HEAD`, `refs/heads/main` …): **eine Zeile**. Entweder ein Hash mit 40 Zeichen (`d819102cb2fd7945…`) oder eine symbolische Ref (`ref: refs/heads/main`).
- **Reflog-Zeile** (`logs/…`): Bei jeder Bewegung einer Ref **hängt** Git eine Zeile an, es wird nichts überschrieben:
  ```
  <alter Hash> <neuer Hash> Name <mail> <Unix-Zeit> <Zeitzone>␉<Grund>
  03f3189… d819102… Nick <nick@example.com> 1792657200 +0200	commit: loader: skip rows without price
  ```

---

## Einrichten

### git init

**Plumbing**: Ein minimal funktionsfähiges Repo besteht aus drei Ordnern und einer Datei:
```bash
mkdir -p .git/objects .git/refs/heads .git/refs/tags
echo "ref: refs/heads/main" > .git/HEAD
git status        # funktioniert: "On branch main, No commits yet"
```

**`.git/` vorher → nachher** (vorher gibt es `.git/` nicht)
```
NEU      .git/HEAD
NEU      .git/config
NEU      .git/description
NEU      .git/info/exclude
NEU      .git/hooks/*.sample            (Beispiel-Hooks, inaktiv)
NEU      .git/objects/  .git/refs/heads/  .git/refs/tags/   (leere Ordner)
```

**Was in den Dateien passiert**

| Datei | Inhalt danach |
|---|---|
| `HEAD` | `ref: refs/heads/main`. Zeigt also schon auf einen Branch, den es **noch nicht gibt** (*unborn branch*). Der Name kommt aus `init.defaultBranch` oder `-b` |
| `config` | Ein Abschnitt `[core]` mit `repositoryformatversion = 0`, `filemode = true`, `bare = false`, `logallrefupdates = true`. Die letzte Zeile sorgt dafür, dass Branches einen Reflog bekommen |
| `info/exclude` | Nur Kommentare. Funktioniert wie `.gitignore`, ist aber lokal und wird nie committet |
| `description` | Platzhaltertext für das alte Web-Interface `gitweb`, praktisch bedeutungslos |
| `hooks/` | Beispielskripte mit der Endung `.sample`. Sie werden erst aktiv, wenn man die Endung entfernt |

**Was es noch nicht gibt:** `refs/heads/main` (entsteht erst beim ersten Commit), `index` (beim ersten `add`), `logs/` (bei der ersten Ref-Bewegung). `objects/` ist leer.

### git clone

**Plumbing**: `clone` ist eine Abkürzung für:
```bash
git init <ordner> && cd <ordner>
git remote add origin <url>                  # Eintrag in .git/config
git fetch origin                             # Objekte + refs/remotes/origin/*
git switch -c main --track origin/main       # lokalen Branch anlegen und auschecken
```

**`.git/` vorher → nachher** (Neuanlage)
```
NEU      alles von git init
NEU      .git/objects/pack/pack-<hash>.pack + .idx    (bei lokalem Klon: einzelne Objekte als Hardlinks)
NEU      .git/packed-refs
NEU      .git/refs/heads/main
NEU      .git/refs/remotes/origin/HEAD
NEU      .git/index
NEU      .git/logs/HEAD
NEU      .git/logs/refs/heads/main
NEU      .git/logs/refs/remotes/origin/HEAD
GEÄNDERT .git/config
```

**Was in den Dateien passiert**

| Datei | Inhalt danach |
|---|---|
| `objects/pack/*.pack` | **Alle** Objekte des Servers in einer einzigen Datei, delta-komprimiert. Die `.idx`-Datei daneben ist das Inhaltsverzeichnis: Hash → Position im Pack |
| `packed-refs` | Eine Zeile pro Remote-Branch und Tag, z. B. `98cb830… refs/remotes/origin/main`. Für die oft vielen Refs des Servers ist eine gemeinsame Datei effizienter als je eine eigene |
| `refs/remotes/origin/HEAD` | `ref: refs/remotes/origin/main`. Merkt sich den Standard-Branch des Servers, deshalb funktioniert `git log origin` als Kurzform |
| `refs/heads/main` | Der Hash von `origin/main`. Der lokale Branch wird als **einzelne Datei** angelegt |
| `HEAD` | `ref: refs/heads/main` |
| `config` | Zusätzlich `[remote "origin"]` mit `url = …` und `fetch = +refs/heads/*:refs/remotes/origin/*` (die **Refspec**: Server-Branches landen unter `refs/remotes/origin/`). Dazu `[branch "main"]` mit `remote = origin` und `merge = refs/heads/main`: die **Tracking-Konfiguration** für `pull`/`push` ohne Argumente |
| `index` | Ein Eintrag pro Datei aus dem ausgecheckten Commit |
| `logs/HEAD`, `logs/refs/heads/main` | Die erste Zeile: `0000000… <hash> … clone: from <url>` |

### git config

**Plumbing**: `git config` *ist* bereits der Low-Level-Befehl. Er liest und schreibt INI-Dateien.

**`.git/` vorher → nachher**
```
GEÄNDERT .git/config        (git config …, Standard = --local)
— oder —
GEÄNDERT ~/.gitconfig       (git config --global …, liegt außerhalb von .git/)
```

**Was in den Dateien passiert**: Git fügt eine Zeile in den passenden Abschnitt ein oder ersetzt sie. Fehlt der Abschnitt, wird er angelegt:
```ini
[user]
	name = Nick
[pull]
	rebase = true
```
Es gibt drei Ebenen, die spätere gewinnt:

| Ebene | Datei | Option |
|---|---|---|
| System | `/etc/gitconfig` | `--system` |
| Benutzer | `~/.gitconfig` | `--global` |
| Repo | `.git/config` | `--local` (Standard) |

`git config --list --show-origin` zeigt, welcher Wert aus welcher Datei kommt. Außer der Config-Datei ändert sich nichts.

---

## Änderungen vorbereiten und festhalten

### git status

**Plumbing**: `status` sind drei Vergleiche, die du einzeln ausführen kannst:
```bash
git diff-index --cached HEAD                # HEAD-Tree ↔ Index           → "Changes to be committed"
git diff-files                              # Index ↔ Arbeitsverzeichnis   → "Changes not staged"
git ls-files --others --exclude-standard    # nicht im Index, nicht ignoriert → "Untracked"
```

**`.git/` vorher → nachher**
```
(meist nichts)
GEÄNDERT .git/index        (nur, wenn Git veraltete Dateisystem-Metadaten auffrischt)
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `HEAD` → `refs/heads/<branch>` → Commit → Trees | Wird **gelesen**: Das ist die Vergleichsbasis für „staged“. Gibt es noch keinen Commit, vergleicht Git mit einem leeren Tree, und alles ist „new file“ |
| `index` | Wird **gelesen** und gelegentlich **neu geschrieben**. Pro Eintrag speichert der Index Änderungszeit, Größe und Inode der Datei. Stimmen die mit dem Dateisystem überein, gilt die Datei ohne Hashen als unverändert. Stimmt nur die Zeit nicht (z. B. nach `touch`), hasht Git den Inhalt, stellt fest, dass er gleich ist, und **schreibt die neue Zeit in den Index**, damit es beim nächsten Mal schneller geht |
| `info/exclude`, `.gitignore` | Werden gelesen, um Untracked von Ignored zu unterscheiden |

Refs und Objekte werden nicht verändert.

### git add

**Plumbing**
```bash
h=$(git hash-object -w datei.py)                         # 1. Blob schreiben
git update-index --add --cacheinfo 100644,$h,datei.py    # 2. Index-Eintrag setzen
# beides in einem:  git update-index --add datei.py
```

**`.git/` vorher → nachher**
```
NEU      .git/objects/xx/…        ← ein Blob pro Datei mit neuem Inhalt
GEÄNDERT .git/index               (bzw. NEU beim allerersten add)
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/xx/…` | Pro Datei, deren Inhalt es noch nicht als Blob gibt, **eine neue Datei**: `blob <größe>\0<inhalt>`, mit zlib komprimiert. Der Dateiname ist der SHA-1 davon, die ersten 2 Zeichen bilden den Ordner. Existiert der Inhalt schon (z. B. eine Datei, die wieder auf einen alten Stand zurückgesetzt wurde), entsteht **nichts** |
| `index` | Der Eintrag für den Pfad zeigt jetzt auf den neuen Blob: `100644 <blob-hash> 0 src/loader.py`, plus frische Dateisystem-Metadaten. Bei einer neuen Datei kommt eine Zeile dazu. Außerdem verwirft Git die gemerkten Tree-Hashes (*Cache-Tree*) für die Ordner auf dem Pfad, denn diese Trees müssen beim nächsten Commit neu gebaut werden |

Es entstehen **keine Trees und keine Commits**, keine Ref bewegt sich.

**Zwei Feinheiten**
- Addest du dieselbe Datei zweimal mit verschiedenem Inhalt, zeigt der Index nur auf den zweiten Blob. Der erste bleibt als **dangling blob** in `objects/` liegen. `git fsck --lost-found` findet ihn, bis `gc` ihn entfernt.
- **`git add -p`** schreibt einen Blob für einen **Zwischenstand**: den alten Inhalt plus die gewählten Hunks. Diesen Inhalt gibt es weder in `HEAD` noch auf der Platte. Deshalb kann dieselbe Datei gleichzeitig staged *und* unstaged sein.

### git restore

**Plumbing**
```bash
# git restore datei            → Index ⇒ Arbeitsverzeichnis
git checkout-index -f -- datei

# git restore --staged datei   → HEAD ⇒ Index
git ls-tree HEAD datei                                     # Modus + Blob-Hash aus dem letzten Commit holen
git update-index --cacheinfo 100644,<blob-hash>,datei      # Index-Eintrag zurücksetzen
# war die Datei in HEAD noch gar nicht vorhanden:
git update-index --force-remove datei
```

**`.git/` vorher → nachher**
```
git restore datei:            GEÄNDERT .git/index   (nur Metadaten)    + Datei im Arbeitsverzeichnis überschrieben
git restore --staged datei:   GEÄNDERT .git/index   (Eintrag zurückgesetzt)
```

**Was in den Dateien passiert**

| Datei | `restore datei` | `restore --staged datei` |
|---|---|---|
| Datei im Arbeitsverzeichnis | **Überschrieben** mit dem Inhalt des Blobs, auf den der Index zeigt | unverändert |
| `index` | Eintrag zeigt auf denselben Blob wie vorher, nur Zeit und Größe werden an die frisch geschriebene Datei angepasst | Der Eintrag zeigt wieder auf den Blob aus `HEAD`. War die Datei neu, wird die Zeile entfernt |
| `objects/` | unverändert | unverändert. Der Blob der zurückgenommenen Fassung bleibt als dangling blob liegen |

> [!warning] Ohne Option endgültig
> Für den überschriebenen ungestagten Inhalt gab es nie einen Blob, und kein Reflog hat ihn gesehen. Er ist weg.

Mit `--source <commit>` holt `restore` den Stand aus einem beliebigen Commit statt aus Index bzw. `HEAD`.

### git rm und git mv

**Plumbing**
```bash
# git rm datei
git update-index --force-remove datei && rm datei
# git rm --cached datei   (Datei bleibt auf der Platte, ist danach untracked)
git update-index --force-remove datei

# git mv alt neu
mv alt neu
git update-index --remove alt     # entfernt den Eintrag, weil die Datei fehlt
git update-index --add neu
```

**`.git/` vorher → nachher**
```
GEÄNDERT .git/index
```

**Was in den Dateien passiert**

| Datei | `rm` | `rm --cached` | `mv alt neu` |
|---|---|---|---|
| `index` | Zeile für `datei` entfernt | Zeile für `datei` entfernt | Zeile `alt` entfernt, Zeile `neu` mit **demselben Blob-Hash** eingefügt (an der alphabetisch richtigen Stelle) |
| Arbeitsverzeichnis | Datei gelöscht | Datei bleibt, ist ab jetzt **untracked** | Datei umbenannt |
| `objects/` | unverändert | unverändert | unverändert. Der Inhalt ist ja derselbe |

**Git speichert keine Umbenennungen.** Im nächsten Commit fehlt im Tree der Eintrag `alt` und es gibt einen Eintrag `neu`, der auf denselben Blob zeigt. Der Commit erzeugt deshalb nur neue Trees und keinen neuen Blob. `log`, `diff` und `status` *erkennen* die Umbenennung erst beim Anzeigen über die Ähnlichkeit der Inhalte (Standard: ≥ 50 %).

### git commit

**Plumbing**
```bash
tree=$(git write-tree)                                   # 1. Index → Trees (von unten nach oben)
commit=$(echo "loader: skip rows without price" \
         | git commit-tree $tree -p HEAD)                # 2. Commit-Objekt (beim ersten Commit ohne -p)
git update-ref -m "commit: loader: skip …" HEAD $commit  # 3. Branch weitersetzen
```

**`.git/` vorher → nachher**
```
NEU      .git/objects/xx/…              ← je ein Tree pro geändertem Ordner (bis zur Wurzel)
NEU      .git/objects/xx/…              ← das Commit-Objekt
GEÄNDERT .git/refs/heads/<branch>       (NEU beim ersten Commit)
GEÄNDERT .git/logs/HEAD                 (NEU beim ersten Commit)
GEÄNDERT .git/logs/refs/heads/<branch>  (NEU beim ersten Commit)
GEÄNDERT .git/COMMIT_EDITMSG            (NEU beim ersten Commit)
GEÄNDERT .git/index
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` — Trees | Git baut aus dem flachen Index die Ordnerstruktur, **von unten nach oben**: Der übergeordnete Tree braucht den Hash des Unterordners. Jeder Tree enthält pro Eintrag `<modus> <typ> <hash>␉<name>`, z. B. `040000 tree 99cca54…␉src`. **Nur Ordner mit Änderungen** (und ihre Eltern bis zur Wurzel) bekommen einen neuen Tree. Unveränderte Ordner werden per Hash wiederverwendet |
| `objects/` — Commit | Ein neues Objekt: `tree <wurzel-tree>`, `parent <bisheriger Commit>` (fehlt beim allerersten Commit), `author …`, `committer …`, Leerzeile, Message |
| `refs/heads/<branch>` | Der alte Hash wird **überschrieben** mit dem neuen Commit-Hash. Beim ersten Commit wird die Datei angelegt, der Branch wird „geboren“. Welcher Branch es ist, bestimmt `HEAD` |
| `HEAD` | **Unverändert** (`ref: refs/heads/<branch>`). Git löst ihn nur auf, um den Branch zu finden |
| `logs/HEAD`, `logs/refs/heads/<branch>` | Je eine angehängte Zeile `<alt> <neu> … commit: <betreff>`. Beim ersten Commit steht als alter Wert `0000…` und als Grund `commit (initial): …` |
| `COMMIT_EDITMSG` | Die zuletzt verwendete Message. Nur ein Hilfsmittel für den Editor |
| `index` | **Die Einträge bleiben gleich**, der Index ist jetzt identisch mit dem neuen Commit-Tree. Geschrieben wird er trotzdem, weil Git die gerade berechneten Tree-Hashes als *Cache-Tree* darin ablegt. Beim nächsten Commit muss es dann nur die Trees geänderter Ordner neu berechnen |

Author und Committer kommen aus `user.name`/`user.email` (bzw. den Umgebungsvariablen `GIT_AUTHOR_*`/`GIT_COMMITTER_*`).

### git commit --amend

**Plumbing**: wie `commit`, nur mit den **Eltern des alten** Commits statt mit ihm selbst:
```bash
tree=$(git write-tree)
neu=$(echo "Neue Message" | git commit-tree $tree -p HEAD~1)   # Eltern von HEAD, nicht HEAD
git update-ref -m "commit (amend): …" HEAD $neu
```

**`.git/` vorher → nachher**
```
NEU      .git/objects/xx/…              ← neuer Commit (+ neue Trees, falls sich Inhalt geändert hat)
GEÄNDERT .git/refs/heads/<branch>
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/logs/refs/heads/<branch>
GEÄNDERT .git/COMMIT_EDITMSG
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` | Ändert sich **nur die Message**, entsteht genau **ein** neues Objekt, der Commit. Er zeigt auf denselben Tree wie der alte. Mit neuem Inhalt entstehen zusätzlich die Trees wie bei `commit` (die Blobs gibt es seit `add`) |
| Der alte Commit | **Bleibt unverändert in `objects/`**. Ein Objekt kann nicht geändert werden, denn sein Hash ist sein Inhalt. Er ist danach nur noch über den Reflog erreichbar |
| `refs/heads/<branch>` | Überschrieben: alter Commit-Hash → neuer. Der neue Commit hat **denselben Parent** wie der alte, ersetzt ihn also an derselben Stelle der Kette |
| `logs/HEAD`, `logs/refs/heads/<branch>` | Neue Zeile `<alter Commit> <neuer Commit> … commit (amend): <betreff>`. Über diese Zeile findest du den alten Commit wieder (`HEAD@{1}`) |

Deshalb gilt: **nach dem Push nicht mehr amenden.** Wer den alten Commit schon hat, hat danach eine andere Historie als du.

### git diff

**Plumbing**
```bash
git diff-files -p                  # git diff            (Index ↔ Arbeitsverzeichnis)
git diff-index -p --cached HEAD    # git diff --staged   (HEAD ↔ Index)
git diff-tree -p A B               # git diff A B        (Tree ↔ Tree)
```

**`.git/` vorher → nachher**
```
(nichts)
```

**Was in den Dateien passiert**: Es wird nur **gelesen**: `index`, die Trees und Blobs der beteiligten Commits und die Dateien im Arbeitsverzeichnis. Der Diff wird **jedes Mal neu berechnet**, denn Git speichert keine Diffs, nur Snapshots. Das geht schnell, weil Einträge mit gleichem Hash sofort übersprungen werden. Bei einem unveränderten Unterordner reicht ein einziger Vergleich der Tree-Hashes, und Git steigt gar nicht erst hinab.

---

## Historie ansehen

### git log und git show

**Plumbing**
```bash
git rev-list HEAD                         # alle erreichbaren Commits, neueste zuerst
git rev-list HEAD | while read c; do git cat-file -p $c; done
# git show <commit>
git cat-file -p <commit>                  # Metadaten + Message
git diff-tree -p <commit>                 # Diff zum Elternteil
```

**`.git/` vorher → nachher**
```
(nichts)
```

**Was in den Dateien passiert**: Es wird nur gelesen, in dieser Reihenfolge:

1. `HEAD` → `ref: refs/heads/main` → `refs/heads/main` (oder `packed-refs`) → Commit-Hash.
2. Das Commit-Objekt aus `objects/` entpacken, anzeigen, und den `parent`-Zeilen zum nächsten folgen. Bei einem Merge-Commit verzweigt die Suche, `log` sortiert dann nach Datum.
3. Mit `-p`, `--stat` oder `-S`: zusätzlich pro Commit Tree und Parent-Tree vergleichen.

Daraus folgt: **Was von keiner Ref erreichbar ist, taucht in `log` nicht auf**, auch wenn es noch in `objects/` liegt. `--all` startet bei *allen* Refs unter `refs/` (inkl. `refs/remotes/`, `refs/tags/`, `refs/stash`). `log -S <text>` behält nur Commits, in denen sich die Anzahl der Vorkommen von `<text>` ändert. Damit findet man den Commit, der eine Funktion eingeführt oder entfernt hat.

### git blame

**Plumbing**: Dafür gibt es keinen Baustein. `blame` läuft von `HEAD` rückwärts durch die `parent`-Kette, berechnet pro Commit den Diff der Datei und ordnet jede Zeile dem ältesten Commit zu, in dem sie in dieser Form auftauchte. Ähnlich, aber für einen Zeilenbereich arbeitet `git log -L 10,20:datei.py`.

**`.git/` vorher → nachher**
```
(nichts)
```

**Was in den Dateien passiert**: Es werden nur Commits, Trees und Blobs entlang der Historie gelesen. Bei langer Historie kann das dauern, weil Git für jeden Commit die Datei-Blobs vergleichen muss.

### git reflog

**Plumbing**: `reflog` ist im Kern eine formatierte Anzeige der Datei `.git/logs/HEAD` (bzw. `git reflog show main` → `.git/logs/refs/heads/main`).

**`.git/` vorher → nachher**
```
(nichts)    — git reflog expire / delete schreiben die logs/-Dateien um
```

**Was in den Dateien passiert**: `logs/HEAD` wird gelesen, **von unten nach oben**, denn die neueste Zeile steht am Ende der Datei. Jede Zeile hat die Form:
```
<alter Hash> <neuer Hash> Name <mail> <Unix-Zeit> <Zeitzone>␉<Grund>
```
- `HEAD@{0}` ist die letzte Zeile, `HEAD@{2}` die drittletzte. `main@{yesterday}` sucht nach Zeitstempel.
- Typische Gründe: `commit:`, `commit (amend):`, `checkout: moving from main to feature`, `merge feature: Fast-forward`, `rebase (finish): …`, `reset: moving to HEAD~1`, `pull:`, `clone: from …`.
- `logs/` ist **rein lokal**, wird nie gepusht und läuft ab: Einträge zu unerreichbaren Commits nach 30 Tagen, alle anderen nach 90 (`gc.reflogExpireUnreachable`, `gc.reflogExpire`).
- Solange eine Reflog-Zeile einen Commit nennt, löscht `gc` ihn nicht. **Das ist das Sicherheitsnetz** nach einem missglückten Rebase oder `reset --hard`.

---

## Branches

### git branch

**Plumbing**
```bash
git for-each-ref refs/heads/                 # git branch          (auflisten)
git update-ref refs/heads/feature HEAD       # git branch feature  (anlegen, am aktuellen Commit)
git update-ref -d refs/heads/feature         # git branch -D feature
```

**`.git/` vorher → nachher**
```
git branch feature:       NEU      .git/refs/heads/feature
                          NEU      .git/logs/refs/heads/feature
git branch -m feat neu:   GELÖSCHT .git/refs/heads/feat        + .git/logs/refs/heads/feat
                          NEU      .git/refs/heads/neu         + .git/logs/refs/heads/neu
git branch -d feature:    GELÖSCHT .git/refs/heads/feature
                          GELÖSCHT .git/logs/refs/heads/feature
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `refs/heads/feature` | **Anlegen:** Eine neue Datei mit dem Hash des aktuellen Commits und einem Zeilenumbruch, insgesamt 41 Bytes. Ein Schrägstrich im Namen (`feature/x`) wird zum Unterordner. **Löschen:** Die Datei verschwindet. Lag der Branch in `packed-refs`, wird dort stattdessen die Zeile entfernt |
| `logs/refs/heads/feature` | Beim Anlegen eine erste Zeile `0000… <hash> … branch: Created from main` (bzw. dem angegebenen Startpunkt). Beim Löschen verschwindet der Reflog **mit** dem Branch. Danach kennt nur noch `logs/HEAD` die Commits |
| `HEAD` | **Unverändert!** Du bist weiterhin auf dem alten Branch, der nächste Commit verschiebt *ihn* |
| `config` | Nur, wenn der Branch einen `[branch "…"]`-Abschnitt hatte (Tracking). Bei `-m` wird er umbenannt, bei `-d` entfernt |
| `objects/` | Unverändert. Auch beim Löschen bleiben die Commits erhalten. `-d` prüft vorher, ob sie in einen anderen Branch gemergt sind, `-D` nicht |

### git switch und git checkout

**Plumbing**
```bash
# git switch feature
git read-tree -u -m HEAD feature          # Index + Arbeitsverzeichnis auf den Tree von feature bringen
git symbolic-ref HEAD refs/heads/feature  # HEAD umhängen

# git switch --detach <hash>   bzw.   git checkout <hash>
git read-tree -u -m HEAD <hash>
git update-ref --no-deref HEAD <hash>     # HEAD enthält jetzt direkt einen Hash
```

**`.git/` vorher → nachher**
```
GEÄNDERT .git/HEAD
GEÄNDERT .git/logs/HEAD
GEÄNDERT .git/index          (nur, wenn sich die Trees der beiden Commits unterscheiden)
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `HEAD` | Überschrieben: `ref: refs/heads/main` → `ref: refs/heads/feature`. Bei `--detach` bzw. `checkout <hash>` steht danach **direkt ein Hash** darin. Das ist der *detached HEAD*: Es gibt keinen Branch, den ein Commit weiterschieben könnte |
| `logs/HEAD` | Neue Zeile `<alter Commit> <neuer Commit> … checkout: moving from main to feature`. Daraus liest `git switch -` den vorherigen Branch |
| `index` | Bekommt die Einträge des Ziel-Trees: Dateien, die es dort nicht gibt, verlieren ihre Zeile, andere bekommen andere Blob-Hashes. Zeigen beide Branches auf denselben Commit, bleibt er unverändert |
| Arbeitsverzeichnis | Wird passend umgeschrieben: Dateien gelöscht, angelegt, überschrieben. Lokale Änderungen an Dateien, die zwischen den Branches gleich sind, werden **mitgenommen** (`-m` = Zwei-Wege-Merge). Würden sie überschrieben, bricht Git ab |
| `refs/heads/*` | **Unverändert.** `switch` bewegt `HEAD`, nie einen Branch |
| `objects/` | Unverändert. Die Dateien aus dem anderen Branch sind nicht weg, sie liegen weiter als Blobs vor |

`checkout` kann beides, Branch wechseln und Dateien wiederherstellen (`checkout -- datei` entspricht `restore datei`). Seit Git 2.23 ist das auf `switch` und `restore` aufgeteilt.

### git merge

**Plumbing**, Fall 1: **Fast-Forward**. Der aktuelle Branch ist Vorfahre des anderen, es gibt also nichts zusammenzuführen:
```bash
git merge-base --is-ancestor HEAD feature && {
  git read-tree -u -m HEAD feature
  git update-ref -m "merge feature: Fast-forward" HEAD feature
}
```

Fall 2: **echter Merge**. Beide Seiten haben eigene Commits:
```bash
git merge-base HEAD feature                         # gemeinsamer Vorfahre (die "Basis")
tree=$(git merge-tree --write-tree HEAD feature)    # Drei-Wege-Merge im Objektspeicher; Exit-Code 1 = Konflikt
commit=$(echo "Merge branch 'feature'" | git commit-tree $tree -p HEAD -p feature)
git read-tree -u -m HEAD $commit                    # Arbeitsverzeichnis nachziehen
git update-ref HEAD $commit
```

**`.git/` vorher → nachher**
```
Fast-Forward:        GEÄNDERT .git/refs/heads/main, .git/ORIG_HEAD, .git/index,
                              .git/logs/HEAD, .git/logs/refs/heads/main
Echter Merge:        zusätzlich NEU .git/objects/xx/…  (Trees + Merge-Commit)
Mit Konflikt:        NEU      .git/MERGE_HEAD, .git/MERGE_MSG, .git/MERGE_MODE, .git/AUTO_MERGE
                     NEU      .git/objects/xx/…  (Blob mit Konfliktmarkern + Tree)
                     GEÄNDERT .git/ORIG_HEAD, .git/index
                     — refs/heads/main bleibt unverändert —
merge --abort:       GELÖSCHT .git/MERGE_HEAD, MERGE_MSG, MERGE_MODE, AUTO_MERGE
                     GEÄNDERT .git/index, .git/logs/HEAD
```

**Was in den Dateien passiert — ohne Konflikt**

| Datei | Was passiert |
|---|---|
| `ORIG_HEAD` | Bekommt den Hash, auf dem `main` **vor** dem Merge stand. `git reset --hard ORIG_HEAD` macht den Merge rückgängig |
| `objects/` | **Fast-Forward: nichts.** Echter Merge: neue Trees nur für Ordner, deren Ergebnis keiner der beiden Seiten entspricht, plus ein Commit mit **zwei `parent`-Zeilen** (erst `HEAD`, dann der gemergte Branch). Die zwei Parents **sind** der Merge, sonst unterscheidet sich das Objekt durch nichts von einem normalen Commit |
| `refs/heads/main` | Überschrieben: beim Fast-Forward mit dem Hash von `feature` („vorgespult“), beim echten Merge mit dem Hash des Merge-Commits |
| `index`, Arbeitsverzeichnis | Auf den Ergebnis-Tree gebracht |
| `logs/HEAD`, `logs/refs/heads/main` | Neue Zeile mit dem Grund `merge feature: Fast-forward` bzw. `merge feature: Merge made by the 'ort' strategy.` |
| `refs/heads/feature` | Unverändert. Der gemergte Branch bewegt sich nie |

**Was in den Dateien passiert — mit Konflikt**

Git hält **vor** dem Commit an. Der Branch bewegt sich nicht, stattdessen wird der Zwischenzustand in Dateien festgehalten:

| Datei | Inhalt |
|---|---|
| `MERGE_HEAD` | Hash des Commits, der hereingemergt wird. Seine Existenz bedeutet: „Ein Merge läuft.“ `git status` sagt deshalb „You have unmerged paths“ |
| `MERGE_MSG` | Die vorbereitete Commit-Message `Merge branch 'feature'` plus eine Liste `# Conflicts:` mit den betroffenen Dateien. Der Editor zeigt sie beim abschließenden Commit |
| `MERGE_MODE` | Leer bzw. Optionen wie `no-ff`, damit der spätere Commit weiß, wie gemergt wurde |
| `AUTO_MERGE` | Hash eines Trees mit dem automatischen Merge-Ergebnis, **inklusive Konfliktmarkern**. Dafür entstehen die zwei neuen Objekte (Blob + Tree). `git diff AUTO_MERGE` zeigt, was du beim Auflösen geändert hast |
| `ORIG_HEAD` | Stand vor dem Merge |
| `index` | Für jede konfliktbehaftete Datei **drei Zeilen statt einer**: Stage 1 = Basis, 2 = ours (`HEAD`), 3 = theirs. Konfliktfreie Dateien stehen schon mit ihrem Ergebnis als Stage 0 drin |
| Arbeitsverzeichnis | Die Konfliktdatei mit `<<<<<<<`, `=======` und `>>>>>>>` |

Danach:
- `git add datei` ersetzt die drei Stage-Zeilen durch eine Zeile mit Stage 0 und schreibt dafür einen neuen Blob.
- `git commit` baut den Merge-Commit mit zwei Parents (der zweite stammt aus `MERGE_HEAD`), setzt `main` weiter und **löscht `MERGE_*` und `AUTO_MERGE`**.
- `git merge --abort` setzt Index und Arbeitsverzeichnis auf `HEAD` zurück und löscht die Dateien ebenfalls.

### git rebase

**Plumbing**: Ein Rebase ist eine Folge von Cherry-Picks auf einem detached `HEAD`. Erst ganz am Ende wird der Branch umgesetzt:
```bash
basis=$(git rev-parse main)
git update-ref --no-deref HEAD $basis            # HEAD detached auf die neue Basis
for c in $(git rev-list --reverse main..feature); do
  # = git cherry-pick $c  (siehe dort)
done
git update-ref refs/heads/feature HEAD           # Branch auf das Ergebnis setzen
git symbolic-ref HEAD refs/heads/feature         # HEAD wieder an den Branch hängen
```
`git replay --onto main main..feature` erledigt den Objekt-Teil davon ohne Arbeitsverzeichnis. Der Befehl ist in Git 2.53 noch als **experimentell** markiert.

**`.git/` vorher → nachher**
```
Ohne Konflikt:     NEU      .git/objects/xx/…   (ein neuer Commit + ggf. Trees pro umgesetztem Commit)
                   GEÄNDERT .git/refs/heads/feature, .git/ORIG_HEAD, .git/index,
                            .git/logs/HEAD, .git/logs/refs/heads/feature
Bei Konflikt:      NEU      .git/rebase-merge/…  (Zustandsordner, siehe unten)
                   NEU      .git/REBASE_HEAD, .git/MERGE_MSG, .git/AUTO_MERGE
                   GEÄNDERT .git/HEAD            ← jetzt ein nackter Hash (detached)
                   GEÄNDERT .git/index, .git/ORIG_HEAD, .git/logs/HEAD
                   — refs/heads/feature bleibt unverändert —
rebase --abort:    GELÖSCHT alles obige, HEAD wieder "ref: refs/heads/feature"
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `ORIG_HEAD` | Der Stand von `feature` **vor** dem Rebase. `git reset --hard ORIG_HEAD` macht den ganzen Rebase rückgängig |
| `HEAD` | Während des Rebase steht hier ein **nackter Hash**, also detached. Er wandert mit jedem umgesetzten Commit weiter. Am Ende steht wieder `ref: refs/heads/feature` darin. Deshalb zeigt `git branch` mitten im Rebase „(no branch, rebasing feature)“ |
| `objects/` | Pro umgesetztem Commit ein **neuer** Commit: gleiche Message, gleicher Author, aber neuer `parent` und dadurch **neuer Hash**. Neue Trees nur, wo sich der Inhalt durch die neue Basis ergibt. Die alten Commits bleiben unverändert liegen |
| `refs/heads/feature` | Wird **erst ganz am Ende einmal** überschrieben: alter Spitzen-Commit → neuer Spitzen-Commit. Während eines Konflikts steht er noch auf dem alten Stand |
| `logs/HEAD` | Mehrere Zeilen: `rebase (start): checkout main`, je eine `rebase (pick): <betreff>` pro Commit, `rebase (finish): returning to refs/heads/feature` |
| `logs/refs/heads/feature` | **Eine** Zeile `rebase (finish): refs/heads/feature onto <hash>`. Darüber kommst du mit `feature@{1}` an den alten Stand |

**Bei einem Konflikt** hält Git an und legt den Ordner **`.git/rebase-merge/`** an:

| Datei in `rebase-merge/` | Inhalt |
|---|---|
| `git-rebase-todo` | Die **noch ausstehenden** Schritte (`pick <hash> <betreff>`). Bei `rebase -i` ist das die Datei aus dem Editor |
| `done` | Die bereits erledigten Schritte, der letzte ist der fehlgeschlagene |
| `onto` | Hash der neuen Basis |
| `orig-head` | Hash des Branches vor dem Rebase, das Ziel von `--abort` |
| `head-name` | `refs/heads/feature`: Welcher Branch am Ende umgesetzt wird |
| `stopped-sha`, `message`, `author-script`, `patch` | Der Commit, an dem es hakt: Hash, Message, Author-Daten und Diff |
| `msgnum`, `end` | Fortschritt „Schritt 2 von 5“ |

Dazu kommen außerhalb des Ordners `REBASE_HEAD` (der Commit, der gerade nicht passt, auch über `git show REBASE_HEAD`), `MERGE_MSG`, `AUTO_MERGE` und die drei Stage-Zeilen im `index`. Beim Rebase ist **Stage 2 („ours“) die neue Basis und Stage 3 („theirs“) dein eigener Commit**, umgekehrt zum Merge. Der Grund ist, dass `HEAD` auf der Basis steht.

- `--continue` macht aus dem aufgelösten Index den Commit und arbeitet `git-rebase-todo` weiter ab.
- `--skip` streicht den aktuellen Schritt.
- `--abort` schreibt `orig-head` zurück, hängt `HEAD` wieder an den Branch und löscht den Ordner.

### git rebase -i

**Plumbing**: wie `rebase`, nur dass Git vorher `.git/rebase-merge/git-rebase-todo` **in deinem Editor öffnet**. Der Rebase-Plan aus [[Quiz 1 - Commits, Objekte & Rebase|Quiz 1]] *ist* diese Datei. Die Befehle darin bedeuten auf Objektebene:

| Befehl | Low Level |
|---|---|
| `pick` | Cherry-Pick: neuer Commit mit gleichem Diff auf dem aktuellen `HEAD` |
| `reword` | Wie `pick`, aber `commit-tree` bekommt die neue Message |
| `edit` | Nach dem `pick` anhalten. Du arbeitest auf dem detached `HEAD` weiter |
| `squash` / `fixup` | Den Diff anwenden, dann **den vorherigen Commit ersetzen**: `commit-tree <neuer Tree> -p HEAD~1` (wie `--amend`). Die Message wird kombiniert bzw. die alte behalten |
| `drop` / Zeile löschen | Den Commit einfach nicht cherry-picken |
| Zeilen tauschen | Cherry-Picks in anderer Reihenfolge |

**`.git/` vorher → nachher**: wie bei [[#git rebase|git rebase]]. Hält er an (Konflikt oder `edit`), steht der Zustand in `.git/rebase-merge/`.

**Was in den Dateien passiert**, zusätzlich zu `rebase`:

| Datei | Was passiert |
|---|---|
| `rebase-merge/git-rebase-todo` | Wird **vor** dem Öffnen des Editors geschrieben (bei `--autosquash` schon umsortiert), danach von Git wieder eingelesen. Was du dort speicherst, ist der Plan. Bei `edit` kannst du ihn mit `git rebase --edit-todo` unterwegs noch ändern |
| `rebase-merge/git-rebase-todo.backup` | Der ursprüngliche, unveränderte Plan |
| `objects/` | `fixup`/`squash`: Die Zwischenstände entstehen als Commits, am Ende zählt nur der zusammengefasste. `drop`: für diesen Commit entsteht nichts. Unveränderte `pick`-Zeilen am **Anfang** des Plans werden wiederverwendet, denn ihr Parent stimmt ja schon. Ab der ersten Änderung entstehen neue Commits |
| `logs/HEAD` | Pro Schritt eine Zeile, z. B. `rebase (fixup): d` und `rebase (reword): …` |

**`commit --fixup <c>`** ist ein ganz normaler Commit mit der Message `fixup! <Betreff von c>`. Erst `rebase -i --autosquash` liest diese Messages und sortiert `git-rebase-todo` vor dem Öffnen um.

### git cherry-pick

**Plumbing**: Drei-Wege-Merge mit dem **Parent des Commits als Basis**. Übertragen wird also genau das, was der Commit gegenüber seinem Parent geändert hat:
```bash
c=<commit>
tree=$(git merge-tree --write-tree --merge-base $c~1 HEAD $c)
commit=$(git log -1 --format=%B $c | \
  GIT_AUTHOR_NAME="$(git log -1 --format=%an $c)" \
  GIT_AUTHOR_EMAIL="$(git log -1 --format=%ae $c)" \
  GIT_AUTHOR_DATE="$(git log -1 --format=%ad $c)" \
  git commit-tree $tree -p HEAD)
git read-tree -u -m HEAD $commit
git update-ref HEAD $commit
```

**`.git/` vorher → nachher**
```
Ohne Konflikt:   NEU      .git/objects/xx/…   (neuer Commit, ggf. neue Trees)
                 GEÄNDERT .git/refs/heads/main, .git/index, .git/logs/HEAD, .git/logs/refs/heads/main
                 NEU      .git/AUTO_MERGE      (Nebenprodukt der Merge-Maschinerie, harmlos)
Mit Konflikt:    NEU      .git/CHERRY_PICK_HEAD, .git/MERGE_MSG, .git/AUTO_MERGE
                 GEÄNDERT .git/index          — refs/heads/main bleibt unverändert —
Mehrere Commits: zusätzlich .git/sequencer/   (Liste der noch ausstehenden Picks)
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` | Ein neuer Commit mit derselben Message und demselben **Author** (inkl. Author-Datum). **Committer** bist du mit dem aktuellen Zeitpunkt, `parent` ist dein `HEAD`. Gibt es den Ergebnis-Tree schon, entsteht nur der Commit |
| `refs/heads/main` | Überschrieben mit dem Hash des neuen Commits |
| `logs/HEAD`, `logs/refs/heads/main` | Neue Zeile `cherry-pick: <betreff>` |
| `CHERRY_PICK_HEAD` | Nur bei Konflikt: Hash des Commits, der übernommen werden soll. `git cherry-pick --continue` liest daraus Author und Message |
| `index` | Bei Konflikt mit den drei Stage-Zeilen wie beim Merge |

Daher zeigt `git log --format=fuller` bei cherry-gepickten und gerebasten Commits zwei verschiedene Daten: Author-Date (ursprünglich) und Commit-Date (jetzt).

---

## Zusammenarbeiten

### git remote

**Plumbing**: `git remote add origin <url>` schreibt nur in die Config:
```bash
git config remote.origin.url <url>
git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
```

**`.git/` vorher → nachher**
```
GEÄNDERT .git/config
```

**Was in den Dateien passiert**: In `config` kommt ein neuer Abschnitt:
```ini
[remote "origin"]
	url = git@gitlab.example.com:team/preisimport.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```
Die zweite Zeile ist die **Refspec**: „Jeder Branch `refs/heads/X` auf dem Server landet bei mir als `refs/remotes/origin/X`.“ Das `+` erlaubt, diese lokalen Kopien auch dann zu überschreiben, wenn der Server einen Branch per Force-Push zurückgesetzt hat.

Es wird **kein Kontakt** zum Server aufgenommen, und es entstehen keine Refs. `refs/remotes/origin/` gibt es erst nach dem ersten `fetch` oder `push`. `git remote remove origin` löscht den Abschnitt **und** alle `refs/remotes/origin/*`. `git remote rename` benennt beides um.

### git fetch

**Plumbing**
```bash
git ls-remote origin           # 1. Refs des Servers abfragen (Name → Hash)
# 2. Git handelt aus, welche Objekte fehlen, und lädt sie als ein Packfile herunter
# 3. Für jeden Server-Branch laut Refspec:
git update-ref refs/remotes/origin/main <hash-vom-server>
```

**`.git/` vorher → nachher**
```
NEU      .git/objects/…                       ← nur die fehlenden Objekte
GEÄNDERT .git/refs/remotes/origin/<branch>    (NEU für neue Server-Branches)
GEÄNDERT .git/logs/refs/remotes/origin/<branch>
NEU      .git/refs/remotes/origin/HEAD        (falls noch nicht vorhanden)
GEÄNDERT .git/FETCH_HEAD
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` | Git und Server vergleichen, welche Commits beide schon kennen. Übertragen werden **nur die fehlenden** Commits, Trees und Blobs, als ein Packfile. Unter 100 Objekten (`fetch.unpackLimit`) entpackt Git sie zu einzelnen Dateien in `objects/xx/`, sonst bleibt das Pack als `objects/pack/pack-*.pack` + `.idx` erhalten |
| `refs/remotes/origin/main` | Überschrieben mit dem Hash, auf dem `main` **auf dem Server** steht. Das ist deine Notiz „so sah es dort zuletzt aus“. Lag die Ref bisher in `packed-refs`, entsteht jetzt eine eigene Datei, und die hat Vorrang |
| `logs/refs/remotes/origin/main` | Neue Zeile `<alt> <neu> … fetch: fast-forward` (hinter `fetch` steht der Befehl so, wie du ihn aufgerufen hast, z. B. `fetch origin:`) (bzw. `forced-update`, wenn der Server-Branch zurückgesetzt wurde) |
| `refs/remotes/origin/HEAD` | `ref: refs/remotes/origin/main`. Seit Git 2.48 legt `fetch` ihn an, falls er fehlt |
| `FETCH_HEAD` | Eine Zeile pro geholtem Branch: `<hash>␉<leer oder not-for-merge>␉branch 'main' of <url>`. Die Zeilen ohne `not-for-merge` sind das, was ein anschließendes `merge FETCH_HEAD` (also der zweite Teil von `pull`) zusammenführen würde |

**Was garantiert unverändert bleibt:** alles unter `refs/heads/`, `HEAD`, `index` und das Arbeitsverzeichnis. Deshalb ist `fetch` immer gefahrlos.

### git pull

**Plumbing**: genau zwei Porcelain-Befehle hintereinander:
```bash
git fetch origin
git merge origin/main          # Standard
git rebase origin/main         # mit --rebase bzw. pull.rebase=true
```
Welcher Remote-Branch gemeint ist, steht in `.git/config` unter `[branch "main"]` (`remote` + `merge`). Mit `@{u}` („upstream“) kann man ihn in jedem Befehl ansprechen, z. B. `git log main..@{u}` für „was ist neu auf dem Server“.

**`.git/` vorher → nachher**: die Summe aus beiden Schritten. Beobachtet bei einem `pull` mit Fast-Forward:
```
NEU      .git/objects/…                     ┐
GEÄNDERT .git/refs/remotes/origin/main      │ fetch
GEÄNDERT .git/logs/refs/remotes/origin/main │
GEÄNDERT .git/FETCH_HEAD                    ┘
GEÄNDERT .git/refs/heads/main               ┐
GEÄNDERT .git/ORIG_HEAD                     │ merge (Fast-Forward)
GEÄNDERT .git/index                         │
GEÄNDERT .git/logs/HEAD                     │
GEÄNDERT .git/logs/refs/heads/main          ┘
```

**Was in den Dateien passiert**
- Der **fetch-Teil** aktualisiert Objekte, `refs/remotes/origin/main` und `FETCH_HEAD` (Details unter [[#git fetch|fetch]]).
- Der **merge-Teil** liest `FETCH_HEAD` bzw. `origin/main` und ändert erst dann **deinen** Branch:
  - beim Fast-Forward nur `refs/heads/main`, `ORIG_HEAD`, `index` und die Reflogs (Grund: `pull: Fast-forward`, wieder mit den Argumenten, mit denen du `pull` aufgerufen hast)
  - bei einem echten Merge zusätzlich ein Merge-Commit in `objects/`
  - bei `--rebase` alles, was unter [[#git rebase|rebase]] steht
- Kommt es zum Konflikt, hält `pull` genau wie `merge` bzw. `rebase` an, mit `MERGE_HEAD` bzw. `rebase-merge/`.

### git push

**Plumbing**: im Prinzip `fetch` rückwärts. Die eigentliche Entscheidung trifft der Server:
```bash
# 1. Lokal: fehlende Objekte bestimmen und als Packfile senden
# 2. Auf dem Server, sinngemäß:
git update-ref refs/heads/main <neu> <alt>   # nur, wenn main dort noch auf <alt> steht
                                             # und <neu> ein Nachfolger von <alt> ist (Fast-Forward)
# 3. Lokal danach:
git update-ref refs/remotes/origin/main <neu>
```

**`.git/` vorher → nachher**
```
Lokal:      GEÄNDERT .git/refs/remotes/origin/<branch>      (NEU beim ersten Push)
            GEÄNDERT .git/logs/refs/remotes/origin/<branch>
            GEÄNDERT .git/config                            (nur mit -u)
Server:     NEU      objects/…                              ← die Objekte, die dort fehlten
            GEÄNDERT refs/heads/<branch>
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` lokal | **Nichts.** Es wird nur gelesen und gesendet |
| `refs/remotes/origin/main` | Überschrieben mit dem gepushten Hash, denn jetzt weißt du sicher, wie es auf dem Server aussieht |
| `logs/refs/remotes/origin/main` | Neue Zeile `… update by push` |
| `config` | Nur mit `-u`/`--set-upstream`: `[branch "main"]` mit `remote = origin` und `merge = refs/heads/main`. Danach reichen `git push` und `git pull` ohne Argumente |
| Server: `objects/` | Alles, was von deinem Commit aus erreichbar ist und dem Server fehlt. Gemergte, lokal schon gelöschte Branches kommen also über den Merge-Commit mit, ihre **Namen** aber nicht |
| Server: `refs/heads/main` | Überschrieben, **wenn** der alte Server-Stand ein Vorfahre deines Commits ist (Fast-Forward). Sonst wird abgelehnt: „rejected (non-fast-forward)“, und keine Ref ändert sich |

**Force-Push**
- **`--force`** lässt die Fast-Forward-Prüfung weg. Auf dem Server wird `refs/heads/main` einfach überschrieben, und fremde Commits, die nur dort lagen, sind von dieser Ref nicht mehr erreichbar.
- **`--force-with-lease`** ist ein *Compare-and-Swap*: Der Server überschreibt nur, wenn sein `refs/heads/main` noch genau den Hash enthält, der lokal in **deinem** `refs/remotes/origin/main` steht. Lokal entspricht das dem dritten Argument von `update-ref`:
  ```
  $ git update-ref refs/heads/main <neu> <falscher-alter-wert>
  fatal: cannot lock ref 'refs/heads/main': is at 0b5a93f… but expected 0000…01
  ```
  Die Lücke: Ein `git fetch` direkt davor schreibt den neuen Server-Stand in `refs/remotes/origin/main`, ohne dass du die neuen Commits angesehen hast. Dann ist der Schutz wirkungslos.

Tags werden nicht automatisch gepusht: `git push origin v1.0` bzw. `--tags`.

---

## Rückgängig machen

### git reset

**Plumbing**: drei Stufen, jede überschreibt einen Bereich mehr:
```bash
# --soft:  nur der Branch
git update-ref -m "reset: moving to X" HEAD X

# --mixed (Standard): + Index
git update-ref HEAD X && git read-tree X

# --hard: + Arbeitsverzeichnis
git update-ref HEAD X && git read-tree -u --reset X
```

**`.git/` vorher → nachher**
```
alle Varianten:  GEÄNDERT .git/refs/heads/<branch>   (falls X ≠ aktueller Commit)
                 GEÄNDERT .git/ORIG_HEAD
                 GEÄNDERT .git/logs/HEAD, .git/logs/refs/heads/<branch>
--mixed/--hard:  GEÄNDERT .git/index                 (falls sich der Tree unterscheidet)
--hard:          + Arbeitsverzeichnis überschrieben
```

**Was in den Dateien passiert**

| Datei | `--soft` | `--mixed` | `--hard` |
|---|---|---|---|
| `refs/heads/<branch>` | Überschrieben mit dem Hash von X | ← ebenso | ← ebenso |
| `ORIG_HEAD` | Der Hash **vor** dem Reset. Dein Rückweg: `git reset ORIG_HEAD` | ← ebenso | ← ebenso |
| `logs/…` | Neue Zeile `reset: moving to X` | ← ebenso | ← ebenso |
| `index` | unverändert, deine Änderungen sind also **staged** | Einträge = Tree von X, die Änderungen sind **unstaged** | Einträge = Tree von X |
| Arbeitsverzeichnis | unverändert | unverändert | **überschrieben** mit dem Stand von X |
| `HEAD` | unverändert (`ref: refs/heads/<branch>`). `reset` bewegt den **Branch**, `switch` bewegt `HEAD` | | |
| `objects/` | Nichts Neues, nichts gelöscht. Die „verlassenen“ Commits bleiben und sind über `logs/` erreichbar | | |

Bei `--hard` sind ungestagte Änderungen **endgültig weg**, für sie gab es nie einen Blob. Gestagte, aber nie committete Änderungen findet `git fsck --lost-found` noch als dangling blob.

**Mit Pfad** (`git reset datei`) wird kein Branch bewegt. Nur der Index-Eintrag für `datei` wird aus `HEAD` geholt, also dasselbe wie `git restore --staged datei`. Geändert wird nur `index`.

### git revert

**Plumbing**: ein Cherry-Pick „rückwärts“. Basis ist der Commit selbst, und angewendet wird sein **Parent**:
```bash
c=<commit>
tree=$(git merge-tree --write-tree --merge-base $c HEAD $c~1)
commit=$(printf 'Revert "%s"\n\nThis reverts commit %s.\n' "$(git log -1 --format=%s $c)" $c \
         | git commit-tree $tree -p HEAD)
git read-tree -u -m HEAD $commit
git update-ref HEAD $commit
```

**`.git/` vorher → nachher**
```
Ohne Konflikt:  NEU      .git/objects/xx/…   (neuer Commit, ggf. Trees/Blobs für den zurückgedrehten Inhalt)
                GEÄNDERT .git/refs/heads/main, .git/index, .git/logs/HEAD, .git/logs/refs/heads/main
Mit Konflikt:   NEU      .git/REVERT_HEAD, .git/MERGE_MSG, .git/AUTO_MERGE   — Branch unverändert —
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` | Ein **zusätzlicher** Commit mit der Message `Revert "<betreff>"` / `This reverts commit <hash>.` und deinem `HEAD` als Parent. Sein Tree entspricht dem Stand ohne die Änderung. Oft gibt es diesen Tree schon, dann entsteht nur der Commit |
| `refs/heads/main` | Weitergesetzt, **vorwärts**. Die Historie wird nicht umgeschrieben, der zurückgenommene Commit bleibt in der Kette |
| `logs/…` | Neue Zeile `revert: Revert "<betreff>"` |
| `REVERT_HEAD` | Nur bei Konflikt: Hash des Commits, der zurückgenommen werden soll |

Deshalb ist `revert` das Mittel der Wahl auf `main` und anderen geteilten Branches, `reset` dagegen nur für Lokales. Merge-Commits haben zwei Parents, deshalb braucht `revert` dort `-m 1`, um zu wissen, welche Seite als „vorher“ gilt.

### git clean

**Plumbing**
```bash
git ls-files --others --exclude-standard -z | xargs -0 rm   # ≈ git clean -f
```

**`.git/` vorher → nachher**
```
(nichts)
```

**Was in den Dateien passiert**: In `.git/` passiert **gar nichts**. Gelesen werden `index` (was ist getrackt?) sowie `.gitignore` und `info/exclude` (was ist ignoriert?). Gelöscht werden Dateien **im Arbeitsverzeichnis**, die nicht im Index stehen:
- `-f`: untracked Dateien
- `-d`: auch untracked Ordner
- `-x`: auch ignorierte Dateien (z. B. `.venv/`, `__pycache__/`!)

**Endgültig:** Für untracked Dateien gab es nie einen Blob, und auch der Reflog hilft hier nicht. Vorher immer `git clean -n` (dry run) ausführen.

---

## Zwischenspeichern und parallel arbeiten

### git stash

**Plumbing**: Ein Stash besteht aus **gewöhnlichen Commits**, die an keinem Branch hängen:
```bash
i_tree=$(git write-tree)                                            # 1. Index-Stand
i=$(echo "index on main: …" | git commit-tree $i_tree -p HEAD)

cp .git/index /tmp/stash-index                                      # 2. Stand des Arbeitsverzeichnisses
GIT_INDEX_FILE=/tmp/stash-index git add -u                          #    (getrackte Dateien) über einen
w_tree=$(GIT_INDEX_FILE=/tmp/stash-index git write-tree)            #    temporären Index
w=$(echo "WIP on main: …" | git commit-tree $w_tree -p HEAD -p $i)

git update-ref --create-reflog -m "WIP on main: …" refs/stash $w   # 3. Ref + Reflog-Eintrag
git reset --hard                                                    # 4. Aufräumen
# Teil 1–3 in zwei Befehlen:  w=$(git stash create) && git stash store -m "…" $w
```

**`.git/` vorher → nachher**
```
git stash:        NEU      .git/objects/xx/…        ← 2 Commits + Trees/Blobs für ungestagte Änderungen
                  NEU      .git/refs/stash          (GEÄNDERT, wenn es schon Stashes gibt)
                  NEU      .git/logs/refs/stash     (GEÄNDERT, …)
                  GEÄNDERT .git/index
                  GEÄNDERT .git/logs/HEAD
git stash list:   (nichts)
git stash pop:    GEÄNDERT .git/index
                  GEÄNDERT .git/logs/refs/stash   — bzw. GELÖSCHT samt .git/refs/stash beim letzten Stash
```

**Was in den Dateien passiert**

| Datei | Was passiert |
|---|---|
| `objects/` | **Zwei Commits**: `index on main: …` (der gestagte Stand, Parent = `HEAD`) und `WIP on main: …` (der Stand der getrackten Dateien im Arbeitsverzeichnis, **zwei Parents**: `HEAD` und der Index-Commit). Für ungestagte Inhalte entstehen hier erstmals Blobs. Mit `-u` kommt ein dritter Commit für untracked Dateien dazu, als dritter Parent |
| `refs/stash` | Der Hash des neuesten WIP-Commits. **Nur einer**, auch wenn es zehn Stashes gibt |
| `logs/refs/stash` | **Das ist die Stash-Liste.** Jeder Stash ist eine Zeile darin. `stash@{0}` ist die letzte Zeile, `stash@{1}` die vorletzte, ganz normale Reflog-Syntax. `git stash list` ist im Grunde `git reflog show refs/stash` |
| `index` | Auf `HEAD` zurückgesetzt (`reset --hard`). Arbeitsverzeichnis ebenso |
| `logs/HEAD` | Zeile `reset: moving to HEAD` vom Aufräumen. `HEAD` selbst bewegt sich nicht |

Im Graphen sieht ein Stash so aus:
```
*   edd936e WIP on main: 0b5a93f c3      ← refs/stash (Arbeitsverzeichnis-Stand)
|\
| * 06e784b index on main: 0b5a93f c3    ← zweiter Parent (Index-Stand)
|/
* 0b5a93f c3                             ← erster Parent (HEAD beim Stashen)
```

**`apply`/`pop`** machen einen Drei-Wege-Merge mit dem damaligen `HEAD` als Basis und schreiben das Ergebnis in Arbeitsverzeichnis und `index`. Den Index-Stand (also was gestaged war) stellen beide nur mit `--index` wieder her, ohne die Option ist danach alles unstaged. `pop` entfernt danach die Zeile aus `logs/refs/stash`. War es der letzte Stash, verschwinden `refs/stash` und `logs/refs/stash` ganz. Die Commits bleiben als unerreichbare Objekte liegen, deshalb findet `git fsck --unreachable | grep commit` einen versehentlich gedroppten Stash wieder.

### git worktree

**Plumbing**: Dafür gibt es keine einfache Entsprechung. Der Trick ist, dass ein zweites Arbeitsverzeichnis **eigene** Zustandsdateien bekommt, aber Objekte und Refs mit dem Haupt-Repo teilt.

**`.git/` vorher → nachher**
```
git worktree add ../hotfix -b hotfix:
  NEU      .git/refs/heads/hotfix, .git/logs/refs/heads/hotfix   (nur mit -b)
  NEU      .git/worktrees/hotfix/HEAD
  NEU      .git/worktrees/hotfix/index
  NEU      .git/worktrees/hotfix/logs/HEAD
  NEU      .git/worktrees/hotfix/ORIG_HEAD
  NEU      .git/worktrees/hotfix/gitdir
  NEU      .git/worktrees/hotfix/commondir
  NEU      ../hotfix/.git            ← eine DATEI, kein Ordner
git worktree remove ../hotfix:
  GELÖSCHT .git/worktrees/hotfix/…   (der Branch hotfix bleibt)
```

**Was in den Dateien passiert**

| Datei | Inhalt |
|---|---|
| `../hotfix/.git` | Eine Zeile: `gitdir: /pfad/zum/repo/.git/worktrees/hotfix`. Jeder Git-Befehl im neuen Ordner folgt diesem Verweis |
| `worktrees/hotfix/HEAD` | Das **eigene** `HEAD` des Worktrees: `ref: refs/heads/hotfix`. Beide Ordner stehen also auf verschiedenen Branches |
| `worktrees/hotfix/index` | Der **eigene** Index. Stagen im Worktree beeinflusst das Haupt-Repo nicht |
| `worktrees/hotfix/logs/HEAD`, `ORIG_HEAD` | Eigener Reflog und eigener Rückweg |
| `worktrees/hotfix/commondir` | `../..`: Verweis zurück auf das Haupt-`.git`, wo `objects/`, `refs/` und `config` **gemeinsam** liegen |
| `worktrees/hotfix/gitdir` | Pfad zur `.git`-Datei im Worktree. Daran erkennt `git worktree prune` verwaiste Einträge, wenn der Ordner von Hand gelöscht wurde |

Ein Commit im Worktree landet in den gemeinsamen `objects/` und bewegt den gemeinsamen `refs/heads/hotfix`. Er ist also sofort im Haupt-Repo sichtbar. Derselbe Branch kann nicht in zwei Worktrees gleichzeitig ausgecheckt sein, sonst würden zwei Indizes denselben Branch weiterschieben.

---

## Releases und Fehlersuche

### git tag

**Plumbing**
```bash
# lightweight: nur eine Ref
git update-ref refs/tags/v1.0 HEAD

# annotated: ein eigenes Tag-Objekt + Ref darauf
printf "object %s\ntype commit\ntag v1.0\ntagger Nick <…> %s +0200\n\nRelease 1.0\n" \
  $(git rev-parse HEAD) $(date +%s) | git mktag     # gibt den Hash des Tag-Objekts aus
git update-ref refs/tags/v1.0 <tag-objekt-hash>
```

**`.git/` vorher → nachher**
```
git tag v1:              NEU .git/refs/tags/v1
git tag -a v2 -m "…":    NEU .git/refs/tags/v2
                         NEU .git/objects/xx/…     ← das Tag-Objekt
git tag -d v1:           GELÖSCHT .git/refs/tags/v1   (bzw. Zeile aus packed-refs)
```

**Was in den Dateien passiert**

| Datei | Lightweight | Annotated |
|---|---|---|
| `refs/tags/<name>` | Direkt der **Commit**-Hash, genau wie ein Branch, nur dass kein Befehl ihn weiterschiebt | Der Hash des **Tag-Objekts** |
| `objects/` | nichts | Ein neues Objekt vom Typ `tag` (der vierte Objekttyp neben Blob, Tree und Commit): `object <commit-hash>`, `type commit`, `tag v2`, `tagger …`, Leerzeile, Message. Optional mit Signatur |
| `logs/` | **Kein Reflog.** Tags werden standardmäßig nicht protokolliert, weil sie sich nicht bewegen sollen | ebenso |

`git cat-file -t v2` gibt `tag` aus, `git cat-file -t v1` gibt `commit` aus. Für Releases nimmt man annotated Tags: Sie speichern, wer wann warum getaggt hat.

### git bisect

**Plumbing**: eine Schleife aus `checkout` und deinem Urteil:
```bash
git rev-list --bisect good..bad       # gibt den Commit in der Mitte des Bereichs aus
git switch --detach <mitte>           # testen, dann good/bad markieren, wiederholen
```

**`.git/` vorher → nachher**
```
git bisect start:        NEU      .git/BISECT_START, .git/BISECT_LOG, .git/BISECT_NAMES
git bisect bad <c>:      NEU      .git/refs/bisect/bad, .git/BISECT_TERMS
                         GEÄNDERT .git/BISECT_LOG
git bisect good <c>:     NEU      .git/refs/bisect/good-<hash>
                         NEU      .git/BISECT_EXPECTED_REV, .git/BISECT_ANCESTORS_OK
                         GEÄNDERT .git/HEAD, .git/index, .git/logs/HEAD, .git/BISECT_LOG
git bisect reset:        GELÖSCHT alle BISECT_*-Dateien und refs/bisect/*
                         GEÄNDERT .git/HEAD, .git/index, .git/logs/HEAD
```

**Was in den Dateien passiert**

| Datei | Inhalt |
|---|---|
| `BISECT_START` | Der Branch, auf dem du angefangen hast (z. B. `main`). Dorthin kehrt `bisect reset` zurück |
| `BISECT_LOG` | Jeder Schritt als ausführbarer Befehl (`git bisect good 5fc00f4…`). `git bisect replay` spielt ihn erneut ab, praktisch nach einer Fehlentscheidung |
| `BISECT_TERMS` | Die Begriffe, standardmäßig `bad` und `good` (umbenennbar, z. B. `old`/`new`) |
| `BISECT_NAMES` | Pfade, falls die Suche auf bestimmte Dateien beschränkt ist |
| `refs/bisect/bad` | Der älteste bekannt schlechte Commit. Wird bei jedem `bad` überschrieben |
| `refs/bisect/good-<hash>` | Eine Ref pro guten Commit. Als **Refs** schützen sie die Commits vor `gc` und sind in `git log --all` sichtbar |
| `BISECT_EXPECTED_REV` | Der Commit, den Git als Nächstes zum Testen ausgecheckt hat |
| `HEAD` | Ab dem ersten Schritt ein **nackter Hash** (detached): der Commit in der Mitte. `index` und Arbeitsverzeichnis werden jedes Mal passend umgeschrieben |

`git bisect run ./test.sh` automatisiert die Schleife: Exit-Code 0 bedeutet good, 1–127 bad (außer 125, das heißt „überspringen“).
