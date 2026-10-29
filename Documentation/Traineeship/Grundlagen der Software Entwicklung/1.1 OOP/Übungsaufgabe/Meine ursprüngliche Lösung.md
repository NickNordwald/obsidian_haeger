# SOLUTION.md

## Inhalt

1. [Überblick](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#1-%C3%BCberblick)
2. [Tech Stack](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#2-tech-stack)
3. [Setup & Ausführung](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#3-setup--ausf%C3%BChrung)
4. [Tests ausführen](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#4-tests-ausf%C3%BChren)
5. [Projektstruktur](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#5-projektstruktur)
6. [Datenmodell](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#6-datenmodell)
7. [Interpretation der Daten](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#7-interpretation-der-daten)
8. [Wichtige Entscheidungen](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#8-wichtige-entscheidungen)
9. [Datenqualität & bekannte Schwächen](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#9-datenqualit%C3%A4t--bekannte-schw%C3%A4chen)
10. [Performance](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#10-performance)
11. [Tests & Verifikation](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#11-tests--verifikation)
12. [Was ich für Produktion anders machen würde](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#12-was-ich-f%C3%BCr-produktion-anders-machen-w%C3%BCrde)
13. [Offene Punkte / Nächste Schritte](https://github.com/NickNordwald/Probeaufgabe/blob/main/SOLUTION.md#13-offene-punkte--n%C3%A4chste-schritte)

---
## 1. Überblick

Diese Lösung implementiert die in `README.md` beschriebene Bewerberaufgabe: einen prototypischen Backend-Service, der tägliche CSV-Drops eines Tourismus-Kunden in eine Postgres-Datenbank einliest, Preisempfehlungen auf Basis einer einfachen Heuristik berechnet und diese Informationen über eine HTTP-API bereitstellt.

Der Service exponiert die drei in der Aufgabenstellung geforderten Endpunkte (`/start-ingestion`, `/price-recommendations`, `/max-recommendation`) sowie einen zusätzlichen Upload-Endpunkt (`/start-ingestion-upload`) als Komfort-Variante für manuelles Testen. Die Ingestion basiert auf Polars, die API auf FastAPI, und das Datenbankschema wird per Alembic migriert.

Diese Datei (`SOLUTION.md`) dokumentiert meine Lösung, die getroffenen Annahmen, bewusste Vereinfachungen und bekannte Schwächen. Die ursprüngliche `README.md` mit der Aufgabenstellung des Kunden bleibt unverändert.

---

## 2. Tech Stack

- **Python 3.11** als Basis-Laufzeit.
- **FastAPI** für die HTTP-API, weil es aus Type-Hints automatisch Request-Validierung und eine interaktive Doku-Seite (`/docs`) generiert, was den Boilerplate im Vergleich zu Flask oder Django deutlich reduziert.
- **Pydantic** für die Wire-Format-Modelle (Request- und Response-Schemas). Wird von FastAPI vorausgesetzt.
- **Polars** für die CSV-Transformationen in der Ingestion und für die Berechnung der Preisempfehlungen. Schneller und speicheroptimierter als Pandas bei den hier relevanten Datenmengen (~1.3M Capacity-Zeilen) und hat eine klarere Expression-API, die Chain-Transformationen gut lesbar macht.
- **Postgres 17** als relationale Datenbank, im Container via Docker Compose.
- **SQLAlchemy** als ORM bzw. für einzelne SQL-Statements (TRUNCATE, `_fetch_top_recommendation`, `_fetch_current_prices`).
- **ADBC (Arrow Database Connectivity)** für die eigentlichen Bulk-Writes und Reads. Schneller als SQLAlchemys Row-by-Row-INSERT — siehe Abschnitt _Performance_.
- **Alembic** für die Schema-Migrationen, wie in der Aufgabenstellung empfohlen.
- **pytest** und **ruff** (Linter + Formatter) als Dev-Tools.
- **Docker Compose** für die lokale Postgres-Instanz und Adminer als optionales Web-Interface zur Datenbank.

---

## 3. Setup & Ausführung

Voraussetzungen: Python 3.11, Docker und Docker Compose.

**Einmaliges Setup:**

```shell
# 1. Virtuelle Umgebung anlegen und aktivieren
python -m venv venv
source venv/bin/activate

# 2. Projekt inkl. Dev-Abhängigkeiten installieren
pip install -e ".[dev]"

# 3. Postgres starten
docker compose up -d

# 4. Datenbank-Schema anlegen
alembic upgrade head
```

**Service starten:**

```shell
python -m src.main
```

Der Service läuft dann auf `http://localhost:8000`. Die interaktive API-Dokumentation (Swagger UI) ist unter `http://localhost:8000/docs` erreichbar und eignet sich gut für manuelles Testen — alle Endpunkte lassen sich dort direkt ausführen.

**Typischer Ablauf zum Ausprobieren:**

1. `POST /start-ingestion` mit Body `{"data_dir": "daily_data"}` — liest die mitgelieferten CSV-Dateien in die Datenbank.
2. `POST /price-recommendations` — berechnet und persistiert die Preisempfehlungen für alle zukünftigen Reisedaten.
3. `GET /max-recommendation` — gibt die Empfehlung mit der höchsten vorgeschlagenen Preiserhöhung zurück.

Alternativ zu `/start-ingestion` existiert `POST /start-ingestion-upload`, bei dem die drei CSV-Dateien direkt im Request hochgeladen werden können (Drag & Drop über `/docs`). Dieser Endpunkt ist nicht Teil der Aufgabenstellung, sondern eine Komfort-Ergänzung.

**Aufräumen:**

```shell
docker compose down -v   # stoppt Postgres und löscht das Volume
```

---

## 4. Tests ausführen

pytest tests/ -v

Der Testlauf umfasst aktuell 17 Tests in zwei Dateien:

- `tests/test_ingestion.py` deckt die beiden Preprocessing-Methoden `_build_capacity` und `_build_prices` ab: Mehrnachtige Buchungen werden korrekt über alle belegten Nächte expandiert, stornierte Buchungen gefiltert, überlappende Buchungen korrekt aufsummiert. Für `_build_prices` wird das Deduplizieren des zusammengesetzten Primärschlüssels geprüft sowie das Entfernen der DWH-ID-Spalte und das Umbenennen auf die Ziel-Schema-Namen.
    
- `tests/test_recommendations.py` deckt die Kernlogik `_compute` ab: Filterung auf zukünftige Reisedaten, Skip-Behandlung von Zeilen mit Gesamtkapazität 0 bzw. negativer Summe aus `bookable + optional`, korrekte Formel-Anwendung inklusive Dezimalpräzision, und die korrekte Weitergabe des Batch-Zeitstempels.
    

Die Tests arbeiten ausschließlich mit In-Memory-Polars-DataFrames — es ist keine laufende Datenbank nötig. Hintergrund dieser Entscheidung und was ausdrücklich _nicht_ getestet wird, steht weiter unten in Abschnitt 11 (_Tests & Verifikation_).

---

## 5. Projektstruktur

```
.
├── alembic/                 # Schema-Migrationen (Alembic)
│   ├── env.py
│   └── versions/
│       └── 001_init.py      # Initiale Tabellen: capacity, prices, price_recommendations
├── alembic.ini
├── daily_data/              # Vom Kunden gelieferte CSV-Drops (Beispieldaten)
├── notebooks/               # EDA-Notebooks und manuelle Verifikations-Zellen
├── src/
│   ├── __init__.py
│   ├── api.py               # FastAPI-App, alle Endpunkte
│   ├── database.py          # SQLAlchemy-Engine und Verbindungs-URL
│   ├── datamodels.py        # SQLAlchemy-ORM-Klassen (Schema)
│   ├── ingestion.py         # Ingestor-Klasse, Preprocessing + Bulk-Write
│   ├── main.py              # uvicorn-Einstiegspunkt
│   ├── recommendations.py   # RecommendationEngine: Berechnung + DB-Zugriffe
│   └── schemas.py           # Pydantic-Request/Response-Modelle
├── tests/
│   ├── fixtures/
│   └── test_ingestion.py
│   └── test_recommendations.py
├── docker-compose.yml       # Postgres + Adminer
├── pyproject.toml           # Abhängigkeiten und Projekt-Metadaten
├── README.md                # Aufgabenstellung des Kunden (unverändert)
└── SOLUTION.md              # Diese Datei
```

Innerhalb von `src/` ist die Struktur bewusst flach gehalten. Bei dem aktuellen Umfang (≈ 8 Dateien) bringen Unterordner wie `src/api/`, `src/services/` oder `src/models/` mehr Navigations-Overhead als Orientierungsgewinn. Sobald einzelne Module spürbar wachsen — z. B. wenn `api.py` Richtung mehrere hundert Zeilen geht — lohnt sich das Aufteilen in `src/routers/`. Für einen Prototyp in dieser Größe wäre das verfrüht.

Funktionale Trennung innerhalb von `src/`:

- **`api.py`** enthält ausschließlich HTTP-Routing. Jeder Endpunkt ist ein dünner Adapter, der Request-Parsing, Aufruf einer Funktion aus `ingestion.py` oder `recommendations.py` und Response-Formatierung übernimmt. Keine Business-Logik.
- **`ingestion.py`** und **`recommendations.py`** enthalten die eigentliche Fachlogik — CSV-Verarbeitung bzw. Empfehlungs-Berechnung. Beide sind ohne laufenden HTTP-Server testbar.
- **`schemas.py`** (Pydantic, Wire-Format) und **`datamodels.py`** (SQLAlchemy, DB-Schema) sind bewusst getrennt: das HTTP-Format und das Datenbank-Schema haben unterschiedliche Lebenszyklen und dürfen sich unabhängig voneinander entwickeln.

---

## 6. Datenmodell


Das Schema besteht aus drei Tabellen:

**`capacity`** — eine Zeile pro `(product_id, calendar_date)`. Primärschlüssel ist die Kombination dieser beiden Spalten. Die drei Zahlenspalten `booked_units`, `bookable_units`, `optional_units` entsprechen den drei Komponenten der Gesamtkapazität nach der Definition aus der Aufgabenstellung (`Gesamtkapazität = booked + bookable + optional`).

**`prices`** — eine Zeile pro `(product_id, change_date, length_of_stay)`. Primärschlüssel ist diese Dreier-Kombination. `price_sterling` hält den Preis als `Numeric` (Postgres) bzw. `Decimal` (Polars) — siehe Abschnitt 8 zum Grund der Entscheidung gegen `Float`.

**`price_recommendations`** — eine Zeile pro `(product_id, calendar_date)` mit empfohlener prozentualer Preisanpassung, `days_until_travel` als vorgerechnete Konstante und `calculated_at` als Batch-Zeitstempel. Primärschlüssel analog zu `capacity`.

**Entscheidungen, die ich dabei getroffen habe:**

Ich habe mich dafür entschieden, **die Rohdaten aus `bookings.csv` nicht zu persistieren**. Für die Aufgabenstellung wird aus den Buchungen nur die Anzahl der bestätigten Buchungen pro bookable unit benötigt — diese Information landet beim Ingestion in der `booked_units`-Spalte der `capacity`-Tabelle. Die übrigen Spalten (Kunden-ID, Umsatzzahlen, Feature-Flags, Stornierungsdatum) spielen für die drei Endpunkte keine Rolle. Das hält das Modell kompakt und auf das tatsächlich Gefragte fokussiert. Für zukünftige Auswertungen, die Buchungsdetails bräuchten (z. B. Umsatzanalysen), wäre das eine bewusste Erweiterung — im aktuellen Umfang wären sie nur Ballast.

Ich habe die `id`-Spalten aus dem Data Warehouse in allen drei Quell-CSVs **nicht in das Ziel-Schema übernommen**. Die Aufgabenstellung stellt das explizit frei, und für die Business-Logik ist diese ID irrelevant.

Ich habe `product_id` **nicht in eine eigene Produkt-Tabelle extrahiert**, obwohl das aus streng relationaler Normalisierungs-Sicht sauberer wäre. Im gelieferten Datensatz gibt es keine Produkt-Attribute außer der ID selbst — eine Produkt-Tabelle wäre eine einspaltige Tabelle ohne zusätzlichen Informationsgehalt und würde nur einen Join kosten.

Ich habe die drei Tabellen bewusst **getrennt gehalten, statt Empfehlungen als zusätzliche Spalten auf `capacity` abzulegen**. Dafür sprechen drei Gründe: Kapazität und Empfehlungen haben unterschiedliche Lebenszyklen (Ingestion schreibt täglich, Empfehlungen werden on demand berechnet); die API-Semantik bleibt sauber (jeder Endpunkt besitzt seine Tabelle); und der `calculated_at`-Zeitstempel auf `price_recommendations` ist zugehörige Metadaten, die nichts auf einer Kapazitäts-Zeile verloren hätten. Würde man beide Konzepte mischen, müsste jede Ingestion darauf achten, die Empfehlungs-Spalten nicht zu überschreiben — ein unnötiger Kopplungspunkt.

Als Schreib-Strategie habe ich mich für **TRUNCATE + INSERT** statt eines merge/upsert entschieden. Die CSV-Drops enthalten laut Aufgabenstellung immer den vollen Datensatz, und die Tabellen sind mit etwa 1.3M Zeilen für `capacity` und 125k für `prices` klein genug, dass ein Vollersatz in wenigen Sekunden abläuft. Ein Upsert wäre komplexer zu implementieren, fehleranfälliger und böte hier keinen praktischen Vorteil. `price_recommendations` wird vom Ingestion bewusst nicht angefasst — sie ist abgeleitete Daten und liegt in der Hoheit von `/price-recommendations`.

---

## 7. Interpretation der Daten


Die drei CSV-Dateien lassen einige Designfragen offen, zu denen die Aufgabenstellung und das mitgelieferte PDF jeweils nur Teil-Antworten geben. Die folgenden Punkte sind meine Lesart der Daten — plausibel gegeben dem, was ich in EDA und Dokumentation gesehen habe, aber nicht zwingend die einzig richtige. Für die Aufgabenstellung habe ich jeweils die Interpretation gewählt, die mir am besten zu den beobachteten Mustern passt, und die Entscheidung konsistent durch den ganzen Code gezogen.

**Was eine "bookable unit" ist.** Die Aufgabenstellung spricht von bookable units als Kombination aus `product_id` und einem Datum. Ich habe das als `(product_id, calendar_date)` umgesetzt — eine Zeile pro Produkt pro Kalendertag. Das passt zur Struktur der `capacity.csv` und zu den drei Endpunkten, deren Granularität ebenfalls diese Ebene ist.

**`prices.csv` als Preis-Changelog.** Meine Annahme ist, dass die `prices`-Tabelle kein vollständiger Snapshot aktueller Preise für alle bookable units ist, sondern eine Historie von **Preisänderungen**: Eine Zeile bedeutet "ab diesem `change_date` gilt für `(product_id, length_of_stay)` der Preis X, bis eine spätere Zeile ihn wieder überschreibt." Daraus folgt, dass der aktuelle Preis für eine konkrete Reisedauer und einen konkreten Reisetag der Preis aus der jüngsten Zeile mit `change_date ≤ Reisetag` ist. Ein Grund gegen die naheliegendere Lesart "strikte Lookup-Tabelle mit exaktem `(product_id, date, length_of_stay)`-Match" ist, dass dieser Lookup in der EDA nur für etwa 80% der Buchungen ein Ergebnis liefert — rund ein Fünftel der Buchungen hat keinen exakt passenden Preis-Eintrag. Die regelmäßigen Abstände zwischen den Änderungsdaten pro Produkt/Reisedauer (oft wöchentlich) passen zusätzlich zur Changelog-Lesart. In `_fetch_current_prices` ist genau dieser Lookup implementiert.

**`length_of_stay = 'XN'` als Auffang-Kategorie.** Die ausgeschriebenen Werte `2`, `3`, `4`, `7` lesen sich als explizit modellierte Standard-Aufenthalte. Meine Annahme ist, dass `XN` der Fallback für alle anderen Aufenthaltsdauern ist (X = beliebig), also z. B. für 5-, 6-, 10-Nächte-Buchungen. Der Code behandelt `XN` wie jeden anderen `length_of_stay`-Wert — diese Interpretation ist an keiner Stelle hart verdrahtet, sondern fließt höchstens in die Bedeutung eines Preises ein, die der Consumer der API interpretiert.

**`capacity.csv` als Voll-Kalender.** Meine Annahme ist, dass `capacity.csv` einen vollständigen Kalender pro Produkt über den gesamten Zeitraum enthält — auch für Daten, an denen das Produkt nicht verkauft wird. Die Gesamtkapazität einer bookable unit ist laut Aufgabenstellung `booked + bookable + optional`; `booked` wird dabei aus `bookings.csv` abgeleitet und beim Import in die `capacity`-Tabelle eingemischt. Ist diese Summe 0, heißt das nicht "ausverkauft", sondern "wird an diesem Tag nicht angeboten" (z. B. saisonal geschlossen, noch nicht freigegebene zukünftige Jahre, Renovierung). Wichtig ist die Abgrenzung: `bookable + optional = 0` alleine wäre bloß "an diesem Tag lässt sich nichts mehr neu buchen" (plausibler Zustand bei vollem Hotel) und wäre kein Grund, die Zeile zu überspringen. Erst wenn auch `booked = 0` ist — also die echte Gesamtkapazität 0 wird — wird klar, dass an diesem Tag gar nichts existiert. Knapp 80% aller Zeilen in `capacity.csv` sind in diesem Zustand, was stark für die Voll-Kalender-Interpretation spricht. Abschnitt 9 geht näher auf die Zahlen ein.

**Gleichzeitige An- und Abreise am selben Tag.** Meine Annahme ist, dass zwei Buchungen dieselbe bookable unit teilen dürfen, wenn die eine an dem Tag endet, an dem die andere beginnt — also wenn der Abreisetag der einen Buchung der Anreisetag der nächsten ist. Eine Buchung mit `bkg_nights = N` ab Datum D belegt Kapazität an den Nächten D, D+1, …, D+N-1, aber **nicht** an D+N (dem Checkout-Tag). Die "Nacht" und nicht der "Tag" ist die belegte Einheit. Das spiegelt branchenübliches Hotel-Verhalten wider und passt zur Art, wie `bkg_nights` in den Buchungen modelliert ist.

**Spalten-Semantik: `is_bookable` und `is_option`.** Die Spaltennamen klingen wie Boolean-Flags, sind aber laut begleitendem PDF tatsächlich Zählungen (wie viele Zimmer noch buchbar bzw. optional sind). Ich habe sie beim Import in `bookable_units` und `optional_units` umbenannt, damit das Schema der Datenbank ehrlich beschreibt, was drinsteht.

**Währung.** Die Spalte `base_currency` in den Buchungen ist dominant `STG` (britische Pfund). Die Aufgabenstellung nennt keine Währung für die `prices`-Tabelle, aber die Kombination legt nahe, dass das ein britisches Unternehmen ist und Preise in GBP angegeben sind. Ich habe den entsprechenden Zielspalten-Namen `price_sterling` gewählt, um das explizit zu machen — auch wenn das streng genommen eine Annahme ist.

**`status`-Werte.** Laut PDF darf nur `status = "Firm"` als "gilt für Kapazität" gewertet werden; `Cancelled`, `Expired` und `Pending` werden bei der Ingestion verworfen. Das spiegelt der Filter in `_build_capacity` wider.

---

## 8. Wichtige Entscheidungen

**"Höchste empfohlene Preisanpassung" interpretiert als vorzeichenbehaftetes Maximum.** Die Aufgabenstellung verlangt für `/max-recommendation` die bookable unit mit der "höchsten empfohlenen Preisanpassung". Der Ausdruck ist mehrdeutig: entweder die **betragsmäßig größte** Anpassung (also auch stark negative Werte) oder die **signierte größte** Anpassung (der positivste Wert). Ich habe mich für die signierte Lesart entschieden, weil das Wort "höchste" im Deutschen typischerweise ordinal (größer/kleiner), nicht betragsmäßig zu lesen ist — eine "sehr niedrige" und eine "sehr hohe" Zahl sind nicht dasselbe. Eine betragsorientierte Interpretation wäre operativ möglicherweise interessanter (auch starke Preissenkungen sind handlungsrelevant), aber weniger im Einklang mit dem Wortlaut. Die Implementierung ist trivial tauschbar, falls die andere Lesart gewünscht wäre — ein Wechsel von `ORDER BY recommended_adjustment_pct DESC` zu `ORDER BY ABS(recommended_adjustment_pct) DESC` in `_fetch_top_recommendation`.

**`/max-recommendation` ist read-only.** Der Endpunkt rechnet keine Empfehlungen neu. Wenn die Tabelle `price_recommendations` leer ist (z. B. weil noch niemand `/price-recommendations` aufgerufen hat), gibt er HTTP 404 mit einem entsprechenden Hinweis zurück, statt automatisch neu zu berechnen. Jeder Endpunkt soll genau eine Sache tun — `/price-recommendations` rechnet, `/max-recommendation` liest —, und die Kopplung zwischen beiden ist damit explizit der Aufruf-Reihenfolge überlassen.

**Referenzdatum aus der Systemuhr, mit Test-Override.** Der Begriff "Tage bis zum Reiseantritt" braucht einen Bezugspunkt für "heute". Ich nehme dafür die Systemuhr (`date.today()`), weil das dem realen Produktions-Szenario ("dieser Service läuft täglich") am nächsten kommt. Der `RecommendationEngine`-Konstruktor akzeptiert aber einen optionalen `reference_date`-Parameter, mit dem Tests und Debug-Läufe das Datum festlegen können. Das Property-Pattern (`@property` auf `reference_date`) sorgt dafür, dass bei System-Uhr-Modus jeder Aufruf das aktuelle Datum sieht und nicht das Datum, an dem die Instanz erzeugt wurde.

**Polars statt reiner SQL-Implementierung für `/price-recommendations`.** Die Heuristik hätte man auch als einzelnes `INSERT ... SELECT` in SQL formulieren können — auf einer lokalen Postgres-Instanz wäre das wahrscheinlich leicht schneller, weil die Daten nicht über die Netzwerkverbindung zum Prozess und zurück müssen. Ich habe mich trotzdem für Polars entschieden: Die Berechnung bleibt damit in derselben Sprache und im selben Stil wie die Ingestion (konsistent für einen Reviewer), die Logik ist als reine Python-Funktion (`_compute`) unabhängig von einer laufenden Datenbank testbar, und der Performance-Unterschied liegt auf einem lokalen System im Sub-Sekunden-Bereich. Für einen entfernten DB-Server wäre eine SQL-native Implementierung der bessere Weg, für diesen Prototyp nicht.

**TRUNCATE + INSERT als Persist-Strategie** — bereits in Abschnitt 6 begründet.

**Behandlung des "ausverkauft"-Sonderfalls.** Die Heuristik aus der Aufgabenstellung vergibt bei 100% Auslastung (also `booked / total_capacity = 1.0`) die maximal mögliche positive Anpassung. Bei voll ausgebuchten Produkten (`bookable + optional = 0`) führt das dazu, dass solche bookable units das obere Ende der Empfehlungs-Rangliste dominieren, obwohl für sie gar nichts mehr zu verkaufen ist — die Empfehlung ist technisch korrekt, aber operativ nicht umsetzbar. Dieser Effekt ist eine Eigenschaft der Formel, nicht der Implementierung. Ich habe mich entschieden, die Empfehlung trotzdem unverändert zu speichern und auch in `/max-recommendation` nicht auszufiltern, weil das eine Abweichung vom Aufgabentext wäre und eine Entscheidung, die in der Praxis mit dem Kunden abzustimmen wäre. Das Verhalten ist in Abschnitt 9 dokumentiert.

**Flache `src/`-Struktur** — bereits in Abschnitt 5 begründet.

**Bonus-Endpunkt `/start-ingestion-upload`.** Die Aufgabenstellung schreibt für `/start-ingestion` einen Ordnerpfad auf dem Server-Dateisystem vor. Das ist für einen Prototyp sinnvoll, in der Praxis aber ungewöhnlich — ein entfernter Aufrufer kann keinen Pfad auf dem Server kennen oder referenzieren. Ich habe deshalb zusätzlich `/start-ingestion-upload` implementiert, bei dem die drei CSV-Dateien im Request hochgeladen werden. Der spezifikationskonforme Endpunkt `/start-ingestion` bleibt unverändert. Beide Endpunkte nutzen dieselbe Ingestor-Klasse; der Upload-Endpunkt schreibt die hochgeladenen Dateien in ein temporäres Verzeichnis und übergibt das an den Ingestor. Das Verzeichnis wird nach dem Request automatisch aufgeräumt.

**Dateinamen per Glob-Pattern statt hardcoded.** Die gelieferten CSVs haben ein Datum-Präfix (`20260108_capacity.csv` etc.), das sich täglich ändert. Statt das Datum zu parametrisieren, sucht der Ingestor per Glob nach `*_capacity.csv`, `*_bookings.csv`, `*_prices.csv` und erwartet genau einen Treffer. Mehrere oder kein Treffer erzeugen einen `FileNotFoundError` mit klarer Meldung. Das vermeidet eine überflüssige Konfiguration, macht aber gleichzeitig den Fall "Bediener hat das Verzeichnis nicht aufgeräumt und zwei Tages-Drops liegen nebeneinander" explizit bemerkbar.

**`price_sterling` als `Decimal`, nicht `Float`.** Preise werden im DataFrame als `pl.Decimal(precision=12, scale=2)` und in Postgres als `Numeric` gespeichert. Float-Storage hätte für die aktuellen Daten funktioniert (alle beobachteten Preise haben ≤ 2 Nachkommastellen und liegen weit innerhalb der Float64-Präzision), aber `Decimal` vermeidet grundsätzlich Rundungs-Artefakte bei späterer Arithmetik — wenn irgendwann Features hinzukommen, die mit Preisen rechnen (Umsätze summieren, Rabatte anwenden), ist das die sicherere Default-Wahl. Der Wechsel von Float auf Decimal kostete nichts und entfernt eine latente Fallgrube.

---

## 9. Datenqualität & bekannte Schwächen

In diesem Abschnitt sammle ich Punkte, an denen die gelieferten Daten entweder Auffälligkeiten zeigen oder die gewählte Lösung Schwächen hat. Die Grenze zwischen "Daten-Problem" und "Lösungs-Schwäche" ist dabei teilweise unscharf — manche Punkte sind beides.

**Zeilen mit Gesamtkapazität 0 werden übersprungen.** Bei der Berechnung der Empfehlungen werden Zeilen mit `booked + bookable + optional = 0` verworfen, weil sich für diese Zeilen keine sinnvolle Auslastung berechnen lässt (Division durch 0). In der gelieferten Daten-Drop sind das etwa 79% aller `capacity`-Zeilen, in zukünftigen Daten etwa 358.000. Wie in Abschnitt 7 beschrieben, lese ich diese Zeilen als "Produkt wird an diesem Tag nicht angeboten" — sie sind also kein Fehler und das Überspringen ist semantisch korrekt. Der entsprechende Zähler `rows_skipped_zero_capacity` in der `/price-recommendations`-Response ist dementsprechend systemisch groß und kein Alarmsignal.

**Zeilen mit negativer Summe `bookable + optional` werden übersprungen.** 197 Zeilen in den gelieferten Daten haben negative Werte in `is_bookable` und führen zu `bookable + optional < 0`, bei positivem `booked`. Ein plausibles Muster dafür ist Überbuchung — mehr bestätigte Buchungen als eigentlich verfügbare Plätze. Davon liegen 26 in der Zukunft, und genau diese 26 werden beim Berechnen der Empfehlungen verworfen, um keine unsinnigen Auslastungs-Zahlen zu produzieren (z. B. Auslastung > 100%). Das ist ein Upstream-Datenqualitäts-Problem, das in einer Produktion an den Datenlieferanten eskaliert werden sollte, nicht im Prototyp technisch umgangen werden.

**Duplikate in `prices.csv`.** Die Daten enthalten eine kleine Anzahl von `(product_id, change_date, length_of_stay)`-Duplikaten mit unterschiedlichen Preisen (im mitgelieferten Drop: 2 Fälle). Der Ingestor behandelt das defensiv, indem er pro Duplikat-Gruppe den höheren Preis behält. Die Wahl "Maximum" ist ein Platzhalter — keine der möglichen Regeln (max, min, erste, letzte, Durchschnitt) ist ohne Kontext korrekt; die richtige Lösung wäre eine Rückfrage beim Datenlieferanten. Der Ingestor meldet die Anzahl der kollabierten Duplikate sowohl in der API-Response als auch im Log.

**Dateninterpretation basiert auf Vermutungen.** Die in Abschnitt 7 dokumentierten Interpretationen (Prices als Changelog, Capacity als Voll-Kalender, XN als Fallback, GBP als Währung) sind meine plausibelsten Lesarten gegeben den Daten und dem mitgelieferten PDF — sie sind nicht offiziell bestätigt. Bei Abweichungen von der tatsächlichen Semantik müsste die Logik entsprechend angepasst werden.

**`/max-recommendation` bei ausgebuchten Produkten.** Wie in Abschnitt 8 diskutiert, liefert die Heuristik bei 100%-Auslastung maximalen positiven Anpassungs-Vorschlag. Wenn mehrere Produkte dieses Niveau erreichen, dominieren ausgerechnet die Zeilen die Spitze des Rankings, für die gar nichts mehr zu verkaufen ist. Das ist eine Eigenschaft der Formel aus der Aufgabenstellung, kein Bug der Implementierung. Eine mögliche pragmatische Gegenmaßnahme wäre, solche Zeilen aus `/max-recommendation` explizit auszufiltern (aber weiterhin zu speichern, weil die Formel sie vorschreibt). Ich habe darauf verzichtet, weil das eine Abweichung von der Aufgabenstellung wäre, die im echten Fall mit dem Kunden abzustimmen wäre.

**Keine Validierung des CSV-Inhalts.** Die beiden Ingestion-Endpunkte vertrauen darauf, dass die übergebenen CSV-Dateien die erwartete Struktur haben. Wird beispielsweise eine Datei hochgeladen, deren Name `*_capacity.csv` lautet, die aber den Inhalt einer Bookings-Datei hat, erzeugt der Parser einen Fehler, der im Endpoint als 500 Internal Server Error beim Aufrufer landet — ohne hilfreiche Meldung, worum es eigentlich geht. Eine Produktions-Härtung würde die Spalten-Header gegen eine Erwartung prüfen, bevor der eigentliche Import läuft.

**Fehlender rowwise-Integrationstest.** Die automatisierten Tests decken ausschließlich die Pipeline-Logik ab (In-Memory-DataFrame-Tests), nicht den tatsächlichen Datenbank-Schreibpfad. Ich habe den Schreibpfad manuell in einem Notebook-Zellen-Roundtrip verifiziert (`_build_*` → Postgres → zurücklesen → Gleichheitsvergleich), aber kein pytest-Fixture für eine Test-DB gebaut. Das ist eine bewusste Scope-Entscheidung für einen Prototyp; in einer Produktion wäre ein testcontainers-basierter Integrationstest die saubere Lösung. Abschnitt 11 geht darauf näher ein.

**Stillschweigendes Fallenlassen von "Orphan"-Buchungen.** Der Ingestor verwirft Firm-Buchungs-Nächte, deren `(product_id, calendar_date)` nicht in `capacity.csv` auftaucht. In der gelieferten Daten-Drop gibt es keine solchen Orphans, aber wenn zukünftig welche auftreten würden, landen sie nur im `orphan_bookings`-Zähler der Response und im Log. Der Ingestor importiert die restlichen Zeilen und signalisiert den Zähler-Wert, bricht aber nicht ab. Das ist für einen Prototyp richtig, aber die Grenze zwischen "interessante Warnung" und "ernster Datenfehler" wäre in einer Produktion mit einem Schwellenwert zu versehen.

---

## 10. Performance

Ingestion und Empfehlungs-Berechnung sollten auf einem lokalen Rechner in wenigen Sekunden durchlaufen. Die initiale Implementierung landete bei etwa 21 Sekunden für eine vollständige Ingestion von ~1.3M `capacity`-Zeilen und ~125k `prices`-Zeilen — in der Größenordnung annehmbar, aber deutlich langsamer als nötig.

**Wo die Zeit lag.** Ein einfaches Profiling der einzelnen Pipeline-Phasen zeigte, dass die Polars-Transformationen zusammen unter einer halben Sekunde liefen; die restlichen knapp 20 Sekunden entfielen fast vollständig auf den DB-Schreibpfad (~18 Sekunden allein für die 1.3M `capacity`-Zeilen). Die Ursache: Polars' Default-Schreibpfad (`write_database` mit SQLAlchemy-Engine) generiert einzelne `INSERT`-Statements und macht damit einen Round-Trip pro Zeile, auch auf `localhost`.

**Lösung: ADBC.** Polars unterstützt als zweite Schreib-Engine ADBC (Arrow Database Connectivity), die Daten als Arrow-Batches über `COPY BINARY` streamt — also Postgres' nativen Bulk-Load. Nach der Umstellung von `engine="sqlalchemy"` auf `engine="adbc"` (und einem zusätzlichen Paket `adbc-driver-postgresql` in `pyproject.toml`) sank die Schreibzeit für `capacity` von ~18 Sekunden auf ~3.4 Sekunden — etwa Faktor 5. Die Gesamt-Ingestion liegt nun bei ~5 Sekunden.

**Zwei Fallstricke bei der Umstellung.** ADBC ist strenger bei Typ-Kompatibilität als SQLAlchemy:

- Postgres `INTEGER` ist 32 Bit, Polars-Defaults sind 64 Bit. SQLAlchemy castet stillschweigend; ADBC lehnt das ab. Die Lösung: Im Ingestor werden `booked_units`, `bookable_units`, `optional_units` explizit auf `pl.Int32` gecastet. Analog in den Empfehlungen für `days_until_travel`.
- Postgres `NUMERIC` akzeptiert aus Polars weder Int noch Float direkt. Die zuverlässige Abbildung ist Polars' `Decimal(precision, scale)` — für `price_sterling` gewählt als `Decimal(12, 2)`, für `recommended_adjustment_pct` als `Decimal(12, 6)`.

Beide Typ-Entscheidungen sind dokumentiert im Code (siehe die Casts in `_build_capacity`, `_build_prices` und `_compute`).

**Auch die Lesepfade profitieren.** Polars bietet zwei Read-APIs: `read_database(connection=engine)` (über SQLAlchemy) und `read_database_uri(uri=..., engine="adbc")`. Ein Vergleich im Notebook zeigte für den `capacity`-Read: ~1.5s über SQLAlchemy, ~0.6s über connectorx, ~0.4s über ADBC. Für Konsistenz mit dem Schreibpfad habe ich `_read_capacity` auf ADBC umgestellt. Das Paket `connectorx` wurde aus den Abhängigkeiten entfernt, nachdem der Umstieg auf ADBC erfolgt war — es wurde nirgendwo mehr verwendet.

**Was als Nächstes lohnen würde, falls nötig.** Der verbleibende Schreib-Engpass (die ~3.4s für `capacity`) ließe sich mit Postgres `COPY FROM STDIN` direkt über psycopg2 noch einmal etwa halbieren, aber für einen Endpunkt, der einmal pro Tag läuft, rechnet sich der Aufwand nicht. ADBC gibt hier das bessere Verhältnis aus Code-Knappheit und Geschwindigkeit.

---

## 11. Tests & Verifikation

Die automatisierten Tests decken die reine Fachlogik der Pipelines ab — also die Teile, in denen Fehler am wahrscheinlichsten auftreten und am schwersten per Inspektion zu erkennen wären. Die Teile, die "nur" die Bibliotheken Polars und SQLAlchemy koordinieren, bewusst nicht.

**Was abgedeckt ist (17 Tests in zwei Dateien).**

`tests/test_ingestion.py` deckt die beiden Preprocessing-Methoden des Ingestors ab. Für `_build_capacity`: Mehrnachtige Buchungen werden korrekt über alle belegten Nächte expandiert, stornierte Buchungen aussortiert, überlappende Buchungen aufsummiert, fehlende Buchungs-Zuordnungen mit 0 aufgefüllt, und der Orphan-Zähler korrekt bilanziert. Für `_build_prices`: das Deduplizieren des zusammengesetzten Primärschlüssels mit der "max Preis"-Regel, das Entfernen der DWH-`id`-Spalte, das Umbenennen auf die Ziel-Schema-Namen, und die korrekte Duplikat-Zählung.

`tests/test_recommendations.py` deckt `_compute` ab — die Kern-Berechnung der Empfehlungen. Geprüft werden: die Filterung auf strikt zukünftige Reisedaten (d. h. Gegenwartsdatum wird ausgeschlossen), das Überspringen und Zählen von Zeilen mit Gesamtkapazität 0, das Überspringen und Zählen von Zeilen mit negativer `bookable + optional`-Summe (auch wenn die Gesamtkapazität positiv bleibt), die rechnerisch korrekte Formel-Anwendung für einen Normalfall und einen ausverkauften Sonderfall, die Dezimal-Präzision des Ergebnisses, das Output-Schema, und die einheitliche Weitergabe des Batch-Zeitstempels an alle Zeilen.

Beide Testdateien arbeiten ausschließlich mit In-Memory-Polars-DataFrames und benötigen keine laufende Postgres-Instanz. Ein kompletter Testlauf dauert Bruchteile einer Sekunde.

**Was bewusst nicht abgedeckt ist.**

Es gibt keinen automatisierten Integrationstest für den DB-Schreibpfad. Ein solcher Test würde prüfen: schreibe einen bekannten DataFrame, lies ihn zurück, vergleiche. Technisch möglich, aber aus meiner Sicht nicht im Verhältnis zum Aufwand: Der Test würde im Kern Polars' und SQLAlchemys Bibliotheks-Code prüfen, nicht meinen eigenen. Fehler auf dieser Ebene würden beim ersten realen Ingestion-Lauf sofort offensichtlich — einen Test zu schreiben, der nur die Interop-Schicht prüft, ist Aufwand ohne Erkenntnisgewinn für eigene Fehler.

Was für den Aufbau eines sauberen Integrationstests zusätzlich nötig wäre: eine separate Test-Datenbank (oder ein Container per Test-Run), Pytest-Fixtures für Setup und Teardown, und eine Migration auf diese Test-DB. Das sind weitere 30-50 Zeilen Test-Infrastruktur für einen Prototyp, in dem kein Bedarf für wiederholte Integrations-Prüfung besteht.

Ebenfalls nicht automatisiert getestet sind die drei DB-lesenden Methoden in `recommendations.py` (`_read_capacity`, `_fetch_top_recommendation`, `_fetch_current_prices`). Die ersten beiden sind triviale Einzel-SELECTs und werden durch das Endpunkt-Happy-Path-Manual-Testing implizit abgedeckt. Die dritte (`_fetch_current_prices` mit dem `DISTINCT ON`-Trick) habe ich stattdessen **manuell im Notebook verifiziert**: hand-angelegte Fixture-Zeilen in `prices`, Aufruf der Methode, Abgleich gegen ein Erwartungs-Dictionary, Cleanup. Der entsprechende Notebook-Ausschnitt liegt in `notebooks/` und ist jederzeit reproduzierbar ausführbar.

**Manueller End-to-End-Roundtrip.**

Als Ergänzung zu den Unit-Tests liegt im Notebook eine Zelle, die die vollständige Pipeline auf realen Daten prüft: Ingest → DataFrame → Postgres → DataFrame → Vergleich. Die Zelle hat einmal live bestätigt, dass Zeilen-Anzahl, Schema und Aggregate nach dem Round-Trip exakt übereinstimmen. Das ist keine Ersatz für einen automatisierten Test, aber es gibt Gewissheit darüber, dass der DB-Schreibpfad nicht irgendwo unterwegs stillschweigend Daten verändert.

---

## 12. Was ich für Produktion anders machen würde

Die vorliegende Implementierung ist ein Prototyp, wie in der Aufgabenstellung ausdrücklich erlaubt. Dieser Abschnitt listet die Punkte auf, die in einem produktiven Einsatz zusätzlich nötig oder deutlich anders wären. Keiner davon ist aktuell umgesetzt, sie sollen aber zeigen, dass ich die Distanz zwischen Prototyp und Produktion kenne.

**Authentifizierung und Autorisierung.** Aktuell kann jeder, der die URL kennt, `/start-ingestion` aufrufen und damit die Datenbank in einen anderen Zustand bringen. Für produktiven Einsatz brauchst du mindestens API-Keys, realistischerweise eine Integration in das SSO des Unternehmens. Die `/docs`-Seite wäre entsprechend hinter Auth zu stellen.

**Secrets-Management.** Die Datenbank-Verbindung liegt aktuell hartcodiert in `src/database.py` (`postgresql://tourism:tourism@localhost:5432/tourism`). In Produktion käme diese URL über eine Umgebungsvariable, gefüllt aus einem Secret-Manager (AWS Secrets Manager, Hashicorp Vault oder vergleichbar). Credentials gehören nicht in den Source-Code oder die Container-Image.

**Asynchrone Ingestion.** `/start-ingestion` hält aktuell die HTTP-Verbindung für die Dauer der Verarbeitung offen (~5 Sekunden). Das skaliert nicht — ein Uvicorn-Worker kann in der Zeit nur wenige parallele Ingestions abwickeln, und längere Läufe würden Timeouts produzieren. In Produktion wäre das Muster typischerweise: POST akzeptiert den Auftrag, gibt sofort eine Job-ID zurück, die Verarbeitung läuft als Background-Task (z. B. über Celery, ARQ oder eine Message-Queue), der Aufrufer pollt einen Status-Endpunkt.

**Validierung der CSV-Inhalte.** Wie in Abschnitt 9 erwähnt, vertrauen die Ingestion-Endpunkte aktuell blind auf die Struktur der Eingabe-Dateien. Eine produktive Variante würde die Spalten-Header und Datentypen gegen ein erwartetes Schema prüfen, bevor der eigentliche Import läuft, und bei Abweichungen einen klaren 400-Fehler zurückgeben.

**Ersatz des Pfad-basierten Ingestion-Endpunkts.** Der `POST /start-ingestion`-Endpunkt mit Pfad-Parameter ist für einen Prototyp mit "Server und User sind dieselbe Person" sinnvoll, aber in einem Mehr-Nutzer-System nicht praktikabel — externe Aufrufer kennen den Server-Dateipfad nicht. Eine produktive Variante würde entweder den Upload-Endpunkt zum primären Weg machen oder eine Cloud-Storage-URL akzeptieren (`s3://bucket/daily_data/`), die der Server selbst herunterlädt.

**Rate-Limiting.** `/price-recommendations` ist rechenintensiv (~5 Sekunden je Aufruf, Voll-Scan über `capacity`). Ohne Rate-Limiter kann ein bösartiger oder versehentlicher Client das System in eine Dauerlast zwingen. In Produktion würde das über einen Reverse-Proxy (nginx, envoy) oder ein FastAPI-Middleware wie `slowapi` laufen.

**Beobachtbarkeit.** Die aktuelle Anwendung loggt über das Standard-`logging`-Modul nach `stderr`. In Produktion brauchst du strukturierte Logs (JSON), die in einen zentralen Log-Aggregator gehen (ELK, Loki, Datadog), Metriken zu Request-Latenzen und Fehler-Raten (Prometheus, OpenTelemetry), und Alerts bei Anomalien.

**CI/CD-Pipeline und Deployment.** Aktuell läuft der Service über `python -m src.main` in einem Terminal. In Produktion gäbe es eine Dockerfile, einen automatisierten Build, automatisierte Test-Ausführung bei jedem Push, ein Deployment-Ziel (Kubernetes, ein Managed-Container-Service oder Ähnliches), und ein Rollback-Verfahren.

**Produktions-Postgres.** Die Instanz in `docker-compose.yml` ist für lokale Entwicklung. Für Produktion: Managed Postgres (RDS, Cloud SQL) mit automatischen Backups, Point-in-Time-Recovery, Monitoring und gegebenenfalls Read-Replicas für lesedominierte Endpunkte wie `/max-recommendation`.

**Rollback-Strategie für Empfehlungen.** `/price-recommendations` überschreibt die Tabelle komplett (TRUNCATE + INSERT). Wenn ein fehlerhafter Lauf schlechte Empfehlungen produziert, ist der vorherige Stand verloren. In Produktion würde man entweder die Empfehlungen versionieren (einen `calculated_at` oder `batch_id` als Teil des Primary Keys, alte Läufe bleiben erhalten) oder zumindest einen Backup/Restore-Mechanismus etablieren.

**Integrations-Tests.** Wie in Abschnitt 11 beschrieben: Produktiv gehört ein testcontainers-basierter Integrationstest für den DB-Schreibpfad dazu.

---

## 13. Offene Punkte / Nächste Schritte

Punkte, die im Rahmen dieser Abgabe bewusst nicht weiter verfolgt wurden, aber im Hinterkopf liegen:

**Genauere Untersuchung der Null-Kapazitäts-Zeilen.** Meine Interpretation der ~79% Zeilen mit Gesamtkapazität 0 als "Produkt wird an diesem Tag nicht angeboten" ist in Abschnitt 7 begründet und in Abschnitt 9 mit Zahlen belegt. Ein nächster Schritt wäre, diese Lesart positiv zu falsifizieren: Gibt es Produkte, die an Null-Kapazitäts-Daten trotzdem Firm-Buchungen hatten? Das wäre ein Widerspruch und würde die Interpretation untergraben. Ich habe diesen Check nicht durchgeführt — im Orphan-Check aus `_build_capacity` zeigt sich zwar, dass jede Buchungs-Nacht in `capacity` eine entsprechende Zeile hat (0 Orphans), aber nicht, ob das auch für Zeilen mit Null-Kapazität gilt.

**Zugriff auf den aktuellen Preis der buchenden bookable unit.** `/max-recommendation` liefert aktuell pro `length_of_stay` den zugehörigen Preis — korrekt gemäß dem Changelog-Modell aus Abschnitt 7. Was fehlt, ist der Preis, der _tatsächlich_ für die Buchung relevant wäre, wenn man `bkg_nights` kennt. Das Matching-Problem (Buchungen matchen nur in ~85% der Fälle auf einen exakten Prices-Eintrag, siehe EDA) würde eine eigene Fallback-Strategie nötig machen. Für die Aufgabenstellung reicht die aktuelle Darstellung; für einen BI-Consumer wäre es eventuell sinnvoller.

**Dedup-Strategie in `_build_prices` verfeinern.** Die aktuelle "Maximum-Preis"-Regel ist ein Platzhalter. Die richtige Lösung wäre eine Rückfrage beim Datenlieferanten, ob eine der Duplikat-Zeilen (z. B. die jüngere, die mit höherer DWH-ID, die spezifischere) als authoritativ gilt.

**"Höchste Anpassung"-Interpretation bestätigen.** Die in Abschnitt 8 diskutierte Interpretation als vorzeichenbehaftetes Maximum ist meine Lesart des Wortlauts. In einem realen Entwicklungs-Szenario wäre das ein Punkt für eine kurze Klärung mit dem Kunden — eine betragsmäßige Interpretation ist operativ naheliegend, und der Tausch ist minimaler Code-Aufwand.

**Erweiterung der Tests um `_fetch_top_recommendation` und `_fetch_current_prices`.** Aktuell sind diese beiden nur manuell verifiziert (siehe Abschnitt 11). Ein testcontainers-basiertes Setup wäre die saubere Lösung, falls die Empfehlungs-Engine weiter wächst.

**Einheitliche Paketverwaltung mit `uv`.** Die Abhängigkeits-Verwaltung läuft aktuell über `pip` und `pyproject.toml`. `uv` (aus dem Astral-Ökosystem, auch Herkunft von `ruff`) bietet deterministische Lockfiles und deutlich schnellere Installs. Der Umstieg kostet wenig und wäre für ein Projekt mit Produktionsanspruch eine sinnvolle Ergänzung.