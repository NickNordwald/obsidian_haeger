---
tags:
  - documentation
type: q&a
date: 2026-07-22
---

Konsolidiertes Frage-Antwort-Dokument zum Kapitel **Relationale Datenbanken & ORM** — die wichtigen Stichwörter und Fragen aus der Umsetzung (von Nick *und* von Claudian), kompakt beantwortet als Wiederhol-/Nachschlage-Sheet. Kontext: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand]].

## Architektur (Ports & Adapters)

**Wie erfüllt ein Adapter das Protocol, ohne es zu importieren?**
Über **strukturelles Typing** (`typing.Protocol`): erfüllt eine Klasse die Methodensignaturen, gilt sie statisch als kompatibel — kein `implements`/Vererbung nötig. mypy prüft das an der Verwendungsstelle (z. B. beim Übergeben an `PricingApp`).

**Was hat die Protocol-basierte DI konkret gebracht?**
Drei DB-Wege (CSV, psycopg, SQLAlchemy) *und* den kompletten Struktur-Refactor gebaut, **ohne eine Zeile** in `domain.py`, `core/builder.py`, `core/recommendations.py` oder `core/app.py` zu ändern. Die Business-Logik kennt nur die Ports.

**Warum liegen die Ports im Kern (`ports.py`), nicht bei den Adaptern?**
Damit die Abhängigkeiten **nach innen** zeigen: Adapter → Ports/Kern, nie umgekehrt. Lägen die Protocols in `adapters/sources.py`, importierte der Kern (`app.py`) aus `adapters/` — die falsche Richtung für Hexagonal.

**Engine, Connection oder Session injizieren?**
Bei psycopg die offene **`Connection`**; bei SQLAlchemy die **`Engine`** (langlebiger Pool), und pro Arbeitseinheit intern eine kurze `Session`. Die Session ist die Unit of Work, nicht das App-Objekt.

## psycopg / rohes SQL

**Warum `COPY` statt `executemany`/`INSERT`?**
Massen-Import (2,4 Mio Zeilen). `COPY` ist der schnellste Bulk-Pfad; zeilenweises `INSERT` wäre um Größenordnungen langsamer.

**Was leistet `conn.transaction()` über `cursor()` hinaus?**
Eine explizite, selbst-beschreibende Atomizitätsgrenze (Commit/Rollback), die sich verschachtelt automatisch als `SAVEPOINT` verhält → komponierbar.

**`row_factory=dict_row` + `Cls(**row)` — warum geht das so glatt?**
SELECT-Spalten werden (nötigenfalls per `AS`) exakt auf die Dataclass-Feldnamen gemappt → Domänenobjekt ohne manuelles Feld-Mapping. Möglich nur, weil psycopg **native Typen** liefert (`DATE`→`date`, `NUMERIC`→`Decimal`).

**Wo lag der `fetchall()`-Overhead?**
`fetchall()` baut eine **komplette Zwischenliste** aller Zeilen; direkt über den Cursor iterieren (`for x in cur`) spart die. Den bereits im Client gepufferten Result spart es *nicht* (dafür bräuchte es einen Server-Side-Cursor).

**Ist binäres `COPY` schneller?**
`FROM STDIN WITH (FORMAT BINARY)` + `copy.set_types([...])` spart das Text-Encoding — aber der Flaschenhals ist eher der Python-Per-Row-Overhead, also **messen**. Binary ist strikt bei Typen (deshalb hätte `source_id`/`bkg_id` als `str` vs. altem `BIGINT` geknallt — mit ein Grund für C4 → `TEXT`).

## SQLAlchemy ORM

**`execute` vs. `scalars` vs. `scalar`?**
`execute(stmt)` → `Result` aus `Row`-Tupeln. `scalars(stmt)` → schält die **erste Spalte jeder** Zeile heraus (Plural → Entitäten). `.scalar()` → **erster Wert der einen** Zeile (Singular). Faustregel: **`scalars` bei *einer* Sache, `execute` bei *mehreren* Spalten.**

**Warum im `DataSource` `scalars`, im Writer `execute`?**
`load_capacity` macht `select(orm.Capacity)` (eine Entität) → `scalars` gibt direkt `Capacity`-Objekte. Der Writer macht `select(id, product_id, calendar_date)` (drei Spalten) → `execute(...).all()`, damit das ganze 3-Tupel ankommt (`for cid, pid, cal in rows`).

**`Mapped[X]` vs. `mapped_column(...)`?**
`Mapped[X]` gibt Python-Typ + „ist gemappt". `mapped_column(...)` nur für Extras nötig: `primary_key`, `ForeignKey`, Typ-Präzisierung (`Numeric`, `Double`, `BigInteger`), `Identity`, `server_default`.

**`insert(Model), [dicts]` vs. `session.add_all([...])`?**
Ersteres ist ein `executemany` (schlank, für viele Zeilen). `add_all` baut echte ORM-Objekte durch die Unit of Work (bequem bei Objektgraphen, aber langsamer bei Masse).

**Wozu `text()`?**
Notluke für rohes SQL, das sich nicht als ORM-Konstrukt lohnt (z. B. `TRUNCATE`). ORM und rohes SQL mischen ist erlaubt.

**Muss ich fürs Mappen das Schema anpassen?**
Nein. **Mappen** (Teil 2) beschreibt ein bestehendes Schema — kein `create_all`, nur die gebrauchten Spalten. Nur **besitzen** (Spike) erzeugt das Schema aus den Modellen.

**Wie drücke ich DDL-Features in Modellen aus (Spike)?**
`GENERATED ALWAYS AS IDENTITY` → `Identity(always=True)`; `GENERATED ... STORED` → `Computed("expr", persisted=True)`; `CHECK`/`UNIQUE`/Index → in `__table_args__`; `DEFAULT 'STG'` → `server_default=text("'STG'")` (SQL-quotiert!); `DEFAULT now()` → `server_default=func.now()`; `NOT NULL`/nullable → `Mapped[X]`/`Mapped[X | None]`.

## Datenmodell / Schema

**Warum `net_revenue GENERATED` und im Ingest weggelassen?**
Die DB berechnet es (`gross - discount`). Das Quell-`net_revenue` dient nur der **Validierung** (`_validate_net_revenue`: zählt Abweichungen der Roh-Invariante).

**`computed_at` — Historie oder Snapshot?**
**Snapshot-Zeitstempel.** Durch `TRUNCATE` + Insert in *einer* Transaktion tragen alle Zeilen dasselbe `now()` = Zeitpunkt der Snapshot-Erzeugung (keine Zeilen-Historie).

**Warum `source_id`/`bkg_id` auf `TEXT`?**
Opake Quell-IDs, nie arithmetisch genutzt → `TEXT` ist ehrlich und passt zu den `str`-DTOs (früher gemischt `BIGINT`/`TEXT`).

**Wie löst man den FK `capacity_id` auf?**
Über den natürlichen Schlüssel `(product_id, calendar_date)` → `capacity.id`. Zwei Wege: **App-Speicher** (1,33-Mio-Lookup-Dict) vs. **DB** (Staging + `INSERT … SELECT`-JOIN). Details: [[FK-Auflösung - App-Speicher vs. Datenbank|FK-Auflösung-Notiz]].

## Design-Entscheidungen

**Warum ist `Booking` *nicht* `frozen`?**
Bewusst mutabel — `status` (Firm/Cancelled/…) ändert sich fachlich; ein eingefrorenes Objekt würde das künstlich verbieten. (`Price`/`CapacityRow` sind frozen, weil reine Werte.)

**Ist das `Decimal` bei `recommended_adjustment_pct` echte Präzision?**
Nein, **kosmetisch**. Die Heuristik rechnet in `float`; das `Decimal`/`NUMERIC(10,6)` ist eine bewusste Rundung fürs stabile, reproduzierbare Ausgabeformat, keine Präzisionsgarantie.

**Wie wurde der doppelte `_compute()` gelöst?**
`PricingApp.run()` rechnet **einmal**, schreibt und gibt das Maximum zurück; `main` nutzt `run()`.

**Was passiert bei `occupancy`, wenn `total_capacity == 0`?**
Ein expliziter Guard wirft `ValueError` (statt stiller `ZeroDivisionError`); die Engine filtert solche Einheiten ohnehin vorher heraus.

## Struktur / Tooling

**`git mv` vs. löschen + neu anlegen?**
`git mv` erhält die History (Git erkennt Renames). Beim Struktur-Refactor wurden so alle 8 Module verschoben, ohne Historie zu verlieren.

**Warum war mypy nach dem `CapacityRow`-Umzug rot?**
`builder.py`/Test importierten `CapacityRow` weiter aus `pricing.sources`, das es nach dem Umzug nur **re-importierte**. Unter `strict` ist ein Re-Import **kein** Export → `attr-defined`-Fehler. Fix: direkt aus `pricing.domain` importieren. **Lehre:** nach „Symbol umziehen" immer mypy laufen lassen.

**Wozu `py.typed`?**
PEP-561-Marker, der das Paket als typisiert ausweist. Die `.gitignore`-Regel `*.typed` hatte ihn versehentlich geschluckt → Downstream-mypy hätte die Typen ignoriert.

## Verifikation / Testing

**Wie teste ich alle Pipelines von Hand?**
CSV (`python -m pricing.main`), Ingest, psycopg-Smoke, SQLAlchemy-Smoke. Bei **gepinntem `reference_date`** müssen alle „Höchste Empfehlung" identisch sein (Referenz: `RFJA3D` / `10.600000`, `price_recommendations` = 98 232); dazu Zähl-/Invarianten-SQL. Protokoll steht im Chat-Verlauf (Handtest).

**Warum ist Testing zurückgestellt?**
Eigenes Kapitel. Dort: Transaktions-Rollback-Fixture gegen die Docker-DB, Integrationstest `ingest → DataSource`, die ungetesteten Warn-Zweige. Die vorhandenen Unit-Tests (reine Kernlogik) blieben durch alle Umbauten grün — Beweis der Entkopplung.
