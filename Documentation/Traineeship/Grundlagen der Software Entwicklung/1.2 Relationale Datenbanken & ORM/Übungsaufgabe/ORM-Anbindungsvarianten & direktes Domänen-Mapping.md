---
tags:
  - documentation
type: notiz
date: 2026-07-23
---

Überblick über die **vier Varianten**, wie ORM, Datenbank-Schema und Domäne zusammenhängen können — und die vertiefte Antwort auf die Frage: *Kann man die Domänenklassen (`Booking` etc.) direkt zu gemappten Klassen machen, was bringt das, und macht man das?* Kontext: [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand]] · Spike: [[ORM besitzt das Schema (Spike) - Tutorial|ORM besitzt das Schema (Spike)]].

## Die vier Varianten im Überblick

| # | Variante | Schema-Quelle | Domäne ↔ ORM | Im Projekt |
|---|---|---|---|---|
| **1** | Mapping auf **bestehendes** Schema | `sql/schema.sql` (handgeschrieben) | **getrennt**, ORM mappt eine *Teilmenge* der Spalten | Teil 2 / `main` |
| **2** | ORM besitzt das **volle** Schema | ORM-Modelle (alle Spalten + alle DDL-Features) | **getrennt** | Spike v1 (verworfen) |
| **3** | ORM besitzt ein **schlankes** Schema | ORM-Modelle (nur was gebraucht wird) | **getrennt**, aber Konvertierung ist 1:1-trivial | Spike (aktuell) |
| **4** | **Domäne = ORM** (kein Unterschied) | ORM-Modelle | **identisch** — die Domänenklasse *ist* die Zeile | nur diskutiert |

**Gemeinsamkeit von 1–3:** Domänenmodell und Persistenzmodell sind **getrennte** Klassen (SQLAlchemys „Data Mapper"-Philosophie). Der Adapter konvertiert dazwischen. Nur **Variante 4** verschmilzt beide.

Variante 3 vs. 4 ist die eigentliche Spannung: In Variante 3 sind `orm.Booking` und `domain.Booking` fast identisch — die „Duplikation" fühlt sich verschwenderisch an, und genau das macht Variante 4 verlockend.

---

## Variante 4 — die Domänenklassen direkt mappen

### Geht das? Ja, auf zwei Wegen
- **Declarative:** die Domänenklassen erben von `Base` und nutzen `mapped_column` → `domain.py` **importiert SQLAlchemy**. Maximale Verschmelzung, aber der Kern kennt jetzt die DB.
- **Imperative (classical) mapping:** `registry.map_imperatively(Booking, bookings_table)` — die `Table` **und** das Mapping liegen im Adapter, die Domänenklasse bleibt eine *pure* Python-Klasse (kein SQLAlchemy-Import). Der elegante Weg, Variante 4 zu bauen, *ohne* die Import-Abhängigkeit umzudrehen.

### Vorteile
- **Keine Duplikation** — eine Klasse statt `orm.Booking` + `domain.Booking`.
- **Keine Konvertierungs-Boilerplate** — `session.scalars(select(Booking))` liefert **direkt** `domain.Booking`; `session.add(booking)` persistiert es. `DataSource`/`Writer` schrumpfen weiter.
- **„Objekt = Zeile" wörtlich** — das klassische Active-Record-Gefühl.

### Nachteile
- **Kopplung Domäne ↔ Persistenz.** Bei declarative importiert der innerste Layer die DB (bricht die Dependency-Rule). Auch bei imperative mapping wird die Klasse zur **Laufzeit instrumentiert**: sie erbt Session-Semantik (Attach/Detach), lazy-loading-Deskriptoren, `DetachedInstanceError`, Attribut-Reload nach `commit`/`expire` — Persistenz-Verhalten leckt in Objekte, die eigentlich reine Werte sein sollten.
- **`frozen`/Validierung leidet.** `frozen` dataclasses lassen sich nicht laden (das ORM *setzt* Attribute); `__init__`/`__post_init__` werden beim Laden **übersprungen** (`__new__`) → die Domänen-Invarianten (`nights >= 1` usw.) greifen nur noch bei „normaler" Konstruktion, nicht beim DB-Laden.
- **Persistenz-Konzepte lecken in die Domäne.** Die Klasse braucht plötzlich einen **Surrogat-`id`**/Primärschlüssel und Spaltentypen — Dinge, die das Domänenmodell gar nicht will (genau der Punkt aus der `id`-Frage).
- **Testbarkeit/Portabilität.** Die Domäne kann ohne SQLAlchemy nicht mehr existieren/getestet werden; reine In-Memory-Tests werden schwerer.
- **Single Responsibility.** Die Klasse dient zwei Herren (Fachverhalten + O/R-Mapping) — jede Schema-Änderung berührt die Domänenklasse.

### Wird das gemacht?
- **In der Praxis sehr häufig, ja.** Active-Record-Frameworks (**Django ORM**, Rails) machen genau das: das Model *ist* die Persistenz. Pragmatisch, wenig Boilerplate, ideal für CRUD-/Web-Anwendungen.
- **SQLAlchemy ist bewusst „Data Mapper"** und *erlaubt* die Trennung — deshalb existiert imperative mapping überhaupt. Viele SQLAlchemy-Projekte nutzen die ORM-Klasse trotzdem als De-facto-Domäne (Variante 4 im Kleid von 3).
- **In Clean-/Hexagonal-/DDD-Architekturen** wird bewusst **getrennt** (Variante 3): ein *persistence model* ≠ *domain model*, um den Kern rein und infrastruktur-frei zu halten. Das ist exakt das Muster, das dieses Projekt fährt (Ports & Adapters).

### Für *dieses* Projekt: Variante 3 ist richtig
Die Domänenklassen tragen **echtes Verhalten und Invarianten**: `Booking.__post_init__` (nights ≥ 1), `is_firm`, `occupied_nights`; `Price`/`CapacityRow` sind `frozen` (Wert-Semantik); `PriceableUnit` wird *berechnet* und ist gar keine Zeile. Variante 4 würde die `frozen`-Semantik brechen, Validierung umgehen und einen `id` in die Domäne drücken. Die „Duplikation" `orm.Booking`/`domain.Booking` ist der **bewusste Preis der Entkopplung** — und sie zahlt sich in dem Moment, in dem Domänen- und Persistenzmodell auseinanderlaufen (die DB *will* einen Surrogat-`id`, die Domäne *nicht* — genau dieser Fall ist schon eingetreten).

### Kernabwägung
> **Pragmatismus** (weniger Code — Variante 4 / Active Record) **vs. Reinheit** (entkoppelter, testbarer Kern — Variante 3 / Data Mapper).

Kein absolutes Richtig. Faustregel: Bleiben Domänen- und Persistenzmodell **auf Dauer identisch** und ist die App CRUD-lastig → Variante 4 spart ehrlich Code. Hat die Domäne **eigenes Verhalten/Invarianten**, die sich unabhängig vom Schema entwickeln → Variante 3 (Trennung) hält den Kern sauber. Für eine domänenreiche, klar geschichtete Anwendung wie diese: **3**.
