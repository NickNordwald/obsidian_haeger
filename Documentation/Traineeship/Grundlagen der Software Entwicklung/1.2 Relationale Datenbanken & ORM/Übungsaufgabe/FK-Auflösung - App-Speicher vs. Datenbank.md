---
tags:
  - documentation
type: notiz
date: 2026-07-22
---

Wie sich der Fremdschlüssel `price_recommendations.capacity_id` auflösen lässt — **im Applikationsspeicher** (aktuell) vs. **in der Datenbank** (set-basiert). Kernmaterial für die Vor-/Nachteil-Diskussion von [[Umsetzungsstand (DB-Erweiterung)|Teil 2]] (Review-Punkt 2.3). Betrifft den `PostgresqlRecommendationWriter` (analog `SqlAlchemyRecommendationWriter`).

## Das Problem

`PriceRecommendation` (Domain-Objekt) bleibt bewusst **DB-agnostisch**: es kennt nur `product_id` + `calendar_date`, **nicht** die Surrogat-`id` der `capacity`-Zeile. Die Zieltabelle verlangt aber `capacity_id NOT NULL REFERENCES capacity(id)`. Beim Schreiben muss also jede Recommendation über ihren *natürlichen* Schlüssel `(product_id, calendar_date)` auf die `capacity.id` abgebildet werden. Genau dafür existiert der `UNIQUE (product_id, calendar_date)`-Constraint auf `capacity` — er garantiert Eindeutigkeit **und** liefert den Index für den Lookup.

## Weg A — Auflösung im App-Speicher (aktueller Stand)

Ein einmaliger Bulk-`SELECT` zieht alle capacity-Schlüssel in ein Python-Dict, dann pro Recommendation ein Dict-Lookup:

```python
cur.execute("SELECT id, product_id, calendar_date FROM capacity")
capacity_ids = {(pid, cal): cid for cid, pid, cal in cur}   # ~1,33 Mio Einträge

# ... pro Recommendation:
cid = capacity_ids.get((r.product_id, r.calendar_date))
if cid is None:
    missing += 1
    continue
```

- **Warum überhaupt so (statt N Einzel-`SELECT`s)?** Ein `SELECT` pro Recommendation wären bei Hunderttausenden Zeilen Hunderttausende Roundtrips → inakzeptabel. Ein Bulk-Lookup ist die naheliegende Rettung.
- **Kosten:** Das Dict hält ~1,33 Mio `(str, date) → int`-Einträge im **Python-Heap** (grob mehrere hundert MB). Der Join passiert in Python, nicht in der DB.

## Weg B — Auflösung in der Datenbank (`INSERT … SELECT`)

Idee: Die Recommendations mit ihrem **natürlichen** Schlüssel in eine Staging-Tabelle laden und die FK-Auflösung dem DB-Join überlassen. Kein großes Dict in Python.

**Schritt 1 — Staging-Tabelle** (temporär, keyt auf `(product_id, calendar_date)`):
```sql
CREATE TEMP TABLE staging_recommendations (
    product_id                 TEXT,
    calendar_date              DATE,
    days_until_travel          INTEGER,
    recommended_adjustment_pct NUMERIC(10, 6),
    occupancy                  DOUBLE PRECISION,
    total_capacity             INTEGER,
    booked_units               INTEGER
) ON COMMIT DROP;
```
`ON COMMIT DROP` → die Temp-Tabelle verschwindet automatisch mit der Transaktion.

**Schritt 2 — Recommendations per `COPY` in Staging** (kein Lookup nötig, `product_id`+`calendar_date` liegen ja auf jeder Recommendation):
```python
with cur.copy(
    "COPY staging_recommendations "
    "(product_id, calendar_date, days_until_travel, "
    "recommended_adjustment_pct, occupancy, total_capacity, booked_units) FROM STDIN"
) as copy:
    for r in recs:
        copy.write_row([
            r.product_id, r.calendar_date, r.days_until_travel,
            r.recommended_adjustment_pct, r.occupancy,
            r.total_capacity, r.booked_units,
        ])
```

**Schritt 3 — Ziel leeren + `INSERT … SELECT` mit JOIN** (nutzt den `UNIQUE`-Index auf `capacity`):
```sql
TRUNCATE price_recommendations RESTART IDENTITY;

INSERT INTO price_recommendations
    (capacity_id, days_until_travel, recommended_adjustment_pct,
     occupancy, total_capacity, booked_units)
SELECT c.id, s.days_until_travel, s.recommended_adjustment_pct,
       s.occupancy, s.total_capacity, s.booked_units
FROM   staging_recommendations s
JOIN   capacity c
  ON   c.product_id    = s.product_id
 AND   c.calendar_date = s.calendar_date;
```
Alles in **einer** Transaktion (Idempotenz + Atomarität, wie gehabt).

**Missing-Keys zählen** (der `INNER JOIN` wirft Nicht-Treffer stumm weg — um die bestehende `logger.warning`-Semantik zu behalten):
```python
# einfachste Variante: Differenz der Zeilenzahlen
missing = len(recs) - cur.rowcount      # rowcount nach dem INSERT
```
oder explizit per `LEFT JOIN … WHERE c.id IS NULL` zählen.

## Weg C — dasselbe in SQLAlchemy (`insert().from_select()`)

Der ORM-/Core-Ausdruck für genau dieses `INSERT … SELECT`:
```python
from sqlalchemy import insert, select

sel = (
    select(
        orm.Capacity.id,
        staging.c.days_until_travel,
        staging.c.recommended_adjustment_pct,
        staging.c.occupancy,
        staging.c.total_capacity,
        staging.c.booked_units,
    )
    .join(orm.Capacity, onclause=(
        (orm.Capacity.product_id == staging.c.product_id)
        & (orm.Capacity.calendar_date == staging.c.calendar_date)
    ))
)
stmt = insert(orm.PriceRecommendation).from_select(
    ["capacity_id", "days_until_travel", "recommended_adjustment_pct",
     "occupancy", "total_capacity", "booked_units"],
    sel,
)
session.execute(stmt)
```
→ derselbe set-basierte Join, nur als SQLAlchemy-Konstrukt. Damit deckt die Diskussion **alle drei** Wege ab.

## Trade-offs

| | Weg A (Python-Dict) | Weg B/C (DB-Join) |
|---|---|---|
| **Speicher** | ~1,33 Mio Einträge im Python-Heap | konstant in Python; Join im DB-Puffer |
| **Wo läuft die Logik** | Applikation (Zeilen-für-Zeile) | Datenbank (mengenbasiert, über Index) |
| **Skalierung** | wird mit N linear teurer im RAM | skaliert mit DB-Ressourcen/Index |
| **Lesbarkeit** | sehr pythonisch, explizit | mehr SQL + Temp-Tabelle |
| **Testbarkeit** | in Isolation leicht (reines Python) | braucht die DB (Integrationstest) |
| **Missing-Keys** | direkt zählbar (`get()` → None) | über `rowcount`-Differenz / `LEFT JOIN` |

## Fazit / Merksatz

Das ORM nimmt einem die FK-Auflösung **nicht** ab — sie bleibt dieselbe Denkarbeit, nur die Mechanik verschiebt sich. Der eigentliche Hebel ist die Frage *„im App-Speicher oder in der DB?"*: Für kleine Ergebnismengen ist der Python-Dict-Weg völlig ok und am lesbarsten; sobald die Auflösungstabelle groß wird (hier 1,33 Mio Zeilen), ist der set-basierte DB-Join die robustere, speicherschonendere Wahl. Genau dieser Kontrast — *Zeilen-für-Zeile im Applikationscode vs. mengenbasiert in SQL* — ist eines der stärksten Argumente in der ORM-vs-rohes-SQL-Diskussion.
