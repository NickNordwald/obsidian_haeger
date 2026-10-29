---
tags:
  - documentation
---

# Leseliste — 2.2 OLAP Datenmodellierung & Datawarehousing

Kuratierte Leseaufgaben zu [[2.2 OLAP Datenmodellierung & Datawarehousing - Leitfaden]]. Pro Block ein **Core**-Teil (Pflicht) und ein **Vertiefung**-Teil (optional, jeweils mit Angabe *andere Perspektive* oder *weiterführend*).

Ablauf: lesen → beim Lesen roh in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung|die Vertiefung]] exzerpieren → Kontrollfragen dort **aus dem Kopf** beantworten → Review durch Claudian → Claudian arbeitet den Abschnitt aus.

Offene Fragen beim Lesen wandern nach [[Q&A/Fragen|Q&A/Fragen]].

> [!info] Diese Liste ist auf Vollständigkeit kuratiert, nicht auf Kürze
> Anders als in 2.1 ist Zeit hier nicht der Engpass. Die Blöcke setzen deshalb durchgehend auf **Primärquellen** statt auf Sekundärliteratur: Kimball selbst statt Kimball-Zusammenfassungen, Linstedt selbst statt Data-Vault-Blogposts. Gesamtumfang Core über alle sechs Blöcke: **rund 18 Stunden** — bei drei Wochen Richtwert für das Kapitel gut machbar.
> Jede Kapitel- und Seitenangabe unten ist verifiziert. Wo etwas *nicht* belegt werden konnte, steht es explizit dabei.

## Die sechs Blöcke

| Block | Stichworte | Core-Umfang | Status |
|---|---|---|---|
| **[[#Block 1 — OLAP vs. OLTP & Warehouse vs. Postgres\|1]]** | OLAP vs. OLTP · Warehouse vs. Postgres | ~2 h 15 | ✅ fertig |
| **[[#Block 2 — Kimball: Dimensionale Modellierung & Denormalisierung\|2]]** | Dimensionale Modellierung nach Kimball · Normalisierung vs. Denormalisierung | ~4 h 00 | ✅ fertig |
| **[[#Block 3 — Slowly Changing Dimensions\|3]]** | Slowly Changing Dimensions | ~1 h 35 | ✅ fertig |
| **[[#Block 4 — Data Vault, OBT & Kimball vs. Inmon\|4]]** | Data Vault · OBT-Modellierung · Kimball vs. Inmon | ~2 h 30 | ✅ fertig |
| **[[#Block 5 — Schichtenarchitektur, Medallion & Master Data\|5]]** | Medallion Architecture · Master Data / Stammdaten | ~3 h 20 | ✅ fertig |
| **[[#Block 6 — Partitionierung & idempotente Ladeverfahren\|6]]** | Partitionierung · Idempotenz | ~4 h 20 | ✅ fertig |

Die Übungsaufgabe folgt **nach allen sechs Blöcken** am Stück. Nach Abgabe zum Abgleich: [[Übungsaufgabe/Musterlösung (Dimensionale Modellierung)|Musterlösung]].

---

## Bücher, die du brauchst

Sechs Titel tragen dieses Kapitel. Alle über O'Reilly Learning bzw. die Firmenbibliothek verfügbar.

| Kürzel | Buch | Wofür |
|---|---|---|
| **DDIA2** | Kleppmann & Riccomini, *Designing Data-Intensive Applications*, **2. Auflage** (O'Reilly, 2026, 670 S.) | Block 1, 2, 6 |
| **Kimball** | Kimball & Ross, *The Data Warehouse Toolkit*, **3. Auflage** (Wiley, 2013, ISBN 978-1-118-53080-1) | Block 2, 3, 5 — **das zentrale Buch des Kapitels** |
| **FDE** | Reis & Housley, *Fundamentals of Data Engineering* (O'Reilly, 2022) — liegt als PDF in `Quellen/weiterführend/` | Block 1, 2, 4 |
| **DEDP** | Konieczny, *Data Engineering Design Patterns* (O'Reilly, **April 2025**, ISBN 978-1-098-16581-9) | Block 6 |
| **Linstedt** | Linstedt & Olschimke, *Building a Scalable Data Warehouse with Data Vault 2.0* (Morgan Kaufmann, 2015) | Block 4 |
| **DMBOK2** | DAMA International, *DAMA-DMBOK*, 2. Auflage (Technics Publications, 2017) | Block 5 |

> [!warning] Kimball — welches der Bücher, und welche Auflage
> Gemeint ist immer: **Ralph Kimball & Margy Ross, *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling*, 3. Auflage, Wiley 2013**, ISBN 978-1-118-53080-1.
> O'Reilly: <https://www.oreilly.com/library/view/the-data-warehouse/9781118530801/>
>
> **Auflage:** Es gibt die 1. (1996), 2. (2002) und 3. (2013); eine 4. existiert nicht. Alle Kapitel- und Seitenangaben unten gelten für die **3. Auflage** — zwischen den Auflagen wurde umsortiert.
>
> **Nicht verwechseln mit den Geschwisterbüchern:**
>
> | Titel | Inhalt | Brauchen wir? |
> |---|---|---|
> | *The Data Warehouse **Toolkit*** (3. Aufl. 2013) | Dimensionale Modellierung | **Ja — das ist gemeint** |
> | *The Data Warehouse **Lifecycle** Toolkit* (2. Aufl. 2008) | Projektmethodik, Rollout | Nein |
> | *The Data Warehouse **ETL** Toolkit* (2004) | ETL-Subsysteme, eigenes Buch | Nein — der ETL-Teil steckt in Kap. 19/20 des Toolkits |
> | *The Kimball Group **Reader*** (2015) | Anthologie aus 300+ Design Tips | Nur eine optionale Quelle in Block 3 |
>
> **Kontrolle beim Aufschlagen:** Kapitel 3 heißt „Retail Sales" und beginnt auf S. 69, Kapitel 5 heißt „Procurement" und enthält ab S. 147 die SCD-Typen 0–7. Passt das nicht, ist es die falsche Auflage.

> [!warning] DDIA — welche Auflage, und warum das diesmal wirklich zählt
> In `Quellen/` liegt noch die **1. Auflage (2017)**. Die 2. Auflage hat vorne ein neues Architektur-Kapitel eingezogen, **wodurch sich alles um +1 verschiebt** — und den Analytics-Stoff, der in der 1. Auflage komplett in Kapitel 3 stand, auf **drei** Kapitel verteilt. Für dieses Kapitel ist das der Unterschied zwischen „findet man" und „findet man nicht":
>
> | Thema | 1. Auflage (2017) | **2. Auflage (2026)** |
> |---|---|---|
> | OLAP vs. OLTP, Data Warehousing | Kap. 3 › „Transaction Processing or Analytics?" | **Kap. 1 › „Operational Versus Analytical Systems"** |
> | Star- und Snowflake-Schema | Kap. 3 › „Stars and Snowflakes" | **Kap. 3 › „Stars and Snowflakes: Schemas for Analytics"** |
> | Normalisierung/Denormalisierung | verstreut in Kap. 2 | **Kap. 3 › „Normalization, Denormalization, and Joins"** (neuer eigener Abschnitt) |
> | Spaltenorientierte Speicherung | Kap. 3 › „Column-Oriented Storage" | **Kap. 4 › „Data Storage for Analytics"** |
> | Partitionierung | Kap. 6 „Partitioning" | **Kap. 7 „Sharding"** (umbenannt!) |
>
> Neu in der 2. Auflage und für uns direkt relevant: **„Cloud Data Warehouses"**, **„Query Execution: Compilation and Vectorization"**, Lakehouse, sowie stark ausgebaute Behandlung der Spaltenformate (Parquet, ORC, Arrow).
>
> **Kapitel 1 ist legal gratis** über den ScyllaDB-Auszug (~150 S., Kap. 1, 2 und 9): <https://lp.scylladb.com/designing-data-intensive-apps-book-offer>

> [!note] Zu den Seitenangaben
> - **Kimball, Linstedt, DMBOK2, DEDP**: gedruckte Seitenzahlen, aus dem jeweiligen Buch-Inhaltsverzeichnis verifiziert. Im O'Reilly-Web-Reader (EPUB) werden möglicherweise keine Seitenzahlen angezeigt — deshalb steht überall zusätzlich die **Abschnittsüberschrift**.
> - **FDE**: Das lokale PDF in `Quellen/` ist ein EPUB-Export **ohne gedruckte Seitenzahlen**. Angaben dafür sind **PDF-Seiten dieser Datei**.
> - **DDIA2**: Abschnittstitel sind verifiziert, **Seitenzahlen einzelner Abschnitte nicht** (kein öffentlicher Index). Lesezeiten dort sind Schätzungen auf Basis von 1,96 Min./Seite (O'Reilly-Angabe: 672 S. = 21 h 56).

---

# Block 1 — OLAP vs. OLTP & Warehouse vs. Postgres

> [!success] Abgeschlossen
> Core vollständig gelesen, Kontrollfragen beantwortet und reviewt (7 sicher · 9 teilweise · 4 leer), Vertiefung ausgearbeitet, 37 Anki-Karten erstellt, Q&A beantwortet.

**Warum das zählt:** Das ganze Kapitel beantwortet die Frage „warum reicht Postgres nicht?". Dieser Block liefert das Fundament auf zwei Ebenen, die man auseinanderhalten muss: die **Workload-Ebene** (was für Abfragen kommen rein?) und die **Speicher-Ebene** (wie liegen die Bytes auf der Platte?). Die Kontrollfrage „Warehouses sind letztlich read-optimierte relationale Datenbanken" ist genau die Falle, in die man tappt, wenn man nur eine der beiden Ebenen kennt.

Du hast in [[2.1 Advanced SQL & ETL - Vertiefung#1.5 Zeilenbasiert vs. spaltenbasiert|2.1, Abschnitt 1.5]] die Grundidee spaltenbasierter Speicherung schon notiert. Hier geht es um die **Mechanik** — Kompression, Sortierreihenfolge, Vektorisierung — und um das, was daraus für die Modellierung folgt. Das ist der Grund, warum Denormalisierung im Warehouse richtig ist und in Postgres falsch bleibt; ohne diesen Block wirkt Block 2 wie Willkür.

**Zeitbudget Core: ~2 h 15 Lesen + ~30 Min. Fragen.**

## Core (Pflicht)

- [x] **1. IBM, „OLAP vs. OLTP: What's the difference?"** — ~7 Min.
  Der schnelle Begriffsabgleich zum Aufwärmen, beide Seiten in einem Dokument. Trägt: *Processing type* als Leitunterscheidung, danach die üblichen Achsen (Datenquellen, Normalisierungsgrad, Response Time, Anfragetyp). Vendor-Bias gering.
  → *Leitfragen:* Welche Achsen unterscheiden die beiden — und welche davon ist die **ursächliche**, aus der die anderen folgen? Was steht hier über den Normalisierungsgrad, und passt das zu dem, was du aus [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.2 Relationale Datenbanken & ORM/1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]] über Normalisierung weißt?
  <https://www.ibm.com/think/topics/olap-vs-oltp>

- [x] **2. DDIA2, Kap. 1 „Trade-Offs in Data Systems Architecture" — Abschnitt „Operational Versus Analytical Systems"** — ~35 Min.
  **Die Pflichtquelle des Blocks.** Unterabschnitte: *Characterizing Transaction Processing and Analytics* · *Data Warehousing* · *Systems of Record and Derived Data*. Beantwortet die Frage in der Reihenfolge, in der sie sich stellt: erst welche Workload, dann welche Systemantwort, dann warum man überhaupt trennt. Enthält außerdem einen kurzen, wohltuend entzaubernden **HTAP-Exkurs**.
  → *Leitfragen:* Was genau ist ein „System of Record" und was ist „Derived Data" — und warum ist diese Unterscheidung wichtiger als die Unterscheidung OLTP/OLAP? Warum hat sich das Warehouse als **separates System** durchgesetzt statt als Modus der operativen Datenbank?

- [x] **3. DDIA2, Kap. 4 „Storage and Retrieval" — Abschnitt „Data Storage for Analytics" (komplett)** — ~45 Min.
  **Das Herzstück.** Unterabschnitte: *Cloud Data Warehouses* · *Column-Oriented Storage* · *Query Execution: Compilation and Vectorization* · *Materialized Views and Data Cubes*. Kapitel 1 sagt *dass* man trennt, dieses Kapitel sagt, *warum es technisch nicht anders geht*. Der Abschnitt steht direkt neben dem OLTP-Pendant („Storage and Indexing for OLTP") im selben Kapitel — der Kontrast ist didaktisch der eigentliche Gewinn.
  Wenn du irgendwo in diesem Block langsam liest, dann hier.
  → *Leitfragen:* Nenne **drei voneinander unabhängige** Gründe, warum spaltenweise Speicherung analytische Queries beschleunigt. Was ist der Unterschied zwischen *Compilation* und *Vectorization* bei der Query-Ausführung? Was ändert sich architektonisch durch die Trennung von Storage und Compute in Cloud-Warehouses?

- [x] **4. FDE, Kap. 5 — „Application Databases (OLTP systems)" bis Ende „Online Analytical Processing System"** — ~6 Min. · PDF-S. 228–231
  Dieselbe Unterscheidung aus der Perspektive des Data Engineers, der auf ein **fremdes** Quellsystem schaut. Enthält den Zwischenabschnitt „OLTP and analytics".
  → *Leitfragen:* Was genau geht kaputt, wenn Analysten direkt auf der Produktivdatenbank abfragen? Warum ist „dann nimm halt eine Read Replica" nur eine Teillösung?

- [x] **5. FDE, Kap. 3 — Abschnitt „Data Warehouse" (inkl. „The cloud data warehouse", „Data marts")** — ~9 Min. · PDF-S. 146–151
  Ergänzt DDIA2 um die Betriebssicht und führt den Begriff **Data Mart** ein, der in Block 5 wiederkommt.
  → *Leitfrage:* Wozu dient ein Data Mart, wenn das Warehouse doch schon da ist?

- [x] **6. Özcan, Tian & Tözün, „Hybrid Transactional/Analytical Processing: A Survey"** — ~35 Min. · 5 S. (SIGMOD 2017)
  Die Substanz zur Leitfaden-Kontrollfrage „OLAP und OLTP sind Idealtypen". Formal ein Tutorial-Paper, inhaltlich dicht und deutlich zugänglicher als die längeren HTAP-Surveys. Leitet HTAP historisch aus Stonebrakers „One Size Fits All" her und ordnet konkrete Systeme nach Designentscheidung. Von Kleppmann in DDIA2 Kap. 1 zitiert, also direkt buchanschlussfähig.
  *Stand 2017 — SingleStore heißt hier noch MemSQL, Snowflake Unistore und AlloyDB existieren noch nicht.*
  → *Leitfragen:* Welchen **Preis** zahlt ein System, das beides gleichzeitig können soll? Wo genau liegt die technische Schwierigkeit — warum ist HTAP nicht einfach „beide Engines in eine Box"?
  <https://humming80.github.io/papers/sigmod-htaptut.pdf>

## Vertiefung (optional)

- [ ] **Abadi et al., „The Design and Implementation of Modern Column-Oriented Database Systems", Kap. 1–3** — ~1 h 45 · 29 S. · *weiterführend*
  Der Referenztext zu Column Stores, frei verfügbar, von Kleppmann selbst an dieser Stelle als weiterführende Lektüre zitiert. Kap. 1–3 sind bewusst als Einführung geschrieben: Historie, Hardware-Trends, Architekturvergleich C-Store/MonetDB/VectorWise. Sehr lohnend zusätzlich **4.1–4.4** (Vectorized Processing, Compression, Operating Directly on Compressed Data, Late Materialization).
  Überspringen: 4.5 (Joins, sehr formal), 4.8 (Database Cracking, praktisch bedeutungslos).
  **Vorbehalt: Stand 2012** — Parquet, Arrow, Snowflake, DuckDB kommen nicht vor. Deshalb nur zusammen mit der nächsten Quelle lesen.
  <https://www.cs.umd.edu/~abadi/papers/abadi-column-stores.pdf>

- [ ] **CMU 15-721 (Spring 2024), Andy Pavlo — Lectures #02/#03 „Data Formats & Encoding"** — ~90 Min. für beide Foliensätze · *weiterführend*
  Füllt genau die Lücke, die Abadi lässt: NSM/DSM/PAX, Parquet, ORC, Arrow, Encoding Schemes, **Zone Maps**, Dremel-Shredding für verschachtelte Daten. Bildlastige Folien, gut durchblätterbar. Videos verfügbar.
  <https://15721.courses.cs.cmu.edu/spring2024/slides/02-data1.pdf> · <https://15721.courses.cs.cmu.edu/spring2024/slides/03-data2.pdf>

- [ ] **Zeng et al., „An Empirical Evaluation of Columnar Storage Formats", Kap. 1–3 + 6** — ~30 Min. · *weiterführend*
  Seziert Parquet vs. ORC im Detail. In DDIA2 Kap. 4 zitiert. **Kap. 4–5 (Benchmark-Methodik) überspringen**, Kap. 6 „Lessons" ist der Ertrag.
  <https://www.vldb.org/pvldb/vol17/p148-zeng.pdf>

- [ ] **Microsoft Learn, „Online transaction processing (OLTP)" — bis einschließlich „Challenges"** — ~5 Min. · *andere Perspektive*
  Erklärt die OLTP-Seite über ACID und die Drei-Schichten-Architektur, konzeptionell schärfer als IBM. **Ab „OLTP in Azure" aufhören.** Die Schwesterseite zu OLAP würde ich weglassen — irreführender Titel, ~40 % Azure-Produktmatrix.
  <https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/online-transaction-processing>

- [ ] **Zhang et al., „HTAP Databases: A Survey", Abschnitte I und II** — ~60 Min. · *weiterführend*
  Die Vier-Architekturen-Taxonomie und — der eigentliche Wert — **II-B „A Trade-off between Freshness and Isolation"** mit harten Zahlen statt Handwedeln (TiDB: ~10 % Performanceverlust, aber bis 1000 ms Log-Anwendung; HyPer: Mikrosekunden-Snapshots, aber bis 40 % Verlust). **Abschnitt IV nicht linear lesen**, setzt MVCC und LSM-Trees voraus.
  <https://arxiv.org/html/2404.15670v1>

- [ ] **Petrov, *Database Internals*, Kap. 1 — „Column- Versus Row-Oriented DBMS", S. 12–17** — ~12 Min. · *andere Perspektive*
  Nur wegen einer Sache: der sauberen Abgrenzung **Wide Column Store (BigTable/HBase) ≠ Column Store (Vertica/Parquet)**. Über diese Verwechslung stolpert fast jeder einmal. Der Rest des Buchs ist für dieses Thema irrelevant (5 von 349 Seiten).

---

# Block 2 — Kimball: Dimensionale Modellierung & Denormalisierung

> [!success] Abgeschlossen
> Kontrollfragen beantwortet und reviewt (10 sicher · 12 teilweise · 8 leer), Vertiefung in 25 Unterabschnitten ausgearbeitet, 41 Anki-Karten erstellt.
> **Zum Protokoll:** Kimball Kap. 4 wurde nur überflogen, DDIA2 Kap. 3 zunächst unkonzentriert und danach nachgelesen. Fünf der acht Ausfälle lassen sich genau diesen Stellen zuordnen — siehe [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#Wo die Ausfälle liegen — und warum das diagnostisch eindeutig ist|die Analyse in Abschnitt 2.23]]. Der Stoff ist in der Vertiefung trotzdem vollständig ausgearbeitet.

**Warum das zählt:** Das ist der Block, für den das Kapitel existiert — und der bekannte blinde Fleck. Star/Snowflake stand schon in Kapitel 1 als Lücke, in [[Kapitelabschluss 2.1 - Stand und offene Punkte#Bekannte Wissenslücken|2.1 blieb der Selbsttest zur dimensionalen Modellierung weitgehend leer]]. Diesmal gehen wir an die Quelle statt an Zusammenfassungen: **Kimball selbst**, das Buch, aus dem die gesamte Terminologie stammt.

Der Leitfaden verlangt außerdem, den Grain-Schritt begründen zu können („DER wichtigste Schritt") und die goldene Regel der Denormalisierung zu erklären. Beides steht bei Kimball wörtlich.

**Zeitbudget Core: ~4 h 00 Lesen + ~45 Min. Fragen.** Der mit Abstand größte Block — teile ihn ruhig auf zwei bis drei Tage auf.

## Core (Pflicht)

In dieser Reihenfolge — Kimball baut aufeinander auf.

- [x] **1. Kimball, Kap. 1 „DW, BI, and Dimensional Modeling Primer" — S. 7–18** — ~22 Min.
  Der Einstieg: *Fact Tables for Measurements* (S. 10) · *Dimension Tables for Descriptive Context* (S. 13) · *Facts and Dimensions Joined in a Star Schema* (S. 16). Hier steht auch die **Begründung für flache, denormalisierte Dimensionen** — die brauchst du für die Denormalisierungs-Kontrollfrage.
  → *Leitfragen:* Was unterscheidet Fakten- und Dimensionstabellen **konzeptionell** (nicht technisch)? Was sind additive, semi-additive und nicht-additive Fakten — und warum ist die Unterscheidung praktisch wichtig?

- [x] **2. Kimball, Kap. 2 „Kimball Dimensional Modeling Techniques Overview" — S. 37–57** — ~40 Min.
  Die offizielle Technik-Bibliothek: *Fundamental Concepts* (inkl. **Four-Step Design Process, S. 38**, und **Grain, S. 39**) · *Basic Fact Table Techniques* (S. 42) · *Basic Dimension Table Techniques* (S. 46) · *Integration via Conformed Dimensions* (S. 50) · *SCD-Übersicht Typ 0–7* (S. 53).
  **Bewusst als Katalog geschrieben, nicht als Fließtext** — nicht am Stück durchackern, sondern als Landkarte lesen und später wieder aufschlagen. Die SCD-Übersicht ab S. 53 nur überfliegen, Block 3 geht in die Tiefe.
  → *Leitfragen:* Was sind die vier Schritte des Design-Prozesses, und in welcher Reihenfolge **müssen** sie stehen? Warum sind Surrogate Keys besser als natürliche Schlüssel?

- [x] **3. Kimball, Kap. 3 „Retail Sales" — komplett, S. 69–110** — ~84 Min.
  **Das wichtigste Kapitel des ganzen Buchs.** Die erste vollständig durchgerechnete Modellierung, an der Kimball den Prozess vorführt statt ihn zu beschreiben. Wichtige Stationen:
  - *Four-Step Dimensional Design Process*, S. 70–72 — darin **„Step 2: Declare the Grain", S. 71**
  - Anwendung auf den Fall, S. 74–79
  - *Date Dimension*, S. 79–83
  - *Degenerate Dimensions*, S. 93–94 · *Factless Fact Tables*, S. 97–98
  - *Dimension and Fact Table Keys* (Surrogate Keys), S. 98–104
  - **„Resisting Normalization Urges", S. 104–109** — darin *Snowflake Schemas with Normalized Dimensions*, S. 104–106
  Kimball zu Grain, S. 71: *„In debugging thousands of dimensional designs over the years, the most frequent error is not declaring the grain of the fact table at the beginning of the design process. If the grain isn't clearly defined, the whole design rests on quicksand."*
  → *Leitfragen:* Warum ist der Grain **zuerst** festzulegen, vor Dimensionen und Fakten — was geht konkret schief, wenn man es andersherum macht? Welche vier Argumente führt Kimball gegen Snowflaking an, und welches davon ist das technisch stärkste?

- [/] **4. Kimball, Kap. 4 „Inventory" — S. 111–140** — ~58 Min. · **nur überflogen**
  Zwei Dinge, die Kap. 3 nicht liefern konnte:
  - **Die drei Faktentabellen-Typen**, S. 113–122 — *Transaction*, *Periodic Snapshot*, *Accumulating Snapshot*, mit kompaktem Vergleich S. 119–122
  - **Enterprise Data Warehouse Bus Architecture**, S. 123–130 (**Bus Matrix**, S. 125) und **Conformed Dimensions**, S. 130–138
  Die Bus Matrix ist gleichzeitig die Vorbereitung auf Block 5 (Stammdaten) — conformed dimensions *sind* die Warehouse-Antwort auf Master Data.
  → *Leitfragen:* Wann nimmst du eine Periodic-Snapshot- statt einer Transaction-Faktentabelle? Was macht eine Dimension „conformed", und was passiert, wenn zwei Fachbereiche dieselbe Dimension unterschiedlich definieren?

- [x] **5. DDIA2, Kap. 3 „Data Models and Query Languages" — Abschnitte „Normalization, Denormalization, and Joins" und „Stars and Snowflakes: Schemas for Analytics"** — ~20 Min.
  Der Gegenpol zu Kimball: dieselbe Sache aus der Perspektive der Systemarchitektur statt der BI-Praxis. Kleppmann behandelt Normalisierung als Trade-off (Schreibkosten vs. Lesekosten vs. Konsistenz) — genau die Denkfigur, die man braucht, um die goldene Regel der Denormalisierung zu begründen.
  → *Leitfrage:* „Eine denormalisierte Tabelle darf nur aus einer normalisierten Form abgeleitet werden" — welches Argument aus diesem Abschnitt stützt die Regel am besten?

- [x] **6. FDE, Kap. 8 — „Data Modeling" bis Ende „Normalization"** — ~15 Min. · PDF-S. 400–412
  Die klassische Normalformen-Herleitung (1NF/2NF/3NF) an einem E-Commerce-Beispiel, plus die Definition von **Grain** auf PDF-S. 402. Auffrischung zu [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.2 Relationale Datenbanken & ORM/1.2 Relationale Datenbanken & ORM - Vertiefung|Kapitel 1.2]], diesmal mit analytischer Zielrichtung.
  → *Leitfrage:* Warum denormalisiert man in OLAP-Systemen — und welcher der Gründe hängt direkt an dem, was du in Block 1 über Column Stores gelernt hast?

## Vertiefung (optional)

- [ ] **Adamson, *Star Schema: The Complete Reference* (McGraw-Hill, 2010), Kap. 3 „Stars and Cubes" + Kap. 7** — ~90 Min. · *andere Perspektive*
  Rezensenten halten Adamson für den besseren Einstieg als Kimball: explizite Schritt-für-Schritt-Erklärungen, und Grain, SCD Typ 1 und Typ 2 werden in **einem** aufbauenden Kapitel eingeführt statt über zwei verteilt. Kap. 7 stellt „Avoiding the Snowflake" (S. 158) und **„Embracing the Snowflake" (S. 161)** gegenüber — differenzierter als Kimballs klare Ablehnung. Genau dann lesen, wenn Kimball Kap. 3 zäh war.

- [ ] **Kimball, Kap. 16 „Insurance" — S. 397–401 („Common Mistakes")** — ~10 Min. · *weiterführend*
  Kimballs Top-10-Liste der Modellierungsfehler, als Selbstprüfung nach den vier Core-Kapiteln. Fehler Nr. 4 betrifft den Grain.

- [ ] **Kimball, Kap. 6 „Order Management" — S. 178–181** — ~8 Min. · *weiterführend*
  *Degenerate Dimensions* (S. 178) und **Junk Dimensions** (S. 179–181) — die beiden Techniken, die einem beim ersten eigenen Modell garantiert fehlen.

- [ ] **Kimball, Kap. 8 — S. 245–249 (Bridge Tables) und Kap. 13 — S. 329–336 (Factless Fact Tables)** — ~30 Min. · *weiterführend*
  Die beiden Muster für die Fälle, die das Star Schema erst mal nicht abbilden kann: mehrwertige Dimensionen und „Ereignisse ohne Kennzahl".

- [ ] **Kimball, Kap. 11 „Telecommunications" — S. 299–306** — ~15 Min. · *weiterführend*
  Eine **Design-Review-Checkliste**. Ideal direkt vor der Übungsaufgabe, um das eigene Modell abzuklopfen.

- [ ] **Kimball Group, „Kimball Dimensional Modeling Techniques" (freies PDF, 24 S.)** — ~50 Min. · *andere Perspektive*
  Alle ~80 Techniken-Definitionen aus Kap. 2 in einem Dokument, offline verfügbar. Als Nachschlagewerk neben der Übungsaufgabe nützlicher denn als Lesestoff.
  <http://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf>

---

# Block 3 — Slowly Changing Dimensions

> [!success] Abgeschlossen
> Kontrollfragen beantwortet und reviewt (9 sicher · 8 teilweise · 7 leer), Vertiefung in 25 Unterabschnitten ausgearbeitet, 34 Anki-Karten erstellt.
> **Zum Protokoll:** Die dbt-Quelle wurde bewusst übersprungen. Fünf der sieben Ausfälle sind die Fragen 8 und 9 — darunter **Deletes**, die Teil 2 der Übungsaufgabe verlangt und die Kimball im SCD-Kontext nicht behandelt. Vollständig ausgearbeitet in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.15 Deletes|Abschnitt 3.15]].
> **Positiv:** Die Korrektur aus Block 2 (Surrogatschlüssel und SCD Typ 2) hat diesmal getragen — Frage 4a war strukturell exakt.

**Warum das zählt:** SCD Typ 2 ist der Kern des zweiten Teils der Übungsaufgabe — du sollst die Pipeline so erweitern, dass sie Historie führt und Deletes verarbeitet. Der Leitfaden fragt außerdem gezielt, *warum* ausgerechnet Typ 2 der verbreitetste ist. Dieser Block liest die kanonische Darstellung **und** eine moderne Referenzimplementierung, damit du beim Bauen weißt, wie es andere gelöst haben.

**Zeitbudget Core: ~1 h 35 Lesen + ~30 Min. Fragen.**

## Core (Pflicht)

- [x] **1. Kimball, Kap. 5 „Procurement" — S. 147–165** — ~38 Min.
  Die vollständige Typologie, an einem durchgehenden Beispiel:
  - *Slowly Changing Dimension Basics*, S. 147 → **Typ 0** (S. 148) · **Typ 1: Overwrite** (S. 149) · **Typ 2: Add New Row** (S. 150–154) · **Typ 3: Add New Attribute** (S. 154) · **Typ 4: Add Mini-Dimension** (S. 156)
  - *Hybrid Techniques*, S. 159 → Typ 5 (S. 160) · Typ 6 (S. 160) · Typ 7 (S. 162)
  - **Recap mit Übersichtstabelle, S. 164** (Typ | Dimension Table Action | Impact on Fact Analysis)
  S. 150–153 beschreibt genau die Spalten, die du bauen wirst: `Row Effective Date`, `Row Expiration Date`, `Current Row Indicator`, und der Natural Key als Klammer über die Versionen.
  → *Leitfragen:* Warum ist Typ 2 der verbreitetste — welchen Anspruch erfüllt er, den Typ 1 und Typ 3 nicht erfüllen? Was kostet Typ 2 dafür? Was passiert mit **bestehenden Faktenzeilen**, wenn eine Dimension eine neue Version bekommt?

- [x] **2. Kimball, Kap. 19 „ETL Subsystems and Techniques" — S. 464–470 und S. 475–477** — ~20 Min.
  Von der Modellierung zur Mechanik: **Subsystem 9 „Slowly Changing Dimension Manager"** und **Subsystem 10 „Surrogate Key Generator"** (S. 464–470), dazu **Subsystem 14 „Surrogate Key Pipeline"** (S. 475–477) — wie die Fakten-Ladung den richtigen Dimensions-Surrogatschlüssel findet, wenn es mehrere Versionen gibt.
  → *Leitfrage:* Wie ordnet die Pipeline einer eingehenden Faktenzeile die **zeitlich passende** Dimensionsversion zu?

- [x] **3. Kimball, Kap. 20 — S. 503–508 und S. 512–515** — ~20 Min.
  *Step 5: Dimension Historic Load* (S. 503–508) und **Step 7: Dimension Incremental Processing** (S. 512–515).
  Direkt relevant für deine Aufgabe — S. 513 beschreibt exakt das Snapshot-Setup aus dem Leitfaden: *„Often it's easiest to pull the current snapshots of the dimension tables in their entirety and let the transformation step determine what has changed and how to handle it."* S. 514 liefert die Änderungserkennung: **Hash-Spalten** (`hash_type1`, `hash_type2`) statt spaltenweisem Vergleich, oder der `EXCEPT`-Operator.
  → *Leitfrage:* Warum vergleicht man Hashes statt Spalte für Spalte — und was ist der Haken daran?

- [/] **4. dbt Docs, „Add snapshots to your DAG"** — ~23 Min. · **überflogen**
  Eine moderne, produktive **Referenzimplementierung** von SCD2, mit benannten Spalten: `dbt_valid_from`, `dbt_valid_to` (`NULL` = aktuell), `dbt_scd_id`, `dbt_is_deleted`. Erklärt beide Erkennungsstrategien (*timestamp* vs. *check*) und die Behandlung von **Hard Deletes** — genau der Teil, den deine Pipeline in Aufgabenteil 2 auch können muss.
  → *Leitfragen:* Warum ignorieren dbt-Snapshots bewusst `--full-refresh`? Was sagt das darüber, wie sich ein SCD2-Lauf zur Idempotenz-Idee aus [[2.1 Advanced SQL & ETL - Vertiefung#5. Idempotenz|2.1, Abschnitt 5]] verhält?
  <https://docs.getdbt.com/docs/build/snapshots>

## Vertiefung (optional)

- [ ] **Kimball Group Reader, Artikel 11.37 „Using the SQL MERGE Statement for Slowly Changing Dimensions", S. 593–595** — ~8 Min. · *weiterführend*
  Drei Seiten, die exakt die Leitfaden-Kontrollfrage „Warum MERGE statt INSERT INTO?" mit SCD2 verbinden. Dazu Artikel 9.25 „SCD Types 0, 4, 5, 6 and 7" (S. 378).

- [ ] **Delta Lake Docs, „Table deletes, updates, and merges" — nur das SCD-Typ-2-Beispiel** — ~15 Min. · *weiterführend*
  Ein ausgearbeitetes SCD2-`MERGE` in Delta Lake. Zeigt die Mehrschritt-Logik (Version schließen + neue Version einfügen), die sich mit einem einzelnen `MERGE` nicht ausdrücken lässt.
  <https://docs.delta.io/delta-update/>

- [ ] **DEDP, Kap. 5 — „Pattern: Static Joiner", S. 116–120** — ~12 Min. · *andere Perspektive*
  SCD als *Pattern* formuliert statt als Kimball-Technik, mit DDL-Beispiel (`start_date` / `end_date DEFAULT '9999-12-31'` / `PRIMARY KEY(id, start_date)`) und einer präzisen Gotcha-Notiz zum Verhältnis SCD ↔ Idempotenz.

---

# Block 4 — Data Vault, OBT & Kimball vs. Inmon

> [!success] Abgeschlossen — der stärkste Block bisher
> Kontrollfragen beantwortet und reviewt (**14 sicher · 7 teilweise · 4 leer** — Trefferquote 56 %, beste des Kapitels), Vertiefung in 31 Unterabschnitten ausgearbeitet, 36 Anki-Karten erstellt, sechs Q&A-Fragen beantwortet.
> **Was anders war:** Alle Quellen gelesen, und die Exzerpte waren deutlich reichhaltiger — zu vier von sieben Quellen mit eigenem Kommentar je Eintrag.
> **Die Ausfälle lagen wieder in den Quellen ohne Exzerpt** (4 und 5). Vierter Block in Folge mit derselben Zuordnung.

**Warum das zählt:** Der Leitfaden fragt „Dimensionale Modellierung vs. Data Vault: Wann nutzt man was? Was ist weiter verbreitet und warum?" — das setzt voraus, dass du **beide** Modelle und die dritte Position (Inmon) kennst. Dazu kommt OBT als der Ansatz, den moderne Cloud-Warehouses und dbt in der Praxis oft bevorzugen. Dieser Block ist der Vergleichsblock: keine neue Technik, sondern Urteilsfähigkeit.

Er schließt außerdem den offenen Punkt aus [[Kapitelabschluss 2.1 - Stand und offene Punkte#Nicht gelesen|2.1, Nachtrag A (Kimball vs. Inmon)]].

**Zeitbudget Core: ~2 h 30 Lesen + ~35 Min. Fragen.**

## Core (Pflicht)

- [x] **1. Keboola, „Kimball vs Inmon"** — ~6 Min.
  Der schnelle, symmetrisch aufgebaute Einstieg: beide Ansätze bekommen exakt dieselbe Vor-/Nachteil-Struktur. Den Schlussabschnitt „Data Warehousing Tools" überspringen, das ist Produktwerbung.
  → *Leitfrage:* Was ist der eine strukturelle Unterschied, aus dem alle anderen folgen (Richtung des Aufbaus)?
  <https://www.keboola.com/blog/kimball-vs-inmon>

- [x] **2. FDE, Kap. 8 — „Techniques for Modeling Batch Analytical Data": Einleitung, „Inmon", „Kimball", „Data vault"** — ~30 Min. · PDF-S. 412–434
  Alle drei Ansätze in einem Text, vom selben Autor, mit **durchgerechnetem Data-Vault-Beispiel** (Hubs PDF-S. 422 · Links 426 · Satellites 431). Das Buch nennt seine eigene Behandlung „cursory" — als Überblick vor dem Primärtext ist sie trotzdem genau richtig.
  → *Leitfragen:* Was ist die Aufgabenteilung zwischen Hub, Link und Satellite? Warum braucht Data Vault so viele Tabellen für so wenig Fachlichkeit — welches Problem kauft man damit?

- [x] **3. Linstedt & Olschimke, Kap. 4 „Data Vault 2.0 Modeling" — S. 89–122** — ~66 Min.
  Die Primärquelle. Hub/Link/Satellite in der Definition ihres Erfinders, mit den Metadatenspalten (Hash Key, Load Date, Record Source), die den Unterschied zwischen „verstanden" und „nur gehört" ausmachen.
  Ergänzend, falls du die Architektur mitnehmen willst: **Kap. 2 „Scalable Data Warehouse Architecture", S. 17–32** (~30 Min.) — dort steht die Schichtung Staging → Raw Vault → Business Vault → Information Mart, die in Block 5 wiederkommt.
  → *Leitfragen:* Warum trennt Data Vault Schlüssel (Hub), Beziehungen (Link) und beschreibende Attribute (Satellite) so strikt? Welche Art von Änderung im Quellsystem ist dadurch **billig**, die bei Kimball teuer wäre?

- [x] **4. Kaminsky, „Data warehouse modeling: Star schema vs. OBT" (Fivetran-Blog)** — ~8 Min.
  Der Benchmark, auf den sich die ganze OBT-Diskussion beruft: 25–30 % (Redshift), ~25 % (Snowflake), ~50 % (BigQuery) schnellere Queries für die denormalisierte Variante bei BI-typischen Workloads. Der Autor ist nicht bei Fivetran angestellt, die Methodik ist offengelegt, der Scope ehrlich abgegrenzt.
  → *Leitfrage:* Auf welche Query-Art bezieht sich der Vorteil — und für welche Art von Abfrage gilt er ausdrücklich **nicht**?
  <https://www.fivetran.com/blog/star-schema-vs-obt>

- [x] **5. FDE, Kap. 8 — „Wide denormalized tables"** — ~8 Min. · PDF-S. 434–437
  **Terminologie-Warnung:** Das Buch verwendet den Begriff „One Big Table"/„OBT" nirgends — es heißt durchgehend *wide denormalized tables*. Gleiche Sache, anderer Name.
  → *Leitfrage:* Welche Nachteile nennt FDE, die im Fivetran-Benchmark nicht vorkommen (weil ein Benchmark sie nicht messen kann)?

- [x] **6. dbt Docs, „How we structure our dbt projects" — Teil 4 „Marts"** — ~10 Min.
  Die Position des meistgenutzten Transformationswerkzeugs — und sie widerspricht Kimball offen: *„Unlike in a traditional Kimball star schema though, in modern data warehousing — where storage is cheap and compute is expensive — we'll happily borrow and add any and all data from other concepts that are relevant … ✅ Wide and denormalized."*
  → *Leitfrage:* Welches Argument steckt hinter „storage is cheap, compute is expensive" — und unter welcher Bedingung wäre es falsch?
  <https://docs.getdbt.com/best-practices/how-we-structure/4-marts>

- [x] **7. dbt Labs, Kathryn Chubb, „Modeling for success: Building data structures that last"** — ~13 Min.
  **Bewusst als Widerspruch zu Quelle 6 lesen.** Derselbe Anbieter, anderes Ressort: Der Marketing-Blog empfiehlt dimensionale Modellierung samt Bus-Matrix, während die technische Doku „wide and denormalized" empfiehlt. Behandelt Medallion, Kimball, Data Vault und OBT nebeneinander.
  → *Leitfrage:* Wie erklärst du dir den Widerspruch zwischen den beiden dbt-Quellen? Reden sie über verschiedene Dinge, oder ist eine von beiden schlicht Marketing?
  <https://www.getdbt.com/blog/modeling-success-dbt>

## Vertiefung (optional)

- [ ] **Arab, „Enterprise Data Modelling Methodologies: A Comparative Analysis of Inmon, Kimball, and Data Vault" (arXiv)** — ~35 Min. · 25 S. · *andere Perspektive*
  Die einzige freie Quelle, die alle **drei** Ansätze systematisch gegenüberstellt, mit Architekturdiagrammen pro Ansatz und einem Benchmark über 12 Kriterien (Tab. 5). Beantwortet nebenbei die Block-5-Frage „welches Modell in welcher Schicht" wörtlich.
  *Einschränkung: Einzelautor-Preprint, nicht peer-reviewed.*
  <https://arxiv.org/abs/2606.29355>

- [ ] **Scalefree, „Data Vault Glossary: Hub, Link, Satellite, Business Vault, and More" — bis „Information Mart"** — ~8 Min. · *andere Perspektive*
  Kompaktes Nachschlagewerk von einem offiziellen Data-Vault-2.0-Trainingspartner. Die Einträge ab „Ghost Record" für den Erstkontakt überspringen.
  <https://www.scalefree.com/blog/data-vault/data-vault-glossary-hub-link-satellite-business-vault-and-more/>

- [ ] **Linstedt & Olschimke, Kap. 7 „Dimensional Modeling" — S. 171–194** — ~46 Min. · *weiterführend*
  Wie man aus einem Data Vault ein Star Schema ableitet. Die praktische Auflösung der Scheinalternative „Kimball **oder** Data Vault" — in großen Häusern ist es meist beides, hintereinander.

- [ ] **Späti, „Inmon vs Kimball" (ssp.sh)** — ~6 Min. · *andere Perspektive*
  Kein Vendor, historische Einordnung, plus Kommentare erfahrener Praktiker unter dem Artikel.
  <https://www.ssp.sh/brain/inmon-vs-kimball/>

- [ ] **Kimball, Kap. 1 — S. 18–30** — ~26 Min. · *andere Perspektive*
  Kimballs eigene Darstellung der Konkurrenzarchitekturen: *Independent Data Mart Architecture* (S. 26), *Hub-and-Spoke Corporate Information Factory (Inmon)* (S. 28), *Hybrid* (S. 29). Die Gegenposition in Inmons Worten zu lesen wäre besser — aber Kimball ist hier fairer, als man erwartet. (Wird in Block 5 ohnehin gelesen.)

---

# Block 5 — Schichtenarchitektur, Medallion & Master Data

> [!success] Abgeschlossen — beste Quote des Kapitels
> Kontrollfragen beantwortet und reviewt (**16 sicher · 9 teilweise · 3 falsch** — 57 % Treffer, 11 % Ausfälle), Vertiefung in 32 Unterabschnitten ausgearbeitet, 39 Anki-Karten erstellt, drei Q&A-Fragen beantwortet.
> **Die Zweiteilung hat funktioniert:** Etappe 1 erreichte **80 %** — das stärkste Einzelsegment des gesamten Kapitels. Etappe 2 (Master Data) fiel auf 31 % ab, und alle vier schwachen Antworten lagen in den drei Quellen **ohne Exzerpt**.
> Konsequenz für Block 6: **drei** Etappen statt zwei.

**Warum das zählt:** Hier hängen zwei der anspruchsvollsten Kontrollfragen des Leitfadens: die **Skizze einer prototypischen Warehouse-Umgebung** (welche Schichten, welche Datenflüsse, welches Modell wo) und die Frage nach **Stammdaten** (was zeichnet sie aus, normalisiert oder denormalisiert, wo im Warehouse). Beide sind Synthesefragen — sie setzen die Blöcke 2 bis 4 voraus und ordnen sie ein.

Die Medallion-Architektur ist dabei der aktuelle Modebegriff für eine sehr alte Idee. Der Leitfaden fragt explizit nach Schwächen; die Herstellerdoku liefert dazu nichts, deshalb stehen unten drei Gegenpositionen im Core.

> [!important] Dieser Block wird in **zwei getrennten Etappen** gelesen
> Konsequenz aus Block 2: Vier Stunden in einer Runde waren zu viel, und die Ausfälle lagen genau in den überflogenen Quellen. Deshalb hier keine Empfehlung, sondern eine Vorgabe:
> - **Etappe 1 — Teil A + B** (Schichtenarchitektur und Medallion, ~2 h 15). Danach Kontrollfragen zu Teil A/B beantworten und pausieren.
> - **Etappe 2 — Teil C** (Master Data, ~1 h 05). Danach die Fragen zu Teil C.
>
> Das Fragenset in der Vertiefung wird entsprechend zweigeteilt.

**Zeitbudget Core: ~3 h 20 Lesen + ~45 Min. Fragen — aufgeteilt auf zwei Etappen.**

## Core (Pflicht)

### Teil A — Schichtenarchitektur (~1 h 30)

- [x] **1. Kaul, „Data Warehouse Layers (Modern Data Warehousing, Part 1)"** — ~5 Min.
  Der Aufwärmer, der in vier Sätzen Schichten **und** Modell pro Schicht benennt: *„The classic data warehouse architecture, going back to Bill Inmon, consists of three layers … Nowadays, the core layer is usually modeled using data vault constructs … The presentation layer is usually modeled in a dimensional way."* Bonus: der Unterschied zwischen *transient* und *persistent staging area*.
  → *Leitfrage:* Wozu dient eine **persistente** Staging Area — wogegen ist sie die Versicherung?
  <https://www.linkedin.com/pulse/data-warehouse-layers-modern-warehousing-part-1-christian-kaul>

- [x] **2. Kimball, Kap. 1 — S. 18–30** — ~26 Min.
  **Kimball's DW/BI Architecture** (S. 18): *Operational Source Systems* (18) · *ETL System* (19) · *Presentation Area* (21) · *BI Applications* (22), dazu die **Restaurant-Metapher** (S. 23) und ab S. 26 die **Alternativarchitekturen**. Aus diesen 13 Seiten kannst du die geforderte Skizze direkt bauen.
  → *Leitfrage:* Welche Regel stellt Kimball für die Presentation Area auf — was darf dort **nicht** passieren?

- [x] **3. DMBOK2, Kap. 11 „Data Warehousing and Business Intelligence" — §1.3, S. 384–394** — ~22 Min.
  Die herstellerneutrale Norm-Sicht: *Corporate Information Factory (Inmon)* S. 386–388 · *Dimensional DW (Kimball)* · **§1.3.7.3 „Data Storage Areas"** — die Speicherzonen einzeln aufgezählt: *Staging area* · **Reference and Master Data conformed dimensions** · *Central Warehouse (atomic layer)* · *Operational Data Store* · *Data marts* · *Cubes*. Dazu §1.3.8.1, das vergleicht, wie Inmon, Kimball und Data Vault jeweils Historie speichern.
  **Diese eine Liste beantwortet beide Kontrollfragen gleichzeitig** — Stammdaten sind hier eine eigene Zone *neben* Staging/Core/Mart.
  Norm-Prosa, aber gut strukturierte. §2–§6 (S. 394–414) sind Gremiensprache, überspringen.
  → *Leitfrage:* Was ist ein **Operational Data Store**, und wodurch unterscheidet er sich vom Warehouse?

- [x] **4. dbt Docs, „How we structure our dbt projects" — Teile 1–3** — ~27 Min.
  Die moderne Entsprechung derselben Logik, in der Sprache des Werkzeugs: *staging* (`stg_<source>_<entity>`, 1:1 zur Quelle) → *intermediate* (`int_*`) → *marts*. Leitprinzip: **„source-conformed → business-conformed"**. Teil 4 hast du in Block 4 schon gelesen.
  → *Leitfrage:* Wie bilden sich Staging/Intermediate/Marts auf die klassischen Schichten aus Quelle 2 und 3 ab — und wo passt die Analogie **nicht**?
  <https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview>

### Teil B — Medallion (~45 Min.)

- [x] **5. Databricks, „What is the medallion lakehouse architecture?"** — ~9 Min.
  Die Definition an der Quelle: Bronze (Rohdaten, minimale Bereinigung), Silber (Validierung, Deduplizierung, erste Modellierung), Gold (Geschäftslogik, Aggregate). Sauber strukturiert und konkret.
  **Kritik: praktisch keine** — der einzige relativierende Satz ist *„a recommended best practice but not a requirement"*. Deshalb die drei folgenden Quellen.
  → *Leitfrage:* Welche Regel gilt für die Bronze-Schicht, und warum ist sie so streng formuliert?
  <https://docs.databricks.com/aws/en/lakehouse/medallion>

- [x] **6. Reis, „Medallion Architecture is NOT a Data Model"** — ~8 Min.
  Das Kernargument des Blocks: Bronze/Silver/Gold sind **Lifecycle-Stufen, kein Datenmodell**. Medallion ersetzt Kimball, Inmon oder Data Vault nicht, sondern liegt orthogonal dazu. Der Text stellt damit die Verbindung zu den Blöcken 2–4 her.
  → *Leitfrage:* „Gold = Star Schema" — warum stimmt das nicht zwangsläufig?
  <https://practicaldatamodeling.substack.com/p/medallion-architecture-is-not-a-data>

- [x] **7. Bellemare, „The End of the Bronze Age: Rethinking the Medallion Architecture" (InfoQ) — Key Takeaways + Flaw #1 bis #5** — ~10 Min.
  Fünf explizit durchnummerierte Schwächen, jede einzeln begründet: Konsument trägt die Zugriffslast · teuer · Datenqualität nachträglich schwer herstellbar · Bronze ist fragil · keine Wiederverwendbarkeit für operative Workloads. Die zweite Hälfte („Shift Left", Data Contracts) ist die Gegenthese des Autors und geht über die Kontrollfrage hinaus — **du brauchst nur die erste Hälfte**.
  → *Leitfrage:* Welche der fünf Schwächen ist ein Problem der *Architektur*, und welche eher eines der *Umsetzung*?
  <https://www.infoq.com/articles/rethinking-medallion-architecture/>

- [x] **8. Packkildurai, „Revisiting Medallion Architecture: Data Vault in Silver, Dimensional Modeling in Gold"** — ~15 Min.
  Der Text, der Block 4 und Block 5 zusammenbindet: welches Modell in welche Medaille gehört, und — wichtiger — was der **Übergang** zwischen ihnen kostet (Abschnitt IV: PIT-Tabellen, Surrogatschlüssel-Verwaltung, inkrementelle Ladelogik).
  ⚠️ Achtung: Es gibt zwei ähnlich benannte Artikel desselben Autors. Gemeint ist der vom **17.10.2025** mit `-760` am Ende der URL.
  → *Leitfrage:* Warum passt Data Vault gut nach Silber und schlecht nach Gold?
  <https://www.dataengineeringweekly.com/p/revisiting-medallion-architecture-760>

### Teil C — Master Data / Stammdaten (~1 h 05)

- [x] **9. IBM Redbooks TIPS1016, „Reference Data Management" — nur der erste Abschnitt** — ~4 Min.
  Die sauberste Vierfach-Abgrenzung in vier Sätzen: *„Reference data is distinct from transactional data and master data. Transactional data is … produced by transactions; master data … represents the key business entities that participate within transactions. Reference data is also distinct from metadata, which describes the structure of an entity."* Ab „Business value" wird es Produktprosa.
  <https://www.redbooks.ibm.com/abstracts/tips1016.html?Open>

- [x] **10. Wikipedia, „Master data" + „Reference data"** — ~9 Min.
  Beide Artikel sind ungewöhnlich sauber (keine Wartungsbausteine) und liefern das schärfere zweite Kriterium: Eine Änderung an **Reference-Data-Werten** kann eine Prozessänderung erfordern, eine Änderung an **Master Data** läuft durch bestehende Prozesse.
  ⚠️ Den Artikel „Master data management" **nicht** lesen — trägt zwei Wartungsbausteine, teils quellenlos.

- [x] **11. DMBOK2, Kap. 10 „Reference and Master Data" — §1.3.1, §1.3.3.2 und §1.3.4** — ~22 Min. · S. 350–353, 358–362, 368–370
  Die Substanz: Chisholms sechsschichtige Taxonomie und die Definition von Master Data als *Aggregat* (§1.3.1) · **Trusted Source / Golden Record** und die MDM Key Processing Steps (§1.3.3.2, Fig. 76 auf S. 361) · **die Hub-Architekturen** (§1.3.4, Fig. 77 auf S. 370).
  ⚠️ **Korrektur zu einer weit verbreiteten Angabe:** DMBOK2 nennt **drei** Ansätze — *Registry*, *Transaction Hub*, *Consolidated* (Hybrid aus beiden). Die oft zitierte Vierer-Taxonomie „Registry / Consolidation / **Coexistence** / Centralized" ist **Hersteller-Vokabular** (Gartner/Profisee/Stibo), nicht DAMA. Beide Begriffssysteme sind im Umlauf; wissen, dass es zwei gibt, ist der eigentliche Punkt.
  → *Leitfragen:* Was macht einen „Golden Record" golden — und wer entscheidet bei widersprüchlichen Quellen, welcher Wert gewinnt? Sollte man Stammdaten normalisiert oder denormalisiert speichern — und warum ist das eine Fangfrage, solange man nicht sagt, **wo** im Warehouse?

- [x] **12. Kimball, Kap. 8 „Customer Relationship Management" — S. 256–259** — ~8 Min.
  Der Brückenschlag Warehouse ↔ MDM, mit **Figure 8-14 „Two styles of master data management"**: *Enterprise MDM* (Hub vor dem Warehouse) gegen *Downstream MDM* (das Warehouse baut den Hub selbst). Kimball mit erfrischendem Realismus: *„Some organizations are lucky enough to have a centralized MDM system … But such centralization is rare in the real world."* Dazu Customer Matching, Fuzzy Logic, „no secret weapon".
  → *Leitfrage:* Wenn Stammdaten im Warehouse als **conformed dimension** landen — was ist dann der Unterschied zwischen „wir machen MDM" und „wir machen dimensionale Modellierung ordentlich"?

- [x] **13. Splink Topic Guide, „The Fellegi-Sunter Model" + Linacre, „An Interactive Introduction"** — ~12 Min.
  Wie Entity Resolution technisch funktioniert: Parameter λ/m/u → Match Weights → Match Probability. Open Source vom UK Ministry of Justice, null Marketing.
  → *Leitfrage:* Warum ist probabilistisches Matching genauer als Fuzzy-String-Vergleich?
  <https://moj-analytical-services.github.io/splink/topic_guides/theory/fellegi_sunter.html> · <https://www.robinlinacre.com/intro_to_probabilistic_linkage/>

## Vertiefung (optional)

- [ ] **Serra, *Deciphering Data Architectures* (O'Reilly, 2024) — S. 63–65** — ~6 Min. · *andere Perspektive*
  Die beste Impfung gegen Begriffsverwirrung, die es gibt: **fünf** Zonen mit allen Synonymen — *Raw layer* („also called the bronze layer, staging layer, or landing area"), *Conformed*, *Cleansed* („also called the silver … layer"), *Presentation* („also called … gold … layer"), *Sandbox*.
  Bemerkenswert: Serra benutzt den Begriff „Medallion Architecture" **nirgends** im Buch. Bronze/Silver/Gold erscheinen nur als Synonyme. Das ist selbst ein Befund — die Zonen-Sprache ist älter und uneinheitlicher, als Databricks es darstellt.

- [ ] **Wolter & Haselden, „The What, Why, and How of Master Data Management" (Microsoft, 2006)** — ~38 Min. · *weiterführend*
  Der meistzitierte freie MDM-Primer, und er hält. Besonders der Abschnitt **„Deciding What to Manage"** mit acht Kriterien (Behavior, Life Cycle, Cardinality, Lifetime, Complexity, Value, Volatility, Reuse) — die beste Darstellung überhaupt davon, dass Stamm- vs. Bewegungsdaten ein **Spektrum** ist, kein Schalter.
  Nur über Wayback erreichbar: <http://web.archive.org/web/20170714193734/https://msdn.microsoft.com/en-us/library/bb190163.aspx>

- [ ] **Enzyklopädie der Wirtschaftsinformatik, „Stammdatenmanagement"** — ~6 Min. · *andere Perspektive*
  Die beste **deutsche** Quelle, akademisches Nachschlagewerk mit echten Literaturangaben. Liefert eine feinere Typologie als die englische Literatur: *Zustandsdaten* (→ Stammdaten, Bestandsdaten) vs. *Abwicklungsdaten* (→ Bewegungsdaten, Änderungsdaten). Für die deutschsprachige Vertiefungsnotiz nützlich.
  <https://wi-lex.de/lexikon/informations-daten-und-wissensmanagement/informationsmanagement/informationsmanagement-aufgaben-des/stammdatenmanagement/>

- [ ] **Cordo, „Medallion Architecture in a Data Product World"** — ~6 Min. · *andere Perspektive*
  Liest Medallion als Wiederauflage der Software-Layered-Architecture und des klassischen EDW: *„there is very little difference between Medallion architecture and how a traditional EDW is composed. If anything, the EDW conceptually was a little cleaner."* Andere Kritikachse als Bellemare.
  <https://medium.com/datafutures/medallion-architecture-in-a-data-product-world-3758d17b6cf6>

- [ ] **Microsoft Learn, „Implement Medallion Lakehouse Architecture in Fabric"** — ~12 Min. · *andere Perspektive*
  Zweiter Hersteller, mit zwei Dingen, die Databricks nicht hat: jede Schicht in einem eigenen Workspace, und die Variante „Gold als Data Warehouse". Auffällig: **kein einziges Vorkommen** von Kimball, Sternschema oder dimensionaler Modellierung — was Joe Reis' These stützt.
  <https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture>

- [ ] **Kimball, Kap. 19 — S. 455–463** — ~18 Min. · *weiterführend*
  „Cleaning and Conforming Data", darunter Subsystem 7 *Deduplication* (S. 460) und Subsystem 8 *Conforming* (S. 461). Hier entstehen Golden Records technisch.

- [ ] **Kimball Group, Ross, „The Matrix: Revisited"** — ~10 Min. · *weiterführend*
  Eröffnet wörtlich mit MDM und positioniert die Bus Matrix als Konformitäts-Mechanismus. Der direkteste Brückenartikel zwischen Kimball und MDM im ganzen Kimball-Archiv.
  <https://www.kimballgroup.com/2005/12/the-matrix-revisited/>

- [ ] **„Delta Lake: The Definitive Guide" (O'Reilly, 2024), Kap. 9 „Architecting Your Lakehouse"** — ~20–25 Min. (*geschätzt*) · *weiterführend*
  Die Buchquelle zu Medallion, von Delta-Lake-Maintainern. Enthält „The Medallion Architecture" mit Abschnitten pro Schicht plus „Streaming Medallion Architecture". **Seitenzahlen konnte ich nicht verifizieren**, die Zeitangabe ist geschätzt.

---

# Block 6 — Partitionierung & idempotente Ladeverfahren

> [!success] Abgeschlossen — damit ist das Kapitel durch
> Kontrollfragen beantwortet und reviewt (**12 sicher · 14 teilweise · 4 falsch** — 40 % Treffer, 13 % Ausfälle), Vertiefung in 34 Unterabschnitten ausgearbeitet, 41 Anki-Karten erstellt, Q&A beantwortet.
> **Zum Protokoll:** Exzerpte nur zu 4 von 17 Quellen, alle aus Etappe 1 — der Block lief parallel zur Übungsaufgabe. Etappe 2 war trotzdem die beste, weil sie auf dem Idempotenz-Wissen aus Kapitel 2.1 aufbaute.

**Warum das zählt:** Zwei Leitfaden-Kontrollfragen: „Wie funktioniert Partitionierung in Theorie und Praxis? Wie entscheide ich meine Partitionierungsstrategie?" und „Warum sollten deine Pipelines nicht `INSERT INTO` nutzen, sondern lieber `MERGE` oder `INSERT OVERWRITE`?"

Die zweite ist der Warehouse-Aufsatz auf etwas, das du schon kannst: **Idempotenz hast du in [[2.1 Advanced SQL & ETL - Vertiefung#5. Idempotenz|2.1, Abschnitt 5]] ausführlich behandelt** — Zustellgarantien, atomares Schreiben, Task = Transaktion, Backfills, Idempotency Keys. Dieser Block wiederholt das nicht, sondern zeigt, wie es im Warehouse konkret instanziiert wird.

**Zeitbudget Core: ~4 h 20 Lesen + ~40 Min. Fragen.**

## Core (Pflicht)

### Teil A — Partitionierung anfassen (~52 Min.)

- [x] **1. Crunchy Data Playground, „Partitioning"** — ~7 Min. · **mitmachen, nicht lesen**
  Ein echtes Postgres-Terminal im Browser, null Setup. Partitionierte Tabelle anlegen, Partitionen erzeugen, Daten einfügen, Partitionen rotieren, `MERGE`/`SPLIT PARTITION` (PG 17). Der einzige Weg, das Thema in den Fingern zu haben statt nur im Kopf.
  *Quelle ohne Autor und ohne Datum — als undatiert zitieren.*
  <https://www.crunchydata.com/developers/playground/partitioning>

- [x] **2. PostgreSQL-Doku, Kap. 5.12 „Table Partitioning" — Abschnitte 5.12.1, 5.12.2, 5.12.4, 5.12.6** — ~33 Min.
  Der Referenztext. **5.12.3 „Partitioning Using Inheritance" überspringen** — Altlast, spart ein Drittel. Die tragenden Teile sind 5.12.2 (Declarative Partitioning inkl. *Limitations*), **5.12.4 Partition Pruning** und **5.12.6 Best Practices**, wo die Strategiefrage schon teilweise beantwortet wird: *„Never just assume that more partitions are better than fewer partitions, nor vice-versa."*
  *Versionshinweis: 5.12 gilt für PG 17/18; in PG ≤ 16 war es 5.11.*
  → *Leitfragen:* Was ist der Unterschied zwischen *Partition Pruning* und *Constraint Exclusion*? Welche Einschränkungen gelten für Primary Keys und Unique Constraints auf partitionierten Tabellen — und warum?
  <https://www.postgresql.org/docs/current/ddl-partitioning.html>

- [x] **3. Christensen, „Partitioning with Native Postgres and pg_partman" (Crunchy Data, 2022)** — ~12 Min.
  Was die offizielle Doku nicht liefert: argumentierende Prosa und die **Automatisierung** (`pg_partman`, Retention Policies, Partitionen im Voraus anlegen). Wichtige didaktische Korrektur: Partitionierung zuerst für Lifecycle und billiges `DROP`, erst danach für Performance.
  → *Leitfrage:* Warum ist `DROP PARTITION` fundamental anders als `DELETE FROM ... WHERE`?
  <https://www.crunchydata.com/blog/native-partitioning-with-postgres>

### Teil B — Im Warehouse ist es etwas anderes (~50 Min.)

- [x] **4. BigQuery, „Introduction to partitioned tables"** — ~14 Min.
  Partitionierung als **Kosteninstrument**: exakte Kostenschätzung vor Ausführung. Konkrete Granularitätsregeln (stündlich bei hohem Volumen über kurze Zeiträume, monatlich/jährlich bei wenig Daten über lange Zeiträume) und die Anti-Over-Partitioning-Regel: clustern statt partitionieren, wenn eine Partition weniger als ~10 GB enthält. Hartes Limit: 10.000 Partitionen pro Tabelle.
  <https://docs.cloud.google.com/bigquery/docs/partitioned-tables>

- [x] **5. BigQuery, „Introduction to clustered tables"** — ~9 Min.
  **Block Pruning** als das Konzept, das Partitionierung ergänzt. Wichtig: Die **Reihenfolge** der Clustering-Spalten ist bedeutsam, max. 4 Spalten, und hohe Kardinalität ist hier eine *Stärke* — anders als bei Partitionierung.
  <https://docs.cloud.google.com/bigquery/docs/clustered-tables>

- [x] **6. Snowflake, „Micro-Partitions & Data Clustering"** — ~10 Min.
  Der wichtigste Kontrast des ganzen Blocks: **50–500 MB pro Micro-Partition, vollautomatisch, nicht vom Nutzer definierbar.** Snowflake nimmt dem Data Engineer die Partitionierungsentscheidung weg und ersetzt sie durch metadatenbasiertes Pruning.
  → *Leitfrage:* Was ist der Preis dieser Automatik — was kannst du bei Snowflake *nicht* mehr steuern?
  <https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions>

- [x] **7. Databricks, „When to partition tables on Databricks"** — ~7 Min.
  Die Seite mit den harten Zahlen: *„Most tables on Databricks with less than 100 TB of data don't need partitioning."* · **„With less than 1 TB of data, don't partition."** · *„verify that each partition contains at least 1 GB of data."*
  <https://docs.databricks.com/aws/en/tables/partitions>

- [x] **8. Databricks, „Use liquid clustering for tables" — nur „When to use" und „Choose clustering keys"** — ~10 Min.
  *„Liquid clustering … replaces table partitioning and `ZORDER`."* — die Richtung, in die sich das Feld bewegt: weg von manuell gewählten Partitionsschlüsseln. Der Rest der Seite (33 Min. gesamt) ist Betriebsdetail.
  <https://docs.databricks.com/aws/en/delta/clustering>

> [!question] Ein Widerspruch, der keiner ist
> Databricks sagt **mindestens 1 GB pro Partition**, BigQuery sagt **unter 10 GB gar nicht erst partitionieren**. Das ist kein Fehler in einer der Quellen — es zeigt, dass diese Zahlen **engine-spezifische Heuristiken sind, keine Naturgesetze**. Halte das beim Lesen fest, es ist eine der besseren Erkenntnisse dieses Blocks.

### Teil C — Strategie wählen (~21 Min.)

- [x] **9. Saktheeswaran, „Knowing Your Data Partitioning Vices on the Data Lakehouse" (Onehouse, 2023)** — ~19 Min.
  **Die gesuchte Quelle zur Strategiefrage** — die einzige, die Schlüsselwahl, Partitionsgröße, Over-Partitioning und **Skew** in einem Text behandelt, und zwar als Anti-Pattern-Erzählung („drei Vices"). Mit Messwerten: flaches Performance-Band von ~105 MB bis 2 GB pro Datei, Absturz im einstelligen MB-Bereich. Und der Satz, den man sich merkt: *„uniform data distribution across naturally partitioned datasets is too perfect to be true."*
  *Vendor-Blog (Hudi-Anbieter), Lakehouse-zentriert — die diagnostische Systematik ist trotzdem herstellerneutral.*
  → *Leitfragen:* Was ist der Zusammenhang zwischen Partitionierungsschema und **Query Shape**? Warum ist Over-Partitioning schlimmer als Under-Partitioning?
  <https://www.onehouse.ai/blog/knowing-your-data-partitioning-vices-on-the-data-lakehouse>

- [x] **10. delta.io, „Best practices" — Abschnitt „Choose the right partition column"** — ~2 Min.
  Zwei Absätze, hohe Dichte: *„You can partition by a column if you expect data in that partition to be at least 1 GB."* Plus die Warnung vor hoher Kardinalität am Beispiel User-ID.
  <https://docs.delta.io/latest/best-practices.html>

### Teil D — Idempotente Ladeverfahren (~1 h 15)

- [x] **11. dbt Docs, „Configure incremental models"** — ~14 Min.
  Die Voraussetzung für Quelle 12: `is_incremental()`, `unique_key`, `on_schema_change`, Rebuild-Verhalten.
  <https://docs.getdbt.com/docs/build/incremental-models>

- [x] **12. dbt Docs, „About incremental strategy"** — ~11 Min.
  **Die Quelle, die die Leitfaden-Kontrollfrage direkt beantwortet.** Alle vier Verfahren nebeneinander: `append` · `delete+insert` · `merge` · `insert_overwrite` (plus `microbatch`), mit Adapter-Matrix. `append` = „insert without updates; doesn't check duplicates" ist exakt das `INSERT INTO`-Antipattern der Kontrollfrage.
  → *Leitfrage:* Für welchen Fall ist `insert_overwrite` besser als `merge`, und warum?
  <https://docs.getdbt.com/docs/build/incremental-strategy>

- [x] **13. dbt Docs, „BigQuery configurations" — nur der Abschnitt `insert_overwrite`** — ~4 Min.
  Wie Partitions-Überschreiben konkret aussieht, inkl. der Unterscheidung **statische vs. dynamische Partitionen** — *„The 'dynamic' approach is simplest (and the default), but the 'static' approach will reduce costs by eliminating multiple queries."* Hier treffen sich Teil A–C und Teil D dieses Blocks.
  <https://docs.getdbt.com/reference/resource-configs/bigquery-configs>

- [x] **14. dbt Docs, „About microbatch incremental models"** — ~15 Min.
  **Der beste Text im ganzen Kapitel zum Warehouse-Aufsatz auf Idempotenz** — und der direkte Anschluss an 2.1: *„It does not matter whether the table already contains data for that day. Given the same input data, the resulting table is the same no matter how many times a batch is reprocessed."* Jeder Batch ist eine atomare, unabhängig ersetzbare Einheit; Backfill über `--event-time-start/--event-time-end`; `dbt retry` verarbeitet nur die fehlgeschlagenen Batches nach.
  → *Leitfrage:* Vergleiche das mit „Task = Transaktion" und „Die Partition als atomare Einheit" aus [[2.1 Advanced SQL & ETL - Vertiefung#5. Idempotenz|2.1]] — was ist hier dasselbe, was ist neu?
  <https://docs.getdbt.com/docs/build/incremental-microbatch>

- [x] **15. PostgreSQL-Doku, `MERGE` — inkl. des „Notes"-Absatzes zu `ON CONFLICT`** — ~16 Min.
  Du kennst `INSERT ... ON CONFLICT` aus [[SQL-Werkzeugkasten]]. Hier steht das genaue Verhältnis der beiden, und es ist präziser, als meist behauptet wird:
  - `ON CONFLICT DO UPDATE` garantiert ein atomares Insert-oder-Update **auch unter hoher Nebenläufigkeit** — diese Garantie gibt `MERGE` *nicht*.
  - `MERGE` ist dafür standardkonform und kann `INSERT`, `UPDATE` und `DELETE` in **einer** Anweisung mischen.
  *Neu seit PG 17: `WHEN NOT MATCHED BY SOURCE` und `RETURNING` mit `merge_action()`. Seit PG 18: `OLD`/`NEW` in `RETURNING`.*
  → *Leitfrage:* Welches der beiden würdest du für einen SCD-Typ-2-Lauf nehmen, und warum?
  <https://www.postgresql.org/docs/current/sql-merge.html>

- [x] **16. Databricks, „Selectively overwrite data with Delta Lake"** — ~14 Min.
  `REPLACE WHERE` als atomares partielles Überschreiben — das Warehouse-Äquivalent zu „schreibe die Partition neu statt Zeilen zu ändern".
  ⚠️ **Aktualitätshinweis:** `replaceWhere` und `partitionOverwriteMode` gelten inzwischen als *legacy*; beide Hersteller empfehlen **`REPLACE USING` / `REPLACE ON`**. Wenn dir ältere Tutorials `replaceWhere` als *das* Delta-Muster verkaufen, sind sie einen Schritt hinterher.
  <https://docs.databricks.com/aws/en/delta/selective-overwrite>

### Teil E — Der Musterkatalog (~72 Min.)

- [x] **17. DEDP, Kap. 4 „Idempotency Design Patterns" — S. 79–114** — ~72 Min.
  Weil du die Grundlagen schon hast, kannst du dieses Kapitel als **Musterkatalog** lesen statt als Einführung — genau die Ebene, die hier gebraucht wird. Acht benannte Patterns in vier Gruppen:
  - *Overwriting*: **Fast Metadata Cleaner** (S. 80) · **Data Overwrite** (S. 86) ← das ist `INSERT OVERWRITE` als Pattern
  - *Updates*: **Merger** (S. 89) · **Stateful Merger** (S. 94) ← das ist `MERGE` als Pattern, mit echtem `MERGE INTO`-Beispiel
  - *Database*: **Keyed Idempotency** (S. 100) · **Transactional Writer** (S. 105)
  - *Immutable Dataset*: **Proxy** (S. 110)
  Jedes Pattern mit *Use case* und *Gotchas*; die Zusammenfassungstabelle steht im Anhang (S. 334).
  → *Leitfrage:* Ordne die Verfahren aus den Quellen 12–16 den Patterns zu. Welches Pattern hat **kein** Gegenstück in dem, was du bisher gelesen hast?

## Vertiefung (optional)

- [ ] **DDIA2, Kap. 7 „Sharding" — nur „Sharding by Key Range", „Sharding by Hash of Key", „Skewed Workloads and Relieving Hot Spots"** — ~25 Min. · *andere Perspektive*
  **Bewusst mit dieser Rahmung lesen: gleiches Wort, anderes Problem.** Sharding verteilt Daten über Knoten, um Schreib- und Punktlesezugriffe zu skalieren; Warehouse-Partitionierung reduziert das **Scan-Volumen**. Rebalancing und Request Routing — der Großteil des Kapitels — sind im Warehouse keine Nutzerprobleme, weil die Engine das löst.
  Echt übertragbar sind genau diese drei Abschnitte: Key-Range ↔ `RANGE`-Partitionierung, Hash ↔ `HASH`-Partitionierung/Bucketing, und **Skew**, das dir in Quelle 9 schon begegnet ist.
  *Ohne diese Rahmung ist das Kapitel aktiv verwirrend — deshalb steht es hier und nicht im Core.*

- [ ] **DEDP, Kap. 8 „Data Storage Design Patterns" — Abschnitt „Partitioning", S. 234–243** — ~20 Min. · *weiterführend*
  *Horizontal Partitioner* (S. 234, inkl. Sharding auf S. 237) und *Vertical Partitioner* (S. 240) als Patterns. Ergänzend im selben Kapitel: *Bucket* (S. 243), *Sorter* (S. 246), *Normalizer* (S. 260) und *Denormalizer* (S. 266) — letztere beide binden zurück an Block 2.

- [ ] **DEDP, Kap. 2 „Data Ingestion Design Patterns" — S. 7–38** — ~62 Min. · *weiterführend*
  *Full Loader* (S. 8) · *Incremental Loader* (S. 12) · *Change Data Capture* (S. 16) · *Compactor* (S. 27) · *Readiness Marker* (S. 30). Der Musterkatalog zu dem, was in 2.1 Stichwort 1 begrifflich behandelt wurde.

- [ ] **Snowflake, „Clustering Keys & Clustered Tables"** — ~25 Min. · *weiterführend*
  Das beste Material zur Kardinalitätsfrage, mit einer beidseitigen Regel: zu niedrige Kardinalität bringt kaum Pruning, zu hohe ist ebenfalls ungeeignet. Ausweg: den Schlüssel als **Ausdruck** auf der Spalte definieren statt auf der Spalte selbst — konzeptionell identisch zu Icebergs `bucket()`/`truncate()`-Transforms.
  <https://docs.snowflake.com/en/user-guide/tables-clustering-keys>

- [ ] **Apache Iceberg, „Partitioning"** — ~4 Min. · *weiterführend*
  Kurz, aber mit einem orthogonalen Punkt: **Partition Evolution** — das Schema kann sich ändern, ohne dass migriert werden muss. Die Entscheidung muss also nicht für die Ewigkeit sein. Dazu die klarste Begründung, warum Hive-Style-Partitionsspalten fehleranfällig sind: *„Using the wrong format, `2018-12-01` instead of `20181201`, produces silently incorrect results, not query failures."*
  <https://iceberg.apache.org/docs/latest/partitioning/>

- [ ] **Raghuwanshi, „How to use table partitioning to scale PostgreSQL" (EDB, 2023)** — ~10 Min. · *andere Perspektive*
  Der systematischste Typ-für-Typ-Durchgang (LIST / RANGE / HASH / mehrstufig) mit einer expliziten „When to use"-Sektion, die den Crunchy-Texten fehlt.
  <https://www.enterprisedb.com/postgres-tutorials/how-use-table-partitioning-scale-postgresql>

- [ ] **Spark-Doku, „INSERT TABLE" + `spark.sql.sources.partitionOverwriteMode`** — ~15 Min. · *weiterführend*
  Die präziseste Ein-Satz-Definition von dynamic partition overwrite: *„In static mode, Spark deletes all the partitions that match the partition specification … before overwriting. In dynamic mode, Spark doesn't delete partitions ahead, and only overwrite those partitions that have data written into it at runtime."*
  ⚠️ Die vielzitierte URL `sql-ref-syntax-dml-insert-overwrite-table.html` ist **tot (404)**. Korrekt: <https://spark.apache.org/docs/latest/sql-ref-syntax-dml-insert-table.html>

---

## Quellen, die ich geprüft und verworfen habe

Damit du nicht dieselben Sackgassen abläufst — und weil ein „das habe ich geprüft, es taugt nicht" genauso Teil der Kuration ist wie eine Empfehlung.

| Quelle | Warum nicht |
|---|---|
| **Petrov, *Database Internals*** — als Partitionierungsquelle | Partitionierung ist dort ein **2-Seiten-Abschnitt** (Kap. 13, S. 270–272) über Consistent Hashing in verteilten Stores. Kategorienfehler für ein Warehouse-Kapitel. Die 5 Seiten zur Row-vs-Column-Abgrenzung stehen als Block-1-Vertiefung drin, mehr gibt es nicht. |
| **FDE** — als Idempotenz-Quelle | Das Wort „idempotent" kommt im ganzen Buch **dreimal vor, alle im selben Absatz** (Kap. 5, Message Queues). Nicht zitierfähig für dieses Thema. Für Upsert/Merge dagegen brauchbar (Kap. 8, „Update patterns"). |
| **Densmore, *Data Pipelines Pocket Reference*** | Richtiges Thema (Kap. 6 hat einen SCD-Abschnitt), aber ~250 Inhaltsseiten auf zehn Themen. Ein Typ-2-Beispiel ohne Typologie. Kimball und DEDP decken es besser ab. |
| **„Data Quality Fundamentals"** (liegt in `Quellen/`) — für Master Data | Volltextsuche: „master data" 0 Treffer, „MDM" 0, „entity resolution" 0, „golden record" 0, „reference data" 0. Das Buch behandelt Observability, nicht Entity Mastering. |
| **Wikipedia „Master data management"** | Zwei Wartungsbausteine („needs more citations", „written like a personal essay"); die Abschnitte zu Implementierungsmodellen sind explizit als quellenlos markiert. Die Artikel „Master data" und „Reference data" sind dagegen gut. |
| **datavaultalliance.com, „Data Vault 2.0 – An Introduction"** | Ist **nicht** von Dan Linstedt (sondern Mark Budzinski) und erklärt Hub/Link/Satellite in genau einem Absatz. Der Rest ist Evangelisierung mit Sales-CTA. |
| **Databricks-Blog „What is Medallion Architecture?"** | Überschneidet sich fast vollständig mit der Doku, die im Core steht. |
| **`learnanalyticsengineering.substack.com`, „Why Medallion Architecture is not…"** | Paywall, nur 143 Wörter Vorschau. |
| **C-Store-Paper (Stonebraker 2005)** | Historisch das Gründungsdokument, als Lernquelle aber deutlich schwächer als der Abadi-Survey — es beschreibt ein System von 2005, keinen Konzeptüberblick. Falls doch: nur Abschnitte 1, 2 und 3.1. |

## Was nicht verifiziert werden konnte

- **DDIA2**: Abschnittstitel und Kapitelzuordnung sind über Kleppmanns eigenes Referenz-Repo und archivierte O'Reilly-TOCs belegt. **Seitenzahlen einzelner Abschnitte nicht** — es gibt keinen öffentlichen Index. Alle Zeitangaben zu DDIA2 sind Schätzungen.
- **Kimball**: Seitenzahlen stammen aus dem gedruckten Inhaltsverzeichnis des Buchs selbst. Im O'Reilly-Web-Reader (EPUB) werden sie möglicherweise nicht angezeigt — dann über die Abschnittsüberschriften navigieren.
- **„Delta Lake: The Definitive Guide"**: Kapitelstruktur belegt, **Seitenzahlen nicht**. Die Zeitangabe ist geschätzt.
- **Linstedt Kap. 4**: Kapitelanfang (S. 89) und Kapiteltitel sind belegt; die genaue Seitenlage von Hub/Link/Satellite **innerhalb** des Kapitels nicht.
- **DMBOK2 Revised Edition (2024)**: wird unter derselben ISBN wie die 2017er-Ausgabe verkauft. Ob die Kapitelnummerierung identisch blieb, ist **nicht** verifiziert.
- **O'Reilly selbst war für alle automatisierten Abrufe gesperrt** (HTTP 403). Sämtliche O'Reilly-Angaben stammen aus Buch-PDFs, Verlagsseiten und Bibliothekskatalogen. Du hast Abo-Zugang und kannst jede Angabe in Sekunden gegenprüfen — wenn dir etwas nicht aufgeht, ist die Kapitelangabe der erste Verdächtige.
