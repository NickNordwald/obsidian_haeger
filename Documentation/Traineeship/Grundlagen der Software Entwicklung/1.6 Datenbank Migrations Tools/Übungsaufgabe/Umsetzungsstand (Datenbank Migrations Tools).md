---
tags:
  - documentation
type: notiz
date: 2026-08-15
---

Abgeschlossener Umsetzungsstand der **Migrations-Iteration** der Pricing-Anwendung (Aufgabe: [[1.6 Datenbank Migrations Tools - Leitfaden#Aufgabe|1.6 Leitfaden – Aufgabe]]). Baut auf [[Umsetzungsstand (REST APIs & Layered Architecture)|Kapitel 1.5]] auf.

Theorie: [[1.6 Datenbank Migrations Tools - Vertiefung|Vertiefung]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Anki Karteikarten|Anki Karteikarten]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Q&A/Weiterführende Stichpunkte.md|Q&A-Register]] · Plan: [[Alembic Plan & Syntax-Tutorial|Alembic Plan & Syntax-Tutorial]] (13 Blöcke, alle abgearbeitet)

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/DB_Migration_Tools/` (Kopie des REST-API-Standes) · Abschluss-Commit `8ded757`

## Endstand

Das bisher per `create_all()` erzeugte Schema — 9 Tabellen aus `orm.py`, mit Constraints, Indizes, `Identity`, einer `Computed`-Spalte und Naming Convention — läuft jetzt vollständig unter **Alembic 1.19.1**. Eine Baseline-Migration deckt alle 9 Tabellen ab, erzeugt per `--autogenerate`; `pricing-initdb` und `schema_admin.py` wurden ersatzlos entfernt.

Der Onboarding-Ablauf ändert dadurch genau eine Zeile:

```bash
docker compose up -d
uv run alembic upgrade head        # ersetzt "uv run pricing-initdb"
uv run pricing-apikey test
uv run uvicorn pricing.presentation.api.app:app --reload
```

**Verifiziert:** `alembic current` → `7f94db1e1511 (head)` · `alembic check` → *No new upgrade operations detected* · ruff, `mypy --strict` und 123 Tests grün · Arbeitsbaum sauber (7 Commits auf `feature/alembic`).

Zusätzlich zu Übungszwecken durchgespielt und danach vollständig zurückgebaut: zwei Probe-Migrationen (Spalte hinzufügen, CHECK-Constraint ändern) — jeweils Upgrade **und** Downgrade.

## Wichtigste Entscheidungen

**`pricing-initdb`/`create_all()` ersatzlos entfernt, nicht umgebaut.** Zwei Wege, dasselbe Schema zu erzeugen, wären genau die Drift, die dieses Kapitel beseitigen soll. Ein halb migriertes Projekt ist schlechter als ein unmigriertes.

**Baseline per `--autogenerate` statt handgeschrieben.** Gegen eine leere Datenbank erzeugt — und sie saß im ersten Anlauf vollständig, inklusive aller fünf vorab als riskant markierten Stellen (`Computed`, `Identity`, `server_default`, CHECK-Constraints, Naming Convention). Bewiesen wurde das durch `alembic check`, nicht durch Lesen.

**Alembic-Untergrenze `>=1.19.1`.** Die Erkennung *benannter* CHECK-Constraints kam erst in 1.19.0, und 1.19.1 behob vier Tage später einen Fehler darin. Bei fünf `CheckConstraint`s im Schema will man die korrigierte Fassung — eine Versionsuntergrenze ist hier eine inhaltliche Entscheidung, keine Formalie.

**Alembic als Hauptabhängigkeit, nicht als `dev`.** Es gehört zum Onboarding-Ablauf und wird von jedem gebraucht, der die Anwendung startet — dieselbe Begründung wie bei FastAPI in 1.5.

**`generic`-Template statt `pyproject`-Template.** Das pyproject-Template hätte trotzdem eine `alembic.ini` erzeugt, deren Hauptinhalt (die URL) hier ohnehin nach `env.py` wandert. Der Gewinn wäre rein kosmetisch gewesen.

**Verbindung kommt aus `shared/config.py`, nicht aus `alembic.ini`.** `env.py` ist — wie `main.py` und `bootstrap.py` — eine **Composition Root** und darf deshalb alle Schichten kennen. Vor allem gehört das Passwort nicht ins Repository. Benutzt wird `get_engine()` direkt statt des `@lru_cache`-gecachten `bootstrap.engine()`: Für einen kurzlebigen CLI-Prozess ist Pooling irrelevant, und die Migration bleibt vom Anwendungs-Bootstrap unabhängig.

**Werkzeugwahl Alembic statt Liquibase bewusst bestätigt**, nicht bloß vom Leitfaden übernommen: SQLAlchemy ist bereits die Quelle der Wahrheit fürs Schema, der Stack ist seit dem uv-Umbau eine einzige Werkzeugkette, und Liquibase' Hauptvorteile (Sprach- und DB-Unabhängigkeit) zahlen sich bei einer Datenbank, einem Team und einer Sprache nicht aus.

**Testdatenbank und Integrationstests bewusst zurückgestellt.** Die 123 bestehenden Tests laufen weiter ohne Datenbank. Preis dieser Entscheidung: Die Migrationskette selbst bleibt ungetestet — siehe [[Aufgehobene offene Punkte (Kapitel 1)|Aufgehobene offene Punkte]].

## Gelernte Lektionen

- **`create_all()` reichte bisher nur durch Glück.** Alle bisherigen Schemaänderungen waren *neue Tabellen* — der einzige Fall, den `create_all()` beherrscht — und alle Daten kamen aus CSVs, waren also wegwerfbar. Seit Kapitel 1.5 gibt es aber Daten, die nur in der Datenbank existieren (`api_keys`, `pricing_runs`, `ingestion_runs`). Die erste Spaltenänderung wäre lautlos gescheitert.
- **CHECK-Constraints erkennt Alembic nur namensbasiert.** Zwei Constraints mit demselben Namen gelten als gleich, unabhängig vom Ausdruck. Ein zusätzlicher erlaubter Status ändert den Ausdruck, nicht den Namen — Autogenerate erzeugt dafür **nichts**, ohne Fehler, bei grünem `alembic check`. **Regel: Ausdrucksänderungen an CHECK-Constraints immer von Hand migrieren.**
- **`op.f()` ist keine Kosmetik.** Die `ck`-Naming-Convention enthält den Platzhalter `constraint_name` und präfixiert deshalb *jeden* übergebenen Namen erneut — ein bereits vollständiger Name wird verdoppelt (`ck_bookings_ck_bookings_status_valid`). `op.f()` markiert den Namen als endgültig. Das erklärt auch, warum in der Baseline Constraint-Namen in `op.f()` stehen und Index-Namen nicht.
- **Transaktionales DDL einmal real erlebt.** Eine mitten im `upgrade()` gescheiterte Migration wurde von PostgreSQL komplett zurückgerollt, inklusive `alembic_version` — kein Halbzustand, kein manuelles Aufräumen. In MySQL wäre an derselben Stelle Handarbeit nötig gewesen.
- **`downgrade()` ist strukturell ehrlich, datenseitig nicht immer.** Der Downgrade der CHECK-Migration scheitert, sobald eine Zeile den neu erlaubten Wert trägt. Auf einer leeren Testdatenbank fällt das nie auf — genau deshalb ist „der Downgrade läuft" ein schwächeres Versprechen, als es klingt.
- **Ein einzelnes fehlendes Zeichen kann Konfiguration lautlos wirkungslos machen.** `hooks = ruff` blieb in `alembic.ini` auskommentiert, während die zugehörigen `ruff.*`-Optionen aktiv waren — Alembic ignoriert Konfiguration für nicht in `hooks` genannte Hooks kommentarlos. Derselbe Fehlertyp wie `[dependency-group]` statt `[dependency-groups]` in 1.5.
- **Ungepushte Übungsmigrationen darf man löschen, ausgelieferte nicht.** Der Unterschied liegt am Wort „ausgeliefert", nicht an „existiert".
- **Die zweite Datenbank war unnötig.** Docker Compose leitet den Projektnamen aus dem Verzeichnisnamen ab — der neue Projektordner bekam ohnehin ein leeres Volume, `upgrade head` reichte. Ein `alembic stamp` gegen ein bestehendes Schema wurde dadurch nie gebraucht.
