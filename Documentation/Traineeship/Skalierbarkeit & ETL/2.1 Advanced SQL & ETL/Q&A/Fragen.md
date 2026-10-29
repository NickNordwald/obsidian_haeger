---
tags:
  - documentation
---

# Offene Fragen — 2.1 Advanced SQL & ETL

Fragen, die beim Lesen oder Umsetzen aufkommen. Einfach notieren, Claudian beantwortet sie gesammelt am Ende jedes Stichworts.

Gehört zu [[2.1 Advanced SQL & ETL - Leitfaden]] · [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste]] · [[2.1 Advanced SQL & ETL - Vertiefung]]

## Stichwort 1 — Daten-Pipelines: ETL, ELT & Batch Processing

### ✅ Batch vs. Streaming?

Beantwortet in [[2.1 Advanced SQL & ETL - Vertiefung#1.2 Drei Systemtypen: Online, Batch, Streaming]]. Kurzfassung: Der Unterschied liegt im **Input** — begrenzt (bounded) vs. unbegrenzt (unbounded) — nicht in der Geschwindigkeit.

### ✅ Mismatch zwischen relationalen Datenbanken und spaltenbasierten Warehouses — wie wird der aufgelöst?

Beantwortet in [[2.1 Advanced SQL & ETL - Vertiefung#1.5 Zeilenbasiert vs. spaltenbasiert]]. Kurzfassung: Es gibt keinen Widerspruch — zwei Systeme für zwei Aufgaben, und **die Pipeline ist das Bindeglied**. Beide bleiben relational; der Unterschied ist die physische Speicherung.

### ✅ ELT wird als klar überlegen beschrieben — was sind die Vorteile von ETL?

Beantwortet in [[2.1 Advanced SQL & ETL - Vertiefung#1.6 Wann ETL weiterhin richtig ist]]. Kurzfassung: schwaches Zielsystem · Logik, die nicht in SQL gehört · Compliance (Rohdaten dürfen nicht landen) · Kostenkontrolle bei Pay-per-Query.

### ✅ Die Unix-Pipeline aus dem Kleppmann-Kapitel

```bash
cat /var/log/nginx/access.log |
  awk '{print $7}' |
  sort |
  uniq -c |
  sort -r -n |
  head -n 5
```

Schritt für Schritt erklärt in [[2.1 Advanced SQL & ETL - Vertiefung#1.1 Was ist eine Pipeline? Die Unix-Wurzel]].

### ✅ SaaS — und die Kritik an der Preisabhängigkeit

**SaaS (Software as a Service)** heißt: Man betreibt die Software nicht selbst, sondern mietet sie als laufenden Dienst über das Netz. Der Anbieter kümmert sich um Server, Updates, Backups, Skalierung; man zahlt pro Monat, pro Nutzer oder pro Verbrauch. Im Data-Engineering-Alltag sind Snowflake, BigQuery, Fivetran, dbt Cloud und Airbyte Cloud typische Beispiele — ebenso die Quellsysteme, aus denen man extrahiert (Salesforce, HubSpot, Stripe, Zendesk).

Die Abgrenzung zu den Nachbarbegriffen:

| Modell | Du verantwortest | Beispiel |
|---|---|---|
| **On-Premise** | alles — Hardware, OS, Laufzeit, Anwendung | eigener Postgres im Serverraum |
| **IaaS** (Infrastructure as a Service) | OS, Laufzeit, Anwendung | EC2-Instanz mit selbst installiertem Postgres |
| **PaaS** (Platform as a Service) | nur die Anwendung | Azure App Service, Heroku |
| **SaaS** | nur Konfiguration und Daten | Snowflake, Salesforce |

**Deine Vermutung stimmt** — Preiserhöhungen sind ein realer Kritikpunkt, aber sie sind das *Symptom*. Die eigentliche Ursache ist **Vendor Lock-in**: Je tiefer man integriert ist, desto teurer wird der Ausstieg, und genau dieses Wissen erlaubt dem Anbieter, die Preise anzuziehen. Wer 400 dbt-Modelle gegen Snowflake-spezifisches SQL geschrieben hat, wechselt nicht mal eben zu BigQuery.

Die weiteren Kritikpunkte, die man kennen sollte:

- **Intransparente Kostenmodelle.** Nutzungsbasierte Abrechnung (pro gescanntem Terabyte, pro Compute-Sekunde) macht die Rechnung schwer vorhersagbar. Eine falsch geschriebene Query kann vierstellige Beträge kosten — der direkte Grund, warum die Performance-Fallen aus Stichwort 8 nicht nur akademisch sind.
- **Datenhoheit und Compliance.** Die Daten liegen bei einem Dritten, oft außerhalb der EU. DSGVO-relevant und einer der Gründe, warum manche Branchen bei ETL und On-Premise bleiben (siehe [[2.1 Advanced SQL & ETL - Vertiefung#1.6 Wann ETL weiterhin richtig ist]]).
- **Kein Einfluss auf den Fahrplan.** Ein Feature, das man braucht, kommt vielleicht nie; ein Feature, auf das man baut, wird abgekündigt.
- **Verfügbarkeit ist fremdbestimmt.** Fällt der Anbieter aus, steht die Pipeline, und man kann nichts tun außer die Statusseite lesen.

Die Gegenseite ist allerdings stark: Ohne SaaS-Warehouses gäbe es die elastische Skalierung nicht, die ELT überhaupt erst praktikabel gemacht hat. Der übliche Kompromiss heißt **offene Formate** — Rohdaten als Parquet/Iceberg im eigenen Objektspeicher halten, damit die *Daten* portabel bleiben, auch wenn die Rechen-Engine austauschbar wird.

### ✅ Hadoop

**Hadoop** ist das Open-Source-Ökosystem, das ab ~2006 „Big Data" praktisch gemacht hat — der direkte Vorläufer dessen, was heute Data Lake heißt. Google hatte 2003/2004 zwei Papers zu GFS und MapReduce veröffentlicht; Hadoop ist die freie Nachbauversion davon.

Drei Kernkomponenten:

- **HDFS** (Hadoop Distributed File System) — verteilt Dateien in Blöcken über viele billige Standardrechner und repliziert jeden Block (typisch dreifach). Dadurch übersteht der Speicher Festplattenausfälle und wächst durch Hinzufügen weiterer Rechner.
- **MapReduce** — das Programmiermodell für Batch-Verarbeitung darauf: eine `map`-Phase, die jeden Datensatz einzeln verarbeitet, und eine `reduce`-Phase, die nach Schlüssel gruppierte Ergebnisse zusammenfasst. Kleppmann beschreibt es in DDIA Kap. 10 explizit als „Unix-Pipeline über tausend Maschinen verteilt" — dieselben Prinzipien aus [[2.1 Advanced SQL & ETL - Vertiefung#1.1 Was ist eine Pipeline? Die Unix-Wurzel]], nur im Cluster.
- **YARN** — die Ressourcenverwaltung, die entscheidet, welcher Job auf welchem Knoten Rechenzeit bekommt.

**Die zentrale Idee:** *Bring the computation to the data.* Statt Terabytes durchs Netzwerk zu einem Rechenknoten zu schaufeln, wird der Job auf die Maschinen geschickt, auf denen die Datenblöcke ohnehin liegen. Bei den Netzwerkgeschwindigkeiten von 2006 war das der entscheidende Trick.

**Warum es heute kaum noch neu gebaut wird:**

- **Spark hat MapReduce verdrängt.** MapReduce schreibt nach jeder Phase Zwischenergebnisse auf die Platte. Spark hält sie im Arbeitsspeicher und ist bei iterativen Aufgaben um Größenordnungen schneller. Densmore erwähnt genau das — Spark hat Hadoop überholt.
- **Storage und Compute wurden entkoppelt.** HDFS' Grundannahme war, dass Rechenleistung und Speicher auf denselben Maschinen liegen. Genau das gibt man heute auf: Daten in S3, Rechenleistung nach Bedarf hochfahren und wieder abschalten. Billiger und flexibler — und wieder derselbe Mechanismus, der ELT ermöglicht hat.
- **Der Betriebsaufwand war enorm.** Ein Hadoop-Cluster zu betreiben war ein Vollzeitjob für ein ganzes Team.

**Warum du es trotzdem kennen solltest:** Das Vokabular ist geblieben. Parquet, ORC und Avro sind Formate aus der Hadoop-Welt und heute Standard in jedem Data Lake. Hive gab dem Ganzen SQL und prägte den Begriff „Schema-on-Read". Und in Bestandssystemen läuft Hadoop weiterhin — in der Praxis begegnet es einem als Altlast, nicht als Neuentwurf.


t is based on the map (also known as collect) and reduce
(also known as fold or inject) functions that exist in many functional
programming languages ??

---

## Stichwort 2 — SQL (fortgeschritten)

*(Keine offenen Fragen notiert.)*

## Block A — Deklarativ, funktional, Idempotenz

*(Keine offenen Fragen notiert. Die im Selbsttest sichtbar gewordenen Lücken sind direkt in der Vertiefung geschlossen — siehe [[2.1 Advanced SQL & ETL - Vertiefung#Block A — Kontrollfragen, Review und Self-Check|Review-Tabelle]].)*

## Block B — Testing in ETL

-

Graphen in Postgres simulieren