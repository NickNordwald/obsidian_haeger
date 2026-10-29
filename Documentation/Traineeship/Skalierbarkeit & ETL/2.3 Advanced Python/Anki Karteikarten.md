---
deck: "Traineeship::Skalierbarkeit & ETL::Advanced Python"
---

Karteikarten zu [[2.3 Advanced Python - Vertiefung|2.3 Advanced Python – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen.

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Skalierbarkeit & ETL::Advanced Python`. Die `oid`-Attribute werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr. Innerhalb eines Blocks stehen die Punkte, die Nick beim Lesen selbst exzerpiert hat, **zuerst** — sie haben Vorrang und bekommen im Zweifel eine eigene Karte.

# Fortschritt

- [x] Block 1 — Concurrency vs. Parallelismus & Thread vs. Prozess
- [x] Block 2 — Warum Python langsam ist & der GIL
- [x] Block 3 — Threads und Prozesse in Python
- [x] Block 4 — Generators, Chunking & Memory Management
- [x] Block 5 — Event Loop, asyncio & asynchrone Programmierung
- [x] Block 6 — SQL vs. Python für ETL

---

# Block 1 — Concurrency vs. Parallelismus & Thread vs. Prozess

*Die ersten zwölf Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obs0u487oTB1HMtqOoSBoKXI" -->
Fowler: Eine Anwendung muss 100 Webseiten laden, jede Anfrage dauert 1 Sekunde. Wie lange dauert das sequenziell, wie lange concurrent — und warum?::
**Sequenziell:** mindestens **100 Sekunden** — jede Anfrage wartet, bis die vorige fertig ist.<br>**Concurrent:** im besten Fall etwa **1 Sekunde** — alle Anfragen werden angestoßen, und das Warten überlappt.<br>Grund: Die Zeit ist fast nur **Warten auf I/O**, nicht Rechnen. Während eine Anfrage wartet, ist der Kern frei für die nächste — dafür reicht schon ein einziger Kern.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYhTby12ElV24Y7EgB8wUr" -->
Was ist das Grundproblem einer synchronen (sequenziellen) Anwendung, wenn eine Codezeile langsam ist?::
Code läuft strikt nacheinander: Die nächste Zeile startet erst, wenn die vorige fertig ist.<br>Ist eine Zeile langsam (z. B. eine Netzwerkanfrage), **hängt alles danach**, obwohl der Prozessor in der Zeit nichts zu tun hat.<br>Für viele Anwendungen ist das in Ordnung — problematisch wird es bei vielen I/O-Operationen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsc4tMaLIa3U6JgB0oknllp" -->
Wie definiert Fowler den Unterschied zwischen Concurrency und Parallelismus?::
**Concurrency:** mehrere Aufgaben sind gleichzeitig *in Arbeit*, aber in einem gegebenen Moment wird womöglich nur an **einer** aktiv gearbeitet.<br>**Parallelismus:** mehrere Aufgaben werden im selben Moment **tatsächlich aktiv ausgeführt** — auf mehreren Kernen.<br>Ein Programm kann concurrent sein, ohne parallel zu laufen (ein Kern), aber nicht umgekehrt (Python-Sprachgebrauch).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbTOSSisKYnt7XBWyM7Og5" -->
Was ist präemptives Multitasking?::
Das **Betriebssystem** entscheidet, wann zwischen Aufgaben gewechselt wird, per **Time Slicing**: Nach einer Zeitscheibe verdrängt (*preempts*) es die laufende Aufgabe.<br>Die Aufgabe hat darauf keinen Einfluss — der Wechsel kann zwischen zwei beliebigen Instruktionen passieren.<br>Umgesetzt über **Threads** oder **Prozesse**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8MNjyHgfeUSQDEtBCPn1u" -->
Was ist kooperatives Multitasking — und welches Python-Werkzeug funktioniert so?::
Die **Aufgabe selbst** entscheidet, wann gewechselt wird: An explizit markierten Stellen sagt sie „ich pausiere, lass andere ran".<br>Vorteil: kein Wechsel an unerwarteter Stelle, billige Wechsel.<br>Risiko: Eine Aufgabe, die nie abgibt, **blockiert alle anderen**.<br>In Python: **asyncio** — die markierten Stellen sind die `await`s.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAmLaNkztdnkMbQFDwlKyB" -->
Wie definiert Fowler einen Prozess? Und was entsteht, wenn du `python main.py` startest?::
Ein Prozess ist ein laufendes Programm mit einem **eigenen Speicherbereich, auf den andere Anwendungen nicht zugreifen können**.<br>Mehrere Prozesse können auf einer Maschine laufen — auf mehreren Kernen gleichzeitig, auf einem Kern per Time Slicing abwechselnd.<br>`python main.py` erzeugt einen **Prozess** mit genau einem Thread, dem **Main Thread**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbYqp5sY5GIBQBcUTuX44s" -->
Wie beschreibt Fowler einen Thread? Was sind Main Thread und Worker Threads?::
Ein Thread ist ein „**leichtgewichtiger Prozess**" und die kleinste Einheit, die das Betriebssystem verwalten kann.<br>Er hat **keinen eigenen Speicher**, sondern teilt den des Prozesses, der ihn erzeugt hat.<br>Jeder Prozess hat mindestens einen Thread, den **Main Thread**; zusätzlich erzeugte heißen **Worker** oder **Background Threads**.<br>Threads können auf mehreren Kernen parallel laufen oder per Time Slicing abwechselnd.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUBt8LO5D90XAxGFMGdcJt" -->
Wie erklärt OSTEP einen Thread über den Program Counter?::
Ein Programm mit einem Thread hat **einen** Ausführungspunkt — einen Program Counter (PC), von dem Instruktionen geholt werden.<br>Ein Programm mit mehreren Threads hat **mehrere PCs**.<br>Jeder Thread ist fast wie ein eigener Prozess — mit einem Unterschied: **Alle Threads teilen sich denselben Adressraum** und sehen dieselben Daten.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9C15PzjymOQvhUy8KisDh" -->
Ein Programm erzeugt erst Thread 1, dann Thread 2. Läuft Thread 1 zuerst?::
**Nicht unbedingt.** Die Reihenfolge bestimmt der **Scheduler** des Betriebssystems, nicht die Reihenfolge der Erzeugung.<br>Thread 2 kann vor Thread 1 laufen, und beide können an beliebigen Stellen unterbrochen werden.<br>Konsequenz: Ein Programm darf nie von einer bestimmten Ausführungsreihenfolge der Threads ausgehen — sonst hängt das Ergebnis vom Zufall ab.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHHvHdBCMrJoA1Itf4yV0l" -->
Welchen Text empfiehlt Rob Pike als wichtigste Lektüre zu Concurrency, und für welches Modell steht er?::
Tony Hoare, **„Communicating Sequential Processes"** (CSP, 1978).<br>CSP steht für **Message Passing**: Unabhängige Prozesse teilen keinen Speicher, sondern kommunizieren über Nachrichten bzw. Kanäle.<br>Go hat das mit seinen Kanälen direkt eingebaut. In Python entspricht dem eine **Queue**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTqh1qlONSOhoM6ica6TQB" -->
Wie lässt sich asyncio in einem Satz beschreiben (Fowler)?::
Eine Bibliothek, die **Coroutinen** asynchron ausführt, mit einem Concurrency-Modell namens **Single-threaded Event Loop**.<br>Also: ein einziger Thread, der zwischen vielen wartenden Aufgaben hin- und herspringt — kooperativ, nicht parallel. (Details in Block 5.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs34CB6L1ylpRjZAeBruYRM" -->
Wer hat die meisten Grundbegriffe der Concurrency geprägt (kritischer Abschnitt, gegenseitiger Ausschluss …)?::
**Edsger Dijkstra** — Pionier des Fachs, unter anderem dafür mit dem Turing Award ausgezeichnet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSvyKBGXueuFKEfjR6o45m" -->
Was meint Rob Pike mit „Concurrency is about structure, parallelism is about execution" — und was folgt daraus?::
**Concurrency** ist eine Eigenschaft der **Struktur**: Das Programm ist in unabhängig voranschreitende Teile zerlegt, plus Koordination dazwischen.<br>**Parallelismus** ist eine Eigenschaft der **Ausführung**: Berechnungen laufen im selben Moment auf mehreren Recheneinheiten.<br>Folge: **Dasselbe concurrent geschriebene Programm läuft auf einem Kern abwechselnd und auf acht Kernen parallel — ohne Codeänderung.** Erst die richtige Zerlegung macht Parallelität möglich (Gopher-Beispiel).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTjMMmGsflExdY0rP9RdMp" -->
Gibt es Parallelismus ohne Concurrency?::
Kommt auf den Sprachgebrauch an.<br>**Python-Literatur** (Ramalho, Fowler): Parallelismus ist ein **Sonderfall** von Concurrency — jedes parallele System ist concurrent.<br>**Pike:** Die Begriffe sind **orthogonal** — dann ja, z. B. **SIMD** (eine Instruktion auf mehreren Datenwerten), Instruktions-Parallelität im Prozessor, eine GPU, die eine einzige Matrixmultiplikation verteilt.<br>Für Data Engineering relevant: Vektorisierte Ausführung in NumPy, Polars, DuckDB ist genau dieser Parallelismus.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4FermF4TQ1c0mJ5afM11y" -->
Woran erkennst du, ob eine Aufgabe I/O-bound oder CPU-bound ist?::
Am **Engpass** — also daran, was helfen würde:<br>Schneller mit stärkerer CPU → **CPU-bound**.<br>Schneller mit mehr Bandbreite, schnellerer Platte oder schnellerer Gegenstelle → **I/O-bound**.<br>„Bound" ist **relativ zum betrachteten Prozess**: Eine lange SQL-Query ist auf dem DB-Server CPU-bound, für das wartende Python-Programm I/O-bound.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2gDyOOqwdxs03kfkc9jvb" -->
Ist das Einlesen und Parsen einer 5-GB-CSV-Datei in Python I/O-bound oder CPU-bound?::
Meist **CPU-bound**.<br>Eine NVMe-SSD liefert mehrere GB pro Sekunde; das Zerlegen in Felder und Umwandeln von Text in Zahlen und Datumswerte schafft ein einzelner Kern nur mit einem Bruchteil davon.<br>Test: Eine schnellere Platte bringt kaum etwas, ein Parser, der mehrere Kerne nutzt (pandas mit `engine="pyarrow"`, Polars, DuckDB), bringt viel.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskW1N5azZx5GTAVqI1SxVW" -->
Welche Concurrency-Werkzeuge helfen bei I/O-bound, welche bei CPU-bound Arbeit (Überblick)?::
**asyncio** (ein Thread): hilft bei I/O, nicht bei CPU.<br>**Threads:** helfen bei I/O; bei CPU-Arbeit in CPython mit GIL nicht, im free-threaded Build schon.<br>**Prozesse:** helfen bei CPU-Arbeit (echte Parallelität); bei I/O funktionieren sie auch, sind aber teurer als nötig.<br>Grund: Bei I/O gibt es Wartezeit, die man füllen kann — bei CPU-Arbeit hilft nur, mehr Kerne gleichzeitig rechnen zu lassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrYIQT9DI6j81Vvi319KHS" -->
Was teilen sich die Threads eines Prozesses — und was hat jeder Thread für sich?::
**Geteilt:** Programmcode, **Heap** (alle Objekte, globale Variablen), offene Dateien und Verbindungen.<br>**Eigen:** **Program Counter**, **Register**, **Stack** (lokale Variablen, Rücksprungadressen), ggf. Thread-Local Storage.<br>Deshalb sind lokale Variablen einer Funktion zwischen Threads *nicht* geteilt — geteilt ist nur, was auf dem Heap liegt und von beiden erreichbar ist.<br>Den Zustand sichert das OS beim Wechsel im **Thread Control Block**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIvdOBnkX0giF6KeNc1ffp" -->
Thread vs. Prozess: Wie unterscheiden sie sich bei Startkosten, Kommunikation und Absturzsicherheit?::
**Startkosten:** Thread ~**100 µs**, Prozess ~**50 ms** in CPython (Zahlen aus PEP 703) — deshalb Prozess-**Pools**.<br>**Kommunikation:** Threads über gemeinsame Objekte (einfach, aber Race-Gefahr); Prozesse über Pipes/Queues/Sockets, Daten müssen **serialisiert** (gepickelt) und kopiert werden.<br>**Absturz:** Ein schwerer Fehler in einem Thread (z. B. Segfault in einer C-Erweiterung) reißt den **ganzen Prozess** mit; ein abgestürzter Prozess ist **isoliert**.<br>Feinheit: Eine unbehandelte Python-Exception beendet nur ihren Thread.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4XWp7ERBZHXGYhtLvWQj9" -->
Was machen `fork()` und `exec()`, und was ist Copy-on-Write?::
**`fork()`** erzeugt eine fast exakte **Kopie** des aufrufenden Prozesses; beide laufen an derselben Stelle weiter. Rückgabewert: `0` im Kind, die PID des Kindes im Elternprozess.<br>**`exec()`** ersetzt das laufende Programm im Prozess durch ein anderes (Shell: erst fork, dann exec).<br>**Copy-on-Write:** Der Speicher wird nicht sofort kopiert; Eltern und Kind teilen die Seiten, bis eine verändert wird — deshalb ist `fork()` schnell.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWKeZKUl188deLQb0oSuhG" -->
Warum ist `fork` als Startmethode für `multiprocessing` heikel, und was hat Python 3.14 geändert?::
`fork` kopiert den laufenden Prozess — aber nur den **aufrufenden Thread**. Halten andere Threads gerade ein Lock, bleibt es im Kind **für immer gesperrt** → Deadlock-Gefahr.<br>Alternativen: `spawn` (frischer Interpreter) und `forkserver` (sauberer Hilfsprozess, der forkt).<br>Seit **Python 3.14** ist unter Linux **`forkserver`** statt `fork` der Standard (macOS und Windows: `spawn`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyARVhmmtoTipenKXe5tdO" -->
Welche drei Grundzustände hat ein Prozess (oder Thread), und wie kommt er von einem in den anderen?::
**Running** — läuft auf einem Kern.<br>**Ready** — könnte laufen, wartet auf den Scheduler.<br>**Blocked** — wartet auf ein Ereignis, meist I/O.<br>Ready → Running: vom Scheduler eingeplant. Running → Ready: verdrängt. Running → Blocked: startet I/O. Blocked → Ready: I/O fertig.<br>Ein Thread, der auf das Netzwerk wartet, ist *blocked* und belegt **keinen Kern**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCq07k3oZFNgzMgQmpwrCt" -->
Was passiert bei einem Context Switch, und was kostet er?::
1. Zustand der laufenden Einheit (Register, PC, Stack Pointer) im Control Block sichern<br>2. Scheduler wählt die nächste<br>3. deren Zustand laden<br>4. bei **Prozess**wechsel zusätzlich den Adressraum (Seitentabelle) umschalten<br>**Direkte Kosten:** Größenordnung Mikrosekunden. **Indirekte:** kalte Caches, nach Prozesswechsel kalter TLB.<br>Daher: Prozesswechsel > Threadwechsel > Wechsel zwischen asyncio-Tasks (nur ein Funktionsaufruf).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMdwVUhD8CjwAu2xI6376z" -->
Warum ergibt `counter = counter + 1` in zwei Threads nicht zuverlässig das erwartete Ergebnis?::
Die Zeile ist für den Prozessor **drei** Instruktionen: Wert **laden** → im Register **erhöhen** → zurück**schreiben**.<br>Wird Thread 1 nach dem Erhöhen, aber vor dem Schreiben unterbrochen, lädt Thread 2 den **alten** Wert, erhöht und schreibt. Danach schreibt Thread 1 denselben Wert noch einmal → **zweimal erhöht, einmal gezählt**.<br>In Python gilt dasselbe eine Ebene höher: `counter += 1` sind mehrere Bytecode-Instruktionen. Der GIL schützt nicht zuverlässig davor.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7CTWGtyaPmbajxhfMoTbf" -->
Definiere Race Condition, kritischer Abschnitt, gegenseitiger Ausschluss und Atomarität.::
**Race Condition:** Das Ergebnis hängt von der zeitlichen Abfolge der Threads ab → das Programm ist **indeterminiert** (gleicher Input, anderer Output).<br>**Kritischer Abschnitt:** Code, der auf eine **geteilte** Ressource zugreift und deshalb **nicht von mehreren Threads gleichzeitig** ausgeführt werden darf (entscheidend: *geteilt*, nicht *global*).<br>**Gegenseitiger Ausschluss:** Garantie, dass höchstens ein Thread im kritischen Abschnitt ist.<br>**Atomarität:** ganz oder gar nicht, kein Zwischenzustand sichtbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsX9XLfs8Zs9ThdrZnc3x4k" -->
Was ist ein Lock, und welche zwei Kosten hat es?::
Ein **Lock** (Mutex) erzwingt gegenseitigen Ausschluss: Wer den kritischen Abschnitt betreten will, muss es erwerben, sonst wartet er. In Python: `with lock:`.<br>**Contention:** Konkurrieren viele Threads um dasselbe Lock, warten sie meist aufeinander — im Extremfall läuft der Code faktisch seriell.<br>**Deadlock:** A hält Lock 1 und wartet auf Lock 2, B umgekehrt — keiner kommt weiter. Vorbeugung: Locks **immer in derselben Reihenfolge** erwerben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXrMtVLeA2I2pDDtHwOpNP" -->
Shared Memory vs. Message Passing — was ist der Unterschied, und welches Python-Werkzeug steht für das zweite?::
**Shared Memory:** Alle greifen auf dieselben Daten zu und schützen sie mit Locks (Modell von Threads).<br>**Message Passing:** Jeder Teil hat eigene Daten, ausgetauscht wird nur über Nachrichten (Modell von Prozessen, CSP, Go-Kanäle).<br>Go-Sprichwort: „Don't communicate by sharing memory; share memory by communicating."<br>In Python: **Queue** — `queue.Queue` zwischen Threads, `multiprocessing.Queue` zwischen Prozessen. Die Queue synchronisiert intern, eigener Code braucht kein Lock.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsL5ut2paDcnHspw484KD0t" -->
Zwei Prozesse haben keinen gemeinsamen Speicher. Können sie trotzdem eine Race Condition haben?::
**Ja — auf externen Ressourcen.** Zwei Prozesse, die dieselbe Datei beschreiben, oder zwei Pipeline-Läufe, die dieselbe Datenbankzeile lesen, ändern und zurückschreiben (**Lost Update**).<br>Datenbanken lösen das mit **Transaktionen und Sperren** — der Grund für „Task = Transaktion" aus 2.1.<br>Das *typische* Problem von Prozessen ist aber ein anderes: Alles, was sie austauschen, muss **serialisiert, kopiert und übertragen** werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVx5AzPcOWoLwZ8sZBpHNf" -->
Was unterscheidet einen Python-Thread von einer Goroutine — und was ist Pythons Gegenstück zu billigen, massenhaften Ausführungseinheiten?::
Pythons `threading` erzeugt echte **OS-Threads** (vom Betriebssystem präemptiv eingeplant).<br>**Goroutinen** sind **Green Threads**: Die Go-Laufzeit verteilt viele tausend davon auf wenige OS-Threads (M:N-Scheduling) — extrem billig.<br>Pythons Gegenstück: **asyncio-Coroutinen** — tausende Tasks in einem OS-Thread, kooperativ. Anders als Goroutinen laufen sie aber **nie parallel**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspskEEqRV99wowxf3Gfpfq" -->
Was sagt Amdahls Gesetz? Rechenbeispiel mit 90 % parallelisierbarem Anteil.::
Ist ein Anteil *p* parallelisierbar, ist der maximale Speedup mit *n* Kernen **S = 1 / ((1−p) + p/n)**.<br>*p* = 0,9, *n* = 4: 1 / (0,1 + 0,225) ≈ **3,1** statt 4.<br>*n* → ∞: höchstens **10**.<br>Der **serielle Rest** (Einlesen, Zusammenführen, Schreiben) bestimmt die Obergrenze — dazu kommen die Kosten der Parallelität selbst (Prozessstart, Serialisierung).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdU4GZa7LlaLEXEUaigQOO" -->
Was ist der Unterschied zwischen `os.cpu_count()` und `os.process_cpu_count()` — und zwischen Kern und logischer CPU?::
**Logische CPUs:** Mit Hyper-Threading/SMT bietet ein physischer Kern zwei logische CPUs an. Sie teilen sich die Recheneinheiten — helfen beim Warten auf Speicher, verdoppeln aber nicht die Rechenleistung.<br>**`os.cpu_count()`:** logische CPUs im **System**.<br>**`os.process_cpu_count()`** (seit 3.13): logische CPUs, die **dieser Prozess** nutzen darf (CPU-Affinität). Seit 3.13 Standard für die Worker-Zahl in `concurrent.futures` und `multiprocessing`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmzvqaoB0k6EeAkXwcRHXb" -->
Nach welchem Muster wird fast jede Datenverarbeitung parallelisiert — und was entscheidet über den Aufwand der Zusammenführung?::
**Aufteilen → jeden Teil unabhängig verarbeiten → Teilergebnisse zusammenführen** (Map/Reduce).<br>Voraussetzung: Die Verarbeitung pro Teil ist eine **reine Funktion** ohne geteilten veränderlichen Zustand — dann braucht es kein Lock.<br>Der **Aufteilungsschlüssel** entscheidet über den Aufwand: Zählt man Belegung pro `(product_id, Nacht)` nach Buchungen aufgeteilt, müssen die Teilzähler addiert werden; teilt man nach `product_id` auf, entfällt die Zusammenführung.
<!-- basicblock-end -->

# Block 2 — Warum Python langsam ist & der GIL

*Die ersten drei Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obs6JK1ozbHKUgQUaxC3QbwW" -->
Wie fasst VanderPlas in einem Satz zusammen, warum Python langsam ist?::
Python ist eine **dynamisch typisierte**, **interpretierte** Sprache, in der Werte nicht in dichten Puffern, sondern in **verstreuten Objekten** gespeichert werden.<br>Drei Ursachen: Typprüfung und Dispatch bei jeder Operation · Bytecode wird Instruktion für Instruktion interpretiert · Zeiger auf einzelne Heap-Objekte statt dicht gepackter Werte.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0Jrhf6k5lYn4xcLRmIv5W" -->
Was ist Hoppers Trilemma, und wie hat Python sich entschieden?::
Eine Sprache kann **dynamisch**, **schnell** und **einfach implementiert** sein — man kann **zwei** davon haben.<br>Python wählte **dynamisch und einfach** (CPython ist klein genug, dass eine Person es lesen kann).<br>Preis: Reiner Python-Code hat eine Leistungsgrenze, die keine schrittweise Optimierung durchbricht. Auswege: eine viel komplexere Implementierung (JIT) oder weniger Dynamik (Cython, mypyc).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRzUMD6CG1Pfy4VTfXdCLz" -->
Warum macht es für die meisten Python-Programme kaum einen Unterschied, dass Python langsam ist?::
Weil sie die meiste Zeit **nicht in Python-Code** verbringen, sondern in Datenbanktreibern und Bibliotheken wie NumPy, die in C/Rust geschrieben sind. Python ist nur der **Klebstoff**.<br>Den Klebstoff zehnmal schneller zu machen, würde das Programm kaum schneller machen.<br>Langsam wird es genau dann, wenn die **innere Schleife über die Daten in Python** läuft.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsARx0vQ08vkm1VxYpZDh0z" -->
Ist „Python ist langsam" eine Aussage über die Sprache?::
Streng genommen über **CPython**, die Referenzimplementierung. Die Sprache legt nur die Bedeutung fest; PyPy führt denselben Code mit JIT oft viel schneller aus.<br>Aber: Die **Dynamik der Sprache** (Typen, Klassen, Attribute jederzeit änderbar) macht es *jeder* Implementierung schwer, schnellen Code zu erzeugen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLQ80PIeUX7uujkGxxqo2Y" -->
Was passiert bei `a + b` in CPython, das in C nicht passiert?::
C: Typen stehen beim Kompilieren fest → eine Additions-Instruktion.<br>CPython: für `a` und `b` je ein **Objekt** mit Typ und Wert; beim Addieren **Typ von `a` nachschlagen, Typ von `b` nachschlagen, passende Addition auswählen** (Dispatch); dann ein **neues Objekt** für das Ergebnis anlegen.<br>Zwei Hauptkosten: **Dispatch** bei jeder Operation und **Objekt-Anlegen** für jedes Zwischenergebnis.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNsUY3IzEULjf0yB6h332C" -->
Wird eine Python-Schleife bei jedem Durchlauf neu kompiliert?::
**Nein.** CPython übersetzt den Quellcode **einmal** in **Bytecode** (`.pyc`).<br>Bei jedem Durchlauf wird derselbe Bytecode erneut **interpretiert**: Instruktion für Instruktion durch die Interpreter-Schleife, mit Typprüfung, Dispatch und Objekt-Anlegen.<br>Ansehen kann man den Bytecode mit `dis.dis(funktion)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsb3OQf4hR3g28XTN0MO0OJ" -->
Warum kostet das Interpretieren von Bytecode Zeit gegenüber kompiliertem Maschinencode?::
1. **Verwaltungsaufwand pro Instruktion:** nächste Instruktion holen, verzweigen, ausführen.<br>2. **Keine Optimierung über Instruktionen hinweg:** Ein C-Compiler sieht die ganze Funktion, hält Werte in Registern, baut Schleifen um. Ein einfacher Interpreter sieht nur die aktuelle Instruktion.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBulNLjMW6rJxCj1bJQlBI" -->
Wie liegen eine Python-Liste und ein NumPy-Array mit einer Million Ganzzahlen im Speicher — und was folgt daraus?::
**Liste:** Array von **Zeigern** auf einzelne, verstreute `int`-Objekte (je 28 Byte + 8 Byte Zeiger ≈ **36 MB**).<br>**NumPy (`int64`):** ein **zusammenhängender Block** roher Werte, Typ einmal für alle (**8 MB**).<br>Folgen: weniger Speicher · **Cache-effizient** statt Pointer Chasing · **keine Typprüfung pro Element** · **SIMD** möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspAmPQE6mePEEdoRl5Mnuq" -->
Warum sind pandas, Polars und DuckDB schnell, obwohl man sie aus Python benutzt? (vier Gründe)::
1. **Ein Aufruf pro Spalte statt pro Wert** — die Schleife läuft in C/Rust, der Python-Overhead fällt einmal an.<br>2. **Dichter, typisierter, spaltenweiser Speicher** (Arrow).<br>3. **Mehrere Kerne** — intern in eigenem Code, ohne GIL.<br>4. **Query-Optimizer** (Polars lazy, DuckDB).<br>„In C/Rust geschrieben" allein reicht nicht: Eine C-Funktion, millionenfach aus einer Python-Schleife aufgerufen, bleibt langsam.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8kRQq1JF77AqQ67qVPVdK" -->
Was ist der spezialisierende adaptive Interpreter (Python 3.11, PEP 659)?::
Der Interpreter **beobachtet**, mit welchen Typen eine Instruktion tatsächlich läuft, und ersetzt sie in heißem Code durch eine **spezialisierte** Variante (z. B. `BINARY_OP` → `BINARY_OP_ADD_INT`).<br>Ein kurzer **Guard** prüft, ob die Typen noch stimmen; sonst Rückfall auf die allgemeine Instruktion.<br>3.11 war im Schnitt **~25 % schneller** als 3.10. Hat **nichts mit dem GIL** zu tun — es beschleunigt Code mit einem Thread.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshYAOGnmemAa2Nhb8b5yC0" -->
Wie ist der Stand des JIT-Compilers in CPython (3.13, 3.14, 3.15)?::
**3.13:** experimentell, nur beim Selbstbauen (`--enable-experimental-jit`), Technik *copy-and-patch*.<br>**3.14:** in den Windows-/macOS-Installern enthalten, aber **standardmäßig aus** — einschalten mit `PYTHON_JIT=1`. Effekt: **10 % langsamer bis 20 % schneller**. Nicht kombinierbar mit dem free-threaded Build.<br>**3.15:** weiter experimentell, im Mittel ~7–8 % schneller (x86-64 Linux).<br>Nicht verwechseln mit PyPy und Numba — das sind eigene JIT-Werkzeuge.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshcPbi7681HK1y0Ry36PTR" -->
Warum machen Type Hints CPython nicht schneller — und welches Werkzeug nutzt sie doch für Geschwindigkeit?::
CPython **ignoriert** Annotationen zur Laufzeit (gespeichert in `__annotations__`, nie geprüft). Eine mit `int` annotierte Variable kann einen String enthalten → der Interpreter muss trotzdem bei jeder Operation prüfen.<br>Ausnahme: **mypyc** übersetzt typannotierten Code in eine C-Erweiterung und nutzt die Annotationen — der Code muss sich dann an sie halten (Dynamik gegen Tempo getauscht).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsyoP7SnGbgyv4v8kjlYuH4" -->
Welche Auswege aus der Langsamkeit von reinem Python gibt es? (fünf)::
**Bibliotheken in C/Rust** (NumPy, Polars, DuckDB) — Schleife verlässt Python<br>**PyPy** — andere Implementierung mit JIT; C-Erweiterungen nur eingeschränkt<br>**Numba** — `@njit` kompiliert numerische Funktionen zur Laufzeit<br>**Cython** / **mypyc** — Python-artiger Code mit Typen → C<br>**Rust über PyO3** — so sind Polars und pydantic-core gebaut
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJ7nq9JSAjo0d5PtsMc6CU" -->
Was ist der GIL?::
Der **Global Interpreter Lock**: ein einziges Lock pro CPython-Interpreter (also pro Prozess), das sicherstellt, dass **immer nur ein Thread Python-Bytecode ausführt**.<br>Threads sind in CPython damit **concurrent, aber für Python-Code nicht parallel**.<br>Zwei Prozesse haben zwei GILs — deshalb ist `multiprocessing` der klassische Weg zu echter Parallelität.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsluAdrm1hNXaYyUsOAADnS" -->
Warum gibt es den GIL — was ginge ohne ihn bei der Referenzzählung kaputt?::
CPython verwaltet Speicher über **Referenzzähler**; jede Zuweisung und jeder Aufruf ändert sie — per Lesen-Ändern-Schreiben, also race-anfällig.<br>**Inkrement verloren** → Zähler zu niedrig → Objekt wird freigegeben, obwohl noch benutzt → **Absturz**/kaputte Daten.<br>**Dekrement verloren** → Zähler erreicht nie 0 → **Speicherleck**.<br>Der GIL schützt alle Zähler mit einem Schlag.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCNJbXPFkRtoZxx9fi8cSk" -->
Warum hat CPython so lange am GIL festgehalten? (drei Vorteile)::
**Einfach** — ein Lock statt tausender feiner Locks, keine Deadlocks dazwischen.<br>**Schnell bei einem Thread** — ein einziges Lock kostet fast nichts, viele feine Locks kosten bei jeder Operation.<br>**Einfach für C-Erweiterungen** — sie müssen sich kaum um Thread-Sicherheit kümmern. Das hat das C-Ökosystem möglich gemacht, auf dem Python heute steht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQVKYwSGySqEMyjyWrf5xY" -->
In welchen drei Situationen gibt ein Thread den GIL ab?::
1. **Bei blockierendem I/O** (Netzwerk, Datei, `time.sleep()`) — „the GIL is always released when doing I/O".<br>2. **In C-Erweiterungen bei Rechenarbeit**, die keine Python-Objekte anfassen (NumPy, `hashlib`, `zlib`).<br>3. **Nach dem Switch Interval:** Der Interpreter zwingt einen rechnenden Thread nach `sys.getswitchinterval()` (Standard **5 ms**) zur Abgabe, damit er die anderen nicht aushungert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss4gf9L4r35aIUXB6wOovq" -->
Wann helfen Threads in CPython mit GIL — und wann nicht?::
✅ **I/O-bound:** Während ein Thread wartet, ist der GIL frei für andere.<br>✅ **CPU-bound in C-Bibliotheken**, die den GIL freigeben (NumPy, Kompression, Hashing).<br>❌ **CPU-bound in reinem Python:** Nur einer rechnet, die anderen warten auf den GIL — durch das ständige Weiterreichen eher **langsamer** als ein Thread.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsllR8CJZ0aJTGXBWq68BTK" -->
Schützt der GIL `counter += 1` vor einer Race Condition?::
**Nein.** Der GIL garantiert nur, dass **eine Bytecode-Instruktion** zur Zeit läuft — nicht, dass eine **Folge** von Instruktionen am Stück läuft.<br>`counter += 1` sind vier: `LOAD_GLOBAL`, `LOAD_CONST`, `BINARY_OP`, `STORE_GLOBAL`. Dazwischen kann der Thread wechseln → verlorenes Inkrement.<br>Der GIL schützt die **Innereien des Interpreters** (Referenzzähler, interne `dict`-Struktur), **nicht die Logik deines Programms**. Für geteilten veränderlichen Zustand braucht man auch mit GIL ein Lock.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswNr6KIepjobYgIZfbZTxF" -->
Was garantiert der GIL (bzw. im free-threaded Build die Sperre pro Objekt) für eingebaute Datenstrukturen?::
Einzelne Operationen wie `list.append()` oder `d[key] = value` hinterlassen die Datenstruktur **nie kaputt** — zwei Threads, die gleichzeitig an dieselbe Liste anhängen, verlieren keine Elemente.<br>Nicht garantiert: **zusammengesetzte** Operationen (lesen, rechnen, zurückschreiben) — die brauchen ein Lock.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKUqAt3LGGJgmWZuL9sAmv" -->
Wie hat sich free-threaded Python von 3.13 über 3.14 zu 3.15 entwickelt?::
**3.13:** experimenteller separater Build ohne GIL (`python3.13t`), deutlicher Verlust bei einem Thread.<br>**3.14:** **offiziell unterstützt, aber optional** (PEP 779, „Phase II"); PEP 703 vollständig umgesetzt, adaptiver Interpreter auch hier aktiv.<br>**3.15:** Verbesserungen unter der Haube, **keine** Entscheidung, free-threaded zum Standard zu machen („Phase III" offen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsH6SjGkiRrkydYSbTQYCrk" -->
Was heißt „officially supported, but still optional" beim free-threaded Build?::
**Supported:** kein Experiment mehr, Fehler werden behoben, stabile APIs; PEP 779 hat messbare Kriterien festgelegt (u. a. ≤ 15 % Verlust bei einem Thread).<br>**Optional:** Der normale Build hat **weiterhin einen GIL**. Den free-threaded Build installiert man bewusst als eigene ausführbare Datei, z. B. `python3.14t` (mit uv: `uv python install 3.14t`).<br>Das Ökosystem zieht nach: C-Erweiterungen müssen erklären, dass sie ohne GIL sicher sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs44fwcp4BSXGe4DfmW0HYP" -->
Wie prüfst du, ob du einen free-threaded Build benutzt — und ob der GIL gerade aktiv ist?::
**Build:** `python -VV` zeigt „free-threading build"; `sysconfig.get_config_var("Py_GIL_DISABLED")` ist `1`.<br>**Zur Laufzeit:** `sys._is_gil_enabled()`.<br>Der Unterschied zählt, weil ein free-threaded Build den GIL **zur Laufzeit wieder einschalten** kann. Die Thread-Auslastung ist kein zuverlässiges Zeichen (NumPy lastet auch mit GIL mehrere Kerne aus).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdUuOTDHDC1NoJM7kahPcr" -->
Welche Falle droht im free-threaded Build bei C-Erweiterungen?::
Importiert man eine C-Erweiterung, die **nicht** erklärt, ohne GIL sicher zu sein, **schaltet Python den GIL automatisch wieder ein** — für den ganzen Prozess.<br>Es gibt eine Warnung, das Programm läuft weiter, aber **ohne Parallelität**.<br>Erzwingen lässt sich der Betrieb ohne GIL mit `PYTHON_GIL=0` bzw. `-X gil=0` — auf eigenes Risiko.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsF29EgPtbm06gYks5cI29x" -->
Was kostet der free-threaded Build bei einem Thread, und was bringt er bei mehreren?::
**Ein Thread:** laut What's New 3.14 **~5–10 %** langsamer (HOWTO: ~1 % macOS/Apple bis ~8 % x86-64 Linux), dazu mehr Speicher (PEP 779: 15–20 %).<br>**Mehrere Threads, CPU-bound:** Grinberg maß bei Fibonacci auf vier Threads gut **3×** so schnell wie der normale Build; bei einem Thread etwa 86–91 % der Geschwindigkeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSRxrwP1KDg7dVAK3JAfkh" -->
Wie kommt free-threaded Python ohne GIL bei der Referenzzählung aus? (Konzepte)::
**Biased Reference Counting:** Der Thread, der ein Objekt erzeugt hat, ändert dessen Zähler billig ohne Synchronisation; andere Threads nutzen einen teureren atomaren Weg.<br>**Unsterbliche Objekte** (PEP 683, seit 3.12): `None`, `True`, kleine Ints werden nie freigegeben → Zähler muss nicht geändert werden.<br>**Sperren pro Objekt** für `list`, `dict` usw.<br>Thread-sicherer Allokator (**mimalloc**).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9Bqxo7I8gnQdau8e0Yjxj" -->
Würden vier Threads eine rechenintensive Pipeline in reinem Python beschleunigen?::
**Normaler Build:** **nein** — der GIL lässt nur einen rechnen; eher leichter Verlust.<br>**Free-threaded Build:** **ja, möglich** — wenn jeder Thread eigenen Zustand hat (z. B. eigener `Counter`) und am Ende zusammengeführt wird; abzüglich 5–10 % Grundkosten und Amdahl.<br>**Vier Prozesse:** Gewinn, aber Daten müssen kopiert werden.<br>Größter Hebel: **spaltenweise mit Polars/DuckDB** — die Schleife verlässt Python ganz.
<!-- basicblock-end -->

# Block 3 — Threads und Prozesse in Python

*Die ersten neun Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="ObsN9jI3WRhoRTXYR7vK4aeL" -->
Wie definiert Ramalho Concurrency und Parallelismus?::
**Concurrency:** die Fähigkeit, mehrere anstehende Aufgaben zu bearbeiten, eine nach der anderen oder parallel (falls möglich), sodass jede irgendwann gelingt oder scheitert. Ein Einkern-Prozessor kann das, wenn ein Scheduler die Aufgaben verschachtelt. Auch: **Multitasking**.<br>**Parallelismus:** die Fähigkeit, mehrere Berechnungen **zur selben Zeit** auszuführen. Braucht mehrere Kerne, mehrere CPUs, eine GPU oder einen Cluster.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiEboOUSPrMTnuRy9kojT5" -->
Was ist eine Ausführungseinheit (execution unit), und welche drei kennt Python?::
Ein Objekt, das Code nebenläufig ausführt, mit **eigenem Zustand und eigenem Call Stack**.<br>Python kennt **Prozesse** (`multiprocessing`), **Threads** (`threading`) und **Coroutinen** (`asyncio`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssSGzqzDFmhJ5bSNDgIMPD" -->
Was sagt Ramalho über die Kommunikation zwischen Prozessen?::
Prozesse kommunizieren über **Pipes, Sockets oder Memory-mapped Files**, die nur **rohe Bytes** transportieren.<br>Python-Objekte müssen deshalb **serialisiert** werden, um von einem Prozess zum anderen zu gelangen. **Das ist teuer, und nicht alle Objekte sind serialisierbar.**<br>Vorteil: Prozesse sind isoliert und präemptiv geplant — ein eingefrorener Prozess legt (theoretisch) nicht das ganze System lahm.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4MHi1aVBav7jGD4tJ507B" -->
Was sagt Ramalho zu Threads: Vorteil und Gefahr des geteilten Speichers?::
Threads eines Prozesses teilen sich denselben Speicher mit den lebenden Python-Objekten.<br>**Vorteil:** einfacher Datenaustausch.<br>**Gefahr:** **beschädigte Daten**, wenn mehrere Threads gleichzeitig dasselbe Objekt verändern.<br>Präemptiv vom OS geplant, verbrauchen weniger Ressourcen als ein Prozess für dieselbe Arbeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9AIwJL6qTXYiM70bvu5H4" -->
Was ist eine Coroutine (Ramalho), und was ist ihre große Gefahr?::
Eine Funktion, die sich **selbst anhalten und später fortsetzen** kann (klassisch über Generatoren, nativ mit `async def`). Läuft meist in einem Thread unter einem **Event Loop** im selben Thread.<br>**Kooperativ:** Jede Coroutine muss mit `await` (bzw. `yield`) selbst abgeben.<br>**Gefahr:** **Jeder blockierende Code** in einer Coroutine **blockiert den Event Loop und alle anderen Coroutinen.**<br>Dafür verbraucht sie weniger Ressourcen als Thread oder Prozess.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxmjAvpCpStHBTGOj9U6XS" -->
Was sind Queue, Lock und Contention in Ramalhos Vokabular?::
**Queue:** Datenstruktur zum Austausch von Daten **und Steuersignalen** (Fehlercodes, Beenden) zwischen Ausführungseinheiten, meist FIFO. `queue`, `multiprocessing` und `asyncio` haben je eigene Queue-Klassen.<br>**Lock:** Objekt zur Synchronisation, das man beim Ändern geteilter Daten hält; einfachste Form: **Mutex**.<br>**Contention:** Streit um eine knappe Ressource — um ein Lock oder Speicher (**Resource Contention**) oder um CPU-Zeit (**CPU Contention**).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEq2x2QCGAh2S7nJOuMAza" -->
Was tut der Interpreter, damit ein Thread den GIL nicht ewig hält?::
Er **pausiert den laufenden Thread alle 5 ms** (Standard) und gibt den GIL frei.<br>Der Thread kann ihn dann wieder anfordern — warten aber andere Threads, kann der OS-Scheduler einen von ihnen auswählen.<br>Abfragen/ändern: `sys.getswitchinterval()` / `sys.setswitchinterval(s)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspQN7s1Uqh0UdRGLXOa7UG" -->
Warum ist Programmieren mit Threads laut Ramalho so schwer zu durchdenken?::
Weil der **Scheduler einen Thread jederzeit unterbrechen** kann.<br>Man muss daran denken, **Locks zu halten**, um kritische Abschnitte zu schützen — sonst wird man mitten in einer mehrschrittigen Operation unterbrochen und hinterlässt Daten in einem **ungültigen Zustand**.<br>Bei Coroutinen kann ein Wechsel dagegen nur an `await`-Stellen passieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszflaAOflM2Kb7TQXwHQtp" -->
Was ist laut Ramalho eine der besten Eigenschaften von `concurrent.futures`?::
Man kann **Concurrency auf bestehenden sequenziellen Code aufsetzen**: Die vorhandenen Funktionen werden einfach wiederverwendet, nur der Aufruf läuft über einen Executor.<br>Und der Wechsel zwischen Threads und Prozessen ist **eine einzige Zeile** (`ThreadPoolExecutor` ↔ `ProcessPoolExecutor`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsquxhunI6Ny9wJfJwZCEZ4" -->
Wie beendet man einen Thread in Python — und warum gibt es keine API, ihn von außen abzubrechen?::
**Kooperativ:** Man schickt ihm ein Signal, z. B. ein `threading.Event` (`done.set()`), das er regelmäßig prüft, oder eine Poison Pill über eine Queue — und er beendet sich selbst.<br>**Keine Kill-API**, weil Threads den Speicher teilen: Ein mitten in der Arbeit abgeschossener Thread könnte ein **Lock nie freigeben** oder eine Datenstruktur **halb verändert** hinterlassen.<br>Prozesse kann man dagegen mit `terminate()`/`kill()` beenden — sie teilen keinen Speicher.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMxfNlPt5MsAPJ1lXpcirv" -->
Warum braucht der Spinner mit Prozessen `multiprocessing.Event` statt `threading.Event`?::
Weil Prozesse **keinen Speicher teilen**: Ein `threading.Event` würde in den Kindprozess **kopiert** — der Elternprozess setzt sein Flag, das Kind schaut auf seine eigene, nie gesetzte Kopie.<br>`multiprocessing.Event` baut auf einem **Synchronisationsmittel des Betriebssystems** auf, das beide sehen.<br>Detail: `multiprocessing.Event` ist eine Fabrikfunktion, der Typ für Type Hints heißt `synchronize.Event`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0l5YIW5b1xOAQiJ9QzePm" -->
Ramalhos Quiz: Der Spinner läuft, der Main Thread rechnet CPU-lastig. Dreht sich der Spinner als Thread, als Prozess, als Coroutine?::
**Prozess:** ✅ — eigener Interpreter, eigener GIL, eigener Kern.<br>**Thread:** ✅ (evtl. leicht ruckelig) — der rechnende Thread muss den GIL nach dem **Switch Interval** (5 ms) abgeben, der Spinner braucht nur kurz.<br>**Coroutine:** ❌ **friert ein** — die Rechenfunktion enthält kein `await` und gibt den Event Loop nie ab.<br>Lehre: Der GIL verhindert nicht das **Abwechseln**, nur das **gleichzeitige Rechnen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqyOtz3luiSZQpCUoLceIi" -->
Was meint Ramalho mit „Thread-Based Nonsolution" beim Primzahl-Checker?::
Die Thread-Variante **sieht aus wie eine Lösung** — Arbeit aufgeteilt, mehrere Worker, korrekte Ergebnisse —, ist aber **nicht schneller als sequenziell, eher langsamer**.<br>Grund: reine CPU-Arbeit in Python; der **GIL** lässt nur einen Thread zur Zeit Bytecode ausführen, und das Weiterreichen kostet zusätzlich.<br>Die Prozess-Variante ist dagegen ungefähr um die Zahl der Kerne schneller. (Im free-threaded Build wären Threads eine echte Lösung.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsoUjCBidYcMxhOJo0RIKck" -->
Wie funktioniert Ramalhos selbstgebauter Process Pool?::
Zwei `multiprocessing.SimpleQueue`s: **Jobs** und **Ergebnisse**.<br>Der Hauptprozess füllt die Job-Queue und startet einen Worker pro Kern. Jeder Worker holt sich in einer Schleife den nächsten Job und legt das Ergebnis in die Ergebnis-Queue — wer schneller ist, holt sich mehr.<br>**Poison Pill:** Pro Worker ein Sonderwert (bei Ramalho `0`) in der Job-Queue = „Feierabend".<br>Ergebnisse kommen in der Reihenfolge des **Fertigwerdens**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPPYqF0mpmSC433VN6zC3M" -->
Wie viele Worker nimmt man für CPU-bound und für I/O-bound Arbeit — und was ist Oversubscription?::
**CPU-bound, Prozesse:** ≈ Zahl der Kerne (Standard `ProcessPoolExecutor`: `os.process_cpu_count()`). Mehr bringt nichts außer Overhead.<br>**I/O-bound, Threads:** deutlich mehr als Kerne (Standard `ThreadPoolExecutor`: `min(32, process_cpu_count + 4)`).<br>**Oversubscription:** Polars, DuckDB, NumPy rechnen intern selbst mit allen Kernen — acht Prozesse mit Polars ergeben schnell 64 Threads auf acht Kernen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscmzQ5SbIBEHpGXgF6yKDT" -->
Was ist ein Future, wer erzeugt ihn, und welche Methoden hat er?::
Ein **Platzhalter für ein Ergebnis, das noch nicht feststeht** (eingeplant, laufend oder fertig).<br>Erzeugt vom **Executor** bei `submit()` — laut Doku nicht selbst anlegen (außer in Tests).<br>Methoden: `done()`, `result(timeout=…)` (blockiert bis fertig), `exception()`, `cancel()` (nur vor dem Start), `add_done_callback()`.<br>Entspricht `asyncio.Future` bzw. einem Promise in JavaScript.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiVuc0SAabg1ysmyIKqoKM" -->
`executor.map()` vs. `submit()` + `as_completed()` — was ist der Unterschied?::
**`map`:** Iterator über **Ergebnisse** in **Eingabereihenfolge** — ist die erste Aufgabe die langsamste, wartet man auf sie. Eine Exception erscheint beim Iterieren an ihrer Stelle, danach ist der Iterator am Ende.<br>**`submit` + `as_completed`:** ein **Future** pro Aufgabe, Ergebnisse in der Reihenfolge des **Fertigwerdens**, Fehler pro Future behandelbar, verschiedene Funktionen/Argumente möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKxYj1d7jNdTwKY1HlOhki" -->
Was bewirken `chunksize` und `buffersize` bei `executor.map()`?::
**`chunksize`** (nur `ProcessPoolExecutor`): wie viele Elemente auf einmal an einen Worker gehen. Standard **1** — bei einer Million kleiner Aufgaben eine Million Mal Pickling und Queue-Verkehr. Größer = weniger Overhead.<br>**`buffersize`** (seit 3.14): begrenzt, wie viele Aufgaben im Voraus eingeplant werden. Ohne ihn liest `map` die **ganze Eingabe sofort** ein — problematisch bei einem riesigen Generator.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPBZs02lhoE0Dgbi5SezeM" -->
Was passiert, wenn eine Funktion in einem Worker eine Exception wirft?::
`concurrent.futures` **fängt sie im Worker**, transportiert sie (bei Prozessen gepickelt) zum Hauptprozess und **speichert sie im Future**.<br>`future.result()` **wirft sie erneut** — im Hauptprozess, mit dem Worker-Traceback als `__cause__`. `future.exception()` gibt sie zurück. Bei `map` erscheint sie beim Iterieren.<br>Die anderen Aufgaben laufen weiter.<br>Stirbt ein Worker-**Prozess** hart (Segfault, OOM), ist der Pool kaputt → `BrokenProcessPool`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGztFSbZADpoXWzr7U7KFZ" -->
`fork`, `spawn`, `forkserver` — wie funktionieren sie, und was ist wo Standard?::
**`fork`:** kopiert den laufenden Interpreter, Kind erbt alles — schnell, aber gefährlich mit Threads (seit 3.12 `DeprecationWarning`).<br>**`spawn`:** frischer Interpreter, importiert das Hauptmodul, bekommt nur das Nötige gepickelt — sicher, langsam. Standard auf **Windows und macOS**.<br>**`forkserver`:** ein sauberer Server-Prozess ohne Threads forkt neue Prozesse — sicher, schneller als `spawn`, nur Unix. Standard auf **Linux seit 3.14** (vorher `fork`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscdIN7aKhGwOwuOAMgpRIv" -->
Warum braucht ein `multiprocessing`-Skript `if __name__ == "__main__":`?::
Bei `spawn` und `forkserver` **importiert jeder neue Prozess das Hauptmodul**, um an die Funktionen zu kommen.<br>Ohne Guard würde der Code auf oberster Ebene (z. B. `ProcessPoolExecutor(...)`) in jedem Kind **erneut** laufen → Kinder starten Kinder; Python bricht mit `RuntimeError` ab.<br>Doku: „Safe importing of main module" — das Hauptmodul muss sich **ohne Nebenwirkungen** importieren lassen.<br>Seit 3.14 (`forkserver`) betrifft das auch Linux.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBjCpUHvcmBDbm9K49yQlp" -->
Was muss über die Prozessgrenze gehen können, und was ist nicht picklebar?::
Funktion, Argumente, Rückgabewerte und Exceptions werden mit **`pickle`** übertragen.<br>**Funktionen per Name** (Modul + Name) → sie müssen auf **oberster Ebene eines importierbaren Moduls** stehen.<br>**Nicht picklebar:** Lambdas, lokal definierte Funktionen, Generatoren, offene Dateien, Sockets, **Datenbankverbindungen**, `threading`-Locks.<br>Faustregel: Workern sagen, **welche** Daten sie holen sollen (ID-Bereich, Dateiname), statt die Daten zu schicken.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsppp0ynK8NpafkGuf46ARo" -->
Wie kann man Zustand zwischen Prozessen teilen — und warum vermeidet man es meist?::
**`multiprocessing.Value`/`Array`:** einzelne Werte/Arrays fester Typen in geteiltem Speicher, mit Lock.<br>**`multiprocessing.shared_memory`:** roher geteilter Block (z. B. für NumPy) — schnell, Synchronisation selbst regeln.<br>**`Manager`:** Server-Prozess hält Dicts/Listen, Zugriff über Stellvertreter — bequem, aber langsam.<br>Besser meist: **aufteilen → unabhängig verarbeiten → Ergebnisse zurückgeben → im Hauptprozess zusammenführen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVItNIB8fV27bSewXErt0x" -->
Was ist ein Subinterpreter (Python 3.14)?::
Ein vollständig getrennter Python-Interpreter **im selben Prozess**, mit **eigenem GIL** (`concurrent.interpreters`, PEP 734; `InterpreterPoolExecutor`).<br>„Die Isolation von Prozessen mit der Effizienz von Threads": echte Parallelität auch mit GIL, billiger als Prozesse.<br>Aber: Objekte werden nicht geteilt, sondern übertragen; viele C-Erweiterungen unterstützen Subinterpreter noch nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjUG4KpJry5gfdAfptFsVQ" -->
Welche Optionen gibt es für echten Parallelismus in Python trotz GIL? (sechs)::
1. **Prozesse** (`ProcessPoolExecutor`) — eigener GIL pro Prozess<br>2. **C-/Rust-Bibliotheken**, die den GIL freigeben und intern parallel rechnen (NumPy, Polars, DuckDB) — **im Data Engineering der wichtigste Weg**<br>3. **Free-threaded Build** (`python3.14t`)<br>4. **Subinterpreter** (3.14)<br>5. **Numba/Cython** mit `nogil`<br>6. **Verteilte Systeme** (Spark, Dask, Ray)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvRzx9YgmyVenYkvLzyuw2" -->
Was sind die typischen Herausforderungen bei echtem Parallelismus in Python?::
**Serialisierung** (Pickling kostet, nicht alles picklebar) · **Speicher** (eigener Interpreter und Datenkopien pro Prozess) · **Startkosten** (~50 ms pro Prozess → Pools) · **Race Conditions** ohne GIL und auf externen Ressourcen · **Zusammenführen** der Teilergebnisse (Amdahl) · **Fehlerbehandlung/Debugging** über Prozessgrenzen · **Plattformunterschiede** bei der Startmethode · **Oversubscription** mit intern parallelen Bibliotheken.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbrK5iJDGOM9Ljs9gxpypw" -->
Wie verteilt man eine Zähl-Aufgabe (z. B. Belegung pro Produkt und Nacht) sinnvoll auf einen `ProcessPoolExecutor`?::
Jeder Worker bekommt einen **Teil der Eingabe** und gibt einen **Teil-`Counter`** zurück, nicht einzelne Werte.<br>**Zusammenführung im Hauptprozess:** `sum(ex.map(zaehle, teile), Counter())`.<br>Besser als Daten zu pickeln: Jeder Worker **lädt seinen Teil selbst** (z. B. einen `product_id`-Bereich) — dann reisen nur die Grenzen, und bei Aufteilung nach `product_id` überschneiden sich die Teil-Counter nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQRz3Jv9O3EQMYto0KGBDy" -->
Warum kann man einem Worker-Prozess nicht die Datenbankverbindung des Hauptprozesses geben — und wie löst man es?::
Eine Verbindung ist **nicht picklebar**: ein offener **Socket** plus Sitzungszustand auf dem Server. Selbst über `fork` geteilt, würden zwei Prozesse über denselben Socket sprechen und das Protokoll zerschießen.<br>**Lösung:** Jeder Worker öffnet **seine eigene Verbindung**, am besten im **`initializer`** des Pools (läuft einmal pro Worker); übergeben wird nur der **DSN-String**:<br>`ProcessPoolExecutor(initializer=init_worker, initargs=(dsn,))`
<!-- basicblock-end -->

# Block 4 — Generators, Chunking & Memory Management

*Zu diesem Block gab es keine Exzerpte. Die ersten Karten decken die Stellen ab, die im Review offen oder ungenau waren.*

<!-- basicblock-start oid="Obsr2iiI3E1HaxIJFt98S98L" -->
Wann will man mit Generators chunken, wann ohne? (Leitfaden-Frage)::
Entscheidend ist das **Motiv**:<br>**Speicher** (Daten passen nicht sicher in den RAM) → **mit Generator**: Chunks entstehen erst beim Gebrauch und werden danach verworfen. Beispiele: `pd.read_csv(..., chunksize=…)`, Server-side Cursor mit `fetchmany`, `itertools.batched` über einem Generator.<br>**Arbeitseinheit** (Daten passen, sollen aber in Stücken verarbeitet werden) → **ohne Generator**, Chunks vorab als Listen. Beispiele: Liste in Teile für einen `ProcessPoolExecutor`, Inserts in Batches von 1000 für kleine Transaktionen, API-Paging.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskyibwE7ttunkBPySbMkoh" -->
Wann gibt CPython den Speicher eines Objekts frei?::
**In dem Moment, in dem sein Referenzzähler auf 0 fällt** — nicht „wenn die Funktion endet".<br>Lokale Variablen verschwinden am Funktionsende, aber ein zurückgegebenes oder anderswo gespeichertes Objekt lebt weiter.<br>Umgekehrt kann ein Objekt mitten in einer Funktion frei werden (`del x`, `x = None`), wenn sonst nichts darauf zeigt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLtBvOoXQ2ZOo8S5P2u4Dm" -->
Wozu braucht CPython zusätzlich zur Referenzzählung einen zyklischen Garbage Collector?::
Für **Referenzzyklen**: Objekte, die aufeinander zeigen (z. B. Elternknoten ↔ Kinder), haben einen Zähler ≥ 1, auch wenn von außen niemand mehr auf sie zeigt — sie würden nie freigegeben (Speicherleck).<br>Der GC sucht gelegentlich Gruppen, die nur noch von sich selbst erreichbar sind, und gibt sie frei. Alles ohne Zyklen erledigt die Referenzzählung sofort.<br>Generationell; in 3.14.0–3.14.4 kurz inkrementell, ab 3.14.5 wieder generationell. Steuerbar über das Modul `gc`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMS40MDPY6NfiWMZszKld7" -->
Wer wirft `StopIteration`, und was passiert damit in einer `for`-Schleife?::
**`__next__` des Iterators wirft** `StopIteration` als **Exception**, wenn nichts mehr kommt (nicht „zurückgegeben", und nicht `__iter__`).<br>Die `for`-Schleife **fängt** sie und endet.<br>`next(it, standardwert)` fängt sie selbst und gibt den Standardwert zurück.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsphIzPifcZpsUYd3pQ2Yi2" -->
Was ist der Unterschied zwischen Iterable und Iterator?::
**Iterable:** implementiert `__iter__`, liefert mit `iter()` einen Iterator (Liste, Dict, String, Datei, `range`).<br>**Iterator:** hält den **Iterationszustand** und liefert mit `__next__` das nächste Element; wirft `StopIteration`, wenn erschöpft.<br>Jeder Iterator ist auch ein Iterable: sein `__iter__` gibt **sich selbst** zurück.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obskgo7B1hQ9td6adXtOKjwD" -->
Warum kann man über eine Liste zweimal mit `for` laufen, über einen Generator nur einmal?::
`iter(liste)` erzeugt **jedes Mal einen neuen Iterator**, der vorne anfängt — die Liste bleibt unverändert.<br>Ein Generator **ist** ein Iterator; `iter(gen)` gibt **ihn selbst** mit seinem aktuellen Zustand zurück. Erschöpft bleibt erschöpft.<br>Ein Generator speichert seine Werte nicht — er berechnet sie bei `next()` und vergisst sie. Die zweite Schleife läuft **still null Mal** (`sum(gen)` → `0`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfQ6KKaKFhaRn3iAw6ysv1" -->
Wann läuft der Code einer Generator-Funktion los, und was passiert bei `yield`?::
Der **Aufruf** führt **keinen** Code aus, er gibt nur ein Generator-Objekt zurück.<br>Das **erste `next()`** startet den Code — bis zum ersten `yield`, das den ersten Wert liefert.<br>Bei `yield` **friert die Funktion ein**: lokale Variablen, Position im Code, Schleifenzustand bleiben im **Frame** (auf dem Heap) erhalten. Das nächste `next()` macht direkt dahinter weiter.<br>Falle: Argumentprüfungen in einer Generator-Funktion schlagen erst beim ersten Zugriff fehl.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsX2E06A41d9iUdAHkNIWAH" -->
`sum([x*x for x in range(10**8)])` vs. `sum(x*x for x in range(10**8))` — was ist der Unterschied?::
Die **List Comprehension** materialisiert alle 100 Mio. Ergebnisse, bevor `sum` beginnt — mehrere **Gigabyte**.<br>Der **Generator-Ausdruck** erzeugt jeden Wert erst auf Anforderung und vergisst ihn — **konstanter, winziger Speicher**.<br>Der Vorteil ist **Speicher, nicht Zeit**; bei kleinen Mengen ist die Liste oft sogar etwas schneller. Beides ist viel langsamer als NumPy.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1d10EygUJW5LRRCxdyFku" -->
In welchen Fällen ist ein Generator die falsche Wahl?::
**Mehrere Durchläufe** über dieselben Daten · **wahlfreier Zugriff** oder Zurückblicken · **`len()`** gebraucht · **Sortieren, Median** (braucht ohnehin alles) · **kleine Datenmengen** (kein Vorteil, schwerer zu debuggen) · Daten gehen an **einen anderen Prozess** (nicht picklebar) · **mehrere Threads** lesen denselben Iterator (nicht thread-sicher).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsglHR9WEjnNCkntwNISNCo" -->
Was machen `send()`, `close()` und `yield from` bei Generatoren?::
**`send(wert)`:** setzt fort und macht `wert` zum Ergebnis des `yield`-Ausdrucks (`x = yield ergebnis`) → Daten **in** den Generator schicken.<br>**`close()`:** wirft `GeneratorExit` an der Haltestelle; ein `finally` räumt auf.<br>**`yield from gen`** (3.3): reicht Werte, `send()` und Rückgabewert eines anderen Generators durch.<br>Ein Generator mit `send()` ist eine **klassische Coroutine** — der Vorläufer von `async`/`await` (PEP 342 → 380 → 492).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsk3hXkmRubI4gakng64m1n" -->
Welche `itertools`-Funktionen sind der Baukasten für Generator-Pipelines?::
**`islice(it, n)`** — die ersten *n* ohne Materialisieren (`head`)<br>**`chain(a, b)`** — mehrere Iterables hintereinander<br>**`batched(it, n)`** (3.12) — **Chunking in einer Zeile**<br>**`groupby(it, key)`** — gruppiert *aufeinanderfolgende* gleiche Schlüssel → **nur auf sortierten Daten korrekt**<br>**`tee(it, n)`** — teilt einen Iterator, puffert dafür intern (Speicher!)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7MELcXJHKmOrJlxRrBUSZ" -->
Wie funktioniert Beazleys Generator-Pipeline zur Log-Auswertung, und wann passiert die Arbeit?::
`wwwlog = open(...)` → `bytecolumn = (line.rsplit(None,1)[1] for line in wwwlog)` → `bytes_ = (int(x) for x in bytecolumn if x != "-")` → `sum(bytes_)`.<br>Die ersten drei Zeilen **tun nichts**, sie bauen nur die Kette.<br>Die Arbeit passiert erst in `sum()`, **gezogen (pull)**: Jede Zeile wandert einzeln durch die ganze Kette, bevor die nächste gelesen wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4JGmxWgaC52BDDWB4vZGf" -->
Wovon hängt der Speicherbedarf einer Generator-Pipeline ab — und wovon nicht?::
Von der **Größe eines Elements** (einer Zeile) plus Lesepuffern.<br>**Nicht** von der **Größe der Gesamtdaten**: Ob das Log 1 MB oder 100 GB hat, ändert nur die Laufzeit, nicht den Speicher.<br>Bonus: Jeder Schritt ist einzeln testbar und austauschbar (z. B. `gzip.open` statt `open`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIJhSjG5mkzqT4DIKsgStk" -->
Was haben Generator-Pipelines mit Unix-Pipelines gemeinsam — und wo hinkt der Vergleich?::
**Gemeinsam:** kleine Bausteine mit **einheitlicher Schnittstelle** (Iterator-Protokoll bzw. `stdin`/`stdout`), beliebig kombinierbar, Stück-für-Stück-Verarbeitung mit konstantem Speicher.<br>**Unterschiede:** Unix = **eigene Prozesse, laufen parallel**; Generatoren = **ein Thread**, immer nur ein Schritt aktiv. Unix = **push** über OS-Puffer; Generatoren = **pull**. Unix = **Bytes**; Generatoren = **Python-Objekte**. Unix-Schritte sind isoliert; eine Exception bricht die ganze Generator-Kette ab.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKExs1id61tP6kRzoWF5bZ" -->
Welche Probleme lösen Chunking und Generators — außer Speicher?::
Hauptproblem: Daten, die **nicht (sicher) in den Arbeitsspeicher passen** — Speicherbedarf hängt dann an einem Stück, nicht an der Gesamtmenge.<br>Dazu: Verarbeitung kann **beginnen, bevor alle Daten da sind**, und **unbegrenzte Datenströme** (Logs, Events) werden überhaupt erst verarbeitbar.<br>Chunks statt einzelner Elemente, weil sonst der **Overhead pro Element** (Roundtrip, Aufruf) dominiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs49dBPTXXIgwL1x26bEAt1" -->
Welche vier Hebel nennt die pandas-Doku für große Datenmengen — in welcher Reihenfolge?::
1. **Weniger Daten laden** — vor allem nur benötigte **Spalten** (`columns=`, `usecols=`), bei Parquet auch Zeilenfilter<br>2. **Effiziente Datentypen** — `category` für Strings mit geringer Kardinalität, Zahlen verkleinern; seit pandas 3.0 eigener `str`-Typ (PyArrow) statt `object`<br>3. **Chunking**<br>4. **Andere Bibliotheken** (Polars, DuckDB, Dask)<br>Die ersten beiden sind billig und reichen oft. Prüfen mit `df.memory_usage(deep=True)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsekJnSoiNPyg82WQIdItBb" -->
Welche Berechnungen lassen sich chunkweise ausführen, welche nicht?::
✅ **Zeilenweise** (filtern, umrechnen) — trivial.<br>✅ **Zerlegbare Aggregate** (`sum`, `count`, `min`, `max`, `mean` über Summe+Anzahl, `groupby().sum()`) — mit Zusammenführungsschritt (Map/Reduce).<br>❌ **Nicht zerlegbar**: Median, Quantile, `count distinct` — exakt nur mit allen Daten (oder Sketches wie HyperLogLog).<br>❌ **Global**: Sortieren, große Joins — brauchen **Spilling** auf die Platte (Datenbank, DuckDB, Polars).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXUliXu1TSChDtfFODE6LK" -->
Client-side vs. Server-side Cursor: Wo liegt das Ergebnis, und was bringt `fetchmany`?::
**Client-side (Standard, `conn.cursor()`):** Postgres schickt beim `execute()` das **komplette Ergebnis** in den Client-Speicher. `fetchmany(1000)` holt aus dem bereits Geladenen — **spart keinen Speicher**.<br>**Server-side (`conn.cursor(name="x")`, SQL `DECLARE … CURSOR`):** Ergebnis bleibt auf dem **Server**, der Client holt Portionen (`itersize`, Standard 100) — konstanter Speicher.<br>Preis: ein Roundtrip pro Portion, die **Transaktion bleibt offen** (sonst `withhold=True`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2CAQfGmzIzVmZ0uMyDQNk" -->
Was macht `cursor.stream()` in psycopg 3, und was ist der Haken?::
Postgres schickt die Zeilen **einzeln** (single-row mode), ohne serverseitigen Cursor — konstanter Speicher.<br>**Haken:** Die **Verbindung ist blockiert**, bis der Stream zu Ende gelesen ist — keine andere Query auf derselben Verbindung. Als experimentell markiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssAIx9IuKCDQmY9hxUayTz" -->
Wie streamt man eine Abfrage mit SQLAlchemy, statt alles in den Speicher zu laden?::
`select(...).execution_options(yield_per=10_000)` — nutzt mit psycopg einen **Server-side Cursor** und liefert Zeilen in Portionen.<br>Daraus eine Generator-Funktion mit `yield` machen, statt eine Liste zu bauen.<br>Ohne `yield_per` hat `session.execute()` bereits **alles geladen**, bevor die erste Zeile verarbeitet wird — eine Generator-Hülle allein bringt dann nichts.<br>Die Session muss offen bleiben, bis der Generator gelesen ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsI0dL7FSD2YAHE2URxVIPL" -->
Welche Stolpersteine hat es, eine Funktion von `list` auf einen Generator als Eingabe umzustellen?::
**`len()`** auf der Eingabe → `TypeError` (z. B. in einer Log-Zeile).<br>**Mehrfache Durchläufe** → zweiter Durchlauf leer, ohne Fehler.<br>Typ-Hinweis ehrlich machen: `Iterable[T]` statt `list[T]`.<br>Und: **Globale Aggregate** (z. B. ein `Counter` über alle Buchungen) bleiben trotzdem vollständig im Speicher — Streaming senkt die Spitze, aber nicht auf ein Element.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9JsVncovTKx0E2MDmAg86" -->
Warum wird ein Python-Prozess nach `del große_liste` oft nicht kleiner?::
Kleine Objekte (≤ 512 Byte) verwaltet CPythons Allokator **pymalloc** in großen Blöcken (**Arenen**).<br>Freier Platz wird **wiederverwendet**, aber eine Arena geht erst an das Betriebssystem zurück, wenn sie **komplett leer** ist — ein paar Überlebende halten sie fest.<br>Folge: **Der Spitzenverbrauch zählt.** Generators und Chunking senken die Spitze; `del` danach hilft dem OS oft wenig.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshVqFFgGVJSVCUfssIeC88" -->
Welche Werkzeuge gibt es, um Speicher zu messen, und was sind ihre Fallen?::
**`sys.getsizeof`** — nur **flach**: bei einer Liste die Zeiger, nicht die Elemente.<br>**`df.memory_usage(deep=True)`** — ohne `deep` fehlen die String-Inhalte.<br>**`tracemalloc`** — alle Python-Speicheranforderungen mit Codezeile; `get_traced_memory()` liefert aktuell und **Spitze**.<br>**RSS** (Systemmonitor) — enthält Arenen, die Python intern schon frei hat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspTXBUTlDnkfq6XEjIqOHY" -->
Wie spart man Speicher bei Millionen kleiner Objekte?::
**`__slots__`** bzw. `@dataclass(slots=True)`: feste Plätze statt eines `dict` pro Instanz → deutlich weniger Speicher, etwas schnellere Attributzugriffe; dafür keine neuen Attribute zur Laufzeit.<br>**Tupel/`NamedTuple`** für reine Datentransporte.<br>**Größter Hebel: Spaltenformate statt Objekte** — DataFrame/Arrow mit dicht gepackten Spalten statt einer Million Objekte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDLjugzeZdqgihD0DRfRzm" -->
Warum wird Chunking von Hand im Data Engineering seltener?::
Moderne Engines übernehmen es: **Polars** (Lazy-API + Streaming-Engine, Standard ab 2.0) verarbeitet Abfragen in Stücken, inklusive Joins und Aggregationen; **DuckDB** lagert bei Joins, Aggregationen und Sortierungen automatisch auf die Platte aus (**Out-of-Core**).<br>Man beschreibt deklarativ, *was* man will, und die Engine entscheidet, *wie* es in den Speicher passt — wie der Query-Planer in SQL.
<!-- basicblock-end -->

# Block 5 — Event Loop, asyncio & asynchrone Programmierung

*Die ersten fünf Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obsp1ZqPQxF2xDfR3G09j2vy" -->
Was tut der Event Loop laut dem asyncio-HOWTO?::
„Everything in asyncio happens relative to the event loop."<br>Er hält eine Sammlung von Jobs, nimmt einen und **gibt ihm die Kontrolle** (wie ein Funktionsaufruf). Der Job läuft, bis er **pausiert oder fertig** ist, und gibt die Kontrolle an den Loop zurück. Dann kommt der nächste.<br>Grob eine Queue, abgearbeitet einer nach dem anderen. Gibt es nichts zu tun, ruht der Loop, statt CPU zu verbrauchen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBDLkN3x006VGQ7Wf3It3Y" -->
Was ist eine Coroutine-Funktion, was ein Coroutine-Objekt, was ein Task?::
**Coroutine-Funktion:** mit `async def` definiert.<br>**Coroutine-Objekt:** was ihr **Aufruf** zurückgibt — **noch kein Code ausgeführt**.<br>**Task:** ein Coroutine-Objekt, das beim Event Loop **angemeldet** ist und eine Liste von Callbacks führt (wer darauf wartet). Empfohlen erzeugt mit `asyncio.create_task()` bzw. `TaskGroup.create_task()`.<br>Aufruf ohne `await` und ohne Task → nichts passiert, Warnung „coroutine … was never awaited".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsY4FrqqssPDrO9Jrn9J73T" -->
Was meint das HOWTO mit „Unlike tasks, awaiting a coroutine does not hand control back to the event loop"?::
`await coroutine` startet die Coroutine **sofort im aktuellen Task**, wie einen Funktionsaufruf — beim **Hineingehen** wird die Kontrolle nicht abgegeben.<br>Aber: Sobald in dieser Kette auf etwas **Unfertiges** gewartet wird (`asyncio.sleep`, Netzwerk, ein laufender Task), **pausiert der ganze Task**, und der Loop lässt andere laufen.<br>**`await` blockiert den Event Loop also nicht.** Blockieren tut nur Code, der lange läuft, ohne an ein wartendes `await` zu kommen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsC0bRd7bqNaFjNEAyPxXaM" -->
Was sind Awaitables, und wann legt man selbst ein Future an?::
Ein **Awaitable** ist alles, was hinter `await` stehen darf. Drei Hauptarten: **Coroutinen**, **Tasks**, **Futures**.<br>**Futures** sind die unterste Ebene: ein Platzhalter für ein Ergebnis, das ein anderer liefert. In asyncio braucht man sie, um callback-basierten Code mit `async`/`await` zu verbinden.<br>Im Anwendungscode muss man **normalerweise keine Futures selbst anlegen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscPiKizh00NUtnHmV0enWA" -->
Welchen Rat gibt die FastAPI-Doku, wenn man unsicher ist, ob eine Route `def` oder `async def` sein soll?::
„If you just don't know, use normal `def`."<br>Grund: `def`-Routen laufen im **Threadpool** — blockierender Code darin schadet nicht. In `async def` läuft die Route **direkt im Event Loop**; ein einziger blockierender Aufruf darin legt den ganzen Server lahm.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJZLMADTbQxbfx0I9x0D2H" -->
Wie kann ein einziger Thread auf hundert Netzwerkantworten gleichzeitig warten?::
Das Betriebssystem bietet **nicht blockierende Sockets** (ein Leseversuch ohne Daten kehrt sofort zurück) und **Ereignis-Benachrichtigung**: Man registriert alle Sockets und macht **einen** wartenden Aufruf — „weck mich, wenn irgendeiner bereit ist".<br>**Linux:** `epoll` · **macOS/BSD:** `kqueue` · **Windows:** IOCP · überall: `select`/`poll`. In Python gekapselt im Modul `selectors`.<br>Ohne das bräuchte man einen blockierten Thread pro Verbindung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaY7BgmJYl8K5kiaZswuZ7" -->
Beschreibe die Schleife des Event Loops in vier Schritten.::
1. **Bereite Tasks** aus der Warteschlange nacheinander ausführen.<br>2. Ein Task läuft **bis zu einem `await`, das wirklich warten muss**, und gibt dort die Kontrolle zurück.<br>3. Ist nichts bereit: auf **Ereignisse vom Betriebssystem** oder den nächsten Timer warten.<br>4. Ereignis tritt ein → zugehöriges **Future** erledigt → wartender Task kommt **zurück in die Warteschlange**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsU1PWAtqTrGRgEAsi3zGVi" -->
Warum laufen `await coro_a(); await coro_b()` nacheinander — und wie lässt man sie gleichzeitig laufen?::
Nicht, weil der Loop blockiert wäre (während `coro_a` wartet, laufen **andere** Tasks), sondern weil `coro_b` **erst nach Ende von `coro_a` erzeugt** wird.<br>Gleichzeitig laufen sie nur, wenn **beide vorher als Tasks angemeldet** sind:<br>`async with asyncio.TaskGroup() as tg:`<br>`    tg.create_task(coro_a())`<br>`    tg.create_task(coro_b())`<br>Merksatz: **Gleichzeitigkeit entsteht durch Tasks, nicht durch `await`.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiHfCigdK9JC3XdpApymRS" -->
Was haben native Coroutinen mit Generatoren zu tun?::
Gleiche Idee: Funktion **hält an und macht später weiter**, mit erhaltenem Frame.<br>Gleiche Methoden: `send()`, `throw()`, `close()` — der Task setzt seine Coroutine mit `coro.send(None)` fort.<br>**`await` ≈ `yield from`**: reicht die Pause durch die Kette bis zu einem Future, dessen `__await__` tatsächlich `yield`et.<br>Aber: In `async def` pausiert man mit **`await`**; ein `yield` darin macht einen **async Generator**.<br>Historie: PEP 255 → 342 (`send`) → 380 (`yield from`) → 492 (`async`/`await`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxkLGoQn4KdSr4Sp1lmCS0" -->
Warum muss man den Rückgabewert von `asyncio.create_task()` festhalten?::
Der Event Loop hält nur eine **schwache Referenz** auf Tasks. Zeigt sonst niemand auf den Task, kann ihn der **Garbage Collector mitten in der Ausführung einsammeln**.<br>Lösung: Task in einer Variable oder Menge speichern — oder gleich `TaskGroup` benutzen, die ihre Tasks hält.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsllcFSJfqQ2WPoMeog6nWu" -->
Was macht `asyncio.run()`, und warum ruft man es nur einmal auf?::
Erzeugt einen **neuen Event Loop**, führt die übergebene Coroutine darin aus, wartet bis sie fertig ist, **räumt auf** (übrige Tasks abbrechen, async Generators und den Standard-Threadpool schließen) und **schließt den Loop**.<br>Deshalb einmal, ganz oben: `asyncio.run(main())`. Aus einer laufenden Coroutine heraus darf man es nicht aufrufen — dort läuft schon ein Loop.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsq6Rb4wN2fPxY6NsPNk4KX" -->
`asyncio.gather` vs. `asyncio.TaskGroup` — was passiert bei einem Fehler?::
**`gather`:** Die **erste** Exception geht an den Aufrufer, die **anderen laufen weiter** (werden nicht abgebrochen). Mit `return_exceptions=True` landen Exceptions als Werte in der Ergebnisliste.<br>**`TaskGroup`** (3.11): **alle anderen Tasks werden abgebrochen**, am Ende alle Fehler als **`ExceptionGroup`** (`except*`). Beim Verlassen des Blocks sind garantiert alle Tasks fertig (Structured Concurrency).<br>Für neuen Code: **TaskGroup**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0SPqyWCObZsRa8O5ZV5jv" -->
Wie setzt man in asyncio ein Zeitlimit, und was passiert beim Abbrechen eines Tasks?::
**Zeitlimit:** `async with asyncio.timeout(5):` (3.11) → bei Ablauf wird abgebrochen und `TimeoutError` geworfen; für ein einzelnes Awaitable auch `asyncio.wait_for(aw, timeout=5)`.<br>**Abbruch:** `task.cancel()` wirft `CancelledError` an der aktuellen `await`-Stelle. Aufräumen in `finally` — aber **nicht verschlucken**, sonst funktionieren Timeouts und TaskGroups nicht. Seit 3.8 erbt sie von `BaseException`.<br>`asyncio.shield(aw)` schützt vor Mit-Abbruch.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs49vLXt3qSvsDADj7taKyJ" -->
Was ist die große Gefahr bei single-threaded Concurrency in Python?::
**Blockierender Code.** Der Wechsel ist kooperativ: Der Loop bekommt die Kontrolle nur an einem wartenden `await` zurück.<br>Ruft eine Coroutine `time.sleep()`, eine synchrone Bibliothek (`requests`, synchrones SQLAlchemy/psycopg, `boto3`) auf oder rechnet lange, **stehen alle anderen Tasks still** — ohne Fehlermeldung, nur durch steigende Antwortzeiten sichtbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8I24NDfSr3airjmSFnjdD" -->
Woran merkt man, dass blockierender Code den Event Loop aufhält?::
**Debug-Modus:** `asyncio.run(main(), debug=True)` oder `PYTHONASYNCIODEBUG=1`.<br>Meldet Task-Schritte, die den Loop **länger als 100 ms** blockieren (`loop.slow_callback_duration`), **nie abgewartete Coroutinen** mit Erzeugungsort und unsichere Aufrufe aus fremden Threads.<br>In Produktion: Antwortzeiten **aller** Endpunkte steigen gleichzeitig, obwohl die CPU kaum ausgelastet ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuMKniAkhmEqxzTmiDDNOU" -->
Wie bringt man blockierenden Code in einem asyncio-Programm unter — I/O vs. CPU?::
**Blockierendes I/O:** `await asyncio.to_thread(fn, arg)` — läuft im Threadpool, der Thread gibt beim Warten den GIL frei.<br>**CPU-Arbeit in reinem Python:** `await loop.run_in_executor(process_pool, fn, arg)` mit `ProcessPoolExecutor` — eigener Prozess, eigener GIL.<br>**CPU-Arbeit in C-Bibliotheken**, die den GIL freigeben: `to_thread` reicht oft.<br>**Am besten:** async Bibliotheken (`httpx`, `AsyncConnection`, `asyncpg`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseJ9fBB8dysSqozxI4bnEz" -->
asyncio vs. Threads: Wer entscheidet über den Wechsel, wo kann gewechselt werden, was kostet es?::
**asyncio:** der **Code** entscheidet (kooperativ) — Wechsel **nur an `await`-Stellen**; Wechsel = Funktionsaufruf im selben Thread, sehr billig; ein Frame von wenigen KB pro Task → **zehntausende** Tasks.<br>**Threads:** das **Betriebssystem** entscheidet (präemptiv) — Wechsel **überall**; OS-Context-Switch (Mikrosekunden), eigener Stack pro Thread (unter Linux mehrere MB reserviert) → **hunderte bis wenige tausend**.<br>Dafür funktionieren Threads mit jeder normalen Bibliothek.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshQ4dYB2lzSU1jtZs5hxeY" -->
Kann es in einem asyncio-Programm Race Conditions geben?::
**Ja — über `await`-Grenzen hinweg.** `counter += 1` ist sicher (kein `await` darin), aber jede Logik **„lesen — `await` — schreiben"** ist anfällig: Am `await` laufen andere Tasks, die denselben Zustand ändern (Lost Update).<br>Lösung: kein `await` zwischen Lesen und Schreiben, oder `asyncio.Lock`.<br>Vorteil gegenüber Threads: **Jedes `await` ist sichtbar** eine mögliche Unterbrechung; alles dazwischen ist atomar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuvOqKLE2KNIPG29dcSgpH" -->
Wann asyncio, wann Threads, wann Prozesse? Je ein Data-Engineering-Beispiel.::
**asyncio:** viele gleichzeitige I/O-Vorgänge mit async Bibliotheken — 5.000 Seiten einer REST-API mit `httpx.AsyncClient`, viele Dateien gleichzeitig aus S3.<br>**Threads:** I/O mit **synchronen** Bibliotheken — parallele Uploads mit `boto3`, mehrere Datenbanken mit synchronen Treibern; Rechenarbeit in Bibliotheken, die den GIL freigeben.<br>**Prozesse:** CPU-Arbeit in reinem Python — Transformation in Python-Schleifen über Partitionen verteilen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYyl4tSqhELuywtDWgwPVP" -->
Warum sind `def`-Routen für eine FastAPI mit synchroner SQLAlchemy-Session richtig, und was passiert mit `async def`?::
Die Routen **warten** die meiste Zeit **auf Postgres** — und zwar **synchron**, der Thread blockiert. In `def` läuft das im **Threadpool**, unschädlich.<br>Mit `async def` + synchroner Session: Bei einem Request kein sichtbarer Unterschied; bei **50 gleichzeitigen** blockiert jeder während seiner Abfrage **den ganzen Event Loop** → alle Requests, auch an andere Routen, werden **nacheinander** abgearbeitet, Antwortzeiten steigen bis zu Timeouts.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsl91Im7IMCRCCaBpGT4AHj" -->
Was passiert mit dem 41. gleichzeitigen Request an `def`-Routen in FastAPI?::
Der Threadpool für `def`-Routen ist über einen **AnyIO-`CapacityLimiter`** mit **40** Plätzen begrenzt.<br>Der 41. wird **nicht abgewiesen**: Er **wartet asynchron**, bis ein Platz frei wird. Der Event Loop bleibt frei und nimmt weitere Verbindungen an; nur die Antwortzeit steigt um die Wartezeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQAXiu9aOadEQx8HPMW2cg" -->
Was sind `async with`, `async for` und async Generators?::
**`async with`:** Kontextmanager, dessen Betreten/Verlassen `await` braucht (`__aenter__`/`__aexit__`), z. B. `async with httpx.AsyncClient() as client:`.<br>**`async for`:** Schleife über einen Iterator, der aufs nächste Element warten muss (`__aiter__`/`__anext__`) — nächste API-Seite, nächste Cursor-Zeile.<br>**async Generator:** `async def` mit `yield` — liefert Werte und darf dazwischen `await`en; async Version der Generator-Pipeline.<br>**`@asynccontextmanager`** macht daraus einen Kontextmanager — so funktioniert FastAPIs `lifespan`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsF1P56u0nBhJ2J26HumSPo" -->
Was ist Function Coloring?::
In Sprachen mit `async` gibt es zwei Sorten Funktionen: `async`-Funktionen kann man nur mit `await` aufrufen, also nur aus anderen `async`-Funktionen.<br>Wird eine Funktion tief unten `async`, müssen **alle Aufrufer darüber** es auch werden — `async` **frisst sich durch die Codebasis** (Bob Nystrom, „What Color is Your Function?").<br>Deshalb gibt es viele Bibliotheken doppelt (SQLAlchemy sync/async, `requests`/`httpx`). Der eigentliche Preis von asyncio: Man entscheidet sich für ein Ökosystem.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obse0DTNh4Vy5VjayfiS1XYU" -->
Was ist Structured Concurrency, und wie setzt Python sie um?::
Nebenläufige Aufgaben sollen wie Codeblöcke eine **klare Lebensdauer** haben: Sie starten in einem Block und sind **garantiert beendet**, wenn der Block verlassen wird; ihre Fehler gehen nicht verloren (Nathaniel J. Smith).<br>Ein lose gestarteter `create_task()` kann dagegen weiterlaufen, nachdem seine Funktion fertig ist.<br>Python: **`asyncio.TaskGroup`** (3.11); seit 3.15 mit `cancel()` für die ganze Gruppe.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXDekMuwM4f9oxhD0CR4Oi" -->
Wie drosselt man in asyncio zehntausend gleichzeitige Anfragen?::
**`asyncio.Semaphore(10)`** + `async with sem:` → höchstens zehn gleichzeitig unterwegs.<br>**`asyncio.Queue`** mit fester Zahl von Worker-Tasks (Producer-Consumer); mit `maxsize` gibt es **Gegendruck** — ist die Queue voll, wartet der Producer.<br>Grund: Tasks sind billig, die Gegenseite (API mit Rate Limit, Datenbank mit begrenzten Verbindungen) nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsW1pmq0Ygx0L3Vyh0LX5Xl" -->
Lohnt sich der Umbau einer kleinen internen FastAPI auf async SQLAlchemy?::
Meist **nein**. Bausteine gäbe es (`AsyncSession`, psycopgs `AsyncConnection`, `asyncpg`), aber der Umbau kostet: alles wird `async` (Function Coloring), **Lazy Loading funktioniert nicht mehr** (`selectinload` nötig), Tests brauchen async Setup.<br>Async lohnt sich bei **sehr vielen gleichzeitigen, überwiegend wartenden** Verbindungen. Bei überschaubarer Last reicht `def` plus Threadpool mit 40 Plätzen — und ist einfacher.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBI8GgCx0Qe2jR6o6Ma34B" -->
Was ist uvloop?::
Ein **Ersatz für asyncios Standard-Event-Loop**, in C auf Basis von **libuv** (dem Event Loop von Node.js) geschrieben und deutlich schneller.<br>Wird mit `uvicorn[standard]` installiert, und Uvicorn benutzt ihn automatisch, wenn er vorhanden ist.
<!-- basicblock-end -->

# Block 6 — SQL vs. Python für ETL

*Die ersten zwei Karten decken die Punkte ab, die Nick beim Lesen selbst exzerpiert hat.*

<!-- basicblock-start oid="Obs1SLCgtdddZeLYxq3sfXE3" -->
Wobei sind SQL- und MPP-Engines stark, wobei schwach (Machado)?::
**Stark** bei „SQL-artigen" Transformationen: zeilenweise Berechnungen, Joins, Aggregationen, Window Functions.<br>**Schwach** bei **komplexen Schleifen und Rekursion** — wie sie z. B. ML-Algorithmen brauchen. DataFrame- und SQL-Systeme können das kaum effizient.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVrS7jEHbFlwQEpwfks9lB" -->
Bis zu welchen Datenmengen reichen native Python, Single-Node-Engines und MPPs grob (Machado)?::
**MPPs** (Spark, Snowflake, BigQuery): für **TB bis PB**.<br>**Single-Node-Engines** (DuckDB, Polars): gut bis grob **100 GB**.<br>**Native Python**: hat schon mit **einigen hundert GB** Probleme (außer man verteilt auf viele unabhängige Serverless-Prozesse).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswGu3m6oR8OSXI6omyOk5I" -->
Welche zwei Entscheidungen stecken in „SQL oder Python" (Machado)?::
1. **Ausführungs-Engine** — wer rechnet: **Language Server** (CPython, JVM), **Single-Node-Engine** (DuckDB, Polars, Postgres), **MPP** (Spark, Snowflake, BigQuery).<br>2. **Schnittstelle** — wie man es beschreibt: **native Python**, **DataFrame**, **SQL**.<br>„Code is an interface to the execution engine." Nicht jede Schnittstelle passt zu jeder Engine, aber eine Engine kann mehrere haben (DuckDB: SQL + DataFrame-artige API).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjaoEdgwis1LbzEtpENz0o" -->
Warum muss man bei „SQL vs. Python" zwischen native Python und Python mit Polars/DuckDB unterscheiden?::
**Native Python** (Schleifen über Listen/Dataclasses) läuft in **CPython** — imperativ, langsam, speicherhungrig (GIL, Objekte, Interpreter).<br>**Polars/DuckDB** sind nur die **Schnittstelle**; gerechnet wird in einer **spezialisierten Engine** (Rust/C++), deklarativ, spaltenweise, parallel.<br>Wer das nicht trennt, vergleicht eine Engine mit einer Schreibweise.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMZJCjnAnwQnUiAmYPCzwE" -->
Nach welchen Kriterien wählt man die Engine (Machado)?::
**Performance** · **Art der Transformation** (SQL-artig vs. Schleifen/Rekursion) · **Kosten** · **Datengröße** · **Datenort** (wo die Daten schon liegen, rechnet man am besten) · **Engine-Ort** (z. B. IoT-Gerät) · **deklarativ vs. imperativ** · **Integration** mit anderen Werkzeugen (Python überlegen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCW2fLhxapm3SYGxhEgTyi" -->
Nach welchen Kriterien wählt man die Schnittstelle — und wer gewinnt jeweils bei Machado?::
**Python:** Code-Qualität (Modularität, Wiederverwendung) · Testen · nicht tabellarische Daten · Typen/Prüfung vor der Laufzeit (mypy) · Bibliotheken.<br>**Beide:** Visualisierung/Reports.<br>**SQL:** **Lernkurve und Personal** — mehr Leute können SQL, auch Stakeholder.<br>Rat: Kriterien für den eigenen Fall 1–5 gewichten, gewichteten Durchschnitt nehmen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsr4jigcmmxOO7CX060deic" -->
Was halten Reis & Housley (FDE) der Behauptung entgegen, SQL tauge nicht für komplexe Pipelines?::
„This is false." Mit **CTEs, SQL-Skripten und Orchestrierungswerkzeugen** lassen sich komplexe DAGs bauen.<br>Und umgekehrt: „We often see engineers doing things in Python and Spark that could be more easily and efficiently done in SQL."
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDvZnZjKcvL7q8eOweyc9o" -->
Welche drei Fragen soll man sich laut Reis & Housley stellen, bevor man eine Transformation in Python statt SQL schreibt?::
1. **Wie schwer** ist die Transformation in SQL?<br>2. **Wie lesbar und wartbar** wird der SQL-Code?<br>3. Soll der Code als **wiederverwendbare Bibliothek** dienen?
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstigJpWlp0tXMAs1Y58qYj" -->
Was ist laut Reis & Housley eine der größten Schwächen von SQL?::
**Keine eingebaute Vorstellung von Bibliotheken oder wiederverwendbarem Code.** Was man in Python als Funktion auslagert und importiert, wird in SQL kopiert oder über Templating (dbt/Jinja-Makros) nachgerüstet.<br>Zugleich warnen sie: Programmierbare DataFrame-APIs (Spark) verlangen **mehr Optimierungsgeschick** als SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGGwQkobEQ5HnKPaX1tN1x" -->
Mit welchen Argumenten macht Robin Linacre SQL zur Standardwahl?::
**Verständlich** für viele, auch Analysten und Stakeholder · **zukunftssicher** — „most likely to still be understood 10 or even 20 years from now" · **Gratis-Verbesserungen**: deklarativer Code profitiert ohne Änderung von besseren Engines, DataFrame-Code muss API-Änderungen mitmachen · **DuckDB** macht SQL auf einer Maschine konkurrenzfähig mit den schnellsten DataFrame-Bibliotheken · wenige Abhängigkeiten, **SQLGlot** übersetzt Dialekte · **testbar**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQG0vvhlOYZeyaivtzHMhL" -->
Wann hält auch Linacre SQL für ungeeignet?::
Bei Logik, die sich in SQL nur **schwer oder unlesbar** ausdrücken lässt (seltener als gedacht), bei **Graph-Strukturen** und bei manchen **Zeitreihen**-Operationen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6orKAr3dH3iU0Ie6go9Fu" -->
Wo widersprechen sich Machado und Linacre beim Testen — und wer hat recht?::
**Machado:** „Testing Python code is much easier than testing a SQL transformation" → Python gewinnt.<br>**Linacre:** SQL ist **gut testbar**, wenn man es richtig baut — große Abfragen in **CTEs** teilen, die **einzeln getestet** werden, mit **DuckDB** schnell lokal und in der CI, mit **SQLGlot** auch für andere Ziel-Engines.<br>Beide haben recht unter verschiedenen Annahmen: Machado vergleicht das Übliche, Linacre das Mögliche. Logik mit vielen Fallunterscheidungen bleibt in Python leichter abzusichern.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVK4lUrVc9mB4cAc5Mftvn" -->
Wie lautet die Faustregel aus der dbt-Doku zu SQL vs. Python?::
Kann man eine Transformation **gleich gut** in SQL oder Python schreiben → **gut geschriebenes SQL** ist vorzuziehen.<br>Geht es in SQL **nicht**, oder sparen zehn Zeilen elegantes Python tausend Zeilen schwer lesbares Jinja-SQL → **Python**.<br>Kurz: **SQL, wo möglich — Python, wo nötig.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbK302KsR8tm0ScPwzT8Uh" -->
Was ist in Polars eine Expression, was ein Context?::
**Expression:** eine Berechnung auf Spalten (`pl.col("preis") * 1.1`, `pl.col("x").sum()`), **unabhängig von einem konkreten DataFrame** — speicherbar, kombinierbar, als Funktion wiederverwendbar und testbar.<br>**Context:** wendet Expressions an — `select` (≈ `SELECT`), `with_columns` (Spalten hinzufügen), `filter` (≈ `WHERE`), `group_by().agg()` (≈ `GROUP BY`).<br>Damit löst Polars die Wiederverwendungs-Schwäche von SQL.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfMmRXy2pjcniZaSXJNLaG" -->
Was macht der Lazy-Modus von Polars, das pandas nicht kann?::
Mit `scan_csv`/`scan_parquet` oder `.lazy()` wird nichts sofort ausgeführt; Polars baut einen **Abfrageplan** und rechnet erst bei `.collect()`.<br>Der Optimizer kennt die **ganze Abfrage**: **Predicate Pushdown** (Filter beim Lesen), **Projection Pushdown** (nur benötigte Spalten), **Slice Pushdown**, **Common Subplan Elimination**, Join-Reihenfolge.<br>Plan ansehen: `lf.explain()`. pandas ist **eager** — jede Zeile sofort, keine Optimierung über Schritte.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1NYFtzKfeEnKUX8dhQU7d" -->
Was bleibt von „SQL oder Python" übrig, wenn DuckDB SQL direkt auf einem DataFrame ausführt?::
Die **Engine-Frage** ist entschieden: Dieselbe spaltenorientierte, parallele Engine rechnet, egal ob SQL oder DataFrame-Methoden → Performance ist kein Argument mehr.<br>Übrig bleibt nur die **Schnittstellen-Frage** — und die darf man **pro Schritt** beantworten: SQL für Joins/Aggregationen, Python für Einlesen, Fallunterscheidungen, Tests, Orchestrierung.<br>Mechanik: **Replacement Scan** — der Variablenname im SQL wird als DataFrame aufgelöst (`duckdb.sql("SELECT … FROM df").pl()`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6j7ALFnfau5tOLe1hCXoH" -->
Welche Rolle spielt Apache Arrow zwischen DuckDB, Polars und pandas?::
**Gemeinsames spaltenorientiertes Speicherformat:** Polars nutzt es intern, DuckDB liest und schreibt es, pandas kann es als Grundlage nutzen.<br>Daten wandern zwischen den Bibliotheken deshalb oft **ohne Kopie** — der Wechsel der Schnittstelle kostet fast nichts.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGDj2kby5Tr2yGCYzGCBjd" -->
Ordne SQL, Polars lazy, Polars eager/pandas und native Python zwischen deklarativ und imperativ ein.::
**SQL** (Postgres, DuckDB): deklarativ — der Query-Planer entscheidet.<br>**Polars lazy:** deklarativ — der Polars-Optimizer entscheidet.<br>**Polars eager / pandas:** Mischform — jede Operation mengenorientiert, aber die **Reihenfolge** legt man selbst fest, keine Optimierung über Schritte.<br>**Native Python** (Schleifen über Objekte): imperativ, zeilenweise.<br>Die Linie verläuft also **quer durch Python**, nicht zwischen SQL und Python.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPHhu4uXXEvHSiDVkOeAgd" -->
Welche Geschwindigkeitsgründe von Polars/DuckDB gelten auch für Postgres — und welche nicht?::
✅ **ein Aufruf pro Menge** statt pro Wert (Schleife in C)<br>✅ **Query-Optimizer** (der ausgereifteste)<br>✅ **Auslagern auf die Platte** bei Sortieren/Hash-Joins über `work_mem` hinaus<br>🟡 **mehrere Kerne**: Parallel Query mit wenigen Workern (Standard 2), nicht für alles<br>❌ **spaltenorientierter, dichter Speicher** — Postgres ist **zeilenorientiert**.<br>Deshalb: DuckDB analytisch oft vielfach schneller als Postgres, Postgres vielfach schneller als native Python.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsdk3oaBJJD51qUdQPF8XAE" -->
Warum ist der Ort der Daten bei einer Pipeline auf Postgres-Daten so wichtig?::
Jede Python-Variante muss die Daten erst **herausholen** — über das Netzwerk, und bei nativem Python Zeile für Zeile in Python-Objekte umgewandelt.<br>Schneller: **spaltenweise** laden — Polars `pl.read_database_uri(query, uri)`, DuckDB-`postgres`-Erweiterung (`ATTACH 'dbname=…' AS pg (TYPE postgres)`).<br>SQL **in** Postgres rechnet dagegen dort, wo die Daten liegen. Für einen fairen Vergleich die **Ladezeit getrennt ausweisen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLQmZiLZeJLR4WHPnMkXf6" -->
Was spricht für und gegen SQL als Sprache für ETL-Pipelines?::
**Dafür:** von vielen lesbar · seit Jahrzehnten stabil · deklarativ, profitiert ohne Änderung von besseren Engines · rechnet, wo die Daten liegen · ausgereifte Optimizer · mit CTEs und DuckDB gut testbar.<br>**Dagegen:** keine Bibliotheken, Wiederverwendung nur über Kopieren oder Templating · keine Typprüfung vor der Laufzeit · schwach bei Schleifen, Rekursion, Graphen · Dialektunterschiede · lange Skripte werden unlesbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxFDiBig2X5mbGLnJZNwdZ" -->
Was spricht für und gegen Python (native oder DataFrame) für ETL-Pipelines?::
**Dafür:** Funktionen, Module, Bibliotheken · pytest und mypy · komplexe Logik, Schleifen, ML · nicht tabellarische Daten, APIs, Dateien · Ökosystem · mit Polars/DuckDB ebenso schnell und deklarativ.<br>**Dagegen:** **native** Python skaliert schlecht (Zeit, Speicher) · Daten müssen aus der Datenbank geholt werden · weniger Leute können es pflegen · Abhängigkeiten und API-Änderungen (Polars 2.0, pandas 3.0) kosten über Jahre Wartung · eager DataFrame-Code verleitet zu ineffizienten Mustern.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSihbKl2LEcDAlF6qJP4iA" -->
Was ändert sich mit Polars 2.0 und pandas 3.0, das für einen Ergebnisvergleich wichtig ist?::
**Polars 2.0** (Release Candidate; stabil ist 1.44): Lazy-Modus nutzt standardmäßig die **Streaming-Engine**; **`group_by` und Joins garantieren keine Zeilenreihenfolge** mehr → vor jedem Vergleich **explizit sortieren**.<br>**pandas 3.0:** **Copy-on-Write** ist Standard (abgeleitete DataFrames verhalten sich wie Kopien, keine `SettingWithCopyWarning`), neuer `str`-Typ für Strings.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsysdewdL0y2ZHBu1aVnFVT" -->
Wie vergleicht man zwei Pipeline-Varianten fair — Performance und Code-Qualität?::
**Performance:** eine Variable pro Messung · mehrere Läufe, Median, Aufwärmlauf · mehrere Datenmengen · Ladezeit getrennt · Spitzenspeicher messen (bei Polars/DuckDB z. B. `/usr/bin/time -v`, da außerhalb von Pythons Allokator).<br>**Lesbarkeit:** findet man jeden fachlichen Schritt als benannten Abschnitt wieder?<br>**Testbarkeit:** jeder Schritt einzeln mit Mini-Datensatz testbar, ohne laufende Datenbank?<br>**Erweiterbarkeit:** ein **konkretes Änderungsszenario** in beiden umsetzen und zählen, wie viele Stellen man anfasst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5fdjGCtEyUp1Sx1adPud0" -->
Was sind dbt Python models?::
Python-Modelle in dbt neben den SQL-Modellen, die als DataFrame-Code **im Warehouse** laufen (z. B. Snowpark auf Snowflake, PySpark auf Databricks) — die Daten verlassen das Warehouse nicht.<br>Damit: Python dort, wo SQL nicht reicht, **ohne die Engine zu wechseln**.
<!-- basicblock-end -->
