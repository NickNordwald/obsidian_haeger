---
tags:
  - documentation
type: notiz
date: 2026-07-28
---

Abhakbarer **Plan für Logging & Exceptionhandling** der Pricing-Anwendung — im Tutorial-Stil, damit die neue Syntax genau dort erklärt wird, wo du sie zum ersten Mal brauchst. Begleitnotiz zum [[Umsetzungsstand (Logging & Exceptionhandling)|Umsetzungsstand]]; Theorie dahinter: [[1.4 Logging & Exceptionhandling - Vertiefung|1.4 Vertiefung]]. Vorbild-Notiz aus dem letzten Kapitel: [[Test-Suite Plan & Syntax-Tutorial|Test-Suite Plan & Syntax-Tutorial]].

> [!info] So arbeitest du mit dieser Liste
> Die Blöcke sind **von einfach nach komplex** sortiert; jeder führt genau **eine** neue Technik ein und baut auf dem vorigen auf. Der Reihe nach abarbeiten, `- [ ]` abhaken, wenn es läuft und die Tests grün sind. **Du schreibst den Code, ich reviewe.** Die Schnipsel sind *Muster*, keine fertigen Lösungen — Namen, Texte und Feinheiten entscheidest du.
>
> In jedem Block sagt ein **→ Was du tust**-Absatz konkret, welche Datei und welche Zeile dran ist. Geänderte Zeilen sind im Code mit `# ←` markiert.

# Das Design — und warum es so aussieht

Bevor du die erste Zeile änderst: Das hier ist das Zielbild. Es ist bewusst **klein** gehalten — die Anwendung ist ein Batch-Lauf mit zwei Entrypoints, kein verteiltes System.

## Die vier Bausteine

| Baustein | Wo | Aufgabe |
|---|---|---|
| **Ein Setup-Modul** | `logging_setup.py` (neu) | Konfiguriert das Logging **einmal**, aufgerufen von beiden Entrypoints |
| **Ein Ausnahme-Stammbaum** | `exceptions.py` (neu) | `PricingError` als Basis + drei konkrete Fälle |
| **Übersetzung an den Rändern** | `db.py`, `sources.py`, `domain.py` | Technische Fehler → sprechende Fehler, mit `raise ... from e` |
| **Entscheidung ganz außen** | `main.py`, `ingest.py` | Fangen, `logger.exception()`, **Exit-Code** |

## Die fünf Design-Entscheidungen

**1. Ein einziger Regler: `LOG_LEVEL`.** Das Level kommt aus einer Umgebungsvariablen (Default `INFO`) und steuert den Logger `pricing`. Keine zweite Schwelle an den Handlern.
*Warum:* Zwei Regler (Logger-Level *und* Handler-Level) sind der übliche Weg, aber man muss dann bei jeder Frage „warum sehe ich das nicht?" an zwei Stellen suchen. Für ein Projekt dieser Größe ist ein Knopf ehrlicher. Der Unterschied zwischen Konsole und Datei liegt bei uns im **Format**, nicht im Level.

**2. Logs auf stderr, Programmausgabe auf stdout.** Dein `print(f"Höchste Empfehlung: {top}")` in `main.py` **bleibt ein `print`**.
*Warum:* Das ist keine Nachlässigkeit, sondern eine Trennung mit Zweck — das *Ergebnis* des Programms kann man in eine Datei umleiten oder weiterpipen (`pricing > ergebnis.txt`), ohne dass Lograuschen mitkommt. Genau deshalb loggt Python standardmäßig nach stderr.

**3. Datei-Handler nur, wenn `LOG_FILE` gesetzt ist.** Ohne die Variable schreibt die Anwendung nur auf die Konsole.
*Warum:* Ein Batch-Job unter cron hat keine Konsole, die jemand liest — da braucht es eine Datei mit Rotation. Beim lokalen Entwickeln wäre eine Logdatei nur Müll im Arbeitsverzeichnis. Die Bedingung kostet drei Zeilen und macht beides richtig.

**4. Handler hängen an **root**, nicht am `pricing`-Logger — und kein `propagate: False`.**
*Warum:* Damit gibt es keine doppelte Ausgabe, das Level von `pricing` steuert trotzdem alles Eigene, und `root: WARNING` hält gesprächige Fremdbibliotheken (`sqlalchemy.engine`) leise. Der eigentliche Grund ist aber ein anderer — siehe die Warnung unten zu `caplog`.

**5. Eine Exception-Basisklasse pro Projekt.** `PricingError`, davon `ConfigError`, `DataSourceError`, `DomainError`.
*Warum:* Erst dadurch kann `main()` in **einer** Zeile „alles Erwartete" fangen (`except PricingError` → Exit-Code 1) und alles andere als Bug behandeln (Exit-Code 2). Drei Unterklassen sind genug — sie entsprechen den drei Fehlerquellen, die es real gibt: Konfiguration, Eingabedaten, Domänen-Invarianten.

## Was bewusst *nicht* drin ist

- **Kein `structlog`/`loguru`** — die Standardbibliothek reicht, siehe [[1.4 Logging & Exceptionhandling - Vertiefung#Was heißt das für unser Projekt?|Vertiefung]].
- **Kein JSON-Logging** — lohnt erst, wenn die Logs in eine Aggregationsplattform gehen.
- **Keine Run-ID / kein `LoggerAdapter`** — bei einem Lauf pro Aufruf gibt es nichts zu korrelieren.
- **Kein Logging in `domain.py`** — reine Wertobjekte loggen nicht, sie werfen. (Deshalb hat `domain.py` bis heute keinen Logger, und das bleibt so.)
- **Kein `logger.exception` in den Adaptern** — die werfen nur; geloggt wird am Rand. Das ist die Regel „Log *oder* raise, nicht beides".

## Zielbild der Dateien

```
src/pricing/
├── logging_setup.py     ← NEU  (Block 1)
├── exceptions.py        ← NEU  (Block 3)
├── main.py              ← Blöcke 2, 7
├── domain.py            ← Block 6
├── core/app.py          ← Block 8
├── core/recommendations.py ← Block 9
└── adapters/
    ├── db.py            ← Block 4
    ├── sources.py       ← Block 5
    └── ingest.py        ← Block 10
```

> [!warning] Eine Falle, die dich sonst Zeit kostet
> **`setup_logging()` darf in Unit-Tests nicht aufgerufen werden.** `dictConfig` **ersetzt die Handler am root-Logger** — und der Handler, mit dem pytest `caplog` befüllt, hängt genau dort. Ich habe das ausprobiert: Ein Test, der erst `setup_logging()` aufruft und dann über `caplog` prüft, sieht **nichts** und wird rot.
> Das ist kein Problem, solange die Konfiguration dort bleibt, wo sie hingehört (im Entrypoint) — deine bestehenden 53 Tests rufen sie nicht auf und bleiben grün. Merk es dir nur für den Moment, in dem du auf die Idee kommst, „das Logging mal im Test zu initialisieren".

---

# Block 0 — Werkzeug schärfen (5 Minuten)

**Neue Technik:** Linter-Regelgruppen, die dich beim Rest des Tutorials automatisch korrigieren.

**→ Was du tust:** In `pyproject.toml` die `select`-Liste um drei Gruppen ergänzen.

```toml
[tool.ruff.lint]
select = [
    "E", "W", "F", "I", "N", "UP", "B", "SIM",
    "G",    # ← NEU: flake8-logging-format — erzwingt parametrisiertes Logging
    "LOG",  # ← NEU: flake8-logging — Fehler im Umgang mit dem logging-Modul
    "TRY",  # ← NEU: tryceratops — Exceptionhandling-Anti-Muster
]
```

Danach einmal `ruff check .` laufen lassen. Wenn jetzt schon etwas gemeldet wird: lies die Meldung, bevor du sie behebst — die `TRY`-Gruppe hat ein paar Regeln, über die man diskutieren kann (z. B. `TRY003`, „keine langen Meldungen direkt im `raise`"). Wenn dich eine davon stört, nimm sie bewusst in `ignore` auf, statt sie stillschweigend zu umgehen.

- [x] `G`, `LOG`, `TRY` aktiviert
- [x] `ruff check .` läuft durch (oder gemeldete Regeln bewusst in `ignore`) — `TRY003` für `domain.py` bewusst per `per-file-ignores` deaktiviert (Details: [[Umsetzungsstand (Logging & Exceptionhandling)#Gelernte Lektionen|Umsetzungsstand]]); die verbleibenden 2 Treffer in `sources.py` sind erwartet und lösen sich mit Block 5

---

# Block 1 — Das Setup-Modul

**Neue Technik:** `logging.config.dictConfig` — Konfiguration als Datenstruktur statt als Aufrufkette.

**→ Was du tust:** Neue Datei `src/pricing/logging_setup.py`. Eine einzige öffentliche Funktion.

```python
import logging.config      # ← wichtig: logging.config muss separat importiert werden
import os


def setup_logging(level: str | None = None) -> None:
    """Konfiguriert das Logging der Anwendung. Nur aus einem Entrypoint aufrufen."""
    level = (level or os.getenv("LOG_LEVEL", "INFO")).upper()

    handlers: dict[str, dict[str, object]] = {
        "konsole": {
            "class": "logging.StreamHandler",
            "formatter": "knapp",
            "stream": "ext://sys.stderr",     # ← "ext://" = Verweis auf ein Python-Objekt
        }
    }

    if log_file := os.getenv("LOG_FILE"):      # ← Walross-Operator: zuweisen + prüfen
        handlers["datei"] = {
            "class": "logging.handlers.RotatingFileHandler",
            "formatter": "ausfuehrlich",
            "filename": log_file,
            "maxBytes": ...,                   # ← du: wie groß darf eine Datei werden?
            "backupCount": ...,                # ← du: wie viele Altstände behalten?
            "encoding": "utf-8",
        }

    logging.config.dictConfig(
        {
            "version": 1,                       # ← Pflichtfeld, immer 1
            "disable_existing_loggers": False,  # ← sonst werden Logger stumm, die vorher entstanden
            "formatters": {
                "knapp": {"format": ...},         # ← du: Level + Nachricht reichen
                "ausfuehrlich": {"format": ..., "datefmt": ...},  # ← du: + Zeit, Modul, Zeile
            },
            "handlers": handlers,
            "loggers": {"pricing": {"level": level}},   # ← nur Level, KEINE Handler
            "root": {"level": "WARNING", "handlers": list(handlers)},
        }
    )
```

Drei Dinge zum Verstehen, bevor du es abtippst:

- **`"loggers": {"pricing": {"level": level}}` ohne `handlers`** ist Absicht. Die Meldung wandert von `pricing.core.app` nach oben zu root und wird dort ausgegeben. Stünde hier zusätzlich ein Handler, käme jede Zeile **doppelt**.
- **`root` auf `WARNING`** betrifft nur Logger *außerhalb* von `pricing` — also Fremdbibliotheken. Dein eigener Baum hängt an `pricing` und hat sein eigenes Level. (Erinnerung aus der [[1.4 Logging & Exceptionhandling - Vertiefung#Der Weg einer Logmeldung — und die zwei Level-Prüfungen|Vertiefung]]: Beim Hochreichen wird das Level der Vorfahren **nicht** noch einmal geprüft.)
- **Für den `ausfuehrlich`-Formatter** lohnen sich `%(asctime)s`, `%(levelname)-8s`, `%(name)s` und `%(lineno)d`. Der Modulname ist der Grund, warum wir überall `getLogger(__name__)` benutzen — ohne ihn im Format verschenkst du diese Information.

- [x] `logging_setup.py` angelegt, `setup_logging()` implementiert
- [x] Manuell geprüft: ohne `LOG_FILE` nur Konsole, mit `LOG_FILE=/tmp/pricing.log` zusätzlich Datei
- [x] `LOG_LEVEL=DEBUG` zeigt mehr als `LOG_LEVEL=INFO`

---

# Block 2 — Exit-Codes: `main()` gibt etwas zurück

**Neue Technik:** `sys.exit(main())` — der Rückgabewert wird zum Prozess-Exit-Code.

**→ Was du tust:** In `src/pricing/main.py` die Signatur ändern, `setup_logging()` als **erste** Anweisung aufrufen, den alten `basicConfig`-Aufruf entfernen. Das `try/except` kommt erst in Block 7 — hier nur das Gerüst.

```python
import sys                                    # ←
from .logging_setup import setup_logging      # ←


def main() -> int:                            # ← war: -> None
    setup_logging()                           # ← ersetzt logging.basicConfig(...)

    app = PricingApp(...)                     # unverändert
    top = app.run()
    print(f"Höchste Empfehlung: {top}")        # ← bleibt print (Ergebnis, kein Log)
    return 0                                  # ←


if __name__ == "__main__":
    sys.exit(main())                          # ←
```

**Warum `setup_logging()` ganz oben steht:** Alles, was vor der Konfiguration geloggt wird, geht durch die Standardeinstellung (nur `WARNING`, kein Format) — also verloren.

Prüfen kannst du es direkt in der Shell:

```bash
pricing ; echo "Exit-Code: $?"      # → 0
```

- [x] `main()` gibt `int` zurück, `sys.exit(main())` am Ende
- [x] `setup_logging()` als erste Anweisung, `basicConfig` entfernt
- [x] `echo $?` zeigt 0 nach erfolgreichem Lauf
- [x] mypy `strict` bleibt grün (Rückgabetyp!)

---

# Block 3 — Der Ausnahme-Stammbaum

**Neue Technik:** Exception-Hierarchie mit **Mehrfachvererbung** für Rückwärtskompatibilität.

**→ Was du tust:** Neue Datei `src/pricing/exceptions.py`.

```python
class PricingError(Exception):
    """Basis für alle erwarteten Fehler dieser Anwendung."""


class ConfigError(PricingError):
    """Eine notwendige Konfiguration fehlt oder ist unbrauchbar."""


class DataSourceError(PricingError):
    """Eine Eingabequelle fehlt, ist mehrdeutig oder nicht lesbar."""


class DomainError(PricingError, ValueError):    # ← zwei Basisklassen!
    """Eine fachliche Invariante wurde verletzt."""
```

Die Zeile `class DomainError(PricingError, ValueError)` ist der interessante Trick. Deine drei bestehenden Tests prüfen `pytest.raises(ValueError, match=...)` — würde `DomainError` nur von `PricingError` erben, wären sie **rot**. Durch die zweite Basisklasse ist eine `DomainError` *gleichzeitig* ein `ValueError`: `except PricingError` fängt sie am Rand, `except ValueError` fängt sie weiterhin auch. Ich habe das nachgestellt, beide Varianten sind grün.

Das ist keine Spielerei, sondern das übliche Vorgehen, wenn man eine bestehende API erweitert, ohne Aufrufer zu brechen — dieselbe Idee steckt z. B. hinter `class TimeoutError(OSError)` in der Standardbibliothek.

> [!tip] Wo liegt die Datei?
> `exceptions.py` gehört **neben** `domain.py` in den Kern, nicht unter `adapters/`. Grund: Sowohl der Kern als auch die Adapter werfen diese Fehler; die Abhängigkeit muss nach innen zeigen (dasselbe Argument, das im Docstring von `ports.py` steht).

- [x] `exceptions.py` mit vier Klassen angelegt
- [x] Docstring pro Klasse (eine Zeile reicht, sagt *wann* sie fliegt)

---

# Block 4 — Erste Übersetzung: `db.py`

**Neue Technik:** `raise Sprechend(...) from e` — technischer Fehler wird fachlicher Fehler, Ursache bleibt erhalten.

Der Ist-Zustand ist das Lehrbuchbeispiel: `os.environ["DB_PRICING_PW"]` wirft bei fehlender Variable einen `KeyError`, dessen gesamte Nachricht `'DB_PRICING_PW'` lautet — ohne jeden Hinweis darauf, was zu tun ist. Und die Zeile steht **zweimal** im Modul (`get_dsn` und `get_engine`). Warum hier `os.environ[...]` (wirft) und nicht `os.getenv(...)` (gibt `None`) — Pflicht- vs. optionale Variable: [[Documentation/Traineeship/Python Basics/Das os-Modul.md#Umgebungsvariablen: der Teil, den ihr wirklich benutzt|Das os-Modul]].

**→ Was du tust:** In `src/pricing/adapters/db.py` eine kleine Hilfsfunktion einziehen und beide Stellen darauf umstellen.

```python
from ..exceptions import ConfigError          # ←


def _require_password() -> str:               # ← NEU
    try:
        return os.environ["DB_PRICING_PW"]
    except KeyError as e:
        raise ConfigError(
            "Umgebungsvariable DB_PRICING_PW ist nicht gesetzt "
            "(z. B. in .env eintragen)"
        ) from e                              # ← das 'from e' ist der Punkt


def get_dsn() -> str:
    pw = _require_password()                  # ← war: os.environ["DB_PRICING_PW"]
    ...
```

**Worauf ich beim Review schauen werde:**

- Steht wirklich `from e` da? Ohne das verlierst du die Vorgeschichte — nachlesen: [[1.4 Logging & Exceptionhandling - Vertiefung#Exception-Chaining: die Vorgeschichte mitloggen|Exception-Chaining]].
- Steht in der Meldung, **was zu tun ist**? „DB_PRICING_PW fehlt" ist halb so nützlich wie „… (z. B. in .env eintragen)".
- Wird hier **nicht** geloggt? Richtig ist: nur werfen. Der Adapter weiß nicht, ob der Aufrufer den Fehler behandeln kann.

- [x] `_require_password()` angelegt, beide Aufrufstellen umgestellt
- [x] `ConfigError` mit `from e`, Meldung nennt die Variable **und** den Lösungsweg
- [x] Manuell geprüft: `unset DB_PRICING_PW` → sprechende Fehlermeldung statt nacktem `KeyError`

---

# Block 5 — Exceptions mit Daten: `sources.py`

**Neue Technik:** Eine Exception, die **Attribute** trägt, nicht nur Text.

`resolve_csv()` wirft heute zweimal `FileNotFoundError` mit einer f-String-Nachricht. Der zweite Fall („mehrere Treffer") ist inhaltlich sogar falsch benannt — es ist kein *nicht gefunden*, sondern ein *mehrdeutig*.

**→ Was du tust:** In `src/pricing/adapters/sources.py` beide `raise` auf `DataSourceError` umstellen und die Bezugsdaten als Attribute mitgeben.

```python
class DataSourceError(PricingError):          # ← in exceptions.py erweitern
    def __init__(self, pattern: str, data_dir: Path, treffer: list[str]) -> None:
        super().__init__(...)                 # ← du: die menschenlesbare Meldung
        self.pattern = pattern                # ← die Daten für den Code
        self.data_dir = data_dir
        self.treffer = treffer
```

```python
# in resolve_csv()
if len(matches) == 0:
    raise DataSourceError(pattern, data_dir, [])                       # ←
if len(matches) > 1:
    raise DataSourceError(pattern, data_dir, [m.name for m in matches])  # ←
```

**Die Designfrage, die du hier beantworten musst:** Ein Fehlertyp mit Attribut `treffer` für beide Fälle — oder zwei Typen (`QuelleFehltError`, `QuelleMehrdeutigError`)? Beides ist vertretbar. Meine Empfehlung ist die erste Variante, weil kein Aufrufer die beiden Fälle *unterschiedlich behandeln* wird (beide Male: Lauf abbrechen). Zwei Typen einzuführen, die niemand getrennt fängt, ist genau die Aufblähung, die du vermeiden wolltest. Wenn du anderer Meinung bist, schreib's in die Notiz — das ist eine legitime Diskussion.

Der Gewinn der Attribute zeigt sich im Test: `assert exc_info.value.pattern == "*_capacity.csv"` ist stabil, `assert "capacity" in str(e)` bricht beim nächsten Umformulieren der Meldung.

- [x] `DataSourceError` mit `__init__` und Attributen
- [x] Beide `raise` in `resolve_csv()` umgestellt
- [x] Meldung nennt Suchmuster **und** Verzeichnis
- [x] `from e` — Trickfrage: Hier gibt es **keine** ursprüngliche Exception (wir prüfen selbst mit `if`), also **kein** `from`. Nur wo du eine fremde Exception *fängst*, verkettest du.

---

# Block 6 — Domänen-Invarianten: `domain.py`

**Neue Technik:** Keine neue — hier wendest du Block 3 an und siehst, ob die Mehrfachvererbung hält.

**→ Was du tust:** In `src/pricing/domain.py` die drei `raise ValueError(...)` auf `DomainError` umstellen. Die Meldungstexte bleiben **wortgleich** — sonst brechen die `match=`-Ausdrücke in den Tests.

```python
from .exceptions import DomainError           # ←

if booked_units < 0:
    raise DomainError(                        # ← war: ValueError
        f"booked_units darf nicht negativ sein "
        f"(product_id={product_id}, date={calendar_date}, got={booked_units})"
    )
```

Danach `pytest -q` — **alle 53 Tests müssen ohne jede Teständerung grün bleiben.** Wenn nicht, ist die Vererbung falsch herum oder ein Meldungstext hat sich verändert.

- [x] Alle drei Stellen in `domain.py` umgestellt (Zeilen ~29, ~59, ~93)
- [x] Meldungstexte unverändert
- [x] 53 Tests weiterhin grün, ohne dass du einen Test angefasst hast
- [x] `domain.py` hat **weiterhin keinen Logger** (bewusst)

---

# Block 7 — Die Fehlerbehandlung am Rand

**Neue Technik:** `logger.exception()` im `except`-Block + differenzierte Exit-Codes.

Jetzt zahlt sich der Stammbaum aus: **eine** Zeile fängt alles Erwartete.

**→ Was du tust:** In `src/pricing/main.py` den Rumpf von `main()` in ein `try` legen, mit **zwei** `except`-Zweigen.

```python
logger = logging.getLogger(__name__)          # ← ganz oben im Modul


def main() -> int:
    setup_logging()

    try:
        app = PricingApp(...)                 # ← CsvDataSource wirft schon HIER
        top = app.run()
    except PricingError:                      # ← erwartete Fehler
        logger.exception("Preislauf abgebrochen")
        return 1
    except Exception:                         # ← der eine legitime breite Fang
        logger.exception("Unerwarteter Fehler im Preislauf")
        return 2

    print(f"Höchste Empfehlung: {top}")
    return 0
```

Vier Dinge, auf die es ankommt:

1. **`logger.exception(...)`, nicht `logger.error(str(e))`.** Ersteres hängt den kompletten Traceback an. Und: **kein `as e`** — du brauchst die Variable gar nicht, `exception()` holt sich die aktive Exception selbst. Wenn du `as e` schreibst und `e` dann in die Meldung packst, meldet ruff dir `TRY401`.
2. **Der Konstruktoraufruf gehört mit ins `try`.** `CsvDataSource(Path("daily_data"))` ruft dreimal `resolve_csv()` auf — der häufigste Fehlerfall überhaupt (Datei fehlt) passiert also schon beim Bauen des Objekts, nicht erst bei `run()`.
3. **Zwei Exit-Codes, nicht einer.** `1` heißt „damit war zu rechnen", `2` heißt „das ist ein Bug". Ein Scheduler kann darauf unterschiedlich reagieren.
4. **`print` steht außerhalb des `try`** — im Fehlerfall gibt es kein Ergebnis auszugeben.

Prüfen:

```bash
mv daily_data daily_data_weg
pricing ; echo "Exit-Code: $?"     # → Traceback im Log, Exit-Code 1
mv daily_data_weg daily_data
```

- [x] Zwei `except`-Zweige, `PricingError` zuerst (Reihenfolge ist zwingend — die spezifischere zuerst!)
- [x] `logger.exception(...)` ohne `as e`
- [x] Exit-Codes 0 / 1 / 2, manuell durchgespielt
- [x] `ruff check .` grün (die `TRY`-Gruppe schaut hier genau hin)

---

# Block 8 — Der Lauf erzählt, was er tut

**Neue Technik:** `time.perf_counter()` für Laufzeitmessung + `INFO`-Logs im Lebenszyklus.

`core/app.py` hat bis heute **keinen Logger**. Das ist die größte Lücke: Nach einem Lauf weiß niemand, ob Daten da waren, wie viele Empfehlungen entstanden sind und wie lange es gedauert hat.

**→ Was du tust:** In `src/pricing/core/app.py` einen Modul-Logger anlegen und `_compute()` sowie `run()` mit `INFO`-Meldungen versehen.

```python
import logging
import time                                   # ←

logger = logging.getLogger(__name__)          # ←


def _compute(self) -> list[PriceRecommendation]:
    capacity = self._source.load_capacity()
    bookings = self._source.load_bookings()
    logger.info("%d Kapazitätszeilen und %d Buchungen geladen",   # ←
                len(capacity), len(bookings))
    units = self._builder.build(capacity, bookings)
    recs = self._engine.calculate_all(units)
    logger.info("%d Empfehlungen berechnet aus %d Einheiten", len(recs), len(units))  # ←
    return recs


def run(self) -> PriceRecommendation | None:
    start = time.perf_counter()               # ←
    logger.info("Preislauf gestartet")        # ←
    recs = self._compute()
    self._writer.write(recs)
    logger.info("Preislauf beendet: %d Empfehlungen in %.2f s",   # ←
                len(recs), time.perf_counter() - start)
    return self._engine.get_max(recs)
```

**Warum `perf_counter()` und nicht `time.time()`:** `perf_counter` ist monoton — er läuft nicht rückwärts, wenn die Systemuhr korrigiert wird (NTP, Sommerzeit). Für Zeit*messung* immer `perf_counter`, für Zeit*stempel* `datetime`.

**Warum die Zahlen als Argumente und nicht im f-String:** parametrisiertes Logging, siehe Block 0 — ruff `G004` würde dich sonst erinnern.

Danach einmal laufen lassen: Du solltest jetzt eine erzählbare Geschichte im Log sehen — gestartet, geladen, berechnet, beendet mit Dauer.

- [x] Modul-Logger in `app.py`
- [x] `INFO`-Logs: Start, geladene Mengen, berechnete Empfehlungen, Ende mit Dauer
- [x] `perf_counter()` statt `time.time()`
- [x] Ein Lauf auf `INFO` liest sich als sinnvolle Zusammenfassung (**nicht mehr als ~6 Zeilen!**)

---

# Block 9 — Feinschliff der Level

**Neue Technik:** Keine neue — hier triffst du zwei **Entscheidungen** aus der [[1.4 Logging & Exceptionhandling - Vertiefung#Vorschlag: Level-Matrix für die Pricing-Anwendung|Level-Matrix]].

**→ Was du tust (a):** In `src/pricing/core/recommendations.py`, Zeile ~76: Entscheide, ob `total_capacity == 0` wirklich eine `WARNING` ist.

```python
if skipped_zero:
    logger.warning("%d Einheiten mit total_capacity == 0 übersprungen.", skipped_zero)
    # ← ist das ein Missstand oder der Normalfall (ausgebucht/geschlossen)?
```

Ist es der Normalfall, gehört es auf `INFO` — sonst warnt die Anwendung bei jedem Lauf über etwas, worauf niemand reagiert. Das ist eine **fachliche** Frage, die ich dir nicht abnehmen kann. Die anderen drei `WARNING`s (negative Kapazität, verwaiste Buchungsnächte, fehlende capacity-Zeile) halte ich für richtig eingestuft.

**→ Was du tust (b):** Zwei bis drei `DEBUG`-Zeilen für Detailschritte ergänzen — z. B. in `sources.py` nach dem Auflösen der Dateien, welche Datei tatsächlich genommen wurde:

```python
logger.debug("Verwende %s für %s", pfad.name, suffix)     # ←
```

**Die eine Regel, die du hier nicht brechen darfst:** Nichts, was **pro Datensatz** passiert, kommt auf `INFO`. Eine `DEBUG`-Zeile pro CSV-Datei ist gut (drei Zeilen); eine `DEBUG`-Zeile pro Buchung wäre bei 50 000 Buchungen schon grenzwertig — dann lieber gar nicht.

- [x] Entscheidung zu `total_capacity == 0` getroffen und im [[Umsetzungsstand (Logging & Exceptionhandling)|Umsetzungsstand]] begründet
- [x] 2–3 sinnvolle `DEBUG`-Zeilen ergänzt
- [x] `LOG_LEVEL=DEBUG` zeigt merklich mehr, `INFO` bleibt aufgeräumt

---

# Block 10 — Der zweite Entrypoint

**Neue Technik:** Keine neue — bewusste Wiederholung, damit das Muster sitzt.

`adapters/ingest.py` hat ein eigenes `main()` und steht als `pricing-ingest` in `[project.scripts]`. Es hat heute **kein** Logging-Setup und **keine** Fehlerbehandlung.

**→ Was du tust:** In `src/pricing/adapters/ingest.py` dieselbe Behandlung wie in den Blöcken 2 und 7 — `setup_logging()`, `-> int`, zwei `except`-Zweige, `sys.exit(main())`. Zusätzlich eine `INFO`-Zeile am Ende von `run_ingest()`, die sagt, wie viel geschrieben wurde.

```python
def main() -> int:                            # ←
    setup_logging()                           # ←
    try:
        with connect() as conn:
            run_ingest(conn, Path("daily_data"))
    except PricingError:
        logger.exception("Ingest abgebrochen")
        return 1
    except Exception:
        logger.exception("Unerwarteter Fehler im Ingest")
        return 2
    return 0
```

**Beobachte dabei etwas:** Du schreibst denselben Block zum zweiten Mal. Bei einem *dritten* Entrypoint wäre das der Moment, ihn in einen Dekorator oder eine `run_entrypoint(func)`-Hilfsfunktion zu ziehen. Bei zwei ist Duplikation die billigere Lösung — abstrahieren lohnt sich ab drei. (Das ist kein Logging-Thema, sondern eine allgemeine Regel, die man beim zweiten Mal aber gut *fühlen* kann.)

- [x] `ingest.main()` mit Setup, Exit-Codes und zwei `except`-Zweigen
- [x] `INFO`-Abschlusszeile in `run_ingest()`
- [x] Beide Entrypoints (`pricing`, `pricing-ingest`) verhalten sich gleich

---

# Block 11 — Die Tests

**Neue Technik:** `record.exc_info` prüfen und `__cause__` in `pytest.raises` auswerten.

Aus [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]] kennst du `caplog` und `pytest.raises`. Neu ist, dass du jetzt prüfst, ob **wirklich ein Traceback** dranhing und ob die **Verkettung** funktioniert hat.

**→ Was du tust:** Neue Datei `tests/test_exceptions.py` (oder Ergänzung der bestehenden Dateien, wo es thematisch passt).

```python
def test_fehlendes_passwort_wird_uebersetzt(monkeypatch: pytest.MonkeyPatch) -> None:
    monkeypatch.delenv("DB_PRICING_PW", raising=False)

    with pytest.raises(ConfigError) as exc_info:
        get_dsn()

    assert isinstance(exc_info.value.__cause__, KeyError)   # ← 'from e' hat gegriffen
    assert "DB_PRICING_PW" in str(exc_info.value)


def test_fehlende_csv_meldet_muster(tmp_path: Path) -> None:
    with pytest.raises(DataSourceError) as exc_info:
        resolve_csv(tmp_path, "capacity")

    assert exc_info.value.pattern == "*_capacity.csv"       # ← Attribut statt Textsuche
    assert exc_info.value.treffer == []
```

Und die Assertion, die das wichtigste Anti-Muster abfängt:

```python
def test_abbruch_loggt_mit_traceback(caplog: pytest.LogCaptureFixture, ...) -> None:
    with caplog.at_level(logging.ERROR):
        rc = main()

    assert rc == 1
    record = caplog.records[0]
    assert record.exc_info is not None      # ← wäre bei logger.error(str(e)) None
```

> [!warning] Für den letzten Test brauchst du einen Kniff
> `main()` ruft `setup_logging()` auf — und das macht `caplog` blind (siehe Warnung ganz oben). Zwei saubere Wege: `setup_logging` im Test per `monkeypatch.setattr` durch eine Leerfunktion ersetzen, **oder** die Fehlerbehandlung aus `main()` gar nicht über `main()` testen, sondern nur die Adapter-Übersetzungen (die ersten beiden Tests) — und `main()` manuell durchspielen. Ich würde mit `monkeypatch` arbeiten; wenn dir das zu fummelig wird, ist das ein legitimes „bewusst nicht getestet".

- [x] Test: `ConfigError` mit korrektem `__cause__`
- [x] Test: `DataSourceError` mit Attributen (beide Fälle: keine Datei / mehrere Dateien)
- [x] Test: Exception-Log hat `exc_info is not None`
- [x] `pytest --cov=pricing --cov-branch` — Kern weiterhin bei 100 %
- [x] Alle Tests grün, `ruff` + `mypy --strict` grün

---

# Fertig, wenn…

- [x] Ein normaler Lauf schreibt **≤ 6 `INFO`-Zeilen**, aus denen hervorgeht: was geladen, was berechnet, was geschrieben, wie lange.
- [x] `LOG_LEVEL=DEBUG` liefert Details, ohne dass der Code angefasst wurde.
- [x] `LOG_FILE=...` schreibt zusätzlich in eine rotierende Datei — mit Zeitstempel und Modulnamen.
- [x] Fehlende Eingabedatei → sprechende Meldung **mit Traceback im Log** und **Exit-Code 1**.
- [x] Fehlende `DB_PRICING_PW` → sagt, welche Variable fehlt und was zu tun ist.
- [x] Kein `except:` ohne Typ, kein `except: pass`, kein `logger.error(str(e))` im Projekt.
- [x] Beide Entrypoints verhalten sich gleich.
- [x] `pytest`, `ruff check`, `mypy` grün.
- [x] Die **gelernten Lektionen** im [[Umsetzungsstand (Logging & Exceptionhandling)|Umsetzungsstand]] sind gefüllt — der Teil, der beim nächsten Kapitel am meisten wert ist.
