---
tags:
  - documentation
type: q&a
date: 2026-08-19
---

Sammlung von Fäden, die in der [[1.4 Logging & Exceptionhandling - Vertiefung|Vertiefung]] bewusst nur gestreift oder ganz weggelassen wurden — als Landkarte für einen späteren, tieferen Durchgang. **Kein Pflichtstoff für das Kapitel.**

Legende: 🔴 lohnt sich sehr / berufsrelevant · 🟡 gut zu wissen · ⚪ Bonus, wenn Zeit und Lust da sind
Quellen: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]] · Kapitelrückblick: [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|Kapitelabschluss]]

*Nachträglich angelegt (19.08.2026) — dieser Ordner war leer, während 1.5 und 1.6 längst eine solche Landkarte hatten. Rekonstruiert aus den Querverweisen der späteren Kapitel und dem tatsächlichen Endstand (`shared/logging_setup.py`, `shared/exceptions.py`).*

---

# Loggen von Exceptions

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Unbehandelte Exceptions global abfangen** — Die Vertiefung behandelt `try`/`except` an definierten Stellen. Was fehlt, ist das Netz darunter: **`sys.excepthook`** (fängt alles, was bis zum Interpreter durchfliegt, und schreibt es ins Log statt auf ein verlorenes stderr), **`threading.excepthook`** (Threads sterben sonst stumm!) und **`faulthandler`** (schreibt einen Traceback sogar bei Segfaults und Hängern — `faulthandler.dump_traceback_later()` ist das Werkzeug für „der Prozess hängt und niemand weiß wo"). In einem Container, in dem niemand auf die Konsole schaut, ist das der Unterschied zwischen „abgestürzt" und „abgestürzt, und wir wissen warum".
- 🔴 **Fehlerklassifikation: transient vs. permanent** ⭐ Für Datenverarbeitung die eigentlich wichtige Unterscheidung, die die Vertiefung nicht trifft. *Transient* (Netzwerk weg, DB kurz nicht erreichbar, `503` von einer fremden API) → Retry hat Sinn. *Permanent* (Schema passt nicht, Datei fehlt, Validierung schlägt fehl) → Retry macht es nur schlimmer. Erst diese Einteilung macht aus „irgendwas ging schief" eine Handlungsanweisung — und sie ist die Voraussetzung für Retry/Backoff (siehe 1.5-Register) und Dead-Letter-Queues.
- 🟡 **Traceback-Objekte programmatisch verarbeiten** — das `traceback`-Modul (`format_exception`, `TracebackException`), `sys.exc_info()`, und `__traceback__` am Exception-Objekt. Braucht man, sobald man Fehler nicht nur loggt, sondern strukturiert weitermeldet (an Sentry, in eine Fehlertabelle, in eine Antwort).
- 🟡 **`logging.raiseExceptions` und Fehler im Logging selbst** — Was passiert, wenn der Handler scheitert (Platte voll, Netzwerk-Handler tot)? Standardmäßig schluckt Python das und schreibt eine Meldung nach stderr. Wichtig zu wissen, bevor man sich auf Logs als Beweismittel verlässt.
- ⚪ **Error-Tracking-Dienste** — Sentry, Rollbar, GlitchTip. Der Schritt von „Traceback steht im Log" zu „gleichartige Fehler werden gruppiert, gezählt, zugewiesen und mit Kontext angereichert". Sentrys Gruppierung ist übrigens genau der Grund, warum parametrisiertes Logging wichtig ist — das lernt man dort auf die harte Tour.

# Parametrisierte Logs

## Nicht behandelt, aber direkt anschlussfähig

- 🟡 **`{}`- und `$`-Stil statt `%`** — `logging.getLogger()` kann über `logging.setLogRecordFactory` bzw. `StrFormatStyle` auch `{}`-Formatierung. In der Praxis bleibt man bei `%`, weil die ganze Bibliothekswelt darauf steht — aber zu wissen, dass es kein Naturgesetz ist, hilft beim Lesen fremder Codebasen.
- 🟡 **`extra=` und der Namenskonflikt** — Die Vertiefung führt `extra=` ein. Der Fallstrick fehlt: Schlüssel, die mit den Standard-Attributen eines `LogRecord` kollidieren (`message`, `asctime`, `name`, `levelname`, `module`, `args`), lösen einen `KeyError` aus. Übliche Konvention dagegen: eigene Felder mit einem Präfix versehen oder alles unter einen Schlüssel packen.
- 🟡 **`logger.isEnabledFor(logging.DEBUG)`** — der explizite Guard für den Fall, dass schon das *Bauen* der Argumente teuer ist (eine Aggregation, ein `json.dumps` über eine große Struktur). Lazy Formatting schützt nur vor dem Formatieren, nicht vor der Auswertung der Argumente.

# Logging in Theorie und Praxis

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Correlation-/Run-IDs mit `contextvars`** ⭐ Der wichtigste offene Punkt des Kapitels, in der Vertiefung nur als Ausblick. Ohne eine ID pro Lauf bzw. pro Anfrage sind Logzeilen bei paralleler Verarbeitung nicht mehr zuordenbar. Die drei Wege im Vergleich: **`LoggerAdapter`** (muss durchgereicht werden), **`Filter`** (hängt am Handler, kann Felder injizieren) und **`contextvars`** (funktioniert über `async`-Grenzen hinweg und ist deshalb der moderne Weg). Praktisch in deinem Projekt: `pricing_runs.id` existiert bereits — sie in jede Logzeile des Laufs zu hängen wäre wenig Code mit viel Wirkung. Anschluss: die **Request-ID-Middleware** aus dem [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/Q&A/Weiterführende Stichpunkte.md|1.5-Register]].
- 🔴 **Wohin gehen die Logs eigentlich?** — Die Vertiefung endet bei Handlern, das Kapitel 1.7 sagt „Container schreiben nach stdout". Was dazwischen fehlt, ist die ganze Betriebsseite: Logging-Treiber des Docker-Daemons (`json-file` mit `max-size`/`max-file` — sonst frisst ein gesprächiger Container die Platte), Aggregatoren (Loki/Grafana, ELK, CloudWatch, Datadog), und die daraus folgende Regel: **Eine containerisierte Anwendung rotiert ihre Logs nicht selbst.** Dein `RotatingFileHandler` ist für den lokalen Lauf richtig und im Container eigentlich schon der falsche Ort — genau deshalb ist er dort per `LOG_FILE`-Weglassen abgeschaltet.
- 🟡 **`QueueHandler` / `QueueListener`** — Logging ist standardmäßig **synchron und blockierend**. Ein langsamer Handler (Netzwerk, volle Platte) bremst den Anwendungs-Thread. Die Standardlösung aus der Stdlib: Der Anwendungs-Thread schreibt nur in eine Queue, ein eigener Thread bedient die echten Handler. Ab Python 3.12 gibt es dafür sogar `QueueHandler`-Unterstützung direkt in `dictConfig`.
- 🟡 **Logging aus mehreren Prozessen** — `RotatingFileHandler` ist **nicht** prozesssicher; mehrere Uvicorn-Worker auf derselben Datei führen zu verschränkten oder verlorenen Zeilen. Antworten: `SocketHandler`/`QueueHandler` auf einen Sammelprozess, `WatchedFileHandler` + externes `logrotate`, oder eben stdout an den Container-Treiber.
- 🟡 **Die Konfiguration testen** — `dictConfig` ist eine große, stumme Datenstruktur: Ein Tippfehler in einem Handler-Namen fällt erst im Betrieb auf. `logging.config.dictConfig` wirft zwar bei groben Fehlern, aber ein Level auf dem falschen Logger bemerkt niemand. Ein einzelner Test, der `setup_logging()` aufruft und prüft, welche Handler am `pricing`-Logger hängen, kostet fünf Zeilen.
- 🟡 **`warnings` vs. `logging`** — zwei getrennte Systeme, die man verbinden kann (`logging.captureWarnings(True)`). Sonst gehen `DeprecationWarning`s aus Bibliotheken an den Logs vorbei.
- ⚪ **Weitere Handler** — `SMTPHandler` (Mail bei `CRITICAL` — meist eine schlechte Idee, siehe *Alerting*), `SysLogHandler`, `HTTPHandler`, `MemoryHandler` (puffert und schreibt erst bei einem Fehler den ganzen Kontext raus — ein hübsches Muster: normalerweise still, im Fehlerfall gesprächig).

# Logging-Level

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Wann wird aus einem Log ein Alarm?** — Die Vertiefung deutet Level richtig um („wer muss reagieren"), zieht die Konsequenz aber nicht: Ein `ERROR`, auf den niemand reagiert, ist ein `WARNING` mit falschem Etikett. Das zugehörige Vokabular: **Alerting-Regeln**, **symptombasiertes statt ursachenbasiertes Alerting** (Google SRE Book), **Alert Fatigue** — der Zustand, in dem ein Team gelernt hat, Rot zu ignorieren. Der Bezug zu deinem Projekt ist konkret: Deine 6 `logger.error`- und 9 `logger.exception`-Aufrufe sind ein Versprechen, dass jemand hinschaut.
- 🟡 **Level zur Laufzeit ändern, ohne Neustart** — Ein Endpunkt oder ein Signal-Handler, der das Level eines Loggers umschaltet. In der Produktion Gold wert: einen Fehler auf `DEBUG` beobachten, ohne den Prozess neu zu starten (und damit den Zustand zu verlieren, den man untersuchen wollte). Dein `LOG_LEVEL` aus der Konfiguration wirkt nur beim Start.
- 🟡 **Sampling und Rate Limiting** — Was tut man, wenn dieselbe Fehlermeldung 100.000-mal pro Minute kommt? Nur jede *n*-te loggen, oder eine Zusammenfassung pro Zeitfenster. Deine Entscheidung, Orphan-Buchungen **als Summe statt einzeln** zu loggen, ist genau dieses Muster — nur von Hand und ohne Namen.
- 🟡 **Die Kosten von Logging** — Log-Volumen ist in gemanagten Diensten ein echter Kostenposten (Datadog/CloudWatch rechnen pro GB). Das ist der unromantische, aber in der Praxis häufigste Grund, warum `DEBUG` in der Produktion aus ist.

# Logging-Frameworks

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Strukturiertes Logging konkret bauen** ⭐ Die Vertiefung erklärt überzeugend, *warum* strukturiert (der eigentliche Trend heißt nicht „Framework", sondern „strukturiert") — aber nicht, *wie*. Die drei Wege: ein **eigener JSON-`Formatter`** (~20 Zeilen, `json.dumps` über `record.__dict__` plus `extra`), **`python-json-logger`** (fertig, Stdlib-kompatibel) oder **`structlog`** mit seiner Processor-Pipeline. Der Knackpunkt, den man einmal gesehen haben muss: Wie kommen `exc_info` und der Traceback sauber in ein JSON-Feld?
- 🔴 **Die Dreiteilung Logs / Metriken / Traces** ⭐ Der eigentliche blinde Fleck dieses Kapitels. Tracing ist im Leitfaden bewusst out of scope, **Metriken kommen gar nicht vor** — dabei beantworten sie eine andere Frage als Logs: nicht „was ist passiert?", sondern „wie viel, wie schnell, wie oft?". Die Begriffe: Counter/Gauge/Histogram, Prometheus als De-facto-Standard, `/metrics`-Endpunkt, und **OpenTelemetry** als heutiger einheitlicher Standard für alle drei Säulen. Für deine Anwendung naheliegend: Dauer eines Preislaufs, Anzahl übersprungener Einheiten, Ingest-Zeilen pro Sekunde — alles Dinge, die du heute in Logtexte schreibst und die als Metrik auswertbar wären.
- 🟡 **`loguru` ehrlich einordnen** — die Vertiefung nennt den Haken. Zum Vertiefen: Warum ist „eine Bibliothek darf ihren Importeuren keine Logging-Konfiguration aufzwingen" bei `loguru` strukturell verletzt, und wie sieht die übliche Brücke aus (ein `InterceptHandler`, der Stdlib-Records an loguru weiterreicht)?
- ⚪ **Logging in anderen Ökosystemen** — Java (SLF4J/Logback, die MDC als Vorbild für `contextvars`), Go (`slog`, seit 1.21 in der Stdlib und bewusst strukturiert von Anfang an), Rust (`tracing`). Nützlich, weil dieselben Ideen dort teils sauberer gelöst sind — Gos `slog` ist praktisch das, was Pythons `logging` gerne wäre.

# Exceptionhandling best practices

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Retry, Backoff und Idempotenz zusammendenken** — Die Vertiefung behandelt „partielles Scheitern" als Entscheidung. Die Fortsetzung: Wenn ein Schritt wiederholbar sein soll, muss er **idempotent** sein — sonst erzeugt der zweite Versuch Dubletten. Werkzeuge: `tenacity` (Retry als Decorator, mit Backoff und Jitter), `backoff`. Und die Regel, die alles zusammenhält: *Retry nur bei transienten Fehlern* (siehe oben). Das ist derselbe Themenkomplex wie „Timeouts, Retries und Backoff" im 1.5-Register — hier von der Fehlerseite aus.
- 🔴 **Wo eine Exception zu einem Statuscode wird** — In deinem Projekt gibt es genau diese Übersetzungsschicht (`presentation/api/errors.py`), aber das Kapitel führt sie nie ein. Die Prinzipien dahinter: Die Fehlertaxonomie ist Teil des **Vertrags** nach außen, nicht Implementierungsdetail; eine Fehlermeldung darf keine Interna verraten; und ein einheitliches Fehlerformat (RFC 9457, siehe 1.5) ist mehr wert als schöne Einzelmeldungen.
- 🟡 **Exception-Hierarchien entwerfen — wie tief, wie breit?** — Deine Hierarchie ist ein gutes Anschauungsobjekt: `PricingError` als Basis, darunter `ConfigError`, `DataSourceError`, `NotFoundError`, `ConflictError` und ein `DomainError`-Zweig. Besonders sehenswert ist der Griff **`class DomainError(PricingError, ValueError)`** — Mehrfachvererbung, damit bestehender Code mit `except ValueError` weiter funktioniert, während neuer Code auf `PricingError` fangen kann. Das ist ein Standardmuster beim Nachrüsten einer Hierarchie in gewachsenen Code und in der Vertiefung nicht benannt.
- 🟡 **Fehler als Rückgabewert statt Exception** — die Gegenposition: `Result`/`Either`-Typen (aus Rust und der funktionalen Welt), in Python z. B. via `returns` oder schlicht `tuple[T | None, Error | None]`. Lohnt sich als Perspektivwechsel, weil es zeigt, dass Exceptions eine *Entwurfsentscheidung* sind und kein Naturgesetz — und weil man beim Lesen von Go- oder Rust-Code sonst stolpert.
- 🟡 **`ExceptionGroup` praktisch nutzen** — die Vertiefung führt Syntax und `except*` ein. Der Anwendungsfall fehlt: Bei paralleler Verarbeitung (mehrere Dateien, mehrere Tasks) scheitern potenziell *mehrere* Dinge gleichzeitig — genau dafür ist `ExceptionGroup` gebaut. Für einen Ingest über N Dateien das passende Werkzeug.
- 🟡 **Graceful Shutdown** — `SIGTERM` abfangen, laufende Arbeit zu Ende bringen, Verbindungen schließen. Verbindet dieses Kapitel direkt mit der **PID-1-Falle aus 1.7**: Die Signalbehandlung nützt nichts, wenn das Signal die Anwendung nie erreicht, weil eine Shell als PID 1 davorsteht. Ein `try`/`finally` schützt nicht gegen `SIGKILL` — und `atexit` läuft dabei auch nicht.
- ⚪ **Fehlerbudgets** — der SRE-Blick: Statt „keine Fehler" ein *akzeptiertes* Fehlerkontingent zu definieren, das Entwicklungstempo und Stabilität gegeneinander abwägt. Ordnet die Frage „ab wie vielen übersprungenen Einheiten ist der Lauf gescheitert?" in einen größeren Rahmen ein.

## Explizit out of scope laut Leitfaden

- 🟡 **Tracing** — bewusst vertagt. Zur Einordnung: Ein Trace ist die Verkettung aller Verarbeitungsschritte einer Anfrage über Dienstgrenzen hinweg (Spans, Parent-Child, Trace-ID). Die Correlation-ID oben ist faktisch die Einstiegsdroge dazu — wer sie hat, hat den halben Weg zu OpenTelemetry schon gemacht.
- 🟡 **Datenschutz** — out of scope, taucht aber in der Motivation des Leitfadens auf. Das praktische Minimum: Was darf **nie** ins Log (Passwörter, Tokens, personenbezogene Daten, vollständige Request-Bodys), wie verhindert man es *strukturell* statt per Disziplin (`SecretStr` aus Pydantic, Redaction-Filter am Handler), und was bedeutet ein Löschanspruch für Logs, die in einem Aggregator liegen?

# Fragen zum eigenen Projekt, die offen geblieben sind

- Ein Preislauf hat eine `pricing_runs.id`. Warum steht sie nicht in jeder Logzeile des Laufs? (Antwort: weil `contextvars` nie eingeführt wurde — es wären ~15 Zeilen.)
- 13 `logger.info`, 9 `logger.exception`, 6 `logger.warning`, 6 `logger.error`, 4 `logger.debug`, **0 `logger.critical`** — ist `CRITICAL` in dieser Anwendung ein toter Level, oder fehlt ein Fall? (Kandidat: Konfiguration fehlt beim Start, die Anwendung kann gar nicht arbeiten.)
- Der `RotatingFileHandler` ist im Container per weggelassenem `LOG_FILE` abgeschaltet. War das eine bewusste Entscheidung oder ein glücklicher Zufall? Und gehört diese Erkenntnis nicht in die Konfiguration selbst dokumentiert?
- `TRY003` wurde für `domain.py` per `per-file-ignores` deaktiviert („weil es nervt", so steht es im Code). Ehrlich, aber: Wäre eine attributtragende `DomainError`-Konstruktion analog zu `DataSourceError` nicht doch die bessere Antwort gewesen — und was genau spricht bei reinen Wertobjekten dagegen?
- Die Schwelle für partielles Scheitern (z. B. „> 5 % übersprungene Einheiten → Exit-Code ≠ 0") wurde geplant. Existiert sie im Endstand — und wenn nein: Ab wann ist ein Lauf mit vielen Skips eigentlich fehlgeschlagen?
