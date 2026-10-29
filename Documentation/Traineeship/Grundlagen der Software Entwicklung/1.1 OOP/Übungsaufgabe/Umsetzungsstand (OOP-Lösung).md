---
tags:
  - documentation
type: notiz
date: 2026-07-09
---

Abgeschlossener Umsetzungsstand der rein objektorientierten Lösung zur [[Ursprüngliche Aufgabe|Übungsaufgabe]] (vgl. [[1.1 OOP - Leitfaden|1.1 Leitfaden]]). Aus der ursprünglichen [[Meine ursprüngliche Lösung|Bewerber-Lösung]] (FastAPI + Postgres + Polars) wurde bewusst **neu und rein objektorientiert** begonnen — ohne Datenbank, REST-API und DataFrames, nur Business-Logik mit der Standard-Library.

Architektur: [[Klassendiagramm (OOP-Loesung).canvas|Klassendiagramm]] · Theorie: [[1.1 OOP - Vertiefung|Vertiefung]] · Python-Lektionen: [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md|Python-Lernnotizen]] · Performance: [[Optimierungsansätze (OOP-Lösung)|Optimierungsansätze]] · Selbsttest: [[Q&A – Code-Verständnis (OOP-Lösung)|Q&A – Code-Verständnis]]

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/OOP` (Code liegt bewusst außerhalb des Vaults)

## Endstand

`src/pricing/` im src-Layout, ein Modul pro Concern. Pipeline: **Eingabe → Assembly → Business-Logik → Ausgabe**, dazu Orchestrierung und Composition Root.

| Modul | Schicht | Inhalt |
|---|---|---|
| `domain.py` | Domänen-Modell | `PriceableUnit`, `Booking`, `Price`, `PriceRecommendation` |
| `sources.py` | Eingabe (I/O) | `DataSource` (`Protocol`), `CapacityRow` (DTO), `CsvDataSource` |
| `builder.py` | Assembly | `PriceableUnitBuilder` — Kapazität + Bookings → Units |
| `recommendations.py` | Business-Logik | `RecommendationStrategy` (`Protocol`), `HeuristicRecommendationStrategy`, `RecommendationEngine` |
| `writers.py` | Ausgabe (I/O) | `RecommendationWriter` (`Protocol`), `CsvRecommendationWriter` |
| `app.py` | Orchestrierung | `PricingApp` mit Konstruktor-DI |
| `main.py` | Entry Point | Composition Root, Start via `python -m pricing.main` |

**Verifiziert an echten Daten:** 1,33 Mio. `PriceableUnit`s in ~8 s, 187.889 Empfehlungen, Spanne −30,75 … +24,78 %. Qualitäts-Gate grün: `pytest` 5 Tests, `ruff check`/`format` sauber, `mypy` sauber (9 Dateien). Laufzeit-Deps bewusst leer — nur Standard-Library.

## Wichtigste Entscheidungen

**Pipeline aus austauschbaren Schichten.** Jede Schicht kennt die nächste nur über eine Schnittstelle. Abhängigkeitsrichtung strikt nach innen: alle importieren `domain`, `domain` importiert nichts zurück — so bleibt die Fachlogik ohne Dateisystem testbar.

**`Protocol` statt `ABC` für alle Interfaces.** Reine Verhaltensverträge ohne geteilte Logik → kein Vererbungszwang, structural typing. `CsvDataSource` erbt bewusst *nicht* von `DataSource`.

**`PriceableUnit` handgeschrieben statt `@dataclass`.** Der Grund ist nicht „hat Methoden" (das darf eine Dataclass auch), sondern **Kapselung mit selektiver Sichtbarkeit**: Aller Zustand privat, nur read-only-Properties nach außen — und `bookable_units`/`optional_units` haben bewusst *keinen* Getter, weil sie nur aggregiert relevant sind (`total_capacity`, `has_negative_available_capacity`). Eine Dataclass würde alle Felder öffentlich machen. Kapazitäts-Argumente sind **keyword-only** (`*,`), damit die drei gleichartigen `int` nicht vertauscht werden können.

**Validierung nur bei echten Invarianten.** Leitfrage: *„ist der Wert unmöglich oder nur ungewöhnlich?"* Hart abgelehnt werden `booked_units < 0` (eine Zählung kann nicht negativ sein) und `nights < 1`. Bewusst **nicht** abgelehnt: negative Restkapazität (reale Überbuchung), `total_capacity == 0` (an dem Tag nicht angeboten), unbekannte `status`-Werte. Diese Fälle behandelt die Engine fachlich — die Domain bleibt tolerant.

**Regel in der Domain, Reaktion beim Aufrufer.** `has_negative_available_capacity` liefert nur das *Prädikat*; geloggt wird im `builder`, und zwar **aggregiert** (eine Zeile statt 197). Hält die Domain I/O-frei und das Log lesbar.

**`Booking` mutabel, `Price` frozen.** `status` ändert sich fachlich, ein eingefrorenes `Booking` würde das künstlich verbieten. `Price` ist ein Value Object — nach Erzeugung konstant und dadurch hashbar. `is_firm` ist eine abgeleitete `@property`, kein gespeichertes Feld: Single Source of Truth, kann nie zu `status` inkonsistent werden.

**`PriceRecommendation` als `@dataclass(order=True)` mit `field(compare=False)`.** Nur `recommended_adjustment_pct` ist Vergleichsfeld → `max(recs)` liefert direkt die höchste Anpassung, ohne `key=`. Bewusster Trade-off: zentralisiert die natürliche Ordnung in der Klasse (DRY), koppelt dafür jedes `max()` an dieses Semantik-Detail.

**`CapacityRow` gehört zu `sources.py`, nicht zu `domain.py`.** Eine rohe CSV-Zeile ist ein **DTO an der I/O-Grenze**, kein Domänen-Konzept. `booked_units` ist bewusst kein Feld — es steht nicht in der Quelle, sondern wird im `builder` abgeleitet.

**Format-Validierung an der Grenze.** Datum per `strptime(…, "%Y%m%d")` — die CSV nutzt `YYYYMMDD` ohne Bindestriche, `date.fromisoformat` würde scheitern. Preis per `Decimal(str)` **direkt aus dem String**, nie über `float`: `float("19.99")` ist bereits eine binäre Näherung. Dateinamen per Glob (`*_<suffix>.csv`), weil die Kundendateien ein täglich wechselndes Datums-Präfix tragen.

**`booked_units` per `Counter` abgeleitet.** Firm-Bookings über `occupied_nights()` in Nächte expandiert, per `Counter[(product_id, date)]` gezählt. Das 0-Default des `Counter` erledigt den „kein Match → 0"-Fall — **faktisch ein `LEFT JOIN`**. Ersetzt Polars' `explode`+`group_by`+`join` durch ~5 Zeilen Standard-Library.

**Strategy-Pattern für die Formel.** `RecommendationStrategy` kapselt *nur* die Berechnung → eine zweite Formel ist eine neue Klasse plus eine geänderte Zeile in `main.py`. Das umgesetzte Prinzip ist **Open/Closed**; Loose Coupling ist die Folge, nicht der Name.

**`reference_date` einmal pro Lauf.** In der Engine aufgelöst und an die Strategy durchgereicht, statt pro Einheit `date.today()`. Sonst könnte ein Lauf über Mitternacht rutschen; als Nebennutzen ist das Datum in Tests pinnbar.

**`Decimal`/`float` bewusst getrennt.** Die Formel rechnet in `float` (schnell, für eine Heuristik ausreichend), das Ergebnis wird via `Decimal(f"{pct:.6f}")` verpackt — deterministische Ausgabe ohne Binär-Artefakte. Durchgängig `Decimal` wäre langsamer ohne Mehrwert.

**`csv.DictWriter` und `fieldnames` aus `fields()` abgeleitet.** Zuordnung über Spaltennamen statt Position; die Ableitung aus der Dataclass verhindert einen **Drift-Bug**, wenn `PriceRecommendation` später ein Feld bekommt.

**Ausgabepfad im Konstruktor, nicht in `write()`.** Hält die Protocol-Signatur sauber: `write(recs)` handelt nur von „*was* schreibe ich", nicht „*wohin*". Läge der Pfad im Aufruf, müsste jeder Aufrufer die I/O-Ortsentscheidung kennen — eine leaky abstraction.

**Bewusst *nicht* getan.** Keine `PriceableUnit`-Subklassen (die Daten kennen nur eine Art Einheit → Speculative Generality). Kein `Protocol` für den Builder (nur eine Assembly-Logik, nichts wird ausgetauscht). Kein DI-Framework — „Poor Man's DI" über Konstruktor-Parameter reicht.

## Gelernte Lektionen

- **Pakete startet man mit `python -m pricing.main`, nie durch direkten Dateiaufruf.** Sonst `ImportError: attempted relative import with no known parent package` — relative Imports brauchen Paketkontext.
- **`logging.basicConfig` gehört in den Entry Point**, nicht in Bibliotheks-Module. Die holen nur `getLogger(__name__)` und überlassen dem `main` das Wie.
- **`py.typed` nicht vergessen** (PEP-561-Marker) — sonst behandelt mypy das installierte Paket als „untyped".
- **Daten-Erkenntnis:** `is_bookable`/`is_option` in `capacity.csv` sind **echte Stückzahlen** (bis 242, auch negative wie −16), keine 0/1-Flags. Das bestätigte das Toleranz-Design der Domain im Nachhinein.
- **Der Zahltag des DI-Designs:** Formel wechseln = eine Zeile in `main.py`. `main.py` ist die einzige Stelle im Projekt mit konkreten Klassen.
