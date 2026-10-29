---
tags:
  - documentation
---

# Leseliste — 2.4 Apache Spark

Kuratierte Leseaufgaben zu [[2.4 Apache Spark - Leitfaden]]. Pro Block ein **Core**-Teil (Pflicht) und ein **Vertiefung**-Teil (optional, jeweils markiert als *andere Perspektive* oder *weiterführend*).

Ablauf: lesen → beim Lesen roh in [[2.4 Apache Spark - Vertiefung|die Vertiefung]] exzerpieren (pro Quelle unter ihrer Überschrift, mit einem Satz in eigenen Worten) → Kontrollfragen dort **aus dem Kopf** beantworten → Review durch Claudian → Claudian arbeitet den Abschnitt aus. Offene Fragen wandern nach [[Documentation/Traineeship/Skalierbarkeit & ETL/2.4 Apache Spark/Q&A/Fragen|Q&A/Fragen]].

> [!info] Umfang und Stand
> Sechs Blöcke mit je zwei bis vier Core-Quellen, zusammen **~7 h Core**. Das ist so viel wie in 2.3, obwohl der Richtwert nur 1,5 statt 2 Wochen beträgt. Dafür sind zwei Posten Übungen statt Lektüre (Block 3 und 4), und beide zahlen direkt auf die Übungsaufgabe ein. Was aus dem Core geflogen ist, steht am Blockende unter *Nicht im Core*; die Vertiefungsnotiz deckt es trotzdem ab.
> Alle Angaben wurden am 12.10.2026 geprüft. **Versionsstand:** Aktuell ist Spark **4.2.0** (14.07.2026), dazu kommen die Wartungsreleases 4.1.3 und 4.0.4. Ab 4.3 erscheinen Feature-Releases vierteljährlich, Spark 5.0 ist laut Versionierungsrichtlinie für 2027 vorgesehen.

## Die sechs Blöcke

| Block                                                                | Stichworte (Leitfaden)                                                                          | Kontrollfragen | Core     | Status  |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | -------------- | -------- | ------- |
| **[[#Block 1 — Skalierung, Distributed Compute & warum Spark\|1]]**  | 3 Horizontale vs. vertikale Skalierung · 4 Grundprinzipien von Distributed Compute              | 6, 10          | ~70 Min. | ⬜ offen |
| **[[#Block 2 — Spark-Architektur, PySpark & die JVM\|2]]**           | 1 Spark vs. PySpark · 2 JVM und JDBC · 5 Cluster-Architektur, Driver, Executor, Cluster Manager | 1              | ~60 Min. | ⬜ offen |
| **[[#Block 3 — DataFrames, Lazy Evaluation & Catalyst\|3]]**         | 6 DataFrames vs. Datasets vs. RDDs · 8 Lazy Evaluation · 5 Catalyst                             | 2, 3           | ~70 Min. | ⬜ offen |
| **[[#Block 4 — Das Parquet-Format\|4]]**                             | 7 Das Parquet-Format                                                                            | 7, 8, 9        | ~60 Min. | ⬜ offen |
| **[[#Block 5 — Ausführung: Jobs, Stages, Tasks, Shuffle & AQE\|5]]** | 5 Partitionierung, Shuffling, Tasks vs. Stages vs. Jobs, AQE                                    | 4, 5           | ~75 Min. | ⬜ offen |
| **[[#Block 6 — Performance: Konfiguration, UDFs & Debugging\|6]]**   | 9 Configuration Best Practices · 10 UDFs                                                        | 11, 12, 13, 14 | ~85 Min. | ⬜ offen |

**Übungsaufgabe:** Die PySpark-Fassung der Python Challenge (`~/HaegerConsulting/Projects/Traineeship/Skalierbarkeit_ETL/DataEngineeringPythonChallenge`) kann nach Block 3 starten, denn ab dann kennst du DataFrame-API und Lazy Evaluation. Den groben Fahrplan lege ich an, wenn du so weit bist. Die Spark UI (Aufgabe 2) wird ab Block 5 interessant; die Quellen dafür stehen [[#Für die Übungsaufgabe|ganz unten]]. Zwei Dinge solltest du vorher erledigen:
- **Java installieren.** PySpark startet im Hintergrund eine JVM und braucht dafür **Java 17, 21 oder 25** mit gesetztem `JAVA_HOME`. In der Shell, in der ich arbeite, ist kein `java` zu finden. Prüf es mit `java -version` und installier spätestens vor der Übung in Block 3 ein JDK.
- **Daten beim Coach anfragen.** Laut Leitfaden stellt er die Daten für den lokalen Lauf bereit. Die 52-MB-`transactions.csv` aus der Challenge reicht für den Einstieg. Für einen ehrlichen Vergleich der Skalierbarkeit brauchst du mehr.

## Die Bücher

| Kürzel | Buch | Wofür |
|---|---|---|
| **DDIA2** | Kleppmann & Riccomini, *Designing Data-Intensive Applications*, **2. Auflage** (O'Reilly, 2026) — liegt als [[designing_data_intensive_applications_2.pdf\|PDF]] in `Quellen/` | Blöcke 1, 3, 5. Seitenangaben = **PDF-Seiten**; die gedruckte Seite ist jeweils 24 niedriger |
| **FDE** | Reis & Housley, *Fundamentals of Data Engineering* — liegt als [[fundamentals_data_engineering.pdf\|PDF]] in `Quellen/weiterführend/` | Block 4. Seitenangaben = PDF-Seiten |
| **LS** | Damji, Wenig, Das & Lee, *Learning Spark*, **2. Auflage** (O'Reilly, 2020) — [O'Reilly](https://www.oreilly.com/library/view/learning-spark-2nd/9781492050032/); bei Databricks gegen ein Formular kostenlos | Rückgrat der Blöcke 3, 5 und 6. Stand Spark 3.0 |
| **DG** | Chambers & Zaharia, *Spark: The Definitive Guide* (O'Reilly, 2018) — [O'Reilly](https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/) | Blöcke 5 und 6. Stand Spark 2.x |
| **Rioux** | Jonathan Rioux, *Data Analysis with Python and PySpark* (Manning, 2022) — [O'Reilly](https://www.oreilly.com/library/view/data-analysis-with/9781617297205/) | Block 2. Das einzige der drei Spark-Bücher, das konsequent aus Python-Sicht schreibt |

> [!note] Seitenzahlen und Aktualität
> Bei LS, DG und Rioux sind die **Abschnittstitel** über das O'Reilly-Inhaltsverzeichnis verifiziert, gedruckte Seitenzahlen nicht. Die Zeiten dort sind Schätzungen.
> Alle drei Bücher sind älter als Spark 4. Die Konzepte (Driver und Executor, Lazy Evaluation, Catalyst, Shuffle, Joins) haben sich seitdem nicht geändert. Neu sind **Spark Connect** (seit 3.4), der **ANSI-SQL-Modus als Standard** und **Java 17+** (beides seit 4.0) sowie **Arrow-optimierte Python-UDFs als Standard** (seit 4.2). Wo das eine Rolle spielt, steht die aktuelle Doku im Core.

---

## Block 1 — Skalierung, Distributed Compute & warum Spark

**Core ~70 Min. · Kontrollfragen 6, 10**

**Warum das zählt:** In der Challenge hast du Daten von Hand zerlegt, auf Prozesse verteilt und wieder zusammengeführt. Dieser Block gibt dem Problem Namen und Geschichte. Wann rüstet man eine Maschine auf, wann verteilt man auf viele? Was muss jedes verteilte Batch-System lösen? Und warum hat Spark das Modell abgelöst, mit dem man das vor 2010 gemacht hat?

### Core (in dieser Reihenfolge)

- [ ] **1. DDIA2, Kap. 2 „Scalability" und Kap. 1 „Distributed Versus Single-Node Systems"** ([[designing_data_intensive_applications_2.pdf#page=73|PDF-S. 73–76]] und [[designing_data_intensive_applications_2.pdf#page=43|PDF-S. 43–45]] bis vor „Microservices and Serverless", ~15 Min.)
      Scale-up vs. Scale-out, Shared-Memory, Shared-Disk und Shared-Nothing, und warum eine einzelne Maschine trotzdem oft gewinnt.
      *Leseauftrag: Warum wächst der Preis beim vertikalen Skalieren schneller als linear — und welchen Preis zahlt man beim horizontalen?*
- [ ] **2. DDIA2, Kap. 11 „Batch Processing" — von „Batch Processing in Distributed Systems" bis einschließlich „Dataflow Engines"** ([[designing_data_intensive_applications_2.pdf#page=481|PDF-S. 481–493]], bis vor „Shuffling Data", ~25 Min.)
      Verteilte Dateisysteme, Object Stores, Job-Orchestrierung, Fehlerbehandlung, MapReduce und seine Nachfolger. „Shuffling Data" folgt in Block 5.
      *Leseauftrag: Wo legt MapReduce Zwischenergebnisse ab, wo Spark — und was folgt daraus für Geschwindigkeit und Fehlertoleranz?*
- [ ] **3. Zaharia et al., „Apache Spark: A Unified Engine for Big Data Processing"** (Communications of the ACM, Nov. 2016, ~25 Min.) — [PDF bei Databricks](https://databricks.com/de/wp-content/uploads/2018/12/Apache-Spark-A-Unified-Engine-for-Big-Data-Processing.pdf)
      Die Spark-Erfinder über ihre eigene Idee. Lesen: Einleitung, „Programming Model" und „Higher-Level Libraries" (S. 56–60), dazu „Why Is the Spark Model General?" (S. 62–64). „Applications" überfliegen.
      *Leseauftrag: Was ist ein RDD, und wie stellt Spark verlorene Daten wieder her, ohne sie vorher zu kopieren?*
- [ ] **4. Spark-Doku, Startseite — [Where to Go from Here](https://spark.apache.org/docs/latest/#where-to-go-from-here)** (~3 Min.)
      Der Stand von 2026 als Gegenstück zum Artikel von 2016.
      *Leseauftrag: Welche Bibliotheken sind seit 2016 dazugekommen, welche sind als veraltet markiert?*

### Vertiefung (optional)

- [ ] **McSherry, Isard & Murray, „Scalability! But at what COST?"** (HotOS 2015, 6 S., ~20 Min.) · *andere Perspektive* — [PDF](https://www.usenix.org/system/files/conference/hotos15/hotos15-paper-mcsherry.pdf)
      Ein Laptop mit einem einzigen Thread schlägt Cluster mit Hunderten Kernen. Kleppmann zitiert das Paper in DDIA2 Kap. 1. Die Lehre: Skalierbarkeit ist wenig wert, wenn schon die Arbeit auf einem Kern ineffizient ist.
- [ ] **Jordan Tigani, „Big Data is Dead"** (MotherDuck, Feb. 2023, ~15 Min.) · *andere Perspektive* — <https://motherduck.com/blog/big-data-is-dead/>
      Die Gegenthese eines Gründungsingenieurs von BigQuery, ebenfalls in DDIA2 Kap. 1 zitiert. Passt zur 100-GB-Faustregel aus [[2.3 Advanced Python - Vertiefung#6.2 Die Engine wählen|2.3, Abschnitt 6.2]].
- [ ] **LS, Kap. 1 — „The Genesis of Spark" und „Apache Spark Components as a Unified Stack"** (~20 Min.) · *andere Perspektive*
      Dieselbe Geschichte aus Sicht von Databricks (Google, Hadoop bei Yahoo!, AMPLab), dazu die Bibliotheken mit Codebeispielen.
- [ ] **Zaharia et al., „Resilient Distributed Datasets"** (NSDI 2012) — Abschnitte 1 und 2 (~25 Min.) · *weiterführend* — [PDF](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf)
      Das Originalpaper zur revolutionären Idee, mit „Advantages of the RDD Model" (2.3) und „Applications Not Suitable for RDDs" (2.4).

*Nicht im Core:* Hadoop-Ökosystem (HDFS, YARN, Hive) im Detail · Datenlokalität und Speculative Execution · Lineage vs. Checkpointing · Alternativen zu Spark (Flink, Dask, Ray, Daft) · die Entwicklung von Spark seit 2016.

---

## Block 2 — Spark-Architektur, PySpark & die JVM

**Core ~60 Min. · Kontrollfrage 1**

**Warum das zählt:** Spark ist ein JVM-Programm, und dein Python-Code steuert es in aller Regel nur. Wer das Bild „ein Driver plant, viele Executors rechnen, ein Cluster Manager verteilt die Ressourcen" nicht vor Augen hat, versteht weder die Spark UI noch die Ratschläge zur Performance in Block 6. Außerdem klärt der Block, was JVM und JDBC mit Spark zu tun haben. Über JDBC liest Spark zum Beispiel aus deiner Postgres-Datenbank.

### Core (in dieser Reihenfolge)

- [ ] **1. Spark-Doku, [Cluster Mode Overview](https://spark.apache.org/docs/latest/cluster-overview.html)** — „Components", „Cluster Manager Types" und „Glossary" (~10 Min.)
      *Leseauftrag: Wer plant, wer rechnet, wer verteilt die Ressourcen — und warum muss der Driver die ganze Zeit erreichbar bleiben?*
- [ ] **2. Rioux, Kap. 1 „Introduction" — 1.1 „What is PySpark?" und 1.2 „Your very own factory: How PySpark works"** (~15 Min.)
      Das Fabrik-Bild für Driver, Executor und Cluster Manager, dazu Lazy Evaluation als Vorschau auf Block 3.
      *Leseauftrag: Ist PySpark eine Neuimplementierung von Spark in Python? Wenn nicht, was dann?*
- [ ] **3. PySpark-Doku: Python, JVM und Spark Connect** (~12 Min.)
      - [Debugging PySpark](https://spark.apache.org/docs/latest/api/python/development/debugging.html) — nur der Einleitungsabsatz: wie Python-Driver, JVM und Python-Worker zusammenhängen
      - [Spark Connect Overview](https://spark.apache.org/docs/latest/spark-connect-overview.html) — „How Spark Connect works", „How Spark Connect client applications differ from classic Spark applications" und „What is supported"
      *Leseauftrag: Wann startet Spark auf den Executors überhaupt Python-Prozesse? Was schickt ein Spark-Connect-Client an den Server — und was nicht mehr?*
- [ ] **4. JVM und JDBC** (~20 Min.)
      - Java Virtual Machine Specification (SE 21), [Kap. 1](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-1.html) — Abschnitte 1.1 „A Bit of History" und 1.2 „The Java Virtual Machine"
      - Oracle Java Tutorial: [JDBC Introduction](https://docs.oracle.com/javase/tutorial/jdbc/overview/index.html) und [JDBC Architecture](https://docs.oracle.com/javase/tutorial/jdbc/overview/architecture.html)
      - Spark-Doku, [JDBC To Other Databases](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html) — Einleitung, dann in der Optionstabelle `url`, `dbtable`/`query`, `partitionColumn`/`lowerBound`/`upperBound`/`numPartitions` und `fetchsize`
      *Leseauftrag: Was weiß die JVM von der Sprache Java? Und wie liest Spark eine Postgres-Tabelle mit vier Executors parallel statt über eine einzige Verbindung?*

### Vertiefung (optional)

- [ ] **DG, Kap. 15 „How Spark Runs on a Cluster" — „The Architecture of a Spark Application", „Execution Modes" und „The Life Cycle of a Spark Application (Outside Spark)"** (~15 Min.) · *andere Perspektive*
      Dasselbe Bild mit Schaubildern, dazu Client- vs. Cluster-Modus und was bei `spark-submit` passiert. Den Rest des Kapitels liest du in Block 5.
- [ ] **LS, Kap. 1 — „Apache Spark's Distributed Execution"** (~8 Min.) · *andere Perspektive*
      Driver, SparkSession, Cluster Manager, Executors und die Deployment-Modi auf wenigen Seiten.
- [ ] **Databricks-Blog, „Project Tungsten: Bringing Apache Spark Closer to Bare Metal"** (2015, ~12 Min.) · *weiterführend* — <https://www.databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html>
      Warum Spark seinen Speicher am Garbage Collector der JVM vorbei selbst verwaltet. Dasselbe Problem wie bei Python-Objekten in [[2.3 Advanced Python - Vertiefung#2.4 Ursache 3: verstreute Objekte im Speicher|2.3, Abschnitt 2.4]], nur auf der JVM.
- [ ] **PySpark-Doku, [Python Package Management](https://spark.apache.org/docs/latest/api/python/tutorial/python_packaging.html)** (~10 Min.) · *weiterführend*
      Wie deine Python-Abhängigkeiten auf die Executors kommen. Lokal egal, im Cluster eine der ersten Stolperfallen.

*Nicht im Core:* JIT-Compiler und Garbage Collector der JVM · `spark-submit` und Deploy-Modi im Detail · Py4J · Scala als Sprache · Einrichtung von YARN und Kubernetes.

---

## Block 3 — DataFrames, Lazy Evaluation & Catalyst

**Core ~70 Min. · Kontrollfragen 2, 3**

**Warum das zählt:** Ab hier wird Spark konkret. Die zentrale Idee kennst du aus 2.3: Polars sammelt im Lazy-Modus erst den ganzen Plan und optimiert ihn, bevor es rechnet ([[2.3 Advanced Python - Vertiefung#6.6 DataFrames, die wie SQL denken: Polars|2.3, Abschnitt 6.6]]). Spark macht dasselbe im Cluster, mit dem Optimizer Catalyst. Das erklärt auch, warum für dich nur DataFrames zählen: Nur dort sieht der Optimizer, was du vorhast.

### Core (in dieser Reihenfolge)

- [ ] **1. LS, Kap. 2 — „Transformations, Actions, and Lazy Evaluation" und „Narrow and Wide Transformations"** (~10 Min.)
      *Leseauftrag: Was passiert bei `df.filter(...)`, und was erst bei `df.count()`?*
- [ ] **2. LS, Kap. 3 „Apache Spark's Structured APIs"** — „Spark: What's Underneath an RDD?", „Structuring Spark", „DataFrames Versus Datasets", „When to Use RDDs" und „Spark SQL and the Underlying Engine" mit „The Catalyst Optimizer" (~30 Min.)
      „The DataFrame API" und „The Dataset API" überspringen. Die Syntax lernst du in der Übung schneller.
      *Leseauftrag: Was weiß Spark bei einem DataFrame, das es bei einem RDD nicht wissen kann?*
- [ ] **3. Zwei kurze Stellen zum Warum** (~10 Min.)
      - CACM-Artikel aus Block 1, „Ongoing Work" → „DataFrames and more declarative APIs" (S. 64)
      - DDIA2, Kap. 11 — „Query Languages" und „DataFrames" ([[designing_data_intensive_applications_2.pdf#page=497|PDF-S. 497–499]])
      *Leseauftrag: Warum konnte Spark auf RDDs einen Filter nicht vor ein `map` ziehen? Und was unterscheidet einen verteilten DataFrame von einem pandas-DataFrame?*
- [ ] **4. Übung: [PySpark Quickstart: DataFrame](https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_df.html)** (~20 Min.)
      Mittippen statt lesen; Java muss dafür installiert sein. Danach auf eine eigene kleine Abfrage mit Filter und `groupBy` die Methode `explain(mode="extended")` aufrufen.
      *Leseauftrag: Welche vier Pläne zeigt `explain`, und woran erkennst du, dass Catalyst deinen Filter verschoben hat?*

### Vertiefung (optional)

- [ ] **Armbrust et al., „Spark SQL: Relational Data Processing in Spark"** (SIGMOD 2015) — Abschnitte 3 und 4 (~40 Min.) · *weiterführend* — [PDF](https://people.csail.mit.edu/matei/papers/2015/sigmod_spark_sql.pdf)
      Das Paper zu DataFrame-API und Catalyst. Abschnitt 4 zeigt an Bäumen und Regeln, wie ein Optimizer intern arbeitet.
- [ ] **Databricks-Blog, „Deep Dive into Spark SQL's Catalyst Optimizer"** (2015, ~15 Min.) · *andere Perspektive* — <https://www.databricks.com/blog/2015/04/13/deep-dive-into-spark-sqls-catalyst-optimizer.html>
      Kurzfassung von Abschnitt 4 des Papers, für den Fall, dass dir das Paper zu dicht ist.
- [ ] **DG, Kap. 4 „Structured API Overview" — „DataFrames Versus Datasets" und „Overview of Structured API Execution"** (~10 Min.) · *andere Perspektive*
      Logische und physische Planung mit Schaubild.
- [ ] **Yin Huai, „A Deep Dive into Spark SQL's Catalyst Optimizer"** (Databricks, Video 28 Min.) · *andere Perspektive* — <https://www.youtube.com/watch?v=RmUn5vHlevc>

*Nicht im Core:* Dataset-API und Encoders (nur Scala und Java) · Tungsten und Whole-Stage Code Generation · Cost-Based Optimizer · pandas API on Spark · Caching (kommt in Block 6).

---

## Block 4 — Das Parquet-Format

**Core ~60 Min. · Kontrollfragen 7, 8, 9**

**Warum das zählt:** Spark-Pipelines lesen und schreiben fast immer Parquet, und du hast das Format in der Challenge schon benutzt, ohne hineinzuschauen. Die Grundlagen kennst du aus 2.2: spaltenorientierte Speicherung, Dictionary Encoding, RLE und Zone Maps ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.6 Kompression — warum Spalten so viel besser packen|2.2, Abschnitt 1.6]] und [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.7 Sortierreihenfolge, mehrere Sortierungen und Zone Maps|1.7]]). Dieser Block zeigt, wie Parquet diese Ideen in eine einzige Datei packt, und warum das mehr ist als eine komprimierte CSV.

### Core (in dieser Reihenfolge)

- [ ] **1. FDE, Anhang A — von „Serialization Formats" bis einschließlich „Hybrid Serialization"** ([[fundamentals_data_engineering.pdf#page=529|PDF-S. 529–535]], ~15 Min.)
      CSV, JSON, Avro, Parquet, ORC, Arrow, Hudi und Iceberg im Vergleich.
      *Leseauftrag: Was löst Parquet, das CSV nicht löst — und was kann ein Column Store grundsätzlich schlecht?*
- [ ] **2. Parquet-Doku und Spark-Doku** (~20 Min.)
      - Parquet: [Concepts](https://parquet.apache.org/docs/concepts/), [File Format](https://parquet.apache.org/docs/file-format/), aus [Encodings](https://parquet.apache.org/docs/file-format/data-pages/encodings/) nur „Dictionary Encoding" und „Run Length Encoding / Bit-Packing Hybrid"
      - Spark: [Parquet Files](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html) — „Partition Discovery" und „Schema Merging"
      *Leseauftrag: In welche Einheiten zerlegt Parquet eine Datei? Welche ist die Einheit der Parallelisierung, welche die der Kompression? Und warum stehen die Metadaten am Ende der Datei?*
- [ ] **3. Nikola Ilic, „Parquet file format – everything you need to know!"** (Data Mozart, 2024, ~10 Min.) — <https://data-mozart.com/parquet-file-format-everything-you-need-to-know/>
      Dasselbe mit vielen Bildern. Der Abschnitt zu Delta Lake ist ein Ausblick.
      *Leseauftrag: Wie viel liest eine Abfrage, die zwei von zwanzig Spalten braucht und filtert — und welche Metadaten machen das möglich?*
- [ ] **4. Übung: Schau in deine eigene Datei** (~10 Min.) — pyarrow-Doku, [Inspecting the Parquet File Metadata](https://arrow.apache.org/docs/python/parquet.html#inspecting-the-parquet-file-metadata)
      Öffne `data/sales_postprocessed_transactions.parquet` aus der Challenge mit `pyarrow.parquet.ParquetFile` und lies Row Groups, Encodings, Kompression und Statistiken ab.
      *Leseauftrag: Was davon hast du beim Schreiben bewusst entschieden, und was hat pyarrow für dich entschieden?*

### Vertiefung (optional)

- [ ] **CMU 15-721 (Spring 2024), Andy Pavlo — Lectures #02 und #03 „Data Formats & Encoding"** (Folien, ~90 Min.) · *weiterführend* — [#02](https://15721.courses.cs.cmu.edu/spring2024/slides/02-data1.pdf) · [#03](https://15721.courses.cs.cmu.edu/spring2024/slides/03-data2.pdf)
      Aus 2.2 noch offen, hier passen sie besser: PAX, Parquet vs. ORC, Encoding-Verfahren und das Dremel-Verfahren für verschachtelte Daten.
- [ ] **Zeng et al., „An Empirical Evaluation of Columnar Storage Formats"** (PVLDB 17, 2023) — Kap. 1–3 und 6 (~30 Min.) · *weiterführend* — [PDF](https://www.vldb.org/pvldb/vol17/p148-zeng.pdf)
      Ebenfalls aus 2.2 offen. Parquet und ORC im Detail zerlegt, mit Lehren für künftige Formate. Pavlo und Wes McKinney sind Mitautoren.
- [ ] **Julien Le Dem, „Efficient Data formats for Analytics with Parquet and Arrow"** (Berlin Buzzwords 2016, Video 46 Min.) · *andere Perspektive* — <https://www.youtube.com/watch?v=43O5BCabBcU>
      Einer der Erfinder von Parquet über Parquet (auf der Platte) und Arrow (im Speicher).
- [ ] **LS, Kap. 9 „Building Reliable Data Lakes with Apache Spark" — „Lakehouses: The Next Step in the Evolution of Storage Solutions"** (~15 Min.) · *weiterführend*
      Was Parquet allein nicht kann (Transaktionen, Updates, Schema-Evolution) und wie Hudi, Iceberg und Delta Lake das lösen.

*Nicht im Core:* verschachtelte Daten (Repetition und Definition Levels) · Bloom Filter und Page Index · Kompressionscodecs (Snappy, ZSTD, gzip) im Vergleich · Avro und ORC im Detail · das Small-Files-Problem · Table Formats (Delta Lake, Iceberg).

---

## Block 5 — Ausführung: Jobs, Stages, Tasks, Shuffle & AQE

**Core ~75 Min. · Kontrollfragen 4, 5**

**Warum das zählt:** Das ist der Kern der „Spark Internals" aus Stichwort 5 und zugleich das Vokabular der Spark UI. Wer weiß, wo ein Shuffle entsteht, kann die meisten langsamen Jobs erklären. In deiner Übung betrifft das zwei Stellen: den Join mit `products` und die Aggregation pro Produkt. Genau dort shuffelt Spark, oder es vermeidet den Shuffle dank Broadcast Join.

### Core (in dieser Reihenfolge)

- [ ] **1. DG, Kap. 15 — „The Life Cycle of a Spark Application (Inside Spark)" und „Execution Details"** (~20 Min.)
      Ein Beispiel-Job wird in Jobs, Stages und Tasks zerlegt, dazu Pipelining und Shuffle Persistence.
      *Leseauftrag: Warum beginnt an jedem Shuffle eine neue Stage, und warum landen Shuffle-Daten auf der Platte?*
- [ ] **2. Shuffle in DDIA2 und in der Spark-Doku** (~15 Min.)
      - DDIA2, Kap. 11 — „Shuffling Data" und „Joins and Grouping" ([[designing_data_intensive_applications_2.pdf#page=493|PDF-S. 493–497]])
      - Spark-Doku, RDD Programming Guide — [Shuffle operations](https://spark.apache.org/docs/latest/rdd-programming-guide.html#shuffle-operations), Unterabschnitte „Background" und „Performance Impact"
      *Leseauftrag: Was passiert beim Shuffle physisch — und was ist an diesem Begriff irreführend?*
- [ ] **3. LS, Kap. 7 — „A Family of Spark Joins"** (Broadcast Hash Join und Shuffle Sort Merge Join, ~20 Min.)
      *Leseauftrag: Wann spart ein Broadcast Hash Join den Shuffle, und wann wird er gefährlich?*
- [ ] **4. Adaptive Query Execution und Partitionen** (~20 Min.)
      - Databricks-Blog, [Adaptive Query Execution: Speeding Up Spark SQL at Runtime](https://www.databricks.com/blog/2020/05/29/adaptive-query-execution-speeding-up-spark-sql-at-runtime.html) (2020)
      - Spark-Doku, [Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html) — „Tuning Partitions" lesen, „Adaptive Query Execution" überfliegen
      *Leseauftrag: Was weiß AQE, das Catalyst vor der Ausführung nicht wissen kann? Welche drei Probleme löst es damit?*

### Vertiefung (optional)

- [ ] **Aaron Davidson, „A Deeper Understanding of Spark Internals"** (Spark Summit 2014, Video 44 Min.) · *andere Perspektive* — <https://www.youtube.com/watch?v=dmL0N3qfSc8>
      Execution Model, Shuffle und Caching live erklärt. Der Vortrag arbeitet mit RDDs, die Mechanik ist dieselbe.
- [ ] **Cloudera, „How-to: Tune Your Apache Spark Jobs (Part 1)"** (2015, ~15 Min.) · *andere Perspektive* — <https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-1/>
      „How Spark Executes Your Program" und „Picking the Right Operators": wann Shuffles entstehen und wie man sie vermeidet. Aus der RDD-Zeit; Cloudera markiert den Artikel inzwischen als historisch.
- [ ] **Spark-Doku, Performance Tuning — „Optimizing the Join Strategy" und „Storage Partition Join"** (~10 Min.) · *weiterführend*
      Join-Hints, und wie man einen Shuffle schon beim Speichern der Daten verhindert.
- [ ] **DDIA2, Kap. 7 „Sharding" — „Sharding by Hash of Key" und „Skewed Workloads and Relieving Hot Spots"** ([[designing_data_intensive_applications_2.pdf#page=282|PDF-S. 282–288]], ~12 Min.) · *weiterführend*
      Hash-Partitionierung ist genau das, womit Spark beim Shuffle die Daten verteilt. Skew kommt in Block 6 als häufigster Grund für langsame Jobs zurück. Aus 2.2 noch offen.

*Nicht im Core:* Shuffle-Implementierung (Sort-based Shuffle, External Shuffle Service, Push-based Shuffle) · Shuffled Hash Join · Bucketing · Dynamic Partition Pruning · Skew von Hand beheben (Salting, kommt in Block 6).

---

## Block 6 — Performance: Konfiguration, UDFs & Debugging

**Core ~85 Min. · Kontrollfragen 11, 12, 13, 14**

**Warum das zählt:** Spark nimmt dir das Verteilen ab, aber nicht das Nachdenken. Vier Kontrollfragen landen in diesem Block, und es sind die praxisnächsten des Kapitels: Wie groß macht man Executors und Partitionen? Warum ist ein Job langsam? Und warum ist eine Python-UDF oft die teuerste Zeile im ganzen Job? Danach bist du bereit für den Performance-Vergleich in der Übungsaufgabe.

### Core (in dieser Reihenfolge)

- [ ] **1. LS, Kap. 7 — „Optimizing and Tuning Spark for Efficiency" und „Caching and Persistence of Data"** (~30 Min.)
      Konfiguration setzen und auslesen, Skalieren für große Workloads, Executor-Speicher, Partitionen, `cache()` vs. `persist()`.
      *Leseauftrag: Auf welchen Wegen setzt man eine Konfiguration? Und wann schadet Caching mehr, als es nützt?*
- [ ] **2. Executors dimensionieren** (~15 Min.)
      - Cloudera, [How-to: Tune Your Apache Spark Jobs (Part 2)](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/) — nur „Tuning Resource Allocation" und „Tuning Parallelism"
      - Spark-Doku, [Configuration](https://spark.apache.org/docs/latest/configuration.html) — in der Tabelle „Application Properties" die Einträge `spark.executor.memory`, `spark.executor.memoryOverhead` und `spark.executor.pyspark.memory`
      Cloudera markiert den Artikel selbst als historisch (Spark 1.3 auf YARN). Gelesen wird er wegen der Beispielrechnung, die bis heute die meistzitierte Faustregel ist.
      *Leseauftrag: Warum weder ein Executor pro Maschine mit allen Kernen noch viele winzige Executors? Und wo liegt der Speicher der Python-Worker?*
- [ ] **3. UDFs** (~20 Min.)
      - DG, Kap. 6 — „User-Defined Functions"
      - PySpark User Guide, [Kap. 5 „Unleashing UDFs & UDTFs"](https://spark.apache.org/docs/latest/api/python/user_guide/udfandudtf.html) — nur „Python UDFs"
      - PySpark-Doku, [Upgrading from PySpark 4.1 to 4.2](https://spark.apache.org/docs/latest/api/python/migration_guide/pyspark_upgrade.html#upgrading-from-pyspark-4-1-to-4-2) — nur die Einträge zu Arrow
      *Leseauftrag: Welchen Weg nimmt ein Datensatz bei einer Python-UDF zwischen JVM und Python? Und welche Aussage aus DG ist seit Spark 4.2 nur noch halb richtig?*
- [ ] **4. DG, Kap. 18 „Monitoring and Debugging" — „Debugging and Spark First Aid"** (~20 Min.)
      Nur „Slow Tasks or Stragglers", „Slow Aggregations", „Slow Joins", „Slow Reads and Writes", „Driver OutOfMemoryError or Driver Unresponsive" und „Executor OutOfMemoryError or Executor Unresponsive".
      *Leseauftrag: Welche Ursachen tauchen in fast jedem dieser Abschnitte wieder auf?*

### Vertiefung (optional)

- [ ] **Databricks, „Comprehensive Guide to Optimize Databricks, Spark and Delta Lake Workloads"** — „Data Shuffling", „Data Spilling", „Data Skewness" und „Data Explosion" (~25 Min.) · *andere Perspektive* — <https://www.databricks.com/discover/pages/optimize-data-workloads-guide>
      Die Ursachen langsamer Jobs nach Symptom sortiert, jeweils mit Gegenmitteln. Abschnitte zu Delta Lake und Databricks überspringen.
- [ ] **Daniel Tomes, „Apache Spark Core—Deep Dive—Proper Optimization"** (Spark + AI Summit 2019, Video 90 Min.) · *weiterführend* — <https://www.youtube.com/watch?v=daXEp4HmS-E>
      Der Klassiker zum Dimensionieren von Partitionen und Shuffles, mit der Spark UI als Werkzeug.
- [ ] **PySpark-Doku, [Apache Arrow in PySpark](https://spark.apache.org/docs/latest/api/python/tutorial/sql/arrow_pandas.html)** — „Pandas UDFs (a.k.a. Vectorized UDFs)", „Pandas Function APIs" und „Arrow Python UDFs" (~20 Min.) · *weiterführend*
      Die vektorisierten Varianten, falls du in der Übung um eine UDF nicht herumkommst.
- [ ] **Rioux, Kap. 11 „Faster PySpark: Understanding Spark's query planning"** (~30 Min.) · *andere Perspektive*
      Spark UI und Abfragepläne aus Python-Sicht. Eine gute Vorbereitung auf Aufgabe 2 der Übung.

*Nicht im Core:* Speicheraufteilung im Executor (Unified Memory Manager) · GC-Tuning · Dynamic Allocation im Detail · Skew von Hand beheben (Salting) · Storage Levels beim Caching · Python UDTFs · die Spark UI im Detail (→ Übungsaufgabe).

---

## Für die Übungsaufgabe

Keine Lektüre für die Blöcke, sondern Material für Aufgabe 2 („Mache dich mit der Spark UI vertraut"). Am besten mit einem eigenen laufenden Job daneben durcharbeiten.

- [ ] **Spark-Doku, [Web UI](https://spark.apache.org/docs/latest/web-ui.html)** — alle Tabs, Schwerpunkt „Jobs Tab", „Stages Tab" und „SQL Tab" (~25 Min.)
      Die offizielle Tour durch die UI, mit Screenshots.
- [ ] **Databricks-Doku, [Diagnose cost and performance issues using the Spark UI](https://docs.databricks.com/aws/en/optimizations/spark-ui-guide/)** (~20 Min.)
      Eine Anleitung zur Fehlersuche in fünf Schritten: Jobs-Timeline → längste Stage → Skew oder Spill → I/O-bound? → sonstige Ursachen. Die Screenshots stammen aus Databricks, die UI ist dieselbe wie bei Open-Source-Spark.
