---
tags:
  - documentation
---
Selbsttest zum Verständnis der rein objektorientierten Lösung der [[Ursprüngliche Aufgabe|Übungsaufgabe]]. Grundlage ist der Code-Stand aus [[Umsetzungsstand (OOP-Lösung)|Umsetzungsstand (OOP-Lösung)]] (Projektort: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/OOP`). Theorie-Hintergrund: [[1.1 OOP - Vertiefung|1]]. Leitfaden: [[1.1 OOP - Leitfaden|1]].

*Stand: 2026-07-09* — Jede Frage dokumentiert: **Frage**, **Nicks Antwort** (Original), **Musterlösung** und ein kurzes **Verdikt** (✅ sitzt · 🟡 im Kern richtig, unvollständig · ⬜ offen geblieben).

## Legende

| Symbol | Bedeutung |
|---|---|
| ✅ | Antwort saß |
| 🟡 | Kern richtig, aber unvollständig / kleiner Begriff daneben |
| ⬜ | offen gelassen — hier nur Musterlösung |

---

## A) `domain.py` — Modell & Kapselung

### 1. Warum ist `PriceableUnit` von Hand geschrieben, nicht `@dataclass`? 🟡

**Nicks Antwort:** Weil diese echt Instanzfunktionen hat und nicht nur Datenfelder.

**Musterlösung:** Auch eine `@dataclass` darf Methoden/Properties haben — das allein ist es nicht. Der Kern ist **Kapselung mit selektiver Sichtbarkeit**: Alle Felder sind privat (`_bookable_units`, `_optional_units`, …) und nur ein Teil ist über read-only-Properties lesbar. `bookable_units`/`optional_units` haben *bewusst keinen* Getter — sie sind nur aggregiert relevant (`total_capacity`, `has_negative_available_capacity`). Eine `@dataclass` würde **alle** Felder öffentlich machen. Zusätzlich trägt die handgeschriebene `__init__` die **Validierung** (`booked_units < 0`), die dank Unveränderlichkeit über die ganze Lebensdauer gilt.

### 2. Warum stehen die Kapazitäts-Argumente hinter `*,` (keyword-only)? ✅

**Nicks Antwort:** Ich verhindere, dass Leute einfach drei Integer eintragen und die Reihenfolge verwechseln, da alle drei Felder danach recht ähnlich sind.

**Musterlösung:** Genau. Drei `int` gleicher Art sind leicht vertauschbar; ohne `*,` würde `PriceableUnit("P1", d, 5, 0, 3)` still falsch zuordnen. Mit `*,` erzwingst du `bookable_units=…, booked_units=…, optional_units=…`.

### 3. Warum wird `booked_units < 0` abgelehnt, `bookable/optional < 0` aber nicht? ✅

**Nicks Antwort:** Nie kann eine negative Anzahl an Units gebucht sein, aber bei einer Überbuchung könnten wir negative *noch buchbare* Einheiten haben.

**Musterlösung:** Korrekt. Leitfrage: **„ist der Wert *unmöglich* oder nur *ungewöhnlich*?"** Eine Zählung kann nicht negativ sein (unmöglich → harter `ValueError`); negative Restkapazität ist real (Überbuchungs-Artefakt → nur ungewöhnlich, toleriert).

### 4. `occupancy` teilt durch `total_capacity` ohne 0-Prüfung — wo ist das abgesichert? 🟡

**Nicks Antwort:** In der RecommendationStrategy, wo `occupancy` konkret verwendet wird, wird auf `total_capacity != 0` geachtet. Ehrlich gesagt wahrscheinlich eher gefährlich.

**Musterlösung:** Der Schutz sitzt nicht in der Strategy, sondern im **Filter der `RecommendationEngine`** (`calculate_all`: `if u.total_capacity == 0: continue`). Die Strategy ruft `unit.occupancy` blind auf und *vertraut* auf die vorherige Filterung. Statt „gefährlich" besser: eine **dokumentierte Vorbedingung** (Docstring: „erwartet total_capacity > 0"). Vertretbar, solange die Invariante an genau einer Stelle erzwungen wird — es bleibt aber eine implizite Kopplung.

### 5. Wer loggt die negative Kapazität, und warum die Trennung Prädikat/Reaktion? ⬜

**Nicks Antwort:** (abgebrochen — „es wird geloggt im builder … Weil das")

**Musterlösung:** Geloggt wird im **`builder`**. `PriceableUnit.has_negative_available_capacity` liefert nur das **Prädikat**. Sinn der Trennung: (a) die Domain bleibt **I/O-frei** — Logging ist ein Seiteneffekt, der nicht in ein reines Geschäftsobjekt gehört; (b) der Builder kann **aggregiert** loggen (eine Zeile statt vieler). Regel in der Domain, Reaktion beim Aufrufer.

### 6. Warum bekommt `days_until_travel` das `reference_date` übergeben statt intern `date.today()`? ✅

**Nicks Antwort:** Sonst könnten z. B. bei einem Aufruf über Mitternacht verschiedene Tage im selben Lauf berücksichtigt werden.

**Musterlösung:** Genau. Zusatznutzen: In Tests lässt sich das Datum **pinnen** (`reference_date=…`) statt vom echten „heute" abzuhängen.

### 7. Warum ist `Booking.is_firm` eine `@property`, kein gespeichertes Feld? ✅

**Nicks Antwort:** Es könnte sich ändern, wenn z. B. ein Objekt firm wird. Die Property kann flexibel darauf reagieren.

**Musterlösung:** Ja. Kernbegriff: **Single Source of Truth** — `status` ist der einzige gespeicherte Zustand, `is_firm` wird abgeleitet und kann so *nie* zu `status` inkonsistent werden. Ein separates `bool`-Feld könnte man versehentlich anders setzen als `status`.

### 8. Warum belegt eine 2-Nächte-Buchung ab dem 01. den 01. + 02., nicht den 03.? ✅

**Nicks Antwort:** Weil man in der Regel am Auscheck-Tag morgens fährt und der Raum abends für die nächste Person bereit ist.

**Musterlösung:** Korrekt — Prinzip **„Nacht ≠ Tag"**: Der Check-out-Tag zählt nicht. Im Code: `range(self.nights)` → `start + 0` und `start + 1`, nicht `start + 2`.

### 9. Warum ist `Price` `frozen=True`, `Booking` aber nicht? ✅

**Nicks Antwort:** Bookings verändern sich, während Price einen vergangenen, feststehenden Price-Change loggt. Frozen-Objekte können gehasht werden, weil sie statisch sind.

**Musterlösung:** Richtig. `frozen=True` → nach Erzeugung unveränderlich → dadurch **hashbar** (nutzbar als `set`-/`dict`-Key). `Price` ist ein Value Object / Referenzdatum; `Booking` bleibt mutierbar.

---

## B) dunder-Methoden & `PriceRecommendation`

### 10. Was bewirkt `@dataclass(order=True)` mit `field(compare=False)` bei `max(recs)`? ✅

**Nicks Antwort:** Das bewirkt, dass dieses Feld für die Sortierung bei der automatisch generierten `get_max`-Methode genutzt wird.

**Musterlösung:** Genau. `order=True` generiert die dunder `__lt__`, `__le__`, `__gt__`, `__ge__`. Weil nur `recommended_adjustment_pct` `compare=True` ist, vergleicht `max()` **ausschließlich** dieses Feld.

### 11. Was wäre der Nachteil von `max(recs, key=lambda r: r.recommended_adjustment_pct)`? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** `key=…` würde funktionieren und ist **expliziter/entkoppelter** — man sieht am Aufrufort, wonach sortiert wird. Nachteil: der Key müsste an **jeder** Sortier-/Max-Stelle wiederholt werden, und die Klasse hätte keine „natürliche Ordnung". Der `order=True`-Weg zentralisiert die natürliche Ordnung *in der Klasse* (DRY), koppelt dafür jedes `max()` an dieses versteckte Semantik-Detail. Bewusster Trade-off.

### 12. Ist die Kopplung von `get_max` an das dunder-Verhalten der Domain-Klasse ok? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** Ja. `PriceRecommendation` definiert seine natürliche Ordnung *absichtlich* (Domänen-Entscheidung), und `get_max` hat genau die Aufgabe „höchste Anpassung". Das ist hohe Kohäsion, keine zufällige Kopplung — und dokumentiert.

---

## C) `sources.py` — I/O-Grenze, Protocol, DTO

### 13. Warum `Protocol` statt `ABC`, und wie „weiß" mypy ohne Vererbung von Konformität? 🟡

**Nicks Antwort:** Weil die DataSources nichts miteinander verbindet außer die Methoden, die sie bereitstellen müssen; kein geteilter Code.

**Musterlösung:** Erster Teil richtig (reiner Verhaltensvertrag ohne geteilte Logik → `Protocol`, kein Vererbungszwang). Ergänzung: `CsvDataSource` erbt *nicht* von `DataSource` — mypy erkennt Konformität über **structural typing** („duck typing für Typen"): Passt die Methodenstruktur/-signatur, gilt der Vertrag als erfüllt, ganz ohne `extends`.

### 14. Fehlt `load_prices`: wann fällt das auf — statisch (mypy) oder Laufzeit? 🟡

**Nicks Antwort:** Erst zur Laufzeit.

**Musterlösung:** Der eigentliche Punkt eines `Protocol`: **mypy fängt es *statisch***, sobald `CsvDataSource` als `DataSource` übergeben wird („missing method"). Zur **Laufzeit** erzwingt ein normales `Protocol` *nichts* — es kracht erst, wenn tatsächlich jemand `load_prices()` aufruft (`AttributeError`). Also: statisch = mypy vor dem Ausführen; Laufzeit = erst beim konkreten Aufruf.

### 15. Warum liegt `CapacityRow` in `sources.py`, nicht in `domain.py`? 🟡

**Nicks Antwort:** Weil es nicht zur Domäne gehört.

**Musterlösung:** Richtig, geschärft: Eine rohe CSV-Zeile ist ein **DTO / Transport-Objekt** an der I/O-Grenze — reine Rohdaten von außen. `PriceableUnit` ist ein **Domänen-Konzept** mit Verhalten (`occupancy`, `total_capacity`) und Invarianten. Deshalb gehört `CapacityRow` zur I/O-Schicht, nicht ins domänenreine `domain.py`.

### 16. Warum hat `CapacityRow` kein `booked_units`-Feld? ✅

**Nicks Antwort:** Weil ich die Datei so einlese, wie ich sie bekomme. `booked` ist ein abgeleitetes Feld einer Priceable Unit und sollte dort berechnet werden.

**Musterlösung:** Genau — `booked_units` steht nicht in der Quelle, wird als abgeleiteter Wert erst im `builder` (per `Counter`) berechnet.

### 17. Warum ist `_resolve_csv` eine `@staticmethod`? ✅

**Nicks Antwort:** Es hat keinen Bezug auf das Objekt und kann so auch ohne Instanziierung verwendet werden.

**Musterlösung:** Richtig. Kriterium: *Braucht eine Methode weder `self` noch `cls`, gehört aber logisch zur Klasse → `@staticmethod`.*

### 18. Warum `strptime(…, "%Y%m%d")` statt `date.fromisoformat`? 🟡

**Nicks Antwort:** Ich weiß nicht genau, was es tun würde, aber es ist einfach kein ISO-Format — das wird ja durch Bindestriche getrennt.

**Musterlösung:** Intuition stimmt. Konkret: `date.fromisoformat` erwartet **„YYYY-MM-DD" mit Bindestrichen**; bei `"20260108"` (ohne Bindestriche) wirft es einen `ValueError`. `strptime(…, "%Y%m%d")` liest genau das kompakte Präfixformat.

### 19. Warum `Decimal(row["current_price"])` direkt aus dem String, nie über `float`? 🟡

**Nicks Antwort:** Es gibt Rundungsfehler.

**Musterlösung:** Kern richtig, präziser: `float("19.99")` ist bereits eine **binäre Näherung** (≈ 19.98999…). `Decimal(float("19.99"))` würde diesen Fehler übernehmen. `Decimal("19.99")` aus dem **String** ist **exakt**.

---

## D) `builder.py` — Assembly & `Counter`

### 20. Warum hat der `PriceableUnitBuilder` keinen `__init__`, Daten fließen durch `build(...)`? 🟡

**Nicks Antwort:** Es braucht halt keinen Konstruktor.

**Musterlösung:** Zirkulär — die Frage ist *warum* nicht. Der Builder ist **zustandslos**: Weil die Daten über den `build(...)`-Parameter kommen (nicht in den `__init__`), hält keine Instanz einen bestimmten Datensatz. Dadurch ein **wiederverwendbarer Mitspieler**, den `app.py` **einmal** verdrahtet und pro Lauf neu füttert — analog zu `source`/`writer`.

### 21. Was hat das 0-Default des `Counter` mit einem `LEFT JOIN` zu tun? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** `Counter` liefert für einen **fehlenden Key `0`** (kein `KeyError`). Die `capacity_rows` treiben die Ausgabe (linke Seite); Zeile `booked[(row.product_id, row.calendar_date)]` gibt `0`, wenn es keine passende Firm-Buchung gibt. Exakt ein **`LEFT JOIN`**, bei dem die rechte (Buchungs-)Seite fehlt → `NULL`, hier durch das 0-Default ersetzt.

### 22. Warum filterst du im `Counter`-Aufbau auf `if b.is_firm`? ✅

**Nicks Antwort:** Eben nicht aktuell gebuchte Units, sondern nur irgendwann mal gebuchte Units — auch wenn das nicht mehr aktuell ist.

**Musterlösung:** Genau — nur `Firm` = aktuell gültige Belegung. Zählte man `Cancelled`/`Option` mit, wäre `booked_units` „irgendwann mal gebucht" statt „jetzt belegt", und `occupancy` überhöht.

### 23. Was ist ein Orphan-Booking, warum still verwerfen, warum die Summe loggen? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** Ein **Orphan** ist eine Firm-Buchungsnacht, deren `(product_id, date)` **keine** passende `CapacityRow` hat. Still verworfen, weil ohne Kapazitätszeile gar keine `PriceableUnit` baubar ist — und weil das Programm defensiv-tolerant gegen unsaubere Daten sein soll (wie der alte Ingestor). Geloggt wird die **Summe**, weil bei Millionen Zeilen Einzelmeldungen das Log fluten würden.

---

## E) `recommendations.py` — Strategy & Engine

### 24. Erkläre das Strategy-Pattern; wie fügst du eine zweite Formel hinzu, welches Prinzip? 🟡

**Nicks Antwort:** Es kapselt die konkrete Berechnung des Discounts. Ich müsste nur eine neue Klasse schreiben, die `calculate` implementiert, und diese in der `main`-Methode statt der Heuristic Strategy übergeben. Es heißt Loose Coupling.

**Musterlösung:** Strategy-Teil korrekt. Aber der benannte Grundsatz ist das **Open/Closed-Prinzip** (offen für Erweiterung, geschlossen für Modifikation) — Loose Coupling ist die *Folge* (die Engine kennt nur das `Protocol`), nicht der Name des Prinzips.

### 25. Warum wird `reference_date` einmal pro Lauf aufgelöst, nicht pro Einheit? ✅

**Nicks Antwort:** Habe ich schon vorher erklärt (vgl. Frage 6).

**Musterlösung:** Ja. Genau: `ref` wird **einmal pro Lauf** aufgelöst und für **alle** Einheiten benutzt, damit „heute" über den ganzen Batch **konsistent** ist. Pro Einheit neu `date.today()` könnte über Mitternacht rutschen.

### 26. Welches Prinzip trennt die Filter (Engine) von der toleranten Domain, warum schützt der 0-Filter? 🟡

**Nicks Antwort:** Separation of Concerns.

**Musterlösung:** SoC korrekt, geschärft: **Domain ist tolerant** (lehnt nichts ab), **Engine reagiert** (filtert). Der zweite Frageteil: Der Filter `total_capacity == 0` ist **nötig**, weil `occupancy` durch `total_capacity` teilt — ohne ihn `ZeroDivisionError`. Der Filter erzwingt die Vorbedingung aus Frage 4.

### 27. Warum Formel in `float`, Ergebnis via `Decimal(f"{pct:.6f}")`? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** Absicht: Die **Formel rechnet in `float`** — schnell, für eine Heuristik völlig ausreichend. Das **Ergebnis** wird via String (`Decimal(f"{pct:.6f}")`) verpackt, damit die serialisierte Zahl **sauber und deterministisch** ist (feste 6 Nachkommastellen, keine Binär-Artefakte). Durchgängig `Decimal` wäre langsamer ohne Mehrwert; durchgängig `float` gäbe unsaubere Ausgabewerte.

---

## F) `writers.py`, `app.py`, `main.py` — Ausgabe & Verdrahtung

### 28. Warum Pfad im Konstruktor und `write(recs)` ohne `path`-Argument? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:** Die Symmetrie zu `CsvDataSource(data_dir)` hält die **Protocol-Signatur sauber**: `RecommendationWriter.write(recs)` handelt nur von „*was* schreibe ich", nicht „*wohin*". Der Pfad ist ein **Konfigurationsdetail**, einmal im Konstruktor injiziert. Läge der Pfad in `write(path, recs)`, müsste jeder Aufrufer den Pfad kennen — die I/O-Ortsentscheidung würde in den Aufrufort durchsickern (leaky abstraction).

### 29. Warum `_FIELD_NAMES` aus `fields(PriceRecommendation)` ableiten, und warum `DictWriter`? ⬜

**Nicks Antwort:** Weiß ich ehrlich gesagt nicht.

**Musterlösung:** Ableiten verhindert einen **Drift-Bug**: Bekommt `PriceRecommendation` später ein Feld, würden hart getippte Spaltennamen und die `asdict(r)`-Zeilen **auseinanderlaufen** (fehlende Spalte / `DictWriter`-Fehler). Aus `fields()` abgeleitet bleiben Header und Zeilen **automatisch synchron**. `DictWriter` statt `csv.writer`: Zuordnung über **Spaltenname statt Position** → robust gegen Umsortieren, weniger Fehlerquellen.

### 30. Wie ermöglicht die Konstruktor-DI die Tests, und was ist der „Zahltag" des Designs? 🟡

**Nicks Antwort:** Ich kann synthetische Beispiele einfach einführen.

**Musterlösung:** Teilweise. Genauer: Weil `PricingApp` seine Abhängigkeiten **injiziert bekommt** statt sie selbst zu erzeugen, setzen Tests **Fakes** ein bzw. bauen Domänenobjekte direkt von Hand — **ganz ohne Dateisystem** (die 5 Tests). **Zahltag des DI-Designs:** Formel wechseln = **eine Zeile** in `main.py` (`HeuristicRecommendationStrategy()` → `AndereStrategy()`), sonst nichts. `main.py` ist die einzige Stelle mit konkreten Klassen.

### 31. (a) Warum `logging.basicConfig` in `main.py`? (b) Warum `python -m pricing.main`? ⬜

**Nicks Antwort:** (offen)

**Musterlösung:**
**(a)** `logging.basicConfig(...)` konfiguriert den **Root-Logger global** (Handler, Level, Format) — eine **Anwendungs-/Einstiegspunkt-Entscheidung**. Eine Library darf ihren Importeuren keine Logging-Konfiguration aufzwingen; die Module holen nur `getLogger(__name__)` und überlassen dem `main` das *Wie*.
**(b)** `python -m pricing.main` startet die Datei **als Modul im Paket** — relative Imports (`from .app import …`) haben dann einen **Eltern-Paket-Kontext**. Direkter Dateiaufruf (`python …/main.py`) macht `__name__ == "__main__"` **ohne** Paketzugehörigkeit → `ImportError: attempted relative import with no known parent package`.

---

## Offene Bonus-Fragen (noch zu beantworten)

- **B1.** Was unterscheidet **Kapselung** von **Abstraktion**? (je ein Beispiel aus dem Projekt)
- **B2.** **Loose Coupling** vs. **strong cohesion** — an einer selbst gewählten Klasse belegen.

## Auswertung

| Kategorie | Fragen |
|---|---|
| ✅ sitzt | 2, 3, 6, 7, 8, 9, 10, 16, 17, 22, 25 |
| 🟡 Kern richtig, unvollständig | 1, 4, 13, 14, 15, 18, 19, 20, 24, 26, 30 |
| ⬜ offen geblieben | 5, 11, 12, 21, 23, 27, 28, 29, 31 |

**Empfohlene Vertiefung:**
- **Open/Closed** vs. **Loose Coupling** vs. **Dependency Injection** (werden gern verwechselt — Frage 24, 30).
- **`Protocol` / structural typing** und **statisch (mypy) vs. Laufzeit** (Frage 13, 14).
- Die Kette **`occupancy` → Engine-Filter `total_capacity == 0` → tolerante Domain** (Frage 4, 26).
- **DTO-Konzept** und der **`fields()`-Trick** gegen Schema-Drift (Frage 15, 29).
