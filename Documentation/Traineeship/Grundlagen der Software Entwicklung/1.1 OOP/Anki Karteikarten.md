---
deck: "Traineeship::Grundlagen der Software Entwicklung::OOP"
---

Karteikarten zu [[1.1 OOP - Vertiefung|1]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::OOP`. In Anki werden die Karten außerdem automatisch mit dem Vault-Namen und `ObsidianAnkiSync` getaggt (kein eigener Obsidian-Tag nötig).

# Fortschritt

- [x] Objektorientierung
- [x] Klassen
- [x] Vererbung
- [x] Polymorphie
- [x] Kapselung
- [x] Abstraktion
- [x] Information Hiding
- [x] Separation of Concerns
- [x] Single Responsibility Prinzip
- [x] Loose Coupling vs. strong cohesion
- [x] Open Closed Prinzip
- [x] Grundzüge von Clean Code
- [x] Dependency Injection
- [x] dunder Methoden & dataclasses

---

# Objektorientierung

<!-- basicblock-start oid="ObsXq5XSOEbgT5BAavnCSfu2" -->
Was bündelt ein Objekt in der objektorientierten Programmierung, und was ist die Idee dahinter?::
Zustand (state — Daten/Attribute) und Verhalten (behavior — Methoden, die auf diesen Daten arbeiten) in einer gemeinsamen Einheit. Die Idee: Statt Daten und die sie verarbeitende Logik lose nebeneinander zu haben (wie im prozeduralen Stil), "weiß" das Objekt selbst, wie mit seinen eigenen Daten umzugehen ist, und ist dafür verantwortlich, seinen Zustand konsistent zu halten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSZirz0oVB8HBKvpXS72FJ" -->
Was ist der zentrale Unterschied zwischen prozeduralem und objektorientiertem Stil im Umgang mit Daten und Logik, und welches konkrete Problem hat der prozedurale Stil?::
Prozedural sind Daten (z. B. ein dict mit Preisinformationen) und die Funktionen, die sie verarbeiten, getrennt — nichts hindert daran, ein Feld wie `base_price` an anderer Stelle im Code fälschlich auf einen ungültigen Wert zu setzen oder eine Funktion mit falscher Parameterreihenfolge aufzurufen, weil die Verantwortung über den ganzen Code verstreut ist. OOP bündelt Daten und die zugehörige Logik in einer Klasse — es gibt dadurch einen zentralen "Ort", an dem Regeln/Validierung für diese Daten durchgesetzt werden, was Wartbarkeit und Konsistenz verbessert, sobald die Anwendung wächst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3ESjapKEGCyZ2wxJAaL1c" -->
Nenne und erkläre alle fünf Gründe, warum OOP in vielen professionellen Software-Projekten als Goldstandard gilt.::
1. Modellierung der Realität: Komplexe Systeme lassen sich oft natürlich als Zusammenspiel von Objekten mit eigenen Zuständigkeiten beschreiben (ein Kunde, eine Bestellung, ein Produkt …).<br>2. Kapselung von Komplexität: Ein Objekt kann seine internen Details verstecken und nach außen nur eine einfache Schnittstelle zeigen.<br>3. Wiederverwendbarkeit & Erweiterbarkeit: Über Vererbung und Polymorphie lässt sich Verhalten wiederverwenden und punktuell anpassen, ohne bestehenden Code zu verändern.<br>4. Wartbarkeit: Wenn Zustand und Logik zusammengehören, ist Code leichter zu verstehen, zu testen und zu ändern — Änderungen an der Preislogik passieren an einer Stelle, nicht an zehn verstreuten Funktionsaufrufen.<br>5. Klarere Verantwortlichkeiten: Jedes Objekt hat eine klar abgegrenzte Aufgabe (Single Responsibility Prinzip).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuK4ScCmAG2K0n23r2RFs2" -->
Ist OOP ein Selbstzweck, den man immer maximal einsetzen sollte? Was kann bei übertriebenem Einsatz schiefgehen?::
Nein — OOP kann überstrapaziert werden, z. B. durch unnötig tiefe Klassenhierarchien oder "God Objects" (Klassen, die zu viel Verantwortung auf sich vereinen). Ein gesundes Maß und ein solides Verständnis der zugrunde liegenden Prinzipien ist wichtiger, als OOP um jeden Preis einzusetzen — manchmal ist eine einfache Funktion oder ein einfaches dict die klarere Lösung.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs1N4LAb9TYjIbs89C0b08i" -->
In Python ist praktisch alles ein ==Objekt== — selbst Funktionen haben Attribute wie `__name__` und einen `type()`; für eine normale, mit `def` definierte Funktion `greet` liefert `type(greet)` z. B. `<class 'function'>`.
<!-- clozeblock-end -->

<!-- basicblock-start oid="Obs9LB88qAWcqEl4pyOwVOeb" -->
Was gibt `type(meine_funktion)` zurück, wenn `meine_funktion` eine normale, mit `def` definierte Python-Funktion ist, und was zeigt das über Pythons Objektmodell?::
`<class 'function'>` — auch Funktionen sind in Python waschechte Objekte mit eigenem Typ und eigenen Attributen (z. B. `meine_funktion.__name__`). Das zeigt: Python ist bis in den Kern objektorientiert aufgebaut, nicht nur "verfügbar" für OOP wie in manch anderer Sprache.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVY3r3UpGXhU9usZEemrpu" -->
Nenne die vier klassischen Grundpfeiler der OOP und beschreibe jeden in einem Satz.::
Abstraktion (nur die relevanten Eigenschaften eines Konzepts modellieren, Details weglassen), Kapselung (Zustand und Verhalten bündeln und den Zugriff auf den internen Zustand kontrollieren), Vererbung (Klassen können Verhalten/Struktur von anderen Klassen übernehmen und erweitern, "ist-ein"-Beziehung) und Polymorphie (Objekte unterschiedlicher Klassen lassen sich über dieselbe Schnittstelle einheitlich ansprechen, obwohl jede Klasse dahinter eine eigene Implementierung hat).
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsQMZ711nlH44iyFmKGZ6vW" -->
==Abstraktion== bedeutet: nur die relevanten Eigenschaften eines Konzepts modellieren, Details weglassen.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsJcR8KbXXRs71FINPHcaUB" -->
==Kapselung== bedeutet: Zustand und Verhalten bündeln und den Zugriff auf den internen Zustand kontrollieren.
<!-- clozeblock-end -->

# Klassen

<!-- basicblock-start oid="ObsB17U5EC87iZThAjfJAyta" -->
Was ist der Unterschied zwischen einer Klasse und einer Instanz? Nenne eine Analogie.::
Eine Klasse ist der Bauplan (blueprint), der definiert, welche Attribute/Methoden Objekte dieses Typs haben werden. Eine Instanz ist ein konkretes, im Speicher existierendes Objekt, das nach diesem Bauplan erzeugt wurde — zwei Instanzen derselben Klasse sind unabhängige Objekte (`unit_a is unit_b` → `False`), auch wenn beide vom selben Typ sind (`type(unit_a) is PriceableUnit` → `True`). Analogie: Die Klasse ist wie der Bauplan für "ein Haus mit Grundriss X", Instanzen sind die tatsächlich gebauten Häuser — gleiche Struktur, aber unabhängige Zustände (unterschiedliche Bewohner, Möbel).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4vvPfLfy7Ie337i1duU36" -->
Wofür steht `self` als erster Parameter einer Python-Instanzmethode, und ist es ein reserviertes Keyword?::
`self` verweist auf die konkrete Instanz, auf der die Methode aufgerufen wird — darüber greift die Methode auf die Attribute und anderen Methoden genau dieses Objekts zu. Es ist KEIN Keyword, sondern reine Konvention (der Name könnte technisch anders lauten), aber als erster Parameter jeder Instanzmethode universeller Python-Standard und sollte nie geändert werden, damit Code für andere lesbar bleibt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6ZiEJq6alMiIiUI2ydazE" -->
Wie übersetzt Python intern den Aufruf `unit.calculate_price_recommendation()`, und wie lässt sich das direkt beobachten?::
Zu `PriceableUnit.calculate_price_recommendation(unit)` — die Instanz wird automatisch als erstes Argument (`self`) mitgegeben. Direkt beobachtbar: `unit.calculate_price_recommendation()` und `PriceableUnit.calculate_price_recommendation(unit)` liefern exakt dasselbe Ergebnis, weil es sich um denselben Aufruf handelt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNpKnY9gVu2zeryyMhDC47" -->
Was ist der Unterschied zwischen einem Instanzattribut und einem Klassenattribut?::
Instanzattribute werden pro Objekt in `__init__` (via `self.x = ...`) gesetzt — jede Instanz hat eine eigene, unabhängige Kopie. Klassenattribute werden im Klassenkörper außerhalb von `__init__` definiert und von ALLEN Instanzen geteilt, solange sie nicht überschrieben werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGfS52ipfYdeltekDscUF9" -->
Eine Klasse hat ein Klassenattribut `currency = "EUR"`. Was passiert, wenn du `a.currency = "USD"` auf einer Instanz `a` ausführst, und wie lässt sich das über `__dict__` überprüfen?::
Es wird KEIN Update des Klassenattributs — es wird ein neues Instanzattribut nur auf `a` angelegt. Lesen fällt (falls kein Instanzattribut existiert) auf das Klassenattribut zurück, Schreiben legt aber immer ein Instanzattribut an. Sichtbar über `a.__dict__` (enthält jetzt `'currency': 'USD'`) im Vergleich zu `PriceableUnit.__dict__` (weiterhin `'currency': 'EUR'`) und `b.__dict__` einer anderen, unveränderten Instanz (enthält `currency` gar nicht).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2qOWme6rsWe9CLKR1NS3I" -->
Warum sind veränderliche (mutable) Klassenattribute wie `applied_discounts = []` in Python gefährlich?::
Weil diese eine Liste zwischen ALLEN Instanzen geteilt wird. `a.applied_discounts.append(...)` ist ein Lesezugriff auf die geteilte Liste gefolgt von einer Mutation — keine neue Zuweisung — daher sieht auch `b.applied_discounts` die Änderung, obwohl nur `a` verändert wurde.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskXG8XT1VMHGFpZup7og7G" -->
Wie behebt man das Problem mit geteilten mutable Klassenattributen korrekt?::
Das veränderliche Objekt (z. B. eine Liste) in `__init__` pro Instanz neu erzeugen (`self.applied_discounts = []`), statt es als Klassenattribut zu definieren. So bekommt jede Instanz ihre eigene, unabhängige Liste.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsVDSMhztVVtTrSFmhswKye" -->
Das gleiche Problem wie bei mutable Klassenattributen (eine gemeinsam genutzte, veränderliche Liste) tritt bei ==mutable default arguments== von Funktionen auf, z. B. `def f(items=[])`: Der Default wird nur einmal beim Definieren der Funktion erzeugt und danach bei jedem Aufruf ohne explizites Argument wiederverwendet und ggf. mutiert.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsFTRt2IrhEBD4jmknTAby5" -->
Was ist eine Instanzmethode technisch gesehen, verglichen mit einer normalen Funktion?::
Eine ganz normale Funktion, die im Klassenkörper definiert ist und als ersten Parameter `self` erwartet. Der einzige "besondere" Mechanismus ist, dass Python beim Aufruf über eine Instanz automatisch diese Instanz als `self` mitgibt (siehe Übersetzung `unit.methode()` → `Klasse.methode(unit)`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6s6IdaBpfxKGEaKZmFuot" -->
Warum wird `isinstance(obj, Klasse)` gegenüber `type(obj) == Klasse` bevorzugt? Nenne ein konkretes Szenario, in dem sich das bemerkbar macht.::
Weil `isinstance()` auch bei Vererbung korrekt funktioniert — sie berücksichtigt die komplette Vererbungskette (MRO), nicht nur den exakten Typ. Ist `obj` z. B. eine Instanz von `DigitalProduct(PriceableUnit)`, liefert `isinstance(obj, PriceableUnit)` → `True`, aber `type(obj) == PriceableUnit` → `False`, obwohl `obj` funktional durchaus als `PriceableUnit` behandelt werden können sollte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbqVMv1z4YNzuGupPC07AL" -->
Womit lässt sich zur Laufzeit anzeigen, welche Instanzattribute ein konkretes Objekt gerade besitzt (als dict)?::
`obj.__dict__` — nützlich zum Debuggen, um direkt zu sehen, welche Instanzattribute tatsächlich (und ggf. abweichend vom Klassenattribut) auf einem konkreten Objekt gesetzt sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswONUdmnRQ9ZOWXzwXqDe0" -->
Ist eine Klasse in Python selbst auch ein Objekt? Was gibt `type(PriceableUnit)` zurück?::
Ja — jede Klasse ist selbst eine Instanz von `type`. `type(PriceableUnit)` ergibt `<class 'type'>`. (Führt zum Konzept der Metaklassen — Out of Scope für dieses Kapitel, aber gut zu wissen, dass es in Python "Objekte bis ganz nach unten" gibt.)
<!-- basicblock-end -->

# Vererbung

<!-- basicblock-start oid="ObsjNGSYVhMDhyz84c9ReCg0" -->
Woran erkennst du, ob zwischen zwei Konzepten eine "ist-ein"-Beziehung (Vererbung) oder eine "hat-ein"-Beziehung (Komposition) sinnvoller ist?::
Prüf-Frage: Ergibt der Satz "Subklasse ist ein/eine Basisklasse" wirklich Sinn — und würde die Subklasse die komplette öffentliche Schnittstelle der Basisklasse sinnvoll erfüllen (nicht nur Teile davon)? Wenn ein Objekt ein anderes nur als Werkzeug/Baustein benutzt, ohne selbst eine speziellere Variante davon zu sein, ist es "hat-ein" → Komposition. Faustregel im Zweifel: *favor composition over inheritance*.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSK1dimTL4pObuoXXY7Sh8" -->
Wie definiert man in Python eine Subklasse, die von einer Basisklasse erbt, und was übernimmt sie dadurch automatisch?::
Syntax: `class Subklasse(Basisklasse): ...`. Die Subklasse übernimmt dadurch automatisch alle Attribute und Methoden der Basisklasse und kann sie unverändert nutzen, erweitern oder überschreiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvGJym37sIYc0OSosllsub" -->
Was macht `super().__init__(...)` in einer Subklasse konkret, und was passiert, wenn man es weglässt, obwohl die Subklasse zusätzliche Attribute hat?::
Es ruft die `__init__` der Basisklasse auf und führt deren Initialisierungscode aus (z. B. `self.unit_id = ...`, `self.base_price = ...`). Lässt man es weg, werden die Attribute der Basisklasse NICHT gesetzt — jeder spätere Zugriff darauf löst einen `AttributeError` aus, weil das Attribut nie angelegt wurde.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvGddNzHItDMQT1154iiIt" -->
Was bedeutet "Method Overriding", und wie kannst du innerhalb einer überschreibenden Methode trotzdem die Elternversion mitausführen?::
Method Overriding bedeutet, dass eine Subklasse eine geerbte Methode mit demselben Namen neu definiert — zur Laufzeit wird immer die Version der konkretesten (Sub-)Klasse verwendet. Mit `super().methodenname()` lässt sich innerhalb der überschreibenden Methode zusätzlich die Implementierung der Basisklasse aufrufen, um Verhalten zu erweitern statt komplett zu ersetzen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJri3XY1IqliJuph1p8UXH" -->
Was ist die Method Resolution Order (MRO), und wie lässt sie sich für eine Klasse anzeigen?::
Die MRO ist die Reihenfolge, in der Python beim Methodenaufruf nach einer Implementierung sucht: zuerst in der Klasse der tatsächlichen Instanz, dann in deren Basisklasse(n), usw. bis `object`. Anzeigen lassen über `KlassenName.__mro__` (Tupel) oder `KlassenName.mro()` (Liste).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAEp3vekpdCO3ORlrW106F" -->
Was unterscheidet Python von Java in Bezug auf Vererbung, und welches klassische Problem kann dadurch entstehen?::
Python erlaubt Mehrfachvererbung — eine Klasse kann gleichzeitig von mehreren Basisklassen erben (Java erlaubt das nicht, dort maximal eine Basisklasse). Das kann zum "Diamond Problem" führen, wenn mehrere Basisklassen dieselbe Methode definieren; Python löst das deterministisch über den C3-Linearisierungsalgorithmus (nachvollziehbar über `__mro__`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsoTyG4nImDTjQDrd2pQ84y" -->
Was ist ein "Mixin", und wofür wird es typischerweise eingesetzt?::
Ein Mixin ist eine kleine, fokussierte Basisklasse, die genau eine Zusatzfähigkeit beisteuert (z. B. `TimestampedMixin` mit einer `touch()`-Methode) und selbst keinen komplexen eigenen Zustand mitbringt. Es wird per Mehrfachvererbung mit "echten" Basisklassen kombiniert, ohne die Klassenhierarchie unübersichtlich zu machen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBnylgBzLYgC8UJETQfItb" -->
Was besagt die Faustregel "favor composition over inheritance", und wann sollte man Komposition statt Vererbung wählen?::
Sie besagt, dass man Objektkomposition ("hat-ein", ein Objekt benutzt ein anderes als Baustein) der Vererbung ("ist-ein") vorziehen sollte, wenn beides technisch möglich wäre. Komposition ist sinnvoll, wenn der Baustein unabhängig getestet, wiederverwendet und ausgetauscht werden soll (z. B. eine austauschbare `PercentageDiscount`-Klasse), statt eine tiefe, starre Klassenhierarchie mit vielen Kombinationen aufzubauen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs83I6ocYwehOZrTxGzoITI" -->
`DigitalProduct` erbt von `PriceableUnit`, `ebook` ist eine `DigitalProduct`-Instanz. Was liefert `isinstance(ebook, PriceableUnit)`? Und was ist der Unterschied zu `issubclass()`?::
`True` — `isinstance()` berücksichtigt die gesamte (transitive) Vererbungskette. `issubclass(KlasseA, KlasseB)` vergleicht dagegen zwei Klassen direkt (ohne Instanz) und prüft, ob `KlasseA` eine (Sub-)Klasse von `KlasseB` ist, z. B. `issubclass(DigitalProduct, PriceableUnit)` → `True`, aber `issubclass(PriceableUnit, DigitalProduct)` → `False`.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obse4XWNGkDQ5D6sShMbq87e" -->
==Vererbung== bedeutet: Klassen können Verhalten/Struktur von anderen Klassen übernehmen und erweitern — Basisklasse/Elternklasse → Subklasse/Kindklasse, typischerweise bei einer "ist-ein"-Beziehung.
<!-- clozeblock-end -->

# Polymorphie

<!-- basicblock-start oid="ObsuFn1S9jcwibayBFnTIqQ1" -->
Was bedeutet Polymorphie ("viele Formen"), und welchen praktischen Vorteil hat sie für aufrufenden Code?::
Objekte unterschiedlicher Klassen lassen sich über dieselbe Schnittstelle (denselben Methodennamen) ansprechen, obwohl jede Klasse dahinter eine eigene, passende Implementierung hat. Der aufrufende Code muss den konkreten Typ nicht kennen und keine Fallunterscheidung (`if isinstance(...): ... elif ...`) vornehmen — er ruft einfach die gemeinsame Methode auf, z. B. beim Iterieren über eine gemischte Liste von `PriceableUnit`-Subklassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss8PeDBI50EIZ3eDUF4d36" -->
Was ist der Unterschied zwischen Polymorphie über Vererbung (nominal) und Polymorphie über Duck Typing (strukturell) in Python?::
Bei nominaler Polymorphie erben die beteiligten Klassen von einer gemeinsamen Basisklasse und überschreiben deren Methoden — die Beziehung ist explizit über die Klassenhierarchie sichtbar. Bei struktureller Polymorphie (Duck Typing) reicht es, dass zwei Klassen ganz ohne gemeinsame Basisklasse dieselbe Methode anbieten — Python prüft zur Laufzeit nur, ob die aufgerufene Methode existiert, nicht die Herkunft der Klasse.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrUpdMYI7mNQIYykFizTxD" -->
Ist Operator Overloading eine Form von Polymorphie? Wie nennt man diese spezielle Form?::
Ja — man spricht von Ad-hoc-Polymorphie: derselbe Operator (z. B. `+`) verhält sich je nach Typ der beteiligten Objekte unterschiedlich (`int.__add__` vs. `Money.__add__` vs. `str.__add__`), weil Python den Operator intern auf die passende Dunder-Methode des jeweiligen Typs übersetzt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbzmBsAwlS7MluHAGYp6HJ" -->
Was macht `functools.singledispatch`, und wie unterscheidet es sich von Polymorphie über Methodenüberschreibung?::
`singledispatch` erlaubt es, für eine freistehende Funktion typspezifische Überladungen zu registrieren (`@funktion.register`), die anhand des Typs des ERSTEN Arguments ausgewählt werden — eine eingeschränkte Form von "multiple dispatch". Anders als bei Polymorphie über Methodenüberschreibung liegt die typabhängige Logik nicht in Methoden einer Klassenhierarchie, sondern in separat registrierten Funktionen außerhalb der Klassen.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsKcMVH7y1L68spNt68ZtPO" -->
==Polymorphie== bedeutet: Objekte unterschiedlicher Klassen können über dieselbe Schnittstelle einheitlich angesprochen werden, obwohl jede ihre eigene, passende Implementierung mitbringt.
<!-- clozeblock-end -->

# Kapselung

<!-- basicblock-start oid="ObsQHnvdFjRoR2Oruvtf3DLN" -->
Was bedeutet Kapselung, und welches Ziel verfolgt sie (nicht nur "Zustand und Verhalten bündeln")?::
Kapselung heißt: Zustand und Verhalten werden in einer Klasse gebündelt, UND der Zugriff auf den internen Zustand wird kontrolliert, statt ihn völlig offenzulegen. Ziel: Ein Objekt soll seine eigenen Invarianten (z. B. "der Preis ist nie negativ") selbst durchsetzen können, statt sich darauf zu verlassen, dass jeder Aufrufer überall im Code brav damit umgeht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGEVD8JNo3AXXkLgung0ra" -->
Hat Python eine echte Zugriffskontrolle wie `private`/`protected`/`public` in Java? Was gibt es stattdessen?::
Nein — Python kennt keine echte, vom Interpreter durchgesetzte Zugriffskontrolle. Stattdessen gibt es zwei Namenskonventionen: einfacher Unterstrich `_attribut` ("bitte nicht von außen anfassen", aber technisch frei zugänglich) und doppelter Unterstrich `__attribut` (löst Name Mangling aus, ist aber ebenfalls kein Sicherheitsmechanismus, sondern primär gegen Namenskollisionen bei Vererbung gedacht).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGjTrxPe0cl8woOcR7lKj4" -->
Was macht Python technisch mit einem Attribut `__base_price`, das in `__init__` gesetzt wird (Name Mangling), und wofür ist dieser Mechanismus eigentlich gedacht?::
Python benennt es intern in `_KlassenName__base_price` um (sichtbar in `obj.__dict__`). Der Zugriff über `obj._KlassenName__base_price` funktioniert weiterhin — es ist also keine echte Zugriffssperre. Der Mechanismus ist primär dafür gedacht, Namenskollisionen bei Vererbung zu vermeiden, wenn mehrere Klassen einer Hierarchie unabhängig voneinander ein `__x` verwenden wollen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTZwrpXKmmR7hzF4htNvNi" -->
Du hast ein öffentliches Attribut `unit.base_price` und willst nachträglich eine Validierung einbauen, ohne den Code aller bestehenden Aufrufer zu ändern. Wie löst du das in Python?::
Mit `@property`/`@x.setter`: Das Attribut wird intern in `self._base_price` umbenannt, ein `@property`-Getter gibt den Wert zurück und ein `@x.setter` validiert bei jeder Zuweisung (z. B. `raise ValueError` bei negativem Wert). Aufrufender Code schreibt weiterhin ganz normal `unit.base_price` bzw. `unit.base_price = 15` — die öffentliche Schnittstelle ändert sich nicht, obwohl jetzt Validierung dahintersteckt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXZk0IY5ICRiImFb6UOvYp" -->
Was ist eine `@property` ohne definierten Setter, und wofür eignet sie sich?::
Ein read-only, berechnetes Attribut: Es gibt nur einen Getter, der einen Wert aus anderen Attributen berechnet (z. B. `price_with_vat`, berechnet aus `base_price` und `vat_rate`). Von außen sieht es wie ein normales Attribut aus (`unit.price_with_vat`), ein Zuweisungsversuch (`unit.price_with_vat = 150`) löst aber `AttributeError: can't set attribute` aus, weil kein Setter existiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXQb4DCXQoOxHlbKmYtkBO" -->
Was ist der Unterschied zwischen `@property` und `functools.cached_property`, und wann ist Letzteres riskant?::
`@property` berechnet den Wert bei JEDEM Zugriff neu. `@cached_property` berechnet ihn nur beim ersten Zugriff und speichert das Ergebnis danach pro Instanz zwischen (spätere Zugriffe sind sofort, ohne erneute Berechnung). Riskant ist es, wenn sich die zugrunde liegenden Attribute später ändern: Der gecachte Wert wird NICHT automatisch neu berechnet und bleibt veraltet stehen, bis er manuell aus `obj.__dict__` gelöscht wird.
<!-- basicblock-end -->

# Abstraktion

<!-- basicblock-start oid="ObshOwHUSubSTOXljfxgiV40" -->
Was bedeutet Abstraktion in der OOP?::
Beim Modellieren eines Konzepts bewusst nur die für den jeweiligen Zweck relevanten Eigenschaften abzubilden und alles andere wegzulassen. Eine `PriceableUnit` braucht z. B. `base_price` und eine Preisberechnungs-Methode — nicht aber Details wie Herstellung oder Lieferanten, weil das für "Preisempfehlungen berechnen" irrelevant ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmIovUByNwqrxK4OLHOWfb" -->
Was unterscheidet Abstraktion von Kapselung? Nenne eine Analogie.::
Abstraktion ist eine Design-Entscheidung: WAS muss ein Nutzer des Konzepts überhaupt wissen/können, welche Details sind irrelevant? Kapselung ist der Implementierungsmechanismus: WIE wird der Zugriff auf interne Details technisch kontrolliert (Konventionen, `@property`). Analogie Autofahren: Der Fahrer nutzt die Abstraktion "Lenkrad, Pedale" (das reicht zum Fahren) - wie der Motor im Detail funktioniert, bleibt dank Kapselung verborgen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6R5VN78je941ZjobxbB2j" -->
Wie definiert man in Python eine abstrakte Klasse, und warum würde man das tun?::
Über das `abc`-Modul: Die Klasse erbt von `ABC`, mindestens eine Methode wird mit `@abstractmethod` markiert (nur Signatur, kein oder ein Platzhalter-Body). Man tut das, um zu erzwingen, dass jede konkrete Subklasse eine bestimmte Fähigkeit garantiert selbst implementiert - dokumentiert direkt im Code statt nur per Kommentar/Konvention.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsT0RCOYy0fFM5qkBO7ZpdE" -->
Was unterscheidet eine abstrakte Klasse von einer nicht-abstrakten (normalen) Klasse? Welcher Fehler tritt wann auf, wenn man es trotzdem versucht?::
Eine normale Klasse ist immer vollständig und direkt instanziierbar. Eine abstrakte Klasse ist absichtlich unvollständig: Sie kann NICHT instanziiert werden, solange mindestens eine `@abstractmethod` nicht überschrieben wurde. Der Versuch (`PriceableUnit(...)` direkt aufrufen) löst einen `TypeError` aus - und zwar erst zur Laufzeit beim Instanziierungsversuch, nicht schon beim Import/Parsen der Datei.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLAsEBBdhpUAnAdbRMIpoz" -->
Wozu dienen Interfaces in Sprachen wie Java, und wie lässt sich vergleichbare Funktionalität in Python erreichen?::
Ein Interface definiert ausschließlich Methodensignaturen ohne Implementierung und ohne eigenen Zustand - ein reiner Vertrag. Eine Klasse kann (anders als bei Vererbung, wo nur eine Basisklasse erlaubt ist) beliebig viele Interfaces implementieren. In Python vergleichbar über: (1) eine abstrakte Basisklasse ohne konkrete Implementierung (nur `@abstractmethod`s), oder (2) `typing.Protocol` für strukturelle Typisierung/Duck Typing ohne erzwungene Vererbungsbeziehung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsddGc0366Ewn1720nBJXfm" -->
Was unterscheidet ein Interface von einer abstrakten Klasse (Implementierung, Zustand, Mehrfachimplementierung)?::
Ein Interface hat KEINE Implementierung und KEINEN eigenen Zustand, eine Klasse kann viele Interfaces implementieren. Eine abstrakte Klasse kann bereits konkrete, fertig implementierte Methoden UND eigenen Zustand (Attribute) mitbringen, erlaubt aber (klassisch, wie in Java) nur eine einzige Basisklasse. Kurzfassung: Interface = reiner Vertrag ohne Substanz, abstrakte Klasse = teilweise fertige Basis mit Verträgen UND schon fertigem Verhalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZ9eqNYf78fvR5MXPbqigQ" -->
Warum ist es oft sinnvoller, nach außen ein Interface/Protocol zu kommunizieren, statt eine konkrete Klasse zu erwarten?::
Wenn eine Funktion/ein Modul nach außen nur ein Interface bzw. Protocol erwartet statt eine konkrete Klasse, bleibt die konkrete Implementierung austauschbar (z. B. CSV-Export → JSON-Export → Datenbank-Export), ohne den aufrufenden Code anzufassen. Kommuniziert man dagegen konkrete Klassen, koppelt man den Aufrufer fest an eine spezifische Implementierung - jede Änderung daran zwingt potenziell auch den Aufrufer zur Änderung (verstößt gegen Loose Coupling).
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs2UMdSDmwUtuMae2bGFrvx" -->
==Abstraktion== ist die Design-Entscheidung, welche Eigenschaften/Schnittstelle relevant sind; ==Kapselung== ist der Mechanismus, mit dem der Zugriff auf die dahinterliegenden Details technisch kontrolliert wird.
<!-- clozeblock-end -->

# Information Hiding

<!-- basicblock-start oid="ObsF7YXwDpT5RTgxCTR8m4qD" -->
Was ist Information Hiding, und welchen konkreten Nutzen bringt es?::
Das übergeordnete Design-Prinzip: Implementierungsdetails werden vor der Außenwelt verborgen, damit sie sich später ändern lassen, ohne Aufrufer zu brechen. Nach außen wird nur eine stabile Schnittstelle gezeigt. Nutzen: weniger Kopplung (Aufrufer hängt nur von der Schnittstelle ab, nicht von der internen Umsetzung) und weniger kognitive Last (man muss die Interna nicht verstehen, um das Objekt zu benutzen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQTOygnexWb2wvYNCWpxqN" -->
Du änderst die interne Speicherung von Rabatt-Codes einer Klasse von einer `list` zu einem `set` (z. B. wegen Performance). Unter welcher Voraussetzung merkt der Aufrufer davon nichts?::
Solange die öffentliche Schnittstelle (Methodennamen, Parameter, Rückgabewerte wie `add_discount()`/`has_discount()`) unverändert bleibt und das interne Attribut mit Unterstrich-Konvention (`_applied_discount_codes`) als nicht-öffentlich markiert ist, kann die interne Datenstruktur beliebig ausgetauscht werden, ohne dass Aufrufer-Code angepasst werden muss.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNQWNw9MGfwLn5G9MeZxQy" -->
Abstraktion, Kapselung und Information Hiding beantworten je eine andere Frage (WAS/WIE/WARUM) — welche ist welche?::
Abstraktion beantwortet WAS nach außen sichtbar/relevant sein soll (Design-Entscheidung). Kapselung beantwortet WIE der Zugriff technisch kontrolliert wird (Mechanismus, z. B. `_attribut`, `@property`). Information Hiding beantwortet WARUM man das überhaupt macht (Ziel: Änderungsfreiheit, lose Kopplung).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLg2EHyCKu0I2Aiw2iSYyc" -->
Was bedeutet `__all__` in einer `__init__.py`, und wie hängt das mit Information Hiding zusammen?::
`__all__` legt explizit fest, welche Namen bei `from package import *` überhaupt exportiert werden — alles andere gilt als internes Detail des Pakets. Das überträgt denselben Gedanken wie `_attribut` auf Klassenebene (interne Details verstecken, nur eine bewusste Auswahl als öffentliche API zeigen) auf die Modul-/Paketebene.
<!-- basicblock-end -->

# Separation of Concerns

<!-- basicblock-start oid="ObstzWMeKqiYDUitYqwhzx5c" -->
Was besagt Separation of Concerns (SoC)?::
Unterschiedliche Belange ("concerns") eines Programms sollten in getrennte, unabhängige Bausteine (Module, Klassen, Funktionen, Schichten) aufgeteilt werden, statt sie miteinander zu verweben. Ändert sich ein Belang (z. B. "wie wir Daten speichern"), soll das möglichst nur den dafür zuständigen Baustein betreffen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsndX19v3TZGAFt5rGX0i8l" -->
Warum sollte `calculate_all_price_recommendations()` (aus der Übungsaufgabe) die CSV-Schreiblogik nicht direkt selbst enthalten?::
Preisberechnung (Business-Logik) und CSV-Export (I/O) sind zwei unterschiedliche Concerns. Vermischt man sie in einer Funktion, lässt sich die Preislogik nicht ohne echtes Dateisystem testen, und ein Wechsel des Export-Formats (CSV → JSON → Datenbank) würde die Preislogik mit anfassen. Getrennt in z. B. `PriceRecommendationCalculator` (nur Business-Logik) und `CsvPriceRecommendationWriter` (nur I/O) bleibt beides unabhängig änderbar und einzeln testbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmzMhUw8FCN8UVusOoS6Fc" -->
Was ist der Unterschied zwischen Separation of Concerns (SoC) und dem Single Responsibility Principle (SRP)?::
SoC ist das allgemeinere Prinzip und gilt auf jeder Granularitätsstufe (Schichten, Module, Klassen, Funktionen). SRP ist im Grunde SoC, konkret angewendet auf die Ebene einer einzelnen Klasse: "Eine Klasse sollte nur einen Grund haben, sich zu ändern." SRP lässt sich also als Spezialfall von SoC für Klassendesign verstehen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsh7BKLvXFb5dGpCd4jFyax" -->
Nenne die drei klassischen Schichten einer Schichtenarchitektur (Layers) und was jede grob verantwortet.::
Präsentationsschicht (UI/API-Endpunkte - nimmt Eingaben entgegen, zeigt Ergebnisse), Business-Logik-Schicht (die eigentlichen fachlichen Regeln/Berechnungen) und Datenzugriffsschicht (Persistenz: Datenbank, Dateien, externe APIs - wie/wo Daten herkommen bzw. landen). Jede Schicht kommuniziert idealerweise nur mit der direkt benachbarten Schicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss1xxA8C06gRU8IUyyTHZa" -->
Warum wird Code, der Concerns sauber trennt, tendenziell leichter testbar?::
Weil der "unsaubere" Teil (Dateisystem, Datenbank, Netzwerk) beim Testen leicht durch ein Test-Double ersetzt werden kann, wenn er nicht mit der eigentlichen Business-Logik verwoben ist. Eine reine Business-Logik-Komponente ohne I/O lässt sich mit einfachen Eingabe-/Rückgabewerten testen, ganz ohne echtes Dateisystem oder echte Datenbank — Brücke zum späteren Stichwort Dependency Injection.
<!-- basicblock-end -->

# Single Responsibility Prinzip

<!-- basicblock-start oid="ObsxowkzO3XOTccyYgQa7Xcu" -->
Was besagt das Single Responsibility Principle (SRP), und was ist mit "ein Grund zur Änderung" konkret gemeint?::
"Eine Klasse sollte genau einen Grund haben, sich zu ändern" (Robert C. Martin). Gemeint ist eine Achse der Veränderung / ein Stakeholder-Bereich: Wie viele fachlich unabhängige Anlässe gäbe es, diese Klasse zu ändern? Nicht gemeint ist "eine Methode" oder "eine Codezeile".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSriusX2Z0wQxTFspX5a5Q" -->
Eine Klasse `PriceableUnit` berechnet Preise, validiert Input-Rohdaten UND verschickt Benachrichtigungen. Warum verletzt das SRP, und wie löst man es auf?::
Es gibt drei unabhängige Gründe zur Änderung (Preisregeln, Validierungsregeln, Benachrichtigungslogik) mit potenziell unterschiedlichen Stakeholdern in einer Klasse gebündelt. Auflösung: in separate Klassen mit je einer Verantwortung aufteilen (z. B. `PriceableUnit`, `PriceableUnitInputValidator`, `PriceChangeNotifier`) — genau Separation of Concerns, angewendet auf Klassenebene.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsldVTXvd5wwHzGtmgoDMrp" -->
Nenne zwei praktische Heuristiken, um eine SRP-Verletzung im eigenen Code zu erkennen.::
(1) "Und"-Test: Lässt sich die Verantwortung der Klasse nur mit einem "und" beschreiben ("macht X UND Y UND Z")? (2) God Object/God Class: Eine Klasse, die immer mehr Verantwortlichkeiten ansammelt, weil es "gerade bequem" war. (Weitere mögliche Antwort: Änderungshäufigkeit aus fachlich völlig unterschiedlichen Anlässen in der Versionshistorie.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvHmA0zNDfE2lTYNDCbk00" -->
Stimmt es, dass SRP verlangt, dass eine Klasse nur eine einzige Methode haben darf?::
Nein, das ist ein verbreitetes Missverständnis. Eine Klasse darf durchaus mehrere Methoden haben, solange sie alle demselben EINEN fachlichen Zweck dienen und aus demselben Grund geändert würden. SRP verbietet nicht viele Methoden, sondern viele voneinander unabhängige Zwecke in derselben Klasse.
<!-- basicblock-end -->

# Loose Coupling vs. hohe Kohäsion

<!-- basicblock-start oid="Obs31PLNbH0K6FJ1vXzt96NX" -->
Was ist der Unterschied zwischen Kopplung (Coupling) und Kohäsion (Cohesion)?::
Kopplung misst, wie stark unterschiedliche Klassen/Module VONEINANDER abhängen (Ziel: lose Kopplung — Änderungen an A betreffen möglichst wenig andere Klassen). Kohäsion misst, wie stark die Dinge INNERHALB einer Klasse zusammengehören (Ziel: hohe Kohäsion — alle Attribute/Methoden dienen einem gemeinsamen Zweck). Faustregel: "loose coupling, high cohesion" gemeinsam anstreben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseFT7ndKFQdgZpEv9SEccX" -->
`PriceRecommendationCalculator` erzeugt sich intern selbst ein `CsvPriceRecommendationWriter()`. Warum ist das enger gekoppelt als eine Variante, die einen `RecommendationWriter` (Protocol) per Konstruktor entgegennimmt?::
Bei der internen Erzeugung ist `PriceRecommendationCalculator` fest an die konkrete Klasse `CsvPriceRecommendationWriter` gebunden — jede Änderung daran (anderes Format, andere Signatur) zwingt zur Anpassung von `PriceRecommendationCalculator`. Nimmt er stattdessen nur ein `RecommendationWriter`-Protocol per Konstruktor entgegen (von außen hereingereicht statt selbst erzeugt), kennt er nur die Schnittstelle — die konkrete Implementierung ist frei austauschbar, ohne ihn anzufassen. Das ist der Kern von Dependency Injection.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszMqKzNtypYkK4ZQrAua2r" -->
Eine `Utils`-Klasse bündelt `calculate_price_recommendation()`, `send_email()` und `parse_csv_line()`. Warum ist das niedrige Kohäsion, und was ist das Warnsignal daran?::
Die drei Methoden teilen sich keinen gemeinsamen Zustand und dienen keinem gemeinsamen Zweck — sie liegen nur aus Bequemlichkeit zusammen in einer Klasse. Warnsignal: Das ist typischerweise ein Vorbote einer SRP-Verletzung bzw. ein "Sammelbecken" für Code, der eigentlich in mehrere fokussierte Klassen aufgeteilt gehört.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7jCtmAmg2gZDQ5tQdG7KU" -->
Wie erkennt man informell (ohne Tooling), ob eine Klasse hohe oder niedrige Kohäsion hat?::
Prüfen, ob (fast) jede Methode (fast) alle Instanzattribute der Klasse nutzt — dann hohe Kohäsion. Gibt es stattdessen "Cluster" von Methoden und Attributen, die untereinander nie interagieren, ist das ein Zeichen niedriger Kohäsion und dass die Klasse eigentlich in zwei separate Klassen aufgeteilt werden sollte.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQdDyEdSTpc6EB86tGDwF2" -->
Warum führt hohe Kohäsion tendenziell fast automatisch auch zu SRP-Konformität, und was ist das gemeinsame übergeordnete Ziel von SRP, loser Kopplung und hoher Kohäsion?::
Eine Klasse mit hoher Kohäsion (alle Methoden gehören wirklich zusammen) hat typischerweise auch nur einen fachlichen Grund zur Änderung — also automatisch SRP erfüllt. Das gemeinsame übergeordnete Ziel aller drei Prinzipien (wie schon bei Information Hiding): Änderungen sollen sich auf einen möglichst kleinen, vorhersehbaren Bereich des Codes beschränken, statt sich durch die ganze Anwendung zu ziehen.
<!-- basicblock-end -->

# Open/Closed Prinzip

<!-- basicblock-start oid="Obsh6AXwxiWDqDl0EPPZzNe5" -->
Was besagt das Open/Closed Prinzip?::
Software-Bausteine (Klassen, Module, Funktionen) sollen offen für Erweiterung, aber geschlossen für Veränderung sein: Neues Verhalten soll sich hinzufügen lassen, ohne bestehenden, bereits getesteten Code zu ändern — das senkt das Risiko, dabei versehentlich etwas Funktionierendes zu beschädigen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseEms2dSP8VooOeB9rZs0F" -->
Warum verletzt eine wachsende `if isinstance(unit, TypA): ... elif isinstance(unit, TypB): ...`-Kette das Open/Closed Prinzip?::
Weil jede neue Produktart einen weiteren `elif`-Zweig in dieser EINEN zentralen Funktion erfordert — die Funktion muss also bei jeder Erweiterung angefasst werden (nicht "geschlossen für Veränderung") und riskiert dabei, bestehende, bereits funktionierende Zweige zu beschädigen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2EwwPSRyKIT01eSlv71PJ" -->
Welche zwei bereits behandelten Konzepte ermöglichen es überhaupt erst, Open/Closed in Python umzusetzen?::
Polymorphie (jede Subklasse implementiert die gemeinsame Methode selbst, z. B. `calculate_price_recommendation()`, neue Subklassen kommen einfach dazu) und lose Kopplung (aufrufender Code wie `calculate_all_price_recommendations()` kennt nur die gemeinsame Schnittstelle/Basisklasse, nicht eine feste Aufzählung konkreter Typen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssGLghrcWjLKXfNsIUDvCM" -->
Warum ist es unrealistisch bzw. unnötig, ein System für jede denkbare zukünftige Änderung von vornherein "offen" zu bauen?::
Das würde in unnötig abstrakten, übermäßig flexiblen Konstruktionen enden ("Speculative Generality"), die niemand tatsächlich braucht. Praktikabler: Open/Closed gezielt für absehbare Erweiterungsachsen anwenden (z. B. "es kommen sicher weitere Produktarten dazu"), statt für alles vorsorglich eine Extension-Schnittstelle zu bauen — sonst wird OOP zum Selbstzweck statt zum Werkzeug.
<!-- basicblock-end -->

# Clean Code

<!-- basicblock-start oid="ObsXdFbseFbvgyEFAprVJHn9" -->
Was ist Clean Code, und warum ist Lesbarkeit dabei so zentral?::
Code, der leicht zu lesen, verstehen und ändern ist — nicht nur "funktioniert irgendwie". Zentral, weil Code im Laufe seines Lebens sehr viel öfter gelesen wird als geschrieben (von Kolleg:innen, dem künftigen eigenen Ich, bei Reviews, beim Debuggen) — Code sollte daher primär für Menschen optimiert sein, nicht nur dafür, irgendwie vom Interpreter ausgeführt werden zu können.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrZ4crQuaDmz2qLPiw4Wx0" -->
Sollten Kommentare eher erklären, WAS der Code tut, oder eher WARUM er es so tut? Begründe.::
Eher WARUM: Das WAS sollte sich durch aussagekräftige Namen und klare Struktur bereits von selbst aus dem Code ergeben — ein Kommentar, der nur wiederholt, was in der nächsten Zeile ohnehin steht, ist überflüssig. Sinnvoll sind Kommentare für nicht offensichtliche Design-Entscheidungen oder Gründe, die im Code selbst nicht sichtbar sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHXrnfjWqcDAujnSQZYKM0" -->
Was sind "Magic Numbers", und wie behebt man sie im Sinne von Clean Code?::
Fest im Code verstreute Werte wie `1.19` oder `"BLACK_FRIDAY"`, deren Bedeutung nicht selbsterklärend ist. Behoben durch benannte Konstanten (z. B. `VAT_RATE = 0.19`), damit die Bedeutung sofort ersichtlich ist und der Wert bei Änderungen nur an einer Stelle angepasst werden muss.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0xRSYFUMSmWZ89ovF7rr8" -->
Nenne mindestens drei bereits behandelte Prinzipien aus dieser Notiz, die man als konkrete Bausteine von Clean Code verstehen kann.::
Z. B.: SRP & Separation of Concerns (kleine, fokussierte Bausteine statt God Classes), Kapselung & Information Hiding (klare, stabile öffentliche Schnittstellen), Loose Coupling & hohe Kohäsion (lokal und risikoarm änderbarer Code), Open/Closed (neues Verhalten hinzufügen, ohne bewährten Code zu riskieren). Clean Code ist kein isoliertes Regelwerk, sondern der Dachbegriff, dem diese Prinzipien dienen.
<!-- basicblock-end -->

# Dependency Injection

<!-- basicblock-start oid="ObsAjNe8njcJjMYrQtg898n7" -->
Was ist Dependency Injection (DI)?::
Eine Klasse fordert ihre Abhängigkeiten (andere Objekte, die sie zum Funktionieren braucht) nicht selbst an bzw. erzeugt sie nicht selbst, sondern bekommt sie von außen "injiziert" — typischerweise beim Erzeugen der Instanz per Konstruktor-Parameter, statt intern z. B. `writer = CsvPriceRecommendationWriter()` aufzurufen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxYGimh6QLyA060bKfT9yp" -->
Welche zwei konkreten Probleme löst Dependency Injection vor allem?::
(1) Testbarkeit: Eine echte Abhängigkeit (Dateisystem, Datenbank, externe API) lässt sich beim Testen durch ein einfaches Test-Double ersetzen, ohne die zu testende Klasse zu verändern. (2) Lose Kopplung: Die abhängige Klasse kennt nur die Schnittstelle, nicht die konkrete Implementierung — austauschbar, ohne sie anzufassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgDcHfiRhAMrhbf4aZXuIb" -->
Nenne die drei klassischen Formen von Dependency Injection und wann man eher zu Method Injection statt Constructor Injection greift.::
Constructor Injection (Abhängigkeit kommt beim Erzeugen der Instanz herein, Standardfall), Setter/Property Injection (Abhängigkeit wird erst nach der Konstruktion gesetzt) und Method Injection (Abhängigkeit wird nur für einen einzelnen Methodenaufruf übergeben, nicht als Attribut gespeichert). Method Injection eignet sich, wenn eine Abhängigkeit nur für einen einzelnen Aufruf gebraucht wird und nicht dauerhaft zur Instanz gehören soll.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslL5OoTGqLUa8i48KNv8PB" -->
Was ist der Unterschied zwischen dem Dependency Inversion Principle (DIP) und Dependency Injection (DI)?::
DIP ist ein Design-Prinzip (das "D" in SOLID): Module sollen von Abstraktionen abhängen, nicht von konkreten Implementierungen — das WAS/WARUM. DI ist die Technik/das Muster, mit dem man dieses Prinzip praktisch umsetzt (die Abstraktion wird injiziert, statt dass die abhängige Klasse sie sich selbst besorgt) — das WIE.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPJnlNSCULdmlP3XPQarH7" -->
Braucht man in Python zwingend ein DI-Framework/einen DI-Container wie in Java/Spring?::
Nein — dank dynamischer Typisierung und Duck Typing reicht in den meisten Fällen "Poor Man's Dependency Injection": ganz normale Konstruktor-Parameter ohne jedes Framework. DI-Container-Bibliotheken existieren, lohnen sich aber meist erst bei sehr großen Anwendungen mit vielen, tief verschachtelten Abhängigkeiten.
<!-- basicblock-end -->

# dunder Methoden & Dataclasses

<!-- basicblock-start oid="ObskouWZ3Ggta9Mk0y6eIUlV" -->
Welche drei dunder Methoden verleihen einer eigenen Klasse "Container-Verhalten" wie bei einer Liste?::
`__len__` (ermöglicht `len(obj)`), `__iter__` (ermöglicht `for x in obj`) und `__getitem__` (ermöglicht `obj[index]`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsjd3QG1nL4XOip66uNhl6I" -->
Was passiert mit `__hash__`, wenn du in einer Klasse `__eq__` selbst definierst, und wie behebst du das, falls du Instanzen in einem `set` verwenden willst?::
Python setzt `__hash__` automatisch auf `None` — die Instanz wird dadurch unhashable (kann nicht mehr als Dict-Key oder in ein `set` gelegt werden). Behoben durch explizite Definition von `__hash__` (z. B. `return hash(self.unit_id)`), passend zur `__eq__`-Logik.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs99UDZVWmLdNdJCshhlbyo" -->
Was generiert `@dataclass` automatisch aus type-annotierten Klassenattributen?::
`__init__` (setzt alle annotierten Felder als Instanzattribute), `__repr__` (lesbare Darstellung mit allen Feldern) und `__eq__` (vergleicht Instanzen anhand aller Felder) — eigene zusätzliche Methoden bleiben trotzdem ganz normal möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrI3lCvSS6SVkUQdA0RG4g" -->
Warum verlangt `@dataclass` bei mutable Defaults (z. B. `list`) explizit `field(default_factory=list)` statt einfach `= []`?::
Weil ein direktes `= []` als Default (genau wie bei mutable Klassenattributen oder mutable default arguments von Funktionen) zwischen ALLEN Instanzen geteilt würde. `dataclasses` verbietet das sogar explizit mit einer Fehlermeldung und verlangt eine `default_factory` — eine Funktion, die bei jeder neuen Instanz frisch aufgerufen wird und einen eigenen, unabhängigen Wert erzeugt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsalnQX0abYcpxxLyZUPcIV" -->
Was bewirken die Parameter `frozen=True` und `order=True` bei `@dataclass`?::
`frozen=True` macht Instanzen unveränderlich — jeder Zuweisungsversuch nach der Erzeugung wirft einen `FrozenInstanceError`. `order=True` generiert zusätzlich `__lt__`/`__le__`/`__gt__`/`__ge__` basierend auf der Feld-Reihenfolge (vergleichbar mit dem, was `functools.total_ordering` manuell ermöglicht, hier aber automatisch).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHjka745kk779XID4XPIgM" -->
Wann ist eine normale, von Hand geschriebene Klasse einer Dataclass vorzuziehen?::
Wenn die Klasse aufwendige Validierung/Logik bei der Erzeugung braucht (mehr als ein einfaches `__post_init__` abdeckt) oder der Zugriff auf einzelne Felder individuell kontrolliert werden soll (z. B. `@property` mit unterschiedlicher Setter-Validierung pro Feld) — Dataclasses eignen sich vor allem für Klassen, deren Hauptzweck das Halten von Daten ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWwSLdEjBflm2klcact6u1" -->
Was sind statische Methoden (`@staticmethod`) in Python, und wofür werden sie typischerweise genutzt?::
Eine Methode, die thematisch zu einer Klasse gehört (Namespace/Organisation), aber weder auf `self` (die Instanz) noch auf `cls` (die Klasse, siehe `@classmethod`) zugreift — im Grunde eine normale Funktion, die nur der Ordnung halber im Klassenkörper steht. Genutzt z. B. für reine Hilfsberechnungen ohne eigenen Zustand.
<!-- basicblock-end -->
