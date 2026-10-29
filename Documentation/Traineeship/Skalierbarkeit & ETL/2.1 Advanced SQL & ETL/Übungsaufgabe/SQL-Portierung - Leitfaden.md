---
tags:
  - documentation
---

# SQL-Portierung — Leitfaden

Der **einmalige** grobe Fahrplan für die Übungsaufgabe aus [[2.1 Advanced SQL & ETL - Leitfaden#2. Business-Logik nach SQL portieren & vergleichen|Aufgabe 2]]. Danach gilt: **Hilfe nur auf Nachfrage.**

Projektpfad: `~/HaegerConsulting/Projects/Traineeship/Skalierbarkeit_ETL/AdvancedSQL`

## Arbeitsmodus

> [!important] Wie wir ab jetzt zusammenarbeiten
> **Du schreibst den Code. Ich melde mich nicht von selbst** — kein ungefragtes Review, keine „mir ist noch aufgefallen"-Hinweise, keine Optimierungsvorschläge, bis du fragst.
>
> Wenn du fragst, antworte ich standardmäßig auf **Stufe 1**. Höher nur, wenn du nachhakst oder eine Stufe ausdrücklich anforderst:
>
> | Stufe | Was du bekommst |
> |---|---|
> | **1 — Denkanstoß** | Eine Leitfrage oder ein Hinweis, wo das Problem sitzt |
> | **2 — Konzept** | Die Mechanik erklärt, ohne Syntax für deinen Fall |
> | **3 — Syntax-Muster** | Syntax an einem **fremden** Datenmodell demonstriert |
> | **4 — Konkrete Lösung** | Angewendet auf deine Tabellen. Nur auf ausdrückliche Ansage. |
>
> Bei einem Fehler oder Traceback: erst Stufe 1 („was sagt dir die Meldung?"), dann eskalieren.
> **Ausnahme:** Bei echtem Datenverlust-Risiko (`DROP`/`DELETE` ohne `WHERE`, Migration gegen Produktivdaten) warne ich sofort und ungefragt.

## Zielbild

Am Ende existieren **zwei Implementierungen derselben Fachlogik**:

1. die bestehende in Python — `PriceableUnitBuilder` plus `RecommendationEngine`
2. eine neue in SQL, die dieselbe Ergebnismenge erzeugt

Dazu ein belastbarer Vergleich: **Äquivalenz nachgewiesen**, **Laufzeiten über mehrere Datenmengen gemessen**, **Code-Review** zu Lesbarkeit, Erweiterbarkeit und Wartbarkeit, und ein begründetes **Fazit**.

Erfolgskriterium ist nicht „SQL ist schneller" — sondern dass du **begründen kannst, wann welche Variante die richtige ist**.

## Die zu portierende Logik — in Worten

Damit klar ist, was überhaupt umzusetzen ist. Die Formeln stehen in deinem Code; hier steht nur, *was* passiert:

**Schritt A — Buchungen zu Nächten auffalten.** Jede Buchung mit Status `Firm` belegt ab `start_date` so viele aufeinanderfolgende Nächte, wie `nights` angibt. Aus **einer** Buchungszeile werden also *n* Nacht-Zeilen.

**Schritt B — Belegung zählen.** Pro `(product_id, Nacht)` zählen, wie viele Firm-Nächte darauf fallen → `booked_units`.

**Schritt C — An die Kapazität heften.** Jede Zeile aus `capacity` bekommt ihre `booked_units`. Kapazitätszeilen **ohne** Buchungen behalten 0 — das entspricht dem `Counter`, der für unbekannte Schlüssel 0 liefert. Buchungsnächte **ohne** passende Kapazitätszeile fallen weg und werden gezählt (im Python-Code die „verwaisten Nächte").

**Schritt D — Filtern.** Verworfen werden Zeilen mit `calendar_date <= Referenzdatum`, mit `total_capacity == 0` und mit negativer verfügbarer Kapazität (`bookable + optional < 0`). Die letzten beiden werden **gezählt und geloggt**.

**Schritt E — Rechnen.** `total_capacity`, `occupancy`, `days_until_travel` und daraus die Empfehlungsformel.

**Schritt F — Schreiben.** Ergebnis nach `price_recommendations`.

**Schritt G — Maximum.** Die Empfehlung mit dem höchsten `recommended_adjustment_pct`.

## Die Schritte

Jeder Schritt nennt **Ziel**, **die neue Technik** und ein **Abbruchkriterium** — woran du merkst, dass er fertig ist.

### Schritt 0 — Baseline messen

**Ziel:** Bevor du irgendetwas neu schreibst, weißt du, wie lange die Python-Variante braucht und was sie liefert.

**Tu das:** Vollen Lauf über die echten Daten, Laufzeit messen, Ergebnis in eine Vergleichstabelle schreiben (nicht in `price_recommendations` — eine eigene, z. B. `recs_python`). Zeilenzahl notieren, ebenso die beiden Skip-Zähler.

**Neue Technik:** keine.

**Fertig, wenn:** Du eine reproduzierbare Zahl für „Python braucht X Sekunden für Y Zeilen" hast und das Ergebnis persistiert ist.

> [!warning] Ohne Schritt 0 kein Schritt 6
> Ohne festgehaltenes Referenzergebnis kannst du Äquivalenz später nicht prüfen. Das ist der häufigste Fehler bei solchen Portierungen.

### Schritt 1 — Nächte auffalten

**Ziel:** Eine Query, die aus jeder Firm-Buchung *n* Zeilen macht — eine pro belegter Nacht.

**Neue Technik:** **`generate_series`** in Kombination mit einem Join auf die Buchungstabelle. Das ist die SQL-Antwort auf `[start_date + timedelta(days=n) for n in range(nights)]`. Übe es vorher in der PGExercises-„Date"-Sektion.

**Fertig, wenn:** Die Zeilenzahl der Ausgabe der Summe von `nights` über alle Firm-Buchungen entspricht. Das ist eine harte, prüfbare Zahl — rechne sie vorher aus.

> [!question] Denk vorher darüber nach
> Belegt eine Buchung mit `nights = 3` die Nächte `start`, `start+1`, `start+2` — oder `start+1` bis `start+3`? Schau in `occupied_nights()` nach und **schreib dir die Antwort auf**, bevor du die Query baust. Ein Off-by-one hier zieht sich durch alles Folgende.

### Schritt 2 — Belegung aggregieren

**Ziel:** Pro `(product_id, Nacht)` die Anzahl der Firm-Nächte.

**Neue Technik:** `GROUP BY` auf dem Ergebnis von Schritt 1 — also eine **CTE-Kette**. Genau der Anwendungsfall aus [[2.1 Advanced SQL & ETL - Vertiefung#8.3 CTEs|Abschnitt 8.3]].

**Fertig, wenn:** Die Summe aller Zählwerte wieder der Gesamtzahl der Firm-Nächte entspricht. Nichts darf verlorengehen.

### Schritt 3 — An die Kapazität heften

**Ziel:** Jede Kapazitätszeile mit ihrer Belegung, Kapazitätszeilen ohne Buchungen mit 0.

**Neue Technik:** **`LEFT JOIN` plus `COALESCE`.** Die Richtung des Joins ist die eigentliche Entscheidung — überleg dir, welche Seite vollständig erhalten bleiben muss.

**Fertig, wenn:** Die Ausgabe **genau so viele Zeilen** hat wie `capacity`. Mehr bedeutet Join-Fanout, weniger bedeutet, dass du versehentlich einen Inner Join gebaut hast.

**Zusätzlich:** Bau eine zweite, kleine Query für die **verwaisten Nächte** — Buchungsnächte ohne Kapazitätszeile. Das ist ein **Anti-Join**, und du brauchst die Zahl für den Äquivalenznachweis. Der Werkzeugkasten warnt dabei vor `NOT IN`.

### Schritt 4 — Filtern und rechnen

**Ziel:** Die drei Ausschlussregeln anwenden, die Kennzahlen berechnen.

**Neue Technik:** Nichts grundsätzlich Neues — aber hier lauern die meisten Fallen (siehe unten).

**Fertig, wenn:** Die Zeilenzahl mit der Python-Variante übereinstimmt **und** die beiden Skip-Zähler dieselben Werte liefern wie im Log des Baseline-Laufs. Für die Zähler brauchst du kein separates Query — [[2.1 Advanced SQL & ETL - Vertiefung#2.6 Bedingte Aggregation: `FILTER`|`FILTER`]] macht das in einem Durchlauf.

### Schritt 5 — Idempotent schreiben

**Ziel:** Das Ergebnis landet in einer Tabelle, und ein zweiter Lauf erzeugt keine Duplikate.

**Neue Technik:** Eines der drei Muster aus [[2.1 Advanced SQL & ETL - Vertiefung#5.4 Die drei Schreibmuster|Abschnitt 5.4]] — Partition Overwrite, `ON CONFLICT` oder `DELETE`+`INSERT` in **einer** Transaktion. Welches passt, hängt davon ab, ob dein Zielmodell einen brauchbaren eindeutigen Schlüssel hat. Schau dir dazu die Constraints auf `price_recommendations` an.

**Fertig, wenn:** Du den Lauf **dreimal hintereinander** ausführst und die Zeilenzahl unverändert bleibt. Das ist die praktische Definition von Idempotenz — und der Test dafür kostet zwei Minuten.

### Schritt 6 — Äquivalenz nachweisen

**Ziel:** Beweisen, dass beide Implementierungen dasselbe liefern. **Vor** jedem Performance-Vergleich.

**Neue Technik:** **`EXCEPT` in beide Richtungen.** `A EXCEPT B` zeigt, was A hat und B nicht; `B EXCEPT A` das Umgekehrte. Beide müssen leer sein. Siehe [[SQL-Werkzeugkasten#4. Das tägliche Kleinzeug|Mengenoperationen]].

**Fertig, wenn:** Beide Richtungen null Zeilen liefern — **oder** du jede Abweichung erklären kannst.

> [!important] Rechne mit Abweichungen bei `recommended_adjustment_pct`
> Der Python-Code rechnet die Prozentzahl bewusst in **`float`** und wandelt erst danach in `Decimal` um — der Kommentar im Code sagt das ausdrücklich. Wenn du in SQL mit `numeric` rechnest, bekommst du **andere Rundungsergebnisse**.
>
> Das ist keine Panne, sondern eine Entscheidung, die du treffen musst: Willst du **bitgenaue Äquivalenz** (dann muss SQL denselben Fließkommaweg gehen) oder **fachliche Äquivalenz** (dann vergleichst du mit einer Toleranz und dokumentierst, warum die SQL-Zahlen die *besseren* sind)? Beides ist vertretbar — aber die Entscheidung gehört in den Umsetzungsstand.

### Schritt 7 — Benchmarks

**Ziel:** Belastbare Aussagen über das Laufzeitverhalten beider Varianten über **verschiedene Datenmengen**.

**Aufbau:**

- **Staffelung:** mindestens vier Größen, logarithmisch gestaffelt — etwa 1 %, 10 %, 50 %, 100 % der Kapazitätszeilen. Du hast ~1,3 Mio.; wenn du den Trend über 100 % hinaus sehen willst, dupliziere die Daten künstlich hoch.
- **Wiederholungen:** jede Messung mindestens dreimal, **Median** statt Mittelwert (ein einzelner Ausreißer verzerrt sonst alles).
- **Aufwärmlauf:** die erste Messung verwerfen — Caches sind kalt, das misst die Platte, nicht die Logik.
- **Was du misst:** Wall-Clock-Zeit · den `EXPLAIN (ANALYZE)`-Plan der SQL-Variante · Spitzen-Speicherverbrauch der Python-Variante (dort liegen 1,3 Mio. Objekte im RAM).
- **Fair bleiben:** Zählt die Zeit für das Laden der Daten aus der DB nach Python mit? Argumente gibt es für beides — **entscheide dich, schreib es auf, und miss dann konsistent.**

**Fertig, wenn:** Du eine Tabelle *Datenmenge × Variante × Median-Laufzeit* hast und den **Verlauf** interpretieren kannst — nicht nur den Endpunkt. Die interessante Frage ist, ob die Kurven **unterschiedlich steil** sind.

> [!question] Die eigentliche Erkenntnis
> Erwarte nicht nur „SQL ist schneller". Interessanter ist: **Ab welcher Datenmenge kippt es?** Und: Skalieren beide linear, oder knickt eine ab? Ein Knick deutet meist auf Speicherdruck hin — bei Python auf RAM, bei SQL auf `work_mem` und Sortieren auf Platte.

### Schritt 8 — Code-Review und Fazit

**Ziel:** Die Bewertung, die der Leitfaden verlangt — Lesbarkeit, Erweiterbarkeit, Wartbarkeit.

Konkrete Fragen, an denen du beide Varianten misst:

- **Lesbarkeit:** Wie lange braucht ein neuer Kollege, um die Regel „negative Kapazität wird verworfen" zu finden?
- **Erweiterbarkeit:** Was kostet eine zweite Strategie neben `HeuristicRecommendationStrategy`? In Python ist das eine neue Klasse hinter einem `Protocol`. Wie sähe das in SQL aus?
- **Wartbarkeit:** Wo liegen die Tests? Was aus [[2.1 Advanced SQL & ETL - Vertiefung#6. Testing in ETL|Stichwort 6]] gilt für welche Variante — Unit Tests für die reine Python-Formel, Data Tests für das SQL-Ergebnis?
- **Fehlerverhalten:** Die Python-Variante wirft bei `booked_units < 0` einen `DomainError`. Was macht SQL an dieser Stelle — und ist das besser oder schlechter?
- **Wo liegt die Fachlogik?** In Python steht die Formel an einer benannten Stelle in einer testbaren Klasse. In SQL steht sie mitten in einer Query. Was bedeutet das für ein Team, in dem Analysten SQL lesen können, aber kein Python?

**Fertig, wenn:** Dein Fazit eine **Bedingung** enthält statt eines Siegers — „SQL, wenn …; Python, wenn …".

## Fallen in genau diesem Datenmodell

Ohne Lösung — damit du sie erkennst, wenn du hineinläufst:

| Falle | Woran du sie merkst |
|---|---|
| **Integer-Division** | `occupancy` kommt als 0 oder 1 heraus statt als Bruch. `SUM(a)/SUM(b)` mit Integer-Spalten ist in SQL Ganzzahldivision (siehe [[SQL-Werkzeugkasten#Casts und Zahlen]]). |
| **Off-by-one bei den Nächten** | Die Gesamtzahl der Nacht-Zeilen weicht von der Summe über `nights` ab. |
| **Join-Fanout** | Schritt 3 liefert mehr Zeilen als `capacity` hat. Fast immer, weil auf einen nicht eindeutigen Schlüssel gejoint wurde. |
| **`now()` statt Referenzdatum** | Der Lauf liefert morgen andere Ergebnisse. Verstoß gegen Idempotenz ([[2.1 Advanced SQL & ETL - Vertiefung#5.3 Die versteckten Brüche\|Abschnitt 5.3]]) — das Referenzdatum muss **hereingereicht** werden. |
| **`<=` vs. `<`** | Die Zeilenzahl weicht um genau die Anzahl der Zeilen eines Tages ab. Schau in `calculate_all` nach, welcher Operator dort steht. |
| **`float` vs. `numeric`** | Die letzten Nachkommastellen weichen ab — siehe Schritt 6. |
| **Nicht-deterministisches Maximum** | Gibt es mehrere Zeilen mit demselben höchsten `recommended_adjustment_pct`, ist die Auswahl willkürlich. `PriceRecommendation` vergleicht **ausschließlich** über dieses Feld — Python nimmt bei Gleichstand das erste, SQL irgendeines. Für Reproduzierbarkeit brauchst du einen eindeutigen Tiebreaker ([[2.1 Advanced SQL & ETL - Vertiefung#2.4 Die vier Funktionsfamilien\|Abschnitt 2.4]]). |
| **Stornierte Buchungen** | `is_firm()` prüft **nur** `status == "Firm"` und ignoriert `cancellation_date`. Das ist möglicherweise ein Bug im bestehenden Code. **Portiere es zunächst 1:1** — sonst vergleichst du zwei verschiedene Fachlogiken. Notier es als offenen Punkt. |

## Was nicht dazugehört

- **Kein Refactoring der bestehenden Python-Variante.** Sie ist die Referenz und bleibt unverändert.
- **Keine Optimierung, bevor gemessen wurde.** Erst Schritt 7, dann Indizes.
- **Keine Architekturänderung.** Die SQL-Variante darf zunächst als `text()`-Query neben der Domänenschicht stehen; sie sauber in die Schichten einzupassen ist eine spätere Frage.

## Begleitdokument

Der laufende Stand gehört in `Umsetzungsstand (SQL-Portierung).md` im selben Ordner — Arbeitsmodus, Ziele, Stand pro Schritt, offene Punkte, gelernte Lektionen. **Das schreibst du**, ich lege bei Bedarf nur das Gerüst an.
