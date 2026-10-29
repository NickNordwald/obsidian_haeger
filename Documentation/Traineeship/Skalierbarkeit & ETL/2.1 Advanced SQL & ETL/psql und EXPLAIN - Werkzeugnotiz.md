---
tags:
  - documentation
---

# psql und EXPLAIN — Werkzeugnotiz

Nachschlagenotiz zu den beiden Werkzeugen, die in Kapitel 2.1 ständig gebraucht wurden. Kein Stichwort, keine Karten — zum Aufschlagen.

> [!note] Ablageort
> Liegt vorerst im Kapitelordner neben dem [[SQL-Werkzeugkasten]]. Beides sind allgemeine Postgres-Werkzeuge, keine Kapitel-2.1-Themen — wenn später ein Ordner für kapitelübergreifende Werkzeugnotizen entsteht, gehören sie dorthin.

## Inhalt

- [[#Teil 1 — psql]]
- [[#Teil 2 — EXPLAIN]]

---

# Teil 1 — psql

## Was es ist

`psql` ist der **offizielle Kommandozeilen-Client** von PostgreSQL — kein Drittanbieter-Werkzeug, sondern Teil der Distribution. Er ist auf jedem Server verfügbar, auf dem Postgres läuft, funktioniert über SSH, in Skripten und in CI-Pipelines.

Für Data Engineering ist er das Werkzeug, das immer da ist, wenn die GUI gerade nicht will. Und er kann Dinge, die keine Erweiterung kann: Ausgabe in Dateien lenken, Dateien clientseitig importieren, Abfragen wiederholt ausführen, Variablen von außen hereinreichen.

**Installation auf dem Host:** `sudo apt install postgresql-client`

## Verbinden

```bash
# Über den veröffentlichten Port (Compose: 127.0.0.1:5432)
psql -h localhost -p 5432 -U pricing -d pricing

# Über den Container, ohne psql auf dem Host
docker compose exec db psql -U pricing -d pricing

# Als Verbindungs-URL
psql "postgresql://pricing:pricing@localhost:5432/pricing"
```

Das Passwort abfragen zu lassen ist der Standard. Für Skripte gibt es `PGPASSWORD=…` (landet in der Prozessliste — nur lokal) oder besser die Datei `~/.pgpass` mit Zeilen der Form `host:port:db:user:passwort` und Rechten `600`.

Nützliche Umgebungsvariablen, die die Flags ersetzen: `PGHOST`, `PGPORT`, `PGUSER`, `PGDATABASE`.

## Der zentrale Unterschied: Meta-Kommandos vs. SQL

Alles, was mit **Backslash** beginnt, ist ein **psql-Meta-Kommando** — es wird vom Client interpretiert und nie an den Server geschickt. Alles andere ist SQL.

Daraus folgen zwei praktische Konsequenzen:

- **Meta-Kommandos brauchen kein Semikolon**, SQL schon. Ein vergessenes Semikolon ist der häufigste Grund, warum psql „nichts tut" — es wartet auf den Rest der Anweisung. Der Prompt zeigt das an: `pricing=#` heißt bereit, `pricing-#` heißt „Anweisung noch nicht beendet".
- **Meta-Kommandos funktionieren nur in psql.** In einer `.sql`-Datei, die von einem anderen Werkzeug ausgeführt wird, scheitern sie.

## Die Meta-Kommandos, die zählen

| Kommando | Wozu |
|---|---|
| `\?` | Hilfe zu allen Meta-Kommandos |
| `\h SELECT` | SQL-Syntaxhilfe zu einer Anweisung |
| `\l` | Datenbanken auflisten |
| `\c dbname` | Datenbank wechseln |
| `\dt` | Tabellen · `\dt+` mit Größe |
| `\d tabelle` | **Spalten, Typen, Indizes, Constraints** — das meistgenutzte Kommando überhaupt |
| `\d+ tabelle` | dasselbe plus Speicherdetails und Kommentare |
| `\di` / `\dv` / `\dm` | Indizes / Views / Materialized Views |
| `\df` / `\dn` | Funktionen / Schemas |
| `\x` | **Expanded Display** — schaltet auf Zeile-pro-Spalte um. Unverzichtbar bei breiten Ergebnissen |
| `\timing` | Zeigt nach jeder Anweisung die Ausführungsdauer |
| `\o datei` | Lenkt die Ausgabe in eine Datei · `\o` ohne Argument schaltet zurück |
| `\i datei.sql` | Führt eine Datei aus |
| `\e` | Öffnet die letzte Anweisung im `$EDITOR` |
| `\copy tabelle FROM 'datei.csv' WITH (FORMAT csv, HEADER true)` | **Clientseitiger** CSV-Import |
| `\set name wert` / `:name` | Variablen setzen und einsetzen |
| `\pset pager off` | Schaltet den Pager ab (nützlich bei langen Plänen) |
| `\watch 5` | Wiederholt die letzte Anweisung alle 5 Sekunden |
| `\q` | Beenden |

> [!important] `\copy` vs. `COPY`
> **`COPY`** ist SQL und liest **serverseitig** — die Datei muss auf dem Datenbankserver liegen, und der Serverprozess braucht Leserechte darauf. Im Container heißt das: Die Datei muss *im Container* liegen.
>
> **`\copy`** ist ein psql-Meta-Kommando und liest **clientseitig** — psql öffnet die Datei dort, wo psql läuft, und schiebt den Inhalt durch die Verbindung. Das ist fast immer das, was man will.
>
> Dritte Variante für Pipes: `COPY … FROM STDIN` mit Umleitung von außen.

## Kommandozeilen-Flags

| Flag | Wozu |
|---|---|
| `-c "SQL"` | Eine Anweisung ausführen und beenden |
| `-f datei.sql` | Eine Datei ausführen und beenden |
| `-v name=wert` | Variable setzen, im Skript als `:name` |
| `-q` | Still — keine `BEGIN`/`COMMIT`-Statusmeldungen |
| `-t` | Nur Datenzeilen, kein Kopf, kein „(n rows)" |
| `-A` | Unausgerichtet — keine Spaltenrahmen und -auffüllung |
| `-X` | `.psqlrc` ignorieren (für reproduzierbare Skripte) |
| `-E` | Zeigt das SQL hinter jedem Meta-Kommando — sehr lehrreich |
| `-1` | Die ganze Datei in **einer** Transaktion ausführen |

**`-qtA` ist die Standardkombination**, wenn die Ausgabe maschinenlesbar sein soll — etwa um einen Plan in eine Datei zu schreiben. Ohne `-t -A` bekommt man ASCII-Rahmen und umbrochene Zeilen.

## Die Rezepte aus diesem Kapitel

```bash
# Skript ausführen, Ausgabe in eine Datei
docker compose exec -T db psql -U pricing -d pricing -qtA < skript.sql > ausgabe.txt

# CSV importieren, Datei liegt auf dem Host
docker compose exec -T db psql -U pricing -d pricing \
  -c "COPY recommendations_csv FROM STDIN WITH (FORMAT csv, HEADER true)" \
  < price_recommendations.csv

# Mit psql auf dem Host
psql -h localhost -U pricing -d pricing \
  -c "\copy recommendations_csv FROM 'price_recommendations.csv' WITH (FORMAT csv, HEADER true)"
```

`-T` bei `docker compose exec` schaltet die TTY-Zuweisung ab — ohne das funktioniert die Eingabeumleitung `<` nicht.

## Doku

- **Referenz:** <https://www.postgresql.org/docs/current/app-psql.html> — vollständig, gut gegliedert, die Meta-Kommandos stehen unter „Meta-Commands"
- **`COPY`:** <https://www.postgresql.org/docs/current/sql-copy.html>

---

# Teil 2 — EXPLAIN

## `EXPLAIN` vs. `EXPLAIN ANALYZE`

| | Was passiert | Was du bekommst |
|---|---|---|
| `EXPLAIN` | Query wird **nicht ausgeführt** | Nur der **geplante** Ablauf mit Schätzungen |
| `EXPLAIN ANALYZE` | Query wird **wirklich ausgeführt** | Plan **plus** tatsächliche Zeiten und Zeilenzahlen |

> [!danger] Bei schreibenden Anweisungen
> `EXPLAIN ANALYZE` auf `INSERT`, `UPDATE`, `DELETE` oder `CREATE TABLE AS` **führt sie aus**. Immer in eine Transaktion klammern:
> ```sql
> BEGIN;
> EXPLAIN (ANALYZE, BUFFERS) INSERT INTO … ;
> ROLLBACK;
> ```

## Die Optionen

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, SETTINGS, FORMAT JSON) SELECT … ;
```

| Option | Zeigt |
|---|---|
| `ANALYZE` | Tatsächliche Ausführung: `actual time`, `rows`, `loops` |
| `BUFFERS` | Gelesene und geschriebene Blöcke — der I/O-Blick |
| `VERBOSE` | Ausgabespalten je Knoten, Schemaqualifizierung |
| `SETTINGS` | Von der Voreinstellung abweichende Parameter (`work_mem`, `jit` …) |
| `WAL` | Erzeugte WAL-Menge bei schreibenden Anweisungen |
| `TIMING off` | Schaltet die Zeitmessung ab — reduziert den Messaufwand bei sehr vielen Knoten |
| `COSTS off` | Blendet Schätzungen aus, gut für Tests und Diffs |
| `FORMAT JSON` | Maschinenlesbar, mehr Details, Eingabeformat für Analysewerkzeuge |

**Der Standard für die Fehlersuche ist `(ANALYZE, BUFFERS)`.** `SETTINGS` lohnt zusätzlich, sobald man mit Parametern experimentiert — dann steht im Plan, mit welcher Konfiguration er entstanden ist.

## Wie ein Plan aufgebaut ist

Ein Plan ist ein **Baum**, dargestellt durch Einrückung. Gelesen wird er **von innen nach außen**: Die am tiefsten eingerückten Knoten laufen zuerst, ihr Ergebnis fließt nach oben.

```text
Hash Right Join                    ← läuft zuletzt, liefert das Endergebnis
  ->  Finalize GroupAggregate      ← Probe-Seite
        ->  Sort
              ->  Partial HashAggregate
                    ->  Nested Loop
  ->  Hash                         ← Hash-Seite, muss vor dem Join fertig sein
        ->  Seq Scan on capacity
```

## Eine Knotenzeile lesen

```text
Seq Scan on capacity c  (cost=0.00..23067.60 rows=1327560 width=18)
                        (actual time=0.019..161.811 rows=1327560.00 loops=1)
```

| Feld | Bedeutung |
|---|---|
| `cost=0.00..23067.60` | Geschätzte Kosten: **Startkosten..Gesamtkosten**, in willkürlichen Einheiten. Nur zum Vergleich von Plänen brauchbar — **keine Millisekunden** |
| `rows=1327560` | **Geschätzte** Zeilenzahl |
| `width=18` | Geschätzte durchschnittliche Zeilenbreite in Byte |
| `actual time=0.019..161.811` | **Tatsächlich**: Zeit bis zur ersten Zeile .. bis zur letzten, in Millisekunden |
| `rows=1327560.00` | **Tatsächliche** Zeilenzahl |
| `loops=1` | Wie oft dieser Knoten ausgeführt wurde |

> [!warning] Die `loops`-Falle
> **`actual time` und `rows` sind Werte *pro Durchlauf*, nicht Summen.** Ein Knoten mit `actual time=0.5 rows=3 loops=869882` hat nicht 0,5 ms gebraucht, sondern rund 435 Sekunden — und 2,6 Mio. Zeilen geliefert.
>
> Dasselbe gilt bei paralleler Ausführung: Dort ist `loops` die Zahl der Prozesse, und die Werte sind Durchschnitte je Worker.
>
> Das ist der häufigste Lesefehler überhaupt. Man sucht die große Zahl und übersieht die kleine, die millionenfach läuft.

## Knotentypen

**Scans**

| Knoten | Was passiert | Einordnung |
|---|---|---|
| **Seq Scan** | Ganze Tabelle lesen | Bei kleinen Tabellen oder hoher Trefferquote **richtig**. Bei „3 von 10 Mio." ein Alarmzeichen |
| **Index Scan** | Über den Index, dann jede Trefferzeile aus dem Heap holen | Gut bei wenigen Treffern |
| **Index Only Scan** | Nur der Index, **kein** Heap-Zugriff | Optimal — braucht alle gelesenen Spalten im Index |
| **Bitmap Index Scan** + **Bitmap Heap Scan** | Erst Zeilenadressen sammeln, dann den Heap in physischer Reihenfolge lesen | Der Mittelweg bei mittlerer Trefferzahl |

**Joins**

| Knoten | Wie | Wann gewählt |
|---|---|---|
| **Nested Loop** | Für jede Zeile links die rechte Seite durchsuchen | Wenige Zeilen links, passender Index rechts. Bei Fehlschätzung der Klassiker für „läuft seit Stunden" |
| **Hash Join** | Eine Seite in eine Hashtabelle, andere durchlaufen | Standard bei großen Mengen und Gleichheitsbedingung |
| **Merge Join** | Beide Seiten sortiert parallel durchlaufen | Wenn beide Seiten bereits sortiert vorliegen |

Bei `Hash Join`/`Hash Right Join` sagt der **`Hash`-Kindknoten**, welche Seite in den Speicher gebaut wird.

**Weitere häufige Knoten**

`Sort` · `HashAggregate` / `GroupAggregate` (bei `GROUP BY`) · `WindowAgg` (Window Functions) · `Gather` / `Gather Merge` (parallele Ausführung) · `Materialize` · `Memoize` (cacht Ergebnisse für wiederholte Parameter) · `Append` (bei `UNION ALL`) · `Function Scan` · `Result`.

## `BUFFERS` lesen

```text
Buffers: shared hit=9762 read=30, temp read=11877 written=17985
```

| Feld | Bedeutung |
|---|---|
| `shared hit` | Block lag in Postgres' eigenem Puffer (`shared_buffers`) |
| `shared read` | Block **nicht** dort — beim Betriebssystem angefordert |
| `shared dirtied` / `written` | Blöcke verändert bzw. auf die Platte geschrieben |
| `temp read` / `written` | **Temporäre Dateien** — ein Sort oder Hash ist auf die Platte ausgewichen |
| `local` | Wie `shared`, aber für Temp Tables |

> [!important] `read` heißt nicht „von der Platte"
> Es heißt nur: **nicht in `shared_buffers`**. Das Betriebssystem liefert den Block meist aus seinem eigenen Page Cache — ebenfalls RAM. Ob echte Platten-I/O stattfand, steht dort nicht.
>
> Um es zu sehen: `SET track_io_timing = on;` — dann erscheint zusätzlich `I/O Timings: shared read=… ms`. Ist der Wert nahe null bei hohem `read`, kam alles aus dem Cache.

## Die vier Diagnosen

**1. Schätzung gegen Wirklichkeit.** `rows=` vs. `actual rows=`. Weichen sie um mehr als etwa Faktor 10 ab, ist jede darauf aufbauende Entscheidung Zufall. Erste Maßnahme: `ANALYZE tabelle`. ⚠️ **Temp Tables müssen immer manuell `ANALYZE`t werden** — Autovacuum erfasst sie nie.

**2. Auslagerung auf die Platte.** `Sort Method: external merge Disk: …` · `Batches: 256` statt `1` · `Disk Usage: …kB` bei HashAggregate. Alles drei heißt: `work_mem` reicht nicht.

**3. Die `loops`-Falle.** Siehe oben. Immer `actual time × loops` rechnen.

**4. Wo die Zeit hingeht.** `actual time` ist **kumulativ**, enthält also die Kinder. Die Eigenzeit eines Knotens ist seine Zeit minus die seiner Kinder — genau die Rechnung, die einem die Analysewerkzeuge abnehmen.

## Werkzeuge

| Werkzeug | Wofür |
|---|---|
| [explain.dalibo.com](https://explain.dalibo.com) | Interaktiver Baum, markiert den teuersten Knoten, rechnet Eigenzeiten aus. Quelloffen als **pev2**, lokal betreibbar |
| [explain.depesz.com](https://explain.depesz.com) | Dasselbe Prinzip, tabellarische Darstellung |
| [pgMustard EXPLAIN-Glossar](https://www.pgmustard.com/docs/explain) | **Jedes Feld eines Plans einzeln erklärt**, nach Flag gruppiert. Das beste Nachschlagewerk, wenn irgendwo ein unbekanntes Feld steht |

> [!warning] Datenschutz
> Beide Web-Werkzeuge sind **externe Dienste**. Ein hochgeladener Plan enthält Query, Tabellen- und Spaltennamen, teilweise Literalwerte. Bei Übungsprojekten unkritisch, bei Kundenprojekten vorher klären — Schemanamen verraten oft mehr über ein Geschäftsmodell als beabsichtigt.

## Doku

- **`EXPLAIN`-Referenz:** <https://www.postgresql.org/docs/current/sql-explain.html> — alle Optionen
- **„Using EXPLAIN" (Kap. 14.1):** <https://www.postgresql.org/docs/current/using-explain.html> — die **beste Einführung**, mit durchgerechneten Beispielen. Drei Abschnitte: *EXPLAIN Basics*, *EXPLAIN ANALYZE*, *Caveats*. Die Caveats erklären, warum die gemessene Zeit von der echten Ausführungszeit abweichen kann
- **Kapitel 14 „Performance Tips":** <https://www.postgresql.org/docs/current/performance-tips.html> — Kontext drumherum: Statistiken, Planerkontrolle, schnelles Befüllen
- **Use The Index, Luke — Ausführungspläne:** <https://use-the-index-luke.com/sql/explain-plan> — dieselben Konzepte für vier Datenbanksysteme nebeneinander
- **pgMustard-Glossar:** <https://www.pgmustard.com/docs/explain>

## Wie man es tatsächlich lernt

Pläne lesen lernt man nicht durch Lesen über Pläne. Der Weg, der funktioniert:

1. Eine Query nehmen, deren Ergebnis man **kennt** — dann kann man Schätzungen gegen die Wahrheit halten
2. `EXPLAIN` ohne `ANALYZE` lesen und **vorhersagen**, welcher Knoten teuer wird
3. `EXPLAIN (ANALYZE, BUFFERS)` laufen lassen und prüfen, ob die Vorhersage stimmte
4. **Eine** Sache ändern — einen Index anlegen, `work_mem` erhöhen, die Query umformulieren — und beobachten, was sich im Plan bewegt

Schritt 2 ist der entscheidende. Wer nur den fertigen Plan liest, gewöhnt sich an, hinterher Erklärungen zu finden. Wer vorher rät, merkt, wo das eigene Modell falsch ist.
