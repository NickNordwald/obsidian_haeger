---
tags:
  - documentation
---
Erklärung, wie Python Namen auflöst: der Unterschied zwischen **Namespace** (Name → Objekt) und **Scope** (Sichtbarkeit), die **LEGB**-Suchregel, die Stolperfallen mit `global`/`nonlocal` und schließlich **Closures** samt dem klassischen Schleifen-Fehler. Ergänzt die Laufzeit-Perspektive aus [[Documentation/Traineeship/Python Basics/Compile-, Typecheck- und Laufzeit in Python.md|Compile-, Typecheck- und Laufzeit]], die OOP-Grundlagen aus [[1.1 OOP - Leitfaden|1.1 OOP – Leitfaden]] und die Klassen-Zusammenfassung in [[Documentation/Traineeship/Python Basics/Klassen - Zusammenfassung Python-Tutorial Kap. 9.md|Klassen – Zusammenfassung (Tutorial Kap. 9)]].

# Der Kernunterschied: Namespace ≠ Scope

Die meisten Erklärungen werfen zwei verschiedene Dinge zusammen. Sobald man sie trennt, wird das Thema einfach.

- **Namespace** = eine Zuordnung von **Namen → Objekt**. Technisch buchstäblich ein Dictionary: `{"x": <Objekt>, "berechne": <Funktion>}`. Es ist die *Landkarte*, welcher Name auf welches Objekt zeigt.
- **Scope** = der **Bereich im Code**, in dem ein bestimmter Namespace *direkt* (ohne Punkt-Notation) erreichbar ist. Es ist die *Sichtbarkeit*.

> **Analogie:** Der **Namespace** ist ein Telefonbuch (Name → Nummer). Der **Scope** ist die Frage: *Welches* Telefonbuch schlage ich von meinem aktuellen Standort aus auf — und in welcher Reihenfolge?

Ein Name existiert nie „an sich" — er lebt immer *in* einem Namespace. Welchen Namespace man gerade sieht, bestimmt der Scope.

```python
x = 10          # Name "x" landet im globalen Namespace (dem Modul-Dict)

def f():
    y = 5       # Name "y" landet im lokalen Namespace von f
    print(x)    # "x" ist hier nicht lokal -> Python sucht weiter außen

# print(y)      # NameError: "y" existiert nur im Namespace von f
```

Namespaces sind echte Dicts, die man anschauen kann:

```python
def f():
    y = 5
    print(locals())    # {'y': 5}   <- der lokale Namespace als Dict
print(globals())       # {..., 'x': 10, 'f': <function f>}  <- der globale
```

# Die LEGB-Regel: die Suchreihenfolge

Stößt Python auf einen Namen, sucht es die Namespaces **von innen nach außen** ab:

```
L  →  Local        Der Namespace der aktuellen Funktion
E  →  Enclosing    Namespaces umgebender (äußerer) Funktionen
G  →  Global       Der Namespace des Moduls (der Datei)
B  →  Built-in     Pythons eingebaute Namen (print, len, list, ...)
```

Beim **ersten Treffer** stoppt die Suche. Findet Python den Namen nirgends → `NameError`.

```python
name = "Global"

def outer():
    name = "Enclosing"
    def inner():
        name = "Local"
        print(name)      # -> "Local"   (L gewinnt, Suche stoppt sofort)
    inner()

outer()
```

Kommentiert man Zeile für Zeile die inneren `name = ...` aus, wandert das Ergebnis nach außen: `"Local"` → `"Enclosing"` → `"Global"` → `NameError`. Genau das ist LEGB, live durchgespielt.

`print`, `len`, `list` funktionieren überall, weil sie im **B**uilt-in-Namespace ganz außen liegen.

# Die wichtigste Falle: Zuweisung erzeugt *lokale* Namen

Quelle von ~90 % der Verwirrung: **Sobald man in einer Funktion einem Namen etwas zuweist, gilt er für die *gesamte* Funktion als lokal** — selbst wenn außen ein gleichnamiger existiert.

```python
counter = 0

def f():
    print(counter)     # UnboundLocalError!
    counter = counter + 1
```

Warum der Fehler? Das `counter = ...` weiter unten macht `counter` zu einem *lokalen* Namen. Damit ist auch beim `print` das lokale gemeint — das aber noch keinen Wert hat. Python entscheidet „lokal oder nicht" **statisch beim Kompilieren**, nicht Zeile für Zeile (siehe [[Documentation/Traineeship/Python Basics/Compile-, Typecheck- und Laufzeit in Python.md|Compile-, Typecheck- und Laufzeit]]).

## Die Lösung: `global` und `nonlocal`

Diese Schlüsselwörter sagen: „Weise *nicht* neu an, sondern greif auf den äußeren Namen zu."

```python
counter = 0
def f():
    global counter        # "counter" meint den globalen Namen
    counter = counter + 1 # verändert jetzt den globalen

def outer():
    count = 0
    def inner():
        nonlocal count    # "count" meint den der umgebenden Funktion (E)
        count += 1
    inner()
    print(count)          # -> 1
```

- `global` → greif in den **G**lobal-Namespace.
- `nonlocal` → greif in den nächsten **E**nclosing-Namespace (**nicht** global!).

## Verändern (mutate) ≠ Neu-Zuweisen (rebind)

Der Ärger entsteht **nur beim Neu-Zuweisen** (`=`). Ein *veränderbares* Objekt zu **mutieren** braucht kein `global`:

```python
data = []
def f():
    data.append(1)   # OK! keine Zuweisung an "data", nur Mutation des Objekts
```

Hier wird `data` nichts Neues zugewiesen — es wird nur eine Methode auf dem Objekt aufgerufen, auf das `data` zeigt. Deshalb bleibt `data` der globale Name und es funktioniert ohne `global`.

> **Merksatz:** Nur `=` (Rebinding) macht einen Namen lokal. `.append()`, `[k] = v` & Co. (Mutation) tun das nicht.

# Closures

Eine **Closure** ist eine Funktion, die sich Namen aus ihrem **umgebenden** Scope „merkt" — auch dann noch, wenn die äußere Funktion längst zurückgekehrt ist.

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor      # "factor" kommt aus dem Enclosing-Scope
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(10))   # 20
print(triple(10))   # 30
```

`double` und `triple` sind zwei Funktionen, die jeweils ihr eigenes `factor` „eingeschlossen" haben — daher der Name *Closure* (engl. *to close over*).

## Der entscheidende Punkt: eine Closure speichert die *Variable*, nicht den *Wert*

Das ist der ganze Knackpunkt. Wenn du `lambda: i` schreibst, passiert **nicht**:

- ❌ „Merke dir den aktuellen *Wert* von `i`."

Sondern:

- ✅ „Merke dir: *wenn ich aufgerufen werde*, schau `i` im umgebenden Namespace nach — was auch immer dann drinsteht."

Eine Closure speichert also einen **Verweis auf die Variable**, keine Momentaufnahme des Wertes. Und den Wert schaut sie **erst beim Aufruf** nach.

## Der klassische Fehler: Closures in Schleifen

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])     # [2, 2, 2] — nicht [0, 1, 2]!
```

Warum? In der Schleife wird `i` **nicht** dreimal neu erzeugt — es ist **dieselbe Variable**, die überschrieben wird: 0 → 1 → 2. Alle drei Lambdas zeigen auf **genau dieses eine `i`**.

| Zeitpunkt | Was passiert | Wert von `i` |
|---|---|---|
| Erstellung Lambda 1 | Funktion gebaut, `i` **nicht** ausgewertet | `i == 0` |
| Erstellung Lambda 2 | Funktion gebaut, `i` **nicht** ausgewertet | `i == 1` |
| Erstellung Lambda 3 | Funktion gebaut, `i` **nicht** ausgewertet | `i == 2` |
| **Schleife fertig** | `i` bleibt auf letztem Wert stehen | **`i == 2`** |
| `funcs[0]()` | *jetzt* wird `i` nachgeschaut → 2 | `i == 2` |
| `funcs[1]()` | *jetzt* wird `i` nachgeschaut → 2 | `i == 2` |
| `funcs[2]()` | *jetzt* wird `i` nachgeschaut → 2 | `i == 2` |

Beim Aufrufen ist die Schleife **längst durch**, `i` steht auf 2. Also geben alle drei `2` zurück.

**Gegenprobe** — wertet man `i` sofort aus (kein Lambda dazwischen), stimmt alles:

```python
values = [i for i in range(3)]   # i wird SOFORT ausgewertet
print(values)                    # [0, 1, 2]  ✓
```

## Der Fix: den Wert pro Iteration einfrieren

```python
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])      # [0, 1, 2]  ✓
```

Der Trick: **Default-Werte von Parametern werden beim *Erstellen* der Funktion ausgewertet**, nicht beim Aufruf.

- Das **rechte** `i` (`=i`) ist die Schleifenvariable → wird *jetzt* gelesen, ihr aktueller Wert wird als Default eingefroren.
- Das **linke** `i` ist ein neuer, *lokaler* Parameter.

| Lambda | wird effektiv zu | eingefrorener Default |
|---|---|---|
| 1 | `lambda i=0: i` | 0 |
| 2 | `lambda i=1: i` | 1 |
| 3 | `lambda i=2: i` | 2 |

Beim Aufruf schaut das `i` im Body jetzt den *lokalen Parameter* nach (**L** in LEGB!) — nicht mehr die geteilte äußere Schleifenvariable.

> **Merksatz:** Eine Closure friert **die Variable ein, nicht den Wert** — und schaut den Wert erst beim *Aufruf* nach. Ändert sich die Variable bis dahin, sieht man den *neuen* Wert.

# Verbindung zu OOP: Objekte *sind* Namespaces

Das schließt den Kreis zum [[1.1 OOP - Leitfaden|OOP-Kapitel]]. Instanz und Klasse haben **je einen eigenen Namespace** (`__dict__`). Der Zugriff über den Punkt `obj.x` läuft **nicht** über LEGB, sondern durchsucht erst den Instanz-Namespace, dann den Klassen-Namespace, dann die Basisklassen (MRO):

```python
class Foo:
    kind = "A"          # landet im Klassen-Namespace (Foo.__dict__)
    def __init__(self):
        self.name = "x" # landet im Instanz-Namespace (self.__dict__)

f = Foo()
print(f.__dict__)       # {'name': 'x'}              <- nur Instanz-Attribute
print(Foo.__dict__)     # {..., 'kind': 'A', ...}    <- Klassen-Attribute
```

Bei `f.kind`: erst `f.__dict__` (nicht da) → dann `Foo.__dict__` (gefunden). Deshalb sind Klassenattribute „geteilt", Instanzattribute individuell — es sind schlicht *zwei verschiedene Namespaces*. Daher auch der berüchtigte Bug: ein *veränderbares* Klassenattribut (`items = []` auf Klassenebene) wird von **allen** Instanzen geteilt.

# Zusammenfassung

| Begriff | Was es ist | Bild |
|---|---|---|
| **Namespace** | Zuordnung Name → Objekt (ein Dict) | Das Telefonbuch |
| **Scope** | Wo/in welcher Reihenfolge Namespaces sichtbar sind | Welches Buch ich aufschlage |
| **LEGB** | Suchreihenfolge: Local → Enclosing → Global → Built-in | Die Suchroute |
| **`global` / `nonlocal`** | Zuweisung soll äußeren Namen treffen | Nach außen greifen |
| **Closure** | Funktion, die äußere Namen „einschließt" | Merkt sich die Variable, nicht den Wert |

> **Die drei tragenden Sätze:**
> 1. Ein Namespace ist ein Dict (Name → Objekt); ein Scope ist die *Sichtbarkeit* eines Namespace.
> 2. Namen werden per **LEGB von innen nach außen** aufgelöst; erster Treffer gewinnt.
> 3. **Zuweisung (`=`) macht einen Namen lokal** — für äußeres Schreiben braucht man `global`/`nonlocal`; reines *Mutieren* nicht.

# Knobelaufgaben

Erst selbst überlegen, was ausgegeben wird — dann die Lösung aufklappen.

## Aufgabe 1 — LEGB

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        print(x)
    inner()

outer()
```

> [!tip]- Lösung
> **`enclosing`.** In `inner` gibt es kein lokales `x` und keine Zuweisung → Python geht nach LEGB eine Ebene raus (**E**nclosing) und findet dort `"enclosing"`. Das globale `x` wird gar nicht erst erreicht, weil die Suche beim ersten Treffer stoppt.

## Aufgabe 2 — Der UnboundLocalError

```python
x = 10

def f():
    print(x)
    x = 20

f()
```

> [!tip]- Lösung
> **`UnboundLocalError`** (kein `10`!). Weil weiter unten `x = 20` steht, gilt `x` für die **ganze** Funktion als lokal — auch schon beim `print`. Dieses lokale `x` hat aber beim `print` noch keinen Wert. Fix: `global x` in die erste Zeile der Funktion, falls man wirklich das globale meint.

## Aufgabe 3 — Mutation vs. Rebinding

```python
liste = [1, 2, 3]
zahl = 0

def f():
    liste.append(4)   # (a)
    zahl = zahl + 1   # (b)

f()
print(liste, zahl)
```

> [!tip]- Lösung
> **`UnboundLocalError` in Zeile (b)** — die Funktion kommt gar nicht bis zum `print`. Zeile (a) ist reine **Mutation** (`.append`) und funktioniert problemlos ohne `global`. Zeile (b) ist aber ein **Rebinding** (`=`) → `zahl` wird lokal → `zahl + 1` liest ein noch wertloses lokales `zahl`. Mit `global zahl` würde es laufen und `[1, 2, 3, 4] 1` ausgeben.

## Aufgabe 4 — Die Closure-Schleife

```python
funcs = []
for i in range(3):
    def f():
        return i
    funcs.append(f)

print([f() for f in funcs])
```

> [!tip]- Lösung
> **`[2, 2, 2]`.** Genau die klassische Falle — auch ohne `lambda`, mit `def`, passiert dasselbe. Alle drei Funktionen schließen dieselbe Variable `i` ein und schauen sie erst beim Aufruf nach; da ist die Schleife durch und `i == 2`. Fix: den Wert einfrieren, z. B. `def f(i=i): return i`.

## Aufgabe 5 — nonlocal

```python
def counter():
    n = 0
    def increment():
        nonlocal n
        n += 1
        return n
    return increment

c = counter()
print(c(), c(), c())
```

> [!tip]- Lösung
> **`1 2 3`.** `increment` ist eine Closure über `n`. Dank `nonlocal n` verändert `n += 1` das `n` der umgebenden `counter`-Funktion (statt ein neues lokales `n` zu erzeugen). Weil beide `c()`-Aufrufe dieselbe Closure teilen, „merkt" sich `n` seinen Stand zwischen den Aufrufen — ein simpler Zähler ohne Klasse. (Ohne `nonlocal` gäbe es `UnboundLocalError`.)
