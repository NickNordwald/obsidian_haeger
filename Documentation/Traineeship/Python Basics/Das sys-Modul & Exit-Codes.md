---
tags:
  - documentation
---
Was das `sys`-Modul ist, wofür es (im Unterschied zu `os`) gedacht ist, und was `sys.exit(main())` beim `pricing`-Entrypoint tatsächlich macht — Anlass war Block 2 der [[Logging & Exceptions Plan & Syntax-Tutorial|Logging & Exceptions Plan & Syntax-Tutorial]]. Ergänzt die praktischen Notizen in [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md|Python-Lernnotizen]] und die Exit-Code-*Begründung* (warum Exit-Codes wichtig sind) in [[1.4 Logging & Exceptionhandling - Vertiefung#Wo im Programm loggt man Exceptions?|1.4 Vertiefung]].

# Was ist `sys` überhaupt?

`sys` ist das Modul, über das dein Code mit dem **laufenden Python-Interpreter selbst** redet — nicht mit dem Betriebssystem. Das ist die entscheidende Abgrenzung zu `os`:

| | `os` | `sys` |
|---|---|---|
| **Wofür** | Schnittstelle zum **Betriebssystem** | Schnittstelle zum **Python-Interpreter** |
| **Beispiele** | Dateien, Umgebungsvariablen, Prozesse starten | Kommandozeilen-Argumente, Exit-Code, Ein-/Ausgabe-Streams, Modul-Cache |
| **Typische Frage** | „Was gibt's im Dateisystem/in der Umgebung?" | „Wie wurde *dieses* Programm gestartet, und wie soll es enden?" |

In eurem Projekt kennt ihr `os` schon aus `os.getenv("LOG_FILE")` (Umgebungsvariable abfragen — eine OS-Angelegenheit) und `os.environ["DB_PRICING_PW"]`, ausführlich in [[Documentation/Traineeship/Python Basics/Das os-Modul.md|Das os-Modul]]. `sys` kommt jetzt neu dazu, für die andere Sorte Frage: „Wie beendet sich dieser Prozess?"

## Die für dieses Projekt relevanten Mitglieder

- **`sys.exit(code)`** — beendet den Prozess mit einem bestimmten Exit-Code. Dazu gleich mehr.
- **`sys.stderr` / `sys.stdout` / `sys.stdin`** — die drei Standard-Streams als Datei-Objekte. Ihr habt das schon benutzt, ohne `sys` zu importieren: `"stream": "ext://sys.stderr"` in `logging_setup.py` (Block 1) ist genau dieses Objekt — `dictConfig` löst den `ext://`-Pfad zu `sys.stderr` auf.
- **`sys.argv`** — die Kommandozeilen-Argumente als Liste (`sys.argv[0]` ist der Programmname selbst). Braucht ihr aktuell nicht, taucht aber überall auf, wo CLI-Argumente verarbeitet werden.
- **`sys.path`** — die Liste der Verzeichnisse, in denen `import` nach Modulen sucht. Relevant, wenn ein Import mal „nicht gefunden" wird, obwohl die Datei da ist.
- **`sys.modules`** — Cache aller bereits importierten Module (`{"pricing.domain": <Modul-Objekt>, ...}`). Der Grund, warum ein zweites `import pricing.domain` nicht das Modul neu ausführt, sondern den Cache-Eintrag zurückgibt.

# `sys.exit()` im Detail

## Es ist keine Funktion, die „einfach beendet" — es wirft eine Exception

Das ist der Kernpunkt, den man auf den ersten Blick nicht erwartet: `sys.exit(code)` **ruft nicht direkt eine Betriebssystem-Funktion auf**, sondern wirft intern `raise SystemExit(code)`. Das bedeutet:

- **`finally`-Blöcke und `with`-Context-Manager laufen noch durch**, bevor der Prozess wirklich endet — genau wie bei jeder anderen Exception. Eine offene Datenbank-Verbindung in einem `with connect() as conn:` wird also sauber geschlossen, auch wenn mittendrin `sys.exit()` aufgerufen wird.
- **Sie kann grundsätzlich gefangen werden** — mit `except SystemExit:` oder einem nackten `except:`. Macht man aber praktisch nie absichtlich.
- Ganz oben (im Interpreter selbst, wenn dein Skript als `__main__` läuft) wird `SystemExit` speziell behandelt: Der Interpreter fängt sie, liest den mitgegebenen Code aus und beendet den **Prozess** wirklich damit.

## Warum `except Exception:` sie *nicht* verschluckt

`SystemExit` erbt **nicht** von `Exception`, sondern direkt von `BaseException` — zusammen mit `KeyboardInterrupt` (Strg+C) und `GeneratorExit`:

```
BaseException
├── SystemExit             ← sys.exit()
├── KeyboardInterrupt       ← Strg+C
├── GeneratorExit
└── Exception               ← ValueError, KeyError, PricingError, … (fast alles andere)
```

Das ist Absicht in der Sprache selbst — und genau der Grund, warum euer breiter Fang in Block 7

```python
except Exception:                        # der eine legitime breite Fang
    logger.exception("Unerwarteter Fehler im Preislauf")
    return 2
```

**sicher** ist: Selbst wenn irgendwo tief im Code jemand `sys.exit(...)` aufrufen würde, liefe dieser `except Exception:`-Zweig nicht dazwischen und würde ihn nicht in einen Exit-Code 2 verwandeln. Ein zu weiter Fang wäre `except BaseException:` — der würde auch ein sauberes `sys.exit()` oder ein Strg+C abfangen und wäre damit die Art von Fehler, vor der die `TRY`-Regelgruppe (Block 0) und die allgemeine Regel „kein `except:` ohne Typ" euch schützen sollen.

## Argument-Semantik: was aus `code` wird

| Argument | Resultierender Exit-Code | Beispiel |
|---|---|---|
| kein Argument / `None` | `0` (Erfolg) | `sys.exit()` |
| `int` | genau dieser Wert (0–255, wird auf POSIX-Systemen modulo 256 gekappt) | `sys.exit(1)`, `sys.exit(2)` |
| irgendein anderes Objekt (z. B. `str`) | wird nach **stderr** ausgegeben, Exit-Code wird `1` | `sys.exit("Fehler: Datei fehlt")` |

Das erklärt, warum `main() -> int` in eurem Fall sauber `0`/`1`/`2` zurückgibt — genau die `int`-Zeile aus der Tabelle, kein Zufallswert.

## Warum `sys.exit(main())` und nicht nur `main()`?

Der Punkt, an dem der Groschen meistens fällt: Würdet ihr im `if __name__ == "__main__":`-Block nur

```python
if __name__ == "__main__":
    main()          # ← Rückgabewert wird einfach verworfen
```

schreiben, würde `main()` zwar ausgeführt und `return 1`/`return 2` *innerhalb von Python* zurückgegeben — aber niemand liest diesen Rückgabewert aus, und das Skript beendet sich trotzdem ganz normal mit Exit-Code `0` (der Python-Default, wenn ein Programm ohne unbehandelte Exception durchläuft). Draußen in der Shell würde `echo $?` also **immer** `0` zeigen, komplett unabhängig davon, ob euer Preislauf erfolgreich war oder nicht — genau das Szenario aus eurer Tutorial-Warnung: „ein geloggter Fehler mit Exit-Code 0 ist für cron/Airflow/CI ein *erfolgreicher* Lauf".

`sys.exit(main())` schließt genau diese Lücke: Der `int`-Rückgabewert von `main()` wird als Argument direkt an `sys.exit()` durchgereicht und dadurch zum **tatsächlichen Prozess-Exit-Code**, den `$?` in der Shell sieht.

## `sys.exit()` vs. `exit()`/`quit()`

In einer interaktiven Python-Shell funktionieren auch `exit()` und `quit()` (ganz ohne Import). Die sind aber **nicht** für Skripte gedacht — sie werden vom `site`-Modul nur für den interaktiven Gebrauch eingerichtet und existieren nicht garantiert (z. B. fehlen sie, wenn Python mit `-S` gestartet wird). In echtem Anwendungscode gehört ausschließlich `sys.exit()` hin — kurz, garantiert vorhanden, genau dafür gemacht.

# Zusammenfassung

- `sys` = Schnittstelle zum **Interpreter selbst** (Gegenstück zu `os` = Schnittstelle zum Betriebssystem).
- `sys.exit(code)` wirft `SystemExit(code)` — eine echte Exception, kein Hard-Stop; `finally`/`with` laufen noch durch.
- `SystemExit` erbt von `BaseException`, nicht `Exception` — ein `except Exception:` fängt sie nie versehentlich ab.
- `main() -> int` + `sys.exit(main())` ist das Muster, das den Python-internen Rückgabewert zum echten, von der Shell sichtbaren Exit-Code macht — ohne das `sys.exit()` bleibt der Prozess-Exit-Code immer `0`.
