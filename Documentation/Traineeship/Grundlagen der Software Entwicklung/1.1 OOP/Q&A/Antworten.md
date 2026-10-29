Antworten zu den eigenen Verständnisfragen aus [[Stichwörter und Fragen OOP|Stichwörter und Fragen OOP]]. Übergeordnete Notiz: [[1.1 OOP - Vertiefung|1]].

## ETL

ETL steht für **Extract, Transform, Load** und beschreibt den klassischen Prozess, mit dem Daten aus einer oder mehreren Quellen ausgelesen (*Extract*), in ein gewünschtes Format überführt bzw. bereinigt (*Transform*) und in ein Zielsystem geschrieben werden (*Load*) – typischerweise ein Data Warehouse oder eine Datenbank.

Als Gegenentwurf gibt es **ELT** (Extract, Load, Transform): Die Rohdaten werden zuerst ins Zielsystem geladen, die Transformation passiert erst danach dort (z. B. per SQL/dbt). Durch günstige, elastisch skalierende Cloud-Data-Warehouses (Snowflake, BigQuery, …) ist ELT heute sehr verbreitet.

Objektorientiert lässt sich eine ETL-Pipeline z. B. abbilden, indem jeder Schritt (Extractor, Transformer, Loader) eine eigene Klasse mit klar definierter Schnittstelle ist – das macht die Pipeline testbar und austauschbar (vgl. später *Loose Coupling*, *Dependency Injection*).

## Überschreiben von Methoden (Python)

Wenn eine Subklasse eine Methode mit demselben Namen definiert wie ihre Elternklasse, **überschreibt** sie deren Implementierung – zur Laufzeit wird immer die Version der konkreten (Sub-)Klasse verwendet. Das ist ein Kernbestandteil von Vererbung/Polymorphie, die wir als Nächstes ausführlich behandeln.

Mit `super().methode()` kannst du innerhalb der überschreibenden Methode trotzdem die Implementierung der Elternklasse aufrufen, statt sie komplett zu ersetzen – nützlich, um Verhalten zu *erweitern* statt neu zu schreiben:

```python
class Shape:
    def describe(self) -> str:
        return "irgendeine Form"

class Circle(Shape):
    def describe(self) -> str:
        base = super().describe()
        return f"{base}, genauer: ein Kreis"

print(Circle().describe())  # "irgendeine Form, genauer: ein Kreis"
```

## Operator Overloading

Python erlaubt es, das Verhalten eingebauter Operatoren (`+`, -, ==, <, `len()`, …) für eigene Klassen zu definieren – indem man die passenden Dunder-Methoden implementiert: `__add__` für `+`, `__eq__` für \=\=, `__lt__` für `<`, `__len__` für `len(obj)` usw.

```python
class Money:
    def __init__(self, amount: float):
        self.amount = amount

    def __add__(self, other: "Money") -> "Money":
        return Money(self.amount + other.amount)

    def __repr__(self) -> str:
        return f"Money({self.amount})"

print(Money(10) + Money(5))  # Money(15)
```

Sinnvoll, wenn es semantisch wirklich passt (zwei Preise addieren) – nicht missbrauchen, nur damit Code "schlau" aussieht, wenn eine normale, klar benannte Methode (`add_to(...)`) verständlicher wäre.

## Dunder Methoden

"Dunder" = *double underscore*. Gemeint sind Methoden wie `__init__`, `__repr__`, `__eq__`, `__len__`, `__iter__`, `__enter__`/`__exit__` usw. Das sind die Hooks, über die Python eingebaute Sprachfeatures (Objekterzeugung, String-Darstellung, Vergleiche, Iteration, `with`-Statements, Operatoren – siehe *Operator Overloading* oben) mit deinen eigenen Klassen verbindet.

Wichtig: Man ruft sie so gut wie nie direkt auf (`obj.__add__(other)`), sondern nutzt die dazugehörige Sprachsyntax (`obj + other`), die Python intern zur Dunder-Methode übersetzt.

Zwei, die man früh kennen sollte:
- `__repr__` – eindeutige, für Entwickler gedachte Darstellung (Debugging, REPL, Logs). Faustregel: sollte idealerweise gültiger Python-Code sein, um das Objekt zu re-konstruieren.
- `__str__` – nutzerfreundliche Darstellung, wird von `print()`/`str()` verwendet. Fehlt `__str__`, fällt Python auf `__repr__` zurück.

## Decorators & Libraries für Klassenprogrammierung „under the hood"

Ein Überblick der wichtigsten Decorators/Module rund um Klassen in Python:

- **`@staticmethod`** / **`@classmethod`** – siehe unten (*Klassenmethoden vs. Instanzmethoden vs. Static Methods*).
- **`@property`** – siehe unten (*Getter/Setter-Methoden in Python*).
- **`@dataclass`** (`dataclasses`-Modul) – generiert automatisch `__init__`, `__repr__`, `__eq__` (und optional `__lt__` etc.) aus einer Attribut-Deklaration, spart Boilerplate für "Datencontainer"-Klassen. Wird im Leitfaden als eigenes Stichwort noch vertieft.
- **`@abstractmethod`** (`abc`-Modul, `ABC`/`ABCMeta`) – markiert eine Methode als verpflichtend zu implementieren in Subklassen; die Basisklasse kann dann nicht direkt instanziiert werden. Zentral für *abstrakte Klassen* (kommt als eigenes Stichwort/Kontrollfrage noch dran).
- **`functools.total_ordering`** – man implementiert nur `__eq__` und eine Vergleichsmethode (z. B. `__lt__`), der Decorator ergänzt automatisch die restlichen (`__le__`, `__gt__`, `__ge__`).
- **`functools.cached_property`** – wie `@property`, aber das Ergebnis wird nach dem ersten Zugriff pro Instanz zwischengespeichert (Cache) statt bei jedem Zugriff neu berechnet.
- **`typing.Protocol`** / **`typing.Generic`** – strukturelle Typisierung bzw. generische Klassen (siehe unten *Protocol aus typing*).
- **`attrs`** / **`pydantic`** (Drittanbieter-Libraries) – populäre Alternativen/Ergänzungen zu `dataclasses`, u. a. mit eingebauter Validierung. `pydantic` ist im Data-/Web-Umfeld (z. B. FastAPI) sehr verbreitet.

## Klassenmethoden vs. Instanzmethoden vs. Static Methods

Drei Arten von Methoden in einer Python-Klasse:

- **Instanzmethode** (Standard): erster Parameter `self`, braucht eine konkrete Instanz.
- **`@classmethod`**: erster Parameter `cls` (die Klasse selbst statt einer Instanz). Häufigster Anwendungsfall: alternative Konstruktoren (`from_json(cls, data)`), die trotzdem korrekt funktionieren, wenn später von der Klasse geerbt wird (da `cls` die tatsächliche, aufrufende Klasse ist – nicht hart codiert).
- **`@staticmethod`**: kein automatischer erster Parameter, verhält sich wie eine normale Funktion, die nur thematisch zur Klasse gehört (Namespace/Organisation), aber weder auf `self` noch auf `cls` zugreift.

```python
class PriceableUnit:
    vat_rate = 0.19

    def __init__(self, base_price: float):
        self.base_price = base_price

    def price_with_vat(self) -> float:          # Instanzmethode
        return self.base_price * (1 + self.vat_rate)

    @classmethod
    def from_dict(cls, data: dict) -> "PriceableUnit":  # alternativer Konstruktor
        return cls(data["base_price"])

    @staticmethod
    def is_valid_price(value: float) -> bool:    # reine Utility-Funktion
        return value > 0
```

## Getter/Setter-Methoden in Python

In Sprachen wie Java/C++ üblich: private Felder + explizite `getX()`/`setX()`-Methoden. In Python macht man das zugehörige Attribut stattdessen normal öffentlich zugänglich und nutzt bei Bedarf `@property`, um nachträglich Kontrolle (Validierung, berechnete Werte, Nebeneffekte) einzubauen – **ohne** die öffentliche Schnittstelle (`obj.attribut`) für alle bestehenden Aufrufer zu ändern.

```python
class PriceableUnit:
    def __init__(self, base_price: float):
        self._base_price = base_price

    @property
    def base_price(self) -> float:
        return self._base_price

    @base_price.setter
    def base_price(self, value: float) -> None:
        if value < 0:
            raise ValueError("Preis darf nicht negativ sein")
        self._base_price = value

unit = PriceableUnit(10)
unit.base_price = 15       # ruft den Setter auf, sieht aus wie normaler Attributzugriff
print(unit.base_price)     # ruft den Getter auf
```

Gutes Beispiel für *Kapselung*/*Information Hiding* (kommt als eigenes Stichwort noch ausführlich).

## Gegen gemeinsames Verhalten statt gegen konkrete Klassen programmieren

Gemeint ist das Prinzip **"Programmiere gegen eine Schnittstelle/ein gemeinsames Verhalten, nicht gegen eine konkrete Implementierung"** – verwandt mit dem *Dependency Inversion Principle* und mit Polymorphie. Statt Code zu schreiben, der explizit weiß "ich bekomme ein `CsvExporter`-Objekt", schreibt man Code, der nur ein gemeinsames Verhalten voraussetzt (z. B. "irgendetwas mit einer `export()`-Methode").

```python
class CsvExporter:
    def export(self, data): ...

class JsonExporter:
    def export(self, data): ...

def run_export(exporter, data) -> None:   # weiß nichts von CsvExporter/JsonExporter konkret
    exporter.export(data)

run_export(CsvExporter(), data)
run_export(JsonExporter(), data)          # austauschbar, ohne run_export anzufassen
```

Vorteil: Die konkrete Implementierung lässt sich austauschen (CSV → JSON → Datenbank), ohne den aufrufenden Code (den "Client Code") zu ändern. In Python funktioniert das dank *Duck Typing* sogar ohne formales Interface – beide Klassen müssen nur eine `export()`-Methode besitzen. Mehr Struktur/Typsicherheit dafür bietet `typing.Protocol` (siehe nächster Punkt).

## Protocol aus typing

`typing.Protocol` (seit Python 3.8) ermöglicht **strukturelle Typisierung** ("structural subtyping", ähnlich TypeScript-Interfaces): Eine Klasse muss ein `Protocol` nicht explizit erben, um als dessen Implementierung zu gelten – es reicht, wenn sie die geforderten Methoden/Attribute besitzt ("wenn's aussieht wie eine Ente und quakt wie eine Ente…"). Das kombiniert Pythons dynamisches Duck Typing (siehe vorheriger Punkt) mit statischer Typüberprüfung durch `mypy`.

```python
from typing import Protocol

class Exporter(Protocol):
    def export(self, data: list) -> None: ...

class CsvExporter:                # erbt NICHT von Exporter, erfüllt das Protocol trotzdem
    def export(self, data: list) -> None:
        print("exporting to csv")

def run_export(exporter: Exporter, data: list) -> None:
    exporter.export(data)

run_export(CsvExporter(), [])     # mypy akzeptiert das, obwohl keine Vererbung vorliegt
```

## SRP, hohe Kohäsion, Open/Closed

Drei zusammenhängende Designprinzipien, die im Leitfaden noch als eigene Stichworte ausführlich drankommen – hier schon mal kompakt vorab:

- **Single Responsibility Principle (SRP)**: Eine Klasse sollte genau einen Grund haben, sich zu ändern – also nur eine klar abgegrenzte Verantwortung tragen. Beispiel: Eine `PriceableUnit`-Klasse sollte Preise berechnen, aber nicht *zusätzlich* CSV-Dateien schreiben oder E-Mails verschicken.
- **Hohe Kohäsion (High Cohesion)**: Was innerhalb einer Klasse/eines Moduls zusammensteht, sollte auch inhaltlich zusammengehören. SRP ist im Grunde ein Weg, hohe Kohäsion zu erreichen.
- **Open/Closed Principle**: Klassen sollen offen für Erweiterung, aber geschlossen für Veränderung sein – neues Verhalten fügt man idealerweise durch neue Klassen (z. B. via Vererbung/Polymorphie) hinzu, statt bestehenden, funktionierenden Code anzufassen und damit das Risiko neuer Bugs einzugehen.

## ruff, mypy

Zwei Standard-Tools im modernen Python-Tooling-Stack:

- **`ruff`** – extrem schneller Linter (mittlerweile auch Formatter), geschrieben in Rust. Prüft Code auf Stilverstöße (u. a. PEP 8, siehe [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]]), unbenutzte Imports, häufige Fehlerquellen usw. Ersetzt in vielen Projekten mittlerweile die Kombination aus flake8, isort und black.
- **`mypy`** – statischer Typchecker für Python. Python selbst prüft Type Hints (`def f(x: int) -> str:`) zur Laufzeit nicht – `mypy` analysiert den Code, *ohne ihn auszuführen*, und meldet Typfehler schon vor der Ausführung. Ist nur wirksam, wenn Type Hints im Code konsequent genutzt werden.

Beide laufen typischerweise als Pre-Commit-Hook oder CI-Schritt, um Codequalität automatisiert durchzusetzen.

---

# ⬇️ Neue Antworten (Stand 2026-07-09)

*Ab hier beginnt die neu ergänzte Fragenrunde (name mangling bis Protocol vs. ABC).*

## Name Mangling (_name vs. __name)

Python kennt **keine** echte Zugriffskontrolle (kein `private` wie in Java). Stattdessen zwei Konventionen mit Unterschieden:

- **Ein Unterstrich `_name`**: reine **Konvention** — „das ist intern, fass es von außen bitte nicht an". Technisch nichts erzwungen; du *kannst* trotzdem drauf zugreifen. Tools respektieren es (z. B. importiert `from modul import *` keine `_namen`). So sind deine `_product_id`, `_resolve_csv` etc. gemeint: „nicht Teil der öffentlichen API".
- **Zwei führende Unterstriche `__name`** (ohne abschließende Dunder): löst **Name Mangling** aus. Python benennt `__name` beim Kompilieren des Klassenkörpers still in `_KlassenName__name` um.

```python
class Base:
    def __init__(self):
        self.__value = 1        # wird zu  _Base__value

class Sub(Base):
    def __init__(self):
        super().__init__()
        self.__value = 2        # wird zu  _Sub__value  → kollidiert NICHT mit Base

b = Base()
# b.__value          # ❌ AttributeError
b._Base__value       # ✅ 1  — zugreifbar, nur "versteckt"
```

Wichtig: Name Mangling dient **nicht** der Geheimhaltung, sondern der **Kollisionsvermeidung** — damit eine Subklasse ein gleichnamiges Attribut nicht versehentlich überschreibt. Für „nur intern" reicht der einfache Unterstrich; die doppelten braucht man selten.

## Private Methods in Python

Folgt direkt aus dem Punkt oben: Es gibt in Python **keine wirklich privaten Methoden**. Man signalisiert „privat" per Konvention mit führendem Unterstrich (`_resolve_csv` in deinem `sources.py` ist genau so eine „private" Hilfsmethode). Sie bleibt aufrufbar und auffindbar — der Unterstrich sagt nur: „Das ist Implementierungsdetail, verlass dich nicht drauf, es kann sich ändern." Wer `__methode` (Name Mangling) nutzt, macht den Zugriff von außen/aus Subklassen nur *umständlicher*, nicht unmöglich. Die eigentliche „Privatheit" ist in Python also eine Frage von **Disziplin und Dokumentation**, nicht von Sprach-Zwang.

## Monkey Patching, Metaklassen, dict-Jonglieren

Drei „dynamische Schlupflöcher" von Python — mächtig, aber in normalem Anwendungscode meist ein Warnsignal. Gut zu wissen, dass es sie gibt:

- **Monkey Patching**: Attribute/Methoden einer Klasse oder eines Moduls **zur Laufzeit** ersetzen oder ergänzen, *nach* deren Definition.
  ```python
  import some_module
  some_module.now = lambda: "2026-01-01"   # echte Funktion durch Fake ersetzt
  ```
  Legitim v. a. in **Tests** (`unittest.mock.patch` macht genau das kontrolliert). In Produktivcode heikel: das Verhalten steht dann nicht dort, wo die Klasse definiert ist → schwer nachvollziehbar.

- **Metaklassen**: die „Klasse einer Klasse". So wie ein Objekt eine Instanz seiner Klasse ist, ist eine Klasse eine Instanz ihrer *Metaklasse* (Standard: `type`). Eine eigene Metaklasse greift in die **Klassen-Erzeugung** ein (Attribute prüfen/injizieren, registrieren). Frameworks nutzen das (Djangos ORM, `ABCMeta` hinter `ABC`), im Alltag braucht man es fast nie — Faustregel von Tim Peters: „Wenn du dich fragst, ob du Metaklassen brauchst, brauchst du sie nicht."

- **dict-Jonglieren**: Attribute eines Objekts liegen in `obj.__dict__` (ein normales Dictionary). Man kann sie direkt manipulieren:
  ```python
  obj.__dict__["x"] = 5      # entspricht obj.x = 5
  ```
  „Jonglieren" = dieses `__dict__` (oder `globals()`/`vars()`) dynamisch umbauen. Sehr flexibel, aber intransparent; `@dataclass(slots=True)` (s. u.) schaltet dieses `__dict__` sogar bewusst ab.

Gemeinsamer Nenner: Pythons Dynamik erlaubt sehr viel. Für sauberen, wartbaren Code (dein Ziel im Leitfaden) nutzt man diese Werkzeuge **sparsam und bewusst**.

## Dunder-Methoden: bekannte, unbekanntere und __new__

Aufbauend auf dem bereits beantworteten Punkt *Dunder Methoden*. 

**Sollte man kennen:**
- `__init__` (Initialisierung), `__repr__` / `__str__` (Darstellung), `__eq__` / `__hash__` (Gleichheit & Hashing), `__lt__` & Co. (Ordnung → nutzt dein `max(recs)`), `__len__`, `__iter__` / `__next__` (Iteration, s. u.), `__getitem__` (`obj[key]`), `__contains__` (`in`), `__enter__` / `__exit__` (`with`-Block), `__call__` (Objekt wie Funktion aufrufbar).

**Eher unbekannt / fortgeschritten:**
- `__post_init__` (Dataclass-Nachbearbeitung — nutzt dein `Booking`!), `__slots__` (s. u.), `__set_name__`, `__init_subclass__`, `__getattr__` / `__setattr__` (Attributzugriff abfangen), `__format__` (für `f"{x:...}"`), `__index__`, und eben `__new__`.

**`__new__` im Detail:** Objekterzeugung läuft in Python **zweistufig**:
1. `__new__(cls, ...)` **erzeugt** die Instanz (allokiert das Objekt) und gibt sie zurück.
2. `__init__(self, ...)` **initialisiert** das bereits erzeugte Objekt (füllt Felder) und gibt *nichts* zurück.

```python
class Foo:
    def __new__(cls, *args):
        print("1. __new__ erzeugt das Objekt")
        return super().__new__(cls)
    def __init__(self, x):
        print("2. __init__ befüllt es")
        self.x = x
```

`__new__` überschreibt man **selten** — nur wenn man in die Erzeugung selbst eingreifen muss: **unveränderliche** Typen ableiten (von `int`/`str`/`tuple` erben, wo `__init__` zu spät käme), Singletons, oder Objekt-Caching. Für 99 % der Klassen reicht `__init__`.

## Generatoren und Iteratoren

- **Iterable**: alles, worüber man mit `for` laufen kann — hat eine `__iter__`-Methode (Listen, Tupel, Dicts, …).
- **Iterator**: das Objekt, das die Werte **einzeln nacheinander** herausgibt — hat `__next__` (liefert das nächste Element bzw. wirft `StopIteration` am Ende) und `__iter__`. Ein Iterator ist „verbraucht", wenn er durch ist.
- **Generator**: der bequemste Weg, einen Iterator zu bauen — eine Funktion mit `yield` (s. nächster Punkt) oder ein Generator-Ausdruck `(x for x in ...)`. Der große Vorteil: **Lazy Evaluation** — Werte werden erst berechnet, wenn sie gebraucht werden, es liegt nie die ganze Sequenz im Speicher.

```python
def erste_n_quadrate(n):
    for i in range(n):
        yield i * i           # produziert eins nach dem anderen, hält nicht alle im RAM

for q in erste_n_quadrate(1_000_000):
    ...                       # verarbeitet je eins, Speicher bleibt konstant
```

Bezug zu deinem Code: `csv.DictReader` ist ein **Iterator** — er liest die Datei **zeilenweise faul**, nicht alles auf einmal. Deine List-Comprehensions (`[... for row in reader]`) ziehen diesen Iterator dann bewusst in eine **vollständige Liste** im Speicher (weil du alle Zeilen brauchst).

## return vs. yield

- **`return`** beendet die Funktion und gibt **einmal** einen Wert zurück. Danach ist die Funktion fertig, ihr Zustand ist weg.
- **`yield`** macht die Funktion zu einem **Generator**: Sie gibt einen Wert heraus und **pausiert** an dieser Stelle (der komplette lokale Zustand bleibt eingefroren). Beim nächsten `next()` läuft sie **genau dort weiter**. So kann sie über die Zeit **viele** Werte liefern.

```python
def mit_return():
    return [1, 2, 3]      # baut die ganze Liste, gibt sie komplett zurück

def mit_yield():
    yield 1               # gibt 1, pausiert
    yield 2               # läuft weiter, gibt 2, pausiert
    yield 3               # gibt 3, dann StopIteration
```

Merksatz: `return` = „hier ist das fertige Ergebnis". `yield` = „hier ist der nächste Happen, ruf mich wieder auf". Nutzt man `yield`, um große/unendliche Datenströme speicherschonend zu verarbeiten.

## slots=True in @dataclass

Standardmäßig speichert jede Instanz ihre Attribute in einem **`__dict__`** (ein eigenes Dictionary pro Objekt). Das ist flexibel (man kann jederzeit neue Attribute anhängen), kostet aber **Speicher** und ist beim Attributzugriff minimal langsamer.

`@dataclass(slots=True)` (ab Python 3.10) generiert automatisch `__slots__` — eine **feste Liste erlaubter Attribute**. Die Instanz speichert sie dann in einer kompakten, array-artigen Struktur **statt in einem Dict**.

- **Vorteile:** deutlich **weniger Speicher** pro Instanz und **schnellerer** Attributzugriff.
- **Preis:** man kann keine neuen Attribute zur Laufzeit mehr anhängen (kein `__dict__`), und es gibt kleine Fallstricke bei Mehrfachvererbung.

```python
from dataclasses import dataclass

@dataclass(slots=True)
class Point:
    x: int
    y: int

p = Point(1, 2)
# p.z = 3   # ❌ AttributeError — 'z' ist kein deklariertes slot
```

**Direkt relevant für dich:** Dein `builder` erzeugt in einem Lauf **1,33 Mio. `PriceableUnit`s**. Bei so vielen Objekten spart `__slots__` spürbar RAM. `PriceableUnit` ist bei dir handgeschrieben — du könntest dort ein `__slots__ = ("_product_id", "_calendar_date", ...)` ergänzen; für die Dataclasses (`Booking`, `Price`, `PriceRecommendation`) ginge `@dataclass(slots=True)`. (Ein sinnvoller Kandidat für die Optimierungs-Notiz, wenn Speicher zum Thema wird.)

## Günstigere Alternative zu csv.DictReader

`csv.DictReader` ist bequem (Zugriff per Spaltenname, `row["product_id"]`), aber er baut **pro Zeile ein neues `dict`** und hasht dafür jeden Spaltennamen. Bei 939k+ Zeilen summiert sich das.

**Günstiger: `csv.reader`** — er liefert pro Zeile eine schlichte **`list`** (Zugriff per **Index**), ohne Dict-Overhead:

```python
import csv

with path.open(newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    header = next(reader)                      # Kopfzeile einmal lesen
    idx = {name: i for i, name in enumerate(header)}   # Spalte→Index EINMAL aufbauen
    rows = [
        CapacityRow(
            product_id=row[idx["product_id"]],
            ...
        )
        for row in reader
    ]
```

**Trade-off** (genau die Kehrseite deiner `DictWriter`-Entscheidung in `writers.py`): Index-Zugriff ist **positions**abhängig und damit fehleranfälliger als der Zugriff über Spaltennamen. Für Lesbarkeit/Robustheit gewinnt `DictReader`, für Tempo/Speicher `csv.reader`. Der gezeigte Kompromiss — Header **einmal** in ein `idx`-Mapping ziehen und dann per `row[idx["..."]]` zugreifen — behält die Lesbarkeit („benannte Spalten") und vermeidet das Dict-pro-Zeile. (Auch das eher ein Thema für die Optimierungs-Notiz, nicht fürs saubere Erstdesign.)

## Protocol vs. ABC (der jeweilige „Vertrag")

Beide beschreiben „diese Klasse muss X können", aber über **völlig verschiedene Vertragsarten**:

**ABC (Abstract Base Class, `abc`-Modul) — der explizite Unterschriftsvertrag.**
- Die Implementierung muss **aktiv erben** (`class CsvSource(DataSource):`) und die `@abstractmethod`s ausfüllen.
- Erzwungen **zur Laufzeit**: Solange eine abstrakte Methode fehlt, kann man die Klasse **nicht instanziieren** (`TypeError`).
- Kann **gemeinsamen konkreten Code** mitbringen (fertige Methoden, die alle Subklassen erben) → gut, wenn Implementierungen sich Logik *teilen* sollen.
- Bindung ist **nominal**: „Ich erkläre mich per Vererbung ausdrücklich zum Subtyp von X und verspreche, dessen abstrakte Methoden zu erfüllen."

```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def load(self) -> list: ...

class CsvSource(DataSource):        # muss erben
    def load(self) -> list:
        return [...]
# Fehlt load(), kracht bereits CsvSource()  → TypeError
```

**Protocol (`typing.Protocol`) — der stille Verhaltensvertrag.**
- **Keine Vererbung** nötig: Jede Klasse mit den passenden Methoden/Signaturen erfüllt das Protocol automatisch (**structural typing**).
- Geprüft **statisch durch mypy**, nicht zur Laufzeit — die Implementierung muss das Protocol **nicht mal kennen** (ideal für fremde/Third-Party-Klassen).
- **Kein geteilter Code** — reiner Verhaltensvertrag.
- Bindung ist **strukturell**: „Wenn deine Klasse so *aussieht* (diese Methoden mit diesen Signaturen hat), gilt der Vertrag als erfüllt — Absicht egal."

```python
from typing import Protocol

class DataSource(Protocol):
    def load(self) -> list: ...

class CsvSource:                   # erbt NICHT von DataSource
    def load(self) -> list:
        return [...]
# mypy akzeptiert CsvSource überall dort, wo DataSource erwartet wird
```

| | **ABC** | **Protocol** |
|---|---|---|
| Vererbung nötig? | ja (nominal) | nein (strukturell) |
| Wann geprüft? | Laufzeit (Instanziierung) | statisch (mypy) |
| Geteilter Code möglich? | ja (Mixin-Methoden) | nein |
| Implementierer muss Interface kennen? | ja | nein |
| „Vertrag" | expliziter Unterschriftsvertrag | stiller Verhaltensvertrag |

**Wann was?** ABC, wenn du gemeinsamen Code teilen, eine echte Typ-Hierarchie erzwingen oder Laufzeit-Sicherheit haben willst. Protocol, wenn du nur ein **Verhalten** verlangst, maximale Entkopplung willst und den Implementierer nicht anfassen kannst/willst. In deiner Lösung fiel die Wahl bewusst auf **Protocol** (`DataSource`, `RecommendationStrategy`, `RecommendationWriter`): reine Verhaltensverträge **ohne** geteilte Logik → kein Vererbungszwang, idiomatischer, entkoppelt. (Nebenbei: `@runtime_checkable` erlaubt bei Protocols zusätzlich `isinstance`-Checks — prüft dann aber nur die *Existenz* der Methoden, nicht die Signaturen.)

---

# ⬇️ Neue Antworten (Stand 2026-07-14)

*Letzte Runde vor dem Themenwechsel. Drei Punkte (Aliasing, Namespaces/Scopes, der „C"-Teil) gehören thematisch zu den **Python Basics** und sind dort ausführlich behandelt — hier nur kurz mit Verweis. Lambda, Mixins und Factory Patterns sind neu ausformuliert.*

## Aliasing und billige Argumentübergabe

Kurz: Mehrere Namen (in mehreren Scopes) können an **dasselbe Objekt** gebunden sein — das ist Aliasing. Weil Python beim Übergeben eines Arguments intern nur einen **Pointer** kopiert (nicht das Objekt), ist die Übergabe **billig**, selbst bei riesigen Objekten. Kehrseite: Ändert eine Funktion ein übergebenes **veränderliches** Objekt (Liste, Dict), sieht der Aufrufer die Änderung — das nutzt man bewusst, muss es aber kennen (deine `builder`-Listen mit 1,33 Mio. Objekten werden so ohne Kopieraufwand herumgereicht).

→ Ausführlich in [[Documentation/Traineeship/Python Basics/Klassen - Zusammenfassung Python-Tutorial Kap. 9.md#Namen & Objekte: Aliasing (9.1)|Klassen – Zusammenfassung (Kap. 9.1 Aliasing)]] und zum Mutation-vs-Rebinding-Unterschied in [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md#Verändern (mutate) ≠ Neu-Zuweisen (rebind)|Scopes, Namespaces und Closures]].

## Namespaces und Scopes

Kurz: Ein **Namespace** ist eine Zuordnung *Name → Objekt* (technisch ein Dict); ein **Scope** ist der Code-Bereich, in dem ein Namespace *direkt* (ohne Punkt) sichtbar ist. Namen werden per **LEGB** (Local → Enclosing → Global → Built-in) von innen nach außen aufgelöst, erster Treffer gewinnt. Wichtig fürs Klassenverständnis: Instanz und Klasse haben **je einen eigenen Namespace** (`__dict__`) — `obj.x` läuft *nicht* über LEGB, sondern durchsucht erst die Instanz, dann die Klasse, dann die Basisklassen.

→ Vollständig inkl. `global`/`nonlocal`, der `UnboundLocalError`-Falle und Knobelaufgaben in [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md|Scopes, Namespaces und Closures]].

## Data Hiding: der „C"-Teil

Kurz: Der Tutorial-Satz sagt, dass in **Python geschriebene** Klassen Daten nur *per Konvention* „verstecken" — ihr Zustand liegt in einem offenen `__dict__`, auf das jeder zugreifen kann. Die Klammer verweist auf **CPython**: die Standard-Python-Version ist selbst in **C** geschrieben, und in C implementierte Typen (die eingebauten wie `list`/`dict` sowie C-Extensions) speichern ihren Zustand in **unzugänglichen C-Strukturen** statt in einem Python-Dict. Deshalb kannst du einer Liste kein `list.__dict__` entlocken und ihr auch kein neues Attribut anhängen — solche C-Typen *können* Implementierungsdetails also **wirklich** verbergen, was reinem Python-Code verwehrt bleibt. (`__slots__` ist der halbe Weg dorthin: ersetzt das `__dict__` durch C-Level-Slots.)

→ Kontext in [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md#Verbindung zu OOP: Objekte *sind* Namespaces|Scopes, Namespaces und Closures]].

## Lambda-Funktionen

Eine `lambda` ist eine **anonyme, einzeilige Funktion**: nur ein einzelner **Ausdruck**, dessen Wert *implizit* zurückgegeben wird (kein `return`, keine Anweisungen, kein Name).

```python
add = lambda a, b: a + b     # entspricht:  def add(a, b): return a + b
add(2, 3)                    # 5
```

Der eigentliche Zweck: kurze **Wegwerf-Funktionen** dort, wo eine Funktion *als Argument* erwartet wird — allen voran der `key`-Parameter beim Sortieren/Vergleichen:

```python
recs.sort(key=lambda r: r.price_change)          # nach Attribut sortieren
teuerste = max(units, key=lambda u: u.base_price)
```

**Grenzen (bewusst eng gehalten):**
- nur **ein Ausdruck** — keine Statements, keine Zuweisungen, keine mehrzeilige Logik (ein `if` geht nur als ternärer Ausdruck `a if cond else b`).
- keine Typannotationen, kein Docstring.
- **PEP 8:** eine Lambda **nicht an einen Namen binden** (`f = lambda x: ...`) — wenn sie einen Namen braucht, nimm `def` (bessere Tracebacks, Name, Docstring). Lambdas gehören *inline* als Argument.

**Bezug:** Lambdas sind **Closures** — sie fangen Namen aus dem umgebenden Scope ein und werten sie erst *beim Aufruf* aus. Genau da lauert die klassische Schleifen-Falle → [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md#Der klassische Fehler: Closures in Schleifen|Scopes, Namespaces und Closures – Closures in Schleifen]].

## Mixins

Ein **Mixin** ist eine Klasse, die *ein* abgegrenztes Stück **wiederverwendbares Verhalten** beisteuert und dazu gedacht ist, per (Mehrfach-)Vererbung in andere Klassen „hineingemischt" zu werden — **nicht**, um selbst instanziiert zu werden. Typisch: kein eigener Zustand/`__init__`, nur Methoden, Namenskonvention `…Mixin`.

```python
class ToDictMixin:
    def to_dict(self) -> dict:
        return {k: v for k, v in self.__dict__.items()}

class SlugMixin:
    def slug(self) -> str:
        return self.name.lower().replace(" ", "-")

class Product(ToDictMixin, SlugMixin):     # mischt beide Fähigkeiten dazu
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

p = Product("Nice Bike", 999)
p.to_dict()   # {'name': 'Nice Bike', 'price': 999}
p.slug()      # 'nice-bike'
```

**Abgrenzung:**
- **vs. ABC/Protocol:** Ein Mixin liefert **fertige, konkrete Implementierung** zum Wiederverwenden (horizontaler Code-Reuse). Eine ABC/ein Protocol definiert dagegen einen **Vertrag** (was implementiert werden *muss*), keinen fertigen Code.
- **vs. normale Basisklasse:** Ein Mixin bildet keine „ist-ein"-Beziehung ab (ein `Product` *ist kein* `ToDictMixin`) — es ist reine Fähigkeits-Beimischung.

**Vorsicht:** Mixins hängen an der **MRO** (Method Resolution Order). Sie sollten fokussiert bleiben, möglichst keinen eigenen State halten und bei Bedarf **kooperativ** `super()` nutzen, damit die Kette bei Mehrfachvererbung sauber durchläuft. → Hintergrund in [[Vererbung und Polymorphie - Tiefenperspektive|Vererbung und Polymorphie – Tiefenperspektive]].

## Factory Patterns

> ⚠️ Laut Leitfaden sind Design-/Architekturpattern eigentlich **Out of Scope**. Hier bewusst nur eine kompakte Orientierung — nichts, was du für dieses Kapitel vertiefen musst.

Ein **Factory Pattern** kapselt die **Objekterzeugung**, damit der Client-Code nicht fest verdrahtet weiß, *welche* konkrete Klasse er instanziiert. Statt `CsvSource(...)` mitten im Code zu schreiben, fragt man eine „Fabrik", die je nach Eingabe das passende Objekt liefert. Das entkoppelt (passt zu *Loose Coupling*/*Dependency Injection*) und macht neue Varianten leicht ergänzbar (*Open-Closed*).

Drei Ausprägungen (grob):
- **Simple Factory** – eine Funktion/Methode, die anhand eines Parameters das richtige Objekt zurückgibt. In Python meist völlig ausreichend.
- **Factory Method** – eine (oft abstrakte) Methode, die Subklassen überschreiben, um zu bestimmen, *welche* Klasse erzeugt wird.
- **Abstract Factory** – erzeugt ganze *Familien* zusammengehöriger Objekte.

**Idiomatisch in Python** braucht man selten schwere „Factory-Klassen" wie in Java — eine **Funktion** oder ein **Dispatch-Dict** genügt oft, und `@classmethod` dient als eleganter *alternativer Konstruktor* (Factory Method light):

```python
def make_source(kind: str, path: str) -> DataSource:      # Simple Factory
    match kind:
        case "csv": return CsvSource(path)
        case "db":  return DbSource(path)
        case _:     raise ValueError(f"unbekannte Quelle: {kind}")

# oder als alternativer Konstruktor direkt an der Klasse:
class Booking:
    @classmethod
    def from_dict(cls, row: dict) -> "Booking":            # Factory Method-Idee
        return cls(row["id"], float(row["price"]))
```

Bezug zu deiner Lösung: Deine `DataSource`/`RecommendationStrategy`-Protocols sind der ideale Boden für so eine Factory — der Aufruf-Code hängt nur am Protokoll, die Factory entscheidet über die konkrete Implementierung. (Für dieses Kapitel nur als Ausblick.)
