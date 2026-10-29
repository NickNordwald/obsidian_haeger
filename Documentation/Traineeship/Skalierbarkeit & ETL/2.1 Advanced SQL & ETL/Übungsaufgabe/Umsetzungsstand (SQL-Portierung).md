---
tags:
  - documentation
---

# Umsetzungsstand — SQL-Portierung

Statusdokument zur Übungsaufgabe aus [[2.1 Advanced SQL & ETL - Leitfaden#2. Business-Logik nach SQL portieren & vergleichen|Aufgabe 2]]. Fahrplan: [[SQL-Portierung - Leitfaden]].

**Projekt:** `~/HaegerConsulting/Projects/Traineeship/Skalierbarkeit_ETL/AdvancedSQL`
**Stichtag aller Läufe:** 2026-09-07
**Datenmenge:** 1.327.560 Kapazitätszeilen · 939.559 Buchungen · 89.643 Empfehlungen

> [!info] Zur Entstehung
> Die Befunde unten stammen aus der gemeinsamen Durcharbeitung. Claudian hat sie festgehalten, damit sie nicht im Chat verloren gehen — das **Fazit in Abschnitt 8 schreibt Nick selbst**, es ist die eigentliche Bewertungsleistung der Aufgabe.

## Stand pro Schritt

| Schritt | Stand |
|---|---|
| 0 — Baseline messen | ✅ Python-Lauf mit festem Stichtag, Ergebnis als `price_recommendations.csv` |
| 1 — Nächte auffalten | ✅ `CROSS JOIN LATERAL generate_series` |
| 2 — Belegung aggregieren | ✅ |
| 3 — An Kapazität heften | ✅ inkl. Waisen-Anti-Join |
| 4 — Filtern und rechnen | ✅ Klassifizierungsspalte, alle Zählwerte bestätigt |
| 5 — Idempotent schreiben | ✅ `TRUNCATE` + `INSERT` in einer Transaktion, Fingerabdruck stabil |
| 6 — Äquivalenz nachweisen | ✅ **bewiesen**, siehe unten |
| 7 — Benchmarks | 🟡 **unvollständig** — eine Messung, keine Staffelung |
| 8 — Code-Review & Fazit | ❌ offen |

## Architektur der Lösung

Die SQL-Variante ist **keine einzelne Query**, sondern eine Prozedur aus mehreren Anweisungen mit Zwischenzustand:

1. `params` (Temp Table) — der Stichtag an genau einer Stelle
2. `einheiten` (Temp Table) — CTE-Kette: Nächte auffalten → aggregieren → an Kapazität heften → klassifizieren. **Nicht gefiltert**, enthält alle 1.327.560 Zeilen
3. `ANALYZE einheiten` — Statistiken für die Folgeschritte
4. `TRUNCATE` + `INSERT` in `recommendations_sql`, gefiltert auf `verworfen_weil IS NULL`

Alles in **einer Transaktion**, `SET LOCAL` für Messparameter.

**Zentrale Entwurfsentscheidung:** Eine `CASE`-Spalte `verworfen_weil` klassifiziert jede Zeile, statt sie wegzufiltern. Das bildet die „erste Bedingung gewinnt"-Semantik der `if/continue`-Kette in Python **strukturell** ab, macht die Zählwerte per `FILTER` in einem Durchlauf verfügbar und hält das Referenzdatum an einer Stelle.

## Äquivalenznachweis (Schritt 6)

Vergleich `recommendations_sql` ⟷ `recommendations_csv` mit `EXCEPT` in beide Richtungen.

**Exakt identisch für alle 89.643 Zeilen:** `product_id`, `calendar_date`, `days_until_travel`, `total_capacity`, `booked_units`.

→ Damit sind Nächte-Expansion, Aggregation, Join, Filterreihenfolge und Kapazitätsberechnung **beweisbar äquivalent**.

**Zählwerte stimmen mit dem Python-Log überein:**

| Kategorie | Wert |
|---|---|
| gesamt | 1.327.560 |
| `vergangen` (`calendar_date <= ref`) | 964.080 |
| `kapazitaet_null` | 273.816 |
| `kapazitaet_negativ` | 21 |
| Empfehlungen | 89.643 |

**Verbleibende Abweichung:** ~40 Zeilen in `recommended_adjustment_pct`, jeweils **genau ±0.000001** — eine Einheit in der letzten Stelle.

> [!note] Bewertung der Abweichung
> Ursache ist Pythons **Fließkomma-Zwischenrechnung** gegen **exakte Dezimalarithmetik** in SQL. Der Python-Code kommentiert das selbst: *„Rechnung in float … Damit hat man technisch gesehen die Rundungsfehler zwar schon gemacht."*
>
> Die SQL-Werte sind an diesen Stellen also die **genaueren**. Es wurde bewusst *keine* bitgenaue Äquivalenz angestrebt — das hätte bedeutet, den float-Pfad in SQL nachzubauen und eine saubere Implementierung zu verschlechtern, um ein Artefakt zu reproduzieren. Gewählt wurde **fachliche Äquivalenz mit dokumentierter Abweichung**.

**Typ-Entscheidung:** `occupancy` als `numeric(12,10)` in **beiden** Vergleichstabellen. Ohne Skalenbindung wichen ~10.500 Zeilen ab, weil Python die float-Darstellung schreibt (`0.029411764705882353`) und Postgres eine andere Stellenzahl erzeugt. Zehn Nachkommastellen liegen weit innerhalb der ~15–16 signifikanten Stellen eines `float` — es wird Rauschen weggerundet, keine Information.

## Performance-Befunde (Schritt 7, unvollständig)

**Messung bei 100 % Datenmenge**, `jit = off`, `work_mem = 256MB`: **3.838 ms** Execution Time.
Vorherige Messung mit JIT an und Standard-`work_mem`: 2.904 ms.

> [!warning] Nicht zuordenbar
> Bei der zweiten Messung wurden **zwei Parameter gleichzeitig** verändert, und zusätzlich änderte sich die Planform (`Hash Left Join` → `Hash Right Join`). Der scheinbare Rückschritt lässt sich damit nicht erklären. Möglich ist, dass JIT bei dieser Last hilft. **Für eine belastbare Aussage fehlt die Messreihe mit je einer Variablen.**

### Zeitbudget aus dem Ausführungsplan

| Phase | Zeit | Anteil |
|---|---|---|
| Seq Scan `bookings` (I/O) | ~88 ms | **2 %** |
| Nächte-Expansion | ~370 ms | 10 % |
| Aggregation (HashAgg → Sort → Gather → Finalize) | ~980 ms | 25 % |
| Hash-Aufbau über `capacity` | ~430 ms | 11 % |
| Join-Probe | ~620 ms | 16 % |
| Ausdrucksauswertung (`numeric`) | ~850 ms | **22 %** |
| Schreiben der Temp Table | ~610 ms | 16 % |

### Befund 1 — I/O ist nicht der Engpass

`shared read=16160` bedeutet **nicht** „von der Platte", sondern „nicht in `shared_buffers`" — geliefert wird in der Regel aus dem OS-Page-Cache, ebenfalls RAM. Der Scan kostet 88 von 3.838 ms. **Die Last ist CPU-gebunden, nicht I/O-gebunden.** Ein größeres `shared_buffers` hätte hier nichts gebracht.

### Befund 2 — 93 % der Arithmetik ist Verschwendung

`occupancy` und `recommended_adjustment_pct` werden für **alle 1.327.560** Zeilen berechnet, behalten werden 89.643. Zusätzlich steht der Divisionsausdruck **zweimal** in der `SELECT`-Liste.

→ **Maßnahme:** Nur `total_capacity` und `verworfen_weil` gehören in `einheiten` (ohne sie kann nicht gefiltert werden). Die beiden Rechenspalten gehören in das `INSERT … SELECT` **nach** dem `WHERE`.

### Befund 3 — `numeric` ist teuer

Postgres' `numeric` rechnet dezimal mit beliebiger Genauigkeit, heap-alloziert. Polars rechnete dieselbe Logik in nativen `f64` vektorisiert und lag **unter 1 s**. Die in Schritt 6 bewusst gewählte Exaktheit hat einen messbaren Preis.

### Befund 4 — `work_mem` ist erledigt

Mit 256 MB sind alle drei Disk-Spills verschwunden (`Sort Method: quicksort`, `Batches: 1`, keine `Disk Usage`). Weiteres Erhöhen bringt nichts.

### Weitere Beobachtungen

- **Memoize**: Postgres cacht `generate_series(1, nights)` nach `nights` — nur 12 verschiedene Werte, ~300.000 Cache-Treffer pro Worker. Ein Optimierungsschritt, den man in Python von Hand hätte bauen müssen.
- **Chronische Fehlschätzung**: `generate_series` mit variablem Argument wird immer mit 1000 Zeilen geschätzt (tatsächlich 3,57). Ab diesem Knoten sind alle Schätzungen im Plan wertlos — geschätzt 18,8 Mio., tatsächlich 200.653 Gruppen. Nicht behebbar, aber wichtig zu wissen.
- **Temp Tables brauchen immer manuelles `ANALYZE`** — Autovacuum erfasst sie nie. Ohne `ANALYZE params` schätzte Postgres 2.550 statt 1 Zeile, was die Kosten auf 1,34 Mrd. trieb und unnötig JIT auslöste.

## Optimierungsideen — geordnet nach erwartetem Hebel

Sammlung für später. **Keine davon ist gemessen**, außer wo vermerkt. Reihenfolge = erwarteter Nutzen.

### Groß — Datenmenge früh reduzieren

**1. Den Datumsfilter nach unten ziehen.** Von 1.327.560 Kapazitätszeilen fallen **964.080 (73 %)** wegen `calendar_date <= ref_date` weg — aber erst *nach* Hash-Aufbau, Join und dem Schreiben der Temp Table. Zieht man `calendar_date > ref_date` direkt in den Scan auf `capacity`, schrumpfen Hash-Tabelle, Join-Probe und Schreibvorgang auf ein Viertel.

Analog für die Buchungsseite: Nächte mit `night_date <= ref_date` können nie auf eine behaltene Kapazitätszeile treffen. Ein Filter in `exploded_nights` würde die 3,1 Mio. expandierten Nächte erheblich reduzieren — die Buchungsdaten reichen bis 2024 zurück, der Ergebniszeitraum beginnt erst 2026-09-08.

Das ist die Regel „Datenmenge früh reduzieren" aus [[2.1 Advanced SQL & ETL - Vertiefung#9.6 Klassische Performance-Fallen|Abschnitt 9.6]], angewendet auf die eigene Query. **Preis:** Der `vergangen`-Zählwert lässt sich dann nicht mehr im selben Durchlauf bilden.

**2. `einheiten` gar nicht materialisieren.** Aktuell werden 1,33 Mio. Zeilen geschrieben, zurückgelesen und davon 89.643 weitergeschrieben. Ein direktes `INSERT INTO recommendations_sql … SELECT … WHERE verworfen_weil IS NULL` mit der CTE-Kette schriebe nur einmal und nur die Zielzeilen. Der Schreibvorgang allein kostete im Plan ~610 ms. **Preis:** Die Zählquery bräuchte einen eigenen Durchlauf.

### Mittel

**3. Arithmetik hinter den Filter.** `occupancy` und `recommended_adjustment_pct` werden für 1,33 Mio. Zeilen berechnet, 89.643 werden behalten — **93 % Verschwendung**, ~850 ms im Plan. Zusätzlich steht der Divisionsausdruck zweimal in der `SELECT`-Liste. → Rechenspalten ins `INSERT … SELECT` nach dem `WHERE`.

**4. `double precision` statt `numeric`.** `numeric` rechnet dezimal mit beliebiger Genauigkeit und heap-alloziert; `float8` nutzt Maschinenbefehle. **Preis:** Man gibt die Exaktheit auf, die in Schritt 6 gerade der Grund war, dass die SQL-Werte die genaueren sind. Bewusste Abwägung, kein Selbstläufer.

**5. Den überflüssigen Sort beseitigen.** Wegen der `generate_series`-Fehlschätzung (18,8 Mio. geschätzt, 200.653 tatsächlich) wählt der Planer `Sort` → `Finalize GroupAggregate` statt einer Hash-Aggregation. ~250 ms. Diagnose: `SET LOCAL enable_sort = off` — wird es schneller, ist die Hypothese bestätigt. Sauberer Fix: `exploded_nights` in eine eigene Temp Table schreiben und `ANALYZE`n, damit der Planer echte Statistiken bekommt.

**6. Mehr Parallelität.** `max_parallel_workers_per_gather` steht standardmäßig auf **2**. Bei dieser Maschine ist mehr möglich.

**7. `UNLOGGED` für `recommendations_sql`.** Die Tabelle ist ein abgeleitetes Artefakt und jederzeit neu erzeugbar — sie braucht keine Crash-Sicherheit. `CREATE UNLOGGED TABLE` überspringt das WAL-Schreiben. Erkauft mit: Inhalt ist nach einem Absturz weg. Für eine Vergleichstabelle unkritisch.

### Klein

**8. Schmalere Temp Table.** `bookable_units`, `optional_units` und `verfuegbare_kapazitaet` werden nur für die Klassifizierung gebraucht und danach nicht mehr. Weniger Breite = weniger zu schreiben und zu lesen.

**9. `SET LOCAL synchronous_commit = off`** für den Lauf — spart das Warten auf den WAL-Flush beim Commit. Bei einem einzelnen Commit marginal.

### Geprüft und verworfen

| Idee | Warum nicht |
|---|---|
| **Index auf `bookings.status`** | **92,6 %** der Buchungen sind `'Firm'` — ein Index bei dieser Trefferquote wird vom Planer korrekt ignoriert. Kostet nur Schreibleistung. Faustregel: Indizes lohnen grob unter 5–10 % Selektivität |
| **`shared_buffers` erhöhen** | ✅ **gemessen widerlegt**: I/O kostet 88 von 3.838 ms (2 %). Die Last ist CPU-gebunden |
| **`work_mem` weiter erhöhen** | ✅ **gemessen erledigt**: Mit 256 MB sind alle drei Disk-Spills verschwunden |

### Die Grenze

Auch mit allen Punkten oben bleibt der Abstand zu Polars (~0,1 s) bestehen, und zwar aus **architektonischen** Gründen:

| | PostgreSQL | Polars |
|---|---|---|
| Ausführung | zeilenweise durch einen Knotenbaum (Volcano) | vektorisiert über ganze Spalten, SIMD |
| Layout | zeilenorientiert | spaltenorientiert (Arrow) |
| Zwischenergebnis | materialisiert | im Arbeitsspeicher |
| Bezahlt zusätzlich | MVCC, Transaktionen, Sperren, Crash-Sicherheit, Mehrbenutzerbetrieb | nichts davon |

Bei 1,33 Mio. Zeilen über ~8 Knoten führt Postgres über **10 Mio. Knotenaufrufe** aus. Der Vergleich ist Postgres' ungünstigster Fall und Polars' bester: eine einzelne In-Memory-Transformation ohne Nebenläufigkeit und ohne Dauerhaftigkeitsanforderung.

**3 Sekunden sind für eine zeilenorientierte, transaktionale Datenbank bei dieser Last normal.** Gemessen wird der Preis für Eigenschaften, die diese Aufgabe nicht braucht — nicht ein Defizit.

## Offene Punkte

- [ ] **Messreihe mit je einer Variablen** — Basis · nur `jit=off` · nur `work_mem` · beides
- [ ] **Staffelung über Datenmengen** (1 % / 10 % / 50 % / 100 %). ⚠️ Entlang der **`product_id`** samplen, nicht zeilenweise — sonst explodieren die Waisen und man misst ein anderes Problem. Deterministisches Kriterium, kein `random()`.
- [ ] **Befund 2 umsetzen und messen** — Arithmetik hinter den Filter
- [ ] **Befund 3 quantifizieren** — `::double precision` statt `::numeric`, sonst identisch
- [ ] **`track_io_timing = on`** zur Bestätigung, dass I/O nahe null ist
- [ ] Code-Review und Fazit (Abschnitt 8)

### Fachlicher Verdacht, nicht Teil der Portierung

`is_firm()` prüft ausschließlich `status == "Firm"` und ignoriert `cancellation_date`. Eine stornierte Buchung mit Status `Firm` würde weiterhin als belegt gezählt. **Bewusst 1:1 portiert**, um nicht zwei verschiedene Fachlogiken zu vergleichen. Sollte mit dem Fachbereich geklärt werden.

## 8. Code-Review und Fazit — *von Nick auszufüllen*

> [!question] Lesbarkeit
> Wie lange braucht ein neuer Kollege, um die Regel „negative Kapazität wird verworfen" zu finden — in Python vs. in SQL?

> [!question] Erweiterbarkeit
> Eine zweite Strategie neben `HeuristicRecommendationStrategy` ist in Python eine neue Klasse hinter einem `Protocol`. Wie sähe das in SQL aus?

> [!question] Wartbarkeit
> Wo liegen die Tests? Unit Tests für die reine Python-Formel, Data Tests für das SQL-Ergebnis — was passt wohin?

> [!question] Fehlerverhalten
> Python wirft `DomainError` bei `booked_units < 0`. Was macht SQL an dieser Stelle, und ist das besser oder schlechter?

> [!question] Wo liegt die Fachlogik?
> In Python an einer benannten Stelle in einer testbaren Klasse, in SQL mitten in einer Query. Was bedeutet das für ein Team, in dem Analysten SQL lesen können, aber kein Python?

> [!question] Das Fazit
> Es soll eine **Bedingung** enthalten, keinen Sieger: „SQL, wenn … · Python, wenn …"

## Gelernte Lektionen

- **Ankerzahlen vor dem Bauen ausrechnen.** `SUM(nights)` und `COUNT(*) FROM capacity` haben zwei Fehler sofort sichtbar gemacht, die sonst erst im Äquivalenztest aufgefallen wären.
- **NULL aus der nicht erhaltenen Seite eines `LEFT JOIN` ist die gefährlichste Fehlerklasse.** Ein Filter auf `night_date` statt `calendar_date` lieferte 4 statt 273.816 — plausibel genug, um unbemerkt zu bleiben, wäre die Erwartung nicht vorher bekannt gewesen.
- **Ein Alias steht auf derselben Query-Ebene nirgends zur Verfügung** — weder in `WHERE` noch in einem anderen Ausdruck derselben `SELECT`-Liste. Zweimal in dieser Aufgabe gestolpert.
- **Syntaxfehler gehören in die Datenbank, nicht ins Review.** Drei Iterationen mit fehlendem `FROM`, die `psql` in einer Sekunde gemeldet hätte.
- **Ein `UNIQUE`-Constraint ist ein kostenloser Data-Quality-Check.** Er hat den Join-Fanout-Test übernommen, ohne dass eine Prüfquery nötig war.
- **Erst messen, dann optimieren.** Die Vermutung war `shared_buffers`/RAM; tatsächlich sind es 2 % I/O und 22 % Arithmetik.
- **Eine Variable pro Messung** — sonst ist das Ergebnis nicht zuordenbar. Einmal verletzt, Ergebnis unbrauchbar.
