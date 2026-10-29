---
tags:
  - documentation
type: notiz
date: 2026-07-29
---

Abgeschlossener Umsetzungsstand der **Logging- & Exceptionhandling-Iteration** der Pricing-Anwendung (Aufgabe: [[1.4 Logging & Exceptionhandling - Leitfaden#Aufgabe|1.4 Leitfaden – Aufgabe]]). Baut auf [[Umsetzungsstand (Testing mit PyTest)|Umsetzungsstand (Testing mit PyTest)]] auf.

Theorie: [[1.4 Logging & Exceptionhandling - Vertiefung|Vertiefung]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.4 Logging & Exceptionhandling/Anki Karteikarten|Anki Karteikarten]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.4 Logging & Exceptionhandling/Q&A/Weiterführende Stichpunkte.md|Q&A-Register]] · Plan: [[Logging & Exceptions Plan & Syntax-Tutorial|Logging & Exceptions Plan & Syntax-Tutorial]] (enthält das Logging-Design mit Begründung)

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Logging_Exceptions` (Kopie des PyTest-Standes) · Abschluss-Commit `logging erweiterung fertig`

Ziel war, die Anwendung so auszustatten, **als ginge sie morgen live**: Aus den Logs muss nachvollziehbar sein, *was* ein Lauf getan hat, *wie lange* er gebraucht hat und *warum* er ggf. fehlgeschlagen ist — ohne die Konsole zuzumüllen.

## Endstand

Zwei neue Module und ein umgebauter Entry Point:

| Datei | Inhalt |
|---|---|
| `logging_setup.py` | `setup_logging(console_level, file_level, log_file)` per `dictConfig` — Konsolen-Handler nach **stderr** (Default `WARNING`), optionaler `RotatingFileHandler` (1 MB, 5 Backups, nur bei gesetztem `log_file`), zwei Formatter (`knapp` / `ausfuehrlich` mit Zeitstempel, Logger-Name und Zeilennummer) |
| `exceptions.py` | Hierarchie `PricingError` → `ConfigError`, `DataSourceError`, `DomainError` |
| `main.py` | `setup_logging()` als erste Zeile, `try`/`except` am Rand mit gestaffelten Exit-Codes |
| `tests/test_exception.py` | Tests für die neuen Exceptions und Logpfade |

**55 Testfunktionen grün**, ruff (mit den neuen Gruppen `G`/`LOG`/`TRY`) und `mypy --strict` sauber. Log-Level-Verteilung im Endstand: 13× `info`, 9× `exception`, 6× `warning`, 6× `error`, 4× `debug`, 0× `critical`.

## Wichtigste Entscheidungen

**Konfiguration per `dictConfig` mit `disable_existing_loggers: False`.** Der `basicConfig`-Einzeiler wurde ersetzt, weil er weder Zeitstempel noch Logger-Namen noch mehrere Handler kann. Das `False` ist entscheidend: Sonst legt die Konfiguration alle bereits importierten Logger still — ein Fehler, der sich erst zeigt, wenn Bibliotheken plötzlich schweigen.

**Konsole nach `stderr`, Programm-Ausgabe nach `stdout`.** Der `print(f"Höchste Empfehlung: …")` in `main.py` blieb bewusst ein `print` und wurde **nicht** zu einem Logaufruf: Es ist das *Ergebnis* des Programms, keine Diagnose. Damit bleibt `python -m pricing.main > ergebnis.txt` sinnvoll, während Logs weiter sichtbar sind. Diese Trennung zahlt sich in 1.7 erneut aus (Container-Logging).

**Zwei Handler mit unterschiedlichem Level.** Konsole knapp (`WARNING`), Datei ausführlich (`INFO`, mit Zeitstempel und Herkunft). Der Gedanke: Wer zuschaut, will Ruhe; wer hinterher sucht, will alles.

**Datei-Handler nur bei gesetztem `log_file`.** Kein Standardpfad, keine Datei „aus Versehen". Das erwies sich in 1.7 als genau richtige Entscheidung — im Container bleibt `LOG_FILE` weg und übrig bleibt der stderr-Handler, den `docker logs` abgreift.

**Eigene Exception-Hierarchie mit `PricingError` als Basis.** Ein einziges `except PricingError` am Rand fängt alles Fachliche, während unerwartete Fehler durchfliegen und getrennt behandelt werden. Die Daten hängen als **Attribute** an der Exception, nicht nur als Text in der Nachricht (`DataSourceError(pattern, data_dir, treffer)`) — so kann der Aufrufer damit rechnen statt zu parsen.

**`class DomainError(PricingError, ValueError)` — bewusste Mehrfachvererbung.** Bestehender Code und bestehende Tests, die `except ValueError` bzw. `pytest.raises(ValueError)` benutzen, funktionieren unverändert weiter; neuer Code kann auf `PricingError` fangen. Standardmuster beim Nachrüsten einer Hierarchie in gewachsenen Code.

**Gestaffelte Exit-Codes am Rand.** `0` = Erfolg, `1` = fachlicher Fehler (`PricingError`), `2` = unerwarteter Fehler. Beide Zweige loggen per `logger.exception`, also **mit** Traceback. Damit ist die Anwendung skript- und cron-tauglich: Der Aufrufer kann zwischen „hat nicht funktioniert, aber wir wissen warum" und „etwas ist grundsätzlich kaputt" unterscheiden.

**ruff-Gruppen `G`, `LOG` und `TRY` aktiviert.** Macht aus „parametrisiertes Logging ist eine gute Idee" eine maschinell erzwungene Regel (`G004` verbietet f-Strings im Logaufruf). Dieselbe Idee wie `test_architektur` in 1.5 und `squawk` in 1.6: eine Regel, die nicht von Disziplin abhängt.

**`TRY003` für `domain.py` bewusst deaktiviert.** Die Regel verlangt, lange Meldungen aus dem `raise`-Aufruf in die Exception-Klasse zu verlagern. Für `DataSourceError` war das richtig und wurde umgesetzt; für reine Wertobjekte in `domain.py` wäre eine attributtragende Fehler-API Overhead ohne Nutzen. Als `per-file-ignores` dokumentiert und damit revidierbar — statt still ignoriert.

**Aggregiert loggen statt pro Zeile.** Die bestehenden `WARNING`s (negative Kapazität, Orphan-Buchungen) loggen **Summen statt Einzelfällen**. Bei Millionen Zeilen ist das der Unterschied zwischen einem lesbaren Log und einem unbrauchbaren. Diese Entscheidung stammt schon aus 1.1 und wurde hier bestätigt statt geändert.

## Gelernte Lektionen

- **`TRY003` hängt am `raise`-Aufruf, nicht am Exception-Typ.** Das Ersetzen von `ValueError` durch `DomainError` allein änderte nichts an der Meldung — die Regel sieht nur, ob am Aufrufort ein langer String steht. Erst das Verlagern der Nachricht in `DataSourceError.__init__()` löste sie auf.
- **`propagate = False` macht `caplog` blind.** Ein Logger-Setup, das die Weitergabe an den Root-Logger abschaltet, lässt bestehende `caplog`-Tests aus 1.3 stillschweigend leerlaufen — sie werden nicht rot, sie sehen nur nichts mehr. Entweder Propagation anlassen oder die Tests auf `caplog.set_level(..., logger="pricing")` umstellen.
- **Die Baseline war besser als erwartet.** Modul-Logger (`getLogger(__name__)`), parametrisiertes Logging und die Konfiguration im Entry Point standen bereits korrekt — aus 1.1 heraus, ohne dass es damals ein Thema war. Die eigentliche Lücke war nicht das *Wie*, sondern das *Ob*: Es gab kein einziges `INFO`, `ERROR` oder `exception`, also weder einen sichtbaren Erfolgsfall noch einen brauchbaren Fehlerfall.
- **Exceptions loggen und weiterwerfen ist doppelt.** Wer an der Fangstelle schon `logger.exception` ruft und die Exception dann erneut wirft, erzeugt denselben Traceback zweimal im Log. Entweder behandeln (und loggen) oder durchreichen (und schweigen) — die Entscheidung fällt dort, wo es eine Handlungsalternative gibt, also am Rand.
