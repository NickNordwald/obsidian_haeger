---
deck: "Traineeship::Grundlagen der Software Entwicklung::Testing mit PyTest"
---

Karteikarten zu [[1.3 Testing mit PyTest - Vertiefung|1]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::Testing mit PyTest`. In Anki werden die Karten außerdem automatisch mit dem Vault-Namen und `ObsidianAnkiSync` getaggt (kein eigener Obsidian-Tag nötig). Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

# Fortschritt

- [x] Mike Cohns Testpyramide
- [x] Modultests
- [x] Mocks / Simulation von Verhalten echter Objekte
- [x] Testabdeckung / Code-Coverage

---

# Mike Cohns Testpyramide

<!-- basicblock-start oid="ObsNciYCmA3u7Q8ju4clQeFL" -->
Von wem stammt die Testpyramide, und was ist ihre Kernaussage in einem Satz?::
Von Mike Cohn (Buch *Succeeding with Agile*, 2009). Kernaussage: Man sollte viele kleine, schnelle, isolierte Tests unten haben (Unit) und nur wenige große, langsame, „echte" Tests oben (E2E) — das Gewicht der Testsuite liegt in der breiten Basis.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjvODMfZFXGZqzlo09Wl7A" -->
Nenne die drei klassischen Ebenen der Testpyramide von unten nach oben und je ein Beispiel.::
1. Unit-/Modultests (Basis): eine isolierte Einheit ohne DB/Netz/Dateisystem, z. B. „berechnet `available_units()` korrekt?".<br>2. Integrations-/Service-Tests (Mitte): Zusammenspiel mehrerer Einheiten oder mit echter Infrastruktur, z. B. „schreibt & liest das Repository korrekt aus PostgreSQL?".<br>3. UI-/End-to-End-Tests (Spitze): das ganze System aus Nutzersicht, z. B. „Nutzer bucht Produkt und sieht korrekten Restbestand".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuZzuFhD4g9TfZmyG52cOz" -->
Welche Eigenschaften verändern sich, je höher man in der Testpyramide klettert (Unit → E2E)?::
Nach oben nehmen zu: Umfang/Scope, Ausführungszeit, Kosten (Schreiben + Warten), Realitätsnähe/Aussagekraft, Flakiness-Gefahr.<br>Nach oben nehmen ab: empfohlene Anzahl der Tests, Präzision der Fehlerlokalisierung, Stabilität/Determinismus.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAxn04RKNRECoztvcL8w2o" -->
Warum ist die Basis der Testpyramide (Unit-Tests) so breit? Nenne die drei Kernargumente.::
1. Feedback-Geschwindigkeit: Tausende Unit-Tests laufen in Sekunden, also nach jeder Änderung ausführbar → enger Feedback-Loop.<br>2. Präzise Fehlerlokalisierung: ein roter Unit-Test zeigt genau die kaputte Funktion (statt „irgendwo in der Kette").<br>3. Bug-Kosten steigen mit der Zeit: ein früh (im Unit-Test) gefangener Fehler kostet Minuten, derselbe Fehler in Produktion ein Vielfaches.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZsn7h3ZCKiXhwyYyv6wPJ" -->
Wozu braucht man die Spitze der Pyramide (E2E-Tests) überhaupt, wenn sie so teuer ist?::
Nur E2E-Tests beweisen, dass die verdrahteten Teile im Ganzen zusammen funktionieren — also das, was der Nutzer wirklich erlebt. Sie sind aber ein „teures Gewürz, kein Grundnahrungsmittel": Man deckt damit nur die wichtigsten durchgehenden Nutzerpfade (happy paths) ab, nicht jeden Sonderfall. Sonderfälle gehören nach unten (Unit).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsIn2yNkJCOWPsopMrAVWn7" -->
Was ist das Anti-Muster „Eistüte" (ice cream cone) und warum ist es problematisch?::
Die umgekehrte Pyramide: viele langsame UI-/manuelle Tests oben, kaum Unit-Tests unten. Problematisch, weil die Suite ewig läuft und flaky ist, Fehler schwer zu lokalisieren sind und sich niemand mehr traut, Code umzubauen. Entsteht oft, wenn Testen erst nachträglich über die fertige Oberfläche „draufgesetzt" wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaGCKodlk6uebh3rez0eoM" -->
Was ist das Anti-Muster „Sanduhr" (hourglass) beim Testen?::
Viele Unit-Tests und viele E2E-Tests, aber die Integrationsschicht in der Mitte fehlt. Folge: Einzelbausteine und das Gesamtsystem funktionieren zwar, aber genau die Übergänge/Verdrahtung zwischen den Modulen sind ungetestet und brechen unbemerkt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsE4cIDfBaRsFRN53Tvz8f0" -->
Was bedeutet ein „flaky" Test, und warum nimmt die Flakiness-Gefahr nach oben in der Pyramide zu?::
Ein flaky Test ist bei unverändertem Code mal grün, mal rot — z. B. durch Timing, Netzwerklatenz, Testreihenfolge oder Zufallsdaten. Gefahr: Man gewöhnt sich an rote Tests und übersieht echte Fehler. Nach oben nimmt sie zu, weil höhere Tests mehr langsame/unzuverlässige externe Faktoren (Netz, Umgebung, UI-Timing) anfassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLFR3rm2d39fPYAiFpdQI2" -->
Warum ist der Begriff „Unit" unscharf? Welche zwei Denkschulen gibt es?::
Weil nicht klar definiert ist, wie viel eine „Einheit" umfasst. Solitary / „London School": eine Unit = eine Klasse/Funktion, alle Abhängigkeiten werden durch Test-Doubles (Mocks/Stubs) ersetzt (maximale Isolation). Sociable / „Detroit/Classicist School": eine Unit = ein Verhalten, mehrere echte schnelle Objekte dürfen zusammenarbeiten, gemockt wird nur an den Rändern (langsame externe Systeme).
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsMn6Jjy9JC0HZPEBnJ3IWr" -->
Die Testpyramide empfiehlt ==viele== Unit-Tests an der ==Basis== (schnell, billig, isoliert) und ==wenige== ==E2E==-Tests an der Spitze (langsam, teuer, realitätsnah).
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsJu47jZt0WG0fiJjB7qs7q" -->
Kent C. Dodds' Weiterentwicklung der Pyramide heißt ==Testing Trophy== und betont mit dem Leitsatz „Write tests. Not too many. ==Mostly integration==." stärker die Integrationsschicht.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObsGlkaEe45W0VmkgO3RrW5x" -->
Was ist ein Modultest / Unit-Test (Definition)?::
Ein Test, der eine einzelne, isolierte „Einheit" des Codes (meist Funktion, Methode oder Klasse) auf ein erwartetes Verhalten prüft — ohne langsame externe Abhängigkeiten wie Datenbank, Netzwerk oder Dateisystem. Er bildet die breite Basis der Testpyramide.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWgInGyYcll7ifbItB4Qp8" -->
Was ist der zentrale Zielkonflikt, den die Testpyramide auflöst?::
Unit-Tests lokalisieren Fehler präzise und laufen schnell & stabil, sagen aber wenig über das Gesamtsystem. E2E-Tests sind maximal aussagekräftig (nah am echten Nutzererlebnis), aber langsam, teuer und flaky. Die Pyramide löst das, indem sie das Gewicht nach unten verlagert: viele billige Unit-Tests, wenige teure E2E-Tests.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMjgXdL0iJIFZP8yELJDrA" -->
Was testet die mittlere Ebene (Integrations-/Service-Tests) und warum ist sie wichtig?::
Das Zusammenspiel mehrerer Einheiten oder das Zusammenspiel mit echter Infrastruktur (echte DB, Dateisystem, API) — also die Verdrahtung/Übergänge zwischen Modulen. Wichtig, weil genau diese Übergänge das sind, was Unit-Tests (durch Isolation/Mocking) per Definition übersehen; fehlt sie, entsteht das „Sanduhr"-Anti-Muster.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdB3SOuE530RG1UgbNG5ZW" -->
Warum sind E2E-Tests ein „teures Gewürz, kein Grundnahrungsmittel"?::
Weil sie langsam, teuer und flaky sind. Man deckt mit ihnen nur die wichtigsten durchgehenden Nutzerpfade (happy paths) ab, nicht jeden Sonderfall. Sonderfälle und Randfälle gehören nach unten in schnelle, präzise Unit-Tests.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0DxQXs5a1hQms2L4PSYrD" -->
Wo verläuft praktisch die Grenze zwischen Unit- und Integrationstest?::
Nicht als harte Linie, sondern als Kontinuum — praktisch dort, wo ein Test anfängt, langsame oder unzuverlässige externe Infrastruktur (DB, Netzwerk, Dateisystem) anzufassen. Solange nur schnelle In-Memory-Objekte beteiligt sind, gilt es als Unit-Test.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSbRAuooaffvqAM3niB8ux" -->
Warum steigen die Kosten eines Bugs, je später er entdeckt wird?::
Ein Bug, den schon ein Unit-Test beim Schreiben fängt, kostet Minuten. Derselbe Bug, der erst in Produktion beim Kunden auffällt, kostet ein Vielfaches (Debugging, Hotfix, evtl. Datenschaden, Vertrauensverlust). Deshalb fangen viele billige Tests an der Basis die meisten Fehler an der günstigsten Stelle ab.
<!-- basicblock-end -->

<!-- clozeblock-start oid="ObsCFfr8iXzZLCEa2A2hkLmz" -->
Je höher man in der Testpyramide steigt, desto ==langsamer==, ==teurer== und ==flakiger== werden die Tests, und desto ==unpräziser== wird die Fehlerlokalisierung — dafür steigt die ==Realitätsnähe== / Aussagekraft.
<!-- clozeblock-end -->

---

# Modultests

<!-- basicblock-start oid="ObsIiZeXZLiP1EP68D2oGh4f" -->
Wozu genau schreibt man Modultests? Nenne Zwecke über „Bugs finden" hinaus.::
1. Korrektheit für typische Fälle, Randfälle und Fehlerfälle nachweisen.<br>2. Regressionsschutz (oft der wichtigste Grund): sofortige Warnung, wenn eine Änderung etwas vorher Funktionierendes bricht — ermöglicht furchtloses Refactoring.<br>3. Design-Feedback: schwer testbarer Code ist meist schlecht entworfen (zu viel Kopplung/Verantwortung).<br>4. Ausführbare Dokumentation: beschreibt das Sollverhalten und veraltet nicht unbemerkt (wird rot).<br>5. Schnelleres Debugging: ein roter Unit-Test zeigt punktgenau die kaputte Einheit.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsdl2EJozen7uuMm9q5QLZB" -->
Was bedeutet die Test-Struktur Arrange–Act–Assert (AAA)?::
Arrange = Ausgangszustand herstellen (Objekte/Eingaben vorbereiten). Act = die zu testende Aktion genau einmal ausführen. Assert = das Ergebnis gegen die Erwartung prüfen. Faustregel: ein logisches Verhalten pro Test, damit bei Rot klar ist, was kaputt ist. (Entspricht Given–When–Then aus dem BDD-Umfeld.)
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4tcrsyVynrAX9EVbGgZv6" -->
Wofür steht das Akronym FIRST für gute Unit-Tests?::
Fast (Millisekunden, ständig ausführbar), Isolated/Independent (unabhängig voneinander und von externer Infrastruktur, reihenfolgeunabhängig), Repeatable (deterministisch, keine Abhängigkeit von Zeit/Zufall/Netz), Self-validating (entscheidet selbst grün/rot per assert), Timely (zeitnah zum Produktivcode geschrieben).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs88Z5SMZdmrgi0lH1fMwrl" -->
Woran erkennt pytest Tests automatisch (Discovery-Konventionen)?::
Dateien `test_*.py` (oder `*_test.py`), Funktionen mit Präfix `test_`, optional in Klassen `Test*` (ohne `__init__`). Tests liegen üblicherweise in einem eigenen `tests/`-Ordner. Assertions über das eingebaute `assert` — dank assertion rewriting zeigt pytest im Fehlerfall die tatsächlichen Werte an.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswErvUXVlScXuqr8INENKS" -->
Wie testet man mit pytest, dass ein Aufruf eine bestimmte Exception wirft?::
Mit dem Kontextmanager `pytest.raises`:<br>`with pytest.raises(ValueError):`<br>&nbsp;&nbsp;&nbsp;&nbsp;`unit.book(-1)`<br>Optional die Meldung prüfen mit `match="…"` (Regex): `pytest.raises(ValueError, match="darf nicht negativ")`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJGkcVTTJfIZfTfOYicODG" -->
Welche Kategorien von Testfällen sollte man abdecken (nicht nur den Happy Path)?::
Happy Path (typischer gültiger Fall), Randfälle/Grenzwerte (0, 1, Maximum, leer, ausgebucht — „an der Grenze und knapp daneben"), Fehlerfälle (ungültige Eingaben, die eine Exception auslösen sollen), Äquivalenzklassen (Eingaben mit gleichem Verhalten zusammenfassen und je einen Vertreter testen). Äquivalenzklassen + Grenzwertanalyse finden mit wenigen Tests viele Fehler.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnXnB12qwYrh4vhEezHHfX" -->
Wie sollte man Tests benennen? Welche Konventionen gibt es?::
Der Name beschreibt das erwartete Verhalten, nicht nur die Methode — bei Rot soll ohne Code-Blick klar sein, was nicht stimmt. Muster: `test_<Einheit>_<Bedingung>_<erwartetes Ergebnis>` (z. B. `test_book_mehr_als_verfuegbar_wirft_error`), „should"-Stil (`test_should_reject_negative_price`), Given-When-Then. Beschreibend statt nummeriert (`test_1` ist wertlos), lieber zu lang als zu kurz.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsxv5tsjxzuo5ng2oe8eWy2" -->
Was ist eine pytest-Fixture, wie bekommt ein Test sie, und wozu dient `conftest.py`?::
Eine Fixture ist wiederverwendbares Setup (mit `@pytest.fixture` dekorierte Funktion), optional mit Teardown. Ein Test bekommt sie, indem er ihren Namen als Parameter deklariert (Dependency Injection). Mit `yield` ist alles davor Setup, alles danach Teardown. Fixtures in `conftest.py` findet pytest automatisch und teilt sie über mehrere Testdateien hinweg (ohne Import).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLUwojyuq9JOgejcL9h9VV" -->
Wie funktionieren parametrisierte Tests in pytest, und warum sind sie besser als eine Schleife im Test?::
Mit `@pytest.mark.parametrize("a, b, erwartet", [(…), (…)])` erzeugt pytest aus jeder Wertetupel-Zeile einen eigenständigen Testfall. Vorteil gegenüber einer Schleife: Schlägt ein Fall fehl, laufen die anderen trotzdem, und man sieht genau, welche Eingabe bricht (eine Schleife wäre nur ein Testfall und bräche beim ersten Fehler ab). Möglichkeiten: `ids=[…]` für Namen, `pytest.param(..., marks=…)`, mehrere Dekoratoren = Kreuzprodukt, `fixture(params=…)`, property-based mit Hypothesis.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVj8ziujj3ZJRofBLca4Zx" -->
Was ist TDD und wie lautet sein Rhythmus?::
Test-Driven Development: den Test zuerst schreiben (rot), dann minimalen Code für Grün, dann aufräumen. Rhythmus: Red – Green – Refactor. Passt zum „Timely" aus FIRST: Test und Code entstehen zeitlich eng beieinander.
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obsdbn2UwL0ZdrLWPDAS1SjS" -->
Ein Modultest folgt der Struktur ==Arrange== (Zustand herstellen) – ==Act== (Aktion ausführen) – ==Assert== (Ergebnis prüfen) und sollte genau ==ein== logisches Verhalten prüfen.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsSeKlHWs9iTSFNv79IYjhc" -->
Das „I" in FIRST steht für ==Isolated== — Tests dürfen sich nicht gegenseitig beeinflussen und nicht von externer Infrastruktur abhängen; genau deshalb braucht man ==Mocking==, sobald eine Einheit langsame Abhängigkeiten (DB, Netzwerk) hat.
<!-- clozeblock-end -->

## Syntax / Dekoratoren (Modultests)

<!-- clozeblock-start oid="ObsQNvWEOEPoVLI4LmX57B82" -->
Wiederverwendbares Setup markiert man mit dem Dekorator ==@pytest.fixture==; ein Test erhält es, indem er den ==Fixture-Namen== als Parameter deklariert.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsEh08bw1P1O70ormCU76Am" -->
In einer Fixture trennt das Schlüsselwort ==yield== Setup (davor) von Teardown (danach); über mehrere Testdateien geteilte Fixtures legt man in die Datei ==conftest.py==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsLAT4eRZgMOCB4uLMSm4dy" -->
Dieselbe Testlogik mit mehreren Eingaben prüft man mit ==@pytest.mark.parametrize==("bookable, booked, erwartet", [(5, 2, 3), (5, 5, 0)]); sprechende Namen je Fall gibt der Parameter ==ids== .
<!-- clozeblock-end -->

<!-- clozeblock-start oid="Obsrbjp5BTsokvazptVu5rXp" -->
Dass ein Aufruf eine Exception wirft, prüft man mit `with ==pytest.raises==(ValueError, ==match==="darf nicht negativ"):`.
<!-- clozeblock-end -->

<!-- basicblock-start oid="ObscIw7791THQOSE1BFXDPLb" -->
Was ist „assertion rewriting" in pytest, und welchen Vorteil bringt es?::
pytest schreibt `assert`-Ausdrücke zur Laufzeit um, sodass es im Fehlerfall die tatsächlichen Werte anzeigt (z. B. `assert 4 == 3` statt nur „AssertionError"). Vorteil: Man braucht keine speziellen Vergleichsmethoden wie `assertEqual` (unittest) — ein einfaches `assert` genügt fast immer und liefert trotzdem aussagekräftige Fehlermeldungen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDAOZC4nYW88kVAywh636e" -->
Warum sollte ein Test genau ein logisches Verhalten prüfen?::
Damit bei einem roten Test sofort klar ist, WELCHES Verhalten kaputt ist. Prüft ein Test drei verschiedene Dinge, weiß man beim Fehlschlag nicht, welches. (Nicht wörtlich „ein assert pro Test" — mehrere Asserts zum selben Sachverhalt sind ok.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscCNHgyMrryzhEjlZOKQme" -->
Was ist Grenzwertanalyse (boundary value analysis) und warum ist sie wichtig?::
Das gezielte Testen an den Rändern des Wertebereichs (0, 1, Maximum, leer, ausgebucht) nach der Faustregel „an der Grenze und knapp daneben". Wichtig, weil an genau diesen Grenzen die meisten Fehler passieren (Off-by-one, <= vs. <). Kombiniert mit Äquivalenzklassen findet man mit wenigen Tests viele Fehler.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs5QNVUl1a5Mo6zuVPYhrlE" -->
Was ist Äquivalenzklassenbildung (equivalence partitioning) beim Testen?::
Eingaben, die sich gleich verhalten, zu Klassen zusammenfassen und je einen Vertreter testen — statt 1000 positive Zahlen genügt eine positive Zahl. Hält die Zahl der Testfälle sinnvoll klein, ohne Abdeckung zu verlieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAjYaqa5nos22TmX9E2RqO" -->
Was ist Given-When-Then, und wie verhält es sich zu Arrange-Act-Assert?::
Given-When-Then ist die Formulierung aus dem BDD-Umfeld (Behaviour-Driven Development): Given = Vorbedingung, When = Aktion, Then = erwartetes Ergebnis. Es ist inhaltlich dasselbe Muster wie Arrange-Act-Assert, nur mit anderen Worten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLDR5gHduuriBzs9noug6U" -->
Nenne nützliche pytest-CLI-Optionen und ihre Bedeutung.::
`pytest` (alle Tests entdecken & laufen), `-v` (verbose, jeder Test einzeln mit Name), `-q` (quiet, knappe Ausgabe), `-k "ausdruck"` (nur Tests, deren Name den Ausdruck enthält), `-x` (beim ersten Fehler abbrechen), `--lf` (last failed: nur die zuletzt fehlgeschlagenen erneut).
<!-- basicblock-end -->

<!-- clozeblock-start oid="Obs9hFOoUWTcUlzb7X0QvGh9" -->
pytest führt nur Tests aus, deren Datei dem Muster ==test_*.py== und deren Funktion dem Präfix ==test_== entspricht (optional gruppiert in Klassen ==Test*== ohne `__init__`).
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsLgbRL8srZReFhlBEWryRt" -->
Mehrere `@pytest.mark.parametrize`-Dekoratoren über derselben Funktion ergeben das ==Kreuzprodukt== aller Wertelisten; automatisch generierte Eingaben (property-based testing) liefert die Bibliothek ==Hypothesis==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="Obsig62xRKcOxEd1dtHbw71l" -->
Nützliche pytest-Flags: ==-v== für verbose, ==-k== zum Filtern nach Testnamen, ==-x== zum Abbruch beim ersten Fehler und ==--lf== (last failed), um nur die zuletzt fehlgeschlagenen erneut zu laufen.
<!-- clozeblock-end -->

---

# Mocks / Simulation von Verhalten echter Objekte

<!-- basicblock-start oid="Obse0uFLVXyeLUww1s3qpzul" -->
Welches Grundproblem lösen Mocks / Test-Doubles?::
Sobald die getestete Einheit eine „echte" Abhängigkeit anfasst (DB, Netzwerk, Dateisystem, Uhr, Zufall), werden Unit-Tests langsam (F), nicht-deterministisch (R), nicht isoliert (I) und bekommen unerwünschte Nebenwirkungen (echte E-Mails/Schreibzugriffe). Man ersetzt die echte Abhängigkeit durch ein kontrolliertes Double, dessen Verhalten man vollständig steuert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTQUguwcGP4RA3xvEWGyAi" -->
Welchen Vorteil bietet das Mocken (Simulation von Verhalten)?::
1. Isolation: nur die eigene Einheit wird getestet (roter Test = eigener Code kaputt).<br>2. Geschwindigkeit: kein DB-/Netz-Roundtrip.<br>3. Determinismus: man bestimmt exakt die Rückgaben.<br>4. Fehlerfälle simulieren: schwer herstellbare Zustände wie „DB wirft Timeout" per side_effect erzwingen (oft der eigentliche Gewinn).<br>5. Nebenwirkungen vermeiden: keine echten Mails/Schreibzugriffe/API-Kosten.<br>6. Unabhängige Entwicklung: gegen noch nicht existierende Komponenten testen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3WLVCdBHZCIzxWJJiao7A" -->
Nenne die fünf Arten von Test-Doubles (nach Meszaros) mit Kurzzweck.::
1. Dummy: nur zum Füllen einer Signatur, wird nie benutzt.<br>2. Stub: liefert vordefinierte Antworten (canned answers) auf Aufrufe.<br>3. Spy: wie Stub, zeichnet zusätzlich auf, wie er aufgerufen wurde.<br>4. Mock: kennt erwartete Aufrufe im Voraus und verifiziert, dass sie so passieren.<br>5. Fake: echte, aber vereinfachte Implementierung (z. B. In-Memory-DB statt PostgreSQL).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqvN0WrA5whwanuSi90xgG" -->
Was ist der Unterschied zwischen einem Stub und einem Mock?::
Stub = liefert nur Eingaben, damit der Code läuft; man prüft danach das Ergebnis/den Zustand (State Verification), der Stub selbst wird nicht geprüft. Mock = hat Erwartungen an die Interaktion; man prüft, OB und WIE der Code ihn aufgerufen hat (Behavior Verification: richtige Methode, Argumente, Anzahl). Faustregel: Stub für Abfragen (queries), Mock für Kommandos/Seiteneffekte (commands).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHrzwSDToeNjihoJHWie7j" -->
Was bedeuten State Verification und Behavior Verification?::
State Verification: man prüft nach der Aktion das Ergebnis bzw. den Zustand des Objekts (passt zum Stub). Behavior Verification: man prüft, welche Aufrufe an einer Abhängigkeit stattfanden (mit welchen Argumenten, wie oft) — passt zum Mock. Interaktionen sollte man nur für echte Seiteneffekte verifizieren, nicht für reine Abfragen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2Jxc5KzcQuRq3CAhnUNXK" -->
Wann nimmt man einen Stub, wann einen Mock? (Query vs. Command)::
Stub für Abfragen (queries): Ob eine Berechnung stimmt, prüft man am Ergebnis — der Stub liefert nur die Eingabedaten. Mock für Kommandos (commands): Ob ein Seiteneffekt ausgelöst wurde (speichern, senden, löschen), prüft man an der Interaktion. So testet man Verhalten statt Implementierung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOhXyidmixjlODiIcubkJY" -->
Sind Stub und Mock in Pythons `unittest.mock` verschiedene Objekte?::
Nein — technisch ist es dasselbe Objekt (`Mock`/`MagicMock`). Ob es als Stub oder Mock wirkt, entscheidet die Nutzung: Nur `return_value` setzen und danach das Ergebnis prüfen = Stub-Nutzung; hinterher mit `assert_called_with(...)` die Interaktion prüfen = Mock-Nutzung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWHv07egnbOZFPZDD9pTUo" -->
Wie provoziert man mit einem Mock einen Fehlerfall (z. B. DB-Timeout)?::
Über `side_effect`: Weist man dem Mock eine Exception zu (`repo.save.side_effect = ConnectionError("...")`), wird sie beim Aufruf geworfen. So testet man den Fehler-Umgang des eigenen Codes, ohne die echte Fehlerbedingung herstellen zu müssen. (side_effect kann auch eine Funktion oder eine Liste von Rückgaben für Folgeaufrufe sein.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSCnXdVOM4HkyeWAoC2wIw" -->
Was besagt die Regel „patch where it's used, not where it's defined"?::
Man patcht einen Namen dort, wo er nachgeschlagen/benutzt wird, nicht wo er definiert ist. Ruft `myapp/service.py` nach `import requests` die Funktion `requests.get` auf, patcht man `"myapp.service.requests.get"`, nicht `"requests.get"`. Grund: `from x import y` bindet `y` als neuen Namen im importierenden Modul; nur diese Bindung sieht der Code. Diese Regel erklärt die meisten „mein Mock greift nicht"-Fälle.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs09IrTxskDb0mmDlodX7D6" -->
Warum macht Dependency Injection das Mocken einfacher?::
Bekommt eine Klasse ihre Abhängigkeiten übergeben (z. B. per Konstruktor), statt sie sich selbst zu beschaffen, reicht man im Test einfach ein Double hinein — ganz ohne `patch` und Modulpfad-Fummelei. Schwer zu mockende Stellen sind ein Designsignal: oft lohnt es, die Abhängigkeit injizierbar zu machen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4wypMkBxjTqxsQRYdm9MF" -->
Was ist der Unterschied zwischen `monkeypatch` und `mocker` in pytest?::
`monkeypatch` ist eine eingebaute pytest-Fixture, die Attribute/Umgebungsvariablen/dict-Einträge temporär setzt und nach dem Test automatisch zurücksetzt (ideal für kleine gezielte Eingriffe). `mocker` stammt aus dem Plugin `pytest-mock` und ist eine komfortable Hülle um `unittest.mock`, die das manuelle Starten/Stoppen von Patches spart (kein with/Dekorator, automatisches Aufräumen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRZOmojRFXBPKYasDQqA3f" -->
Was bringt `autospec` (bzw. `create_autospec`) und welches Problem verhindert es?::
Ein rohes `Mock()` akzeptiert JEDEN Aufruf — auch Tippfehler oder Methoden, die es am echten Objekt gar nicht gibt (Test bleibt grün, obwohl der echte Code bräche). `autospec` leitet das Double aus der echten Signatur ab: falsche Methodennamen oder falsche Argumentzahlen fliegen sofort auf. Faustregel: in ernsthaftem Code `autospec=True` / `create_autospec` nutzen, damit Tests nicht „grün lügen".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqsF4DsOLebWRVXIgBHIAG" -->
Was ist ein „Fake" und wann ist er einem Mock vorzuziehen?::
Ein Fake ist eine echte, aber vereinfachte Arbeits-Implementierung (z. B. In-Memory-/SQLite-DB statt PostgreSQL). Vorteilhaft für Integrationstests: Er verhält sich realistischer als ein Mock und fängt Fehler, die reine Mocks (die nur vorgegebene Antworten liefern) per Definition übersehen — ohne die volle Langsamkeit/Abhängigkeit des echten Systems.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiOyZ7io9yEoRBd1u70Gbu" -->
Wann sollte man NICHT (oder weniger) mocken? (Over-Mocking)::
1. Reine Logik/Berechnungen/Wert-Objekte nie mocken — mit echten Objekten testen; Mocks nur für langsame/unzuverlässige Ränder (DB, Netz, Zeit, Zufall, Datei).<br>2. Nicht jeden internen Aufruf verifizieren → sonst brüchige (brittle) Tests, die bei harmlosem Refactoring rot werden (testet das Wie statt das Was).<br>3. Über-Isolation verschleiert Integrationsfehler → ein paar Tests mit echter DB/Fake ergänzen.<br>Faustregel: so viel wie nötig, so wenig wie möglich.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsn7IM2e77wO4AqmodLNPzo" -->
Was sind „brittle" (brüchige) Tests und wodurch entstehen sie beim Mocken?::
Tests, die schon bei einem harmlosen Refactoring rot werden, obwohl sich das Verhalten nicht geändert hat. Sie entstehen durch Over-Mocking: Wenn man jeden internen Methodenaufruf per Mock verifiziert, testet man die Implementierung (das Wie) statt des Verhaltens (das Was). Abhilfe: Interaktionen nur für echte Seiteneffekte prüfen.
<!-- basicblock-end -->

## Syntax / API (Mocks)

<!-- clozeblock-start oid="Obs2DXWCyli7jmQ641C7FRxV" -->
Das Bordmittel zum Mocken in Python ist das Modul ==unittest.mock==; die Universal-Double-Klassen heißen ==Mock== und ==MagicMock== (letztere unterstützt zusätzlich magische Methoden wie `__len__`).
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsqsGLSQuDZBMAQFvo2uvVc" -->
Einen festen Rückgabewert setzt man mit ==return_value== (`repo.get.return_value = 2`); eine Exception oder Reaktion mit ==side_effect==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="Obspd2T83TrTJAT7xqyZ5ueo" -->
Dass ein Mock genau einmal mit bestimmten Argumenten aufgerufen wurde, prüft man mit ==assert_called_once_with==(...); die Zahl der Aufrufe steht in ==call_count==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsgCMrgOKTQT3LfeEKYdEGT" -->
Eine Abhängigkeit, die der Code selbst erzeugt, ersetzt man zur Testzeit mit ==patch==("myapp.service.requests.get") — als Dekorator, Kontextmanager oder über die pytest-mock-Fixture ==mocker==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsFCc2wxGcawJivAFa9nakc" -->
Ein Double, das nur die echten Methoden/Signaturen des Originals zulässt, erzeugt man mit ==create_autospec==(Repository) bzw. dem Argument ==autospec===True.
<!-- clozeblock-end -->

---

# Testabdeckung / Code-Coverage

<!-- basicblock-start oid="ObsBjne2XpPlReZ8wxXRHzpi" -->
Was ist Code-Coverage (Testabdeckung) und wie wird sie technisch gemessen?::
Eine Kennzahl (in %), wie viel des Produktivcodes beim Ausführen der Testsuite tatsächlich durchlaufen wird — sie beantwortet „welche Teile hat noch nie ein Test angefasst?". Technisch über Instrumentierung: Das Werkzeug klinkt sich in die Ausführung ein (in Python via `sys.settrace`), protokolliert jede ausgeführte Zeile/Verzweigung und vergleicht sie mit dem Gesamtcode.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGtikBjgQ4pyTw8eQOFCAK" -->
Mit welchen Werkzeugen misst man Coverage in der pytest-Welt?::
`coverage.py` ist die zugrunde liegende Bibliothek; `pytest-cov` ist das pytest-Plugin, das sie in den Testlauf integriert. Aufruf z. B. `pytest --cov=myapp --cov-report=term-missing` (zeigt die nicht abgedeckten Zeilennummern) oder `--cov-report=html` (interaktiver HTML-Report).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJCAoRA4zX8cIhwTiYx3XZ" -->
Was ist der Unterschied zwischen Zeilen-, Zweig- und Pfadabdeckung?::
Zeilen-/Statement-Coverage: Anteil der Zeilen, die ≥ 1× ausgeführt wurden (schwächstes Maß). Zweig-/Branch-Coverage: ob jeder Verzweigungsausgang (if True UND if False) genommen wurde (stärker, empfohlenes Ziel). Pfad-/Path-Coverage: ob jede Kombination von Verzweigungen (jeder ganze Weg) durchlaufen wurde (strengstes Maß, kombinatorisch explodierend → praktisch unerreichbar).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDv4QcO9sbzQ49Pgeld5b3" -->
Warum ist 100 % Zeilenabdeckung trügerisch? Beispiel mit einem `if`.::
Ein `if ist_stammkunde: preis *= 0.9` wird von einem einzigen Test `rabatt(100, True)` komplett durchlaufen → 100 % Zeilenabdeckung. Der Fall `False` (Bedingung nicht erfüllt, Rabatt übersprungen) wurde aber nie getestet. Zeilenabdeckung übersieht diesen Verhaltenspfad — Zweigabdeckung würde ihn mit einem zweiten Test `rabatt(100, False)` erzwingen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObskL3d5RInpmfPG4pq0B9b7" -->
Warum ist vollständige Pfadabdeckung in der Praxis kaum erreichbar?::
Weil die Zahl der Pfade kombinatorisch explodiert: bei n unabhängigen if-Anweisungen bis zu 2^n Pfade, Schleifen machen es formal unendlich. Schon 10 Verzweigungen = 1024 Pfade. Deshalb ist Pfadabdeckung eher ein theoretisches Ideal als ein praktisches Ziel.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsG5rXBrjJI7PjrE56r9awt" -->
Was ist Condition Coverage (Bedingungsabdeckung)?::
Ein noch feineres Maß als Branch Coverage: Bei zusammengesetzten Bedingungen wie `if a and b` prüft sie, ob jede Teilbedingung (a, b) einzeln schon True und False war. In der Praxis selten explizit gefordert — Branch Coverage ist das übliche Ziel.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnV1QXFttFaLg2DZ3fc2EW" -->
Wie viel Testabdeckung macht Sinn — und warum ist „100 %" die falsche Zielsetzung?::
Coverage ist Diagnosewerkzeug, nicht Ziel. Sie ist notwendig, aber nicht hinreichend: Sie misst Ausführung, nicht Prüfqualität (ein Test ohne assert hat Coverage, prüft aber nichts). Goodhart's Law: Wird 100 % vorgeschrieben, entstehen wertlose Alibi-Tests. Abnehmender Grenznutzen: die letzten % sind am teuersten. Richtwert ~70–85 %, kritische Logik ~100 % Branch, Trivialcode niedriger; wichtiger als der Absolutwert ist, dass die Coverage nicht sinkt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsOPJXMnijKWlZ6uJHgCGfJ" -->
Warum ist Coverage „notwendig, aber nicht hinreichend"?::
Notwendig: ungetesteter Code (0 % Coverage) ist definitiv ungeprüft. Nicht hinreichend: Coverage misst nur, ob Code AUSGEFÜHRT wurde, nicht ob er sinnvoll GEPRÜFT wurde. Ein Test ohne `assert` erhöht die Coverage, bliebe aber selbst dann grün, wenn die Funktion Unsinn zurückgibt. Coverage sagt, was NICHT getestet ist — nicht, ob ein Test gut ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsg5GWA7qTJU4yNWbvSBqcS" -->
Was besagt Goodhart's Law im Kontext von Code-Coverage?::
„When a measure becomes a target, it ceases to be a good measure." Wird eine feste Coverage-Quote (z. B. 100 %) vorgeschrieben, schreiben Entwickler assertion-lose Alibi-Tests nur, um die Zahl zu erreichen — die Metrik wird erfüllt, das eigentliche Ziel (Vertrauen in den Code) verfehlt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvdHZdGGPFwMc3QOn0sPu5" -->
Wozu dient der `term-missing`- bzw. HTML-Coverage-Report konkret?::
Er zeigt nicht nur die Prozentzahl, sondern die konkrete Liste der NICHT abgedeckten Zeilen/Zweige. Das ist der eigentlich wertvolle Teil: Er deckt blinde Flecken auf (ungetestete Invarianten, Fehlerpfade), die man dann gezielt schließen kann.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKGyoSghQglFdmNEwsDLQN" -->
Was ist Mutation Testing und welche Schwäche der Coverage-Zahl adressiert es?::
Ein Werkzeug (z. B. mutmut, cosmic-ray) verändert den Produktivcode in winzigen Mutationen (aus `>` wird `>=`, aus `+` wird `-`) und lässt die Tests laufen. Schlägt ein Test fehl, ist die Mutation „getötet" (gut). Überlebt sie trotz Codeänderung, war die Stelle zwar abgedeckt, aber nicht wirklich geprüft. Es misst also die QUALITÄT der Asserts — genau das, was reine Coverage (nur Ausführung) nicht kann.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0hJ7i3FvurSJjsHcRSQ4R" -->
Wie schließt sich der rote Faden des Kapitels „Testing mit PyTest"?::
Testpyramide (WO teste ich? — breite Unit-Basis) → Modultests (WIE schreibe ich Tests? — AAA, FIRST, pytest) → Mocks (WIE isoliere ich? — Test-Doubles für langsame Ränder) → Coverage (habe ich GENUG und das RICHTIGE getestet? — Lücken finden statt % maximieren).
<!-- basicblock-end -->

## Syntax / API (Coverage)

<!-- clozeblock-start oid="Obs1wLFT7N3zb8C2iUGfOrCv" -->
Coverage misst man in pytest mit dem Plugin ==pytest-cov==; Branch-Coverage schaltet man mit dem Flag ==--cov-branch== ein.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="Obs4lPAhWEWTd5OlGEVoonmE" -->
Die nicht abgedeckten Zeilennummern zeigt der Report ==term-missing== (`--cov-report=term-missing`); einen interaktiven HTML-Report erzeugt `--cov-report=`==html==.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="Obs1PJOM1ke8eTcrD5gwuzrX" -->
In der CI erzwingt man eine Mindestabdeckung mit ==--cov-fail-under==80 (bricht ab, wenn die Coverage darunter liegt); einzelne Zeilen schließt man mit dem Kommentar `# ==pragma: no cover==` aus.
<!-- clozeblock-end -->

<!-- clozeblock-start oid="ObsRm3uno2MjW6UvSsv4tlVs" -->
Die Reihenfolge der Coverage-Maße von schwach nach streng: ==Zeilen==-/Statement-, ==Zweig==-/Branch-, Condition- und ==Pfad==-/Path-Abdeckung.
<!-- clozeblock-end -->
