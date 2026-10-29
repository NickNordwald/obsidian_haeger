**Claudians Register** identifizierter Lücken und lohnender Vertiefungsthemen rund um relationale Datenbanken, SQL und ORM, die über den offiziellen [[1.2 Relationale Datenbanken & ORM - Leitfaden|Leitfaden]] hinausgehen oder in der [[1.2 Relationale Datenbanken & ORM - Vertiefung|Vertiefung]] (noch) nicht behandelt sind. Übergeordnete Notiz: [[1.2 Relationale Datenbanken & ORM - Vertiefung|1]].

Für Nicks **eigene** Fragen/Stichworte gibt es ein separates Dokument: [[Stichwörter und Fragen|Stichwörter und Fragen]].

**Beantwortet** in: [[Antworten (Stichwörter & Fragen)|Antworten (Stichwörter & Fragen)]].

Diese Notiz ist ein **laufendes Register**: Claudian trägt hier proaktiv Lücken und lohnende Vertiefungsthemen ein, während wir das Kapitel durcharbeiten. Abgehakt = später in der Vertiefung ergänzt oder eigenständig gelernt.

Legende: 🧩 = alltägliche Syntax/Grundlagen, die man kennen sollte · 🚀 = über den Stoff hinaus (optional, aber lohnend).

---

# SQL

*→ [[Antworten (Stichwörter & Fragen)#SQL|Antworten zu diesem Abschnitt]]*

## Alltägliche Syntax-Lücken (im Vertiefungs-Abschnitt SQL bewusst noch offen gelassen)

- [x] 🧩 **`COALESCE(a, b, …)` / `NULLIF(a, b)`** — praktischer NULL-Umgang: `COALESCE` gibt den ersten Nicht-NULL-Wert zurück (z. B. Default statt NULL anzeigen), `NULLIF` macht aus einem bestimmten Wert wieder NULL. Passt direkt an den NULL-/dreiwertige-Logik-Abschnitt.
- [x] 🧩 **`INSERT … ON CONFLICT … DO UPDATE/DO NOTHING` (UPSERT)** — „einfügen, oder bei existierendem Schlüssel aktualisieren/ignorieren". Sehr Postgres-typisch und für die Übungsaufgabe relevant, um CSV-Daten **idempotent** (mehrfach ausführbar ohne Duplikate/Fehler) einzuspielen.
- [x] 🧩 **`INSERT/UPDATE/DELETE … RETURNING`** — gibt die gerade geschriebenen Zeilen (z. B. die neu vergebene `SERIAL`-ID) direkt zurück, ohne separate `SELECT`-Abfrage. Ebenfalls Postgres-Stärke.
- [x] 🧩 **`OFFSET`** (zusammen mit `LIMIT`) — Pagination/Blättern durch Ergebnisseiten. Wichtig zu wissen: bei großen Offsets ineffizient (Keyset-Pagination als Alternative → 🚀).
- [x] 🧩 **`CASE WHEN … THEN … ELSE … END`** — bedingte Ausdrücke direkt in `SELECT` (SQLs „if/else"), z. B. Kategorien/Buckets bilden.
- [x] 🧩 **Kommentar-Syntax** — `-- Zeilenkommentar` und `/* Blockkommentar */`.

## Über den Stoff hinaus (optional, aber lohnend)

- [x] 🚀 **Subqueries (Unterabfragen)** — eine `SELECT`-Abfrage innerhalb einer anderen (in `WHERE`, `FROM` oder `SELECT`), inkl. korrelierter Subqueries und `EXISTS`/`IN`.
- [x] 🚀 **CTEs / `WITH`-Klausel** — benannte Zwischenergebnisse für lesbarere, mehrstufige Abfragen; inkl. **rekursiver** CTEs (z. B. Hierarchien/Bäume abfragen).
- [x] 🚀 **Window Functions** (`OVER (PARTITION BY … ORDER BY …)`, `ROW_NUMBER`, `RANK`, `LAG`/`LEAD`, laufende Summen) — Aggregieren *ohne* die Zeilen zu Gruppen zu kollabieren. Sehr mächtig für Analytics/Data Engineering.
- [x] 🚀 **`EXPLAIN` / `EXPLAIN ANALYZE`** — den Ausführungsplan des Query Planners lesen, um zu verstehen, warum eine Abfrage schnell/langsam ist (direkte Brücke zum Stichwort *Indexes*).
- [x] 🚀 **String-/Datums-Funktionen** — `||` (Konkatenation), `LOWER`/`UPPER`/`TRIM`, `SUBSTRING`, `NOW()`, `AGE()`, `DATE_TRUNC`, `EXTRACT` — im Alltag ständig gebraucht.
- [x] 🚀 **Cast-Syntax** — `wert::TYP` bzw. `CAST(wert AS TYP)` für explizite Typumwandlung.

# Datenbankdesign

*→ [[Antworten (Stichwörter & Fragen)#Datenbankdesign|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **ER-Diagramm-Notationen** — Chen-Notation vs. Crow's-Foot- (IE-)Notation; wie man Kardinalitäten (1:1, 1:n, n:m) und optionale/obligatorische Beziehungen zeichnet. Nützlich, um Schemata visuell zu kommunizieren (Tools: dbdiagram.io, Mermaid `erDiagram`).
- [x] 🚀 **Natural- vs. Surrogate-Key-Debatte, vertieft** — konkrete Fälle, wann natürliche Schlüssel doch besser sind, Umgang mit sich ändernden fachlichen Schlüsseln, Nachteile zusammengesetzter FKs in Kindtabellen.
- [x] 🚀 **`SERIAL`/`IDENTITY` vs. `UUID` als Surrogatschlüssel** — Vor-/Nachteile (Kompaktheit & Index-Lokalität vs. globale Eindeutigkeit/verteilte Systeme); `UUIDv4` (zufällig) vs. `UUIDv7` (zeitsortiert) und warum zufällige UUIDs Index-Performance kosten können.
- [x] 🚀 **Schema-Migrationen & Versionierung** — wie Schemaänderungen im Team nachvollziehbar ausgerollt werden (Alembic für SQLAlchemy, Flyway/Liquibase generell). Berührt später den ORM-Abschnitt.
- [x] 🚀 **Weitere ER-Konzepte** — schwache Entitäten (existenzabhängig vom Elternobjekt), mehrwertige/abgeleitete Attribute, Generalisierung/Spezialisierung (Vererbung im Datenmodell und ihre Abbildungsstrategien: single/class/joined table).

# Normalisierung / Normalformen

*→ [[Antworten (Stichwörter & Fragen)#Normalisierung / Normalformen|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Boyce-Codd-Normalform (BCNF)** — strengere Fassung der 3NF: *jede* Determinante muss ein Kandidatenschlüssel sein. Wann 3NF ohne BCNF vorliegt (Beispiel mit überlappenden Kandidatenschlüsseln) und warum das selten vorkommt.
- [x] 🚀 **4NF & 5NF (nur konzeptionell)** — mehrwertige Abhängigkeiten (4NF) und Join-Abhängigkeiten (5NF). Laut Leitfaden Out of Scope, aber gut, den Begriff einordnen zu können.
- [x] 🚀 **Star- vs. Snowflake-Schema** — bewusst denormalisierte Modelle im Data Warehousing (Fakten- vs. Dimensionstabellen). Direkt relevant fürs Data Engineering; Gegenpol zur OLTP-Normalisierung.
- [x] 🚀 **Materialized Views & berechnete/generierte Spalten** — technische Mittel, um denormalisierte/aggregierte Daten konsistent vorzuhalten (`MATERIALIZED VIEW`, `GENERATED ALWAYS AS ... STORED`), statt Redundanz von Hand zu pflegen.

# Indexes

*→ [[Antworten (Stichwörter & Fragen)#Indexes|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Partielle & Ausdrucks-Indizes** — `CREATE INDEX ... WHERE bedingung` (nur Teilmenge indizieren, z. B. nur aktive Zeilen) und Indizes auf Ausdrücken (`LOWER(name)`) für case-insensitive Suche.
- [x] 🚀 **Covering Index / `INCLUDE`** — Index, der alle für die Abfrage nötigen Spalten enthält, sodass die Tabelle gar nicht mehr angefasst werden muss („Index-Only Scan").
- [x] 🚀 **`EXPLAIN`-Output lesen** — Scan-Typen (Seq/Index/Bitmap Heap Scan), Kosten- vs. Zeitangaben, `ANALYZE`/Statistiken des Planners; wie man erkennt, warum ein Index NICHT genutzt wird.
- [x] 🚀 **Index-Typen im Detail** — wann GIN/GiST/BRIN/Hash statt B-Tree; Trade-offs bei Größe & Schreiblast.
- [x] 🚀 **Clustered vs. non-clustered / Heap** — wie Postgres Tabellen physisch ablegt (Heap) vs. das clustered-index-Modell anderer DBs (z. B. MySQL/InnoDB, SQL Server); `CLUSTER`-Befehl.
- [x] 🚀 **B+-Tree im Detail** *(von Nick angefragt)* — interner Aufbau: Knoten/Blätter, Verzweigungsgrad (fan-out), warum Blätter verkettet sind (effiziente Bereichs-Scans), Balancierung bei Insert/Delete (Split/Merge), Höhe vs. Zeilenzahl, Unterschied B-Tree ↔ B+-Tree.

# Foreign Keys

*→ [[Antworten (Stichwörter & Fragen)#Foreign Keys|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **`DEFERRABLE` Constraints** — Fremdschlüssel-Prüfung ans Transaktionsende verschieben (`DEFERRABLE INITIALLY DEFERRED`); nötig z. B. bei zirkulären Verweisen oder Bulk-Reordering.
- [x] 🚀 **`MATCH FULL` / `MATCH SIMPLE`** — Verhalten zusammengesetzter Fremdschlüssel, wenn nur ein Teil der FK-Spalten NULL ist.
- [x] 🚀 **FK-Performance bei großen Löschungen** — warum fehlende Indizes auf FK-Spalten `ON DELETE CASCADE`/Elternlöschungen dramatisch verlangsamen; Locking-Verhalten.

# Beziehungstypen

*→ [[Antworten (Stichwörter & Fragen)#Beziehungstypen|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Assoziative Entität vs. reine Verknüpfungstabelle** — wann die Zwischentabelle eigene Attribute/eine eigene ID bekommt (z. B. `bookings` mit Menge, Preis, Zeitpunkt) und dadurch selbst zur vollwertigen Entität wird.
- [x] 🚀 **Optionalität / Teilnahme (Partizipation)** — Pflicht- vs. Kann-Beziehung: FK `NOT NULL` (obligatorisch) vs. FK nullable (optional); wie man „mindestens eine"-Regeln durchsetzt (nicht rein per FK).
- [x] 🚀 **Ternäre & höhergradige Beziehungen** — Beziehungen zwischen drei+ Entitäten und warum sie oft in mehrere binäre Beziehungen zerlegt werden.
- [x] 🚀 **Self-referential n:m** — Beziehung einer Entität zu sich selbst über eine Verknüpfungstabelle (z. B. „Produkt X passt zu Produkt Y", Freundschaften).

# Constraints

*→ [[Antworten (Stichwörter & Fragen)#Constraints|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **`EXCLUDE`-Constraints** — Postgres-Spezialität: verhindert, dass sich Zeilen nach einem Kriterium „überschneiden" (z. B. keine überlappenden Buchungszeiträume `tstzrange` per GiST). Mächtiger als `UNIQUE`.
- [x] 🚀 **Partielle UNIQUE-Constraints / Unique-Index mit `WHERE`** — Eindeutigkeit nur für eine Teilmenge (z. B. „nur ein aktiver Datensatz pro Nutzer").
- [x] 🚀 **`NOT VALID` + `VALIDATE CONSTRAINT`** — Constraints auf großen Produktivtabellen ohne langes Sperren nachrüsten (erst nur für neue Zeilen, dann Alt-Bestand validieren).
- [x] 🚀 **`DEFERRABLE` Constraints** *(auch bei Foreign Keys notiert)* — Prüfung ans Transaktionsende verschieben.
- [x] 🚀 **Domains** — wiederverwendbare, benannte Typen mit eingebautem CHECK (`CREATE DOMAIN positive_price AS NUMERIC(10,2) CHECK (VALUE >= 0)`).

# Joins und Unions

*→ [[Antworten (Stichwörter & Fragen)#Joins und Unions|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Join-Algorithmen des Planners** — Nested Loop, Hash Join, Merge Join: wann Postgres welchen wählt und wie man das im `EXPLAIN` sieht (verbindet Joins mit Indexes).
- [x] 🚀 **`USING` & natürliche Joins** — `JOIN ... USING (product_id)` als Kurzform bei gleichnamigen Spalten; `NATURAL JOIN` (und warum man es meist meidet).
- [x] 🚀 **`LATERAL` Joins** — Unterabfrage rechts darf auf Spalten der linken Zeile zugreifen (z. B. „die 3 neuesten Buchungen je Produkt", Top-N-per-Group).
- [x] 🚀 **`INTERSECT` / `EXCEPT`** — Mengenoperatoren neben UNION, vertieft mit Anwendungsfällen.
- [x] 🚀 **Semi-/Anti-Join-Varianten** — `EXISTS`/`NOT EXISTS` vs. `IN`/`NOT IN` vs. `LEFT JOIN ... IS NULL`; Performance- und NULL-Fallstricke (`NOT IN` mit NULL!).

# Transactions

*→ [[Antworten (Stichwörter & Fragen)#Transactions|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Pessimistisches vs. optimistisches Locking** — `SELECT ... FOR UPDATE`/`FOR SHARE` (Zeilen sperren) vs. Versionsspalte/`WHERE version = ...` (Konflikt erst beim Schreiben erkennen). Wann was.
- [x] 🚀 **Deadlocks** — wie sie entstehen (zwei Transaktionen sperren sich über Kreuz), wie Postgres sie erkennt/auflöst, wie man sie vermeidet (konsistente Sperrreihenfolge).
- [x] 🚀 **Lange Transaktionen & ihre Kosten** — warum offene Transaktionen VACUUM blockieren, Bloat verursachen und Sperren halten; „Transaktionen kurz halten".
- [x] 🚀 **Advisory Locks** — anwendungsdefinierte Sperren (`pg_advisory_lock`) für Koordination außerhalb des Zeilenmodells.
- [x] 🚀 **Verteilte Transaktionen / Two-Phase Commit (2PC)** — `PREPARE TRANSACTION`; warum man verteilte Transaktionen in Microservices meist meidet (stattdessen Saga-Pattern).

# Views

*→ [[Antworten (Stichwörter & Fragen)#Views|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **Updatable Views & `WITH CHECK OPTION`** — wann eine View beschreibbar ist, `INSTEAD OF`-Trigger für komplexe Views, `WITH CHECK OPTION` (verhindert INSERTs, die durch die View selbst nicht mehr sichtbar wären).
- [x] 🚀 **`REFRESH MATERIALIZED VIEW CONCURRENTLY`** — Aktualisieren ohne die View zu sperren (braucht einen Unique-Index); Strategien für den Refresh-Zeitpunkt (Cron/Job vs. Trigger).
- [x] 🚀 **Indizes auf Materialized Views** — Materialized Views können eigene Indizes tragen (normale Views nicht); Zusammenspiel mit Query-Performance.
- [x] 🚀 **View vs. CTE vs. Subquery vs. temporäre Tabelle** — wann welches Abstraktionsmittel; Performance-Implikationen (View wird „inlined", CTE in Postgres ≥12 auch).

# ACID

*→ [[Antworten (Stichwörter & Fragen)#ACID|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **CAP-Theorem & PACELC** — Consistency/Availability/Partition-Tolerance: warum man in verteilten Systemen bei Partition nur zwei haben kann; PACELC als Erweiterung (Latenz vs. Konsistenz auch ohne Partition).
- [x] 🚀 **Write-Ahead-Log (WAL) im Detail** — wie WAL Atomarität & Dauerhaftigkeit umsetzt; Checkpoints; Grundlage von Point-in-Time-Recovery und Replikation.
- [x] 🚀 **MVCC im Detail** — Zeilenversionen (xmin/xmax), Snapshots, warum tote Tupel entstehen → VACUUM/Autovacuum; Zusammenhang mit Isolationsstufen.
- [x] 🚀 **Serializable Snapshot Isolation (SSI)** — wie Postgres `SERIALIZABLE` ohne klassisches Sperren umsetzt; Serialization Failures + Retry-Pattern in der Anwendung.
- [x] 🚀 **Synchrone vs. asynchrone Replikation** — Auswirkung auf das „D" in verteilten Setups; wann Durability bei asynchroner Replikation gefährdet ist.

# SQLAlchemy und ORM in Python

*→ [[Antworten (Stichwörter & Fragen)#SQLAlchemy und ORM in Python|Antworten zu diesem Abschnitt]]*

- [x] 🚀 **N+1 lösen: Eager Loading Strategien** — `selectinload`, `joinedload` in der Vertiefung behandelt; offen bleibt: `subqueryload`, „wann welche" im Detail, N+1 im SQL-Echo/Log sicher erkennen.
- [x] 🚀 **Alembic (Schema-Migrationen)** — in der Vertiefung kurz eingeführt; offen: Migrationen tatsächlich autogenerieren, Up-/Downgrade praktisch durchspielen, Zusammenspiel mit `Base.metadata`.
- [x] 🚀 **Core vs. ORM in der Praxis** — wann man bewusst auf die Core-/Expression-Ebene oder rohes SQL (`text()`) heruntergeht; Hybrid-Ansätze. *(In der Vertiefung nur erwähnt.)*
- [x] 🚀 **Session-Lebenszyklus & Objektzustände** — transient/pending/persistent/detached + `flush` vs. `commit` + expire/refresh in der Vertiefung behandelt; offen: Sonderfälle (merge, `DetachedInstanceError` beheben, scoped_session).
- [x] 🚀 **Async SQLAlchemy** — `asyncio`-Variante (`AsyncSession`, `create_async_engine`) und wann sie sich lohnt.
- [x] 🚀 **Vererbungs-Mapping** — single table / joined table / concrete table inheritance (verbindet OOP-Vererbung aus 1.1 mit dem relationalen Modell).
- [x] 🚀 **ORM-Alternativen in Python** — Django ORM, Peewee, Tortoise (async), SQLModel (Pydantic+SQLAlchemy); Einordnung ggü. SQLAlchemy.
