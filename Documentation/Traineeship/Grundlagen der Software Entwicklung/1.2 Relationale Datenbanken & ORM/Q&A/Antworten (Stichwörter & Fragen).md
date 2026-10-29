---
tags:
  - documentation
type: q&a
date: 2026-07-23
---

Antworten auf die gesammelten Stichwörter und Fragen aus [[Stichwörter und Fragen|Stichwörter und Fragen]] (Nick) und [[Stichwörter Claudian|Stichwörter Claudian]]. Gleiche Gliederung wie das Register, damit man 1:1 gegenlesen kann. Kompakt gehalten (Nachschlage-Sheet, kein Lehrbuch). Übergeordnet: [[1.2 Relationale Datenbanken & ORM - Vertiefung|1.2 Vertiefung]].

## Nicks direkte Fragen
- **B+-Tree im Detail** → beantwortet unter [[#Indexes|Indexes]].
- **Star Schema** → beantwortet unter [[#Normalisierung / Normalformen|Normalisierung]] (Star vs. Snowflake).

---
# SQL

## Alltägliche Syntax

**`COALESCE(a, b, …)` / `NULLIF(a, b)`** — `COALESCE` gibt den **ersten Nicht-NULL**-Wert der Liste zurück (typisch: Default statt NULL, `COALESCE(rabatt, 0)`). `NULLIF(a, b)` gibt `NULL`, wenn `a = b`, sonst `a` (typisch: Division-durch-0 vermeiden, `x / NULLIF(y, 0)`).

**`INSERT … ON CONFLICT … DO UPDATE/DO NOTHING` (UPSERT)** — „einfügen, oder bei kollidierendem Schlüssel aktualisieren/ignorieren". `ON CONFLICT (product_id, calendar_date) DO UPDATE SET bookable_units = EXCLUDED.bookable_units`. `EXCLUDED` ist die *vorgeschlagene* (nicht eingefügte) Zeile. `DO NOTHING` überspringt Kollisionen. Macht CSV-Ingest **idempotent** — Alternative zu deinem `TRUNCATE`-Voll-Reload.

**`… RETURNING`** — `INSERT/UPDATE/DELETE … RETURNING id, *` liefert die gerade geschriebenen Zeilen direkt zurück (z. B. die neue IDENTITY-`id`), ohne separates `SELECT`. Spart einen Roundtrip; in ORMs die Grundlage, warum du nach `add`+`flush` sofort die `id` hast.

**`OFFSET` (mit `LIMIT`)** — `LIMIT 20 OFFSET 40` = Seite 3. **Falle:** großer Offset ist langsam, weil die DB die übersprungenen Zeilen trotzdem berechnet und wegwirft. Skalierbare Alternative: **Keyset-Pagination** (`WHERE id > :last ORDER BY id LIMIT 20`).

**`CASE WHEN … THEN … ELSE … END`** — SQLs if/else im `SELECT`: `CASE WHEN occupancy > 0.8 THEN 'hoch' WHEN occupancy > 0.4 THEN 'mittel' ELSE 'niedrig' END`. Für Buckets/Kategorien; auch in `ORDER BY`/Aggregaten nutzbar (`SUM(CASE WHEN status='Firm' THEN 1 ELSE 0 END)`).

**Kommentar-Syntax** — `-- bis Zeilenende` und `/* Block über mehrere Zeilen */`.

## Über den Stoff hinaus

**Subqueries** — `SELECT` in `SELECT`. Skalar (`WHERE preis > (SELECT avg(preis) …)`), in `FROM` (abgeleitete Tabelle), mit `IN`/`EXISTS`. **Korreliert** = die Unterabfrage referenziert die äußere Zeile (läuft pro Zeile, kann teuer sein). `EXISTS` bricht beim ersten Treffer ab.

**CTEs / `WITH`** — benannte Zwischenergebnisse für lesbare, mehrstufige Abfragen: `WITH firm AS (SELECT … WHERE status='Firm') SELECT … FROM firm`. **Rekursiv** (`WITH RECURSIVE`) für Hierarchien/Bäume: Basisfall `UNION ALL` rekursiver Fall (z. B. Org-Chart, Stückliste). Seit PG12 werden CTEs standardmäßig „inlined" (nicht mehr automatisch materialisiert).

**Window Functions** — aggregieren **ohne** Zeilen zu Gruppen zu kollabieren: `func() OVER (PARTITION BY produkt ORDER BY datum)`. `ROW_NUMBER`/`RANK`/`DENSE_RANK` (Ranglisten), `LAG`/`LEAD` (Vor-/Nachzeile, z. B. Differenz zum Vortag), laufende Summen (`SUM(x) OVER (ORDER BY …)`). Kernwerkzeug für Analytics/Data Engineering.

**`EXPLAIN` / `EXPLAIN ANALYZE`** — `EXPLAIN` zeigt den geschätzten **Ausführungsplan** (Scan-Typen, geschätzte Kosten/Zeilen); `EXPLAIN ANALYZE` führt die Query wirklich aus und zeigt **echte** Zeiten/Zeilen. Vergleich Schätzung↔Realität verrät veraltete Statistiken; direkte Brücke zu *Indexes*.

**String-/Datums-Funktionen** — `||` (Konkatenation), `LOWER`/`UPPER`/`TRIM`, `SUBSTRING`, `NOW()`, `AGE()`, `DATE_TRUNC('day', ts)` (auf Tag/Monat runden), `EXTRACT(YEAR FROM ts)`. Im Alltag ständig gebraucht.

**Cast-Syntax** — `wert::TYP` (Postgres-Kurzform) bzw. Standard-`CAST(wert AS TYP)`, z. B. `'2026-01-01'::date`, `preis::text`.

# Datenbankdesign

**ER-Diagramm-Notationen** — **Chen** (Entitäten = Rechtecke, Beziehungen = Rauten, Attribute = Ovale; akademisch) vs. **Crow's Foot / IE** (Linien mit „Krähenfuß"-Symbolen für Kardinalität; in der Praxis Standard). Kardinalitäten 1:1/1:n/n:m + Optionalität (Kreis = optional, Strich = obligatorisch). Tools: dbdiagram.io, Mermaid `erDiagram`.

**Natural- vs. Surrogate-Key, vertieft** — Surrogat (künstliche ID) ist **stabil** und schlank in Kind-FKs; natürlicher Schlüssel ist fachlich sprechend, aber **ändert sich evtl.** (dann kaskadieren Änderungen durch alle FKs). Zusammengesetzte natürliche PKs vererben sich als breite FKs in Kindtabellen → oft nimmt man dort lieber ein Surrogat. In deinem Projekt: `products`/`customers` natürlich (stabile fachliche IDs), `capacity`/`bookings` Surrogat.

**`SERIAL`/`IDENTITY` vs. `UUID`** — Sequenz-IDs sind **kompakt** und index-lokal (neue Zeilen landen am Baumende, wenig Page-Splits), aber lokal/erratbar. UUID ist **global eindeutig** (gut für verteilte Systeme/Merge ohne Kollision), aber größer. **`UUIDv4`** (zufällig) verteilt Inserts über den ganzen Index → Page-Splits, schlechte Cache-Lokalität; **`UUIDv7`** (zeitsortiert) behebt das weitgehend, indem der Zeitanteil vorn steht.

**Schema-Migrationen & Versionierung** — Schemaänderungen als versionierte, nachvollziehbare Skripte im VCS statt „von Hand am Prod". **Alembic** (SQLAlchemy, kann aus `Base.metadata` autogenerieren), **Flyway**/**Liquibase** (sprachneutral). Up-/Downgrade, ein „current revision"-Marker in der DB. Relevant für den Spike „ORM besitzt das Schema".

**Weitere ER-Konzepte** — **Schwache Entität** (existenzabhängig vom Eltern-Objekt, identifiziert über Eltern-PK + partieller Schlüssel). **Mehrwertige** Attribute (→ eigene Tabelle) / **abgeleitete** Attribute (berechnet, nicht gespeichert → generated column/View). **Generalisierung/Spezialisierung** (Vererbung im Datenmodell) → Abbildungsstrategien single/joined/concrete table (siehe SQLAlchemy-Vererbungs-Mapping unten).

# Normalisierung / Normalformen

**Boyce-Codd (BCNF)** — Verschärfung der 3NF: **jede Determinante muss Kandidatenschlüssel sein** (3NF erlaubt Ausnahmen für Prime-Attribute). Unterschied zeigt sich nur bei **überlappenden Kandidatenschlüsseln** — selten. Faustregel: 3NF sauber → meist schon BCNF.

**4NF & 5NF (konzeptionell)** — **4NF**: keine nicht-trivialen **mehrwertigen** Abhängigkeiten (zwei unabhängige 1:n in einer Tabelle → aufsplitten). **5NF**: keine **Join-Abhängigkeiten**, die sich nicht aus Kandidatenschlüsseln ergeben. Out of Scope, aber den Begriff einordnen können.

**Star- vs. Snowflake-Schema** *(Nicks Frage)* — bewusst **denormalisierte** Data-Warehouse-Modelle. **Star**: zentrale **Faktentabelle** (Kennzahlen + FKs, z. B. Verkäufe) umringt von **Dimensionstabellen** (Zeit, Produkt, Kunde) — flach, wenige Joins, schnell für OLAP. **Snowflake**: die Dimensionen sind zusätzlich **normalisiert** (Produkt → Kategorie → Abteilung als eigene Tabellen) — weniger Redundanz, aber mehr Joins. Gegenpol zur OLTP-Normalisierung: hier optimiert man auf **Lesen/Aggregieren**, nicht auf schlanke Schreibvorgänge.

**Materialized Views & generierte Spalten** — Mittel, um denormalisierte/aggregierte Daten **konsistent** vorzuhalten, statt Redundanz von Hand zu pflegen. `MATERIALIZED VIEW` speichert ein Query-Ergebnis physisch (muss aktualisiert werden). `GENERATED ALWAYS AS (…) STORED` (dein `net_revenue`!) berechnet eine Spalte automatisch aus anderen. Beide: „abgeleiteter Wert, aber die DB garantiert die Konsistenz".

# Indexes

**Partielle & Ausdrucks-Indizes** — Partiell: `CREATE INDEX … WHERE aktiv` indiziert nur eine **Teilmenge** (kleiner, schneller; z. B. nur offene Buchungen). Ausdruck: `CREATE INDEX … (LOWER(name))` für case-insensitive Suche (`WHERE LOWER(name)=…` nutzt ihn).

**Covering Index / `INCLUDE`** — ein Index, der **alle** von der Abfrage gebrauchten Spalten enthält (`CREATE INDEX … (a) INCLUDE (b, c)`), sodass die DB die Tabelle gar nicht mehr anfassen muss → **Index-Only Scan**.

**`EXPLAIN`-Output lesen** — Scan-Typen: **Seq Scan** (ganze Tabelle), **Index Scan** (Index → Tabelle), **Index Only Scan** (nur Index), **Bitmap Heap Scan** (viele Treffer, zuerst Bitmap sammeln). Achte auf `rows`-Schätzung vs. `actual rows` (weit auseinander = veraltete Statistiken → `ANALYZE`). Index wird **nicht** genutzt bei geringer Selektivität, Typ-Mismatch, oder Funktion auf der Spalte ohne Ausdrucks-Index.

**Index-Typen im Detail** — **B-Tree** (Default: Gleichheit, Bereich, Sortierung). **Hash** (nur Gleichheit). **GIN** (mehrwertige Werte: Arrays, `jsonb`, Volltext). **GiST** (geometrisch, Bereiche, Nearest-Neighbor). **BRIN** (riesige, physisch sortierte Tabellen — winzig, grob, z. B. Zeitreihen). Trade-off: mehr/größere Indizes = schnellere Reads, langsamere Writes + Platz.

**Clustered vs. non-clustered / Heap** — Postgres speichert Zeilen als **Heap** (ungeordnet); der PK ist ein **separater** Index, der auf Heap-Positionen zeigt. MySQL/InnoDB & SQL Server nutzen einen **Clustered Index** (die Tabelle *ist* physisch nach PK sortiert). `CLUSTER tabelle USING index` ordnet in Postgres einmalig physisch um (hält aber nicht automatisch nach).

**B+-Tree im Detail** *(Nicks Frage)* — balancierter Suchbaum, Grundlage fast aller DB-Indizes.
- **Innere Knoten** enthalten nur **Routing-Schlüssel + Zeiger** (keine Daten) → hoher **Verzweigungsgrad/fan-out** (hunderte Kinder pro Knoten). Dadurch ist der Baum **flach**: Millionen Zeilen in ~3–4 Ebenen → ~3–4 Seitenzugriffe pro Suche.
- **Blätter** enthalten die Schlüssel (+ Zeiger auf die Tabellenzeile) und sind **untereinander verkettet** (verkettete Liste) → **Bereichs-Scans** (`WHERE datum BETWEEN …`, `ORDER BY`) laufen einfach am Blattlevel entlang, ohne wieder hochzusteigen.
- **Balance** bei Insert/Delete über **Split** (voller Knoten teilt sich, Median wandert nach oben) und **Merge/Redistribute** (unterfüllter Knoten). So bleibt die Höhe für alle Blätter gleich.
- **B-Tree vs. B+-Tree:** klassischer B-Tree speichert Daten in **allen** Knoten; B+-Tree nur in den **Blättern** → höherer fan-out (innere Knoten passen mehr Schlüssel) und die verketteten Blätter machen Bereichs-Scans effizient. Deshalb nutzen DBs B+.

# Foreign Keys

**`DEFERRABLE` Constraints** — FK-Prüfung ans **Transaktionsende** verschieben (`… DEFERRABLE INITIALLY DEFERRED`). Nötig bei **zirkulären** Verweisen (A→B und B→A) oder Bulk-Reordering, wo die Referenz kurzzeitig „ins Leere" zeigt.

**`MATCH FULL` / `MATCH SIMPLE`** — Verhalten **zusammengesetzter** FKs bei teilweisem NULL. `MATCH SIMPLE` (Default): ist **irgendeine** FK-Spalte NULL, wird nicht geprüft. `MATCH FULL`: entweder **alle** FK-Spalten NULL oder **keine** — sonst Fehler.

**FK-Performance bei großen Löschungen** — fehlt ein **Index auf der FK-Spalte des Kindes**, muss Postgres beim Löschen/Ändern der Elternzeile die ganze Kindtabelle scannen (und sperrt dabei) → `ON DELETE CASCADE`/Elternlöschungen werden dramatisch langsam. Deshalb indizierst du in deinem Schema `bookings.product_id`, `bookings.cust_id`, `price_recommendations.capacity_id`.

# Beziehungstypen

**Assoziative Entität vs. reine Verknüpfungstabelle** — eine n:m-Zwischentabelle ist eine **reine Junction** (nur die zwei FKs). Bekommt sie **eigene Attribute** (Menge, Preis, Zeitpunkt) und/oder eine eigene ID, wird sie zur vollwertigen **assoziativen Entität** (z. B. `bookings` mit Nächten/Umsatz — kein bloßer Link mehr).

**Optionalität / Partizipation** — **Pflicht** vs. **Kann**: FK `NOT NULL` = jede Kindzeile *muss* ein Elternteil haben (obligatorisch); nullable FK = optional. „**Mindestens eine**"-Regeln (jede Rechnung braucht ≥1 Position) lassen sich **nicht** rein per FK erzwingen → Trigger, `DEFERRABLE` Constraint oder Applikationslogik.

**Ternäre & höhergradige Beziehungen** — Beziehung zwischen **drei+** Entitäten (z. B. Lieferant–Produkt–Projekt). Wird oft in mehrere **binäre** Beziehungen zerlegt, außer die Semantik verlangt echt die Dreier-Kombination (dann Junction mit drei FKs).

**Self-referential n:m** — Entität verweist über eine Junction auf **sich selbst**: `product_related(product_id, related_product_id)`, Freundschaften, „passt zu". Beide FKs zeigen auf dieselbe Tabelle.

# Constraints

**`EXCLUDE`-Constraints** — Postgres-Spezialität, mächtiger als `UNIQUE`: verhindert, dass sich Zeilen nach einem Kriterium **überschneiden**. Klassiker: keine überlappenden Buchungszeiträume — `EXCLUDE USING gist (raum WITH =, zeitraum WITH &&)` (`&&` = „überlappt", `tstzrange`).

**Partielle UNIQUE / Unique-Index mit `WHERE`** — Eindeutigkeit nur für eine **Teilmenge**: `CREATE UNIQUE INDEX … (user_id) WHERE aktiv` → „nur **ein aktiver** Datensatz pro Nutzer", inaktive dürfen mehrfach.

**`NOT VALID` + `VALIDATE CONSTRAINT`** — Constraint auf großer Produktivtabelle **ohne** langes Sperren nachrüsten: `ADD CONSTRAINT … NOT VALID` (gilt sofort für **neue** Zeilen, kein Full-Scan), später `VALIDATE CONSTRAINT` (prüft Altbestand mit schwächerem Lock).

**`DEFERRABLE` Constraints** — Prüfung ans Transaktionsende verschieben (siehe Foreign Keys; gilt auch für `UNIQUE`/CHECK, z. B. bei Bulk-Reorder).

**Domains** — wiederverwendbarer, benannter Typ mit eingebautem CHECK: `CREATE DOMAIN positive_price AS NUMERIC(10,2) CHECK (VALUE >= 0)`. Danach `preis positive_price` in vielen Tabellen — Regel einmal definiert.

# Joins und Unions

**Join-Algorithmen des Planners** — **Nested Loop** (für jede linke Zeile die rechte suchen; gut bei kleinen Mengen/Index). **Hash Join** (kleinere Seite in Hash-Tabelle, andere durchsuchen; gut bei großen Equi-Joins ohne passende Sortierung). **Merge Join** (beide Seiten sortiert, parallel durchlaufen; gut bei bereits sortierten/indizierten Daten). Der Planner wählt kostenbasiert — im `EXPLAIN` sichtbar.

**`USING` & natürliche Joins** — `JOIN … USING (product_id)` als Kurzform bei **gleichnamiger** Join-Spalte (erscheint dann nur einmal im Ergebnis). `NATURAL JOIN` joint automatisch über **alle** gleichnamigen Spalten — bequem, aber **fragil** (neue gleichnamige Spalte ändert stillschweigend die Semantik) → meist meiden.

**`LATERAL` Joins** — die rechte Unterabfrage darf auf Spalten der **linken** Zeile zugreifen (wie eine korrelierte Subquery, aber als Join): `… JOIN LATERAL (SELECT … WHERE b.product_id = p.id ORDER BY datum DESC LIMIT 3) …` = **Top-N pro Gruppe** (die 3 neuesten Buchungen je Produkt).

**`INTERSECT` / `EXCEPT`** — Mengenoperatoren neben `UNION`: `INTERSECT` = Zeilen, die in **beiden** Abfragen sind; `EXCEPT` = in der ersten, aber **nicht** in der zweiten (Differenz). Jeweils `ALL`-Variante behält Duplikate.

**Semi-/Anti-Join** — **Semi** = „existiert ein Treffer?" (`EXISTS`/`IN`, gibt linke Zeile 1× zurück). **Anti** = „existiert **kein** Treffer?" (`NOT EXISTS`, `LEFT JOIN … WHERE rechts IS NULL`). **Falle:** `NOT IN (subquery)` liefert **gar nichts**, sobald die Subquery **ein NULL** enthält (dreiwertige Logik) → `NOT EXISTS` ist robuster.

# Transactions

**Pessimistisches vs. optimistisches Locking** — Pessimistisch: Zeile beim Lesen **sperren** (`SELECT … FOR UPDATE`/`FOR SHARE`), andere warten — gut bei hoher Konkurrenz. Optimistisch: **nicht** sperren, sondern eine **Versionsspalte** mitführen und beim Schreiben prüfen (`UPDATE … WHERE version = :gelesen`); schlägt der Update fehl → Konflikt, retry. Gut bei seltenen Konflikten.

**Deadlocks** — zwei Transaktionen sperren Ressourcen **über Kreuz** (T1 hält A will B, T2 hält B will A). Postgres **erkennt** den Zyklus und **bricht eine** ab (Fehler → retry). Vermeiden: immer **konsistente Sperrreihenfolge**, Transaktionen kurz halten.

**Lange Transaktionen & ihre Kosten** — eine lange offene Transaktion hält einen alten **Snapshot** → **VACUUM** kann tote Tupel nicht aufräumen → **Bloat**; außerdem gehaltene Sperren blockieren andere. Regel: **Transaktionen kurz halten**, keine Nutzer-Wartezeiten in offener TX.

**Advisory Locks** — anwendungsdefinierte Sperren, die **nichts** mit Zeilen zu tun haben: `pg_advisory_lock(key)`. Für Koordination auf App-Ebene (z. B. „nur ein Cron-Job gleichzeitig"), die man selbst freigibt.

**Verteilte Transaktionen / 2PC** — `PREPARE TRANSACTION` = **Two-Phase Commit** über mehrere DBs (Phase 1: alle „bereit?", Phase 2: alle committen). Fragil (blockiert bei Ausfall des Koordinators) → in Microservices meist **gemieden**, stattdessen **Saga-Pattern** (Kette lokaler Transaktionen + Kompensationen).

# Views

**Updatable Views & `WITH CHECK OPTION`** — einfache Views (eine Tabelle, kein `GROUP BY`/`DISTINCT`) sind automatisch **beschreibbar**. `WITH CHECK OPTION` verhindert `INSERT`/`UPDATE`, deren Ergebnis durch die View selbst **nicht mehr sichtbar** wäre. Komplexe Views macht man über **`INSTEAD OF`-Trigger** schreibbar.

**`REFRESH MATERIALIZED VIEW CONCURRENTLY`** — aktualisiert eine Materialized View **ohne** sie zu sperren (Leser sehen währenddessen die alte Version). Braucht einen **Unique-Index** auf der View. Refresh-Strategie: per Cron/Job oder nach relevanten Änderungen.

**Indizes auf Materialized Views** — Materialized Views können **eigene Indizes** tragen (normale Views nicht, die sind nur gespeicherte Abfragen). Damit werden Abfragen *auf* der MV schnell — plus der Unique-Index, den `CONCURRENTLY` braucht.

**View vs. CTE vs. Subquery vs. temp. Tabelle** — **View**: benannte, wiederverwendbare Abfrage, kein Speicher, wird „inlined". **CTE**: Zwischenergebnis für *eine* Abfrage (PG12+ auch inlined, außer `MATERIALIZED`). **Subquery**: inline, anonym. **Temp-Tabelle**: **materialisiert**, session-lebt, kann indiziert werden — sinnvoll, wenn ein Zwischenergebnis **mehrfach** teuer genutzt wird.

# ACID

**CAP-Theorem & PACELC** — CAP: bei einer **Netzwerk-Partition (P)** kann ein verteiltes System nur **Consistency ODER Availability** wahren, nicht beides. **PACELC** erweitert: *Else* (ohne Partition) muss man zwischen **Latenz** und **Konsistenz** wählen. Ordnet ein, warum verteilte DBs bewusst Konsistenz aufweichen.

**Write-Ahead-Log (WAL)** — Änderungen werden **zuerst ins Log** geschrieben, bevor die Datenseiten geändert werden → garantiert **Atomarität & Dauerhaftigkeit** (nach Crash wird aus dem WAL wiederhergestellt). **Checkpoints** schreiben Datenseiten periodisch fest. Grundlage von **Point-in-Time-Recovery** und **Replikation**.

**MVCC** — Multiversion Concurrency Control: ein `UPDATE` erzeugt eine **neue Zeilenversion** (alte bleibt mit `xmin`/`xmax` sichtbar für alte Snapshots) → **Leser blockieren Schreiber nicht** und umgekehrt. Preis: **tote Tupel**, die **VACUUM/Autovacuum** aufräumt. Isolationsstufen sind Snapshot-Regeln darüber.

**Serializable Snapshot Isolation (SSI)** — Postgres' `SERIALIZABLE` ohne klassisches Sperren: es fährt Snapshot-Isolation und **erkennt** gefährliche Schreib-/Lese-Muster (Prädikat-Konflikte); im Konfliktfall **Serialization Failure** → die Anwendung muss die Transaktion **wiederholen** (Retry-Pattern).

**Synchrone vs. asynchrone Replikation** — Synchron: der Commit wartet, bis mindestens ein **Replikat** die Daten hat → kein Datenverlust bei Failover, aber höhere Latenz. Asynchron: Commit sofort, Replikat zieht nach → schneller, aber bei Primär-Ausfall können die **letzten Commits verloren** gehen (das „D" ist im Cluster-Kontext gefährdet).

# SQLAlchemy und ORM in Python

**N+1 lösen: Eager-Loading-Strategien** *(in Vertiefung behandelt)* — `selectinload` (lädt die Kinder mit **einem** separaten `IN`-Query — meist der Default-Griff), `joinedload` (ein **JOIN**, gut für 1:1/kleine 1:n), `subqueryload` (ältere Variante via Subquery, heute selten). N+1 erkennst du sicher am **SQL-Echo** (`create_engine(echo=True)`): dieselbe Query in einer Schleife = N+1.

**Alembic (Migrationen)** *(in Vertiefung eingeführt)* — `alembic revision --autogenerate` **difft** `Base.metadata` gegen die DB und schreibt ein Migrations-Skript mit `upgrade()`/`downgrade()`; `alembic upgrade head` spielt ein. Autogenerate ist ein **Vorschlag** — immer gegenlesen (erkennt z. B. Typänderungen nicht immer sauber).

**Core vs. ORM in der Praxis** — bewusst auf die **Core-/Expression**-Ebene (`select()`, `insert()`) oder **rohes SQL** (`text()`) heruntergehen bei: Massen-Operationen (Bulk-Insert/Update), komplexen analytischen Queries (Window/CTE), Performance-kritischen Pfaden, oder DB-Spezifika. Hybrid ist normal — dein Writer nutzt `text("TRUNCATE …")` mitten im ORM-Code.

**Session-Lebenszyklus & Objektzustände** *(in Vertiefung behandelt)* — **transient** (neu, nicht in Session) → **pending** (`add`ed, noch nicht geflusht) → **persistent** (in DB/Session) → **detached** (Session zu, Objekt lebt weiter). **`flush`** sendet SQL (macht pending → persistent), **`commit`** flusht **und** committet die Transaktion + expired die Objekte. Sonderfälle: `merge()` (detached zurück in eine Session), **`DetachedInstanceError`** (Zugriff auf lazy-Attribut nach Session-Ende → vorher laden oder `expire_on_commit=False`), `scoped_session` (Thread-lokale Session).

**Async SQLAlchemy** — `create_async_engine` + `AsyncSession` (mit `await session.execute(…)`), Treiber z. B. `asyncpg`/`psycopg` async. Lohnt bei **hoher IO-Nebenläufigkeit** (viele parallele DB-Requests, z. B. async Web-Backend); für Batch/ETL wie hier bringt es nichts.

**Vererbungs-Mapping** — OOP-Vererbung ins relationale Modell: **Single Table** (alle Klassen in **einer** Tabelle + Discriminator-Spalte; schnell, aber viele NULLs). **Joined Table** (Basis + je Kindklasse eine Tabelle, per Join verbunden; normalisiert, mehr Joins). **Concrete Table** (pro konkreter Klasse eine vollständige Tabelle; kaum genutzt). Verbindet die OOP-Vererbung aus 1.1 mit dem Datenmodell.

**ORM-Alternativen in Python** — **Django ORM** (an Django gekoppelt, „active record"-Stil, sehr produktiv), **Peewee** (leichtgewichtig), **Tortoise** (async-nativ, Django-ähnliche API), **SQLModel** (Pydantic + SQLAlchemy 2.0, ein Modell für DB *und* API-Validierung). SQLAlchemy bleibt das mächtigste/flexibelste (Core + ORM, „data mapper"-Stil).
