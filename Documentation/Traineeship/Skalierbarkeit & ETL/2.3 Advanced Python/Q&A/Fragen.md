---
tags:
  - documentation
---

# Offene Fragen — 2.3 Advanced Python

Fragen, die beim Lesen oder Umsetzen aufkommen. Einfach notieren, Claudian beantwortet sie gesammelt am Ende jedes Blocks.

Gehört zu [[2.3 Advanced Python - Leitfaden]] · [[Documentation/Traineeship/Skalierbarkeit & ETL/2.3 Advanced Python/Leseliste|Leseliste]] · [[2.3 Advanced Python - Vertiefung]]

> [!note] Eine Ausnahme
> Fragen, die schon in den Kontrollfragen der Vertiefung stehen, beantworte ich **vorher nicht** — sonst ist der Selbsttest wertlos. Ich weise dann nur darauf hin, dass du sie erst selbst versuchen sollst.

## Block 1 — Concurrency vs. Parallelismus & Thread vs. Prozess

### ✅ Wie gaukelt das Betriebssystem jedem Prozess eine eigene CPU und einen eigenen Speicher vor?

> [!quote]- Deine Frage (mit dem OSTEP-Zitat aus Kap. 26)
> hus far, we have seen the development of the basic abstractions that
> the OS performs. We have seen how to take a single physical CPU and
> turn it into multiple virtual CPUs, thus enabling the illusion of multiple
> programs running at the same time. We have also seen how to create the
> illusion of a large, private virtual memory for each process; this abstrac-
> tion of the address space enables each program to behave as if it has its
> own memory when indeed the OS is secretly multiplexing address spaces
> across physical memory (and sometimes, disk). - Mich interessiert irgendwie auch wie das geht?

Zwei Tricks, beide mit Hardware-Unterstützung. OSTEP nennt das **Virtualisierung** — es ist der erste der „drei einfachen Teile" des Buchs.

**1. Die CPU: Time Sharing mit Timer-Interrupt.** Ein Programm läuft direkt auf dem Prozessor, ohne dass das Betriebssystem jede Instruktion überwacht (*Limited Direct Execution*). Damit das Betriebssystem trotzdem die Kontrolle zurückbekommt, stellt es vor dem Start einen **Hardware-Timer**: Alle paar Millisekunden löst er einen Interrupt aus, und der Prozessor springt zurück in den Betriebssystem-Code. Der sichert die Register des laufenden Prozesses, lässt den **Scheduler** den nächsten auswählen und lädt dessen Register — der [[2.3 Advanced Python - Vertiefung#1.11 Der Context Switch und was er kostet|Context Switch]]. Weil das sehr oft passiert, wirkt es, als liefen alle Programme gleichzeitig. Bestimmte Operationen (Hardware ansprechen, Speichertabellen ändern) darf ein normales Programm gar nicht selbst ausführen; dafür muss es per **System Call** das Betriebssystem bitten.

**2. Der Speicher: virtuelle Adressen und Seitentabellen.** Jedes Programm arbeitet mit **virtuellen Adressen**, die bei 0 beginnen, als gehöre ihm der ganze Speicher. Bei jedem Speicherzugriff übersetzt ein Hardware-Baustein im Prozessor, die **MMU**, die virtuelle Adresse in eine physische. Die Übersetzung geschieht in Blöcken von typischerweise 4 KiB (**Seiten**, *pages*); welche virtuelle Seite in welcher physischen landet, steht in einer **Seitentabelle** — eine pro Prozess. Dadurch:
- kann ein Prozess die Seiten eines anderen gar nicht adressieren — die **Isolation** aus [[2.3 Advanced Python - Vertiefung#1.5 Prozesse|Abschnitt 1.5]] ist eine Eigenschaft der Hardware, nicht des guten Willens;
- muss beim Wechsel zwischen **Prozessen** die Seitentabelle umgeschaltet werden, bei **Threads** desselben Prozesses nicht — einer der Gründe, warum Prozesswechsel teurer sind. Damit die Übersetzung nicht jedes Mal langsam in der Tabelle nachgeschlagen werden muss, gibt es einen Cache dafür, den **TLB**, und der ist nach einem Prozesswechsel kalt;
- kann das Betriebssystem selten benutzte Seiten auf die Festplatte auslagern (**Swapping**). Greift der Prozess darauf zu, löst die MMU einen **Page Fault** aus, und das Betriebssystem lädt die Seite nach. Daher die „sometimes, disk" im Zitat;
- funktioniert **Copy-on-Write** bei `fork()` ([[2.3 Advanced Python - Vertiefung#1.8 Prozesse erzeugen: `fork()` und `exec()`|Abschnitt 1.8]]): Eltern- und Kindprozess zeigen zunächst auf dieselben physischen Seiten, erst beim Schreiben wird kopiert.

**Zum Weiterlesen**, falls dich das packt — alles frei auf der [OSTEP-Seite](https://pages.cs.wisc.edu/~remzi/OSTEP/):
- CPU: Kap. 6 „Limited Direct Execution" (`cpu-mechanisms.pdf`), dann Kap. 7 „CPU Scheduling" (`cpu-sched.pdf`)
- Speicher: Kap. 13 „Address Spaces" (`vm-intro.pdf`), Kap. 15 „Address Translation" (`vm-mechanism.pdf`), Kap. 18 „Introduction to Paging" (`vm-paging.pdf`), Kap. 19 „Translation Lookaside Buffers" (`vm-tlbs.pdf`), Kap. 21 „Swapping: Mechanisms" (`vm-beyondphys.pdf`)

Kap. 6 und Kap. 18 sind die beiden, die die Frage im Kern beantworten (je ~30 Min.). Für das Traineeship ist das Kür, keine Pflicht.

## Block 2 — Warum Python langsam ist & der GIL

### ✅ Warum braucht SQLAlchemy Metaklassen und Dynamik, um Klassen auf Tabellen abzubilden?

> [!quote]- Deine Notiz
> SQLAlchemy mapping classes to database tables with metaclasses
> Warum brauch man dafür metaklassen und dynamik

Weil das Mapping **in dem Moment passieren muss, in dem Python deine Klasse definiert** — und dafür braucht SQLAlchemy einen Haken in die Klassenerzeugung selbst.

**Was beim Schreiben von `class Booking(Base): ...` passieren muss.** Eine `class`-Anweisung ist in Python kein Deklarationsschritt für einen Compiler, sondern **ausführbarer Code**: Python sammelt die Attribute im Klassenrumpf ein und ruft dann den **Typ der Klasse** auf, um daraus das Klassenobjekt zu bauen. Dieser Typ ist normalerweise `type` — ändert man ihn, ist das eine **Metaklasse**. SQLAlchemy nutzt diesen Moment, um

1. die Attribute im Klassenrumpf zu lesen (`Mapped[int]`, `mapped_column(...)`, `relationship(...)`),
2. daraus ein `Table`-Objekt mit Spalten, Typen und Fremdschlüsseln zu bauen und in der `MetaData` zu registrieren — das ist die Grundlage von `Base.metadata.create_all()` und von Alembics Autogenerate,
3. einen **Mapper** zu erzeugen, der Klasse und Tabelle verbindet,
4. die Klassenattribute durch **Deskriptoren** (`InstrumentedAttribute`) zu **ersetzen**.

**Schritt 4 ist der eigentliche Trick.** Danach verhält sich dasselbe Attribut je nach Zugriff völlig verschieden:

```python
Booking.status == "Firm"        # auf der KLASSE: erzeugt einen SQL-Ausdruck, kein True/False
booking.status = "Cancelled"    # auf der INSTANZ: ändert den Wert UND merkt sich die Änderung für den nächsten flush
```

Das funktioniert über das Deskriptor-Protokoll (`__get__`/`__set__`), das du in [[Documentation/Traineeship/Python Basics/Kapselung - Properties, Descriptors und Slots|Kapselung — Properties, Descriptors und Slots]] schon kennst: `@property` ist dieselbe Mechanik.

**Wo genau die Metaklasse sitzt — in SQLAlchemy 2.0.** Nachgesehen in deiner installierten Version (2.0.51, `orm/decl_api.py`): Das eigentliche Mapping passiert in `DeclarativeBase.__init_subclass__` — einem Haken, den Python seit 3.6 für „eine Unterklasse wird gerade erzeugt" anbietet und der für die meisten Fälle eine Metaklasse ersetzt. Eine Metaklasse (`DeclarativeAttributeIntercept`) gibt es trotzdem, für den Rest: Sie fängt `setattr` und `delattr` **auf der Klasse** ab, damit auch ein Attribut, das man *nach* der Definition hinzufügt (`Booking.neue_spalte = mapped_column(...)`), noch gemappt wird. Wer die Metaklasse nicht will, etwa weil sie mit einer anderen kollidiert, nimmt `DeclarativeBaseNoMeta`. In der 1.x-Welt (`declarative_base()`) hat dagegen die Metaklasse `DeclarativeMeta` die ganze Arbeit gemacht.

**Was das mit „Python ist langsam" zu tun hat.** Hopper nennt solche Bibliotheken als Beispiel dafür, wie viel Python-Code von der Dynamik **abhängt**: Klassen, die zur Laufzeit gebaut und umgebaut werden, Attribute, die mal ein Wert und mal ein SQL-Ausdruck sind. Genau diese Freiheit macht es jedem Compiler schwer, vorab zu wissen, was `booking.status` tut — er müsste damit rechnen, dass ein Deskriptor, eine Metaklasse oder ein `__getattr__` dazwischenfunkt. Deshalb kann Python die Dynamik nicht einfach aufgeben, um schneller zu werden: Das Ökosystem ist auf ihr gebaut. Das ist die Kehrseite des Trilemmas aus [[2.3 Advanced Python - Vertiefung#2.1 Die Frage hinter der Frage: Sprache oder Implementierung?|Abschnitt 2.1]].

Zum Nachlesen: [[SQLAlchemy Schema & Beziehungen - Syntax]] aus Kapitel 1.5 zeigt die deklarative Syntax, auf die sich das bezieht.

## Block 3 — Threads und Prozesse in Python

## Block 4 — Generators, Chunking & Memory Management

## Block 5 — Event Loop, asyncio & asynchrone Programmierung

## Block 6 — SQL vs. Python für ETL

## Übungsaufgabe
