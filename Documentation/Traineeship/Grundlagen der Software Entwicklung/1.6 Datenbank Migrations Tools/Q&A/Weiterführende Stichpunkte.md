Sammlung von Fäden, die in der [[1.6 Datenbank Migrations Tools - Vertiefung|Vertiefung]] bewusst nur gestreift oder ganz weggelassen wurden — als Landkarte für einen späteren, tieferen Durchgang. **Kein Pflichtstoff für das Kapitel.**

Legende: 🔴 lohnt sich sehr / berufsrelevant · 🟡 gut zu wissen · ⚪ Bonus, wenn Zeit und Lust da sind
Quellen zu den meisten Punkten: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]] (Abschnitt *Datenbank-Migrationen & Schema-Versionierung*).

Diese Notiz wächst mit dem Kapitel mit — bisher ist nur das erste Stichwort abgedeckt.

---

# Grundlagen der Datenbank-Migration und Versionierung

## Das große Gegenmodell, das nur als Nebensatz vorkam

- 🔴 **State-based vs. migration-based (deklarativ vs. imperativ)** — Die Vertiefung stellt das Deklarative (`create_all()`) als *Vorstufe* dar und das Imperative (Migrationen) als Lösung. Das ist die verbreitete, aber nicht die ganze Wahrheit. Es gibt eine komplette Werkzeugfamilie, die **deklarativ bleibt und den Übergang selbst ausrechnet**: du beschreibst nur den Sollzustand, das Werkzeug diffft gegen die echte Datenbank und generiert den Migrationsplan zur Laufzeit. Vertreter: **Atlas** (das derzeit meistdiskutierte, auch mit `migrate lint`), **Skeema** (MySQL), **migra** (PostgreSQL), **SQL Server DACPAC**, in Teilen auch Liquibases `diff`. Die Debatte „state-based vs. migration-based" ist eine echte, laufende Kontroverse — Kernargument der Gegenseite: Ein generierter Plan kennt deine Absicht nicht (Umbenennung!) und du reviewst etwas, das erst beim Deployment entsteht. Wer beide Lager kennt, argumentiert deutlich souveräner als jemand, der nur „man nimmt halt Alembic" sagt.
- 🟡 **Repeatable Migrations** — Flyway kennt neben versionierten (`V1__`) auch *wiederholbare* Migrationen (`R__`), die bei jeder Änderung ihrer Prüfsumme erneut laufen. Gedacht für Objekte, die man sinnvollerweise deklarativ pflegt: **Views, Stored Procedures, Functions, Trigger**. Ein hübscher Mittelweg — imperativ für Tabellen, deklarativ für alles, was keinen Zustand hat.
- 🟡 **Datenbankobjekte jenseits von Tabellen** — Die Vertiefung behandelt fast nur Tabellen/Spalten/Indizes/Constraints. Unter Versionskontrolle gehören genauso: Views, **Materialized Views** (für ein Pricing-Reporting hochrelevant), Sequences, Functions, Trigger, Extensions (`CREATE EXTENSION`), Rollen und Rechte. Autogenerate erkennt davon herzlich wenig.

## Der Notfallplan, den die Vertiefung nur benennt

- 🔴 **Backup & Point-in-Time-Recovery (PITR)** — Die Vertiefung sagt „der Notfallplan heißt Backup, nicht Downgrade", erklärt aber nicht, was das ist. Zum Nachholen: `pg_dump`/`pg_restore` (logisch, pro Datenbank) vs. `pg_basebackup` (physisch, ganzer Cluster), **WAL-Archivierung** als Grundlage für PITR („stelle den Stand von 14:32:07 wieder her — eine Minute vor der kaputten Migration"), und die einzige Frage, die zählt: **Wann wurde das Restore zuletzt geübt?** Ein Backup, das nie zurückgespielt wurde, ist eine Vermutung.
- 🟡 **Die Kopie vor der Migration** — In der Praxis oft die pragmatischste Absicherung bei destruktiven Schritten: `CREATE TABLE bookings_backup_20260814 AS SELECT * FROM bookings` in der Migration selbst, Löschen der Kopie in einer späteren Migration. Billig, wirksam, und der Grund, warum Contract-Schritte gern ein paar Wochen warten.
- 🟡 **Migration auf einer Kopie proben** — Der Standardweg für riskante Produktionsmigrationen: Restore eines Produktions-Backups in eine Staging-Umgebung, Migration dort laufen lassen, **Laufzeit messen**. Erst diese Messung sagt dir, ob die 5-ms-Migration von deinem Laptop auf 1,3 Mio. Zeilen 20 Sekunden oder 20 Minuten braucht.

## Große Datenmengen: der Teil, der bei dir real wird

- 🔴 **Batched Backfill** — Die Vertiefung zeigt Daten-Migrationen als einzelnes `UPDATE`. Bei 939.559 Buchungen ist das ein Problem: eine einzige riesige Transaktion hält Locks minutenlang, bläst das WAL auf und blockiert `VACUUM`. Die Praxislösung ist ein **Backfill in Blöcken** (z. B. 10.000 Zeilen pro Transaktion, in einer Schleife, ggf. als eigenes Skript **außerhalb** der Migration). Damit ändert sich auch die Struktur des Deployments: Die Migration legt nur die Spalte an, das Backfill läuft nebenher, und erst danach kommt die Constraint.
- 🔴 **MVCC, Bloat und `VACUUM`** — Der Grund, warum ein `UPDATE` in PostgreSQL teurer ist, als man denkt: Es schreibt jede betroffene Zeile **neu** und markiert die alte als tot. Ein Full-Table-Update verdoppelt die Tabellengröße auf der Platte, bis `VACUUM` aufräumt (und gibt den Platz nicht automatisch ans Betriebssystem zurück — dafür `VACUUM FULL` oder `pg_repack`). Ohne diesen Mechanismus versteht man weder die Laufzeiten noch die Plattenfüllstände nach einer Migration.
- 🟡 **Online-Schema-Change-Werkzeuge** — Für Datenbanken ohne transaktionale DDL bzw. für Änderungen, die zwingend die Tabelle umschreiben: `gh-ost` und `pt-online-schema-change` (MySQL-Welt, Schattentabelle + Trigger/Binlog + atomarer Rename), `pg_repack` in der PostgreSQL-Welt. Gut zu kennen, weil es zeigt, wie weit man gehen kann, wenn ein Wartungsfenster ausgeschlossen ist.
- 🟡 **Partitionierung** — Bei `capacity` (1,33 Mio. Zeilen, kalenderdatumsbasiert) der naheliegende nächste Schritt. Für dieses Kapitel relevant, weil **das Einführen einer Partitionierung selbst eine der anspruchsvollsten Migrationen überhaupt** ist: Die Tabelle muss faktisch neu aufgebaut und umgehängt werden.

## Migrationen und Tests — der praktisch wichtigste offene Faden

- 🔴 **Testcontainers / echte Datenbank im Test** — Deine 123 Tests laufen ohne Datenbank (Fakes über `dependency_overrides`), was für Unit-Tests genau richtig ist. Für Migrationen reicht es nicht: Eine Migration kann man nur gegen eine echte PostgreSQL-Instanz testen. Werkzeuge: **`testcontainers-python`** (startet einen Postgres-Container pro Testlauf), `pytest-postgresql`, oder schlicht der bereits vorhandene `docker-compose`-Dienst mit einer separaten Testdatenbank. Damit kommt eine Testkategorie ins Projekt, die es noch nicht gibt: **Integrationstests**.
- 🔴 **Was man an einer Migration überhaupt testet** — Nicht offensichtlich, deshalb als Liste: (1) *Die Kette läuft* — aus leerer DB `upgrade head` ohne Fehler. (2) *Kein Modell-Drift* — nach `upgrade head` erzeugt Autogenerate keinen Unterschied mehr (der Test aus der Vertiefung). (3) *Round-Trip* — `upgrade head` → `downgrade base` → `upgrade head` funktioniert. (4) *Datenerhalt* — Zeilen einfügen, migrieren, prüfen dass die Werte korrekt transformiert sind. Punkt 4 ist der einzige, der die eigentliche Fachlichkeit prüft, und der einzige, den man von Hand schreiben muss.
- 🟡 **Migrations-Linter** — Werkzeuge, die eine Migration statisch auf gefährliche Muster prüfen, bevor sie irgendwo läuft: **`squawk`** (PostgreSQL-spezifisch, sehr gut), `atlas migrate lint`, im Rails-Umfeld `strong_migrations`. Sie kennen genau die Fallen aus Abschnitt 8 der Vertiefung (`SET NOT NULL` ohne Vorbereitung, `CREATE INDEX` ohne `CONCURRENTLY`, Typänderungen mit Rewrite) und machen daraus einen CI-Check. Passt exakt zu der Idee, die dein Projekt mit `test_architektur` schon verfolgt: eine Regel, die nicht von Disziplin abhängt.
- 🟡 **Seed- und Referenzdaten** — In der Vertiefung ganz ausgelassen. Die Frage: Gehören Stammdaten (Länderlisten, Statuswerte, Standard-Konfiguration) in Migrationen, in eigene Seed-Skripte oder in Fixtures? Faustregel: Daten, ohne die die *Anwendung* nicht startet, gehören in Migrationen; Daten, ohne die nur der *Entwickler* nicht arbeiten kann, gehören in Seed-Skripte. Für dich direkt greifbar an `api_keys` — braucht eine frische Datenbank einen initialen Schlüssel, und wenn ja, woher?

## Betrieb, Team und Prozess

- 🟡 **Least Privilege für Datenbank-Rollen** — Ein Punkt, den die Vertiefung nur in einem Nebensatz streift („jede Instanz bräuchte DDL-Rechte"). Sauber ist die Trennung: ein **Migrations-Benutzer** mit DDL-Rechten, der nur im Deployment-Schritt verwendet wird, und ein **Anwendungs-Benutzer**, der ausschließlich `SELECT/INSERT/UPDATE/DELETE` darf. Dann kann ein kompromittierter Anwendungsprozess keine Tabelle löschen. Dein Projekt hat aktuell genau einen Benutzer in der `.env`.
- 🟡 **Migrations-Squashing** — Nach zwei Jahren hat ein Projekt schnell 300 Migrationsdateien, und der Aufbau einer frischen Datenbank dauert Minuten. Django hat dafür `squashmigrations`; anderswo macht man es von Hand (neue Baseline aus dem aktuellen Schema, alte Kette archivieren). Die Frage dahinter ist interessanter als die Technik: **Wann darf man die Vergangenheit wegwerfen?**
- 🟡 **Multi-Tenancy** — Wenn jeder Kunde ein eigenes Schema oder eine eigene Datenbank hat: Wie migriert man 400 Datenbanken, was tut man, wenn Nr. 137 fehlschlägt, und wie sieht „teilweise migriert" dann aus? Verbreiteter Fall in der Praxis und ein hübsches Argument dafür, warum Migrationen unbedingt idempotent *aufrufbar* sein müssen.
- 🟡 **Datenbankänderungen im Code Review** — Was schaut man sich in einem PR mit Migration konkret an? Checkliste: Ist sie destruktiv? Läuft sie in einer Transaktion? Wie lange dauert sie auf Produktionsdatengröße? Ist sie N-1-kompatibel? Gibt es ein `downgrade`? Wird ein Index `CONCURRENTLY` gebaut? — Das wäre eine gute eigene kleine Notiz.
- ⚪ **Wartungsfenster als legitime Antwort** — Die ganze Zero-Downtime-Maschinerie ist teuer. Für viele interne Systeme ist „Sonntag 3 Uhr, 20 Minuten Ausfall, dafür ein einziges einfaches `ALTER TABLE`" die wirtschaftlich richtige Entscheidung. Wichtig, weil die Literatur fast ausschließlich den Zero-Downtime-Fall behandelt und dabei so tut, als gäbe es keine Alternative.

## Für dich als Data Engineer besonders anschlussfähig

- 🔴 **Schema-Evolution außerhalb relationaler Datenbanken** — Dasselbe Problem, ganz andere Werkzeuge, und genau dein künftiges Berufsfeld: **Avro** (die kanonische Schema-Evolution mit Reader-/Writer-Schema und Kompatibilitätsregeln), **Parquet**, **Confluent Schema Registry** (mit `BACKWARD`/`FORWARD`/`FULL`-Kompatibilitätsmodi — die formale Version dessen, was in der Vertiefung „N-1-Kompatibilität" heißt), **Delta Lake / Apache Iceberg** (Schema Evolution und *Time Travel* — Migrationen und Rollback auf Tabellenformat-Ebene). Wer die Konzepte aus diesem Kapitel hat, liest die Doku dieser Systeme mit direktem Wiedererkennen.
- 🟡 **dbt und die analytische Seite** — Im Analytics-Umfeld verwaltet man Modelle deklarativ (`dbt run` baut Tabellen/Views neu auf), weil die Daten **ableitbar** sind. Das ist genau die Ausnahme, die auch dein Projekt bisher gerettet hat („alles kommt aus CSVs"). Der Kontrast lohnt sich: Wo Daten reproduzierbar sind, braucht es keine Migrationen — wo sie es nicht sind, unbedingt.
- 🟡 **Schemaless / Schema-on-Read** — MongoDB & Co. verschieben das Problem nur: Statt einer Migration schreibt man Code, der beide Dokumentformate versteht („migration on read"), plus einen Hintergrundprozess, der nachzieht. Nützlich zum Verständnis, warum „wir brauchen kein Schema" nie stimmt — es ist dann eben im Anwendungscode.
- ⚪ **Event Sourcing** — Das radikale Gegenmodell: Nicht den Zustand speichern, sondern die Folge der Ereignisse; der Zustand wird daraus errechnet. Schema-Migrationen werden dadurch nicht überflüssig, verlagern sich aber auf Event-Versionierung und *Upcasting*.

## Fragen zum eigenen Projekt, die offen geblieben sind

- **Was passiert mit `pricing-initdb` und `create_all()`?** Drei Optionen: (a) ersatzlos streichen, Schema entsteht nur noch über `upgrade head`; (b) behalten, aber intern auf Alembic umstellen (das Kommando bleibt, die Mechanik wechselt); (c) beides behalten für unterschiedliche Zwecke — riskant, weil dann zwei Wege ein Schema erzeugen, die auseinanderlaufen können.
- **Reicht *eine* Baseline-Migration für alle 9 Tabellen, oder schneidet man sie?** Eine Datei ist ehrlicher (so sieht der Ist-Zustand aus), mehrere wären eine erfundene Geschichte. Argument für eine: Sie ist ohnehin nie „gelaufen", sondern nur gestempelt.
- **Wird die Baseline per Autogenerate erzeugt oder von Hand geschrieben?** Autogenerate gegen eine leere Datenbank ist der Standardweg — aber `net_revenue` (`Computed`), die `server_default`-Werte und die `CheckConstraint`s sind genau die Kandidaten, bei denen nachgearbeitet werden muss.
- **Braucht das Projekt jetzt eine echte Testdatenbank?** Ohne sie kann die Migrationskette nie geprüft werden. Mit ihr kommt eine ganz neue Testkategorie (Integrationstests) samt Laufzeit und Docker-Abhängigkeit ins Projekt. Das ist eine bewusste Entscheidung, keine Nebensache.
- **Wie verträgt sich der `TRUNCATE` im Ingest mit Migrationen?** `pricing-ingest` leert `products`, `customers`, `capacity`, `bookings`, `prices`. Was passiert dabei mit `price_recommendations`, das per FK an `capacity` hängt — und mit `pricing_runs`, das auf Läufe verweist, deren Datengrundlage gerade gelöscht wurde? Diese Frage ist unabhängig von Alembic, wird durch das Kapitel aber sichtbar.
- **Sollten „abgeleitete" und „echte" Daten getrennt werden?** `products`…`prices` sind jederzeit aus CSVs reproduzierbar; `api_keys`, `pricing_runs`, `ingestion_runs` sind es nicht. Wäre eine Trennung (eigenes Schema, andere Backup-Strategie, andere Migrationsvorsicht) sinnvoll — oder überkonstruiert?
- **Gibt es überhaupt eine Backup-Strategie?** Aktuell hängt alles an einem Docker-Volume. Für ein Übungsprojekt in Ordnung — aber die Frage einmal bewusst zu beantworten, gehört zum Thema.
- **Wäre ein `squawk`-Lauf über die erste selbstgeschriebene Migration ein lohnender kleiner CI-Schritt?** Passt zum bereits etablierten Muster (`ruff`, `mypy`, `test_architektur`): Regeln, die nicht von Disziplin abhängen.

---

# Alembic

## Konfiguration und Umgebung, tiefer

- 🔴 **`env.py` als Composition Root ernst nehmen** — Die Vertiefung sagt „hol dir die URL aus `shared/config.py`", lässt aber die interessantere Frage offen: Soll `env.py` `bootstrap.engine()` **mitbenutzen** oder sich eine **eigene** Engine bauen? Argumente für mitbenutzen: eine Quelle für Verbindungsaufbau, kein zweiter Ort für Pool-Einstellungen. Argumente dagegen: `bootstrap.engine()` ist `@lru_cache`-gecacht und für den Anwendungsbetrieb gedacht; ein Migrationslauf will vielleicht andere Einstellungen (kein Pooling, längeres Timeout, ein *anderer* Datenbank-Benutzer mit DDL-Rechten). Das ist eine echte Entwurfsentscheidung mit Argumenten auf beiden Seiten.
- 🟡 **`version_table`, `version_table_schema`, `include_schemas`** — Alembic kann die Versionstabelle umbenennen oder in ein eigenes Schema legen und auch mehrere PostgreSQL-Schemas verwalten. Relevant, sobald eine Datenbank mehr als eine Anwendung beherbergt.
- 🟡 **`include_object` / `include_name`** — Callbacks, mit denen man Tabellen gezielt vom Autogenerate ausschließt. Der klassische Anwendungsfall: fremde Tabellen in derselben Datenbank, die nicht dir gehören und die Alembic sonst fröhlich zu löschen vorschlägt.
- 🟡 **Post-Write-Hooks** — Alembic kann nach dem Erzeugen einer Migration automatisch Formatter/Linter darüberlaufen lassen. Für dieses Projekt naheliegend: `ruff format` und `ruff check --fix`, damit generierte Dateien dieselbe Prüfkette bestehen wie handgeschriebener Code.
- 🟡 **`file_template` und datumsbasierte Unterordner** — Die Standardnamen (`a3f21c9d4b17_channel_spalte.py`) sind unsortierbar. Ein `file_template` mit Datumspräfix macht `versions/` chronologisch lesbar; mit `recursive_version_locations` gehen sogar Jahres-/Monatsordner. Bei 300 Migrationen ist das kein Luxus mehr.
- ⚪ **`alembic_version` mit mehreren Zeilen** — Die Vertiefung sagt „in der Regel genau eine Zeile". Bei mehreren unabhängigen Zweigen (`branch_labels`) stehen dort tatsächlich mehrere. Gut zu wissen, bevor man das für einen Defekt hält.

## Autogenerate über die Standardfälle hinaus

- 🔴 **`compare_server_default` am eigenen Schema erproben** — Die Doku schaltet den Vergleich bewusst standardmäßig ab, damit man ihn erst testet. Für dieses Projekt ist das eine konkrete kleine Untersuchung: einschalten, `alembic check` laufen lassen und sehen, ob `base_currency`, `created_at` und `computed_at` ruhig bleiben oder bei jedem Lauf Phantom-Unterschiede melden. Das Ergebnis entscheidet, ob der Schalter dauerhaft an bleibt.
- 🔴 **Umbenennungen richtig hinschreiben** — Autogenerate liefert Drop + Add. Die Handkorrektur ist `op.alter_column("tabelle", "alt", new_column_name="neu")` bzw. `op.rename_table()`. Wert der Vertiefung: einmal bewusst durchspielen, wie eine solche Korrektur aussieht — und dass sie **das eigentliche Argument** gegen blindes Übernehmen generierter Migrationen ist.
- 🟡 **Erweiterungen für Views, Funktionen, Trigger und Enums** — `alembic-utils` bringt Autogenerate-Unterstützung für PostgreSQL-Views, -Funktionen und -Trigger; `alembic-postgresql-enum` für das Anlegen/Ändern/Löschen von Enums. Beides Standardantworten auf Lücken, die man sonst von Hand füllt.
- 🟡 **`autogenerate_plugins`** — Der Mechanismus, über den die neue CHECK-Erkennung ein- und abschaltbar ist (`"~alembic.autogenerate.checkconstraint_byname"`). Dahinter steckt ein allgemeines Plugin-System, mit dem man eigene Vergleichsregeln beisteuern kann.
- 🟡 **Eigene Typvergleiche** — `compare_type` akzeptiert auch eine Callable. Nötig, sobald man eigene `TypeDecorator`-Typen benutzt, die Alembic sonst bei jedem Lauf als geändert meldet.

## Rund um die Migration selbst

- 🔴 **Transaktionsverhalten pro Migration steuern** — Alembic klammert den Lauf standardmäßig in eine Transaktion. Für `CREATE INDEX CONCURRENTLY` (siehe Stichwort 1, Abschnitt 8) muss man da heraus: `transaction_per_migration` bzw. ein expliziter `AUTOCOMMIT`-Block. Das ist die praktische Brücke zwischen der Theorie „auf großen Tabellen zählt das Verfahren" und dem, was Alembic tatsächlich tut.
- 🟡 **`op.bulk_insert()` vs. `op.execute()`** — Für Seed-Daten braucht `bulk_insert()` eine im Skript **lokal deklarierte** Tabelle (`sa.table(...)`) — genau die Technik, die die Regel „keine ORM-Modelle importieren" praktisch macht. Lohnt sich, einmal gesehen zu haben.
- 🟡 **Offline-Modus richtig nutzen** — `--sql` erzeugt ein Skript, das die `alembic_version`-Updates mit enthält. Wissenswert: In diesem Modus kann Alembic nichts aus der Datenbank *lesen*, weshalb Migrationen mit `op.get_bind()`-Abfragen dort schlicht nicht funktionieren. Ein Grund, Datenmigrationen als reines SQL zu formulieren.
- ⚪ **`alembic ensure_version`, `alembic edit`, `alembic list_templates`** — die selten genutzten Kommandos. `edit` öffnet eine Revision im `$EDITOR`, was beim Nacharbeiten generierter Migrationen bequemer ist als der Umweg über den Dateipfad.

## Fragen zum eigenen Projekt

*Beantwortet durch die Umsetzung am 15.08. — Details im [[Umsetzungsstand (Datenbank Migrations Tools)|Umsetzungsstand]].*

- ✅ **Benutzt `env.py` `bootstrap.engine()` oder eine eigene Engine?** → **`get_engine()` direkt** aus `persistence/db.py`, nicht der gecachte `bootstrap.engine()`. Für einen kurzlebigen CLI-Prozess ist Pooling irrelevant, und die Migration bleibt vom Anwendungs-Bootstrap unabhängig. *(Die Anschlussfrage bleibt: Falls später ein separater Migrations-Benutzer mit DDL-Rechten kommt, ist dieser Weg der einfachere — er hat genau einen Ort, an dem die Verbindung entsteht.)*
- ✅ **`pyproject`-Template oder `generic`?** → **`generic`.** Das `pyproject`-Template erzeugt trotzdem eine `alembic.ini`, und deren Hauptinhalt (die URL) wandert hier ohnehin nach `env.py`. Der Gewinn wäre nur kosmetisch gewesen.
- ✅ **Tragen die Constraints in der laufenden Datenbank die Namen der `NAMING_CONVENTION`?** → Frage hat sich erübrigt: Die Datenbank war leer (neues Docker-Volume durch den anderen Compose-Projektnamen), das Schema entstand komplett neu aus der Migration.
- ✅ **Was macht Autogenerate aus `Computed(...)`?** → Korrekt gerendert als `sa.Computed("gross_revenue - discount_amount", persisted=True)` innerhalb von `create_table`.
- ✅ **Bleibt `Identity(always=True)` erhalten?** → **Ja, 7 von 7** — und ohne ergänztes `start`/`increment`, also näher an `orm.py` als erwartet.
- ⬜ **Wird `alembic check` Teil der Prüfkette?** → Noch offen; hängt an der Testdatenbank-Frage. `ruff` deckt `alembic/` inzwischen mit ab.
- ⬜ **Wie geht Alembic mit `price_recommendations` und `TRUNCATE … RESTART IDENTITY` um?** → Für das Schema weiterhin irrelevant, die Kombination mit `Identity(always=True)` bleibt beobachtenswert.

**Neu aufgeworfen durch die Umsetzung:**

- 🔴 **Wann greift eine Naming Convention auf einen *explizit* übergebenen Namen?** Die Umsetzung hat es auf die harte Tour gezeigt: Conventions mit dem Platzhalter `%(constraint_name)s` (bei uns `ck`) greifen **auch auf schon vollständige Namen** und präfixieren doppelt; Conventions ohne diesen Platzhalter (`ix`, `uq`, `fk`, `pk`) greifen nur bei unbenannten Objekten. Das erklärt, warum in der Baseline Constraint-Namen in `op.f()` stehen und Index-Namen nicht. Lohnt sich, in der SQLAlchemy-Doku zu „Configuring Constraint Naming Conventions" einmal ganz zu lesen.
- 🟡 **Was passiert bei einer gescheiterten Migration in einer Datenbank ohne transaktionales DDL?** Bei uns hat PostgreSQL alles zurückgerollt. Zum Verständnis lohnt der Gegentest gedanklich durchzuspielen: Welche Aufräumarbeit wäre in MySQL nötig gewesen, und wie repariert man dort ein `alembic_version`, das nicht mehr zum Schema passt? (Antwort führt zu `alembic stamp`.)

---

# Liquibase

## Funktionen, die nur gestreift wurden

- 🟡 **`generate-changelog` gegen eine bestehende Datenbank** — Liquibase' Baseline-Weg: Es liest ein vorhandenes Schema aus und schreibt daraus ein Changelog. Interessant als Kontrast zu Alembics `--autogenerate` gegen eine *leere* Datenbank, und weil es für „ich erbe eine 15 Jahre alte Datenbank ohne jede Versionierung" das gängige Werkzeug ist.
- 🟡 **`snapshot` und `diff` als eigenständige Werkzeuge** — Liquibase kann einen Schema-Snapshot als JSON ablegen und später gegen eine Datenbank diffen. Damit lässt sich Schema-Drift *nachweisen*, auch ohne Migrationen zu schreiben. Nützlich als Audit-Werkzeug selbst in Projekten, die Liquibase gar nicht zum Migrieren benutzen.
- 🟡 **`modifySql`** — nachträgliches Anpassen des generierten SQL pro Datenbanktyp (z. B. `append` einer PostgreSQL-spezifischen Klausel). Der Notausgang, wenn die Abstraktion nicht ganz reicht, ohne gleich auf rohes `<sql>` umzusteigen.
- 🟡 **`runInTransaction`, `failOnError`, `runAlways`, `runOnChange`** — die Changeset-Attribute, die das Ausführungsverhalten steuern. `runOnChange` ist Liquibase' Antwort auf Flyways „repeatable migrations" (für Views und Prozeduren), `runInTransaction="false"` der Weg für `CREATE INDEX CONCURRENTLY`.
- ⚪ **`liquibase-hub` / Reporting und Policy Checks** — Der Teil, der im kommerziellen **Liquibase Secure** liegt: automatisierte Regelprüfungen auf Changesets („kein `DROP TABLE` ohne Freigabe"), Reports, Freigabe-Workflows. Konzeptionell dasselbe wie `squawk` aus dem ersten Stichwort, nur als Produkt.

## Einordnung und Marktumfeld

- 🔴 **Flyway als dritter im Bunde** — Der Leitfaden nennt nur Alembic und Liquibase, aber jede reale Werkzeugdiskussion hat drei Kandidaten. Flyway ist **SQL-first** (`V1__beschreibung.sql`), radikal einfach und damit gewissermaßen die Gegenposition zu beiden: keine Abstraktion wie Liquibase, kein Code wie Alembic. Lohnt sich, weil es die Achse „wie viel Abstraktion will ich?" erst vollständig macht. Auch interessant: Flyway bietet `undo` nur kommerziell an, mit der ausdrücklichen Begründung, dass Undo-Skripte in der Praxis selten funktionieren.
- 🔴 **Der Lizenztrend bei Infrastruktur-Werkzeugen** — Liquibase' Wechsel zu FSL ist kein Einzelfall, sondern Teil einer Bewegung: BSL, SSPL, Elastic License, FSL. Die Muster ähneln sich (Nutzung frei, „Konkurrenzprodukt bauen" verboten, oft Umwandlung in eine echte Open-Source-Lizenz nach 2–4 Jahren). Wer Werkzeuge auswählt, muss diese Lizenzfamilie inzwischen lesen können — und wissen, dass OSI-Konformität und „frei verfügbar" nicht mehr dasselbe sind.
- 🟡 **Framework-eigene Migrationssysteme** — Django Migrations, Rails Active Record Migrations, Entity Framework Core Migrations, Laravel Migrations. Alle lösen dasselbe Problem, alle sind ans jeweilige ORM gebunden (wie Alembic), und alle haben eigene Antworten auf Squashing, Branching und Datenmigrationen. Ein Blick auf Django lohnt besonders, weil es Migrationen **automatisch** erzeugt und damit noch einen Schritt weiter geht als Alembic.
- 🟡 **Warum Java-Werkzeuge in Datenbankprojekten überrepräsentiert sind** — Liquibase und Flyway kommen beide aus der Java-Welt. Der Grund ist historisch interessant: JDBC war lange die einzige einheitliche Datenbankschnittstelle über viele Systeme hinweg, und Enterprise-Java war dort, wo viele verschiedene Datenbanken in einem Haus standen.

## Fragen zum eigenen Projekt, die offen geblieben sind

- **Wäre Liquibase' `contexts`-Mechanismus etwas, das man in Alembic nachbauen sollte?** Die Frage wird konkret, sobald Seed-Daten oder Testdaten ins Spiel kommen: In Alembic bräuchte man dafür eine eigene Konvention (z. B. eine Umgebungsvariable, die `env.py` auswertet). Lohnt sich das, oder ist es ein Problem, das dieses Projekt gar nicht hat?
- **Fehlt dem Projekt eine Audit-Historie?** `DATABASECHANGELOG` beantwortet „wer hat wann was geändert". `alembic_version` nicht. Für ein Übungsprojekt irrelevant — aber wäre es das auch, wenn die Anwendung Preisempfehlungen für echtes Geld erzeugt? Und wenn ja: Reicht dafür die git-Historie der Migrationsdateien plus die Deployment-Logs, oder braucht es die Information in der Datenbank selbst?
- **Der Prüfsummen-Gedanke ohne Liquibase** — Ließe sich Liquibase' `MD5SUM`-Schutz in Alembic nachbilden, z. B. als CI-Test, der prüft, dass bereits in `main` gemergte Migrationsdateien nicht mehr verändert wurden (`git diff` gegen den Merge-Base)? Das wäre dieselbe Idee wie `test_architektur`: eine Regel, die nicht von Disziplin abhängt.
- **`update-testing-rollback` in Alembic nachbauen** — Der Test „hoch, runter, wieder hoch" ist in pytest mit einer Testdatenbank in wenigen Zeilen machbar und schließt genau die Lücke, die die Vertiefung bei `downgrade()` benennt. Guter Kandidat für die Übungsaufgabe, sobald eine Testdatenbank existiert.
