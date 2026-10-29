---
tags:
  - documentation
---

# Leseliste — 2.1 Advanced SQL & ETL

Kuratierte Leseaufgaben zu [[2.1 Advanced SQL & ETL - Leitfaden]]. Pro Stichwort ein **Core**-Block (Pflicht, im Zeitbudget) und ein **Vertiefung**-Block (optional, falls Zeit übrig).

Ablauf: lesen → abhaken → Abschnitt in [[2.1 Advanced SQL & ETL - Vertiefung]] **aus dem Kopf** schreiben (~300–500 Wörter, ohne nochmal nachzuschlagen) → Review durch Claudian.

Offene Fragen beim Lesen wandern nach [[Q&A/Fragen]].

---
## Stichwort 1 — Daten-Pipelines: ETL, ELT & Batch Processing

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt, Vertiefung ausgearbeitet, Karten erstellt.

**Warum das zählt:** Das ist das Vokabular, in dem der Rest des Traineeships stattfindet. „Pipeline", „Batch", „ETL", „ELT" werden im Alltag ständig benutzt und oft unscharf — wer die Begriffe sauber trennt, versteht anschließend Architekturdiskussionen, Tool-Entscheidungen und Stellenausschreibungen. Der Abschnitt legt außerdem den Kontrast an, der sich durchs ganze Kapitel zieht: **zeilenweise-imperativ vs. mengenorientiert-deklarativ**.

**Zeitbudget Core: ~75 Min.**

### Core (Pflicht)

In dieser Reihenfolge lesen — sie geht von der Begriffsklärung über den historischen Ursprung zur Entscheidungspraxis.

- [x] **1. Densmore, *Data Pipelines Pocket Reference*, Kap. 1 + Kap. 3** — ~30 Min.
  Kap. 1 „Introduction to Data Pipelines" (What/Who/Why/How) und Kap. 3 „Common Data Pipeline Patterns", dort besonders *ETL and ELT*, *The Emergence of ELT over ETL* und *EtLT Subpattern*.
  Das kompakteste saubere Vokabular zum Thema, ohne Vendor-Bias.
  → *Leitfragen:* Was genau ist der Unterschied zwischen ETL und ELT — und was ist an **EtLT** anders als an beiden? Warum hat sich ELT überhaupt erst ab einem bestimmten Zeitpunkt durchgesetzt (was hat sich technisch geändert)?
  <https://www.oreilly.com/library/view/data-pipelines-pocket/9781492087823/ch03.html>

- [x] **2. Kleppmann, *Designing Data-Intensive Applications*, Kap. 10 — nur der erste Abschnitt** — ~20 Min.
  „Batch Processing with Unix Tools" lesen, **aufhören vor** „MapReduce and Distributed Filesystems".
  Klingt nach einem Umweg über alte Unix-Werkzeuge, ist aber die beste Erklärung überhaupt für die Frage, was eine Pipeline im Kern ausmacht: `awk | sort | uniq -c | sort -rn` *ist* eine Batch-Pipeline. Kleppmann arbeitet daran die Prinzipien heraus (uniforme Schnittstelle, Komposition, Immutability des Inputs, Wiederholbarkeit), die später in Spark und dbt genauso gelten.
  → *Leitfragen:* Warum lassen sich Unix-Tools so beliebig zusammenstecken — welche eine Designentscheidung macht das möglich? Was heißt es, dass der Input unverändert bleibt, und was gewinnt man dadurch? (Merk dir das — es ist die Vorbereitung auf *Idempotenz*, Stichwort 5.)
  <https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/ch10.html>

- [x] **3. Azure Architecture Center, „Extract, transform, load (ETL)"** — ~15 Min.
  Alle Abschnitte lesen **außer** den „Relevant service"-/„Technology choices"-Linklisten (reine Microsoft-Produktwerbung, inhaltlich wertlos für dich).
  Der praxisnächste Teil: explizite Entscheidungskriterien „Choose ETL when / Choose ELT when", dazu die Unterscheidung **control flow vs. data flow** und die klare Abgrenzung Batch ↔ Streaming.
  → *Leitfragen:* Nenne je zwei Situationen, in denen ETL bzw. ELT die richtige Wahl ist — und begründe warum. Was unterscheidet Control Flow von Data Flow? Was ist *Reverse* ETL und wozu?
  <https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/etl>

- [x] **4. dbt Labs, „ETL vs ELT: Key differences explained"** — ~10 Min.
  ~2.000 Wörter. **Bewusst mit Bias lesen:** dbt ist das Tool, das die ELT-Welle getragen hat, und argumentiert entsprechend. Genau deshalb interessant — du siehst die Pro-ELT-Argumentation in ihrer stärksten Form und kannst sie gegen die neutraleren Quellen 1–3 halten.
  → *Leitfrage:* Welches Argument für ELT hältst du nach den ersten drei Quellen für stichhaltig, welches für Marketing?
  <https://www.getdbt.com/blog/etl-vs-elt>

### Vertiefung (optional)

Nur wenn du schneller durch bist als geplant.

- [ ] **Reis & Housley, *Fundamentals of Data Engineering*, Kap. 8** — ~30 Min. · *Perspektivwechsel*
  „Queries, Modeling, and Transformation", Abschnitt zu Transformations/ETL vs. ELT. Derselbe Stoff, aber eingebettet in den **Data Engineering Lifecycle** (Generation → Storage → Ingestion → Transformation → Serving). Nützlich, wenn dir nach Quelle 1–4 noch das große Bild fehlt, wo Transformation eigentlich sitzt.
  <https://www.oreilly.com/library/view/fundamentals-of-data/9781098108298/ch08.html>

- [ ] **Kleppmann, DDIA Kap. 10 — der Rest** — ~60 Min. · *weiterführend*
  MapReduce, verteilte Filesysteme, Join-Algorithmen im Batch, Dataflow Engines (Spark, Flink). Das ist der direkte Unterbau für die späteren Kapitel zu Skalierbarkeit. Anspruchsvoll — nur angehen, wenn Zeit *und* Kopf frei sind.

- [ ] **Reis & Housley, *FDE*, Kap. 3 — Lambda & Kappa Architecture** — ~20 Min. · *weiterführend*
  Wie Batch und Streaming in einer Architektur koexistieren. Beantwortet die Anschlussfrage „und was, wenn ich beides brauche?".
  <https://www.oreilly.com/library/view/fundamentals-of-data/9781098108298/ch03.html>

- [ ] **Meltano, „ELT 101 — The Why and What of ELT"** — ~10 Min. · *Perspektivwechsel*
  Dieselbe Pro-ELT-These wie bei dbt, aber von einem anderen Anbieter mit anderem Produktinteresse. Gut, um herauszufiltern, welche Argumente **beide** bringen (wahrscheinlich substanziell) und welche nur einer (wahrscheinlich Positionierung).
  <https://meltano.com/blog/elt-101-the-why-and-what-of-elt-or-the-why-not-of-etl>

### Danach beantwortbar (Kontrollfragen aus dem Leitfaden)

Diese Fragen stelle ich dir nach deiner Zusammenfassung — schreib sie nicht vorab ab, sondern prüfe beim Lesen, ob du sie könntest:

- Was verbirgt sich hinter den Begriffen „Batch-Processing", „ETL" & „ELT"?
- Was genau sind eigentlich „Pipelines" in der Programmierung?
- Wodurch unterscheidet sich ETL-Code von klassischem Backend-Code? *(teilweise — der Rest kommt mit Stichwort 3/4)*

---

## Stichwort 2 — SQL (fortgeschritten)

> [!success] Abgeschlossen
> Core 1, 2, 4 vollständig · Core 3 (Tanimura) nach Teilkapitel abgebrochen · Core 5 vollständig plus zusätzliche Übung (gesamte *Aggregates*-Sektion durchgearbeitet).

> [!warning]- Zeitschätzung war zu niedrig — Konsequenz für künftige Listen
> Core 3 und Core 5 haben deutlich länger gedauert als veranschlagt. Zwei Fehler in der Kuration:
> 1. **Ein ganzes Buchkapitel als „~35 Min." angesetzt.** Tanimura Kap. 3 ist ein volles Analysekapitel mit vielen Beispielen. Künftig: Buchkapitel nur mit **Abschnitts- oder Seitenbereich** einplanen, nie als Ganzes — oder realistisch mit 60–90 Min. veranschlagen.
> 2. **Übungszeit ist nicht deckelbar.** „~45–60 Min. PGExercises" ist eine Fiktion, wenn die Sektion 20+ Aufgaben hat. Künftig: **Aufgabenanzahl** statt Zeit vorgeben („die ersten 8 Aufgaben"), damit ein klarer Endpunkt existiert.

> [!important] Zuschnitt
> Die SQL-**Grundlagen** sind in [[1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]] abgehakt: `SELECT`/`WHERE`, `GROUP BY` mit Aggregaten, `HAVING`, NULL und dreiwertige Logik, Joins und Unions, logische Ausführungsreihenfolge, Indizes, `EXPLAIN`. **Das wird hier nicht wiederholt.**
> Ebenfalls nicht hier: **komplexe Datentypen** (Struct/Array/Map) sind Stichwort 7, **CTEs/Subqueries/Views** sind Stichwort 8, **Performance-Fallen** sind Stichwort 9.
> Übrig bleibt genau das, was dir noch fehlt, um echte Business-Logik in SQL zu schreiben: **Window Functions**, bedingte und mehrstufige Aggregation, sowie die Konstrukte, mit denen man Schleifen loswird.

**Warum das zählt:** Das ist das Stichwort, an dem der Leitfaden dich misst — „GroupBys, Joins, Aggregationen und Window Functions sollten flüssig formuliert werden können, ohne viel Unterstützung von KI". Window Functions sind dabei der Wendepunkt: Vor ihnen muss man für „Umsatz dieses Monats vs. Vormonat" oder „Top 3 pro Kategorie" entweder Self-Joins basteln oder in Python schleifen. Mit ihnen wird daraus eine Zeile. **Das meiste, was du bisher in Python zeilenweise gelöst hast, wird hier zu einer Query.**

**Zeitbudget Core: ~2 Std. — davon ~1 Std. Übung.** Die Übung ist nicht der optionale Teil; „flüssig formulieren" lernt man nicht durch Lesen.

### Core (Pflicht)

- [x] **1. PostgreSQL-Doku, Tutorial 3.5 „Window Functions"** — ~15 Min.
  Der Einstieg. Kurz, mit gut gewählten Beispielen. Hier geht es nur ums Konzept: Was ist ein *Window*, was macht `OVER`, was `PARTITION BY`, und warum ist eine Window Function **kein** `GROUP BY`.
  → *Leitfragen:* Was ist der eine entscheidende Unterschied zwischen `GROUP BY` und einer Window Function im Ergebnis? Warum darf man eine Window Function nicht in `WHERE` benutzen — und was folgt daraus für „nimm nur die Top 3 pro Gruppe"?
  <https://www.postgresql.org/docs/current/tutorial-window.html>

- [x] **2. PostgreSQL-Doku, 9.22 „Window Functions"** — ~15 Min.
  Die Referenz. Nicht auswendig lernen — einmal durchgehen und ein Gefühl dafür bekommen, was es gibt: `row_number`, `rank`, `dense_rank`, `lag`, `lead`, `first_value`, `last_value`, `nth_value`, `ntile`. Dazu der wichtige Teil: die **Frame-Spezifikation** (`ROWS` / `RANGE` / `GROUPS`).
  → *Leitfragen:* Was ist der Default-Frame, wenn du `ORDER BY` im `OVER` schreibst — und warum liefert `last_value()` deshalb oft ein überraschendes Ergebnis? Wo ist der Unterschied zwischen `rank` und `dense_rank`?
  <https://www.postgresql.org/docs/current/functions-window.html>

- [/] **3. Tanimura, *SQL for Data Analysis*, Kap. 3 „Time Series Analysis"** — ~35 Min.
  Der wichtigste Core-Text. Kein Referenzmaterial, sondern **angewandte Muster**: gleitende Durchschnitte, laufende Summen, Vergleich mit Vorperiode, Wachstumsraten. Genau die Sorte Logik, die du bisher in Python geschrieben hättest.
  → *Leseauftrag:* Nimm dir bei jedem Beispiel 10 Sekunden und überleg, **wie du dasselbe in Python geschrieben hättest** — und wie viele Zeilen das gewesen wären.

- [x] **4. modern-sql.com, „The FILTER clause"** — ~10 Min.
  Bedingte Aggregation. Statt `SUM(CASE WHEN status='paid' THEN betrag ELSE 0 END)` schreibt man `SUM(betrag) FILTER (WHERE status='paid')`. Klein, aber du wirst es täglich brauchen — und es ist der saubere Weg, mehrere verschiedene Kennzahlen in **einem** Durchlauf über die Tabelle zu berechnen.
  → *Leitfrage:* Warum ist ein Durchlauf mit mehreren `FILTER`-Aggregaten besser als mehrere Queries, die je einmal über die Tabelle gehen?
  <https://modern-sql.com/feature/filter>

- [x] **5. PGExercises — Sektion „Aggregates"** — ~45–60 Min. · **Praxis**
  Über 80 Aufgaben insgesamt, die Aggregates-Sektion deckt Gruppierung *und* Window Functions ab. Datensatz ist ein Country Club mit Mitgliedern, Anlagen und Buchungen. Direkt im Browser lösbar, mit Musterlösung und Erklärung.
  → *Regel:* Erst selbst versuchen, dann die Lösung aufdecken. Wenn du eine Aufgabe nach 10 Minuten nicht knackst: Lösung ansehen, verstehen, **schließen und aus dem Kopf neu tippen**.
  <https://pgexercises.com/questions/aggregates/>

### Vertiefung (optional)

- [ ] **PostgreSQL-Doku, 7.2.4 „GROUPING SETS, CUBE, and ROLLUP"** — ~15 Min. · *weiterführend*
  Mehrere Aggregationsebenen in einer Query (z. B. Umsatz pro Produkt, pro Kategorie und gesamt gleichzeitig) statt drei Queries mit `UNION ALL`. Im Reporting ständig gebraucht.
  <https://www.postgresql.org/docs/current/queries-table-expressions.html>

- [ ] **PostgreSQL-Doku: `DISTINCT ON` und `LATERAL`** — ~20 Min. · *weiterführend*
  Zwei Postgres-Spezialitäten mit hohem Alltagsnutzen. `DISTINCT ON` löst „neuester Eintrag pro Kunde" ohne Window Function. `LATERAL` erlaubt einer Subquery, auf Spalten der linken Tabelle zuzugreifen — das SQL-Äquivalent einer verschachtelten Schleife, aber mengenorientiert.
  <https://www.postgresql.org/docs/current/sql-select.html>

- [ ] **Molinaro, *SQL Cookbook* (O'Reilly, 2. Aufl.)** — selektiv, ~30 Min. · *Perspektivwechsel*
  Rezeptsammlung nach Problem statt nach Sprachkonstrukt sortiert. Kapitel zu Ranking und Reporting quer lesen. Nützlich, wenn du merkst, dass du das Konstrukt kennst, aber nicht siehst, wann du es einsetzen sollst.

- [ ] **modern-sql.com, „QUALIFY clause"** — ~5 Min. · *weiterführend*
  Der Filter auf Window-Function-Ergebnisse, den es in Postgres (noch) nicht gibt — aber in Snowflake und BigQuery. Zeigt im Umkehrschluss, warum man in Postgres den Umweg über eine Subquery/CTE braucht.
  <https://modern-sql.com/caniuse/qualify>

- [ ] **PGExercises — Sektion „Recursion"** — ~30 Min. · *weiterführend*
  Rekursive Queries (Hierarchien, Stücklisten, Pfade). Baut auf CTEs auf, die erst in Stichwort 8 drankommen — deshalb hier optional.
  <https://pgexercises.com/questions/recursive/>

### Kontrollfragen

Stehen ab sofort **direkt in der Vertiefung** unter [[2.1 Advanced SQL & ETL - Vertiefung#2. SQL (fortgeschritten)]] — dort die Antworten daruntersetzen. Lies sie **vor** dem Lesen einmal durch, dann weißt du, worauf du achten musst.

---

## Block A — Stichworte 3, 4 & 5: Deklarativ, funktional, Idempotenz

> [!info] Tagesplan
> Morgen: **Block A und B**. Danach wird anhand der tatsächlich gebrauchten Zeit neu geplant, statt weiter auf Schätzungen zu bauen. Blöcke C und D werden erst danach zugeschnitten.

> [!info] Warum diese drei zusammen
> Sie sind **ein** Gedankengang, kein dreifacher: SQL beschreibt das *Was* statt das *Wie* (**deklarativ**) → das geht nur, weil die Operationen keine Seiteneffekte haben und allein vom Input abhängen (**funktional**) → und genau deshalb kann man einen Lauf beliebig oft wiederholen, ohne dass sich das Ergebnis ändert (**Idempotenz**). Getrennt gelesen würde man dieselben Quellen dreimal anfassen.

**Warum das zählt:** Das ist der theoretische Unterbau unter allem, was du in Stichwort 1 und 2 praktisch gesehen hast. Nach diesem Block solltest du erklären können, *warum* eine unveränderte `access.log` und eine wiederholbare Pipeline dasselbe Prinzip sind — und warum ein `INSERT` in einem Batch-Job ein Bug ist.

**Zeitbudget Core: ~85 Min. Lesen + ~30 Min. Fragen.** Diesmal mit Abschnittsgrenzen statt ganzer Kapitel.

### Core (Pflicht)

- [x] **1. Kleppmann, DDIA Kap. 2 — Abschnitt „Query Languages for Data"** — ~25 Min. · *Stichwort 3*
  Nur dieser Abschnitt: von „Query Languages for Data" bis vor „Graph-Like Data Models". Enthält den Vergleich imperativ vs. deklarativ, die **CSS-Analogie** (CSS-Selektor vs. dasselbe in JavaScript) und die Begründung, warum Deklarativität Parallelisierung und Optimierung überhaupt erst ermöglicht.
  → *Leitfragen:* Warum kann eine deklarative Query schneller werden, ohne dass man sie ändert? Warum lässt sich deklarativer Code leichter parallelisieren als imperativer — was genau steht bei einer `for`-Schleife im Weg?
  <https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/ch02.html>

- [x] **2. Python-Doku, „Functional Programming HOWTO" — nur der Abschnitt *Introduction*** — ~10 Min. · *Stichwort 4*
  **Aufhören vor „Iterators".** Das gesamte Dokument sind über 8.000 Wörter; gebraucht wird nur die Einleitung. Sie definiert **pure functions**, Seiteneffekte und Unveränderlichkeit in der Sprache, die du kennst.
  → *Leitfrage:* Was genau macht eine Funktion „rein" — und welche zwei Eigenschaften folgen daraus für ihre Testbarkeit?
  <https://docs.python.org/3/howto/functional.html>

- [x] **3. Beauchemin, „Functional Data Engineering — a modern paradigm for batch data processing"** — ~35 Min. · *Stichworte 4 + 5* · **Kerntext**
  Der einflussreichste Text zu diesem Thema, 2018 vom Airflow-Erfinder geschrieben. Überträgt funktionale Prinzipien auf Datenpipelines: **pure, idempotente Tasks**, **unveränderliche Partitionen** als atomare Einheit statt einzelner Zeilen, ein **persistenter Roh-Layer**, aus dem sich alles neu berechnen lässt.
  → *Leitfragen:* Warum ist die **Partition** und nicht die Zeile die richtige atomare Einheit? Was ist der Unterschied zwischen „reproduzierbar" und „wiederberechenbar" (*reproducibility* vs. *recomputability*), und warum braucht man beides?
  <https://maximebeauchemin.medium.com/functional-data-engineering-a-modern-paradigm-for-batch-data-processing-2327ec32c42a>

- [x] **4. Airflow-Doku, „Best Practices" — Abschnitte zu Idempotenz und Task-Design** — ~15 Min. · *Stichwort 5*
  Die operative Umsetzung in konkreten Regeln: `UPSERT` statt `INSERT` · immer eine **feste Partition** lesen und schreiben, nie „die neuesten Daten" · einen Task wie eine **Datenbanktransaktion** behandeln, also keine unvollständigen Ergebnisse hinterlassen · nichts im lokalen Dateisystem ablegen.
  → *Leitfrage:* Warum ist „lies die neuesten verfügbaren Daten" ein Idempotenz-Bruch, obwohl der Code selbst deterministisch aussieht?
  <https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html>

> [!note] Bewusste Verknappung bei Idempotenz
> Der Core deckt Idempotenz nur auf Konzeptebene plus Airflows operative Regeln ab. Tiefergehendes — Zustellgarantien, atomares Schreiben, Watermarking, Determinismus-Fallen — steht **nicht** in der Leseliste, wird aber von Claudian vollständig in den Vertiefungs-Abschnitt geschrieben. Du liest also einen Ausschnitt, bekommst aber die ganze Notiz. Die optionalen Quellen unten sind der Weg, es trotzdem selbst zu lesen.

### Vertiefung (optional)

- [ ] **Kleppmann, DDIA Kap. 11 — Abschnitt „Fault Tolerance" (Exactly-once execution)** — ~25 Min. · *weiterführend*
  Die Zustellgarantien at-most-once / at-least-once / exactly-once und warum letzteres in verteilten Systemen eigentlich *effectively-once* heißt. Die theoretische Untermauerung von allem, was Beauchemin praktisch fordert.
  <https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/ch11.html>

- [ ] **PostgreSQL-Doku: `INSERT … ON CONFLICT` (UPSERT) und `MERGE`** — ~15 Min. · *weiterführend*
  Die konkreten Sprachmittel hinter Airflows Regel „`UPSERT` statt `INSERT`".
  <https://www.postgresql.org/docs/current/sql-insert.html>

- [ ] **dbt-Doku: Incremental Models** — ~20 Min. · *weiterführend*
  Idempotenz im Alltagswerkzeug: Wie schreibt man ein Modell, das nur neue Daten verarbeitet und trotzdem bei jedem Rerun dasselbe Ergebnis liefert? Zeigt auch, wo der Kompromiss liegt — inkrementell ist schneller, aber schwerer korrekt zu halten als „alles neu bauen".
  <https://docs.getdbt.com/docs/build/incremental-models>

- [ ] **Kleppmann, DDIA Kap. 10 — der MapReduce-Teil** — ~40 Min. · *weiterführend*
  Der Rest des Kapitels aus Stichwort 1. MapReduce-Jobs sind das Lehrbuchbeispiel für pure Funktionen im Großen: deterministisch, seiteneffektfrei, deshalb beliebig wiederholbar und parallelisierbar. Schließt den Kreis zwischen Stichwort 1 und diesem Block.

- [ ] **ssp.sh, „Functional Data Engineering"** — ~10 Min. · *Perspektivwechsel*
  Kompakte Zusammenfassung derselben Ideen von anderer Hand. Gut zum Querlesen **nach** Beauchemin, um zu prüfen, was hängen geblieben ist.
  <https://www.ssp.sh/brain/functional-data-engineering/>

### Kontrollfragen

Stehen in der Vertiefung unter [[2.1 Advanced SQL & ETL - Vertiefung#Block A — Stichworte 3, 4 & 5]]. **Vor** dem Lesen einmal durchgehen.

---

## Block B — Stichwort 6: Testing in ETL

**Warum das zählt:** In [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]] hast du gelernt, Code zu testen. Hier kommt die zweite Achse dazu: **Daten** testen. Der Grund steht schon in [[2.1 Advanced SQL & ETL - Vertiefung#1.8 ETL-Code vs. klassischer Backend-Code|Abschnitt 1.8]] — eine Pipeline stürzt nicht ab, sie läuft grün durch und liefert falsche Zahlen. Gegen diese Fehlerklasse ist eine vollständige Unittest-Suite prinzipiell machtlos, weil der Fehler nicht beim Deploy entsteht, sondern wenn **neue Daten ankommen**.

**Zeitbudget Core: ~86 Min. Lesen + ~30 Min. Fragen.**

### Core (Pflicht)

- [x] **1. Thoughtworks, „Get back to basics with testing data pipelines: two orthogonal planes"** — ~8 Min.
  Ganzer Artikel, nur ~900 Wörter. Der Einstiegsrahmen fürs ganze Stichwort: Code-Ebene (Code wandert durch Environments → klassische Unittests) vs. Daten-Ebene (Daten fließen durch Transformationen → Data-Quality-Checks). Zwei rechtwinklige Achsen, die beide gebraucht werden.
  → *Leitfragen:* Warum reicht eine grüne Unittest-Suite in einer Datenpipeline nicht aus? Welche Testart gehört auf welche Achse?
  <https://www.thoughtworks.com/insights/blog/testing/get-back-to-basics-with-testing-data-pipelines-two-orthogonal-planes>

- [x] **2. Great Expectations, „Down with pipeline debt"** — ~10 Min.
  Lesen **bis einschließlich** „Test the pipeline's data, not just its structure". Die letzten zwei Abschnitte („Why GX does it best", „Collaboration is the key") sind Produktwerbung — überspringen.
  Kernargument: Die Komplexität einer Pipeline steckt in den **Daten**, die sie transportiert, nicht in ihrer Struktur.
  → *Leitfragen:* Was ist „Pipeline Debt" und wie unterscheidet sie sich von normaler Technical Debt? Warum kann ein Test, der zum Deploy-Zeitpunkt läuft, eine ganze Fehlerklasse prinzipiell nicht abfangen?
  <https://greatexpectations.io/blog/down-with-pipeline-debt-introducing-great-expectations/>

- [x] **3. dbt-Doku, „Add data tests to your DAG"** — ~18 Min.
  Von „Overview" **bis einschließlich** „Storing data test failures". Alles ab „New `data_tests:` syntax" sowie der FAQ-Block sind Migrationsdetails — überspringen.
  Das Grundprinzip ist elegant und knüpft direkt an deklarative Programmierung an: **Ein Data Test ist ein `SELECT`, das die fehlerhaften Zeilen zurückgibt — null Zeilen bedeutet grün.**
  → *Leitfragen:* Was unterscheidet einen *generic* (parametrisiert, wiederverwendbar) von einem *singular* Test? Wie sieht eine Assertion auf Daten technisch aus, und wie werden Fehlschläge für die Analyse persistiert?
  <https://docs.getdbt.com/docs/build/data-tests>

- [x] **4. dbt Developer Blog, „Unit testing in dbt for test-driven development"** — ~18 Min.
  Schwerpunkt auf dem Abschnitt **„Unit tests vs. model contracts vs. data tests"** — dort steht die Vergleichstabelle, die Kontrollfrage 9 direkt beantwortet. „Hello, unit testing world" und „Real world example" bei Zeitdruck überfliegen.
  Die Unterscheidungsachse ist der **Zeitpunkt**: Unit Test läuft *vor* der Materialisierung auf statischen Fixture-Daten und prüft **Logik**; Data Test läuft *danach* auf echten Daten und prüft **Dateneigenschaften**. Deine pytest-Fixtures aus 1.3 sind 1:1 übertragbar.
  → *Leitfragen:* Wann lohnt sich ein Unittest auf einer Transformation — und wann nicht? Worin unterscheiden sich Unit Test, Data Test und Model Contract in Zeitpunkt, Input und Zweck?
  <https://docs.getdbt.com/blog/announcing-unit-testing>

- [x] **5. Moses/Gavish/Vorwerck, *Data Quality Fundamentals*, Kap. 3** — ~30 Min.
  **Nur ab „Running Analytical Data Transformations" (S. 54) bis Kapitelende (S. 67).** Die erste Kapitelhälfte (Collecting/Cleaning/Normalizing) wird hier nicht gebraucht.
  Die einzige Quelle, die dbt, Great Expectations und Airflow-Checks **nebeneinander an denselben Beispieldaten** durchspielt — und daraus das **Circuit-Breaker**-Muster ableitet: Was passiert eigentlich, *wenn* ein Check fehlschlägt?
  → *Leitfragen:* An welchen Stellen der ETL-Strecke setzt man Checks an (Ingestion / Transformation / Output)? Was ist ein Circuit Breaker, und wie verhindert er, dass schlechte Daten nach downstream durchsickern?
  <https://www.oreilly.com/library/view/data-quality-fundamentals/9781098112035/ch03.html>

### Vertiefung (optional)

- [ ] **Astronomer, „Data quality and Airflow"** — ~15 Min. · *weiterführend*
  Von „Types of data quality challenges" bis Ende „Dag-level checks with Airflow", dann „Third-party frameworks" und „Quick reference". Den Block „Platform-level checks with Astro Observe" überspringen (Produktmarketing).
  Beantwortet die Platzierungsfrage: *wo im DAG welcher Check?* — inkl. Vergleichsmatrix der Frameworks.
  <https://www.astronomer.io/docs/learn/data-quality>

- [ ] **Great Expectations, „GX Core overview"** — ~6 Min. · *weiterführend*
  Das Vokabular eines Python-nativen (statt SQL-nativen) DQ-Frameworks: Data Context, Batch Definition, **Expectation** („a verifiable assertion about data"), Suite, Checkpoint.
  <https://docs.greatexpectations.io/docs/core/introduction/gx_overview>

- [ ] **Soda, „Data Quality Dimensions: The No-BS Guide"** — ~15 Min. · *weiterführend*
  Die Taxonomie: Completeness, Accuracy, Consistency, Freshness, Uniqueness, Validity und weitere — jede mit ausformulierter Check-Definition. Herstellernah, aber inhaltlich sauber; die SodaCL-Syntax ist nicht das Lernziel.
  → *Gute Frage dazu:* Was unterscheidet **Validity** (entspricht den Regeln) von **Accuracy** (entspricht der Realität) — und warum lässt sich nur eines davon automatisiert prüfen?
  <https://soda.io/blog/guide-to-data-quality-dimensions>

- [ ] **Start Data Engineering, „How to add tests to your data pipelines"** — ~7 Min. · *Perspektivwechsel*
  Derselbe Stoff wie Core 1 und 4, aber in **reinem Python/pytest** statt dbt/SQL — mit Fixture-Beispiel. Nennt außerdem die empfohlene Einführungsreihenfolge, wenn die Zeit knapp ist.
  <https://www.startdataengineering.com/post/how-to-add-tests-to-your-data-pipeline/>

> [!note]- Geprüft und verworfen
> - **Reis/Housley, *Fundamentals of Data Engineering*** — hat **kein** Kapitel zu Testing/Data Quality. Das Thema kommt nur verstreut in Halbseiten vor (DataOps-Undercurrent in Kap. 2, Trust in Kap. 9). Trägt die Kontrollfragen nicht. Erwähnt, weil es die naheliegende Vermutung gewesen wäre.
> - **dbt-Doku „Unit tests"** — gut, aber inhaltlich von Core 4 abgedeckt. Als **Nachschlagewerk für die Übungsaufgabe** empfehlenswert, besonders „When to add a unit test to your model".
> - **Dagster, „Write-Audit-Publish Pattern"** — inhaltlich stark, aber eher Pipeline-Design als Testing; überschneidet sich mit dem Circuit-Breaker-Teil aus Core 5. Guter Anschluss für später: <https://dagster.io/blog/python-write-audit-publish>
> - **Medium-Artikel zum Thema** — durchgehend nicht zuverlässig abrufbar (403/Paywall), deshalb keine aufgenommen.

### Kontrollfragen

Stehen in der Vertiefung unter [[2.1 Advanced SQL & ETL - Vertiefung#Block B — Stichwort 6: Testing in ETL]].

---

## Block C — Stichwort 7: Komplexe Datentypen

> [!success] Abgeschlossen
> Core vollständig. **Korrektur nachträglich:** Die Abschnittsnummern zu Core 3 (Postgres 8.14) waren falsch — 8.14.3 ist *Containment and Existence*, 8.14.4 ist *Indexing*, und der `json`/`jsonb`-Vergleich steht im Einleitungstext vor 8.14.1. Oben korrigiert.

> [!important] Vokabel-Orientierung vorweg
> Der Leitfaden nennt *„Struct/Composite types, Map, Array"* — das ist **gemischtes Vokabular aus zwei Welten**, und das zu wissen erspart Verwirrung:
>
> | Konzept | Postgres | Spark / Parquet / BigQuery |
> |---|---|---|
> | Verschachtelter Datensatz | **Composite Type** (`ROW`) | **StructType** / `STRUCT` / `RECORD` |
> | Liste gleichartiger Werte | **Array** (`integer[]`) | **ArrayType** / `ARRAY` / *repeated field* |
> | Schlüssel-Wert-Paare | **kein nativer Typ** — `hstore` oder `jsonb` | **MapType** / `MAP` |
> | Schemafreies Dokument | **`jsonb`** | JSON-Spalte, oft als Struct geparst |
>
> **Postgres hat keinen Map-Typ.** Die Erweiterung `hstore` und `jsonb` sind die Ersatzlösungen. Umgekehrt ist `jsonb` eine Postgres-Besonderheit, die es so in Spark nicht gibt. Wir behandeln beide Welten, weil du in Postgres arbeitest, aber im Data Engineering das Spark-Vokabular hörst.

**Warum das zählt:** Komplexe Datentypen brechen absichtlich mit der **1. Normalform** aus [[1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]] — dort galt „atomare Werte" als Gesetz. Hier lernst du, wann dieser Bruch die *richtige* Entscheidung ist und wann er dich in eine Sackgasse führt. Das ist die eigentliche Kontrollfrage des Leitfadens: **Pros und Cons.**

**Zeitbudget Core: ~72 Min. Lesen + ~25 Min. Fragen.**

### Core (Pflicht)

- [x] **1. PostgreSQL-Doku, 8.16 „Composite Types"** — ~12 Min.
  Abschnitte **8.16.1 bis 8.16.5**. Den letzten Abschnitt (8.16.6 Input/Output-Syntax) überspringen — reine Escaping-Details.
  Der Postgres-Struct: ein benannter Typ mit mehreren Feldern, als Spaltentyp verwendbar. Achte auf die **Klammer-Falle** beim Feldzugriff: `(meine_spalte).feld`, nicht `meine_spalte.feld`.
  → *Leitfragen:* Wie deklariert und wie liest man einen Composite Type? Warum braucht der Feldzugriff Klammern?
  <https://www.postgresql.org/docs/current/rowtypes.html>

- [x] **2. PostgreSQL-Doku, 8.15 „Arrays"** — ~20 Min.
  Bis einschließlich **8.15.5 („Searching in Arrays")**. Den Abschnitt zur Ein-/Ausgabesyntax überspringen.
  Wichtig sind: Deklaration, **1-basierte Indizierung** (nicht 0!), Slices, `unnest()`, `array_agg()`, `ANY`/`ALL` und der Containment-Operator `@>`.
  → *Leitfragen:* Wie sucht man performant nach einem Wert *innerhalb* eines Arrays? Was macht `unnest()` — und warum ist es die Umkehrung von `array_agg()`?
  <https://www.postgresql.org/docs/current/arrays.html>

- [x] **3. PostgreSQL-Doku, 8.14 „JSON Types"** — ~20 Min.
  Zuerst der **Einleitungstext von 8.14** (vor 8.14.1) — dort steht der Vergleich `json` vs. `jsonb`. Dann Schwerpunkt auf **8.14.2 (Designing JSON Documents)**, **8.14.3 (`jsonb` Containment and Existence)** und **8.14.4 (`jsonb` Indexing)**. Die Abschnitte 8.14.5 (Subscripting), 8.14.6 (Transforms) und 8.14.7 (jsonpath) überspringen — Nachschlagewerk.
  → *Leitfragen:* Was genau ist der Unterschied zwischen `json` und `jsonb` — in Speicherung, Geschwindigkeit und Funktionsumfang? Warum verliert `jsonb` die Schlüsselreihenfolge und Duplikate, und warum ist das meist gut?
  <https://www.postgresql.org/docs/current/datatype-json.html>

- [x] **4. Crunchy Data, „Indexing JSONB in Postgres"** — ~8 Min.
  Kurz (~1.300 Wörter). Der praktische Teil: Wann greift ein **GIN-Index**, wann nicht, und wann ist ein **Expression-Index** die bessere Wahl.
  → *Leitfrage:* Warum nutzt `WHERE data->>'status' = 'paid'` den GIN-Index **nicht**, `WHERE data @> '{"status":"paid"}'` aber schon? *(Diese eine Falle kostet in der Praxis am meisten Zeit.)*
  <https://www.crunchydata.com/blog/indexing-jsonb-in-postgres>

- [x] **5. Google Cloud, „BigQuery explained: Working with joins, nested & repeated data"** — ~12 Min.
  **Erst ab dem Abschnitt „Denormalizing Data with Nested and Repeated Structures" lesen.** Die erste Hälfte behandelt Joins und Star-Schemas — das kennst du aus [[1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]].
  Der analytische Blickwinkel: Verschachtelung als **Alternative zum Join**. Hier siehst du auch das Spark/BigQuery-Vokabular (STRUCT, ARRAY, `UNNEST`) und die Entwurfsrichtlinien.
  → *Leitfragen:* Warum ersetzt Verschachtelung im Warehouse einen Join — und was gewinnt man dadurch? Wann ist ein normalisiertes Schema trotzdem besser?
  <https://cloud.google.com/blog/topics/developers-practitioners/bigquery-explained-working-joins-nested-repeated-data>

### Vertiefung (optional)

- [ ] **Spark-Doku, „Data Types"** — ~10 Min. · *Perspektivwechsel*
  Die Referenz zu `StructType`, `ArrayType`, `MapType` samt `containsNull`/`valueContainsNull`. Dieselben Konzepte in dem Vokabular, das dir im Data Engineering ständig begegnen wird.
  <https://spark.apache.org/docs/latest/sql-ref-datatypes.html>

- [ ] **PostgreSQL-Doku, `hstore`** — ~10 Min. · *weiterführend*
  Postgres' Antwort auf den fehlenden Map-Typ: flache Schlüssel-Wert-Paare, beide als Text. Heute meist von `jsonb` verdrängt — aber gut zu wissen, warum es existiert und wann es schlanker ist.
  <https://www.postgresql.org/docs/current/hstore.html>

- [ ] **PostgreSQL-Doku, 9.16 „JSON Functions and Operators"** — nach Bedarf · *Nachschlagewerk*
  Nicht am Stück lesen. Die Referenz für `->`, `->>`, `#>`, `@>`, `jsonb_set`, `jsonb_array_elements` — beim Schreiben von Queries aufschlagen.
  <https://www.postgresql.org/docs/current/functions-json.html>

- [ ] **Devoteam, „Improve BigQuery Performance: Normalisation vs Denormalisation"** — ~10 Min. · *weiterführend*
  Vertieft die Abwägung aus Core 5 mit konkreten Größenschwellen (z. B. Dimensionstabellen ab ~10 GB denormalisieren).
  <https://www.devoteam.com/expert-view/improve-bigquery-performance-normalisation-vs-denormalisation/>

### Kontrollfragen

Stehen in der Vertiefung unter [[2.1 Advanced SQL & ETL - Vertiefung#Block C — Stichwort 7: Komplexe Datentypen]].

---

## Block D — Stichworte 8 & 9: Zwischenergebnisse, Performance & dimensionale Modellierung

> [!warning] Der größte Block — bewusst zweigeteilt
> Block D trägt **drei** Kontrollfragen des Leitfadens (Nr. 11, 12 und 13), davon eine Synthesefrage („alle Formen der Zwischenspeicherung"). Dazu kommt **Star-/Snowflake-Schema**, das seit Kapitel 1 als offener Punkt in [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|Kapitelabschluss - Q&A-Ernte]] steht.
>
> **Gesamt: ~120 Min. Lesen.** Das ist zu viel für eine Sitzung. Die zwei Teile unten sind eigenständig — nach **D1** kann man aufhören, und die Fragen zu D1 in der Vertiefung sind separat gruppiert.

---

### Teil D1 — Nicht materialisierte Zwischenergebnisse (~45 Min.)

**Warum das zählt:** Du hast bisher Subqueries benutzt, ohne die Alternativen zu kennen. Hier lernst du die drei Formen, ein Zwischenergebnis zu benennen, ohne es zu speichern — und warum die Wahl zwischen ihnen in Postgres seit Version 12 anders ausfällt als davor.

- [x] **1. PostgreSQL-Doku, 7.8 „WITH Queries" — Abschnitte 7.8.1 und 7.8.3** — ~20 Min.
  **7.8.1 (`SELECT` in `WITH`)** und **7.8.3 (Common Table Expression Materialization)**. Abschnitt 7.8.4 (Data-Modifying Statements) überspringen.
  7.8.3 ist der wichtigste Teil des ganzen Stichworts: Seit **PostgreSQL 12** wird eine CTE **inline gefaltet**, wenn sie nicht rekursiv, seiteneffektfrei und **nur einmal referenziert** ist — bei mehrfacher Referenz wird sie materialisiert. Mit `MATERIALIZED` / `NOT MATERIALIZED` kann man beides erzwingen.
  → *Leitfragen:* Was bedeutet „Optimization Fence", und warum war das vor PG 12 ein Problem? Wann willst du `MATERIALIZED` erzwingen — und wann `NOT MATERIALIZED`?
  <https://www.postgresql.org/docs/current/queries-with.html>

- [x] **2. PostgreSQL-Doku, 7.8.2 „Recursive Queries"** — ~15 Min.
  Die Unterabschnitte 7.8.2.1 (Search Order) und 7.8.2.2 (Cycle Detection) überfliegen, nicht auswendig lernen.
  Rekursive CTEs sind das einzige, was eine Subquery **prinzipiell nicht** kann — für Hierarchien, Stücklisten, Pfade.
  → *Leitfrage:* Woraus besteht eine rekursive CTE strukturell (Anker + rekursiver Teil), und was passiert ohne Abbruchbedingung?
  <https://www.postgresql.org/docs/current/queries-with.html>

- [x] **3. PostgreSQL-Doku, 3.2 „Views" + `CREATE VIEW` (nur die Beschreibung oben)** — ~10 Min.
  Der Schlüsselsatz steht in der Referenz: *„The view is not physically materialized. Instead, the query is run every time the view is referenced."* Eine View ist ein **gespeicherter Name für eine Query**, kein gespeichertes Ergebnis.
  → *Leitfrage:* Was gewinnt man durch eine View, wenn sie das Ergebnis gar nicht speichert?
  <https://www.postgresql.org/docs/current/tutorial-views.html> · <https://www.postgresql.org/docs/current/sql-createview.html>

---

### Teil D2 — Materialisierung, Performance & Modellierung (~75 Min.)

- [x] **4. PostgreSQL-Doku, `CREATE MATERIALIZED VIEW` und `REFRESH MATERIALIZED VIEW`** — ~15 Min.
  Beide Referenzseiten, jeweils nur den Beschreibungsteil.
  Achte auf drei Dinge: Eine Materialized View **merkt sich ihre Query** und kann später aufgefrischt werden · `REFRESH` ersetzt den **kompletten** Inhalt · `CONCURRENTLY` vermeidet die Lesesperre, **braucht aber einen Unique-Index** und ist langsamer.
  → *Leitfragen:* Wodurch unterscheidet sich `CREATE MATERIALIZED VIEW` von `CREATE TABLE AS`? Warum gibt es **keine** temporären Materialized Views?
  <https://www.postgresql.org/docs/current/sql-creatematerializedview.html> · <https://www.postgresql.org/docs/current/sql-refreshmaterializedview.html>

- [x] **5. PostgreSQL-Doku, `CREATE TABLE` — nur der `TEMPORARY`-Teil** — ~8 Min.
  Suche auf der Seite nach `TEMPORARY` und `ON COMMIT`. Der Rest ist Referenz.
  → *Leitfragen:* Wie lange lebt eine Temp Table, und wer sieht sie? Was bewirkt `ON COMMIT DROP` / `DELETE ROWS`?
  <https://www.postgresql.org/docs/current/sql-createtable.html>

- [x] **6. Use The Index, Luke! — Kapitel 2.3 „Functions" und 2.7 „Obfuscated Conditions"** — ~25 Min.
  Markus Winand, die beste freie Ressource zu SQL-Performance. **2.3** erklärt, warum eine Funktion um eine indizierte Spalte den Index ausschaltet — und was ein **Function-Based Index** dagegen tut. **2.7** ist eine Sammlung von Anti-Pattern (Datumsvergleiche, numerische Strings, kombinierte Spalten, „smart logic", Rechnen in der `WHERE`-Klausel).
  Das ist derselbe Mechanismus wie die GIN-Falle aus [[2.1 Advanced SQL & ETL - Vertiefung#7.5 Indizierung: GIN und die häufigste Falle|Stichwort 7]] — hier in seiner allgemeinen Form.
  → *Leitfragen:* Warum macht `WHERE lower(name) = 'meier'` den Index nutzlos? Was ist an `WHERE datum + interval '1 day' > now()` falsch, und wie schreibt man es richtig?
  <https://use-the-index-luke.com/sql/where-clause/functions> · <https://use-the-index-luke.com/sql/where-clause/obfuscation>

- [x] **7. dbt Labs, „A complete guide to dimensional modeling"** — ~18 Min. · **Nachtrag zu Kapitel 1**
  ~2.500 Wörter. Deckt **Star- und Snowflake-Schema**, Fakten- und Dimensionstabellen, den vierstufigen Entwurfsprozess und die Abwägungen ab.
  Das war seit Kapitel 1.2 ein offener Punkt und ist der Rahmen, in den Materialisierung gehört: Eine Materialized View oder Aggregatstabelle ist meist genau eine **vorberechnete Faktentabelle**.
  → *Leitfragen:* Was ist der Unterschied zwischen einem Fakt und einer Dimension? Was bedeutet **Grain**, und warum ist er die wichtigste Entwurfsentscheidung? Was normalisiert das Snowflake-Schema gegenüber dem Star-Schema — und was kostet das?
  <https://www.getdbt.com/blog/guide-to-dimensional-modeling>

### Vertiefung (optional)

- [ ] **Kimball, „Fact Tables and Dimension Tables" (2003)** — ~12 Min. · *Perspektivwechsel / Primärquelle*
  Der Originaltext von Ralph Kimball, am Beispiel eines Lebensmittelgeschäfts. Behandelt **Grain** deutlich gründlicher als der dbt-Artikel, dazu Slowly Changing Dimensions und degenerierte Dimensionen. Dichter, aber es ist die Quelle, auf die sich alle anderen beziehen.
  <https://www.kimballgroup.com/2003/01/fact-tables-and-dimension-tables/>

- [ ] **Use The Index, Luke! — 1.3 und 2.1.3 „Slow Indexes"** — ~15 Min. · *weiterführend*
  Warum ein vorhandener Index trotzdem langsam sein kann. Ergänzt die Index-Grundlagen aus [[1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]] um die Frage, warum `EXPLAIN` manchmal einen Seq Scan wählt, obwohl ein Index da ist.
  <https://use-the-index-luke.com/sql/anatomy/slow-indexes>

- [ ] **PostgreSQL-Wiki, „CTEReadme"** — ~10 Min. · *weiterführend*
  Die Entstehungsgeschichte des CTE-Inlinings von PG 12 aus Entwicklersicht. Interessant, wenn dich interessiert, *warum* der Optimization Fence so lange bestand.
  <https://wiki.postgresql.org/wiki/CTEReadme>

- [ ] **PGExercises — Sektion „Recursion"** — 5 Aufgaben · *Praxis*
  Der Übungsteil zu rekursiven CTEs, den wir in Stichwort 2 zurückgestellt hatten. Jetzt hast du das Rüstzeug.
  <https://pgexercises.com/questions/recursive/>

### Kontrollfragen

Stehen in der Vertiefung unter [[2.1 Advanced SQL & ETL - Vertiefung#Block D — Stichworte 8 & 9]], getrennt nach **D1** und **D2**.

---

## Nachtrag A — Kimball vs. Inmon (30–60 Min.)

**Warum:** Stichwort 9 hat Kimballs dimensionale Modellierung behandelt. Die Gegenposition fehlt — und die Frage „normalisierter Enterprise-Kern oder direkt dimensionale Data Marts?" ist die grundlegendste Architekturentscheidung im Data Warehousing.

Wähl **eine** der beiden Varianten:

- [ ] **Reis & Housley, *Fundamentals of Data Engineering*, Kap. 8 — Abschnitt „Modeling"** — ~30 Min. · *empfohlen*
  Behandelt **Inmon, Kimball und Data Vault** nebeneinander, eingebettet in den Data-Engineering-Lifecycle. Die beste Einzelquelle, weil sie alle drei vergleicht statt zwei gegeneinanderzustellen.
  <https://www.oreilly.com/library/view/fundamentals-of-data/9781098108298/ch08.html>

- [ ] **ssp.sh, „Inmon vs Kimball"** — ~8 Min. · *die schnelle Variante*
  Kompakt, mit Praktiker-Kommentaren. Ordnet auch **One Big Table (OBT)** ein, das in modernen Warehouses zunehmend auftaucht.
  <https://www.ssp.sh/brain/inmon-vs-kimball/>

> [!tip] Die Kernfrage im Kopf behalten
> **Inmon (top-down):** erst ein normalisierter Enterprise-Kern als *single source of truth*, daraus dann dimensionale Data Marts. Konsistent, aber langsam bis zum ersten Ergebnis.
> **Kimball (bottom-up):** direkt dimensionale Data Marts pro Geschäftsprozess, die über **konforme Dimensionen** zusammenwachsen. Schnell nutzbar, aber die unternehmensweite Konsistenz muss man sich erarbeiten.
> Die Praxis ist meist hybrid — und in ELT-Architekturen übernimmt der Roh-Layer faktisch Inmons Rolle als konsistente Quelle, ohne dessen Normalisierungsaufwand.

---

## Nachtrag B — Übungsaufgaben zum Festigen

> [!important] Warum das jetzt kommt
> Über vier Blöcke hinweg war das Muster identisch: **Konzepte sicher, getippte Queries fehlerhaft oder übersprungen.** Der Leitfaden misst „flüssig formulieren ohne KI-Hilfe" — das ist eine Fertigkeit, keine Wissensfrage. Die Aufgaben unten sind nach **Relevanz für die Übungsaufgabe** sortiert, nicht nach Schwierigkeit.

### Tier 1 — direkt relevant für die Portierung (~2,5 Std.)

- [x] **PGExercises — „Date"** (~45 Min.) · **höchste Relevanz**
  Datumsarithmetik, `generate_series`, Zeitreihen, Intervalle. Genau das, was du für die **Nächte-Expansion** einer Buchung brauchst (`start_date` + `nights` → eine Zeile pro Nacht). Wenn du nur eine Sektion machst: diese.
  <https://pgexercises.com/questions/date/>

- [x] **PGExercises — „Joins and Subqueries"** (~60 Min.)
  Inner/Outer Joins, Self-Joins, korrelierte Subqueries, `EXISTS`/`NOT EXISTS`. Der Anti-Join wird in der Aufgabe gebraucht, um **verwaiste Buchungsnächte** ohne Kapazitätszeile zu finden.
  <https://pgexercises.com/questions/joins/>

- [x] **PGExercises — „Recursive"** (~45 Min.)
  Die Praxis zu `WITH RECURSIVE`, die seit Stichwort 2 offen ist. Du hast rekursive CTEs verstanden und nie eine geschrieben.
  <https://pgexercises.com/questions/recursive/>

- [x] **`EXPLAIN (ANALYZE, BUFFERS)` auf der eigenen Datenbank** (~30 Min.) · *keine externe Quelle*
  Du hast **1,3 Mio. Kapazitätszeilen und 940 k Buchungen** — eine bessere Übungsumgebung gibt es nicht. Nimm dir fünf Queries unterschiedlicher Bauart vor und lies die Pläne mit der Checkliste aus [[SQL-Werkzeugkasten#Checkliste beim Plan-Lesen]]. Ziel ist nicht Optimieren, sondern **Pläne lesen können**.

### Tier 2 — breite Absicherung (~2–3 Std., falls Zeit)

- [ ] **Kaggle, „Advanced SQL"** (~3 Std., 4 Lektionen) — *aus dem Leitfaden*
  Joins & Unions · **Analytic Functions** (= Window Functions) · Nested and Repeated Data (= genau die komplexen Datentypen aus Stichwort 7) · Writing Efficient Queries. Deckt inhaltlich vier deiner Stichworte ab und ist der einzige strukturierte Kurs in der Liste.
  <https://www.kaggle.com/learn/advanced-sql>

- [ ] **PGExercises — „Modifying data"** (~30 Min.)
  `INSERT`/`UPDATE`/`DELETE`, und damit die Praxis zu `ON CONFLICT` und `RETURNING` aus dem [[SQL-Werkzeugkasten]]. Relevant für den **idempotenten Schreibschritt** der Aufgabe.
  <https://pgexercises.com/questions/updates/>

- [ ] **PGExercises — „String"** (~30 Min.)
  Textfunktionen und Regex — Alltagswerkzeug jeder ETL-Bereinigung.
  <https://pgexercises.com/questions/string/>

- [ ] **HackerRank — SQL Track** (nach Bedarf) — *aus dem Leitfaden*
  Interview-Stil, gemischte Schwierigkeit. Gut zum Warmlaufen, aber weniger systematisch als PGExercises und teilweise MySQL-Dialekt.
  <https://www.hackerrank.com/domains/sql>

- [ ] **DataLemur — Window-Function-Fragen** (~45 Min.)
  Die Grundfragen sind kostenlos, mit Lösungen und interaktivem Editor. Gezielte Übung zu Ranking, `LAG`/`LEAD` und Frames — deine schwächste getippte Stelle in Stichwort 2.
  <https://datalemur.com/questions>

### Tier 3 — zur Auflockerung

- [ ] **Knight Lab, „SQL Murder Mystery"** (~45 Min.)
  Ein Mordfall, gelöst durch Joins und Filter. Konzeptionell unter deinem Niveau, aber ein gutes Aufwärmen und macht Spaß.
  <https://mystery.knightlab.com/>

### Was diese Seiten *nicht* abdecken

Ehrlichkeitshalber: Für drei behandelte Themen gibt es keine guten Online-Aufgaben — **JSON/Arrays** (Stichwort 7), **`GROUPING SETS`** und **Materialized Views vs. Temp Tables** (Stichwort 9). Die übst du am besten **an der eigenen Datenbank**: eine `jsonb`-Spalte an eine Tabelle hängen und indizieren, eine Aggregatstabelle einmal als Matview und einmal als Temp Table bauen und die Pläne vergleichen.

> [!tip] Regel fürs Üben
> Erst selbst versuchen. Nach zehn Minuten ohne Fortschritt: Lösung ansehen, **verstehen**, dann **schließen und aus dem Kopf neu tippen**. Der letzte Schritt ist der, der zählt — ohne ihn hast du gelesen, nicht geübt.
