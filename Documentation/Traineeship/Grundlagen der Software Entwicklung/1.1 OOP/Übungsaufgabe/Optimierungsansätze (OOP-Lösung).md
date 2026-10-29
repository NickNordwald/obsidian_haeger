---
tags:
  - documentation
---

Vertiefungsnotiz zur [[Umsetzungsstand (OOP-Lösung)|OOP-Lösung]]: **Wie hätte man den Code schneller/anders machen können?** Gesammelt aus der Reflexions-Session. Schwerpunkte: JIT-Compilation (PyPy & mypyc — will ich später ausprobieren), Memoization und ihre Fallstricke, sowie OOP vs. vektorisierte Verarbeitung. Praktische Python-Lektionen: [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md|Python-Lernnotizen]].

> [!info] Kontext
> Die reine OOP-Lösung baut pro Datenzeile ein Python-Objekt (`PriceableUnit`, `PriceRecommendation`) — ~1,33 Mio. Stück. Das ist **klar und testbar**, aber inhärent langsamer als die vektorisierte Ursprungslösung (Polars). Diese Notiz sammelt die Optionen entlang der Achse **Klarheit ↔ Geschwindigkeit**. Algorithmisch ist übrigens kaum was zu holen (der `Counter` ist schon O(1) pro Lookup) — es geht fast nur um **Konstanten-Faktoren**: Python-Overhead pro Objekt.

---

## 1. JIT-Compilation

„JIT" (Just-in-Time) heißt: Code wird **zur Laufzeit** in Maschinencode übersetzt, statt Zeile für Zeile interpretiert zu werden. Wichtig: Nicht jeder JIT passt zu jeder Art Code.

| Werkzeug | Passt zu dieser OOP-Lösung? | Kurz |
|---|---|---|
| **Numba** (`@njit`) | ❌ | Kompiliert nur numerische Arrays/Skalare. Kein Support für Dataclasses, `Decimal`, `date`, dict-of-tuple, `csv`. Man müsste alles vorher zu Zahlen-Arrays reduzieren — das ist dann schon Vektorisierung (Abschnitt 3). |
| **PyPy** | ✅ | Alternativer Interpreter mit Tracing-JIT. Beschleunigt bestehende OOP-Schleifen **ohne Code-Änderung**. |
| **mypyc** | ✅✅ | Kompiliert **typannotierten** Python zu einer C-Extension. Perfekter Fit, weil dieser Code bereits strict-mypy-sauber ist + `py.typed` hat. |

### 1a. PyPy — der „mein Python schneller, ohne Rewrite"-Weg

**Was es ist:** Eine *eigene Python-Implementierung* (nicht CPython), die dieselbe Sprache spricht. Man startet sein bestehendes `.py`-Programm einfach mit `pypy3` statt `python3`.

**Wie es beschleunigt (Tracing-JIT):** PyPy beobachtet das laufende Programm und erkennt **„heiße" Schleifen** (die, die oft laufen — z. B. der `build`-Loop über 1,33 Mio. Zeilen). Diese heißen Pfade übersetzt es zur Laufzeit in optimierten Maschinencode und **spezialisiert** ihn auf die tatsächlich gesehenen Typen (z. B. „hier ist `x` immer ein `int`"). Dadurch fällt der Interpreter-Overhead (Bytecode-Dispatch, Typprüfung pro Schritt) für diese Schleifen weg. Typische Beschleunigung für schleifen­lastigen reinen Python-Code: oft **mehrfach** (3–10×), gelegentlich mehr.

**Wie ausprobieren:**
```bash
# PyPy installieren (z. B. via pyenv, Paketmanager oder pypy.org)
pypy3 -m venv .venv-pypy
source .venv-pypy/bin/activate
pypy3 -m pip install -e ".[dev]"
pypy3 -m pricing.main          # keine Code-Änderung nötig
```

**Fallstricke:**
- **C-Extension-Kompatibilität:** PyPy emuliert die CPython-C-API (`cpyext`), aber *langsamer*. Reiner Python-Code profitiert am meisten. Dieses Projekt nutzt nur die Stdlib (`csv`, `decimal`, `dataclasses`) — meist unproblematisch, aber `decimal`/`csv` haben in CPython C-Beschleuniger, die unter PyPy anders (teils langsamer) sein können.
- **Warm-up:** Der JIT muss eine Schleife erst „oft genug" sehen, bevor er sie kompiliert. **Kurze** Skripte profitieren kaum; **lange Batch-Jobs** (wie dieser) sehr wohl.
- **Speicher:** PyPy braucht tendenziell mehr RAM.
- Nicht jede Bibliothek läuft (v. a. schwere C-Extensions wie manche ML-Libs).

**Wann:** Langlaufender, schleifenlastiger *reiner* Python-Code. Genau der Batch-Job-Fall hier.

### 1b. mypyc — der „meine Typen zahlen sich aus"-Weg

**Was es ist:** Ein **Ahead-of-Time-Compiler** (Teil des mypy-Projekts), der typannotierte Python-*Module* in **C-Extensions** übersetzt. Das Programm bleibt normaler Python-Code; beim Import wird aber die kompilierte `.so`-Version geladen und läuft schneller. Kein JIT im engeren Sinn (nicht zur Laufzeit) — man kompiliert **einmal**, läuft dann schnell.

**Wie es beschleunigt:** mypyc nutzt deine **Typannotationen**, um effizientes C zu erzeugen:
- Annotierte Klassen werden zu **„native classes"** mit implizitem `__slots__` → kein `__dict__` pro Objekt, direkter Feldzugriff statt Dict-Lookup.
- Wo der Typ statisch bekannt ist, entfällt „Boxing/Unboxing" (das ständige Ein-/Auspacken von Python-Objekten).
- Je **strenger/vollständiger** die Typen, desto mehr kann es optimieren. Typische Beschleunigung: **1,5–4×** auf objektlastigem Code — mypy selbst ist mit mypyc kompiliert und dadurch ~4× schneller.

**Warum es *besonders* zu diesem Projekt passt:** Der Code ist bereits **strict-mypy-sauber** und hat den **`py.typed`-Marker**. Genau das braucht mypyc. Die ganze Typisierungsarbeit zahlt hier direkt aus — und der lesbare OOP-Aufbau bleibt **1:1 erhalten**.

**Wie ausprobieren:**
```bash
# mypy ist schon installiert; einen C-Compiler braucht es (gcc/clang)
mypyc src/pricing/domain.py src/pricing/builder.py   # baut .so-Extensions
# danach nutzt "import pricing" automatisch die kompilierten Module
```
(Für größere Projekte konfiguriert man das über `setup.py`/Build-Backend, damit gezielt Module kompiliert werden.)

**Fallstricke:**
- Braucht eine **C-Compiler-Toolchain**.
- Sehr dynamische Python-Tricks (Monkey-Patching, exotische Metaklassen, `__dict__`-Jonglieren) werden nicht/langsam unterstützt — der saubere Dataclass-/`Protocol`-Stil hier ist aber ideal geeignet.
- Es ist ein **Build-Schritt** (AOT), kein transparenter Laufzeit-JIT wie PyPy.

**Wann:** Gut typisierter, „eher statischer" Python-Code, der nativen Speed will, ohne zu Arrays umgeschrieben zu werden. Behält das OOP-Design — wohl der eleganteste Fit für genau diese Lösung.

### PyPy vs. mypyc — die Kurzfassung

| | PyPy | mypyc |
|---|---|---|
| Mechanismus | Laufzeit-JIT (Tracing) | AOT-Kompilierung zu C-Extension |
| Ausführung | eigener Interpreter (`pypy3`) | normaler CPython lädt `.so` |
| Code-Änderung | keine | keine (aber Typen helfen enorm) |
| Voraussetzung | — | gute Typannotationen (hier ✅) |
| Stärke | schleifenlastiger reiner Python | gut typisierter, statischer OOP-Code |
| Haken | C-Ext-Kompatibilität, Warm-up | Build-Schritt, C-Compiler nötig |

---

## 2. Memoization / Lazy Caching

**Grundidee:** Das Ergebnis einer **teuren, deterministischen** Berechnung merken, damit wiederholte Aufrufe es **wiederverwenden** statt neu zu rechnen. Formen:
- Funktions-Ebene: `@functools.lru_cache` / `@functools.cache`
- Attribut-Ebene: `@functools.cached_property`
- manuell: `if self._x is None: self._x = ...; return self._x`

Angewandt auf die `PricingApp` (löst zugleich die Doppelberechnung):
```python
def _compute(self) -> list[PriceRecommendation]:
    if self._recommendations is None:          # nur beim ersten Mal rechnen
        capacity = self._source.load_capacity()
        bookings = self._source.load_bookings()
        units = self._builder.build(capacity, bookings)
        self._recommendations = self._engine.calculate_all(units)
    return self._recommendations               # danach: gecachtes Ergebnis
```

**Kern-Annahme:** *Gleiche Eingaben → gleiches Ergebnis, und die Eingaben ändern sich zwischen den Aufrufen nicht* (referentielle Transparenz). Genau diese Annahme ist die Quelle **aller** folgenden Nachteile.

### Die konkreten Nachteile (das Spannende)

1. **Staleness / Cache-Invalidierung.** *„There are only two hard things in Computer Science: cache invalidation and naming things."* Ändern sich die Eingaben (neuer Tages-CSV-Drop, editierte Daten), liefert der Cache **still veraltete** Ergebnisse. Man braucht eine Invalidierungs-Strategie (TTL, Versions-Key, explizites `refresh()`) — und die *richtig* hinzubekommen ist wirklich schwer.

2. **Verstecktes Einfrieren von externem Zustand / Zeit.** Hängt die Berechnung an `date.today()` oder am Dateisystem, friert der Cache einen **Schnappschuss** davon ein. Konkret hier: das `reference_date`-Default (`today()`) — beim ersten Aufruf eingefroren, rechnet eine langlebige App danach ewig mit dem *alten* „heute".

3. **Speicherkosten.** Der Cache hält Ergebnisse über die gesamte Objekt-/Prozess-Lebensdauer **am Leben**. Hier: ~188k `PriceRecommendation` bleiben im RAM. Bei `lru_cache` auf einer Funktion mit vielen verschiedenen Argumenten kann der Cache **unbegrenzt wachsen** → faktisch ein Memory-Leak.

4. **Zustand statt Zustandslosigkeit.** Aus dem sauberen, *zustandslosen* Orchestrator wird ein *zustandsbehaftetes* Objekt. Schwerer zu verstehen, schwerer zu testen (**reihenfolge­abhängig**: erster Aufruf rechnet, zweiter nimmt Cache — Tests müssen das berücksichtigen) und **nicht thread-safe** (zwei Threads rechnen gleichzeitig → Race/Doppelberechnung, außer man sperrt mit einem Lock).

5. **Caching + Mutabilität = Bug-Magnet.** Cacht man ein **veränderliches** Ergebnis (eine `list`), und ein Aufrufer mutiert es (`recs.append(...)`), sieht der nächste „Cache-Treffer" die Mutation. Der Cache ist dann korrumpiert. (Genau hier relevant: die `recs`-Liste ist mutierbar.)

6. **Überraschung / Nicht-Lokalität.** Ein Aufrufer erwartet nicht, dass ein *identischer* zweiter Aufruf sich anders verhält (plötzlich schnell — oder stale). Verletzt das *Principle of Least Astonishment*.

7. **Debugging-Hölle.** „Funktioniert beim zweiten Mal, aber nicht beim ersten" oder „liefert alte Daten" — solche Bugs sind schwer zu finden, weil der Cache **unsichtbar** ist.

8. **Schlüssel & Gleichheit** (bei Funktions-Memoization). Argumente müssen **hashbar** sein, und ihre Gleichheit muss „gleiche Berechnung" korrekt abbilden. Subtile Bugs bei unhashbaren Argumenten oder wenn `__eq__` nicht der semantischen Identität entspricht.

### Sauberere Alternative

Für einen Batch-Job: **explizit statt versteckt** — einmal rechnen, Ergebnis herumreichen:
```python
def run(self) -> PriceRecommendation | None:
    recs = self._compute()          # einmal
    self._writer.write(recs)
    return self._engine.get_max(recs)
```
**Faustregel:** *„Einmal rechnen und das Ergebnis herumreichen" schlägt „einen Cache im Objekt verstecken"* — außer die App ist wirklich langlebig mit stabilen Eingaben, dann ist Memoization das richtige Werkzeug (mit **expliziter** Invalidierung).

---

## 3. Geht OOP + Vektorisierung zusammen? (ausführlich)

### 3a. Warum Vektorisierung schneller ist, obwohl auch geloopt wird

Der Loop verschwindet nicht — er **wandert**:

- **Reine Python-Schleife:** Für jede der 1,33 Mio. Iterationen macht der CPython-Interpreter Bytecode-Dispatch, Typprüfung, Attribut-Lookup, Methodenaufruf, Objekt-Erzeugung/-Zerstörung, Reference-Counting — **interpretiert, pro Element**.
- **Vektorisiert (NumPy/Polars):** Der Loop läuft in **kompiliertem C/Rust** über zusammenhängende, typisierte Arrays. *Ein* Python-Aufruf (`a - 0.5`) löst *eine* C-Schleife über das ganze Array aus — kein Interpreter-Overhead pro Element, oft **SIMD** (CPU verarbeitet mehrere Werte pro Takt).

> [!quote] Merksatz
> Vektorisierung **entfernt den Loop nicht** — sie schiebt ihn runter nach C und wirft den Objekt-Overhead pro Zeile weg.

**Gemessen** (nur die Arithmetik der Empfehlungs-Formel, 1,3 Mio. Werte):
```
Python (nur Arithmetik):   117 ms
NumPy vektorisiert     :    22 ms   -> 5x schneller
```
Das ist die **Untergrenze**: die echte `calculate_all` baut zusätzlich pro Zeile ein `PriceRecommendation`-Objekt, ruft `days_until_travel()`, konvertiert zu `Decimal` — alles reiner Python-Overhead, den NumPy spart. Für die *ganze* Pipeline ist der Faktor also deutlich größer.

### 3b. Der Kern: zeilenweise Objekte vs. spaltenweise Arrays (AoS vs. SoA)

Das ist der Begriff, der vorhin unklar war. „AoS" und „SoA" beschreiben **zwei Arten, dieselbe Tabelle im Speicher abzulegen**.

Stell dir diese Mini-Tabelle vor (3 Zeilen):

| product_id | booked | bookable | optional |
|---|---|---|---|
| P1 | 3 | 5 | 0 |
| P2 | 1 | 2 | 0 |
| P3 | 0 | 4 | 1 |

**Array of Structs (AoS) = zeilenweise Objekte — das machst du gerade.**
`list[PriceableUnit]` ist eine Liste, in der jedes Element ein **komplettes Objekt** ist, das *alle Felder einer Zeile* zusammen bündelt:
```
[ PriceableUnit(product_id="P1", booked=3, bookable=5, optional=0),   # ein "Struct"
  PriceableUnit(product_id="P2", booked=1, bookable=2, optional=0),
  PriceableUnit(product_id="P3", booked=0, bookable=4, optional=1) ]
```
Im Speicher: Die Liste hält **Zeiger** auf 1,33 Mio. einzelne, über den Heap **verstreute** Objekte. Alle Felder *einer* Zeile liegen zusammen; verschiedene Zeilen liegen weit auseinander.
→ Bild: ein **Stapel Karteikarten**, jede Karte trägt alle Spalten *eines* Eintrags.

**Struct of Arrays (SoA) = spaltenweise Arrays — was Vektorisierung braucht.**
Statt Objekten pro Zeile: **ein Array pro Feld**, das den Wert dieses Felds für *alle* Zeilen enthält:
```
product_ids = ["P1", "P2", "P3"]     # alle IDs
booked      = [3, 1, 0]              # alle booked-Werte, zusammenhängend
bookable    = [5, 2, 4]
optional    = [0, 0, 1]
```
→ Bild: die Tabelle **spaltenweise** gespeichert — pro Spalte ein langer, zusammenhängender Streifen.

**Warum SoA für Massen-Mathe schneller ist:**
1. **Zusammenhängender Speicher pro Spalte** → rechnest du über eine Spalte, streamt die CPU durch *zusammenhängenden* Speicher (cache-freundlich), kein Pointer-Chasing zu verstreuten Objekten.
2. **Keine Python-Objekte pro Zeile** → die Werte sind rohe Maschinenzahlen (int64/float64), dicht gepackt, statt geboxter Python-Objekte mit Header + Refcount.
3. **Eine Operation über das ganze Array** → `booked + bookable + optional` ist *ein* Ausdruck, der als *eine* C-Schleife alle Zeilen addiert (oft mit SIMD).

`total_capacity` für alle Zeilen wird damit zu **einer** Array-Addition:
```python
total = booked + bookable + optional     # -> [8, 3, 5], eine C-Schleife
occupancy = booked / total               # -> [0.375, 0.333, 0.0], vektorisiert
```

Die Umwandlung **AoS → SoA** ist also ein „Kippen" der Daten von Zeilen-Objekten zu Spalten-Arrays. Und genau das **ist** ein DataFrame: das DataFrame-Objekt (ein „Struct") hält die Spalten (Arrays) = **Struct of Arrays**.

### 3c. Kann OOP dann überhaupt vektorisieren?

- Das Hindernis ist **nicht „OOP" an sich**, sondern das **AoS-Layout** (ein Objekt pro Zeile). Sobald du `list[PriceableUnit]` hast, hast du den Preis bezahlt.
- Vektorisierung braucht **SoA**.
- **Aber OOP bleibt möglich** — nur auf **Sammlungs-Ebene** statt Zeilen-Ebene. Eine Klasse *ist* die Kollektion und hält Spalten-Arrays, mit vektorisierten Methoden. Kapselung wandert vom Einzel-Objekt (`PriceableUnit`) hoch zur ganzen Sammlung. Ein `DataFrame` ist genau so ein Objekt.

Konkret: statt `PriceableUnit` (eine Zeile) mit `.occupancy` hättest du z. B.
```python
class PriceableUnitTable:                 # kapselt die ganze Sammlung (SoA)
    def __init__(self, booked, bookable, optional, ...):
        self._booked = np.asarray(booked)          # Spalten-Arrays
        self._bookable = np.asarray(bookable)
        self._optional = np.asarray(optional)

    def occupancy(self) -> np.ndarray:             # rechnet ALLE auf einmal
        total = self._booked + self._bookable + self._optional
        return self._booked / total
```
Du **verlierst** das Einzel-Zeilen-Objekt (und dessen per-Zeile-Invarianten/Kapselung), **behältst** aber eine objektorientierte API auf Kollektions-Ebene.

### 3d. Direkter Vergleich an zwei Stellen

**Die Formel (`calculate_all`):**
```python
# OOP (Objekt pro Zeile):
recs = []
for u in units:                                  # 1,3 Mio. Python-Iterationen
    if u.total_capacity == 0: continue
    a = u.occupancy
    d = u.days_until_travel(ref) / 300
    pct = 15.0 * (a - 0.5) * (0.5 + d)
    recs.append(PriceRecommendation(...))        # Objektbau pro Zeile
```
```python
# Vektorisiert (Spalten, Polars):
df.filter(pl.col("total_capacity") > 0).with_columns(
    (15.0 * (pl.col("occupancy") - 0.5) * (0.5 + pl.col("days")/300)).alias("pct")
)
```
Die Formel sieht gleich aus — aber `occupancy`, `days` sind ganze *Spalten*; ein Ausdruck rechnet alle 1,3 Mio. Ergebnisse in kompiliertem Code, ohne Python-Loop, ohne Zeilen-Objekte.

**Die `booked_units`-Aggregation (`Counter`):**
```python
# OOP:
booked = Counter()
for b in bookings:                               # 939k Iterationen in Python
    if b.is_firm:
        for night in b.occupied_nights():        # Listen-Alloc + innerer Loop
            booked[(b.product_id, night)] += 1
```
```python
# Vektorisiert (Polars — genau die alte Lösung!):
(bookings
 .filter(pl.col("status") == "Firm")
 .with_columns(pl.date_ranges(start, end).alias("night"))   # Nächte expandieren
 .explode("night")                                          # = occupied_nights()
 .group_by(["product_id", "night"]).agg(pl.len()))          # = Counter-Increment
```

> [!tip] Die Pointe
> Die **ursprüngliche Polars-Lösung war die vektorisierte Version** von genau dem, was in OOP nachgebaut wurde: `explode` = `occupied_nights()`, `group_by().agg(len)` = der `Counter`.

---

## 4. Randnotizen (kleinere Konstanten-Faktor-Gewinne)

- **`@dataclass(slots=True)` / `__slots__`** auf `PriceableUnit`: bei Mio. Objekten weniger Speicher (kein `__dict__`) + schnellerer Attributzugriff.
- **`occupied_nights()` als Generator** (`yield`): spart die Zwischenliste pro Buchung.
- **„Build only what you keep":** Der Builder baut *alle* 1,33 Mio. Units; die Engine wirft ~654k (Null-Kapazität) + Vergangenheit weg. Früher filtern (v. a. der Datums-Filter braucht den `Counter` nicht) spart massiv Objektbau — **koppelt** aber Engine-Filterlogik in den Builder (SoC-Bruch).
- **`csv.DictReader` vs. `csv.reader`:** DictReader baut ein Dict pro Zeile (Overhead); `reader` mit Index-Zugriff ist schneller, aber weniger lesbar.

---

## 5. Fazit — die Achse

Vektorisiert/kompiliert ist schnell — aber **Invarianten pro Zeile** (die `nights >= 1`-Validierung, die Kapselung) und **komplexe fachliche Einzelfall-Regeln** sind spaltenweise umständlich. OOP ist langsam bei Masse, aber jede `PriceableUnit` *ist* ein Domänenkonzept mit erzwungenen Invarianten und liest sich wie die Fachsprache.

**Für „schneller, ohne das OOP-Design aufzugeben":** zuerst **mypyc** probieren (passt dank Typisierung + `py.typed` ideal), alternativ **PyPy**. **Für echten Masse-Durchsatz:** zu einer **SoA/DataFrame-Struktur** wechseln (vektorisiert) — dann aber bewusst die Kapselung von der Zeile auf die Sammlung heben.
