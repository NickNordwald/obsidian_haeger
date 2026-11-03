---
deck: "Traineeship::Skalierbarkeit & ETL::Advanced SQL & ETL"
---

Karteikarten zu [[2.1 Advanced SQL & ETL - Vertiefung|2.1 Advanced SQL & ETL – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen.

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Skalierbarkeit & ETL::Advanced SQL & ETL`. Die `oid`-Attribute werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr. Innerhalb eines Stichworts stehen die Punkte, die Nick beim Lesen selbst exzerpiert hat, **zuerst** — sie haben Vorrang und bekommen im Zweifel eine eigene Karte.

# Fortschritt

- [x] Daten-Pipelines: ETL, ELT & Batch Processing
- [x] SQL (fortgeschritten)
- [x] Deklarative Programmierung
- [x] Funktionale Programmierung
- [x] Idempotenz
- [x] Testing in ETL
- [x] Komplexe Datentypen
- [x] Nicht materialisierte Zwischenergebnisse: Views, CTEs, Subqueries
- [x] Materialisierte Zwischenergebnisse: Temp. Tables vs. Materialized Views

---

# Daten-Pipelines: ETL, ELT & Batch Processing

<!-- basicblock-start oid="Obs36V9vD5z9GJ56FKbrmDJ6" -->
Was bedeuten **ETL** und **ELT** — und worin genau unterscheiden sie sich?::
Beide bestehen aus **Extract, Transform, Load** und unterscheiden sich **nur in der Reihenfolge der letzten beiden Schritte**.<br>**ETL:** Die Transformation läuft in einer **separaten Engine außerhalb des Ziels**; ins Warehouse fließen nur fertig transformierte Daten.<br>**ELT:** Rohdaten werden direkt geladen, die Transformation passiert **im Zielsystem selbst** mit dessen Rechenleistung (meist SQL).<br>Densmore: der Unterschied ist klein, „but that has major implications".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsR3SW96memIUUTo1IKRAgv" -->
Was ist die **wichtigste Konsequenz** von ELT gegenüber ETL — jenseits der Frage, wo gerechnet wird?::
**Die Rohdaten bleiben erhalten.** Muss eine Kennzahl später anders berechnet werden, transformiert man einfach neu — bei ETL müsste man aus den Quellsystemen **neu extrahieren**, die historische Zustände oft nicht mehr vorhalten.<br>Zweiter Effekt: Bei ELT skaliert man die Pipeline, indem man das Warehouse skaliert; bei ETL ist die Transformations-Engine ein **zweites System** mit eigenem Betrieb und eigener Kapazitätsplanung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQLx8oUzSKIhPAyjL3a1Dh" -->
Was bedeutet **Ingestion**?::
**Extract + Load** — das Beschaffen der Daten und das Ablegen im Ziel, **ohne** Transformation.<br>⚠️ Nicht Extract + Transform. Im ELT-Modell markiert der Begriff exakt den Verantwortungsbereich der Data Engineers.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsW0myBIvqXiQta1tSWfLWR" -->
Was ist **EtLT** und warum wird das `t` klein geschrieben?::
Ein Zwischenschritt zwischen Extract und Load für Transformationen, die **keine Business-Logik** enthalten und eng begrenzt sind — deshalb klein.<br>**Gehört hinein:** Deduplizierung · URL in Bestandteile zerlegen · PII maskieren/hashen · Typen casten, Zeitstempel auf UTC.<br>**Gehört nicht hinein:** Umsatz pro Kunde berechnen · Bestellungen mit Stammdaten joinen · Kundensegmente bilden.<br>**Faustregel:** Wenn eine Fachabteilung über die Regel diskutieren könnte, ist es Business-Logik → großes T.<br>**Gründe fürs Vorziehen:** Compliance (PII darf nicht roh landen) und Ladeperformance.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjgeJHkhlK0EPTtHVMDDQH" -->
Warum hat sich **ELT erst in den letzten ~10 Jahren** durchgesetzt, obwohl das Konzept alt ist?::
Weil sich das **Angebot** geändert hat, nicht die Nachfrage:<br>1. **Cloud-Warehouses mit elastischem Compute** — Rechenleistung nur für die Dauer der Transformation, bezahlt nach Verbrauch<br>2. **Objektspeicher wurde extrem billig** — Rohdaten aufheben kostet fast nichts<br>3. **Trennung von Storage und Compute** — beides skaliert unabhängig<br>4. **Spaltenbasierte Speicherung** — macht Bulk-Transformationen im Ziel überhaupt erst effizient<br>Vorher: teure eigene Hardware, zeilenbasierte Warehouses → das Ziel war zu schwach, die Transformation **musste** vorgelagert werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZhWB1QypaqXlnJyUP16ze" -->
Kleppmann unterscheidet **drei Systemtypen**. Nenne sie mit Auslöser und primärer Kennzahl.::
**Online / Service** — ausgelöst durch einen Client-Request; Kennzahl **Response Time (Latenz)**, dazu **Verfügbarkeit**; Millisekunden.<br>**Batch / Offline** — ausgelöst durch einen Zeitplan; Kennzahl **Durchsatz** (Zeit für einen Datensatz gegebener Größe); Minuten bis Tage.<br>**Streaming / Near-Real-Time** — ausgelöst durch ein eintreffendes Event; Kennzahl **Latenz pro Event** bei unbegrenztem Input.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrqrCz6C1SI7rnkBamxa96" -->
Warum misst man **Batch nach Durchsatz** und **Online nach Latenz**?::
Beim Online-System **wartet ein Mensch**: Ob der Server 10.000 oder 12.000 Requests/s schafft, merkt der Einzelne nicht — dass seine Antwort 3 Sekunden braucht, sehr wohl.<br>Beim Batch-Job wartet **niemand**; er läuft nachts. Ob er 40 oder 55 Minuten braucht, ist egal, solange er vor Geschäftsbeginn fertig ist. Es zählt, wie viel Volumen pro Zeit durchgeht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsI1TihE9NeDfajvQyhkzrq" -->
Was ist der **eine entscheidende Unterschied** zwischen Batch und Streaming?::
Der **Input**, nicht die Geschwindigkeit.<br>**Batch:** begrenzter (*bounded*) Input — vollständig bekannt und vorhanden, wenn der Job startet. Der Job weiß, wann er fertig ist.<br>**Streaming:** unbegrenzter (*unbounded*) Ereignisstrom — verarbeitet Events kurz nach ihrem Auftreten, ist prinzipiell nie fertig.<br>Daraus folgt alles Weitere: Streaming braucht **Fenster**, **Watermarking** für verspätete Events und **Checkpointing**; ein Batch-Job kann bei Fehlern einfach komplett neu laufen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsbz9Pal2jxrBWRmVEfCpFL" -->
Warum lassen sich Unix-Tools wie `awk | sort | uniq -c | head` **beliebig aneinanderreihen**?::
Wegen der **uniformen Schnittstelle**: Alle lesen einen Bytestrom von **stdin** und schreiben einen nach **stdout**, konventionell als zeilenweise Textdatensätze.<br>Der ökonomische Effekt: Bei *n* Programmen mit gemeinsamer Schnittstelle bekommt man alle *n²* Kombinationen geschenkt — sonst bräuchte jede einen eigenen Adapter.<br>Dasselbe Prinzip: SQL (jede Operation nimmt Relationen, gibt eine Relation zurück), Spark-DataFrames, dbt-Modelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTJIAPBXwKtI6lEU04YBD3" -->
Kleppmann: `sort` ist „barely useful in isolation". Was ist die allgemeine Lehre für den Bau von Pipelines?::
**Ein Werkzeug, eine Aufgabe — und die Mächtigkeit entsteht aus der Komposition.**<br>`sort` ist eine bessere Sortierimplementierung als die Standardbibliotheken der meisten Sprachen (spillt auf Platte, nutzt mehrere Threads), aber allein fast nutzlos. Diese Spezialisierung ist erst bezahlbar, weil `sort` sonst nichts kann und in beliebige Ketten passt.<br>= Single Responsibility Principle auf der Ebene ganzer Prozesse.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdGLPfepzMvCHrv7N4Ecfg" -->
Was passiert in `cat access.log | awk … | sort | uniq -c` mit der **Datei `access.log`** — und was gewinnt man dadurch?::
**Gar nichts — sie wird nur gelesen, nie verändert.** Der Input ist *immutable*, jede Ausgabe geht nach stdout.<br>**Gewinn:** 1. **Wiederholbarkeit** — beliebig oft ausführbar, immer dasselbe Ergebnis (= praktische Idempotenz) · 2. **Gefahrloses Experimentieren** — Kette umbauen, `head` zwischenschalten, ohne Quelldaten zu riskieren · 3. **Triviale Fehlerbehandlung** — bricht ein Schritt ab, gibt es nichts aufzuräumen: Fehler beheben, neu starten.<br>Eine Pipeline, die ihren Input *in place* ändert, hinterlässt nach einem Abbruch einen unklaren Zustand.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAZsFKdoFupRZFBR4b98vZ" -->
Wann wählt man **ETL** statt ELT? (4 Fälle)::
1. **Das Zielsystem ist zu schwach** — altes zeilenbasiertes On-Premise-Warehouse; sonst konkurriert die Transformationslast mit den Abfragen der Analysten.<br>2. **Komplexe Geschäftsregeln brauchen eine spezialisierte Engine** — iterative Algorithmen, Aufrufe externer Dienste, exotische Formate; nicht alles lässt sich sinnvoll in SQL ausdrücken.<br>3. **Compliance** — wenn regulierte Daten **gar nicht erst roh** im Warehouse landen dürfen, ist ELTs Grundidee ausgeschlossen.<br>4. **Kostenkontrolle** bei Pay-per-Query-Modellen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8xjVOjJy0CznPTESkOcEM" -->
Welches **organisatorische** Argument spricht für ELT — ganz ohne Performance?::
**Saubere Trennung der Verantwortlichkeiten.**<br>**Data Engineers** verantworten die **Ingestion** (Extract + Load): Quellanbindung, Zuverlässigkeit, Monitoring, Schema-Drift.<br>**Analysten / Analytics Engineers** verantworten die **Transformation** — in SQL, das sie ohnehin beherrschen, direkt im Warehouse.<br>Jeder arbeitet an seiner Stärke, mit **weniger Abhängigkeiten**. Bei ETL steckt alle Transformationslogik in der Engineer-Pipeline → Analysten müssen Tickets schreiben und warten, obwohl die Engineers die Fachlogik oft gar nicht beurteilen können.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXDK2Ydr09cEMdK24TnpQ3" -->
Wie liegen Daten **zeilenbasiert** vs. **spaltenbasiert** auf der Platte — und warum ist das für OLTP bzw. OLAP entscheidend?::
Die Platte ist eindimensional, eine Tabelle zweidimensional — man muss sich entscheiden, in welche Richtung man „aufrollt".<br>**Zeilenbasiert:** `[1, Meier, DE, 100] [2, Dupont, FR, 250]` → ein Blockzugriff liefert einen **kompletten Datensatz**. Optimal für **OLTP** (`SELECT * WHERE id = 2`, günstige INSERTs/UPDATEs).<br>**Spaltenbasiert:** `[1,2,3] [Meier,Dupont,Rossi] [DE,FR,IT] [100,250,80]` → bei `SELECT land, SUM(umsatz) GROUP BY land` werden **nur zwei Blöcke** gelesen. Optimal für **OLAP**.<br>Bei 200 Spalten × 500 Mio. Zeilen ist das kein gradueller Unterschied, sondern eine Größenordnung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsp19UdHICXIEP1wJHoSIVt" -->
Welche **zwei zusätzlichen Effekte** machen spaltenbasierte Speicherung schnell — und was ist ihr Nachteil?::
**1. Kompression:** Ein Block enthält nur Werte desselben Typs mit oft geringer Kardinalität (`DE, DE, DE, FR …`) → komprimiert um Faktoren, die bei gemischten Zeilen unerreichbar sind → weniger I/O.<br>**2. Vektorisierte Verarbeitung:** Eine ganze Spalte gleichen Typs lässt sich in einem Rutsch verarbeiten (SIMD) statt Wert für Wert.<br>**Nachteil:** Ein einzelner neuer Datensatz bedeutet einen Schreibzugriff in **jeden** Spaltenblock → analytische Systeme sind auf **Bulk-Loads** ausgelegt, nicht auf Einzeltransaktionen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBBlj7U4JIHc4NGFKqIS4V" -->
Ist es ein Widerspruch, dass die App eine zeilenbasierte relationale DB nutzt, das Warehouse aber spaltenbasiert ist?::
Nein — es sind **zwei Systeme für zwei Aufgaben**. Die OLTP-Datenbank bedient die Anwendung (viele kleine, schnelle Transaktionen), das OLAP-Warehouse die Analyse (wenige riesige Abfragen).<br>**Die Pipeline ist genau das Bindeglied** — das ist der Daseinszweck von ETL/ELT: Daten vom OLTP- ins OLAP-Format überführen, ohne die Produktivdatenbank mit analytischen Abfragen lahmzulegen.<br>Beide sind weiterhin **relational** (SQL, Tabellen, Joins); der Unterschied liegt in der physischen Speicherung, nicht im Datenmodell.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAM5w7XtjDGRRTRwXGxMIP" -->
Unterschied **Control Flow** vs. **Data Flow** in einer Pipeline?::
**Control Flow** = die Ablaufsteuerung: *welche Aufgabe läuft wann, in welcher Reihenfolge, unter welcher Bedingung.* Aufgaben sind über **Precedence Constraints** verbunden (bei Erfolg / Fehler / Abschluss). Das ist die Ebene eines Airflow-DAG.<br>**Data Flow** = was *innerhalb* einer einzelnen Aufgabe mit den Daten passiert (extrahieren, transformieren, laden). Aus Sicht des Control Flow ist ein ganzer Data Flow nur **eine** Aufgabe.<br>Kurz: Control Flow orchestriert, Data Flow verarbeitet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslIB4U6jHoziyKEmOgoSfw" -->
Was ist **Reverse ETL**?::
Der umgekehrte Weg: aufbereitete Daten **aus** dem Warehouse zurück **in operative Systeme** schieben — CRM, Marketing-Automation, Support-Tools, Anwendungsdatenbanken.<br>**Sinn:** Erkenntnisse dorthin bringen, wo Mitarbeiter tatsächlich arbeiten, statt in ein Dashboard, das niemand öffnet.<br>Technisch wieder Extract-Transform-Load mit vertauschten Rollen von Quelle und Ziel; das „T" ist hier vor allem Formatanpassung an das Zielsystem.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIVJ4muJPfkzKLIHUCDAHf" -->
Nenne drei Kernunterschiede zwischen **ETL-Code** und **klassischem Backend-Code**.::
**Denkweise:** Backend imperativ/objektorientiert, **eine Entität zur Zeit** — ETL deklarativ/mengenorientiert, **alle Zeilen gleichzeitig**.<br>**Fehlerwirkung:** Backend — ein Request scheitert, ein Nutzer sieht einen Fehler, der Rest läuft. ETL — der ganze Job scheitert und hinterlässt womöglich **teilweise geschriebene Daten** → Wiederholbarkeit (Idempotenz) ist Grundanforderung.<br>**Was schiefgeht:** Backend stürzt ab (Exception, 500) — ETL produziert **stille Fehler**: Die Pipeline läuft grün durch und liefert falsche Zahlen. Deshalb Data Quality Checks statt nur Unit-Tests.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs23jLbhUR5CcEhMYHqXukT" -->
Warum ist ein Fehler in einer Datenpipeline oft **gefährlicher** als ein Absturz im Backend?::
Weil er **still** ist. Ein abgestürzter Backend-Service fällt binnen Minuten auf — Nutzer beschweren sich, Monitoring schlägt an.<br>Eine Pipeline, die wegen einer Schema-Änderung in der Quelle seit drei Wochen 8 % der Umsätze verschluckt, **läuft grün durch** — bis jemand die Zahlen anzweifelt. In der Zwischenzeit wurden Entscheidungen auf falscher Basis getroffen.<br>Deshalb braucht ETL eine grundsätzlich andere Qualitätsstrategie: Plausibilität des **Datenbestands** (Zeilenzahlen, Wertebereiche, Eindeutigkeit, NULLs) statt nur Korrektheit einzelner Funktionen.
<!-- basicblock-end -->

---

# SQL (fortgeschritten)

*Priorisierung: Nick nannte auf die Frage „was war neu?" die **Syntax**. Die Karten mit konkreten Code-Mustern stehen deshalb vorn.*

<!-- basicblock-start oid="ObsC6uizdl4ZNIQBwZt0bEMx" -->
Schreib die Syntax einer **laufenden Summe pro Kunde** (kumulierter Umsatz nach Datum).::
```sql<br>SUM(betrag) OVER (<br>    PARTITION BY kunde<br>    ORDER BY datum<br>    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW<br>)<br>```<br>**Ohne `ORDER BY`** wäre es die **Gesamtsumme** pro Kunde (in jeder Zeile derselbe Wert). Erst `ORDER BY` macht daraus die kumulierte Summe — weil es den Default-Frame ändert.<br>Der Frame ist hier technisch redundant (entspricht fast dem Default), aber `ROWS` statt des Default-`RANGE` schützt vor Sprüngen bei doppelten Datumswerten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIhaghfUuel1ISCirNXyqD" -->
Schreib eine Query: pro Monat **Umsatz und prozentuale Veränderung zum Vormonat**.::
```sql<br>SELECT monat, umsatz,<br>  lag(umsatz) OVER (ORDER BY monat) AS vormonat,<br>  round(100.0 * (umsatz - lag(umsatz) OVER (ORDER BY monat))<br>        / nullif(lag(umsatz) OVER (ORDER BY monat), 0), 1) AS pct<br>FROM monatsumsaetze<br>ORDER BY monat;<br>```<br>Zwei Fallen: **`nullif(…, 0)`** gegen Division durch null · in der **ersten Zeile ist `lag` NULL** → die Änderung ist korrekt NULL, nicht 0.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEGvdiQ3rEdIp9kayoFk2g" -->
Schreib die Syntax für **Top 3 pro Kategorie** — und begründe, warum es nicht direkt geht.::
```sql<br>SELECT kategorie, produkt, umsatz FROM (<br>  SELECT kategorie, produkt, umsatz,<br>    row_number() OVER (PARTITION BY kategorie<br>                       ORDER BY umsatz DESC) AS rn<br>  FROM produktumsaetze<br>) t<br>WHERE rn <= 3;<br>```<br>**Warum die Zwischenebene:** Window Functions laufen **nach** `WHERE` — dort existiert `rn` noch nicht. Es braucht Subquery oder CTE.<br>Snowflake/BigQuery haben dafür `QUALIFY`; Postgres nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseM9YR3n74dG2YVIXYlT2y" -->
Was ist der **eine entscheidende Unterschied** zwischen `GROUP BY` und einer Window Function?::
**`GROUP BY` reduziert, die Window Function nicht.**<br>`GROUP BY`: aus N Zeilen wird **eine pro Gruppe** — die Detailzeilen sind weg.<br>Window: **alle N Zeilen bleiben**, jede bekommt eine zusätzliche Spalte, die aus ihrer Gruppe berechnet wurde.<br>Der Gewinn: **Detail und Aggregat in derselben Zeile** (z. B. Einzelbetrag *und* Kundengesamtumsatz). Ohne Window Functions bräuchte man dafür Subquery + Join zurück.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuxAelW3eAPUkkYTMX6ki2" -->
Was bewirken **`PARTITION BY`**, **`ORDER BY`** und die **Frame-Angabe** in `OVER (…)` — und was heißt es, wenn sie fehlen?::
**`PARTITION BY`** — bildet Gruppen; die Funktion startet je Partition neu. *Fehlt es:* die ganze Ergebnismenge ist **eine** Partition.<br>**`ORDER BY`** — Reihenfolge innerhalb der Partition; nötig für alles mit „vorher/nachher". *Fehlt es:* der Frame ist die **ganze Partition**.<br>**Frame** — welche Zeilen der Partition einfließen. *Fehlt er bei vorhandenem `ORDER BY`:* `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — erste Zeile bis aktuelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxjHWtcqXrT8glRpADNe7q" -->
Warum liefert **`last_value()`** fast nie das erwartete Ergebnis — und wie repariert man es?::
Weil der Default-Frame bei der **aktuellen Zeile endet**. Der „letzte Wert im Frame" ist damit die aktuelle Zeile selbst (genauer: ihr letzter **Peer**).<br>**Reparatur:** Frame explizit aufziehen:<br>```sql<br>last_value(x) OVER (<br>  PARTITION BY k ORDER BY d<br>  ROWS BETWEEN UNBOUNDED PRECEDING<br>            AND UNBOUNDED FOLLOWING)<br>```<br>`first_value()` ist unproblematisch — die erste Zeile liegt immer im Frame. Praxis-Trick: `first_value` mit umgekehrtem `ORDER BY`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsO1AWtSczpa1jXqxLE177r" -->
Unterschied **`ROWS`** vs. **`RANGE`** im Frame — mit Beispiel.::
**`ROWS`** zählt **physische Zeilen** („2 zurück" = genau 2 Zeilen).<br>**`RANGE`** zählt **Werte**: Alle Zeilen mit demselben `ORDER BY`-Wert (*Peers*) gehören gemeinsam zum Frame — alle oder keine.<br>**Beispiel** (01.03.: 100 · 02.03.: 200 · 02.03.: 300), laufende Summe:<br>`ROWS` → 100, 300, 600<br>`RANGE` → 100, **600, 600**<br>**`RANGE` ist der Default** — bei laufenden Summen über Datumswerte mit Dubletten also `ROWS` explizit hinschreiben, sonst springt die Summe.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYm3cyi2VoXYNzKXzFx72v" -->
`row_number()` vs. `rank()` vs. `dense_rank()` — Verhalten bei Gleichstand **und** je ein Einsatzfall.::
**`row_number()`** → 1, 2, 3, 4 — vergibt immer verschiedene Nummern, bei Gleichstand **nicht deterministisch**. *Einsatz:* **deduplizieren**, „genau eine Zeile pro Gruppe" (`WHERE rn = 1`).<br>**`rank()`** → 1, 2, 2, **4** — gleiche Nummer, danach Lücke. *Einsatz:* echte **Wettbewerbs-Rangliste**.<br>**`dense_rank()`** → 1, 2, 2, **3** — gleiche Nummer, keine Lücke. *Einsatz:* „die drei besten **Werte**" (nicht Zeilen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskchGffbQyRazcDOnclXLT" -->
Warum ist `WHERE row_number() OVER (…) <= 3` ein Fehler? Nenne die logische Ausführungsreihenfolge.::
`FROM → WHERE → GROUP BY → HAVING → `**`⟨Window Functions⟩`**` → SELECT → DISTINCT → ORDER BY → LIMIT`<br>Window Functions laufen **nach** `WHERE`/`HAVING` — dort existieren sie noch nicht.<br>Zweite Folge: Sie sehen **nur, was `WHERE` übrig gelassen hat**. Ein `WHERE jahr = 2026` verändert also auch alle Window-Ergebnisse — manchmal gewollt, manchmal ein subtiler Bug.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszFHCbLbAu4EropTJXDZ32" -->
Wie funktioniert die **`FILTER`**-Klausel, und was ist die ältere Alternative?::
```sql<br>COUNT(*) FILTER (WHERE status = 'paid')<br>SUM(betrag) FILTER (WHERE status = 'paid')<br>```<br>Sie schränkt **eine einzelne Aggregatfunktion** auf passende Zeilen ein.<br>**Ältere, portablere Alternative:** `COUNT(CASE WHEN status='paid' THEN 1 END)` — funktioniert ohne `ELSE`, weil `COUNT` NULLs ignoriert.<br>**Feiner Unterschied:** `SUM(...) FILTER` ohne Treffer → **NULL**; `SUM(CASE … ELSE 0 END)` → **0**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8Gt1PuhuMSErgAvKIvTkm" -->
Warum drei Kennzahlen mit `FILTER` in **einer** Query statt in drei Queries?::
1. **Ein Tabellen-Scan statt drei.** Bei 200 Mio. Zeilen der Unterschied zwischen einem Durchlauf und dreien — und im Cloud-Warehouse zahlt man **pro gescanntem Byte**.<br>2. **Garantiert konsistent** — alle Zahlen stammen aus demselben Snapshot.<br>3. **Keine Joins nötig**, um die Einzelergebnisse wieder zusammenzuführen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDu0jcgrrueilhlEXeA5ls" -->
Nenne sechs **Python-Schleifenmuster** und ihr SQL-Gegenstück.::
`for x in rows: x.wert * 1.19` → **Ausdruck im `SELECT`**<br>`if/elif/else` in der Schleife → **`CASE WHEN … THEN … ELSE … END`**<br>`total += x.betrag` → **`SUM(...)`** bzw. `SUM(...) OVER (…)` für den laufenden Stand<br>`if x.status=='paid': count += 1` → **`COUNT(*) FILTER (WHERE …)`**<br>Vergleich mit `rows[i-1]` → **`lag(...) OVER (ORDER BY …)`**<br>`for a: for b: if a.id==b.id` → **`JOIN`**<br>`for produkt in produkte: query(produkt)` (**N+1**) → **eine** Query mit `PARTITION BY produkt`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnRa7QS2r3BGDnchRmNQ3f" -->
Warum hat SQL **keine Schleife** — und was ist die richtige Frage stattdessen?::
Weil **jede Operation implizit auf allen Zeilen läuft**. `SELECT betrag * 1.19 FROM rechnungen` **ist** bereits die Schleife („für jede Zeile"). Man schreibt nur das *Was*, die Datenbank bestimmt das *Wie*.<br>**Falsche Frage:** „Wie schreibe ich diese Schleife in SQL?"<br>**Richtige Frage:** „**Welche Mengenoperation ersetzt diese Schleife?**"<br>Besonders teuer ist das **N+1-Muster** — eine Query pro Schleifendurchlauf bedeutet bei 10.000 Elementen 10.000 Netzwerk-Roundtrips statt einem.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsA1Iy5c0WhVyb43M8yKbpz" -->
Nenne **drei Wege** zu „pro Kunde die letzte Bestellung" und ihre Unterschiede.::
**1. `DISTINCT ON` (Postgres):** `SELECT DISTINCT ON (kunde_id) … ORDER BY kunde_id, datum DESC` — am kürzesten, aber nicht portabel und nur „die eine".<br>**2. Window:** `row_number() OVER (PARTITION BY kunde_id ORDER BY datum DESC)`, außen `WHERE rn = 1` — portabel; mit `rn <= 3` bekommt man die letzten drei, mit `rank()` bei Gleichstand **alle**.<br>**3. `LEFT JOIN LATERAL (… ORDER BY … LIMIT 1) ON true`** — behält auch Kunden **ohne** Bestellung.<br>⚠️ **Nicht** `ORDER BY datum DESC LIMIT 1` — das liefert **eine** Zeile für die ganze Tabelle, nicht eine pro Kunde.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEAnWAgs1Rv43v44ApWSjZ" -->
Warum ist `row_number()` bei Gleichständen ein Risiko in Pipelines — und wie behebt man es?::
Bei Gleichstand im `ORDER BY` ist die Zuordnung **willkürlich** und kann zwischen zwei Läufen **unterschiedlich** ausfallen. Eine Pipeline, die damit dedupliziert, ist dann **nicht reproduzierbar** — Verstoß gegen Idempotenz (Stichwort 5).<br>**Behebung:** eindeutigen Tiebreaker ergänzen: `ORDER BY datum DESC, id DESC`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNWpBwuCcTJvmWOhQV3u9f" -->
Was macht **SQL** so besonders, dass es seit über 30 Jahren dominiert? (4 Gründe)::
1. **Deklarativ** — man beschreibt das Ergebnis, der **Query-Planer** wählt Zugriffspfade und Join-Reihenfolge nach aktuellen Statistiken. Dieselbe Query wird schneller, wenn ein Index dazukommt, **ohne Codeänderung**.<br>2. **Stabiler Standard** — SQL-92-Queries laufen heute noch; das Wissen veraltet nicht.<br>3. **Richtige Abstraktionsebene** — die relationale Algebra trifft genau das, was man mit Tabellen tun will. Deshalb liegen Pandas, Polars und Spark nah an SQL.<br>4. **Zugänglich über die Entwicklerrolle hinaus** — Analysten schreiben SQL; das ermöglicht die Arbeitsteilung, die ELT attraktiv macht.<br>**Preis:** Man kann den Ausführungsplan nicht direkt steuern — Fehlersuche läuft indirekt über `EXPLAIN`.
<!-- basicblock-end -->

---

# Block A — Deklarativ, funktional, Idempotenz

*Priorisierung: Der Review zeigte, dass Querverweise zu bereits Gelerntem nicht hergestellt werden. Am Ende dieses Blocks stehen deshalb bewusst mehrere **Rückbezugskarten**.*

<!-- basicblock-start oid="Obsd1xd1W5K2KhOhluRpag4F" -->
Was unterscheidet eine **deklarative** von einer **imperativen** Sprache? Erkläre am CSS-Beispiel.::
**Imperativ sagt, *wie* man zum Ziel kommt. Deklarativ sagt, *was* das Ziel ist.**<br>**CSS (deklarativ):** `li.selected > p { background-color: blue; }`<br>**JavaScript (imperativ):** verschachtelte Schleifen über alle `li`, Klassenprüfung, `setAttribute` auf jedem `p`.<br>**Der versteckte Unterschied:** JS setzt das Attribut **direkt** — wird ein `li` deselektiert, bleibt das Blau stehen. CSS beschreibt eine **Regel**, die der Browser fortlaufend durchsetzt.<br>Formale Grundlage von SQL ist die **relationale Algebra**: `σ_{family='Sharks'}(animals)`. Historisch nicht selbstverständlich — IMS und CODASYL wurden noch **imperativ** abgefragt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3ZCx0w4qRW24Ovkz4sqsY" -->
Warum kann dieselbe SQL-Query **schneller werden, ohne dass man sie ändert**? Was weiß der Planer, das du nicht wusstest?::
Weil du nur das *Was* festlegst, bleibt das *Wie* frei und wird **zur Laufzeit** entschieden. Der Planer kennt dann:<br>· **aktuelle Tabellengrößen** (1.000 Zeilen beim Schreiben, heute 50 Mio.)<br>· **Wertverteilungen/Selektivität** aus `ANALYZE`-Statistiken<br>· **verfügbare Indizes** (auch neue)<br>· **was im Cache liegt** · **Kerne und RAM des Servers**<br>**Folge:** Dieselbe Query nimmt bei 1.000 Zeilen einen Seq Scan und bei 50 Mio. einen Index Scan — ohne Codeänderung. Plus: jede Planer-Verbesserung in neuen Postgres-Versionen wirkt gratis.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsb9PqeoEm6UIcpCHSllly8" -->
Was genau verhindert bei einer `for`-Schleife die **Parallelisierung**? (2 Gründe)::
Sie legt eine Reihenfolge fest, die sie meist gar nicht braucht — aber die Sprache kann das nicht unterscheiden.<br>1. **Sequentielle Semantik:** `for i in range(n)` heißt ausdrücklich erst `i=0`, dann `i=1`. Ein Compiler dürfte nur umordnen, wenn er Unabhängigkeit **beweisen** kann — im Allgemeinen unentscheidbar.<br>2. **Zustand über Iterationen:** `total += x` erzeugt eine echte Abhängigkeitskette (Iteration n braucht n−1).<br>**SQL macht keine Zusage über die Zeilenreihenfolge** (ohne `ORDER BY`) — das ist die Erlaubnis, zu zerlegen, zu verteilen und Teilergebnisse zu mergen. Funktioniert bei `SUM`, weil assoziativ und kommutativ.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0u60N86BU6RbrWR7ei5MW" -->
Nenne drei **Nachteile** deklarativer Programmierung — und drei Situationen, in denen **imperativ** besser ist.::
**Nachteile:** 1. Schlechte Planer-Entscheidungen sind nicht direkt korrigierbar (Postgres hat bewusst **keine Query-Hints**) · 2. **Performance-Klippen** — eine kleine Änderung der Datenverteilung kippt den Plan · 3. Fehlersuche ist indirekt (`EXPLAIN` statt Debugger) · 4. begrenzte Ausdrucksstärke.<br>**Imperativ gewinnt bei:** Logik pro Entität in Echtzeit · echter sequentieller Abhängigkeit (iterative Verfahren, Simulationen) · Aufrufen externer Dienste (Geokodierung, ML-Inferenz) · komplexen Kontrollflüssen · Parsen exotischer Formate.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ATc1ABWbQGRY5DKJ5vXd" -->
Unterschied: **schlechter Query-Plan** vs. **N+1-Problem**?::
Zwei verschiedene Abstraktionsebenen mit zwei Ausfallarten.<br>**Query-Planer** optimiert *innerhalb* **einer** Query. Er kann einen schlechten Plan wählen, setzt aber nie zusätzliche Queries ab. → Hilfe: `EXPLAIN`, `ANALYZE`, Umformulieren.<br>**ORM** entscheidet über die **Anzahl** der Queries. Bei N+1 werden aus einem Attributzugriff in einer Schleife 100 einzelne `SELECT`s — jedes für sich perfekt geplant. → Hilfe: Query-Log lesen, `selectinload`/`joinedload`.<br>**Pointe:** SQL ist deklarativ, ein **ORM ist es nicht** — es ist eine imperative Fassade vor einer deklarativen Sprache. Genau daher rührt das Problem.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8ijc0BqQW86b3QNPHOdjW" -->
Zeilenorientiert vs. mengenorientiert denken — Stärken, Schwächen, Einsatzbereich.::
**Zeilenorientiert:** gedankliche Einheit ist *ein* Objekt („nimm diese Bestellung, prüfe, berechne, speichere").<br>*Stark:* Fachlogik am Einzelfall lesbar · beliebig komplexe Verzweigung · Breakpoint-Debugging. *Schwach:* skaliert linear · N+1-Gefahr · Interpreter-Overhead pro Zeile. *Passt zu:* Backend, APIs, Echtzeit.<br>**Mengenorientiert:** gedankliche Einheit ist die *ganze Tabelle* („alle Bestellungen mit Status X bekommen Rabatt Y").<br>*Stark:* skaliert · parallelisierbar · optimierbar · viel weniger Code. *Schwach:* ungewohnt · Fehler treffen alles auf einmal · Debugging über Zwischenergebnisse. *Passt zu:* ETL, Reporting, alles ab ~10.000 Zeilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsf7XT0oTTyVCNhoTdjt7A5" -->
Was macht eine Funktion **rein** (*pure*)? Nenne die zwei Bedingungen mit je einem Gegenbeispiel.::
**1. Der Rückgabewert hängt ausschließlich von den Argumenten ab** — kein Zugriff auf Globals, Uhrzeit, Zufall, DB, Dateisystem, Netzwerk.<br>*Verletzt:* `def brutto(netto): return netto * (1 + MWST)` (globale Variable) · `def ist_abgelaufen(d): return d < date.today()` (Uhrzeit)<br>**2. Keine Seiteneffekte** — verändert nichts außerhalb ihrer selbst.<br>*Verletzt:* `def rabatt(bestellung): bestellung.summe *= 0.9` (verändert das übergebene Objekt)<br>**Rein:** `def ist_abgelaufen(datum, stichtag): return datum < stichtag` — **Zeit wird nicht gelesen, sondern übergeben.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYUm0GeCyvpAfk8qChP9Ve" -->
Warum ist eine **reine Funktion so viel leichter testbar**? Leite es aus den zwei Bedingungen ab.::
**Aus Bedingung 1 (nur Argumente) folgt: kein Setup.** Keine Testdatenbank, keine Beispieldateien, keine Umgebungsvariablen, **keine Fixtures**. Der Testinput *ist* das Argument.<br>**Aus Bedingung 2 (keine Seiteneffekte) folgt: kein Mocking, kein Teardown.** Mocks brauchte man in Kap. 1.3, um DB, Netzwerk und Uhrzeit zu ersetzen — **eine reine Funktion berührt nichts davon.**<br>**Zusätzlich:** keine Flakiness · `pytest -n auto` problemlos · `@parametrize` mit 20 Fällen trivial.<br>**Konsequenz für ETL:** Transformationslogik so schneiden, dass der **rechnende Kern rein** ist und Lesen/Schreiben an den Rändern sitzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYzhXzvJvro5TZfgyw0Ai6" -->
Warum ist funktionale Programmierung im **Batch-Processing** verbreiteter als im Backend? (Nicht zirkulär antworten!)::
**Nicht** weil Batch-Jobs funktional sein *sollten* — sondern weil **Batch-Arbeit ihrer Natur nach eine reine Transformation ist und Backend-Arbeit nicht.**<br>Backend **verwaltet Zustand** (Warenkörbe, Sitzungen, Kontostände) — der Zustand *ist* das Produkt, Seiteneffekte sind der Sinn der Sache. Batch **leitet Zustand ab**: aus Input X wird Output Y.<br>Drei konkrete Gründe:<br>1. **Automatische Retries erzwingen Reinheit** — ein Task mit Seiteneffekten wird zweimal ausgeführt und erzeugt Doppelbuchungen<br>2. **Verteilung erzwingt Zustandsfreiheit** — gemeinsamer veränderlicher Zustand bräuchte Koordination, und die verhindert Skalierung<br>3. **Wiederberechnung ist der Normalfall** — ETL-Logik läuft über Jahre immer wieder auf dieselben historischen Daten
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs91XTMnGeBr66JuoNEG8Pq" -->
Was heißt **Immutability** in einer Datenpipeline — was genau wird nicht verändert?::
**Was einmal geschrieben wurde, wird nicht mehr verändert.** Neue Erkenntnisse führen zu *neuen* Daten, nicht zu geänderten alten.<br>· **Der Input wird nie modifiziert** — Quelldaten bleiben, wie sie sind<br>· **Geschriebene Ausgaben werden nicht zeilenweise editiert** — muss sich etwas ändern, wird die **Partition komplett neu erzeugt und ersetzt**<br>Beauchemin: *„immutable data along with versioned logic are key to reproducibility."*
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxkXIFfCfoGLS09VNRhMWX" -->
Warum ist die **Partition** und nicht die **Zeile** die richtige atomare Einheit? Welche SQL-Operationen ändern sich?::
Eine Partition ist meist ein Zeitausschnitt („alle Bestellungen vom 2026-09-01"). Sie wird **im Ganzen neu berechnet und ersetzt**.<br>**`UPDATE`/`DELETE` auf einzelnen Zeilen verschwinden** — es gibt keine Zeilenkorrektur, nur Neuerzeugung der Partition.<br>⚠️ **`INSERT` fällt nicht weg**, es ändert seine Bedeutung: nicht „Zeile anhängen", sondern „diese Partition befüllen".<br>**Vorteile:** idempotent per Konstruktion · kein Zwischenzustand · leicht prüfbar · Backfill = Schleife über Partitionen · versionierbar.<br>**Preis:** Man rechnet mehr als nötig — eine geänderte Zeile bedeutet 10 Mio. neu geschriebene. Deshalb ist **billiges Cloud-Compute die Voraussetzung** dieses Musters.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAqpvaL8fKrKVajXz2Ct9i" -->
Was bedeutet **idempotent** — mathematisch und für einen ETL-Job?::
**Mathematisch:** `f(f(x)) = f(x)` — mehrfache Anwendung ändert nichts gegenüber einmaliger.<br>**ETL-Job:** Ein zweiter (dritter, zehnter) Lauf **mit denselben Parametern** hinterlässt das System im **selben Zustand** wie der erste. Nicht „er stürzt nicht ab" — sondern **das Ergebnis ist identisch**.<br>Die Präzisierung „mit denselben Parametern" ist wichtig: Ein Job für den 2026-09-01 muss beliebig oft für den 2026-09-01 laufen können; dass er für den 02. etwas anderes tut, ist kein Widerspruch.<br>*Alltag:* Lichtschalter = **nicht** idempotent, Schalter mit fester Position „an" schon. HTTP: `GET`/`PUT`/`DELETE` ja, `POST` nein.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCU86dlyvE0ibyjKJAhvIR" -->
Nenne fünf **Determinismus-Fallen**, bei denen der Code deterministisch aussieht und es nicht ist.::
1. **„Lies die neuesten Daten"** → immer eine **feste Partition** lesen, parametrisiert über das Ausführungsdatum<br>2. **`now()` / `CURRENT_DATE`** → Stichtag als **Parameter hereinreichen**; sonst rechnet ein Backfill für letzten Monat mit dem heutigen Datum<br>3. **`random()` / `uuid4()`** → Seed setzen oder Schlüssel deterministisch aus den Daten hashen<br>4. **`row_number()` ohne eindeutigen Tiebreaker** → `ORDER BY datum DESC, id DESC`<br>5. **Zeitzonen / lokale Systemzeit** → alles UTC, Zeitzone explizit<br>6. **Externe API-Aufrufe** → Antworten in den Roh-Layer materialisieren<br>**Faustregel:** Ein Job ist idempotent, wenn sein Ergebnis **ausschließlich** von seinen Parametern und dem unveränderlichen Roh-Layer abhängt. Das ist die Bedingung einer **reinen Funktion**, nur auf Job-Ebene.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZIxrWcycSjGculMjUsRdh" -->
Nenne die **drei Schreibmuster** für idempotente Jobs und sag, wann welches passt.::
**1. Partition Overwrite** — `DELETE WHERE tag = X; INSERT … WHERE tag = X`. Idempotent per Konstruktion, kein Schlüsselabgleich nötig. *Wann:* natürliche Zeitpartitionierung (ETL-Normalfall).<br>**2. UPSERT** — `INSERT … ON CONFLICT (id) DO UPDATE SET …` (bzw. `MERGE` ab PG 15). Braucht einen verlässlichen Unique-/Primärschlüssel. *Wann:* Datensätze werden über Partitionsgrenzen hinweg aktualisiert (Stammdaten, SCDs).<br>**3. `DELETE` + `INSERT` in EINER Transaktion** — `BEGIN; DELETE …; INSERT …; COMMIT;`. *Wann:* Postgres ohne echte Partitionierung — in der Praxis oft.<br>⚠️ **Das gemeinsame `BEGIN`/`COMMIT` ist entscheidend:** Ohne Transaktion gibt es ein Fenster, in dem die Daten gelöscht, aber noch nicht neu geschrieben sind — stürzt der Job dort ab, sind sie weg.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5vMJz5BFJd80LuBPrxv1U" -->
„Behandle einen Task wie eine Transaktion" — was heißt das, und **wie** stellt man es praktisch sicher?::
**Was:** Nach einem Absturz muss entweder der **vollständige neue** Zustand da sein oder der **unveränderte alte** — nie eine Mischung. Das ist die **Atomarität** aus ACID, auf einen ganzen Job angewendet.<br>**Warum kritisch:** Eine halb gefüllte Tabelle sieht **völlig normal aus** — jeder Bericht rechnet mit 60 % der Daten weiter (stiller Fehler).<br>**Wie:**<br>· *kleine Tabelle:* alles in **einer Transaktion**<br>· *große Tabelle:* in **Staging-Tabelle** schreiben, dann atomar umschalten (`RENAME`, `ATTACH PARTITION`) — der teure Teil außerhalb der Transaktion<br>· *Dateien/Data Lake:* in **temporären Pfad** schreiben, am Ende umbenennen<br>· *Iceberg/Delta:* atomare Commits eingebaut<br>Das Muster heißt **Write-Audit-Publish**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsl5YBIlKAM8th6Krf4c9sZ" -->
Die drei **Zustellgarantien** — und warum echtes Exactly-once unmöglich ist.::
**At-most-once:** höchstens einmal, im Zweifel gar nicht → Risiko **Datenverlust**. Nur wo Verlust tolerierbar ist.<br>**At-least-once:** mindestens einmal, im Zweifel mehrfach → Risiko **Duplikate**. Der Normalfall.<br>**Exactly-once:** wirkt sich genau einmal aus → das Wunschziel.<br>**Warum unmöglich:** Erhält ein Sender keine Bestätigung, kann er **nicht unterscheiden**, ob die Nachricht nicht ankam oder nur die Bestätigung verlorenging. Er muss wählen: nochmal senden (Duplikat) oder aufgeben (Verlust). Ein drittes Verhalten gibt es nicht.<br>**Was „Exactly-once" praktisch heißt: *effectively-once* = at-least-once + Idempotenz beim Empfänger.** Man verhindert Duplikate nicht — man macht sie **wirkungslos**. Deshalb ist `UPSERT` statt `INSERT` keine Stilfrage, sondern die Umsetzung einer Unmöglichkeitsaussage.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsC4I0qFod0DQfYzmj7ql68" -->
**Event Time** vs. **Processing Time** — und was ist ein **Watermark**?::
**Event Time:** wann das Ereignis tatsächlich passiert ist. **Processing Time:** wann die Pipeline es gesehen hat.<br>Fast immer partitioniert man nach **Event Time**, weil das die fachliche Wahrheit ist. Dann müssen alte Partitionen nachträglich korrigierbar sein — mit Partition Overwrite trivial.<br>**Watermark:** die Aussage *„Ereignisse vor Zeitpunkt T sind jetzt vollständig da."* Alles Spätere gilt als verspätet und wird verworfen, gesondert abgelegt oder löst eine Neuberechnung aus.<br>Im Batch entspricht das der pragmatischen Regel, **die letzten N Tage bei jedem Lauf mit neu zu rechnen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJrMaJl26DlogkYXAmr7y9" -->
Was ist ein **Backfill**, und warum ist er der Härtetest für Idempotenz?::
Das nachträgliche **Neuberechnen historischer Partitionen** — nach einem Bugfix, einer Logikänderung oder wenn eine neue Spalte dazukommt. Im Kern eine Schleife über Partitionen, jede einzeln neu erzeugt.<br>**Härtetest, weil:** Ein Job, der nur „vorwärts" funktioniert — weil er `CURRENT_DATE` liest oder an bestehende Daten anhängt — lässt sich **nicht rückwirkend** laufen lassen.<br>Und man merkt es genau dann, wenn man es am dringendsten braucht: wenn drei Monate Zahlen falsch sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbnJBg6FKP4wUkZ2uuBmYs" -->
Was ist ein **Idempotency Key**, und was hat er mit `UPSERT` zu tun?::
**Problem:** `POST` ist nicht idempotent. Geht die Antwort auf eine Zahlung im Netzwerk verloren, weiß der Client nicht, ob sie durchging — sendet er erneut, droht eine Doppelbuchung.<br>**Lösung (Stripe-Standard):** Der Client erzeugt einen eindeutigen Schlüssel und schickt ihn im Header. Der Server speichert Schlüssel **und Antwort**. Beim zweiten Mal führt er die Operation **nicht erneut aus**, sondern liefert die gespeicherte Antwort.<br>**Der Zusammenhang:** Exakt dasselbe Muster wie `UPSERT` in ETL — **At-least-once-Zustellung wird durch empfängerseitige Deduplizierung wirkungslos gemacht.** Nur heißt der Schlüssel dort Primärschlüssel und hier Idempotency Key.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsO8OpEx1LZ5Tnh1S33MnVW" -->
**Reproducibility** vs. **Recomputability** — Unterschied, Notwendigkeit, und welche Architekturentscheidung Letzteres ermöglicht.::
**Reproducibility:** derselbe Job, dieselbe Logik, derselbe Input → dasselbe Ergebnis. Frage: *Kann ich zeigen, dass diese Zahl richtig zustande kam?* Es geht um **Vertrauen**.<br>**Recomputability:** historische Daten mit **geänderter** Logik neu berechnen. Frage: *Kann ich einen Fehler rückwirkend korrigieren?* Es geht um **Korrigierbarkeit**.<br>**Warum beides:** Reproduzierbarkeit allein bedeutet nur, dass die falsche Zahl **zuverlässig falsch** ist. Fachliche Definitionen ändern sich, Bugs laufen monatelang unentdeckt.<br>**Die Architekturentscheidung: ELT und der persistente Roh-Layer.** Bei ELT bleiben die Rohdaten liegen → man transformiert neu. Bei ETL müsste man aus den Quellsystemen neu extrahieren, die historische Zustände nicht vorhalten — die Vergangenheit ist in der damaligen Logik eingefroren.<br>Beauchemin: **unveränderliche Daten + versionierte Logik** = beides zusammen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEKg61GvQ4UwfKdhRX35dE" -->
**Rückbezug:** Wo ist dir Idempotenz in diesem Kapitel schon begegnet, bevor sie so hieß? (2 Stellen)::
**1. Stichwort 1 — `access.log`.** Die Unix-Pipeline `cat access.log | awk … | sort | uniq -c` **verändert die Datei nicht**. Deshalb kann man sie beliebig oft laufen lassen, immer mit demselben Ergebnis. In Abschnitt 1.1 wörtlich als *„die praktische Definition von Idempotenz"* bezeichnet.<br>**2. Stichwort 2 — `row_number()` ohne Tiebreaker.** Bei Gleichstand ist die Zuordnung willkürlich und kann zwischen Läufen variieren. Eine Pipeline, die damit dedupliziert, ist **nicht reproduzierbar**.<br>*(Diese Karte existiert, weil genau dieser Rückbezug im Selbsttest nicht hergestellt wurde.)*
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRO2Tc0X4NdAQaHadVcs1y" -->
**Rückbezug:** Zeichne die Kette, die Stichwort 1 bis 5 verbindet.::
```text<br>deklarativ    →  ich sage WAS, nicht WIE<br>   ↓ funktioniert nur, weil …<br>funktional    →  keine Seiteneffekte,<br>                 Ergebnis hängt nur vom Input ab<br>   ↓ und daraus folgt …<br>idempotent    →  beliebig wiederholbar<br>   ↓ plus unveränderliche Rohdaten …<br>reproduzierbar + wiederberechenbar<br>```<br>Die drei Stichworte sind **ein** Gedankengang, keine drei Themen. Und die Basis darunter — der unveränderliche Roh-Layer — ist die **ELT**-Entscheidung aus Stichwort 1.
<!-- basicblock-end -->

---

# Testing in ETL

*Priorisierung: Die drei roten Punkte des Selbsttests (Generic Tests, `given:`-Block, Circuit Breaker) stehen vorn.*

<!-- basicblock-start oid="ObshKhliO4rUnY3idFAmgSJb" -->
Nenne die **vier eingebauten Generic Tests** von dbt und je einen Einsatzfall.::
`unique` — keine Duplikate → **Primärschlüssel**, Bestellnummer, E-Mail<br>`not_null` — keine NULL-Werte → **Pflichtfelder**, Fremdschlüssel, Beträge<br>`accepted_values` — nur Werte aus einer Liste → **Status** (`open`/`paid`/`cancelled`), Ländercodes<br>`relationships` — jeder Wert existiert in der Referenztabelle → **referenzielle Integrität**<br>⚠️ `relationships` ist besonders wichtig, weil analytische Warehouses oft **keine Foreign Keys** haben (zu teuer beim Bulk-Laden). Was im OLTP-System die Datenbank garantierte, muss hier ein Test übernehmen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCYj6CQVBCjdI5Zn50Tgmn" -->
Was ist ein **Circuit Breaker** in einer Datenpipeline? Woher der Name, welches Problem?::
Ein Check, der die Pipeline **anhält**, sobald die Daten eine Qualitätsschwelle unterschreiten.<br>**Name:** vom Sicherungsautomaten — er unterbricht den Stromkreis, **bevor** Schaden entsteht.<br>**Problem:** Schlechte Daten sollen sich nicht nach **downstream** ausbreiten. Ohne ihn rechnen 20 nachgelagerte Modelle, 3 Dashboards und ein ML-Modell weiter — und man repariert hinterher alles einzeln. Mit ihm steht die Pipeline und man repariert **eine** Stelle.<br>**In Airflow:** `SQLColumnCheckOperator` (Spaltenebene) und `SQLTableCheckOperator` (aggregierte Regeln). Task schlägt fehl → alle nachgelagerten laufen nicht an.<br>**Abwägung:** `error` nur auf Invarianten, bei denen Weiterrechnen schlimmer ist als Stillstand — sonst `warn`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss7TjLv0Dj5UxbrMT4ov53" -->
Was ist die Parallele zwischen einem **dbt Unit Test** und einer **pytest-Fixture**?::
Der **`given:`-Block ist die Fixture.** Man schreibt die Eingabezeilen von Hand hin:<br>```yaml<br>unit_tests:<br>  - name: test_rabattlogik<br>    model: bestellungen_aufbereitet<br>    given:<br>      - input: ref('bestellungen_roh')<br>        rows:<br>          - {betrag: 100, kundentyp: 'premium'}<br>    expect:<br>      rows:<br>        - {betrag_final: 90}<br>```<br>Genau die Idee von `@pytest.fixture`: kleiner, kontrollierter, handgeschriebener Datensatz für deterministisches Prüfen. `given:` auf ein Upstream-Modell = **Mocking einer Abhängigkeit**.<br>Funktioniert nur, weil die Transformation eine **reine Funktion** ihrer Inputs ist — läse sie `now()`, wäre sie mit Fixtures nicht testbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2dSdgYSYelMR0xO4261vN" -->
Die **zwei Achsen** des Testens in Datenpipelines — was bewegt sich worauf, und was löst jeweils einen Testlauf aus?::
**Code-Ebene (Y):** Der Code wandert durch Umgebungen (lokal → CI → Staging → Prod). Werkzeuge: Unittests, TDD, Mock-Daten. **Ausgelöst durch eine Codeänderung.**<br>**Daten-Ebene (X):** Die Daten fließen durch Transformationen (Quelle → Roh → Zwischenmodelle → Auswertung). Werkzeuge: Data-Quality-Checks. **Ausgelöst dadurch, dass neue Daten ankommen.**<br>**Warum eine grüne Unittest-Suite trotzdem eine kaputte Pipeline bedeuten kann:** Sie prüft nur, ob der **Code** tut, was er soll — nicht, ob die **Daten** in Ordnung sind. Die Transformation kann fehlerfrei sein und trotzdem Unsinn liefern, weil die Quelle seit gestern Cent statt Euro schickt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLOhfbH0ABpcmu7tgq7S5i" -->
Was ist **Pipeline Debt**, und wodurch unterscheidet sie sich von gewöhnlicher Technical Debt?::
Eine Sonderform der Technical Debt mit drei Merkmalen: **undokumentiert** (niemand weiß, was eine Spalte bedeutet) · **ungetestet** (keine geprüften Annahmen über die Daten) · **instabil** (bricht bei Änderungen stromaufwärts).<br>**Der Unterschied:** Gewöhnliche Technical Debt steckt im **Code** — man kann ihn lesen und die Schuld sehen. Pipeline Debt steckt in **unverifizierten Annahmen über die Daten**. Der Code kann tadellos sein; die Schuld ist trotzdem da und **unsichtbar**, bis jemand die Zahlen anzweifelt.<br>GX-Kernsatz: *Die Komplexität einer Pipeline steckt in den Daten, die sie transportiert, nicht in ihrer Struktur.*
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWbEOsRn3VN0cLH30m0CgK" -->
Warum kann ein Test, der **beim Deploy** läuft, eine ganze Fehlerklasse prinzipiell nicht abfangen?::
Weil die **Auslöser entkoppelt** sind: Deploy-Tests laufen, wenn sich der **Code** ändert. Datenprobleme entstehen, wenn **neue Daten ankommen** — kontinuierlich, oft Monate nach dem letzten Deploy.<br>Ein Test, der nur an Ereignis A hängt, kann Fehler aus Ereignis B nicht sehen, egal wie gut er ist.<br>**Konsequenz:** Data-Quality-Checks müssen bei **jedem Pipeline-Lauf** mitlaufen, nicht bei jedem Deploy.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsS1OSPzVaaHnlQi92jH1Ci" -->
Wie ist ein **dbt Data Test** technisch aufgebaut — und was ist der Bezug zur deklarativen Programmierung?::
**Ein `SELECT`, das die *fehlerhaften* Zeilen zurückgibt. Null Zeilen = bestanden.**<br>Für NOT NULL fragt man also die **NULLs ab**:<br>```sql<br>SELECT * FROM {{ ref('bestellungen') }}<br>WHERE kunde_id IS NULL<br>```<br>Vorteil: Bei Fehlschlag hat man **direkt die betroffenen Zeilen** (`--store-failures` persistiert sie).<br>**Deklarativ, weil:** Man deklariert eine **Invariante** — was wahr sein muss —, statt eine Prüfprozedur zu programmieren. Genau Kleppmanns *„specify the pattern of the data you want"*. Und daraus folgen dieselben Vorteile: skaliert auf beliebige Datenmengen, wird optimiert und parallelisiert, ohne dass man ihn anfasst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ssV00Pc0ERTiYYqPvODI" -->
**Unit Test** vs. **Model Contract** vs. **Data Test** — Zeitpunkt, Input, Zweck.::
**Unit Test:** **vor** der Materialisierung · statische **Fixture-Daten** · prüft die **Transformationslogik**, exakter erwarteter Output.<br>**Model Contract:** **während** der Materialisierung · das Modell selbst · prüft die **Form** des Containers — Spaltennamen, Datentypen, Constraints.<br>**Data Test:** **nach** der Materialisierung · die **real materialisierten Daten** · prüft **Assertions über Datensätze** — Wertebereiche, Eindeutigkeit, Beziehungen.<br>**Blockieren:** Unit Test → Bau des Modells · Contract → Modell **und** nachgelagerte · Data Test → nachgelagerte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNwFOxGgfJzm6Bxczbdogn" -->
Wann lohnt sich ein **Unittest** in ETL — und wann ist er Aufwand ohne Nutzen?::
**Lohnt sich, wenn die Logik eigenständig falsch sein kann:**<br>· **komplexe Ausdrücke** — Regex, Datumsarithmetik, verschachtelte `CASE`, Window Functions mit Frames<br>· **Regression nach Bugfix** — der Fehlerfall wird als Fixture festgeschrieben<br>· **Edge Cases, die in echten Daten nicht vorkommen** — Schaltjahre, negative Beträge, Zeitzonengrenzen. Ein Data Test kann sie nicht finden, sie sind ja nicht da<br>· **Refactoring-Netz**<br>**Nutzlos bei:** tautologischen Tests (`rev = brutto − netto` definieren und dann prüfen, ob `rev = brutto − netto`) · trivialen Transformationen · wenn eigentlich **die Daten** das Problem sind → dann braucht es einen **Data Test**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGv2Ykx4mwTLMaynvEhpkW" -->
Welche **drei Reaktionen** auf einen fehlgeschlagenen Data-Quality-Check gibt es?::
**Warnen** (`severity: warn`) — loggen, alarmieren, **weiterlaufen**. Für weiche Kriterien, wo ein Ausreißer normal sein kann.<br>**Blockieren** (`severity: error`) — Task schlägt fehl, **nachgelagerte Schritte laufen nicht an**. Für harte Invarianten (Primärschlüssel doppelt, Pflichtfeld leer). Das ist der **Circuit Breaker**.<br>**Quarantäne** — fehlerhafte Zeilen aussortieren, den Rest verarbeiten, Ausschüsse gesondert ablegen. Wenn Teilverarbeitung besser ist als gar keine.<br>Ein Check, der fehlschlägt und **nichts bewirkt**, ist wertlos.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdmcjyVP0YFlkZQ1vIJajm" -->
An welchen **drei Stellen** der ETL-Strecke setzt man Checks an, und wonach fragt man jeweils?::
**Ingestion** — *Ist angekommen, was ankommen sollte?* Zeilenzahl in erwarteter Größenordnung · Partition vorhanden · **Schema unverändert** · Aktualität.<br>**Transformation** — *Hat die Logik getan, was sie soll?* Schlüssel eindeutig · keine unerwarteten NULLs nach einem Join · Zeilenzahl vor/nach dem Join plausibel (**Join-Fanout**) · Wertebereiche.<br>**Output** — *Ist das Ergebnis fachlich plausibel?* Summen gegen unabhängige Quelle · Kennzahlen im Korridor · keine Sprünge gegenüber der Vorperiode.<br>**Die wichtigste Stelle ist die erste** — ein Ingestion-Fehler pflanzt sich durch alles fort.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg3YV6UvtFnXYnleKDf1S0" -->
Was ist **Schema Drift** und warum ist es besonders gefährlich?::
Das Quellsystem benennt eine Spalte um, ändert einen Datentyp oder fügt eine Spalte ein.<br>**Gefährlich, weil nichts abstürzt.** Die Pipeline läuft grün durch — aber eine Spalte ist plötzlich leer, oder eine Zahl wird als Text interpretiert und sortiert falsch (`"10" < "9"`).<br>Der häufigste **stille Killer** in Datenpipelines. Deshalb gehört ein Schema-Check an die **Ingestion**-Stelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOzblTlCi6PP6z4Lshsdha" -->
Die Dimensionen von Datenqualität — und was unterscheidet **Validity** von **Accuracy**?::
**Completeness** (fehlt etwas? NULL-Anteil) · **Uniqueness** (Duplikate? `COUNT(*)` vs. `COUNT(DISTINCT)`) · **Validity** (entspricht der Wert den **Regeln**? Format, Bereich, Typ) · **Consistency** (passen Werte **zueinander**? Summenabgleich, `ende >= start`) · **Timeliness/Freshness** (aktuell genug?) · **Accuracy** (entspricht der Wert der **Realität**?)<br>**Der entscheidende Unterschied:** „Hauptstraße 5, 10115 Berlin" ist **valide** — Format stimmt, PLZ existiert, Stadt passt. Ob der Kunde dort **tatsächlich wohnt**, ist **Accuracy**.<br>**Validity ist automatisiert prüfbar, Accuracy nicht** — man bräuchte eine unabhängige Wahrheitsquelle, und hätte man die, bräuchte man die Daten nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEWE2zXkqrVBYN1bvWtr7A" -->
Was ist **Write-Audit-Publish**?::
**Erst schreiben, dann prüfen, dann erst veröffentlichen.**<br>1. **Write** — in eine Staging-Zone schreiben, für Konsumenten unsichtbar<br>2. **Audit** — dort die Data-Quality-Checks laufen lassen<br>3. **Publish** — nur bei bestandener Prüfung atomar auf die Zieltabelle umschalten<br>Fällt die Prüfung durch, sehen die Konsumenten die schlechten Daten **nie**.<br>Verbindet den **Circuit Breaker** mit dem **atomaren Schreiben** aus Stichwort 5 — es ist dasselbe Staging-plus-Swap-Muster, nur mit einem Prüfschritt dazwischen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsY8lLQQR645H2fkDkfV4bX" -->
**Rückbezug:** Warum wären Data-Quality-Checks ohne **Idempotenz** praktisch nutzlos?::
Weil man auf einen fehlgeschlagenen Check mit einem **Rerun** reagieren muss. Nur wenn der Job idempotent ist, entstehen dabei **keine Duplikate und kein Mischzustand** — der zweite Lauf überschreibt die Partition mit dem korrigierten Ergebnis.<br>Ohne Idempotenz wäre jeder Rerun nach einem Fehlschlag ein Risiko, und man würde zögern, ihn auszulösen. Ein Check, auf den man nicht reagieren kann, ist ein Alarm ohne Feuerlöscher.<br>**Umgekehrt gilt dasselbe für Tests:** Ein Check mit `CURRENT_DATE` liefert beim Backfill falsche Ergebnisse. Die Determinismus-Fallen gelten für Prüfcode genauso wie für Transformationscode.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDeCLIdrvOnUaIGMYHepOV" -->
Vergleiche die Rolle von **Unittests** in ETL vs. klassischer Softwareentwicklung.::
**Stellenwert:** dort **das** primäre Sicherheitsnetz — hier **ein** Netz von zweien, meist das kleinere.<br>**Was schiefgeht:** dort ist der Code falsch — hier sind meist **die Daten anders als angenommen**.<br>**Coverage:** dort eine sinnvolle Kennzahl — hier aussagearm: 100 % getesteter Code kann falsche Zahlen liefern.<br>**Auslöser:** dort Codeänderung — hier **neue Daten**, was außerhalb der Reichweite von Unittests liegt.<br>**Testdaten:** dort repräsentieren Fixtures die Realität ausreichend — hier **nie** die Vielfalt echter Produktionsdaten.<br>**Zusatzaufgabe in ETL:** die Zusicherungen ersetzen, die im OLTP-System **die Datenbank** gab (Foreign Keys, Constraints) — im Warehouse gibt es sie oft nicht.
<!-- basicblock-end -->

---

# Komplexe Datentypen

*Priorisierung: Der Selbsttest zeigte solide Konzepte, aber Syntaxfehler. Die **Schreibkarten** stehen deshalb vorn.*

<!-- basicblock-start oid="ObsQJ1zK5G21jl5o57lA0XYj" -->
Schreib **zwei** Queries, die alle Zeilen finden, deren `tags`-Array `'urgent'` enthält — und sag, welche indexfähig ist.::
```sql<br>-- 1. Containment — nutzt einen GIN-Index ✅<br>SELECT * FROM tickets<br>WHERE tags @> ARRAY['urgent'];<br><br>-- 2. ANY — nutzt den GIN-Index NICHT ❌<br>SELECT * FROM tickets<br>WHERE 'urgent' = ANY(tags);<br><br>CREATE INDEX ON tickets USING GIN (tags);<br>```<br>**Drei Syntaxfallen:** `FROM` kommt **vor** `WHERE` · Array-Literal ist `ARRAY['urgent']` oder `'{urgent}'`, **nicht** `['urgent']` (das ist Python/JSON) · bei `ANY` steht der **gesuchte Wert links**: `'urgent' = ANY(tags)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsmzi5n4hclPYs5H8zR40Uh" -->
Deklariere einen **Composite Type**, benutze ihn als Spaltentyp, füge eine Zeile ein und lies ein Feld aus.::
```sql<br>CREATE TYPE adresse AS (<br>    strasse text,<br>    plz     text,<br>    stadt   text<br>);<br><br>CREATE TABLE kunden (<br>    id       serial PRIMARY KEY,<br>    name     text,<br>    rechnung adresse<br>);<br><br>INSERT INTO kunden (name, rechnung)<br>VALUES ('Meier',<br>        ROW('Hauptstr. 5','10115','Berlin'));<br><br>SELECT (rechnung).stadt FROM kunden;  -- Klammern!<br>```
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAIBkpSjq51mEN2F61bimo" -->
Warum braucht der Feldzugriff auf einen Composite Type **Klammern** — `(spalte).feld` statt `spalte.feld`?::
Wegen **Mehrdeutigkeit**: In SQL bedeutet `x.y` normalerweise *„Spalte y der Tabelle x"*. Ohne Klammern würde der Parser `rechnung` als **Tabellennamen** lesen und dort eine Spalte `stadt` suchen.<br>Die Klammern sagen: **werte `rechnung` erst als Ausdruck aus, dann greife auf das Feld zu.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6PrR739UmYL14MJNUIh5T" -->
Warum nutzt `WHERE data->>'status' = 'paid'` **keinen** GIN-Index — und was sind die zwei Alternativen?::
**Grund:** Ein GIN-Index unterstützt bestimmte **Operatoren** (`@>`, `?`, `?&`, `?|`) und indiziert die `jsonb`-Spalte **als Ganzes**. `data->>'status'` ist ein **Funktionsaufruf**, dessen Ergebnis nirgends im Index steht — links vom `=` steht ein berechneter Ausdruck. (Derselbe Mechanismus wie `WHERE lower(name) = 'meier'`.)<br>**Alternative 1 — Containment:** `WHERE data @> '{"status":"paid"}'` ✅<br>**Alternative 2 — Expression-Index** (besser bei *einem* bekannten heißen Pfad):<br>```sql<br>CREATE INDEX ON bestellungen ((data->>'status'));<br>```<br>**Faustregel:** GIN für flexible/explorative Abfragen · Expression-Index für einen bekannten Pfad · **echte Spalte**, wenn die Anwendung ständig danach filtert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnNk6baUe6lkR3K2B97c6T" -->
Wogegen verstößt ein Array-Feld in der **1. Normalform** — und war die Regel damit falsch?::
1NF verlangt **atomare Werte**: eine Zelle = **ein** Wert. Ein Array hält mehrere.<br>**Nicht falsch — sie optimiert nur für einen anderen Zweck:**<br>**1NF/normalisiert** → optimiert für **Schreibintegrität** und **unbekannte** Abfragemuster (jeder Join bleibt möglich). Welt: OLTP.<br>**Verschachtelt** → optimiert fürs **Lesen** bei **bekanntem** Zugriffsmuster. Welt: OLAP.<br>⚠️ **Das Kriterium ist nicht die Datenmenge**, sondern die Zugriffscharakteristik: *Wie oft schreibe ich, und weiß ich schon, wie ich lesen werde?* Ein riesiges schreibintensives OLTP-System bleibt normalisiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsw4OZdO6OeBN1k4vEdMfk1" -->
Nenne fünf Unterschiede zwischen **`json`** und **`jsonb`**.::
| | `json` | `jsonb` |<br>|---|---|---|<br>| Speicherung | exakte **Textkopie** | zerlegte **Binärform** |<br>| Schreiben | schneller (nur Validierung) | langsamer (parsen + normalisieren) |<br>| Lesen | langsam (jedes Mal neu parsen) | **schnell** |<br>| Platz | kompakter | ~30–50 % größer |<br>| Erhält | Whitespace, Reihenfolge, Duplikate | **nichts davon** |<br>| GIN-Index, `@>`, `?` | ❌ | ✅ |<br>**Faustregel: Nimm `jsonb`.** `json` nur, wenn das Dokument **byte-genau** erhalten bleiben muss.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2RO2ywpJLLHgg5Y3O6uXI" -->
`jsonb` verliert Schlüsselreihenfolge, Whitespace und Duplikate. Warum ist das meist **gut** — und wann ein **Problem**?::
**Gut, weil `jsonb` normalisiert:** `{"a":1,"b":2}` und `{"b":2, "a":1}` werden zum selben Wert. Daraus folgt:<br>· **Vergleiche werden semantisch** statt textuell<br>· **Duplikate werden aufgelöst** statt eine kaputte Struktur zu konservieren<br>· **Indizierung wird überhaupt erst möglich** — man kann nur indizieren, was eine kanonische Form hat<br>**Problem, wenn das Dokument byte-genau erhalten bleiben muss:** kryptografisch **signierte Payloads** (Signatur bricht bei einem geänderten Byte) · **Audit** („was genau hat der Client geschickt?") · rechtlich relevante Rohbelege.<br>→ Dann Original als `text`/`json` speichern und fachlich mit einer `jsonb`-Kopie arbeiten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsG7WGA8wIdnSkrwdZxsjRn" -->
Nenne die vier **Postgres-Indextypen** und ihr Einsatzgebiet.::
**B-Tree** — general purpose, Sortieren und Filtern auf skalaren Werten<br>**GIN** — **Arrays und JSON**<br>**GiST / SP-GiST** — Volltextsuche und räumliche Daten<br>**BRIN** — Zeitbereiche, Blockdaten, Millionen von Zeilen in physischer Sortierung<br>**GIN = Generalized Inverted Index**, ein *invertierter* Index: statt „Zeile → Wert" speichert er **„Wert → Liste der Zeilen"** — wie das Stichwortverzeichnis eines Buchs. Genau deshalb passt er zu Arrays und JSON, wo eine Zelle *viele* Werte enthält.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssQzQ1yyUnWMLAfZwTGu5T" -->
Warum ist Verschachtelung statt Join gerade im **spaltenbasierten** System attraktiv? (4 Gründe — „schnell" reicht nicht)::
1. **Kein Shuffle.** Ein verteilter Join muss beide Tabellen nach dem Join-Schlüssel **umverteilen**, damit passende Zeilen auf derselben Maschine landen. Bei Verschachtelung liegen Eltern und Kinder physisch schon zusammen.<br>2. **Keine Duplizierung der Elternzeile** — anders als bei flacher Denormalisierung, die Kunde und Datum je Position wiederholt.<br>3. **Columnar zahlt weiter aus:** Parquet speichert `positionen.menge` als **eigene Spalte** — man liest ein Unterfeld, ohne den ganzen Struct zu lesen.<br>4. **Parallelisierbar** — wiederholte Felder werden unabhängig verarbeitet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuwBq0qwdDkfqwTnLiLwSF" -->
Was passiert, wenn du **ein einzelnes Element** in einem Array oder `jsonb`-Feld änderst?::
**Der gesamte Wert wird neu geschrieben.** Es gibt keine Teilaktualisierung: `jsonb_set()` liest den kompletten Wert, ändert ihn im Speicher, schreibt ihn vollständig zurück. Ein Boolean in einem 200-KB-Dokument zu ändern kostet 200 KB Schreiblast.<br>**Dazu MVCC:** Ein `UPDATE` schreibt ohnehin eine komplett neue Zeilenversion → Tabellen-Bloat, plus Aktualisierung aller GIN-Indizes.<br>**Im Lake noch drastischer:** Parquet-Dateien sind unveränderlich → die **ganze Datei bzw. Partition** wird neu erzeugt (siehe Partition als atomare Einheit).<br>⚠️ Das *Finden* ist der einfache Teil — dafür gibt es Pfadausdrücke und Indizes. Teuer ist das **Schreiben**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRcmKkO5X47efBCHg4wvl5" -->
Welche Datenbankgarantien gibt man mit `jsonb` auf — und wie holt man sie zurück?::
**Aufgegeben:** Typsicherheit · `NOT NULL` (Schlüssel kann einfach fehlen) · **Existenz der Spalte** · `CHECK` auf Inhalte · Foreign Keys · `UNIQUE` · Selbstdokumentation (`\d` zeigt nichts)<br>⚠️ **Am gefährlichsten:** `data->>'stauts'` (Tippfehler) wirft **keinen Fehler**, sondern liefert NULL — der klassische **stille Fehler**. Bei einer echten Spalte hätte Postgres sofort gemeldet.<br>**Zurückholen:**<br>1. **Data-Quality-Checks** — die Zusicherung wandert von der DB in die **Testsuite** (`not_null`, `accepted_values`)<br>2. `CHECK (data ? 'status')` — erzwingt Schlüsselexistenz<br>3. **Generierte Spalten:** `GENERATED ALWAYS AS ((data->>'status')) STORED`<br>4. `pg_jsonschema`<br>5. **Am ehrlichsten:** Braucht man ein Feld dauerhaft → **echte Spalte** daraus machen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsy5DEVUGUY5fa8lexu2apV" -->
Nenne je **drei Pros und Cons** komplexer Datentypen.::
**Pro:** weniger Joins (im verteilten System: kein Shuffle) · Elternzeile wird **nicht dupliziert** · fachliche Zusammengehörigkeit wird abgebildet · Flexibilität bei variablen Strukturen · **Schema-Evolution ohne Migration** · weniger Tabellen<br>**Contra:** **teure Updates** (kein partielles Schreiben) · **verlorene DB-Garantien** (Tippfehler → stille NULLs) · **systemspezifische Syntax** (`UNNEST` / `explode` / `LATERAL FLATTEN`) · Indizierung subtil · kein Schema zum Nachschlagen · falsche Entscheidung teuer korrigierbar · Analysen **quer über die Kinder** werden teurer<br>**Merksatz: Normalisiert ist der Standard; Verschachtelung braucht eine Begründung** — nämlich *„wird praktisch nie getrennt gelesen, geschrieben wird partitionsweise."*
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0VRV9JmrF3b98rQPVAaM0" -->
Ordne zu: **Composite Type**, **Array**, **Map** in Postgres / Spark / BigQuery. Welchen hat Postgres nicht?::
| Konzept | Postgres | Spark | BigQuery |<br>|---|---|---|---|<br>| Verschachtelter Datensatz | **Composite Type** (`ROW`) | `StructType` | `STRUCT` (= `RECORD`) |<br>| Liste | **Array** (`text[]`) | `ArrayType` | `ARRAY` |<br>| Schlüssel-Wert | **fehlt** → `hstore`/`jsonb` | `MapType` | `JSON`/`STRUCT` |<br>| Aufklappen | `unnest()` | `explode()` | `UNNEST()` |<br>| Einpacken | `array_agg()` | `collect_list()` | `ARRAY_AGG()` |<br>⚠️ **`STRUCT` und `RECORD` sind dasselbe** (BigQuery nutzt beide Namen) — es sind zwei Konzepte, nicht drei.<br>⚠️ **Postgres fehlt die Map.** `hstore` ist flach und alles Text; `jsonb` ist allgemeiner, aber ohne Typgarantie.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspvSBF36JT5v3C5Gv36VAy" -->
Was machen `unnest()` und `array_agg()` — und wie heißen sie anderswo?::
Sie sind **Umkehrungen voneinander**.<br>**`unnest(arr)`** — set-returning function: aus **einer Zeile** mit Array werden **mehrere Zeilen**. Die Brücke zurück ins normalisierte Modell.<br>**`array_agg(spalte)`** — Aggregatfunktion wie `SUM`: aus **mehreren Zeilen** wird **ein Array**. Funktioniert deshalb auch als **Window Function** mit `OVER`.<br>```sql<br>SELECT id, unnest(tags) AS tag FROM tickets;<br>SELECT id, array_agg(tag) FROM ticket_tags GROUP BY id;<br>```<br>**Anderswo:** Spark `explode()` / `collect_list()` · BigQuery `UNNEST()` / `ARRAY_AGG()`.
<!-- basicblock-end -->

---

# Nicht materialisierte Zwischenergebnisse: Views, CTEs, Subqueries

<!-- basicblock-start oid="ObsdOvhqHH2lmtTYm2kJuceZ" -->
Subquery, CTE und View speichern alle **kein** Ergebnis. Wodurch unterscheiden sie sich dann?::
| | **Subquery** | **CTE** | **View** |<br>|---|---|---|---|<br>| Wo | inline an der Verwendungsstelle | am Anfang der Query | **Datenbankobjekt** |<br>| Sichtbar für | nur den umgebenden Ausdruck | die ganze Query | **alle Sessions/Nutzer** |<br>| Lebensdauer | ihre Auswertung | die Parent-Query | **unbegrenzt** bis `DROP` |<br>| Mehrfach nutzbar | nein (kopieren) | ja, per Name | ja |<br>| Rekursion | ❌ | ✅ | ✅ |<br>| Zugriff auf äußere Query | ✅ **korreliert** | ❌ | ❌ |<br>| `INSERT`/`UPDATE`/`DELETE` enthalten | ❌ | ✅ | ❌ |
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskcFvN4cX93yc9zUzkZfls" -->
Was ist ein **Optimization Fence** — und warum war jede CTE vor PostgreSQL 12 einer?::
Eine Grenze, über die hinweg der Planer **nicht optimieren darf**. Er muss den Teil dahinter vollständig und separat ausrechnen und kann insbesondere **keine Bedingungen der äußeren Query nach innen schieben** (*predicate pushdown*).<br>**Bis PG 11 wurde jede CTE materialisiert — immer.** Bei<br>```sql<br>WITH w AS (SELECT * FROM riesige_tabelle)<br>SELECT * FROM w WHERE id = 123;<br>```<br>wurde die **ganze Tabelle** gelesen und erst danach gefiltert. Ein Index auf `id` blieb ungenutzt.<br>Daher die Faustregel *„CTEs sind in Postgres langsam"* — bis 2019 richtig, **heute falsch**. Ältere Quellen geben hier veraltete Ratschläge.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstdXsBRzrYJCrV2OdVYUbm" -->
Unter welchen **drei** Bedingungen faltet PostgreSQL 12+ eine CTE inline?::
1. sie ist **nicht rekursiv**<br>2. sie ist **seiteneffektfrei** (`SELECT` ohne volatile Funktionen)<br>3. sie wird **genau einmal referenziert**<br>⚠️ **Die dritte ist entscheidend:** Wird eine CTE **mehrfach** referenziert, bleibt sie auch heute materialisiert — sonst müsste sie mehrfach berechnet werden. Zwei von drei reichen nicht.<br>Ist sie gefaltet, optimiert der Planer **beide Ebenen gemeinsam**: Bedingungen wandern nach innen, Indizes greifen, die Join-Reihenfolge ist frei.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuAPw3urBcJZke78QuR2O8" -->
Wann erzwingt man `MATERIALIZED`, wann `NOT MATERIALIZED` — und was ist jeweils das **Risiko**?::
**`MATERIALIZED`:** wenn eine teure Berechnung garantiert **nur einmal** laufen soll · bei Funktionen mit Seiteneffekten · wenn man den Zaun absichtlich als Planer-Bremse einsetzt.<br>*Risiko:* Man baut den alten Zaun wieder auf — **kein Predicate Pushdown, keine Indexnutzung**. Millionen Zeilen lesen, um drei zu behalten.<br>**`NOT MATERIALIZED`:** wenn eine mehrfach referenzierte CTE trotzdem gefaltet werden soll, damit Bedingungen nach innen geschoben werden.<br>*Risiko:* Die CTE wird **pro Referenz neu berechnet** — bei teurer Berechnung multipliziert sich der Aufwand.<br>**Faustregel:** Default nehmen. Die Schlüsselwörter sind für Ausnahmefälle, die man mit `EXPLAIN (ANALYZE)` belegt hat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZClABCe9Yeu6nZOxhlBSf" -->
Schreib das Grundgerüst einer **rekursiven CTE** aus dem Kopf.::
```sql<br>WITH RECURSIVE untergebene AS (<br>    -- Anker (non-recursive term)<br>    SELECT id, name, chef_id, 1 AS ebene<br>    FROM mitarbeiter WHERE id = 42<br><br>    UNION ALL<br><br>    -- rekursiver Term: referenziert die CTE selbst<br>    SELECT m.id, m.name, m.chef_id, u.ebene + 1<br>    FROM mitarbeiter m<br>    JOIN untergebene u ON m.chef_id = u.id<br>)<br>SELECT * FROM untergebene;<br>```<br>**Anker → `UNION ALL` → rekursiver Teil.** `RECURSIVE` steht direkt nach `WITH` und gilt für alle CTEs des Blocks. Der rekursive Teil läuft wiederholt auf dem *zuletzt hinzugekommenen* Ergebnis, bis nichts Neues mehr kommt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuIjKEEc9saLxVHrDy4qym" -->
Wann braucht ein Problem **wirklich** Rekursion — und wie verhindert man eine Endlosschleife?::
**Kriterium: Die Tiefe steht erst zur Laufzeit fest.**<br>· **Hierarchien** — Organigramme, Kategoriebäume, Dateisysteme<br>· **Stücklisten** — Bauteil aus Bauteilen aus Bauteilen<br>· **Graphen/Pfade** — Erreichbarkeit, transitive Hülle<br>· **Reihen erzeugen** — Kalendertabellen, lückenlose Zeitreihen<br>⚠️ **Kein Fall für Rekursion:** „eine Liste aufsummieren" — dafür gibt es `SUM()` bzw. `SUM(...) OVER (…)`. Die Tiefe ist dort bekannt.<br>**Endlosschleife verhindern** (Postgres erkennt Zyklen **nicht** von selbst):<br>1. `WHERE u.ebene < 10` im rekursiven Teil<br>2. die `CYCLE`-Klausel (ab PG 14)<br>3. Pfad-Array mitschleppen: `WHERE NOT m.id = ANY(u.pfad)`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsB0rdBHdbGKi6HtFy5cSDY" -->
Eine **View** speichert kein Ergebnis — die Query läuft jedes Mal neu. Was gewinnt man dann? (4 Gründe)::
1. **Abstraktion/Entkopplung** — kapselt die Tabellenstruktur hinter einer **stabilen Schnittstelle**. Schema ändert sich → View anpassen, alle Konsumenten laufen weiter.<br>2. **Sicherheit** — Rechte auf die View statt auf die Basistabelle; Spalten oder Zeilen ausblenden (Gehälter, fremde Abteilungen).<br>3. **Wiederverwendung** — komplexe Join-/Filterlogik **einmal** schreiben und benennen.<br>4. **Immer aktuell** — weil nichts gespeichert wird, sieht man stets den aktuellen Stand.<br>⚠️ **`REFRESH` gehört NICHT dazu** — das gibt es nur bei der **Materialized View**.<br>**Merksatz: View = immer aktuell, jedes Mal teuer. Materialized View = billig zu lesen, potenziell veraltet.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmJDMWcmsKoGv7LJGuhKWC" -->
Was ist eine **korrelierte Subquery** — und warum ist das der einzige echte Vorteil gegenüber CTEs?::
Eine Subquery, die auf Spalten der **äußeren** Query zugreift und deshalb konzeptionell **pro Zeile der äußeren Query neu** ausgewertet wird:<br>```sql<br>SELECT name,<br>  (SELECT COUNT(*) FROM bestellungen b<br>   WHERE b.kunde_id = k.id)<br>FROM kunden k;<br>```<br>**Eine CTE kann das nicht** — sie wird einmal ausgewertet, bevor die äußere Query sie benutzt, und kennt deren Zeilen nicht.<br>Alternativen für korrelierte Logik: `EXISTS` oder ein **`LATERAL`-Join**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfLcvwWvNMnmIJN4UlGcOi" -->
Nenne fünf **Nachteile von Subqueries gegenüber CTEs** (Kontrollfrage des Leitfadens).::
1. **Lesbarkeit** — Subqueries liest man von innen nach außen, CTE-Ketten von oben nach unten. Ab drei Ebenen gravierend.<br>2. **Keine Wiederverwendung** — dasselbe Zwischenergebnis zweimal heißt: Code **kopieren**, doppelt pflegen, doppelt berechnen.<br>3. **Keine Rekursion** — Hierarchien, Stücklisten, Graphen sind unerreichbar.<br>4. **Keine Data-Modifying Statements** — `INSERT`/`UPDATE`/`DELETE … RETURNING` gehen nur in CTEs.<br>5. **Schlechter debuggbar** — eine CTE-Kette kann man schrittweise ausführen, indem man das finale `SELECT` austauscht.<br>**Gegenrichtung:** Nur die Subquery kann **korrelieren**. Das historische Performance-Argument ist seit PG 12 weitgehend entfallen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8DSaTbexu9F5j2Xhroiaz" -->
Welches mächtige ETL-Muster ermöglichen **Data-Modifying Statements in `WITH`**?::
Löschen und Archivieren in **einer** Anweisung — also **atomar**:<br>```sql<br>WITH geloescht AS (<br>    DELETE FROM alte_daten<br>    WHERE datum < '2020-01-01'<br>    RETURNING *<br>)<br>INSERT INTO archiv SELECT * FROM geloescht;<br>```<br>Eine Subquery kann das nicht. Der Bezug zu **Idempotenz** (Stichwort 5): Entweder beides passiert oder keines — kein Zwischenzustand, in dem die Daten gelöscht, aber nicht archiviert sind.
<!-- basicblock-end -->

---

# Materialisierte Zwischenergebnisse, Performance & Modellierung

*Priorisierung: Der Selbsttest zeigte eine klare Lücke bei der **dimensionalen Modellierung** (D13 unbeantwortet, D15 „keine Ahnung"). Diese Karten stehen deshalb vorn.*

<!-- basicblock-start oid="Obs5hfLMPZ4zzrRh99IAhem3" -->
Was unterscheidet einen **Fakt** von einer **Dimension**? Gib je zwei Beispiele.::
**Fakten = Messungen.** Meist **numerisch**, wiederholt erhoben, typischerweise **additiv**. *Beispiele:* Verkaufsbetrag, verkaufte Menge, Klicks, Anrufdauer.<br>Die **Faktentabelle** enthält die Kennzahlen plus Fremdschlüssel auf die Dimensionen — **schmal, aber sehr lang** (Millionen bis Milliarden Zeilen).<br>**Dimensionen = Kontext.** Die beschreibenden Attribute: *wer, was, wann, wo, wie*. *Beispiele:* Kunde (Name, Segment, Region), Produkt (Kategorie, Marke), Zeit (Tag, Monat, Feiertag), Filiale.<br>**Dimensionstabellen** sind **breit, aber kurz**.<br>**Merkhilfe: Fakten sind das, was man summiert. Dimensionen das, wonach man gruppiert.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ZPjzJ2FmURwPG5fEmaTF" -->
Was ist der **Grain**, und warum ist er die wichtigste Entwurfsentscheidung? (4 Gründe)::
Die Antwort auf: **Was genau bedeutet eine einzelne Zeile der Faktentabelle?** (z. B. „eine Position auf einem Kassenbon" vs. „ein Kassenbon" vs. „ein Produkt pro Tag pro Filiale")<br>1. **Er bestimmt, welche Dimensionen möglich sind** — ist die Zeile ein ganzer Bon, kann es keine Produktdimension geben.<br>2. **Er bestimmt, welche Fragen beantwortbar sind** — was darunter liegt, ist **unwiederbringlich verloren**.<br>3. **Er muss einheitlich sein** — Kimball: *„only facts of a uniform grain"*. Gemischte Grains machen jede Summe falsch, ohne dass es auffällt.<br>4. **Er wird früh festgelegt und ist teuer zu ändern.**<br>**Kimballs Vierschritt:** Geschäftsprozess → **Grain** → Dimensionen → Fakten. Der Grain steht bewusst *vor* den Dimensionen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsR1djxjCZMCNx4McC18i5j" -->
**Star-** vs. **Snowflake-Schema** — Aufbau, Unterschied, und wann nimmt man welches?::
**Star:** Faktentabelle in der Mitte, Dimensionstabellen ringsum direkt per FK verbunden → sieht aus wie ein **Stern**. Die Dimensionen sind **denormalisiert** (`dim_produkt` enthält Name, Kategorie *und* Abteilung).<br>**Snowflake:** Die Dimensionen sind zusätzlich **normalisiert** (`dim_produkt → dim_kategorie → dim_abteilung`) → der Stern verzweigt sich zur **Schneeflocke**.<br>| | Star | Snowflake |<br>|---|---|---|<br>| Joins | **wenige** | mehr |<br>| Speicher | höher (Redundanz) | niedriger |<br>| Für Analysten | **einfach** | anspruchsvoller |<br>| BI-Tools | **sehr gut** | teils eingeschränkt |<br>**Star ist der Standard** — Speicher ist billig, Joins sind teuer. **Snowflake**, wenn Dimensionen sehr groß werden, ein Zweig von mehreren Faktentabellen geteilt wird, oder hierarchische Attribute sich häufig ändern.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs87mBsss4R3OaDaT8tiBVG" -->
Was haben eine **Aggregatstabelle**, eine **Materialized View** und eine **Faktentabelle** gemeinsam — und was folgt daraus für Idempotenz?::
**Alle drei sind abgeleitete, vorberechnete Ergebnisse.** Sie enthalten keine Information, die nicht schon in den Quelldaten steckt — sie speichern nur eine **Rechnung**, die man sonst bei jeder Abfrage wiederholen müsste.<br>**Drei Folgen:**<br>1. **Sie sind entbehrlich** — wegwerfen und neu erzeugen, ohne Informationsverlust. Deshalb ist der **persistente Roh-Layer** (ELT) die Voraussetzung für alles.<br>2. **Ihre Erzeugung muss idempotent sein** — `REFRESH MATERIALIZED VIEW` ersetzt den *kompletten* Inhalt: das ist **Partition Overwrite** unter anderem Namen.<br>3. **Sie werden nie von Hand editiert** — ein `UPDATE` auf einer Aggregatstabelle ist immer ein Fehler. Korrektur gehört in die Quelle oder die Logik, danach neu erzeugen.<br>Das ist auch die Antwort, warum **Recomputability** möglich ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPmwVIIhxKHl9ZTYcUNZmy" -->
Die **drei Stufen der Denormalisierung** — was, wann, zu welchem Preis?::
**1. Normalisiert (3NF)** — Redundanz eliminieren, jede Tatsache genau einmal. *Wann:* **OLTP** — viele kleine Schreibvorgänge, Abfragemuster **unbekannt**. *Preis:* jede Auswertung kostet Joins.<br>**2. Flach denormalisiert (Star-Schema)** — Dimensionsattribute redundant in breite Tabellen ziehen. *Wann:* **OLAP** — leselastig, Joins dominieren, Dimensionen sollen **einzeln** analysierbar bleiben. *Preis:* Die Pipeline muss Redundanz konsistent halten.<br>**3. Verschachtelt (Array/Struct)** — Kindzeilen **in** die Elternzeile. *Wann:* Eltern und Kinder werden **nie getrennt** gelesen. *Preis:* teure Updates, DB-Garantien weg, Queranalysen aufwendig.<br>**Zunehmende Festlegung:** Normalisiert hält alle Fragen offen, Star weniger, Verschachtelung legt fest. **Jede Abweichung braucht eine Begründung aus dem Zugriffsmuster.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuJmXfm7bUGgLM7JAfjYVK" -->
Wodurch unterscheidet sich `CREATE MATERIALIZED VIEW` von `CREATE TABLE AS`?::
Beide legen das Ergebnis **physisch auf der Platte** ab. Der Unterschied:<br>> *„…except that it also **remembers the query** used to initialize the view, so that it can be refreshed later upon demand."*<br>Eine mit `CREATE TABLE AS` erzeugte Tabelle ist danach **nur noch Daten** — die erzeugende Query ist vergessen.<br>| | `CREATE TABLE AS` | Matview |<br>|---|---|---|<br>| Query gespeichert | ❌ | ✅ |<br>| `REFRESH` | ❌ | ✅ |<br>| Direkt beschreibbar | ✅ | ❌ |<br>| Temporär möglich | ✅ | ❌ |<br>| `WITH NO DATA` | ❌ | ✅ |
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswuVMxgmtweTIlpoe5rx2y" -->
Was macht `REFRESH MATERIALIZED VIEW`, was ändert `CONCURRENTLY` — und welche **Voraussetzung** hat es?::
**`REFRESH` ersetzt den kompletten Inhalt** — es gibt in Postgres **keine inkrementelle** Aktualisierung. Die hinterlegte Query läuft komplett neu.<br>**`CONCURRENTLY`** aktualisiert, **ohne parallele Lesezugriffe zu sperren** (ohne die Option nimmt `REFRESH` eine exklusive Sperre).<br>⚠️ **Voraussetzung: ein `UNIQUE`-Index auf der Materialized View.** Ohne ihn schlägt der Befehl fehl — Postgres muss alte und neue Zeilen **einander zuordnen** können, um die Differenz einzuspielen.<br>Zusätzlich: `CONCURRENTLY` ist **langsamer** und ressourcenintensiver, und **nicht mit `WITH NO DATA` kombinierbar**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZadSyoIUPBW19ZiUhZ0zc" -->
Was kann eine **Temp Table**, das eine **CTE** nicht kann? (2 Dinge)::
**1. Sie kann indiziert und mit `ANALYZE` versehen werden.** Der wichtigere Punkt: Eine CTE ist für den Planer eine **Blackbox ohne Statistiken** — er *schätzt* ihre Zeilenzahl, oft daneben. Bei einer Temp Table sammelt `ANALYZE` **echte Statistiken**. Bei mehrstufigen Transformationen über große Zwischenmengen oft der Unterschied zwischen Minuten und Stunden.<br>**2. Sie überlebt die Query.** Eine CTE existiert nur innerhalb *einer* Anweisung; eine Temp Table steht der **ganzen Session** zur Verfügung und kann von mehreren aufeinanderfolgenden Queries genutzt werden.<br>Deshalb sind Temp Tables in ETL-Prozeduren so verbreitet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWbj1WcvgmJIQnHnYIhs4s" -->
Temp Tables: Lebensdauer, Sichtbarkeit und die drei `ON COMMIT`-Varianten?::
**Lebensdauer:** bis zum Ende der **Session** (oder der Transaktion).<br>**Sichtbarkeit:** **nur die erzeugende Session** — auch derselbe Nutzer in einer zweiten Verbindung sieht sie nicht. Zwei Sessions können gleichnamige Temp Tables haben, ohne sich zu stören. Sie verdeckt eine gleichnamige permanente Tabelle.<br>**`ON COMMIT PRESERVE ROWS`** (Standard) — Daten bleiben bis Sessionende<br>**`ON COMMIT DELETE ROWS`** — Tabelle bleibt, **Inhalt** wird geleert<br>**`ON COMMIT DROP`** — Tabelle wird **komplett gelöscht**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7lmFha79LSE4rxYcy7acr" -->
**Die wichtigste Performance-Regel**: Warum killt `WHERE lower(name) = 'meier'` den Index — und wie lautet die allgemeine Form?::
Der Index speichert `name`, **nicht** `lower(name)`. Der Planer sieht links vom `=` einen **berechneten Ausdruck** — eine Blackbox — und muss die ganze Tabelle lesen.<br>**Die allgemeine Regel:**<br>> **Die indizierte Spalte muss auf einer Seite des Vergleichs *nackt* stehen.** Alles Rechnen, Umwandeln und Formatieren gehört auf die **Konstantenseite**.<br>**Sie deckt drei Fallen zugleich ab:**<br>· `lower(name) = 'meier'` → Function-Based Index: `CREATE INDEX ON t (lower(name))`<br>· `datum + interval '1 day' > now()` → **`datum > now() - interval '1 day'`**<br>· `varchar_spalte = 123` (impliziter Cast) → Typen angleichen<br>Markus Winand nennt das **obfuscated conditions** — logisch korrekt, trotzdem katastrophal langsam.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxTaaPzjvvQN57pHrG6Agt" -->
Warum ist ein **fehlender Index auf einem Fremdschlüssel** teuer?::
Postgres legt für einen Foreign Key **automatisch nur auf der referenzierten Seite** (dem Primärschlüssel) einen Index an — **auf der Kindtabelle nicht**.<br>Löscht oder aktualisiert man dann Zeilen in der **Elterntabelle**, muss Postgres bei **jeder Zeile** prüfen, ob noch Kindzeilen darauf verweisen. Ohne Index bedeutet das einen **Sequential Scan auf der Kindtabelle — pro gelöschter Zeile**.<br>Ein `DELETE` von 1.000 Elternzeilen wird so zu 1.000 vollständigen Scans der Kindtabelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDblpVD5FSZSUIoudq4x89" -->
Nenne die Performance-Fallen der Kategorie **„zu viele Daten bewegen"**.::
· **`SELECT *`** — im spaltenbasierten System besonders teuer; verhindert außerdem **Index-Only-Scans**<br>· **Spät filtern** — je früher `WHERE` greift, desto weniger Zeilen wandern durch Joins und Sortierungen<br>· **Join-Fanout** — ein Join auf einen **nicht eindeutigen** Schlüssel vervielfacht Zeilen. Klassische Ursache falscher Summen: erst joinen, dann aggregieren statt umgekehrt<br>· **`OFFSET` bei tiefer Paginierung** — `OFFSET 100000` liest 100.000 Zeilen und wirft sie weg → **Keyset-Pagination** (`WHERE id > :letzte_id`)<br>· **`DISTINCT` als Pflaster** — meist Symptom eines versehentlich duplizierenden Joins
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsP2GPJq2Jyz73laNer3QdQ" -->
Worauf achtest du grundsätzlich, wenn du performante Queries schreiben willst? (8 Punkte)::
1. **Messen statt raten** — `EXPLAIN (ANALYZE, BUFFERS)`; die wichtigste Zahl ist die **Abweichung zwischen geschätzter und tatsächlicher Zeilenzahl**<br>2. **Datenmenge früh reduzieren** — filtern und aggregieren vor dem Join<br>3. **Die Spalte nackt lassen**<br>4. **In Mengen denken** — Schleifen sind fast immer der Fehler<br>5. **Statistiken frisch halten** — `ANALYZE` nach großen Ladevorgängen<br>6. **Nur die gebrauchten Spalten lesen**<br>7. **Indizes gezielt setzen** — jeder Index kostet Schreibleistung<br>8. **In Pipelines: die Stufe optimieren, die das meiste Volumen bewegt** — meist die erste
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDFR0zfjqPiHWH8Ds9fwyI" -->
**Der Grundtausch**: nicht materialisiert vs. materialisiert — in einem Satz?::
> **Nicht materialisiert: immer aktuell, aber jede Abfrage kostet die volle Rechenzeit.**<br>> **Materialisiert: billig zu lesen, aber der Inhalt ist ein Schnappschuss und kann veralten.**<br>⚠️ Häufige Verwechslung: **„Immer aktuell" ist die Eigenschaft der normalen View** — gerade *weil* sie nichts speichert. Eine **Materialized View ist per Definition veraltet** zwischen zwei Refreshes; genau das ist der Preis fürs billige Lesen.<br>Alles Weitere sind Varianten dieser Abwägung — bis hinauf zur Modellebene, wo eine **Faktentabelle** nichts anderes ist als ein vorberechnetes Ergebnis.
<!-- basicblock-end -->
