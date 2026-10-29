---
tags:
  - documentation
---

# Leseliste — 2.3 Advanced Python

Kuratierte Leseaufgaben zu [[2.3 Advanced Python - Leitfaden]]. Pro Block ein **Core**-Teil (Pflicht) und ein **Vertiefung**-Teil (optional, jeweils markiert als *andere Perspektive* oder *weiterführend*).

Ablauf: lesen → beim Lesen roh in [[2.3 Advanced Python - Vertiefung|die Vertiefung]] exzerpieren (pro Quelle unter ihrer Überschrift, mit einem Satz in eigenen Worten) → Kontrollfragen dort **aus dem Kopf** beantworten → Review durch Claudian → Claudian arbeitet den Abschnitt aus. Offene Fragen wandern nach [[Q&A/Fragen|Q&A/Fragen]].

> [!info] Schlanker als 2.2 — mit Absicht
> Pro Block **zwei bis vier Core-Quellen und höchstens 90 Minuten**, insgesamt **~7 h Core** für das ganze Kapitel. Bei Buchkapiteln stehen Seitenzahlen, bei Webseiten die Abschnitte. Pro Quelle gibt es **einen** Leseauftrag. Was aus dem Core geflogen ist, steht am Blockende unter *Nicht im Core* — die Vertiefungsnotiz deckt es trotzdem ab.
> Alle Angaben wurden am 02.10.2026 geprüft. **Versionsstand:** Python **3.14.8** ist die aktuelle stabile Version. **3.15** steht als RC2 bereit; das finale Release war für den 01.10. angesetzt, ist aber noch nicht erschienen.

## Die sechs Blöcke

| Block | Stichworte (Leitfaden) | Kontrollfragen | Core | Status |
|---|---|---|---|---|
| **[[#Block 1 — Concurrency vs. Parallelismus & Thread vs. Prozess\|1]]** | 2 Concurrency vs. Parallelismus · 3 Thread vs. Process | 3, 4 | ~80 Min. | ✅ fertig |
| **[[#Block 2 — Warum Python langsam ist & der GIL\|2]]** | 1 GIL | 1, 2 | ~45 Min. | ✅ fertig |
| **[[#Block 3 — Threads und Prozesse in Python\|3]]** | 10 ThreadPool & ProcessPool Executors · 11 multiprocessing vs. multithreading | 8 | ~85 Min. | ✅ fertig |
| **[[#Block 4 — Generators, Chunking & Memory Management\|4]]** | 5 Chunking & Memory Management · 6 Generators | 6 | ~65 Min. | ✅ fertig |
| **[[#Block 5 — Event Loop, asyncio & asynchrone Programmierung\|5]]** | 4 Single- vs. Multi-threaded Concurrency · 7 Event Loop · 8 asyncio · 9 asynchrone Programmierung | 5, 7 | ~80 Min. | ✅ fertig |
| **[[#Block 6 — SQL vs. Python für ETL\|6]]** | — (Kontrollfrage 9, Vorbereitung der Übungsaufgabe) | 9 | ~50 Min. | ✅ fertig |

**Übungsaufgabe:** Aufgabe 1 (Experimentieren) läuft nebenher, gern pro Block. Aufgabe 2 (SQL-Pipeline in Python neu schreiben) folgt nach Block 6 — den groben Fahrplan lege ich an, wenn du so weit bist. Bei Aufgabe 3 (Python Challenge) **endet der Leitfaden nach dem Doppelpunkt** — da fehlt vermutlich ein Link.

## Die Bücher

| Kürzel | Buch | Wofür |
|---|---|---|
| **Fowler** | Matthew Fowler, *Python Concurrency with asyncio* (Manning, 2022) — [O'Reilly](https://www.oreilly.com/library/view/python-concurrency-with/9781617298660/) | **Kap. 1 ist der rote Faden durch die Blöcke 1, 2 und 5** — 22 Seiten Überblick über das ganze Kapitel |
| **FP2** | Luciano Ramalho, *Fluent Python*, **2. Auflage** (O'Reilly, 2022) — [O'Reilly](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/) | Block 3 Core, sonst Vertiefung. Eine 3. Auflage ist bisher nicht erschienen |
| **OSTEP** | Arpaci-Dusseau, *Operating Systems: Three Easy Pieces* — [frei online](https://pages.cs.wisc.edu/~remzi/OSTEP/) | Block 1: Threads aus Sicht des Betriebssystems |
| **FDE** | Reis & Housley, *Fundamentals of Data Engineering* — liegt als [[fundamentals_data_engineering.pdf\|PDF]] in `Quellen/weiterführend/` | Block 6. Seitenangaben = **PDF-Seiten** dieser Datei |
| **HPP3** | Gorelick & Ozsvald, *High Performance Python*, **3. Auflage** (O'Reilly, 2025) — [O'Reilly](https://www.oreilly.com/library/view/high-performance-python/9781098165956/) | Nur Vertiefung. Gedruckte Seitenzahlen nicht belegbar, deshalb Abschnittstitel |

> [!note] Wie aktuell die Bücher sind
> Fowler und FP2 stammen aus der Zeit von Python 3.10. **`TaskGroup` (3.11) und free-threaded Python (3.13/3.14) kommen darin nicht vor** — dafür steht in Block 2 und 5 jeweils die offizielle Doku im Core.

---

## Block 1 — Concurrency vs. Parallelismus & Thread vs. Prozess

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt (8 sicher · 8 teilweise · 0 falsch · 1 übersprungen), [[2.3 Advanced Python - Vertiefung#1. Concurrency vs. Parallelismus & Thread vs. Prozess|Vertiefung ausgearbeitet]], 32 Anki-Karten erstellt, Q&A beantwortet.

**Core ~80 Min. · Kontrollfragen 3, 4**

**Warum das zählt:** Ohne dieses Vokabular ist jede Diskussion über GIL, Threads und asyncio Rätselraten. Zwei Unterscheidungen tragen das ganze Kapitel: *Concurrency vs. Parallelismus* (Struktur vs. Ausführung) und *I/O-bound vs. CPU-bound* (wartet das Programm, oder rechnet es?). Die zweite entscheidet in der Praxis, welches Werkzeug du nimmst.

### Core (in dieser Reihenfolge)

- [x] **1. Fowler, Kap. 1 „Getting to know asyncio" — Abschnitte 1.2–1.4** (S. 3–12, ~25 Min.)
      I/O-bound vs. CPU-bound · Concurrency, Parallelismus, Multitasking · Prozesse und Threads. Abschnitt 1.1 darfst du überfliegen.
      *Leseauftrag: Was ist der Unterschied zwischen präemptivem und kooperativem Multitasking — und wer entscheidet jeweils, wann gewechselt wird?*
- [x] **2. Rob Pike, „Concurrency is not Parallelism"** (Heroku Waza 2012, Video 31 Min.) — <https://vimeo.com/49718712> · [Folien](https://go.dev/talks/2012/waza.slide)
      Der Klassiker zur Begriffstrennung. Die Gopher-Beispiele sind der Kern; den Go-Code in der zweiten Hälfte darfst du überfliegen.
      *Leseauftrag: Warum nennt Pike Concurrency eine Eigenschaft der **Struktur** und Parallelismus eine der **Ausführung**?*
- [x] **3. OSTEP, Kap. 26 „Concurrency: An Introduction" — 26.1–26.7** (~25 Min.) — <https://pages.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf>
      Was ein Thread aus Sicht des Betriebssystems ist und warum geteilter Speicher gefährlich wird. Die Hausaufgaben am Ende überspringen.
      *Leseauftrag: Warum ergibt `counter = counter + 1` in zwei Threads nicht zuverlässig das erwartete Ergebnis?*

### Vertiefung (optional)

- [ ] **FP2, Kap. 19 — „A Bit of Jargon"** (S. 697–701, ~10 Min.) · *andere Perspektive*
      Dieselben Begriffe aus Python-Sicht, ergänzt um *Supervisor*, *Worker*, *Contention*. Der Unterabschnitt zum GIL leitet zu Block 2 über.
- [ ] **OSTEP, Kap. 4 „The Abstraction: The Process" — 4.1–4.4** (~20 Min.) · *andere Perspektive* — <https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf>
      Was ein Prozess ist: Adressraum, Register, Prozesszustände (running/ready/blocked).
- [ ] **OSTEP, Kap. 5 „Interlude: Process API" — 5.1–5.4** (~20 Min.) · *weiterführend* — <https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf>
      `fork()` und `exec()`. Das ist die Grundlage für die Startmethoden `fork`/`spawn`/`forkserver` in Block 3.

*Nicht im Core:* Amdahls Gesetz · Locks und Deadlocks als Lösung · Shared Memory vs. Message Passing · Prozesszustände und Scheduler · Kerne vs. logische CPUs.

---

## Block 2 — Warum Python langsam ist & der GIL

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt (4 sicher · 9 teilweise · 5 falsch), [[2.3 Advanced Python - Vertiefung#2. Warum Python langsam ist & der GIL|Vertiefung ausgearbeitet]], 27 Anki-Karten erstellt, Q&A beantwortet.

**Core ~45 Min. · Kontrollfragen 1, 2**

**Warum das zählt:** Das ist die Motivationsfrage des Leitfadens. Wer weiß, *woher* die Langsamkeit kommt (dynamische Typen, Interpreter, verstreute Objekte im Speicher), versteht auch, warum NumPy, Polars und DuckDB schnell sind: Sie verlassen Python für die eigentliche Arbeit. Der GIL ist der zweite Teil der Antwort — und gerade in Bewegung: Seit 3.14 gibt es offiziell eine Variante ohne ihn.

### Core (in dieser Reihenfolge)

- [x] **1. Jake VanderPlas, „Why Python is Slow: Looking Under the Hood"** (2014) — nur der erste Teil „Why Python is Slow" bis einschließlich „So Why Use Python?" (~10 Min.) — <https://jakevdp.github.io/blog/2014/05/09/why-python-is-slow/>
      *Leseauftrag: Was genau passiert bei `a + b` in Python, das in C nicht passiert?*
- [x] **2. Tim Hopper, „Why is Python Slow?"** (pydevtools, 2026, ~12 Min.) — <https://pydevtools.com/handbook/explanation/why-is-python-slow/>
      Die aktuelle Fortsetzung zu VanderPlas: adaptiver Interpreter seit 3.11, JIT, C-Erweiterungen, und warum Type Hints nichts beschleunigen.
      *Leseauftrag: Welche der drei Ursachen von VanderPlas haben die neuen Python-Versionen angegangen — und welche nicht?*
- [x] **3. Fowler, Kap. 1 — Abschnitt 1.5 „Understanding the global interpreter lock"** (S. 12–17, ~12 Min.)
      *Leseauftrag: Welches Problem löst der GIL — und warum wird er bei I/O trotzdem freigegeben?*
- [x] **4. Python-Doku: der aktuelle Stand in drei kurzen Stellen** (~10 Min.)
      - Glossar: [global interpreter lock](https://docs.python.org/3/glossary.html#term-global-interpreter-lock) und [free threading](https://docs.python.org/3/glossary.html#term-free-threading)
      - What's New 3.14: [Free-threaded Python is officially supported](https://docs.python.org/3/whatsnew/3.14.html#free-threaded-python-is-officially-supported)
      - HOWTO [Python support for free threading](https://docs.python.org/3/howto/free-threading-python.html) — bis einschließlich „Known limitations"
      *Leseauftrag: Was heißt „officially supported, but still optional" konkret — und was kostet der free-threaded Build bei Code mit nur einem Thread?*

### Vertiefung (optional)

- [ ] **Miguel Grinberg, „Python 3.14 Is Here. How Fast Is It?"** (Okt. 2025, ~13 Min.) · *weiterführend* — <https://blog.miguelgrinberg.com/post/python-3-14-is-here-how-fast-is-it>
      Echte Messungen statt Versprechen: Standard-Build, free-threaded Build und JIT im Vergleich — bei welcher Art von Programm der free-threaded Build etwas bringt und bei welcher er schadet.
- [ ] **PEP 703 „Making the Global Interpreter Lock Optional in CPython" — Abstract + Motivation** (~11 Min.) · *weiterführend* — <https://peps.python.org/pep-0703/>
      Warum der GIL weichen sollte, aus Sicht des Autors — mit Beispielen aus Data Science und KI. Zusätzlich lohnt der kurze Abschnitt *Alternatives › Multiprocessing*.
- [ ] **David Beazley, „Understanding the Python GIL"** (PyCon 2010, Video 46 Min.) · *andere Perspektive* — <https://www.youtube.com/watch?v=Obt-vMVdM8s>
      Die Mechanik des GIL-Wechsels zwischen Threads (Switch Interval, Konvoi-Effekt). Stand 2010, also ohne free-threaded Python.
- [ ] **HPP3, Kap. 1 — „Idealized Computing Versus the Python Virtual Machine" und „The Future of Python"** · *andere Perspektive*
      Die Hardware-Sicht auf die Langsamkeit, dazu die Unterabschnitte *Where Did the GIL Go?* und *Does Python Have a JIT?*.
- [ ] **Thinh Dang, „Free-Threaded Python on a Real Workload"** (Sept. 2026, ~9 Min.) · *weiterführend* — <https://thinhdanggroup.github.io/measuring-free-threaded-python/>
      Messung an einer echten Arbeitslast, inklusive der Falle, dass eine C-Erweiterung den GIL stillschweigend wieder einschaltet.

*Nicht im Core:* Referenzzählung und GIL im Detail · Subinterpreter (kommt in Block 3) · PyPy, Cython, Numba · Stand von 3.15 (JIT schneller, keine Entscheidung über free-threaded als Standard).

---

## Block 3 — Threads und Prozesse in Python

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt (6 sicher · 7 teilweise · 3 falsch), [[2.3 Advanced Python - Vertiefung#3. Threads und Prozesse in Python|Vertiefung ausgearbeitet]], 28 Anki-Karten erstellt.

**Core ~85 Min. · Kontrollfrage 8 · Stichwort 4 (Multi-threaded) wird hier vorbereitet**

**Warum das zählt:** Hier wird aus Begriffen Code. `threading` und `multiprocessing` sind die beiden Wege zu Concurrency mit mehreren Ausführungseinheiten. `concurrent.futures` ist die bequeme Schicht darüber, die du in der Praxis fast immer nimmst. Die eigentliche Lektion steckt in den Messungen: Threads helfen bei CPU-Arbeit nicht, Prozesse helfen — aber nicht umsonst.

### Core (in dieser Reihenfolge)

- [x] **1. FP2, Kap. 19 „Concurrency Models in Python" — S. 701–725** (~40 Min.)
      „A Concurrent Hello World" (Spinner mit Threads und mit Prozessen), „The Real Impact of the GIL", „A Homegrown Process Pool". **„Spinner with Coroutines" (S. 706–711) überspringen** — der kommt in Block 5 dran.
      *Leseauftrag: Warum ist die Thread-Variante des Primzahl-Checkers eine „Nonsolution"?*
- [x] **2. FP2, Kap. 20 „Concurrent Executors" — S. 744–758** (~30 Min.)
      „Concurrent Web Downloads" (inkl. „Where Are the Futures?") und „Launching Processes with concurrent.futures".
      *Leseauftrag: Was ist ein Future, und warum erzeugst du ihn praktisch nie selbst?*
- [x] **3. Python-Doku: Executors und Startmethoden** (~15 Min.)
      - `concurrent.futures`: [ThreadPoolExecutor](https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor) und [ProcessPoolExecutor](https://docs.python.org/3/library/concurrent.futures.html#processpoolexecutor), jeweils ohne die Beispiele
      - `multiprocessing`: [Contexts and start methods](https://docs.python.org/3/library/multiprocessing.html#contexts-and-start-methods) und [Programming guidelines](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming)
      *Leseauftrag: Was muss ein Objekt können, damit es an einen anderen Prozess geht? Was hat sich in 3.14 an der Standard-Startmethode unter Linux geändert?*

### Vertiefung (optional)

- [ ] **Real Python, „Speed Up Your Python Program With Concurrency"** (Nov. 2024, ~40 Min.) · *andere Perspektive* — <https://realpython.com/python-concurrency/>
      Dasselbe an zwei Beispielprogrammen (I/O-bound, CPU-bound) in je vier Varianten: synchron, Threads, asyncio, Prozesse. Gut zum Mittippen. Die asyncio-Varianten passen auch zu Block 5.
- [ ] **FP2, Kap. 20 — „Experimenting with Executor.map" und „Using futures.as_completed"** (S. 758–762 und ab S. 769, ~15 Min.) · *weiterführend*
      Warum `map` die Ergebnisse in Eingabereihenfolge liefert und wann `as_completed` die bessere Wahl ist.
- [ ] **Python-Doku `concurrent.interpreters`** (~8 Min.) · *weiterführend* — <https://docs.python.org/3/library/concurrent.interpreters.html>
      Der dritte Weg seit 3.14: Subinterpreter, „die Isolation von Prozessen mit der Effizienz von Threads".
- [ ] **FP2, Kap. 19 — „Python in the Multicore World"** (S. 725–733, ~15 Min.) · *weiterführend*
      Wie Python in der Praxis mit vielen Kernen umgeht — der Teil „Data Science" ist für dich der relevante.

*Nicht im Core:* Locks, Queues und `threading`-Primitive im Detail · Shared Memory zwischen Prozessen · `chunksize` bei `ProcessPoolExecutor.map` · Abbruch und Fehlerbehandlung in Pools.

---

## Block 4 — Generators, Chunking & Memory Management

> [!success] Abgeschlossen
> Kontrollfragen beantwortet und reviewt (7 sicher · 8 teilweise · 2 falsch), [[2.3 Advanced Python - Vertiefung#4. Generators, Chunking & Memory Management|Vertiefung ausgearbeitet]], 25 Anki-Karten erstellt.

**Core ~65 Min. · Kontrollfrage 6**

**Warum das zählt:** Der häufigste Grund, warum eine Python-Pipeline abstürzt, ist nicht Langsamkeit, sondern zu wenig Speicher: Jemand hat die ganze Datei oder die ganze Tabelle auf einmal geladen. Generators sind Pythons Werkzeug, um Daten Stück für Stück durch eine Pipeline zu schieben — dasselbe Prinzip wie die Unix-Pipeline aus [[2.1 Advanced SQL & ETL - Vertiefung#1.1 Was ist eine Pipeline? Die Unix-Wurzel|2.1, Abschnitt 1.1]]. Dazu kommt `yield` als Konzept: eine Funktion, die anhalten und später weiterlaufen kann. Das ist die Grundlage für asyncio in Block 5.

### Core (in dieser Reihenfolge)

- [x] **1. Python-Doku: Iteratoren und Generators** (~20 Min.)
      - Tutorial [9.8 Iterators, 9.9 Generators, 9.10 Generator Expressions](https://docs.python.org/3/tutorial/classes.html#iterators) — zum Aufwärmen
      - Functional Programming HOWTO: [Iterators](https://docs.python.org/3/howto/functional.html#iterators), „Generator expressions and list comprehensions", „Generators" — „Passing values into a generator" überfliegen
      *Leseauftrag: Was ist der Unterschied zwischen einem Iterable und einem Iterator? Was passiert mit den lokalen Variablen einer Generator-Funktion bei `yield`?*
- [x] **2. David Beazley, „Generator Tricks for Systems Programmers"** (Version 3, 2018) — Teil 1 und 2, Folien 1–45 (~25 Min.) — [Archivkopie (PDF)](http://web.archive.org/web/20260218130556/http://www.dabeaz.com/generators/Generators.pdf)
      Generators als Datenpipeline am Beispiel von Webserver-Logs. Beazley hat seine Folien inzwischen [hinter ein Formular](http://www.dabeaz.com/access.html) gelegt; die Archivkopie ist dieselbe Datei.
      *Leseauftrag: Wie viel Speicher braucht seine Pipeline — und wovon hängt das ab, und wovon nicht?*
- [x] **3. pandas-Doku, „Scaling to large datasets"** (~10 Min.) — <https://pandas.pydata.org/docs/user_guide/scale.html>
      *Leseauftrag: Welche drei Hebel nennt die Seite, bevor sie zu Chunking kommt — und in welcher Reihenfolge würdest du sie ziehen?*
- [x] **4. psycopg-3-Doku: Server-side cursors** (~8 Min.)
      Der Abschnitt [Server-side cursors](https://www.psycopg.org/psycopg3/docs/advanced/cursors.html#server-side-cursors) und in der API-Referenz [`Cursor.stream()`](https://www.psycopg.org/psycopg3/docs/api/cursors.html#psycopg.Cursor.stream). Knüpft an [[psycopg 3 Grundlagen]] an.
      *Leseauftrag: Wo liegt das Ergebnis einer Query bei einem normalen Cursor, wo bei einem Server-side Cursor?*

### Vertiefung (optional)

- [ ] **Python-Doku `tracemalloc` — Examples** (~5 Min.) · *weiterführend* — <https://docs.python.org/3/library/tracemalloc.html#examples>
      Nur „Display the top 10" und „Compute differences". Das Werkzeug, um Speicherbedarf zu messen statt zu raten — spätestens in der Übungsaufgabe nützlich.
- [ ] **FP2, Kap. 17 „Iterators, Generators, and Classic Coroutines" — S. 593–619** (~50 Min.) · *andere Perspektive*
      Gründlicher als die Doku, mit „How a Generator Works" (S. 607). Ab „Generator Functions in the Standard Library" wird es zum Nachschlagewerk.
- [ ] **HPP3, Kap. 5 „Iterators and Generators"** · *andere Perspektive*
      Dieselbe Idee aus der Performance-Sicht: Lazy Evaluation und was sie an Speicher spart.
- [ ] **HPP3, Kap. 12 „Using Less RAM" — „Objects for Primitives Are Expensive" und „Understanding the RAM Used in a Collection"** · *weiterführend*
      Warum eine Python-Liste mit einer Million Zahlen ein Vielfaches eines NumPy-Arrays belegt.
- [ ] **Python-Doku, Functional HOWTO — „The itertools module"** (~10 Min.) · *weiterführend* — <https://docs.python.org/3/howto/functional.html>
      `islice`, `chain`, `groupby` und Co. — der Baukasten für Generator-Pipelines. `itertools.batched` (seit 3.12) ist Chunking in einer Zeile.

*Nicht im Core:* Referenzzählung und Garbage Collector · Speicherbedarf von Python-Objekten im Detail · `yield from` · Chunking in Polars und DuckDB (kommt in Block 6).

---

## Block 5 — Event Loop, asyncio & asynchrone Programmierung

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt (9 sicher · 6 teilweise · 3 falsch), [[2.3 Advanced Python - Vertiefung#5. Event Loop, asyncio & asynchrone Programmierung|Vertiefung ausgearbeitet]], 27 Anki-Karten erstellt.

**Core ~80 Min. · Kontrollfragen 5, 7**

**Warum das zählt:** asyncio ist die dritte Antwort auf Concurrency — ein einziger Thread, der zwischen vielen wartenden Aufgaben hin- und herspringt. Du kennst das schon von außen: In [[FastAPI, Uvicorn & Pydantic - Syntax|1.5]] stand die Faustregel „`def` läuft im Threadpool, `async def` im Event Loop — dort darf nichts blockieren", und in [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/Q&A/Weiterführende Stichpunkte|den weiterführenden Stichpunkten aus 1.5]] ist „async/await und der Event Loop" als 🔴 offen markiert. Hier kommt der Unterbau.

### Core (in dieser Reihenfolge)

- [x] **1. Fowler, Kap. 1 — Abschnitte 1.6–1.7** (S. 17–22, ~12 Min.)
      „How single-threaded concurrency works" und „How an event loop works" — damit ist Kap. 1 komplett.
      *Leseauftrag: Wie kann ein einziger Thread auf hundert Netzwerkantworten gleichzeitig warten?*
- [x] **2. Python-Doku, HOWTO „A Conceptual Overview of asyncio"** (Teil 1 + 2, ~25 Min.) — <https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html>
      Der beste Einstieg in asyncio. Teil 2 („the nuts and bolts") zeigt, wie Coroutinen auf Generators aufbauen — der Rückbezug auf Block 4.
      *Leseauftrag: Was ist der Unterschied zwischen einer Coroutine und einem Task? Was tut `await` tatsächlich?*
- [x] **3. Python-Doku: asyncio-Referenz in Auswahl** (~25 Min.)
      - [Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html) — nur Coroutines, Awaitables, Creating Tasks, Task Groups, Sleeping, Running Tasks Concurrently, Timeouts, Running in Threads
      - [Developing with asyncio](https://docs.python.org/3/library/asyncio-dev.html) — Debug Mode, Concurrency and Multithreading, [Running Blocking Code](https://docs.python.org/3/library/asyncio-dev.html#running-blocking-code)
      *Leseauftrag: Was passiert mit allen anderen Tasks, wenn einer `time.sleep(5)` aufruft — und wie bringt man blockierenden Code trotzdem unter?*
- [x] **4. FastAPI-Doku, „Concurrency and async / await"** (~18 Min.) — <https://fastapi.tiangolo.com/async/>
      Die Burger-Analogie für Concurrency vs. Parallelismus, danach „Very Technical Details": wann FastAPI einen Threadpool benutzt.
      *Leseauftrag: Deine 24 Routen in der Pricing-API sind alle `def` — warum war das richtig?*

### Vertiefung (optional)

- [ ] **FP2, Kap. 21 „Asynchronous Programming" — S. 777–787 und S. 825–827** (~30 Min.) · *andere Perspektive*
      „A Few Definitions" bis „Asynchronous Context Managers", dazu „How Async Works and How It Doesn't" mit „The Myth of I/O-Bound Systems" und „Avoiding CPU-Bound Traps". Davor lohnt sich der übersprungene „Spinner with Coroutines" aus Kap. 19 (S. 706–713).
- [ ] **Łukasz Langa, „import asyncio" — Folge 2 „The Event Loop"** (EdgeDB, 2020, Video 36 Min.) · *andere Perspektive* — [Playlist](https://www.youtube.com/playlist?list=PLhNSoGM2ik6SIkVGXWBwerucXjgP1rHmB)
      Der Event Loop live erklärt von einem CPython-Core-Entwickler. Älter als `TaskGroup`.
- [ ] **Bob Nystrom, „What Color is Your Function?"** (2015, ~17 Min.) · *weiterführend* — <https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/>
      Warum `async` sich durch eine ganze Codebasis „durchfrisst". Beispiele in JavaScript, das Argument gilt für Python genauso.
- [ ] **Nathaniel J. Smith, „Notes on structured concurrency, or: Go statement considered harmful"** (2018, ~32 Min.) · *weiterführend* — <https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/>
      Die Idee hinter `asyncio.TaskGroup`.

*Nicht im Core:* Event Loop auf Betriebssystemebene (`select`/`epoll`) · async Generators und async Context Managers · asyncio mit Prozess-Pools kombinieren · async SQLAlchemy.

---

## Block 6 — SQL vs. Python für ETL

> [!success] Abgeschlossen
> Core gelesen, Kontrollfragen beantwortet und reviewt (12 sicher · 4 teilweise · 3 falsch oder offen), [[2.3 Advanced Python - Vertiefung#6. SQL vs. Python für ETL|Vertiefung ausgearbeitet]], 25 Anki-Karten erstellt. Damit sind alle sechs Blöcke abgeschlossen.

**Core ~50 Min. · Kontrollfrage 9 · Vorbereitung der Übungsaufgabe**

**Warum das zählt:** Die Kontrollfrage schließt an deinen eigenen Vergleich aus 2.1 an ([[Umsetzungsstand (SQL-Portierung)]]) — dort ist das Fazit noch offen. Die Übungsaufgabe stellt dieselbe Frage noch einmal, diesmal mit DataFrame-Bibliotheken auf der Python-Seite. Die wichtigste Einsicht des Blocks: Polars im Lazy-Modus und DuckDB sind *deklarativ* wie SQL, mit eigenem Query-Optimizer. Die Front verläuft also nicht mehr zwischen „SQL" und „Python", sondern zwischen deklarativ und zeilenweise-imperativ — siehe [[2.1 Advanced SQL & ETL - Vertiefung#3.5 Zeilenweise vs. mengenorientiert denken|2.1, Abschnitt 3.5]].

### Core (in dieser Reihenfolge)

- [x] **1. FDE, Kap. 8 — „SQL and general-purpose code-transformation tools"** ([[fundamentals_data_engineering.pdf#page=441|PDF-S. 441–442]], ~6 Min.) **und Kap. 1 — Kasten „The Unreasonable Effectiveness of SQL"** ([[fundamentals_data_engineering.pdf#page=42|PDF-S. 42]], ~2 Min.)
      Im PDF über das Lesezeichen „Batch Transformations" (S. 438) springen.
      *Leseauftrag: Welche drei Fragen soll man sich laut den Autoren stellen, bevor man eine Transformation in Python statt SQL schreibt?*
- [x] **2. Joseph Machado, „SQL or Python for Data Transformations?"** (Start Data Engineering, 2024, ~10 Min.) — <https://www.startdataengineering.com/post/sql-v-python/>
      Ausgewogen: SQL und DataFrames sind nur zwei Schnittstellen zu einer Ausführungs-Engine.
      *Leseauftrag: Nach welchen Kriterien wählt er die Engine, nach welchen die Schnittstelle?*
- [x] **3. Robin Linacre, „SQL should be the default choice for data transformation logic"** (2023, ~10 Min.) — <https://www.robinlinacre.com/recommend_sql/>
      Die Gegenposition, klar pro SQL, aber mit einem Abschnitt zu den Ausnahmen. Zum Abschluss den letzten Absatz von „Limitations" in der [dbt-Doku zu Python models](https://docs.getdbt.com/docs/build/python-models) lesen (~2 Min.) — die Faustregel eines Werkzeugs, das beides kann.
      *Leseauftrag: Wo widersprechen sich Machado und Linacre beim Thema **Testbarkeit** — und wer hat für deinen Fall recht?*
- [x] **4. Die Werkzeuge für die Übungsaufgabe: Polars und DuckDB** (~20 Min.)
      - Polars User Guide: [Expressions and contexts](https://docs.pola.rs/user-guide/concepts/expressions-and-contexts/) und [Lazy API](https://docs.pola.rs/user-guide/concepts/lazy-api/)
      - DuckDB: [Python API](https://duckdb.org/docs/current/clients/python/overview.html) (bis „Writing Data to Disk") und [SQL on Pandas](https://duckdb.org/docs/current/guides/python/sql_on_pandas.html)
      *Leseauftrag: Was macht der Lazy-Modus von Polars, das pandas nicht kann? Und was bleibt von „SQL vs. Python" übrig, wenn DuckDB SQL direkt auf einem DataFrame ausführt?*

> [!warning] Polars 2.0 steht vor der Tür
> Aktuell stabil ist Polars **1.44**. **2.0** ist als Release Candidate draußen: Dort läuft der Lazy-Modus standardmäßig über die Streaming-Engine, und `group_by` und Joins garantieren **keine Zeilenreihenfolge** mehr. Für den Ergebnisvergleich in der Übungsaufgabe also vorher explizit sortieren.

### Vertiefung (optional)

- [ ] **Polars User Guide, „Coming from Pandas"** (~10 Min.) · *andere Perspektive* — <https://docs.pola.rs/user-guide/migration/pandas/>
      Die Unterschiede im Denkmodell, nicht nur in der Syntax — hilfreich, falls du zwischen pandas und Polars schwankst.
- [ ] **Anna Geller, „Polars, DuckDB, Pandas, Fugue, Daft: Which Dataframe and SQL Tool Is Best?"** (Kestra, ~17 Min.) · *andere Perspektive* — <https://kestra.io/blogs/dataframes>
      Dieselbe kleine Pipeline in jedem Werkzeug. Einige Links und Werkzeuge sind veraltet.
- [ ] **Polars „Lazy" — Optimizations, Query plan, Query execution + DuckDB „Larger-than-Memory Workloads"** (~12 Min.) · *weiterführend*
      [Optimizations](https://docs.pola.rs/user-guide/lazy/optimizations/) · [Query plan](https://docs.pola.rs/user-guide/lazy/query-plan/) · [Query execution](https://docs.pola.rs/user-guide/lazy/execution/) · [DuckDB Out-of-Core](https://duckdb.org/docs/current/guides/performance/how_to_tune_workloads.html). Für den Performance-Teil der Übung: Projection und Predicate Pushdown sind dieselben Ideen wie in [[2.2 OLAP Datenmodellierung & Datawarehousing - Vertiefung#1.5 Zeilen- vs. spaltenorientierte Speicherung|2.2, Abschnitt 1.5]].
- [ ] **Benchmarks: [db-benchmark](https://duckdblabs.github.io/db-benchmark/) (DuckDB Labs, Stand Aug. 2026) und [Polars PDS-H](https://pola.rs/posts/benchmarks/) (Mai 2025)** · *weiterführend*
      Zur Einordnung deiner eigenen Messungen. Beide stammen von Herstellern — mit entsprechendem Vorbehalt lesen.

*Nicht im Core:* Spark und verteilte DataFrames · dbt Python models im Detail · HPP3 Kap. 7 „Pandas, Dask, and Polars".
