---
tags:
  - documentation
type: notiz
date: 2026-07-24
---

Abhakbarer **Plan für die komplette Unit-Test-Suite** der Pricing-Anwendung — im Tutorial-Stil, damit die pytest-**Syntax** genau dort erklärt wird, wo du sie zum ersten Mal brauchst. Begleitnotiz zum [[Umsetzungsstand (Testing mit PyTest)|Umsetzungsstand (Testing mit PyTest)]]; Theorie dahinter: [[1.3 Testing mit PyTest - Vertiefung|1.3 Testing mit PyTest – Vertiefung]].

> [!info] So arbeitest du mit dieser Liste
> Die Blöcke sind **von einfach nach komplex** sortiert — jeder führt eine neue Technik ein, die auf der vorigen aufbaut. Arbeite sie **der Reihe nach** ab, hak die `- [ ]`-Kästchen in Obsidian ab, wenn ein Test steht und grün ist. **Du schreibst die Tests, Claudian reviewt.** Die Code-Schnipsel hier sind *Syntax-Muster*, keine fertigen Lösungen — die konkreten Werte/Fälle baust du selbst.

## Umfang: nur Unit-Tests

Diese Liste enthält **ausschließlich Modultests** — die breite Basis der [[1.3 Testing mit PyTest - Vertiefung#Mike Cohns Testpyramide|Testpyramide]]. Getestet wird der **Kern** (`domain.py`, `core/`), der ohne DB/Dateisystem/Netz auskommt. Der letzte Block (`PricingApp`) nutzt **Test-Doubles** (Stub/Mock) — das ist immer noch ein Unit-Test (die Einheit isoliert, Nachbarn ersetzt), kein Integrationstest.

**Bewusst *nicht* hier:** die Adapter (`sources.py`, `writers.py`, `ingest.py`, `orm.py`, `db.py`) fassen echte Infrastruktur an → das wären **Integrationstests** ([[1.3 Testing mit PyTest - Leitfaden#Out of Scope|laut Leitfaden Out of Scope]]).

## Setup & Konventionen (einmalig)

Wir spiegeln die Modulstruktur im `tests/`-Ordner (eine Testdatei pro getestetem Modul):

```
tests/
├── conftest.py            # geteilte Fixtures (kommt in Block 7/13)
├── test_domain.py         # Blöcke 1–6
├── test_builder.py        # Blöcke 7–8   (evtl. bestehende builder-Tests hierher ziehen)
├── test_recommendations.py# Blöcke 9–12  (evtl. bestehende recommendations-Tests hierher)
└── test_app.py            # Block 13
```

- pytest **findet** Tests automatisch: Datei `test_*.py`, Funktion `test_*`, optional Klasse `Test*`.
- Ausführen: `pytest -q` (knapp), `pytest -v` (jeder Test einzeln), `pytest tests/test_domain.py -v` (eine Datei), `pytest -k occupancy` (Namensfilter).
- Import im Test: `from pricing.domain import PriceableUnit, Booking, ...`

> [!tip] Der bestehende `tests/test_pricing.py` (5 Tests, grün)
> Die schon vorhandenen Tests sind unten als `- [x]` markiert. **Vorschlag:** beim Anlegen der neuen Dateien die 5 bestehenden Tests thematisch mitnehmen (builder → `test_builder.py`, recommendations → `test_recommendations.py`) und `test_pricing.py` auflösen — sauberer als eine Sammeldatei. (Erst wenn dir die Aufteilung zusagt; sonst bleibt sie erst mal liegen.)

---

# Block 1 — `PriceableUnit`: reine Berechnungen

**Neue Syntax:** das nackte `assert`. Kein Framework-Ballast — pytest schreibt die Assertion so um, dass du im Fehlerfall die *echten* Werte siehst (`assert 4 == 3`).

```python
from datetime import date
from pricing.domain import PriceableUnit


def test_total_capacity_summiert_alle_drei_toepfe() -> None:
    # Arrange — Konstruktor-Args sind keyword-only (das * in der Signatur)!
    unit = PriceableUnit("P1", date(2026, 1, 1),
                         bookable_units=3, booked_units=2, optional_units=1)
    # Act + Assert
    assert unit.total_capacity == 6
```

- [x] `total_capacity` = bookable + optional + booked (eine klare Kombination)
- [x] `has_negative_available_capacity` == **False**, wenn bookable + optional == 0 (Grenzwert)
- [x] `has_negative_available_capacity` == **True**, wenn bookable + optional == -1 (knapp daneben)
- [x] `days_until_travel`: Zukunft → positiv (z. B. calendar 10., ref 1. → 9)
- [x] `days_until_travel`: selber Tag → 0 (Grenzwert)
- [x] `days_until_travel`: Vergangenheit → negativ

> [!note] Denk-Muster: Grenzwertanalyse
> Die drei `has_negative...`/`days_until_travel`-Fälle sind bewusst „an der Grenze und knapp daneben" gewählt — dort sitzen die meisten Bugs.

> [!todo] Refactor-Übung (geplant für Block 9)
> Die drei `days_until_travel`-Tests teilen dieselbe Logik und unterscheiden sich nur im Datumspaar → idealer `parametrize`-Kandidat. Wenn `parametrize` in Block 9 eingeführt ist, kommen wir hierher zurück und bauen die drei Tests als „Refactor"-Schritt in einen parametrisierten Test um.

---

# Block 2 — `PriceableUnit`: `occupancy` (Wert + Fehlerfall)

**Neue Syntax:** `pytest.approx` (Floats nie mit `==` vergleichen) und `pytest.raises` (prüfen, dass korrekt *fehlgeschlagen* wird).

```python
import pytest


def test_occupancy_normalfall() -> None:
    unit = PriceableUnit("P1", date(2026, 1, 1),
                         bookable_units=2, booked_units=2, optional_units=0)
    assert unit.occupancy == pytest.approx(0.5)   # 2 / (2+0+2)


def test_occupancy_bei_null_kapazitaet_wirft_valueerror() -> None:
    unit = PriceableUnit("P1", date(2026, 1, 1),
                         bookable_units=0, booked_units=0, optional_units=0)
    with pytest.raises(ValueError, match="occupancy undefiniert"):
        _ = unit.occupancy          # Property-Zugriff löst den Fehler aus
```

- [x] `occupancy` Normalfall → korrekter Bruch (mit `approx`)
- [x] `occupancy` bei `total_capacity == 0` → `ValueError` (mit `match=`)
- [x] `occupancy` == 0.0, wenn `booked_units == 0`, aber Kapazität > 0 (Grenzwert „leer")
- [x] `occupancy` == 1.0, wenn alles gebucht (bookable=optional=0, booked>0)

> [!warning] `pytest.raises` ist ein Kontextmanager
> Der erwartete Fehler muss **innerhalb** des `with`-Blocks passieren. `match=` ist ein **Regex** auf die Fehlermeldung — Sonderzeichen ggf. escapen.

---

# Block 3 — `PriceableUnit`: Konstruktor-Invariante

Gleiche Technik (`pytest.raises`), jetzt auf die Validierung im `__init__`.

- [x] `booked_units = -1` → `ValueError` (mit `match="booked_units darf nicht negativ"`)
- [x] `booked_units = 0` konstruiert **ohne** Fehler (Grenzwert: 0 ist erlaubt)

---

# Block 4 — `Booking`

**Neue Syntax:** noch keine — `pytest.raises` + `assert` wiederholen und festigen. `occupied_nights` gibt eine Liste zurück → auf **Länge und Inhalt** prüfen.

```python
from pricing.domain import Booking


def test_occupied_nights_zwei_naechte() -> None:
    b = Booking("P1", date(2026, 1, 1), nights=2, status="Firm")
    assert b.occupied_nights() == [date(2026, 1, 1), date(2026, 1, 2)]
```

- [x] `nights = 0` → `ValueError` (Grenzwert, `match="mindestens 1 Nacht"`)
- [x] `nights = 1` konstruiert ohne Fehler (knapp daneben)
- [x] `is_firm` == True bei `status="Firm"`
- [x] `is_firm` == False bei irgendeinem anderen Status (z. B. `"Cancelled"`)
- [x] `occupied_nights()` liefert genau `[start, start+1]` bei `nights=2` (Nacht ≠ Tag — Check-out zählt nicht)
- [x] `occupied_nights()` hat Länge == `nights`

---

# Block 5 — `CapacityRow` & `Price`: `frozen`-Wertsemantik (optional)

**Neue Syntax:** prüfen, dass ein `frozen`-Dataclass **unveränderlich** ist — er soll bei Zuweisung `FrozenInstanceError` (Unterklasse von `AttributeError`) werfen.

```python
from dataclasses import FrozenInstanceError
from pricing.domain import CapacityRow


def test_capacityrow_ist_unveraenderlich() -> None:
    row = CapacityRow("P1", date(2026, 1, 1), bookable_units=5, optional_units=0)
    with pytest.raises(FrozenInstanceError):
        row.bookable_units = 3        # type: ignore[misc]
```

- [x] `CapacityRow` ist `frozen` (Zuweisung wirft `FrozenInstanceError`)
- [x] `Price` ist `frozen` (analog)
- [-] *(optional)* zwei `Price` mit identischen Feldern sind gleich (`==`) — Wertsemantik von Dataclasses

> [!note] „Optional" heißt hier
> Diese Tests prüfen letztlich Python-/Dataclass-Verhalten, nicht *deine* Logik. Sie schaden nicht (dokumentieren die bewusste `frozen`-Entscheidung), haben aber niedrige Priorität. In der [[1.3 Testing mit PyTest - Vertiefung#Wann man *nicht* mocken sollte (Over-Mocking)|Vertiefung]]-Logik: nah am Kern viel testen, Triviales sparsam.

---

# Block 6 — `PriceRecommendation`: Sortierordnung

`PriceRecommendation` ist `@dataclass(order=True)`, aber **nur** `recommended_adjustment_pct` zählt für den Vergleich (alle anderen Felder `compare=False`). Genau das nutzt `RecommendationEngine.get_max` (`max(recs)`).

- [x] Von zwei Recommendations mit unterschiedlichem `pct` ist die mit **höherem** `pct` die „größere" (`r_hoch > r_niedrig`)
- [x] `max([r_niedrig, r_hoch])` liefert `r_hoch` — auch wenn `r_hoch` in der Liste zuerst steht
- [x] Zwei Recommendations mit **gleichem** `pct`, aber unterschiedlichem `product_id`, sind bzgl. Ordnung „gleich" (`not r_a < r_b and not r_b < r_a`) — belegt, dass die anderen Felder wirklich `compare=False` sind

---

# Block 7 — `PriceableUnitBuilder`: Aggregation

**Neue Syntax:** **Fixtures** — wiederkehrendes „Arrange" auslagern. Eine `@pytest.fixture`-Funktion, die ein Test per **Parametername** anfordert (Dependency Injection).

```python
import pytest
from pricing.core.builder import PriceableUnitBuilder
from pricing.domain import CapacityRow, Booking


@pytest.fixture
def builder() -> PriceableUnitBuilder:
    return PriceableUnitBuilder()


def test_zwei_firm_buchungen_gleiche_nacht_summieren(builder) -> None:
    caps = [CapacityRow("P1", date(2026, 1, 1), bookable_units=5, optional_units=0)]
    bookings = [
        Booking("P1", date(2026, 1, 1), nights=1, status="Firm"),
        Booking("P1", date(2026, 1, 1), nights=1, status="Firm"),
    ]
    units = builder.build(caps, bookings)
    assert units[0].booked_units == 2
```

- [x] Firm-Buchung expandiert über die Nächte (Nacht ≠ Tag) — *existiert:* `test_builder_expandiert_firm_buchung_ueber_naechte`
- [x] Nicht-Firm-Buchungen werden ignoriert — *existiert:* `test_builder_ignoriert_nicht_firm_buchungen`
- [x] Zwei Firm-Buchungen auf dieselbe Nacht → `booked_units == 2` (Akkumulation)
- [x] Kapazitätszeile ohne passende Buchung → `booked_units == 0`
- [x] Leere Eingaben (`build([], [])`) → leere Liste (kein Crash)
- [x] Eine Buchung, deren Nächte **teilweise** außerhalb der Kapazitätszeilen liegen, belegt nur die vorhandenen Tage

> [!tip] Fixture-Reichweite
> Fixtures, die **mehrere Testdateien** teilen sollen, wandern nach `conftest.py` (pytest findet sie automatisch, ohne Import). Für nur eine Datei reicht die Fixture lokal in der Datei.

---

# Block 8 — `PriceableUnitBuilder`: Warn-Logs

**Neue Syntax:** die eingebaute Fixture **`caplog`** — prüfen, dass der Code bei problematischen Daten *warnt* (statt zu crashen). Das sind die „Fehlerpfade", die die Coverage später sichtbar macht.

```python
import logging


def test_negative_kapazitaet_wird_geloggt(builder, caplog) -> None:
    caps = [CapacityRow("P1", date(2026, 1, 1), bookable_units=-1, optional_units=0)]
    with caplog.at_level(logging.WARNING):
        builder.build(caps, [])
    assert "negativer Kapazität" in caplog.text
```

- [x] Kapazitätszeile mit negativer Kapazität → `WARNING` „… Einheiten mit negativer Kapazität" (`caplog`)
- [x] Firm-Buchungsnacht **ohne** passende Kapazitätszeile → `WARNING` „… Firm-Buchungsnächte ohne passende Kapazitätszeile" (orphan nights)
- [x] „Sauberer" Fall (alles passt) → **keine** Warnung (`caplog.records == []`)

> [!warning] `caplog` braucht ein Level
> Ohne `caplog.at_level(logging.WARNING)` (oder `caplog.set_level(...)`) fängt caplog je nach Konfiguration nichts. Prüfe entweder `caplog.text` (der ganze Text) oder gezielter `caplog.records[0].levelno == logging.WARNING`.

---

# Block 9 — `HeuristicRecommendationStrategy`: die Formel

**Neue Syntax:** **`@pytest.mark.parametrize`** — dieselbe Prüf-Logik mit vielen Eingabe/Erwartungs-Paaren, jedes als *eigener* Testfall. Ideal für eine Formel.

```python
from decimal import Decimal
from pricing.core.recommendations import HeuristicRecommendationStrategy
from pricing.domain import PriceableUnit


@pytest.mark.parametrize(
    "bookable, booked, optional, tage_vor_reise, erwartet",
    [
        (0, 2, 0, 300, Decimal("11.250000")),   # occ=1.0, d=1.0  -> 15*0.5*1.5
        # weitere Fälle: occ=0.5 (neutral -> 0), occ<0.5 (negativ), d=0 ...
    ],
)
def test_formel(bookable, booked, optional, tage_vor_reise, erwartet) -> None:
    unit = PriceableUnit("P1", date(2026, 1, 1),
                         bookable_units=bookable, booked_units=booked, optional_units=optional)
    ref = date(2026, 1, 1) - timedelta(days=tage_vor_reise)
    assert HeuristicRecommendationStrategy().calculate(unit, ref) == erwartet
```

- [x] Ein konkreter Formelwert (occ=1.0, d=1.0 → 11.25 %) — *existiert:* `test_heuristische_formel` (in `parametrize` überführen)
- [x] `occupancy == 0.5` → Ergebnis exakt `0` (der „neutrale" Punkt: Faktor `(occ-0.5)` wird 0)
- [x] `occupancy < 0.5` (z. B. wenig gebucht) → **negativer** Prozentwert (Preis-Senkung)
- [x] `days_until_travel == 0` (Reisetag = Referenztag) → Faktor `(d+0.5)` = 0.5
- [x] Rückgabetyp ist `Decimal` und hat 6 Nachkommastellen (String-Form prüfen)

> [!tip] Weitere `parametrize`-Möglichkeiten
> `ids=[...]` gibt jedem Fall einen sprechenden Namen in der Ausgabe. Mehrere `@parametrize`-Dekoratoren stapeln = **Kreuzprodukt** (wächst schnell!). Ein einzelner Fall lässt sich per `pytest.param(..., marks=pytest.mark.xfail)` markieren.

---

# Block 10 — `RecommendationEngine`: `reference_date`-Fallback

**Neue Syntax:** **`monkeypatch`** — eine Abhängigkeit (hier: das „heutige" Datum) temporär ersetzen, damit der Test **deterministisch** (Repeatable) ist. `reference_date` fällt auf `date.today()` zurück, wenn keins übergeben wird — genau die Art beweglicher Abhängigkeit, die man im Test einfriert.

```python
import pricing.core.recommendations as rec_mod


def test_reference_date_faellt_auf_heute_zurueck(monkeypatch) -> None:
    class FakeDate(date):
        @classmethod
        def today(cls) -> "date":
            return date(2026, 7, 24)
    monkeypatch.setattr(rec_mod, "date", FakeDate)   # 'date' im Modul rec_mod ersetzen
    engine = rec_mod.RecommendationEngine(rec_mod.HeuristicRecommendationStrategy())
    assert engine.reference_date == date(2026, 7, 24)
```

- [x] `reference_date` gibt das **übergebene** Datum zurück, wenn eins gesetzt ist
- [x] `reference_date` fällt auf `date.today()` zurück, wenn `None` (mit `monkeypatch` eingefroren)

> [!warning] „patch where it's used"
> Wir patchen `date` **im Modul `recommendations`** (`rec_mod.date`), nicht global `datetime.date` — der Code schaut den Namen dort nach, wo er ihn per `from datetime import date` gebunden hat. Das ist dieselbe Regel wie beim Mocken (siehe [[1.3 Testing mit PyTest - Vertiefung#`patch` — und die goldene Regel „patch where it's used"|Vertiefung]]).

---

# Block 11 — `RecommendationEngine.calculate_all`: Filterlogik

Die Engine überspringt drei Kategorien (Vergangenheit / Null-Kapazität / negative Kapazität) und warnt für zwei davon. Alles rein in-memory testbar.

- [x] Zukünftige, gültige Einheit → genau eine Empfehlung; Vergangenheit + Null-Kapazität übersprungen — *existiert:* `test_engine_ueberspringt_vergangenheit_und_null_kapazitaet`
- [x] Einheit **am Referenztag** (`calendar_date == ref`) → übersprungen (Grenzwert: `<=`, nicht `<`)
- [x] Einheit mit **negativer** verfügbarer Kapazität (bookable+optional < 0, aber total_capacity > 0) → übersprungen + `WARNING` (`caplog`) — *Claudian*
- [x] `WARNING` bei übersprungener Null-Kapazität (`caplog`) — *Claudian*
- [x] Alle Felder der erzeugten `PriceRecommendation` korrekt befüllt (`product_id`, `days_until_travel`, `occupancy`, `total_capacity`, `booked_units`) — nicht nur das `pct` — *Claudian*
- [x] Leere Eingabe → leere Liste, keine Warnung

---

# Block 12 — `RecommendationEngine.get_max`

- [x] Leere Liste → `None` — *existiert:* `test_get_max_gibt_none_bei_leerer_liste`
- [x] Liste mit mehreren → die Recommendation mit dem **höchsten** `pct` (baut auf Block 6, Ordering) — *Claudian*
- [x] Liste mit **einem** Element → genau dieses Element (Grenzwert) — *Claudian*

---

# Block 13 — `PricingApp`: Test-Doubles (Stub + Mock)

**Neue Syntax:** `unittest.mock.Mock` als **Test-Double**. `PricingApp` bekommt `DataSource`, `builder`, `engine`, `writer` **injiziert** (Dependency Injection) — deshalb brauchen wir *kein* `patch`, sondern reichen Doubles herein. Das ist das Paradebeispiel **Stub vs. Mock** aus der [[1.3 Testing mit PyTest - Vertiefung#Stub vs. Mock — der entscheidende Unterschied (Kontrollfrage)|Vertiefung]]:

- Die **`DataSource`** wird als **Stub** benutzt: sie *liefert Daten rein* (`return_value`), wir prüfen danach das Ergebnis (state verification).
- Der **`Writer`** wird als **Mock** benutzt: wir prüfen die *Interaktion* — wurde `write(...)` genau einmal mit den richtigen Recs aufgerufen? (behavior verification, weil Schreiben ein **Seiteneffekt** ist).

```python
from unittest.mock import Mock
from pricing.core.app import PricingApp
from pricing.core.builder import PriceableUnitBuilder
from pricing.core.recommendations import (
    HeuristicRecommendationStrategy, RecommendationEngine,
)


def test_run_liest_rechnet_schreibt_und_gibt_max_zurueck() -> None:
    source = Mock()                                  # Stub-Nutzung
    source.load_capacity.return_value = [ ... ]      # canned rows
    source.load_bookings.return_value = [ ... ]
    writer = Mock()                                  # Mock-Nutzung

    app = PricingApp(
        source=source,
        builder=PriceableUnitBuilder(),              # echte, schnelle Logik
        engine=RecommendationEngine(HeuristicRecommendationStrategy(),
                                    reference_date=date(2026, 1, 1)),
        writer=writer,
    )
    result = app.run()

    writer.write.assert_called_once()                # Interaktion verifizieren
    # result ist die höchste Recommendation (oder None)
```

- [x] `run()` ruft `source.load_capacity` **und** `source.load_bookings` auf (Stub liefert Daten)
- [x] `run()` ruft `writer.write` **genau einmal** auf (`assert_called_once`)
- [x] `run()` übergibt an `writer.write` **genau** die berechneten Recs (`assert_called_once_with(...)` oder `call_args` prüfen)
- [x] `run()` **gibt** die höchste Recommendation zurück (Rückgabewert prüfen) — *Claudian; Stub im Test umkonfiguriert, damit **zwei** Recs entstehen — mit nur einer wäre „höchste" nicht prüfbar*
- [x] `calculate_all_price_recommendations()` schreibt, gibt aber nichts zurück — *Claudian*
- [x] `get_max_recommendation()` schreibt **nicht** (`writer.write.assert_not_called()`) — *Claudian*
- [x] *(Fehlerpfad, Kür)* `source.load_capacity.side_effect = ConnectionError(...)` → `run()` propagiert den Fehler (`pytest.raises`) — zeigt, wie man Infrastruktur-Ausfälle *ohne* echte DB testet — *Claudian*

> [!tip] Fixture liefert den Normalfall, der Sonderfall-Test justiert nach
> Die `harness`-Fixture gibt `app`, `source` und `writer` heraus. Dadurch kann ein einzelner Test den **Stub umkonfigurieren** (`harness.source.load_capacity.return_value = [...]`), ohne dass die Fixture für alle anderen Tests komplizierter wird — genutzt in `test_run_returns_highest_of_multiple` und im `side_effect`-Fehlerpfad.

> [!tip] `autospec` gegen „grün lügende" Mocks
> Ein rohes `Mock()` akzeptiert **jeden** Aufruf — auch Tippfehler. Wenn du sichergehen willst, dass dein Double nur die *echten* Methoden hat: `create_autospec(DataSource)` bzw. `Mock(spec=DataSource)`. Für den Einstieg reicht `Mock()`, aber merk dir die Option.

> [!note] Warum kein `patch` nötig ist
> Weil `PricingApp` alle Abhängigkeiten **per Konstruktor** bekommt (DI), reichen wir die Doubles einfach herein — keine Modulpfad-Fummelei. Genau das meinte 1.1 mit „testbarem Design": gut entkoppelter Code ist leicht zu testen.

---

# Fortschritt (Gesamtüberblick)

| Block | Einheit | Neue Technik | Tests offen / gesamt |
|---|---|---|---|
| 1 | `PriceableUnit` Berechnungen | `assert` | 0/6 |
| 2 | `PriceableUnit.occupancy` | `approx`, `raises` | 0/4 |
| 3 | `PriceableUnit` Konstruktor | `raises` | 0/2 |
| 4 | `Booking` | (Wdh.) | 0/6 |
| 5 | `frozen`-Dataclasses | `FrozenInstanceError` | 0/3 |
| 6 | `PriceRecommendation` Ordering | (Wdh.) | 0/3 |
| 7 | `builder` Aggregation | Fixtures | 2/6 ✅✅ |
| 8 | `builder` Warn-Logs | `caplog` | 0/3 |
| 9 | Strategy-Formel | `parametrize` | 1/5 ✅ |
| 10 | Engine `reference_date` | `monkeypatch` | 0/2 |
| 11 | Engine `calculate_all` | (Wdh. caplog) | 1/6 ✅ |
| 12 | Engine `get_max` | (Wdh.) | 1/3 ✅ |
| 13 | `PricingApp` | `Mock`/Stub | 0/7 |

**Legende:** ✅ = ein bereits existierender Test deckt diese Zeile. Beim Umziehen von `test_pricing.py` (Setup-Hinweis oben) die vorhandenen Tests in die passenden Blöcke einsortieren.

## Nach der Liste: Coverage (Aufgabenteil 2)

Wenn die Blöcke stehen: `pytest-cov` installieren und
```bash
pytest --cov=pricing --cov-branch --cov-report=term-missing
```
laufen lassen — die **`missing`-Spalte** zeigt, welche Zeilen/Zweige diese Liste *doch* noch übersehen hat. Details/Einordnung in der [[1.3 Testing mit PyTest - Vertiefung#Testabdeckung / Code-Coverage|Vertiefung → Coverage]].
