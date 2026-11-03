---
deck: "Traineeship::Skalierbarkeit & ETL::OLAP Datenmodellierung & Datawarehousing"
---

Karteikarten zu [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung|2.2 OLAP Datenmodellierung & Datawarehousing – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen.

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Skalierbarkeit & ETL::OLAP Datenmodellierung & Datawarehousing`. Die `oid`-Attribute werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr. Innerhalb eines Blocks stehen die Punkte, die Nick beim Lesen selbst exzerpiert hat, **zuerst** — sie haben Vorrang und bekommen im Zweifel eine eigene Karte.

# Fortschritt

- [x] Block 1 — OLAP vs. OLTP & Warehouse vs. Postgres
- [x] Block 2 — Kimball & Normalisierung vs. Denormalisierung
- [x] Block 3 — Slowly Changing Dimensions
- [x] Block 4 — Data Vault, OBT & Kimball vs. Inmon
- [x] Block 5 — Schichtenarchitektur, Medallion & Master Data
- [x] Block 6 — Partitionierung & Idempotenz

---

# Block 1 — OLAP vs. OLTP & Warehouse vs. Postgres

*Die ersten neun Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="ObsUe7haJLZhH2DkVrLcKA9P" -->
Kleppmann ersetzt „OLTP vs. OLAP" durch ein präziseres Begriffspaar. Wie lauten die beiden Systemtypen, und was ist der Kernunterschied?::
**Operative Systeme** und **analytische Systeme**.<br>**Operativ:** Backend-Dienste und Infrastruktur, in denen Daten *entstehen* — der Anwendungscode liest **und verändert** sie.<br>**Analytisch:** enthalten eine **schreibgeschützte Kopie** der Daten aus den operativen Systemen, optimiert für die Verarbeitungsarten der Analytik.<br>Der entscheidende Punkt steckt in „read-only copy": Ein analytisches System ist definitionsgemäß **abgeleitet** — nicht die Wahrheit, sondern eine Verarbeitung der Wahrheit.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3yz4z0Z3yMyA9td3gHRey" -->
Wie lautet Bill Inmons Definition des Data Warehouse von 1990 — die **vier** Eigenschaften?::
„a **subject-oriented**, **integrated**, **nonvolatile**, and **time-variant** collection of data in support of management's decisions."<br>**subject-oriented** — nach Geschäftsthemen organisiert (Kunde, Produkt, Umsatz), nicht nach Anwendungen<br>**integrated** — führt viele Quellsysteme in ein einheitliches Modell zusammen<br>**nonvolatile** — einmal geladen wird nicht überschrieben oder gelöscht<br>**time-variant** — jedes Datum trägt Zeitbezug, Historie bleibt erhalten
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszK1WMi9sZNnFixbPo6cC7" -->
Warum widerlegen Inmons vier Eigenschaften die Aussage „Data Warehouses sind letztlich read-optimierte relationale Datenbanken"?::
Weil **drei von vier** mit Lesegeschwindigkeit nichts zu tun haben.<br>**integrated** ist Integrationsarbeit über mehrere Quellsysteme.<br>**time-variant** und **nonvolatile** sind Aufbewahrungs- und Modellierungsregeln — Historie statt Momentaufnahme.<br>**subject-oriented** ist eine Modellierungsentscheidung.<br>Nur „read-optimiert" wäre auch ein Postgres mit vielen Indizes und Read Replica — und das ist trotzdem kein Warehouse.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs861qMdx6IQ1jZMElEfzPE" -->
Wie unterscheiden sich operative und analytische Systeme im **Lese-** und im **Schreibmuster**?::
**Lesen** — operativ: *Point Query*, einzelne Datensätze per Schlüssel. Analytisch: **Aggregation** über sehr viele Datensätze.<br>**Schreiben** — operativ: einzelne Datensätze anlegen, ändern, löschen. Analytisch: **Bulk-Import** (ETL) oder Event-Stream.<br>Genau daraus folgt die Wahl des Speicherformats: Row Store für Zeilenzugriff, Column Store für Spaltenzugriff.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspReD4zxIhaggVG4gxz8Kk" -->
Operative Systeme haben **feste**, von der Anwendung vorgegebene Queries, analytische **beliebige** Ad-hoc-Queries. Welche architektonische Konsequenz hat das?::
Im OLTP-System kennst du beim Schemaentwurf **alle Zugriffspfade** und kannst gezielt Indizes anlegen.<br>Im Warehouse weiß **niemand vorher**, wonach der Analyst nächste Woche filtert — Indizes für alle denkbaren Kombinationen sind unmöglich.<br>Also braucht man eine Architektur, in der ein **Scan billig** ist, statt einer, die Scans vermeidet. Das ist die eigentliche Wurzel der Spaltenspeicherung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRsr3R26Qdy601mj5flo37" -->
Operative Daten zeigen den *aktuellen Zustand*, analytische die *Historie von Ereignissen*. Welches Folgethema ergibt sich daraus direkt?::
**Slowly Changing Dimensions.**<br>Ein OLTP-System überschreibt: Ändert ein Kunde seine Adresse, ist die alte weg. Ein Warehouse muss beides behalten — sonst könnte man Umsatz nicht nach der Region auswerten, die zum **Kaufzeitpunkt** galt.<br>Dieser eine Punkt ist der ganze Grund, warum es SCDs gibt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs21BaD8CZALa0CxS1xgoU5" -->
Nenne die **drei** unabhängigen Gründe, warum Analytik in ein separates System ausgelagert wird::
**1. Interferenz** — analytische Queries sind teuer und langlaufend, verdrängen den Buffer Cache und halten lange MVCC-Snapshots; ein Analyst kann die Kundenanwendung ausbremsen.<br>**2. Integration** — ein Unternehmen hat viele operative Datenbanken mit eigenen Schemata und Schlüsseln; „wie viel Umsatz macht Kunde X?" lässt sich in keiner einzelnen beantworten. *Der am häufigsten übersehene Grund.*<br>**3. Optimierung** — ein separates System kann völlig anders gebaut werden (spaltenweise, vektorisiert, MPP).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs38xX0ORCiQjv8q1PiJEUY" -->
Was ist der Unterschied zwischen einem **System of Record** und **Derived Data** — und welche Betriebsregel folgt daraus?::
**System of Record** (*source of truth*): hält die maßgebliche Version; neue Daten werden hier zuerst geschrieben; jedes Faktum genau einmal, typischerweise normalisiert.<br>**Derived Data**: entsteht durch Transformation aus einem System of Record; bei Verlust **neu erzeugbar**; typischerweise denormalisiert und leseoptimiert.<br>**Regel:** Bei Zweifeln neu berechnen, **niemals von Hand korrigieren**. Ein Warehouse ist Derived Data — deshalb muss die Pipeline idempotent sein.<br>Caches, Suchindizes und Feature Stores fallen unter dieselbe Regel.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfMQ6Kr2TivYIIxoeMOSIf" -->
Wo steht das Data Warehouse in der ETL-Kette — und was macht es mit den Daten, was Postgres nicht macht?::
Es ist das **Ziel** der Pipeline (das L in ETL). Bei **ELT** ist es zusätzlich der Ort, an dem transformiert wird.<br>Was es tut, was Postgres nicht tut: **integrieren** (mehrere Quellschemata in ein Modell), **Historie führen**, **spaltenweise und komprimiert speichern**, **Storage von Compute trennen**, **Bulk-Loads** statt Einzelsatz-Writes verarbeiten.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obslv5lPFtwFTa7ct8UNkmEV" -->
Warum reicht eine **Read Replica** nicht als Ersatz für ein Data Warehouse?::
Sie löst nur die **Interferenz** — die Last liegt nicht mehr auf dem Primär.<br>Nicht gelöst: **keine Integration** (nur das Schema des einen Quellsystems) · **keine Historie** (Updates werden mitgespiegelt, Altes ist weg) · **kein anderes Speicherformat** (immer noch Row Store mit denselben Seiten).<br>Zusätzlich neu: Langlaufende Queries **kollidieren mit dem WAL-Replay** — entweder wird die Query abgebrochen (`canceling statement due to conflict with recovery`) oder das Replay verzögert sich. Steuerung über `max_standby_streaming_delay`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLjpWBeBGovIBdIznoOqQg" -->
Wie ist die Zeilenzugehörigkeit in einem Column Store gespeichert?::
**Gar nicht explizit — sie ist positionell.**<br>Position *k* in jeder Spaltendatei gehört zur selben logischen Zeile. Es gibt keine gespeicherte Zeilen-ID; die Position **ist** die Identität. Voraussetzung: alle Spalten sind in identischer Reihenfolge abgelegt.<br>Daraus folgt direkt das Schreibproblem — ein Einfügen in der Mitte würde jede Spaltendatei ab dieser Stelle verschieben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsigtslixvPoKz3KJbEdKAq" -->
Was ist **Projection Pushdown**, und wie groß ist der Effekt?::
Der Scan liest **nur die Spalten, die die Query nennt**.<br>Beispiel: Tabelle mit 100 Spalten und 200 Mio. Zeilen, Query `SELECT category, SUM(amount) ... GROUP BY category`.<br>Row Store: muss alle 100 Spalten aller 200 Mio. Zeilen lesen, weil die Zeile die kleinste physische Einheit ist.<br>Column Store: liest 2 von 100 Spaltendateien → rund **50× weniger Bytes**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0fNysNLyAvf5EjhjDoYvm" -->
Warum ist ein Row Store bei `SELECT * FROM customers WHERE customer_id = 4711` schneller als ein Column Store?::
Weil der Column Store **100 Spaltendateien anfassen** und aus jeder einen Wert an derselben Position holen müsste; im Row Store ist es **ein einziger Seitenzugriff**.<br>Merksatz: Der Row Store optimiert Zeilenzugriff, der Column Store Spaltenzugriff. Welcher „schneller" ist, ist keine Eigenschaft des Formats, sondern **eine Eigenschaft der Query**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmrR8cNuIRCqxEgcLr610E" -->
Warum lässt sich eine **Spalte** deutlich besser komprimieren als eine Zeile? Zwei Gründe::
**1. Homogenität** — eine Zeile mischt Integer, Timestamp, String und Bool; eine Spalte enthält ausschließlich Werte **desselben Typs aus derselben Wertedomäne**. Kompression lebt von Redundanz, und homogene Folgen haben davon ungleich mehr.<br>**2. Niedrige Kardinalität** — Spalten wie `country` oder `category` haben oft nur einige hundert verschiedene Werte bei Millionen Zeilen, jeder Wert wiederholt sich also millionenfach.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqCnFGS5K7MtASB01GfFif" -->
Erkläre **Dictionary Encoding**, **Bitmap Encoding** und **Run-Length Encoding**::
**Dictionary:** Die *n* verschiedenen Werte stehen einmal in einem Wörterbuch, die Spalte speichert nur noch Indizes (bei 200 Ländern 8 Bit statt eines Strings).<br>**Bitmap:** Pro möglichem Wert eine Bitmap über alle Zeilen — Bit *k* ist 1, wenn Zeile *k* diesen Wert hat.<br>**RLE:** Statt der Bits die **Lauflängen** speichern — „370 Nullen, dann 8 Einsen". Besonders wirksam bei sortierten Daten oder dünn besetzten Bitmaps.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseObEjhlLJkey2oBp8tVAf" -->
Warum ist bei komprimierten Spalten das **Rechnen auf komprimierten Daten** wichtiger als die Platzersparnis?::
Weil viele Operationen gar keine Dekompression brauchen.<br>`WHERE product_id IN (30, 68)` wird zu einem bitweisen **OR** zweier Bitmaps, `... AND store_id = 3` zu einem bitweisen **AND** — CPU-Instruktionen auf ganzen Maschinenwörtern statt Zeile-für-Zeile-Vergleich.<br>`COUNT` über eine RLE-Spalte addiert Lauflängen. Filter auf dictionary-kodierten Werten vergleichen Integer statt Strings.<br>Die Daten bleiben komprimiert bis kurz vor der Ausgabe — das spart **Speicherbandbreite**, den eigentlichen Engpass im analytischen Betrieb.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbKpzVgIgd8tVko7xRAtGQ" -->
Ein Column Store hat keine B-Tree-Indizes. Was tritt an ihre Stelle, und was bringt es?::
Die **physische Sortierung** der Daten. Zwei Effekte:<br>**1. Bessere Kompression** — nach `date` sortiert entstehen lange Läufe gleicher Werte, RLE komprimiert sie auf fast nichts. Der Effekt ist für die **erste** Sortierspalte am größten und nimmt schnell ab.<br>**2. Blöcke überspringen** — filtert die Query auf einen Zeitraum, lassen sich alle Blöcke außerhalb komplett auslassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsG0NQ4BygdZ1WyeNmq3tXZ" -->
Was sind **Zone Maps**, und warum wirken sie auf sortierten Daten viel stärker?::
Metadaten pro Block: der **kleinste und größte Wert je Spalte** (auch Min/Max-Statistiken; in Parquet *row group statistics*, in Snowflake die Micro-Partition-Metadaten).<br>Die Engine liest zuerst die Metadaten, verwirft alle Blöcke, deren Wertebereich den Filter nicht enthalten kann, und scannt nur den Rest.<br>Bei **zufälliger** Verteilung enthält fast jeder Block fast den ganzen Wertebereich — es lässt sich nichts überspringen. **Sortierung ist das, was Zone Maps scharf macht.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdAde6748Bp2gPjscG4pOG" -->
Warum speichern Column Stores dieselben Daten manchmal **mehrfach in verschiedenen Sortierreihenfolgen** — und warum ist das kaum teurer?::
Eine Tabelle kann physisch nur **eine** Sortierung haben, aber verschiedene Queries filtern auf verschiedene Spalten. Also legt man mehrere vollständige Kopien an, jede anders sortiert (in C-Store und Vertica: **Projections**); der Planer wählt die passende.<br>Kaum teurer, weil die Daten zur Ausfallsicherheit **ohnehin repliziert** werden — statt drei identischer Repliken gibt man jeder eine andere Sortierung. Und weil jede Kopie nach ihrer eigenen Leitspalte sortiert ist, komprimiert **jede** gut.<br>*Nicht zu verwechseln mit einem mehrspaltigen Sortierschlüssel — das ist eine Sortierung mit mehreren Ebenen.*
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGGqDspBy9ViiSwSOI5zYU" -->
Was ist **vektorisierte Ausführung**, und warum passt sie so gut zu Spaltenspeicherung?::
Statt einer Zeile pro `next()`-Aufruf (Volcano-/Iterator-Modell) reicht jeder Operator einen **Batch von typischerweise 1024 Werten einer Spalte** weiter. Der Aufruf-Overhead verteilt sich auf 1024 Werte statt auf einen.<br>Die innere Schleife läuft über ein **dichtes Array gleichartiger Werte**: passt in den L1/L2-Cache, **keine Verzweigungen** pro Wert, der Compiler kann **SIMD** erzeugen.<br>Passt zu Spaltenspeicherung, weil die Daten **bereits in genau dieser Form vorliegen** — ein Row Store müsste erst umkopieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYHBT8aGhzWZEuhyRSaI00" -->
Was ist der Unterschied zwischen **vektorisierter** und **kompilierter** Query-Ausführung?::
**Vektorisiert:** Die Engine interpretiert weiterhin, aber in Batches von ~1024 Spaltenwerten — der Interpretations-Overhead wird amortisiert.<br>**Kompiliert:** Die Engine erzeugt zur Laufzeit **Maschinencode für genau diese eine Query** und führt ihn aus; der Overhead verschwindet ganz (HyPer, Umbra).<br>Kompilierung kostet Zeit **vor** der Ausführung und lohnt erst ab einer gewissen Query-Laufzeit. Viele Engines kombinieren beides.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2iGUgPlFaiTWCcWLosQck" -->
Nenne die **zwei unabhängigen** Ursachen dafür, dass Column Stores bei Einzelsatz-Writes ein Problem haben::
**1. Positionelle Identität** — Zeile *k* ist über die Position *k* in allen Spaltendateien definiert; ein Einfügen in der Mitte würde jede Spaltendatei ab dieser Stelle verschieben. Bei sortierter Speicherung ist genau das der Normalfall.<br>**2. Kompression** — auch ohne Sortierung ginge kein Update in-place: Man müsste den RLE- oder dictionary-kodierten Block dekomprimieren, ändern, neu komprimieren — und dabei ändert sich die Größe, er passt evtl. nicht mehr an seinen Platz.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGVGzCFLu9Z843dcRIr2e3" -->
Wie lösen Column Stores das Schreibproblem?::
Mit einem **LSM-Ansatz**, derselben Idee wie bei LSM-Trees im OLTP-Teil.<br>**1.** Writes gehen in einen kleinen, schreiboptimierten Store im Hauptspeicher — zeilenorientiert oder zumindest unkomprimiert und unsortiert (in Vertica historisch *Write-Optimized Store* vs. *Read-Optimized Store*).<br>**2.** Queries lesen **beides** und führen die Ergebnisse zusammen.<br>**3.** Ist genug angesammelt, wird der Puffer **in einem Rutsch** mit den Spaltendateien gemerged, neu sortiert und komprimiert.<br>Einzelne Writes sind billig, die teure Reorganisation passiert gebündelt und amortisiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSlwkqOytAsahu4saet0gp" -->
Welche Postgres-Plan-Operation, die dort als teuer gilt, ist im Column-Store-Warehouse der **normale** Zugriffsweg — und warum?::
Der **Seq Scan** (Full Table Scan).<br>In Postgres ein Warnsignal im `EXPLAIN` — meist fehlt ein Index. Im Column Store ist er erwartet und optimiert, weil er nur die gebrauchten **Spalten** liest, diese **komprimiert** sind, **Zone Maps** ganze Blöcke überspringen lassen und der Scan **vektorisiert** läuft.<br>Deshalb bieten Snowflake und BigQuery **gar keine Indizes** im Postgres-Sinn an — keine fehlende Funktion, sondern die Konsequenz daraus, dass der Scan selbst billig ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDgnk0g2qHha1vK0jVnHep" -->
Was ist ein **Data Cube**, und warum spielt er heute eine kleinere Rolle als vor 20 Jahren?::
Ein **Gitter vorberechneter Aggregate über mehreren Dimensionen gleichzeitig** (Gray et al. 1996) — inkl. aller Teilsummen. „Umsatz pro Produkt über alle Filialen" ist ein Lesezugriff statt einer Aggregation über Millionen Zeilen.<br>Nachteil: **starr**. Er kennt nur die beim Bau festgelegten Dimensionen; „Umsatz nur für Bestellungen über 100 €" geht nicht.<br>Heute kleiner, weil Spaltenspeicherung, Kompression und Vektorisierung Ad-hoc-Aggregation so billig gemacht haben, dass der Vorberechnungsvorteil die Starrheit nicht mehr aufwiegt.<br>In SQL lebt die Idee als `CUBE`, `ROLLUP` und `GROUPING SETS` weiter.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAWP76X50vBvyPJJkxy4tr" -->
Was bedeuten **MOLAP**, **ROLAP** und **HOLAP** — und welches ist heute der Normalfall?::
**MOLAP** (*multidimensional*): vorberechnet in einem proprietären mehrdimensionalen Array (Essbase, SSAS Multidimensional).<br>**ROLAP** (*relational*): in relationalen Tabellen, meist Star Schema; aggregiert wird **zur Query-Zeit**.<br>**HOLAP** (*hybrid*): Detaildaten relational, häufige Aggregate im Würfel.<br>**Heute praktisch immer ROLAP** — moderne Warehouses aggregieren ad hoc schnell genug.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQAC5dqdOrb0DWUv0XglkY" -->
Wie läuft eine Query in einem **MPP**-Warehouse (shared-nothing) ab?::
**1.** Der Koordinator zerlegt die Query und schickt **denselben Plan an alle Knoten**.<br>**2.** Jeder Knoten führt ihn auf **seinem** Datenausschnitt aus und bildet lokale Teilergebnisse.<br>**3.** Für Joins und Aggregationen über Knotengrenzen werden Daten **umverteilt** (*shuffle* / *redistribute*) — **das ist der teure Schritt**.<br>**4.** Der Koordinator führt die Teilergebnisse zusammen.<br>Deshalb zählt, wonach die Daten verteilt sind: Liegen zwei zu joinende Tabellen nach demselben Schlüssel verteilt, ist der Join lokal und der Shuffle entfällt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsB5AdVP1VISkmGG090xxHK" -->
Was ändert die **Trennung von Storage und Compute** betrieblich? Vier Punkte::
Daten liegen im **Object Store** (S3/GCS/Blob) statt auf lokalen Platten der Rechenknoten.<br>**Mehr Rechenleistung:** größeren Cluster starten — Sekunden statt stundenlanger Datenumverteilung.<br>**Mehr Speicher:** nichts tun, der Object Store ist elastisch.<br>**Leerlauf:** Compute **abschalten**, nur Storage zahlen.<br>**Mehrere Workloads:** getrennte Cluster auf **denselben Daten** — ETL und Analysten bremsen sich nicht mehr gegenseitig.<br>Preis: Netzwerklatenz — deshalb sind lokales SSD-Caching und „wenig lesen" (Zone Maps, Partitionierung) noch wichtiger.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQ49m7lAkumfvdNhZxVFDu" -->
**Data Mart**, **OLAP-Cube** und **Semantic Layer** — was ist der Unterschied?::
**Data Mart:** ein **Ausschnitt** des Warehouses für einen Fachbereich, physische Tabellen. Motivation vor allem organisatorisch. Risiko: unabhängig gebaute Marts erzeugen genau die Inkonsistenzen wieder, gegen die das Warehouse antrat.<br>**OLAP-Cube:** **vorberechnete mehrdimensionale Aggregate**, ebenfalls physisch gespeichert.<br>**Semantic Layer:** eine **Definitionsschicht ohne eigene Daten** — definiert Kennzahlen einmal zentral („was heißt Umsatz?") und generiert daraus SQL (dbt Semantic Layer, Cube, LookML).<br>Die ersten beiden sind physische Daten, der dritte ist **Metadaten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2z2hF2LDmeDebNxY8iRgY" -->
Nenne die **vier** strukturellen Grenzen von Postgres als Analyse-Store::
**1. Kein natives Columnar Storage** — nur Heap-Tabellen; Projection Pushdown, spaltenweise Kompression und Vektorisierung fallen alle aus.<br>**2. Overhead pro Zeile** — **23 Byte** Tupel-Header (MVCC-IDs, NULL-Bitmap, Flags) plus **4 Byte** Item-Pointer plus Padding. Bei einer Faktentabelle aus fünf Integern ist der Verwaltungsanteil größer als die Nutzdaten.<br>**3. MVCC-Bloat** — jedes `UPDATE` schreibt eine neue Zeilenversion; ein Update über 50 Mio. Zeilen verdoppelt kurzfristig die Tabellengröße.<br>**4. Ein Prozess pro Query** — begrenzte Parallelisierung, immer nur **innerhalb einer Maschine**, kein Scale-out über Knoten.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3UYPJ7a4aSicPTFgQd2zU" -->
Welche Postgres-Erweiterungen schließen die analytische Lücke teilweise — und was ist ihre jeweilige Einschränkung?::
**Citus Columnar** — spaltenorientierter Table Access Method mit Kompression; im Wesentlichen **append-only**, kein `UPDATE`/`DELETE`.<br>**`pg_duckdb`** — bettet die DuckDB-Engine ein, vektorisierte Queries auch direkt auf Parquet; junges Projekt.<br>**TimescaleDB** — spaltenweise Kompression, auf Zeitreihen zugeschnitten.<br>**Citus (verteilt)** — verteilt Tabellen über mehrere Knoten; der Verteilungsschlüssel muss zur Query passen.<br>Oft übersehener Zwischenschritt: **DuckDB auf Parquet** — spaltenorientiert und vektorisiert, ohne Server und ohne Cluster.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspDqHjS2bCYmiQOtUskxz3" -->
Nenne fünf Kriterien, nach denen man sich für **Postgres statt eines Warehouses** entscheidet::
**1. Betriebsaufwand** — ein System weniger zu betreiben, sichern, absichern, bezahlen. In kleinen Teams meist das stärkste Argument.<br>**2. Frische** — wenn Auswertungen ohne ETL-Latenz auf dem Live-Zustand laufen müssen.<br>**3. Query-Muster** — besteht die „Analytik" aus vielen selektiven Lookups und kleinen Aggregaten, gewinnt der Row Store mit Index.<br>**4. Transaktionale Konsistenz** zwischen operativen und analytischen Daten.<br>**5. Team und Werkzeuge** — ohne Datenteam ist die realistische Alternative nicht „Warehouse", sondern „gar keine Auswertung".<br>Größenordnung: bis in den zweistelligen GB-Bereich ist Postgres meist völlig ausreichend.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg6m7F5PXeiBcva7Jq6WtJ" -->
Was ist **HTAP**, und warum will man es überhaupt?::
*Hybrid Transactional/Analytical Processing* — Systeme, die transaktionale Einzelsatz-Zugriffe **und** analytische Scans auf demselben Datenbestand bedienen, **ohne ETL dazwischen**.<br>Motivation: Die Trennung erkauft Performance mit **Latenz**. Für Betrugserkennung, dynamische Preisgestaltung oder Echtzeit-Personalisierung muss die Analyse auf sekundenfrischen Daten laufen und sofort in eine operative Entscheidung zurückfließen.<br>Historisch **war** bis in die 1990er jede Datenbank HTAP; die Trennung entstand als Performance-Kompromiss (Stonebraker, „One Size Fits All", 2005), nicht als Naturgesetz.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsv2gF5RM5aQc2bSLvaPvWh" -->
Welchen **Preis** zahlen HTAP-Systeme? Zwei Punkte, einer mit Zahlen::
**1. Speicherverdopplung** — praktisch alle halten die Daten **zweimal**, zeilen- und spaltenorientiert. Kostet Speicher und erzeugt Synchronisationsaufwand.<br>**2. Zielkonflikt Frische gegen Isolation** — je aktueller die analytische Kopie, desto enger die Kopplung, desto stärker die Störung des transaktionalen Betriebs.<br>**TiDB** koppelt lose über die Replikation: nur ~**10 % Performanceverlust**, dafür bis zu **1000 ms** Verzögerung.<br>**HyPer** erzeugt Snapshots in **Mikrosekunden**, dafür bis zu **40 % Performanceverlust**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDr5IqAysTTrkpiz0kO5bz" -->
Warum ersetzt HTAP **kein** Data Warehouse?::
Zwei Gründe.<br>**1.** Viele als HTAP vermarktete Systeme bestehen intern weiterhin aus einem transaktionalen **und** einem separaten analytischen System hinter gemeinsamer Fassade — der ETL-Schritt ist nicht verschwunden, sondern ins Produkt gewandert.<br>**2.** HTAP löst das **Latenz**-Problem für eine einzelne Anwendung, aber nicht das **Integrations**-Problem über viele Quellsysteme hinweg. Und Integration ist laut Inmon der halbe Zweck der Übung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg2wrdtT0JeHVqHj76NiJb" -->
Was sind **Pinot**, **Druid** und **ClickHouse** — und wo sitzen sie zwischen OLTP und Warehouse?::
**Real-Time-Analytics-Stores** (auch *product analytics*).<br>Spaltenorientiert wie ein Warehouse, aber auf **niedrige Latenz bei hoher Query-Rate** ausgelegt statt auf beliebige Ad-hoc-Queries — typischerweise für Dashboards, die vielen Nutzern gleichzeitig frische Kennzahlen zeigen.<br>Sie nehmen Daten oft direkt aus einem **Event-Stream** entgegen statt aus einem Batch-Load. Damit sind sie ein Beleg dafür, dass OLAP und OLTP Idealtypen mit einer durchgehenden Skala dazwischen sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2lb5ZGK2fR2PXvQ8nLVga" -->
Ist ein Data Warehouse dasselbe wie ein MPP-System?::
**Nein.** Ein Unternehmen kann ein Data Warehouse **ohne** MPP betreiben, und man kann ein MPP-System betreiben, das kein Data Warehouse ist.<br>**MPP** ist die *technische* Architektur (shared-nothing, Query über viele Knoten verteilt), **Data Warehouse** die *organisatorische* Funktion (integriertes, historisiertes Abbild mehrerer Quellsysteme).<br>Die beiden Begriffe sind historisch zusammengewachsen und werden oft gleichgesetzt — sie bezeichnen aber verschiedene Dinge.
<!-- basicblock-end -->

# Block 2 — Kimball & Normalisierung vs. Denormalisierung

*Die ersten acht Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="ObsAhxGKcC6Vdf3ZDHDzDUy3" -->
Warum sagt Kimball, ein Data Warehouse sei „only as good as the dimension attributes"?::
> „the analytic power of the DW/BI environment is **directly proportional to the quality and depth of the dimension attributes**"<br>Die Faktentabelle enthält Zahlen — die sind meist unstrittig. **Was man mit den Zahlen fragen kann, bestimmen ausschließlich die Dimensionen.** Fehlt in der Produktdimension das Attribut „Bio ja/nein", ist der Bio-Umsatz nicht auswertbar, egal wie perfekt die Faktentabelle ist.<br>Deshalb gilt: breite Dimensionen sind **erwünscht**, jedes sinnvolle Attribut erweitert den Fragenraum.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRz4wg0MzPp4TUKf0AX71t" -->
Warum darf ein Fremdschlüssel in der Faktentabelle **nicht** `NULL` sein — und was macht man stattdessen?::
Ein `NULL`-Fremdschlüssel würde die Zeile beim **Inner Join verschwinden** lassen und beim Outer Join zu unvorhersehbaren Ergebnissen führen. Kimball nennt das eine „referential integrity violation".<br>Stattdessen enthält jede Dimensionstabelle eine **Default-Zeile mit eigenem Surrogatschlüssel** für „unknown" bzw. „not applicable".<br>Nebeneffekt und eigenes Argument für Surrogatschlüssel: Für so eine Zeile gibt es **keinen natürlichen Schlüssel**, den man einsetzen könnte.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsw9LtgATeYqkCW7vpVKP1i" -->
„Fact table normalized, dimension table denormalized" — was heißt das genau?::
Die **Faktentabelle** ist normalisiert in dem Sinne, dass sie nur Fremdschlüssel, Kennzahlen und ggf. eine degenerate dimension enthält — nichts Beschreibendes, keine Redundanz.<br>Die **Dimensionstabelle** ist absichtlich denormalisiert: Die Hierarchie `product → brand → category → department` steckt als **redundante Spalten in einer flachen Tabelle**, nicht als Kette normalisierter Tabellen.<br>Die Asymmetrie ist ökonomisch begründet: Die Faktentabelle trägt **über 90 % des Volumens**, die Dimensionen sind vergleichsweise winzig. Man bläht die kleinen Tabellen auf, um die große schmal zu halten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsemFPesyZ9pKwZT9E2VD1Z" -->
Wann conformieren Dimensionstabellen — und welchen Mechanismus ermöglicht das?::
> „Dimension tables conform when attributes in separate dimension tables have the **same column names and domain contents**."<br>Der Mechanismus heißt **Drill-Across**: Wird ein konformiertes Attribut als **Zeilenkopf** (Gruppierungsspalte) verwendet, lassen sich Ergebnisse aus **getrennten Faktentabellen auf denselben Zeilen ausrichten**.<br>Kimball nennt das „the essence of integration in an enterprise DW/BI system". Conformed Dimensions werden **einmal** in Zusammenarbeit mit den Data-Governance-Vertretern definiert und über Faktentabellen hinweg wiederverwendet.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5hwUNtItFb3zoNLJloPtz" -->
Was passiert, wenn zwei Fachbereiche dieselbe Dimension **unabhängig** voneinander definieren?::
Sie **conformieren nicht** — und damit ist Drill-Across **unmöglich**. Drei Varianten des Problems:<br>Die Spalten heißen anders (`kundengruppe` vs. `customer_segment`).<br>Sie heißen gleich und bedeuten Verschiedenes (Vertrieb zählt als Kunde, wer bestellt hat; Finanzen, wer bezahlt hat).<br>Die Wertebereiche weichen ab (`DE` vs. `Deutschland` vs. `Germany`).<br>In jedem Fall lassen sich die Zeilen nicht aufeinander ausrichten. Der Schaden ist nicht nur technisch: Zwei Berichte nennen für dieselbe Frage verschiedene Zahlen, beide sind in ihrem Kontext korrekt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDxfWKxY0nqUZVVRgJJ1Ty" -->
Warum braucht man eine explizite **Date Dimension**, statt einfach ein `DATE` in der Faktentabelle zu filtern? Vier Argumente::
**1.** Der Join ist nicht das Problem — Kimball: *„if your relational database can't handle an efficient join to the date dimension table, you're in deep trouble."* Eine Tabelle mit ~4.000 Zeilen pro Dekade kostet nichts.<br>**2.** Fachanwender können kein SQL-Datumsrechnen.<br>**3.** **SQL kennt die Begriffe nicht**: Werktag vs. Wochenende, Feiertage, Fiskalperioden, Saisons. Das ist Geschäftswissen und gehört in eine Tabelle.<br>**4.** *„Calendar logic belongs in a dimension table, not in the application code"* — sonst implementiert es jedes Werkzeug neu und jedes anders.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQHE7ymZtVccpl8TrvcGLL" -->
Wieso besteht Kimball auf „Collaborative Dimensional Modeling Workshops"?::
> *„Dimensional models should not be designed in isolation by folks who don't fully understand the business and their needs; collaboration is critical!"*<br>Der Grund ist nicht Höflichkeit: **Der Grain ist eine fachliche Entscheidung**, und sie ist technisch schwer revidierbar. Dasselbe gilt für die Wahl des Geschäftsprozesses (Schritte 1 und 2 des Vier-Schritte-Prozesses).<br>Wie fachlich das ist, zeigt die Frage „was heißt eigentlich *Kunde*?" — jemand, der in den letzten 30 Tagen gekauft hat? In sechs Monaten? Die Antwort verändert jeden Report über Kundenverhalten und jedes Churn-Modell.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXady6XXBUp8FK0MnEFhPA" -->
Was tut man, wenn einer bestehenden Faktentabelle neue Kennzahlen hinzugefügt werden sollen?::
Das geht nur, **wenn sie sich dem bestehenden Grain zuordnen lassen**. Kimball: *„if the new facts cannot be allocated or assigned to the original grain of the fact table"* — dann braucht die Kennzahl eine **eigene Faktentabelle**.<br>Man verändert **nicht** den Grain der bestehenden Tabelle. Das wäre der Weg in einen **gemischten Grain** — Zeilen unterschiedlicher Bedeutung in einer Tabelle, wodurch jede Aggregation stillschweigend falsch wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHHZHeICou8leXK2XLucWP" -->
Wie unterscheidet man bei einer konkreten Spalte, ob sie ein **Fakt** oder ein **Dimensionsattribut** ist?::
Der Test ist **inhaltlich, nicht strukturell**:<br>Wird die Spalte **gerechnet** — summiert, gemittelt, gezählt? → **Fakt**<br>Wird nach ihr **gefiltert oder gruppiert** (Zeilen- bzw. Spaltenkopf im Report)? → **Dimensionsattribut**<br>Der Grenzfall sind **numerische Werte, die keine Fakten sind**: Ein Listenpreis ist ein Fakt, wenn man mit ihm rechnet (Umsatz = Preis × Menge), und ein Dimensionsattribut, wenn man Produkte nach Preisband gruppiert. Kimballs Regel: Dient eine Spalte überwiegend dem Gruppieren, gehört sie in die Dimension — auch als Zahl.<br>„Ist es ein Fremdschlüssel?" greift nur *innerhalb* einer Faktentabelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2YncQnJNYeLtZu8sJarPY" -->
Aus welchen **drei** Sorten Spalten besteht eine Faktentabelle?::
**1.** **Fremdschlüssel** auf die Dimensionstabellen — einer pro Dimension<br>**2.** **Kennzahlen** (die Fakten) — numerisch, in der Regel additiv<br>**3.** gelegentlich eine **degenerate dimension**<br>Mehr nicht. **Alles Beschreibende gehört in eine Dimension.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCHJ5Hn5surTrmoCU8VYk1" -->
Was bedeutet „Faktentabellen sind **sparse**", und wo ist der Unterschied zum Data Cube?::
Es gibt nur Zeilen für Ereignisse, die **tatsächlich stattgefunden haben**. Wurde ein Produkt an einem Tag in einer Filiale nicht verkauft, existiert **keine Zeile mit Menge 0**.<br>Der Data Cube hält dagegen **jede Zellkombination** vor. 100 Filialen × 60.000 Produkte × 365 Tage wären 2,2 Milliarden Kombinationen; tatsächlich verkauft wird ein Bruchteil.<br>Konsequenz: Eine Faktentabelle kann **negative Fragen nicht beantworten** („was wurde *nicht* verkauft?") — dafür braucht man eine Coverage Table.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiwKjbUbGKnDMF2XCXY8GC" -->
Was bedeuten **additiv**, **semi-additiv** und **nicht-additiv** — je mit Beispiel?::
**Additiv** — summierbar über **alle** Dimensionen. Verkaufsmenge, Umsatzbetrag, Kosten.<br>**Semi-additiv** — summierbar über alle **außer der Zeit**. **Bestände**: Lagerbestand, Kontostand, Anzahl offener Tickets. 40 Stück Montag plus 45 Dienstag sind nicht 85; über Filialen summiert dagegen sinnvoll. Für die Zeitachse nimmt man **Durchschnitt** oder Periodenendwert.<br>**Nicht-additiv** — über **keine** Dimension summierbar. **Verhältnisse**: Marge in Prozent, Stückpreis, Quoten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstrntzYEvSsYUZT6Fxdhtd" -->
Wie speichert man eine **nicht-additive** Kennzahl richtig — und warum ist die Additivität praktisch so wichtig?::
Nicht das Verhältnis speichern, sondern **Zähler und Nenner getrennt** als additive Fakten. Der Quotient entsteht erst im Report, **nach** der Aggregation. Der Durchschnitt zweier Prozentsätze ist nur dann die Gesamtmarge, wenn beide Grundgesamtheiten gleich groß sind.<br>Warum es zählt: **BI-Werkzeuge summieren standardmäßig.** Eine semi-additive oder nicht-additive Kennzahl über die falsche Dimension summiert liefert eine Zahl, die **plausibel aussieht und falsch ist** — und niemand bemerkt es, weil kein Fehler auftritt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFts5LIxxEhA8cqjTye5Hz" -->
Nenne die **drei** Faktentabellen-Typen und was eine Zeile jeweils bedeutet::
**Transaction** — eine Zeile pro **Ereignis** zum Zeitpunkt seines Auftretens. Der Normalfall, atomarster Grain, sparse.<br>**Periodic Snapshot** — eine Zeile pro **Zeitraum pro Entität**, unabhängig davon ob etwas passiert ist. Dicht statt sparse.<br>**Accumulating Snapshot** — eine Zeile pro **Vorgang** mit definiertem Anfang und Ende, die **wiederholt aktualisiert** wird.<br>*Nicht verwechseln:* „Factless fact table" und „allocated facts" sind andere Konzepte, keine Typen dieser Taxonomie.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obspfj1iT7GVEr9SUMcFnQ8C" -->
Wann nimmt man einen **Periodic Snapshot** statt einer Transaction-Faktentabelle?::
Wenn die interessante Größe ein **Zustand** ist und kein Ereignis: Lagerbestand, Kontostand, Vertragsbestand.<br>Aus einer Transaction-Tabelle ließe sich „Bestand am 3. März" nur rekonstruieren, indem man **alle Zu- und Abgänge seit Beginn aufsummiert** — bei Milliarden Zeilen nicht praktikabel.<br>Zwei Eigenschaften: Die Tabelle ist **dicht** (jede Kombination bekommt pro Periode eine Zeile, auch mit Wert 0), und ihre Kennzahlen sind typischerweise **semi-additiv**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskBoSk3rJZwDfilamTJWwL" -->
Wofür braucht man den **Accumulating Snapshot**, und was ist an ihm ungewöhnlich?::
Er modelliert **Pipeline-Prozesse** mit einer überschaubaren Zahl bekannter Meilensteine: eine Bestellung (bestellt → bezahlt → verpackt → versandt → geliefert), ein Kreditantrag, ein Schadensfall.<br>**Das Ungewöhnliche:** Seine Zeilen werden **wiederholt aktualisiert** — er bricht die Regel, dass Faktenzeilen unveränderlich sind. Pro Meilenstein gibt es eine eigene Datumsspalte; noch nicht erreichte Meilensteine zeigen auf die „unbekannt"-Zeile der Datumsdimension.<br>Der Gewinn: **Durchlaufzeiten** sind direkt auswertbar, ohne Selbst-Join über Ereigniszeilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbFdZBp6eCi7yAytfmi9Zj" -->
Nenne die vier Schritte des Kimball-Design-Prozesses und begründe die Reihenfolge::
**1.** Select the business process · **2.** Declare the grain · **3.** Identify the dimensions · **4.** Identify the facts<br>**1 vor 2:** Der Grain ist eine Aussage über ein Ereignis — ohne zu wissen, *welcher Prozess*, gibt es kein Ereignis, über das man eine Aussage machen könnte.<br>**2 vor 3 und 4:** Eine Dimension ist nur verwendbar, wenn sie **beim gewählten Grain eindeutig** ist. Bei „ein Verkauf pro Produkt pro Bon" ist der Kunde eindeutig; bei „ein Tag pro Filiale" gibt es viele Kunden und die Kundendimension ist **nicht anschließbar**. Genauso müssen sich Kennzahlen auf den Grain herunterbrechen lassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmdzYMyBAYOIR7jwgjhCwt" -->
Wie legt man den **Grain** konkret fest, und welche Faustregel gilt?::
Man formuliert einen **Satz, der beschreibt, was eine Zeile ist**. Für Supermarkt-Verkäufe: *„eine Zeile pro Produkt pro Bon."* Die Menge ist dann eine **Kennzahl** in dieser Zeile — zwei Packungen Tomatensauce auf einem Bon sind eine Zeile mit `quantity = 2`, nicht zwei Zeilen.<br>**Faustregel: so fein wie möglich** — der atomare Grain. Zwei Gründe: Man kann jederzeit **aggregieren, aber niemals nachträglich verfeinern**. Und der atomare Grain ist **maximal dimensionierbar** — je feiner das Ereignis, desto mehr Kontext ist eindeutig zuordenbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDgzGjMgZFx6KrU1eqOQIA" -->
Was sind die **zwei** Fehlerarten bei einem unsauberen Grain — und welche ist gefährlicher?::
**Zu grob.** Ist der Grain „täglicher Umsatz pro Filiale", ist jede Frage nach einzelnen Produkten unbeantwortbar. Die Information ist **vernichtet** und kommt ohne Neuladen nicht zurück.<br>**Gemischt — der gefährlichere Fall.** Zeilen mit **unterschiedlicher Bedeutung** in derselben Tabelle: Einzelverkäufe *und* Tagessummen, oder Bestellpositionen *und* Versandkosten auf Bestellebene. Dann ist **jede Aggregation falsch** (Doppelzählung) — und es fällt lange nicht auf, weil die Query fehlerfrei läuft und eine Zahl liefert.<br>Kimball: *„the most frequent error is not declaring the grain … the whole design rests on quicksand."*
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsd4c5vPsVJxaOAuFscEIPN" -->
Nenne die Gründe für **Surrogatschlüssel** in Dimensionstabellen, nach Wichtigkeit::
**1. SCD Typ 2 erzwingt sie** — dieselbe Entität existiert in mehreren Zeilen, der natürliche Schlüssel ist damit **nicht mehr eindeutig** und kann kein Primärschlüssel sein.<br>**2. Integration mehrerer Quellsysteme** — „Kunde 123" bedeutet im CRM und im Shop zwei verschiedene Menschen.<br>**3. Pufferung gegen Quellsystem-Änderungen** — Umnummerierung betrifft eine Spalte, nicht Milliarden Fremdschlüssel.<br>**4. Spezialzeilen** für „unbekannt"/„nicht anwendbar", die kein natürliches Gegenstück haben.<br>**5. Platz und Performance** — 4-Byte-Integer statt zusammengesetztem Textschlüssel.<br>**6. Wiederverwendete operative Schlüssel** — sonst werden alte Fakten stillschweigend dem falschen Objekt zugeordnet.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2HoDo89GUrohFOPfbTzT1" -->
Warum **erzwingt** SCD Typ 2 Surrogatschlüssel? Das strukturelle Argument::
Bei Typ 2 existiert **dieselbe Entität in mehreren Zeilen** — eine pro Gültigkeitszeitraum. Der natürliche Schlüssel kommt damit **mehrfach vor** und **kann** kein Primärschlüssel mehr sein.<br>Man braucht einen Schlüssel **pro Version**, und die Faktenzeile verweist auf genau die Version, die zum Ereigniszeitpunkt gültig war.<br>**Der Punkt ist nicht, dass sich der natürliche Schlüssel ändert — sondern dass er nicht mehr eindeutig ist.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSLMwlCXez7sl1RkkrlJOh" -->
Was ist eine **degenerate dimension**?::
Ein operatives Ordnungsmerkmal **ohne eigene Attribute**: Bestellnummer, Bonnummer, Rechnungsnummer, Ticketnummer.<br>Man legt dafür **keine** Dimensionstabelle an — sie hätte nur eine Spalte — sondern lässt den Wert **direkt in der Faktentabelle** stehen.<br>Es ist trotzdem eine Dimension: Man gruppiert danach, etwa um „durchschnittliche Anzahl Artikel pro Bon" zu berechnen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrGwF2lg8AphGYF9SPLk0R" -->
Was ist eine **junk dimension**, und welches Problem löst sie?::
Quellsysteme liefern oft ein Dutzend niedrigkardinale Flags: `is_return`, `payment_type`, `channel`, `promo_applied`, `gift_wrap`. Zwölf Mini-Dimensionen wären absurd, zwölf Spalten in der Faktentabelle verschwenden Platz.<br>Die Lösung: **alle auftretenden Kombinationen in eine Tabelle** mit einem Surrogatschlüssel. Die Faktentabelle hat dann **einen** Fremdschlüssel statt fünf Spalten.<br>Wichtig: Man legt nur die **tatsächlich auftretenden** Kombinationen an, nicht das kartesische Produkt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqQVxWnhuRNURd2MSUdQQF" -->
Was ist eine **role-playing dimension**, und was eine **outrigger dimension**?::
**Role-Playing:** Eine Faktentabelle verweist mehrfach auf **dieselbe** Dimension in verschiedenen Rollen — Bestelldatum, Versanddatum, Lieferdatum. Physisch eine `dim_date`; logisch legt man **Views** darüber (`dim_order_date`, `dim_ship_date`), damit Spaltennamen eindeutig sind.<br>**Outrigger:** Eine Dimensionstabelle verweist auf eine **weitere** Dimensionstabelle — z. B. „Datum des Erstkaufs" in `dim_customer` zeigt auf `dim_date`, statt 20 Kalenderattribute zu kopieren. Kimballs **eigene Ausnahme** vom Snowflake-Verbot, aber ausdrücklich nur **„in Maßen"**: Wer jede Hierarchie auslagert, hat ein Snowflake-Schema gebaut und nur einen anderen Namen dafür.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaKJ9cRNDt9LX2uS3Fb7nh" -->
Was ist eine **factless fact table**, und welche zwei Typen gibt es?::
Eine Faktentabelle **ohne Kennzahlen** — nur Fremdschlüssel.<br>**Typ 1: Ereignis-Verfolgung.** Manche Ereignisse haben keine Messgröße: Vorlesungsteilnahme, Anzeigenklick, absolvierte Schulung. Ausgewertet wird mit `COUNT(*)` — die **Zeilenzahl ist die Kennzahl**. Manchmal fügt man eine Dummy-Spalte mit konstant 1 hinzu, damit BI-Werkzeuge etwas zum Summieren haben.<br>**Typ 2: Coverage Table.** Bildet die **Möglichkeit** ab statt des Ereignisses.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSwgUR6IT9mScMsbI8NTLk" -->
Wozu braucht man eine **Coverage Table** — welche Frage ist ohne sie unbeantwortbar?::
**Negative Fragen**: *„Welche Produkte waren in der Aktion und wurden trotzdem nicht verkauft?"*<br>In `fct_sales` steht nichts über nicht verkaufte Produkte, weil Faktentabellen **sparse** sind. Die Lösung ist eine faktenlose Tabelle, die jede Kombination enthält, die *hätte* eintreten können:<br>`fct_promotion_coverage: date_key | store_key | product_key | promotion_key`<br>Die Frage wird dann zur **Mengendifferenz** zwischen Coverage- und Sales-Tabelle.<br>Merksatz: **Negative Fragen brauchen eine Tabelle, die das Erwartete enthält** — aus den Ereignissen allein sind sie nicht zu beantworten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsccFu06dewa57Xxl6Mu4RD" -->
Wozu dient eine **Bridge Table**, und warum ist der Gewichtungsfaktor entscheidend?::
Für **mehrwertige Dimensionen**, wenn eine Dimension beim gegebenen Grain *nicht* eindeutig ist: ein Krankenhausaufenthalt mit mehreren Diagnosen, ein Konto mit mehreren Inhabern, ein Buch mit mehreren Autoren.<br>Die Faktenzeile verweist auf eine **Gruppe**, die Bridge löst sie in die Einzelwerte auf. Der **Gewichtungsfaktor** summiert sich pro Gruppe auf 1,0 und ermöglicht zwei korrekte Auswertungen:<br>**Impact Report** (mit Gewichtung) — Kosten anteilig verteilt, Summe ergibt die tatsächlichen Gesamtkosten.<br>**Correlated Report** (ohne Gewichtung) — „wie viele Fälle hatten Diagnose X?", Summe **absichtlich** größer als die Fallzahl.<br>Ohne Gewichtungsfaktor liefert der Join **Doppelzählungen**, und niemand merkt es (dasselbe wie das Fan-Out-Problem bei OBT).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7aLgJ1EZmUTnzSad3TWiv" -->
Was sind **Aggregate Fact Tables**, und welche zwei Regeln machen sie erst brauchbar?::
Vorberechnete Faktentabellen auf **grobkörnigerem Grain** — z. B. `fct_sales_monthly` (Produkt pro Filiale pro Monat) neben dem atomaren `fct_sales`.<br>**Regel 1 — Konsistenz:** Eine Auswertung darf nicht davon abhängen, gegen welche Tabelle sie läuft. Die Dimensionen müssen **shrunken dimensions** sein: echte Teilmengen der Basisdimensionen mit **identischen Spaltennamen und Wertebereichen**. Eine Monatsdimension mit anderen Monatsnamen als die Tagesdimension ist ein Fehler.<br>**Regel 2 — Aggregate Navigation:** Eine Zwischenschicht (Query-Rewrite, BI-Werkzeug, Semantic Layer) leitet Queries automatisch auf die Aggregattabelle um. Ohne sie sind Aggregate eine Belastung — jeder muss wissen, welche Tabelle er nehmen soll, und die Hälfte nimmt die falsche.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJCgHQc2kpL9kBkAfnkgz3" -->
Was ist der strukturelle Unterschied zwischen **Star** und **Snowflake**?::
**Star = denormalisierte Dimensionen. Snowflake = normalisierte Dimensionen.**<br>Im Star enthält `dim_product` die Spalten `brand`, `category`, `department` **direkt** — redundant, flach, in einer Tabelle.<br>Im Snowflake sind die Hierarchiestufen in **eigene Tabellen** ausgelagert und über Fremdschlüssel verkettet, also in 3NF gebracht.<br>Konsequenz für Queries: Im Star ist jede Dimension **ein Hop** von der Faktentabelle entfernt; im Snowflake kommen pro Hierarchiestufe **verkettete** Hops dazu.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvkNolvFDdP2lhrmyOh9uT" -->
Nenne Kimballs vier Argumente **gegen** Snowflaking — und welches ist das technisch stärkste, welches das schwächste?::
**1. Verständlichkeit** — Fachanwender müssen die Hierarchie kennen, um zu navigieren.<br>**2. Query-Performance** — mehr und tiefere Joins, schlechtere Pläne.<br>**3. Die Platzersparnis ist irrelevant** — Dimensionen sind gegenüber der Faktentabelle winzig.<br>**4. Bitmap-Indizes und Star-Join-Optimierung werden ausgehebelt** — das Star-Muster wird unkenntlich.<br>**Technisch am stärksten ist Nr. 4**: ein qualitativer Ausfall, kein gradueller Aufschlag. **Am schwächsten Nr. 3** — also genau das, was in Kapitel 1.2 der Hauptgrund für Normalisierung war.<br>Nicht Nr. 2: Kimball selbst schreibt, Optimizer lösten dimensionale Queries effizient, *„it is not necessary to avoid joins like the plague."*<br>Fünftes, separat genanntes Argument: **Browse-Performance** — `SELECT DISTINCT category` ist im Star ein kleiner Scan, im Snowflake ein Join.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscInNjrLBnRELugGsRGBre" -->
In welchen Fällen ist **Snowflaking** trotzdem vertretbar?::
**Sehr große Dimension mit hochkardinalem Attribut** — Kimballs eigene Ausnahme; im Column Store aber kaum noch relevant, weil genau diese Redundanz dictionary-kodiert auf fast nichts komprimiert.<br>**Outrigger für geteilte Hierarchien** — wenn die Unterstruktur selbst eine vollwertige, konformierte Dimension ist.<br>**Die Unterstruktur ist Dimension einer anderen Faktentabelle** — dann ist es Wiederverwendung, keine Auslagerung (Adamsons Gegenposition zu Kimball).<br>**Das Modell ist generiert** — wird der Mart aus einem Data Vault oder einer 3NF-Kernschicht abgeleitet, ist die normalisierte Form ohnehin vorhanden.<br>**Mehrsprachigkeit und volatile Attribute** mit eigener Änderungsfrequenz.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssQaE5KPODWQyshqZTKC2I" -->
Wann ist normalisiert besser, wann denormalisiert? (Kleppmanns Abwägung)::
> „Normalization tends to be better for **OLTP** systems, where both reads and updates need to be fast; **analytical** systems often fare better with denormalized data, since they perform **updates in bulk** and the performance of read-only queries is the dominant concern."<br>Ergänzend: In **kleinen bis mittleren** Systemen ist normalisiert meist besser, *„because you don't have to worry about keeping multiple copies of the data consistent with one another, and the cost of performing joins is acceptable"* — in **sehr großen** Systemen kann der Join-Aufwand zum Problem werden.<br>Die zwei Achsen sind also **Zugriffsmuster** (lesen vs. ändern) und **Maßstab**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyCwUfNXX9ABEYTmh3nPIw" -->
Warum ist eine redundante Textspalte im **Column Store** fast kostenlos, in Postgres aber nicht?::
Eine denormalisierte Dimensionsspalte wie `brand = 'Coca-Cola'`, tausendfach wiederholt, ist **niedrigkardinal** — und damit der **Idealfall für Dictionary Encoding und RLE**. Statt des Strings steht pro Zeile ein kleiner Integer-Index, bei Sortierung schrumpfen die Läufe weiter.<br>In Postgres liegt der **vollständige String in jeder Zeile**.<br>Merksatz: **Nicht der Speicher ist billig — die Redundanz selbst ist im Column Store fast kostenlos.** Deshalb trägt das Argument „Speicher ist billig" hier und ist nicht bloß eine Ausrede.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8BjHCRK0HGx3dvn9dN1ND" -->
Was ist der **Hauptgrund**, warum Denormalisierung im OLTP-System falsch ist — und warum nicht die Performance?::
**Update-Anomalien und Inkonsistenz.** Ändert sich ein Markenname, muss er an *n* Stellen geändert werden. Schlägt die Operation in der Mitte fehl, stehen **widersprüchliche Werte** in der Datenbank, und es gibt keine Instanz, die entscheidet, welcher gilt. Genau davor schützt 3NF.<br>Ein OLTP-System ändert **einzelne Datensätze im laufenden Betrieb** — dort ist das ein reales Risiko. Ein Warehouse wird gebündelt und kontrolliert geladen.<br>Warum nicht die Performance: **Die ist reparierbar. Widersprüchliche Daten sind es nicht.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQJfKtZkzfZzdXoWkh3r9R" -->
Warum darf eine denormalisierte Tabelle „nie im Vakuum existieren"?::
Weil sie damit zum **System of Record** geworden ist — in einer dafür untauglichen Form. Jedes Faktum steht vielfach darin. Drei Folgen, die gleichzeitig gelten:<br>**1. Keine Autorität bei Widersprüchen** — stehen 1.000 Zeilen auf `'Coca-Cola'` und 3 auf `'Coca Cola'`, gibt es keine Stelle, an der die Marke *einmal* definiert ist.<br>**2. Nicht wiederherstellbar** — Derived Data ist dadurch definiert, dass man es neu erzeugen kann. Ohne Quelle ist ein Fehler in der Transformationslogik **nicht mehr korrigierbar**.<br>**3. Keine Prüfbarkeit** — es gibt keinen Abgleich, der eine Abweichung zeigen würde.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsV9y62ncVGCnLaoKbCw3qE" -->
Was passiert konkret, wenn man direkt in einer denormalisierten Tabelle korrigiert? Zwei Fälle::
**Fall 1 — die Pipeline läuft wieder:** Die Korrektur wird **überschrieben**. Die Arbeit ist weg, und weil niemand dokumentiert hat, dass hier eingegriffen wurde, wiederholt sich das beim nächsten Report.<br>**Fall 2 — die Pipeline läuft nicht wieder:** Die abgeleitete Tabelle ist **dauerhaft von der Quelle abgewichen**, ohne Vermerk. Ein späterer Neuaufbau würde die Korrektur verlieren, und niemand weiß, dass er sie verlieren würde.<br>**Fall 2 ist der gefährlichere, weil er still ist.** Er zerstört die Eigenschaft, auf der der Warehouse-Betrieb beruht: dass man jederzeit neu laden kann und dasselbe herauskommt — also **Idempotenz**.<br>Richtig ist: in der **Quelle** korrigieren, abgeleitete Tabelle neu aufbauen. Nie umgekehrt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnANEe2QGP7X0EJSxVS3EX" -->
Wie hängt die goldene Regel der Denormalisierung mit *System of Record* und *Derived Data* zusammen?::
Sie ist **kein eigenes Prinzip**, sondern dieselbe Unterscheidung auf Tabellen angewendet:<br>**Die normalisierte Form ist das System of Record. Die denormalisierte ist Derived Data.**<br>Und für Derived Data gilt die Regel: *bei Zweifeln neu berechnen, niemals von Hand korrigieren.*<br>Deshalb hat jedes Warehouse eine normalisierte oder zumindest quellnahe Eingangsschicht — Staging, Raw, Bronze, Raw Vault, je nach Vokabular.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAcTvTZjPpMWDS36U85dHi" -->
Was steht in Zeilen und Spalten der **Enterprise Data Warehouse Bus Matrix**, und welche Frage beantwortet sie?::
**Zeilen: Geschäftsprozesse** (die künftigen Faktentabellen) — Verkauf, Lagerbestand, Einkauf, Retouren.<br>**Spalten: Dimensionen** — Datum, Produkt, Filiale, Kunde, Lieferant. **Kreuz**, wenn der Prozess die Dimension nutzt.<br>Was sie leistet: Eine Spalte mit **mehreren Kreuzen** zeigt eine Dimension, die mehrere Prozesse verbindet — und die deshalb **konformiert sein muss**. Aus einer abstrakten Governance-Forderung wird eine priorisierte Liste.<br>Zweiter Nutzen: Sie ist ein **Projektplan** — jede Zeile ist ein inkrementell lieferbares Datenprodukt, das trotzdem ins Gesamtbild passt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obst5lUdjc9vTXMO3MTPP3Ym" -->
Was passiert in BigQuery oder Snowflake, wenn du eine Faktenzeile mit einem **nicht existierenden** Dimensionsschlüssel einfügst?::
**Nichts — sie akzeptieren es.** Diese Warehouses nehmen Primär- und Fremdschlüssel-Deklarationen an, **erzwingen sie aber nicht**; sie sind Metadaten für den Optimierer, keine Garantie. `NOT NULL` wird in der Regel erzwungen, der Rest nicht.<br>Konsequenz: **Referentielle Integrität wird zur Aufgabe der Pipeline.** Der klassische Mechanismus ist die **Surrogate Key Pipeline** — beim Laden wird jeder natürliche Schlüssel gegen die Dimension aufgelöst; findet sich kein Treffer, wird die „unbekannt"-Zeile verwendet und der Fall protokolliert.<br>In Postgres dagegen lohnt es, Fremdschlüssel zu setzen: Sie dokumentieren das Modell und fangen Ladefehler früh — kosten aber beim Bulk-Load Prüfzeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZ5OdjwIzUeG3V8cNGq2CN" -->
Warum ist bei Joins die **Tiefe** wichtiger als die **Anzahl** — und was hat der Query-Planer damit zu tun?::
Eine Star-Query joint die Faktentabelle mit **jeder** benötigten Dimension — bei fünf Dimensionen fünf Joins. Die Anzahl ist also nicht klein. Entscheidend ist, dass alle Joins **ein Hop** von der Faktentabelle weg und **unabhängig voneinander** auflösbar sind. Im Snowflake sind die Hops **verkettet**.<br>**Für den Planer:** Die Join-Reihenfolge ist kombinatorisch — bei *n* Tabellen faktoriell viele Möglichkeiten. Postgres sucht erschöpfend nur bis `join_collapse_limit` / `from_collapse_limit` (Standard **8**) und schaltet darüber auf Heuristiken bzw. ab `geqo_threshold` (Standard 12) auf einen **genetischen Optimierer** um.<br>Mehr Tabellen heißt also nicht nur mehr Arbeit, sondern **potenziell schlechtere Pläne** — und das trifft als Sprung ein, nicht linear.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspmVBH9HXue3eHZ2WHlG7w" -->
Welche zwei Star-spezifischen Optimierungen setzen voraus, dass das Star-Muster erkennbar ist?::
**Star Transformation / Star Join** — erst die kleinen, gefilterten Dimensionen auswerten, daraus die Schlüsselmenge bilden, dann die Faktentabelle **gezielt einschränken**, statt sie zu scannen und danach zu joinen.<br>**Bitmap-Index-Verschneidung** — mehrere Dimensionsfilter werden als Bitmaps verknüpft, **bevor** die Faktentabelle angefasst wird.<br>Eine tiefe Snowflake-Kette macht das Muster unkenntlich — deshalb ist das Kimballs technisch stärkstes Argument gegen Snowflaking.<br>Im Postgres-`EXPLAIN` erkennst du das Muster als `Bitmap Index Scan` gefolgt von `BitmapAnd`.
<!-- basicblock-end -->

# Block 3 — Slowly Changing Dimensions

*Die ersten beiden Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="ObsqSTcl0qrc5vos04SUnALj" -->
Was passiert bei einer **Typ-1**-Änderung, und was ist das Problem daran?::
> Kimball: „you merely **overwrite** the preexisting value with the current assignment. The fact table is untouched. The problem with a type 1 response is that you **lose all history** of attribute changes."<br>Richtiger Anwendungsfall: **Korrektur eines Fehlers** (Tippfehler, falsche PLZ) — hier *will* man, dass die Korrektur rückwirkend gilt, weil der alte Wert nie wahr war.<br>Zwei Nebenwirkungen: Berichte ändern sich **rückwirkend**, und **Aggregattabellen** werden ungültig, wenn das überschriebene Attribut dort als Gruppierung dient.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsN45lXAeEr32x9L26nGn5o" -->
Was ist der **SCD Manager** (Kimball, ETL-Subsystem 9)?::
Das ETL-Subsystem, das die gesamte SCD-Logik kapselt. Kimball: Es behandelt *„the time variance in the dimensions"* systematisch mit den Typ-1-, Typ-2- und Typ-3-Techniken und *„should maintain appropriate housekeeping columns for type 2 changes"*.<br>Zusammen mit **Subsystem 10 (Surrogate Key Generator)** und **Subsystem 14 (Surrogate Key Pipeline)** bildet es die Kette, die aus einem Quell-Snapshot eine historisierte Dimension plus korrekt verschlüsselte Fakten macht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsL2b9YtoUQVxKeYF44uyTj" -->
Warum kann man im Warehouse nicht einfach die Adresse überschreiben wie im Quellsystem?::
Weil beide Fragen legitim sind und nur eine davon nach dem Überschreiben noch beantwortbar wäre:<br>„Wie viel Umsatz im Februar in **Berlin**?" — nach dem **damaligen** Zustand.<br>„Wie viel Umsatz macht dieser Kunde, der heute in **München** wohnt?" — nach dem **heutigen**.<br>Das operative System kennt nur den aktuellen Zustand und überschreibt. Die erste Frage ist deshalb nur beantwortbar, wenn **das Warehouse die Historie selbst führt** — Inmons *time-variant*.<br>„Slowly" heißt dabei: Änderungen kommen **unvorhersehbar und selten**, nicht in einem festen Takt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCEtm0ekPG6lAVz3FJigtT" -->
Wird der SCD-Typ pro Tabelle oder pro Attribut festgelegt?::
**Pro Attribut.** In derselben Kundendimension kann die Adresse **Typ 2** sein (Historie wichtig), der Tippfehler im Nachnamen **Typ 1** (Korrektur soll rückwirkend gelten) und das Geburtsdatum **Typ 0**.<br>Genau deshalb braucht die Änderungserkennung **zwei Hashes** — einen über die Typ-1-Attribute, einen über die Typ-2-Attribute.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNuSMCOisiSoyvwsYhV0vh" -->
Was bedeutet **SCD Typ 0** — und warum ist „es ändert sich nichts" die falsche Erklärung?::
Typ 0 heißt: Der Wert wird bei der ersten Erfassung geschrieben und danach **nie wieder verändert — auch dann nicht, wenn die Quelle einen neuen Wert liefert**. Das Attribut ist als *durable* deklariert.<br>Es ist eine **aktive Entscheidung**, kein fehlendes Ereignis. Wäre Letzteres gemeint, bräuchte man keinen eigenen Typ dafür — ein Attribut, das sich nie ändert, verhält sich unter jedem Typ gleich.<br>Typische Fälle: ursprüngliches Kreditrating bei Kontoeröffnung, Vertriebskanal des Erstkaufs, Registrierungsdatum. Kimball nennt solche Spalten oft `original_*`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsv7dtrL16AC09qNIn745bC" -->
Welche Entscheidungsfrage trennt **Typ 1** von **Typ 2**?::
**War der alte Wert jemals richtig?**<br>**Nein** (Tippfehler, Fehlerfassung) → **Typ 1** überschreiben. Die Historie soll gar nicht erhalten bleiben, denn sie war falsch.<br>**Ja** (Umzug, Umgruppierung, Statuswechsel) → **Typ 2** neue Version. Der alte Wert war zu seiner Zeit korrekt und muss auffindbar bleiben.<br>Diese eine Frage ersetzt fast die gesamte Typ-Entscheidung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPAW0ZlFpV9VbXMDCp0fJO" -->
Welche drei Housekeeping-Spalten braucht eine Typ-2-Dimension?::
**`row_effective_date`** — ab wann diese Version gilt<br>**`row_expiration_date`** — bis wann sie galt (für die aktuelle Version meist `9999-12-31`)<br>**`is_current`** — Kennzeichen der aktuellen Version<br>Dazu kommt die Schlüsseltrennung: Der **natürliche Schlüssel** klammert alle Versionen als dieselbe Entität, der **Surrogatschlüssel** identifiziert **eine Version**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsI8xTryaK82xfRqpdd7nQy" -->
Warum bleibt die Faktentabelle bei einer Typ-2-Änderung unberührt — und warum ist die Auswertung trotzdem historisch korrekt?::
Faktenzeilen von vor der Änderung zeigen weiter auf den **alten** Surrogatschlüssel, neue Fakten auf den neuen. Es gibt keine Migration und keine Nachbearbeitung.<br>Die Zuordnung stimmt damit **automatisch**: Ein simples `GROUP BY city` liefert den Februar-Umsatz unter Berlin und den April-Umsatz unter München, ohne jede Sonderlogik.<br>Merksatz: **Die Historie steckt im Join, nicht in der Query.** Das ist der eigentliche Charme von Typ 2 und der Grund für seine Verbreitung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAz5gMxQSBPEEUbTR31aMj" -->
Nenne die drei Konventionen für „aktuelle Version" mit je einem Vor- und Nachteil::
**`is_current`-Flag** — kürzeste, gut indizierbare Query; aber **redundant** und kann inkonsistent werden.<br>**`row_expiration_date IS NULL`** — keine Redundanz, semantisch ehrlich; aber `NULL` bricht `BETWEEN`-Abfragen und ist ungünstiger im Index.<br>**`row_expiration_date = '9999-12-31'`** — `BETWEEN` funktioniert **ohne Sonderfall**; aber magische Konstante mit Zeitzonenfallen.<br>In der Praxis kombiniert man oft Flag **und** `9999-12-31` und stellt die Redundanz über die Ladelogik sicher.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstD0Q82A8jFDuNGovDYQBO" -->
Warum müssen SCD2-Gültigkeitsintervalle lückenlos **und** überschneidungsfrei sein?::
Weil die Surrogate Key Pipeline über **Schlüssel plus Zeitfenster** joint, nicht über `is_current`.<br>**Eine Lücke** lässt Faktenzeilen ins Leere laufen — sie finden keine Version.<br>**Eine Überschneidung dupliziert sie** — der Join liefert zwei Dimensionszeilen, und die Faktenzeile wird doppelt gezählt.<br>Typische Ursache: `row_expiration_date` der alten Zeile auf denselben Tag setzen wie `row_effective_date` der neuen. Richtig ist der **Vortag** oder ein durchgängig halboffenes Intervall (`>= from AND < to`).<br>In Postgres lässt sich das mit einem `EXCLUDE`-Constraint über `daterange` erzwingen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obse1AWnpt3ZzURRhVBBl0w3" -->
Was ist der **größte praktische** Nachteil von SCD Typ 2 im Alltag?::
**Jede Query, die nur den Ist-Zustand will, muss auf die aktuelle Version filtern.**<br>Vergisst man das, bekommt man jede Entität so oft, wie sie Versionen hat — stillschweigend falsche Zählungen und Summen, **ohne Fehlermeldung**.<br>Weitere Kosten: Die Dimension wächst mit jeder Änderung; bei großen, schnell wechselnden Dimensionen kippt der Ansatz (→ Typ 4); und die ETL-Logik muss Änderungen erkennen und die Gültigkeitszeiträume korrekt fortschreiben.<br>Gegenmaßnahme: eine materialisierte **Aktuell-Sicht** als eigene Tabelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ksOM1k6CHTLExz9X7YE7" -->
Was macht **SCD Typ 3**, und wo sind seine Grenzen?::
Statt einer neuen Zeile bekommt die Dimension eine **zusätzliche Spalte** für den Vorwert (`city` und `prior_city`). Es entsteht **keine** neue Version.<br>Anwendungsfall: eine **Reorganisation**, bei der dieselben Fakten wahlweise nach alter oder neuer Einteilung ausgewertet werden sollen — eine **alternative Sicht auf dieselbe Historie**.<br>Grenzen: **nur eine Vorversion** (beim zweiten Wechsel geht der Ursprungswert verloren) · **Schemaänderung pro Attribut** · **kein Zeitbezug** — es steht nirgends, *wann* gewechselt wurde.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKJjCMVSPQGzETHuBk0vsB" -->
Welche Kombination erzeugt eine **rapidly changing monster dimension**, und wie löst Typ 4 das?::
Die Kombination aus **groß** (z. B. 50 Mio. Kunden) **und schnell wechselnden Attributen** (Segment, Bonitätsstufe, Aktivitätsstatus). Groß allein ist unproblematisch.<br>Unter Typ 2 erzeugt jede Änderung eine komplette neue Zeile **mit allen 80 Spalten** — bei monatlichem Segmentwechsel nach zwei Jahren 1,2 Milliarden Zeilen.<br>**Typ 4** spaltet genau diese Attribute in eine kleine **Mini-Dimension** ab (eine Zeile pro Attribut**kombination**, ähnlich einer Junk Dimension). Die Faktentabelle bekommt dann **zwei Fremdschlüssel** — Basisdimension und Mini-Dimension.<br>Die Historie wandert damit **von der Dimension in die Faktentabelle**. Preis: Ohne Fakt in einem Monat ist das damalige Profil nicht rekonstruierbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMknodh4lyLfjN7W3FBsZ5" -->
Was machen die Hybrid-Typen **5**, **6** und **7** — und welche Frage lösen alle drei?::
Alle drei lösen: **Wie bekommt man gleichzeitig die „damalige" und die „heutige" Sicht?**<br>**Typ 5 = 4 + 1** — die Mini-Dimension wird zusätzlich als **Typ-1-Outrigger** an die Basisdimension gehängt; dort steht damit immer das *aktuelle* Profil, während die Faktentabelle das *damalige* kennt.<br>**Typ 6 = 1 + 2 + 3** — eine Typ-2-Dimension, die zusätzlich **Aktuellwert-Spalten** führt, die bei jeder Änderung über **alle** Versionen der Entität überschrieben werden.<br>**Typ 7** — die Faktentabelle trägt **beide** Schlüssel (Surrogatschlüssel *und* dauerhaften natürlichen Schlüssel); darüber zwei Views auf dieselbe Dimension.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNSkwaDF0Qj3qdRrrZGzHF" -->
Was ist der Nutzen von **Typ 6** gegenüber reinem Typ 2?::
Dieselbe Faktenzeile lässt sich **wahlweise nach dem damaligen oder dem heutigen Attributwert** auswerten, ohne die Query umzubauen.<br>`GROUP BY city` → historisch korrekte Sicht (Typ-2-Spalte)<br>`GROUP BY current_city` → alles dem heutigen Wert zugerechnet (Typ-1-Spalte)<br>Bei reinem Typ 2 wäre Letzteres nur über einen **zusätzlichen Join** auf die aktuelle Version möglich.<br>Preis: redundante Spalten, die bei jeder Änderung über alle Versionen hinweg gepflegt werden müssen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdokbwMthnmeMLXZMWyhLP" -->
Wie funktioniert **Typ 7**, und was ist sein Vorteil gegenüber Typ 6?::
Die Faktentabelle trägt **beide** Schlüssel: den **Surrogatschlüssel** (zeigt auf die damalige Version) und den **dauerhaften natürlichen Schlüssel** (zeigt auf die Entität). Darüber legt man zwei Views auf dieselbe Dimensionstabelle:<br>**Typ-2-Sicht** über den Surrogatschlüssel → „as was"<br>**Typ-1-Sicht**, gefiltert auf `is_current` und gejoint über den natürlichen Schlüssel → „as is"<br>**Vorteil gegenüber Typ 6:** keine redundanten Aktuellwert-Spalten und keine Pflege über alle Versionen hinweg.<br>**Nachteil:** eine Spalte mehr in der Faktentabelle und zwei Views, die Fachanwender auseinanderhalten müssen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfC2I50CwzP3WU5CjRTn3d" -->
In welche vier Fälle klassifiziert die Pipeline eine Zeile aus einem Vollsnapshot?::
**Neue Entität** — natürlicher Schlüssel nicht in der Dimension → einfügen, `effective = heute`, `is_current = true`<br>**Unverändert** — alle Attribute identisch → nichts tun<br>**Typ-1-Attribut geändert** → Werte **überschreiben**, keine neue Zeile<br>**Typ-2-Attribut geändert** → alte Zeile schließen, neue Version einfügen<br>Dazu der fünfte Fall, der keine Zeile im Snapshot hat: **Entität fehlt** → Delete-Behandlung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrUmjoJPspteXyDG8B7Kze" -->
Warum braucht die Änderungserkennung **zwei** Hashes statt einem?::
Weil Typ-1- und Typ-2-Änderungen **unterschiedlich behandelt** werden müssen.<br>`hash_type1` über die Typ-1-Attribute, `hash_type2` über die Typ-2-Attribute. Die Logik wird damit trivial:<br>`hash_type1` weicht ab → **UPDATE** der aktuellen Zeile<br>`hash_type2` weicht ab → alte Zeile schließen, **neue Version** einfügen<br>Mit **einem** Hash über alle Attribute könnte die Pipeline die beiden Fälle nicht unterscheiden — und würde bei **jeder Tippfehlerkorrektur** eine neue Typ-2-Version anlegen. Die Dimension füllt sich dann mit Versionen, die fachlich nichts bedeuten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFO1Cq4mjfvicdgcUnnEke" -->
Was ist der wichtigste Haken an Hash-Spalten zur Änderungserkennung?::
**Falsch-positive Änderungen.** Der Hash reagiert auf **jede byteweise Abweichung**: geänderte Spaltenreihenfolge in der Verkettung, Zahlenformat (`1.50` vs. `1.5`), NULL-Behandlung (`NULL` vs. leerer String), ein Leerzeichen am Ende. Die Pipeline legt dann eine neue Version an, obwohl fachlich nichts passiert ist.<br>**Gegenmaßnahme:** vor dem Hashen konsequent normalisieren — `COALESCE` auf einen festen Platzhalter, `TRIM`, explizite Casts, feste Spaltenreihenfolge.<br>Weiter: Kommt eine Spalte hinzu, sind **alle** bestehenden Hashes ungültig. Und beim Debuggen sieht man dem Hash nicht an, *was* sich geändert hat — dafür ist Kimballs Alternative `EXCEPT` lesbarer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslUGP0Q2kYmW7ntjql3q9P" -->
Wie findet die **Surrogate Key Pipeline** die zeitlich passende Dimensionsversion?::
Über einen Join auf **natürlichen Schlüssel plus Zeitfenster** — nicht über `is_current`:<br>`ON d.customer_id = f.customer_id AND f.event_date >= d.row_effective_date AND f.event_date <= d.row_expiration_date`<br>Findet sich keine Version, greift der `LEFT JOIN` und der Schlüssel bleibt leer. Dann wird auf die **„unbekannt"-Zeile** der Dimension zurückgefallen und der Fall **protokolliert** — niemals `NULL` in den Fremdschlüssel schreiben.<br>Genau dieser Join löst auch **late-arriving facts** automatisch korrekt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdipzgG7G7sK4w8wRxjJdy" -->
Wie erkennt man in einem **Vollsnapshot** eine gelöschte Zeile — und warum ist Abwesenheit kein Beweis?::
Per **Anti-Join**: Entitäten, die in der Dimension `is_current` sind, aber im heutigen Snapshot fehlen (`NOT EXISTS`).<br>**Aber:** Eine fehlende Zeile kann auch heißen — Extraktion abgebrochen, Export gefiltert, Quelle ausgefallen, Datei unvollständig übertragen.<br>**Ohne Vollständigkeitsprüfung löscht ein halb übertragener Snapshot die halbe Dimension** — und zwar leise, denn technisch lief alles fehlerfrei.<br>Minimale Absicherung: Zeilenzahl gegen den Vortag prüfen und bei Abweichung über einem Schwellwert (etwa 10 %) **abbrechen** statt verarbeiten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskeY3Wcm8ZtpAt6qlSWbHl" -->
Nenne die drei Umgangsweisen mit einem erkannten Delete und wann welche richtig ist::
**Ignorieren** — die Zeile bleibt für immer `is_current`. Fast nie richtig; die Dimension behauptet dauerhaft, es gebe die Entität noch.<br>**Invalidieren (soft close)** — `row_expiration_date` auf den Erkennungstag, `is_current = false`, **keine** neue Zeile. Danach hat die Entität **gar keine** aktuelle Version. Richtig, wenn nur interessiert, *dass* es sie nicht mehr gibt.<br>**Tombstone (neue Zeile)** — neue Version mit `is_deleted = true`, `is_current = true`, offenes Enddatum. Richtig, wenn der Zustand „gelöscht" **abfragbar** sein soll („wie viele Kunden wurden dieses Quartal deaktiviert?").<br>Der Unterschied ist praktisch bedeutsam: Beim Invalidieren verschwindet die Entität aus jeder `WHERE is_current`-Query, beim Tombstone bleibt sie sichtbar und markiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjOhMHxK6h6i5M8RqPkOOA" -->
Warum ist im Warehouse fast immer der **Soft Delete** richtig? Drei Gründe::
**1. Nonvolatile** — ein Warehouse löscht per Definition nicht (Inmon). Was einmal wahr war, bleibt auffindbar.<br>**2. Verwaiste Fakten** — Faktenzeilen zeigen auf die Dimensionsversion. Physisch gelöscht, zeigen Milliarden Fakten ins Leere. Und weil moderne Warehouses Fremdschlüssel **nicht erzwingen**, merkt es niemand, bis ein Join stillschweigend Zeilen verliert.<br>**3. Die Löschung ist selbst eine Information** — „wann wurde dieser Kunde deaktiviert?" ist eine legitime analytische Frage. Ein Hard Delete vernichtet sie.<br>**Ausnahme:** Löschpflichten (DSGVO). Dann physisch löschen oder anonymisieren — über einen eigenen, dokumentierten Prozess **außerhalb** der normalen Pipeline.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDVQdIy61LmsAqgB3C6usH" -->
Was ist der Unterschied zwischen **late-arriving facts** und **late-arriving dimensions**?::
**Late-arriving Fact** — ein Fakt trifft mit einem Ereignisdatum ein, das Wochen zurückliegt. Der Zeitfenster-Join der Surrogate Key Pipeline löst das **automatisch korrekt**. Genau deshalb joint man über das Zeitfenster und nicht über `is_current`.<br>**Late-arriving Dimension** — die Information über eine Änderung trifft verspätet ein, nachdem bereits Fakten geladen wurden. Korrekt wäre: neue Version **rückwirkend** einfügen, alte rückwirkend schließen, und **alle betroffenen Faktenzeilen umhängen**.<br>Schritt drei ist teuer und verletzt die Unveränderlichkeit der Fakten. Kimball behandelt das als eigenes ETL-Subsystem (*Late Arriving Data Handler*, Subsystem 16); in der Praxis entscheidet man bewusst, ob man korrigiert oder die Zuordnung als „Stand der Kenntnis" dokumentiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3rFCimNGbZ6XQappofjPX" -->
Was ist der Unterschied zwischen **valid time** und **transaction time**?::
**Valid Time** (Gültigkeitszeit) — wann war der Wert **in der Welt** wahr? Der Kunde ist am 15. März umgezogen.<br>**Transaction Time** (Erfassungszeit) — wann hat **das Warehouse** davon erfahren? Am 2. Mai.<br>Eine gewöhnliche SCD-Typ-2-Tabelle hat **ein** Datumspaar und **vermischt beide Achsen**. Bitemporale Modellierung führt beide Paare (`valid_from/valid_to` und `recorded_from/recorded_to`).<br>Der `todays_date`-Trick aus dem Leitfaden simuliert genau die **Transaktionszeit**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsT8Mr4QmT2snKhOYNuVpos" -->
Welche zwei Probleme entstehen, wenn valid time und transaction time vermischt werden?::
**1. Eine Korrektur sieht aus wie eine Änderung.** Stand im Januar versehentlich „Berlin" statt „Bremen" und wird das im März korrigiert, erzeugt eine naive Typ-2-Logik eine neue Version ab März. Die Tabelle behauptet dann, der Kunde sei im März umgezogen — er hat aber immer in Bremen gewohnt. **Die Historie ist erfunden.** Richtig wäre Typ 1, aber die Pipeline kann die Fälle nicht unterscheiden, weil beide als „Wert hat sich geändert" ankommen.<br>**2. Reports sind nicht reproduzierbar.** „Gib mir den Bericht so, wie er am 1. April aussah" ist mit einer Zeitachse **nicht** beantwortbar. Man kann rekonstruieren, was damals *galt* — aber nicht, was man damals *wusste*.<br>Deshalb bitemporal vor allem in Finanz-, Versicherungs- und regulierten Umgebungen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obshp57jq2c7cDx57yzZJ6Ch" -->
Wie korrigiert man einen Fakt nachträglich? Drei Wege, einer davon falsch::
**In-place-Update** (`UPDATE fct_sales SET ...`) — **vermeiden.** Zerstört die Prüfbarkeit und macht den Lauf nicht wiederholbar.<br>**Gegenbuchung** — neue Zeile mit **negativen** Kennzahlen, die die alte neutralisiert, danach ggf. die korrekte Zeile. Buchhalterisch sauber; die Historie zeigt Buchung *und* Storno. Funktioniert, weil additive Fakten sich aufheben.<br>**Partition neu schreiben** — den betroffenen Tag löschen und aus der Quelle neu laden. Der **Standard in modernen Warehouses** (`INSERT OVERWRITE` / `MERGE`).<br>Wahl: Gegenbuchung, wenn der **Vorgang** dokumentiert werden soll (Storno ist ein Geschäftsereignis). Neuschreiben, wenn es eine **Datenkorrektur** ist und der Fehler verschwinden soll.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsoazKKille53MvnBZCmthT" -->
Warum partitioniert man eine SCD2-Tabelle nach `row_effective_date` und **nicht** nach `is_current`?::
Weil `row_expiration_date` und `is_current` auf **bestehenden Zeilen geändert** werden. In Delta Lake, Iceberg oder BigQuery gibt es kein In-place-Update — Dateien müssen neu geschrieben werden.<br>**Nach `row_effective_date`**: Der Wert einer Zeile ändert sich **nie**, die Zeile bleibt für immer in ihrer Partition. Gut.<br>**Nach `row_expiration_date` oder `is_current`**: Beim Schließen einer Version **wandert die Zeile in eine andere Partition** — Löschen plus Einfügen, der teuerste denkbare Fall.<br>Verbreitete Ergänzung: eine separat materialisierte **Aktuell-Sicht**, die bei jedem Lauf neu aufgebaut wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs519KFFTTFBxgWvbtttAly" -->
Welche Meta-Spalten legt **dbt** bei Snapshots an, und wie heißen sie bei Kimball?::
`dbt_valid_from` → Kimballs **`row_effective_date`**<br>`dbt_valid_to` → **`row_expiration_date`**; **`NULL` für die aktuelle Version** (konfigurierbar auf ein Datum wie `9999-12-31`)<br>`dbt_scd_id` → der **Surrogatschlüssel**, eindeutig **pro Version**<br>`dbt_updated_at` → der Zeitstempel, aus dem die Gültigkeit abgeleitet wurde<br>`dbt_is_deleted` → Markierung für gelöschte Sätze, wenn die Delete-Behandlung aktiv ist<br>Die Zuordnung ist eins zu eins — es sind dieselben Housekeeping-Spalten, nur anders benannt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCdxn6RzubtpfIkw7KxEqg" -->
dbt-Snapshots: Was ist der Unterschied zwischen der **timestamp**- und der **check**-Strategie?::
**`timestamp`** (empfohlen) — setzt eine verlässliche `updated_at`-Spalte in der Quelle voraus. Ist deren Wert neuer als der gespeicherte, gilt der Satz als geändert. Billig: ein Spaltenvergleich, unabhängig von der Attributzahl.<br>**`check`** — der Rückfallweg **ohne** verlässlichen Zeitstempel. Man gibt Spalten an (`check_cols`, oder `all`), die bei jedem Lauf verglichen werden. Teurer, aber unabhängig von der Quelldisziplin. Konzeptionell dasselbe wie Kimballs Hash-Vergleich — mit derselben Schwäche, dass Formatierungsänderungen falsch-positive Versionen erzeugen.<br>**Entscheidung in einem Satz:** `timestamp`, wenn man der Quelle traut, sonst `check`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsH4USZwimA5jUACBOm11pT" -->
Warum ignorieren dbt-Snapshots das `--full-refresh`-Flag?::
Weil ein Neuaufbau **unwiederbringlicher Datenverlust** wäre. Die Historie existiert **nur in der Snapshot-Tabelle selbst**, nicht in der Quelle — das Quellsystem hat die alten Werte überschrieben.<br>Ein Full Refresh würde alles bis auf den heutigen Zustand vernichten, und es gäbe nichts, woraus man es rekonstruieren könnte.<br>Das ist keine Bequemlichkeitsentscheidung, sondern eine **Schutzmaßnahme** — und die direkte Folge daraus, dass eine SCD2-Tabelle ein System of Record ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNBKYyakVibTYgLWbXkddr" -->
Ist eine SCD2-Dimensionstabelle **System of Record** oder **Derived Data** — und welche Folgen hat das?::
**System of Record** — für die Historie. Sie enthält Werte, die die Quelle überschrieben hat; sie ist **nicht aus der Quelle ableitbar**.<br>Folgen: **1.** Sichern wie ein Quellsystem (Backup, Aufbewahrung, Wiederherstellungstest). **2.** **Kein Drop-and-Reload** — die Standardantwort „einfach neu laden" ist hier nicht verfügbar. **3.** Die **goldene Regel gilt hier nicht** — die SCD2-Tabelle ist die dokumentierte Ausnahme. **4.** `--full-refresh` ist eine Waffe; deshalb ignoriert dbt es.<br>**Der Ausweg:** eine **Persistent Staging Area**, die jeden Roh-Snapshot dauerhaft aufbewahrt. Dann *ist* die SCD2-Tabelle wieder Derived Data und die goldene Regel gilt wieder. Preis ist Speicher.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIShHfCBuuLA8SfkKUjhzY" -->
In welchem Sinn ist ein SCD2-Snapshot-Lauf idempotent und in welchem nicht?::
**Innerhalb desselben Quellzustands: ja.** Läuft der Job zweimal am selben Tag gegen dieselbe Quelle, ändert der zweite Lauf nichts — es gibt keine Abweichung zu erkennen.<br>**Im Sinne von „aus der Quelle rekonstruierbar": nein.** Die Tabelle ist **pfadabhängig** — ihr Inhalt hängt davon ab, wann welcher Lauf welchen Quellzustand gesehen hat. Zwei verschiedene Laufhistorien über dieselbe Quelle erzeugen verschiedene Tabellen.<br>Das ist ein Idempotenzbegriff, der in Kapitel 2.1 nicht vorkam: Eine Transformation aus **unveränderlichem** Input ist immer rekonstruierbar. Eine SCD2-Tabelle beobachtet dagegen einen **veränderlichen** Input über die Zeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswyXBtZdBOAGAIhPMtyDA8" -->
⭐ Warum ist SCD Typ 2 der am weitesten verbreitete Typ?::
**Der einzige Grundtyp mit vollständiger Historie** — Typ 1 vernichtet sie, Typ 3 bewahrt genau eine Vorversion und braucht pro Attribut eine Spalte.<br>**Die Zuordnung passiert automatisch richtig** — die Faktenzeile zeigt auf die zum Ereigniszeitpunkt gültige Version, ein simples `GROUP BY` liefert die historisch korrekte Auswertung **ohne Sonderlogik in der Query**. Kein anderer Typ leistet das so beiläufig.<br>**Er passt zur Natur analytischer Systeme** — direkte Umsetzung von Inmons *time-variant* und *nonvolatile*.<br>**Er ist einfach genug** — drei Spalten und eine überschaubare Ladelogik; die Typen 4–7 lösen Spezialprobleme und kosten deutlich mehr Komplexität.
<!-- basicblock-end -->

# Block 4 — Data Vault, OBT & Kimball vs. Inmon

*Die ersten zehn Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obs1wbgKbPLvJ9lGdq58CY51" -->
Was meint Inmon mit dem **„single physical corporate image"** — und warum ist Integration für ihn die wichtigste Eigenschaft?::
> „Of all the aspects of a data warehouse, **integration is the most important**. Data is fed from multiple, disparate sources into the data warehouse. As the data is fed, it is converted, reformatted, resequenced, summarized, etc. The result is that data — once it resides in the data warehouse — has a **single physical corporate image**."<br>Die Kernforderung: **Ein Faktum existiert im Unternehmen genau einmal, an einer Stelle, in einer Form.** Alles andere wird davon abgeleitet.<br>Daraus folgt die ganze Top-down-Architektur: erst die integrierte 3NF-Kernschicht, dann die Marts.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2gTCX8ANUTDAdZjcvGCrC" -->
Warum sagt Inmon „A data mart is never a substitute for a data warehouse"?::
Weil ein Mart eine **Sicht** ist, kein **Fundament**. Er ist für einen Fachbereich und einen Zweck gebaut und enthält per Konstruktion nur einen Ausschnitt.<br>Wer nur Marts baut, hat kein Warehouse — er hat **mehrere unverbundene Auswertungsdatenbanken**, die sich widersprechen können, weil es keine gemeinsame Wahrheit darunter gibt.<br>Das ist Inmons zentrales Argument gegen den Kimball-Ansatz. Kimballs Gegenmittel ist nicht eine Kernschicht, sondern die **Bus-Architektur mit conformed dimensions**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxfZJZzaVlcBA2pYaJ5812" -->
„Redundant data is OK with the Kimball method, but …" — wie geht der Satz weiter, und warum ist das die ganze Kimball-Disziplin?::
> „… but **avoid replicating the same dimension tables** to avoid drifting business definitions and data integrity."<br>Kimball erlaubt Redundanz in den **Daten** — denormalisierte Dimensionen, wiederholte Textwerte. Er verbietet Redundanz in den **Definitionen**.<br>Deshalb ist eine Dimension, die in zwei Fachbereichen unabhängig gepflegt wird, der schwerste Fehler im Kimball-Modell: Die Daten dürfen doppelt sein, die Bedeutung nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscZtYBZTNMS9y5SqhlcvM2" -->
Was ist der Grundgedanke von **Data Vault** — was genau wird getrennt, und warum?::
> „the Data Vault methodology **separates the structural aspects of a source system's data from its attributes**."<br>Getrennt werden **Schlüssel** (Hub), **Beziehungen** (Link) und **beschreibende Attribute** (Satellite) — weil sie sich **unterschiedlich schnell ändern**:<br>Geschäftsschlüssel sind extrem stabil (eine Kundennummer überlebt Jahrzehnte) · Beziehungen ändern sich gelegentlich · Attribute ändern sich ständig.<br>Wer alles in eine Tabelle packt, macht das Änderungstempo des **schnellsten** Teils zum Tempo des Ganzen. Data Vault entkoppelt sie.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsp1HLjXpd9Q39UJ77AMJPY" -->
Welche vier Standardfelder enthält ein **Hub** immer?::
**Hash Key** — der Primärschlüssel, deterministisch berechnet aus dem Business Key (MD5 o. ä.)<br>**Business Key** — der Schlüssel, unter dem Fachanwender das Objekt kennen<br>**Load Date** — wann der Satz ins Warehouse geladen wurde<br>**Record Source** — aus welchem Quellsystem er stammt<br>Eine Zeile pro **jemals gesehenem** Geschäftsschlüssel. Nie geändert, nie gelöscht — nur eingefügt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPdvV7LY9gL9xUIgjKE3sD" -->
Woran erkennt man einen guten **Business Key** für einen Hub?::
> Linstedt: „Wherever a business user can **search for a business object with a reference code**, there is a good chance that the business user is using business keys for that purpose."<br>Der Geschäftsschlüssel ist das, **wonach Menschen suchen** — Kundennummer, Bestellnummer, Artikelnummer. **Nicht** der technische Primärschlüssel des Quellsystems.<br>Der Unterschied ist zentral: Ein technischer Schlüssel gehört zu **einem System**, ein Geschäftsschlüssel gehört dem **Unternehmen** — und ist damit das, worüber sich Systeme integrieren lassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrTLLMwn6S2AEBIYwpJgY1" -->
Warum sind **Links** im Data Vault immer many-to-many, auch wenn die Quelle 1:n ist?::
Weil genau darin die Flexibilität liegt. Ändert sich die Kardinalität — aus „ein Kunde hat eine Adresse" wird „ein Kunde hat mehrere" —, muss im Data Vault **gar nichts** geändert werden.<br>Bei Kimball wäre dasselbe ein Umbau: Bridge Table oder anderer Grain, und beides betrifft bestehende Fakten.<br>Dazu die zweite Eigenschaft: Links enthalten **keine Attribute**. Alles Beschreibende hängt in Satellites am Link.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrGcM4r2ed4mkWISKXtFUs" -->
Wie reagiert ein Data Vault auf eine neue Beziehung zwischen bestehenden Entitäten?::
> „The Data Vault model's relationships are straightforward and handled through changes to the links. This provides excellent flexibility in the inevitable event that the underlying data changes. **You simply create a new link** that ties business concepts (or hubs) to represent the new relationship. **That's it.**"<br>Und allgemeiner, Linstedt zum Änderungsaufwand: „The existing model has not changed. Therefore, the **ETL jobs that depend on it do not need to be refactored**. The impact of the change to the existing data warehouse is **zero**."
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyQ3KvdbuJwh5UTahmYGRP" -->
Nach welchen **drei Kriterien** teilt man Satellites auf?::
**Nach Quellsystem** — liefern CRM und Shop beide eine Adresse, bekommt jedes System seinen eigenen Satellite. Es wird **nicht** entschieden, welche richtig ist.<br>**Nach Änderungsrate** — Attribute, die sich stündlich ändern, sollen nicht die mitschleppen, die sich jährlich ändern. (Dieselbe Idee wie Kimballs Typ-4-Mini-Dimension.)<br>**Nach Klassifikation oder Datentyp** — etwa große Textblobs getrennt von Kennzahlen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs32bm9cDpaHZEvI3G7gLpw" -->
Woher stammt die Grundidee des Data-Vault-Modells?::
Von **komplexen Netzwerken**, wie sie in der Natur vorkommen — „structures consisting of **nodes or vertices connected by links or edges**".<br>Ein Data Vault ist strukturell näher an einem **Graphen** als an einem klassischen relationalen Schema: **Hubs sind die Knoten, Links die Kanten, Satellites die Beschriftungen.**<br>Linstedts Ziel dahinter: das Geschäft so genau wie möglich abbilden — mit Agilität, Integration mehrerer Quellen, Transparenz gegenüber dem Auditor und Skalierbarkeit als Leitkriterien.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIeIBIomVloAof5VRX8Uer" -->
Was optimieren **Inmon**, **Kimball** und **Data Vault** jeweils?::
**Inmon** — **Konsistenz**. Zuerst eine integrierte Unternehmensschicht in 3NF, daraus abgeleitete Marts. Top-down.<br>**Kimball** — **Verständlichkeit und Time-to-Value**. Zuerst ein Geschäftsprozess als dimensionaler Mart. Bottom-up.<br>**Data Vault** — **Veränderbarkeit und Nachvollziehbarkeit**. Zuerst eine auditierbare Integrationsschicht aus Hubs, Links, Satellites.<br>Wichtig: Inmon und Data Vault modellieren die **Integrationsschicht**, Kimball die **Präsentationsschicht**. Sie sind nur teilweise Alternativen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAJDN63yxnvBOLdow6YkcV" -->
Was ist bei **Kimball** der Integrationsmechanismus, wenn es keine normalisierte Kernschicht gibt?::
Die **Bus-Architektur mit conformed dimensions**.<br>Bei Kimball **sind die Marts das Warehouse** — es gibt keine darunterliegende Normalschicht. Die Präsentationsschicht ist die Wahrheit: atomar, dimensional, direkt abfragbar.<br>Was verhindert, dass daraus unverbundene Inseln werden, ist **Konvention statt Struktur**: Dimensionen werden einmal definiert und über Prozesse hinweg wiederverwendet.<br>Das ist gleichzeitig die Schwäche — konformiert wird, **weil man es vereinbart**, nicht weil das Schema es erzwingt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3JFmu1lzNDXx8SKe6Cclr" -->
Kimball vs. Inmon — nenne die wichtigsten Unterschiede in einer Gegenüberstellung::
**Richtung** — Inmon top-down, Kimball bottom-up<br>**Kernschicht** — Inmon EDW in 3NF; Kimball keine, die Marts *sind* das Warehouse<br>**Integration über** — Inmon **Struktur**, Kimball **Konvention** (conformed dimensions)<br>**Erster Nutzen nach** — Inmon Monaten bis Jahren, Kimball Wochen<br>**Redundanz** — Inmon minimal, Kimball bewusst hoch<br>**Für Fachanwender direkt nutzbar** — Inmon nein, Kimball ja
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0JyqBFUUJwKbvOL9Lsalg" -->
Welcher Ansatz hat sich durchgesetzt — und warum ist die Frage schief gestellt?::
In der Breite **Kimball**: schnellere Lieferung, niedrigeres Anfangsinvestment, direkte Verständlichkeit. Dazu ein Faktor, den die Bücher der 2000er nicht kannten — **Cloud-Warehouses haben den Preis von Redundanz drastisch gesenkt** und damit Inmons stärkstes Argument entwertet.<br>**Schief ist die Frage**, weil sich in der Praxis eine **Hybridarchitektur** durchgesetzt hat:<br>`Quellen → Staging/Raw → Integrationsschicht (3NF oder Data Vault) → Präsentationsschicht (dimensional) → BI`<br>Die Integrationsschicht ist Inmons oder Linstedts Beitrag, die Präsentationsschicht Kimballs. Die vermeintlichen Konkurrenten sitzen in **verschiedenen Schichten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIdiLMmTr6yOGA0tHmZglR" -->
Was ist der **eigentliche** Grund für Hash Keys im Data Vault — jenseits schnellerer Joins?::
**Sie ermöglichen paralleles Laden.**<br>Weil der Hash **deterministisch aus dem Business Key** berechnet wird, kann **jede Tabelle ihn selbst ausrechnen** — ohne vorher irgendwo nachzuschlagen. Hubs, Links und Satellites lassen sich **alle gleichzeitig** laden.<br>Bei Kimball ist das anders: Die Surrogate Key Pipeline muss den Surrogatschlüssel in der Dimension **nachschlagen** — also müssen Dimensionen **vor** den Fakten geladen sein. Das erzwingt eine Reihenfolge.<br>Bei hunderten Tabellen ist das der Unterschied zwischen einem achtstündigen und einem einstündigen Ladefenster.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZP85W6H709c2pT2CgU3x2" -->
Was haben **Record Source** und **Load Date** mit Auditierbarkeit zu tun?::
Zusammen beantworten sie jederzeit: **Woher stammt dieser Wert, und seit wann wissen wir ihn?**<br>Damit kann man nicht nur sagen, was in einem Bericht stand, sondern auch, **auf welcher Quelle er beruhte** — der Grund, warum Data Vault in regulierten Branchen (Banken, Versicherungen, Pharma) verbreitet ist.<br>Zweiter Nutzen: Kommt derselbe Geschäftsschlüssel in mehreren Quellsystemen vor, unterscheidet der Record Source die Herkunft. Das Kollisionsproblem, das bei Kimball der Surrogatschlüssel **auflöst**, wird hier **dokumentiert**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9e85EDCTFnQKu3VYAkN7P" -->
Was unterscheidet **Raw Vault** und **Business Vault** — und welcher ist System of Record?::
**Raw Vault** — die Daten **wie geliefert**. Nur *hard rules* (technische Umformungen, die nichts interpretieren). Keine Geschäftsregeln, keine Bereinigung. Widersprüchliche Angaben aus zwei Quellen stehen nebeneinander, jede mit ihrem Record Source. → **System of Record, unantastbar.**<br>**Business Vault** — die **abgeleitete** Schicht darüber. *Soft rules*: berechnete Satellites, vereinheitlichte Schlüssel, Geschäftslogik, Qualitätsregeln. Hier fällt die Entscheidung, welcher Wert gilt. → **Derived Data, jederzeit neu berechenbar.**<br>Praktischer Nutzen: **Ändert sich eine Geschäftsregel, wirft man den Business Vault weg und baut ihn neu.** Der Raw Vault bleibt unberührt, weil er nie eine Meinung enthielt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3PE7olwxSmsgIGCcmdVve" -->
Wie ordnen sich Data-Vault-Bausteine und Kimball-Konzepte einander zu?::
**Hub** ≈ der **dauerhafte natürliche Schlüssel** der Dimension<br>**Satellite** ≈ die **SCD-Typ-2-Attributhistorie** dieser Dimension<br>**Link** ≈ die **Fremdschlüsselbeziehung**, die eine Faktentabelle herstellt<br>Merksatz: **Ein Satellite ist SCD Typ 2, angewendet auf eine Gruppe von Attributen eines Geschäftsschlüssels.**<br>Achtung: Ein Satellite hängt an einem **Hub oder Link** — nie an einer Faktentabelle. Einer Faktentabelle entspricht am ehesten ein **Link mit Satellite** (Beziehung plus Kennzahlen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPZRaC1NhoiPYMfn9fTpkK" -->
Nenne vier strukturelle Unterschiede zwischen einem **Satellite** und einer **SCD2-Dimension**::
**1. Insert-only vs. Update** — SCD2 schließt die alte Zeile per `UPDATE` auf `valid_to`. Ein Satellite ändert **nie** eine bestehende Zeile; die aktuelle Version ist die mit dem höchsten `load_date`. Das macht das Laden parallelisierbar, die Abfrage aber aufwendiger.<br>**2. Mehrere Satellites pro Entität** — eine Dimension ist **eine** Tabelle; ein Hub kann beliebig viele Satellites haben.<br>**3. Keine Konfliktauflösung** — eine Dimension hat **einen** Wert, irgendwer hat entschieden. Der Raw Vault hat **zwei Satellites** und entscheidet nicht.<br>**4. Kein Surrogatschlüssel pro Version** — bei SCD2 zeigt die Faktenzeile auf **eine Version**; ein Link zeigt auf den **Hub**, also auf die Entität. Die zeitliche Zuordnung passiert erst bei der Abfrage.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsL5pikhzOioebZ9L8NHPAj" -->
Welche vier Änderungen macht Data Vault billig — und was kosten sie bei Kimball?::
**Neues Quellsystem** — DV: neuer Satellite am bestehenden Hub. Kimball: Integration in die Dimension, Schlüssel mappen, Konflikte auflösen, alle ETL-Strecken betroffen.<br>**Neues Attribut** — DV: neuer Satellite oder neue Spalte. Kimball: neue Spalte plus Entscheidung Typ 1 oder Typ 2.<br>**Neue Beziehung** — DV: neuer Link, „that's it". Kimball: evtl. neue Faktentabelle oder neuer Grain.<br>**Kardinalitätsänderung** (1:n wird n:m) — DV: **gar nichts**, Links sind ohnehin many-to-many. Kimball: **Bridge Table oder neuer Grain** — der stärkste Einzelfall.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTw1MDeiuRHRhY5LFCtPqH" -->
Wozu dienen **PIT-** und **Bridge-Tabellen** im Data Vault?::
Sie lösen die **Query-Komplexität**. Eine Abfrage muss für jede Entität die zeitlich passenden Zeilen aus mehreren Satellites zusammensuchen — bei drei Satellites drei Bereichs-Joins, und das pro Hub im Pfad.<br>**PIT (Point-In-Time)** — pro Hub eine Tabelle, die für jeden relevanten Zeitpunkt vorab ausrechnet, welche Satellite-Zeile dann gültig war. Die Bereichs-Joins werden zu Gleichheits-Joins.<br>**Bridge** — verkürzt Join-Pfade über mehrere Hubs und Links hinweg. *(Nicht zu verwechseln mit Kimballs Bridge Table für mehrwertige Dimensionen — gleicher Name, anderes Problem.)*<br>Beide liegen im **Business Vault**, enthalten keine neue Information und werden bei Bedarf verworfen und neu gebaut.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOFx4wJPpJgbvZvYamR9mm" -->
Wie leitet man aus einem Data Vault ein **Star Schema** ab?::
**Hub + seine Satellites** → **Dimensionstabelle** (flach gejoint, denormalisiert)<br>**Link + sein Satellite** → **Faktentabelle**<br>**Load Dates der Satellites** → **SCD-Typ-2-Gültigkeitszeiträume**<br>Damit zerfällt die Frage „Kimball **oder** Data Vault": In einem Haus mit Data Vault gibt es **beides** — DV als integrierte, auditierbare Wahrheit, dimensionale Marts als Nutzeroberfläche darauf.<br>Ein Data Vault ist **nicht dafür gemacht, abgefragt zu werden**. Kein Analyst schreibt Queries gegen Hubs und Satellites.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyiqZtuEh8SmOtscLFT4Ht" -->
Nenne vier Kritikpunkte an **Data Vault**::
**Tabellenexplosion** — ein Quellsystem mit 20 Tabellen wird leicht zu **60–100** Tabellen, mit entsprechend vielen ETL-Strecken, Tests und Monitoring-Punkten.<br>**Query-Komplexität** — was im Star Schema ein Join ist, wird zur Kette Hub → Link → Hub → Satellite plus Zeitlogik. Braucht PIT-/Bridge-Tabellen, die selbst gepflegt werden wollen.<br>**Steile Lernkurve** — ohne Vorwissen nicht lesbar; die meisten produktiven Data Vaults werden **generiert**, nicht von Hand gebaut. Ohne Automatisierung kaum tragbar.<br>**Kein Nutzen für Fachanwender** — liefert für sich genommen **keinen einzigen** benutzbaren Bericht. Man baut zwei Schichten, bevor jemand etwas sieht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsChqYVOW8aRYSzNRvreJ9t" -->
Wann ist **Data Vault Overkill**?::
Es lohnt sich, wenn **mehrere** zutreffen: viele Quellsysteme mit überlappenden Entitäten · häufig wechselnde Quellschemata · Auditpflicht/regulierte Branche · Historie **aller** Rohdaten nötig · großes Team, langfristige Plattform · Automatisierung vorhanden.<br>**Overkill** bei: ein bis drei Quellen · stabile Schemata · keine Nachweispflicht · Historie nur für einzelne Dimensionen · kleines Team · alles von Hand.<br>Kurzfassung: **Bei wenigen stabilen Quellen und ohne Auditpflicht ist Kimball direkt aus dem Staging einfacher, schneller und billiger.** Data Vault zahlt sich über Zeit und Quellenzahl aus — nicht am ersten Tag.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3BgiA9Bh1hpU93aqFWGKS" -->
Was ist die **eigentliche Entscheidungsfrage** zwischen Kimball und Data Vault?::
**Nicht „welches von beiden", sondern: „Brauche ich überhaupt eine eigene Integrationsschicht?"**<br>**Ja** → Data Vault ist eine Option dafür (die andere ist Inmons 3NF) — und Kimball kommt trotzdem obendrauf als Präsentationsschicht.<br>**Nein** → man baut Kimball direkt aus dem Staging.<br>Die beiden konkurrieren nicht um dieselbe Schicht: Data Vault ist Integration (Zielgruppe ETL-Entwickler und Auditoren), Kimball ist Präsentation (Zielgruppe Analysten und BI-Werkzeuge).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsERXsBORYSx25ztUnci2JB" -->
Was ist **OBT (One Big Table)**, und warum ist es heute überhaupt praktikabel?::
Eine **einzige breite Tabelle**, in der Fakten und alle zugehörigen Dimensionsattribute nebeneinanderstehen. Keine Aufteilung in Fakten- und Dimensionstabellen, keine Joins zur Abfragezeit. `brand = 'Coca-Cola'` steht in **jeder** Verkaufszeile.<br>Praktikabel ist es nur durch **Spaltenspeicherung**: Die tausendfach wiederholte Textspalte ist **niedrigkardinal** und kostet durch Dictionary Encoding und RLE fast nichts. In einem Row Store wäre OBT unbezahlbar.<br>**Terminologiefalle:** In „Fundamentals of Data Engineering" heißt das Muster nicht OBT, sondern **wide denormalized tables**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsu4Kwr3sPsYAKXoUtGCaif" -->
Was besagt der OBT-Benchmark — und für welche zwei Query-Arten gilt sein Ergebnis **nicht**?::
Kaminsky 2022, identische Auswertungen gegen Star Schema vs. denormalisierte Tabelle: **Redshift 25–30 %**, **Snowflake ~25 %**, **BigQuery ~50 %** schneller für OBT.<br>**Scope: ausdrücklich BI-typische Queries** — Dashboard-Abfragen mit bekannten Filtern und Aggregationen.<br>**Gilt nicht für:** **1.** beliebige **Ad-hoc-Queries**, deren Muster beim Bau nicht bekannt waren. **2.** **Browse-Queries** — ein `SELECT DISTINCT category` fürs Dropdown scannt bei OBT die Milliarden-Zeilen-Tabelle statt einer kleinen Dimension. Das ist kein Unterschied von 30 %, sondern von Größenordnungen.<br>Grundsätzlicher Vorbehalt: Der Benchmark misst **Laufzeit, nicht Betriebskosten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJGi5yxQdE811UeXdjTDR9" -->
Erkläre den **Fan-Trap** — und warum merkt man ihn nicht?::
Er entsteht, wenn man zwei Faktentabellen **unterschiedlichen Grains** zusammenführt. Eine Bestellung über 100 € mit drei Positionen erscheint nach dem Join **dreimal**, jedes Mal mit `order_amount = 100`.<br>`SELECT SUM(order_amount)` liefert dann **300 € statt 100 €** — der Betrag der gröberen Tabelle wurde durch die Zeilenzahl der feineren **vervielfacht**.<br>**Warum man es nicht merkt:** Die Query läuft **fehlerfrei** und liefert eine **plausible Zahl**. Keine Fehlermeldung, keine Warnung, kein NULL. Ob 300 € richtig ist, sieht nur, wer die korrekte Summe schon kennt.<br>Derselbe Mechanismus wie eine Bridge Table ohne Gewichtungsfaktor — nur steht dort eine Warnung in der Literatur.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7VBSldfI87FB1KYkRt036" -->
Was ist der **Chasm-Trap**, und wie schützt ein Star Schema vor Fan- und Chasm-Trap?::
**Chasm-Trap:** zwei **unabhängige 1:n-Beziehungen** von derselben Entität aus. Ein Kunde hat Bestellungen *und* Supportfälle — joint man beide, entsteht ein **Kreuzprodukt** zwischen den Zweigen (3 Bestellungen × 4 Tickets = 12 Zeilen). Beide Seiten sind danach aufgebläht.<br>**Der Schutz ist keine Magie, sondern Disziplin:** Jede Faktentabelle hat genau **einen Grain**, und Fakten verschiedener Grains werden **nicht gejoint**. Braucht man beide, fragt man sie **getrennt** ab und stellt die Ergebnisse über eine conformed dimension nebeneinander — **Drill-Across**.<br>Der Grain ist damit nicht nur eine Modellierungsentscheidung, sondern ein **Schutzmechanismus**. OBT gibt ihn auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9HNr6ljrOMcMAHEPUufRq" -->
Nenne die Nachteile von OBT **jenseits** von Performance und Fan-Trap::
**Konsistenzdrift** — dasselbe Attribut steckt in jeder OBT, die es braucht. Fünf Fachbereiche bauen fünf breite Tabellen mit je eigener Segment-Logik, und die laufen auseinander. Genau das Problem, gegen das **conformed dimensions** erfunden wurden — ohne den Mechanismus, der es löst.<br>**Wartbarkeit** — ändert sich ein Dimensionsattribut, muss die **gesamte** breite Tabelle neu geschrieben werden.<br>**Eingebrannter Grain** — eine Frage auf anderer Ebene braucht eine neue Tabelle.<br>**Historisierung unklar** — der Attributwert ist zum Ereigniszeitpunkt eingefroren; die „as is"-Sicht ist ohne Neubau nicht möglich.<br>**Keine Wiederverwendung** — eine Dimension dient vielen Faktentabellen, eine OBT genau einem Zweck.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOAJYn3UFwGsIRuMvae1M7" -->
Warum ist „storage is cheap, compute is expensive" nur die halbe Wahrheit?::
**Was stimmt:** Der Speicher für redundante Attribute ist im Column Store fast kostenlos, und jeder eingesparte Join spart Rechenzeit — also Geld bei verbrauchsabhängiger Abrechnung.<br>**Was übersehen wird:** Die Rechenzeit **verschiebt sich nur**. Der Join, den die Abfrage spart, wird **beim Bau der Tabelle** bezahlt — bei **jedem** Ladelauf. Ändert sich ein Dimensionsattribut, kostet der Neuaufbau deutlich mehr als ein `UPDATE` auf eine Dimension.<br>**Ehrliche Formulierung:** *Bei hohem Abfrage-zu-Ladelauf-Verhältnis lohnt es sich, Rechenzeit von der Abfrage in den Ladelauf zu verschieben.* Derselbe Gedanke wie bei Materialized Views und Aggregattabellen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7BBI4k3K9x5MErMPN6aQ0" -->
In welcher Schicht sitzt **OBT** — und warum **nicht** in der Integrationsschicht?::
**Am ehesten in der Präsentationsschicht** (Gold, Marts): Der Zweck und damit der Grain stehen fest, die Zielgruppe muss nicht joinen können, Abfragegeschwindigkeit zählt, und die Tabelle ist aus einer darunterliegenden Schicht **ableitbar**.<br>**Nicht in der Integrationsschicht** (Silber, Core, Raw Vault) — und der Grund ist **nicht Skalierung**, sondern **Wiederverwendbarkeit**: Diese Schicht bedient viele Konsumenten mit **verschiedenen Grains**. Eine OBT würde den Grain **eines** Konsumenten einbrennen und genau das zerstören, wofür die Schicht existiert.<br>**Nicht in der Rohschicht** (Bronze, Staging) — die ist per Definition **quellförmig**; dort wird nichts zusammengeführt, weil nichts interpretiert wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYEquP21LfBYC1Doe7UV6z" -->
Wie löst sich der **dbt-Widerspruch** auf — Doku sagt „wide and denormalized", Blog sagt Kimball?::
**Sie sprechen über verschiedene Reichweiten.**<br>**Die Doku** meint einen **einzelnen Mart** für **einen** Konsumenten — dort stellt sich die Konsistenzfrage nicht, weil es nur eine Definition gibt. Sie setzt ihre Empfehlung sogar selbst unter Vorbehalt: sie gilt, *solange man den Semantic Layer nicht nutzt*.<br>**Der Blog** meint **unternehmensweite** Modellierung mit **mehreren** Fachbereichen — dort kehrt genau das Problem zurück, für das es conformed dimensions gibt.<br>Belegt durch dbts eigenes Eingeständnis: *„dbt might have made things **too** easy"* — niedrige Einstiegshürden führten zu Wildwuchs, daraus folgt der Ruf nach mehr Modellierungsdisziplin.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskFlj0Y2b7a9Nkxy2aBWdi" -->
Unter welcher Bedingung ist eine **OBT mit der goldenen Regel vereinbar**?::
**Vereinbar**, wenn sie aus einer normalisierten oder atomaren Quellschicht **abgeleitet** ist und daraus **jederzeit neu erzeugt** werden kann. Dann ist sie Derived Data — eine Materialisierung, nichts weiter. Genau deshalb sitzt OBT in der Präsentationsschicht.<br>**Nicht vereinbar**, wenn sie selbst zum System of Record wird: wenn Daten nur noch dort existieren, direkt darin korrigiert wird, oder sie nicht rekonstruierbar ist.<br>**Der Praxistest ohne Theorie:** *Könnte ich diese Tabelle löschen und aus der Schicht darunter neu bauen?* **Ja** → Regel eingehalten. **Nein** → gebrochen — unabhängig davon, wie die Tabelle heißt oder wo sie liegt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5QRTTwU5ldIoDj03D1ve4" -->
Die goldene Regel der Denormalisierung in **einem** Satz — plus die drei Folgerungen::
**Jede redundante Tabelle braucht einen Ort, an dem die Wahrheit *einmal* steht.**<br>**1.** Redundanz ist **erlaubt** — im Warehouse sogar erwünscht.<br>**2.** Sie muss **ableitbar** sein — aus etwas, das sie erzeugen kann.<br>**3.** Korrigiert wird **an der Quelle**, nie in der Kopie.<br>Fehlt dieser eine Ort, hat man viele gleichberechtigte Kopien und **keine Instanz, die bei Widersprüchen entscheidet** — und keine Möglichkeit, die Tabelle neu zu bauen.<br>Die eine Ausnahme: **SCD2-Tabellen** sind nicht neu erzeugbar und deshalb der dokumentierte Sonderfall — es sei denn, eine Persistent Staging Area stellt die Regel wieder her.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEAhI35bPv3gmvjA7xdBqL" -->
Was ist ein **Activity Schema**, und was ist sein Reiz?::
Der vierte Modellierungsansatz: Statt **Entitäten** (Kunden, Produkte) modelliert er **Ereignisse**. Eine einzige Tabelle — der *Activity Stream* — mit Spalten wie `activity_id, ts, entity, activity, feature_json, revenue_impact, link`.<br>„Bestellung aufgegeben", „E-Mail geöffnet", „Abo gekündigt" sind alles Zeilen derselben Tabelle, unterschieden durch die Spalte `activity`. Abgefragt wird über **zeitliche Verknüpfungen** („die erste Aktivität X **nach** jeder Aktivität Y").<br>**Reiz:** Kein Schema, das sich bei einer neuen Aktivität ändern müsste — man fügt Zeilen ein, keine Spalten. Und Fragen nach **Abfolgen** sind natürlich ausdrückbar.<br>**Kehrseite:** ungewohnte Abfragemuster, starke Werkzeugabhängigkeit, schwach für klassische Kennzahlenberichte.
<!-- basicblock-end -->

# Block 5 — Schichtenarchitektur, Medallion & Master Data

*Die ersten zehn Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="ObsRfVzQ8W2wkSF7HkoJZChB" -->
Was kritisiert Kimball an „independent data marts" — und warum ist dieses Muster trotzdem so verbreitet?::
Er nennt sie *„standalone analytic silos … a DW/BI **architecture that's essentially un-architected**"*.<br>**Warum verbreitet:** *„It mirrors the way many organizations fund IT projects, plus it requires **zero cross-organizational data governance and coordination**. It's the path of least resistance for fast development at relatively low cost, at least in the short run."*<br>**Warum schädlich:** mehrfache unkoordinierte Extrakte aus denselben Quellen, redundante Speicherung — und vor allem *„incompatible views of the organization's performance, resulting in unnecessary organizational debate and reconciliation."*<br>Das ist dasselbe Problem wie die **Konsistenzdrift** bei OBT.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgHNCFkssvw2pnm48thBuG" -->
Welche drei Modellordner nutzt dbt, und was leistet jeder?::
**staging** — *„creating our atoms, our initial modular building blocks, from source data"*. Ein Modell pro Quelltabelle, benannt `stg_<quelle>_<entität>`, Unterordner **nach Quellsystem**.<br>**intermediate** — *„stacking layers of logic with clear and specific purposes **to prepare our staging models to join** into the entities we want"*.<br>**marts** — *„bringing together our modular pieces into a **wide, rich vision** of the entities our organization cares about"*.<br>Die Analogie der Doku: Quelldaten sind „a soup of raw energy and quarks", Staging kondensiert sie zu **Atomen**, aus denen später Moleküle gebaut werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEd8Q7CHPAAhpxNFgdjkuQ" -->
Warum gliedert dbt den Staging-Ordner nach **Quellsystem** und ausdrücklich nicht nach Fachbereich?::
> *„A key goal of any great dbt project should be establishing a **single source of truth**. By breaking things up too early, we open ourselves up to creating **overlap and conflicting definitions** (think marketing and financing having different fundamental tables for orders). We want everybody to be building with the **same set of atoms**."*<br>Auch abgelehnt: Gliederung **nach Lader** (Fivetran, Stitch) — „too broad to be useful on a project of any real size".<br>Begründung für Quellsystem: Tabellen aus derselben Quelle teilen Ladeverfahren und Eigenschaften, also lassen sie sich gemeinsam behandeln.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsI52nidYDGxJmPY2JfVQ6d" -->
Was ist das DRY-Prinzip in der Staging-Schicht, und welches Beispiel nennt dbt?::
Transformationen, die man **immer** haben will, gehören **so weit stromaufwärts wie möglich** — einmal statt wiederholt.<br>> *„This saves us from potentially wasting code, complexity, and compute doing the same transformation more than once. For instance, if we know we always want our monetary values as **floats in dollars**, but the source system is **integers and cents**, we want to do the division and type casting **as early as possible** so that we can reference it rather than redo it repeatedly downstream."*<br>Merksatz für Staging: **normalisieren, was unstrittig ist — interpretieren, was strittig ist, erst später.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxHuYOEv3snFl7h0tAyKxB" -->
Welche Haltung empfiehlt die dbt-Doku gegenüber ihren eigenen Konventionen?::
> *„This guide is just a starting point. You may decide that you prefer Birkenstocks or a purple hoodie for your project over Jobs-ian minimalism. That's fine. What's important is that you **think through the reasoning** for those changes in your organization, **explicitly declare them** in a thorough, accessible way for all contributors, and above all **stay consistent**."*<br>Der übertragbare Kern: Nicht die konkrete Konvention ist entscheidend, sondern dass sie **begründet, dokumentiert und konsequent** ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRyYnc80KXkz8vw15htogh" -->
Woher kommt der Begriff „Medallion", und was heißt das für seine Einordnung?::
Er wurde **von Databricks um 2020 populär gemacht** — es ist ein **Marketingbegriff** für ein sehr altes Muster, entstanden im **Lakehouse**-Kontext.<br>Joe Reis dazu: *„Medallion is the **same thing**, but here we use the stages of bronze → silver → gold."*<br>Elliott Cordo schärfer: *„there is **very little difference** between Medallion architecture and how a traditional EDW is composed. If anything, the EDW conceptually was a little cleaner, with better separation of concerns."*<br>Die Einordnung: neues Vokabular für die klassische Staging/Core/Presentation-Schichtung — mit drei echten Unterschieden (Speichertechnologie, Schema-Drift-Toleranz, Modellagnostik).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshVby35tSVYtF3deJk9DB4" -->
Was passiert in Bronze, Silber und Gold — in Joe Reis' Kurzfassung?::
**Bronze:** *„Raw ingestion. Think **append-only** files or tables, minimal transformation, **schema drift tolerated**, heavy lineage focus."*<br>**Silber:** *„Cleansed and conformed. **De-duplicated, typed, and standardized IDs, business keys resolved**, and quality rules enforced."*<br>**Gold:** *„Business-ready. Data is modeled specifically for **consumption** — dashboards, reports, self-serve analytics."*<br>Zwei Details, die den Unterschied zur klassischen Staging-Schicht ausmachen: Bronze **toleriert Schema Drift** (eine klassische Staging-Tabelle mit festem DDL würde abbrechen), und Silber ist die Schicht, in der **Business Keys aufgelöst** werden — der eigentliche Integrationsschritt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLYPd1cZQbtH8kqFQg0phk" -->
Was meint Joe Reis mit „Medallion is NOT a data model"?::
> *„**At no point do we prescribe how the data is modeled. Each stage is model agnostic.**"*<br>Er stützt das auf die Definition: *„A data model organizes and standardizes data in a **precise, structured representation**, enabling and guiding human and machine behavior."* Medallion tut nichts davon — es benennt **Reifegrade im Lebenszyklus**, keine Strukturen.<br>**Folge:** Medallion ist zu Kimball, Inmon und Data Vault **orthogonal**, nicht konkurrierend. Die Achsen stehen senkrecht: **Medallion sagt, wie weit die Daten sind. Die Modelle sagen, wie sie strukturiert sind.**<br>Und: „Gold = Star Schema" ist eine verbreitete, aber **keine notwendige** Zuordnung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsh0HLEpHtWGWyu9z7igMUc" -->
Was ist laut Bellemare das gemeinsame Grundproblem operativer und analytischer Anwendungsfälle?::
> *„Operational and analytical use cases all face the same problem: they are unable **to reliably access relevant, complete, and trustworthy data** from across their organization."*<br>Und sein Urteil über die Medallion-Antwort darauf: *„**Multi-hop architectures** have been around for decades as part of bridging the operational-analytical divide. However, they are inherently **inefficient, slow, expensive, and difficult to reuse**."*<br>Daraus folgt seine Gegenthese („Shift Left"): Datenqualität **an der Entstehung** sicherstellen — über **Data Contracts** — statt sie nachgelagert Schicht für Schicht zu reparieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsy6fdfoFjq0plEDltnh9Q5" -->
Warum Data Vault in Silber und dimensionale Modellierung in Gold? (Packkildurais Begründung)::
> *„**Rigid star schemas** provide blazing-fast analytics but **crack under schema changes**. **Data Vaults** offer unmatched schema flexibility and auditability but impose **significant query complexity** and performance limitations."*<br>Jede Schicht bekommt das Modell, dessen **Stärke dort zählt** und dessen **Schwäche dort nicht wehtut**:<br>In **Silber** zählt Änderungsresistenz, und niemand fragt interaktiv ab.<br>In **Gold** zählt Geschwindigkeit und Verständlichkeit — und das Schema ist stabil, weil es abgeleitet wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiDgR29bWQzl3rOaRgeCwz" -->
Nenne drei Gründe, warum man ein Warehouse überhaupt **schichtet** — und wann Schichten Overhead sind::
**1. Die goldene Regel erzwingt eine Quelle** — eine denormalisierte Präsentationstabelle braucht einen Ort, aus dem sie ableitbar ist. Also mindestens zwei Schichten.<br>**2. Integration ist ein eigener Arbeitsschritt** — Daten aus fünf Quellen zu vereinheitlichen ist eine andere Aufgabe als sie abfragbar zu machen, und braucht ein anderes Modell.<br>**3. Wiederverwendbarkeit** — wer für jede Frage direkt von der Quelle baut, baut dieselbe Bereinigung zehnmal und jedes Mal anders.<br>**Overhead** bei zwei Quellen und einem Konsumenten. **Die Schichtung zahlt sich über die Anzahl der Konsumenten aus, nicht über die Datenmenge.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLxOJN8EqfUUqmFERFRjUY" -->
An welchen zwei Stellen ist der Datenfluss durch die Schichten **nicht** linear?::
Der Kern ist linear: `Quellen → Staging → Core → Presentation → Konsumenten`. Zwei Durchbrechungen:<br>**Stammdaten fließen quer** — sie speisen die Integrationsschicht und werden im Downstream-MDM-Fall von ihr gespeist. Nach DMBOK sind sie eine **eigene Speicherzone**, keine Schicht im Fluss.<br>**Reverse ETL fließt zurück** — Ergebnisse aus der Präsentationsschicht gehen in operative Systeme, etwa berechnete Kundensegmente ins CRM.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNdTrdGGvnV6QLVVF51lPW" -->
Was ist der Unterschied zwischen **transienter** und **persistenter** Staging Area — und warum ist das architektonisch folgenreich?::
**Transient:** Rohdaten werden geladen, verarbeitet und **verworfen**. Reiner Arbeitsspeicher.<br>**Persistent (PSA):** Jeder Roh-Snapshot bleibt **dauerhaft** erhalten, mit Ladezeitstempel und Herkunft.<br>**Die Folge:** Ohne PSA ist eine SCD2-Dimension ein **System of Record** — nicht rekonstruierbar, Backup-Pflicht, kein Drop-and-Reload. **Mit** PSA liegt die Rohhistorie vor, die SCD2-Tabelle ist **neu berechenbar** und damit wieder **Derived Data**. Die goldene Regel gilt dann für das **gesamte** Warehouse.<br>Preis: Speicher — und in regulierten Kontexten Konflikt mit **Löschpflichten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs81DHqsER4s4tv9mFRN2pF" -->
Was ist ein **Operational Data Store**, und wodurch unterscheidet er sich von Staging **und** vom Warehouse?::
Eine **integrierte, aktuelle, veränderliche** Datenbank für **operatives** Reporting.<br>**Gegen Staging:** Der ODS ist **integriert** über Quellsysteme; Staging ist quellförmig.<br>**Gegen das Warehouse:** Der ODS ist **nicht historisiert** und **veränderlich** (Update-in-place) — das Warehouse ist nonvolatile und time-variant.<br>**Anwendungsfall:** Ein Callcenter braucht **jetzt** eine konsolidierte Kundensicht über alle Systeme — aber keine Dreijahreshistorie.<br>Heute seltener, weil CDC, Streaming-Ingestion und Minuten-Latenz im Warehouse den Zweck übernommen haben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsexbYURAYSFlJcLQEQtsHv" -->
Warum bleibt die Integrationsschicht **atomar und modellneutral**?::
Weil sie **keinem Konsumenten gehört.** Sie bedient **viele** nachgelagerte Verbraucher mit **verschiedenen Fragen und verschiedenen Grains**.<br>Jede Entscheidung, die den Grain oder die Struktur an einen konkreten Anwendungsfall bindet, macht die Schicht für **alle anderen** schlechter.<br>Genau deshalb hat **OBT dort nichts zu suchen** — es würde den Grain eines einzelnen Konsumenten einbrennen.<br>Was dort passiert: Typisierung, Wertenormalisierung, Deduplizierung, **Schlüsselauflösung**, Integration, Historisierung, Qualitätsregeln.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAvmIlvQrbweJm47LDP0KT" -->
Was ist ein **Semantic Layer**, und wie verhält er sich zu conformed dimensions?::
Eine **Definitionsschicht ohne eigene Daten**, über der Präsentationsschicht. Sie beantwortet nicht „wo liegen die Zahlen", sondern **„was heißt diese Kennzahl"** — und generiert daraus SQL. Vertreter: dbt Semantic Layer, Cube, LookML.<br>**Problem, das sie löst:** Wenn fünf BI-Werkzeuge ihre eigene `SUM(...)`-Definition von „Umsatz" mitbringen, liefert jedes eine andere Zahl.<br>**Zusammenhang:** Ein Semantic Layer erledigt auf **Metadatenebene**, was conformed dimensions auf **Datenebene** leisten — beide sorgen dafür, dass ein Begriff **einmal** definiert ist.<br>Deshalb steht die dbt-Empfehlung „wide and denormalized" ausdrücklich unter dem Vorbehalt, dass **kein** Semantic Layer im Einsatz ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs77OyYl7GPGNUMvgEWwGXu" -->
Nenne die üblichen Modell-Kombinationen über die drei Schichten::
**Data Vault in Silber, Star Schema in Gold** — für große Landschaften mit vielen, häufig wechselnden Quellen. Auditierbar und änderungsresistent unten, konsumfreundlich oben.<br>**3NF in Silber, Star Schema in Gold** — die klassische Inmon-Variante. Maximale Konsistenz, hoher Vorabaufwand.<br>**Dimensional-atomar in Silber, Marts in Gold** — die Kimball-Variante ohne eigene Integrationsschicht. Schnell, aber Integration hängt an conformed dimensions statt an Struktur.<br>**OBT in Gold** neben dem Star Schema — für einzelne Hochlast-Anwendungen, aus dem Star Schema abgeleitet.<br>Bronze bleibt in allen Varianten **Quellkopie**, oft untypisiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1tFtAhDlpBdAyJV0irexj" -->
An welchen **vier** Stellen passt die dbt-Struktur **nicht** auf die klassischen Schichten?::
**1. `intermediate` ist keine Integrationsschicht** — sie existiert *„to prepare our staging models to join into the entities we want"*, hat **keine eigenen Konsumenten** und ist oft nicht materialisiert. Eine Core-Schicht ist ein **dauerhaftes, wiederverwendbares Produkt**.<br>**2. `marts` ist nicht die klassische dimensionale Präsentationsschicht** — die Doku empfiehlt **wide and denormalized**, also OBT.<br>**3. dbt-Schichten sind logisch, nicht physisch** — ein Staging-Modell ist oft nur eine **View**.<br>**4. Es gibt keine persistente Staging Area** — dbt beginnt bei `sources`, die es **nicht besitzt**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVganyRDkPyyurra4B6LW7" -->
Welche **drei echten** Unterschiede hat Medallion gegenüber der klassischen Schichtung?::
**1. Speichertechnologie** — Tabellen in **offenen Formaten** (Delta, Iceberg) auf Object Storage, statt Schemata in einer relationalen DW-Datenbank.<br>**2. Schemadisziplin in Schicht 1** — Bronze **toleriert Schema Drift**; eine klassische Staging-Tabelle mit festem DDL bricht ab.<br>**3. Modellvorgabe** — Medallion ist **ausdrücklich modellagnostisch**; die klassische Core-Schicht ist typischerweise modelliert (3NF oder Data Vault).<br>Alles andere ist neues Vokabular. Serra listet in seinem Buch sogar **fünf** Zonen mit je mehreren Synonymen — und benutzt den Begriff „Medallion" **nie**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4F5IeeBFkUNxtf5A00V4L" -->
Was ist am Übergang von Silber (Data Vault) nach Gold (Star Schema) teuer? Drei Punkte::
**1. PIT-Tabellen** — um aus Satellites mit unterschiedlichen Load Dates eine dimensionale Zeile zu bauen, braucht man die zeitliche Auflösung; ohne PIT ist das eine Kette von Bereichs-Joins.<br>**2. Surrogatschlüssel-Verwaltung** — der Data Vault hat **Hash Keys pro Entität**, das Star Schema braucht **Surrogatschlüssel pro Version**. Diese Zuordnung muss gebaut und **stabil gehalten** werden, sonst zeigen Faktenzeilen nach dem nächsten Lauf auf andere Dimensionszeilen. *Der in Projekten am häufigsten unterschätzte Punkt.*<br>**3. Inkrementelle Ladelogik** — nur die betroffenen Dimensionsversionen und Faktenzeilen nachziehen, nicht den ganzen Mart.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsD4mHZgzcSlwuEbJTYZdmx" -->
Nenne Bellemares **fünf** Schwachstellen der Medallion-Architektur::
**1.** Der **Konsument trägt die Last des Datenzugriffs** — jeder baut die letzte Meile selbst, jedes Mal anders.<br>**2.** **Teuer** — dieselben Daten mehrfach gespeichert und verarbeitet.<br>**3.** **Datenqualität lässt sich nachgelagert schwer herstellen** — was in Bronze kaputt ankommt, wird in Silber geflickt; die **Ursache** beim Produzenten bleibt unberührt.<br>**4.** **Bronze ist fragil** — man landet, was ankommt; ändert die Quelle etwas, bricht es weiter unten, oft **still**.<br>**5.** **Keine Wiederverwendbarkeit für operative Workloads.**<br>Dazu: **Latenz akkumuliert** (drei Sprünge, drei Ladezyklen) und **Logik wandert in die falsche Schicht** (Packkildurai: *Silver Overload*, *Cross-Layer Entanglement*).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsydI0ygh8dF2xEPsmj34NA" -->
Was ist in einer **Streaming**-Medallion-Architektur die schwierige Schicht, und warum?::
**Silber.** Bronze ist append-only Ingest aus dem Event-Stream, Gold sind inkrementell gepflegte Materialized Views — beide unproblematisch.<br>Silber bedeutet im Stream **zustandsbehaftete Verarbeitung**: Deduplizierung über ein Zeitfenster, Qualitätsregeln pro Event. **Zustand braucht eine Entscheidung darüber, wie lange man wartet** — Watermarks. Ein Duplikat, das drei Tage später eintrifft, wird nicht erkannt, wenn das Fenster zwei Tage groß ist.<br>Zusätzlich: Ein **klassischer Kimball-Ladeprozess** passt nicht in einen Stream (die Surrogate Key Pipeline braucht einen Lookup gegen eine fertige Dimension). Deshalb ist Gold im Streaming häufiger **OBT-artig** — das Event trägt seinen Kontext selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5CqlJx4POBliSDjvPGieS" -->
Was ist ein **Data Contract**, und was unterscheidet ihn von Dokumentation?::
Eine explizite Vereinbarung zwischen **Datenproduzent und Konsument** über: **Schema** (Felder, Typen, Pflichtfelder) · **Semantik** · **Qualitätszusagen** (Wertebereiche, Eindeutigkeit, Vollständigkeit) · **SLA** (Aktualität, Verfügbarkeit) · **Änderungsprozess** für Breaking Changes.<br>**Der Unterschied zur Dokumentation:** Ein Contract wird **automatisiert geprüft**, und zwar **beim Produzenten** — eine vertragsbrechende Schemaänderung scheitert in **dessen CI**, nicht drei Wochen später in der Pipeline des Analysten.<br>Das ist die **„Shift Left"**-Idee und die Antwort auf Bellemares Schwachstellen 3 und 4.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZJK4UOGAk1PJknJ4TJTyM" -->
Was kostet die Schichtung — und wogegen rechnet man das auf?::
**Kosten:** **Speicher** (grob das Dreifache, entschärft durch billigen Object Storage für Bronze) · **Rechenzeit** (drei Jobs statt einem) · **Latenz** — die kritischste Größe, weil sie sich **addiert**: stündliche Läufe pro Schicht heißen Gold ist drei Stunden alt.<br>**Nutzen:** **Wiederherstellbarkeit** (jede Schicht aus der vorigen neu baubar) · **Wiederverwendung** (Silber speist viele Gold-Produkte) · **Debugging** (man kann zwischen den Schichten hineinschauen).<br>**Konsequenz:** Schichten sind kein Selbstzweck. Bei zwei Quellen und einem Konsumenten reichen Staging und Mart.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHY8ryPxLRNKzrkCMhUHoc" -->
Was unterscheidet **Data Lake**, **Data Warehouse** und **Lakehouse**?::
**Warehouse** — verwalteter, proprietärer Speicher, **schema-on-write**, relational, Transaktionen, teuer.<br>**Lake** — **Object Storage** mit offenen Dateien, **schema-on-read**, beliebige Formate, **keine Transaktionen**, billig. Typische Schwäche: der **„Data Swamp"**.<br>**Lakehouse** — **offene Tabellenformate** (Delta, Iceberg, Hudi) über Dateien im Object Storage. Die Metadatenschicht bringt **ACID-Transaktionen**, Schema-Durchsetzung, **Time Travel** und Pruning über Statistiken (Zone Maps) — Warehouse-Eigenschaften zu Lake-Kosten.<br>**Der Zusammenhang:** Die Medallion-Architektur ist im Kern **die Antwort darauf, wie man ein Lakehouse organisiert** — daher Schema-Drift-Toleranz und Modellagnostik.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsemagoElyA8BGHupRer8uz" -->
Nenne fünf Merkmale von **Stammdaten** — und welches erzeugt das eigentliche Problem?::
Definition: *„data about the **business entities that provide context for business transactions**"*. Kategorien: Parteien (Kunden, Lieferanten, Mitarbeiter), Produkte, Finanzstrukturen, Orte.<br>**1.** Entitäten, keine Ereignisse — ein Kunde *ist*, eine Bestellung *passiert*<br>**2.** relativ **stabil** — deshalb ist SCD die passende Technik<br>**3.** vielfach **wiederverwendet**<br>**4. systemübergreifend** ← **das erzeugt das Problem**<br>**5.** kleines Volumen gegenüber Transaktionen<br>Punkt 4 ist der Kern: Stammdaten sind das, was **mehrere Systeme gleichzeitig** über dasselbe Objekt wissen — und worüber sie sich widersprechen können.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3JX2TlZlTNHUge9NteGpC" -->
Grenze **Stammdaten**, **Bewegungsdaten**, **Referenzdaten** und **Metadaten** ab::
> IBM: *„**Transactional data** is the data that is produced by transactions within applications; **master data** is the data that represents the key business entities that participate within transactions. Reference data is also distinct from **metadata**, which describes the structure of an entity."*<br>**Stammdaten** — die Entitäten (Kunde Müller, Produkt „Wireless Mouse")<br>**Bewegungsdaten** — was durch Vorfälle entsteht (Bestellung 1001 am 3. April)<br>**Referenzdaten** — Kategorisierung und Klassifikation (Ländercode `DE`, Währung `EUR`, Status `versandt`)<br>**Metadaten** — beschreiben die **Struktur** anderer Daten
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1njPM3P04G40ngaefQlb1" -->
Was ist das **schärfste** Kriterium, um Stammdaten von Referenzdaten zu trennen?::
**Eine Änderung an einem Referenzdaten-*Wert* kann eine Prozessänderung erzwingen. Eine Änderung an Stammdaten läuft durch bestehende Prozesse.**<br>Neuer **Kunde** → nichts Besonderes, jedes System kann ihn verarbeiten.<br>Neuer **Bestellstatus** → möglicherweise müssen fünf nachgelagerte Systeme lernen, was sie damit tun sollen.<br>Zwei weitere trennscharfe Merkmale: Referenzdaten sind oft **extern definiert** (ISO-Ländercodes, Währungen) — Stammdaten entstehen im Unternehmen. Und Referenzdaten haben **kleine, abgeschlossene Wertemengen**, Stammdaten wachsen unbegrenzt.<br>*Die Grenze ist fließend:* Ein „Kundensegment" ist beides, je nachdem wen man fragt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTPrdWo3i9fEd38UoCrqLp" -->
Sollte man Stammdaten **normalisiert oder denormalisiert** speichern?::
**Beides — je nach Schicht.** Die Frage ist ohne Angabe der Schicht nicht beantwortbar.<br>**Im MDM-Hub / der Integrationsschicht: normalisiert.** Eine Stelle pro Entität, eine Definition — sonst lässt sich gar nicht entscheiden, welcher Wert gilt. Constraints durchsetzbar, Governance greift an **einem** Punkt, Änderungen sind billig. Es ist ein **System of Record**.<br>**In der Präsentationsschicht: denormalisiert.** Als flache **conformed dimension**, mit SCD-Typ-2-Historisierung. Die Redundanz kostet im Column Store fast nichts.<br>Dasselbe Muster wie im ganzen Kapitel: **Die Schicht bestimmt das Modell.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfXypiBwZFEa2j5hrHOBeH" -->
Was macht einen **Golden Record** golden — und warum existiert er oft in keiner Quelle?::
Er ist die **konsolidierte, bestmögliche Version** einer Entität, **zusammengesetzt** aus mehreren widersprüchlichen Quellen — nicht ausgewählt, sondern **konstruiert**.<br>Der Name kommt aus dem CRM, die Adresse aus dem Versandsystem, die Telefonnummer aus dem Support-Tool. **Kein einziges Quellsystem enthält ihn in dieser Form.**<br>Er entsteht in zwei Schritten: **Entity Resolution** (welche Sätze meinen dasselbe Objekt?) und **Survivorship** (welcher Wert gewinnt pro Attribut?).<br>Verwandte Begriffe: **System of Record** (maßgebliches System) · **System of Reference** (verlässliche Kopie, nicht Quelle) · **Trusted Source** (die anerkannte beste Version).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ditQirC6QzcrreKS4Rpo" -->
**Wer** entscheidet bei widersprüchlichen Werten, und nach welchen Regeln?::
**Kein System.** Ein MDM-Hub **führt Regeln aus**, er stellt sie nicht auf — das tut ein **Data Steward** bzw. die **Data Governance**. Derselbe Punkt wie bei conformed dimensions („defined in collaboration with the business's data governance representatives"): Integration ist am Ende eine **organisatorische** Frage.<br>**Survivorship-Regeln**, meist kombiniert und **pro Attribut**:<br>**Quellpriorität** (das ERP gilt für Rechnungsadressen) · **Aktualität** (jüngster Zeitstempel) · **Vollständigkeit** (Wert schlägt NULL) · **Häufigkeit/Voting** (die Mehrheit der Quellen) · **Spezifität** („Berlin-Mitte" schlägt „Berlin") · **manuelle Übersteuerung**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDjAxmu3p9Occ0Bhg4bhpJ" -->
Was ist der Unterschied zwischen **best-of-breed** und **best record** bei Survivorship?::
**Best-of-breed (attributweise):** Pro Attribut gewinnt die jeweils beste Quelle. Ergebnis ist ein **zusammengesetzter** Satz. Genauer — aber der entstehende Datensatz kann **in sich inkonsistent** sein (Straße aus System A, Hausnummer aus B).<br>**Best record (satzweise):** Ein Quellsatz gewinnt **komplett**. Weniger genau, dafür **in sich konsistent**.<br>Ehrlicher Hinweis: Zu Survivorship gibt es praktisch **keine herstellerneutrale freie Literatur** — alles Lesbare ist Produktdokumentation. Das ist selbst ein Befund: Es ist der Teil von MDM, der am stärksten vom Werkzeug abhängt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4rfaL6uMmcRGYS9Jug5vg" -->
Warum ist probabilistisches Matching genauer als Fuzzy-Matching — was ist der **mechanische** Unterschied?::
**Fuzzy misst Ähnlichkeit. Probabilistisch gewichtet Evidenz.**<br>Fellegi-Sunter fragt nicht „wie ähnlich sind diese Strings", sondern **„wie aussagekräftig ist es, dass diese beiden Sätze in diesem Feld übereinstimmen?"** — und die Antwort hängt vom **Wert** ab:<br>Übereinstimmung bei einem **seltenen** Nachnamen ist **starke** Evidenz.<br>Übereinstimmung bei „Müller" ist **schwache** Evidenz.<br>Übereinstimmung beim **Geschlecht** ist fast **wertlos** — das passiert in der Hälfte der Fälle zufällig.<br>Ein Fuzzy-Verfahren kann das **prinzipiell nicht** leisten, weil es keinen Begriff davon hat, wie selten ein Wert ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbkYg7Wz12y3WSqXGmLpQz" -->
Was sind **m-** und **u-Wahrscheinlichkeit**, und wozu dient **Blocking**?::
**m-Wahrscheinlichkeit** — die Felder stimmen überein, **gegeben** die Sätze sind ein **echter Treffer**. Hoch, wenn das Feld verlässlich erfasst ist.<br>**u-Wahrscheinlichkeit** — die Felder stimmen überein, **gegeben** die Sätze sind **kein** Treffer. Hoch bei häufigen Werten, niedrig bei seltenen.<br>Aus beiden entsteht pro Feld ein **Match Weight**; alle Gewichte plus eine Grunderwartung (λ) ergeben eine **Gesamtwahrscheinlichkeit**. Zwei Schwellen trennen sicheren Treffer, sicheren Nicht-Treffer und ein manuell zu prüfendes Mittelfeld.<br>**Blocking:** Man vergleicht nicht alle Paare (bei 10 Mio. Sätzen wären das 50 Billionen), sondern nur Kandidaten innerhalb eines Blocks — etwa gleiche Postleitzahl. **Der Schritt, der das Verfahren überhaupt rechenbar macht.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIckhXCX942S6PhNkFm8gy" -->
Welche **MDM-Architekturstile** gibt es — und welche Falle lauert bei der Frage?::
**DAMA nennt drei:**<br>**Registry** — Daten bleiben in den **Quellsystemen**; der Hub hält nur **Schlüssel und Querverweise**, der Golden Record wird **zur Laufzeit** zusammengesetzt. Geringe Eingriffstiefe.<br>**Transaction Hub** — Anwendungen schreiben Stammdaten **direkt in den Hub**; er ist das System of Entry. Höchste Kontrolle, höchster Umbauaufwand.<br>**Consolidated** — physische **Kopie im Hub**, Quellsysteme schreiben weiter, der Golden Record wird berechnet und **zurückverteilt**. Der verbreitete Kompromiss.<br>**Die Falle:** In der Herstellerliteratur (Gartner, Profisee, Stibo) kursiert eine **Vierer**-Taxonomie — Registry / Consolidation / **Coexistence** / Centralized. „Coexistence" gibt es bei DAMA **nicht**. Beide Begriffssysteme sind im Umlauf.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3nOzEK9OFtRHzy6awHVZ9" -->
Warum ist **Reference Data Management** schwieriger, als Codelisten aussehen?::
> IBM: *„Reference data **changes relatively infrequently, but it does change over time**, and given its **ubiquity**, synchronizing reference data values and managing changes across the enterprise is a **major challenge**."*<br>Die operativen Fragen: **Wer besitzt die Liste?** (oft niemand — sie ist irgendwann in einer Anwendung entstanden) · **Wie versioniert man?** (ein Ländercode-Satz von 1995 ≠ heute; für historische Auswertungen braucht man die **damals gültige** Liste — **SCD Typ 2 gilt auch hier**) · **Wie mappt man zwischen Codesätzen?** (die Mapping-Tabellen sind selbst Referenzdaten) · **Wie verteilt man Änderungen**, bevor Transaktionen mit dem neuen Wert eintreffen?
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsE0KXKFrndFiENHsba4Omy" -->
Was ist der Zusammenhang zwischen **MDM** und **conformed dimensions** — und der Unterschied?::
**Zwei Antworten auf dasselbe Problem**, in verschiedenen Schichten und mit verschiedenen Eigentümern.<br>**MDM** — vor oder neben dem Warehouse, Ergebnis ist der **Golden Record**, Eigentümer ist die Data Governance.<br>**Conformed Dimensions** — im Warehouse, Präsentationsschicht, Ergebnis ist eine flache historisierte Dimensionstabelle, Eigentümer ist das Warehouse-Team.<br>**Die entscheidende Einsicht:** Conformed Dimensions **erfordern MDM-artige Arbeit — ob man sie so nennt oder nicht.** Damit eine Kundendimension konformiert ist, muss vorher jemand Matching und Survivorship betrieben haben.<br>**Der Unterschied ist also nicht, ob die Arbeit passiert, sondern wo sie passiert und wer sie besitzt.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs44HEwo7HwyOT2Ruz2CPgB" -->
**Enterprise MDM** vs. **Downstream MDM** — was ist der Unterschied, und welcher ist häufiger?::
**Enterprise MDM:** Ein zentraler Hub sitzt **vor** dem Warehouse. `Quellsysteme → MDM-Hub → Warehouse → Marts`. Das Warehouse konsumiert bereits gemasterte Daten.<br>**Downstream MDM:** Kein zentraler Hub. `Quellsysteme → Warehouse (macht das Mastering hier) → Marts`.<br>> Kimball: *„Some organizations are lucky enough to have a centralized MDM system … But such centralization is **rare in the real world**. More frequently, the data warehouse extracts multiple incompatible customer data files and builds a **'downstream' MDM system**."*<br>**Der zweite Fall ist der Normalfall** — mit der unangenehmen Folge, dass das Warehouse-Team faktisch **unternehmensweite Governance** ausübt, ohne Mandat und meist ohne Ressourcen. Kimball dazu: *„there is no secret weapon."*
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswshEMfiVUn6C0O2TDOLlY" -->
Wo sitzen **Stammdaten** in der Warehouse-Architektur?::
Nach DMBOK sind sie eine **eigene Speicherzone**, nicht eine Schicht im Fluss — gleichrangig neben *Staging area*, *Central Warehouse*, *ODS*, *Data marts* und *Cubes*.<br>Sie wirken an **zwei** Stellen:<br>**In die Integrationsschicht hinein** — dort werden Business Keys aufgelöst und Entitäten zusammengeführt; ohne gemasterte Stammdaten ist das nicht möglich.<br>**Aus ihr heraus** — im häufigen **Downstream-MDM**-Fall entsteht das Mastering genau dort und wird von dort an andere Systeme zurückgegeben.<br>In der **Präsentationsschicht** erscheinen dieselben Daten als **conformed dimensions** — flach, denormalisiert, historisiert. Das ist ihr letztes Format auf dem Weg zum Analysten.
<!-- basicblock-end -->

# Block 6 — Partitionierung & idempotente Ladeverfahren

*Die ersten zehn Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obswi7gbo0itFrTFjPklIzV6" -->
Ab wann lohnt sich Partitionierung in Postgres? (Faustregel der Doku und Crunchy Data)::
> Postgres-Doku: Die Vorteile sind meist erst den Aufwand wert, *„when a table would otherwise be very large … a rule of thumb is that the size of the table should **exceed the physical memory** of the database server."*<br>Crunchy Data praktischer: Eine **200-GB**-Datenbank partitioniert man eher nicht; wer etwas baut, das in den **Terabyte**-Bereich wächst, sollte es sich ansehen.<br>Darunter reichen Indizes und die eingebauten Mechanismen — Partitionierung bringt dann vor allem Verwaltungsaufwand.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssBnTw8JNDSaE2zgIiIpXW" -->
Welche drei Partitionierungsarten kennt Postgres, und wofür eignet sich jede?::
**Range** — Wertebereiche ohne Überlappung, typischerweise **Zeit**. Der Normalfall im Warehouse: Abfragen filtern Zeiträume, alte Daten werden zusammenhängend entfernt.<br>**List** — explizite Wertelisten je Partition, etwa Region oder Mandant.<br>**Hash** — Modulus und Rest auf den Hash des Schlüssels; verteilt gleichmäßig ohne fachliche Gruppierung.<br>Hash hilft **weder** beim Bereichs-Pruning **noch** beim Lebenszyklus — es gibt keine „alte" Hash-Partition. Im Warehouse deshalb selten.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obslou4iYd8qmMFoQ2cHV4MS" -->
Sind die Grenzen einer Range-Partition in Postgres inklusiv oder exklusiv?::
**Untere Grenze inklusiv, obere exklusiv** (halboffen).<br>Geht eine Partition von `1` bis `10` und die nächste von `10` bis `20`, gehört der Wert `10` zur **zweiten** Partition.<br>Für Datumsbereiche heißt das: `FOR VALUES FROM ('2025-04-01') TO ('2025-05-01')` umfasst genau den April. Dieselbe Konvention wie bei halboffenen SCD-Intervallen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7JxzBuJ20Ig0UpRMApM1c" -->
Was ist **Sub-Partitionierung**, und was dürfen einzelne Partitionen unterschiedlich haben?::
Eine Partition kann **selbst wieder partitioniert** sein — etwa erst nach Monat, darin nach Region.<br>Alle Partitionen müssen **dieselben Spalten** wie die Elterntabelle haben, dürfen aber eigene **Indizes, Constraints und Default-Werte** besitzen.<br>Praktischer Nutzen: Alte, selten gelesene Partitionen können andere Indizes bekommen als die aktuelle, viel beschriebene.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0gyFgvnqNZEq8V3w5HYWz" -->
Warum ist das Entfernen einer Partition fundamental anders als `DELETE FROM … WHERE`?::
> Postgres-Doku: Partitionierung erlaubt es, diese sonst mühsame Aufgabe *„**nearly instantaneously** by manipulating the partition structure, rather than physically moving large amounts of data around"* zu erledigen.<br>**DELETE:** jede Zeile einzeln markiert · WAL pro Zeile · tote Tupel, die `VACUUM` später aufräumen muss · Tabelle bleibt bis dahin aufgebläht · Laufzeit wächst mit der Zeilenzahl.<br>**Partition entfernen:** eine **Metadatenänderung** · praktisch kein WAL · Speicher sofort frei · Laufzeit nahezu konstant.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObserBpzuHuFfEkm1LuLAYAe" -->
Warum `DETACH PARTITION … CONCURRENTLY` statt direkt `DROP TABLE` auf die Partition?::
**Die Sperre:** `DROP TABLE` auf eine Partition braucht einen **ACCESS EXCLUSIVE**-Lock auf die **Elterntabelle** und blockiert kurz jede Query auf der ganzen Tabelle. `DETACH … CONCURRENTLY` braucht nur **SHARE UPDATE EXCLUSIVE**.<br>**Die Daten bleiben erhalten:** Die abgehängte Partition ist eine eigenständige Tabelle — man kann sie archivieren oder prüfen, bevor man sie löscht.<br>Einschränkungen: `CONCURRENTLY` darf **nicht in einem Transaktionsblock** laufen und nicht, wenn eine **Default-Partition** existiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspDSckw2Rx7VCKyXIrJIix" -->
Welche Vorteile von **Clustering** nennt die BigQuery-Doku?::
> *„Clustering addresses how a table is stored so it's generally a **good first option** for improving query performance."*<br>Tabellen und Partitionen **über 64 MB** profitieren wahrscheinlich; darunter ist der Effekt vernachlässigbar.<br>Filtert eine Query auf Clustering-Spalten, werden nur die passenden **Blöcke** gelesen (Block Pruning).<br>Auch **hohe Kardinalität** profitiert — BigQuery bekommt detaillierte Metadaten darüber, wo welche Werte liegen.<br>Die Speicherblöcke werden **adaptiv** an die Tabellengröße angepasst.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYjryaNB2p1Z9rOwlIN9Z7" -->
Was sagt Databricks dazu, ob man Tabellen überhaupt partitionieren sollte?::
> *„Most tables on Databricks with **less than 100 TB** of data don't need partitioning. Databricks uses Delta Lake for all tables by default and **automatically clusters data in unpartitioned tables by ingestion time**, so you get partitioning-like performance without manual tuning."*<br>Die vollständigen Schwellwerte: **unter 1 TB** nicht partitionieren · **1–100 TB** Liquid Clustering statt Partitionierung · darüber ggf. partitionieren, dann **mindestens 1 GB pro Partition**.<br>Und: *„Tables with **fewer, larger partitions** tend to outperform tables with many smaller partitions."*
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs26oNQpNf941sHHg997maL" -->
Was gewinnt man durch Partitionierung — und was ist ausdrücklich **nicht** der Gewinn?::
**Die vier Gewinne:** **1.** Partition Pruning — Queries mit Filter auf den Schlüssel lesen nur passende Partitionen. **2.** Billiger Lebenszyklus — alte Daten durch Entfernen einer Partition statt per `DELETE`. **3.** Kleinere Indizes pro Partition. **4.** Wartung (`VACUUM`, Statistiken) pro Teil.<br>**Nicht der Gewinn:** Skalierung über mehrere Maschinen. In Postgres liegen alle Partitionen auf **demselben Server**. Das Verteilen über Knoten ist **Sharding** — gleiches Wort in manchen Systemen, anderes Problem (Rebalancing, Request Routing).<br>Im Warehouse ist Partitionierung ein Werkzeug, **um Scans zu verkleinern**, nicht um Last zu verteilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNsnoDYdiSnK5BdIE10CTx" -->
Was ist der Unterschied zwischen **Partition Pruning** und **Constraint Exclusion**?::
**Partition Pruning** nutzt die **Partitionsgrenzen** der deklarativen Partitionierung — und wirkt beim Planen **und zur Laufzeit**.<br>**Constraint Exclusion** nutzt **CHECK-Constraints** der einzelnen Tabellen — und wirkt **nur beim Planen**. Gehört zur alten Vererbungs-Partitionierung.<br>Der praktisch wichtige Unterschied ist der Zeitpunkt: Steht der Filterwert erst bei der Ausführung fest (Parameter eines Prepared Statements, Subquery), kann nur Pruning ihn nutzen. Im `EXPLAIN` sichtbar als **`Subplans Removed: n`**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjmfYzKE70QgltbMd0tp8K" -->
Warum verhindert ein Filter über einen Join zur Datumsdimension das Partition Pruning — und was hilft?::
Bei `WHERE d.year = 2025` (Join von Fakt zu `dim_date`) sieht der Planer nur einen Join, **nicht** den Wertebereich des Partitionsschlüssels in der Faktentabelle — also kann er beim Planen keine Partition ausschließen.<br>**Abhilfe:** Kimballs **Smart Key `YYYYMMDD`** für die Datumsdimension. Damit lässt sich direkt `fact.date_key BETWEEN 20250101 AND 20251231` filtern, und die Partitionen werden schon beim Planen ausgeschlossen.<br>Das ist der Grund, warum Kimball ausgerechnet beim Datum vom bedeutungslosen Surrogatschlüssel abweicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxtzcnsiwRSEikaoMjjFPO" -->
Warum muss ein Primärschlüssel auf einer partitionierten Postgres-Tabelle den Partitionsschlüssel enthalten?::
Postgres kennt für partitionierte Tabellen **nur lokale Indizes** — ein Index auf der Elterntabelle ist in Wahrheit je ein Index **pro Partition**. Einen globalen Index gibt es nicht.<br>Eindeutigkeit kann deshalb nur **innerhalb** einer Partition geprüft werden. Enthält der Schlüssel den Partitionsschlüssel, folgt aus „eindeutig in jeder Partition" automatisch „eindeutig in der Tabelle", weil ein Wert nur in **einer** Partition liegen kann.<br>Folge: Auf einer nach `order_date` partitionierten Faktentabelle muss es `PRIMARY KEY (order_item_id, order_date)` sein — dieselbe `order_item_id` an zwei Tagen fällt der Datenbank nicht auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdoMlAU2ERHUnhowizF3EP" -->
Was automatisiert `pg_partman` — und warum braucht man es trotz nativer Partitionierung?::
**Postgres legt Partitionen nicht selbst an und entfernt sie nicht selbst.** `pg_partman` arbeitet **auf** der nativen deklarativen Partitionierung und automatisiert:<br>**Vorab anlegen (premake)** — Partitionen für die nächsten Tage/Monate, **bevor** Daten eintreffen. Ohne das schlägt am Monatsersten jeder `INSERT` fehl oder alles landet in der Default-Partition.<br>**Retention** — alte Partitionen automatisch abhängen oder löschen.<br>**Wartung** per Hintergrundprozess oder `pg_cron`, und das Einsortieren von Zeilen aus der Default-Partition.<br>*Keine Altlast* — nur ältere Versionen konnten zusätzlich trigger-basiert partitionieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsofn8bJIFDlRh3jOzq4aaP" -->
Wie partitioniert man eine bestehende große Postgres-Tabelle nachträglich, ohne alles umzukopieren?::
**1.** Neue partitionierte Tabelle mit gleicher Struktur anlegen.<br>**2.** Die **bestehende Tabelle als eine große „Alt"-Partition einhängen** (`ATTACH`) — für den gesamten bisherigen Bereich.<br>**3.** Künftige Partitionen normal anlegen.<br>**4.** In einer kurzen Transaktion die Namen tauschen.<br>**5.** Die Alt-Partition später in Ruhe aufteilen.<br>**Der Trick:** Beim `ATTACH` scannt Postgres alle Zeilen auf Gültigkeit. Ein **vorab** angelegter passender `CHECK`-Constraint (erst `NOT VALID`, dann `VALIDATE CONSTRAINT` ohne exklusive Sperre) lässt diesen Scan entfallen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgGlYyzDBCuSIBUuLMZvxD" -->
Wie rahmt **BigQuery** Partitionierung, und welche Granularitätsregel und welches Limit gelten?::
Als **Kosteninstrument**: Bei verbrauchsabhängiger Abrechnung zahlt man pro gescanntem Byte, und mit Partitionierung lässt sich der Preis **vor** der Ausführung **exakt** abschätzen.<br>**Granularität:** **stündlich** bei hohem Volumen über kurzen Zeitraum (unter ~6 Monaten) · **täglich** als Normalfall · **monatlich/jährlich** bei wenig Daten pro Tag über langen Zeitraum.<br>**Hartes Limit:** **10.000 Partitionen pro Tabelle** (4.000 pro Job).<br>Unter etwa **10 GB pro Partition** empfiehlt BigQuery, statt zu partitionieren zu **clustern**. Nützlich: `require_partition_filter` verhindert versehentliche Vollscans.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXlfXQhwV5yjkUdTtNkg8m" -->
Warum ist hohe Kardinalität beim **Clustering** eine Stärke, bei **Partitionierung** aber ein Problem?::
**Partitionierung** nach einer Spalte mit Millionen Werten (Kunden-ID) erzeugt **Millionen winziger Partitionen** — Small-Files-Problem, Planungs-Overhead, Limits.<br>**Clustering** sortiert die Daten dagegen nur **innerhalb** von Blöcken und führt Metadaten über die Wertebereiche. Viele verschiedene Werte bedeuten feinere Metadaten und präziseres Überspringen.<br>Faustregel: **niedrige bis mittlere Kardinalität → Partitionierung, hohe → Clustering.** Bei Clustering zählt zudem die **Reihenfolge** der (max. vier) Spalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsamL99rI6NzmokhNW4bNCG" -->
Wie funktionieren **Snowflakes Micro-Partitions** — und welchen Partitionierungsvorteil gibt man auf?::
Snowflake zerlegt **jede** Tabelle automatisch in Micro-Partitions von **50–500 MB** unkomprimiert, spaltenweise gespeichert, in **Ladereihenfolge** gebildet. Wertebereiche dürfen sich **überlappen**. Zu jeder führt Snowflake Metadaten (Min/Max je Spalte u. a.), über die Abfragen Micro-Partitions überspringen.<br>**Aufgegeben wird der billige Lebenszyklus:** Ohne selbst definierte Grenzen gibt es keine Partition „Januar 2023", die man in einem Schritt entfernen könnte.<br>Zudem hängt die Pruning-Qualität von der **natürlichen Ladereihenfolge** ab; bei durcheinander geladenen Daten hilft nur ein **Clustering Key**, dessen Hintergrund-Sortierung laufend Geld kostet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFJsBlVOdNLbgfuLNvR6rl" -->
Was ist **Liquid Clustering** in Delta Lake, und was kann es, das Partitionierung nicht kann?::
Der Nachfolger von Partitionierung und Z-Ordering. Man gibt Clustering-Spalten an, und das System ordnet die Daten **inkrementell** — nicht bei jedem Schreiben alles neu.<br>**Der entscheidende Unterschied:** Die Clustering-Spalten lassen sich **ändern, ohne die Tabelle neu zu schreiben**. Bei Partitionierung heißt ein falscher Schlüssel: alles umschreiben.<br>Databricks empfiehlt es für Tabellen zwischen **1 und 100 TB** statt Partitionierung — und für alle neuen Tabellen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7Ru0B7mkfFj6T74nWh1OH" -->
Partitionierung oder Clustering — wie entscheidet man?::
**Klein?** Gar nichts tun — die eingebauten Mechanismen reichen.<br>**Groß, Abfragen filtern auf ein Datum, alte Daten werden regelmäßig entfernt?** Nach Datum **partitionieren**, grob genug.<br>**Groß, Abfragen filtern auf andere Spalten oder IDs?** **Clustern.**<br>**Beides?** Grob nach Datum partitionieren, **innerhalb clustern** — die verbreitetste Kombination.<br>Partitionierung: harte Grenzen, exaktes Pruning, billiger Lebenszyklus, teuer bei falscher Wahl. Clustering: weiche Ordnung, auch hohe Kardinalität, keine Fragmentierungsgefahr, Spalten änderbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obst4HFN3ibkSTwJ3n51pjQT" -->
Was ist **Hidden Partitioning** in Apache Iceberg, und was ist **Partition Evolution**?::
**Hidden Partitioning:** Die Partitionierung ist eine **Transformation** einer bestehenden Spalte, etwa `PARTITIONED BY (days(ts), bucket(16, user_id))`. Queries filtern ganz normal auf `ts`; Iceberg leitet die Partitionen selbst ab. Der Nutzer muss die Partitionierung **nicht kennen**.<br>Gelöst wird das Hive-Problem einer zusätzlichen Partitionsspalte, auf die jede Query filtern muss: *„Using the wrong format … produces **silently incorrect results**, not query failures."*<br>**Partition Evolution:** Die Partitionierung lässt sich **ändern, ohne bestehende Daten umzuschreiben** — alte Daten behalten ihr Schema, neue bekommen das neue, Abfragen laufen über beide.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZnYW4RTD5Et9MZz2cApmM" -->
Databricks sagt „mindestens 1 GB pro Partition", BigQuery „unter ~10 GB lieber clustern" — warum ist das kein Widerspruch?::
**1. Sie messen verschiedene Dinge.** Die Databricks-Zahl ist eine **Mindestgröße**, falls man partitioniert. Die BigQuery-Zahl ist eine **Entscheidungsschwelle** zwischen zwei Verfahren. Eine Partition mit 5 GB ist für Databricks in Ordnung und für BigQuery ein Hinweis auf Clustering — beides gleichzeitig wahr.<br>**2. Es sind engine-spezifische Heuristiken, keine Naturgesetze.** Sie folgen aus der jeweiligen Speicherarchitektur (Dateigrößen in Delta Lake, Blockverwaltung und Abrechnung in BigQuery).<br>Lehre: Bei jedem Schwellwert fragen, **woraus er sich ergibt** — dann sieht man, ob er übertragbar ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3U9hCnFlCOOLmErurdKdr" -->
Was haben **Partition Pruning** und **Zone Maps** gemeinsam, und worin unterscheiden sie sich?::
**Dasselbe Prinzip:** „Kann in diesem Stück überhaupt etwas Passendes stehen?" — sonst überspringen.<br>**Partition Pruning:** Einheit = Partition (groß, grob) · Wertebereich **deklariert** vom Engineer · nie überlappend · Wirkung hängt von der **Wahl des Schlüssels** ab.<br>**Zone Maps:** Einheit = Block / Datei / Micro-Partition (klein, fein) · Wertebereich **beobachtet** (Min/Max je Spalte) · überlappend möglich · automatisch · Wirkung hängt von der **Sortierung** ab.<br>Snowflake und Databricks **ersetzen deklarierte Partitionen durch Zone Maps**; Clustering sortiert die Daten, damit die Zone Maps **scharf** werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJrBKLr2fCFPO06N7DvATN" -->
Nach welchen vier Kriterien wählt man einen **Partitionsschlüssel** — und warum erfüllt Zeit fast immer alle?::
**1. Passt zu den häufigsten `WHERE`-Filtern** — sonst kein Pruning, nur Overhead (Onehouse: Partitionierung passt nicht zur *query shape*).<br>**2. Stabil** — der Wert einer Zeile ändert sich **nie**; sonst wandert die Zeile zwischen Partitionen (Löschen plus Einfügen).<br>**3. Passende Kardinalität** — nicht zwei Werte, nicht Millionen; hohe Kardinalität gehört ins Clustering.<br>**4. Halbwegs gleichmäßige Verteilung.**<br>**Zeit** erfüllt alle vier: Analytik filtert fast immer auf Zeiträume, das Ereignisdatum ändert sich nie, die Granularität ist steuerbar, und alte Daten werden zusammenhängend entfernt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFlOrNzgHjNiTpYMM2K6yx" -->
Warum partitioniert man eine SCD2-Tabelle nach `row_effective_date` und nicht nach `is_current`?::
**Hauptgrund — Stabilität:** `is_current` und `valid_to` werden auf **bestehenden Zeilen geändert**. Jede neue Version lässt die alte Zeile in eine **andere Partition wandern** — ein Löschen plus Einfügen; in Delta Lake oder Iceberg heißt das, Dateien neu zu schreiben. `row_effective_date` ändert sich dagegen **nie**.<br>**Nebengrund — Kardinalität:** `is_current` hat nur **zwei Werte**, also zwei riesige Partitionen ohne sinnvolles Pruning.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsh9V6BKUsqrpAQ3GQR5MTz" -->
Was ist **Skew**, und was sind typische Ursachen?::
Die Daten verteilen sich **ungleichmäßig** auf die Partitionen — einige riesig, viele winzig. Man bekommt beide Nachteile gleichzeitig: die riesigen sind langsam, die winzigen erzeugen Overhead.<br>Onehouse: *„uniform data distribution across naturally partitioned datasets is **too perfect to be true**."*<br>Ursachen: **Lastspitzen** (Black Friday, Monatsabschluss) · **Wochenmuster** (am Wochenende kaum Bestellungen) · vor allem **unterschiedlich große Kunden** — nach Mandant partitioniert ist der größte oft hundertmal so groß wie der Median.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaltS1pvXGdof3vB73SFhk" -->
Welche vier Mechanismen machen **Over-Partitioning** schädlich — und welcher begründet, dass es **schlimmer** ist als Under-Partitioning?::
**1. Small-Files-Problem** — jede Datei kostet festen Aufwand (öffnen, Metadaten, Netzwerkanfrage), unabhängig vom Inhalt. Onehouse: bei 1,5-MB-Dateien ~8 Min., zwischen 105 MB und 2 GB kein Unterschied.<br>**2. Schlechtere Kompression und Vektorisierung** — kleine Spaltenblöcke.<br>**3. Planungs- und Metadaten-Overhead** — jede nicht ausgeschlossene Partition zählt; BigQuery-Limit 10.000.<br>**4. Die Asymmetrie** ← der Grund für „schlimmer": Under-Partitioning lässt sich **innerhalb** der Partitionen ausgleichen (Clustering, Sortierung, Zone Maps, Indizes). Over-Partitioning **fragmentiert** die Daten — reparierbar nur durch Neuschreiben. Und es kostet bei **jeder** Query, besonders bei denen ohne Filter auf den Schlüssel.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsP87ZhyTdaFZVWT6XYFcV7" -->
Warum sollen Pipelines nicht `INSERT INTO` verwenden? In welchen vier Situationen entstehen Duplikate?::
`INSERT INTO` ist **nicht idempotent** — die Datenbank hat kein Konzept davon, dass ein Eintrag schon existiert.<br>Duplikate entstehen bei: **Retry nach Teilfehler** (ohne Transaktion stehen die ersten 80 % doppelt) · **Backfill** ohne vorheriges Löschen · **At-least-once-Zustellung** (Event doppelt) · **überlappenden Extraktionsfenstern** („die letzten 25 Stunden").<br>Besonders unangenehm: Die Summen blähen sich auf, **ohne dass irgendwo ein Fehler erscheint** — strukturell wie der Fan-Trap.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2me9ggNSYEdiYwXpXDVgs" -->
Welche drei Voraussetzungen und Fallen hat `MERGE`?::
**1. Stabiler Schlüssel.** Fehlt einer, gibt es nichts zu vergleichen. Erzeugt die Pipeline ihn selbst (Zufalls-UUID, Ladezeitstempel), ist er bei jedem Lauf anders — `MERGE` fügt ein statt abzugleichen.<br>**2. Quelle pro Schlüssel eindeutig.** Sonst bricht Postgres mit *„cannot affect row a second time"* ab; Snowflake warnt vor **nichtdeterministischen** Ergebnissen. Vorher deduplizieren.<br>**3. Deletes nur bei vollständiger Quelle.** `WHEN NOT MATCHED BY SOURCE` wertet alles Fehlende als gelöscht — eine Teillieferung würde den Rest löschen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIKWsrkaOXjPjL3qgpgZn2" -->
Was macht `INSERT OVERWRITE` — und welche Bedingung muss erfüllt sein, damit es keine Daten vernichtet?::
Es **ersetzt einen ganzen Zielbereich** — Tabelle oder Partitionen — durch das Ergebnis des Laufs, **ohne zeilenweisen Vergleich**. Idempotent, weil derselbe Lauf denselben Inhalt schreibt. Braucht **keinen Schlüssel**.<br>**Die Bedingung:** Der Lauf muss den **vollständigen** Inhalt des Zielbereichs liefern. Liefert er für den 3. April nur die Hälfte, ist die andere Hälfte **verloren** — `MERGE` hätte nur weniger aktualisiert, `INSERT OVERWRITE` löscht.<br>Spark: **statischer** Modus löscht alle Partitionen der Angabe (ohne Angabe: **die ganze Tabelle**), **dynamischer** nur die, für die Daten geschrieben werden. Delta Lake: `REPLACE WHERE`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMmhIkoxiwAW6mmozanv4e" -->
Welche Garantie gibt **nur** `INSERT … ON CONFLICT`, und was kann **nur** `MERGE`?::
**Nur `ON CONFLICT`:** atomares Einfügen-oder-Aktualisieren **auch unter hoher Nebenläufigkeit** (*„even under high concurrency"*). Fügen zwei Prozesse gleichzeitig dieselbe Zeile ein, kann `MERGE` mit einer Unique-Verletzung abbrechen — die `MERGE`-Doku verweist dafür selbst auf `ON CONFLICT`.<br>**Nur `MERGE`:** **löschen** · beliebig viele bedingte Zweige · beliebige Join-Bedingung **ohne Unique-Index** · `WHEN NOT MATCHED BY SOURCE` (PG 17) · `RETURNING` mit `merge_action()` · SQL-Standard.<br>**Idempotent können beide sein** — das hängt von der Bedingung ab, nicht vom Werkzeug.<br>Faustregel: reiner Upsert mit gleichzeitigen Schreibern → `ON CONFLICT`; mehrere Aktionsarten im Batch → `MERGE`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsm2uxInzrys4Qyb9CFSKaX" -->
Warum kann ein einfacher `MERGE` keine SCD-Typ-2-Änderung durchführen — und wie umgeht man das?::
`MERGE` erlaubt **pro Quellzeile nur eine Aktion**. Eine Typ-2-Änderung braucht für **eine** Quellzeile **zwei**: alte Version schließen **und** neue einfügen.<br>**Verdopplungstrick** (Muster der Delta-Lake-Doku): Die geänderten Zeilen kommen **zweimal** in die Quelle — einmal mit echtem Schlüssel (trifft die alte Version → schließen), einmal mit **`NULL`** als Schlüssel (trifft nie → einfügen).<br>**Haken in Postgres:** Die Reihenfolge der Aktionen innerhalb eines `MERGE` ist nicht garantiert; ein Unique-Index auf „eine aktuelle Version" kann anschlagen. Deshalb dort **`UPDATE` + `INSERT` in einer Transaktion**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNRQjR5cAzqyNXhXmwofhg" -->
Nenne die fünf dbt-**incremental strategies** und was jede tut::
**`append`** — fügt an, ohne zu prüfen. **Nicht idempotent**: das `INSERT INTO`-Antipattern.<br>**`delete+insert`** — löscht im Ziel die Zeilen mit den Schlüsseln des Batches, fügt dann ein. Zwei Anweisungen, geht auch ohne `MERGE`-Unterstützung.<br>**`merge`** — Upsert über `unique_key` per `MERGE`.<br>**`insert_overwrite`** — ersetzt **ganze Partitionen**, die der Batch berührt. Idempotent, wenn der Batch sie vollständig enthält.<br>**`microbatch`** — zerlegt nach Ereigniszeit in Batches und ersetzt jeden einzeln.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEtJRkqvIlYus7ZPyxavxg" -->
dbt auf BigQuery: Was ist der Unterschied zwischen **statischem** und **dynamischem** `insert_overwrite`?::
**Dynamisch** (Standard): dbt ermittelt zuerst mit einer **eigenen Abfrage**, welche Partitionen der neue Batch berührt, und ersetzt genau diese. Einfach zu konfigurieren, kostet eine **zusätzliche Abfrage**.<br>**Statisch:** Die zu ersetzenden Partitionen stehen **in der Konfiguration**. Spart die Ermittlungsabfrage — **billiger** —, aber man muss sie selbst richtig angeben.<br>Fehler bei statisch: zu wenige angegeben → Daten bleiben liegen; Partitionen angegeben, die der Batch nicht vollständig enthält → **Daten gehen verloren**.<br>`insert_overwrite` schlägt `merge`, wenn ohnehin **ganze Partitionen** neu berechnet werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEjiPDaSr34d38T1qadCaW" -->
dbt Microbatch: Was ist **dasselbe** wie „Task = Transaktion" aus Kapitel 2.1, und was ist **neu**?::
**Dasselbe:** Eine wohldefinierte **Einheit** — Partition bzw. Batch — wird **als Ganzes und atomar ersetzt**. Weil ihr Inhalt nur vom Input abhängt, ergibt jede Wiederholung dasselbe: *„the resulting table is the same no matter how many times a batch is reprocessed."*<br>**Neu:** **1.** Das Framework bildet die Einheiten selbst (`event_time`, `batch_size`). **2.** **Backfill** als eingebauter Befehl (`--event-time-start/--event-time-end`). **3.** **`dbt retry`** verarbeitet nur fehlgeschlagene Batches nach. **4.** Batches sind unabhängig → **parallel**. **5.** Verspätete Daten über **`lookback`**. **6.** Kein eindeutiger Schlüssel nötig — ersetzt statt verglichen.<br>Kurz: **2.1 lieferte das Prinzip, Microbatch liefert es als Werkzeug.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaAtnYGKYuztElamTanv2b" -->
Nenne die vier Gruppen der **Idempotency Patterns** (Konieczny, DEDP) mit ihren Mustern::
**Overwriting** — vorhandene Daten restlos ersetzen: **Fast Metadata Cleaner** (über **Metadaten** aufräumen: `TRUNCATE`, Partition/Tabelle entfernen) · **Data Overwrite** (über **Daten** überschreiben: `INSERT OVERWRITE`, `REPLACE WHERE`).<br>**Updates** — angleichen: **Merger** (`MERGE`) · **Stateful Merger** (Merger plus Zustandstabelle der Tabellenversionen je Lauf — erlaubt sauberen Backfill durch Zurücksetzen).<br>**Database** — die Datenbank sichert ab: **Keyed Idempotency** (deterministische, aus den Daten abgeleitete Schlüssel) · **Transactional Writer** (alles in einer Transaktion).<br>**Immutable Dataset** — nie überschreiben: **Proxy**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssXJZsRHuVuSPbnK7luLtb" -->
Was macht das **Proxy**-Pattern, und wann ist es der Ausweg?::
**1.** Jeder Lauf schreibt in einen **neuen**, unveränderlichen Bestand — etwa `sales_v20250403`.<br>**2.** Erst wenn er vollständig und geprüft ist, wird ein **Zeiger** (View, Alias) **atomar** auf ihn umgeschaltet.<br>**3.** Der vorige Stand bleibt erhalten — **Rollback** ist erneutes Umschalten.<br>Leser sehen **nie einen halbfertigen Zustand**, ohne dass man auf Transaktionen angewiesen ist. Das Daten-Gegenstück zum Blue-Green-Deployment — und der Ausweg, wenn ein Warehouse keine Transaktionen über mehrere Anweisungen kennt (*Write-Audit-Publish*).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJipgvQUcFiZY8DQRX6Mej" -->
Welche Idempotency Patterns **vernichten** eine SCD2-Historie — und warum?::
**Fast Metadata Cleaner** und **Data Overwrite** — beide **leeren erst und befüllen dann neu**.<br>Eine SCD2-Tabelle ist **System of Record**: Ihre Historie lässt sich nicht aus der Quelle neu erzeugen, weil die Quelle die alten Werte überschrieben hat. Wer sie leert, verliert sie endgültig.<br>Passend sind **Merger** bzw. **Stateful Merger** (inkrementell angleichen) — und der **Transactional Writer** ohnehin.<br>**Ausnahme:** Mit einer **Persistent Staging Area**, die alle Roh-Snapshots behält, wird ein Neuaufbau wieder möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFZQZHuX15ODIGMiYxIZpE" -->
Was tut man, wenn ein Warehouse **keine Transaktionen über mehrere Tabellen** kennt?::
Viele Systeme garantieren Atomarität nur **pro Anweisung** bzw. **pro Tabellen-Commit** (etwa das Transaktionslog von Delta Lake). „Task = Transaktion" aus 2.1 ist dann nicht direkt umsetzbar.<br>**1. Jeden Schritt einzeln idempotent machen** — dann ist ein Abbruch harmlos, die Wiederholung repariert alles. Der wichtigste Ausweg, funktioniert überall.<br>**2. Die Reihenfolge so wählen, dass Zwischenzustände unschädlich sind** — Dimensionen vor Fakten.<br>**3. Write-Audit-Publish** — separat schreiben, prüfen, in **einem** atomaren Schritt veröffentlichen (Proxy).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6OvHOP1sI7N3HtVndK7Su" -->
Warum ist eine Pipeline nur so idempotent wie ihr Input reproduzierbar ist? Ursachen und Gegenmittel::
Liefert die Quelle beim zweiten Lesen etwas anderes, nützt das beste `MERGE` nichts.<br>**Ursachen:** relative Zeitfenster („die letzten 24 Stunden" ab Laufzeit) · `now()`, `CURRENT_DATE`, `random()` in der Transformation · veränderliche Quellen (API, Datenbank) · **erzeugte** Schlüssel (Zufalls-UUID je Lauf).<br>**Gegenmittel:** **1.** Input **einmal** extrahieren und unverändert ablegen (Bronze, Persistent Staging Area); Wiederholungen lesen die Kopie. **2.** Mit einem **logischen Datum** als Parameter arbeiten statt mit der Uhr — wie `todays_date` in der Übungsaufgabe. **3.** Schlüssel aus den Daten **ableiten** (Keyed Idempotency).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7bgzqwN4LSf4Hy3ISc6ks" -->
Welche Grundsätze gelten für **Backfills** — und warum kann ein SCD2-Backfill nicht parallel laufen?::
**In Einheiten statt am Stück** — bei Abbruch ist der Fortschritt nicht verloren. **Jede Einheit idempotent** — ein abgebrochener Backfill wird einfach neu gestartet; ein Laufprotokoll überspringt fertige Einheiten. **Unabhängige Einheiten parallel**, abhängige in Reihenfolge. **Nicht mit dem Tagesbetrieb kollidieren.** **Nachgelagerte Tabellen mitziehen** — Aggregate und Marts sind sonst veraltet.<br>**SCD2:** Der Inhalt ist **pfadabhängig** — jeder Snapshot baut auf dem vorigen auf. Die Snapshots müssen **in Reihenfolge** nachgespielt werden — und überhaupt noch existieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvSFXwzQQAX0w9DiRRG1y5" -->
Wovon hängen die Kosten von `MERGE` gegenüber `INSERT OVERWRITE` ab — und was ist der häufigste teure Fehler?::
**`MERGE`** joint die neuen Daten mit dem Ziel und schreibt jede **Datei** neu, die eine betroffene Zeile enthält (*copy-on-write* in Delta/Iceberg). Eine geänderte Zeile in einer 500-MB-Datei → ganze Datei neu. Kosten wachsen mit der **Zahl der berührten Dateien**.<br>**`INSERT OVERWRITE`** braucht **keinen Join** mit dem Ziel; Kosten wachsen mit der **Größe der ersetzten Partitionen**.<br>Ganze Tage ohnehin neu berechnet → `INSERT OVERWRITE`. Wenige verstreute Änderungen → `MERGE`. Konzentrierte Änderungen → `MERGE` **mit Prädikat auf die Zielpartitionen** (dbt: `incremental_predicates`).<br>**Häufigster Fehler:** `MERGE` **ohne** Einschränkung auf die Zielpartitionen — aus einem kleinen täglichen Update wird ein Vollscan.
<!-- basicblock-end -->
