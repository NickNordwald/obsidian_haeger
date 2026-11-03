---
tags:
  - karteikarten
erstellt: 2026-11-03
cards-deck: "Traineeship::Grundlagen der Software Entwicklung::SQLAlchemy Tutorial"
flashcards:
  q-zzev: { nid: 1793715437105, hash: 8ve9rd76, sync: ikxrvp3y }
  q-ttfy: { nid: 1793715437130, hash: e75y6487, sync: k25yhkta }
  q-4qn7: { nid: 1793715437154, hash: zx74e9cd, sync: c5xjqttc }
  q-ierx: { nid: 1793715437179, hash: tp2sjw88, sync: 93i65dbq }
  q-apjr: { nid: 1793715437204, hash: t3mbyecy, sync: h7mhs6nv }
  q-y6vw: { nid: 1793715437240, hash: jz9qkpmc, sync: 9tps96vi }
  q-e7cw: { nid: 1793715437282, hash: 8xyge49i, sync: 2n2ect3b }
  q-mghc: { nid: 1793715437306, hash: 4bk2uy4s, sync: dkcsiu6z }
  q-weyp: { nid: 1793715437331, hash: yu7j6n4z, sync: bk65gxg7 }
  q-3ebz: { nid: 1793715437356, hash: vxfz3ktn, sync: xfknfazs }
  q-dypc: { nid: 1793715437381, hash: yyegp342, sync: a2uwnka3 }
  q-qdpc: { nid: 1793715437406, hash: n759wgq3, sync: bqtrcf32 }
  q-magk: { nid: 1793715437431, hash: jscxhidm, sync: uazf4525 }
  q-a5mf: { nid: 1793715437456, hash: zf5jdaru, sync: r8xntpxk }
  q-tjbb: { nid: 1793715437481, hash: isq9jkrx, sync: imhc5xyj }
  q-chtj: { nid: 1793715437506, hash: 79942rvy, sync: y889g7ac }
  q-t83n: { nid: 1793715437531, hash: 6pcy3e2k, sync: 5t2d2yix }
  q-uk7s: { nid: 1793715437555, hash: 7ta7fkyn, sync: qa8x2bhm }
  q-sz5w: { nid: 1793715437578, hash: rkvx9ejb, sync: 25j6pqbn }
  q-a8yg: { nid: 1793715437603, hash: ucgnt65n, sync: 9xmbqyi4 }
  q-45jf: { nid: 1793715437626, hash: b4ksauhf, sync: nn3pej7z }
  q-4273: { nid: 1793715437652, hash: gp5tywnm, sync: ae8563rk }
  q-2q58: { nid: 1793715437678, hash: zixpec4f, sync: rufnrmsk }
  q-adhi: { nid: 1793715437706, hash: cr8854mr, sync: xvrf8g7r }
  q-d7wb: { nid: 1793715437733, hash: wvd7i9bi, sync: eix4ve45 }
  q-6thz: { nid: 1793715437766, hash: a7amiu5y, sync: e9twcy82 }
  q-4bz4: { nid: 1793715437816, hash: 5jusvzqi, sync: ck7nnawy }
  q-ubnp: { nid: 1793715437866, hash: 6fzxjyrz, sync: q7nu7sh5 }
  q-d233: { nid: 1793715437917, hash: fat7s2pi, sync: j5xjegvp }
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
^q-zzev

Was ist die DBAPI?::Pythons Standardschnittstelle zu Datenbanktreibern (PEP 249), z. B. `psycopg2`. SQLAlchemy setzt Engine und Pooling darauf.
^q-ttfy

Wie ist eine SQLAlchemy-Datenbank-URL aufgebaut?::`dialekt+treiber://user:pass@host/dbname`  
Dialekt = DB-Typ, Treiber = DBAPI, danach der Ort der Datenbank.
^q-4qn7

## Engine und Transaktionen

Was ist die Engine, und wie oft erstellt man sie?::Fabrik für Verbindungen plus Connection-Pool zu einer Datenbank. Einmal pro Anwendung, verbindet sich lazy erst bei der ersten Operation.
^q-ierx

Unterschied zwischen `engine.connect()` und `engine.begin()`?::`connect()`: „commit as you go“, man ruft `conn.commit()` selbst auf, sonst Rollback am Blockende.  
`begin()`: committet am Blockende automatisch, Rollback bei Exception.
^q-apjr

## text() und Result

Wie übergibt man Parameter an `text()`, und warum nie per f-String?::Platzhalter `:name` plus dict: `conn.execute(text("… WHERE y > :y"), {"y": 6})`. Gebundene Parameter schützen vor SQL-Injection, eine Liste von dicts löst `executemany` aus.
^q-y6vw

Welche Methoden bietet ein `Result`?::`.all()` alle Zeilen, `.first()` erste oder `None`, `.one()` genau eine, sonst Fehler  
`.scalars()` erste Spalte je Zeile, `.mappings()` dict-artige Zeilen
^q-e7cw

## Metadaten und Mapping

Was sind `MetaData` und `table.c` im Core?::`MetaData` sammelt alle `Table`-Objekte, `create_all(engine)` legt sie an. `table.c.name` liefert das `Column`-Objekt für Expressions.
^q-mghc

Wie sieht eine ORM-Klasse im 2.0-Stil aus?::Erbt von einer `DeclarativeBase`, setzt `__tablename__` und deklariert Spalten als `name: Mapped[str]`.  
`mapped_column()` nur für Details wie `primary_key`, `ForeignKey`, `String(30)` oder `default`
^q-weyp

Wie bestimmt das ORM, ob eine Spalte nullable ist?::Aus dem Type-Hint: `Mapped[int]` ergibt NOT NULL, `Mapped[Optional[int]]` eine nullable Spalte.
^q-3ebz

Wie liest man eine bestehende Tabelle per Reflection ein?::`Table("name", metadata_obj, autoload_with=engine)`
^q-dypc

## SQL-Expressions

Was ergibt `User.name == "spongebob"`, und wie prüft man deshalb auf NULL?::Kein `bool`, sondern per Operator-Overloading ein SQL-Ausdruck für `WHERE name = :name_1`.  
NULL-Test deshalb mit `.is_(None)` bzw. `.is_not(None)`
^q-qdpc

Wie kombiniert man mehrere Bedingungen?::Mit `and_()`, `or_()`, `not_()`. Mehrere `.where()`-Aufrufe werden mit AND verknüpft.
^q-magk

Welche Spalten-Operatoren gibt es neben Vergleichen?::`.in_()`, `.not_in()`, `.like()`, `.ilike()`, `.between()`, `.contains()`, dazu `.label("alias")` für `AS` und `.desc()` zum Sortieren
^q-a5mf

## INSERT, SELECT, Joins, UPDATE und DELETE

Wie fügt man im Core mehrere Zeilen ein, und wie erhält man den neuen Primärschlüssel?::`conn.execute(insert(table), [{…}, {…}])` ohne `.values()`. Bei einer Zeile liefert `result.inserted_primary_key` den Schlüssel, `.returning(...)` gezielte Spalten.
^q-tjbb

Was liefern `session.execute(stmt)` und `session.scalars(stmt)` für `select(User)`?::`execute` liefert Rows, die Entität steckt in `row.User`. `scalars` liefert direkt die `User`-Objekte.
^q-chtj

Wie schreibt man eine Aggregat-Abfrage?::Mit `func`, `group_by` und `having`: `select(User.name, func.count(Address.id)).join(Address).group_by(User.name)`
^q-t83n

Wofür sind `aliased()`, `.subquery()`/`.cte()` und `.exists()`?::`aliased()` für Self-Joins, `.subquery()`/`.cte()` für Unterabfragen, `.exists()` für `EXISTS`-Bedingungen
^q-uk7s

Woher kennt `select(User).join(Address)` die ON-Bedingung?::Aus dem Fremdschlüssel bzw. der `relationship`. Explizit als zweites Argument, LEFT JOIN mit `.outerjoin()`.
^q-sz5w

Wie sehen mengenbasierte UPDATE- und DELETE-Statements im Core aus?::`update(t).where(...).values(...)` und `delete(t).where(...)`. Sie wirken direkt auf die Zeilen, ohne Objekte.
^q-a8yg

## ORM Session

Welche Zustände durchläuft ein ORM-Objekt?::transient → pending (`add()`) → persistent (nach `flush`/`commit`) → detached (Session geschlossen)
^q-45jf

Was ist der Unterschied zwischen `flush()` und `commit()`?::`flush()` sendet das SQL innerhalb der offenen Transaktion. `commit()` schreibt endgültig und expired danach alle Objekte, sie laden beim nächsten Zugriff neu.
^q-4273

Was sind Identity Map und Autoflush?::Identity Map: pro Session genau ein Objekt je Primärschlüssel.  
Autoflush: offene Änderungen werden vor jeder Query automatisch geflusht.
^q-2q58

Wie ändert man eine Zeile im ORM?::Attribut des Objekts setzen. Das Objekt wird „dirty“ und beim nächsten Flush geschrieben.
^q-adhi

Wozu dient `sessionmaker`?::Erzeugt eine konfigurierte Session-Fabrik, die man mit `with Session() as session:` wiederverwendet.
^q-d7wb

## Relationships

Was bewirkt `back_populates` bei `relationship()`?::Hält beide Seiten synchron: `u.addresses.append(a)` setzt automatisch `a.user`. Beim Commit werden die Fremdschlüssel befüllt.
^q-6thz

Wie modelliert man One-to-One und Many-to-Many mit `relationship()`?::One-to-One: `uselist=False`  
Many-to-Many: `secondary=association_table`
^q-4bz4

Welche Cascade löscht Kinder mit dem Elternobjekt und entfernt verwaiste Kinder?::`relationship(cascade="all, delete-orphan")`
^q-ubnp

Was ist das N+1-Problem, und wie löst man es?::Lazy Loading lädt jede Beziehung einzeln nach, bei N Objekten N Zusatz-Queries. Lösung über `.options(selectinload(...))` für Collections oder `joinedload(...)` für Many-to-One.
^q-d233
