---
tags:
  - documentation
---
Zusammenfassung von Kapitel 9 („Classes") des offiziellen Python-Tutorials: wie Klassen-, Instanz- und Methodenobjekte funktionieren, Klassen- vs. Instanzvariablen (samt der Mutable-Falle), Vererbung/MRO, das `_`/`__`-Privacy-Modell — und ein ausführlicher Abschnitt zu `eval()` und `exec()`. Baut auf [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md|Scopes, Namespaces und Closures]] auf (dort stehen 9.1 Aliasing und 9.2 Scopes/LEGB im Detail) und ergänzt den [[1.1 OOP - Leitfaden|OOP-Leitfaden]].

# Namen & Objekte: Aliasing (9.1)

Mehrere Namen (in mehreren Scopes) können an **dasselbe Objekt** gebunden sein — in anderen Sprachen „Aliasing" genannt. Bei unveränderlichen Basistypen (Zahlen, Strings, Tupel) merkt man davon nichts. Bei **veränderlichen** Objekten (Listen, Dicts …) ist es aber wichtig, weil sich Aliase wie **Pointer** verhalten:

```python
def f(items):
    items.append(4)   # ändert das Objekt beim Aufrufer mit!

data = [1, 2, 3]
f(data)
print(data)           # [1, 2, 3, 4]
```

- **Objektübergabe ist billig**: intern wird nur ein Pointer übergeben, nicht kopiert.
- **Ändert eine Funktion ein übergebenes Objekt, sieht der Aufrufer die Änderung.** Deshalb braucht Python (anders als Pascal) keine zwei Übergabemechanismen.

> Verwandt: der Unterschied *Mutation vs. Rebinding* in der [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md#Verändern (mutate) ≠ Neu-Zuweisen (rebind)|Scopes-Notiz]].

# Klassen-, Instanz- und Methodenobjekt (9.3)

Beim Betreten einer `class`-Definition wird ein **neuer Namespace** angelegt und als lokaler Scope benutzt; alle Zuweisungen (auch `def`) landen darin. Am Ende der Definition entsteht ein **Klassenobjekt** — im Grunde ein Wrapper um diesen Namespace.

Drei Objekttypen sauber unterscheiden:

| Objekt | Was es ist | Beispiel |
|---|---|---|
| **Klassenobjekt** | das `class`-Ergebnis; unterstützt Attributzugriff + Instanziierung | `MyClass`, `MyClass.i` |
| **Instanzobjekt** | ein Exemplar, erzeugt per `MyClass()` | `x = MyClass()` |
| **Methodenobjekt** | eine an eine Instanz *gebundene* Funktion | `x.f` |

**Der `self`-Mechanismus (9.3.4):** Ein Methodenaufruf ist reine Umschreibung — der Aufruf `x.f()` ist **exakt äquivalent** zu `MyClass.f(x)`. Die Instanz wird automatisch als erstes Argument eingeschoben. `self` ist dabei nur **Konvention**, kein Schlüsselwort.

```python
class MyClass:
    def f(self):
        return "hello world"

x = MyClass()
x.f()            # "hello world"
MyClass.f(x)     # "hello world" — dasselbe
xf = x.f         # Methodenobjekt kann man speichern und später aufrufen
```

**Data attributes** müssen nicht deklariert werden — sie entstehen bei der ersten Zuweisung (`x.counter = 1`) und können mit `del` wieder verschwinden. (Details zum offenen `__dict__` in der Scopes-Notiz.)

# Klassen- vs. Instanzvariablen + die Mutable-Falle (9.3.5)

- **Instanzvariable** (`self.name = ...` in `__init__`) → einzigartig pro Objekt.
- **Klassenvariable** (direkt im Klassenkörper) → von **allen** Instanzen geteilt.

```python
class Dog:
    kind = "canine"          # Klassenvariable — geteilt
    def __init__(self, name):
        self.name = name     # Instanzvariable — individuell
```

⚠️ **Die Falle:** Ein *veränderbares* Klassenattribut wird von allen Instanzen geteilt — jede Mutation wirkt global:

```python
class Dog:
    tricks = []              # FALSCH als Klassenvariable
    def add_trick(self, t):
        self.tricks.append(t)

d, e = Dog(), Dog()
d.add_trick("roll over")
e.add_trick("play dead")
print(d.tricks)             # ['roll over', 'play dead'] — unerwartet geteilt!
```

✅ **Fix:** die Liste als Instanzvariable in `__init__` anlegen (`self.tricks = []`) → jedes Objekt bekommt seine eigene.

> **Merksatz:** Veränderbare Objekte (Listen, Dicts) gehören *fast nie* auf Klassenebene, sondern in `__init__`.

# Lookup-Priorität & Vererbung (9.4 – 9.5)

- **Instanz schlägt Klasse:** Kommt ein Name in beiden vor, gewinnt die Instanz (`w.region = "east"` überdeckt die Klassenvariable — aber nur für diese Instanz).
- **Attributsuche bei Vererbung:** nicht in der Klasse gefunden → weiter in der Basisklasse, rekursiv die Kette hoch.
- **Alle Methoden sind „virtual":** Ruft eine Basisklassen-Methode eine andere Methode desselben Objekts auf, landet man ggf. bei der **überschriebenen** Version der abgeleiteten Klasse. (C++-Sprech: alle Methoden effektiv virtual.)
- **Basisklassenmethode direkt aufrufen:** `BaseClassName.methodname(self, args)` — oder idiomatisch `super().methodname(args)`.
- **`isinstance(obj, T)`** prüft den Typ (inkl. Subklassen), **`issubclass(A, B)`** prüft Klassen-Vererbung.
- **Mehrfachvererbung (9.5.1):** vereinfacht „depth-first, links-rechts", real aber via **MRO** (Method Resolution Order, C3-Linearisierung) — jede Basisklasse genau einmal, monoton, damit `super()` kooperativ funktioniert.

> Tiefer dazu: [[Vererbung und Polymorphie - Tiefenperspektive|Vererbung und Polymorphie – Tiefenperspektive]].

# „Private" Variablen: nur Konvention + Name Mangling (9.6)

Echte private Variablen gibt es in Python **nicht** — es ist alles Konvention (siehe auch [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md|Scopes-Notiz]], „C-`struct` vs. Python-`__dict__`").

- **`_spam`** (ein Unterstrich): „nicht-öffentlicher" Teil der API, Implementierungsdetail — reine Absprache, technisch nicht geschützt.
- **`__spam`** (zwei Unterstriche): **Name Mangling.** Der Name wird *textuell* zu `_Klassenname__spam` umgeschrieben. Zweck ist **nicht** Geheimhaltung, sondern das Vermeiden von Namenskollisionen mit Subklassen — so kann eine Subklasse `update()` überschreiben, ohne einen internen `self.__update()`-Aufruf der Basisklasse zu brechen.

```python
class Mapping:
    def __init__(self, it):
        self.__update(it)     # wird zu self._Mapping__update(it)
    def update(self, it): ...
    __update = update         # private Kopie, gegen Subklassen-Override geschützt
```

Wichtig: Mangling verhindert nur *Unfälle* — man **kann** weiterhin über `_Klassenname__spam` drankommen.

# Iteratoren & Generatoren (9.8 – 9.10)

- **Iterator-Protokoll:** `for` ruft intern `iter(obj)` auf → liefert ein Objekt mit `__next__()`; ist nichts mehr da, wirft `__next__()` ein `StopIteration`. Eigene Klassen werden iterierbar, indem sie `__iter__()` (gibt ein Objekt mit `__next__()` zurück, oft `self`) und `__next__()` definieren.
- **Generatoren:** Funktionen mit `yield`. Bei jedem `next()` läuft die Funktion **da weiter, wo sie aufhörte** — lokale Variablen und Ausführungszustand bleiben automatisch erhalten. `__iter__()`/`__next__()` und das `StopIteration` am Ende werden **automatisch** erzeugt → viel kompakter als eine Iterator-Klasse.

```python
def reverse(data):
    for i in range(len(data) - 1, -1, -1):
        yield data[i]

list(reverse("golf"))   # ['f', 'l', 'o', 'g']
```

- **Generator-Ausdrücke:** wie List Comprehensions, aber mit runden Klammern; speicherfreundlich, weil sie Werte *lazy* liefern statt eine ganze Liste zu bauen: `sum(i*i for i in range(10))`.

# `eval()` und `exec()` im Detail

Kapitel 9.6 erwähnt beide nur beiläufig (im Kontext Name Mangling). Hier die gründliche Einordnung — sie hängen direkt an dem Namespace-Konzept aus der [[Documentation/Traineeship/Python Basics/Scopes, Namespaces und Closures.md|Scopes-Notiz]]: Beide führen Code in einem Namespace aus, den man ihnen als Dict mitgeben kann.

## Der Kernunterschied

| | `eval()` | `exec()` |
|---|---|---|
| Nimmt | **einen Ausdruck** (expression) | **beliebige Anweisungen** (statements) |
| Gibt zurück | **den Wert** des Ausdrucks | immer **`None`** |
| Kann | rechnen, Werte liefern | zuweisen, `def`, `import`, Schleifen … (Seiteneffekte) |

```python
eval("3 + 4 * 2")          # 14   — Ausdruck → Wert
eval("len('abc')")         # 3
# eval("x = 5")            # SyntaxError! Zuweisung ist kein Ausdruck

exec("y = 5")              # führt aus, y existiert jetzt; Rückgabe None
exec("def greet(): return 'hi'")
```

> **Merksatz:** `eval` = *ein* Ausdruck rein, *ein Wert* raus. `exec` = *Code* rein, *Seiteneffekte* raus (Rückgabe `None`).

## Der Namespace-Bezug: `globals` und `locals`

Beide akzeptieren optionale Dicts, die als **globaler** und **lokaler Namespace** dienen. Das sind buchstäblich die Namespaces aus deiner Scopes-Notiz — nur explizit übergeben:

```python
eval("x + y", {"x": 10, "y": 5})     # 15  (globals = {x, y})

ns = {}
exec("z = 2 ** 10", ns)              # legt z im Dict ns an
print(ns["z"])                       # 1024
```

Ohne diese Argumente laufen `eval`/`exec` im **aktuellen** globalen und lokalen Namespace — können also Variablen der Umgebung lesen (und `exec` sogar verändern).

## Die Quirk aus 9.6

Code, der an `eval()`/`exec()` übergeben wird, betrachtet die aufrufende **Klasse nicht als „current class"**. Deshalb greift dort das **Name Mangling nicht** wie erwartet — `__spam` wird *nicht* automatisch zu `_Klassenname__spam` umgeschrieben. Dieselbe Einschränkung gilt für `getattr()`, `setattr()`, `delattr()` und den direkten Zugriff auf `__dict__`. Grund: Mangling passiert beim Byte-Kompilieren *im Klassenkörper* — dynamisch übergebener String-Code ist davon abgekoppelt.

## ⚠️ Sicherheit — die wichtigste Warnung

`eval()`/`exec()` auf **nicht vertrauenswürdige Eingaben** anzuwenden ist eine der klassischsten Sicherheitslücken: Es bedeutet **beliebige Codeausführung**.

```python
eval(input())   # Nutzer tippt: __import__('os').system('rm -rf ~')  -> Desaster
```

Regel: **Niemals** `eval`/`exec` auf Nutzereingaben, Netzwerkdaten oder Dateiinhalte, die man nicht selbst kontrolliert.

## Sichere Alternativen (fast immer besser)

| Ziel | Statt `eval`/`exec` → | |
|---|---|---|
| String mit Literal (Liste, Dict, Zahl) parsen | `ast.literal_eval(s)` | wertet **nur** Literale aus, kein Code |
| Attribut dynamisch per Name holen/setzen | `getattr(obj, name)` / `setattr(...)` | |
| „String → Funktion" (Dispatch) | ein **Dict** `{"add": add_fn}` | |
| Zahl aus String | `int(s)` / `float(s)` | |

```python
import ast
ast.literal_eval("[1, 2, 3]")     # [1, 2, 3] — sicher
# ast.literal_eval("os.system('...')")  -> ValueError, kein Literal
```

**Legitime Einsätze** von `eval`/`exec` sind selten: REPLs, Debugger, dynamische Codegenerierung in Frameworks, Template-Engines. Für Anwendungscode gilt: Wenn du meinst, `eval` zu brauchen, gibt es fast immer eine sicherere, klarere Lösung.

# Zusammenfassung (Merksätze)

> 1. `x.f()` ist nur Kurzschrift für `MyClass.f(x)` — die Instanz wird als `self` eingeschoben; `self` ist Konvention.
> 2. Klassenvariablen sind geteilt, Instanzvariablen individuell — **veränderbare** Klassenvariablen sind fast immer ein Bug.
> 3. Instanz-Attribute schlagen Klassen-Attribute; alle Methoden sind effektiv *virtual*; Mehrfachvererbung nutzt die **MRO**.
> 4. „Privat" ist Konvention: `_spam` (bitte nicht anfassen), `__spam` (Name Mangling gegen Subklassen-Kollisionen, keine echte Sperre).
> 5. `eval` = ein Ausdruck → Wert; `exec` = Anweisungen → `None`; beide laufen in übergebbaren Namespace-Dicts — **nie** auf ungeprüfte Eingaben anwenden, lieber `ast.literal_eval`/`getattr`/Dispatch-Dict.
