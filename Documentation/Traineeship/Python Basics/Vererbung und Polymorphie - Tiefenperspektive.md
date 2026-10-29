Geht über [[1.1 OOP - Vertiefung#Vererbung|Vererbung]] und [[1.1 OOP - Vertiefung#Polymorphie|Polymorphie]] aus der Vertiefung hinaus — für alle, die verstehen wollen, was hinter `super()` und der MRO wirklich passiert, statt es nur anzuwenden. Übersicht siehe [[1.1 OOP - Zusammenfassung|1]].

# Die C3-Linearisierung im Detail

Die Vertiefung erwähnt, dass Python die MRO über den **C3-Linearisierungsalgorithmus** deterministisch auflöst — hier, wie das konkret funktioniert, am klassischen **Diamond-Problem**:

```python
class Base:
    def greet(self) -> str:
        return "Base"

class Left(Base):
    def greet(self) -> str:
        return f"Left -> {super().greet()}"

class Right(Base):
    def greet(self) -> str:
        return f"Right -> {super().greet()}"

class Diamond(Left, Right):
    def greet(self) -> str:
        return f"Diamond -> {super().greet()}"

print(Diamond.__mro__)
# (Diamond, Left, Right, Base, object)

print(Diamond().greet())
# Diamond -> Left -> Right -> Base
```

Naiv würde man eine **Tiefensuche von links nach rechts** erwarten (Diamond → Left → Base → Right → Base), bei der `Base` doppelt aufgetaucht wäre. C3 verhindert das: Jede Klasse taucht in der MRO genau **einmal** auf, und zwar erst, nachdem alle ihre Subklassen bereits „bedient" wurden. Der Algorithmus berechnet die MRO als **Merge** dreier Listen — der MRO von `Left`, der MRO von `Right`, und der Liste `[Left, Right]` selbst (die explizite Basisklassen-Reihenfolge aus `class Diamond(Left, Right)`):

```
L[Diamond] = Diamond + merge(L[Left], L[Right], [Left, Right])
           = Diamond + merge([Left, Base, object], [Right, Base, object], [Left, Right])
           = Diamond, Left, Right, Base, object
```

Merge-Regel: Nimm aus der ersten Liste den Kopf, der in **keiner** der anderen Listen (außer an deren eigenem Kopf) als Element weiter hinten vorkommt ("good head"); wiederhole, bis alle Listen leer sind. Ist kein gültiger Kopf zu finden, ist die Hierarchie inkonsistent → Python wirft beim Definieren der Klasse einen `TypeError: Cannot create a consistent method resolution order`. Das passiert z. B., wenn zwei Basisklassen sich in widersprüchlicher Reihenfolge selbst referenzieren.

## Warum `super()` NICHT „die Elternklasse" bedeutet

Der wohl wichtigste Fehlschluss beim Einstieg: `super()` ruft **nicht** "die Basisklasse" auf, sondern **die nächste Klasse in der MRO, ausgehend von der aktuellen Klasse**. Im Beispiel oben ruft `super()` in `Left.greet()` nicht `Base.greet()` direkt auf, sondern das nächste Element nach `Left` in `Diamond.__mro__` — das ist zur Laufzeit `Right`, weil `Diamond` von der MRO von `Diamond()`-Instanzen bestimmt wird, nicht von `Left` isoliert betrachtet. Das ist der Kern von **kooperativer Mehrfachvererbung**: Jede Klasse in der Kette ruft brav `super()` auf und vertraut darauf, dass die MRO den Rest schon richtig zusammensetzt — niemand "kennt" seine tatsächlichen Nachbarn im Voraus.

Praktische Konsequenz: Mixins (siehe Vertiefung) müssen **ebenfalls** `super().__init__(**kwargs)` aufrufen (auch wenn sie selbst von `object` erben), damit die Aufrufkette in Mehrfachvererbungs-Konstellationen nicht abreißt:

```python
class TimestampedMixin:
    def __init__(self, **kwargs):
        super().__init__(**kwargs)   # WICHTIG: auch Mixins müssen kooperativ sein!
        self.created_at = "now"

class LoggingMixin:
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.log = []

class DigitalProduct(TimestampedMixin, LoggingMixin):
    def __init__(self, unit_id: str, **kwargs):
        super().__init__(**kwargs)
        self.unit_id = unit_id

p = DigitalProduct(unit_id="EBOOK-001")
print(p.created_at, p.log, p.unit_id)
```

Ohne die `super().__init__(**kwargs)`-Aufrufe in den Mixins würde `LoggingMixin.__init__` nie erreicht, weil `TimestampedMixin.__init__` die Kette selbst abgebrochen hätte.

# `__init_subclass__`: auf Vererbung reagieren, ohne eine Metaklasse zu brauchen

Eine seltener bekannte, aber sehr nützliche Erweiterung von Vererbung: Eine Basisklasse kann `__init_subclass__` definieren, um automatisch zu reagieren, sobald **irgendeine** Subklasse definiert wird — z. B. um eine Registry aller Subklassen zu pflegen (praktisch als Alternative zu einer wachsenden `if isinstance(...)`-Kette, siehe Open/Closed in der Vertiefung):

```python
class PriceableUnit:
    registry: dict[str, type["PriceableUnit"]] = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        PriceableUnit.registry[cls.__name__] = cls   # jede neue Subklasse meldet sich automatisch an

class DigitalProduct(PriceableUnit):
    pass

class PhysicalProduct(PriceableUnit):
    pass

print(PriceableUnit.registry)
# {'DigitalProduct': <class 'DigitalProduct'>, 'PhysicalProduct': <class 'PhysicalProduct'>}
```

Das ist der Mechanismus, den z. B. viele ORM- oder Plugin-Systeme (SQLAlchemy, Pydantic-ähnliche Bibliotheken) nutzen, um automatisch alle Modellklassen einzusammeln, ohne dass man sie manuell an einer zentralen Stelle registrieren muss.

# Virtuelle Subklassen: `register()` und `__subclasshook__`

Die Vertiefung zeigt `abc.ABC` mit klassischer Vererbung. Das `abc`-Modul erlaubt aber auch, eine Klasse **ohne** echte Vererbungsbeziehung als „virtuelle Subklasse" zu registrieren — `isinstance()`/`issubclass()` geben dann trotzdem `True` zurück:

```python
from abc import ABC, abstractmethod

class Exportable(ABC):
    @abstractmethod
    def export(self) -> str: ...

class LegacyReport:
    """Existierte schon vor Exportable, hat aber zufällig eine export()-Methode."""
    def export(self) -> str:
        return "legacy export"

Exportable.register(LegacyReport)   # keine echte Vererbung, nur eine Registrierung!

print(issubclass(LegacyReport, Exportable))   # True
print(isinstance(LegacyReport(), Exportable)) # True
```

Das verbindet nominale Polymorphie (Vererbung) mit struktureller Polymorphie (Duck Typing): Man bekommt die Typprüfbarkeit einer gemeinsamen Basisklasse, ohne bestehenden Code (`LegacyReport`) tatsächlich anfassen oder umschreiben zu müssen. `collections.abc` nutzt genau dieses Prinzip intern (z. B. `Sequence.register(tuple)`).

# Liskov Substitution Principle — die „fehlende" Grundregel hinter Vererbung

Die Vertiefung erklärt, *wann* Vererbung sinnvoll ist (die „ist-ein"-Beziehung), aber nicht, was passiert, wenn diese Beziehung nur *syntaktisch*, aber nicht *semantisch* stimmt. Genau das behandelt das **Liskov Substitution Principle (LSP)** — ausführlich in [[SOLID-Prinzipien#Liskov Substitution Principle (LSP)|SOLID-Prinzipien]]: Jede Stelle, die eine Basisklasse erwartet, muss auch mit jeder Subklasse korrekt funktionieren — sonst ist die „ist-ein"-Beziehung eine Falle, die Polymorphie (siehe oben, `calculate_all_price_recommendations()`) zur Laufzeit unbemerkt bricht.

---

Zurück zur Übersicht: [[1.1 OOP - Zusammenfassung|1]] · Grundlage: [[1.1 OOP - Vertiefung|1]]
