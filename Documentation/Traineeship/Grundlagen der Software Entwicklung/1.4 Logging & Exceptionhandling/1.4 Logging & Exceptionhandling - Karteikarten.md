---
deck: "Traineeship::Grundlagen der Software Entwicklung::Logging & Exceptionhandling"
---

Karteikarten zu [[1.4 Logging & Exceptionhandling - Vertiefung|1.4 Logging & Exceptionhandling – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::Logging & Exceptionhandling`. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

**Konvention:** max. **10 Karten pro Stichwort**, nur Basic-Karten (keine Cloze). Details, die hier nicht auf eine Karte passen, stehen in der Vertiefung.

# Fortschritt

- [x] Loggen von Exceptions
- [x] Parametrisierte Logs
- [x] Logging in Theorie und Praxis (Context Python)
- [x] Logging-Level
- [x] Logging-Frameworks
- [x] Exceptionhandling best practices

---

# Loggen von Exceptions

<!-- basicblock-start oid="ObsP8acPqNeVFYC5r9CCJZXC" -->
Aus welchen drei Teilen besteht die Information einer Exception (Pythons `exc_info`)?::
1. **Typ** (z. B. `KeyError`) — was für ein Fehler.<br>2. **Wert/Nachricht** (z. B. `'DB_PRICING_PW'`) — welcher konkrete Fall.<br>3. **Traceback** — wo und über welche Aufrufkette. Dazu kommen die Verweise `__cause__` (explizit via `raise ... from e`) und `__context__` (implizit).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRvQqp0XNE30xWnS29byYp" -->
Warum ist `logger.error(str(e))` bzw. `logger.error(f"Fehler: {e}")` ein Anti-Muster?::
Weil `str(e)` nur die Nachricht liefert — **Typ und Traceback gehen verloren**. Bei einem `KeyError` bleibt sogar nur der nackte Schlüsselname übrig (`'DB_PW'`), ohne jeden Hinweis auf Fehlerart oder Ort. Richtig ist `logger.exception("...")`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOhvCJkzpEgtl0syfGC1eC" -->
Was macht `logger.exception("...")` genau — und wo darf es *nicht* stehen?::
Es loggt auf Level **ERROR** und hängt automatisch den **kompletten Traceback** der gerade behandelten Exception an; äquivalent zu `logger.error("...", exc_info=True)`. Es gehört ausschließlich in einen `except`-Block — sonst gibt es keine aktive Exception und im Log steht `NoneType: None`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obszxo8zB6o91eRmUJvWVZBx" -->
Warum ist `except: pass` das teuerste Anti-Muster beim Exceptionhandling?::
Der Fehler verschwindet spurlos (kein Log, kein Traceback), das Programm läuft mit falschen/fehlenden Daten weiter und die Ursache ist später nicht mehr rekonstruierbar. Das **bare** `except:` ohne Typ fängt zusätzlich `KeyboardInterrupt` und `SystemExit` — dann lässt sich das Programm nicht einmal mehr mit Strg+C beenden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyVfbmCpSC6dzsIymo8fY4" -->
Was ist „log and rethrow" und was ist die Faustregel dagegen?::
Jede Schicht loggt denselben Fehler und wirft ihn weiter → ein Vorfall steht mehrfach fast identisch im Log, Monitoring-Alarme zählen ihn doppelt. Faustregel: **Log *oder* raise, nicht beides.** Geloggt wird dort, wo der Fehler **behandelt** wird (wo eine Entscheidung fällt). Nur Kontext ergänzen geht ohne zweiten Logeintrag via `e.add_note(...)` (3.11+).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEz3EKleUAfNXZ4Bk8uAEy" -->
Warum ist ein breites `except Exception` in normalen Modulen problematisch — und wo ist es trotzdem richtig?::
Es fängt auch **Programmierfehler** (`AttributeError`, `TypeError`), die man nicht behandeln kann, und degradiert echte Bugs zu Betriebsmeldungen. Man fängt so eng wie möglich. Legitim ist der breite Fang an genau **einer** Stelle: am äußersten Rand (`main`), damit kein Fehler unbemerkt hinausfällt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnxxbZZKX4OtCsx4J96mWm" -->
Warum ist ein Logeintrag allein noch keine Fehlerbehandlung?::
Weil danach eine **Entscheidung** folgen muss: Fallback, Überspringen (mit gezähltem Ergebnis), Abbruch oder **Exit-Code ≠ 0**. Für einen Scheduler (cron, Airflow, CI) ist ein geloggter Fehler mit Exit-Code 0 ein *erfolgreicher* Lauf — daher `sys.exit(main())`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBonkPGjHrsEx8v4lwDZhz" -->
Welchen Unterschied machen `raise Neu() from e`, `raise Neu()` und `raise Neu() from None` im Traceback?::
`from e` setzt `__cause__` → *„The above exception was the **direct cause** …"*. Ohne `from` setzt Python implizit `__context__` → *„During handling of the above exception, **another exception occurred**"*. `from None` **unterdrückt** die Vorgeschichte — die einzige Form, die Information bewusst verwirft (nur an API-Grenzen sinnvoll).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs44wFPMoWD9CYoKaMhqttV" -->
Nach welcher Regel entscheidet man, **wo** in der Anwendung eine Exception geloggt wird?::
**Fangen und loggen, wo man entscheiden kann — weiterwerfen, wo man es nicht kann.** Kern/Domäne wirft (kennt weder Nutzer noch Log-Ziel), Adapter übersetzen technische in sprechende Fehler (`raise ... from e`), der Rand/Entrypoint (`main`) fängt, loggt mit `logger.exception()` und setzt den Exit-Code.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszXoB9VzvhmBBT1lxz3GUm" -->
Wie prüft man in einem PyTest-Test, dass eine Exception wirklich **mit Traceback** geloggt wurde?::
Über `caplog`: `record = caplog.records[0]`, dann `assert record.exc_info is not None`. Genau diese Assertion fängt das Anti-Muster `logger.error(str(e))` ab — dort wäre `exc_info` nämlich `None`. Die Übersetzung prüft man ergänzend mit `pytest.raises(...)` + `assert isinstance(exc_info.value.__cause__, KeyError)`.
<!-- basicblock-end -->

---

# Parametrisierte Logs

<!-- basicblock-start oid="ObspVDWvSqSIRgdN8gbYz2r3" -->
Was ist ein „parametrisiertes" Log und wie unterscheidet es sich syntaktisch vom f-String-Log?::
`logger.info("%d Zeilen aus %s geladen", n, datei)` — Template und Werte werden **getrennt** übergeben (Komma statt Formatierung). Der f-String `logger.info(f"{n} Zeilen aus {datei}")` baut den fertigen Satz **vor** dem Aufruf und übergibt eine einzige Zeichenkette.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1Cm9B2iCcrHSOcUURQCbk" -->
Wann und wo wird die Meldung eines parametrisierten Logs tatsächlich zusammengesetzt?::
Erst im Handler über `record.getMessage()` (`msg % args`) — also **nur, wenn der Record wirklich ausgegeben wird** (Level passt, Handler zuständig). Deshalb kostet ein unterdrücktes `logger.debug("%s", obj)` die String-Konvertierung nicht: `__str__` wird gar nicht aufgerufen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrErfb4DUAQnBwTmZMtOAA" -->
Wie groß ist der Performance-Vorteil parametrisierten Loggings wirklich?::
Er hängt an den Werten: Bei billigen Werten (`int`, kurzer `str`) ~**1,3×** — vernachlässigbar. Bei teurem `__str__` (Dict-Dump, DataFrame, großes `repr`) ~**460×** (gemessen an 200 000 unterdrückten `debug`-Aufrufen). Performance allein ist deshalb das *schwächste* Argument; man gewöhnt sich das Muster an, weil man nicht weiß, welche Logzeile später in einer heißen Schleife steht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg2vuxPAgsBDnXpBgGipY1" -->
Was ist an einem parametrisierten Log **nicht** lazy — und was folgt daraus?::
Die **Argumente selbst** werden immer ausgewertet: `logger.debug("Ergebnis: %s", teure_berechnung())` ruft die Funktion in jedem Fall auf. Lazy ist nur die String-Konvertierung. Ist schon das Beschaffen des Wertes teuer, braucht es `if logger.isEnabledFor(logging.DEBUG):` davor — die einzige Situation, in der dieser `if` gerechtfertigt ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdRz2lRnnsRqGmDWYbrLEy" -->
Was passiert bei einem Formatfehler in einem parametrisierten Log (z. B. `logger.info("%d Zeilen", "abc")`)?::
Das `logging`-Modul fängt ihn ab, schreibt `--- Logging error ---` samt Traceback, Message und Arguments nach **stderr** — und das Programm **läuft weiter**. Richtig so: Logging ist Beobachtung, keine Fachlogik, und darf nie der Grund für einen Abbruch sein. Ein f-String-Fehler würde dagegen als normale Exception mitten im Anwendungscode zuschlagen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1nG0omIa4W9bmlmCYKAgZ" -->
Welche zwei `LogRecord`-Attribute bleiben nur bei parametrisiertem Logging getrennt — und warum ist das der wichtigste Grund?::
`record.msg` (das Template, z. B. `"%d Zeilen aus %s geladen"`) und `record.args` (`(5, "prices.csv")`); `getMessage()` liefert den gerenderten Text. `msg` ist damit eine **stabile Kennung des Ereignistyps**: Grundlage für Aggregation („wie oft trat das auf?"), Fehler-Gruppierung in Monitoring-Tools (Sentry-Fingerprint), JSON-/strukturiertes Logging und zuverlässiges `grep`. Mit f-Strings ist jede Zeile ein Unikat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUhVP3wNUiT0k0Ym0uHRWo" -->
Merksatz: Was ist ein parametrisiertes Log im Gegensatz zu einem f-String-Log?::
Ein parametrisiertes Log ist eine **Ereignisklasse mit Daten**, ein f-String-Log ein **fertiger Satz**. Sätze kann man lesen, Ereignisklassen kann man **zählen** — und als Data Engineer sitzt man oft auf der auswertenden Seite fremder Logs.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsa0TjdGIdkYkODWpU3RkVw" -->
Funktioniert `logger.info("{} Zeilen", 5)`, wenn der Formatter mit `style="{"` konfiguriert ist?::
**Nein** — „Logging error: not all arguments converted". Message-Argumente werden **immer** mit dem `%`-Operator zusammengesetzt. Der `style="{"`-Parameter betrifft nur das **Format-Template des Formatters** (Anordnung von Level, Zeitstempel, Message), nicht die Meldung selbst. Häufige Verwechslung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKlB6KGhW3lr161s1ZddTD" -->
Wofür ist `extra=` beim Loggen da und welche zwei Fallstricke hat es?::
`logger.info("Empfehlung geschrieben", extra={"unit_id": 42})` hängt Werte als **Attribute an den `LogRecord`**, ohne den Meldungstext zu ändern — Einstieg ins strukturierte Logging (`%(unit_id)s` im Formatter, eigene JSON-Felder). Fallstricke: (1) reservierte Namen (`message`, `name`, `asctime`, …) lösen `KeyError: "Attempt to overwrite 'message' in LogRecord"` aus → Präfix benutzen; (2) steht das Feld im Formatter, muss **jeder** Record es haben, sonst scheitern alle anderen Logzeilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZY0wJ01Ud9XJTvwHoMlG2" -->
Welche ruff-Regeln erzwingen parametrisiertes Logging?::
Die Gruppe **`G`** (`flake8-logging-format`): `G001` = `.format()`, `G002` = `%`-Operator, `G003` = `+`-Verkettung, **`G004` = f-String** („Convert to lazy `%` formatting"). Dazu passt `TRY401` — „Redundant exception object included in `logging.exception` call". Was ein Werkzeug prüfen kann, soll kein Mensch im Review prüfen müssen.
<!-- basicblock-end -->

---

# Logging in Theorie und Praxis (Context Python)

<!-- basicblock-start oid="Obs3MHme1GcRoJeMENhzVRga" -->
Aus welchen vier Bausteinen besteht Pythons Logging-Pipeline und wofür ist jeder zuständig?::
**Logger** (nimmt die Meldung an, kennt Herkunft über `__name__` und Level) → **Filter** (ob durchlassen, jenseits des Levels) → **Handler** (wohin: Konsole, Datei, …) → **Formatter** (wie die Zeile aussieht). Kernpunkt: Der **Formatter hängt am Handler**, nicht am Logger — deshalb kann dasselbe Log gleichzeitig knapp auf die Konsole und ausführlich in die Datei gehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmTZYuiRIdD2O8iAjfaml9" -->
Warum ist die Trennung von Logger und Handler der eigentliche Grund für Logging-Frameworks?::
**Der loggende Code weiß nicht, wohin geloggt wird.** `builder.py` sagt nur „das ist passiert"; das Ziel bestimmt die Anwendung an ihrem Rand. Dadurch läuft dasselbe Modul unverändert im CLI-Skript, im Airflow-Task und im Test. Genau das kann `print` nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs071g0PvldAwsNfjZNBIsj" -->
An welchen **zwei** Stellen wird beim Logging ein Level geprüft — und wie teilt man die Aufgaben auf?::
(1) Am **Logger**: `level >= logger.getEffectiveLevel()`? Sonst entsteht nicht einmal ein `LogRecord`. (2) An **jedem Handler**, der ein eigenes Level hat. Faustregel: **Logger-Level = „was interessiert mich überhaupt", Handler-Level = „was schreibe ich in dieses Ziel"** — z. B. Logger auf `DEBUG`, Konsole auf `INFO`, Datei auf `DEBUG`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIlayQcTzNsWvojfOecXVb" -->
Was bringt die Logger-Hierarchie und warum schreibt man `logging.getLogger(__name__)`?::
Logger bilden über Punkt-Namen einen Baum (`pricing.core.builder` → `pricing.core` → `pricing` → Root `""`). Mit `__name__` **ist der Modulpfad die Hierarchie** — geschenkt. Nutzen: Ein Logger ohne eigenes Level steht auf `NOTSET` (0) und erbt via `getEffectiveLevel()` vom nächsten Vorfahren. So steuert man ganze Teilbäume mit einer Zeile — z. B. `sqlalchemy.engine` auf `WARNING` drehen, ohne fremden Code anzufassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstBe0jQeRemgRzwU0RCCta" -->
Root steht auf `ERROR`, der Logger `app` auf `DEBUG` — wird `logging.getLogger("app").debug(...)` ausgegeben?::
**Ja.** Bei der Weitergabe nach oben werden nur die **Handler** der Vorfahren benutzt, deren **Level wird ignoriert**. Entscheidend sind allein das effective level des *aufrufenden* Loggers und danach das Level der einzelnen Handler. Wer die Ausgabe drosseln will, setzt das Level **am Handler**, nicht am Root-Logger.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVSoLbh9LFU86UXcHw8QUf" -->
Wodurch entsteht eine **doppelte** Logausgabe und was sind die zwei sauberen Lösungen?::
Ein Modul-Logger bekommt einen eigenen Handler, obwohl der Root schon einen hat — wegen `propagate = True` läuft der Record durch beide. Lösungen: (1) **Normalfall** — Handler nur an *einer* Stelle (Root) registrieren, Modul-Logger bekommen nie welche. (2) **Sonderfall** (eigenes Ziel, z. B. Audit-Datei) — eigener Handler **plus** `propagate = False`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBg1OCGT07LpxlJLQA1Cui" -->
Warum kann `propagate = False` einen `caplog`-Test rot machen?::
pytests `caplog` hängt seinen Handler an den **Root**-Logger. Bricht man die Propagation darüber ab, erreicht der Record den Root nie → `caplog.records` ist leer und der Test schlägt fehl, obwohl die Anwendung korrekt loggt. Abhilfe: `caplog.set_level(..., logger="pricing")` bzw. direkt am betroffenen Logger ansetzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBCV2N4iOjEyi4JFoBbMd5" -->
Wer darf das Logging konfigurieren — und wer auf keinen Fall?::
**Nur der Rand der Anwendung** (`main.py`, Airflow-Task, `conftest.py`) legt Handler, Formatter und Level fest, und zwar **einmal**. **Fach- und Bibliotheksmodule konfigurieren nie** — sie holen sich nur `logging.getLogger(__name__)` und loggen. Grund: Wohin Logs gehen, entscheidet der Betreiber der Anwendung, nicht der Autor eines Moduls.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQufKmtzoTrgtDA4cjW602" -->
Was ist ein `NullHandler` und was ist Pythons `lastResort`?::
`logging.getLogger(__name__).addHandler(logging.NullHandler())` in der `__init__.py` einer **Bibliothek** verhindert Ausgaben, solange die Anwendung nichts konfiguriert hat. Ohne jede Konfiguration greift sonst `lastResort` = `<_StderrHandler <stderr> (WARNING)>`: schreibt `WARNING`+ unformatiert nach stderr und **verschluckt `INFO`/`DEBUG` still**. Wer „nur Warnungen, keine Infos" sieht, hat meist gar keine Konfiguration.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsynkin1vnS8Z6ZtaW0vUSJ" -->
Wann tut `logging.basicConfig(...)` **gar nichts** — und wie erzwingt man die Konfiguration?::
Wenn der Root-Logger **bereits Handler hat**. Kein Fehler, keine Warnung — der zweite Aufruf ist samt `level` und `format` wirkungslos. Deshalb „gewinnt" ein `basicConfig` in einem früh importierten Modul stillschweigend. Abhilfe seit Python 3.8: `logging.basicConfig(..., force=True)` entfernt bestehende Handler und setzt neu auf.
<!-- basicblock-end -->

---

# Logging-Level

<!-- basicblock-start oid="ObsceIJp0B7tZdUB6vwdCDUN" -->
Welche numerischen Werte haben die Logging-Level in Python, und wofür steht `NOTSET`?::
`NOTSET` 0 · `DEBUG` 10 · `INFO` 20 · `WARNING` 30 · `ERROR` 40 · `CRITICAL` 50. Verglichen wird **numerisch**, eine Schwelle meint immer „dieses Level und alles darüber". `NOTSET` (0) heißt „nicht gesetzt" → der Logger erbt das Level vom nächsten Vorfahren mit gesetztem Level.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3TtGldZO3tHSiJhEBgin5" -->
Welche Frage stellt man sich bei der Level-Wahl — statt „wie schlimm ist das?"::
**„Wer liest diese Zeile, und was tut die Person daraufhin?"** `DEBUG` = Entwickler bei konkretem Problem · `INFO` = Betrieb, nichts zu tun · `WARNING` = beobachten, kein Alarm · `ERROR` = jemand muss zeitnah ran · `CRITICAL` = jemand wird geweckt. Level als *Gefühlsskala* zu lesen führt dazu, dass alles auf `WARNING` steht und die Stufe nichts mehr aussagt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3DkZ3luCynbuWlf34YiG2" -->
Nach welchen zwei Faustregeln trennst du `WARNING`/`ERROR` und `INFO`/`DEBUG`?::
**`WARNING` vs. `ERROR`:** Konnte die Anwendung ihre Aufgabe trotzdem erledigen? Ja → `WARNING`, nein → `ERROR`. Merksatz: `ERROR` heißt, dass etwas **nicht getan wurde**, das getan werden sollte — ein erfolgreicher Retry oder ein greifender Fallback ist deshalb kein `ERROR`. **`INFO` vs. `DEBUG`:** Willst du die Zeile auch dauerhaft in Produktion sehen, wenn alles glattgeht? Ja → `INFO`, nur bei der Fehlersuche → `DEBUG`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsINRziMPG67VkZXYCW6fwT" -->
Warum steht der Root-Logger standardmäßig auf `WARNING`?::
Weil eine Bibliothek **ohne Konfiguration nur melden soll, was den Nutzer wirklich angeht** — ungefragtes `INFO` auf der Konsole gilt als schlecht erzogen. (Auch der `lastResort`-Handler steht auf `WARNING`.) Die eigene *Anwendung* dreht das im Entrypoint bewusst auf `INFO`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAY4xPgVLsKt5xgJBIDcJy" -->
Nenne vier Vorteile abgestufter Logging-Level, die über „man sieht weniger" hinausgehen.::
1. **Ein Code, viele Umgebungen** — lokal `DEBUG`, produktiv `INFO`, ohne Code-Änderung. 2. **Nachträgliche Vertiefung** im Fehlerfall (Level hochdrehen statt neu deployen) — der eigentliche Grund, `DEBUG`-Zeilen überhaupt zu schreiben. 3. **Granularität pro Modul** über die Hierarchie (auch gegen gesprächige Fremdbibliotheken). 4. **Mehrere Ziele mit eigener Schwelle** (Konsole `INFO`, Datei `DEBUG`). Dazu: Alarmierbarkeit und Kostensteuerung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDbxKMyESHZaL8G8KyU7b9" -->
Was ist der **teuerste** Nachteil von zu niederschwelligem Logging?::
Das **Signal-Rausch-Verhältnis** — teurer als Speicherplatz. Bei 10 000 `DEBUG`-Zeilen/Minute findet niemand die eine `ERROR`-Zeile, und schlimmer: Man *gewöhnt* sich an ein volles Log, Auffälligkeiten fallen nicht mehr auf. Ein Log, in dem alles steht, ist informationsärmer als eins, in dem das Wichtige steht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsG0mdzMhpPipxmaSDgUOfs" -->
Welche Nachteile hat niederschwelliges Logging außer dem Rauschen — und warum ist „lieber wenig loggen" trotzdem falsch?::
**Volumen/Kosten** (100 000 Zeilen/Tag à ~120 B ≈ 12 MB/Tag ≈ **4,4 GB/Jahr**; 10 Mio./Tag ≈ 438 GB/Jahr — Cloud-Logging kostet pro GB), **Laufzeit** (Formatieren + synchrones I/O in heißen Schleifen), **Betriebsrisiko** (volle Platte), **Datenschutz** (PII/Secrets im Log, z. B. Passwort in der DSN). Aber: Zu wenig Logging ist nur ein *leiserer* Fehler — er fällt nachts auf, wenn das Log nichts hergibt. Richtig ist nicht „wenig loggen", sondern **auf der richtigen Stufe**: großzügig `DEBUG`, sparsam `INFO`, sehr sparsam `WARNING`/`ERROR`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZXsvi7CHcY7d5slIEhdzz" -->
Wie ändert man das Logging-Level in Produktion, ohne den Code anzufassen?::
Über eine **Umgebungsvariable** (Standard für Dienste/Container): `logging.basicConfig(level=os.getenv("LOG_LEVEL", "INFO").upper())` — `basicConfig` akzeptiert den Levelnamen als String. Für CLI-Werkzeuge zusätzlich ein **Verbose-Flag** (`-v` → `DEBUG`). Erst dadurch wird der Vorteil „im Fehlerfall vertiefen" real.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssTGKy3brDm14QigBBia7b" -->
Kann man eigene Logging-Level definieren — und sollte man?::
Technisch ja: `TRACE = 5; logging.addLevelName(TRACE, "TRACE"); logger.log(TRACE, "...")`. Praktisch fast immer falsch: Fremdwerkzeuge, Formatter-Konfigurationen und Kollegen kennen das Level nicht, und `logger.trace()` gibt es nicht ohne zusätzliche Methode auf der Logger-Klasse. Was man eigentlich will, ist **Granularität pro Modul** — die gibt es über die Hierarchie gratis.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshCvkQosSONUX2qj2cwKBB" -->
Warum gehört alles, was *pro Datensatz* passiert, auf `DEBUG` — und was ist das bessere Muster für Datenqualitätsprüfungen?::
Weil das Log sonst **linear mit der Datenmenge wächst** (die einzige Level-Regel, die man nie brechen sollte; ausschlaggebend ist die **Frequenz**, nicht die Wichtigkeit). Besser: eine **Summe am Ende** statt einer Zeile pro Vorfall — `logger.warning("%d Einheiten mit negativer Kapazität übersprungen.", n)` erzeugt bei 50 000 Vorfällen **eine** Zeile; die Einzelfälle bekommt man auf `DEBUG`.
<!-- basicblock-end -->

---

# Logging-Frameworks

<!-- basicblock-start oid="ObsYapYbW3tVKjkPXCRKEISH" -->
Welches Problem löst ein Logging-Framework, das „Text in eine Datei schreiben" nicht löst?::
Ein **Standardisierungsproblem**, kein technisches. Jede Anwendung baut sich sonst ihr eigenes `log()`; sobald zwei Bibliotheken zusammenkommen, gibt es zwei Systeme, zwei Konfigurationen, zwei Formate. Ein Framework sorgt dafür, dass alle Beteiligten dieselbe Pipeline benutzen und der Betrieb **eine** Stelle zum Steuern hat. Deshalb hat Python `logging` in der Standardbibliothek.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7zZjXcpZWZ9GWiroXyvIq" -->
Nenne zwei der größten Logging-Frameworks (Kontrollfrage) und ordne sie zu.::
**Apache Log4j 2** und **Logback** — beide Java, meist über die Fassade **SLF4J** benutzt. Log4j hat die Begriffe geprägt, die überall gelten: Logger, **Appender** (= Pythons Handler), **Layout** (= Formatter), Level-Hierarchie; Pythons `logging` ist direkt daran angelehnt. Im Python-Umfeld: die Standardbibliothek **`logging`** plus **`structlog`** / **`loguru`**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsdo7q5zuW2hS5HtoyZXvTG" -->
Was ist SLF4J und warum ist eine Logging-**Fassade** wichtig?::
SLF4J ist selbst kein Framework, sondern eine **einheitliche API**, hinter der austauschbar Log4j 2, Logback o. a. steckt. Sinn: Eine *Bibliothek* programmiert gegen die Fassade und zwingt der *Anwendung* kein Framework auf — dasselbe Prinzip wie „Bibliotheken konfigurieren nicht, Anwendungen konfigurieren". In Python übernimmt `logging` beide Rollen (Fassade **und** Implementierung) gleichzeitig.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHE8AFCObHKt261QvUL7y2" -->
Was ist die Kernidee von `structlog` und was leistet `bind()`?::
Eine Logmeldung ist kein Satz, sondern ein **Dictionary**: `log.info("empfehlungen_geschrieben", anzahl=1500, dauer_ms=812)` → wahlweise farbige Konsole oder **JSON**. Also *Ereignisname plus Felder* statt *Satz mit angehängten Feldern* (`extra=`). `bind()` heftet Kontext an, der **mitwandert**: einmal `log = log.bind(run_id=...)`, und jede weitere Zeile trägt das Feld — die Korrelations-ID fällt als Nebenprodukt ab. Wichtig: `structlog` **ersetzt `logging` nicht**, es baut darauf auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEX3NqtC9t8ZeNGNSB2SWA" -->
Warum ist `loguru` für Anwendungen attraktiv, für Bibliotheken aber ungeeignet?::
Attraktiv: Nullkonfiguration (`from loguru import logger` und es läuft), `logger.add("app.log", rotation="10 MB", retention="7 days")` als Einzeiler, `@logger.catch`-Dekorator. Ungeeignet für Bibliotheken: Es arbeitet mit **einem einzigen globalen Logger-Objekt statt einer Hierarchie** — damit entfällt die Granularität pro Modul, und es zwingt der Anwendung die Logging-Entscheidung auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTdBzqjmKkR2Z5K62gQQNb" -->
Warum ist strukturiertes Logging (JSON) für einen Data Engineer relevant?::
Weil ein JSON-Log eine **Tabelle** ist: Man kann es laden und mit SQL abfragen, statt mit `grep` und Regex zu arbeiten. Logs werden heute nicht mehr gelesen, sondern **abgefragt** (ELK/OpenSearch, Loki, Splunk, CloudWatch). Das ist die Motivation des Leitfadens („Logging anderer Systeme ist eine Datenquelle") — nur von der Seite dessen, der die Logs *erzeugt*.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsu5Od82ikBMvCsrlyXz7uV" -->
Was war Log4Shell und welche allgemeine Regel folgt daraus?::
CVE-2021-44228 (Dezember 2021): Log4j 2 interpretierte in Logmeldungen Ausdrücke wie `${jndi:ldap://...}` und lud daraufhin Code aus dem Netz nach — es genügte, einen präparierten String irgendwo hinzuschicken, wo er **geloggt** wurde (z. B. als Benutzername). Lehren: (1) Logging-Bibliotheken sind sicherheitsrelevante Abhängigkeiten und gehören gepatcht; (2) **ungeprüfte Fremdeingabe darf nie in einen Teil geraten, der interpretiert wird** — dieselbe Regel wie „keine Fremddaten in den Format-String".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxEU3gmNzm1cKgRrlWi6T0" -->
Welche zwei Eigenschaften muss der eigene Code haben, damit der Wechsel auf JSON-Logging nur Konfiguration ist?::
1. **Template und Daten getrennt** (parametrisiertes Logging) — sonst gibt es keine Felder, nur Text. 2. **Konfiguration ausschließlich am Rand** (Entrypoint, `dictConfig`) — dann tauscht man den Formatter aus, statt Code umzubauen. Beides hatten wir schon, bevor das Thema JSON überhaupt auftauchte.
<!-- basicblock-end -->

---

# Exceptionhandling best practices

<!-- basicblock-start oid="ObsHDEhuuVjsmLiLmkQM39ZR" -->
Wofür stehen EAFP und LBYL, welches bevorzugt Python — und wann gilt das Gegenteil?::
**LBYL** = *Look Before You Leap* (erst prüfen, dann handeln), **EAFP** = *Easier to Ask Forgiveness than Permission* (handeln, Fehler abfangen). Python bevorzugt **EAFP**: (1) **keine Race Condition** — zwischen `if exists(p)` und `open(p)` kann die Datei verschwinden, die Prüfung garantiert nichts; (2) **Vollständigkeit** — eine Prüfung deckt ab, woran man gedacht hat, `except` deckt ab, was eintritt; (3) seit 3.11 kostet ein `try` ohne Exception fast nichts.<br>**Aber:** Gemessen (dict-Zugriff, 300 000×) ist EAFP bei vorhandenem Schlüssel ~3,5× schneller, bei fehlendem ~6,5× **langsamer**. Regel: **EAFP gewinnt, wenn der Fehler die Ausnahme ist** — tritt er regelmäßig ein, ist es keine Ausnahme, sondern eine Fallunterscheidung, also prüfen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEjmHlRasi9rJSz4gpw5q6" -->
Wozu dient der `else`-Block — und warum ist `return` im `finally` gefährlich?::
**`else`** läuft nur, wenn **keine** Exception auftrat. Zweck: den `try`-Block **klein** halten — alles im `try` ist potenziell vom `except` betroffen, die Weiterverarbeitung gehört also ins `else`. Das ist „so eng wie möglich fangen", angewandt auf den `try`-*Block* statt auf den Exception-*Typ*.<br>**`finally`** läuft immer — aber ein `return` (oder `break`/`continue`) darin **überschreibt eine laufende Exception** spurlos: `try: raise ValueError("wichtig!") finally: return "harmlos"` liefert `'harmlos'`, ohne Traceback. `except: pass` mit Tarnung; ruff meldet es als **`B012`**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvvJXpwfM0tONUSJAzsgfA" -->
Warum liegen `KeyboardInterrupt` und `SystemExit` **neben** `Exception` statt darunter?::
Weil sie keine Fehler sind, sondern **Abbruchwünsche**. Der Baum: `BaseException` → `KeyboardInterrupt`, `SystemExit`, `GeneratorExit` und daneben `Exception` (mit `ValueError`, `KeyError`, `OSError`, …). Deshalb sind `except Exception:` und `except:` **nicht** dasselbe — Letzteres macht das Programm unabbrechbar. Konsequenz: Eigene Exceptions immer von `Exception` ableiten, nie von `BaseException`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRN2RAoGYj5NitZCHf8cF4" -->
Wie baut man eine Exception-Hierarchie für ein Paket — und was gewinnt `main()` dadurch?::
**Eine Basisklasse pro Paket**, davon die konkreten Fälle abgeleitet: `class PricingError(Exception)` → `KonfigurationsFehler`, `QuelleFehltError`, `DatenqualitaetsFehler`. Gewinn am Rand: `except PricingError` fängt mit einer Zeile **alles Eigene** (= erwartete Fehler, Exit-Code 1) und lässt alles Fremde zum zweiten, breiten `except` durch (= Bugs, Exit-Code 2). Aufrufer können trotzdem feiner unterscheiden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfwDxe1QZIvP5xfiGP8qvR" -->
Warum gehören Daten als **Attribute** an eine Exception statt in die Nachricht?::
```python
class QuelleFehltError(PricingError):
    def __init__(self, pfad): super().__init__(f"nicht gefunden: {pfad}"); self.pfad = pfad
```
Zwei Vorteile: (1) Der `except`-Block kann **entscheiden** statt nur zu loggen (`if e.pfad.suffix == ".csv"`). (2) Tests prüfen `exc_info.value.pfad` statt Textbausteine zu vergleichen — Textvergleiche sind brüchig. Merksatz: **Die Nachricht ist für Menschen, die Attribute sind für Code.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYAB1OhQE78YrOjZZIDtZu" -->
Warum ist `assert` keine Fehlerbehandlung?::
Weil `python -O` alle `assert`-Anweisungen **ersatzlos entfernt**: `assert p > 0` in `preis_setzen(-5)` schlägt normal fehl, unter `-O` liefert die Funktion klaglos `-5`. `assert` ist für **Annahmen, die nie falsch sein dürfen**, und für Tests. Eingabeprüfung braucht ein echtes `raise ValueError(...)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYmGiFuAazg0JKTftvEpVo" -->
Was unterscheidet `contextlib.suppress(X)` von `except X: pass`?::
Technisch nichts — kommunikativ alles. `with suppress(FileNotFoundError): datei.unlink()` ist eine **Absichtserklärung**: Hier wurde entschieden, dass der Fehler egal ist. Beim `pass` weiß der nächste Leser nie, ob jemand nachgedacht oder nur einen Fehler weggeklickt hat. Bedingungen bleiben: **konkreter Typ**, nur um **eine** Anweisung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss8ZcvZFe7p12PHJNQkoI5" -->
Wofür sind `ExceptionGroup` und `except*` (ab Python 3.11) gedacht?::
Für Fälle, in denen **mehrere Fehler gleichzeitig** auftreten — klassisches `raise` transportiert nur einen. Typisch bei Stapelverarbeitung („500 Zeilen geprüft, 12 fehlerhaft") und nebenläufigen Tasks. `except* ValueError as eg` fängt **alle passenden** Teilfehler der Gruppe (`eg.exceptions`), und mehrere `except*`-Zweige können nacheinander greifen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZc4weUAvOF98n5LlLkOGd" -->
Warum muss man in Python dokumentieren, welche Exceptions eine Funktion wirft?::
Weil Python **keine geprüften Exceptions** kennt (kein `throws` wie Java) und auch mypy `strict` das nicht prüft — welche Fehler eine Funktion wirft, steht **nirgends im Typsystem**. Einziger Ersatz ist der Docstring (`Raises: KonfigurationsFehler: wenn ...`). Kein Formalismus: Der Aufrufer kann nur fangen, was er kennt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOKvI7Ns9rQrZbmoBE1zQd" -->
Welche vier Strategien gibt es bei **partiellem Scheitern** in einer Datenpipeline?::
1. **Fail fast** — ganzer Lauf bricht ab; wenn ein Teilergebnis schlimmer ist als keins (falsche Preise gehen live). 2. **Skip & count** — überspringen, Summe am Ende loggen; wenn Einzelausfälle normal sind. 3. **Skip mit Schwelle** — Abbruch ab X % Ausfall; der übliche Kompromiss. 4. **Dead-Letter** — Fehlerfälle beiseitelegen zur Nachbearbeitung. Unser Projekt macht heute implizit *Skip & count*; es fehlt die **Schwelle**, denn: auf das Loggen muss eine Entscheidung folgen.
<!-- basicblock-end -->
