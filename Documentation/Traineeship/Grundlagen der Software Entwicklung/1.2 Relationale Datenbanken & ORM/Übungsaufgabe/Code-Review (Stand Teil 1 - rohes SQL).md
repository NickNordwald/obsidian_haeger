---
tags:
  - documentation
type: code-review
date: 2026-07-22
---

Strenges, ausführliches Code-Review der Pricing-Anwendung am Ende von **Teil 1 (rohes SQL, psycopg 3)**, bevor Teil 2 (SQLAlchemy) beginnt. Kontext & Historie: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand (DB-Erweiterung)]]. Projektort: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/RDB_ORM`.

> [!info] Lesart
> Bewusst streng und vollständig, weil Nick explizit Lernwert will. Dateiverweise als `datei.py:zeile`. Reihenfolge: erst die harten Punkte (Korrektheit/Repo-Hygiene), dann Design, dann Kleinkram, am Ende **was richtig gut ist** und eine priorisierte To-do-Liste.

> [!success] Abgearbeitet im Aufräum-Durchgang (2026-07-22)
> **Abschnitt 1:** 1.1–1.6 alle erledigt (Hygiene + `net_revenue`-Validierung B3b + `_bedroom_types`-Konflikterkennung + `occupancy`-Guard). **Abschnitt 2:** 2.1 (Kommentar), 2.2 (Snapshot-Stempel), 2.4 (`CapacityRow`→`domain.py`), 2.5 (Kommentar, bewusst), 2.6 (Booking bewusst mutable — `status` ändert sich fachlich), 2.7 (`run()`), 2.8 (DRY + `${DB_PRICING_PW}`), 2.9 (verifiziert) erledigt; **2.3** bewusst als Diskussionsmaterial belassen → [[FK-Auflösung - App-Speicher vs. Datenbank|FK-Auflösung-Notiz]]. **Abschnitt 4:** Docstrings, `source_id`→`TEXT`, `[project.scripts]`, `mappings`-Typ erledigt (`main`-`print` bewusst gelassen). **Abschnitt 3 (Tests):** bewusst zurückgestellt → eigenes Kapitel. Beim Review nachträglich gefunden & gefixt: zwei `mypy`-`attr-defined`-Fehler, weil `CapacityRow` nach dem Umzug noch aus `sources` re-importiert wurde.

## Gesamturteil

Solide, überdurchschnittlich saubere Übungslösung. Die Architektur (Protocol-basierte DI, reine Domänenschicht, Layering) ist das Beste daran und trägt nachweislich — die DB-Erweiterung hat die Business-Logik nicht angefasst. Die Schwächen liegen **nicht** im Anwendungscode, sondern drumherum: Repo-Hygiene (echte Kundendaten + Artefakte eingecheckt, `py.typed` versehentlich ignoriert), **null automatisierte Abdeckung des kompletten DB-Pfads**, und ein paar konzeptionelle Spannungen (Float-vs-`Decimal`, `computed_at` vs. TRUNCATE-Reload, `CapacityRow`-Platzierung). Nichts davon ist ein Neuschreiben wert — es sind genau die Stellen, an denen man am meisten lernt.

Note im Geiste: **gut, mit klaren, benennbaren Baustellen.** Kein „passt schon".

---

## 1. Korrektheit & Repo-Hygiene (die harten Punkte)

### 1.1 `.gitignore` ignoriert versehentlich den eigenen `py.typed`-Marker — Bug
`.gitignore:5` enthält `*.typed`. `git check-ignore -v src/pricing/py.typed` bestätigt: der PEP-561-Marker ist **untracked**. Damit ist das Paket für Downstream-mypy **nicht** als typisiert erkennbar — die ganze `strict`-Typisierung verpufft, sobald jemand `pricing` von außen importiert. Vermutlich war `*.env` o. Ä. gemeint. **Fix:** die Zeile `*.typed` löschen und `py.typed` explizit committen.

### 1.2 Echte Kundendaten sind eingecheckt — Datenschutz
`git ls-files` zeigt `daily_data/20260108_{bookings,capacity,prices}.csv` als **getrackt**. `bookings` enthält `cust_id`, `gross_revenue`, `discount_amount` — also echte, personenbezogene Umsatzdaten des Arbeitgebers/Kunden in der Git-History. Das ist der ernsteste Punkt der ganzen Liste, unabhängig von Code-Qualität. **Fix:** `daily_data/` in `.gitignore`, aus dem Index nehmen (`git rm --cached`), und falls das Repo je geteilt/gepusht wird, History bereinigen (`git filter-repo`). Für die Übung reicht ein winziges synthetisches Fixture-Set.

### 1.3 Output-Artefakt & Scratch-Skript eingecheckt
`price_recommendations.csv` (generierter Output) und `scratch_test.py` (loses Ad-hoc-Skript im Root) sind ebenfalls getrackt. Generierte Artefakte gehören nicht ins VCS (Merge-Konflikte, Rauschen). `scratch_test.py` hat zusätzlich **Modul-Level-Seiteneffekte** (baut beim bloßen Import eine DB-Verbindung auf, Zeile 20) und einen irreführenden Namen: das `*_test.py`-Muster würde es von pytest eingesammelt — hier nur gerettet, weil `testpaths = ["tests"]` gesetzt ist (`pyproject.toml:62`). **Fix:** nach `scripts/` verschieben oder in `smoke_postgres.py` umbenennen, und beides gitignoren.

### 1.4 Toter Code: `BookingRow.net_revenue`
`ingest.py:24` liest `net_revenue` aus der CSV in das DTO, aber `_copy_bookings` (`ingest.py:158`) lässt die Spalte bewusst weg (DB berechnet sie `GENERATED`). Das Feld wird **nie gelesen** — es ist eingelesen und dann tot. Der Umsetzungsstand behauptet „`net_revenue = gross - discount` (0 Abweichungen)" als Verifikation, aber diese Prüfung steht **nirgends im Code** (war ein manueller SQL-Check). Zwei saubere Optionen: (a) Feld entfernen, oder (b) es sinnvoll machen — beim Ingest ein `assert net_revenue == gross_revenue - discount_amount` bzw. eine gezählte Warnung. (b) ist der Lernweg: die Invariante im Code festhalten statt im Kopf.

### 1.5 `_bedroom_types`: stille „last write wins"-Annahme
`ingest.py:117` baut `{b.product_id: b.feature_1 for b in bookings}` — bei mehreren Buchungen pro Produkt gewinnt die **letzte** in Iterationsreihenfolge, ohne Prüfung, ob `feature_1` je Produkt überhaupt konsistent ist. Die Annahme „ein Produkt hat genau einen Bedroom-Type" wird nirgends validiert. Wenn die Quelle das mal verletzt, bekommst du stumm einen willkürlichen Wert. Mindestens ein Kommentar, besser eine Konsistenzprüfung mit Warnung (analog zum schönen `orphan_nights`-Pattern).

### 1.6 `PriceableUnit.occupancy` erzwingt die eigene Vorbedingung nicht
`domain.py:57` dokumentiert „erwartet `total_capacity > 0`", teilt aber ungeschützt (`ZeroDivisionError` möglich). Aktuell gerettet, weil `RecommendationEngine.calculate_all` (`recommendations.py:54`) vorher filtert. Das ist eine **Fernkopplung**: die Klasse verlässt sich darauf, dass jeder Aufrufer die Vorbedingung kennt. Für eine „reine Domänenschicht", die Invarianten sonst konsequent selbst durchsetzt (`booked_units < 0` in `__post_init__`, `nights >= 1`), ist das inkonsistent. Entweder hier ein expliziter Guard/`raise`, oder zumindest bewusst dokumentiert lassen — aber die Inkonsistenz benennen.

---

## 2. Design & Architektur (das Interessante)

### 2.1 `Decimal` bei `recommended_adjustment_pct` ist kosmetisch — Scheinpräzision
`recommendations.py:21-30`: Die ganze Rechnung läuft in **float** (`occupancy` ist float-Division `domain.py:59`, `pct = 15.0 * ...`), und erst das Endergebnis wird via `Decimal(f"{pct:.6f}")` in ein `Decimal` gegossen. Das `Decimal` in `PriceRecommendation.recommended_adjustment_pct` (`domain.py:116`) und die Schema-Spalte `NUMERIC(10,6)` suggerieren dezimale Exaktheit, die es nie gab — der Fehler ist schon im float-Pfad passiert. Das ist ein **lehrreicher Klassiker**: `Decimal` schützt nur, wenn die gesamte Kette dezimal ist. Für einen Heuristik-Prozentsatz ist float völlig ok — dann aber ehrlich `float` bis zum Schluss, oder bewusst begründen, warum die Ausgabe-Spalte `NUMERIC` ist (Reproduzierbarkeit/Speicherformat). Nicht so tun, als sei es präzise.

### 2.2 `computed_at TIMESTAMPTZ DEFAULT now()` widerspricht dem TRUNCATE-Reload
`schema.sql:60` legt eine Zeitstempel-Spalte an — das signalisiert **Historie/Append** („wann wurde diese Empfehlung berechnet"). Der Writer macht aber `TRUNCATE price_recommendations RESTART IDENTITY` + Voll-Reload bei jedem Lauf (`writers.py:47`). Damit gibt es nie mehr als einen Zeitstempel-Jahrgang; `computed_at` ist praktisch nutzlos. **Entscheide dich:** entweder Output ist ein *Snapshot* (dann `computed_at` raus, konsistent zur CSV-Parität-Philosophie) oder du willst *Historie* (dann `INSERT` statt `TRUNCATE` + eine `run_id`/Batch-Spalte). Aktuell ist es ein Zwitter. Guter Diskussionspunkt fürs ORM: mit SQLAlchemy fällt Append-Historie leichter als mit dem TRUNCATE+COPY-Muster.

### 2.3 FK-Auflösung: 1,3 Mio Zeilen als Python-Dict
`writers.py:44-45` zieht die **komplette** `capacity`-Tabelle (`SELECT id, product_id, calendar_date`) bei *jedem* `write()` in ein Dict `{(pid, cal): id}`. Bei 1,33 Mio Kapazitätszeilen sind das etliche hundert MB Python-Objekte, nur um FKs aufzulösen. Es funktioniert und war die richtige Wahl gegenüber „N Einzel-`SELECT`s". Aber die *dritte* Option — die im ORM-Teil interessant wird — ist, die Auflösung **in der DB** zu lassen: Recommendations in eine Staging-Tabelle `COPY`en, dann ein `INSERT ... SELECT ... JOIN capacity USING (product_id, calendar_date)`. Kein Datentransfer nach Python, kein Dict. Merk dir das als expliziten Vergleichspunkt: *„FK-Auflösung im Applikationsspeicher vs. im Set-basierten SQL"*.

### 2.4 `CapacityRow` wohnt in `sources.py`, nicht in `domain.py`
`builder.py:11` importiert `CapacityRow` aus `sources` — die Kern-Business-Logik zieht also einen Typ aus der *äußeren* Schicht. `Booking`/`Price` liegen dagegen sauber in `domain.py`. `CapacityRow` ist konzeptionell genauso ein Domänen-DTO wie die beiden. Die aktuelle Platzierung dreht die Abhängigkeitsrichtung leicht gegen die im Modul-Docstring von `domain.py` deklarierte Regel („jedes andere Modul importiert diese, nicht andersherum"). **Empfehlung:** `CapacityRow` nach `domain.py` verschieben; `sources.py` und `builder.py` importieren es von dort. Kleiner Umzug, stimmigeres Layering.

### 2.5 `_load_capacity` im Ingest instanziiert eine ganze `CsvDataSource`
`ingest.py:73` ruft `CsvDataSource(data_dir).load_capacity()`. Der Konstruktor (`sources.py:62`) resolved aber **alle drei** CSV-Globs (capacity *und* bookings *und* prices), nur um dann capacity zu lesen. Kleiner Overhead und eine unnötige Kopplung „Ingest hängt am CSV-Reader-Konstruktor". Da `resolve_csv` ohnehin schon eine öffentliche Funktion ist, wäre ein direkter `_load_capacity` analog zu `_load_prices`/`_load_bookings` konsistenter — oder die Wiederverwendung bewusst als „DRY schlägt Mikro-Overhead" kommentieren.

### 2.6 `Booking` mutable, `Price`/`CapacityRow` frozen — inkonsistent
`domain.py:70` `@dataclass` (mutabel) vs. `Price` `@dataclass(frozen=True)` (`domain.py:95`) und `CapacityRow` frozen (`sources.py:22`). `Booking` validiert in `__post_init__`, aber weil es mutabel ist, kann die Invariante nach der Konstruktion wieder gebrochen werden (`b.nights = 0`). Für wertartige Zeilen-DTOs ist `frozen=True` der Default, den man begründet verlässt — hier fehlt der Grund. Entweder alle drei frozen, oder die Ausnahme benennen.

### 2.7 Doppelter `_compute()`-Aufruf — schon dokumentiert, hier für Vollständigkeit
`app.py:32` und `app.py:35` rufen `_compute()` je einmal → zwei volle Läufe (Laden + Bauen + Rechnen) pro Programmausführung. Bei `PostgresqlDataSource` = zwei komplette Table-Scans über `capacity`/`bookings`. Im Umsetzungsstand bereits als bewusste Nicht-Behebung vermerkt; korrekt so. Sauberster Fix wäre, `_compute()` einmal aufzurufen und die Liste an beide Konsumenten zu geben (die `PricingApp`-API leicht umbauen), nicht Caching im Objekt. Aber okay für später.

### 2.8 `db.py`: nur Passwort aus Env, Rest hardcodiert
`db.py:12` zieht nur `password` aus der Umgebung; `host`/`dbname`/`user` sind fest. Für die Übung fine und bewusst begründet (nur das Geheimnis gehört in Env). Anzumerken: Das Passwort lebt jetzt an **zwei** Orten — `.env` (`DB_PRICING_PW`) *und* im Klartext in `docker-compose.yml:8` (`POSTGRES_PASSWORD: pricing`). Idealerweise speist eine Quelle beide (Compose kann `${DB_PRICING_PW}` aus `.env` lesen). Kleine Duplikation, leicht auseinanderzudriften.

### 2.9 Docker: Volume mountet `/var/lib/postgresql` statt `.../data`
`docker-compose.yml:12` mountet das benannte Volume auf `/var/lib/postgresql` — das offizielle Postgres-Image nutzt üblicherweise `PGDATA=/var/lib/postgresql/data`, also eine Ebene tiefer. Das läuft zwar (PG18 hat hier Änderungen), ist aber untypisch und kann bei Image-Updates zu subtilen „warum ist das Volume plötzlich anders befüllt"-Effekten führen. Kurz gegenprüfen, ob das bewusst so gewählt ist; im Zweifel auf den Image-Default-Pfad mounten.

---

## 3. Tests — die größte echte Lücke

> [!note] Bewusst zurückgestellt (2026-07-22)
> Nick behandelt Testing in einem **eigenen Kapitel**. Dieser gesamte Abschnitt 3 wird daher **nicht** im Aufräum-Durchgang von Teil 1/2/4 gefixt, sondern dort vollständig aufgegriffen (Rollback-Fixture gegen die Docker-DB, Integrationstest für ingest→`SqlAlchemyDataSource`/`PostgresqlDataSource`, ungetestete Warn-Zweige). Hier nur zur Erinnerung dokumentiert.

Die vorhandenen Tests (`tests/test_pricing.py`) sind **gut gemacht**: gezielt, ohne Dateisystem, testen die Aggregations- und Formel-Kernlogik, mit sprechenden deutschen Namen und einem klugen `reference_date`-Pin. **Aber:**

- **Der gesamte DB-Pfad ist ungetestet.** `sources.PostgresqlDataSource`, `writers.PostgresqlRecommendationWriter`, `ingest.py` — **null** automatisierte Abdeckung. Genau der Code, der in Teil 1 neu dazukam und am ehesten bricht (Spaltenlisten, `COPY`-Reihenfolgen, FK-Auflösung), wird nur *manuell* verifiziert. Das ist die Lücke, die ich zuerst schließen würde.
  - Pragmatischer Weg: eine `pytest`-Fixture, die gegen die Docker-DB verbindet und **jeden Test in einer Transaktion laufen lässt, die am Ende zurückgerollt wird** (`conn.rollback()` in teardown) → schnell, isoliert, kein Zustand bleibt. Oder `testcontainers-python` für eine Wegwerf-DB.
  - Ein einziger Round-Trip-Test (ingest → `PostgresqlDataSource.load_*` → gleiche Zeilenzahl/Werte) fängt 80 % der realistischen Regressionen.
- **Ungetestete Zweige in der Logik:** die Warn-Pfade `has_negative_available_capacity`-Skip (`recommendations.py:57`), `orphan_nights` (`builder.py:43`), und der `missing`-Key-Pfad im Writer (`writers.py:56-77`) haben keinen Test. Gerade diese „Datenqualitäts-Signale" sind bewusst gebaut — dann sollten sie auch abgesichert sein, sonst verrotten sie stumm.
- **Kein Test für `get_max`-Tie-Break / `order=True`-Semantik.** `PriceRecommendation` vergleicht nur `recommended_adjustment_pct` (`domain.py:113-119`); ein Test, der zwei gleiche pct-Werte gegeneinander stellt, würde die Absicht dokumentieren.

---

## 4. Kleinkram / Stil

- `ingest.py` hat als **einziges** Modul keinen Docstring — die anderen sind konsequent bekopft.
- `domain.py:2-3` Modul-Docstring ist ein angefangener Satz ohne Verb („werden alle relevanten Entitätsklassen, welche … benötigt werden." → „… definiert."). Ein paar Docstrings haben kleine Grammatikhaken (`sources.py:31` „für einen Schnittstelle"). Kosmetisch, aber wenn schon durchgängig dokumentiert, dann rund.
- **Typ-Inkonsistenz im Schema:** `bookings.source_id BIGINT` (`schema.sql:24`) vs. `prices.source_id TEXT` (`schema.sql:47`) — dasselbe Konzept „Quell-Zeilen-ID", zwei Typen. Und die DTOs (`BookingRow.source_id: str`, `BookingRow.bkg_id: str`) typisieren numerische Spalten als `str`. `COPY` im Text-Modus parsed das zwar, aber die Typen lügen ein bisschen. Einheitlich `TEXT` (Quell-IDs sind selten arithmetisch) oder einheitlich `BIGINT` + `int` im DTO.
- `main.py:23` gibt per `print` aus, während der Rest über `logging` läuft. Für ein CLI-Ergebnis vertretbar, aber inkonsistent zum Logging-Setup zwei Zeilen darüber.
- Keine `[project.scripts]`-Entry-Points in `pyproject.toml` — es gibt zwei `main()` (`pricing.main:main` = CSV-App, `pricing.ingest:main` = Ingest), beide nur via `python -m`/Import erreichbar. Ein paar Console-Scripts würden die Bedienung dokumentieren.

---

## 5. Was richtig gut ist (nicht nur Kritik)

- **Protocol-basierte DI + reine Domänenschicht.** Das ist das Rückgrat und es zahlt sich messbar aus: `PostgresqlDataSource`/`-Writer` einzubauen hat `app.py`/`builder.py`/`recommendations.py` mit **keiner** Zeile verändert. Genau so soll Ports-&-Adapters aussehen. Der Domänen-Docstring, der DB/FS explizit verbietet, ist gelebte Disziplin.
- **`resolve_csv`** (`sources.py:40`) — Globbing über Suffix statt hartkodiertem Datum, mit **explizitem Fehler bei 0 *und* bei >1 Treffer**. Kleine Funktion, aber genau die richtige Paranoia.
- **`COPY` statt `executemany`** für den Massen-Import, plus `TRUNCATE ... RESTART IDENTITY CASCADE` + alles in *einer* `conn.transaction()` → idempotenter, atomarer Voll-Reload. Lehrbuchmäßig für ETL.
- **`Counter` für die `booked_units`-Aggregation** (`builder.py:21`) mit sauberer „Nacht ≠ Tag"-Semantik über `occupied_nights()` — und ein Test, der genau diese Kante prüft.
- **Datenqualität als gezählte Warnung statt Crash** (`orphan_nights`, `missing` keys, `skipped_*`). Das Muster „ein Ausreißer ist harmlos, aber die *Anzahl* verrät strukturelle Fehler" ist reif und konsistent durchgezogen.
- **`row_factory=dict_row` + Spalten-Aliasing → `Cls(**row)`** — verschlankt die Reader elegant und macht den psycopg-Vorteil (native Typen) sichtbar. Gutes Kontrastmaterial für den ORM-Vergleich.
- **Der `reference_date`-Pin im Vergleichslauf** (`scratch_test.py:18`) zeigt Verständnis dafür, dass zwei zeitversetzte Läufe sonst nicht vergleichbar sind. Denkt an die Testmethodik, nicht nur an den Code.

---

## 6. Priorisierte To-do-Liste (vor/parallel zu Teil 2)

1. **[Hygiene, sofort]** `daily_data/`, `price_recommendations.csv`, `scratch_test.py` aus dem Git-Index nehmen und gitignoren; `*.typed`-Regel löschen und `py.typed` committen. (Datenschutz + kaputte Typisierung.)
2. **[Tests]** Transaktions-Rollback-Fixture gegen die Docker-DB + **ein** Round-Trip-Integrationstest für ingest→`PostgresqlDataSource`. Schließt die größte Lücke, bevor SQLAlchemy als dritte Implementierung dazukommt.
3. **[Design-Klärung]** `computed_at` entscheiden (Snapshot vs. Historie) — die Antwort prägt direkt, wie du den SQLAlchemy-Writer baust.
4. **[Konsistenz]** `CapacityRow` → `domain.py`; `Booking` frozen; toter `net_revenue`-Pfad in echte Ingest-Validierung verwandeln.
5. **[Bewusst dokumentieren, nicht unbedingt fixen]** Float-vs-`Decimal`-Scheinpräzision und die FK-Auflösung-im-RAM — beides sind perfekte **Vergleichsachsen** für die ORM-Diskussion in Teil 2. Nicht wegräumen, sondern als Fragestellung mitnehmen.

> [!tip] Brücke zu Teil 2 (SQLAlchemy)
> Drei Punkte aus diesem Review werden im ORM-Teil zu konkreten Vor-/Nachteil-Argumenten: **(a)** FK-Auflösung — Relationship/Identity-Map vs. Bulk-Lookup-Dict vs. `INSERT…SELECT`; **(b)** `computed_at`/Historie — Unit-of-Work macht Append natürlicher als TRUNCATE+COPY; **(c)** Massen-Insert-Performance — ORM-`bulk_insert_mappings`/Core-`insert` vs. das `COPY` von hier (das ORM wird hier fast sicher *verlieren*, und genau das ist die Lehre). Behalte die psycopg-Zahlen (~110 s / 2,4 Mio Zeilen) als Baseline.
