---
tags:
  - documentation
---
Was das `os`-Modul ist, wie es sich von [[Documentation/Traineeship/Python Basics/Das sys-Modul & Exit-Codes.md|sys]] abgrenzt, und die konkreten Bausteine, die in der Pricing-Anwendung schon benutzt werden: `os.environ["DB_PRICING_PW"]` in `db.py`, `os.getenv("LOG_FILE")` in `logging_setup.py` (Block 1 der [[Logging & Exceptions Plan & Syntax-Tutorial|Logging & Exceptions Plan & Syntax-Tutorial]]) und `load_dotenv()` aus `python-dotenv`, das genau auf `os.environ` aufsetzt.

> [!info] Weiterführend
> Die Stufen **über** `os.environ` — `.env`-Dateien, `pydantic-settings` und Kommandozeilen-Flags mit `argparse`, samt der Frage, welcher Wert in welchen Kanal gehört: [[Konfiguration - Umgebungsvariablen, .env und CLI-Flags|Konfiguration – Umgebungsvariablen, .env und CLI-Flags]].

# Was ist `os` überhaupt?

`os` ist die Schnittstelle zum **Betriebssystem** — Dateisystem, Prozesse, Umgebungsvariablen. Die Abgrenzung zu `sys` (Interpreter-intern) hattet ihr schon in der [[Documentation/Traineeship/Python Basics/Das sys-Modul & Exit-Codes.md#Was ist sys überhaupt?|sys-Notiz]]:

| | `sys` | `os` |
|---|---|---|
| **Wofür** | Schnittstelle zum **Python-Interpreter** | Schnittstelle zum **Betriebssystem** |
| **Beispiele** | Exit-Code, `sys.argv`, `sys.stderr`, Modul-Cache | Umgebungsvariablen, Dateisystem, Prozesse |
| **Typische Frage** | „Wie endet/startet *dieser Python-Prozess*?" | „Was gibt die Maschine drumherum her?" |

`os` ist ein großes Modul (Dateisystem, Prozess-Management, Berechtigungen, …). Relevant für euer Projekt ist bisher fast ausschließlich **ein** Teilbereich: Umgebungsvariablen. Der Rest hier ist knapp gehalten, damit du die Landkarte kennst.

# Umgebungsvariablen: der Teil, den ihr wirklich benutzt

## `os.environ` — ein Dictionary auf die Prozess-Umgebung

`os.environ` verhält sich wie ein `dict[str, str]`, ist aber kein normales Dict — es ist ein **Fenster auf die tatsächliche Umgebung des laufenden Prozesses** (unter Linux letztlich das, was `env` in der Shell zeigt). Zugriff funktioniert wie bei jedem Dict:

```python
os.environ["DB_PRICING_PW"]          # KeyError, wenn nicht gesetzt
```

Das ist exakt die Zeile aus `db.py`. Bewusst gewählt: **fehlt das Passwort, soll es laut knallen** — kein stiller Fallback auf ein leeres Passwort. Genau dieser `KeyError` ist der, den ihr in Block 4 des Tutorials in einen sprechenden `ConfigError` übersetzt (`raise ConfigError(...) from e`).

## `os.getenv(key, default=None)` — dieselbe Sache, ohne zu knallen

```python
os.getenv("LOG_FILE")              # None, wenn nicht gesetzt — KEIN KeyError
os.getenv("LOG_LEVEL", "INFO")     # "INFO", wenn nicht gesetzt
```

Das ist eure Zeile aus `logging_setup.py`. **Bewusst die andere Variante** als in `db.py` — und das ist keine Stilfrage, sondern eine inhaltliche Entscheidung:

| Variable | Zugriff | Warum |
|---|---|---|
| `DB_PRICING_PW` | `os.environ[...]` (wirft) | **Pflicht.** Ohne Passwort kann die Anwendung nichts Sinnvolles tun — lieber sofort sichtbar scheitern als später mit einem kryptischen DB-Fehler. |
| `LOG_FILE` | `os.getenv(...)` (gibt `None`) | **Optional.** Design-Entscheidung 3 aus dem Tutorial: Kein `LOG_FILE` gesetzt → einfach nur Konsolen-Logging, kein Fehler. |
| `LOG_LEVEL` | `os.getenv(..., "INFO")` (Default) | **Optional mit sinnvollem Default.** Fehlt die Variable, läuft die Anwendung trotzdem — nur eben auf `INFO`. |

Merksatz: **`os.environ[key]` für „muss da sein, sonst ist alles kaputt", `os.getenv(key, default)` für „nice to have, hat einen sinnvollen Normalfall".** Das ist derselbe Gedanke wie bei Pflicht- vs. optionalen Funktionsargumenten — nur eine Ebene weiter außen, auf Prozessebene.

## `os.environ` ist beschreibbar — und das ist der Haken bei `load_dotenv()`

`os.environ` lässt sich nicht nur lesen, sondern auch **beschreiben**:

```python
os.environ["LOG_LEVEL"] = "DEBUG"   # setzt die Variable für den Rest des Prozesses (und Kindprozesse)
```

Genau das macht `python-dotenv`s `load_dotenv()` (Aufruf in `db.py`, Zeile 8) im Hintergrund: Es liest die `.env`-Datei zeilenweise und schreibt jedes `KEY=VALUE` per `os.environ[KEY] = VALUE` in die Umgebung — **aber nur, wenn `load_dotenv()` tatsächlich ausgeführt wurde**, sprich nur, wenn das Modul, das den Aufruf enthält (`db.py`), auch importiert wurde. Das war genau der Grund, warum euer `LOG_FILE` in der `.env` beim `pricing`-Entrypoint bisher ins Leere lief: Solange `main.py` nichts importiert, das `db.py` mitzieht, sieht `os.getenv("LOG_FILE")` in `setup_logging()` nur die *echte* Shell-Umgebung, nicht den Inhalt der `.env`-Datei.

> [!tip] Warum das in Tests nützlich ist
> Weil `os.environ` ein ganz normales, veränderliches Objekt ist, kann `pytest`s `monkeypatch`-Fixture es gezielt für einen einzelnen Test verändern und danach automatisch zurücksetzen: `monkeypatch.setenv("LOG_LEVEL", "DEBUG")`, `monkeypatch.delenv("DB_PRICING_PW", raising=False)` — genau das nutzt ihr später in Block 11 (`test_fehlendes_passwort_wird_uebersetzt`). Ohne dass `os.environ` beschreibbar wäre, gäbe es diesen Mechanismus nicht.

# Der Rest von `os` — kurz, zur Einordnung

Nicht aktuell im Projekt gebraucht, aber gut zu wissen, wo es hingehört:

- **`os.getcwd()`** — aktuelles Arbeitsverzeichnis. Relevant für euch indirekt: `Path("daily_data")` in `main.py` ist ein **relativer** Pfad, der genau von diesem Arbeitsverzeichnis abhängt (daher der Hinweis in der `.env`-Notiz zur Frage, aus welchem Ordner heraus ihr `pricing` startet).
- **`os.name`** — Plattform-Kürzel (`"posix"` unter Linux/Mac, `"nt"` unter Windows). Für portable Fallunterscheidungen.
- **`os.path`** — der **alte**, string-basierte Weg, mit Pfaden zu arbeiten (`os.path.join(...)`, `os.path.exists(...)`, `os.path.basename(...)`). Euer Projekt nutzt komplett `pathlib.Path` (`resolve_csv()`, `data_dir.glob(...)`) — das ist die modernere, objektorientierte Alternative, die seit Python 3.4 `os.path` für neuen Code weitgehend abgelöst hat. Faustregel: **neuer Code → `pathlib.Path`**, `os.path` begegnet dir vor allem in älteren Codebasen.
- **`os.makedirs()` / `os.listdir()` / `os.remove()`** — Dateisystem-Operationen; die `pathlib`-Entsprechungen sind `Path.mkdir(parents=True)`, `Path.iterdir()`, `Path.unlink()`.
- **Prozess-Funktionen** (`os.system()`, `os.fork()`, `os.getpid()`) — Prozess-Steuerung/-Erzeugung. Kommt in einer Batch-Anwendung wie eurer normalerweise nicht vor; `os.system()` insbesondere gilt als veraltet (Sicherheitsrisiko durch Shell-Injection) — dafür heute `subprocess`.

# Eine Sache, die aus dem Logging-Kapitel besonders wichtig ist

**Niemals `os.environ` (oder Teile davon) ungefiltert loggen.** `DB_PRICING_PW` steckt da drin — ein `logger.debug("Umgebung: %s", dict(os.environ))` irgendwo im Code wäre ein Credential-Leak im Logfile. Das ist auch, warum eure `ConfigError`-Meldung aus Block 4 zwar den **Namen** der fehlenden Variable nennt (`DB_PRICING_PW ist nicht gesetzt`), aber nie deren **Wert** — der Name ist zum Debuggen hilfreich, der Wert wäre ein Geheimnis, das nichts im Log verloren hat.

# Zusammenfassung

- `os` = Schnittstelle zum **Betriebssystem** (Gegenstück zu `sys` = Interpreter).
- `os.environ[key]` wirft `KeyError` bei fehlendem Key → für **Pflicht**-Variablen (`DB_PRICING_PW`).
- `os.getenv(key, default)` gibt `None`/Default zurück, wirft nie → für **optionale** Variablen (`LOG_FILE`, `LOG_LEVEL`).
- `os.environ` ist beschreibbar — Grundlage für `load_dotenv()` (`.env` → Umgebung) und `monkeypatch.setenv/delenv` in Tests.
- Für Pfade: `pathlib.Path` statt `os.path` — Standard in eurem Projekt und moderner Python-Stil generell.
- Secrets aus `os.environ` gehören nie ins Log — nur der Variablenname, nie der Wert.
