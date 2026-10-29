---
tags:
  - documentation
---

Kompakte Zusammenfassung des **SQLAlchemy Unified Tutorials** (2.0-Stil) — die wichtigsten Konzepte und die Syntax zum Nachschlagen. Karteikarten dazu: [[SQLAlchemy Tutorial - Karteikarten|SQLAlchemy Tutorial - Karteikarten]]. Übergeordnetes Kapitel: [[1.2 Relationale Datenbanken & ORM - Vertiefung|Relationale Datenbanken & ORM - Vertiefung]].

## Grundidee

SQLAlchemy bietet **zwei APIs**, die aufeinander aufbauen:

- **Core** — SQL-Toolkit: Engine, Verbindungen, `Table`/`MetaData`, SQL-Expression-Language (`select()`, `insert()` …). Arbeitet mit Zeilen/Spalten.
- **ORM** — baut auf Core auf: mappt Python-Klassen auf Tabellen, arbeitet mit Objekten. Persistenz läuft über die **`Session`** (erweitert die Core-Ausführung um die *Unit of Work*).

"Unified" = beide APIs nutzen dieselben Konstrukte (`select()` etc.), nur Ausführung/Ergebnis unterscheiden sich.

## 1. Engine — Verbindung herstellen

```python
from sqlalchemy import create_engine
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)
```

URL kommuniziert drei Dinge: `dialect+driver://user:pass@host:port/dbname`
1. **Datenbank-Typ** (dialect, z. B. `sqlite`, `postgresql`)
2. **DBAPI-Treiber** (z. B. `pysqlite`, `psycopg2`)
3. **Ort** der Datenbank

- **Lazy Initialization**: Engine verbindet erst bei der ersten Operation, nicht beim Erstellen.
- `echo=True` → loggt das generierte SQL (praktisch zum Lernen/Debuggen).
- Die Engine ist eine **globale Fabrik + Connection-Pool** — einmal pro Anwendung erstellen.

## 2. Transaktionen & DBAPI

```python
from sqlalchemy import text

# "commit as you go" — manuell committen
with engine.connect() as conn:
    result = conn.execute(text("SELECT x, y FROM some_table WHERE y > :y"), {"y": 6})
    conn.commit()

# "begin once" — Block committet am Ende automatisch (rollback bei Fehler)
with engine.begin() as conn:
    conn.execute(text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
                 [{"x": 1, "y": 1}, {"x": 2, "y": 4}])  # executemany
```

- `text()` kapselt rohes SQL; **Parameter immer gebunden** (`:name` + dict), niemals per String-Formatierung → schützt vor SQL-Injection.
- Liste von dicts → `executemany` (mehrere INSERTs auf einmal).
- **`Result`**-Objekt ist iterierbar; Zeilen sind named tuples:
  - `row.y`, `row[1]`, Tuple-Unpacking `for x, y in result:`
  - `result.all()`, `.first()`, `.scalars()` (nur 1. Spalte), `.mappings()` (dict-artig).
- Die ORM-**`Session`** führt Textstatements genauso aus:

```python
from sqlalchemy.orm import Session
with Session(engine) as session:
    session.execute(text("UPDATE ..."), {...})
    session.commit()
```

## 3. Datenbank-Metadaten

**Core** — `Table`/`MetaData` explizit:

```python
from sqlalchemy import MetaData, Table, Column, Integer, String, ForeignKey
metadata_obj = MetaData()

user_table = Table(
    "user_account", metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("name", String(30)),
    Column("fullname", String),
)
address_table = Table(
    "address", metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("user_id", ForeignKey("user_account.id"), nullable=False),
    Column("email_address", String, nullable=False),
)

metadata_obj.create_all(engine)   # emittiert CREATE TABLE
metadata_obj.drop_all(engine)
```

- Spalten über `table.c.name` ansprechbar. `MetaData` ist der Container aller `Table`-Objekte.

**ORM Declarative** — Klassen statt `Table`:

```python
from typing import List, Optional
from sqlalchemy import String, ForeignKey
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "user_account"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30))
    fullname: Mapped[Optional[str]]                       # Optional → nullable
    addresses: Mapped[List["Address"]] = relationship(back_populates="user")

class Address(Base):
    __tablename__ = "address"
    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("user_account.id"))
    user: Mapped["User"] = relationship(back_populates="addresses")

Base.metadata.create_all(engine)
```

- `Mapped[int]` → NOT NULL, `Mapped[Optional[int]]` → nullable (aus dem Type-Hint abgeleitet).
- `mapped_column()` nur nötig, wenn Details (PK, FK, Länge …) gesetzt werden.
- **Reflection** (Schema aus bestehender DB lesen): `Table("user_account", metadata_obj, autoload_with=engine)`.

## 4. Mit Daten arbeiten (SQL-Expression-Language)

```python
from sqlalchemy import select, insert, update, delete, func

# INSERT
stmt = insert(user_table).values(name="spongebob", fullname="Spongebob Squarepants")
# oder ORM-Objekte per session.add() (siehe 5.)

# SELECT (Core: Zeilen | ORM: Entitäten)
stmt = select(User).where(User.name == "spongebob").order_by(User.id)
stmt = select(User.name, Address.email_address).join(Address).where(User.id == Address.user_id)
```

Wichtige Bausteine:
- **Operator-Overloading**: `User.name == "spongebob"` gibt **kein** Python-`bool` zurück, sondern ein SQL-Expression-Objekt (`BinaryExpression`/`ColumnElement`). Deshalb baut man Bedingungen direkt mit `==`, `>`, `<`; NULL-Test mit `.is_(None)`/`.is_not(None)`; außerdem `.in_()`, `.like()`, `.between()`, `.label()` (AS), `.desc()`/`.asc()`.
- `.where(...)` — mehrere `.where()` bzw. Kommas = **AND**; explizit `and_()`, `or_()`, `not_()`.
- `.order_by()`, `.group_by()`, `.having()`, `.limit()`, `.offset()`, `.distinct()`.
- **Joins**: `.join(Address)` (nutzt FK/`relationship`), `.join_from(User, Address)`, `contains_eager()`.
- **Aggregat/Funktionen**: `func.count()`, `func.now()`, `func.max()` …
- **Aliase**: `aliased(User)`; **Subqueries**: `.subquery()`, `.cte()`; `union_all()`.
- Ausführen: `session.execute(stmt)` → Rows; `session.scalars(stmt)` → direkt Entitäten.

```python
# UPDATE / DELETE (Core, mengenbasiert)
update(user_table).where(user_table.c.name == "spongebob").values(fullname="Mr. Squarepants")
delete(user_table).where(user_table.c.name == "spongebob")
```

## 5. Datenmanipulation mit dem ORM (Unit of Work)

```python
with Session(engine) as session:
    squidward = User(name="squidward", fullname="Squidward Tentacles")
    session.add(squidward)      # Objekt ist "pending"
    session.flush()             # SQL wird emittiert, noch in der Transaktion
    print(squidward.id)         # jetzt befüllt

    u = session.get(User, 1)    # PK-Lookup (nutzt Identity Map, ggf. ohne SQL)
    u.fullname = "Neuer Name"   # einfach Attribut ändern → "dirty"

    session.delete(u)           # zum Löschen vormerken
    session.commit()            # autoflush + COMMIT
```

- **Objekt-Zustände**: *transient* → *pending* (`add`) → *persistent* (`flush`/`commit`) → *detached* (Session geschlossen).
- **Identity Map**: pro Session existiert genau **ein** Objekt je Primärschlüssel.
- **Autoflush**: vor jeder Query werden ausstehende Änderungen automatisch geflusht.
- **Unit of Work**: die Session sammelt Änderungen und schreibt sie in korrekter Reihenfolge beim `commit()`.
- `session.rollback()` verwirft die Transaktion und setzt Objekte zurück; `session.close()` löst Objekte ab.

## 6. Verknüpfte ORM-Objekte (`relationship`)

```python
addresses: Mapped[List["Address"]] = relationship(back_populates="user",
                                                   cascade="all, delete-orphan")
user: Mapped["User"] = relationship(back_populates="addresses")
```

- `back_populates` hält **beide Seiten synchron** (Objekt in `u.addresses` einfügen setzt automatisch `address.user`).
- Zusammenhängende Objekte über die Collection anlegen — beim `commit()` werden FKs automatisch gesetzt (Cascade):

```python
u.addresses.append(Address(email_address="squidward@example.com"))
```

- **Loading-Strategien** (gegen das N+1-Problem):
  - *Lazy* (Default): lädt beim Attributzugriff nach — bequem, aber N+1-Gefahr.
  - `selectinload(User.addresses)` — beste Wahl für **Collections** (1 zusätzliches SELECT `IN`).
  - `joinedload(Address.user)` — gut für **many-to-one** (JOIN).
  - Gesetzt via `select(User).options(selectinload(User.addresses))`.

## 7. Weiterführend

- Offizielles Tutorial: https://docs.sqlalchemy.org/en/20/tutorial/
- ORM-Vertiefung: Querying-Guide, Relationship-Patterns, Session-API in der Doku.
