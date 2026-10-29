---
tags:
  - documentation
---

# Git Leseliste

Ein Lernpfad für Git im Team. Die Basics sitzen; mir fehlt, was erst mit mehreren Leuten wichtig wird: Branches, die auseinanderlaufen, Konflikte, eine aufgeräumte Historie und gemeinsame Konventionen. Deshalb ist der Pfad in **Häppchen von etwa 30 Minuten** geschnitten. Jedes passt in eine Lücke im Arbeitstag.

**Ablauf pro Block:** Core lesen, dann den Teil *Ausprobieren* in einem Wegwerf-Repo unter `/tmp` durchspielen. Danach folgt ein **Quiz aus dem Kopf** (`Quiz <Nr> - <Thema>.md` in diesem Ordner), das Claudian reviewt. Aus den Lücken entstehen auf Wunsch Anki-Karten. Nachschlagen, was ein Befehl intern tut: [[Git Befehlsreferenz]]. Ein kompletter Beispiel-Durchlauf mit allen Änderungen in `.git/`: [[Git von innen - ein Durchlauf]]. Fürs Ausprobieren am besten immer zuerst ein Sandbox-Repo anlegen:

```bash
git init /tmp/sandbox && cd /tmp/sandbox && git commit --allow-empty -m "Initial commit"
```

> [!info] Stand und Quellen
> Alle URLs und Abschnittstitel wurden am 21.10.2026 geprüft. Lokal installiert ist Git **2.53**, aktuell ist **2.56**. **Git 3.0** soll Ende 2026 erscheinen, ein Datum steht aber noch nicht fest: Neue Repos bekommen dann SHA-256 statt SHA-1 und `main` als Standard-Branch. Bestehende Repos ändern sich nicht. Die Hauptquelle ist das frei verfügbare Buch **[Pro Git](https://git-scm.com/book/en/v2)** (2. Auflage, im Folgenden „Pro Git").

## Die Blöcke

| Block | Thema | Core | Quiz | Status |
|---|---|---|---|---|
| **[[#Block 1 — Commits, Objektmodell & Rebase\|1]]** | Wie Git speichert, gute Commit-Messages, interaktiver Rebase | ~70 Min. | [[Quiz 1 - Commits, Objekte & Rebase\|Quiz 1]] | 🔄 läuft |
| **[[#Block 2 — Branches & Merge vs. Rebase\|2]]** | Branches als Zeiger, Merge oder Rebase, die Goldene Regel | ~30 Min. | — | ⬜ offen |
| **[[#Block 3 — Mit Remotes zusammenarbeiten\|3]]** | `fetch` und `pull`, Tracking-Branches, `--force-with-lease` | ~30 Min. | — | ⬜ offen |
| **[[#Block 4 — Merge-Konflikte lösen\|4]]** | Konfliktmarker lesen, abbrechen, `diff3`, `rerere` | ~30 Min. | — | ⬜ offen |
| **[[#Block 5 — Fehler rückgängig machen\|5]]** | `reset`, `restore`, `revert`, `reflog` | ~35 Min. | — | ⬜ offen |
| **[[#Block 6 — Team-Workflows & Pull Requests\|6]]** | GitHub Flow, Trunk-Based, Git Flow, kleine Changes | ~30 Min. | — | ⬜ offen |
| **[[#Block 7 — Gute Commits bauen\|7]]** | `add -p`, atomare Commits, Commit-Richtlinien | ~25 Min. | — | ⬜ offen |
| **[[#Block 8 — Unterbrechungen: stash & worktree\|8]]** | Schnell etwas anderes erledigen, ohne WIP-Commit | ~20 Min. | — | ⬜ offen |
| **[[#Block 9 — Historie durchsuchen & Fehler finden\|9]]** | `log -S`, `blame`, `bisect` | ~25 Min. | — | ⬜ offen |
| **[[#Block 10 — Automatisieren: .gitignore, Hooks, pre-commit\|10]]** | Prüfungen vor jedem Commit, z. B. ruff | ~30 Min. | — | ⬜ offen |
| **[[#Block 11 — Tags, Releases & Changelogs\|11]]** | Annotated Tags, SemVer, Changelog statt Git-Log | ~20 Min. | — | ⬜ offen |

Die Reihenfolge ist eine Empfehlung: **Die Blöcke 2–6 sind der Team-Kern** und bauen aufeinander auf. Die Blöcke 7–11 sind unabhängig voneinander und können in beliebiger Reihenfolge drankommen, je nachdem, was im Arbeitsalltag gerade auftaucht.

> [!question] Offen: Welche Plattform nutzt Haeger?
> GitLab, GitHub, Bitbucket oder Azure DevOps? Für Block 6 relevant (Merge Request oder Pull Request, Merge-Optionen in der Oberfläche). Sobald das klar ist, bekommt die Plattform eine Notiz unter `Documentation/Services/`.

---

## Block 1 — Commits, Objektmodell & Rebase

**Core ~70 Min. · [[Quiz 1 - Commits, Objekte & Rebase|Quiz 1]]**

**Warum das zählt:** Wer weiß, dass ein Commit nur ein kleines Objekt mit Zeiger auf einen Tree und seine Eltern ist, versteht auch, warum Rebase „neue" Commits erzeugt und warum fast nichts wirklich verloren geht. Die beiden Commit-Message-Artikel sind die erste Team-Konvention: Im Solo-Projekt liest niemand deine Messages, im Team ist das Log das Gedächtnis des Projekts.

### Core (in dieser Reihenfolge)

- [ ] **1. Pro Git, Kap. 10.2 „Git Objects"** (~20 Min.) — <https://git-scm.com/book/en/v2/Git-Internals-Git-Objects>
      Blobs, Tree Objects, Commit Objects, Object Storage. Die Befehle am besten im Sandbox-Repo mittippen.
      *Leseauftrag: Was steht in einem Commit-Objekt, und was steht **nicht** darin? Warum ändert sich der Hash eines Commits, wenn man nur seinen Vorgänger ändert?*
- [ ] **2. Drew DeVault, „git rebase in depth"** (~25 Min.) — <https://git-rebase.io/>
      Komplett, inkl. Fußnoten. Jeder Abschnitt läuft in einer frischen Sandbox, also auch hier mittippen.
      *Leseauftrag: Was ist der Unterschied zwischen `squash` und `fixup`? Und warum verkehren sich `--ours` und `--theirs` beim Rebase?*
- [ ] **3. Chris Beams, „How to Write a Git Commit Message"** (2014, ~15 Min.) — <https://cbea.ms/git-commit/>
      Ließ sich am 21.10. von hier aus nicht laden. Falls das bei dir auch so ist: [Archivkopie](https://web.archive.org/web/2026/https://cbea.ms/git-commit/).
      *Leseauftrag: Welche drei Dinge sollte ein Team laut Beams für seine Commit-Messages festlegen? Wie prüft man mit einem Satz, ob ein Betreff im Imperativ steht?*
- [ ] **4. Scoped Commits** (~8 Min.) — <https://scopedcommits.com/>
      *Leseauftrag: Warum gehört der Scope nach vorn? Und warum soll man aus dem Commit-Log **kein** Changelog erzeugen?*

> [!tip] Beim Lesen auf den Widerspruch achten
> Beams schreibt den Betreff groß, Scoped Commits schreibt nach dem Scope meist klein weiter (`net/http: add godoc links`). Was davon gilt, legt in der Praxis jedes Projekt selbst fest. Das kommt im Quiz.

### Vertiefung (optional)

- [ ] **Pro Git, Kap. 10.3 „Git References"** (~10 Min.) · *weiterführend* — <https://git-scm.com/book/en/v2/Git-Internals-Git-References>
      Was Branches, `HEAD` und Tags auf Dateiebene sind. Die perfekte Brücke zu Block 2.
- [ ] **Conventional Commits 1.0.0** (~10 Min.) · *andere Perspektive* — <https://www.conventionalcommits.org/en/v1.0.0/>
      Der verbreitete Gegenentwurf zu Scoped Commits: `feat(parser): …`, `fix: …`. Viele Teams nutzen ihn, also solltest du ihn erkennen.
- [ ] **Sumner Evans, „Stop Using Conventional Commits"** (~10 Min.) · *andere Perspektive* — <https://sumnerevans.com/posts/software-engineering/stop-using-conventional-commits/>
      Die Kritik, auf die sich Scoped Commits beruft. Am besten direkt nach Conventional Commits lesen.

*Nicht im Core:* Packfiles und Garbage Collection (Pro Git 10.4) · Git-Trailer wie `Co-authored-by:` · signierte Commits.

---

## Block 2 — Branches & Merge vs. Rebase

**Core ~30 Min.**

**Warum das zählt:** Solo ist die Historie linear, im Team laufen Branches auseinander und müssen wieder zusammen. Ob das per Merge oder Rebase passiert, ist die erste Frage, über die sich Teams streiten. Dazu gehört die eine Regel, die man nie brechen darf.

### Core

- [ ] **1. Pro Git, Kap. 3.1 „Branches in a Nutshell"** (~10 Min., überfliegen, was du kennst) — <https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>
      *Leseauftrag: Woher weiß Git, auf welchem Branch du gerade bist?*
- [ ] **2. Pro Git, Kap. 3.6 „Rebasing"** — nur „The Perils of Rebasing", „Rebase When You Rebase" und „Rebase vs. Merge" (~15 Min.) — <https://git-scm.com/book/en/v2/Git-Branching-Rebasing>
      *Leseauftrag: Was passiert genau, wenn jemand bereits gepushte Commits rebased und force-pusht? Welche zwei Sichtweisen auf „Historie" stellt das Buch gegenüber?*
- [ ] **3. Atlassian, „Merging vs. Rebasing"** — bis einschließlich „The Golden Rule of Rebasing" (~8 Min.) — <https://www.atlassian.com/git/tutorials/merging-vs-rebasing>

**Ausprobieren:** Zwei Branches mit je zwei Commits anlegen. Einmal per `merge` (mit und ohne `--no-ff`), einmal per `rebase` zusammenführen und das Ergebnis jeweils mit `git log --oneline --graph --all` vergleichen.

### Vertiefung (optional)

- [ ] **Learn Git Branching** (interaktiv, ~30 Min.) · *andere Perspektive* — <https://learngitbranching.js.org/>
      Visualisiert jeden Befehl als Graph. Die Level „Main" 1–4 und „Remote" passen zu Block 2 und 3.

---

## Block 3 — Mit Remotes zusammenarbeiten

**Core ~30 Min.**

**Warum das zählt:** `origin/main` ist nicht `main`. Wer das Remote-Tracking-Modell verstanden hat, wird von `pull`, abgelehnten Pushes und „diverged branches" nicht mehr überrascht.

### Core

- [ ] **1. Pro Git, Kap. 3.5 „Remote Branches"** (~15 Min.) — <https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches>
      *Leseauftrag: Was ist der Unterschied zwischen `git fetch` und `git pull`? Was ist ein Tracking-Branch, und wozu dient `@{u}`?*
- [ ] **2. Git-Doku `git push`, Option `--force-with-lease`** (~7 Min.) — <https://git-scm.com/docs/git-push>
      Auf der Seite nach `--force-with-lease` suchen und den Abschnitt lesen.
      *Leseauftrag: Wovor schützt `--force-with-lease`, wovor `--force` nicht? Und warum ist er trotzdem nicht narrensicher, wenn vorher ein `git fetch` lief?*
- [ ] **3. Wiederholung: git-rebase.io, Abschnitt „git pull --rebase"** (~3 Min.) — <https://git-rebase.io/#pull>

**Ausprobieren:** Zwei Entwickler auf einem Rechner simulieren:

```bash
git init --bare /tmp/remote.git
git clone /tmp/remote.git /tmp/alice && git clone /tmp/remote.git /tmp/bob
```

Alice und Bob committen beide auf `main`, Alice pusht zuerst. Was sagt Git bei Bobs Push? Das einmal mit `pull` (Merge) und einmal mit `pull --rebase` lösen.

---

## Block 4 — Merge-Konflikte lösen

**Core ~30 Min.**

**Warum das zählt:** Konflikte sind im Team Alltag und kein Unfall. Wer die Marker lesen kann und weiß, wie man sicher abbricht, verliert die Angst davor.

### Core

- [ ] **1. Pro Git, Kap. 3.2 — nur Abschnitt „Basic Merge Conflicts"** (~7 Min.) — <https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging>
- [ ] **2. Pro Git, Kap. 7.8 „Advanced Merging" — nur Abschnitt „Merge Conflicts"** (~20 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging>
      Abbrechen, Whitespace ignorieren, `--conflict=diff3`, `git log --merge`. Den Teil zum manuellen Re-Merging einzelner Dateien darfst du überfliegen.
      *Leseauftrag: Was zeigt der `diff3`-Stil zusätzlich zu „ours" und „theirs", und warum hilft genau das beim Entscheiden?*

**Ausprobieren:** Einen Konflikt bewusst herbeiführen (dieselbe Zeile in zwei Branches ändern), einmal mit `git merge --abort` abbrechen und einmal auflösen. Danach `git config --global merge.conflictStyle zdiff3` setzen und den Konflikt erneut erzeugen. Was ändert sich?

### Vertiefung (optional)

- [ ] **Pro Git, Kap. 7.9 „Rerere"** (~10 Min.) · *weiterführend* — <https://git-scm.com/book/en/v2/Git-Tools-Rerere>
      Git merkt sich, wie du einen Konflikt gelöst hast. Praktisch, wenn ein langer Branch immer wieder rebased wird.

---

## Block 5 — Fehler rückgängig machen

**Core ~35 Min.**

**Warum das zählt:** Gerade bei fortgeschrittenen Befehlen geht mal etwas schief. Mit `reflog` als Sicherheitsnetz traut man sich mehr. Und im Team ist es entscheidend, ob man eine Änderung lokal **umschreibt** (`reset`) oder öffentlich **zurücknimmt** (`revert`).

### Core

- [ ] **1. Pro Git, Kap. 2.4 „Undoing Things"** (~10 Min.) — <https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things>
      Vor allem „Undoing things with git restore", die moderne Variante.
- [ ] **2. Pro Git, Kap. 7.7 „Reset Demystified"** — bis einschließlich „The Role of Reset" (~15 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified>
      *Leseauftrag: Welche der drei „Trees" (HEAD, Index, Working Directory) verändern `--soft`, `--mixed` und `--hard` jeweils?*
- [ ] **3. Pro Git, Kap. 7.1 — nur Abschnitt „RefLog Shortnames"** (~5 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection>
- [ ] **4. „Oh Shit, Git!?!"** (~5 Min.) — <https://ohshitgit.com/>
      Kochrezepte für die typischen Pannen, gut zum Wiederfinden.

**Ausprobieren:** Mit `git reset --hard HEAD~2` zwei Commits „verlieren" und mit `git reflog` zurückholen. Dann einen Commit per `git revert` zurücknehmen und vergleichen: Was steht danach im Log?

---

## Block 6 — Team-Workflows & Pull Requests

**Core ~30 Min.**

**Warum das zählt:** Wie lange leben Branches? Wer merged, und wie? Darauf hat jedes Team eine Antwort, oft unausgesprochen. Wenn du die gängigen Modelle kennst, erkennst du schnell, wie ein Projekt tickt.

### Core

- [ ] **1. GitHub Docs, „GitHub flow"** (~5 Min.) — <https://docs.github.com/en/get-started/using-github/github-flow>
      Das einfachste Modell: Branch, Pull Request, Review, Merge, Deploy.
- [ ] **2. trunkbaseddevelopment.com — Startseite** (~10 Min.) — <https://trunkbaseddevelopment.com/>
      *Leseauftrag: Wie kurz sollen Branches hier leben, und was macht man mit halbfertigen Features?*
- [ ] **3. Vincent Driessen, „A successful Git branching model" — nur die „Note of reflection" oben und das Diagramm** (~5 Min.) — <https://nvie.com/posts/a-successful-git-branching-model/>
      Das ist Git Flow. Der Autor selbst rät heute bei Web-Apps davon ab. Trotzdem begegnet es einem noch oft.
- [ ] **4. Google Engineering Practices, „Small CLs"** (~10 Min.) — <https://google.github.io/eng-practices/review/developer/small-cls.html>
      *Leseauftrag: Warum werden kleine Changes besser reviewt, und wie schneidet man eine große Änderung?*

### Vertiefung (optional)

- [ ] **Pro Git, Kap. 5.2 „Contributing to a Project" — „Private Small Team"** (~10 Min.) · *andere Perspektive* — <https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project>
      Ein Team-Workflow Befehl für Befehl durchgespielt.
- [ ] **Martin Fowler, „Patterns for Managing Source Code Branches"** (lang, ~60 Min., gern in Etappen) · *weiterführend* — <https://martinfowler.com/articles/branching-patterns.html>
      Die gründlichste Einordnung aller Branching-Strategien.

---

## Block 7 — Gute Commits bauen

**Core ~25 Min.**

**Warum das zählt:** Eine gute Message hilft wenig, wenn der Commit drei Dinge auf einmal tut. Mit `add -p` wird aus einem unaufgeräumten Arbeitsverzeichnis eine Folge sauberer, einzeln reviewbarer Commits.

### Core

- [ ] **1. Pro Git, Kap. 7.2 „Interactive Staging" — Abschnitt „Staging Patches"** (~5 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging>
- [ ] **2. Pro Git, Kap. 5.2 — nur Abschnitt „Commit Guidelines"** (~10 Min.) — <https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project>
      *Leseauftrag: Was prüft `git diff --check`, und warum soll jeder Commit eine einzige logische Änderung sein?*
- [ ] **3. Wiederholung: git-rebase.io, „Splitting one commit into several" und „Using git rebase --autosquash"** (~8 Min.) — <https://git-rebase.io/>

**Ausprobieren:** In einer Datei zwei unabhängige Änderungen machen und sie mit `git add -p` in zwei Commits aufteilen. Danach einen Tippfehler im älteren Commit per `git commit --fixup` und `git rebase -i --autosquash` korrigieren.

---

## Block 8 — Unterbrechungen: stash & worktree

**Core ~20 Min.**

### Core

- [ ] **1. Pro Git, Kap. 7.3 „Stashing and Cleaning"** — bis einschließlich „Creating a Branch from a Stash" (~12 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning>
- [ ] **2. Git-Doku `git worktree` — „DESCRIPTION" und „EXAMPLES"** (~8 Min.) — <https://git-scm.com/docs/git-worktree>
      *Leseauftrag: Wann ist ein zweiter Worktree besser als ein Stash?*

**Ausprobieren:** Mitten in einer Änderung einen „dringenden Hotfix" auf `main` machen: einmal mit `stash`, einmal mit `git worktree add ../hotfix main`.

---

## Block 9 — Historie durchsuchen & Fehler finden

**Core ~25 Min.**

### Core

- [ ] **1. Pro Git, Kap. 2.3 — Abschnitt „Limiting Log Output"** (~8 Min.) — <https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History>
      *Leseauftrag: Wie findest du den Commit, der eine bestimmte Funktion eingeführt oder gelöscht hat (`-S`)?*
- [ ] **2. Pro Git, Kap. 7.10 „Debugging with Git"** (~12 Min.) — <https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git>
      `blame` und `bisect`, inkl. `git bisect run` mit einem Testskript.

**Ausprobieren:** Ein Repo mit 20 Commits erzeugen (Schleife wie bei git-rebase.io), in einem davon einen „Bug" verstecken und ihn mit `git bisect run` automatisch finden lassen.

---

## Block 10 — Automatisieren: .gitignore, Hooks, pre-commit

**Core ~30 Min.**

**Warum das zählt:** Was eine Maschine prüfen kann, sollte kein Reviewer prüfen müssen. In Python-Projekten ist `pre-commit` mit ruff der Standard, um Formatierung und Linting vor jedem Commit laufen zu lassen.

### Core

- [ ] **1. Git-Doku `gitignore` — „PATTERN FORMAT"** (~5 Min.) — <https://git-scm.com/docs/gitignore>
- [ ] **2. Pro Git, Kap. 8.3 „Git Hooks"** — „Installing a Hook" und „Client-Side Hooks" (~10 Min.) — <https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks>
      *Leseauftrag: Warum werden Hooks nicht mit dem Repo geklont, und welches Problem löst das pre-commit-Framework damit?*
- [ ] **3. pre-commit.com — Introduction und Quick Start** (~12 Min.) — <https://pre-commit.com/#intro>

**Ausprobieren:** In einem eigenen Traineeship-Projekt unter `~/HaegerConsulting/Projects/Traineeship/` eine `.pre-commit-config.yaml` mit ruff einrichten.

---

## Block 11 — Tags, Releases & Changelogs

**Core ~20 Min.**

### Core

- [ ] **1. Pro Git, Kap. 2.6 „Tagging"** (~10 Min.) — <https://git-scm.com/book/en/v2/Git-Basics-Tagging>
      *Leseauftrag: Was ist der Unterschied zwischen einem Lightweight und einem Annotated Tag auf Objektebene? Rückbezug auf Block 1.*
- [ ] **2. Semantic Versioning 2.0.0 — „Summary" und „Introduction"** (~5 Min.) — <https://semver.org/>
- [ ] **3. Keep a Changelog** (~5 Min.) — <https://keepachangelog.com/en/1.1.0/>
      Schließt den Kreis zu Scoped Commits: Changelog und Commit-Log haben verschiedene Leser.

---

## Ideen für später

Wenn die Blöcke durch sind und noch Lust da ist: `git cherry-pick` · `git submodule` und Monorepos · Git LFS für große Dateien (relevant bei Datensätzen) · signierte Commits (Bezug zu [[PGP]]) · Julia Evans, [„Confusing git terminology"](https://jvns.ca/blog/2023/11/01/confusing-git-terminology/) als Wiederholung zwischendurch.
