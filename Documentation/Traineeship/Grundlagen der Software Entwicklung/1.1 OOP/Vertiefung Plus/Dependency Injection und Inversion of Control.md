Geht über [[1.1 OOP - Vertiefung#Dependency Injection|Dependency Injection]] aus der Vertiefung hinaus: Dort werden die drei DI-Formen (Constructor/Setter/Method Injection) an einem Beispiel gezeigt — hier geht es um das größere Prinzip dahinter (**Inversion of Control**), das Gegenbeispiel **Service Locator**, den **Composition Root** als Ort, an dem alles zusammengesetzt wird, sowie Test-Double-Begriffe im Detail. Übersicht siehe [[1.1 OOP - Zusammenfassung|1]].

# Inversion of Control (IoC) — das übergeordnete Prinzip

**Dependency Injection ist ein Spezialfall von Inversion of Control.** IoC bedeutet ganz allgemein: Die Kontrolle darüber, *wann* und *wie* etwas passiert, wird von der eigenen Klasse an einen externen Rahmen abgegeben — „Don't call us, we'll call you." Klassische Beispiele für IoC, die über DI hinausgehen:

- Ein UI-Framework ruft deine `on_click()`-Methode auf (du rufst das Framework nicht selbst zyklisch ab).
- Ein Test-Framework (`pytest`) ruft deine Testfunktionen auf und verwaltet Fixtures.
- Ein Web-Framework ruft deine Route-Handler bei eingehenden Requests auf.

Dependency Injection wendet dieselbe Umkehrung nur auf **Objekterzeugung** an: Statt dass eine Klasse selbst entscheidet und erzeugt, *welche* konkrete Abhängigkeit sie benutzt, gibt sie diese Kontrolle nach außen ab (an den Aufrufer bzw. einen „Composition Root", siehe unten). Das erklärt auch, warum der Fachbegriff „Inversion" heißt: Die natürliche Kontrollrichtung (eine Klasse organisiert sich ihre Abhängigkeiten selbst) wird umgedreht.

# Der Composition Root — wo wird eigentlich „verdrahtet"?

Die Vertiefung zeigt, *wie* man eine Abhängigkeit injiziert (`PriceRecommendationCalculator(writer=CsvPriceRecommendationWriter())`), aber nicht **wo im Programm** diese Verdrahtung passieren sollte. Die Antwort: an möglichst **einer zentralen Stelle**, dem sogenannten **Composition Root** — typischerweise ganz am Anfang von `main()`, wo alle konkreten Implementierungen ausgewählt und die Objektgraphen zusammengebaut werden:

```python
# main.py - der Composition Root: DER EINZIGE Ort, der konkrete Klassen kennt
def main() -> None:
    units = load_units_from_source()

    # Hier, und wirklich nur hier, wird entschieden, WELCHE konkrete Implementierung läuft:
    writer = CsvPriceRecommendationWriter()          # könnte genauso JsonPriceRecommendationWriter() sein
    calculator = PriceRecommendationCalculator(writer=writer)

    calculator.calculate_and_export(units, path="recommendations.csv")


if __name__ == "__main__":
    main()
```

Der Rest der Anwendung (`PriceRecommendationCalculator` selbst) kennt zu keinem Zeitpunkt `CsvPriceRecommendationWriter` als konkreten Typ — nur `main()` (bzw. bei größeren Anwendungen ein dediziertes „Wiring"/"Bootstrap"-Modul) tut das. Verstreut sich diese Entscheidung dagegen über viele Stellen im Code (jede Klasse erzeugt sich ihre Abhängigkeiten selbst, aber „mal CSV, mal JSON, je nach Stelle"), geht der zentrale Überblick verloren, welche Implementierung wo tatsächlich verwendet wird — genau das soll ein Composition Root verhindern.

# Das Gegenbeispiel: Service Locator (Anti-Pattern in den meisten Fällen)

Eine auf den ersten Blick ähnliche, aber in der Praxis oft problematischere Alternative zu DI ist der **Service Locator**: Eine Klasse fragt eine globale Registry aktiv nach ihren Abhängigkeiten, statt sie injiziert zu bekommen:

```python
# Service Locator - Abhängigkeiten werden ANGEFORDERT statt injiziert
class ServiceLocator:
    _services: dict[type, object] = {}

    @classmethod
    def register(cls, interface: type, implementation: object) -> None:
        cls._services[interface] = implementation

    @classmethod
    def resolve(cls, interface: type):
        return cls._services[interface]


class PriceRecommendationCalculator:
    def calculate_and_export(self, units, path: str) -> None:
        writer = ServiceLocator.resolve(RecommendationWriter)   # versteckte Abhängigkeit!
        recommendations = [(u.unit_id, u.calculate_price_recommendation()) for u in units]
        writer.write(path, recommendations)
```

Warum das meist schlechter ist als Constructor Injection:

1. **Versteckte statt sichtbare Abhängigkeiten**: Am Konstruktor von `PriceRecommendationCalculator` (jetzt ganz ohne Parameter!) ist nicht mehr ablesbar, dass die Klasse überhaupt einen `RecommendationWriter` braucht — man muss den gesamten Methodenkörper lesen, um das herauszufinden. Bei Constructor Injection steht die Abhängigkeit direkt und ehrlich in der Signatur.
2. **Schlechtere Testbarkeit**: Ein Test muss den globalen `ServiceLocator` vor jedem Testlauf präparieren (und danach wieder zurücksetzen!) statt einfach ein Test-Double direkt zu übergeben — mehr Setup, mehr Risiko von Seiteneffekten zwischen Tests.
3. **Globaler, geteilter Zustand**: Die Registry ist implizit global — zwei Teile der Anwendung, die „gleichzeitig" unterschiedliche Implementierungen bräuchten (z. B. in parallelen Tests), geraten sich in die Quere.

Merksatz zur Abgrenzung: Bei DI **sieht** man von außen (am Konstruktor/an der Methodensignatur), was eine Klasse braucht. Beim Service Locator **muss man es wissen** (oder im Code nachlesen) — die Abhängigkeit ist da, aber unsichtbar von außen.

# Test-Doubles im Detail — mehr als nur „Mock"

Die Vertiefung nennt ein `FakeWriter`-Beispiel als „Test-Double". In der Literatur (Gerard Meszaros, *xUnit Test Patterns*) werden fünf Arten von Test-Doubles unterschieden, die in Alltagssprache oft alle unter „Mock" zusammengefasst werden:

| Art | Verhalten | Beispiel im PriceableUnit-Kontext |
|---|---|---|
| **Dummy** | Wird nur übergeben, um eine Signatur zu erfüllen, nie tatsächlich benutzt | Ein Platzhalter-Objekt, das nie `.write()` aufgerufen bekommt |
| **Stub** | Liefert vordefinierte Antworten auf Aufrufe, keine Prüfungen | `writer.write()` tut einfach nichts, ohne etwas zu protokollieren |
| **Fake** | Funktionierende, aber vereinfachte Implementierung | `FakeWriter` aus der Vertiefung — sammelt echt im Speicher statt auf Platte |
| **Spy** | Wie Fake/Stub, zeichnet zusätzlich Aufrufe zur späteren Prüfung auf | `FakeWriter.written`-Liste wird nach dem Test inspiziert |
| **Mock** | Hat vorab definierte **Erwartungen** und lässt den Test fehlschlagen, wenn diese nicht erfüllt werden | `unittest.mock.Mock` mit `assert_called_once_with(...)` |

`FakeWriter` aus der Vertiefung ist also streng genommen eine Mischung aus **Fake** und **Spy**: eine simple, aber funktionierende Implementierung (Fake), die zusätzlich ihre Aufrufe protokolliert, um sie im Test zu prüfen (Spy). Python bringt mit `unittest.mock.Mock`/`MagicMock` ein eingebautes Werkzeug speziell für die Mock/Spy-Rolle mit, das Aufrufe automatisch aufzeichnet, ohne dass man wie bei `FakeWriter` eine eigene Klasse schreiben muss — für einfache Fälle reicht das oft aus, für komplexeres, zustandsbehaftetes Verhalten (wie z. B. eine In-Memory-Datenbank) ist ein handgeschriebener Fake trotzdem oft klarer und wartbarer.

# Wann lohnt sich ein echtes DI-Framework?

Die Vertiefung erwähnt kurz `dependency-injector` als Beispielbibliothek. Als Faustregel, wann sich der zusätzliche Abstraktionslevel eines DI-Containers lohnt statt „Poor Man's DI" (einfache Konstruktor-Parameter, manuell im Composition Root verdrahtet):

- **Poor Man's DI reicht**, solange der Objektgraph überschaubar ist (wenige, klar erkennbare Abhängigkeiten, eine Handvoll Klassen) — das trifft auf die meisten Python-Anwendungen zu, auch mittelgroße.
- **Ein DI-Container lohnt sich**, wenn Abhängigkeiten tief verschachtelt sind, häufig zwischen mehreren Umgebungen (Test/Staging/Produktion) ausgetauscht werden müssen, oder wenn Lifecycle-Management (Singletons, Scoped-Instanzen pro Request) manuell zu fehleranfällig würde.

In den meisten Data-Engineering-Kontexten (Pipelines, ETL-Skripte, siehe [[Antworten#ETL|Antworten]]) ist der Objektgraph meist klein genug, dass Constructor Injection ohne Framework die pragmatischere Wahl bleibt.

---

Zurück zur Übersicht: [[1.1 OOP - Zusammenfassung|1]] · Grundlage: [[1.1 OOP - Vertiefung|1]] · Verwandt: [[SOLID-Prinzipien|SOLID-Prinzipien]] (Dependency Inversion Principle)
