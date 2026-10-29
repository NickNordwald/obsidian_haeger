Geht über [[1.1 OOP - Vertiefung#Kapselung|Kapselung]] aus der Vertiefung hinaus: Dort wird `@property` als fertiges Werkzeug benutzt — hier geht es darum, **wie** `@property` unter der Haube tatsächlich funktioniert, und um zwei verwandte Mechanismen (`__slots__`, dynamische Attributzugriffs-Hooks), die in der Vertiefung nicht vorkommen. Übersicht siehe [[1.1 OOP - Zusammenfassung|1]].

# Das Descriptor-Protokoll: Was `@property` wirklich ist

`@property` ist kein Sprach-Feature für sich, sondern nur eine bequeme, eingebaute Anwendung des allgemeineren **Descriptor-Protokolls**. Ein **Descriptor** ist eine Klasse, die mindestens eine der Methoden `__get__`, `__set__` oder `__delete__` implementiert und als **Klassenattribut** (nicht Instanzattribut!) in einer anderen Klasse verwendet wird. Sobald das der Fall ist, übernimmt der Descriptor die Kontrolle über den Attributzugriff.

```python
class ValidatedFloat:
    """Ein eigener Descriptor - macht von Hand ungefähr das, was @property tut."""

    def __set_name__(self, owner, name):
        self.name = f"_{name}"   # z.B. "_base_price"

    def __get__(self, instance, owner):
        if instance is None:      # Zugriff über die Klasse selbst, nicht eine Instanz
            return self
        return getattr(instance, self.name)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"{self.name} darf nicht negativ sein")
        setattr(instance, self.name, value)


class PriceableUnit:
    base_price = ValidatedFloat()   # Descriptor als KLASSENATTRIBUT

    def __init__(self, base_price: float):
        self.base_price = base_price   # ruft ValidatedFloat.__set__ auf!


unit = PriceableUnit(19.99)
print(unit.base_price)      # ruft ValidatedFloat.__get__ auf -> 19.99
unit.base_price = -5        # ruft ValidatedFloat.__set__ auf -> ValueError
```

`property(fget, fset, fdel)` (das, was `@property` im Hintergrund erzeugt) ist selbst nichts anderes als eine eingebaute Descriptor-Klasse, die `fget`/`fset`/`fdel` intern auf `__get__`/`__set__`/`__delete__` abbildet. Der Unterschied zu einem selbstgebauten Descriptor wie oben: `@property` ist **pro Attribut** an eine Klasse gebunden (du schreibst für jedes Attribut einen eigenen Getter/Setter), ein eigener Descriptor wie `ValidatedFloat` ist **wiederverwendbar** über beliebig viele Attribute und Klassen hinweg (hier könnte man z. B. auch `demand_factor = ValidatedFloat()` in derselben Klasse ergänzen, ohne Code zu duplizieren).

## Data- vs. Non-Data-Descriptors — warum das für die Attributsuche wichtig ist

Ein Descriptor mit `__set__` und/oder `__delete__` heißt **Data-Descriptor**, einer nur mit `__get__` (z. B. eine reine Methode/Funktion, oder `@property` ohne Setter im Sinne von "kein `__set__` definiert" — Achtung: `property` definiert intern immer `__set__`, auch wenn kein Setter übergeben wurde, es wirft dann nur `AttributeError`) heißt **Non-Data-Descriptor**. Das bestimmt die **Reihenfolge**, in der Python beim Attributzugriff sucht:

1. Data-Descriptors aus der Klasse (haben **immer Vorrang**, auch vor dem Instanz-`__dict__`)
2. Einträge im Instanz-`__dict__`
3. Non-Data-Descriptors bzw. sonstige Klassenattribute (z. B. normale Methoden)

Das erklärt ein Detail, das beim Vererbungs-/Klassenattribut-Kapitel der Vertiefung offenbleibt: Wenn `base_price` ein `@property` (Data-Descriptor) ist, kann `self.base_price = x` **nicht** einfach einen Instanz-Dict-Eintrag anlegen, der die Property „überschreibt" (wie es bei einem normalen Klassenattribut der Fall wäre) — die Property gewinnt immer. Genau deshalb funktioniert Validierung über `@property` zuverlässig, während ein normales, überschreibbares Klassenattribut (siehe Vertiefung, „Instanzattribute vs. Klassenattribute") das nicht garantieren könnte.

# `__slots__`: Instanzattribute ohne `__dict__`

Normalerweise hat jede Instanz ein eigenes `__dict__`, in dem beliebige Attribute zur Laufzeit angelegt werden können (siehe Vertiefung, Introspection-Abschnitt). `__slots__` deklariert stattdessen eine **feste, geschlossene Liste** erlaubter Attributnamen und verhindert, dass überhaupt ein `__dict__` pro Instanz erzeugt wird:

```python
class PriceableUnit:
    __slots__ = ("unit_id", "base_price")   # NUR diese zwei Attribute sind erlaubt

    def __init__(self, unit_id: str, base_price: float):
        self.unit_id = unit_id
        self.base_price = base_price


unit = PriceableUnit("SKU-001", 19.99)
unit.discount = 0.1   # AttributeError: 'PriceableUnit' object has no attribute 'discount'
```

Zwei Effekte, die direkt an Kapselung/Information Hiding anknüpfen:

1. **Strengere Kapselung „for free"**: Aufrufender Code kann nicht versehentlich (Tippfehler, unbedachte Erweiterung) neue Attribute an eine Instanz anhängen — die Klasse legt die vollständige Menge ihres Zustands abschließend fest.
2. **Speicher-/Performance-Vorteil**: Ohne `__dict__` pro Instanz sinkt der Speicherverbrauch spürbar, was bei sehr vielen Instanzen (z. B. tausende `PriceableUnit`-Objekte) messbar wird — ein Grund, warum `__slots__` in datenintensiven Anwendungen (Data Engineering!) gerne eingesetzt wird.

Wichtige Einschränkung: `__slots__` und mehrfache Vererbung mit unterschiedlichen, nicht-leeren Slot-Definitionen vertragen sich schlecht (Layout-Konflikte im Speicher) — meist reicht höchstens eine Basisklasse mit „echten" Slots. Auch `@property`/Descriptoren funktionieren mit `__slots__` weiterhin problemlos zusammen, weil sie ohnehin über die Klasse (nicht über das Instanz-`__dict__`) aufgelöst werden.

# `__getattr__`, `__setattr__`, `__getattribute__` — Kapselung auf der nächsten Ebene

Drei weitere Hooks, mit denen sich der Attributzugriff einer ganzen Klasse (nicht nur eines einzelnen Attributs wie bei `@property`) kontrollieren lässt:

- **`__getattr__(self, name)`**: wird **nur** aufgerufen, wenn die normale Attributsuche (Instanz-`__dict__`, Klasse, MRO) **fehlschlägt** — nützlich für „fallback"-Verhalten, z. B. dynamisch generierte Attribute oder aussagekräftigere Fehlermeldungen.
- **`__setattr__(self, name, value)`**: wird bei **jeder** Zuweisung `obj.x = y` aufgerufen, auch bei bereits existierenden Attributen — Grundlage für z. B. ein generisches „alle Attribute sind read-only nach der Erzeugung"-Verhalten (das, was `@dataclass(frozen=True)` intern nutzt).
- **`__getattribute__(self, name)`**: wird bei **jedem** Attributzugriff aufgerufen, noch bevor die normale Suche überhaupt beginnt — mächtig, aber gefährlich (leicht, versehentlich unendliche Rekursion zu bauen, wenn man darin `self.x` statt `super().__getattribute__("x")` verwendet).

```python
class FrozenUnit:
    def __init__(self, unit_id: str, base_price: float):
        super().__setattr__("unit_id", unit_id)
        super().__setattr__("base_price", base_price)

    def __setattr__(self, name, value):
        raise AttributeError(f"{self.__class__.__name__} ist unveränderlich (Versuch: {name}={value!r})")


unit = FrozenUnit("SKU-001", 19.99)
unit.base_price = 25.0   # AttributeError: FrozenUnit ist unveränderlich
```

Das ist im Kern dasselbe Prinzip, das `@dataclass(frozen=True)` (siehe Vertiefung, letztes Kapitel) automatisch für dich generiert — nur von Hand nachgebaut, um zu zeigen, *warum* es funktioniert.

---

Zurück zur Übersicht: [[1.1 OOP - Zusammenfassung|1]] · Grundlage: [[1.1 OOP - Vertiefung|1]]
