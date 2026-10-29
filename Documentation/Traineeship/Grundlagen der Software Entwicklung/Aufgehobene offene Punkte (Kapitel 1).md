---
tags:
  - documentation
type: notiz
date: 2026-08-19
---

# Aufgehobene offene Punkte (Kapitel 1)

Beim Eindampfen der `Umsetzungsstand`-Notizen am 19.08.2026 wurden alle „noch offen"/„als Nächstes"-Abschnitte entfernt — das Projekt wird nicht weitergeführt. **Was hier steht, ist die Auswahl, die den Schnitt überlebt hat:** echte konzeptionelle Lücken und Punkte, die als Übung wirklich etwas beibringen.

Alles andere (Kosmetik, Formatierung, Fleißarbeit, durch spätere Kapitel längst Erledigtes) ist ersatzlos entfallen und steht auch hier nicht mehr.

> [!note] Kein Auftragsbuch
> Das ist eine **Landkarte, keine To-do-Liste.** Nichts davon muss umgesetzt werden. Der Wert liegt darin, die Lücken benannt zu haben — falls das Projekt je wieder aufgemacht wird oder eine dieser Fragen im Job auftaucht.

## Inhalt

- [[#A. Echte konzeptionelle Lücken]]
- [[#B. Lohnende Übungen]]
- [[#C. Bekannte Schwächen im Endstand]]

---

# A. Echte konzeptionelle Lücken

Dinge, die im Projekt fehlen und deren Fehlen man begründen können muss.

## Die Migrationskette ist nie getestet worden

*(aus 1.6)* Alle 123 Tests laufen ohne Datenbank. Damit ist **keine** der vier Migrations-Prüfungen automatisiert: läuft die Kette aus leerer DB durch? Erzeugt Autogenerate danach keinen Unterschied mehr (`alembic check`)? Funktioniert hoch → runter → hoch? Bleiben die Daten bei einer Transformation korrekt?

Der Punkt ist deshalb HOCH, weil er zwei Kapitel gleichzeitig betrifft: 1.3 hat Integrationstests als Out-of-Scope deklariert, 1.6 braucht sie zwingend. Werkzeug wäre `testcontainers-python` (startet einen Postgres-Container pro Testlauf) oder der vorhandene Compose-Dienst mit separater Testdatenbank. Kernmuster: die **Transaktions-Rollback-Fixture**.

→ Details im [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Q&A/Weiterführende Stichpunkte.md|1.6-Register]] und im [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.3 Testing mit PyTest/Q&A/Weiterführende Stichpunkte.md|1.3-Register]]

## Die gesamte `persistence/`-Schicht ist ungetestet

*(aus 1.3)* Branch-Coverage im Kern: 100 %. In den Adaptern: 0 %. Das war eine bewusste Scope-Entscheidung des Leitfadens und ist als solche verteidigbar — aber es ist die Schicht, die tatsächlich mit PostgreSQL und dem Dateisystem redet, also die mit den meisten realen Fehlerquellen. Dieselbe Antwort wie oben: Integrationstests.

## Die Diskussion „ORM vs. rohes SQL" wurde nie ausformuliert

*(aus 1.2)* Der Leitfaden verlangt sie ausdrücklich: *„Im Anschluss: Nutze SQLAlchemy und diskutiere die Vor- und Nachteile der beiden Ansätze."* Beide Wege wurden gebaut (psycopg 3 **und** SQLAlchemy), die Bausteine für die Antwort liegen verstreut vor — [[FK-Auflösung - App-Speicher vs. Datenbank|FK-Auflösung]], [[ORM-Anbindungsvarianten & direktes Domänen-Mapping|ORM-Anbindungsvarianten]], [[Q&A (Kapitelabschluss)|Q&A (Kapitelabschluss)]] — aber die zusammenfassende Gegenüberstellung fehlt.

**Der einzige Punkt in dieser Liste, der eine unerfüllte Aufgabenstellung ist**, nicht nur eine Idee. Wenn die Trainer nachfragen, ist das die Stelle.

## Nebenläufigkeit im Schreibpfad

*(aus 1.5)* Zwei zusammenhängende Lücken, die dieselbe Wurzel haben:

1. **Die `409`-Prüfung beim Lauf-Start ist nicht race-frei.** Zwei gleichzeitige `POST /pricing-runs` können beide durchkommen, weil zwischen „gibt es einen laufenden Lauf?" und „lege einen an" keine Sperre liegt. Saubere Lösungen: partieller Unique-Index (`CREATE UNIQUE INDEX … WHERE status = 'running'`) oder ein `pg_advisory_lock`. Beides steht als Konzept im [[Antworten (Stichwörter & Fragen)|1.2-Q&A]] — hier wäre der Anwendungsfall gewesen.
2. **Preislauf und Ingest sind nur je für sich exklusiv, nicht gegeneinander.** Ein Ingest während eines laufenden Preislaufs zieht diesem die Eingangsdaten unter den Füßen weg (`TRUNCATE`!).

Übertragbar weit über dieses Projekt hinaus: *Job-Locking in der Datenbank* ist ein Muster, das man in jeder Pipeline wiederbraucht.

## Ein abgestürzter Lauf bleibt für immer `running`

*(aus 1.5)* Stirbt der Prozess mitten im Lauf, bleibt der Datensatz auf `running` stehen und blockiert jeden weiteren `POST` — dauerhaft, ohne manuellen Eingriff in die Datenbank. Das Standardmuster dagegen heißt **Stale-Job-Erkennung**: ein Heartbeat-Zeitstempel, den der Lauf regelmäßig aktualisiert, plus eine Altersgrenze, ab der ein Lauf als tot gilt.

Klassische Lücke jedes selbstgebauten Job-Systems und einer der Gründe, warum man irgendwann eine echte Task-Queue nimmt (Celery, RQ, Dramatiq, arq).

---

# B. Lohnende Übungen

Nichts davon ist ein Mangel. Aber jede dieser Übungen bringt eine Technik bei, die man sonst nirgends geübt hat.

## `create_autospec` / `Mock(spec=…)` statt rohem `Mock()`

*(aus 1.3)* Steht schon im ursprünglichen Umsetzungsstand als offener Punkt und ist die lehrreichste Kleinigkeit der ganzen Liste: Ein rohes `Mock()` akzeptiert **jeden** Methodennamen und **jede** Signatur. Ein Tippfehler wie `assert_called_once` statt `assert_called_once_with` erzeugt damit einen Test, der **grün lügt** — er prüft nichts und meldet Erfolg.

Einmal absichtlich herbeiführen und dann mit `autospec` reparieren: fünf Minuten, und man vergisst es nie wieder. Verwandt mit der Lektion „ein stiller Erfolg ist teurer als ein lauter Fehler", die in diesem Kapitel mehrfach aufgetaucht ist.

## Mutation Testing über den Kern laufen lassen

Der Kern hat 160 Statements und 100 % Branch-Coverage — das ist genau die Größe, bei der ein `mutmut`-Lauf noch in vertretbarer Zeit durchläuft und die Frage beantwortet, die Coverage nicht beantworten kann: *Prüfen die Tests die Logik, oder führen sie sie nur aus?*

Die konkrete Testfrage: Welche der 123 Tests werden rot, wenn man in der Heuristik `>` zu `>=` ändert?

## Eine Correlation-/Run-ID durch die Logs ziehen

*(aus 1.4)* `pricing_runs.id` existiert bereits. Sie über `contextvars` in jede Logzeile des Laufs zu hängen sind ~15 Zeilen — und es ist der Punkt, ab dem Logging bei parallelen Läufen überhaupt auswertbar wird. Gleichzeitig die Einstiegsdroge zu Tracing/OpenTelemetry.

## Fehler mit Datei und Zeilennummer melden

*(aus 1.1)* Beim CSV-Einlesen `enumerate(reader, start=2)` mitführen und im Fehlerfall `raise … from e` mit Datei und Zeilennummer. Kleine Sache, aber der Unterschied zwischen „`ValueError: invalid literal for int()`" und „Zeile 48.213 in `20260108_capacity.csv`: Spalte `is_bookable` ist kein Integer" ist im Betrieb enorm — und für einen Data Engineer die typischste Fehlermeldung überhaupt.

## Property-based Testing an einer Invariante ausprobieren

*(aus 1.3)* Der größte konzeptionelle Sprung, den das Testkapitel auslässt. Passende Kandidaten im eigenen Code: „`occupancy` liegt immer in [0, 1], solange `total_capacity > 0`" oder „eine Buchung über *n* Nächte belegt genau *n* Tage". Hypothesis sucht selbst nach Gegenbeispielen und schrumpft sie auf den minimalen Fall.

---

# C. Bekannte Schwächen im Endstand

Dokumentiert, damit sie beim Wiederlesen des Codes nicht für Absicht gehalten werden. Kein Handlungsbedarf.

| Wo | Was | Einordnung |
|---|---|---|
| `presentation/api/errors.py` | `DataSourceError → 503` steht in der Zuordnungstabelle, ist aber nicht implementiert — der Fall wird `500` | Doku/Code-Drift, schmaler Einzelfall |
| `SqlAlchemyRecommendationWriter.write()` | Temp-Tabelle ohne `checkfirst` — kollidiert, wenn zwei Läufe in einer Transaktion landen | realer Randfall, tritt im vorgesehenen Ablauf nie auf |
| `GET /v1/recommendations` | keine Filter nach `product_id` oder Zeitraum, kein Default-Limit | genau die Paginierungs-/Filter-Lücke, die das 1.5-Q&A-Register als 🔴 beschreibt |
| `persistence/ingest.py` | mischt Parsen, fachliche Validierung und `COPY` auf ~290 Zeilen | die einzige Datei, die der Schichtenlogik des Projekts nicht folgt |
| CRUD-Endpunkte für `bookings` | `POST /bookings` ändert weder Buchungslage noch Kapazität | bewusst so gebaut: folgt der Übungsvorgabe, nicht der Domäne — nicht für ein fachliches Modell halten |
| `pricing-apikey` | muss nach jedem frischen Volume manuell laufen, ist nicht Teil von `docker compose up` | bewusst manuell gelassen; automatisches Anlegen eines Schlüssels beim Start wäre in Produktion ein Sicherheitsproblem |
| Ingest-Laufzeit | ~2 min; ob der Engpass im Server (Index/WAL) oder im Client (Python-Schleife) liegt, wurde nie gemessen | offene Messfrage — `EXPLAIN ANALYZE` und ein Profiler wären die Werkzeuge |
| OOP-Snapshot (`OOP/`) | `.vscode/setting.json` ist falsch benannt (fehlendes „s"), `_compute()` läuft bei beiden Operationen doppelt | beides in den Folgekapiteln behoben; der Ordner ist ein eingefrorener Zwischenstand |

---

## Verwandte Notizen

- [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|Kapitelabschluss — Q&A-Ernte und blinde Flecken]] — der inhaltliche Rückblick über alle sieben Kapitel
- Die `Q&A/`-Register der einzelnen Kapitel enthalten die *thematischen* Auslassungen; diese Notiz hier nur die *projektbezogenen*.
