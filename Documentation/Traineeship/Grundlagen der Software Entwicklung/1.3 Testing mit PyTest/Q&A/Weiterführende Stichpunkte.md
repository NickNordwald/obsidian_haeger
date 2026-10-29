---
tags:
  - documentation
type: q&a
date: 2026-08-19
---

Sammlung von Fäden, die in der [[1.3 Testing mit PyTest - Vertiefung|Vertiefung]] bewusst nur gestreift oder ganz weggelassen wurden — als Landkarte für einen späteren, tieferen Durchgang. **Kein Pflichtstoff für das Kapitel.**

Legende: 🔴 lohnt sich sehr / berufsrelevant · 🟡 gut zu wissen · ⚪ Bonus, wenn Zeit und Lust da sind
Quellen: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]] · Kapitelrückblick: [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|Kapitelabschluss]]

*Nachträglich angelegt (19.08.2026) — dieser Ordner war leer, während 1.5 und 1.6 längst eine solche Landkarte hatten. Die Punkte sind aus den Querverweisen der späteren Kapitel und dem tatsächlichen Endstand des Projekts rekonstruiert.*

---

# Mike Cohns Testpyramide

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Die Gegenentwürfe zur Pyramide kennen** — Die Vertiefung behandelt die Pyramide als Konsens. Sie ist umstritten: Die **Testing Trophy** (Kent C. Dodds) gewichtet Integrationstests am stärksten, weil moderne Werkzeuge sie billig gemacht haben; die **Testing Honeycomb** (Spotify) tut dasselbe für Microservices. Kernargument der Gegenseite: Unit-Tests mit vielen Mocks prüfen am Ende nur, ob der Code so verdrahtet ist, wie man ihn verdrahtet hat. Wer beide Positionen kennt, kann eine Teststrategie begründen statt sie zu zitieren.
- 🟡 **Das „Ice Cream Cone"-Anti-Muster mit Zahlen** — Die Vertiefung nennt die umgekehrte Pyramide als Anti-Muster. Interessanter ist der ökonomische Grund: Ein E2E-Test kostet nicht nur mehr Laufzeit, sondern vor allem mehr *Wartung* — er bricht bei jeder UI- oder Schema-Änderung, auch wenn die Logik stimmt.
- 🟡 **Wo Contract-Tests in die Pyramide passen** — Zwischen Unit und Integration gibt es eine Ebene, die die Pyramide nicht kennt: Verträge zwischen Diensten (Pact, `schemathesis`). Sie ersetzen einen Teil der E2E-Tests, ohne deren Kosten zu haben.

# Modultests

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Fixtures über den Kurzüberblick hinaus** — Die Vertiefung zeigt Fixtures als „wiederkehrendes Setup". Was fehlt, ist genau das, wofür man sie wirklich braucht: **`yield`-Fixtures mit Teardown** (die Transaktions-Rollback-Fixture!), **`scope=`** (`function`/`class`/`module`/`session` — und die Falle, dass eine `session`-Fixture Zustand zwischen Tests schleppt), **`conftest.py`** zum Teilen ohne Import, **Fixture-Komposition** (eine Fixture fordert andere an), **`autouse=True`**, **Factory-as-Fixture** (die Fixture gibt eine *Funktion* zurück, damit der Test Parameter setzen kann) und **indirekte Parametrisierung**. Der `AppHarness`-Trick aus deinem `test_app.py` ist ein handgebautes Factory-as-Fixture.
- 🔴 **Flaky Tests und Testisolation** — Der Punkt, der in echten Projekten am meisten Zeit frisst und in der Vertiefung gar nicht vorkommt. Ursachen: geteilter Zustand zwischen Tests, Reihenfolgeabhängigkeit, echte Uhrzeit, echtes Netzwerk, Nebenläufigkeit, unsortierte Mengen. Werkzeuge: **`pytest-randomly`** (mischt die Reihenfolge und deckt versteckte Abhängigkeiten auf), `pytest-repeat`, `-p no:randomly` zum Gegenprüfen. Faustregel: Ein Test, der nur manchmal rot ist, ist schlimmer als gar kein Test — er erzieht das Team dazu, Rot zu ignorieren.
- 🔴 **Testdaten-Aufbau: Builder, Factories, Object Mother** — In deinen Tests werden Domänenobjekte von Hand konstruiert. Ab einer gewissen Feldzahl wird das unlesbar und die eigentliche Aussage des Tests verschwindet im Setup. Muster dagegen: **Test Data Builder** (`BookingBuilder().firm().nights(2).build()`), **Object Mother** (benannte Standardfälle), `factory_boy`. Ziel: Im Test steht nur, was für *diesen* Test relevant ist — der Rest ist Default.
- 🟡 **Weitere pytest-Bordmittel** — `tmp_path`/`tmp_path_factory` (echtes Dateisystem, automatisch aufgeräumt — genau das, was deinen `writers.py`-Tests fehlt), `capsys`/`capfd` (stdout/stderr abfangen — für die `print`-Frage aus 1.4), `pytest.approx` mit `rel`/`abs`, `recwarn`, `pytest.ini`-Marker plus `-m`-Selektion, `--lf`/`--ff` (last-failed/failed-first), `-x`, `--sw` (stepwise), `-k` als Ausdruck.
- 🟡 **Parametrisierung über den Grundfall hinaus** — `pytest.param(..., id="…")` für lesbare Testnamen, `marks=pytest.mark.xfail` **innerhalb** einer Parametrisierung (ein einzelner Fall darf bekannt-rot sein), gestapelte `@parametrize`-Decorator (kartesisches Produkt), `indirect=True` (der Parameter geht an eine Fixture statt an den Test).
- 🟡 **`xfail` vs. `skip` vs. `skipif`** — In der Vertiefung nicht behandelt, in der Praxis dauernd gebraucht. `skip` = „läuft hier nicht" (Plattform, fehlende Abhängigkeit), `xfail` = „bekannter Bug, wir wissen es" — und `xfail(strict=True)`, damit ein *unerwartet grüner* xfail rot wird, wenn der Bug repariert ist.
- 🟡 **Zeit testbar machen** — `freezegun` bzw. `time-machine` statt handgebautem `monkeypatch` auf `date`. Deine `FakeDate`-Lösung ist didaktisch besser (sie erzwingt das Verständnis von „patch where it's used"), aber im Alltag nimmt man die Bibliothek.
- 🟡 **Testlaufzeit** — `pytest-xdist` (`-n auto`, Tests auf alle Kerne verteilen) und `--durations=10` (die zehn langsamsten Tests anzeigen). Wird ab ein paar hundert Tests relevant.
- ⚪ **`doctest`** — ausführbare Beispiele im Docstring. Hübsch für Bibliotheks-Code, praktisch selten die richtige Wahl für Anwendungscode.

## Explizit out of scope laut Leitfaden (später relevant)

- 🔴 **Integrationstests** — Der Leitfaden schließt sie aus, und genau deshalb hat dein Projekt bis zum Schluss **0 % Coverage in `persistence/`**. Die Werkzeuge: **`testcontainers-python`** (startet einen echten Postgres-Container pro Testlauf), `pytest-postgresql`, oder schlicht der vorhandene Compose-Dienst mit einer separaten Testdatenbank. Das Kernmuster ist die **Transaktions-Rollback-Fixture**: Test in einer Transaktion laufen lassen, danach `rollback()` — schnell, isoliert, ohne Aufräumcode. → Ausführlicher im [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.6 Datenbank Migrations Tools/Q&A/Weiterführende Stichpunkte.md|1.6-Register]], Abschnitt *Migrationen und Tests*.
- 🔴 **API-Tests** — FastAPIs `TestClient` (bzw. `httpx.AsyncClient` für async), **`dependency_overrides`** zum Einsetzen von Fakes. Hast du in 1.5 faktisch gebaut (`test_api_auth.py`, `test_api_fehler.py`, `test_api_routing.py`), ohne dass dieses Kapitel es je eingeführt hätte. Die zugehörige Faustregel aus dem 1.5-Register: **Wenn ein Controller-Test Fachlogik prüft, ist die Fachlogik am falschen Ort.**
- 🟡 **E2E-Tests** — der Vollständigkeit halber: die ganze Anwendung über ihre echte Außenschnittstelle, hier also Compose hochfahren und HTTP-Anfragen schicken. Teuer, langsam, brüchig — deshalb ganz oben in der Pyramide und dünn gesät.

# Mocks / Simulation von Verhalten echter Objekte

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Die Test-Double-Familie vollständig und mit Haltung** — Die Vertiefung führt Stub und Mock sauber ab. Es fehlen: **Dummy** (nur zum Befüllen einer Signatur, wird nie benutzt), **Fake** (funktionierende Billigimplementierung — ein In-Memory-Repository ist der wichtigste Vertreter und in deinem Projekt der eigentliche Arbeitspferd-Fall) und **Spy** (zeichnet Aufrufe auf, ohne das Verhalten zu ersetzen). Dazu die große methodische Frage: **London School vs. Chicago/Detroit School** — mockt man jede Abhängigkeit (interaktionsbasiert) oder nur die I/O-Ränder (zustandsbasiert)? Du hast intuitiv Chicago gewählt (`builder` und `engine` blieben echt); den Namen dafür zu kennen macht die Entscheidung diskutierbar.
- 🔴 **`create_autospec` / `Mock(spec=…)`** — Steht in deinem eigenen Umsetzungsstand als offener Punkt und ist der wichtigste einzelne Mock-Härtungsgriff: Ein rohes `Mock()` akzeptiert **jeden** Methodennamen und **jede** Signatur. Ein Tippfehler (`assert_called_once` statt `assert_called_once_with`) oder eine geänderte Signatur führt dann zu einem Test, der **grün lügt**. `autospec` bindet den Mock an die echte Schnittstelle.
- 🟡 **`pytest-mock` (`mocker`-Fixture)** — die pytest-idiomatische Hülle um `unittest.mock`, die das Aufräumen automatisch erledigt und verschachtelte `with patch(...)`-Blöcke überflüssig macht.
- 🟡 **Assertion-Werkzeuge am Mock** — über `assert_called_once_with` hinaus: `call_args`/`call_args_list`, `mock_calls` (auch verschachtelte Aufrufe), das `call`-Objekt für Vergleiche, `ANY` als Platzhalter für uninteressante Argumente, `side_effect` als **Liste** (unterschiedliche Rückgabe pro Aufruf — ideal für „erst Fehler, dann Erfolg" beim Testen von Retries).
- 🟡 **`patch.object`, `patch.dict`, `new_callable=PropertyMock`** — die Varianten für „nur diese eine Methode", „nur dieser eine Dict-Eintrag" (z. B. `os.environ`) und „das ist eine `@property`, kein Attribut".
- 🟡 **Wann Dependency Injection das Mocken überflüssig macht** — Die Vertiefung erwähnt den Zusammenhang. Die schärfere Formulierung: *Jedes `patch()` ist ein Hinweis darauf, dass eine Abhängigkeit nicht injiziert wird.* Dein Projekt braucht fast nirgends `patch`, weil die Ports injiziert werden — das ist kein Zufall, sondern der Zahltag des Designs aus 1.1.
- ⚪ **Mocking über Prozessgrenzen** — `responses`/`respx` (HTTP-Antworten fälschen), `moto` (AWS), `vcrpy` (echte Antworten einmal aufzeichnen und danach abspielen). Sobald man fremde APIs anzapft, ist das der Standardwerkzeugkasten.

# Testabdeckung / Code-Coverage

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Mutation Testing praktisch** — In der Vertiefung nur als Ausblick. Die Idee ist die einzige ernsthafte Antwort auf „100 % Coverage sagt nichts über Testqualität": Das Werkzeug verändert deinen Code (`>` → `>=`, `+` → `-`, Konstanten kippen) und prüft, ob ein Test rot wird. Bleibt alles grün, ist die Zeile zwar *ausgeführt*, aber nicht *geprüft*. Werkzeuge: `mutmut`, `cosmic-ray`. Bei deinem 160-Statement-Kern wäre ein Lauf realistisch machbar und vermutlich sehr lehrreich.
- 🔴 **Property-based Testing (Hypothesis)** ⭐ Der größte konzeptionelle Sprung, den dieses Kapitel auslässt. Statt Beispiele anzugeben, beschreibt man **Eigenschaften**, die für *alle* Eingaben gelten müssen — die Bibliothek sucht selbst nach Gegenbeispielen und **schrumpft** sie auf den minimalen Fall. Für dich besonders passend: Invarianten wie „`occupancy` liegt immer in [0, 1], solange `total_capacity > 0`" oder „eine Buchung über *n* Nächte belegt genau *n* Tage". Für Data Engineering (Parser, Transformationen, Round-Trips) das mit Abstand wirkungsvollste Testverfahren.
- 🟡 **Coverage-Feinsteuerung** — `# pragma: no cover` (und wann das legitim ist statt Selbstbetrug), `[tool.coverage.run] omit`/`source`, `--cov-fail-under` als Gate, `exclude_lines` für `if TYPE_CHECKING:` und `raise NotImplementedError`. Und der Punkt, den dein Umsetzungsstand schon richtig benennt: Eine Schwelle gehört auf den **Kern**, nicht aufs Gesamtprojekt.
- 🟡 **Diff-Coverage** — statt „wie viel ist insgesamt abgedeckt?" die praktisch nützlichere Frage: „ist der **geänderte** Code abgedeckt?". Werkzeuge: `diff-cover`, in GitHub-Setups meist Codecov/Coveralls. Löst das Henne-Ei-Problem beim Nachrüsten von Tests in einem gewachsenen Projekt.
- 🟡 **Snapshot-/Approval-Testing** — `syrupy`, `pytest-regressions`: Das Ergebnis wird einmal geprüft und dann eingefroren; künftige Läufe vergleichen dagegen. Gut für große strukturierte Ausgaben (deine `price_recommendations.csv`!), gefährlich als Gewohnheit, weil man Änderungen irgendwann blind bestätigt.
- ⚪ **Weitere Coverage-Arten** — Condition/MC-DC-Coverage (aus der Luftfahrt- und Medizintechnik-Normung, DO-178C). Nur zur Einordnung: Es gibt Domänen, in denen Branch-Coverage als Untergrenze gilt.

## Fitness Functions — was du schon gebaut hast

- 🟡 **Architektur-Tests** — Dein `tests/test_architektur.py` ist ein Test, der keine Funktion prüft, sondern eine **Regel** („die API-Schicht importiert nicht aus `persistence/`"). Der Fachbegriff dafür ist **Fitness Function** (Richards/Ford), das etablierte Werkzeug in der Java-Welt heißt ArchUnit, in Python gibt es `import-linter`. Wert der Vertiefung: zu erkennen, dass das eine eigene Testgattung ist — eine Regel, die nicht von Disziplin abhängt. Dasselbe Muster taucht in 1.6 als `squawk` und in 1.5 als Spectral wieder auf.
- 🟡 **Schema-Vertragstests** — dein `test_schema_vertrag.py` gehört in dieselbe Familie: nicht „rechnet die Funktion richtig?", sondern „passen zwei Repräsentationen noch zueinander?".

# Für dich als Data Engineer besonders anschlussfähig

- 🔴 **Datenpipelines testen** — Ein anderes Problem als Anwendungscode: Die Eingabe ist groß, veränderlich und nicht unter deiner Kontrolle. Die etablierten Antworten: **Datenqualitätsprüfungen als Tests** (Great Expectations, Soda), **Schema-Validierung** (Pandera für DataFrames, Pydantic für Zeilen), **dbt tests** (`unique`, `not_null`, `accepted_values`, `relationships` als deklarative Assertions im Modell) und **Golden-Dataset-Tests** (kleiner, eingefrorener Beispieldatensatz mit bekanntem Erwartungswert). Dein `_validate_net_revenue` ist genau so eine Prüfung — nur handgeschrieben und ohne Framework.
- 🔴 **Idempotenz und Wiederaufsetzbarkeit testen** — Für einen Batch-Lauf die eigentlich wichtige Eigenschaft: *Zweimal laufen lassen muss dasselbe Ergebnis liefern wie einmal.* Das ist eine Aussage, die man testen kann — und die dein `TRUNCATE`-Voll-Reload zufällig erfüllt, ein `INSERT`-Ingest ohne `ON CONFLICT` aber nicht.
- 🟡 **Tests in der CI** — Der Schritt, an dem Tests von „ich führe sie aus, wenn ich dran denke" zu „sie sind eine Bedingung" werden: Test-Matrix über Python-Versionen, Caching der Abhängigkeiten, Dienste (Postgres) im Workflow, `pre-commit` für ruff/mypy vor dem Commit. Dein Projekt hat alle Bausteine (ruff, mypy, pytest, `test_architektur`) — es fehlt nur der Ort, an dem sie automatisch laufen.
- ⚪ **Last- und Performance-Tests** — `locust`, `k6`, `pytest-benchmark`. Relevant, sobald „schnell genug" eine Anforderung mit Zahl ist (siehe *Architekturmerkmale messbar machen* im 1.5-Register).

# Fragen zum eigenen Projekt, die offen geblieben sind

- Die Adapter/`persistence`-Schicht ist bis zum Schluss ungetestet geblieben. Das war eine bewusste Scope-Entscheidung — aber wäre es das auch, wenn die Anwendung Preisempfehlungen für echtes Geld erzeugt? Wo verläuft die Grenze zwischen „Out of Scope" und „unverantwortlich"?
- Welche der 123 Tests würden rot, wenn du `>` in der Heuristik zu `>=` änderst? Genau diese Frage beantwortet Mutation Testing — und sie ist die einzige ehrliche Prüfung der Testqualität.
- `test_domain.py` prüft `days_until_travel` mit Einzelfällen statt `parametrize`. Steht als Übung im Umsetzungsstand — lohnt sich das noch, oder ist es reine Kosmetik? (Antwort: Kosmetik, solange die Fälle lesbar sind.)
- Ist die `AppHarness`-Dataclass in `test_app.py` ein Muster, das man wiederverwenden sollte — oder war sie eine Notlösung für die fehlende Factory-as-Fixture?
