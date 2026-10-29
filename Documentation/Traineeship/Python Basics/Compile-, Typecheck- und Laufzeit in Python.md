---
tags:
  - documentation
---
Überblick über die verschiedenen „Zeiten" (Phasen), die ein Stück Python-Code durchläuft — von der Editierzeit über die (optionale) statische Typprüfung bis zur Laufzeit. Kernaussage: Python ist **interpretiert und dynamisch**, deshalb passiert fast alle Fehlerprüfung erst zur **Laufzeit**. Ergänzt die praktischen Sprach-Eigenheiten in [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md|Python-Lernnotizen]].

# Der grundsätzliche Unterschied: Python ist *interpretiert* und *dynamisch*

In kompilierten Sprachen (C++, Java) gibt es eine klare Trennung:

1. **Compile-Zeit** → aus Quellcode wird Maschinen-/Bytecode, inhaltliche Fehler (Typen, unbekannte Namen) werden **hier** gefunden.
2. **Laufzeit** → das fertige Programm läuft.

Bei Python verschwimmt das. Es gibt zwar einen Kompilierschritt, aber der prüft fast **nichts** inhaltlich — die eigentliche „Wahrheit" entsteht erst zur **Laufzeit**.

> **Merksatz:** In Python ist fast alles Laufzeit.

# Die Phasen im Detail

## 1. Schreib-/Editierzeit („edit time")

Der Moment, in dem der Code **getippt** wird. Hilfe kommt hier nur vom Editor/der IDE:

- Autovervollständigung
- rote Schlängellinien
- Anzeige von Typ-Hinweisen

⚠️ Das ist noch **kein** echter Check durch Python selbst — es ist der Editor bzw. ein Tool im Hintergrund (siehe Phase 2).

## 2. Typecheck-Zeit („static type checking")

Ein **optionaler, separater** Schritt, der *nicht* von Python selbst ausgeführt wird. Bei Verwendung von Type Hints:

```python
def add(a: int, b: int) -> int:
    return a + b
```

…prüft ein **externes Tool** (`mypy`, `pyright`, `pyre`), ob die Typen zusammenpassen — **bevor** das Programm überhaupt startet.

Zentral zu verstehen:

- Python selbst **ignoriert** diese Type Hints zur Laufzeit komplett — sie sind reine Annotationen.
- Wird `mypy` nie ausgeführt, findet der Typ-Check **nie** statt.
- „Statische" Analyse heißt: der Code wird *gelesen*, aber **nicht ausgeführt**.

Großer Unterschied zu Java: dort erzwingt der Compiler die Typen. In Python ist der Typecheck freiwillig und läuft außerhalb des normalen Ablaufs.

> Verwandt: Auch eine nackte Annotation wie `self._x: int` legt zur Laufzeit **kein** Attribut an — siehe [[Documentation/Traineeship/Python Basics/Python-Lernnotizen.md#Typannotation ist keine Zuweisung|Python-Lernnotizen – Typannotation ist keine Zuweisung]].

## 3. Compile-Zeit / Import-Zeit (Bytecode-Kompilierung)

Beim Ausführen von `python mein_script.py` passiert zuerst intern:

- Python **parst** den Quellcode (Syntax-Analyse)
- und übersetzt ihn in **Bytecode** (die `.pyc`-Dateien im `__pycache__`-Ordner).

Was hier gefunden wird:

- ✅ **Syntaxfehler** (vergessener Doppelpunkt, falsche Einrückung)

Was hier **nicht** gefunden wird:

- ❌ ob eine Variable existiert
- ❌ ob ein Typ passt
- ❌ ob eine Funktion aufrufbar ist

```python
def foo():
    print(nicht_existierende_variable)  # KEIN Fehler beim Kompilieren!
```

Diese Zeile kompiliert problemlos. Der Fehler kommt erst, wenn `foo()` tatsächlich *aufgerufen* wird.

## 4. Laufzeit („runtime")

Hier läuft der Bytecode Zeile für Zeile. **Das meiste** an Fehlern zeigt sich erst jetzt:

- `NameError` (Variable existiert nicht)
- `TypeError` (falscher Typ, z. B. `"5" + 3`)
- `AttributeError` (Methode/Attribut existiert nicht)
- `ValueError`, `KeyError`, `ZeroDivisionError` …

Weil Python **dynamisch** ist, werden Typen und Namen erst in dem Moment aufgelöst, in dem eine Zeile ausgeführt wird. Deshalb kann ein Fehler in einer selten benutzten Funktion lange unentdeckt bleiben.

# Zusammenfassung als Ablauf

| Phase | Wer macht's? | Was wird gefunden? | Pflicht? |
|---|---|---|---|
| **Editierzeit** | Editor/IDE | visuelle Hinweise | nein |
| **Typecheck-Zeit** | `mypy`/`pyright` (extern) | Typ-Fehler | **optional** |
| **Compile-Zeit** | Python (`.pyc`) | nur Syntaxfehler | ja (automatisch) |
| **Laufzeit** | Python-Interpreter | fast alles andere | ja |

> **Merksatz:** In Python findet der Compiler nur Syntaxfehler. Typprüfung ist ein *freiwilliger* Extraschritt mit externen Tools, und alles Übrige knallt erst zur Laufzeit.
