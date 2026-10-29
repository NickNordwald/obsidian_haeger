---
tags:
  - documentation
---

**uv** ist ein Paket- und Umgebungsmanager für Python — geschrieben in Rust von *Astral*, denselben Leuten wie **Ruff**, das im Projekt schon als Linter läuft. Er ersetzt in einem Werkzeug, wofür man bisher vier bis fünf brauchte: `pip`, `venv`, `pip-tools`, `pyenv` und `pipx`.

Diese Notiz ist die Vorbereitung auf den Umbau der Pricing-Anwendung, siehe [[Umsetzungsstand (REST APIs & Layered Architecture)#Werkzeugwechsel auf uv (geplant)|Umsetzungsstand → Werkzeugwechsel auf uv]]. Alle Beispiele stammen aus diesem Projekt.

# Inhalt

- [[#Das Problem, das uv löst]]
- [[#Die vier Bausteine]]
- [[#Manifest vs. Lockfile — der zentrale Begriff]]
- [[#Drei Sorten Abhängigkeiten]]
- [[#Die Befehle im Alltag]]
- [[#uv run — warum es kein activate mehr gibt]]
- [[#Von pip-Wissen zu uv-Wissen]]
- [[#Werkzeuge außerhalb des Projekts (uv tool / uvx)]]
- [[#Was in git gehört]]
- [[#Fallstricke]]
- [[#Spickzettel]]

---

# Das Problem, das uv löst

Nicht abstrakt, sondern die drei Pannen, die dieses Kapitel tatsächlich gekostet hat — nachzulesen unter [[Umsetzungsstand (REST APIs & Layered Architecture)#Gelernte Lektionen|Gelernte Lektionen]]:

| Panne | Ursache | Was uv daran ändert |
|---|---|---|
| Kopiertes `.venv` zeigte auf ein fremdes Kapitel | Der Editable-Install merkt sich einen **absoluten Pfad** | Man kopiert keine venv mehr. Man kopiert den Projektordner *ohne* `.venv` und ruft `uv sync` — die Umgebung entsteht neu und exakt. |
| `pip install -e .` lief im falschen Interpreter | Shebangs in `.venv/bin/` trugen den Pfad des Originals | `uv run` benutzt nie ein Shebang-Skript, sondern immer den Interpreter der Projekt-venv |
| Entry Point zeigte nach dem Umbau ins Leere | Entry Points werden beim **Installieren** geschrieben, nicht beim Import | `uv run pricing` synchronisiert vor jedem Lauf — die Neuinstallation kann man nicht mehr vergessen |

Dazu kommt das Problem, das man erst merkt, wenn es weh tut: **`pip install -e ".[dev,db,orm]"` beschreibt nicht, was danach installiert ist.** Es installiert *irgendeine* Version, die zum Zeitpunkt des Aufrufs zu den Constraints passt. Wer denselben Befehl drei Monate später auf einem anderen Rechner ausführt, bekommt eine andere Umgebung — und wenn dann etwas kaputt ist, ist nicht mehr feststellbar, was sich geändert hat.

> [!important] Der eigentliche Gewinn
> Nicht die Geschwindigkeit (die ist ein netter Nebeneffekt), sondern die **Reproduzierbarkeit**. Die Umgebung ist danach kein Zustand mehr, der auf der Platte gewachsen ist, sondern ein **Artefakt, das aus zwei eingecheckten Dateien exakt wiederherstellbar ist**.

---

# Die vier Bausteine

| Datei | Rolle | Wer schreibt sie | In git? |
|---|---|---|---|
| `pyproject.toml` | **Manifest** — was das Projekt braucht, in Constraints (`>=2.0`) | Du (oder `uv add`) | ✅ ja |
| `uv.lock` | **Lockfile** — welche exakten Versionen daraus aufgelöst wurden, inkl. aller transitiven Abhängigkeiten und Hashes | uv | ✅ ja (bei Anwendungen) |
| `.python-version` | Welche Python-Version dieses Projekt fährt | `uv python pin` | ✅ ja |
| `.venv/` | Das **Ergebnis** — die tatsächliche Umgebung | `uv sync` | ❌ nein, wie bisher |

Das Verhältnis in einem Satz: **`pyproject.toml` ist der Wunsch, `uv.lock` ist das Protokoll, `.venv/` ist die Wirklichkeit.**

---

# Manifest vs. Lockfile — der zentrale Begriff

Der Unterschied ist der ganze Trick, deshalb ausführlich.

Im Manifest steht ein **Constraint**:

```toml
dependencies = ["sqlalchemy>=2.0"]
```

Das heißt: „mindestens 2.0, gerne neuer". Es sagt *nichts* darüber, was installiert ist. Zwei Rechner, derselbe Constraint, ein halbes Jahr Abstand → 2.0.51 hier, 2.1.3 dort.

Im Lockfile steht die **Auflösung**:

```toml
[[package]]
name = "sqlalchemy"
version = "2.0.51"
```

… und zwar für **jedes** Paket, auch für die, die du nie selbst angefordert hast (`greenlet`, `typing-extensions`, `annotated-types` …). Genau die sind der Grund, warum ein Constraint-Satz allein nicht reicht: Deine 4 direkten Abhängigkeiten ziehen ein Dutzend indirekte nach, und die haben ihre eigenen Constraints.

Daraus folgt die Arbeitsteilung im Alltag:

- **Manifest ändern** (`uv add`, oder von Hand) → sagen, *was* man will.
- **`uv lock`** → die Constraints in exakte Versionen auflösen (passiert bei `uv add`/`uv sync` automatisch mit).
- **`uv sync`** → die Umgebung dem Lockfile angleichen.

**`uv sync` ist dabei exakt**: Pakete, die in der venv liegen, aber nicht im Lockfile stehen, werden **entfernt**. Das ist gewollt — es verhindert die klassische „auf meinem Rechner läuft's, weil da noch was Altes rumliegt"-Situation. (Wer das ausnahmsweise nicht will: `--inexact`.)

---

# Drei Sorten Abhängigkeiten

Hier steckt die inhaltliche Entscheidung des Umbaus, nicht nur Mechanik. Es gibt drei Töpfe, und sie bedeuten verschiedene Dinge:

### 1. `dependencies` — was das Programm zum Laufen braucht

```toml
[project]
dependencies = ["pydantic-settings>=2", "sqlalchemy>=2.0", "psycopg[binary]>=3.2"]
```

Wird immer installiert. **Faustregel: Wird das Paket irgendwo auf Modulebene importiert, gehört es hierhin.**

### 2. `[project.optional-dependencies]` — echte Extras

```toml
[project.optional-dependencies]
api = ["fastapi>=0.115", "uvicorn[standard]>=0.30"]
```

Ein Extra ist eine **Fähigkeit, die man abwählen kann**: `pip install pricing[api]` bringt die API mit, `pip install pricing` nicht — und beides funktioniert. Extras sind Teil der veröffentlichten Paket-Metadaten und damit für die *Nutzer* des Pakets gedacht.

> [!warning] Die Falle, die im Projekt genau so drinsteckt
> Ein Extra ist nur dann ehrlich, wenn das Programm **ohne** es noch läuft. Bei uns sind `sqlalchemy` (Extra `orm`) und `psycopg` (Extra `db`) als optional deklariert, werden aber in `sources.py` und `writers.py` auf Modulebene importiert — und `main.py` importiert `sources`. Ohne die Extras stirbt also schon der CSV-Lauf mit `ImportError`. Sie sind faktisch Kern-Abhängigkeiten und gehören nach `dependencies`.
>
> Ein Extra, das nicht optional ist, ist keine Konfiguration — es ist eine falsche Angabe in der Paketbeschreibung.

### 3. `[dependency-groups]` — Werkzeuge für die Entwicklung

```toml
[dependency-groups]
dev = ["pytest>=8.0", "pytest-cov>=5.0", "ruff>=0.6", "mypy>=1.11"]
```

Der neuere, saubere Weg (PEP 735) für alles, was nur *beim Entwickeln* gebraucht wird. Der entscheidende Unterschied zu Extras:

| | Extra | Dependency-Group |
|---|---|---|
| Landet im veröffentlichten Paket | ✅ ja | ❌ nein |
| Für wen gedacht | Nutzer des Pakets | Entwickler am Repository |
| Wird von `uv sync` automatisch installiert | nur mit `--extra`/`--all-extras` | die Gruppe `dev` **ja**, andere mit `--group` |

`pytest` ist keine Fähigkeit unserer Anwendung — niemand installiert `pricing[dev]`, um sie zu benutzen. Deshalb: Dev-Werkzeuge in die Gruppe, nicht in die Extras.

---

# Die Befehle im Alltag

| Befehl | Was er tut |
|---|---|
| `uv sync` | Umgebung an den Lockfile angleichen. Legt `.venv` an, falls sie fehlt; installiert das Projekt selbst editable; installiert die `dev`-Gruppe mit |
| `uv sync --all-extras` | zusätzlich alle Extras |
| `uv sync --no-dev` | ohne Dev-Werkzeuge (so würde ein Server es machen) |
| `uv run <befehl>` | Befehl in der Projektumgebung ausführen — **synchronisiert vorher automatisch** |
| `uv add <paket>` | Abhängigkeit ins Manifest schreiben, Lockfile aktualisieren, installieren |
| `uv add --group dev <paket>` | dasselbe für die Dev-Gruppe |
| `uv add --optional api <paket>` | dasselbe für ein Extra |
| `uv remove <paket>` | Gegenstück zu `add` |
| `uv lock` | nur neu auflösen, nichts installieren |
| `uv lock --upgrade` | Versionen anheben, soweit die Constraints es erlauben |
| `uv tree` | Abhängigkeitsbaum anzeigen — beantwortet „wer zieht eigentlich `greenlet` rein?" |
| `uv python pin 3.13` | `.python-version` schreiben |
| `uv python list` | verfügbare und installierte Python-Versionen |

Für unser Projekt heißt das konkret:

```bash
uv run pytest
uv run ruff check
uv run mypy
uv run pricing -v
uv run pricing-ingest -v
uv run python scripts/smoke_orm.py
```

---

# uv run — warum es kein activate mehr gibt

Bisher: `source .venv/bin/activate`, dann `pytest`. Das Aktivieren setzt `PATH` und `VIRTUAL_ENV` in der **Shell** um — ein Zustand, den man vergessen kann (und der die Quelle des „falscher Interpreter"-Fehlers war).

`uv run pytest` braucht das nicht: uv findet die Projekt-venv anhand der `pyproject.toml` im aktuellen oder einem übergeordneten Verzeichnis, sorgt dafür, dass sie zum Lockfile passt, und startet den Prozess direkt mit dem richtigen Interpreter. **Kein Shell-Zustand, kein Vergessen, keine Kreuzverdrahtung.**

Aktivieren *geht* weiterhin und ist nicht falsch — es ist nur nicht mehr nötig.

Zwei Schalter für Sonderfälle:

- `uv run --frozen …` — nicht neu auflösen, Lockfile so nehmen, wie er ist.
- `uv run --locked …` — **fehlschlagen**, falls der Lockfile nicht zum Manifest passt. Das ist der Schalter für CI: Er macht aus „die Umgebung ist reproduzierbar" eine geprüfte Aussage statt einer Hoffnung.

---

# Von pip-Wissen zu uv-Wissen

| Bisher | Mit uv |
|---|---|
| `python -m venv .venv` | entfällt — `uv sync` macht es |
| `source .venv/bin/activate` | entfällt — `uv run …` |
| `pip install -e .` | entfällt — das Projekt wird von `uv sync` editable installiert |
| `pip install -e ".[dev]"` | `uv sync` (Dev-Gruppe ist Standard) |
| `pip install fastapi` | `uv add fastapi` |
| `pip uninstall fastapi` | `uv remove fastapi` |
| `pip list` | `uv tree` (aussagekräftiger) oder `uv pip list` |
| `pip freeze > requirements.txt` | `uv.lock` — und wenn wirklich `requirements.txt` gebraucht wird: `uv export` |
| `pyenv install 3.13` | `uv python install 3.13` |
| `pipx install ruff` | `uv tool install ruff` |

`uv pip …` gibt es als pip-kompatible Notausgang-Schnittstelle. Sie ist nützlich zum Nachschauen, umgeht aber den Lockfile — im Alltag also die Projektbefehle nehmen.

---

# Werkzeuge außerhalb des Projekts (uv tool / uvx)

```bash
uvx ruff check          # einmalig ausführen, nichts dauerhaft installieren
uv tool install ruff    # global verfügbar machen (wie pipx)
```

Praktisch für Werkzeuge, die man überall braucht. **Für unser Projekt trotzdem lieber in der Dev-Gruppe lassen**: Dann ist die Ruff-Version im Lockfile festgehalten, und ein Ruff-Update kann die Prüfung nicht überraschend rot färben, ohne dass eine Datei im Repository sich geändert hätte.

---

# Was in git gehört

```
✅ pyproject.toml      Manifest
✅ uv.lock             Lockfile — bei Anwendungen einchecken!
✅ .python-version     Python-Version des Projekts
❌ .venv/              wie bisher ignoriert
```

Die Regel für den Lockfile lautet: **Anwendung → einchecken. Bibliothek → nicht einchecken.**

Warum? Eine Anwendung wird *deployt*; man will exakt wissen, was läuft. Eine Bibliothek wird von anderen *installiert*; deren Auflösung soll sich an ihrem Projekt orientieren, nicht an deiner. Die Pricing-Anwendung ist eine Anwendung → `uv.lock` gehört ins Repository.

---

# Fallstricke

- **`uv sync` räumt auf.** Ein Paket, das du mal von Hand installiert hast, verschwindet beim nächsten Sync. Richtig so — aber gut zu wissen, bevor man sich wundert.
- **Der Lockfile ist eine Datei mit Merge-Konflikt-Potenzial.** Bei Konflikten nicht von Hand zusammenflicken: Konfliktmarker wegwerfen, `uv lock` neu laufen lassen.
- **`requires-python` ist bindend.** Steht dort `>=3.11`, löst uv die Versionen so auf, dass sie unter 3.11 *und* neuer funktionieren — auch wenn du auf 3.13 entwickelst. Wer das nicht will, hebt `requires-python` an.
- **`.python-version` ≠ `requires-python`.** Ersteres ist „womit entwickle ich", Letzteres „was unterstütze ich". Sie dürfen auseinanderlaufen, sollten es aber bewusst tun.
- **uv ersetzt kein Build-Backend.** `hatchling` bleibt, wo es ist; uv ruft es auf.

---

# Spickzettel

```bash
# Einmalig beim Umbau
uv python pin 3.13            # .python-version schreiben
uv sync                       # venv anlegen, alles installieren, uv.lock erzeugen

# Alltag
uv run pytest                 # testen
uv run ruff check             # linten
uv run mypy                   # typprüfen
uv run pricing -v             # Anwendung starten

# Abhängigkeiten
uv add fastapi                # Kern-Abhängigkeit
uv add --group dev pytest-cov # Dev-Werkzeug
uv add --optional api uvicorn # Extra
uv remove psycopg             # entfernen
uv tree                       # wer zieht was?

# Reproduzieren (anderer Rechner, CI)
uv sync --locked              # exakt aus dem Lockfile, Abweichung = Fehler
```

---

# Bezug zum eigenen Projekt

Der geplante Umbau steht als abhakbare Liste im [[Umsetzungsstand (REST APIs & Layered Architecture)#Werkzeugwechsel auf uv (geplant)|Umsetzungsstand]]. Die drei inhaltlichen Entscheidungen darin, hier noch einmal auf den Punkt:

1. **Python 3.13 pinnen** und `requires-python`, `ruff target-version` und `mypy python_version` darauf nachziehen — heute sagen sie 3.11, gelaufen wird 3.13. Geprüft wird also gegen eine Version, die niemand benutzt.
2. **`sqlalchemy` und `psycopg` von Extras zu `dependencies`** machen — sie werden hart importiert, sind also keine Extras (siehe Kasten oben).
3. **Dev-Werkzeuge in `[dependency-groups]`** statt in die Extras — sie sind keine Fähigkeit der Anwendung.

Danach kommt FastAPI als *echtes* Extra dazu (`uv add --optional api …`) — und dann stimmt die Aussage „Extra" auch inhaltlich: Der Batch-Lauf über die CLI funktioniert weiterhin ohne FastAPI, genau wie es die [[Umsetzungsstand (REST APIs & Layered Architecture)#Architekturentscheidung: vierschichtige Struktur (2026-08-04)|Architekturentscheidung]] verlangt.
