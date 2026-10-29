---
tags:
  - documentation
---

Kompakte Referenz zu **psycopg 3** — dem PostgreSQL-Treiber für Python, mit dem in der Übungsaufgabe **rohes SQL** geschrieben wird (bewusst *ohne* ORM). Die wichtigsten Objekte, ihre Methoden und typische Nutzung zum Nachschlagen. Gegenstück mit ORM: [[SQLAlchemy Tutorial|SQLAlchemy Tutorial]]. Übergeordnetes Kapitel: [[1.2 Relationale Datenbanken & ORM - Vertiefung|Relationale Datenbanken & ORM - Vertiefung]]. Praktischer Einsatz: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand (DB-Erweiterung)]].

## Grundidee

**psycopg 3** ist ein **DBAPI-Treiber**: eine Bibliothek, die Python-Code mit einem PostgreSQL-Server verbindet und SQL-Strings hinschickt bzw. Ergebniszeilen zurückbringt. Kein ORM, keine Query-Abstraktion — du schreibst SQL, psycopg kümmert sich nur um **Verbindung**, **Typ-Umwandlung** (Python ↔ PostgreSQL) und **Transaktionen**.

Installation als optionale Dependency (`pyproject.toml`): `psycopg[binary]>=3.2` — die `[binary]`-Variante bringt die vorkompilierte libpq mit (kein System-Postgres nötig).

Zwei zentrale Objekte, alles andere hängt daran:
- **`Connection`** — eine offene Verbindung zur DB (+ die aktuelle Transaktion).
- **`Cursor`** — das Werkzeug, über das du Befehle absetzt und Ergebnisse liest.

## 1. Connection — verbinden

```python
import psycopg

conn = psycopg.connect("host=localhost dbname=pricing user=pricing password=...")
```

Der Verbindungs-String (**DSN**, Data Source Name) kann in **zwei Formaten** kommen, psycopg erkennt automatisch welches:

```python
"host=localhost dbname=pricing user=pricing password=geheim"   # libpq Keyword/Value
"postgresql://pricing:geheim@localhost:5432/pricing"            # URI-Form (postgres:// als Alias)
```

**DSN nie von Hand zusammenbauen** (Escaping-Fallen bei Sonderzeichen im Passwort). Stattdessen aus Teilen bauen lassen:

```python
from psycopg.conninfo import make_conninfo
dsn = make_conninfo(host="localhost", dbname="pricing", user="pricing", password=pw)
```

Als **Context-Manager** (empfohlen — schließt automatisch, committet/rollt zurück):

```python
with psycopg.connect(dsn) as conn:
    ...   # Block-Ende: Transaktion commit (bzw. rollback bei Exception), Verbindung offen bis GC
```

## 2. Cursor — Befehle absetzen & Ergebnisse lesen

Der Cursor ist das Objekt, über das SQL läuft:

```python
with conn.cursor() as cur:
    cur.execute("SELECT count(*) FROM bookings")
    print(cur.fetchone())        # (939559,)
```

**Ergebnisse abholen:**

| Methode | Liefert |
|---|---|
| `cur.fetchone()` | nächste Zeile als Tupel, oder `None` wenn keine mehr |
| `cur.fetchall()` | **alle** restlichen Zeilen als `list[tuple]` |
| `cur.fetchmany(n)` | die nächsten `n` Zeilen |
| `for row in cur:` | über den Cursor **iterieren** (speicherschonend bei vielen Zeilen) |

```python
cur.execute("SELECT product_id, bedroom_type FROM products")
for product_id, bedroom_type in cur:
    ...
```

## 3. Parameter — immer `%s`, nie f-string

Werte **niemals** per f-string/`+` in den SQL-String basteln (SQL-Injection + falsches Typ-Adapting). Platzhalter ist `%s`, die Werte kommen als zweites Argument:

```python
cur.execute(
    "INSERT INTO products (product_id, bedroom_type) VALUES (%s, %s)",
    ("XRKO4", "1 Bedroom"),
)
```

psycopg setzt die Werte sicher ein und wandelt Python-Typen korrekt um: `date` → `DATE`, `Decimal` → `NUMERIC`, `int` → `INTEGER`, **`None` → `NULL`**.

⚠️ **`%s` gilt nur für Werte, nicht für Bezeichner** (Tabellen-/Spaltennamen). Die müssen fest im String stehen — oder, wenn dynamisch nötig, sicher über `psycopg.sql`:

```python
from psycopg import sql
cur.execute(sql.SQL("SELECT * FROM {}").format(sql.Identifier("bookings")))
```

**Mehrere Zeilen** auf einmal (kleine Mengen) mit `executemany`:

```python
cur.executemany("INSERT INTO customers (cust_id) VALUES (%s)", [("A",), ("B",)])
```

## 4. Transaktionen

psycopg 3 ist standardmäßig **nicht autocommit** — Änderungen gelten erst mit `commit`. Drei Wege:

```python
# a) Connection als Context-Manager: Block-Ende committet, Exception rollt zurück
with psycopg.connect(dsn) as conn:
    with conn.cursor() as cur:
        cur.execute("INSERT ...")
    # hier automatisch committet

# b) Explizite Transaktion (auch mitten in einer offenen Connection)
with conn.transaction():
    cur.execute("INSERT ...")
    cur.execute("UPDATE ...")     # beides atomar: alles-oder-nichts

# c) Manuell
conn.commit()
conn.rollback()
```

**„alles-oder-nichts" nutzen**, wenn mehrere Schritte zusammengehören — z. B. der Ingest lädt alle Tabellen in **einer** `conn.transaction()`, damit ein Fehler bei `bookings` auch schon geladene `capacity`-Zeilen zurückrollt.

### `conn.transaction()` + `conn.cursor()` kombiniert — warum, und Unterschied zu nur `cursor()`

In Code wie Ingest/Writer steht oft `with conn.transaction(), conn.cursor() as cur:` (Kurzform für zwei verschachtelte `with`-Blöcke). Wichtig zu verstehen, weil die beiden Objekte unterschiedliche Jobs haben:

- **`conn.cursor()`** regelt nur den *Lifecycle des Cursors* (schließt ihn am Blockende) — sagt **nichts** über Commit/Rollback aus. Ob committet wird, hängt dann komplett davon ab, wie die Connection *außenrum* gehalten wird (z. B. via `with psycopg.connect(...) as conn:`, das erst am äußersten Blockende committet).
- **`conn.transaction()`** definiert explizit eine atomare Einheit: normales Blockende → Commit, Exception im Block → Rollback — unabhängig davon, was der Aufrufer mit der Connection sonst noch vorhat.

Der Mechanismus dahinter (nachgeschaut im psycopg-Quellcode, `psycopg/transaction.py`, Methode `_push_savepoint`):

```python
self._outer_transaction = self.pgconn.transaction_status == IDLE
```

`conn.transaction()` prüft beim Betreten, ob die Connection gerade **IDLE** ist (keine Transaktion offen):

- **IDLE** → wird zur **äußeren** Transaktion: echtes `BEGIN`, am Ende echtes `COMMIT`/`ROLLBACK`.
- **Schon eine Transaktion aktiv** (z. B. weil man verschachtelt aufgerufen wird) → wird zur **inneren**: setzt nur ein `SAVEPOINT`, am Ende `RELEASE SAVEPOINT` bzw. bei Exception `ROLLBACK TO SAVEPOINT` — die äußere Transaktion bleibt unberührt.

**Warum explizit `conn.transaction()` nutzen**, auch wenn der Aufrufer die Connection eh schon per `with connect() as conn:` hält (was am Ende ohnehin committen würde)?

1. **Atomizität, die nicht von der Aufrufer-Disziplin abhängt.** Bei „TRUNCATE + mehrere COPYs" (Ingest) oder „TRUNCATE + COPY" (Postgres-Writer) soll bei einem Fehler mittendrin die alte Tabelle unverändert bleiben, nicht leer. Verlässt man sich nur auf „der Aufrufer wird schon korrekt `with connect()` benutzen", ist das eine stille Annahme, die bricht, sobald die Connection mal anders gehalten/wiederverwendet wird.
2. **Komponierbar dank automatischer Savepoints.** Weil `transaction()` selbst erkennt, ob es outer oder nested ist, kann dieselbe Methode sowohl **top-level** (eigenes BEGIN/COMMIT) als auch **eingebettet** in einen größeren mehrstufigen DB-Vorgang (wird automatisch zum SAVEPOINT, rollt nur sich selbst zurück) aufgerufen werden — ohne dass der Code selbst unterscheiden muss, in welcher Situation er läuft.

Kurz: `cursor()` = Werkzeug zum SQL-Absetzen, `transaction()` = explizite, selbst-beschreibende Grenze für „das hier ist eine atomare Einheit".

## 5. COPY — Massen-Import

`COPY` ist PostgreSQLs **Bulk-Load** — um Größenordnungen schneller als tausende einzelne `INSERT`s, weil die Daten als ein Strom reingehen. In psycopg 3 ein Context-Manager, in den man Zeile für Zeile schreibt:

```python
with cur.copy(
    "COPY capacity (product_id, calendar_date, bookable_units, optional_units) FROM STDIN"
) as copy:
    for r in rows:
        copy.write_row([r.product_id, r.calendar_date, r.bookable_units, r.optional_units])
```

Merkpunkte:
- **`FROM STDIN`** ist Pflicht — sagt Postgres, dass die Daten aus dem Stream kommen.
- Die **Spaltenliste** im `COPY (...)` steuert, *was* geschrieben wird — Weggelassenes füllt Postgres selbst (z. B. `IDENTITY`- oder `GENERATED`-Spalten überspringen).
- **`write_row` erwartet die Werte in exakt der Reihenfolge der Spaltenliste** — Vertauschen landet ohne Fehler in der falschen Spalte.
- Native Python-Typen übergeben (`date`, `Decimal`, `int`, `None`) — kein manuelles Formatieren.

Für den umgekehrten Weg (Export) gibt es `COPY ... TO STDOUT` analog mit `copy.read_row()`.

## 6. Fehlerbehandlung

Alle DB-Fehler erben von `psycopg.Error`; spezifische Unterklassen in `psycopg.errors`:

```python
import psycopg
try:
    cur.execute("INSERT ...")
except psycopg.errors.UniqueViolation:
    ...          # z. B. PK-/UNIQUE-Konflikt
except psycopg.OperationalError:
    ...          # z. B. „connection refused" (Server aus)
```

Häufig in der Praxis: `OperationalError` (Server nicht erreichbar), `UniqueViolation`, `ForeignKeyViolation`, `NotNullViolation`, `CheckViolation` — spiegeln direkt die Constraints aus dem [[Datenmodell|Datenmodell]].

## 7. Row Factories (optional)

Standardmäßig sind Zeilen **Tupel**. Will man stattdessen dicts oder benannte Objekte, setzt man eine `row_factory`:

```python
from psycopg.rows import dict_row
with psycopg.connect(dsn, row_factory=dict_row) as conn:
    cur = conn.execute("SELECT product_id FROM products")
    cur.fetchone()     # {"product_id": "XRKO4"}
```

## Zusammenfassung — der typische Ablauf

```python
with psycopg.connect(dsn) as conn:          # 1. verbinden (Transaktion startet)
    with conn.cursor() as cur:              # 2. Cursor holen
        cur.execute("SELECT ...", params)   # 3. SQL mit %s-Parametern
        rows = cur.fetchall()               # 4. Ergebnis lesen
    # 5. Block-Ende: commit (oder rollback bei Exception), Verbindung schließt
```

**Kernprinzip:** `Connection` = Verbindung + Transaktion, `Cursor` = Befehle + Ergebnisse, Werte immer über `%s`, Masse über `COPY`, Zusammengehöriges in eine Transaktion.
