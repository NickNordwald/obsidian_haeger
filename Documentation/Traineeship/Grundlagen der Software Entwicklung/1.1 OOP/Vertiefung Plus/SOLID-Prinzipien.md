Setzt [[1.1 OOP - Vertiefung#Single Responsibility Prinzip|Single Responsibility Prinzip]], [[1.1 OOP - Vertiefung#Open/Closed Prinzip|Open/Closed Prinzip]] und [[1.1 OOP - Vertiefung#Dependency Injection|Dependency Injection]] aus der Vertiefung zum vollständigen **SOLID**-Akronym zusammen und ergänzt die zwei dort fehlenden Buchstaben — **Liskov Substitution** und **Interface Segregation**. SOLID sind fünf Design-Prinzipien (Robert C. Martin), die zusammen beschreiben, wie man Klassen so gestaltet, dass ein System änderbar bleibt, ohne bei jeder Erweiterung instabil zu werden. Übersicht siehe [[1.1 OOP - Zusammenfassung|1]].

# Überblick: die fünf Buchstaben

| Buchstabe | Prinzip | Kurzfassung | Bereits behandelt in |
|---|---|---|---|
| **S** | Single Responsibility | Eine Klasse, ein Grund zur Änderung | [[1.1 OOP - Vertiefung#Single Responsibility Prinzip\|Vertiefung]] |
| **O** | Open/Closed | Offen für Erweiterung, geschlossen für Veränderung | [[1.1 OOP - Vertiefung#Open/Closed Prinzip\|Vertiefung]] |
| **L** | Liskov Substitution | Subklassen müssen sich wie ihre Basisklasse verhalten | neu, siehe unten |
| **I** | Interface Segregation | Viele kleine, spezifische Schnittstellen statt einer großen | neu, siehe unten |
| **D** | Dependency Inversion | Von Abstraktionen abhängen, nicht von Konkretem | [[1.1 OOP - Vertiefung#Dependency Injection\|Vertiefung]] (dort als Abgrenzung zu DI erwähnt) |

Wichtig zu verstehen: SOLID ist kein zusammenhängender Algorithmus, den man Schritt für Schritt abarbeitet, sondern fünf **unabhängige, sich gegenseitig verstärkende** Heuristiken. In der Praxis begegnen sie einem selten einzeln — eine LSP-Verletzung zieht z. B. oft auch eine OCP-Verletzung nach sich (siehe unten).

# Liskov Substitution Principle (LSP)

## Grundidee

Formuliert von Barbara Liskov: *"Objekte eines Obertyps müssen durch Objekte eines Untertyps ersetzbar sein, ohne dass die Korrektheit des Programms beeinträchtigt wird."* Anders gesagt: Überall, wo Code eine `PriceableUnit` erwartet, muss **jede** Subklasse (`DigitalProduct`, `PhysicalProduct`, …) genauso gut funktionieren — nicht nur syntaktisch (die Methode existiert), sondern auch **semantisch** (die Methode hält sich an die erwarteten Spielregeln).

## Das klassische Gegenbeispiel: Rechteck/Quadrat

```python
class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def set_width(self, width: float) -> None:
        self.width = width

    def set_height(self, height: float) -> None:
        self.height = height

    def area(self) -> float:
        return self.width * self.height


class Square(Rectangle):
    """Mathematisch ist ein Quadrat ein Sonderfall eines Rechtecks - scheint also 'ist-ein' zu sein."""
    def set_width(self, width: float) -> None:
        self.width = width
        self.height = width   # muss synchron bleiben, sonst ist es kein Quadrat mehr!

    def set_height(self, height: float) -> None:
        self.width = height
        self.height = height


def test_rectangle_behavior(rect: Rectangle) -> None:
    rect.set_width(4)
    rect.set_height(5)
    assert rect.area() == 20, f"Erwartet 20, bekommen {rect.area()}"   # gilt für JEDES Rectangle...

test_rectangle_behavior(Rectangle(2, 2))   # OK
test_rectangle_behavior(Square(2, 2))      # AssertionError: bekommen 25 - LSP verletzt!
```

Obwohl `Square` **mathematisch** ein Sonderfall von `Rectangle` ist, verletzt es LSP: Code, der sich auf das Verhalten von `Rectangle` verlässt (`set_width` und `set_height` sind unabhängig voneinander), bricht unbemerkt bei `Square`. Das zeigt den entscheidenden Unterschied zur „ist-ein"-Faustregel aus dem Vererbungskapitel der Vertiefung: **„ist-ein" im Sinne der Fachdomäne reicht nicht** — entscheidend ist, ob sich die Subklasse **verhaltenskompatibel** zur Basisklasse verhält.

## Formale Regeln: Prä- und Postconditions

Etwas technischer, angelehnt an Design-by-Contract-Denken:

- Eine Subklasse darf **Preconditions nicht verschärfen** (keine strengeren Anforderungen an Eingaben stellen als die Basisklasse).
- Eine Subklasse darf **Postconditions nicht abschwächen** (muss mindestens das liefern/garantieren, was die Basisklasse garantiert).
- Invarianten der Basisklasse müssen erhalten bleiben.

Übertragen auf das laufende `PriceableUnit`-Beispiel aus der Vertiefung: Würde eine Subklasse `calculate_price_recommendation()` so überschreiben, dass sie plötzlich `None` statt eines `float` zurückgeben kann (Postcondition abgeschwächt) oder zusätzlich einen speziellen, undokumentierten Parameter zwingend verlangt (Precondition verschärft), wäre das eine LSP-Verletzung — der Code aus `calculate_all_price_recommendations()` (siehe Polymorphie-Kapitel der Vertiefung) würde für diese eine Subklasse unerwartet crashen, obwohl er „polymorph, unabhängig vom konkreten Typ" geschrieben wurde.

## Zusammenhang mit Open/Closed und Polymorphie

LSP ist die stillschweigende **Voraussetzung**, damit Open/Closed und Polymorphie (beide in der Vertiefung behandelt) überhaupt sicher funktionieren: `calculate_all_price_recommendations()` ist nur deshalb "geschlossen für Veränderung", weil man sich darauf verlassen kann, dass *jede* aktuelle und zukünftige Subklasse sich an den Vertrag von `calculate_price_recommendation()` hält. Bricht eine Subklasse LSP, bricht implizit auch das Versprechen von Open/Closed — der aufrufende Code bleibt zwar unverändert, verhält sich aber für diese eine Subklasse plötzlich falsch, was oft erst spät (zur Laufzeit, bei genau diesem einen Typ) auffällt.

# Interface Segregation Principle (ISP)

## Grundidee

*"Kein Client sollte gezwungen sein, von Methoden abzuhängen, die er nicht benutzt."* Statt eines großen, „fetten" Interfaces mit vielen Methoden lieber mehrere kleine, fokussierte Interfaces — eine Klasse implementiert dann nur die, die sie tatsächlich braucht.

## Beispiel: ein „fettes" Interface zwingt zu sinnlosen Implementierungen

```python
from typing import Protocol

class PriceableUnitOperations(Protocol):
    """Ein einziges, großes Interface für alles, was mit einer Unit passieren könnte."""
    def calculate_price_recommendation(self) -> float: ...
    def export_to_csv(self, path: str) -> None: ...
    def send_notification(self) -> None: ...
    def archive(self) -> None: ...


class FreeSample:
    """Braucht nur die Preisberechnung, muss aber wegen ISP-Verletzung ALLES implementieren."""
    def calculate_price_recommendation(self) -> float:
        return 0.0

    def export_to_csv(self, path: str) -> None:
        raise NotImplementedError("FreeSample wird nie exportiert")   # Code-Geruch!

    def send_notification(self) -> None:
        raise NotImplementedError("FreeSample verschickt keine Benachrichtigungen")

    def archive(self) -> None:
        raise NotImplementedError("FreeSample wird nicht archiviert")
```

Drei `NotImplementedError`-Methoden sind ein klares Warnsignal: `FreeSample` wird gezwungen, eine Schnittstelle vorzutäuschen, die es gar nicht sinnvoll erfüllen kann — das ist dieselbe Symptomatik wie eine SRP-Verletzung (siehe Vertiefung), nur auf Schnittstellenebene statt Klassenebene.

## Auflösung: mehrere kleine Protocols

```python
class PriceCalculable(Protocol):
    def calculate_price_recommendation(self) -> float: ...

class Exportable(Protocol):
    def export_to_csv(self, path: str) -> None: ...

class Notifiable(Protocol):
    def send_notification(self) -> None: ...


class FreeSample:
    """Implementiert jetzt nur, was es tatsächlich kann - keine leeren/fehlerwerfenden Methoden."""
    def calculate_price_recommendation(self) -> float:
        return 0.0


def calculate_all_price_recommendations(units: list[PriceCalculable]) -> None:
    for unit in units:
        print(unit.calculate_price_recommendation())   # braucht NUR PriceCalculable, nicht das ganze Bündel
```

`calculate_all_price_recommendations()` verlangt jetzt nur genau das schmale Protocol, das es wirklich benutzt (`PriceCalculable`) — nicht ein aufgeblähtes Interface mit Methoden, die für diese Funktion irrelevant sind. Das ist direkt eine Verfeinerung von **Loose Coupling** (siehe Vertiefung): Je kleiner die Schnittstelle, von der eine Funktion abhängt, desto weniger Gründe gibt es, dass sich diese Funktion durch unabhängige Änderungen an anderer Stelle des großen Interfaces „mitbewegen" muss.

## ISP in Python: `Protocol`-Komposition statt Vererbungshierarchien

Ein Vorteil von `typing.Protocol` gegenüber klassischen `abc.ABC`-Interfaces für ISP: Eine Klasse kann mehrere kleine Protocols gleichzeitig erfüllen, **ohne** sie explizit zu erben oder zu kombinieren — sie muss nur die jeweils geforderten Methoden besitzen (strukturelle Typisierung, siehe Polymorphie-Kapitel der Vertiefung). Das macht ISP in Python leichtgewichtiger als z. B. in Java, wo Interfaces explizit mit `implements A, B, C` deklariert werden müssen.

# Dependency Inversion Principle (DIP) — kurze Ergänzung

Die Vertiefung grenzt DIP bereits knapp von Dependency Injection ab: DIP = *"Module sollen von Abstraktionen abhängen, nicht von konkreten Implementierungen"* (das Design-Prinzip), DI = die Injektions-Technik, mit der man DIP umsetzt. Ergänzend hier die **Inversions-Idee** im Namen: Klassischerweise hängt eine High-Level-Komponente (Business-Logik) direkt von einer Low-Level-Komponente ab (`PriceRecommendationCalculator` importiert `CsvPriceRecommendationWriter`). DIP „invertiert" diese Abhängigkeitsrichtung: **Beide** Seiten hängen stattdessen von einer dazwischenliegenden Abstraktion ab (`RecommendationWriter`-Protocol) — die Low-Level-Komponente (`CsvPriceRecommendationWriter`) muss sich nach der Abstraktion richten, nicht umgekehrt. Dadurch kann sich die Business-Logik-Schicht entwickeln, ohne von Details der Persistenzschicht abzuhängen (vgl. Separation of Concerns/Schichtenarchitektur in der Vertiefung).

# Wie die fünf Prinzipien zusammenspielen

- **SRP** hält einzelne Klassen fokussiert → macht es überhaupt erst praktikabel, **OCP** einzuhalten (kleine, fokussierte Klassen lassen sich leichter erweitern statt ändern).
- **LSP** ist die stille Voraussetzung, damit **OCP** und Polymorphie *sicher* funktionieren (siehe oben).
- **ISP** ist SRP, angewendet auf Schnittstellen statt Klassen — schmale Schnittstellen fördern **lose Kopplung** (siehe Vertiefung, Loose-Coupling-Kapitel).
- **DIP** bindet alles zusammen: Abhängigkeiten laufen über Abstraktionen (die durch ISP idealerweise ohnehin schon schmal sind), umgesetzt über **Dependency Injection** (siehe Vertiefung und [[Dependency Injection und Inversion of Control|Dependency Injection und Inversion of Control]]).

---

Zurück zur Übersicht: [[1.1 OOP - Zusammenfassung|1]] · Grundlage: [[1.1 OOP - Vertiefung|1]]
