---
tags:
  - documentation
type: kapitelabschluss
date: 2026-08-19
---

# Kapitelabschluss — Q&A-Ernte und blinde Flecken

Rückblick über alle sieben Unterkapitel von *Grundlagen der Software Entwicklung*. Zweck: **das einsammeln, was nicht in den Vertiefungen steht** — also den Inhalt der `Q&A/`-Ordner — und darüber hinaus benennen, was in diesem Kapitel *gar nirgends* vorkommt.

**Annahme dieser Notiz:** Die `Vertiefung`-Notizen sitzen. Die `Q&A/`-Ordner sind ungelesen. Jeder Punkt hier wurde gegen die Vertiefungen gegengeprüft — was dort schon steht, ist als solches markiert und nicht als „neu" verkauft.

## Inhalt

- [[#Teil 0 — Was in den Q&A-Ordnern überhaupt liegt]]
- [[#Teil 1 — Die Ernte pro Kapitel]]
- [[#Teil 2 — Die Rückstandsliste aus dem OOP-Selbsttest]]
- [[#Teil 3 — Echte blinde Flecken]]
- [[#Teil 4 — Priorisierung]]
- [[#Teil 5 — Vorschlag zur Struktur]]

---

# Teil 0 — Was in den Q&A-Ordnern überhaupt liegt

Erste Erkenntnis: Die `Q&A/`-Ordner sind **nicht eine Gattung, sondern vier**. Sie brauchen unterschiedliche Leseweisen — und zwei Kapitel haben gar keinen.

| Kapitel | Q&A | Gattung | Wie zu lesen |
|---|---|---|---|
| 1.1 OOP | ✅ 3 Dateien | **Nachgeholte Antworten** + **Selbsttest** | Antworten: Nachschlagen. Selbsttest: konkrete Rückstandsliste → [[#Teil 2 — Die Rückstandsliste aus dem OOP-Selbsttest\|Teil 2]] |
| 1.2 DB & ORM | ✅ 3 + 1 Dateien | **Zwei Register** (Nicks + Claudians) + Antworten | Nachschlage-Sheet. Dichteste SQL-Substanz des Kapitels — aber **unpriorisiert**, siehe Warnung unten |
| 1.3 Testing | ❌ **leer** | — | **Lücke** |
| 1.4 Logging | ❌ **leer** | — | **Lücke** |
| 1.5 REST APIs | ✅ 1 Datei (301 Z.) | **Landkarte des Ausgelassenen**, 🔴/🟡/⚪ priorisiert | Nicht durchlesen — als Index benutzen |
| 1.6 Migrations | ✅ 1 Datei (133 Z.) | **Landkarte des Ausgelassenen**, dito | dito |
| 1.7 Docker | ✅ 1 Datei (466 Z.) | **Fehler-/Troubleshooting-Katalog** | Nachschlagen, wenn etwas bricht — nicht auswendig lernen |

> [!warning] Zwei strukturelle Befunde
> **1.** **1.3 Testing** und **1.4 Logging** haben leere `Q&A/`-Ordner. Für diese beiden Kapitel existiert keine „was wurde bewusst ausgelassen"-Landkarte — obwohl beide reichlich Ausgelassenes haben. Das ist keine Wissenslücke, sondern eine *Sichtbarkeitslücke*: Du weißt nicht, was du nicht weißt.
>
> **2.** [[Stichwörter Claudian|Stichwörter Claudian]] (1.2) ist mit 141 Zeilen das dichteste Register des Kapitels — nutzt aber **nur 🧩 (Alltagssyntax) und 🚀 (Bonus)**, nicht das 🔴/🟡/⚪-System aus 1.5/1.6. Beide Symbole lesen sich als „optional". Genau deshalb wirkt dieses Register harmloser, als es ist: Darin steckt Handwerkszeug, das ein Data Engineer **täglich** braucht. Teil 1 unten priorisiert es nach.

> [!info] Der Wegweiser, an dem du vorbeigelaufen bist
> In der [[1.2 Relationale Datenbanken & ORM - Vertiefung|1.2-Vertiefung]] steht am Ende des SQL-Abschnitts ein Callout:
> > 📌 *Bewusst noch offen gelassen (`COALESCE`, UPSERT/`ON CONFLICT`, `RETURNING`, `OFFSET`, `CASE`, Subqueries, CTEs, Window Functions, `EXPLAIN`) sind in Claudians Lücken-Register gesammelt.*
>
> Die Q&A-Ordner waren also nicht versteckt — sie waren im Fließtext verlinkt. Für die Zukunft heißt das: **📌-Callouts in den Vertiefungen sind Auslagerungen, keine Randnotizen.**

Dritter Punkt: In 1.5 und 1.6 steht am Ende jedes Abschnitts **„Fragen zum eigenen Projekt, die offen geblieben sind"**. Das sind keine Lernfragen, sondern **echte offene Design-Entscheidungen** deiner Pricing-Anwendung. Die will man nicht lesen, sondern entscheiden.

---

# Teil 1 — Die Ernte pro Kapitel

Pro Kapitel: die Punkte aus dem `Q&A/`-Ordner. Gegen die Vertiefungen geprüft — Punkte, die dort schon behandelt sind, stehen separat als *Schärfung*.

## 1.1 OOP — [[Antworten|Antworten]]

Die Vertiefung deckt die Prinzipien ab. Die Q&A-Antworten decken die **Python-Mechanik darunter** ab.

**Wirklich neu (steht nicht in der Vertiefung):**

- [ ] **Generatoren, Iteratoren, `yield`** ⭐ Iterable (`__iter__`) vs. Iterator (`__next__`) vs. Generator. `return` = fertiges Ergebnis, `yield` = pausieren und beim nächsten `next()` weiterlaufen. Für dich der **zentrale Speicherhebel**: `csv.DictReader` *ist* ein Iterator und liest faul — deine List-Comprehension zieht ihn bewusst komplett in den RAM.
- [ ] **`@dataclass(slots=True)`** — ersetzt das Pro-Instanz-`__dict__` durch eine feste Slot-Liste: weniger Speicher, schnellerer Zugriff, Preis = keine dynamischen Attribute. Direkt relevant bei deinen 1,33 Mio. `PriceableUnit`s.
- [ ] **`__new__` vs. `__init__`** — Objekterzeugung ist zweistufig: `__new__` *erzeugt*, `__init__` *befüllt*. Braucht man nur für unveränderliche Typen (von `int`/`str`/`tuple` erben), Singletons, Objekt-Caching.
- [ ] **Monkey Patching, Metaklassen, `__dict__`-Jonglieren** — Pythons dynamische Schlupflöcher. In Tests legitim (`unittest.mock.patch` *ist* kontrolliertes Monkey Patching), in Produktivcode ein Warnsignal.
- [ ] **Factory Patterns** — Simple Factory / Factory Method / Abstract Factory, und warum man in Python selten eine Factory-*Klasse* braucht: eine Funktion, ein Dispatch-Dict oder ein `@classmethod` als alternativer Konstruktor genügt meist.
- [ ] **Lambda + PEP 8** — eine Lambda niemals an einen Namen binden (`f = lambda x: …`); braucht sie einen Namen, nimm `def`. Lambdas gehören *inline* als Argument (`key=`).
- [ ] **`csv.reader` statt `csv.DictReader`** — `DictReader` baut pro Zeile ein neues Dict. Der Kompromiss: Header einmal in ein `idx`-Mapping ziehen, dann `row[idx["spalte"]]` — Lesbarkeit ohne Dict-pro-Zeile.
- [ ] **ETL vs. ELT** — der Begriffs-Anker für dein Berufsfeld. ELT (erst laden, dann im Warehouse transformieren, z. B. per dbt) ist durch billige Cloud-Warehouses heute der Normalfall.
- [ ] **`functools.total_ordering` und `cached_property`** — Boilerplate-Sparer.

**Schärfung (steht schon in der Vertiefung, die Q&A stellt es nur knapper gegenüber):**

- [ ] **Protocol vs. ABC — die Vergleichstabelle** ⭐ Der Punkt, den du im Selbsttest nur halb hattest (F13/F14). ABC = *expliziter Unterschriftsvertrag*: Vererbung nötig, zur **Laufzeit** bei der Instanziierung erzwungen, kann geteilten Code mitbringen. Protocol = *stiller Verhaltensvertrag*: keine Vererbung, **statisch** von mypy geprüft, kein geteilter Code, der Implementierer muss das Protocol nicht einmal kennen. Die Tabelle in der Q&A ist die kompakteste Fassung davon im ganzen Vault.
- ~~Name Mangling~~ — steht bereits ausführlich in der Vertiefung (inkl. der Klarstellung „kein Sicherheitsmechanismus").
- ~~Mixins~~ — ebenfalls, inkl. Diamond Problem und C3-Linearisierung.

## 1.2 Relationale Datenbanken & ORM — [[Antworten (Stichwörter & Fragen)|Antworten (Stichwörter & Fragen)]] · [[Stichwörter Claudian|Register]]

**Der inhaltlich wertvollste Q&A-Ordner des Kapitels.** Hier steht das SQL-Handwerkszeug, das die Vertiefung ausdrücklich ausgelagert hat (siehe 📌-Callout in Teil 0). Ein großer Teil davon ist für einen Data Engineer **Alltagswerkzeug, nicht Kür** — das Register selbst markiert es leider nur mit „🚀 optional".

### Tier 1 — tägliches Handwerkszeug

- [ ] **Window Functions** ⭐⭐ `func() OVER (PARTITION BY … ORDER BY …)` — aggregieren, **ohne** Zeilen zu Gruppen zu kollabieren. `ROW_NUMBER`/`RANK`, `LAG`/`LEAD` (Differenz zum Vortag!), laufende Summen. Für Analytics/Data Engineering das wichtigste einzelne SQL-Feature. In der Vertiefung nur namentlich erwähnt, nie gezeigt.
- [ ] **CTEs / `WITH`** — benannte Zwischenergebnisse für mehrstufige, lesbare Abfragen; `WITH RECURSIVE` für Hierarchien und Bäume.
- [ ] **`EXPLAIN ANALYZE` wirklich lesen** ⭐ Die Vertiefung zeigt, *dass* man `EXPLAIN` benutzt. Die Q&A zeigt, *was* dort steht: Scan-Typen (Seq / Index / Index-Only / Bitmap Heap) und vor allem `rows`-**Schätzung** vs. `actual rows`. Weichen sie weit ab → veraltete Statistiken → `ANALYZE`.
- [ ] **UPSERT: `INSERT … ON CONFLICT … DO UPDATE`** — macht deinen CSV-Ingest **idempotent** und ist die saubere Alternative zum `TRUNCATE`-Voll-Reload. `EXCLUDED` = die vorgeschlagene Zeile.
- [ ] **`… RETURNING`** — die geschriebenen Zeilen direkt zurückbekommen, ohne zweites `SELECT`. Erklärt nebenbei, warum du im ORM nach `add`+`flush` sofort die `id` hast.
- [ ] **`NOT IN` mit NULL — die Falle** ⭐ `NOT IN (subquery)` liefert **gar nichts**, sobald die Subquery ein einziges NULL enthält (dreiwertige Logik). `NOT EXISTS` ist robust. Klassischer stiller Bug.
- [ ] **Keyset- statt Offset-Pagination** — `LIMIT 20 OFFSET 40000` ist langsam, weil die DB die übersprungenen Zeilen trotzdem berechnet und wegwirft. `WHERE id > :last ORDER BY id LIMIT 20` skaliert. Taucht in 1.5 auf API-Ebene **erneut** auf.
- [ ] **`CASE WHEN`**, **`COALESCE`/`NULLIF`**, **`DATE_TRUNC`/`EXTRACT`/`AGE`**, **Cast `wert::TYP`** — die Kleinteile, die man täglich braucht.

### Tier 2 — Interna, die Verhalten erklären

- [ ] **MVCC, WAL, Bloat, VACUUM** ⭐ Ein `UPDATE` schreibt eine **neue Zeilenversion**, die alte bleibt als totes Tupel liegen. Deshalb blockieren Leser keine Schreiber — und deshalb wächst eine Tabelle bei Massen-Updates auf der Platte. WAL (erst ins Log, dann in die Datenseiten) ist die Grundlage von Atomicity, Durability, PITR und Replikation. **Erklärt gleichzeitig die Migrationskosten aus 1.6.**
- [ ] **Optimistisches vs. pessimistisches Locking** ⭐ Pessimistisch = `SELECT … FOR UPDATE`, andere warten. Optimistisch = Versionsspalte + `UPDATE … WHERE version = :gelesen`, bei Fehlschlag Retry. **Dasselbe Muster taucht in 1.5 als ETag/`If-Match`/`412` auf HTTP-Ebene wieder auf** — ein Konzept, zwei Schichten.
- [ ] **Deadlocks** — über Kreuz gesperrt; Postgres erkennt den Zyklus und bricht eine Transaktion ab → **Retry**. Vermeiden durch konsistente Sperrreihenfolge.
- [ ] **Lange Transaktionen kosten** — halten einen alten Snapshot → VACUUM kann nicht aufräumen → Bloat. Regel: nie eine Nutzer-Wartezeit in einer offenen Transaktion.
- [ ] **B+-Tree im Detail** — innere Knoten tragen nur Routing-Schlüssel (hoher fan-out → Millionen Zeilen in 3–4 Ebenen), Blätter sind **untereinander verkettet** (deshalb sind Bereichs-Scans und `ORDER BY` billig). Genau deswegen nehmen DBs B+ statt B.
- [ ] **Index-Typen** — B-Tree (Default), Hash (nur `=`), **GIN** (Arrays, `jsonb`, Volltext), **GiST** (Bereiche, Geometrie), **BRIN** (riesige, physisch sortierte Tabellen wie Zeitreihen — winzig).
- [ ] **Partielle & Ausdrucks-Indizes**, **Covering Index / `INCLUDE`** (→ Index-Only Scan).
- [ ] **Clustered vs. Heap** — Postgres speichert Zeilen ungeordnet im Heap, der PK ist ein *separater* Index. MySQL/InnoDB und SQL Server sind physisch nach PK sortiert.
- [ ] **Join-Algorithmen** — Nested Loop / Hash Join / Merge Join und wann der Planner welchen wählt. Macht `EXPLAIN`-Output erst lesbar.

### Tier 3 — Modellierung, Constraints, ORM

- [ ] **Star- vs. Snowflake-Schema** ⭐ Bewusst denormalisierte Data-Warehouse-Modelle (Faktentabelle + Dimensionen). Der Gegenpol zur OLTP-Normalisierung — und genau die Modellierungswelt, in die dein Berufsfeld führt.
- [ ] **`EXCLUDE`-Constraints** — Postgres-Spezialität, mächtiger als `UNIQUE`: „keine überlappenden Buchungszeiträume" (`EXCLUDE USING gist (raum WITH =, zeitraum WITH &&)`). Für eine Buchungsdomäne wie deine geradezu maßgeschneidert.
- [ ] **`NOT VALID` + `VALIDATE CONSTRAINT`** — Constraint auf einer großen Produktivtabelle nachrüsten, ohne lange zu sperren. Verbindet direkt zu 1.6.
- [ ] **`DEFERRABLE`** (Prüfung ans Transaktionsende), **`MATCH FULL`/`SIMPLE`**, **Domains** (`CREATE DOMAIN positive_price …`).
- [ ] **FK-Performance** — fehlt der Index auf der FK-Spalte des **Kindes**, scannt Postgres bei jedem Elternteil-Löschen die ganze Kindtabelle.
- [ ] **`LATERAL` Join** — die rechte Unterabfrage darf auf die linke Zeile zugreifen: das Standardrezept für **Top-N pro Gruppe**.
- [ ] **`SERIAL`/`IDENTITY` vs. `UUID`** — und warum **UUIDv4** (zufällig) Index-Performance kostet (Page-Splits über den ganzen Baum), **UUIDv7** (zeitsortiert) aber nicht.
- [ ] **ORM-Vererbungs-Mapping** — Single Table / Joined Table / Concrete Table. Die Brücke von der OOP-Vererbung aus 1.1 ins relationale Modell.
- [ ] **Session-Sonderfälle** — `merge()`, `DetachedInstanceError` (Zugriff auf lazy-Attribut nach Session-Ende), `scoped_session`, `expire_on_commit=False`.
- [ ] **Core vs. ORM bewusst mischen** — Bulk-Operationen, analytische Queries mit Window/CTE, Performance-Pfade gehen auf die Core-Ebene oder `text()`. Hybrid ist normal, kein Stilbruch.

### Tier 4 — Einordnungswissen (einmal lesen, dann liegen lassen)

Materialized Views + `REFRESH … CONCURRENTLY` · Updatable Views + `WITH CHECK OPTION` · View vs. CTE vs. Subquery vs. Temp-Tabelle · Advisory Locks · 2PC und das Saga-Pattern · CAP/PACELC · Serializable Snapshot Isolation · synchrone vs. asynchrone Replikation · ER-Notationen (Chen vs. Crow's Foot) · BCNF/4NF/5NF · Async SQLAlchemy · ORM-Alternativen (Django ORM, Peewee, Tortoise, SQLModel) · `INTERSECT`/`EXCEPT` · Semi-/Anti-Join.

Ergänzend: [[Q&A (Kapitelabschluss)|Q&A (Kapitelabschluss)]] — projektspezifische Antworten (warum `COPY` statt `executemany`, `execute` vs. `scalars` vs. `scalar`, `Mapped[X]` vs. `mapped_column(...)`, warum `net_revenue` `GENERATED` ist).

## 1.3 Testing mit PyTest — ⚠️ kein Q&A vorhanden

Nichts zu ernten. Was fehlen dürfte, aus den Querverweisen der Nachbarkapitel rekonstruiert:

- **Integrationstests existieren im Projekt gar nicht** — alle 123 Tests laufen ohne Datenbank (Fakes über `dependency_overrides`). Aus 1.6: **`testcontainers-python`** startet einen echten Postgres-Container pro Testlauf.
- **API-Tests** — FastAPIs `TestClient`, `dependency_overrides` als Testwerkzeug (aus 1.5).
- **Faustregel aus 1.5:** *Wenn ein Controller-Test Fachlogik prüft, ist die Fachlogik am falschen Ort.*
- **Property-based Testing** (Hypothesis), **`schemathesis`** (prüft die laufende API gegen ihr eigenes OpenAPI-Schema, mit generierten Zufallseingaben).
- Mutation Testing wird in der Vertiefung nur als Ausblick genannt.

## 1.4 Logging & Exceptionhandling — ⚠️ kein Q&A vorhanden

Ebenfalls nichts zu ernten. Rekonstruierte Anschlussthemen:

- **Correlation-/Request-IDs** (aus 1.5) — eine ID pro Anfrage, in `X-Request-ID` **und** in jeder Logzeile. Der Punkt, ab dem Logging bei parallelen Anfragen überhaupt nutzbar wird. `contextvars` ist der Python-Mechanismus dafür.
- **Uvicorn-Logging vs. eigene Konfiguration** — die beiden beißen sich; doppelte oder verschluckte Logzeilen sind der Normalfall, bis man es explizit löst.
- **Was nicht ins Log darf** — Bodys mit personenbezogenen Daten, Tokens. `SecretStr` (aus 1.5) löst das *strukturell* statt per Disziplin.
- **`PYTHONUNBUFFERED=1`** (aus 1.7) — ohne das erscheinen stdout-Logs im Container verspätet und bei einem Absturz gar nicht.
- **Metriken** — kommen im ganzen Kapitel nicht vor. (Tracing ist im Leitfaden bewusst out of scope, Metriken sind schlicht vergessen.)

## 1.5 REST APIs & Layered Architecture — [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/Q&A/Weiterführende Stichpunkte.md|Weiterführende Stichpunkte]]

301 Zeilen, bereits mit 🔴/🟡/⚪ priorisiert. **Nicht durchlesen — als Index benutzen.** Die 🔴-Punkte mit dem größten Praxisnutzen:

- [ ] **Paginierung** ⭐ Der wichtigste ungelöste Punkt an *jedem* `GET` auf eine Sammlung. Ohne Grenze liefert `GET /bookings` irgendwann eine Million Zeilen. Offset/Limit vs. Cursor/Keyset — dieselbe Abwägung wie in 1.2, hier auf API-Ebene.
- [ ] **Filtern, Sortieren, Suchen** — Konventionen (`?product_id=X&from=…&sort=-calendar_date`) und die Grenze, ab der eine Query in einen `POST /search`-Body wandert.
- [ ] **Bulk-Operationen** — CRUD ist auf *ein* Element ausgelegt. Dein Ingest-Fall: 10.000 Datensätze. Und die Frage nach dem **Teilerfolg** (`207 Multi-Status`).
- [ ] **`async`/`await`, Event Loop und die GIL** ⭐ Der häufigste Einsteigerfehler: eine blockierende DB-Abfrage in einer `async def`-Route legt den **gesamten** Server lahm. `await` heißt „Kontrolle abgeben", nicht „warten". Steht hier **zweimal** als 🔴 — offensichtlich der am dringlichsten markierte Punkt des ganzen Kapitels.
- [ ] **Conditional Requests vollständig** — `If-None-Match`/`304` (Caching) vs. `If-Match`/`412` (**optimistisches Locking über HTTP**, ganz ohne Serverzustand) vs. `428 Precondition Required`. Und die Frage, woraus man einen ETag überhaupt bildet.
- [ ] **Cookie-Attribute im Detail** — `HttpOnly` (die zentrale XSS-Verteidigung), `SameSite` (entschärft CSRF), `Secure`, die `__Host-`-Präfixe. Plus die ehrliche Abwägung **JWT im localStorage vs. Session-Cookie**: zwei verschiedene Bedrohungsmodelle, keine eindeutig richtige Antwort — und das ungelöste Problem des **Token-Widerrufs** bei zustandslosen JWTs.
- [ ] **Transaktionsgrenzen über Schichten hinweg** — wer öffnet und committet: Controller, Service oder Repository? (Standardantwort: Serviceschicht, „Unit of Work".) Praktisch die wichtigste offene Architekturfrage deines Projekts.
- [ ] **API-Versionierung und Breaking Changes** — was *ist* ein Breaking Change? (Feld entfernen, Pflichtfeld hinzufügen, Typ verengen, Statuscode ändern — ein *optionales* Feld hinzufügen dagegen nicht.) Später sehr teuer nachzuholen.
- [ ] **Contract Testing** — `schemathesis`, Pact. Schließt die Lücke, die Code-First hinterlässt.
- [ ] **Deployment des Stacks** — Uvicorn hinter nginx/Traefik, Worker-Anzahl, Graceful Shutdown, Health-/Readiness-Endpunkte.
- [ ] **Timeouts, Retries, Backoff, Circuit Breaker** — was man *tut*, wenn eine fremde Datenquelle langsam oder tot ist. Stichwort „Retry Storm".
- [ ] **Request-ID-Middleware** — wenig Code, macht das Logging aus 1.4 erst wirklich nutzbar.
- [ ] **Soft Delete vs. Hard Delete** — `404` oder `410 Gone`? Als Data Engineer relevant, weil gelöschte Daten sonst in Auswertungen wieder auftauchen.
- [ ] **CORS/Preflight praktisch** — ein Fehler, der **ausschließlich im Browser** auftritt und in Postman nie.
- [ ] **Netzwerkanalyse** — die Spalte **Initiator** („wo zum Teufel kommt dieser Request her?"), **Size vs. Transferred**, Throttling/Offline simulieren, `curl -w` für Timing ohne Browser, **mitmproxy** für alles, was kein Browser ist.

Für Data Engineering am anschlussfähigsten: **Pydantic ohne FastAPI** — CSV-Zeilen beim Ingest validieren, Verträge zwischen Pipeline-Schritten (`TypeAdapter`), plus der Vergleich mit **Pandera** (DataFrames) und **Great Expectations**.

## 1.6 Datenbank Migrations Tools — [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Q&A/Weiterführende Stichpunkte.md|Weiterführende Stichpunkte]]

Kompakter und dichter als 1.5. Die 🔴-Punkte:

- [ ] **State-based vs. migration-based** ⭐ Die Vertiefung stellt das Deklarative als *Vorstufe* dar. Das ist die verbreitete, aber nicht die ganze Wahrheit: **Atlas**, **migra**, **Skeema** bleiben deklarativ und berechnen den Übergang selbst. Eine laufende, echte Kontroverse — wer beide Lager kennt, argumentiert souveräner als jemand, der „man nimmt halt Alembic" sagt.
- [ ] **Backup & Point-in-Time-Recovery** — `pg_dump` (logisch) vs. `pg_basebackup` (physisch), WAL-Archivierung als PITR-Grundlage. Und die einzige Frage, die zählt: **Wann wurde das Restore zuletzt geübt?** Ein Backup, das nie zurückgespielt wurde, ist eine Vermutung.
- [ ] **Batched Backfill** ⭐ Bei 939.559 Buchungen ist ein einzelnes großes `UPDATE` ein Problem: minutenlange Locks, aufgeblähtes WAL, blockiertes `VACUUM`. Praxislösung: Blöcke à ~10.000 Zeilen, oft als eigenes Skript **außerhalb** der Migration. Ändert die Deployment-Struktur: Migration legt die Spalte an, Backfill läuft nebenher, erst danach die Constraint.
- [ ] **Was man an einer Migration überhaupt testet** ⭐ Vier Dinge: (1) Kette läuft aus leerer DB, (2) kein Modell-Drift (`alembic check`), (3) Round-Trip hoch–runter–hoch, (4) **Datenerhalt** — der einzige Punkt, der Fachlichkeit prüft und den man von Hand schreiben muss. Voraussetzung: **Testcontainers** und damit eine ganz neue Testkategorie im Projekt.
- [ ] **`squawk`** — statischer Linter für Migrationen, kennt die gefährlichen Muster (`SET NOT NULL` ohne Vorbereitung, `CREATE INDEX` ohne `CONCURRENTLY`). Passt zum Muster, das dein Projekt mit `test_architektur` schon verfolgt: eine Regel, die nicht von Disziplin abhängt.
- [ ] **Flyway als dritter Kandidat** — SQL-first, radikal einfach. Macht die Achse „wie viel Abstraktion will ich?" erst vollständig. (Flyway bietet `undo` nur kommerziell an — ausdrücklich begründet damit, dass Undo-Skripte in der Praxis selten funktionieren.)
- [ ] **Least Privilege für DB-Rollen** — ein Migrations-Benutzer mit DDL-Rechten (nur im Deployment), ein Anwendungs-Benutzer mit `SELECT/INSERT/UPDATE/DELETE`. Dein Projekt hat aktuell genau einen Benutzer in der `.env`.
- [ ] **Schema-Evolution außerhalb relationaler Datenbanken** ⭐⭐ **Dein künftiges Berufsfeld:** Avro (Reader-/Writer-Schema, Kompatibilitätsregeln), Confluent **Schema Registry** mit `BACKWARD`/`FORWARD`/`FULL` (die formale Fassung dessen, was hier „N-1-Kompatibilität" heißt), **Delta Lake / Apache Iceberg** (Schema Evolution + Time Travel). Wer die Konzepte aus diesem Kapitel hat, liest diese Doku mit direktem Wiedererkennen.
- [ ] **Wartungsfenster als legitime Antwort** — die ganze Zero-Downtime-Maschinerie ist teuer. „Sonntag 3 Uhr, 20 Minuten Ausfall, dafür ein einziges einfaches `ALTER TABLE`" ist für viele interne Systeme wirtschaftlich richtig. Die Literatur behandelt fast nur den Zero-Downtime-Fall.

> [!important] Hier stehen keine Lernfragen, sondern Entscheidungen
> Der Abschnitt „Fragen zum eigenen Projekt" ist bereits teilweise abgehakt (✅ von der Umsetzung am 15.08.). **Offen und entscheidungsreif** sind u. a.:
> - Was passiert mit `pricing-initdb` und `create_all()`? (streichen / auf Alembic umstellen / beides behalten — Letzteres riskant, weil dann zwei Wege ein Schema erzeugen)
> - Braucht das Projekt jetzt eine echte Testdatenbank? (bringt Integrationstests + Docker-Abhängigkeit ins Projekt)
> - Wie verträgt sich der `TRUNCATE` im Ingest mit `price_recommendations` und `pricing_runs`?
> - Sollten „abgeleitete" (aus CSV reproduzierbare) und „echte" Daten getrennt werden?
> - Gibt es überhaupt eine Backup-Strategie? (aktuell hängt alles an einem Docker-Volume)
>
> Das sind ~30 Minuten Nachdenken mit mehr Wirkung als jedes gelesene Stichwort.

## 1.7 Docker — [[Fragen aus der Umsetzung|Fragen aus der Umsetzung]]

Andere Gattung: **kein Ausblick, sondern ein Fehlerkatalog** aus der eigenen Umsetzung. Nicht auswendig lernen — aber diese Punkte einmal bewusst gelesen zu haben spart später Stunden:

- [ ] **`0.0.0.0` ist nicht localhost** ⭐ `127.0.0.1` ist eine konkrete Adresse (`lo`), `0.0.0.0` ist **keine Adresse**, sondern ein Platzhalter *beim Lauschen*. Im Container **immer** `--host 0.0.0.0` — ein Server auf `127.0.0.1` nimmt Verkehr über `eth0` gar nicht erst an.
- [ ] **Die PID-1-Falle ist zwei Fallen** ⭐ (A) **Signale**: Die Shell-Form von `CMD` macht `/bin/sh` zu PID 1, und die reicht `SIGTERM` nicht weiter → `docker stop` dauert *immer* 10 s und endet hart. Lösung: exec-Form oder `exec "$@"`. (B) **Zombies**: PID 1 muss verwaiste Kinder einsammeln → `init: true`. Verschärfend: Der Kernel stellt Signale mit Standardverhalten an PID 1 gar nicht zu.
- [ ] **Layer-Caching ist eine Kette** — der Schlüssel jeder Anweisung enthält den der vorherigen. Ein `COPY . .` reagiert auf *jede* Dateiänderung und macht alle folgenden Schichten ungültig. **Beweisen statt messen:** auf das Wort `CACHED` in der Build-Ausgabe achten — es muss beim teuren `uv sync` stehen.
- [ ] **`HEALTHCHECK` ist nur ein Signal** — Docker startet einen `unhealthy`-Container **nicht** neu. Auswerten müssen es andere: `depends_on: condition: service_healthy`, `up --wait`, Kubernetes. `--start-period` ist die Karenzzeit, in der Fehlschläge nicht zählen.
- [ ] **`down` muss dieselben Profile kennen wie `up`** — sonst bleibt ein Container stehen, hängt weiter im Projektnetz, und das Netzwerk lässt sich nicht entfernen („Resource is still in use"). Dauerhaft lösbar mit `COMPOSE_PROFILES=dev` in der `.env`.
- [ ] **`restart: "no"` muss gequotet sein** — in YAML 1.1 sind `no`/`yes`/`on`/`off` **Booleans**. Ohne Anführungszeichen liest der Parser `false`. Pflicht für Einmal-Dienste wie `migrate`; für die API ist `unless-stopped` richtig.
- [ ] **Named Volume vs. Bind Mount = ein Zeichen** — beginnt der linke Teil mit `.` oder `/`, ist es ein Bind Mount, sonst ein Named Volume. Ein Tippfehler im Host-Pfad erzeugt **stillschweigend** ein neues, leeres, root-gehörendes Verzeichnis. Die Langform (`type: volume|bind`) macht es explizit.
- [ ] **YAML-Anker (`&name` / `*name` / `<<: *name`) und `x-`-Extension-Fields** — kein Compose-Feature, sondern YAML. Der Parser löst das auf, *bevor* Compose die Datei sieht. Kontrolle mit `docker compose config`.
- [ ] **`PYTHONUNBUFFERED=1`** — `stderr` ist zeilengepuffert, **`stdout` ist blockgepuffert (~8 KB), wenn es kein TTY ist**. Uvicorns Access-Logs gehen nach stdout → ohne die Variable erscheinen sie verspätet, bei einem Absturz gar nicht.
- [ ] **`${VAR:?meldung}`** — `${VAR}` wird bei fehlender Variable zum **leeren String** (still und gefährlich), `${VAR:?…}` bricht ab. Und die Ehrlichkeit dazu: Das hält Werte aus git heraus, macht sie aber **nicht geheim** — sie stehen weiter in `docker inspect`. Echte Geheimhaltung braucht Docker Secrets (`/run/secrets/…`, `_FILE`-Konvention).
- [ ] **`--chown` am `COPY` statt `RUN chown -R`** — `chown` ändert Metadaten *jeder* Datei, und eine geänderte Datei landet **vollständig** in der neuen Schicht. Rekursives `chown` über ein 150-MB-`.venv` verdoppelt dessen Platzbedarf im Image.
- [ ] **`USER 1000:1000` statt `USER app`** — Kubernetes kann mit `runAsNonRoot: true` einen *Namen* nicht auf Nicht-root prüfen und verweigert den Start. Namen sind ohnehin nur ein Nachschlagewerk für Menschen; der Kernel vergleicht Integer.
- [ ] **`/docs` ist ein Client deiner eigenen API** — FastAPI baut ~25 Zeilen HTML mit zwei CDN-Verweisen; die Substanz kommt aus `/openapi.json`. Folge: Das Internet braucht der **Browser**, nicht der Server. Und `/docs` ist standardmäßig öffentlich, auch wenn alle Router Auth verlangen.

> [!tip] Die einzige generalisierbare Lehre des ganzen Kapitels
> Alle vier dokumentierten Fehler waren **stille** Fehler — nichts brach ab, es passierte nur nicht das Gewünschte (`PYTHONUNBUFFERD` mit Tippfehler, `start_perios` mit Tippfehler, `COPY . .` an falscher Stelle, `--no-install-project` in beiden `uv sync`).
>
> **„Bei Docker prüft man das Ergebnis, nicht die Absicht."** — `docker compose config`, das Wort `CACHED`, `docker compose exec api env`, `docker inspect`.
>
> Das gilt weit über Docker hinaus: In jeder Pipeline ist die Frage „hat es wirklich getan, was ich wollte?" eine andere als „ist es ohne Fehler durchgelaufen?". Das ist die Haltung, die man aus diesem Kapitel mitnimmt.

---

# Teil 2 — Die Rückstandsliste aus dem OOP-Selbsttest

[[Q&A – Code-Verständnis (OOP-Lösung)|Q&A – Code-Verständnis (OOP-Lösung)]] ist kein Ausblick, sondern ein **dokumentierter Wissensstand vom 09.07.** — 31 Fragen zu deinem eigenen Code, jeweils mit deiner Original-Antwort, Musterlösung und Verdikt. Die einzige Stelle im ganzen Kapitel mit einer *messbaren* Lückenliste.

**Bilanz:** 11× ✅ · 11× 🟡 (Kern richtig, unvollständig) · 9× ⬜ (offen geblieben)

Die neun offen gebliebenen Fragen — jede hat dort eine Musterlösung, du hast sie nur nie gelesen:

- [ ] **F5** — Wer loggt die negative Kapazität, und warum die Trennung Prädikat/Reaktion? *(Domain bleibt I/O-frei; der Builder loggt aggregiert.)*
- [ ] **F11** — Nachteil von `max(recs, key=lambda …)` gegenüber `order=True`?
- [ ] **F12** — Ist die Kopplung von `get_max` an das dunder-Verhalten der Domain-Klasse ok?
- [ ] **F21** — Was hat das 0-Default des `Counter` mit einem `LEFT JOIN` zu tun? *(Sehr hübsche Brücke von 1.1 zu 1.2.)*
- [ ] **F23** — Was ist ein Orphan-Booking, warum still verwerfen, warum die Summe loggen?
- [ ] **F27** — Warum Formel in `float`, Ergebnis via `Decimal(f"{pct:.6f}")`?
- [ ] **F28** — Warum Pfad im Konstruktor und `write(recs)` ohne `path`-Argument? *(Leaky abstraction.)*
- [ ] **F29** — Warum `_FIELD_NAMES` aus `fields(PriceRecommendation)` ableiten? *(Drift-Bug verhindern.)*
- [ ] **F31** — (a) Warum `logging.basicConfig` in `main.py`? (b) Warum `python -m pricing.main`?

Dazu zwei **nie beantwortete Bonusfragen**:

- [ ] **B1** — Was unterscheidet Kapselung von Abstraktion? (je ein Beispiel aus dem Projekt)
- [ ] **B2** — Loose Coupling vs. strong cohesion, belegt an einer selbst gewählten Klasse

Und die vier Themen, die das Dokument selbst als Vertiefungsempfehlung markiert — weil sie in mehreren Fragen gleichzeitig wackelten:

1. **Open/Closed vs. Loose Coupling vs. Dependency Injection** (werden gern verwechselt; in F24 hast du „Loose Coupling" gesagt, wo „Open/Closed" das Prinzip war)
2. **`Protocol` / structural typing** und vor allem **statisch (mypy) vs. Laufzeit** (F13, F14 — in F14 lag die Antwort inhaltlich daneben)
3. Die Kette **`occupancy` → Engine-Filter `total_capacity == 0` → tolerante Domain** (F4, F26)
4. **DTO-Konzept** und der **`fields()`-Trick** gegen Schema-Drift (F15, F29)

> [!note] Warum das die wertvollste halbe Stunde des ganzen Rückblicks ist
> Alles andere in dieser Notiz ist *Stoff, den du noch nicht kennst*. Dieser Punkt ist **Stoff, bei dem dokumentiert ist, dass du ihn nicht sicher konntest** — an deinem eigenen Code, nicht an einem Lehrbuchbeispiel.

---

# Teil 3 — Echte blinde Flecken

Per Volltextsuche über den gesamten `Traineeship/`-Ordner geprüft. Nur was hier steht, hat wirklich **null Treffer** oder ist nachweislich nirgends zusammenhängend behandelt.

## Null Treffer im gesamten Vault

- [ ] **Die Twelve-Factor App** ⭐ Der fehlende Klammerbegriff. Zwölf Regeln für Anwendungen, die in Containern laufen — und du hast in diesem Kapitel unbewusst mehrere davon umgesetzt: Config aus der Umgebung (1.4/1.7), **Logs als Ereignisstrom nach stdout** (genau die `PYTHONUNBUFFERED`-Erkenntnis aus 1.7), Backing Services als angehängte Ressourcen (dein Postgres via `DB_HOST`), Prozesse zustandslos, **Dev/Prod-Parität** (das Docker-Argument), Admin-Prozesse als Einmal-Läufe (dein `migrate`-Dienst!). Eine halbe Stunde Lesezeit, die rückwirkend vier Kapitel zusammenbindet. → <https://12factor.net/de/>
- [ ] **Semantic Versioning** — `MAJOR.MINOR.PATCH` und was jede Stelle *verspricht*. Direkt anschlussfähig an „was ist ein Breaking Change?" aus 1.5 und an die Frage, wie man eigene Pakete und APIs versioniert.
- [ ] **Authentifizierung und Autorisierung als Thema** — in 1.5 als Faden benannt (`OAuth2PasswordBearer`, Scopes, API-Keys), aber der Leitfaden lässt es aus. SQL-Injection steht in 1.2, XSS/CSRF in 1.5, Secrets in 1.7 — nirgends als zusammenhängende Denkweise. Einstieg: OWASP Top 10 einmal überfliegen und je Punkt fragen „betrifft das mein Projekt?".
- [ ] **Performance messen statt raten** — Profiling kommt nirgends vor. Die Bausteine liegen verstreut herum (`EXPLAIN ANALYZE` in 1.2-Q&A, `curl -w` in 1.5-Q&A), aber das Python-Stück fehlt ganz (`cProfile`, `py-spy`) und die Methode wird nie benannt: **erst messen, dann optimieren, dann wieder messen.**
- [ ] **Metriken** — die Dreiteilung *Logs / Metriken / Traces* fehlt. Tracing ist in 1.4 bewusst out of scope, Metriken sind schlicht nicht vorgekommen. OpenTelemetry als heutiger Standard schließt 1.4 erst ab.

## Identifiziert, aber nirgends behandelt

Diese Punkte *sind* im Vault als to-do markiert — es gibt nur keine Notiz dazu. Kein blinder Fleck im engeren Sinn, aber offen:

- [ ] **Nebenläufigkeit am Stück** ⭐ Die GIL steht in Python Basics, `async`/`await` ist in 1.5-Q&A **zweimal 🔴**, Locking in 1.2, Race Conditions in 1.5 — nirgends zusammenhängend. Die eine Frage, die alles verbindet: *Warum bringen Threads in Python bei Rechenarbeit nichts, bei I/O aber schon?* Daraus folgt die ganze Landkarte: Prozesse vs. Threads vs. Event Loop, `multiprocessing`/`threading`/`asyncio`, `asyncio.to_thread`. Für Data Engineering (Parallelisierung von Ingest-Läufen) unmittelbar praktisch — und durch den optionalen No-GIL-Build ab Python 3.13 gerade wieder in Bewegung.

## Für dein konkretes Berufsfeld

- [ ] **Datenqualität als eigene Disziplin** — in 1.5-Q&A nur als Nebensatz („Pandera", „Great Expectations"). Der Gedanke dahinter ist aber eigenständig: Erwartungen an Daten *deklarieren und prüfen*, statt sie im Code zu unterstellen. Dein `_validate_net_revenue` ist genau das, nur handgeschrieben.
- [ ] **Zeit, Zeitzonen und `datetime`** — naive vs. aware `datetime`, warum man intern in UTC rechnet und erst an der Grenze lokalisiert. Kommt an vielen Stellen vor (`AwareDatetime`, `computed_at`, `reference_date`), aber nie als eigenes Thema. Klassische Fehlerquelle in jeder Pipeline.
- [ ] **Dateiformate für Daten** — CSV ist der Ausgangspunkt aller Übungsaufgaben; Parquet/Avro/ORC (spaltenorientiert, komprimiert, mit Schema) tauchen nur in 1.6-Q&A als Begriffe auf. Der Kontrast „zeilen- vs. spaltenorientiert" erklärt sehr viel darüber, warum Analytics-Systeme anders gebaut sind als OLTP-Datenbanken.

*(Nicht als blinder Fleck gelistet, weil im Vault vorhanden: Orchestrierung/Airflow/Dagster/Prefect und Dead-Letter-Queues stehen in der 1.4-Vertiefung, Retry/Backoff/Circuit Breaker in 1.5.)*

---

# Teil 4 — Priorisierung

## Wenn du drei Stunden hast

| # | Was | Warum zuerst | ca. |
|---|---|---|---|
| 1 | **Die 9 offenen + 2 Bonus-Fragen aus dem OOP-Selbsttest** ([[#Teil 2 — Die Rückstandsliste aus dem OOP-Selbsttest\|Teil 2]]) | Einziger Punkt mit *dokumentierter* Lücke, Musterlösungen liegen vor, Bezug zum eigenen Code. Höchster Ertrag. | 45 min |
| 2 | **1.2 Tier 1** — Window Functions, CTEs, `EXPLAIN` lesen, `NOT IN`-NULL-Falle, UPSERT, Keyset-Pagination | Tägliches Handwerkszeug für einen Data Engineer, in der Vertiefung ausdrücklich ausgelagert. | 55 min |
| 3 | **1.6: die offenen Projektfragen entscheiden** (nicht lesen — entscheiden) | `create_all()` vs. Alembic, Testdatenbank ja/nein, Backup-Strategie. Blockiert real deine nächsten Schritte. | 25 min |
| 4 | **Twelve-Factor App** | Bindet rückwirkend 1.4, 1.5, 1.6 und 1.7 zusammen. Bestes Verhältnis Erkenntnis/Aufwand. | 30 min |
| 5 | **1.7: PID-1-Falle, `0.0.0.0`, Layer-Cache-Kette, `PYTHONUNBUFFERED`** | Konkrete Fehler, die dich sonst garantiert ein zweites Mal kosten. | 25 min |

## Danach, in dieser Reihenfolge

1. **1.2 Tier 2 — MVCC/WAL/VACUUM + optimistisches vs. pessimistisches Locking.** Ein Konzept, drei Kapitel: erklärt gleichzeitig 1.6 (warum Migrationen teuer sind) und 1.5 (ETag/`If-Match` ist dasselbe Muster auf HTTP-Ebene).
2. **1.5 🔴-Block** — Paginierung, Filtern, Bulk, Conditional Requests, Transaktionsgrenzen. Die Löcher in jeder selbstgebauten API.
3. **1.6 🔴-Block** — Batched Backfill, Backup/PITR, Testcontainers, state-based vs. migration-based.
4. **Nebenläufigkeit am Stück** (GIL → Threads/Prozesse/Event Loop → `async`/`await`).
5. **1.6: Schema-Evolution im Data Engineering** (Avro, Schema Registry, Iceberg) — die Brücke von diesem Kapitel in dein Berufsfeld.

## Was du getrost liegen lassen kannst

- **1.2 Tier 4** — einmal überfliegen, dann liegen lassen. Begriff einordnen können reicht.
- Alle **⚪-Punkte** in 1.5 und 1.6 (Peer-to-Peer, Big Ball of Mud, Third-Party-Cookies, Event Sourcing, AsyncAPI, `alembic edit`).
- **HATEOAS** über das Konzept hinaus. Die 1.5-Q&A sagt es selbst: Der Nutzen entsteht erst, wenn Clients Links wirklich folgen — und das tun sie praktisch nie.
- **Liquibase-Details** — du hast Alembic. Die Kontrollfrage ist beantwortet; mehr braucht es erst in einem Java-Haus.
- Der **1.7-Katalog als Ganzes** — das ist ein Nachschlagewerk, kein Lernstoff. Die oben markierten Punkte lesen, den Rest wiederfinden, wenn etwas bricht.

---

# Teil 5 — Vorschlag zur Struktur

Diese Notiz ist eine **Erntenotiz**, kein Ersatz für die vorhandenen Register. Drei Anschluss-Schritte, die dem bestehenden Vault-Muster folgen statt ein neues zu erfinden:

1. **Die 1.1-Rückstandsliste dort abarbeiten, wo sie steht** — die Verdikte in [[Q&A – Code-Verständnis (OOP-Lösung)|Q&A – Code-Verständnis (OOP-Lösung)]] von ⬜ auf ✅ ändern, statt die Antworten hier zu duplizieren.
2. **Für 1.3 und 1.4 je eine `Q&A/Weiterführende Stichpunkte.md` anlegen**, nach der Vorlage aus 1.5/1.6 (🔴/🟡/⚪). Das schließt exakt die Sichtbarkeitslücke aus Teil 0 — das Rohmaterial dafür steht in Teil 1 dieser Notiz bereits.
3. **[[Stichwörter Claudian|Stichwörter Claudian]] (1.2) nachträglich priorisieren** — 🧩/🚀 auf 🔴/🟡/⚪ umstellen, damit es wie 1.5/1.6 überflogen werden kann. Die Tier-Einteilung in Teil 1 oben ist der Vorschlag dafür.

---

## Verwandte Notizen

- [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]] — die Quellen zu den meisten Punkten aus 1.5 und 1.6
- [[1.1 OOP - Vertiefung|1.1]] · [[1.2 Relationale Datenbanken & ORM - Vertiefung|1.2]] · [[1.3 Testing mit PyTest - Vertiefung|1.3]] · [[1.4 Logging & Exceptionhandling - Vertiefung|1.4]] · [[1.5 REST APIs & Layered Architecture - Vertiefung|1.5]] · [[1.6 Datenbank Migrations Tools - Vertiefung|1.6]] · [[1.7 Docker - Vertiefung|1.7]]
- [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md|Python Basics]] — dort liegen Scopes/Closures, das `os`- und `sys`-Modul, Konfiguration und `uv`
