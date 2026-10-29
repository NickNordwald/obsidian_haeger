---
deck: "Traineeship::Grundlagen der Software Entwicklung::Relationale Datenbanken & ORM"
---

Karteikarten zu [[1.2 Relationale Datenbanken & ORM - Vertiefung|1]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::Relationale Datenbanken & ORM`. In Anki werden die Karten außerdem automatisch mit dem Vault-Namen und `ObsidianAnkiSync` getaggt (kein eigener Obsidian-Tag nötig). Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

# Fortschritt

- [x] RDBMS insbesondere PostgreSQL
- [x] SQL
- [x] Datenbankdesign
- [x] Normalisierung / Normalformen
- [x] Indexes
- [x] Foreign Keys
- [x] Beziehungstypen (One-to-One, One-to-Many, Many-to-Many)
- [x] Constraints
- [x] Joins und Unions
- [x] Transactions
- [x] Views
- [x] ACID
- [x] SQLAlchemy und ORM in Python

---

# RDBMS insbesondere PostgreSQL

<!-- basicblock-start oid="Obs4gJ7wF8AoQn6Q31h4rJhI" -->
Was ist der Unterschied zwischen einer Datenbank, einem DBMS und einem RDBMS?::
Datenbank (database) = die eigentliche, strukturierte Sammlung von Daten (die Tabellen und ihr Inhalt). DBMS = die Software, die Datenbanken verwaltet (nimmt Anfragen entgegen, sorgt für Nebenläufigkeit, Persistenz, Zugriffsrechte, Backups; z. B. PostgreSQL, MySQL, SQLite). RDBMS = ein DBMS, das dem relationalen Modell folgt, d. h. Daten in Tabellen (Relationen) hält, die über Werte miteinander in Beziehung stehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAWY0V6DcXXzjEVFn9AgRZ" -->
Was ist die Kernidee des relationalen Modells (Codd, 1970)?::
Daten werden als Relationen (Tabellen) dargestellt, und Beziehungen zwischen Daten werden über WERTE hergestellt — nicht über Zeiger oder Speicheradressen. Eine Zeile verweist auf eine andere, indem sie deren Schlüssel-Wert (Fremdschlüssel) speichert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKZEYciLBGDlC2xQztSMkm" -->
Übersetze die formalen Begriffe des relationalen Modells in ihre Praxis-Entsprechungen: Relation, Tupel, Attribut, Domäne.::
Relation → Tabelle. Tupel → Zeile (row). Attribut → Spalte (column). Domäne → der Datentyp/erlaubte Wertebereich einer Spalte (z. B. `NUMERIC(10,2)`, `TEXT`, `DATE`). Zusätzlich: Grad (degree) = Anzahl Spalten, Kardinalität = Anzahl Zeilen.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsBRY7ZIErIpUkWSl4p5QiK" -->
Im relationalen Modell werden Beziehungen über ==Werte== hergestellt (nicht über Zeiger/Speicheradressen), und jede Zeile ist über ihren ==Primärschlüssel== eindeutig identifizierbar.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsywmnnKzohV0lSnqyiW2f6" -->
Warum ist die Reihenfolge von Zeilen in einer relationalen Tabelle logisch bedeutungslos, und welche Konsequenz hat das beim Abfragen?::
Weil eine Relation formal eine MENGE von Tupeln ist — Mengen haben keine inhärente Reihenfolge. Konsequenz: Man verlässt sich nie darauf, dass „die Zeile an dritter Stelle steht". Will man eine bestimmte Reihenfolge, fordert man sie beim Abfragen explizit mit `ORDER BY` an.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspSsE9l1zSO3vjcYho6N3L" -->
Nenne drei konkrete Grenzen der rein datei-gestützten (z. B. CSV-)Datenhaltung, die ein RDBMS als Infrastruktur löst.::
1. Nebenläufigkeit: Bei gleichzeitigem Schreiben mehrerer Prozesse kann die Datei korrupt werden — es fehlt eingebaute Sperr-/Koordinationslogik.<br>2. Suchen/Filtern: Eine Teilmenge zu finden bedeutet, die ganze Datei einzulesen und in Python zu durchiterieren (langsam, speicherhungrig).<br>3. Integrität: Nichts verhindert ungültige Daten (negativer Preis, fehlende ID, Verweis auf nicht existierendes Produkt); die Regeln leben nur im Anwendungscode.<br>(Auch: Beziehungen zwischen mehreren Dateien konsistent zu halten ist Handarbeit.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7cpuVbyCtPAjgPFUjy8vi" -->
Wie unterscheidet sich eine relationale von einer nicht-relationalen (NoSQL) Datenbank? (Schema, Beziehungen, Konsistenz)::
Relational: festes, vorab definiertes Schema (schema-on-write), Beziehungen sind Kernkonzept (Foreign Keys, Joins), starke Konsistenz über ACID, standardisierte Abfragesprache SQL. NoSQL: flexibles/dynamisches Schema (schema-on-read), Beziehungen oft in der Anwendung gelöst bzw. Daten dupliziert/verschachtelt, häufig eventual consistency (Fokus auf Verfügbarkeit/horizontale Skalierung), produktspezifische Abfragesprachen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUMphWDB97h3e3jeBq8qJV" -->
Nenne die vier gängigen Familien von NoSQL-Datenbanken mit je einem Beispiel und typischem Einsatz.::
1. Document Stores (JSON-artige Dokumente, flexibles Schema) — MongoDB, für heterogene/verschachtelte Daten.<br>2. Key-Value Stores (Schlüssel→Wert, sehr schnell) — Redis, für Caching/Sessions/Zähler.<br>3. Wide-Column Stores (Spaltenfamilien) — Cassandra, für sehr große verteilte Schreiblasten.<br>4. Graph-Datenbanken (Knoten + Kanten) — Neo4j, für stark vernetzte Daten (soziale Netze, Empfehlungen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvC1rCN4lUUcfdvHg3VkY8" -->
Gibt es „die bessere" Wahl zwischen relational und NoSQL? Wonach entscheidet man?::
Nein, es gibt kein pauschales „besser" — es ist eine Frage des Anwendungsfalls. Strukturierte Daten mit klaren Beziehungen und Integritätsanforderungen (wie Produkte, Preise, Buchungen) sprechen für relational. NoSQL wählt man bewusst dort, wo dessen Stärken (flexibles Schema, extreme horizontale Skalierung, sehr hohe Schreiblast) tatsächlich gebraucht werden — und nimmt dafür schwächere Konsistenz-/Beziehungsgarantien in Kauf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKZLwMSIokMe6YKIjk2ajM" -->
Was ist PostgreSQL, und warum ist es eine der beliebtesten Datenbanklösungen? Nenne mehrere Gründe.::
PostgreSQL ist ein freies, quelloffenes, sehr ausgereiftes RDBMS (35+ Jahre). Gründe für die Beliebtheit:<br>1. Open Source & kostenlos, kein Vendor-Lock-in.<br>2. Sehr hohe SQL-Standardkonformität (vorhersehbares Verhalten, übertragbares Wissen).<br>3. Voll ACID-konform & zuverlässig — auch für geschäftskritische Daten.<br>4. Objekt-relational & enorm erweiterbar (eigene Typen/Operatoren, Extensions wie pgvector, PostGIS).<br>5. Reicher Funktionsumfang out of the box (JSONB, Arrays, UUID, Volltextsuche, Window Functions, viele Index-Typen).<br>6. Nebenläufigkeit ohne unnötiges Blockieren dank MVCC.<br>7. Starkes Ökosystem (Python-Anbindung, Managed-Angebote aller Clouds).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTrMfA35PrYReSiBHkOrUH" -->
Was bedeutet MVCC (Multi-Version Concurrency Control) in PostgreSQL, und warum ist es vorteilhaft?::
Bei MVCC blockieren Lesende keine Schreibenden und umgekehrt — jede Transaktion sieht einen konsistenten Snapshot der Daten. Vorteil: PostgreSQL bleibt auch unter hoher paralleler Last performant, ohne dass Leseoperationen ständig auf Schreiboperationen warten müssen (ein wesentlicher Grund für seinen guten Ruf bei Nebenläufigkeit).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQFEWvdiMlKUys8mw3IOUY" -->
Warum speichert man einen Geldbetrag in einer Datenbank als `NUMERIC`/`DECIMAL` und nicht als `FLOAT`/`REAL`?::
Weil Fließkommatypen (`FLOAT`/`REAL`) Werte wie `19.99` nicht exakt darstellen können und Rundungsfehler verursachen. Für Geldbeträge nimmt man immer einen exakten Dezimaltyp (`NUMERIC`/`DECIMAL`), damit Beträge und Summen präzise bleiben.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsE9umKyRsxwHMtnlDDBrBb" -->
`NOT NULL`, `DEFAULT` und `PRIMARY KEY` sind bereits ==Constraints== — Regeln, die das RDBMS selbst durchsetzt, sodass sie für ==jeden== Schreibzugriff gelten (nicht nur, wenn der Anwendungscode brav mitspielt).
<!-- clozeblock-end -->

# SQL

<!-- basicblock-start oid="Obs4Y69C487avpsXTctMSlta" -->
Was bedeutet es, dass SQL eine „deklarative" Sprache ist, und wie unterscheidet sich das von imperativem Code (z. B. Python)?::
Deklarativ heißt: Man beschreibt nur das WAS (welches Ergebnis man haben will, z. B. „alle Produkte mit booked_units > 0, sortiert nach Preis"), nicht das WIE. Wie das Ergebnis effizient beschafft wird (welcher Index, welche Reihenfolge), entscheidet der Query Planner des RDBMS selbst. Imperativ (Python) beschreibt dagegen Schritt für Schritt den Weg (Datei öffnen, iterieren, Bedingung prüfen, sammeln). Deshalb kann die Datenbank Abfragen frei optimieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0WsUDaGukuYj1tCfCaZ1N" -->
Nenne die fünf Teilsprachen von SQL (Abkürzung, Zweck, je ein Beispielbefehl).::
1. DDL (Data Definition Language) — Struktur/Schema definieren: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.<br>2. DML (Data Manipulation Language) — Dateninhalt ändern: `INSERT`, `UPDATE`, `DELETE`.<br>3. DQL (Data Query Language) — Daten abfragen: `SELECT`.<br>4. DCL (Data Control Language) — Rechte: `GRANT`, `REVOKE`.<br>5. TCL (Transaction Control Language) — Transaktionen: `BEGIN`, `COMMIT`, `ROLLBACK`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrFDLDW2inFAFTczIAReuI" -->
Was ist der Unterschied zwischen `DELETE`, `TRUNCATE` und `DROP` (bezogen auf eine Tabelle)?::
`DELETE FROM t` (DML) entfernt Zeilen einzeln, kann mit `WHERE` gefiltert werden, ist innerhalb einer Transaktion rückgängig machbar und löst Trigger aus. `TRUNCATE t` (DDL) leert die ganze Tabelle in einem Rutsch (schneller, kein `WHERE`), lässt die Tabellenstruktur aber bestehen. `DROP TABLE t` (DDL) entfernt die Tabelle samt Struktur — danach existiert sie nicht mehr.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsI0etONCuqw0xVrnkamPmJ" -->
Warum verwendet man für Geldbeträge `NUMERIC`/`DECIMAL` und nicht `REAL`/`DOUBLE PRECISION`, und welchen Typ nimmt man für automatisch hochzählende IDs?::
`NUMERIC(p, s)`/`DECIMAL(p, s)` sind exakte Dezimaltypen ohne Rundungsfehler — Pflicht für Geld. Fließkommatypen (`REAL`, `DOUBLE PRECISION`) sind nur Näherungen (für Mess-/Wissenschaftswerte), nicht für Beträge. Für automatisch hochzählende IDs nutzt man `SERIAL`/`BIGSERIAL` (Auto-Inkrement).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseTwsljTW7SddKEYOYqJpv" -->
Was ist beim `INSERT` mit mehreren Zeilen die Syntax, und welche Gefahr besteht bei `UPDATE`/`DELETE`?::
Mehrere Zeilen: `INSERT INTO t (spalten...) VALUES (...), (...), (...);` — mehrere Wertetupel durch Komma getrennt. Gefahr: `UPDATE`/`DELETE` OHNE `WHERE` betrifft ALLE Zeilen der Tabelle (`DELETE FROM t;` leert die ganze Tabelle). Faustregel: erst mit `SELECT ... WHERE ...` prüfen, welche Zeilen betroffen wären, bevor man dieselbe Bedingung in `UPDATE`/`DELETE` übernimmt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs57kDKbCS5vC9jx0TAvEDa" -->
Nenne die wichtigsten Klauseln einer `SELECT`-Abfrage in ihrer geschriebenen Reihenfolge und was jede tut.::
`SELECT` (welche Spalten / Projektion), `FROM` (aus welcher Tabelle), `WHERE` (Zeilen-Filter vor Aggregation), `GROUP BY` (Zeilen zu Gruppen zusammenfassen), `HAVING` (Gruppen-Filter nach Aggregation), `ORDER BY` (Sortierung), `LIMIT` (nur die ersten N Zeilen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLwJMqQzXGhUP4CSHCJwqr" -->
Wofür stehen `DISTINCT` und `AS` in einer `SELECT`-Abfrage?::
`DISTINCT` entfernt doppelte Werte/Zeilen aus dem Ergebnis (`SELECT DISTINCT calendar_date FROM ...`). `AS` vergibt einen Alias, d. h. benennt eine Spalte oder Tabelle im Ergebnis um (`SELECT base_price AS preis`) — nützlich für lesbarere Ergebnisse und berechnete Spalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMblxw6Or0ROIKW98MTIDs" -->
Nenne die gängigen `WHERE`-Operatoren für Vergleich, Bereich, Mengenzugehörigkeit und Textmuster.::
Vergleich: `=`, `<>` (ungleich), `<`, `<=`, `>`, `>=`; Logik: `AND`, `OR`, `NOT`. Bereich: `x BETWEEN a AND b`. Mengenzugehörigkeit: `x IN (a, b, c)`. Textmuster: `LIKE 'P-%'` (`%` = beliebig viele Zeichen, `_` = genau eines), `ILIKE` für case-insensitive.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTjtGHSQdZvCEC2ORALEB1" -->
Was bedeutet `NULL` in SQL, und warum liefert `WHERE spalte = NULL` nie ein Ergebnis?::
`NULL` bedeutet „unbekannt / kein Wert" — nicht `0` und nicht der leere String. Jeder Vergleich MIT `NULL` (z. B. `spalte = NULL`) ergibt nicht `TRUE`/`FALSE`, sondern `UNKNOWN` (dreiwertige Logik). Da `WHERE` nur Zeilen durchlässt, deren Bedingung echt `TRUE` ist, kommt nie eine Zeile durch. Korrekt fragt man mit `IS NULL` bzw. `IS NOT NULL`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obsf6L3jdtonHFtVH8rrDHis" -->
Wegen SQLs ==dreiwertiger Logik== (TRUE/FALSE/UNKNOWN) darf man nie mit `= NULL` vergleichen, sondern nutzt die Operatoren ==IS NULL== bzw. ==IS NOT NULL==.
<!-- clozeblock-end -->

<!-- basicblock-start oid="Obs2nq5h6ogHxrG17CyQUyqz" -->
Nenne die fünf gängigen Aggregatfunktionen und was sie tun.::
`COUNT(*)` (Anzahl Zeilen), `SUM(spalte)` (Summe), `AVG(spalte)` (Durchschnitt), `MIN(spalte)` (kleinster Wert), `MAX(spalte)` (größter Wert). Ohne `GROUP BY` verdichten sie die gesamte Ergebnismenge auf eine Zahl; mit `GROUP BY` werden sie pro Gruppe berechnet.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obskeo2fQKcFUuC2tebvNDcX" -->
Welche Regel gilt für Spalten in der `SELECT`-Liste, sobald `GROUP BY` verwendet wird?::
Jede Spalte in der `SELECT`-Liste muss entweder in der `GROUP BY`-Klausel stehen ODER in einer Aggregatfunktion (`SUM`, `AVG`, …) stecken. Sonst wüsste die Datenbank nicht, welchen der vielen Werte einer Gruppe sie zeigen soll — PostgreSQL wirft in dem Fall einen Fehler.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmLitFZtymvHdMvOANu7oZ" -->
Was ist der Unterschied zwischen `WHERE` und `HAVING`?::
`WHERE` filtert einzelne Zeilen, BEVOR gruppiert/aggregiert wird. `HAVING` filtert ganze Gruppen, NACHDEM aggregiert wurde — und kann sich deshalb auf Aggregatwerte beziehen (z. B. `HAVING AVG(base_price) > 25`). Bedingungen auf Aggregatwerten gehören daher in `HAVING`, nicht in `WHERE`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsN1nOayb89IDXqKABz4Bqq" -->
Nenne die logische Ausführungsreihenfolge der `SELECT`-Klauseln und erkläre, warum man einen Alias aus `SELECT` in `ORDER BY`, aber nicht in `WHERE` verwenden kann.::
Logische Reihenfolge: 1. `FROM`, 2. `WHERE`, 3. `GROUP BY`, 4. `HAVING`, 5. `SELECT`, 6. `ORDER BY`, 7. `LIMIT`. Ein in `SELECT` (Schritt 5) definierter Alias existiert erst ab diesem Schritt. `WHERE` (Schritt 2) läuft davor — der Alias ist dort logisch noch unbekannt (Fehler). `ORDER BY` (Schritt 6) läuft nach `SELECT` — dort darf der Alias verwendet werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsjgb9cAevbQ4clR6mGU1TS" -->
Womit ändert man das Schema einer bestehenden Tabelle nachträglich, z. B. eine Spalte hinzufügen oder entfernen?::
Mit `ALTER TABLE`: `ALTER TABLE t ADD COLUMN currency TEXT NOT NULL DEFAULT 'EUR';` fügt eine Spalte hinzu, `ALTER TABLE t DROP COLUMN currency;` entfernt sie.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsueiNLaELBDrPZsPvaQuVk" -->
Wie redet man interaktiv mit PostgreSQL, und wofür stehen die psql-Meta-Befehle `\dt` und `\d tabelle`?::
Über das mitgelieferte Terminal-Tool `psql`. Meta-Befehle beginnen mit Backslash (kein Semikolon): `\l` (Datenbanken auflisten), `\c name` (zu einer DB verbinden), `\dt` (alle Tabellen auflisten), `\d tabelle` (Struktur einer Tabelle: Spalten, Typen, Constraints), `\q` (beenden).
<!-- basicblock-end -->

# Datenbankdesign

<!-- basicblock-start oid="ObsG57be0O1HWTnIynS629eL" -->
Was ist Datenbankdesign (Datenmodellierung), und warum lohnt sich Sorgfalt dabei?::
Der Schritt, in dem man festlegt, welche Tabellen es gibt, welche Spalten sie haben und wie sie verknüpft sind — bevor die erste Zeile Daten hineinkommt. Sorgfalt lohnt sich, weil ein gutes Schema Abfragen einfach macht, Daten konsistent hält und erweiterbar bleibt, während ein schlechtes Schema später zu widersprüchlichen Daten, komplizierten Abfragen und schmerzhaften Migrationen führt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDGE0BYoCpJNbyrKE7kkgl" -->
Nenne die drei Modellierungsebenen des Datenbankdesigns und was auf jeder entsteht.::
1. Konzeptionell: Welche fachlichen Dinge/Beziehungen gibt es? → ER-Diagramm, technologieunabhängig.<br>2. Logisch: Welche Tabellen, Spalten, Schlüssel, Beziehungen? → relationales Schema, noch DBMS-neutral.<br>3. Physisch: Wie konkret in diesem RDBMS? → `CREATE TABLE`-Statements mit konkreten Datentypen und Indizes. Merksatz: erst fachlich verstehen, dann Tabellen ableiten, dann physisch optimieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDJ2ikmhQvEF2ob6wZg27H" -->
Was sind Entität, Attribut und Beziehung im ER-Modell, und wozu werden sie im relationalen Schema jeweils?::
Entität = fachliches „Ding", über das man Daten speichert → wird zu einer Tabelle (z. B. Produkt). Attribut = Eigenschaft einer Entität → wird zu einer Spalte (z. B. base_price). Beziehung = fachliche Verbindung zwischen Entitäten → wird über einen Fremdschlüssel umgesetzt (z. B. „ein Produkt hat viele Buchungen").
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0XQOSuWqjjM2p1VnUIigz" -->
Welche Heuristik hilft, aus einer fachlichen Beschreibung ein ER-Modell zu gewinnen?::
Substantive sind Kandidaten für Entitäten/Attribute, Verben sind Kandidaten für Beziehungen. Beispiel: „Ein Kunde bucht ein Produkt an einem Tag" → Entitäten Kunde und Produkt, Beziehung „bucht".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsonWPOLwpZAYjtCGUU6QAj" -->
Erkläre den Unterschied zwischen Superschlüssel, Kandidatenschlüssel und Primärschlüssel.::
Superschlüssel = jede Spaltenkombination, die eine Zeile eindeutig identifiziert (auch mit überflüssigen Spalten). Kandidatenschlüssel = ein minimaler Superschlüssel (keine Spalte lässt sich weglassen, ohne die Eindeutigkeit zu verlieren). Primärschlüssel = der eine Kandidatenschlüssel, den man als offizielle Identität der Zeile auswählt (automatisch NOT NULL und eindeutig). Übrige Kandidatenschlüssel = Alternativschlüssel (in SQL als UNIQUE).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVeg4gqhthZ8lFLY4Ze31j" -->
Was ist ein zusammengesetzter Primärschlüssel (composite key)? Nenne das Beispiel aus der Übungsaufgabe.::
Ein Primärschlüssel, der aus mehreren Spalten gemeinsam besteht, weil erst deren Kombination eine Zeile eindeutig identifiziert. Beispiel: In der tagesabhängigen Buchungslage identifiziert nicht `product_id` allein eine Zeile, sondern erst die Kombination `(product_id, calendar_date)` — dasselbe Produkt taucht an mehreren Tagen auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs85lgFYXtw0WjBXHvhaINi" -->
Was ist der Unterschied zwischen einem natürlichen und einem künstlichen (Surrogat-)Schlüssel? Nenne je einen Vor- und Nachteil.::
Natürlicher Schlüssel = aus fachlich vorhandenen Daten (z. B. ISBN, oder `(product_id, calendar_date)`); Vorteil: keine Zusatzspalte, fachlich aussagekräftig; Nachteil: ändert sich die Fachlichkeit, muss der Schlüssel überall mitgezogen werden. Surrogatschlüssel = bedeutungslose künstliche ID (z. B. `SERIAL`/`UUID`); Vorteil: stabil, kompakt, einfache Joins; Nachteil: trägt keine Bedeutung — man braucht zusätzlich einen `UNIQUE`-Constraint auf den fachlichen Feldern, um echte Duplikate zu verhindern.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsI5cgeu18ASG5I6HFI5q1o" -->
Nutzt man einen ==Surrogatschlüssel== (z. B. `id BIGSERIAL PRIMARY KEY`), sollte man die fachliche Eindeutigkeit zusätzlich mit einem ==UNIQUE==-Constraint auf den fachlichen Spalten absichern, da die künstliche ID selbst keine Duplikate verhindert.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsMXShW8UczEgR9KTosQMz6" -->
Warum ist es ein Designfehler, `product_name` und `category` in jeder Zeile der tagesabhängigen `priceable_units`-Tabelle zu speichern, und wie behebt man das?::
Weil diese Stammdaten dann bei jedem Tag redundant wiederholt werden: Ein Umbenennen erfordert ein Update in vielen Zeilen, und vergisst man eine, entstehen widersprüchliche Daten. Behebung: die eine Entität in zwei aufteilen — Produkt-Stammdaten einmal in `products`, die tagesabhängige Buchungslage separat in `priceable_units` mit einem Fremdschlüssel `product_id` auf `products`. So steht jede Information nur an einer Stelle (Grundgedanke der Normalisierung).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3L3MVKV7PdEk9OKgMBVPm" -->
Nenne gängige Namenskonventionen für Tabellen und Spalten in PostgreSQL.::
`snake_case` (klein, mit Unterstrich) für Tabellen- und Spaltennamen (Postgres faltet unquotete Namen ohnehin auf Kleinschreibung); Tabellennamen einheitlich im Plural (`products`) oder konsistent im Singular; Fremdschlüsselspalten als `<tabelle_singular>_id` (z. B. `product_id`); Primärschlüssel schlicht `id`; keine reservierten Wörter als Namen (`order`, `user`, `date` → besser `orders`, `users`, `calendar_date`).
<!-- basicblock-end -->

# Normalisierung / Normalformen

<!-- basicblock-start oid="Obs7w4pfIKb4MMszNYRFOvCJ" -->
Welche drei Anomalien verhindert Normalisierung? Beschreibe jede kurz.::
1. Update-Anomalie: Eine redundant gespeicherte Information muss in vielen Zeilen geändert werden — vergisst man eine, widersprechen sich die Daten.<br>2. Insert-Anomalie: Man kann eine Information nicht anlegen, weil dafür andere (noch nicht vorhandene) Daten nötig wären (z. B. ein Produkt ohne buchbaren Tag).<br>3. Delete-Anomalie: Löscht man eine Zeile, verschwinden ungewollt auch andere Informationen, die nur in dieser Zeile existierten (z. B. Stammdaten).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9dP6LIbmHW6YHgIx8eNi1" -->
Was bedeutet die funktionale Abhängigkeit `A → B`?::
„A bestimmt B": Kennt man den Wert von A, ist der Wert von B dadurch eindeutig festgelegt. Beispiel: `product_id → product_name` (die Produkt-ID bestimmt eindeutig den Namen). Normalisierung dreht sich um die Frage, ob jedes Nicht-Schlüssel-Attribut wirklich (nur) vom ganzen Primärschlüssel abhängt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQxY0rdID1n2vG1RSVWyYy" -->
Formuliere die Kurzregel von 1NF, 2NF und 3NF.::
1NF: alle Werte atomar — keine Listen/Wiederholgruppen in einer Zelle. 2NF: 1NF UND jedes Nicht-Schlüssel-Attribut hängt vom GANZEN (zusammengesetzten) Primärschlüssel ab, nicht nur von einem Teil (keine partiellen Abhängigkeiten). 3NF: 2NF UND kein Nicht-Schlüssel-Attribut hängt von einem ANDEREN Nicht-Schlüssel-Attribut ab (keine transitiven Abhängigkeiten).
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs9ad8WWZ74MKLJBWOQnroE" -->
Eselsbrücke für die 3NF: Jedes Nicht-Schlüssel-Attribut hängt vom ==Schlüssel== ab (1NF), vom ==ganzen== Schlüssel (2NF) und von ==nichts als== dem Schlüssel (3NF).
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsUGZP35AcqA1YiqbjsJC44" -->
Was ist der Unterschied zwischen einer partiellen und einer transitiven Abhängigkeit, und welche Normalform adressiert jeweils welche?::
Partielle Abhängigkeit: ein Nicht-Schlüssel-Attribut hängt nur von einem TEIL eines zusammengesetzten Schlüssels ab (z. B. `product_name` hängt nur von `product_id`, nicht von `(product_id, calendar_date)`) → von 2NF adressiert. Transitive Abhängigkeit: ein Nicht-Schlüssel-Attribut hängt über ein anderes Nicht-Schlüssel-Attribut vom Schlüssel ab (`id → category → category_tax_rate`) → von 3NF adressiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4xxbkAcwo8QeG02gkYZA2" -->
Warum wird 2NF erst bei einem zusammengesetzten Primärschlüssel relevant?::
Weil eine partielle Abhängigkeit nur entstehen kann, wenn ein Attribut von einem TEIL des Schlüssels abhängt. Besteht der Primärschlüssel nur aus einer einzigen Spalte, gibt es keinen „Teil", von dem etwas partiell abhängen könnte — jede 1NF-Tabelle mit einspaltigem Schlüssel ist automatisch in 2NF.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs09OvpALIOcE0A3EfasRrj" -->
Welche Normalformen gibt es, und welche ist in der Praxis der Standard?::
1NF, 2NF, 3NF, Boyce-Codd-Normalform (BCNF, strengere 3NF), sowie 4NF, 5NF und 6NF. Praxisstandard und meist völlig ausreichend ist die 3NF (auch von der Übungsaufgabe gefordert). BCNF wird gelegentlich angestrebt; NF4+ behandeln seltene Spezialfälle und sind hier Out of Scope.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs80KzJg9Liz7suiviELkZu" -->
Warum ist es nicht immer sinnvoll, maximal zu normalisieren? Was ist Denormalisierung, wann nutzt man sie und was kostet sie?::
Höhere Normalisierung bedeutet mehr Tabellen und damit mehr Joins beim Lesen (Performance-/Komplexitätskosten). Denormalisierung = das bewusste Wiedereinführen von Redundanz, um Lesezugriffe zu beschleunigen — erst wenn ein gemessenes Performance-Problem es rechtfertigt. Szenarien: leselastige Analytics/Data-Warehouses (Star Schema), teure häufig gelesene Aggregate, Caching selten geänderter Werte. Preis: Man handelt sich die Update-Anomalie wieder ein und muss die Redundanz selbst konsistent halten.
<!-- basicblock-end -->

# Indexes

<!-- basicblock-start oid="ObsAnNbt3kVw4zSe6iHSS4WB" -->
Was ist ein Sequential Scan, und welches Laufzeitverhalten (O-Notation) hat er?::
Ein Sequential Scan (Full Table Scan) bedeutet, dass die Datenbank ohne passenden Index jede einzelne Zeile der Tabelle liest und die Bedingung prüft, um eine Abfrage zu beantworten. Die Laufzeit wächst dabei linear mit der Tabellengröße — O(n). Bei kleinen Tabellen egal, bei Millionen Zeilen sehr teuer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUs211vwJBEODg3v1GWPNf" -->
Was ist ein Index, und welche Analogie beschreibt ihn treffend?::
Ein Index ist eine zusätzliche, sortierte Datenstruktur neben der Tabelle, die gezieltes Nachschlagen statt „alle Zeilen durchsuchen" ermöglicht. Analogie: das Stichwortverzeichnis am Ende eines Buches — man blättert nicht das ganze Buch durch, sondern schlägt den Begriff im sortierten Index nach und springt direkt zur richtigen Seite.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6Hp4xlonbaFeEXOt6MY2B" -->
Welche Datenstruktur nutzt ein Standard-Index in PostgreSQL, und warum ist die Suche dadurch nur noch logarithmisch (O(log n))?::
Einen B-Tree (genauer B+-Baum) — eine balancierte Baumstruktur mit sortiert abgelegten Werten. Statt linear zu suchen, halbiert die Datenbank bei jedem Schritt den Suchraum (wie beim Nachschlagen im Wörterbuch), sodass die Zahl der Schritte nur logarithmisch mit der Tabellengröße wächst (z. B. ~30 Schritte für 1 Mrd. Zeilen statt 1 Mrd. Prüfungen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqLz47oFto4YXqk14h3ym5" -->
Welche Indizes legt PostgreSQL automatisch an, und für welche Spalten muss man oft selbst einen anlegen?::
Automatisch: für jeden `PRIMARY KEY` und jeden `UNIQUE`-Constraint wird im Hintergrund ein Index erzeugt. Selbst anlegen muss man ihn oft für Fremdschlüssel-Spalten — für die legt Postgres KEINEN Index automatisch an, obwohl sie häufig in Joins/Filtern genutzt werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRTevL1WfNGNhfhLy65KXh" -->
Nenne die drei wesentlichen negativen Effekte von Indizes.::
1. Langsamere Schreibzugriffe: Bei jedem INSERT/UPDATE (der indizierten Spalte)/DELETE muss jeder Index der Tabelle mitgepflegt werden.<br>2. Speicherplatz: Jeder Index ist eine eigene, auf der Platte gespeicherte Struktur.<br>3. Wartung/Overhead: Indizes können aufblähen (bloat), und der Query Planner muss bei jeder Abfrage entscheiden, ob sich ein Index lohnt. Deshalb: Indizes gezielt setzen, nicht wahllos.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg2dwU8V7QZJh2iniY6cPM" -->
Für welche Spalten lohnt sich ein Index — und für welche kaum? (Stichwort Selektivität)::
Lohnend für Spalten, nach denen häufig gefiltert (`WHERE`), verknüpft (`JOIN`) oder sortiert (`ORDER BY`) wird UND die viele unterschiedliche Werte haben (hohe Selektivität). Kaum sinnvoll auf Spalten mit sehr wenigen verschiedenen Werten (z. B. ein boolean, das bei 90 % der Zeilen `true` ist) — dort müsste ohnehin fast alles gelesen werden, und die DB wählt oft bewusst den Sequential Scan.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrabBp3EcWob3sF6HYgi6S" -->
Was besagt die Leftmost-Prefix-Regel bei einem zusammengesetzten Index `(product_id, calendar_date)`?::
Der Index kann nur für Abfragen genutzt werden, die (mindestens) mit der ersten Spalte beginnen. `WHERE product_id = 5` ✅ und `WHERE product_id = 5 AND calendar_date = '...'` ✅ nutzen ihn; `WHERE calendar_date = '...'` (nur zweite Spalte) ❌ kann ihn nicht sinnvoll nutzen. Faustregel: die am häufigsten/selektivsten gefilterte Spalte nach vorne.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslGiM7I02OrTgHwy7YTDOM" -->
Mit welchem Befehl prüfst du, ob eine Abfrage tatsächlich einen Index nutzt, und woran erkennst du es im Output?::
Mit `EXPLAIN` bzw. `EXPLAIN ANALYZE` vor der Abfrage. Im Ausführungsplan steht dann entweder `Seq Scan on <tabelle>` (kein/ungenutzter Index) oder `Index Scan using <indexname>` (Index greift). So ersetzt man Performance-Vermutungen durch Messung.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObslQARneSw83ODlTa8TsPXx" -->
Ein Index tauscht schnellere ==Lesezugriffe== gegen langsamere ==Schreibzugriffe== (jeder INSERT/UPDATE/DELETE muss alle Indizes mitpflegen) und zusätzlichen ==Speicherplatz== ein.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsNQ2XtCbkU0EGJgGR5BS6f" -->
Nenne neben dem B-Tree zwei weitere PostgreSQL-Index-Typen und wofür sie gedacht sind.::
Z. B.: Hash (nur exakte Gleichheit `=`), GIN (Inhalt-von-Suchen: Volltext, JSONB, Arrays), GiST (geometrische/räumliche Daten, PostGIS), BRIN (sehr große, physisch sortierte Tabellen wie Zeitreihen — winzig und grob). Der B-Tree bleibt der Standard für Gleichheits- und Bereichsvergleiche.
<!-- basicblock-end -->

# Foreign Keys

<!-- basicblock-start oid="ObsV7cm57EfW21bKKhh68ZA6" -->
Was ist ein Fremdschlüssel (Foreign Key), und welche zwei Rollen (Tabellen) sind an ihm beteiligt?::
Ein Fremdschlüssel ist eine Spalte (oder Spaltenkombination), deren Werte auf den Primärschlüssel einer anderen Tabelle verweisen — er macht aus zwei Tabellen eine Beziehung. Beteiligt sind die Kindtabelle (child/referencing), die den FK enthält, und die Elterntabelle (parent/referenced), auf deren Primärschlüssel verwiesen wird. Beispiel: `priceable_units.product_id` (Kind) verweist auf `products.id` (Eltern).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsf3T3krUEd2eQVFvMRv603" -->
Was bedeutet referenzielle Integrität, und welche zwei Fehlerklassen verhindert ein Fremdschlüssel?::
Referenzielle Integrität = die Datenbank stellt sicher, dass ein Fremdschlüssel immer entweder auf eine tatsächlich existierende Elternzeile zeigt oder NULL ist (keine „verwaisten" Verweise). Verhindert: (1) einen INSERT/UPDATE mit ungültigem Verweis (auf ein nicht existierendes Elternteil) und (2) das Löschen/Ändern eines Elternteils, auf das noch Kinder verweisen. Beides wird auf Infrastrukturebene durchgesetzt, egal welcher Code schreibt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs25ZN0GEaBRehhFxOFAXNR" -->
Was bewirken die referenziellen Aktionen `ON DELETE CASCADE`, `SET NULL` und `RESTRICT`/`NO ACTION`?::
`NO ACTION`/`RESTRICT` (Default): Löschen/Ändern des Elternteils wird abgelehnt, solange noch Kinder verweisen (RESTRICT prüft sofort, NO ACTION am Statement-/Transaktionsende). `CASCADE`: Die Kind-Zeilen werden mitgelöscht bzw. mitgeändert. `SET NULL`: Der Fremdschlüssel der Kinder wird auf NULL gesetzt. (Zusätzlich gibt es `SET DEFAULT`.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss8OIYIeApJUz6RCmEkwPO" -->
Warum ist `ON DELETE CASCADE` mit Vorsicht zu genießen?::
Weil ein einziges `DELETE` am Elternteil eine ganze Kette von Kind-, Enkel- … Zeilen mitreißen kann. Man sollte es nur dort einsetzen, wo das Kind wirklich existenzabhängig vom Elternteil ist (z. B. „Bestellposition gehört zu Bestellung"). Oft ist das absichtliche `RESTRICT`/`NO ACTION` (erst die Kinder aufräumen müssen) die sicherere Wahl.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs5fVbP3eMTE8u4nE1WPX4d" -->
PostgreSQL legt für einen Fremdschlüssel ==keinen== Index automatisch an (anders als für PRIMARY KEY / UNIQUE) — daher sollte man FK-Spalten meist ==selbst indizieren==.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsGAr3ihArTBvtvGFVWqIzh" -->
Warum sollte man Fremdschlüsselspalten in der Regel selbst indizieren? Nenne beide Gründe.::
1. Joins über den FK (`... JOIN products ON products.id = priceable_units.product_id`) profitieren stark von einem Index auf der FK-Spalte. 2. Beim Löschen/Ändern eines Elternteils muss die DB prüfen, ob noch Kinder existieren — ohne Index ist das ein Sequential Scan der Kindtabelle bei jedem solchen Vorgang. PostgreSQL legt diesen Index nicht automatisch an.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4xn7RD724vqo16i2WYzwV" -->
Wozu dient ein selbstreferenzierender Fremdschlüssel?::
Ein FK, der auf dieselbe Tabelle zeigt — der klassische Weg, Hierarchien abzubilden. Beispiel: eine `categories`-Tabelle mit `parent_id BIGINT REFERENCES categories(id)`, um Ober-/Unterkategorien darzustellen.
<!-- basicblock-end -->

# Beziehungstypen (One-to-One, One-to-Many, Many-to-Many)

<!-- basicblock-start oid="ObsAwo1bB20NgUhYGKbHh6g0" -->
Was bedeutet die „Kardinalität" einer Beziehung, und welche drei Grundformen gibt es?::
Kardinalität beschreibt, wie viele Zeilen der einen Seite mit wie vielen der anderen zusammenhängen. Drei Grundformen: 1:1 (one-to-one, eine A gehört zu höchstens einer B und umgekehrt, z. B. Nutzer↔Profil), 1:n (one-to-many, eine A zu vielen B, aber jede B zu genau einer A, z. B. Produkt↔Buchungslagen), n:m (many-to-many, viele A zu vielen B und umgekehrt, z. B. Produkt↔Tag).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssSdLrlN1FTEB56NlRsKpE" -->
Auf welcher Seite steht der Fremdschlüssel bei einer 1:n-Beziehung, und warum kann es nicht andersherum sein?::
Der Fremdschlüssel steht auf der „many"-Seite und zeigt auf den Primärschlüssel der „one"-Seite (z. B. `priceable_units.product_id` → `products.id`). Andersherum ginge nicht, weil eine FK-Spalte pro Zeile nur EINEN Wert halten kann — eine Liste von Kind-IDs in der „one"-Zeile würde die 1NF (atomare Werte) verletzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsv1GtrgFgpHmWHrbcbOJMv" -->
Wie macht man aus einer 1:n- technisch eine 1:1-Beziehung, und wann nutzt man 1:1 überhaupt?::
Man nutzt denselben Fremdschlüssel wie bei 1:n, fügt aber zusätzlich einen `UNIQUE`-Constraint auf der FK-Spalte hinzu — dann kann kein Elternteil mehr als ein Kind haben. Sinnvoll z. B., um selten genutzte/optionale Attribute auszulagern, sensible Daten in eine Tabelle mit strengeren Rechten zu trennen oder sehr große Spalten nicht bei jedem Zugriff mitzuladen. 1:1 ist eher selten — oft ist eine gemeinsame Tabelle die richtige Wahl.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6ogat92vu1kJ84o7CDN77" -->
Warum lässt sich eine n:m-Beziehung nicht mit einem einzelnen Fremdschlüssel abbilden, und wie löst man es stattdessen?::
Weil eine einzelne FK-Spalte nur einen Wert halten kann, man aber „viele auf beiden Seiten" bräuchte. Lösung: eine dritte Tabelle — die Verknüpfungstabelle (junction/join/association table) — mit je einem Fremdschlüssel auf beide Seiten. Sie zerlegt das n:m in zwei 1:n-Beziehungen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslzkSUs3lCfZ7HjhaRJUdF" -->
Welche Rolle spielt der zusammengesetzte Primärschlüssel `(product_id, tag_id)` in einer Verknüpfungstabelle, und wann bekommt sie eigene Attribute?::
Der zusammengesetzte PK stellt sicher, dass dieselbe Zuordnung nicht doppelt entstehen kann, und dient als Identität der Zeile. Eigene Attribute bekommt die Verknüpfungstabelle, wenn die Beziehung selbst Daten trägt (z. B. `assigned_at`, Menge, Preis) — dann wird aus der reinen Zuordnung eine eigenständige (assoziative) Entität.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObszEFAuyabwwbTJxDqp0kow" -->
Eine n:m-Beziehung wird über eine ==Verknüpfungstabelle== abgebildet, die je einen Fremdschlüssel auf beide Seiten enthält und das n:m damit in ==zwei 1:n==-Beziehungen zerlegt.
<!-- clozeblock-end -->

<!-- basicblock-start oid="Obssv80wCsZsaP45Y0nILYTe" -->
Wie fragt man „alle Produkte mit Tag 'Aktiv'" aus einem n:m-Modell (products, tags, product_tags) ab?::
Man „hangelt" sich über die Verknüpfungstabelle mit zwei Joins: `SELECT p.name FROM products p JOIN product_tags pt ON pt.product_id = p.id JOIN tags t ON t.id = pt.tag_id WHERE t.name = 'Aktiv';`. Das Muster ist immer: von der einen Entität über die Zwischentabelle zur anderen.
<!-- basicblock-end -->

# Constraints

<!-- basicblock-start oid="ObsXe8J7OV6N4ZRhLqjkggwC" -->
Was ist ein Constraint, und was ist der zentrale Vorteil gegenüber einer reinen Prüfung im Anwendungscode?::
Ein Constraint ist eine Regel über die erlaubten Werte in einer Tabelle, die das RDBMS selbst durchsetzt — bei Verletzung wird der INSERT/UPDATE mit einem Fehler abgelehnt. Zentraler Vorteil: Die Regel gilt auf Infrastrukturebene für JEDEN Schreibpfad (Anwendung, Migrationsskript, manuelles psql, andere App), während eine `if`-Prüfung in einer Python-Funktion nur greift, wenn der Code auch durch genau diese Funktion läuft. Ein Constraint kann niemand umgehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAdhG17K2Lk0zZalJZUqkt" -->
Nenne die fünf wichtigsten Constraint-Typen und was jeder erzwingt.::
`NOT NULL` (Spalte muss einen Wert haben), `UNIQUE` (keine zwei Zeilen mit gleichem Wert), `PRIMARY KEY` (NOT NULL UND UNIQUE zugleich — Identität der Zeile), `FOREIGN KEY` (Wert muss auf eine existierende Zeile einer anderen Tabelle verweisen), `CHECK` (ein frei formulierter boolescher Ausdruck muss erfüllt sein). (Dazu `DEFAULT` als Standardwert, streng genommen kein Constraint.)
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsUx9PqWC0TjaK8Ao9brghM" -->
Ein `PRIMARY KEY` ist die Kombination der Constraints ==NOT NULL== und ==UNIQUE==, ausgezeichnet als offizielle Identität der Zeile (mit automatischem Index).
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsgpC43SO2Xjgc7O4TvwUSJ" -->
Wofür ist ein `CHECK`-Constraint da, und worauf kann er sich beziehen — worauf nicht?::
`CHECK` erlaubt einen beliebigen booleschen Ausdruck über die Spaltenwerte, um fachliche Invarianten durchzusetzen (z. B. `CHECK (base_price >= 0)` oder mehrspaltig `CHECK (booked_units <= bookable_units)`). Er kann sich nur auf die EIGENE Zeile beziehen — nicht auf andere Zeilen oder Tabellen (dafür bräuchte man Trigger).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslAZJEE0OVHiGRCtRCBka3" -->
Warum lässt `CHECK (base_price >= 0)` einen NULL-Wert durch, und wie verhindert man das?::
Weil ein CHECK als erfüllt gilt, wenn er TRUE ODER UNKNOWN ergibt. Ein Vergleich mit NULL (`NULL >= 0`) ergibt UNKNOWN — also besteht die Zeile den Check. Verhindern durch einen zusätzlichen `NOT NULL`-Constraint auf der Spalte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCpWSM7jtAnRBE53xmvJxi" -->
Wann muss ein Constraint als Tabellen-Constraint statt als Spalten-Constraint geschrieben werden?::
Sobald sich der Constraint über MEHRERE Spalten erstreckt (z. B. `UNIQUE (product_id, calendar_date)` oder `CHECK (booked_units <= bookable_units)`). Ein Spalten-Constraint steht direkt hinter einer Spaltendefinition und kann sich nur auf diese eine Spalte beziehen; mehrspaltige Regeln müssen als eigene Zeile (Tabellen-Constraint) stehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXMabIaGwS4kNQkxVFnUwk" -->
Warum gehört eine wichtige Invariante idealerweise sowohl in die Datenbank als auch in die Anwendung?::
Die Datenbank (Constraint) ist die letzte Verteidigungslinie: zentral, unumgehbar, selbstdokumentierend, fail-fast — ungültige Daten kommen gar nicht erst hinein. Die Anwendung prüft zusätzlich früh, um dem Nutzer schnelle, freundliche Fehlermeldungen zu geben, bevor überhaupt die DB kontaktiert wird. Beides zusammen = frühe UX-Fehler plus garantiertes Sicherheitsnetz.
<!-- basicblock-end -->

# Joins und Unions

<!-- basicblock-start oid="ObsBYc6xZHiIMcKkPLhrWIiH" -->
Was kombiniert ein JOIN, und was ist der Unterschied zu einem UNION?::
Ein JOIN kombiniert Zeilen aus zwei (oder mehr) Tabellen anhand einer Verknüpfungsbedingung (meist FK = PK) und fügt sie NEBENEINANDER zusammen — mehr Spalten („horizontal"). Ein UNION stapelt die Ergebnisse zweier SELECT-Abfragen UNTEREINANDER — mehr Zeilen („vertikal"). Merksatz: JOIN = mehr Spalten, UNION = mehr Zeilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss3HGp3lxdGLqIVShlmuVS" -->
Erkläre den Unterschied zwischen INNER JOIN und LEFT JOIN und nenne je ein Beispielszenario.::
INNER JOIN liefert nur Zeilenkombinationen mit Treffer auf BEIDEN Seiten (Schnittmenge) — Zeilen ohne Partner fallen weg. Szenario: „alle buchbaren Einheiten MIT Produktnamen" (Produkte ohne Buchungslage sind egal). LEFT (OUTER) JOIN liefert ALLE Zeilen der linken Tabelle, dazu passende rechte (sonst NULL). Szenario: „alle Produkte auflisten, auch die OHNE Buchungslage" (z. B. um Lücken zu finden).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs80w1W9J337DvepbYib40I" -->
Wie findest du mit einem LEFT JOIN gezielt die linken Zeilen, die keinen Partner rechts haben (Anti-Join)?::
Man macht einen LEFT JOIN und filtert dann auf `WHERE rechte_tabelle.<spalte> IS NULL` (typischerweise die id/PK-Spalte der rechten Tabelle). Beispiel: `... FROM products p LEFT JOIN priceable_units pu ON pu.product_id = p.id WHERE pu.id IS NULL;` liefert Produkte ganz ohne Buchungslage.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs16iih6wwW5Ie6I15cbarn" -->
Was ist ein CROSS JOIN, und wie entsteht er oft versehentlich?::
Ein CROSS JOIN bildet das kartesische Produkt: jede Zeile links × jede Zeile rechts (alle Kombinationen). Er entsteht oft ungewollt, wenn man beim Join die `ON`-Bedingung vergisst — dann explodiert die Zeilenzahl (z. B. 3 × 3 = 9 statt 3 erwarteter Zeilen).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs35KxA81DRUDOKYfsq46hA" -->
Nenne alle fünf Join-Typen mit einem Merksatz.::
INNER JOIN (nur Treffer auf beiden Seiten — Schnittmenge), LEFT JOIN (alle linken + passende rechte, sonst NULL — alles links), RIGHT JOIN (alle rechten + passende linke — alles rechts), FULL OUTER JOIN (alle Zeilen beider Seiten, NULL wo kein Partner — Vereinigung), CROSS JOIN (jede × jede — alle Kombinationen). In der Praxis dominieren INNER und LEFT.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGRno9nyjHPiCeF19u2NCl" -->
Warum kann eine Bedingung auf die rechte Tabelle in der `WHERE`-Klausel einen LEFT JOIN faktisch zu einem INNER JOIN machen?::
Weil `WHERE` NACH dem Join wirkt: Die mit NULL aufgefüllten linken Zeilen (ohne Partner) scheitern an einer Bedingung wie `WHERE pu.base_price > 10` (NULL > 10 ist UNKNOWN) und werden herausgefiltert — übrig bleiben nur Zeilen mit Partner, also das INNER-JOIN-Ergebnis. Bedingungen an die rechte Tabelle eines LEFT JOIN gehören daher meist ins `ON`, nicht ins `WHERE`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsndlfxrzKl2cV2uKtxkPvK" -->
==UNION== entfernt Duplikate (mit zusätzlicher Deduplizierung, etwas teurer), während ==UNION ALL== alle Zeilen inklusive Duplikate behält (schneller). Beide setzen gleiche Spaltenanzahl mit kompatiblen Datentypen voraus.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsrIDbJzAyVlaiZcVsctSaM" -->
Was ist ein Self-Join und wann braucht man ihn?::
Ein Self-Join ist ein Join einer Tabelle mit sich selbst (mit zwei verschiedenen Aliassen). Man braucht ihn bei selbstreferenzierenden Fremdschlüsseln, z. B. um zu jeder Kategorie ihre Oberkategorie auszugeben: `... FROM categories c LEFT JOIN categories parent ON parent.id = c.parent_id;`.
<!-- basicblock-end -->

# Transactions

<!-- basicblock-start oid="ObsRP2ZB4p4FFcwWibvPevlL" -->
Was ist eine Transaktion, und was bedeutet „ganz oder gar nicht"?::
Eine Transaktion ist eine Gruppe von SQL-Anweisungen, die als eine einzige, unteilbare Einheit ausgeführt werden. „Ganz oder gar nicht": Entweder gelingen alle Schritte und werden gemeinsam wirksam (commit), oder — wenn irgendwo etwas schiefgeht — wird die gesamte Gruppe rückgängig gemacht (rollback), als hätte sie nie stattgefunden. Nach außen gibt es keinen „halben" Zwischenzustand (= Atomarität).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsT15an42lfuRkPJhp9owmD" -->
Erkläre das klassische Geldüberweisungs-Beispiel für Transaktionen.::
Konto A abbuchen und Konto B gutschreiben sind zwei Schritte, die nur gemeinsam Sinn ergeben. Stürzt der Server ohne Transaktion zwischen beiden UPDATEs ab, ist das Geld bei A weg, aber nie bei B angekommen. In einer Transaktion (`BEGIN; UPDATE A -100; UPDATE B +100; COMMIT;`) wird ohne COMMIT die erste Abbuchung automatisch zurückgerollt — das Geld ist nie „halb überwiesen".
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8hKjbreAkog01BO1MwnAP" -->
Was bewirken `BEGIN`, `COMMIT` und `ROLLBACK`?::
`BEGIN` (bzw. `START TRANSACTION`) startet eine Transaktion — Änderungen werden ab hier gesammelt, aber noch nicht endgültig. `COMMIT` macht alle Änderungen seit BEGIN dauerhaft und für andere sichtbar. `ROLLBACK` verwirft alle Änderungen seit BEGIN — der Stand von vor BEGIN bleibt erhalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRSCrTIlblr7nAtNLbnaNd" -->
Was ist Autocommit, und läuft ein einzelnes `UPDATE` ohne `BEGIN` atomar?::
Autocommit heißt: Jede SQL-Anweisung, die nicht explizit in `BEGIN … COMMIT` eingeschlossen ist, läuft in ihrer eigenen, automatisch committeten Transaktion. Ein einzelnes `UPDATE` ist daher bereits atomar — es wirkt vollständig oder gar nicht. Einen expliziten `BEGIN … COMMIT`-Block braucht man erst, wenn MEHRERE Anweisungen zusammen atomar sein sollen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstEJRzTDAPBzBXKBnbnyz8" -->
Wofür ist ein `SAVEPOINT` innerhalb einer Transaktion gut?::
Ein SAVEPOINT setzt eine Zwischenmarke, zu der man mit `ROLLBACK TO SAVEPOINT name` teilweise zurückrollen kann, ohne die ganze Transaktion aufzugeben. Nützlich in längeren Transaktionen, in denen ein einzelner Teilschritt scheitern darf, während die bisherige Arbeit erhalten bleibt.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsWauTEtLqW6O350wyJGR3X" -->
Transaktionen bieten zwei Kernnutzen: Fehler-/Absturzsicherheit durch ==Atomarität== (kein „halber" Zustand) und Konsistenz bei Nebenläufigkeit durch ==Isolation== (deren Strenge über die Isolationsstufen geregelt wird).
<!-- clozeblock-end -->

# Views

<!-- basicblock-start oid="Obsfm4UUA9Ewcyp1TPjaqfMs" -->
Was ist eine View, und speichert sie eigene Daten?::
Eine View ist eine benannte, gespeicherte SELECT-Abfrage, die sich wie eine (virtuelle) Tabelle abfragen lässt. Sie speichert selbst KEINE Daten — nur die Definition der Abfrage. Beim Abfragen führt die DB die hinterlegte SELECT aus und liefert den aktuellen Stand der zugrunde liegenden Tabellen („gespeichertes Rezept, aktuelle Zutaten").
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsys62H7gfpyg2cWVsyiASj" -->
Nenne drei Zwecke, für die man Views einsetzt.::
1. Komplexität verstecken / Wiederverwendung: eine komplizierte Abfrage (Joins, Aggregationen) einmal definieren, danach einfach `SELECT * FROM view` (DRY, Abstraktion). 2. Stabile Schnittstelle / Entkopplung: die Tabellenstruktur nach außen stabil halten, interne Umbauten hinter der View verstecken (Information Hiding). 3. Sicherheit: einer Nutzergruppe nur eine View mit ausgewählten Spalten/Zeilen geben, ohne Zugriff auf die volle Tabelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspxfrvoREv9XsRNDjwofw0" -->
Macht eine normale View eine teure Abfrage schneller? Begründe.::
Nein. Eine normale View ist kein Snapshot und kein Cache — sie wird bei JEDER Abfrage neu ausgewertet und zeigt immer den aktuellen Stand. Ist die zugrunde liegende Abfrage teuer, ist auch jede Abfrage auf die View teuer. Die View macht es nur bequemer und wiederverwendbar, nicht schneller.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbVG2n7saUweo1mWZYAsoP" -->
Was ist der Unterschied zwischen einer normalen und einer Materialized View (Speicherung, Aktualität, Performance)?::
Normale View: speichert keine Daten (nur die Abfrage), immer live aktuell, Abfrage so teuer wie die zugrunde liegende Query. Materialized View: speichert das Ergebnis physisch auf der Platte, Abfragen darauf sind schnell, aber die Daten sind nur so aktuell wie das letzte `REFRESH MATERIALIZED VIEW` (das man explizit anstoßen muss, z. B. per nächtlichem Job). Typisch für teure Auswertungen/Reports.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObssQirNjQ6Os7fio4HAnMmW" -->
Eine ==normale View== speichert keine Daten und ist immer aktuell, während eine ==Materialized View== das Ergebnis physisch speichert (schnelle Abfragen) und per `REFRESH` aktualisiert werden muss.
<!-- clozeblock-end -->

<!-- basicblock-start oid="Obs6ikB022b84EnevkPwYT9f" -->
Unter welcher Bedingung kann man in eine View schreiben (INSERT/UPDATE) — und wann nicht mehr?::
Ein INSERT/UPDATE auf eine EINFACHE View (die genau einer Tabelle ohne Aggregation/Join entspricht) leitet PostgreSQL automatisch an die Basistabelle weiter. Sobald die View Joins, Aggregationen (`GROUP BY`) o. Ä. enthält, ist sie nicht ohne Weiteres beschreibbar — dann bräuchte man spezielle Konstrukte (INSTEAD OF-Trigger). Views sind primär eine Lese-Abstraktion.
<!-- basicblock-end -->

# ACID

<!-- basicblock-start oid="ObsaRrTzqmD6ThZ7sYKioS9F" -->
Wofür stehen die vier Buchstaben von ACID? Beschreibe jede Eigenschaft in einem Satz.::
A – Atomicity (Atomarität): eine Transaktion wird ganz oder gar nicht wirksam. C – Consistency (Konsistenz): eine Transaktion führt von einem gültigen Zustand in einen anderen gültigen (alle Constraints/Regeln gewahrt). I – Isolation: gleichzeitige Transaktionen stören sich nicht gegenseitig. D – Durability (Dauerhaftigkeit): committete Änderungen überleben auch Absturz/Stromausfall.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDyG0dxAUr2PD9fW5guTIL" -->
In welchem Kontext ist ACID besonders wichtig?::
Überall, wo mehrstufige Änderungen und/oder gleichzeitige Zugriffe auf Daten passieren, deren Korrektheit wichtig ist — z. B. Finanztransaktionen, Bestellungen, Lagerbestände, Buchungen. Es ist das Gegenteil des CSV-Ansatzes, bei dem keine dieser Garantien besteht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssHnlODE151MGgjjnUJZiC" -->
Erkläre die drei Nebenläufigkeits-Phänomene Dirty Read, Non-Repeatable Read und Phantom Read.::
Dirty Read: T1 liest eine noch nicht committete Änderung von T2, die anschließend zurückgerollt wird — T1 reagierte auf „nie existierende" Daten. Non-Repeatable Read: T1 liest dieselbe Zeile zweimal und bekommt unterschiedliche Werte, weil eine andere Transaktion sie zwischendurch geändert und committet hat. Phantom Read: T1 führt dieselbe Bereichs-Abfrage zweimal aus und bekommt zusätzliche/weniger Zeilen, weil zwischendurch Zeilen eingefügt/gelöscht wurden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAJvPGWYATSz5QJEkDTQU0" -->
Nenne die vier Isolationsstufen und welche Phänomene sie jeweils (nicht) verhindern.::
Read Uncommitted: erlaubt Dirty, Non-Repeatable, Phantom Read. Read Committed: verhindert Dirty Read; Non-Repeatable und Phantom noch möglich. Repeatable Read: verhindert Dirty + Non-Repeatable Read; Phantom (im Standard) noch möglich. Serializable: verhindert alle drei (Transaktionen verhalten sich, als liefen sie streng nacheinander). Strenger = mehr Schutz, aber mehr Aufwand/Sperren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs83r8ZwICzlxQZtqOp7AGx" -->
Was ist PostgreSQLs Default-Isolationsstufe, und welche Besonderheiten hat Postgres dank MVCC?::
Default ist Read Committed. Dank MVCC kennt Postgres gar kein echtes „Read Uncommitted" — ein Dirty Read ist grundsätzlich unmöglich, selbst auf der niedrigsten Stufe. Zudem verhindert Postgres' Repeatable Read dank MVCC sogar Phantom Reads schon mit (strenger als der Standard verlangt).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxlHg0N6AucV0niwnXdgSE" -->
Wie hängen Atomarität und Dauerhaftigkeit mit dem Write-Ahead-Log (WAL) zusammen?::
Änderungen werden zuerst ins WAL geschrieben, bevor sie die eigentlichen Datendateien berühren. Atomarität: ein abgebrochener Vorgang lässt sich aus dem Log sauber zurückrollen. Dauerhaftigkeit: vor der Bestätigung des COMMIT wird das Log auf ein dauerhaftes Medium geschrieben (fsync), sodass die DB nach einem Absturz den committeten Stand aus dem Log wiederherstellen kann.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs24I3cuaWiEchvE69tHpy2" -->
Der grundlegende Trade-off: ==ACID== (klassisch relational) steht für starke, sofortige Konsistenz, während ==BASE== (viele verteilte NoSQL-Systeme) Konsistenz zugunsten von Verfügbarkeit/Skalierung lockert (eventual consistency) — im Kern das CAP-Theorem.
<!-- clozeblock-end -->

# SQLAlchemy und ORM in Python

<!-- basicblock-start oid="Obs98D9w9GCb80XudUq5ieIH" -->
Wofür steht ORM, und welche zwei Welten überbrückt es?::
ORM = Object-Relational Mapping. Es überbrückt die objektorientierte Welt des Programms (Klassen, Objekte, Attribute, Referenzen) und die relationale Welt der Datenbank (Tabellen, Zeilen, Spalten, Fremdschlüssel), indem eine Bibliothek beide automatisch aufeinander abbildet — man arbeitet mit normalen Objekten, das ORM übersetzt das im Hintergrund in SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFZAS9sJlut4Fg5Xwsr02q" -->
Nenne die vier Kern-Zuordnungen, die ein ORM zwischen Objekt- und relationaler Welt herstellt.::
Klasse ↔ Tabelle, Objekt (Instanz) ↔ Zeile (Tupel), Attribut ↔ Spalte, Referenz auf ein anderes Objekt ↔ Fremdschlüssel-Beziehung. Diese Abbildung ist sprachunabhängig die immer gleiche Grundidee eines ORM.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8cWMxcxCd80wzhOvT1Zgo" -->
Welche typischen Objekt-Operationen übersetzt ein ORM in welche SQL-Anweisungen?::
`session.add(obj)` → `INSERT`; Ändern eines Attributs eines geladenen Objekts (`obj.name = "Neu"`) → `UPDATE`; das Laden per Primärschlüssel (`session.get(Product, 1)`) → `SELECT ... WHERE id = 1`; Löschen (`session.delete(obj)`) → `DELETE`. Man formuliert also Objektoperationen, das ORM erzeugt das passende SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMqH5SFENuiTRvNvgkXnLm" -->
Was ist die „Object-Relational Impedance Mismatch"?::
Die konzeptionelle Reibung, dass Objektgraphen (mit Vererbung, Referenzen, Objekt-Identität) und flache relationale Tabellen (mit Werten, Schlüsseln, Joins) grundsätzlich NICHT dasselbe Modell sind. Ein ORM verwaltet die Übersetzung zwischen beiden, damit man nicht ständig manuell zwischen „Zeile ↔ Objekt" konvertieren muss — beseitigt die Reibung aber nicht vollständig.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKanlwkgPIaAlHP6YjB6B0" -->
Aus welchen zwei Schichten besteht SQLAlchemy, und wofür ist jede da?::
1. Core (SQL Expression Language): baut SQL programmatisch, aber sehr nah an SQL — ohne volles Objekt-Mapping, mit maximaler Kontrolle. 2. ORM: die höhere Schicht mit dem eigentlichen Klassen-↔-Tabellen-Mapping, auf der man meistens arbeitet; sie baut auf dem Core auf. Man kann also je nach Bedarf die Abstraktionsebene wählen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsw1q3FLfgOA3GPSsVpzcpT" -->
Was sind Engine, Declarative Base, Modelle und `relationship` in einem SQLAlchemy-Setup?::
Engine (`create_engine(...)`): die Verbindung zur Datenbank inkl. Connection Pool. Declarative Base (`class Base(DeclarativeBase)`): gemeinsame Basisklasse, von der alle Modelle erben. Modelle: Klassen mit `__tablename__` und `mapped_column`-Attributen, die auf Tabellen/Spalten gemappt werden. `relationship(...)`: bildet eine Beziehung zwischen Modellen ab (z. B. 1:n), zusammen mit `ForeignKey` auf der Spalte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZqoxTPw4akjEUvlxnt4H4" -->
Wie erkennt man im SQLAlchemy-Modellcode die Datenbankkonzepte PK, FK und 1:n-Beziehung wieder?::
`mapped_column(primary_key=True)` = Primärschlüssel; `mapped_column(ForeignKey("products.id"))` = Fremdschlüssel; `relationship(back_populates=...)` auf beiden Seiten = die (z. B. 1:n-)Beziehung. `back_populates` verbindet die beiden Richtungen (Product.units ↔ PriceableUnit.product), sodass beide Seiten synchron bleiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszS4BJwoIBnSodLVoQ00Rk" -->
Was ist die `Session` in SQLAlchemy und warum ist sie der Dreh- und Angelpunkt der ORM-Arbeit?::
Die Session ist die standardmäßige Schnittstelle zum ORM: Sie verwaltet die Verbindung/Transaktion, verfolgt alle Objektänderungen und schreibt sie beim `commit()` gebündelt in die DB. Man ändert Objekte einfach im Speicher; die Session merkt sich, welche INSERT/UPDATE/DELETE nötig sind, und führt sie in korrekter Reihenfolge in einer Transaktion aus.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRE2vg5eW1bDnFAr7a53H1" -->
Was bedeutet das Unit-of-Work-Muster in SQLAlchemy?::
Die Session sammelt alle Änderungen (neue, geänderte, gelöschte Objekte) und schreibt sie GEBÜNDELT beim `commit()` in EINER Transaktion in die Datenbank — statt jede Änderung sofort einzeln auszuführen. So entspricht ein `commit()` genau einer atomaren Transaktion (das Transaktions-Prinzip, automatisiert).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2loIlwvpABoI3SkAq3zM4" -->
Was ist die Identity Map in SQLAlchemy?::
Innerhalb einer Session gibt es pro Datenbankzeile genau EIN Python-Objekt: Lädt man dieselbe Zeile (gleicher Primärschlüssel) zweimal, bekommt man dasselbe Objekt zurück. Das sorgt für Konsistenz (keine widersprüchlichen Kopien derselben Zeile) und dient als Cache innerhalb der Session.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsDsODZPhTGUXXkofTYxO9p" -->
Die SQLAlchemy-Session implementiert zwei klassische Muster: ==Unit of Work== (Änderungen sammeln und beim commit gebündelt in einer Transaktion schreiben) und ==Identity Map== (pro DB-Zeile genau ein Objekt innerhalb der Session).
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsVfB3oelomL8vIHEeRenSy" -->
Was ist Lazy Loading in einem ORM, und wo lauert dabei eine Falle?::
Lazy Loading heißt: Eine Beziehung (z. B. `product.units`) wird erst dann per zusätzlichem SELECT aus der DB geladen, wenn man tatsächlich darauf zugreift — nicht schon beim Laden des Hauptobjekts. Die Falle: In einer Schleife über viele Objekte löst jeder Beziehungszugriff eine eigene Query aus (das N+1-Problem).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7eVE1RQZA4FLXTI04j8VO" -->
Erkläre das N+1-Query-Problem an einem konkreten Beispiel.::
Iteriert man über 100 Produkte und greift je auf `product.units` zu, feuert das ORM bei naivem Lazy Loading 1 Query für die Produkte + 100 Queries für die Units = 101 Queries, wo idealerweise 2 reichen würden. Es ist die klassische versteckte ORM-Performance-Falle, weil das ORM die Datenbankzugriffe verschleiert. Lösung: gezieltes Eager Loading (z. B. `selectinload`/`joinedload`) — man muss das Problem aber erst kennen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspIYQkKk1AqYVwfKZsB5LU" -->
Warum werden ORMs kritisiert? Nenne mehrere Kritikpunkte.::
1. N+1-Query-Problem (versteckte Performance-Fallen durch Lazy Loading). 2. Leaky Abstraction: das ORM versteckt SQL, aber für Performance muss man doch verstehen, welches SQL erzeugt wird. 3. Komplexe Abfragen (Window Functions, komplexe CTEs) werden durch die ORM-API umständlicher als handgeschriebenes SQL. 4. Overhead & „Magie" (Mapping, Change Tracking, Identity Map) kosten Performance und erschweren Debugging. 5. Die Impedance Mismatch bleibt an den Rändern (Vererbung, komplexe Beziehungen) bestehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOQ8lMe2QLGGKXzEUYxraS" -->
Was ist der ausgewogene Konsens zum ORM-Einsatz („ORM oder SQL")?::
Kein Entweder-oder, sondern „ORM UND, wo nötig, SQL": Ein ORM ist hervorragend für den Großteil der alltäglichen CRUD-Operationen (spart viel Boilerplate, hält Code objektorientiert). Für die wenigen performance-kritischen oder komplexen Abfragen greift man bewusst auf handgeschriebenes SQL zurück (bei SQLAlchemy über den Core/`text()`). Genau deshalb ist solides SQL-Fundament die Voraussetzung, ein ORM verantwortungsvoll einzusetzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEt4Qb4L8JFoeSOpL3H5B8" -->
Warum ist es didaktisch sinnvoll, SQL erst OHNE ORM zu lernen (wie in der Übungsaufgabe)?::
Weil das ORM eine „leaky abstraction" ist: Es versteckt SQL, aber sobald Performance oder komplexe Abfragen ins Spiel kommen, muss man verstehen, welches SQL erzeugt wird (N+1, Joins, Indizes, Transaktionen). Ohne SQL-Fundament kann man ORM-Fehler weder erkennen noch beheben — das ORM setzt SQL-Verständnis voraus, ersetzt es nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWmWlvv2rMt7RLMZaZw1vY" -->
Welche vier Dinge muss ein ORM „unter der Haube" leisten?::
1. Metadaten-Verwaltung: wissen, welche Klasse↔Tabelle und welches Attribut↔Spalte (mit Typ) gehört (das Mapping). 2. SQL-Generierung: aus Objektoperationen die passenden INSERT/UPDATE/DELETE/SELECT erzeugen (inkl. korrekter Reihenfolge bei Beziehungen). 3. Identität & Zustandsverfolgung (Change Tracking): erkennen, welche geladenen Objekte sich geändert haben. 4. Typkonvertierung: zwischen DB-Typen und Sprachtypen übersetzen (z. B. NUMERIC↔Decimal, TIMESTAMP↔datetime).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscYsgYeyPXGjjokELYFfNF" -->
Was ist der Unterschied zwischen dem Active-Record- und dem Data-Mapper-Muster, und welchem folgt SQLAlchemy bzw. Django?::
Active Record: Das Modellobjekt kennt sich selbst und weiß, wie es sich speichert (`product.save()`); Modell und DB-Zugriff sind eng verwoben (z. B. Django ORM, Rails). Data Mapper: Die Persistenz liegt in einer separaten Schicht; das Modell weiß nichts von der DB, eine Session vermittelt (`session.add(); session.commit()`) — bessere Trennung der Belange, testbarer (z. B. SQLAlchemy, Hibernate). SQLAlchemy folgt dem Data-Mapper-Muster, Django dem Active-Record-Muster.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsW4i0blUL0vFuCKVJ6JDyZ" -->
Wie sieht ein UPDATE im SQLAlchemy-ORM aus, und warum braucht man keine „save"-Methode?::
Man lädt ein Objekt, ändert einfach ein Attribut (`p.name = "Neu"`) und ruft `session.commit()`. Eine eigene Speicher-Methode auf dem Objekt gibt es nicht (Data Mapper!) — die Session erkennt beim commit durch Change Tracking selbst, dass sich dieses persistente Objekt geändert hat, und erzeugt das passende UPDATE.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmF5L4E0aT0LxgUyxCfNtP" -->
Nenne die vier Objektzustände einer SQLAlchemy-Session und was jeder bedeutet.::
transient: frisch mit `Product(...)` erzeugt, der Session nicht bekannt, keine DB-Zeile. pending: mit `session.add()` vorgemerkt, aber noch nicht in die DB geschrieben. persistent: hat eine DB-Zeile und wird von der Session verwaltet (Änderungen werden verfolgt). detached: hat eine DB-Zeile, gehört aber zu keiner aktiven Session mehr (z. B. nach `session.close()`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZOW6PvG7pBehS9ua7QFoZ" -->
Was ist der Unterschied zwischen `flush()` und `commit()` in SQLAlchemy?::
`flush()` sendet die gesammelten INSERT/UPDATE/DELETE an die DB (innerhalb der laufenden Transaktion) — z. B. wird dabei eine SERIAL-id vergeben —, aber die Transaktion ist noch NICHT committet und kann zurückgerollt werden. `commit()` macht ein flush UND committet danach die Transaktion; erst jetzt sind die Änderungen dauerhaft und für andere sichtbar. (SQLAlchemy flusht oft automatisch vor einer Abfrage.)
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsnBKq0Tho0Wuf3iGye0Dkr" -->
Nach einem `commit()` sind SQLAlchemy-Objekte standardmäßig ==expired== — der nächste Attributzugriff lädt frische Werte aus der DB nach (==refresh==), was veraltete Daten verhindert.
<!-- clozeblock-end -->

<!-- basicblock-start oid="Obse1dwaiqIvD79G7h24hHlt" -->
Wie bildet man eine n:m-Beziehung in SQLAlchemy ab?::
Man definiert die Verknüpfungstabelle (als `Table` mit zwei FK-Spalten) und weist sie in `relationship(...)` per `secondary=verknüpfungstabelle` auf beiden Seiten zu (mit `back_populates`). Danach arbeitet man mit einer normalen Python-Liste (`product.tags.append(tag)`) — das ORM pflegt die Zeilen in der Verknüpfungstabelle und die sonst nötigen zwei Joins selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3DpYwzgTdvKrgthRWPcGS" -->
Was ist der Unterschied zwischen Lazy und Eager Loading, und wie löst man damit das N+1-Problem?::
Lazy Loading (Default): eine Beziehung (`product.units`) löst erst beim Zugriff ein eigenes SELECT aus — bequem, aber Quelle des N+1-Problems. Eager Loading lädt die Beziehung gleich mit: `selectinload(Product.units)` holt alle Kinder in EINER zusätzlichen `SELECT ... WHERE product_id IN (...)`-Abfrage (meist beste Wahl), `joinedload(...)` per JOIN. Mit `select(Product).options(selectinload(Product.units))` sinkt die Abfragezahl von N+1 auf konstant 2.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxRWy9ZpAouHrYIBXGqjsx" -->
Wie legt man ohne ORM mit `psycopg` ein Produkt an, und worauf muss man bei den Werten zwingend achten?::
Man öffnet eine Verbindung, führt das SQL selbst aus (`cur.execute("INSERT INTO products (name) VALUES (%s) RETURNING id", ("Segeltörn",))`), holt ggf. die id mit `cur.fetchone()` und ruft am Ende `conn.commit()`. Zwingend: Werte IMMER als Parameter (`%s`) übergeben, NIEMALS per f-String/String-Konkatenation in den SQL-Text einbauen — sonst droht SQL-Injection.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAnGzEbQrZsWAhg8j8AbYI" -->
Nenne die wichtigsten Trade-offs zwischen rohem SQL (psycopg) und dem ORM (SQLAlchemy).::
Roh (psycopg): man schreibt jedes Statement selbst, bekommt tuples/dicts (manuelles Mapping), reicht FK-Werte von Hand durch, steuert Transaktion/Reihenfolge selbst — maximale Kontrolle über das SQL, aber viel Boilerplate. ORM: SQL wird generiert, man bekommt fertige Objekte, Beziehungen über `.append(...)`, Session/Unit of Work erledigt Transaktion & Reihenfolge — wenig Boilerplate, weniger Injection-Risiko, aber indirektere Kontrolle über das SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsb7kIjrYYhC4KUsojHUIsM" -->
Wozu dient Alembic im SQLAlchemy-Ökosystem?::
Alembic ist das Migrationswerkzeug: Ändern sich die Modelle (neue Spalte/Tabelle), vergleicht Alembic die Modelle mit dem Ist-Zustand der DB und generiert versionierte Migrationsskripte (`upgrade`/`downgrade`), die man ins Git eincheckt. So bleibt das Schema im Team nachvollziehbar und reproduzierbar.
<!-- basicblock-end -->
