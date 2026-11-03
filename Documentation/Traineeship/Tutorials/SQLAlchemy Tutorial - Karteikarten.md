---
tags:
  - karteikarten
erstellt: 2026-11-03
cards-deck: "Traineeship::Grundlagen der Software Entwicklung::SQLAlchemy Tutorial"
---

# SQLAlchemy Tutorial - Karteikarten

Karteikarten zum [[SQLAlchemy Tutorial]]. Format und Abgleich mit Anki beschreibt [[Anki-Karten erstellen]].

- [x] Grundlagen
- [x] Engine und Transaktionen
- [x] `text()` und Result
- [x] Metadaten und Mapping
- [x] SQL-Expressions
- [x] INSERT, SELECT, Joins, UPDATE und DELETE
- [x] ORM Session
- [x] Relationships

## Grundlagen

Welche zwei APIs bietet SQLAlchemy?::**Core**: SQL-Toolkit mit Engine, `Table` und Expression Language, arbeitet mit Zeilen.  
**ORM**: baut auf Core auf, mappt Klassen auf Tabellen, Persistenz über die `Session`.

Was ist die DBAPI?::Pythons Standardschnittstelle zu Datenbanktreibern (PEP 249), z. B. `psycopg2`. SQLAlchemy setzt Engine und Pooling darauf.

Wie ist eine SQLAlchemy-Datenbank-URL aufgebaut?::`dialekt+treiber://user:pass@host/dbname`  
Dialekt = DB-Typ, Treiber = DBAPI, danach der Ort der Datenbank.

## Engine und Transaktionen

Was ist die Engine, und wie oft erstellt man sie?::Fabrik für Verbindungen plus Connection-Pool zu einer Datenbank. Einmal pro Anwendung, verbindet sich lazy erst bei der ersten Operation.

Unterschied zwischen `engine.connect()` und `engine.begin()`?::`connect()`: „commit as you go“, man ruft `conn.commit()` selbst auf, sonst Rollback am Blockende.  
`begin()`: committet am Blockende automatisch, Rollback bei Exception.

## text() und Result

Wie übergibt man Parameter an `text()`, und warum nie per f-String?::Platzhalter `:name` plus dict: `conn.execute(text("… WHERE y > :y"), {"y": 6})`. Gebundene Parameter schützen vor SQL-Injection, eine Liste von dicts löst `executemany` aus.

Welche Methoden bietet ein `Result`?::`.all()` alle Zeilen, `.first()` erste oder `None`, `.one()` genau eine, sonst Fehler  
`.scalars()` erste Spalte je Zeile, `.mappings()` dict-artige Zeilen

## Metadaten und Mapping

Was sind `MetaData` und `table.c` im Core?::`MetaData` sammelt alle `Table`-Objekte, `create_all(engine)` legt sie an. `table.c.name` liefert das `Column`-Objekt für Expressions.

Wie sieht eine ORM-Klasse im 2.0-Stil aus?::Erbt von einer `DeclarativeBase`, setzt `__tablename__` und deklariert Spalten als `name: Mapped[str]`.  
`mapped_column()` nur für Details wie `primary_key`, `ForeignKey`, `String(30)` oder `default`

Wie bestimmt das ORM, ob eine Spalte nullable ist?::Aus dem Type-Hint: `Mapped[int]` ergibt NOT NULL, `Mapped[Optional[int]]` eine nullable Spalte.

Wie liest man eine bestehende Tabelle per Reflection ein?::`Table("name", metadata_obj, autoload_with=engine)`

## SQL-Expressions

Was ergibt `User.name == "spongebob"`, und wie prüft man deshalb auf NULL?::Kein `bool`, sondern per Operator-Overloading ein SQL-Ausdruck für `WHERE name = :name_1`.  
NULL-Test deshalb mit `.is_(None)` bzw. `.is_not(None)`

Wie kombiniert man mehrere Bedingungen?::Mit `and_()`, `or_()`, `not_()`. Mehrere `.where()`-Aufrufe werden mit AND verknüpft.

Welche Spalten-Operatoren gibt es neben Vergleichen?::`.in_()`, `.not_in()`, `.like()`, `.ilike()`, `.between()`, `.contains()`, dazu `.label("alias")` für `AS` und `.desc()` zum Sortieren

## INSERT, SELECT, Joins, UPDATE und DELETE

Wie fügt man im Core mehrere Zeilen ein, und wie erhält man den neuen Primärschlüssel?::`conn.execute(insert(table), [{…}, {…}])` ohne `.values()`. Bei einer Zeile liefert `result.inserted_primary_key` den Schlüssel, `.returning(...)` gezielte Spalten.

Was liefern `session.execute(stmt)` und `session.scalars(stmt)` für `select(User)`?::`execute` liefert Rows, die Entität steckt in `row.User`. `scalars` liefert direkt die `User`-Objekte.

Wie schreibt man eine Aggregat-Abfrage?::Mit `func`, `group_by` und `having`: `select(User.name, func.count(Address.id)).join(Address).group_by(User.name)`

Wofür sind `aliased()`, `.subquery()`/`.cte()` und `.exists()`?::`aliased()` für Self-Joins, `.subquery()`/`.cte()` für Unterabfragen, `.exists()` für `EXISTS`-Bedingungen

Woher kennt `select(User).join(Address)` die ON-Bedingung?::Aus dem Fremdschlüssel bzw. der `relationship`. Explizit als zweites Argument, LEFT JOIN mit `.outerjoin()`.

Wie sehen mengenbasierte UPDATE- und DELETE-Statements im Core aus?::`update(t).where(...).values(...)` und `delete(t).where(...)`. Sie wirken direkt auf die Zeilen, ohne Objekte.

## ORM Session

Welche Zustände durchläuft ein ORM-Objekt?::transient → pending (`add()`) → persistent (nach `flush`/`commit`) → detached (Session geschlossen)

Was ist der Unterschied zwischen `flush()` und `commit()`?::`flush()` sendet das SQL innerhalb der offenen Transaktion. `commit()` schreibt endgültig und expired danach alle Objekte, sie laden beim nächsten Zugriff neu.

Was sind Identity Map und Autoflush?::Identity Map: pro Session genau ein Objekt je Primärschlüssel.  
Autoflush: offene Änderungen werden vor jeder Query automatisch geflusht.

Wie ändert man eine Zeile im ORM?::Attribut des Objekts setzen. Das Objekt wird „dirty“ und beim nächsten Flush geschrieben.

Wozu dient `sessionmaker`?::Erzeugt eine konfigurierte Session-Fabrik, die man mit `with Session() as session:` wiederverwendet.

## Relationships

Was bewirkt `back_populates` bei `relationship()`?::Hält beide Seiten synchron: `u.addresses.append(a)` setzt automatisch `a.user`. Beim Commit werden die Fremdschlüssel befüllt.

Wie modelliert man One-to-One und Many-to-Many mit `relationship()`?::One-to-One: `uselist=False`  
Many-to-Many: `secondary=association_table`

Welche Cascade löscht Kinder mit dem Elternobjekt und entfernt verwaiste Kinder?::`relationship(cascade="all, delete-orphan")`

Was ist das N+1-Problem, und wie löst man es?::Lazy Loading lädt jede Beziehung einzeln nach, bei N Objekten N Zusatz-Queries. Lösung über `.options(selectinload(...))` für Collections oder `joinedload(...)` für Many-to-One.
