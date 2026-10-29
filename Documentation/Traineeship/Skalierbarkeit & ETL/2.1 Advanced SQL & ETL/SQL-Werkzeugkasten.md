---
tags:
  - documentation
---

# SQL-Werkzeugkasten

Nachschlagenotiz, **kein Stichwort**: keine Kontrollfragen, kein Selbsttest, keine Karten. Beim Schreiben von Queries aufschlagen.

Gehört zu [[2.1 Advanced SQL & ETL - Leitfaden]] · [[2.1 Advanced SQL & ETL - Vertiefung]]

Deckt die vier Lücken ab, die nach dem Kapitel offen geblieben sind — drei davon stehen seit Kapitel 1 in [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|der Blinde-Flecken-Liste]].

## Inhalt

- [[#1. `EXPLAIN ANALYZE` lesen]]
- [[#2. `INSERT … ON CONFLICT` und `RETURNING`]]
- [[#3. Die `NOT IN`-NULL-Falle]]
- [[#4. Das tägliche Kleinzeug]]
- [[#5. `GROUPING SETS`, `ROLLUP`, `CUBE`]]

---

# 1. `EXPLAIN ANALYZE` lesen

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT …;
```

- **`EXPLAIN`** allein zeigt nur den **geplanten** Ablauf — die Query läuft nicht.
- **`ANALYZE`** führt sie **wirklich aus** und zeigt echte Zeiten und Zeilenzahlen. ⚠️ Bei `INSERT`/`UPDATE`/`DELETE` wird tatsächlich geschrieben — in eine Transaktion packen und zurückrollen.
- **`BUFFERS`** zeigt, wie viele Datenblöcke aus dem Cache (`shared hit`) und wie viele von der Platte (`read`) kamen.

## Die Zeile verstehen

```text
Index Scan using idx_bestellungen_kunde on bestellungen
  (cost=0.43..8.45 rows=1 width=64) (actual time=0.021..0.023 rows=3 loops=1)
```

| Feld | Bedeutung |
|---|---|
| `cost=0.43..8.45` | Geschätzte Kosten in willkürlichen Einheiten: **Startkosten..Gesamtkosten**. Nur zum *Vergleich* von Plänen brauchbar, keine Zeit. |
| `rows=1` | **Geschätzte** Zeilenzahl |
| `actual … rows=3` | **Tatsächliche** Zeilenzahl |
| `loops=1` | Wie oft dieser Knoten ausgeführt wurde. ⚠️ **`actual time` ist pro Loop** — bei `loops=1000` mit `actual time=0.5` sind das 500 ms, nicht 0,5. |

> [!important] Die eine Zahl, auf die es ankommt
> **`rows` (geschätzt) vs. `actual rows`.** Weichen sie um mehr als etwa Faktor 10 ab, hat der Planer sich verschätzt — und **jede** Entscheidung, die er darauf aufgebaut hat (Join-Reihenfolge, Join-Algorithmus, Index oder nicht), ist potenziell falsch.
>
> Erste Gegenmaßnahme: `ANALYZE tabelle;`. Hilft das nicht, sind die Statistiken zu grob (`ALTER TABLE … ALTER COLUMN … SET STATISTICS 1000;`) oder es gibt eine Korrelation zwischen Spalten, die der Planer nicht kennt (`CREATE STATISTICS`).

Der Plan wird **von innen nach außen** gelesen: Die am tiefsten eingerückten Knoten laufen zuerst.

## Scan-Typen

| Knoten | Was passiert | Gut oder schlecht? |
|---|---|---|
| **Seq Scan** | Ganze Tabelle lesen | Bei kleinen Tabellen oder wenn ein Großteil der Zeilen gebraucht wird **richtig**. Bei „ich will 3 von 10 Mio." ein Alarmzeichen. |
| **Index Scan** | Über den Index gehen, für jeden Treffer die Zeile aus dem Heap holen | Gut bei wenigen Treffern |
| **Index Only Scan** | Nur den Index lesen, **Heap-Zugriff entfällt** | Optimal — möglich, wenn der Index **alle** gebrauchten Spalten enthält (`INCLUDE`) und die Visibility Map aktuell ist |
| **Bitmap Index Scan** + **Bitmap Heap Scan** | Erst alle passenden Zeilenadressen sammeln, sortieren, dann den Heap **in physischer Reihenfolge** lesen | Der Mittelweg bei mittlerer Trefferzahl — vermeidet wildes Springen auf der Platte |

## Join-Algorithmen

| Knoten | Wie | Wann der Planer ihn wählt |
|---|---|---|
| **Nested Loop** | Für jede Zeile links die rechte Seite durchsuchen | Wenn **links wenige** Zeilen sind und rechts ein Index passt. Bei falscher Schätzung der Klassiker für „läuft seit zwei Stunden". |
| **Hash Join** | Rechte Seite komplett in eine Hash-Tabelle im RAM, dann links durchlaufen | Der Standard bei großen Mengen und Gleichheits-Joins |
| **Merge Join** | Beide Seiten sortieren, dann parallel durchlaufen | Wenn beide Seiten schon sortiert vorliegen (z. B. über Indizes) |

Weitere häufige Knoten: **Sort** (achte auf `Sort Method: external merge Disk: …` — dann reicht `work_mem` nicht), **HashAggregate**/**GroupAggregate** (`GROUP BY`), **WindowAgg** (Window Functions), **Gather**/**Gather Merge** (parallele Ausführung), **Materialize**, **Memoize**.

## Checkliste beim Plan-Lesen

1. Wo geht die meiste **Zeit** hin? (`actual time` mal `loops`)
2. Wo weichen **geschätzte und echte Zeilen** stark ab?
3. Gibt es einen **Seq Scan** auf einer großen Tabelle mit selektivem Filter?
4. Steht ein **Nested Loop** über einer großen linken Seite?
5. Sortiert etwas auf **Disk** statt im RAM?
6. Bei `BUFFERS`: viel `read`, wenig `hit` → die Daten passen nicht in den Cache

---

# 2. `INSERT … ON CONFLICT` und `RETURNING`

Das konkrete Sprachmittel für die Idempotenz aus [[2.1 Advanced SQL & ETL - Vertiefung#5. Idempotenz|Stichwort 5]].

## UPSERT

```sql
INSERT INTO kunden (id, name, email, aktualisiert_am)
VALUES (42, 'Meier', 'meier@example.com', now())
ON CONFLICT (id) DO UPDATE
SET name           = EXCLUDED.name,
    email          = EXCLUDED.email,
    aktualisiert_am = EXCLUDED.aktualisiert_am;
```

- **`EXCLUDED`** ist die Pseudotabelle mit der Zeile, die **eingefügt werden sollte**. So kommt man an die neuen Werte, während `kunden.spalte` den alten Wert liefert.
- Der Konfliktziel-Ausdruck `(id)` muss einem **Unique- oder Primärschlüssel** entsprechen (oder einem Unique-Index bzw. `ON CONSTRAINT name`).

**Nur einfügen, wenn nicht vorhanden:**

```sql
INSERT INTO kunden (id, name) VALUES (42, 'Meier')
ON CONFLICT (id) DO NOTHING;
```

**Nur aktualisieren, wenn sich etwas geändert hat** — spart Schreiblast und unnötige Zeilenversionen (MVCC!):

```sql
ON CONFLICT (id) DO UPDATE
SET name = EXCLUDED.name
WHERE kunden.name IS DISTINCT FROM EXCLUDED.name;
```

`IS DISTINCT FROM` ist dabei wichtig: `<>` liefert bei NULL auf einer Seite **NULL**, nicht `true` — die Zeile würde übersprungen.

## `RETURNING`

Gibt die tatsächlich geschriebenen Zeilen zurück, ohne zweites `SELECT`:

```sql
INSERT INTO bestellungen (kunde_id, betrag) VALUES (42, 99.90)
RETURNING id, erstellt_am;

UPDATE tickets SET status = 'closed' WHERE alter_status = 'open'
RETURNING id;

DELETE FROM alte_daten WHERE datum < '2020-01-01'
RETURNING *;
```

Funktioniert bei `INSERT`, `UPDATE`, `DELETE` — und ist die Grundlage des Archivierungsmusters aus [[2.1 Advanced SQL & ETL - Vertiefung#8.3 CTEs|Abschnitt 8.3]]. Erklärt nebenbei, warum du im ORM nach `add()` + `flush()` sofort die `id` hast.

## `MERGE` (ab PostgreSQL 15)

```sql
MERGE INTO ziel z USING quelle q ON z.id = q.id
WHEN MATCHED     THEN UPDATE SET wert = q.wert
WHEN NOT MATCHED THEN INSERT (id, wert) VALUES (q.id, q.wert);
```

SQL-Standard und mächtiger (`WHEN MATCHED THEN DELETE` ist möglich), aber `ON CONFLICT` ist in Postgres die eingespieltere Wahl für den reinen Upsert-Fall.

---

# 3. Die `NOT IN`-NULL-Falle

> [!danger] Der klassische stille Bug
> **`NOT IN (subquery)` liefert *gar nichts*, sobald die Subquery einen einzigen NULL-Wert enthält.** Keine Fehlermeldung, keine Warnung — ein leeres Ergebnis.

```sql
-- Wenn bestellungen.kunde_id irgendwo NULL ist: liefert IMMER null Zeilen
SELECT * FROM kunden
WHERE id NOT IN (SELECT kunde_id FROM bestellungen);
```

**Warum:** `x NOT IN (a, b, NULL)` bedeutet `x <> a AND x <> b AND x <> NULL`. Der letzte Vergleich ergibt **NULL** (unbekannt), nicht `false` — und `true AND NULL` ist `NULL`, was in `WHERE` wie `false` wirkt. Das ist die dreiwertige Logik aus [[1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]].

**Die robusten Alternativen:**

```sql
-- 1. NOT EXISTS — bevorzugt: NULL-sicher und meist der bessere Plan
SELECT * FROM kunden k
WHERE NOT EXISTS (SELECT 1 FROM bestellungen b WHERE b.kunde_id = k.id);

-- 2. Anti-Join
SELECT k.* FROM kunden k
LEFT JOIN bestellungen b ON b.kunde_id = k.id
WHERE b.kunde_id IS NULL;

-- 3. NULLs explizit ausschließen (funktioniert, ist aber die schwächste Variante)
SELECT * FROM kunden
WHERE id NOT IN (SELECT kunde_id FROM bestellungen WHERE kunde_id IS NOT NULL);
```

**Faustregel: `NOT IN` mit einer Subquery gar nicht erst verwenden — immer `NOT EXISTS`.** Bei `IN` (ohne `NOT`) besteht das Problem nicht.

Verwandte NULL-Fallen:

- `spalte <> 'wert'` findet **keine** Zeilen mit NULL → `spalte IS DISTINCT FROM 'wert'`
- `COUNT(spalte)` zählt NULLs **nicht**, `COUNT(*)` schon
- `SUM` über eine leere Menge liefert **NULL**, nicht 0 → `COALESCE(SUM(x), 0)`
- Bei `LEFT JOIN` gehört die Bedingung auf die rechte Tabelle in die **`ON`**-Klausel; in `WHERE` wird daraus stillschweigend ein Inner Join

---

# 4. Das tägliche Kleinzeug

## Bedingungen und NULL

```sql
CASE WHEN betrag > 1000 THEN 'gross'
     WHEN betrag > 100  THEN 'mittel'
     ELSE 'klein' END                        -- ohne ELSE ist der Default NULL

COALESCE(rabatt, 0)                          -- erster nicht-NULL-Wert
NULLIF(divisor, 0)                           -- macht 0 zu NULL → schützt vor Division durch null
GREATEST(a, b, c) / LEAST(a, b, c)           -- Max/Min über Spalten (nicht über Zeilen!)
wert IS DISTINCT FROM anderer                -- NULL-sicherer Ungleich-Vergleich
```

## Datum und Zeit

```sql
date_trunc('month', ts)          -- 2026-09-17 14:33 → 2026-09-01 00:00
date_trunc('week', ts)           -- Wochenanfang (Montag)
EXTRACT(year FROM ts)            -- 2026        (auch: month, day, dow, doy, hour, epoch)
AGE(ende, start)                 -- Intervall zwischen zwei Zeitpunkten
ts AT TIME ZONE 'Europe/Berlin'  -- Zeitzonen-Umrechnung
now() - interval '7 days'
generate_series('2026-01-01'::date, '2026-12-31', '1 day')   -- lückenlose Kalendertabelle
```

⚠️ **`timestamptz` statt `timestamp`** verwenden, sonst gibt es später Zeitzonenchaos — siehe die Determinismus-Fallen in [[2.1 Advanced SQL & ETL - Vertiefung#5.3 Die versteckten Brüche|Abschnitt 5.3]].

## Casts und Zahlen

```sql
wert::integer            -- Kurzform
CAST(wert AS integer)    -- Standard
wert::numeric(10,2)

round(x, 2) · ceil(x) · floor(x) · abs(x) · mod(a, b)
```

⚠️ **`numeric` für Geld, niemals `float`/`double precision`.** Fließkomma rundet binär: `0.1 + 0.2 <> 0.3`. Bei Beträgen wird daraus irgendwann eine Differenz, die niemand erklären kann.

⚠️ **Integer-Division:** `5 / 2` ergibt in SQL **2**, nicht 2.5. Für Quoten `5::numeric / 2` oder `5.0 / 2`.

## Text

```sql
lower(s) · upper(s) · trim(s) · btrim(s, '"')
s1 || ' ' || s2                  -- Konkatenation; ⚠️ NULL || 'x' ergibt NULL
concat_ws(', ', a, b, c)         -- ignoriert NULLs
substring(s from 1 for 3) · left(s, 3) · right(s, 3)
split_part(s, ',', 2)            -- 2. Element (1-basiert!)
replace(s, 'alt', 'neu')
length(s) · position('x' in s)
lpad(s, 5, '0')                  -- '42' → '00042'
format('Kunde %s hat %s Bestellungen', name, anzahl)
```

**Regex:**

```sql
s ~ '^[A-Z]{2}\d{4}$'                        -- passt? (~* = case-insensitive)
regexp_replace(s, '\s+', ' ', 'g')           -- 'g' = alle Vorkommen
regexp_match(s, '(\d{4})-(\d{2})')           -- Array der Gruppen
regexp_split_to_table(s, ',')                -- eine Zeile pro Element
```

⚠️ Eine Funktion um eine indizierte Spalte **schaltet den Index aus** — siehe [[2.1 Advanced SQL & ETL - Vertiefung#9.6 Klassische Performance-Fallen|Abschnitt 9.6]]. Für `lower(name)` einen Ausdrucksindex anlegen.

## Mengenoperationen

```sql
SELECT … UNION     SELECT …   -- vereinigt, entfernt Duplikate (dedupliziert = teuer)
SELECT … UNION ALL SELECT …   -- vereinigt, behält alles → im Zweifel diese Variante
SELECT … INTERSECT SELECT …   -- Schnittmenge
SELECT … EXCEPT    SELECT …   -- Differenz (was links steht und rechts nicht)
```

`EXCEPT` ist praktisch zum **Datenabgleich**: „welche Zeilen sind in der alten Version und nicht in der neuen?" — genau der Vergleich, den du beim SQL-vs-Python-Benchmark brauchst.

---

# 5. `GROUPING SETS`, `ROLLUP`, `CUBE`

Mehrere Aggregationsebenen in **einer** Query statt mehrerer mit `UNION ALL`.

```sql
-- Umsatz pro Produkt, pro Kategorie und insgesamt — in einem Durchlauf
SELECT kategorie, produkt, SUM(umsatz)
FROM verkaeufe
GROUP BY GROUPING SETS ((kategorie, produkt), (kategorie), ());
```

| Konstrukt | Erzeugt |
|---|---|
| `GROUPING SETS ((a,b), (a), ())` | genau die aufgezählten Kombinationen |
| `ROLLUP (a, b, c)` | die **Hierarchie**: `(a,b,c)`, `(a,b)`, `(a)`, `()` — für Drill-down entlang einer Ebene |
| `CUBE (a, b)` | **alle** Kombinationen: `(a,b)`, `(a)`, `(b)`, `()` |

Aggregatzeilen enthalten in den nicht gruppierten Spalten **NULL**. Um echte NULLs von Aggregatzeilen zu unterscheiden, gibt es `GROUPING()`:

```sql
SELECT kategorie, produkt, SUM(umsatz),
       GROUPING(kategorie) AS ist_gesamt_ueber_kategorie
FROM verkaeufe
GROUP BY ROLLUP (kategorie, produkt);
```

**Vorteil gegenüber `UNION ALL`:** Die Tabelle wird **einmal** gelesen statt dreimal — derselbe Gedanke wie bei `FILTER` in [[2.1 Advanced SQL & ETL - Vertiefung#2.6 Bedingte Aggregation: `FILTER`|Abschnitt 2.6]].
