Praktische Python-Lektionen, die beim Umsetzen der [[Ursprüngliche Aufgabe|OOP-Übungsaufgabe]] neu waren oder sich als Stolperfalle herausgestellt haben. Ergänzt die konzeptionelle [[1.1 OOP - Vertiefung|1]] um die *handwerkliche* Seite (Syntax, Sprach-Eigenheiten, Tooling). Wächst über die Kapitel hinweg. Aktueller Projektstand: [[Umsetzungsstand (OOP-Lösung)|Umsetzungsstand (OOP-Lösung)]].

*Begonnen: 2026-07-07 (OOP-Kapitel)*

# Klassen & Objekte

## Typannotation ist **keine** Zuweisung

`self._x: int` im `__init__` legt **kein Attribut an** — es ist nur eine Annotation ohne Wert. Das Attribut existiert danach nicht, und ein späterer Zugriff wirft `AttributeError`.

```python
def __init__(self, x: int):
    self._x: int      # ❌ tut zur Laufzeit NICHTS
    self._x = x       # ✅ so wird das Attribut tatsächlich gesetzt
```

Merksatz: **Ein Attribut entsteht erst durch eine Zuweisung (=), nie durch einen Doppelpunkt allein.**

## `@property` für abgeleitete & read-only Werte

Alles, was sich aus vorhandenem Zustand *berechnen* lässt, gehört als `@property`, nicht als gespeicherte Variable — dann kann es nie veralten (eine Quelle der Wahrheit). Eine Property **ohne Setter** ist automatisch read-only:

```python
@property
def total_capacity(self) -> int:
    return self._booked + self._bookable + self._optional   # bei jedem Zugriff frisch

# unit.total_capacity = 5   -> AttributeError (kein Setter)
```

Anwendung „echte Kapselung": Zustand privat (`_x`) speichern, nach außen nur einen kontrollierten Lese-Getter geben. Faustregel: **read-only-Property nur, wenn es (a) eine validierte Invariante zu schützen gibt oder (b) abgeleitete Werte davon abhängen.** Reine, regellose Felder dürfen public bleiben. Details: [[1.1 OOP - Vertiefung#Kapselung|Vertiefung – Kapselung]].

## Keyword-only-Argumente mit `*`

Ein nacktes `*` in der Signatur zwingt alle folgenden Argumente dazu, benannt übergeben zu werden. Sehr wertvoll bei **mehreren Argumenten gleichen Typs**, die man sonst leicht vertauscht:

```python
def __init__(self, product_id: str, calendar_date: date, *,
             booked_units: int, bookable_units: int, optional_units: int):
    ...

PriceableUnit("P1", d, 5, 3, 2)                       # ❌ TypeError
PriceableUnit("P1", d, booked_units=5, bookable_units=3, optional_units=2)  # ✅
```

# Dataclasses

Grundlagen (autogeneriertes `__init__`/`__repr__`/`__eq__`) siehe [[1.1 OOP - Vertiefung#dunder Methoden & Dataclasses|Vertiefung]]. Hier die praktischen Feinheiten aus der Umsetzung:

## `__post_init__` für Validierung

`@dataclass` generiert das `__init__` — eigene Validierung hängt man in `__post_init__` ein (läuft direkt nach dem generierten Konstruktor):

```python
@dataclass
class Booking:
    product_id: str
    nights: int
    def __post_init__(self) -> None:
        if self.nights < 1:
            raise ValueError(f"mind. 1 Nacht nötig (got {self.nights})")
```

## `frozen=True` — unveränderlich **und** hashbar

Macht Instanzen read-only (`FrozenInstanceError` bei Zuweisung) und erzeugt automatisch `__hash__` (nutzbar als Dict-Key / in `set`). Ideal für **Value Objects / Referenzdaten**, die sich nie ändern sollen.

## `order=True` vergleicht **alle** Felder als Tupel — die Falle

`@dataclass(order=True)` erzeugt `__lt__` etc., die **alle Felder in Definitionsreihenfolge** vergleichen. Steht ein ungewolltes Feld vorne, sortiert `max()`/`sorted()` danach:

```python
@dataclass(order=True)
class PriceRecommendation:
    product_id: str                      # ❌ max() würde primär hiernach sortieren!
    recommended_adjustment_pct: Decimal
```

Lösung: alle Nicht-Schlüsselfelder mit `field(compare=False)` vom Vergleich ausnehmen:

```python
from dataclasses import dataclass, field

@dataclass(order=True)
class PriceRecommendation:
    product_id: str = field(compare=False)
    recommended_adjustment_pct: Decimal          # das EINZIGE Vergleichsfeld
    occupancy: float = field(compare=False)
```

Wichtig: `field(compare=False)` **ohne** `default=`/`default_factory=` gibt **keinen** Standardwert — das Feld bleibt Pflicht, und die Reihenfolge löst *keinen* „non-default argument follows default argument"-Fehler aus. (Alternative ganz ohne `order`: `max(recs, key=lambda r: r.recommended_adjustment_pct)` — expliziter, keine Stolperfalle.)

# Standard-Library

## `date` / `timedelta`-Arithmetik

`date - date` ergibt ein `timedelta`; `.days` ist ein `int`. Für „N Tage weiter" ein `timedelta` addieren:

```python
from datetime import date, timedelta
(travel_date - reference_date).days                       # Differenz in Tagen (int)
[start + timedelta(days=n) for n in range(nights)]        # start … start+nights-1
```

## `Decimal` und `float` **nicht** mischen

Arithmetik zwischen `Decimal` und `float` wirft `TypeError`. Wenn Geldbeträge als `Decimal` geführt werden (keine Rundungsfehler), aber Zwischenwerte `float` sind: entweder komplett in `float` rechnen und am Ende **einmal** zu `Decimal` wandeln, oder durchgängig `Decimal`.

```python
Decimal("15") * 0.7          # ❌ TypeError
Decimal("15") * Decimal("0.7")   # ✅
```

## `logging.warning` vs. `warnings.warn`

- `warnings.warn(...)` → an **Entwickler**, für **Code-Probleme** (veraltete API, verdächtige Nutzung).
- `logging.warning(...)` → an den **Betrieb**, für **Laufzeit-/Daten-Ereignisse** („Zeile X ist auffällig").

Daten-Qualitäts-Warnungen also über `logging`. Best Practice: **aggregieren** (eine Zeile „N Fälle …") statt pro Objekt zu spammen — und den Log-Seiteneffekt an der zuständigen Schicht auslösen, nicht tief im Wertobjekt-Konstruktor.

# Pakete & Projektstruktur

Ausführliche Einordnung im [[Umsetzungsstand (OOP-Lösung)|Umsetzungsstand]]; die Kernbegriffe:

- **Modul** = eine `.py`-Datei. **Paket** = ein Ordner. Import-Pfad bildet die Ordnerstruktur mit Punkten ab (`pricing.domain` ↔ `pricing/domain.py`).
- **`__init__.py`** macht einen Ordner zum Paket und läuft beim ersten Import („Eingangstür"). Am Anfang leer lassen; später optional als „Schaufenster" mit Re-Exports / `__all__`.
- **`src/`-Layout** (`src/pricing/…`): zwingt dazu, gegen das *installierte* Paket zu arbeiten statt gegen lose Dateien — verhindert „lief nur, weil ich im richtigen Ordner stand".
- **`pip install -e ".[dev]"`** installiert das eigene Paket *editierbar* (nur ein Verweis auf `src/`, keine Kopie) — nötig, damit `import pricing` von überall funktioniert. Läuft im **venv**, berührt das System-Python nicht. *(Ab Kapitel 1.5 übernimmt das **uv**: `uv sync` legt venv an und installiert das Projekt editierbar mit — siehe [[uv - Paket- und Umgebungsverwaltung|uv – Paket- und Umgebungsverwaltung]].)*
- **Ein `.venv` ist nicht portabel.** Der Editable-Install merkt sich einen absoluten Pfad, und jedes Skript in `.venv/bin/` trägt den Interpreter des Originals im Shebang. Einen Projektordner also nie **mit** `.venv` kopieren — Umgebung stattdessen neu erzeugen.
- **`python -m pricing`** startet ein Paket über dessen `__main__.py` (sauberer als eine Datei direkt aufzurufen, weil Imports korrekt aufgelöst werden).

# Design-Muster, die hängengeblieben sind

- **Prädikat in der Domain, Policy beim Aufrufer.** Das Objekt liefert nur die *Regel* (`has_negative_available_capacity`), der Aufrufer entscheidet die *Reaktion* (loggen/zählen/ignorieren). Hält die Domain seiteneffektfrei und testbar; die Reaktion ist austauschbar (Open/Closed).
- **Validieren: „unmöglich" vs. „ungewöhnlich".** Nur echte Invarianten hart ablehnen (`ValueError`); ungewöhnliche-aber-mögliche Realwerte durchlassen und weiter oben behandeln.
- **Format prüft die Grenze, Invarianten prüft die Domain.** Parsing/Format-Checks in die I/O-Schicht (`sources`), fachliche Invarianten in die Domänen-Konstruktoren.
- **`.vscode`-Einstellungsdatei muss `settings.json` heißen** (Plural, mit „s") — sonst werden die Editor-Settings still ignoriert.
