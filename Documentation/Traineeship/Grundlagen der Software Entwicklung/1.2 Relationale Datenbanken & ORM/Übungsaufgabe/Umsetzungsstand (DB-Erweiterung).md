---
tags:
  - documentation
type: notiz
date: 2026-07-22
---

Abgeschlossener Umsetzungsstand der **Datenbank-Erweiterung** der Pricing-Anwendung (Aufgabe: [[1.2 Relationale Datenbanken & ORM - Leitfaden#Übungsaufgabe|1.2 Leitfaden – Übungsaufgabe]]). Baut auf [[Umsetzungsstand (OOP-Lösung)|Umsetzungsstand (OOP-Lösung)]] auf.

Theorie: [[1.2 Relationale Datenbanken & ORM - Vertiefung|Vertiefung]] · [[psycopg 3 Grundlagen|psycopg 3 Grundlagen]] · [[SQLAlchemy Tutorial|SQLAlchemy Tutorial]] · Schema: [[Datenmodell|Datenmodell]] · Vertiefende Einzelfragen: [[FK-Auflösung - App-Speicher vs. Datenbank|FK-Auflösung]] · [[ORM-Anbindungsvarianten & direktes Domänen-Mapping|ORM-Anbindungsvarianten]] · [[Q&A (Kapitelabschluss)|Q&A (Kapitelabschluss)]]

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/RDB_ORM` (Kopie des OOP-Standes, eigener Ordner bewusst statt reinem Branch)

## Endstand

Die CSV-Datenhaltung wurde um PostgreSQL **erweitert**, nicht ersetzt — beide Wege funktionieren weiterhin. Zuerst rohes SQL (`psycopg` 3), danach derselbe Weg mit SQLAlchemy. Drei `DataSource`-/`RecommendationWriter`-Paare (CSV, psycopg, SQLAlchemy) erfüllen strukturell dieselben zwei Protocols.

Die Projektstruktur wurde dabei zu **Ports & Adapters** (Hexagonal) refaktoriert:

```
src/pricing/
├── domain.py · ports.py · main.py
├── core/       (builder.py, recommendations.py, app.py)
└── adapters/   (sources.py, writers.py, db.py, orm.py, ingest.py)
```

**Datenmodell:** 6 Tabellen (`products`, `customers`, `capacity`, `bookings`, `prices`, `price_recommendations`) in 3. Normalform, mit Constraints und Indizes — Details in [[Datenmodell|Datenmodell]].

**Voll-Lauf verifiziert:** 1,33 Mio. `capacity`-, 940k `bookings`-, 857k `customers`-Zeilen fehlerfrei per `COPY` geladen (~110 s). CSV- und Postgres-Pipeline liefern bei gepinntem `reference_date` **identische** Empfehlungen. ruff, mypy und pytest grün.

## Wichtigste Entscheidungen

**Rohes SQL (`psycopg` 3) vor SQLAlchemy.** Der Leitfaden verlangt es so, und der Grund trägt: SQL erst verstehen, dann die Abstraktion darüberlegen. Dadurch war beim ORM-Teil jederzeit klar, welches SQL erzeugt wird.

**Massen-Import per `COPY` statt `INSERT`/`executemany`.** Bei 2,4 Mio. Zeilen der einzig praktikable Weg — zeilenweises `INSERT` wäre um Größenordnungen langsamer.

**Idempotenz per `TRUNCATE … RESTART IDENTITY CASCADE` in *einer* Transaktion.** Voll-Reload statt inkrementellem Update: einfacher, alles-oder-nichts, und für einen täglichen Datenabzug völlig ausreichend. (Die elegantere Alternative wäre `INSERT … ON CONFLICT` — siehe [[Antworten (Stichwörter & Fragen)|Q&A]].)

**Nur das Passwort in `.env`, DSN per `make_conninfo`.** Korrektes Escaping statt String-Konkatenation — und der Rest der Verbindungsdaten ist kein Geheimnis.

**`row_factory=dict_row` + Spalten-Aliasing → `Cls(**row)`.** Die SELECT-Spalten werden per `AS` exakt auf die Dataclass-Feldnamen gemappt, sodass Domänenobjekte ohne manuelles Feld-Mapping entstehen. Möglich **nur**, weil psycopg native Typen liefert (`DATE` → `date`, `NUMERIC` → `Decimal`) — bei CSV ist alles ein String und dieser Trick unmöglich.

**FK-Auflösung per einmaligem Bulk-Lookup-Dict statt N Einzel-Queries.** `capacity_id` wird über den natürlichen Schlüssel `(product_id, calendar_date)` aufgelöst. Die Domänen-Objekte bleiben DB-agnostisch (sie kennen keine `capacity_id`), der Writer löst auf. N Einzel-Queries wären bei hunderttausenden Zeilen unbrauchbar langsam.

**Fehlende FK-Keys werden übersprungen und *gezählt* geloggt, statt hart abzubrechen.** Eine einzelne Inkonsistenz ist harmlos; die Anzahl im Log verrät dagegen strukturelle Fehler — etwa versehentlich kombinierte Adapter.

**`reference_date`-Spalte nachträglich aus dem Schema entfernt.** CSV- und Postgres-Ausgabe sollen strukturell identisch bleiben; den Wert künstlich zu rekonstruieren hätte die Vergleichbarkeit zerstört, die den ganzen Doppelweg erst beweisbar macht.

**Natürliche PKs für `products`/`customers`, Surrogat-PKs sonst.** Die Quell-IDs sind dort stabil und eindeutig; bei `bookings` ist die Quell-`id` das nachweislich nicht (~82k Zeilen mit doppelter `bkg_id`).

**Kein Index auf `bookings.status`.** Selektivität liegt bei 92 % „Firm" — ein Index würde kaum filtern und nur Schreiblast erzeugen.

**Protocols in den Kern gezogen (`ports.py`), nicht bei den Adaptern gelassen.** Damit zeigen die Abhängigkeiten konsequent nach innen: Adapter → Ports → Kern, nie umgekehrt. `git mv` erhielt dabei die Datei-History.

**`source_id`/`bkg_id` als `TEXT`.** Opake Quell-IDs, die nie arithmetisch benutzt werden — `TEXT` ist ehrlicher als `BIGINT` und passt zu den `str`-DTOs.

**`net_revenue` als `GENERATED … STORED`.** Die Datenbank berechnet `gross − discount` selbst; das `net_revenue` aus der Quelle dient nur noch der **Validierung** (Abweichungen werden gezählt).

## Gelernte Lektionen

- **Protocol-basierte DI zahlt sich messbar aus.** Zwei neue Adapter (psycopg, dann SQLAlchemy) **und** der komplette Struktur-Refactor liefen ohne eine einzige geänderte Zeile in `domain.py`, `core/builder.py`, `core/recommendations.py` oder `core/app.py`. Die Business-Logik kennt nur die Ports.
- **Zwei Läufe vergleichen braucht ein gepinntes Datum.** Sonst verfälscht die Zeit-Drift zwischen den Läufen den Vergleich — und man sucht den Fehler im Code statt in der Uhr.
- **`TRUNCATE … CASCADE` wirkt über Tabellengrenzen.** Abhängige Tabellen (`price_recommendations`) sind nach jedem Ingest automatisch leer und müssen neu geschrieben werden. Erwartetes Verhalten, aber beim ersten Mal überraschend.
- **Volltreue Zeilen-DTOs und schlanke Domain-Objekte haben unterschiedliche Anforderungen** an Parsing und Leerstring-Behandlung. Beide Ebenen bewusst getrennt zu halten verhindert, dass Ingest-Workarounds in den schlanken Pfad durchsickern.
- **Ein Re-Import ist unter mypy `strict` kein Export.** Nach dem Umzug von `CapacityRow` nach `domain.py` importierten Aufrufer weiter aus `pricing.sources`, das den Namen nur noch re-importierte → `attr-defined`-Fehler. Lehre: Nach „Symbol umziehen" immer mypy laufen lassen.
- **`py.typed` kann von `.gitignore` geschluckt werden.** Die Regel `*.typed` hatte den PEP-561-Marker versehentlich mit ausgeschlossen — Downstream-mypy hätte alle Typen ignoriert.
