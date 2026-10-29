---
deck: "Traineeship::Grundlagen der Software Entwicklung::Datenbank Migrations Tools"
---

Karteikarten zu [[1.6 Datenbank Migrations Tools - Vertiefung|1.6 Datenbank Migrations Tools – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::Datenbank Migrations Tools`. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr, bei dünnem weniger — nicht auf eine Zielzahl auffüllen. Details, die nicht auf eine Karte passen, stehen in der Vertiefung.

# Fortschritt

- [x] Grundlagen der Datenbank-Migration und Versionierung
- [x] Alembic
- [x] Liquibase (inkl. Kontrollfrage Alembic vs. Liquibase)

---

# Grundlagen der Datenbank-Migration und Versionierung

<!-- basicblock-start oid="ObsNbsL5fPErj8UyUFOK3A0o" -->
Was ist eine **Migration** (im Datenbankkontext)?::
Eine **versionierte, ausführbare, im Repository liegende Änderung am Datenbankschema**, die die Datenbank von einem definierten Zustand in den nächsten überführt.<br>Vier tragende Wörter: **versioniert** (eindeutige Kennung + Position in einer Reihenfolge), **ausführbar** (Code, kein Dokument), **im Repository** (durchläuft Review und Deployment wie jeder Code), **Übergang** (beschreibt einen Weg, keinen Sollzustand).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsnu3MfqOr2CeN4QFvvybPV" -->
Warum reicht `Base.metadata.create_all()` nicht als Schemaverwaltung — obwohl es dasselbe Schema kennt wie eine Migration?::
`create_all()` vergleicht nur **Tabellennamen** und legt **fehlende Tabellen** an. Es schaut nicht in bestehende Tabellen hinein: neue Spalten, geänderte Typen, neue Indizes und Constraints entstehen nie.<br>Das Tückische: **keine Fehlermeldung** — der Lauf ist grün, der Fehler erscheint später als `UndefinedColumn` oder als lautlos falsches Ergebnis.<br>Grundsätzlich: `create_all()` ist **deklarativ** („so soll es aussehen"), eine bestehende Datenbank braucht aber **imperativ** („so kommst du von hier dorthin").
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsewvsr2nOCFfIGpEjYXMnH" -->
Warum sind Datenbanken das schwierige Ende jedes Deployments? Nenne den Kernunterschied zu Code.::
**Code ist ersetzbar, Daten sind es nicht.** Ein Deployment tauscht Code vollständig aus; eine Datenbank kann man nicht austauschen, nur *transformieren*, ohne Information zu verlieren.<br>Dazu vier Eigenschaften: **zustandsbehaftet** (kein „neu deployen"), **geteilt** (das Schema ist oft ein öffentlicher Vertrag mehrerer Anwendungen), **groß** (Änderungen dauern und sperren), **unumkehrbar bei Löschung** (`DROP COLUMN` ist in Sekunden erledigt und endgültig).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNm1AZdZp8wE3npfOoZlI2" -->
Was ist **Schema-Drift**?::
Der Zustand, in dem Datenbanken, die dasselbe Schema haben *sollten* (lokal, CI, Staging, Produktion), es nicht mehr haben — weil Änderungen manuell, unvollständig oder in unterschiedlicher Form angewandt wurden.<br>Niemand kann mehr sagen, welche Datenbank welchen Stand hat. Genau **dieses** Problem lösen Migrationswerkzeuge — nicht „SQL schreiben", sondern **wissen, welches SQL wo schon gelaufen ist**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRaxVwC0BwfPimSN0MTppK" -->
Unterscheide **Schema-Migration (DDL)** und **Daten-Migration (DML)** — und warum stehen sie oft in derselben Datei?::
**DDL** verändert die *Struktur* (`ALTER TABLE … ADD COLUMN`), **DML** den *Inhalt* (`UPDATE … SET …`).<br>DDL ist meist kurz und oft strukturell umkehrbar; DML skaliert mit der Zeilenzahl und ist fast nie umkehrbar (der alte Wert ist überschrieben).<br>Sie hängen voneinander ab: Eine neue `NOT NULL`-Spalte braucht erst die Spalte (DDL), dann Werte für bestehende Zeilen (DML), dann die Constraint (DDL) — in dieser Reihenfolge und idealerweise einer Transaktion.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRpxQX5e4E5VxYQrUspBpk" -->
Nenne die vier **Reifegrade** der Schemaverwaltung.::
**Stufe 0 – Ad hoc:** `ALTER TABLE` in der Konsole, keine Aufzeichnung.<br>**Stufe 1 – `schema.sql` im Repo:** versioniert und reviewbar, beschreibt aber nur den Sollzustand; bestehende DBs bleiben Handarbeit.<br>**Stufe 2 – ORM besitzt das Schema (`create_all()`):** eine Quelle der Wahrheit, aber kann nur anlegen, nicht ändern.<br>**Stufe 3 – Migrationswerkzeug:** Schema als Folge von Übergängen, die **DB kennt ihren eigenen Stand**.<br>Der Sprung 2 → 3 besteht aus genau zwei Fähigkeiten: *bestehende DB änderbar* und *DB kennt ihre Version*.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseFNiXOyi68QIhIV6HYKbS" -->
Nenne die **fünf Bausteine**, die jedes Migrationswerkzeug hat.::
1. **Versionstabelle in der Datenbank** (`alembic_version`, `DATABASECHANGELOG`, `flyway_schema_history`, …)<br>2. **Migrationsdateien** im Repository, eine pro Übergang<br>3. **Definierte Reihenfolge** (Sequenznummern/Zeitstempel oder verkettete Revisionen)<br>4. **Ein Runner** (`alembic upgrade head`, `liquibase update`, `flyway migrate`)<br>5. **Autogenerate/Diff** zwischen Modell und Datenbank (optional, aber der Komfortgewinn)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsx3wydQFKT2nZZQevSuTHa" -->
Warum steht die Versionsnummer **in der Datenbank** und nicht in einer Datei im Repository? Nenne zwei Konsequenzen.::
Weil Zustand und Versionsangabe damit **am selben Ort liegen** und nicht mehr versehentlich getrennt werden können: Wird die DB kopiert oder ein Backup eingespielt, kommt die Version mit.<br>**Konsequenz 1:** Weil PostgreSQL DDL transaktional kann, laufen Änderung *und* Fortschreiben der Version **in derselben Transaktion** — es gibt keinen Zustand „Migration lief, wurde aber nicht vermerkt".<br>**Konsequenz 2:** Die Frage „welchen Stand hat Produktion?" wird von einer Recherche zu einer Abfrage: `SELECT version_num FROM alembic_version;`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqfQ4lKUsbHFtRq1aSKriP" -->
Warum darf eine bereits ausgelieferte Migration **nicht mehr editiert** werden — und was macht man stattdessen?::
Datenbanken, auf denen sie schon lief, führen sie nie wieder aus (sie steht in der Versionstabelle). Die Korrektur würde nur auf Datenbanken wirken, die die Migration noch vor sich haben → **zwei DBs mit derselben Versionsnummer und unterschiedlichem Schema** — Drift, die das Werkzeug nicht mehr erkennen *kann*.<br>Stattdessen: eine **neue** Migration („roll forward, don't edit").<br>Ausnahme: solange sie nur lokal lief und nicht gepusht ist.<br>*Liquibase* erzwingt das über **Prüfsummen** je Changeset; *Alembic* verlässt sich auf Disziplin.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWQaZE09Qrj7hpX6NiFkK6" -->
Was passiert, wenn zwei Feature-Branches je eine Migration mit **demselben Vorgänger** anlegen — und warum ist das ein Vorteil?::
Der Revisionsgraph bekommt **zwei Köpfe (heads)**, und das Werkzeug weigert sich, `upgrade head` auszuführen, weil unklar ist, welcher gemeint ist. Aufgelöst wird das mit einer **Merge-Migration**, die beide Köpfe zusammenführt (oft mit leerem `upgrade()`).<br>Der Vorteil: Der Konflikt wird **sichtbar**, statt zu einer stillen Drift zu werden — ein Mensch entscheidet, ob die Zweige fachlich kollidieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstOiX5EqPeCWv9ypk5i37D" -->
Was leistet **Autogenerate** — und warum ist das Ergebnis nur ein Entwurf?::
Es vergleicht das ORM-Metadata-Objekt mit dem Ist-Zustand der DB und schreibt den Migrationsentwurf (`alembic revision --autogenerate`, Liquibase `diffChangeLog`).<br>**Grenze 1 – Absicht ist nicht sichtbar:** „Spalte `a` weg, Spalte `b` da" kann eine **Umbenennung** (Daten behalten) oder Drop+Add (Daten weg) sein. Der Generator wählt Drop+Add — mit Datenverlust.<br>**Grenze 2 – nicht alles wird erkannt:** Check-Constraints, `Computed`-Spalten und `server_default` sind klassische Kandidaten.<br>Merksatz: **Autogenerate erspart das Tippen, nicht das Denken.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRjUPUxsuWfFA1dRCfspWX" -->
Nenne die **drei Bedeutungen von „Rollback"** und wer sie jeweils auslöst.::
1. **Transaktions-Rollback** — `ROLLBACK` einer DB-Transaktion; verwirft alles seit `BEGIN`. Verlustfrei und von der DB garantiert (Atomarität). *Auslöser: die Datenbank, automatisch.*<br>2. **Migrations-Rollback (Downgrade)** — `downgrade()` nimmt eine angewandte Migration zurück und senkt die Versionsnummer. Stellt nur die **Struktur** wieder her. *Auslöser: ein Mensch, bewusst.*<br>3. **Deployment-Rollback** — Zurückrollen der Softwareversion. *Auslöser: die CI/CD-Pipeline.*<br>Der gefährliche Fall ist Nr. 3, weil der Code zurückspringt, das Schema aber nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAXfzYqydb8LpqHZKU23xS" -->
In welchem Sinn ist `downgrade()` unvollständig — und wofür schreibt man es trotzdem?::
**Die Struktur ist umkehrbar, die Daten sind es nicht.** Nach `DROP COLUMN` bringt `downgrade()` die Spalte zurück — **leer**. Ebenso bei `DROP TABLE` oder einer Typverengung (abgeschnittene Nachkommastellen).<br>Deshalb wird in Produktion fast nie downgegradet, sondern **vorwärts korrigiert** („roll forward"): `downgrade` läuft auf einem Zustand, den es beim Schreiben nicht kannte, und wird praktisch nie getestet.<br>Geschrieben wird es trotzdem — für die **lokale Entwicklung** (Branch-Wechsel, Migration verwerfen und neu erzeugen). Der Notfallplan bei Datenverlust heißt **Backup / Point-in-Time-Recovery**, nicht Downgrade.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKAuzSfRTEHfS6xnNPfeWT" -->
Warum hinterlässt eine gescheiterte Migration in **PostgreSQL** keinen Halbzustand, in **MySQL** aber schon?::
PostgreSQL (und SQL Server, SQLite) können **DDL in Transaktionen**: Schlägt Schritt 2 fehl, wird auch Schritt 1 zurückgenommen — die DB steht exakt im Ausgangszustand, inklusive unveränderter Versionsnummer.<br>MySQL/MariaDB und Oracle lösen bei **jeder DDL-Anweisung einen impliziten Commit** aus: Schritt 1 ist fest eingebrannt, die Versionsnummer steht noch auf alt, und die DB ist in einem Zustand, den keine Version beschreibt.<br>Daraus die allgemeine Regel: **eine Migration = eine logische Änderung.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsi6Dq4KhGGUFic2jrtuHRO" -->
Was bedeutet **N-1-Kompatibilität** bei Schemaänderungen, und warum ist sie nötig?::
Das Schema **nach** der Migration muss auch von der **vorherigen Codeversion** benutzbar sein.<br>Zwei Gründe: (1) Bei einem **Rolling Deployment** laufen alte und neue Codeversion minutenlang gleichzeitig gegen dieselbe Datenbank — es muss ein Schema geben, mit dem beide zurechtkommen. (2) Ein **Deployment-Rollback** muss jederzeit möglich sein, ohne die Datenbank anzufassen.<br>Das Verfahren, das das leistet, ist **Expand/Contract**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyEP2gMvtNk0aR2yES1bdw" -->
Sortiere nach Risiko: neue nullable Spalte · `SET NOT NULL` · neuer Index · Spalte umbenennen · neue Tabelle · Constraint lockern.::
**Zerstörungsfrei (additiv, N-1-kompatibel):** neue Tabelle, neue **nullable** Spalte, neuer Index, neuer Default, Constraint **lockern**.<br>**Zerstörerisch:** Tabelle/Spalte löschen, **Spalte umbenennen** (= löschen + anlegen!), Typ verengen, Constraint **verschärfen** (`SET NOT NULL`, neue `CHECK`), Tabelle umbenennen.<br>Alter Code kann die linke Gruppe ignorieren; die rechte bricht ihn sofort.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsi5aFDxRTIrVTugM1PtEO8" -->
Erkläre **Expand / Migrate / Contract** (Parallel Change) am Beispiel einer Spaltenumbenennung.::
**1. Expand:** Migration legt die neue Spalte **nullable** an; der Code schreibt in *beide* Spalten, liest noch aus der alten. Alter Code ignoriert die neue Spalte.<br>**2. Migrate:** Migration kopiert die Daten (`UPDATE … SET neu = alt WHERE neu IS NULL`); der Code liest jetzt aus der neuen Spalte, schreibt weiter beide.<br>**3. Contract:** Der Code nutzt nur noch die neue Spalte; **danach** löscht eine Migration die alte.<br>Kern: Nach **jedem** Schritt ist das System mit alter *und* neuer Codeversion funktionsfähig, und der destruktive Schritt kommt zuletzt und allein.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsj1kwu86kgp76nqM9Edpwz" -->
Warum läuft eine **additive** Migration vor dem Code-Deployment und eine **destruktive** danach?::
**Additiv vorher:** Die Spalte/Tabelle muss existieren, *bevor* der neue Code hineinschreibt — sonst läuft der neue Code ins Leere.<br>**Destruktiv nachher:** Niemand darf das Objekt mehr benutzen, *bevor* es verschwindet — sonst bricht der noch laufende alte Code.<br>Faustregel: **Additives vor dem Deployment, Destruktives nach dem Deployment.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsb0MzUgoW440zSFD5UoK6K" -->
Warum darf eine Migration **niemals die ORM-Modelle importieren**? Nenne zwei Gründe.::
**1. Zeitversatz:** Die Migration beschreibt das Schema von **damals**, das importierte Modell das von **heute**. Entwickelt man `orm.py` weiter, erzeugt eine alte Migration plötzlich SQL mit Spalten, die es zu ihrem Zeitpunkt nicht gab — der Aufbau einer frischen DB aus der Kette scheitert an einer Datei, die niemand angefasst hat.<br>*Migrationen sind eingefrorene Vergangenheit, Modelle lebende Gegenwart.*<br>**2. Performance:** Eine Python-Schleife über hunderttausende Zeilen braucht ein Vielfaches eines einzigen `UPDATE`.<br>Richtig: `op.execute()` mit rohem SQL oder eine im Skript **lokal deklarierte** Minimaltabelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBOPQjo4RiOifgAdhHPKhL" -->
Was ist das **Lock-Queue-Problem** bei Migrationen, und was hilft dagegen?::
Eine DDL-Operation will ein `ACCESS EXCLUSIVE`-Lock (blockiert **jeden** Zugriff, auch lesenden). Läuft gerade ein langer Query auf der Tabelle, **stellt sich die Migration in die Warteschlange** — und PostgreSQL lässt keine neuen Anfragen an ihr vorbei. Ergebnis: Alles auf dieser Tabelle blockiert, wegen einer Migration, die selbst noch **nichts tut** und vielleicht nur 5 ms bräuchte.<br>Gegenmittel: **`SET lock_timeout = '3s'`** vor der Migration — sie scheitert dann schnell und laut, statt eine Blockade aufzubauen. **Schnell scheitern ist hier die freundlichere Variante.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAHhfOU40YDkfkhgFUtL0K" -->
Welche DDL-Operationen sind auf großen Tabellen teuer, welche billig?::
**Billig:** `ADD COLUMN` nullable ohne Default (nur Katalogeintrag); `ADD COLUMN … DEFAULT <konstant>` seit **PostgreSQL 11**.<br>**Teuer:** `ADD COLUMN … DEFAULT now()` (nicht konstant → volles Umschreiben), `ALTER COLUMN … TYPE` (meist Rewrite), `SET NOT NULL` (voller Scan unter `ACCESS EXCLUSIVE`; ab PG 12 vermeidbar über eine bestehende gültige `CHECK`), `CREATE INDEX` (Scan + Schreibsperre), `ADD FOREIGN KEY` (validiert alle Zeilen).<br>**Entschärfungen:** `CREATE INDEX CONCURRENTLY` (ohne Schreibsperre, läuft aber nicht in einer Transaktion) und FK als `NOT VALID` anlegen + später `VALIDATE CONSTRAINT`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXv9PNlhQaYZhiajnxB7Ih" -->
Was ist eine **Baseline**, und was bedeutet „**stempeln**"?::
Der Vorgang, ein **bestehendes** Schema in ein Versionssystem zu überführen. Zwei Schritte:<br>**1.** Eine **Initial-Migration** erzeugen, die das gesamte aktuelle Schema anlegt — der korrekte Aufbauweg für *leere* Datenbanken (CI, neue Kollegen).<br>**2.** Bestehende Datenbanken **stempeln**: die Versionsnummer dieser Migration in die Versionstabelle schreiben, **ohne sie auszuführen** — die Tabellen sind ja schon da.<br>Heißt: Alembic `stamp`, Flyway `baseline`, Liquibase `changelog-sync`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGQdN6XaIpJ40paeV6PtzA" -->
Sind ORM-Modelle und Migrationen zwei konkurrierende „Quellen der Wahrheit"?::
**Nein** — sie beschreiben verschiedene Dinge:<br>**`orm.py`** ist die Quelle der Wahrheit für den **Zustand** (dort wird das Schema entworfen, dort liest die Anwendung es).<br>**Die Migrationen** sind die Quelle der Wahrheit für den **Weg dorthin** — das ausführbare Protokoll.<br>**Autogenerate** ist die Brücke, die beide synchron hält.<br>Das ist der Unterschied zur alten Situation `orm.py` + `schema.sql`: Das waren **zwei Beschreibungen desselben Zustands** — und deshalb ein echtes Duplikat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFIMmjHt3Ry3ZsNRmC2z0Y" -->
Wie deckt man auf, dass ORM-Modelle und Migrationen auseinandergelaufen sind?::
Ein **Drift-Test in CI**: Schema aus der Migrationskette aufbauen, danach **Autogenerate** laufen lassen und prüfen, dass **kein Unterschied** herauskommt.<br>Er wird genau dann rot, wenn jemand `orm.py` ändert und die Migration vergisst (oder umgekehrt).<br>Hintergrund: Wenn Produktion per `upgrade head` und die Tests per `create_all()` aufgebaut werden, erzeugen beide *ein* Schema — aber nicht bewiesenermaßen **dasselbe**. Dann sind die Tests grün und sagen nichts.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAUWsB5qt4tNo7NuiZrglN" -->
Wer sollte Migrationen ausführen — und warum nicht die Anwendung beim Start?::
**Normalfall:** ein eigener **CI/CD-Schritt vor dem Deployment** (bzw. Init-Container/Job im Cluster): ein Lauf, ein Log, ein klarer Fehlerort, stoppbar bevor der Code live geht.<br>**Beim Anwendungsstart ist riskant:** bei *n* Instanzen migrieren *n* Prozesse gleichzeitig (Sperren nötig); der Start dauert plötzlich Minuten und läuft ins Health-Check-Timeout; ein Migrationsfehler wird zur Crash-Loop; und jede Instanz bräuchte DDL-Rechte auf der Datenbank.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssI5zAF14AfYe6clGfUp84" -->
Warum sind Migrations-Tools fester Bestandteil fast aller Web-Projekte? (Kontrollfrage)::
**1. Das Schema ändert sich fortlaufend** — jedes zweite Feature braucht Spalte, Tabelle oder Index.<br>**2. Es gibt immer mehrere Datenbanken** (lokal, CI, Staging, Produktion, ggf. pro Mandant) — dieselbe Änderung muss überall gleich ankommen.<br>**3. Daten kann man nicht neu deployen** — nur transformieren, ausgehend vom tatsächlichen Ist-Zustand.<br>**4. Deployments sind automatisiert** — ein Schritt, der einen Menschen mit psql-Zugang braucht, bricht die Automatisierung an der riskantesten Stelle.<br>**5. Nachvollziehbarkeit** — Migrationen durchlaufen Review, CI und git-Historie; ein `ALTER TABLE` in der Konsole hinterlässt nichts.<br>**6. Kein Stillstand erlaubt** — Rolling Deployments erzwingen abwärtskompatible Schritte (Expand/Contract), was ohne verwaltete Einzelschritte nicht geht.<br>*Gegenprobe:* Bei Wegwerf-Skripten und Analyse-DBs, die jederzeit neu befüllt werden können, braucht man es nicht.
<!-- basicblock-end -->

---

# Alembic

<!-- basicblock-start oid="ObsqnOXTTxiBRwmGK2b5weE7" -->
Was ist **Alembic**, und warum ist es an SQLAlchemy gebunden?::
Das Migrationswerkzeug von **Mike Bayer**, demselben Autor wie SQLAlchemy.<br>Die Bindung ist kein Zufall, sondern die Voraussetzung für Autogenerate: Es vergleicht **zwei Seiten** — das Modell über `Base.metadata` und die echte Datenbank über **SQLAlchemys Reflection-/Dialekt-System**. Beide Seiten sind dadurch **derselbe Objekttyp**. Ohne diese Grundlage müsste man SQL-Text vergleichen, was daran scheitert, dass jede Datenbank ihr DDL anders zurückgibt.<br>Kehrseite: kein SQLAlchemy → kein Alembic (Ausschlusskriterium für Java-/.NET-Teams).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqW1omdemUbkKLyTPrur7V" -->
Ordne die fünf Bausteine eines Migrationswerkzeugs den Alembic-Begriffen zu.::
**Versionstabelle** → `alembic_version` (eine Spalte `version_num`, i. d. R. genau eine Zeile)<br>**Migrationsdateien** → `alembic/versions/*.py` (Python, nicht SQL)<br>**Reihenfolge** → verkettete Revisionen: zufällige Hex-ID + `down_revision` → gerichteter Graph<br>**Runner** → `alembic upgrade head`<br>**Autogenerate** → `alembic revision --autogenerate`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsydtnjxBJlTdL3geJ1ABKl" -->
Welche Dateien legt `alembic init` an, und was ist die Aufgabe jeder?::
`alembic.ini` — Konfiguration (Skriptpfad, Dateinamen, Logging, `sqlalchemy.url`); Alembic sucht sie im aktuellen Verzeichnis, `--config`/`ALEMBIC_CONFIG` überschreiben das.<br>`alembic/env.py` — **das Herzstück**: Python, das bei *jedem* Aufruf läuft und die Verbindung herstellt.<br>`alembic/script.py.mako` — Mako-Vorlage für neue Migrationsdateien.<br>`alembic/versions/` — die Migrationen selbst; **keine aufsteigenden Zahlen**, sondern Teil-GUIDs, weil die Reihenfolge *in* den Dateien steht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsoctNMgfGKqkvRWTm9yddk" -->
Welche vier `alembic init`-Templates gibt es?::
`generic` — Standard: eine Datenbank, synchron.<br>`pyproject` — **seit 1.16.0**: PEP-621-konforme Konfiguration unter `[tool.alembic]` in `pyproject.toml` (Listen sind dort echte Listen). Erzeugt zusätzlich weiterhin eine `alembic.ini` und hängt an eine bestehende `pyproject.toml` an.<br>`async` — für async-DBAPI (z. B. `asyncpg`).<br>`multidb` — mehrere Datenbanken, mit `upgrade_engine1()`/`upgrade_engine2()`.<br>Anzeigen mit `alembic list_templates`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJiSrnia8mYg2hwVuopc9a" -->
Was ist `env.py`, und welche zwei Betriebsarten enthält es?::
Kein Konfigurationsfile, sondern ein **Skript, das bei jedem einzelnen `alembic`-Aufruf ausgeführt wird** — es baut die Verbindung auf und startet den Lauf. Deshalb liegt es in deinem Projekt, nicht in der Bibliothek.<br>`run_migrations_online()` — Normalfall: verbinden, migrieren, `alembic_version` fortschreiben.<br>`run_migrations_offline()` — bei `--sql`: **keine Verbindung**, es wird nur SQL ausgegeben (inkl. der `UPDATE alembic_version`-Anweisungen), das jemand anderes ausführt. Üblich in regulierten Umgebungen, wo ein DBA die Produktion bedient.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaaEJQmO6iyZUBgKAusM8W" -->
Was bewirkt `target_metadata` in `env.py` — und was passiert, wenn man es vergisst?::
Es ist das `MetaData`-Objekt, das dein Schema kennt (bei uns aus `Base` in `persistence/orm.py`), und damit die **Modellseite** des Autogenerate-Vergleichs.<br>Bleibt es auf dem Vorgabewert `None`, erzeugt `--autogenerate` **leere Migrationen** — bzw. schlägt vor, *alle* Tabellen zu löschen, weil Alembic das Modell für leer hält.<br>Der mit Abstand häufigste Anfängerfehler mit Alembic.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAGzF3gWmpIS7QSwkXqT6n" -->
Warum gehört die Datenbank-URL nicht in `alembic.ini`, sondern nach `env.py`?::
**1.** `alembic.ini` wird **eingecheckt** — ein Passwort gehört nicht in eine versionierte Datei. (In unserem Projekt liegen die Zugangsdaten in `.env` und werden über `shared/config.py` mit `pydantic-settings` gelesen.)<br>**2.** Sonst hätte man **zwei Quellen für dieselbe Information** — genau der Fehler, den der Schema-Umbau in 1.5 beseitigt hat.<br>`env.py` ist dafür der architektonisch richtige Ort, weil es — wie `main.py` und `bootstrap.py` — eine **Composition Root** ist und alle Schichten kennen darf.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsx9mOU2JAq4GlcD6hoUcno" -->
Nenne die wichtigsten Alembic-Kommandos.::
`alembic revision -m "..."` — leere Migration · `--autogenerate` — aus Modell/DB-Diff<br>`alembic upgrade head` / `+1` · `downgrade -1` / `base`<br>`alembic current` — welche Version hat diese DB?<br>`alembic history -v` · `heads` · `branches` · `show <rev>`<br>`alembic merge <rev1> <rev2>` — zwei Köpfe zusammenführen<br>**`alembic stamp <rev>`** — Version setzen **ohne auszuführen** (Baseline!)<br>`alembic check` — CI-Prüfung · `upgrade head --sql` — Offline-Modus<br>Revisionen dürfen per **Präfix** angesprochen werden (`upgrade a3f2`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBVFIdHfl7xiQSeMO2FRDc" -->
Wann benutzt man `alembic revision` **ohne** `--autogenerate`?::
Für alles, was ein Diff prinzipiell **nicht sehen kann**: Umbenennungen von Tabellen/Spalten, Daten-Migrationen und Änderungen am **Ausdruck** einer CHECK-Constraint.<br>Ohne `--autogenerate` entsteht eine leere Datei mit `pass` in beiden Funktionen.<br>Nebenbei: `--autogenerate` setzt eine **laufende, erreichbare Datenbank** voraus, weil dagegen verglichen wird — die leere Variante nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEh9VwJ2hGfsVo3hcR3F1J" -->
Welche vier Modulvariablen ordnen eine Alembic-Revision ein?::
`revision` — die eigene ID (zufällig erzeugt, **nicht** aufsteigend)<br>`down_revision` — der Vorgänger; `None` = allererste Migration<br>`branch_labels` — optionaler Name für einen Zweig (mehrere unabhängige Ketten)<br>`depends_on` — „braucht jene Revision, auch wenn sie nicht der Vorgänger ist" — Abhängigkeit *quer* zum Graphen<br>Bonus: Der Kopf-Docstring ist keine Deko — `alembic history` zeigt seine erste Zeile als Beschreibung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBieF2bIjnpCmZLR9AQ4Yi" -->
Was ist das `op`-Objekt, und welche zwei Muster braucht man ständig?::
Alembics datenbankunabhängiges **DDL-Vokabular**, das der SQLAlchemy-Dialekt in passendes SQL übersetzt: `create_table`, `add_column`, `alter_column`, `create_index`, `create_check_constraint`, `drop_constraint`, `rename_table`, `bulk_insert`, `batch_alter_table`.<br>**Muster 1 – rohes SQL** (Datenmigration, ohne ORM-Import):<br>`op.execute("UPDATE bookings SET channel = 'unknown' WHERE channel IS NULL")`<br>**Muster 2 – etwas lesen:**<br>`conn = op.get_bind()` → `conn.execute(sa.text("SELECT count(*) …")).scalar_one()`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQf8lhWXxpR1qbQptHwQbF" -->
Was erkennt Alembics Autogenerate **standardmäßig**?::
- Tabellen hinzugefügt/entfernt<br>- Spalten hinzugefügt/entfernt<br>- Änderung der **Nullbarkeit**<br>- Grundlegende Änderungen an **Indizes** und **explizit benannten Unique-Constraints**<br>- Grundlegende Änderungen an **Foreign Keys**<br>- **Benannte CHECK-Constraints** (hinzugefügt/entfernt) — erst **seit Version 1.19.0**, und **nur namensbasiert**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3rR1luj7sCvkRpw4MFo6u" -->
Was erkennt Alembics Autogenerate **nicht**?::
- **Tabellen-Umbenennungen** → kommen als Drop + Create heraus<br>- **Spalten-Umbenennungen** → als Drop + Add, laut Doku „which is not at all the same as a name change"<br>- **Anonyme (unbenannte) Constraints** → deshalb die Naming Convention<br>- Spezielle Typen wie `Enum` auf Backends ohne echtes ENUM<br>*Noch nicht implementiert:* freistehende PRIMARY-KEY- und EXCLUDE-Constraints sowie **Sequences**.<br>Grundsatz: Der Generator sieht **Zustände, keine Absichten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGLFut6Pck5DExB1SZ8JNY" -->
`compare_type` und `compare_server_default` — welcher ist standardmäßig an, welcher aus?::
**`compare_type` ist AN.** Erkennt Typänderungen: große Unterschiede zuverlässig (`Numeric` ↔ `String`), vergleicht auch Länge und Präzision. Argumente, die nur **eine** Seite kennt, werden nicht verglichen.<br>**`compare_server_default` ist AUS.** Muss aktiv eingeschaltet werden; funktioniert für einfache Fälle, und der PostgreSQL-Dialekt lässt die beiden Werte zum Vergleich sogar von der Datenbank auswerten. Bewusst standardmäßig aus, damit man es erst am eigenen Schema erprobt.<br>Praxisfolge: Solange es aus ist, wird eine Änderung an `server_default` **nie** erkannt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6oPxFX1yceBhMG80Zgd9U" -->
Warum bemerkt Autogenerate es nicht, wenn du einen Wert zu einer CHECK-Constraint hinzufügst?::
Weil die Erkennung (neu seit 1.19.0) **rein namensbasiert** ist. Doku-Zitat: Zwei Constraints mit demselben Namen gelten „always presumed to be equivalent, **regardless of any difference in their expression text**" — SQL-Ausdrücke lassen sich über Dialekte hinweg nicht zuverlässig vergleichen, weil sie den DDL-Text normalisieren.<br>Ändert sich also der **Ausdruck** bei gleichem **Namen**, passiert nichts: keine Migration, keine Warnung, `alembic check` bleibt grün.<br>**Regel:** Ausdrucksänderung an einer CHECK-Constraint → Migration **von Hand** (`drop_constraint` + `create_check_constraint`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2t7LB6HEDg30i3rxWhtW7" -->
Wozu dient eine **Naming Convention** auf dem `MetaData`, und was hat sie mit `op.drop_constraint()` zu tun?::
`op.drop_constraint()` braucht einen **Namen** — und ungenannte Constraints benennt die Datenbank selbst, jede anders: PostgreSQL `user_order_user_account_id_fkey`, Oracle `SYS_C0029334` („entirely unpredictable alphanumeric codes"). Man müsste den Namen pro Umgebung nachschlagen.<br>Eine Convention (`MetaData(naming_convention={...})` mit `ix/uq/ck/fk/pk`) macht die Namen **deterministisch aus dem Modell ableitbar** und in jeder Umgebung gleich.<br>Zusatznutzen: Anonyme Constraints stehen ausdrücklich auf der „wird nicht erkannt"-Liste von Autogenerate, und die CHECK-Erkennung ist rein namensbasiert — ohne Convention funktioniert beides nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKJ8mVTdMEtlL93wmgvXpX" -->
Wie äußert sich ein Branch-Konflikt in Alembic, und wie löst man ihn?::
```
$ alembic upgrade head
FAILED: Multiple head revisions are present for given argument 'head'
$ alembic heads
b2c3d4e5f6a7 (head)
f9d1a2b3c4d5 (head)
$ alembic merge -m "..." b2c3 f9d1
```
Die erzeugte **Merge-Migration** hat **zwei Vorgänger** (`down_revision = ("b2c3…", "f9d1…")`) und meist ein leeres `upgrade()`. Sie führt den Graphen zusammen; ob sich die Zweige *fachlich* vertragen, prüft ein Mensch.<br>Verwandt: `--splice` (bewusst hinter eine Nicht-Kopf-Revision hängen) und `depends_on`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsc2C86RGBigYjLqt5D6hq1" -->
Was tut `alembic check` — und was beweist es **nicht**?::
Es durchläuft **denselben Vergleich wie `revision --autogenerate`**, schreibt aber keine Datei, sondern liefert einen Exit-Code plus Meldung („No new upgrade operations detected."). Damit ist der Drift-Test aus Stichwort 1 fertig eingebaut (seit 1.9.0) und gehört in die CI neben `ruff`, `mypy` und `pytest`.<br>**Es beweist nicht**, dass Modelle und Migrationen übereinstimmen — nur, dass sie sich **in dem, was Autogenerate sehen kann**, nicht unterscheiden. Alle Grenzen aus Abschnitt 7 gelten unverändert (Umbenennungen, CHECK-Ausdrücke, `server_default` bei ausgeschaltetem Vergleich).<br>Außerdem: `check` braucht eine **erreichbare Datenbank**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYxNbsrkHYD2AOEdWMOg2y" -->
Wozu gibt es `op.batch_alter_table()`?::
Für **SQLite**, das fast kein `ALTER TABLE` beherrscht (keine Spalten löschen, keine Typen ändern, keine Constraints hinzufügen). Alembic behilft sich mit „Move-and-Copy": neue Tabelle mit Zielstruktur anlegen, Daten kopieren, alte löschen, neue umbenennen.<br>Mit PostgreSQL **nicht nötig**. Und das ist zugleich ein Argument dafür, **nicht** auf SQLite zu testen: Ein SQLite-Testschema ist nicht dasselbe Schema wie in Produktion (keine `Identity`, andere Typen, andere Constraint-Semantik).
<!-- basicblock-end -->

---

# Liquibase

<!-- basicblock-start oid="Obs9ZlqQMnYcPudY5fLDkctJ" -->
Was ist Liquibase' **Kernidee**, und wie unterscheidet sie sich von Alembic?::
Eine Migration wird **abstrakt beschrieben** („lege Spalte `channel` vom Typ Text in `bookings` an") statt als SQL oder Code programmiert. Liquibase übersetzt die Beschreibung in den SQL-Dialekt der jeweiligen Zieldatenbank — **dieselbe Changelog-Datei läuft gegen PostgreSQL, Oracle, MS SQL, MySQL** usw.<br>Ziel ist **Unabhängigkeit von Programmiersprache und Datenbank** (Java-Werkzeug, braucht nur eine JVM).<br>Merksatz: **Alembic ist ein Entwicklerwerkzeug, Liquibase ein Werkzeug für den Änderungsprozess einer Organisation.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIxbE8BT7AHzIowwRA4Cvi" -->
Welche vier Changelog-Formate kennt Liquibase — und was gibt man bei Formatted SQL auf?::
**XML** (kanonisch, mit XSD-Validierung), **YAML**, **JSON** und **Formatted SQL** (SQL mit Metadaten in Kommentaren: `--changeset nick:7`, `--rollback …`).<br>Inhaltlich sind sie gleichwertig — **aber wer Formatted SQL wählt, gibt die Datenbankunabhängigkeit auf**, also den Hauptvorteil des Werkzeugs. Man behält Historie, Prüfsummen, Contexts und Rollback-Verwaltung. Legitime Entscheidung, sollte nur bewusst getroffen werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjbDfC94Epgdg5NZz2MMJi" -->
Woraus besteht die **Identität eines Changesets** — und was passiert beim Umbenennen der Datei?::
Aus dem **Tripel `id` + `author` + `filename`** (nicht aus der `id` allein; `id` ist ein *String* und muss nur innerhalb einer Datei eindeutig sein).<br>Folge: **Verschiebt oder benennt man eine Changelog-Datei um, ändert sich die Identität aller darin enthaltenen Changesets** — Liquibase hält sie für neu und will sie erneut ausführen. Gegenmittel: das Attribut `logicalFilePath`.<br>Kontrast: Bei Alembic ist die Identität die zufällige Revisions-ID, die beim Verschieben unverändert bleibt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsewZ5lUJMRTqB5KZC7QAEG" -->
Wie bestimmt Liquibase die **Reihenfolge** der Migrationen — und was folgt daraus für Branches?::
Über die **Reihenfolge der `<include>`-Einträge im Master-Changelog**. Es gibt **keinen Revisionsgraphen** wie bei Alembic.<br>Folge: Zwei Feature-Branches, die je ein `<include>` an derselben Stelle einfügen, erzeugen einen ganz gewöhnlichen **git-Merge-Konflikt in der Master-Datei** — sichtbar und mit vertrauten Mitteln lösbar, aber ohne die Fähigkeit, echte Parallelität abzubilden.<br>Kurz: **Liquibase verlagert das Merge-Problem von der Datenbank in die Versionskontrolle.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsq1wKkMsqD4AhLGIxNr2iM" -->
Was steht in `DATABASECHANGELOG`, und warum ist das mehr als Alembics `alembic_version`?::
Alembic: **1 Spalte, 1 Zeile** („wo stehe ich?"). Liquibase: **14 Spalten, eine Zeile je ausgeführtem Changeset** — ein **Audit-Log**.<br>Spalten: `ID`/`AUTHOR`/`FILENAME` (Identität), `DATEEXECUTED`, `ORDEREXECUTED`, `EXECTYPE` (EXECUTED/FAILED/SKIPPED/RERAN/MARK_RAN), **`MD5SUM`**, `DESCRIPTION`, `COMMENTS`, `TAG`, `LIQUIBASE` (Version), `CONTEXTS`, `LABELS`, `DEPLOYMENT_ID` (klammert alle Changesets *eines* Laufs).<br>Damit beantwortbar: „Was ist hier je passiert, wann, durch wen, in welchem Deployment?" — in regulierten Branchen der Kaufgrund.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTEiE5a7pa3Kva5X7AA7RL" -->
Wozu dient `MD5SUM` in `DATABASECHANGELOG`?::
Liquibase speichert eine **Prüfsumme jedes ausgeführten Changesets** und vergleicht sie bei jedem Lauf. Wurde ein bereits gelaufenes Changeset nachträglich verändert, **bricht Liquibase mit einem Validierungsfehler ab**, statt den Unterschied stillschweigend zu übergehen.<br>Damit wird aus der Regel „ausgelieferte Migrationen sind unveränderlich" eine **Kontrolle** statt einer Konvention — bei Alembic ist sie reine Disziplin.<br>Für legitime Ausnahmen: `runOnChange="true"` bzw. `clear-checksums`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUVZSXs2TzVfsNOv9l8U1f" -->
Wofür gibt es `DATABASECHANGELOGLOCK` — und was ist die Kehrseite?::
Eine **explizite Sperrtabelle**, die verhindert, dass zwei Läufe gleichzeitig migrieren (z. B. wenn mehrere Container gleichzeitig starten).<br>Alembic hat so etwas **nicht eingebaut** — dort löst man das über Datenbank-Advisory-Locks oder dadurch, dass ohnehin nur ein CI-Schritt migriert.<br>Kehrseite: **Bricht ein Lauf hart ab, bleibt die Sperre stehen** und muss mit `release-locks` gelöst werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDRwtC3QIqaSGqqsWTCDGF" -->
Warum kann Liquibase viele Rollbacks **automatisch ableiten**, Alembic aber nicht?::
Weil ein Changeset die Änderung **beschreibt** statt sie zu programmieren. Aus einer Beschreibung lässt sich die Umkehrung errechnen: `createTable` → `dropTable`, `addColumn` → `dropColumn`, `renameColumn` → zurückbenennen.<br>Für alles, was nicht ableitbar ist (rohes SQL, Datenänderungen), schreibt man einen expliziten `<rollback>`-Block.<br>Bei Alembic ist `downgrade()` **Python-Code und immer von Hand zu schreiben** — es gibt nichts, woraus man die Umkehrung ableiten könnte.<br>**Dieselbe Grenze gilt trotzdem:** Auch Liquibase holt nach `dropColumn` die Werte nicht zurück.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsjijmzaq1ZFOmq6vztU4vK" -->
Nenne Liquibase' Rollback-Kommandos — und das eine, das eine Schwäche von Alembic direkt adressiert.::
`rollback <tag>` (bis zu einem benannten Punkt) · `rollback-count <n>` · `rollback-to-date <datum>` · `rollback-sql …` (nur anzeigen) · `future-rollback-sql` (wie sähe das Rollback der noch ausstehenden Änderungen aus?).<br>**`update-testing-rollback`** — anwenden → zurückrollen → erneut anwenden. Es ist die Antwort auf den Einwand „`downgrade`-Funktionen werden fast nie getestet": Dieses Kommando testet sie automatisch.<br>Dazu **`tag`** als benannter Wiederherstellungspunkt (`v2.4-release`) — ein Konzept, das Alembic fehlt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNmRuiX6gnhMlf6L7Oz5d9" -->
Was sind `preConditions`, `contexts` und `labels`?::
**`preConditions`** — Bedingungen, die vor dem Lauf geprüft werden (`columnExists`, `tableExists`, `sqlCheck`), mit `onFail`: `HALT` (abbrechen), `CONTINUE` (überspringen, später erneut versuchen), `MARK_RAN` (als erledigt eintragen ohne auszuführen), `WARN`. Für uneinheitlich gewachsene Datenbanken.<br>**`contexts`** — Changesets nur in bestimmten Umgebungen ausführen (`context="dev,test"`, aufgerufen mit `--contexts=dev`). Löst z. B. die Seed-Daten-Frage deklarativ.<br>**`labels`** — zweite, unabhängige Markierungsachse, beim Aufruf mit booleschen Ausdrücken filterbar.<br>Faustregel: **Contexts beschreiben *wo*, Labels *was*.** Für all das würde man in Alembic Python schreiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLYPWOxNH7rLXEAvcSEZua" -->
Ordne zu: `liquibase update`, `changelog-sync`, `diff-changelog`, `status`, `history` — was ist das Alembic-Gegenstück?::
`update` → `alembic upgrade head`<br>**`changelog-sync` → `alembic stamp`** (der Baseline-Befehl: eintragen ohne ausführen)<br>`diff-changelog` → `alembic revision --autogenerate`<br>`status` → grob `alembic current`, zeigt aber die **ausstehenden** Changesets<br>`history` → `alembic history`, liest aber aus der **Datenbank** statt aus den Dateien<br>Ohne Gegenstück: `tag`, `validate`, `release-locks`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRK1arMIC5LYxt8T08KR5B" -->
Warum vergleicht `liquibase diff` **zwei Datenbanken**, während `alembic --autogenerate` Modell gegen Datenbank vergleicht?::
Weil es bei Liquibase **kein ORM-Modell gibt**, das als Wahrheit dienen könnte — Liquibase ist sprachunabhängig und kennt deine Klassen nicht. Also vergleicht es eine **Referenz-Datenbank** mit der Ziel-Datenbank.<br>Wer das Alembic-Muster will („mein Code beschreibt das Soll"), braucht eine Referenzdatenbank, die diesen Sollzustand verkörpert.<br>Genau deshalb passt Alembic besser zu einem Projekt, in dem — wie hier — **die SQLAlchemy-Modelle das Schema besitzen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsB2g0AsXBUUjXoyyNiQRcb" -->
Was hat sich mit **Liquibase 5.0** geändert?::
**Lizenz:** von Apache 2.0 auf **FSL-1.1-ALv2** (Functional Source License) — „Fair Source", **kein OSI-Open-Source** mehr. Nutzen/Ändern erlaubt, ein konkurrierendes kommerzielles Angebot nicht; **konvertiert nach zwei Jahren automatisch zu Apache 2.0**.<br>**Produktschnitt:** Über GitHub/Docker/Maven gibt es nur noch **Liquibase Community**; das kommerzielle Produkt heißt **Liquibase Secure** (früher „Pro").<br>**Schlanke Auslieferung:** Community 5.0 kommt **ohne Extensions und Treiber** — Nachinstallation über den **Liquibase Package Manager** (`liquibase lpm`).<br>**Java 17 ist Mindestversion** (Java 8/11 fallengelassen).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs52pfGyCaswL3AB0dep6pz" -->
Was spricht **für Alembic**, was **für Liquibase**? (Kontrollfrage, Kurzfassung)::
**Pro Alembic:** lebt in derselben Sprache wie das Projekt (kein zweites Ökosystem) · Autogenerate vergleicht gegen das **Modell** · Revisionsgraph bildet parallele Entwicklung ehrlich ab · Migrationen sind Code (Schleifen, Debugging) · MIT-Lizenz.<br>**Pro Liquibase:** sprach- **und** datenbankunabhängig · **Prüfsummen erzwingen** Unveränderlichkeit · Rollback ernstgenommen (ableitbar, `tag`, `update-testing-rollback`) · `DATABASECHANGELOG` als **Audit-Log** · `contexts`/`labels`/`preConditions` deklarativ · explizite Sperre gegen Parallelläufe.<br>**Contra Alembic:** nur Python · kein Schutz vor nachträglicher Änderung · `downgrade()` immer von Hand · keine Audit-Historie.<br>**Contra Liquibase:** JVM + JDBC-Treiber + `lpm` · XML/YAML statt Code · kein Revisionsgraph · kein Modell-Diff · Lizenz seit 5.0 nicht mehr OSI-open-source.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIdGvjUEZeRgWruQv2o7J2" -->
Welches Werkzeug passt besser zu unserem Pricing-Projekt — und wann würde die Antwort umschlagen?::
**Alembic**, aus vier Gründen: (1) **SQLAlchemy ist bereits die Quelle der Wahrheit** fürs Schema — genau dafür ist Autogenerate gebaut; Liquibase bräuchte eine Referenzdatenbank als Umweg. (2) Der Stack ist nach dem uv-Umbau **eine einheitliche Werkzeugkette**; JVM, JDBC-Treiber und `lpm` stünden außerhalb von `uv.lock`. (3) **Eine Datenbank, ein Team, eine Sprache** — Liquibase' Hauptvorteile zahlen sich auf null aus. (4) Der Leitfaden verlangt es.<br>**Umschlagen würde es**, wenn: ein **Java-Service** dieselbe DB mitbenutzt · die Anwendung bei Kunden auf **verschiedenen Datenbanksystemen** läuft · ein **Auditor** Nachweise über Produktionsänderungen braucht · **DBAs** die Änderungen prüfen und ein Format brauchen, das nicht Python ist.
<!-- basicblock-end -->
