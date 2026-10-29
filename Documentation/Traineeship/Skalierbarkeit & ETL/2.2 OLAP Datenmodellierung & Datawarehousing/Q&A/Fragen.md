---
tags:
  - documentation
---

# Offene Fragen — 2.2 OLAP Datenmodellierung & Datawarehousing

Fragen, die beim Lesen oder Umsetzen aufkommen. Einfach notieren, Claudian beantwortet sie gesammelt am Ende jedes Blocks.

Gehört zu [[2.2 OLAP Datenmodellierung & Datawarehousing - Leitfaden]] · [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste]] · [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung]]

> [!note] Eine Ausnahme
> Fragen, die schon in den Kontrollfragen der Vertiefung stehen, beantworte ich **vorher nicht** — sonst ist der Selbsttest wertlos. Ich weise dann nur darauf hin, dass du sie erst selbst versuchen sollst.

## Block 1 — OLAP vs. OLTP & Warehouse vs. Postgres

### ✅ Pinot, Druid, ClickHouse — „product analytics or real-time analytics"?

Beantwortet in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.11 Cloud-Warehouses: Storage, Compute und MPP|Abschnitt 1.11]].

Kurzfassung: eine **dritte Kategorie zwischen OLTP und klassischem Warehouse**. Spaltenorientiert wie ein Warehouse, aber auf **niedrige Latenz bei hoher Query-Rate** ausgelegt statt auf beliebige Ad-hoc-Exploration. Typischer Einsatz: ein Dashboard, das vielen Nutzern gleichzeitig frische Kennzahlen zeigt — deshalb „product analytics". Sie nehmen Daten oft direkt aus einem **Event-Stream** entgegen statt aus einem Batch-Load.

Für den Leitfaden relevant, weil sie ein konkreter Beleg dafür sind, dass OLAP und OLTP Idealtypen mit einer durchgehenden Skala dazwischen sind — genau das, wonach Kontrollfrage 3 gefragt hat.

### ✅ Object Storage — „adjust storage capacity and compute resources independently"

Beantwortet in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.11 Cloud-Warehouses: Storage, Compute und MPP|Abschnitt 1.11]], inklusive Vergleichstabelle.

Kurzfassung: In der klassischen shared-nothing-Architektur besitzt jeder Knoten seine eigenen Platten. Mehr Rechenleistung heißt dann **Knoten hinzufügen und Daten umverteilen** — ein Vorgang, der Stunden bis Tage dauert. Liegen die Daten dagegen im Object Store, holt sich jeder Rechenknoten, was er braucht: Compute skaliert in Sekunden, Storage ist ohnehin elastisch, und im Leerlauf lässt sich Compute komplett abschalten.

Der praktisch wichtigste Effekt ist der vierte: **mehrere getrennte Cluster auf denselben Daten**. ETL-Team und Analysten bremsen sich nicht mehr gegenseitig aus, und beide werden getrennt abgerechnet.

### ✅ MPP — „a company can have a data warehouse without an MPP system"

Beantwortet in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.11 Cloud-Warehouses: Storage, Compute und MPP|Abschnitt 1.11]].

Die Stelle sagt genau das, wonach du fragst: **MPP und „Data Warehouse" sind zwei verschiedene Begriffe**, die historisch zusammengewachsen sind. MPP ist die *technische* Architektur (shared-nothing, eine Query über viele Knoten verteilt, Teilergebnisse, Shuffle). „Data Warehouse" ist die *organisatorische* Funktion (integriertes, historisiertes Abbild mehrerer Quellsysteme).

Beides ist einzeln möglich: ein Warehouse auf einer einzelnen Maschine ohne MPP, oder ein MPP-Cluster, der reine Log-Verarbeitung macht und kein Warehouse ist. Dass beide meist zusammen auftreten, ist ein Ergebnis der Geschichte, keine Definition.

### ✅ Serverless computation platforms?

Der Begriff wird in zwei verschiedenen Bedeutungen benutzt, und die Verwechslung ist häufig.

**1. Serverless im Warehouse-Sinn.** Du provisionierst **keinen Cluster**. Du schickst eine Query, der Anbieter weist ihr im Hintergrund Rechenressourcen zu und rechnet **pro Query** ab — meist nach gescannten Bytes. Der Archetyp ist **BigQuery**: Es gibt keinen Knopf für „Clustergröße", und genau deshalb ist dort Partitionierung so zentral, dass die Doku sie primär als *Kosten*instrument beschreibt (kommt in Block 6). Weniger gescannte Bytes heißt unmittelbar weniger Rechnung.

Der Gegenentwurf ist **Snowflake**: elastisch, aber nicht serverless — du definierst *virtual warehouses* in T-Shirt-Größen, startest und stoppst sie. Der Unterschied ist nicht die Elastizität, sondern **wer die Kapazitätsentscheidung trifft**.

**2. Serverless im allgemeinen Sinn.** Funktionsplattformen wie AWS Lambda, Google Cloud Functions oder Cloud Run: Code läuft auf Abruf, skaliert auf null, Abrechnung nach Laufzeit. Im Data Engineering typischerweise für Ingestion, Orchestrierungs-Trigger und leichte Transformationen — nicht für die schweren Aggregationen.

Gemeinsam ist beiden: **kein Kapazitätsmanagement, Abrechnung nach Verbrauch, Skalierung auf null**. Der Preis ist in beiden Fällen weniger Kontrolle über Latenz und Kosten pro Einheit — und bei Pay-per-Scan die unangenehme Eigenschaft, dass eine unbedachte Query teuer ist, statt nur langsam zu sein.

### ⏭️ Data Vault — Zusammenhang zu Data Warehouse und Data Lake?

**Vertagt auf Block 4** ([[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste#Block 4 — Data Vault, OBT & Kimball vs. Inmon|Leseliste]]), wo Data Vault das Hauptthema ist. Die Kurzantwort als Orientierung, damit die Frage nicht bis dahin offen im Raum steht:

Data Vault ist eine **Modellierungsmethode**, kein Systemtyp — es steht also nicht *neben* Warehouse und Data Lake, sondern *in* ihnen. Konkret ist es meist die Modellierung der **Integrationsschicht** eines Warehouses: Hubs (Geschäftsschlüssel), Links (Beziehungen) und Satellites (beschreibende Attribute mit Historie). Aus dieser Schicht werden dann dimensionale Marts abgeleitet.

Genau deshalb heißt eine der Quellen in Block 5 „Data Vault in Silver, Dimensional Modeling in Gold" — die beiden Methoden konkurrieren weniger, als es zunächst aussieht.

### ⏭️ „Warehouse vs Postgres"

Das ist das Stichwort selbst, nicht eine Einzelfrage — beantwortet über [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.13 Postgres als Analyse-Store: die konkreten Grenzen|Abschnitt 1.13]] (die vier strukturellen Grenzen, die Erweiterungen, die Größenordnungen und die Gegenrichtung) sowie über die Abschnitte [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.5 Zeilen- vs. spaltenorientierte Speicherung|1.5]] bis [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.9 Das Schreibproblem und wie Column Stores es lösen|1.9]].

Der zweite Teil folgt in **Block 6**, wo derselbe Kontrast praktisch wird: Partitionierung von Hand in Postgres gegen automatische Micro-Partitions in Snowflake, und `MERGE`/`ON CONFLICT` gegen `insert_overwrite`.

## Block 2 — Kimball & Normalisierung vs. Denormalisierung

> [!note] Keine offenen Fragen notiert
> Beim Lesen von Block 2 sind keine Fragen hier gelandet. Eine Beobachtung aus deinen Exzerpten habe ich stattdessen direkt eingearbeitet: Dein Vergleich der Frage „was heißt eigentlich Kunde?" mit **Feature Engineering** trifft zu — in beiden Fällen legt eine Definitionsentscheidung fest, was später überhaupt messbar ist. Steht in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#2.8 Der Vier-Schritte-Prozess|Abschnitt 2.8]].

## Block 3 — Slowly Changing Dimensions

> [!note] Keine offenen Fragen notiert
> Beim Lesen von Block 3 sind keine Fragen hier gelandet. Die inhaltlich offene Stelle war der Widerspruch zwischen Kontrollfrage 10a und 10b — beantwortet in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.21 Ist eine SCD2-Tabelle System of Record oder Derived Data?|Abschnitt 3.21]].

## Block 4 — Data Vault, OBT & Kimball vs. Inmon

### ✅ Was genau ist mit „Data Integration" gemeint?

Es ist der Vorgang, Daten aus **mehreren unabhängig entworfenen Quellsystemen in ein kohärentes Modell** zu überführen. Inmon nennt es „the most important aspect" — zu Recht, denn es ist das, was ein Warehouse von einer Read Replica unterscheidet ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.4 Warum eine Read Replica nicht reicht|Abschnitt 1.4]]).

Konkret zerfällt Integration in vier Teilprobleme, und jedes ist für sich unangenehm:

| Ebene | Problem | Beispiel |
|---|---|---|
| **Schema** | dieselbe Sache, andere Struktur | CRM hat `cust_name`, Shop hat `first_name` + `last_name` |
| **Schlüssel** | dieselbe Nummer, verschiedene Objekte | „Kunde 123" ist im CRM und im Shop ein anderer Mensch |
| **Werte** | dieselbe Bedeutung, andere Kodierung | `DE` vs. `Deutschland` vs. `Germany` |
| **Semantik** | dasselbe Wort, andere Definition | „Umsatz" ist netto im einen, brutto im anderen System |

Die ersten drei sind technisch lösbar. Die vierte ist **organisatorisch** — jemand muss entscheiden, wessen Definition gilt, und das ist keine Datenbankfrage.

Der eigentliche Grund, warum sich das durch das ganze Kapitel zieht: **Alle drei Modellierungsansätze sind Antworten auf genau dieses Problem.**

- **Inmon** löst es **strukturell**: eine einzige normalisierte Wahrheit, in die alles integriert wird.
- **Kimball** löst es **konventionell**: conformed dimensions, einmal definiert und wiederverwendet.
- **Data Vault** löst es **durch Vertagung**: Der Raw Vault speichert alle Varianten mit ihrer Herkunft, entschieden wird erst im Business Vault.

Und **Master Data Management** (Block 5) ist die vierte Antwort, diesmal als eigenes System.

### ✅ Warum ist das Kimball-Modell nur für Batch geeignet und nicht für Streaming?

Gute Frage, denn FDE stellt die Behauptung auf, ohne sie zu begründen. Es sind drei Gründe, und alle hängen an derselben Stelle:

**1. Die Surrogate Key Pipeline braucht einen Nachschlagevorgang.** Beim Laden eines Fakts muss der natürliche Schlüssel gegen die Dimension aufgelöst werden ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.14 Die Surrogate Key Pipeline|Abschnitt 3.14]]) — und zwar gegen die **zum Ereigniszeitpunkt gültige Version**. Das setzt voraus, dass die Dimension zu diesem Zeitpunkt **fertig geladen** ist. Im Batch ist das eine Reihenfolge: erst Dimensionen, dann Fakten. Im Stream gibt es diesen Punkt nicht — der Input ist unbegrenzt, es ist nie „fertig".

**2. Verspätete Dimensionen werden zum Normalfall.** Im Batch ist eine late-arriving dimension ein Sonderfall ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.16 Late-arriving Facts und late-arriving Dimensions|Abschnitt 3.16]]). Im Stream trifft der Fakt regelmäßig **vor** der Dimensionsänderung ein. Man müsste also ständig rückwirkend Faktenzeilen umhängen — genau die Operation, die als teuer und vermeidenswert gilt.

**3. Die Standardkorrektur ist eine Batch-Operation.** „Partition löschen und neu laden" ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.18 Korrekturen in Faktentabellen|Abschnitt 3.18]]) setzt abgegrenzte, wiederholbar verarbeitbare Einheiten voraus. Ein Stream hat die nicht.

**Was man stattdessen macht** — und hier schließt sich der Kreis zu Block 4: In Streaming-Pipelines **denormalisiert man früh**. Das Ereignis wird beim Durchfließen mit den Dimensionsattributen **angereichert**, die gerade gelten, und fließt dann als breites, in sich vollständiges Event weiter. Das ist faktisch **OBT** — und genau deshalb sind ereigniszentrierte Modelle wie das [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#4.27 Activity Schema: der vierte Ansatz|Activity Schema]] im Streaming-Umfeld verbreiteter als das Star Schema.

Die Einschränkung ist also keine prinzipielle Unmöglichkeit, sondern: **Der klassische Kimball-Ladeprozess setzt Batch voraus.** Das dimensionale *Modell* kann man streamen — den *Prozess*, mit dem man es befüllt, nicht.

### ✅ Wie unterscheidet sich ein Business Key von einem Primary Key?

Sie liegen auf verschiedenen Ebenen, deshalb schließen sie sich nicht aus — ein PK **kann** ein Business Key sein.

| | Primary Key | Business Key |
|---|---|---|
| Was er garantiert | technische Eindeutigkeit **in einer Tabelle** | Identität eines **Objekts in der Welt** |
| Gültigkeitsbereich | ein System | idealerweise das **ganze Unternehmen** |
| Wer ihn kennt | die Datenbank | **Menschen** — er steht auf der Rechnung |
| Stabilität | kann bei Migration neu vergeben werden | soll **dauerhaft** sein |
| Kann bedeutungslos sein | ja (Auto-Increment) | nein, er ist per Definition bedeutungstragend |

Der entscheidende Punkt für Data Vault: **Nur der Business Key taugt als Integrationsanker.** Zwei Systeme haben verschiedene technische Primärschlüssel für denselben Kunden — aber beide kennen die Kundennummer, weil der Mensch danach fragt. Deshalb Linstedts Faustregel: *wonach der Fachanwender sucht, ist der Business Key.*

Das Gegenstück bei Kimball ist der **durable natural key**, der die SCD2-Versionierung überlebt ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#2.10 Surrogate Keys|Abschnitt 2.10]]).

Ehrliche Einschränkung: In der Praxis sind Business Keys selten so sauber, wie die Theorie hätte. Sie werden wiederverwendet, sie ändern sich, manchmal existiert keiner. Dann behilft man sich mit quellqualifizierten Schlüsseln oder einem *business key collision code*.

### ✅ Hat ein Satellite einen PK, der gleichzeitig FK ist? Ist er 1:1?

Fast — der erste Teil stimmt, der zweite nicht.

Der Primärschlüssel eines Satellites ist **zusammengesetzt**:

```
PRIMARY KEY (customer_hk, load_date)
```

- `customer_hk` ist tatsächlich zugleich **Fremdschlüssel** auf den Hub (oder Link).
- `load_date` ist der zweite Teil — und **genau der macht die Beziehung zu 1:n**.

Ein Hub hat also **viele** Satellite-Zeilen: eine pro Version über die Zeit. Das ist die Historisierung. Nur wenn du auf die aktuelle Version filterst (`MAX(load_date)` je Hash Key), ist es **an einem Zeitpunkt** effektiv 1:1.

Dazu kommt die zweite Dimension: **Ein Hub kann mehrere Satellite-Tabellen haben** — getrennt nach Quellsystem, Änderungsrate oder Datentyp. Die Beziehung ist also in zwei Richtungen 1:n.

### ✅ Data Vault für NoSQL und Streaming — wie soll das gehen?

FDE erwähnt es im Vorbeigehen. Der Mechanismus dahinter ist aber konkret und hängt an **einer** Eigenschaft: dem **Hash Key**.

Weil der Hash **deterministisch aus dem Business Key** berechnet wird, kann jeder Verarbeiter ihn **selbst ausrechnen, ohne nachzuschlagen** ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#4.10 Die Metadatenspalten — und warum sie paralleles Laden ermöglichen|Abschnitt 4.10]]). Daraus folgt beides:

- **Streaming:** Ein eingehendes Ereignis lässt sich verarbeiten, **ohne gegen einen Zustand zu joinen**. Genau der Nachschlagevorgang, der Kimball an Batch bindet (siehe Frage oben), entfällt. Dazu sind Satellites **insert-only** — kein Update-in-place, was zu append-only-Senken passt.
- **NoSQL / verteilt:** In Data Vault 1.0 waren die Schlüssel noch **Sequenznummern** — die brauchen einen zentralen Generator und sind in einem verteilten System ein Engpass. Data Vault 2.0 hat sie **genau deshalb** durch Hashes ersetzt. Jeder Knoten rechnet unabhängig.

> [!warning] Mit Vorbehalt
> Das Prinzip — Hash Keys beseitigen die Nachschlage-Abhängigkeit — ist solide und gut belegt. Wie verbreitet echte Data Vaults auf NoSQL-Stores oder in Streaming-Pipelines tatsächlich **sind**, kann ich nicht belastbar sagen; die Literatur dazu ist überwiegend methodennahes Marketing. Behandle es als „strukturell möglich", nicht als „gängige Praxis".

### ✅ Das Beispiel mit den „multivalued dependencies" — und die Unit of Work

Das ist die beste Frage des Blocks, weil die Antwort etwas ist, das du **schon kennst**, nur unter anderem Namen.

**Was Linstedt beschreibt:** Ein Geschäftsvorgang verbindet drei Dinge gleichzeitig — sagen wir, ein Verkauf verbindet *Kunde*, *Produkt* und *Filiale*. Modelliert man das als **einen** Link mit drei Hub-Verweisen, bleibt erhalten, dass diese drei **zusammen** aufgetreten sind.

Zerlegt man denselben Vorgang dagegen in **zwei** Links — `(Kunde, Produkt)` und `(Kunde, Filiale)` —, ist die Verbindung zwischen Produkt und Filiale verloren. Beim Wiederzusammenfügen entstehen Kombinationen, **die es nie gab**:

```
Realität:   Alice kauft Maus in Filiale Berlin
            Alice kauft Stuhl in Filiale München

Zerlegt:    (Alice, Maus) · (Alice, Stuhl)
            (Alice, Berlin) · (Alice, München)

Rejoin:     Alice–Maus–Berlin   ✓ echt
            Alice–Maus–München  ✗ erfunden
            Alice–Stuhl–Berlin  ✗ erfunden
            Alice–Stuhl–München ✓ echt
```

Aus zwei echten Vorgängen werden vier, zwei davon **frei erfunden**. Linstedts Formulierung: *„a record has been created that doesn't exist in the original source system"* — weil die **Unit of Work** beim Zerlegen zerbrochen wurde.

> [!tip] Das ist der Chasm-Trap — und du kennst ihn bereits
> Genau dieses Phänomen steht in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#4.21 Fan-Trap und Chasm-Trap|Abschnitt 4.21]]: zwei unabhängige 1:n-Beziehungen vom selben Elternknoten, gejoint → **Kreuzprodukt** statt Realität.
> Drei Fachgebiete, drei Namen für dieselbe Sache:
> - **Data Vault:** „breaking the unit of work"
> - **BI/Dimensional:** **Chasm-Trap**
> - **Relationale Theorie:** Verletzung der **4. Normalform** durch eine mehrwertige Abhängigkeit
>
> Und derselbe Mechanismus liegt der Warnung vor Bridge Tables ohne Gewichtungsfaktor zugrunde ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#2.13 Bridge Tables für mehrwertige Dimensionen|Abschnitt 2.13]]).

**Die Regel, die daraus folgt:** Ein Link muss die **vollständige Unit of Work** abbilden — alle Hubs, die an **einem** atomaren Geschäftsvorgang beteiligt waren, gehören in **denselben** Link. Man zerlegt ihn nicht, auch wenn es „normalisierter" aussieht.

Das ist im Data Vault dieselbe Rolle, die der **Grain** bei Kimball spielt ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#2.9 Der Grain — der wichtigste Schritt|Abschnitt 2.9]]): eine Festlegung darauf, was **eine Zeile bedeutet** — und ein Schutz davor, Dinge zu verbinden, die nicht zusammengehören.

## Block 5 — Schichtenarchitektur, Medallion & Master Data

### ✅ „Configure most reads from bronze as streaming reads" — wie ist das gemeint?

Der Begriff **streaming read** ist hier irreführend: Er bedeutet **nicht** „in Echtzeit", sondern **inkrementell mit Fortschrittsmerker**.

Ein Streaming-Reader auf einer Delta-Tabelle führt einen **Checkpoint** mit, der festhält, bis wohin er verarbeitet hat. Beim nächsten Lauf liest er **nur die seitdem hinzugekommenen Dateien**. Das funktioniert, weil Bronze **append-only** ist — es gibt eine wohldefinierte Vorstellung von „neu seit zuletzt".

Und der entscheidende Punkt: So ein Job kann trotzdem **wie ein Batch-Job laufen**. Man startet ihn stündlich oder täglich im Modus „verarbeite alles Verfügbare und beende dich". Streaming-**Semantik** bei Batch-**Zeitplan**.

Der Grund für die Empfehlung: Würde Silber jedes Mal die **ganze** Bronze-Tabelle lesen, wäre das verschwenderisch und würde unbegrenzt teurer, weil Bronze nur wächst. Der Checkpoint-Mechanismus liefert das „was ist neu?" **vom Framework**, statt dass du selbst eine High-Watermark-Spalte pflegst — genau das Muster, das du in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.11 Änderungserkennung im Snapshot-Verfahren|Abschnitt 3.11]] von Hand gebaut hast.

Die Ausnahme („batch reads reserved for small datasets, for example small dimensional tables") ist konsequent: Bei einer Dimensionstabelle mit 5.000 Zeilen ist ein Vollabzug billiger als die Verwaltung von Checkpoint-Zustand.

**Und der erste Satz des Zitats ist der wichtigere:** *„Databricks does not recommend writing to silver tables directly from ingestion."* Das ist die eigentliche Begründung dafür, **warum es Bronze überhaupt gibt**. Silber hat ein striktes Schema und Qualitätsregeln. Schriebe die Ingestion direkt dorthin, würde **jede** Schemaänderung und **jeder** korrupte Satz in der Quelle den Ladejob zum Absturz bringen.

Bronze ist der **Stoßdämpfer**: Dort wird Schema Drift toleriert, die Aufnahme gelingt immer, und der Fehler taucht kontrolliert am Übergang Bronze → Silber auf — wo man schlechte Sätze in Quarantäne stellen kann, statt die ganze Aufnahme scheitern zu lassen.

Bemerkenswert daran: Das ist das **Gegenargument** zu Bellemares Flaw #4 („Bronze ist fragil"). Bronze ist fragil, **weil es die Fragilität absorbieren soll** — die Frage ist nur, ob man das Problem damit löst oder bloß verschiebt.

### ✅ Wo ist das System of Record — und wie lange werden Daten aufbewahrt?

Die beste Frage des Blocks, weil sie das ganze Kapitel zusammenzieht. Und die Antwort hat eine unangenehme Pointe.

**Wo das System of Record liegt — es hängt an einer einzigen Entscheidung:**

| Schicht | Rolle | Bedingung |
|---|---|---|
| **Bronze** | **System of Record** für die Rohhistorie | **wenn** persistent und append-only |
| **Silber** | **Derived Data** | **nur wenn** Bronze persistent ist |
| **Gold** | **immer Derived Data** | — |
| **MDM-Hub** | **System of Record** für die gemasterten Entitäten | — |

Der mittlere Punkt ist der Haken, und er ist [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#3.21 Ist eine SCD2-Tabelle System of Record oder Derived Data?|Abschnitt 3.21]] in neuer Kleidung:

> [!danger] Silber ist nur so lange Derived Data, wie Bronze zurückreicht
> Eine SCD2-Dimension in Silber enthält Historie, die das **Quellsystem überschrieben** hat. Solange die Roh-Snapshots in Bronze liegen, lässt sie sich daraus **neu berechnen** — sie ist ableitbar.
> Kürzt jemand die Bronze-Aufbewahrung auf 90 Tage (aus Kostengründen, völlig nachvollziehbar), dann ist alles, was älter ist, **nicht mehr rekonstruierbar**. In diesem Moment wird Silber **stillschweigend zum System of Record** — mit Backup-Pflicht und ohne Drop-and-Reload.
> **Das merkt niemand, bis man es braucht.** Es ist die folgenreichste Retention-Entscheidung der ganzen Architektur, und sie wird meist von jemandem getroffen, der die Konsequenz nicht kennt.

**Wie lange aufbewahrt wird — die Faustregel:**

> **Die Aufbewahrungsdauer nimmt nach unten ab, weil die Wiederherstellbarkeit nach unten zunimmt.**

- **Bronze** — am längsten, oft Jahre bis unbegrenzt. Billiger Object Storage plus Lifecycle-Regeln in kalte Speicherklassen. Die Grenze ist **rechtlich** (Löschpflichten), nicht technisch.
- **Silber** — hält **Historie** (SCD2, Satellites), aber nicht jeden Roh-Snapshot. Typischerweise so lange, wie fachlich historisch ausgewertet wird. Wichtig: Silber ist **nicht** append-only im selben Sinn — SCD2 aktualisiert `valid_to` auf bestehenden Zeilen.
- **Gold** — am kürzesten. Vollständig ableitbar, also behält man, was Abfragen brauchen. Manche Gold-Tabellen werden bei **jedem Lauf komplett neu** gebaut.

### ✅ Bellemares Flaw #1 — „ich verstehe das nicht ganz"

Deine Rückfrage ist produktiv, weil du genau das **Idealbild** beschreibst, das die Architektur verspricht — und Bellemare beschreibt, was tatsächlich passiert.

Dein Modell: *„Jemand macht alle Daten fertig, und wenn ein Konsument etwas braucht, ist es meistens schon da."*

Das stimmt für **bekannte** Anwendungsfälle. Bellemares Punkt betrifft die **neuen**, und er hat zwei Teile:

**Teil 1 — „predicated on a pull system".** Es gibt niemanden, der die Daten **für deine Frage** vorbereitet hat. In Bronze liegen sie **quellförmig**. Kommt eine neue Frage auf, muss **der Konsument** die Strecke bauen: extrahieren, umformen, bereinigen, modellieren — bevor die eigentliche Arbeit anfängt. Ein zentrales Team kann unmöglich jede künftige Sicht vorbauen.

Die Zahl der vorbereiteten Datenprodukte wächst also mit der Zahl der Fragen, und jede neue Frage ist ein Projekt.

**Teil 2 — „tight and brittle coupling", und das ist der eigentliche Vorwurf.** Die Strecke, die aus Bronze liest, muss das **Quellschema kennen**. Ein Beispiel, das es konkret macht:

> Das Shop-Team benennt `customer_email` in `email` um und deployt am Dienstag. Es informiert niemanden — es ist ihre Tabelle.
> Am Mittwoch brechen drei Silber-Jobs ab. Oder schlimmer: Sie laufen durch und schreiben `NULL`.
> Das Datenteam debuggt, findet die Ursache und bittet um Rücknahme. Antwort: „Das ist unsere Tabelle, wir brauchten die Änderung."

Und genau darin liegt die Asymmetrie, die Bellemare benennt: *„the consumer bears **all the responsibility** … without having **ownership or even influence** of the source model."* Der Konsument trägt das Risiko für eine Entscheidung, die ein anderer trifft und über die er nicht mitreden kann.

**Deshalb gibt es Data Contracts** ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#5.15 Data Contracts|Abschnitt 5.15]]). Sie verwandeln „der Konsument entdeckt den Bruch drei Wochen später" in „die CI des Produzenten schlägt fehl, bevor deployt wird". Die Verantwortung wandert dorthin zurück, wo die Entscheidung fällt — das ist die ganze „Shift Left"-Idee.

## Block 6 — Partitionierung & idempotente Ladeverfahren

### ✅ Delta Lakes?

Zuerst eine Begriffsfalle: Es heißt **Delta Lake**, nicht „Delta Lakes" — und es ist **kein** Data Lake, obwohl der Name es nahelegt. Delta Lake ist ein **offenes Tabellenformat**: eine Konvention, wie man Dateien auf Object Storage so ablegt, dass sie sich wie eine Datenbanktabelle verhalten. Ursprünglich von Databricks entwickelt, heute Open Source unter der Linux Foundation.

**Wie es funktioniert:** Die eigentlichen Daten liegen als ganz normale **Parquet-Dateien** (spaltenorientiert, [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.5 Zeilen- vs. spaltenorientierte Speicherung|Abschnitt 1.5]]). Daneben liegt ein Ordner `_delta_log` mit einem **Transaktionslog**: eine fortlaufend nummerierte Folge kleiner JSON-Dateien. Jede Datei beschreibt einen Commit — welche Parquet-Dateien hinzugekommen sind und welche ab jetzt nicht mehr zur Tabelle gehören.

Die Tabelle ist also **nicht** „alle Dateien im Ordner", sondern **„alle Dateien, die laut Log gerade gültig sind"**. Daraus folgt alles Weitere:

| Fähigkeit | Wie das Log sie ermöglicht |
|---|---|
| **ACID-Transaktionen** | Ein Commit ist genau dann sichtbar, wenn seine Log-Datei geschrieben ist — ein Schreibvorgang ist ganz oder gar nicht da |
| **`UPDATE`, `DELETE`, `MERGE`** | Betroffene Dateien werden **neu geschrieben** (*copy-on-write*), das Log ersetzt die alten durch die neuen — Parquet selbst ist unveränderlich |
| **Time Travel** | Alte Dateien bleiben eine Weile liegen; man kann die Tabelle „wie sie bei Version 42 war" lesen |
| **Schema-Durchsetzung** | Das Log enthält das Schema; Schreibvorgänge, die nicht passen, werden abgelehnt |
| **Data Skipping** | Das Log führt **Min/Max-Statistiken pro Datei** — das sind die **Zone Maps** aus [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#6.16 Partition Pruning und Zone Maps — zwei Ebenen derselben Idee|Abschnitt 6.16]] |
| **Liquid Clustering** | Daten werden inkrementell so umsortiert, dass diese Statistiken scharf werden |

**Wo es im Kapitel vorkam:** Delta Lake ist die Grundlage des **Lakehouse** ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#5.17 Data Lake, Warehouse, Lakehouse|Abschnitt 5.17]]): Warehouse-Eigenschaften (Transaktionen, Schema, `MERGE`) zu Data-Lake-Kosten (billige Dateien auf Object Storage). Und es ist das Format, in dem die **Medallion-Architektur** typischerweise umgesetzt wird — Bronze, Silber und Gold sind dort jeweils Delta-Tabellen.

**Die Geschwister:** **Apache Iceberg** (Netflix, heute Apache) und **Apache Hudi** (Uber) lösen dasselbe Problem mit anderer Metadatenstruktur. Iceberg hat mit Hidden Partitioning und Partition Evolution zwei Stärken, die in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#6.14 Iceberg: Hidden Partitioning und Partition Evolution|Abschnitt 6.14]] stehen. Die drei Formate nähern sich seit Jahren an.

**Warum es für dieses Kapitel relevant ist:** Die Kosten von `MERGE` in Delta Lake ([[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#6.30 Kosten: `MERGE` gegen `INSERT OVERWRITE`|Abschnitt 6.30]]) ergeben sich direkt aus dem Mechanismus oben: Weil Parquet-Dateien unveränderlich sind, kostet die Änderung einer einzigen Zeile das Neuschreiben ihrer ganzen Datei.