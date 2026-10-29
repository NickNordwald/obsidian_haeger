---
tags:
  - documentation
type: notiz
date: 2026-07-26
---

Abgeschlossener Umsetzungsstand der **Test-Erweiterung** der Pricing-Anwendung (Aufgabe: [[1.3 Testing mit PyTest - Leitfaden#Übungsaufgabe|1.3 Leitfaden – Übungsaufgabe]]). Baut auf [[Umsetzungsstand (DB-Erweiterung)|Umsetzungsstand (DB-Erweiterung)]] auf.

Theorie: [[1.3 Testing mit PyTest - Vertiefung|Vertiefung]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.3 Testing mit PyTest/Anki Karteikarten|Anki Karteikarten]] · [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.3 Testing mit PyTest/Q&A/Weiterführende Stichpunkte.md|Q&A-Register]] · Plan: [[Test-Suite Plan & Syntax-Tutorial|Test-Suite Plan & Syntax-Tutorial]] (13 Blöcke, alle abgearbeitet)

Projekt: `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/PyTest` (Kopie des DB/ORM-Standes)

## Endstand

Die Sammeldatei `tests/test_pricing.py` (5 Tests) wurde aufgelöst; die Teststruktur spiegelt jetzt die Module:

| Datei | Tests | Eingeführte Techniken |
|---|---|---|
| `tests/test_domain.py` | 23 | `assert`, `approx`, `raises`, `FrozenInstanceError`, Ordering |
| `tests/test_builder.py` | 9 | Fixture, `caplog` |
| `tests/test_recommendations.py` | 15 | `parametrize`, `monkeypatch`, `caplog` |
| `tests/test_app.py` | 6 | `Mock` (Stub + Mock), `side_effect`, `cast` |
| **Gesamt** | **53** | ruff + mypy `strict` grün |

**Branch-Coverage im Kern: 100 %** (`domain.py`, `core/`, `ports.py` — 160 Statements, 28 Branches, 0 Misses), gemessen mit `pytest --cov=pricing --cov-branch --cov-report=term-missing`. Gesamtprojekt: 35 %.

> [!important] Die 35 % sind die *unehrlichere* Zahl
> Der Rest sind die Adapter (`db.py`, `ingest.py`, `orm.py`, `sources.py`, `writers.py`, `main.py`) mit 0 %. Sie fassen echte Infrastruktur an (PostgreSQL, Dateisystem) und bräuchten **Integrationstests**, die laut Leitfaden Out of Scope sind. „35 %" klingt nach einer schlecht getesteten Anwendung — tatsächlich ist jede Zeile und jeder Zweig der Geschäftslogik abgedeckt. Die aussagekräftige Kennzahl ist die **Kern**-Coverage, nicht der Projektschnitt.

## Wichtigste Entscheidungen

- **Testdateien spiegeln Module, nicht Features** — eine Sammeldatei skaliert nicht; bei einem Fehlschlag will man sofort wissen, welches Modul betroffen ist.
- **Nur die I/O-Ränder mocken.** Im App-Test blieben `builder` und `engine` **echt** — reine, schnelle Logik mockt man nicht. Dadurch läuft im Orchestrierungs-Test die echte Rechenkette mit, statt nur die Verdrahtung zu prüfen.
- **Stub für Abfragen, Mock für Kommandos.** In `test_app.py` ist `source` ein **Stub** (liefert Daten, geprüft wird das Ergebnis), `writer` ein **Mock** (geprüft wird die Interaktion). Damit wurde die Lehrbuchregel real anwendbar statt nur zitierbar.
- **Coverage als Landkarte, nicht als Zielzahl** — der Wert des Reports liegt in der `missing`-Spalte. Eine Schwelle würde, wenn überhaupt, auf den **Kern** gehören, nicht aufs Gesamtprojekt.
- **`cast(DataSource, mock)` statt `type: ignore`** — sagt mypy präzise „behandle das als dieses Protocol", ohne alle anderen Fehler der Zeile zu unterdrücken.

## Gelernte Lektionen

- **Ein roter Test ist meist ein Erfolg, kein Defekt.** Praktisch jeder Fehlschlag lag im *Test*, nicht im Code — und deckte eine echte Modell-Unschärfe auf: `days_until_travel` (Reisetag vs. Bezugspunkt vertauscht) und `occupancy == 1.0` (dass `total_capacity` die gebuchten Einheiten *einschließt*, „ausgebucht" also `bookable + optional == 0` bedeutet). Tests zwingen zur Präzision über das eigene Domänenmodell.
- **Erwartete Werte müssen von Hand nachrechenbar sein.** Ein Testfall mit `occupancy = 3/13` war wertlos, weil das erwartete Ergebnis nur noch aus dem Code ablesbar gewesen wäre — „testet der Code, was der Code tut". Eingabewerte so wählen, dass glatte Zwischenwerte entstehen (`1/4`, `1/2`, `1/1`).
- **Format ≠ Wert bei `Decimal`.** „Hat 6 Nachkommastellen" lässt sich **nicht** über `==` prüfen (`Decimal("0") == Decimal("0.000000")` ist `True`). Die Skala steckt in `.as_tuple().exponent == -6`.
- **`parametrize` statt Schleife im Test.** Von drei Formel-Fällen war genau einer rot — die anderen liefen trotzdem durch und die Fehlermeldung zeigte punktgenau die kaputte Kombination. Eine Schleife wäre beim ersten Fehler abgebrochen.
- **„Patch where it's used".** `date` wurde im Modul `recommendations` gepatcht, nicht global — weil `from datetime import date` eine *eigene Bindung im importierenden Modul* anlegt.
- **Fake-Klassen müssen nicht erben.** `class FakeDate(date)` provozierte einen mypy-`[override]`-Fehler (die Stubs deklarieren `today() -> Self`). Eine schlichte Klasse **ohne** Vererbung, die nur `today()` anbietet, löst das eleganter als ein `type: ignore` — sie muss nur haben, was der Aufrufer tatsächlich benutzt.
- **Manche Aussagen sind nur per Mock prüfbar.** „`get_max_recommendation()` schreibt *nicht*" ist am Ergebnis nicht ablesbar — nur ein Mock kann die **Abwesenheit** einer Interaktion bezeugen (`assert_not_called()`).
- **Ein Test mit nur einem Element beweist kein Maximum.** `assert result.product_id == "P1"` bei genau einer Recommendation hätte auch für `recs[0]` oder `min()` gehalten. Erst zwei Einheiten mit unterschiedlichem `pct` machen die Aussage belastbar.
- **Fixture vs. Hilfsfunktion, ehrlich betrachtet:** Für reines Setup ohne Aufräumen ist eine Fixture *Idiom, keine Notwendigkeit*. Fixtures gewinnen erst bei **Teardown (`yield`)**, **Scope**, **Komposition** und **`conftest.py`-Sharing**. Der eigentlich wertvolle Trick in `test_app.py` ist die `AppHarness`-Dataclass, die die Doubles mit **herausgibt**, statt sie zu verstecken — nur dadurch kann ein einzelner Test den Stub umkonfigurieren.
- **Was der Typ garantiert, muss der Test nicht prüfen.** `assert result is None` bei einer `-> None`-Methode meldet mypy als `[func-returns-value]`. Übrig bleibt die eigentliche Aussage: der Seiteneffekt.
