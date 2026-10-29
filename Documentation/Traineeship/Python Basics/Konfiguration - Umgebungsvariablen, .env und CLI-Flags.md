---
tags:
  - documentation
---

Wie Konfiguration von außen in ein Python-Programm kommt — die vier Wege (`os.environ`, `python-dotenv`, `pydantic-settings`, `argparse`), was jeder kann, und wann man welchen nimmt. Baut auf [[Documentation/Traineeship/Python Basics/Das os-Modul.md|Das os-Modul]] auf, wo `os.environ` und `os.getenv` schon eingeführt sind.

Alle Beispiele stammen aus der Pricing-Anwendung: `DB_PRICING_PW`, `LOG_FILE`, `LOG_LEVEL` und die `-v`/`-vv`-Flags.

# Inhalt

- [[#Die Grundfrage]]
- [[#Ebene 1 - Umgebungsvariablen und os.environ]]
- [[#Ebene 2 - .env-Dateien und python-dotenv]]
- [[#Ebene 3 - pydantic-settings]]
- [[#Der andere Kanal - argparse]]
- [[#Wann Flag, wann Umgebung, wann Datei?]]
- [[#Die Rangfolge]]
- [[#Sicherheit]]
- [[#Spickzettel]]

---

# Die Grundfrage

Ein Programm braucht Werte, die **nicht im Code stehen dürfen**: Passwörter, Datenbankadressen, Pfade, Schalter. Warum nicht im Code?

- **Geheimnisse** gehören nicht ins Repository.
- **Umgebungen unterscheiden sich**: Dein Laptop, ein Testsystem und die Produktion brauchen andere Werte — bei identischem Code.
- **Manches ändert sich pro Aufruf**: „heute bitte ausführlicher loggen".

Daraus ergeben sich zwei getrennte Kanäle, die man nicht verwechseln sollte:

| Kanal | Ändert sich … | Beispiel |
|---|---|---|
| **Umgebung** | pro *Maschine/Deployment* | `DB_PRICING_PW`, `LOG_FILE` |
| **Kommandozeile** | pro *Aufruf* | `pricing -vv` |

---

# Ebene 1 — Umgebungsvariablen und `os.environ`

Eine **Umgebungsvariable** ist ein Name-Wert-Paar, das zum Prozess gehört. Zwei Eigenschaften erklären fast alles:

1. **Werte sind immer Strings.** `PORT=8000` ist die Zeichenkette `"8000"`, nicht die Zahl.
2. **Kindprozesse erben die Umgebung des Elternprozesses** — aber nie umgekehrt.

Punkt 2 ist der Grund für eine häufige Verwirrung: Ein Python-Skript kann `os.environ` verändern, aber **die Shell merkt davon nichts**. Die Änderung gilt nur für den eigenen Prozess und alles, was er danach startet.

```python
import os

os.environ["DB_PRICING_PW"]        # KeyError, wenn nicht gesetzt
os.getenv("LOG_FILE")              # None, wenn nicht gesetzt
os.getenv("LOG_LEVEL", "INFO")     # Default als zweites Argument
```

**Welche Form wann?**

- `os.environ[...]` für **Pflichtwerte** — es soll laut scheitern.
- `os.getenv(...)` für **optionale** Werte mit sinnvollem Rückfall.

In der Pricing-Anwendung war genau das die Unterscheidung: Das Passwort war Pflicht (`os.environ`, übersetzt in einen `ConfigError`), die Logdatei optional (`os.getenv` → ohne sie gibt es einfach keinen Datei-Handler).

**Setzen kann man sie auf drei Arten:**

```bash
export LOG_LEVEL=DEBUG      # gilt für die ganze Shell-Sitzung
LOG_LEVEL=DEBUG pricing     # gilt nur für diesen einen Aufruf
```
```python
os.environ["LOG_LEVEL"] = "DEBUG"   # gilt ab hier im Prozess (und für Kindprozesse)
```

---

# Ebene 2 — `.env`-Dateien und `python-dotenv`

> [!important] Eine `.env` ist **keine** Betriebssystem-Funktion
> Das Betriebssystem kennt keine `.env`-Dateien. Es ist reine Konvention plus eine Bibliothek, die die Datei liest und die Werte **nach `os.environ` schreibt**. Ohne den Aufruf passiert gar nichts.

```python
from dotenv import load_dotenv

load_dotenv()          # liest ./.env und fuellt os.environ
```

Danach funktioniert `os.getenv(...)` wie gewohnt — das ist der ganze Zauber. `load_dotenv()` ist ein **Ladeschritt**, kein Konfigurationssystem.

**Drei Verhaltensweisen, die man kennen sollte:**

**1. Bestehende Variablen werden nicht überschrieben.**

```
--- ohne gesetzte Variable ---
vorher : None
nachher: aus_datei

--- mit gesetzter Variable ---
vorher : aus_shell
nachher: aus_shell       <- .env verliert
```

Das ist der Default (`override=False`) und meistens genau richtig: Echte Umgebungsvariablen (Docker, CI) sollen eine mitkopierte `.env` schlagen. Mit `load_dotenv(override=True)` dreht man es um.

**2. Die Datei wird ab dem aktuellen Verzeichnis aufwärts gesucht.** Startest du dein Programm aus einem anderen Ordner, findet es die `.env` womöglich nicht — ein häufiger „bei mir läuft's"-Fall.

**3. Der Zeitpunkt entscheidet.** `load_dotenv()` muss laufen, **bevor** irgendjemand die Werte liest:

```python
def main() -> int:
    load_dotenv()          # MUSS zuerst kommen
    setup_logging(...)     # liest LOG_LEVEL
    engine = get_engine()  # liest DB_PRICING_PW
```

> [!warning] Der Klassiker
> Steht `load_dotenv()` als Nebeneffekt beim Import in einem Modul, hängt es davon ab, *wer wann importiert*. In der Pricing-Anwendung stand es in `db.py` — und weil der CSV-Pfad dieses Modul nie importierte, wurde die `.env` dort **nie gelesen**. Ohne Fehlermeldung, ohne Hinweis: `LOG_LEVEL` und `LOG_FILE` waren einfach wirkungslos.
>
> **Regel: Umgebung laden gehört in den Entrypoint (Composition Root), nicht in Bibliothekscode.**

Und in `.gitignore` gehört die Datei sowieso.

---

# Ebene 3 — `pydantic-settings`

Die Stufe darüber. `load_dotenv` bringt Strings in `os.environ`; danach musst du weiterhin selbst lesen, umwandeln und prüfen. `pydantic-settings` übernimmt das:

```python
from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    db_host: str = "localhost"        # sucht DB_HOST, sonst "localhost"
    db_user: str = "pricing"          # sucht DB_USER
    db_password: str = Field(validation_alias="DB_PRICING_PW")   # Pflicht
    log_file: str | None = None       # sucht LOG_FILE, optional
    log_level: str = "INFO"           # sucht LOG_LEVEL
```

**Die zentrale Regel: Der Feldname ist der Suchschlüssel** — groß-/kleinschreibungsunabhängig. `log_file` sucht `LOG_FILE`, `db_user` sucht `DB_USER`. Mehr braucht es nicht.

**`Field(...)` brauchst du nur, wenn du vom Standardverhalten abweichen willst.** Im Beispiel oben: Das Feld heißt `db_password`, würde also `DB_PASSWORD` suchen — die Datei hat aber `DB_PRICING_PW`. Der `validation_alias` verschiebt den Suchnamen. Hieße das Feld `db_pricing_pw`, wäre kein `Field` nötig.

Das `= None` bzw. `= "INFO"` ist der **Rückfallwert**, nicht der Wert. Es verhindert das Einlesen nicht — es macht das Feld optional. **Kein Default bedeutet Pflicht**: Fehlt der Wert, gibt es beim Start eine `ValidationError`.

**Was das gegenüber `os.getenv` bringt:**

| | `os.getenv` | `pydantic-settings` |
|---|---|---|
| Typen | alles `str` | `int`, `bool`, `date`, `Literal` … |
| Pflicht vs. optional | im Code verteilt | an der Felddefinition ablesbar |
| Fehlender Pflichtwert | `KeyError` beim ersten Zugriff | Fehler **beim Start**, mit Namen |
| Übersicht | verstreut über Module | **eine Klasse listet alles auf** |
| Validierung | selbst schreiben | `Literal[...]`, `ge=`, `le=` … |

Ein Beispiel, das den Unterschied greifbar macht: `LOG_LEVEL=DEBUGG` (Tippfehler) knallt sonst irgendwann tief in `logging.config`. Mit

```python
log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"
```

sagt Pydantic beim Start, welcher Wert falsch ist und welche erlaubt wären.

> [!tip] Es ersetzt `python-dotenv` nicht — es benutzt es
> Für `env_file=".env"` greift `pydantic-settings` intern auf `python-dotenv` zurück. Es ist also **dotenv plus eine typisierte Schicht darüber**, nicht die Alternative dazu. Nur der direkte `load_dotenv()`-Aufruf verschwindet.

Weitere nützliche Optionen: `env_prefix="PRICING_"` (dann sucht `log_file` nach `PRICING_LOG_FILE`) und `case_sensitive=True`, falls Groß-/Kleinschreibung doch zählen soll.

---

# Der andere Kanal — `argparse`

Umgebungsvariablen beschreiben, *wo* das Programm läuft. Kommandozeilen-Flags beschreiben, *was es dieses Mal tun soll*.

```python
import argparse

parser = argparse.ArgumentParser(prog="pricing")
parser.add_argument("-v", "--verbose", action="count", default=0)
args = parser.parse_args(argv)
```

**Die wichtigsten `action`-Typen:**

| `action` | Verhalten | Aufruf |
|---|---|---|
| *(Default)* | nimmt einen Wert entgegen | `--data-dir daily_data` |
| `"store_true"` | Schalter, `True` wenn vorhanden | `--dry-run` |
| `"count"` | zählt Vorkommen | `-v`, `-vv`, `-vvv` |
| `"append"` | sammelt mehrere Werte | `--tag a --tag b` |

Weitere Bausteine: `type=int` (konvertiert *und* validiert), `choices=["a", "b"]`, `required=True`, `nargs="+"` für mehrere Werte, und `add_subparsers()` für Unterbefehle wie `git commit` / `git push`.

> [!important] `parse_args(argv)` statt `parse_args()`
> Ohne Argument liest argparse `sys.argv` — und das enthält unter pytest die **pytest-Argumente**. Ergebnis: `SystemExit: 2` mit „unrecognized arguments: -q". Deshalb:
> ```python
> def main(argv: list[str] | None = None) -> int:
>     args = parser.parse_args(argv)     # Test ruft main([]) auf
> ```
> Nur so bleibt der Entrypoint testbar.

**Verbosity relativ statt fest zuordnen.** Eine feste Tabelle (`0→WARNING, 1→INFO, 2→DEBUG`) macht `-v` bei Kommandos wirkungslos, die ohnehin auf `INFO` starten. Eine Leiter löst das:

```python
_LADDER = ("ERROR", "WARNING", "INFO", "DEBUG")

idx = _LADDER.index(default_level.upper())
level = _LADDER[min(idx + args.verbose, len(_LADDER) - 1)]
```

Damit verschiebt `-v` immer um eine Stufe — egal, wo das Kommando startet — und `min(...)` deckelt oben.

## Wie argparse intern arbeitet

Das Betriebssystem liefert nur eine **flache Liste von Strings**. argparse ist die deklarative Schicht darüber: Du *beschreibst* die Argumente, die Bibliothek läuft die Liste ab.

**Es gibt genau drei Objekte:**

| Objekt | Was es ist |
|---|---|
| `ArgumentParser` | Die Beschreibung *einer* Kommandozeile. Enthält eine Liste von Aktionen. |
| **Aktion** | *Ein* Argument. Entsteht durch `add_argument()`. Weiß, auf welche Strings sie reagiert, wie viele Wörter sie frisst, wie sie umwandelt und unter welchem Namen sie ablegt. |
| `Namespace` | Das Ergebnis. Ein dummes Objekt mit Attributen. |

**Phase 1 — Bauzeit.** `add_argument()` parst nichts, es lässt nur eine Liste wachsen:

```
_HelpAction    dest='help'      option_strings=['-h','--help']   nargs=0
_CountAction   dest='verbose'   option_strings=['-v','--verbose'] nargs=0  default=0
_StoreAction   dest='data_dir'  option_strings=['--data-dir']     nargs=None
_StoreAction   dest='name'      option_strings=[]                 ← LEER = Positionsargument
```

**`option_strings=[]` ist das Erkennungsmerkmal eines Positionsarguments** — es wird nicht am Namen erkannt, sondern an der Stelle in der Liste.

**Phase 2 — Parse-Zeit.** `parse_args(argv)` läuft die Liste von links nach rechts:

- Wort beginnt mit `-` → in der Options-Tabelle nachschlagen; die Aktion sagt, wie viele *folgende* Wörter sie mitnimmt
- Wort beginnt nicht mit `-` → an die nächste Positions-Aktion, die noch Werte braucht

Danach: Defaults einsetzen, `required` prüfen, `Namespace` zurückgeben.

```
['hallo']                        -> Namespace(verbose=0, data_dir=None, name='hallo')
['-vvv', 'hallo']                -> Namespace(verbose=3, data_dir=None, name='hallo')
['--data-dir', 'daily', 'hallo'] -> Namespace(verbose=0, data_dir='daily', name='hallo')
['--data-dir=daily', 'hallo']    -> Namespace(verbose=0, data_dir='daily', name='hallo')
['hallo', '-v']                  -> Namespace(verbose=1, data_dir=None, name='hallo')
```

`-vvv` ist **ein** Wort, das die Aktion dreimal auslöst. Und Optionen dürfen vor *oder* nach dem Positionsargument stehen.

**Woher der Attributname kommt (`dest`):** Striche vorne weg, `-` wird `_`, bei mehreren Namen gewinnt der lange. `--data-dir` → `args.data_dir`, `-v/--verbose` → `args.verbose`.

> [!important] Fehler werfen keine Exception
> ```
> []                  -> SystemExit(2)  "the following arguments are required: name"
> ['hallo','zuviel']  -> SystemExit(2)  "unrecognized arguments: zuviel"
> ['--data-dir']      -> SystemExit(2)  "expected one argument"
> ```
> argparse druckt die Verwendung nach stderr und ruft `sys.exit(2)` — das ist **kein** fangbarer `ValueError`. Konsequenz fürs Design: `parse_args` gehört **vor** den `try`-Block eines Entrypoints, damit ein Aufruffehler nicht in die Fehlerbehandlung der Fachlogik gerät. (2 ist die Unix-Konvention für „falsch aufgerufen".)

## Unterbefehle (`add_subparsers`) — wie sie wirklich funktionieren

Sobald ein Kommando mehrere *Verben* hat (`pricing-apikey create` / `revoke` / `list`, wie `git commit` / `git push`), nimmt man Unterbefehle statt sich gegenseitig ausschließender Flags.

```python
parser = argparse.ArgumentParser(prog="pricing-apikey")
parser.add_argument("-v", action="count", default=0)
sub = parser.add_subparsers(dest="command", required=True)

p_create = sub.add_parser("create")
p_create.add_argument("--name", required=True)

p_revoke = sub.add_parser("revoke")
p_revoke.add_argument("key_id", type=int)
```

Der Mechanismus dahinter, in vier Sätzen:

1. **`add_subparsers()` erzeugt keinen Parser.** Es hängt dem Eltern-Parser ein **Positionsargument** an — sichtbar daran, dass die zugehörige Aktion `option_strings=[]` hat. Deshalb lautet die Fehlermeldung bei nacktem Aufruf „the following arguments are required: **command**".
2. **`sub.add_parser("create")` liefert einen vollwertigen, eigenständigen `ArgumentParser`.** Seine Argumente gelten nur *nach* diesem Wort — `--name` existiert für `revoke` schlicht nicht.
3. **Beim Parsen** verarbeitet der Eltern-Parser erst seine eigenen Optionen (`-v`), trifft dann auf das erste Nicht-Options-Wort, schlägt es unter den registrierten Namen nach und übergibt **den Rest der Liste** an diesen Unterparser.
4. **Beide schreiben in denselben `Namespace`.** Deshalb enthält `args` am Ende die Eltern-Argumente *und* die des gewählten Unterbefehls:

```
["-vv", "create", "--name", "X"]  →  Namespace(v=2, command='create', name='X')
["revoke", "7"]                   →  Namespace(v=0, command='revoke', key_id=7)
```

### Warum überhaupt Unterbefehle — der eigentliche Grund

Nicht Bequemlichkeit: Sie machen ungültige Kombinationen **nicht darstellbar**. Drei Varianten desselben Kommandos im Vergleich:

**A — alles flach in einen Parser** (`--create`, `--revoke N`, `--list`, `--name`):

```
['--create', '--name', 'X']              -> ok, gewollt
['--create', '--revoke', '3', '--list']  -> AKZEPTIERT — alle drei Verben gleichzeitig
['--list', '--name', 'X']                -> AKZEPTIERT — --name bei list ist sinnlos
[]                                       -> AKZEPTIERT — nichts gewählt, alles leer
```

Drei Unsinnsfälle, die du **selbst** im Code abfangen musst.

**B — `add_mutually_exclusive_group(required=True)`** löst zwei davon:

```
--create --revoke 3  -> SystemExit(2)  ✅ "not allowed with argument --create"
(nichts)             -> SystemExit(2)  ✅ "one of the arguments ... is required"
--list --name X      -> AKZEPTIERT     ❌ nicht ausdrückbar
```

Eine Gruppe kennt nur „höchstens eines von diesen" — nicht „dieses Argument gehört zu jenem".

**C — Unterbefehle** lehnen alle vier Fälle ab, mit **null Zeilen Prüfcode**:

```
revoke --name X   -> SystemExit(2)   create ohne --name -> SystemExit(2)
list --name X     -> SystemExit(2)   kein Verb          -> SystemExit(2)
```

Weil `--name` gar nicht am Eltern-Parser hängt, *existiert* die Kombination `revoke --name X` nicht. Sie ist nicht „verboten", sie ist **undarstellbar** — dasselbe Prinzip wie eine Invariante im Konstruktor, die ein ungültiges Objekt nie entstehen lässt.

*Gegenpunkt: Bei **einem** Verb sind Unterbefehle reiner Überbau. Der Break-even liegt bei zwei Verben mit unterschiedlichen Argumenten.*

### `required=` bedeutet an drei Stellen etwas anderes

| Wo | Standard | Wirkung |
|---|---|---|
| **Option** (`--name`) | **optional** — „Option" ist wörtlich gemeint | `required=True` macht sie zur Pflicht |
| **Positionsargument** (`key_id`) | **immer Pflicht** | `required=` ist **verboten**: `TypeError: 'required' is an invalid argument for positionals` |
| **`add_subparsers`** | **optional** | `required=True` erzwingt die Wahl eines Verbs |

Bei Positionsargumenten ist die Pflicht logisch zwingend: Sie werden an ihrer *Stelle* erkannt — fehlt eines, verschiebt sich alles danach, „optional" wäre nicht auflösbar.

> [!warning] Bei `add_subparsers` ist der Standard ein historischer Unfall
> In Python 2 waren Unterbefehle Pflicht, beim Übergang zu Python 3 wurden sie versehentlich optional, `required=` kam erst in 3.7 nach. Ohne `required=True` liefert ein nackter Aufruf `Namespace(command=None)` — **kein Fehler, keine Hilfemeldung** — und ein `if args.command == …` verfehlt alle Zweige und landet still im `else`.

> [!warning] Ein globales Flag muss **vor** dem Verb stehen
> Hängt `-v` am Eltern-Parser, gilt:
> ```
> pricing-apikey -vv create --name X    ✅ verbose=2
> pricing-apikey create --name X -vv    ❌ unrecognized arguments: -vv
> ```
> Sobald delegiert wurde, parst der Unterparser, und der kennt `-v` nicht. Bei `git` ist es genauso (`git -C /pfad status`).
>
> **Der naive Fix macht es kaputt:** `-v` *zusätzlich* am Unterparser zu definieren, lässt das Kind beim Parsen seinen eigenen Default in den gemeinsamen Namespace schreiben — und damit den Wert des Eltern-Parsers überschreiben:
> ```
> ['create','--name','X','-vv'] -> v=2   ✅
> ['-vv','create','--name','X'] -> v=0   ❌ !!
> ```
> Wer beide Positionen erlauben will, braucht am Kind `default=argparse.SUPPRESS`.

> [!note] Was das für wiederverwendbare Helfer bedeutet
> Eine Hilfsfunktion, die den Parser **selbst erzeugt und `parse_args` aufruft**, ist nur so lange brauchbar, wie das Kommando keine weiteren Argumente hat — `parse_args` behandelt alles Unbekannte als Fehler. Sobald ein zweites Kommando eigene Argumente braucht, zerlegt man sie in *„Flag an einen fremden Parser hängen"* und *„Wert auswerten"*. Der Aufrufer besitzt dann den Parser, der Helfer nur noch sein Argument.
>
> Umgekehrt gehört die **Beschreibung eines konkreten Kommandos** (welche Verben, welche Argumente) *nicht* in das gemeinsame Modul: Sonst müsste `shared/cli.py` wissen, dass es API-Schlüssel gibt. Gemeinsame Module enthalten **Mechanismus**, die Aufrufer entscheiden die **Politik** — genau wie `logging_setup` die Handler liefert und jeder Entrypoint seine Level selbst wählt.

---

# Wann Flag, wann Umgebung, wann Datei?

Die eigentliche Entwurfsfrage. Eine Faustregel, die trägt:

> **Willst du es beim *nächsten Aufruf* anders → Flag.
> Ist es auf einer *anderen Maschine* anders → Umgebung.**

| Wert | Kanal | Warum |
|---|---|---|
| Ausführlichkeit der Ausgabe | **Flag** (`-vv`) | ändert sich von Aufruf zu Aufruf |
| Probelauf ohne Schreiben | **Flag** (`--dry-run`) | dito |
| Datenbank-Passwort | **Umgebung** | Geheimnis, pro Deployment verschieden |
| Datenbank-Host | **Umgebung** | pro Deployment verschieden |
| Pfad der Logdatei | **Umgebung** | Betriebsentscheidung |
| Rundungsregel im Pricing | **keins von beidem** | Das ist Fachlogik, sie gehört in den Code |

Die letzte Zeile ist die wichtigste: Nicht alles, was sich ändern *könnte*, gehört in die Konfiguration. Jede Stellschraube ist ein Wert, der später falsch gesetzt sein kann.

---

# Die Rangfolge

Wenn alle Kanäle zusammenwirken, gilt üblicherweise (stark → schwach):

```
1. Kommandozeilen-Flag        pricing -vv
2. Echte Umgebungsvariable    export LOG_LEVEL=DEBUG
3. .env-Datei                 LOG_LEVEL=DEBUG
4. Default im Code            log_level: str = "INFO"
```

`pydantic-settings` bringt die Stufen 2–4 mit (plus Argumente an `Settings(...)` ganz oben, praktisch für Tests). Stufe 1 verdrahtet man selbst — in der Pricing-Anwendung: `parse_verbosity()` liefert das Konsolen-Level, `settings.log_level` das für die Datei.

Dass **echte Variablen die `.env` schlagen**, ist praktisch: In Docker oder CI setzt man Variablen, und eine versehentlich mitkopierte `.env` stört nicht.

---

# Sicherheit

- **Geheimnisse niemals als Kommandozeilenargument.** `ps aux` zeigt die vollständige Befehlszeile *aller* Prozesse — auch anderen Benutzern. Außerdem landet sie in der Shell-Historie. Passwörter gehören in die Umgebung, nicht in `argv`.
- **`.env` in `.gitignore`** — und zwar von Anfang an. Ein einmal committetes Passwort bleibt in der Git-Historie, auch wenn du die Datei später löschst.
- **Nicht loggen.** Ein `logger.debug("Settings: %s", settings)` schreibt dir das Passwort ins Logfile. Pydantic bietet dafür `SecretStr`: Der Wert wird beim Ausgeben als `**********` dargestellt und erst über `.get_secret_value()` sichtbar.
- **In Produktion keine `.env`.** Dort setzt man echte Umgebungsvariablen oder benutzt einen Secret-Manager. Die `.env` ist ein Werkzeug für die Entwicklung.

---

# Spickzettel

| Ich will … | Werkzeug |
|---|---|
| Pflicht-Umgebungsvariable lesen | `os.environ["NAME"]` |
| Optionale mit Rückfall | `os.getenv("NAME", "default")` |
| `.env` in die Umgebung laden | `load_dotenv()` — **im Entrypoint** |
| `.env` soll Bestehendes überschreiben | `load_dotenv(override=True)` |
| Typisierte, geprüfte Konfiguration | `class Settings(BaseSettings)` |
| Anderer Variablenname als Feldname | `Field(validation_alias="ANDERER_NAME")` |
| Einheitliches Präfix | `SettingsConfigDict(env_prefix="PRICING_")` |
| Geheimnis nicht versehentlich ausgeben | `SecretStr` |
| Schalter ohne Wert | `add_argument("--dry-run", action="store_true")` |
| Mehrfach steigerbar | `add_argument("-v", action="count", default=0)` |
| Nur bestimmte Werte erlauben | `add_argument("--modus", choices=[...])` |
| Entrypoint testbar halten | `main(argv=None)` + `parse_args(argv)` |
