---
deck: "Traineeship::Grundlagen der Software Entwicklung::SQLAlchemy Tutorial"
---

Karteikarten zum [[SQLAlchemy Tutorial|SQLAlchemy Tutorial]]. Synchronisierung wie bei [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.2 Relationale Datenbanken & ORM/Anki Karteikarten|Anki Karteikarten]] über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect): Anki im Hintergrund laufen lassen, dann in Obsidian **„Start Obsidian Anki Sync"** ausführen. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

---

# Grundlagen

<!-- basicblock-start oid="ObscMKOGpNbIJlfqgMPN4VLV" -->
Welche zwei APIs bietet SQLAlchemy und wie verhalten sie sich zueinander?::
**Core** = SQL-Toolkit (Engine, Verbindungen, `Table`/`MetaData`, SQL-Expression-Language, arbeitet mit Zeilen/Spalten). **ORM** = baut auf Core auf, mappt Python-Klassen auf Tabellen und arbeitet mit Objekten; Persistenz läuft über die `Session`. "Unified": beide teilen dieselben Konstrukte (z. B. `select()`), nur Ausführung/Ergebnis unterscheiden sich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsS6d1hKM8Ds3FbBg5t0GWh" -->
Was ist die DBAPI und welche Rolle spielt sie für SQLAlchemy?::
Die **DBAPI** (PEP 249) ist Pythons standardisierte Low-Level-Schnittstelle zu Datenbanktreibern (z. B. `pysqlite`, `psycopg2`). SQLAlchemy nutzt sie als „Fahrer" im Hintergrund und legt darüber Engine, Connection-Pooling und die SQL-Expression-Language.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsYJTZCmk2Q4wXlOYyRUfRv" -->
Die drei Bestandteile einer SQLAlchemy-Datenbank-URL sind ==Dialekt (DB-Typ)==, ==DBAPI-Treiber== und ==Ort der Datenbank== — Schema: `dialekt+treiber://user:pass@host/dbname`.
<!-- clozeblock-end -->

# Engine & Transaktionen

<!-- basicblock-start oid="Obs2wX82tKV8pkBRQ4q8K5dk" -->
Wie erstellt man eine Engine und wann verbindet sie sich tatsächlich mit der Datenbank?::
`from sqlalchemy import create_engine` → `engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)`. Durch **Lazy Initialization** verbindet sich die Engine erst bei der ersten Operation, nicht beim Erstellen. `echo=True` loggt das generierte SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdVFXBpDzoFO4fOwiJGdo5" -->
Was ist die Engine konzeptionell und wie oft erstellt man sie?::
Die Engine ist eine **globale Fabrik für Verbindungen plus Connection-Pool** zu einer bestimmten Datenbank. Man erstellt sie **genau einmal pro Anwendung** (Modul-Ebene) und teilt sie — nicht pro Anfrage neu.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYqoIyhDzkNKQ9DzyJjxC7" -->
Unterschied zwischen „commit as you go" (`engine.connect()`) und „begin once" (`engine.begin()`)?::
`with engine.connect() as conn:` — Transaktion offen, man muss selbst `conn.commit()` aufrufen. `with engine.begin() as conn:` — der Block committet am Ende automatisch und macht bei einer Exception ein Rollback.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs5qvFSo3x1QfloDQMWW6bS" -->
Verlässt man einen `with engine.connect()`-Block ohne `commit()`, wird die Transaktion automatisch ==zurückgerollt (rollback)== — SQLAlchemy committet nie implizit.
<!-- clozeblock-end -->

# text() & Result

<!-- basicblock-start oid="Obsf4Iy0eh0WxAxqsGD2D25f" -->
Wofür braucht man `text()` und wie übergibt man Parameter?::
`text()` kapselt rohes SQL als ausführbares Konstrukt. Parameter kommen als benannte Platzhalter `:name` ins SQL plus ein dict: `conn.execute(text("... WHERE y > :y"), {"y": 6})`. Eine **Liste von dicts** löst `executemany` aus (mehrere Statements).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEVt2gH12KaRNNclZ5MUPR" -->
Warum niemals Werte per f-String/`%` ins SQL einsetzen, sondern gebundene Parameter nutzen?::
Gebundene Parameter (`:name` + dict) werden vom Treiber getrennt vom SQL übertragen und korrekt escaped → Schutz vor **SQL-Injection** und richtige Typbehandlung. String-Formatierung öffnet Injection-Lücken und bricht bei Sonderzeichen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCWkC2UGE66EfB9tX97BBM" -->
Was liefert `execute()` zurück und wie greift man auf die Zeilen zu?::
Ein **`Result`**-Objekt (iterierbar). Zeilen sind named-tuple-artige `Row`-Objekte: `row.y`, `row[0]`, Tuple-Unpacking `for x, y in result:`. Zugriff auch per String-Key über `row._mapping`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obsek1JdwpdO9mMPT4ghzPoa" -->
Result-Methoden: ==`.all()`== gibt alle Zeilen als Liste, ==`.first()`== die erste (oder None), ==`.one()`== genau eine (Fehler sonst), ==`.scalars()`== nur die erste Spalte je Zeile, ==`.mappings()`== dict-artige Zeilen.
<!-- clozeblock-end -->

# Metadaten & Mapping

<!-- basicblock-start oid="Obs0HbZMTAo2aCj2m2NibSX5" -->
Wie definiert man im Core eine Tabelle und erzeugt sie in der Datenbank?::
Über `MetaData()` + `Table("name", metadata_obj, Column("id", Integer, primary_key=True), ...)`. Erzeugen/Löschen aller Tabellen mit `metadata_obj.create_all(engine)` bzw. `drop_all(engine)`. Spalten sind über `table.c.name` erreichbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaQKUlCNt7l9q1g5GFQYRG" -->
Was ist `MetaData` und wozu dient die `.c`-Collection einer Tabelle?::
`MetaData` ist der **Container/Registry aller `Table`-Objekte** (Schema-Katalog im Python-Speicher). `table.c` ist die Sammlung der Spalten-Objekte einer Tabelle — `table.c.name` liefert das `Column`-Objekt, das in SQL-Expressions verwendet wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsftWGCioQ6XZh6WW4oxEEm" -->
Nenne wichtige `Column`-Optionen zur Constraint-/Verhaltenssteuerung.::
`primary_key=True`, `nullable=False` (NOT NULL), `unique=True`, `index=True`, `default=` (Python-seitiger Default), `server_default=` (DB-seitig), `autoincrement=`. Fremdschlüssel über `ForeignKey("user_account.id")` als Column-Argument.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRmKHkHvK6OQO2VS6ODdp2" -->
Wie sieht eine ORM-Declarative-Klasse (SQLAlchemy 2.0) aus?::
```python
class Base(DeclarativeBase): pass

class User(Base):
    __tablename__ = "user_account"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30))
    fullname: Mapped[Optional[str]]
```
`Base.metadata.create_all(engine)` erzeugt die Tabellen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDocr1ysqIxIw6cXTE2aUA" -->
Wann braucht man `mapped_column()` und wann reicht die reine Typannotation?::
Reicht die bloße Annotation (`fullname: Mapped[Optional[str]]`), wenn keine Extra-Details nötig sind. `mapped_column(...)` ist nötig, sobald man **Details** setzt: `primary_key`, `ForeignKey`, Länge (`String(30)`), `unique`, `default`, `nullable`-Override usw.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsgcD3lVy2Jk4Tsh8LdCM19" -->
Im ORM bedeutet der Type-Hint ==`Mapped[int]`== NOT NULL, während ==`Mapped[Optional[int]]`== eine nullable Spalte ergibt — die Nullability wird also aus dem Typ abgeleitet.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsARI8ep3PuZekr1CgWfpF8" -->
Eine bestehende DB-Tabelle liest man per **Reflection** ein mit `Table("name", metadata_obj, ==autoload_with=engine==)`.
<!-- clozeblock-end -->

# SQL-Expressions & Operatoren

<!-- basicblock-start oid="Obs0908wLNTuhh7fXD2aVgH8" -->
Was passiert, wenn man einen Vergleichsoperator auf eine Spalte/ein Attribut anwendet, z. B. `User.name == "spongebob"`?::
Es wird **kein Python-`bool` berechnet**, sondern per **Operator-Overloading** ein SQL-Expression-Objekt (`BinaryExpression`, ein `ColumnElement`) erzeugt, das die WHERE-Klausel `user_account.name = :name_1` repräsentiert. Deshalb baut man Bedingungen direkt mit `==`, `>`, `<` usw.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obso5X7qMls6AmeKAWoUCofD" -->
Weil Spalten-Operatoren SQL erzeugen statt Booleans, testet man auf NULL nicht mit `== None`, sondern mit ==`.is_(None)`== bzw. ==`.is_not(None)`== (→ `IS NULL` / `IS NOT NULL`).
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsuvnrsmFMmoTGhwVIrjUPF" -->
Nenne gängige Spalten-Operatoren jenseits der Vergleiche und was sie erzeugen.::
`.in_([1, 2, 3])` → `IN`, `.not_in(...)`, `.like("%foo%")` / `.ilike(...)` → LIKE, `.between(a, b)`, `.startswith()/.contains()/.endswith()`. Arithmetik/`+` auf Spalten erzeugt ebenfalls SQL (bei Strings Konkatenation).
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs4o14VrgFmtXPOJwsAWMfn" -->
Mehrere Bedingungen kombiniert man mit ==`and_()`==, ==`or_()`== und ==`not_()`==; mehrere aufeinanderfolgende `.where()`-Aufrufe werden implizit mit ==AND== verknüpft.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsoNRd8T7k17OjFNbnCz7qq" -->
Wie benennt man eine Spalte/Expression im Ergebnis um (SQL `AS`)?::
Mit `.label("neuer_name")`, z. B. `select(User.name.label("username"))`. Im Ergebnis ist die Spalte dann über `row.username` erreichbar. Für Sortierrichtung: `User.name.desc()` / `.asc()`.
<!-- basicblock-end -->

# INSERT

<!-- basicblock-start oid="ObsJGXsTPQ41a9LthItqvKio" -->
Wie sieht ein Core-INSERT aus und wie fügt man mehrere Zeilen ein?::
`insert(user_table).values(name="spongebob", fullname="Spongebob Squarepants")`. Für viele Zeilen (executemany) das Statement ohne `.values()` und eine Liste von dicts an `execute` übergeben: `conn.execute(insert(user_table), [{...}, {...}])`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsqdYR8LRvJCw29eDPMuk6W" -->
Nach einem Single-Row-INSERT liefert ==`result.inserted_primary_key`== den generierten Primärschlüssel; mit ==`.returning(...)`== kann man gezielt Spalten zurückgeben lassen (INSERT ... RETURNING).
<!-- clozeblock-end -->

# SELECT

<!-- basicblock-start oid="ObsrolXlNZzs3kbBPdLBzIDZ" -->
Wie baut man ein SELECT mit Filter und Sortierung, und wie holt man Entitäten statt Rows?::
`stmt = select(User).where(User.name == "spongebob").order_by(User.id)`. Ausführen: `session.execute(stmt)` → Rows (Objekt in `row.User`), `session.scalars(stmt)` → direkt die Entitäten. `select(User.name, Address.email)` selektiert einzelne Spalten statt ganzer Entitäten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswpFZrVWW7fgNFHSVUWl8u" -->
Wie nutzt man SQL-Funktionen und Aggregate mit `func`?::
Über das `func`-Namespace: `func.count()`, `func.count(User.id)`, `func.max(...)`, `func.now()`, `func.lower(...)`. Kombiniert mit `.group_by(...)` und `.having(...)` für Aggregat-Abfragen, z. B. `select(User.name, func.count(Address.id)).join(Address).group_by(User.name)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsX8fz7MafDbHNZ1zoqONgV" -->
Wozu dienen `aliased()`, `.subquery()`/`.cte()` und `.exists()`?::
`aliased(User)` erzeugt einen Tabellen-Alias (z. B. Self-Join / dieselbe Tabelle mehrfach). `.subquery()` bzw. `.cte()` verpacken ein SELECT als Unterabfrage / Common Table Expression zum Weiterverwenden. `.exists()` / `select(...).exists()` erzeugt eine `EXISTS`-Bedingung.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsJXTb8o6plLMBIlPLk2DOY" -->
Ergebnisse zweier SELECTs vereinigt man mit ==`union()`== (Duplikate entfernt) bzw. ==`union_all()`== (Duplikate behalten); Seitenbegrenzung über ==`.limit()`== und ==`.offset()`==.
<!-- clozeblock-end -->

# Joins

<!-- basicblock-start oid="ObsLxFkhuZtc8qYvnsWZQbKT" -->
Wie fügt man in einem SELECT einen JOIN hinzu und woher kennt SQLAlchemy die ON-Bedingung?::
`select(User).join(Address)` — die ON-Klausel wird **automatisch aus dem Fremdschlüssel** (bzw. der `relationship`) abgeleitet. Explizit: `.join(Address, User.id == Address.user_id)`. `.join_from(User, Address)` setzt den linken Startpunkt explizit.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsJw82DB4tVBfw8rVbSHdJo" -->
Einen LEFT OUTER JOIN erzeugt man mit ==`.outerjoin(...)`== (bzw. `.join(..., isouter=True)`); im ORM kann man auch beziehungsbasiert joinen: `select(User).join(==User.addresses==)`.
<!-- clozeblock-end -->

# UPDATE / DELETE

<!-- basicblock-start oid="ObstWWnDa69EkckYv2klXUGZ" -->
Wie sehen mengenbasierte Core-`update()`/`delete()`-Statements aus?::
`update(user_table).where(user_table.c.name == "spongebob").values(fullname="Mr. Squarepants")` und `delete(user_table).where(user_table.c.name == "spongebob")`. Sie wirken direkt auf die passenden Zeilen (keine Objekte nötig). Im ORM ändert man Zeilen dagegen meist über Objekt-Attribute.
<!-- basicblock-end -->

# ORM Session (Unit of Work)

<!-- basicblock-start oid="Obsn5cuKNTs3lq3D92X3ElS3" -->
Welche Lebenszyklus-Zustände durchläuft ein ORM-Objekt?::
**transient** (neu, unbekannt) → **pending** (nach `session.add()`) → **persistent** (nach `flush`/`commit`, in DB) → **detached** (Session geschlossen). Ein `flush()` emittiert das SQL innerhalb der Transaktion, `commit()` schreibt endgültig.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9EsoUSzt55zt6IrxwPEg8" -->
Was ist die Identity Map und was macht Autoflush?::
**Identity Map**: pro Session existiert genau ein Objekt je Primärschlüssel (`session.get(User, 1)` liefert dasselbe Objekt, ggf. ohne neues SQL). **Autoflush**: ausstehende Änderungen werden vor jeder Query automatisch in die DB geflusht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKFG8D4Nrt79wzaVpYoT3h" -->
Nenne die wichtigsten Session-Methoden und was sie tun.::
`add()`/`add_all()` (pending machen), `flush()` (SQL senden, noch in Transaktion), `commit()` (endgültig schreiben), `get(Klasse, pk)` (PK-Lookup), `delete()` (löschen vormerken), `rollback()` (verwerfen), `refresh()`/`expire()` (aus DB neu laden), `close()` (Objekte ablösen).
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obsw3kFupfBnrPfxPQ5txZ7w" -->
Ein Objekt aktualisiert man im ORM, indem man einfach sein ==Attribut ändert== (Objekt wird „dirty"); nach einem `commit()` sind Objekte standardmäßig ==expired== und werden beim nächsten Zugriff automatisch neu geladen.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsvyG7sTLrqZ5zbkGts6Nhr" -->
Wie verwaltet man die Session-Lebensdauer sauber, und wozu `sessionmaker`?::
Als Kontextmanager: `with Session(engine) as session: ... session.commit()` — schließt automatisch. `sessionmaker(engine)` erzeugt eine konfigurierte Session-Fabrik, die man in der App wiederverwendet, statt überall `Session(engine)` zu tippen.
<!-- basicblock-end -->

# Verknüpfte Objekte (relationship)

<!-- basicblock-start oid="ObsMQkoHvDYBHmW1cCdCDqEY" -->
Wozu dient `back_populates` bei `relationship()`?::
Es verbindet beide Seiten einer Beziehung und hält sie **synchron**: Fügt man ein Objekt in die Collection der einen Seite ein (`u.addresses.append(addr)`), wird die Rückreferenz (`addr.user`) automatisch gesetzt. Beim `commit()` werden die Fremdschlüssel automatisch befüllt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWzilKJu8wIHoEs1mmAq5E" -->
Wie modelliert man die Beziehungstypen mit `relationship()`?::
**One-to-Many**: Sammlung auf der „Eltern"-Seite `Mapped[List["Address"]]`, `Mapped["User"]` auf der Kindseite. **Many-to-One**: nur die Skalar-Seite. **One-to-One**: `relationship(..., uselist=False)`. **Many-to-Many**: `relationship(secondary=association_table)`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obsi0NkL5O2JURVxZnITayuN" -->
Damit Kind-Objekte beim Löschen des Elternteils mitgelöscht werden bzw. „verwaiste" Kinder verschwinden, setzt man `relationship(cascade=="all, delete-orphan"==)`.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsaHD2Seyji3ChQRo5NQv0p" -->
Was ist das N+1-Problem und mit welchen Loading-Strategien begegnet man ihm?::
Lazy Loading (Default) lädt eine Beziehung erst beim Attributzugriff — bei N Objekten also N zusätzliche Queries. Gegenmittel: **`selectinload()`** (beste Wahl für Collections, ein SELECT ... IN) und **`joinedload()`** (für many-to-one, per JOIN), gesetzt via `select(User).options(selectinload(User.addresses))`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsPLFnJxkDyYOLojqWvLg95" -->
Loader-Strategien setzt man über ==`.options(...)`== am `select()`; neben `selectinload`/`joinedload` gibt es u. a. ==`subqueryload`==, `raiseload` (verhindert Lazy Loads) und `contains_eager()` (für bereits gejointe Daten).
<!-- clozeblock-end -->
