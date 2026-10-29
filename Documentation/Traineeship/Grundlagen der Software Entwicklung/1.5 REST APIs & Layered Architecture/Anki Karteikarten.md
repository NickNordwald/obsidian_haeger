---
deck: "Traineeship::Grundlagen der Software Entwicklung::REST APIs & Layered Architecture"
---

Karteikarten zu [[1.5 REST APIs & Layered Architecture - Vertiefung|1.5 REST APIs & Layered Architecture – Vertiefung]], wachsen mit dem Kapitel mit. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::REST APIs & Layered Architecture`. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**, bei dichtem Stoff mehr, bei dünnem weniger — nicht auf eine Zielzahl auffüllen. Details, die nicht auf eine Karte passen, stehen in der Vertiefung.

# Fortschritt

- [x] n-tier Architektur
- [x] Client/Server
- [x] Softwarearchitektur
- [x] Architekturmuster
- [x] HTTP Basics
- [x] CRUD Basics
- [x] Die 4 zentralen HTTP-Methoden in REST-Controllern
- [x] FastAPI + Uvicorn + Pydantic in Python
- [x] Validierung von Request/Response (Pydantic)
- [x] Swagger & OpenAPI

---

# n-tier Architektur

<!-- basicblock-start oid="ObswcP5dQbm4C3aOeUbWEo73" -->
Was ist der Unterschied zwischen einem **Layer** und einem **Tier**?::
**Layer** = *logische* Aufteilung des Codes (Pakete, Module, Namespaces) — eine Entwurfsentscheidung.<br>**Tier** = *physische* Verteilung auf Maschinen/Prozesse (Deployment, Server, Container) — eine Betriebsentscheidung.<br>Beides fällt oft zusammen, muss aber nicht: Unser Projekt hat 3 Layer, läuft aber als ein Prozess gegen eine DB → faktisch 2 Tiers.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBzztILkJn9csbONlEWGUa" -->
Welche drei klassischen Schichten kennt das Standard-Schichtenmodell und wofür ist jede zuständig?::
1. **Presentation Layer** — nimmt Anfragen von außen entgegen und gibt Antworten zurück (REST-Controller, CLI, UI).<br>2. **Business/Domain Layer** — *was* fachlich passieren soll (Regeln, Berechnungen).<br>3. **Data Access/Persistence Layer** — *wie* Daten gespeichert und geholt werden (Repository, ORM, SQL).<br>Häufig zusätzlich eine **Application/Service-Schicht**, die Abläufe orchestriert, ohne selbst Regeln zu enthalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMQaWgbT7vR1weDo9IdRw2" -->
Was ist die eiserne Regel der Schichtenarchitektur — und darf eine tiefere Schicht eine höhere aufrufen?::
**Abhängigkeiten zeigen nach unten, niemals nach oben.** Nein, eine tiefere Schicht darf keine höhere aufrufen: Das erzeugt einen Zyklus, macht die untere Schicht ohne die obere unbenutzbar und untestbar und zerstört die Austauschbarkeit. Muss eine untere Schicht doch „nach oben" wirken, dreht man die Abhängigkeit per **Dependency Inversion** um (Interface unten definiert, oben implementiert).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obso63BpS6WtDNIFmdmAymCH" -->
Strict Layering vs. Layer Bridging — was ist der Unterschied?::
**Strict Layering (geschlossen):** Jede Schicht darf nur die *direkt* darunterliegende aufrufen.<br>**Layer Bridging / Relaxed Layering (offen):** Schichten dürfen übersprungen werden, z. B. Presentation → Persistence direkt.<br>Strict = entkoppelter, aber viel Durchreicherei. Bridging = weniger Boilerplate, aber die übersprungene Schicht kann ihre Regeln nicht mehr garantieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBco6rg8yEqRmLR9PUVUtJ" -->
Wann ist Layer Bridging in der Praxis vertretbar?::
Als **bewusste, begründete Ausnahme** — typischerweise für reine **Lesepfade** (Report-/Listenansichten), bei denen die Business-Schicht nichts beizutragen hätte und nur eine Pass-Through-Methode entstünde. Nicht vertretbar ist Bridging, das schleichend passiert: Dann hat man am Ende keine Schichten mehr, sondern nur noch Ordner.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7pnzWLK7uEHfayKM4eceh" -->
Nenne die wichtigsten **Vorteile** einer Schichtenarchitektur.::
**Austauschbarkeit** (CSV-Quelle → DB-Quelle, ohne die Fachlogik anzufassen), **Testbarkeit** (jede Schicht einzeln testbar, Nachbarschicht wird gemockt), **Verständlichkeit & Arbeitsteilung** (klar, wo man sucht; parallele Arbeit möglich), **Wiederverwendung** (dieselbe Business-Schicht bedient API, CLI und Batch-Job).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRnXibbTSQIRiDQ5nkXmkq" -->
Nenne die wichtigsten **Nachteile** einer Schichtenarchitektur — und wann sie sich nicht lohnt.::
**Overhead/Boilerplate** (jede Funktion muss durch alle Schichten), **Sinkhole-Anti-Pattern** (Schichten, die nur weiterreichen), **Performance** (Indirektion, mehrfaches Mapping Entity → Domain → DTO), **kein Skalierungsmodell** (ein geschichteter Monolith bleibt ein Monolith).<br>Nicht lohnend bei: kleinen Skripten/Prototypen, latenzkritischen Pfaden, Systemen ohne fachliche Komplexität.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsc3GFtrNh2P2qC4ZyU3ORs" -->
Welche Schicht verursacht bei Änderungen den größten Aufwand und die höchsten Kosten — und warum?::
Die **Datenschicht** bzw. das Datenmodell/Schema. Daten sind **persistent** und überleben jeden Release: Ein Umbau erfordert Migration von Bestandsdaten, meist unter Verfügbarkeitsdruck und ohne einfaches Rollback. Zusätzlich hängt jede höhere Schicht daran — und oft greifen fremde Systeme (Reports, BI) direkt auf dieselbe DB zu, die man nicht mitdeployen kann.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxIrvBEOdLXG1u3pILif11" -->
Warum trennt man Serviceschicht und Datenzugriffsschicht?::
Weil es zwei Arten von Wissen sind: Die Serviceschicht weiß, **was** fachlich passieren soll (Regeln, Abläufe), die Datenzugriffsschicht weiß, **wie** Daten geholt/gespeichert werden (SQL, ORM, Transaktionen).<br>Vorteile: austauschbare Persistenz, Testbarkeit ohne laufende DB, keine Vermischung von SQL-Details mit Fachlogik, technische Querschnittsthemen (Retry, Pooling) an einem Ort, fachliche Regeln nicht über mehrere Repositories dupliziert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCIIh9iJUnMVSXooFuwd3L" -->
In welche Schicht gehört ein REST-Controller — und was darf er *nicht* tun?::
In den **Presentation Layer**. Er übersetzt nur zwischen HTTP und Anwendung: Request entgegennehmen, validieren, an die Service-/Business-Schicht delegieren, Ergebnis + Statuscode zurückgeben.<br>Er darf **keine** Fachlogik enthalten und **nicht** direkt auf Persistenz zugreifen (kein SQL, kein ORM/SQLAlchemy) — sonst ist die Anwendung ohne HTTP-Server nicht mehr benutz- und testbar.
<!-- basicblock-end -->

# Client/Server

<!-- basicblock-start oid="ObsytGfjUL7zumv4RrwK3VZm" -->
Wodurch sind die Rollen **Client** und **Server** definiert — und warum ist „ist das ein Client oder ein Server?" meist die falsche Frage?::
Der **Client** ergreift die Initiative, kennt die Adresse des Servers, kommt und geht. Der **Server** wartet passiv, kennt seine Clients nicht im Voraus, läuft dauerhaft.<br>Die Rollen hängen an der **Interaktion**, nicht an der Maschine: Unser Pricing-Service ist Server gegenüber dem HTTP-Aufrufer und gleichzeitig Client gegenüber Postgres. Richtig gefragt ist also immer „in welcher Beziehung?".
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8llOcS6DCwwzyTLyNihtm" -->
Wodurch unterscheidet sich ein Netzwerkaufruf von einem lokalen Funktionsaufruf?::
**Latenz** (Nanosekunden vs. Millisekunden, Faktor ~10⁶) · **Zuverlässigkeit** (kann *hängen*, statt nur zu gelingen oder zu werfen) · **Serialisierung** (Daten müssen zu JSON o. ä. werden und verlieren Typinformation) · **Sicherheit** (jede Eingabe kommt von einem potenziellen Angreifer) · **Versionierung** (Client und Server werden unabhängig deployt).<br>Merksatz: Ein Methodenaufruf über das Netzwerk ist kein Methodenaufruf — er sieht im Code nur so aus.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsU90TM51BpdPq5PVYtx5nz" -->
Warum weiß ein Client bei einem **Timeout** nicht, ob seine Anfrage ausgeführt wurde — und was folgt daraus?::
Drei Fälle sehen für ihn identisch aus: Die Anfrage kam **nie an**, sie wird **noch verarbeitet**, oder sie war **erfolgreich** und nur die Antwort ging verloren.<br>Daraus folgt die praktische Bedeutung von **Idempotenz**: Operationen sollten so gestaltet sein, dass ein wiederholter Aufruf dasselbe Ergebnis hat wie ein einzelner — nur dann darf der Client gefahrlos erneut senden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCGIANLCTi6TGwh7ZpjZ5a" -->
Was bedeutet **stateless** bei einem Server — und was bedeutet es ausdrücklich *nicht*?::
Der Server merkt sich zwischen zwei Anfragen **nichts über den Client**; jede Anfrage bringt alles Nötige selbst mit.<br>Es bedeutet **nicht** „speichert nichts": Die Datenbank ist selbstverständlich zustandsbehaftet. Gemeint ist nur der **Sitzungszustand** des Clients — der gehört in die Anfrage (Token, IDs) oder in die DB, nicht in den Arbeitsspeicher des Servers.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBd7JGKG83E5MYLJOIKhgA" -->
Welche Vorteile bringt ein zustandsloser (stateless) Server?::
**Skalierbarkeit** (jede Anfrage kann von jeder Instanz beantwortet werden — einfach eine zweite danebenstellen, keine Sticky Sessions nötig) · **Ausfallsicherheit** (stirbt eine Instanz, geht keine Sitzung verloren) · **Nachvollziehbarkeit** (eine einzelne Anfrage im Log erklärt sich selbst) · **Testbarkeit** (jeder Endpunkt einzeln testbar, ohne vorher eine Sitzung aufzubauen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJR6Y4nN2ZOjzfWsfuJUlv" -->
Was sind die „Fallacies of Distributed Computing" und wozu dienen sie?::
Eine Liste von 8 falschen Annahmen beim Bau verteilter Systeme (Peter Deutsch, Sun): 1. Das Netzwerk ist zuverlässig · 2. Latenz ist null · 3. Bandbreite ist unbegrenzt · 4. Das Netzwerk ist sicher · 5. Die Topologie ändert sich nicht · 6. Es gibt einen Administrator · 7. Transportkosten sind null · 8. Das Netzwerk ist homogen.<br>Merksatz: Wer eine API baut, baut ein verteiltes System — auch wenn beides auf demselben Laptop läuft.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsc2ektLdIKCAeTLF16bYei" -->
Thin Client vs. Fat Client — und welche Sicherheitsregel folgt daraus für den Server?::
**Thin Client:** zeigt nur an, der Server rechnet (Terminal, serverseitig gerenderte Webseite). **Fat/Rich Client:** hält selbst Logik und Daten, der Server liefert Rohdaten (Desktop-App, React-SPA).<br>Regel: Der Server darf sich **nie** auf korrektes Client-Verhalten verlassen. Validierung im Client ist Bedienkomfort, Validierung im Server ist Sicherheit — jede Anfrage kann auch von `curl` mit beliebigem Inhalt kommen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs41AgeEwsC2viggcM82zKv" -->
Was ändert sich für eine Anwendung grundlegend, wenn sie vom Batch-Job zum Server wird?::
Der Prozess **läuft dauerhaft** statt durchzulaufen · Eingaben kommen **von außen und sind nicht vertrauenswürdig** · Anfragen kommen potenziell **gleichzeitig** statt seriell · Ein Fehler darf nicht mehr den Prozess beenden, sondern muss **eine** Anfrage mit passendem Statuscode scheitern lassen · Die **Antwortzeit wird Teil der Schnittstelle**.
<!-- basicblock-end -->

# Softwarearchitektur

<!-- basicblock-start oid="ObsXZi72Lg9kVO69SJgaGde9" -->
Wie definiert Martin Fowler Softwarearchitektur — und was ist das praktische Kriterium daraus?::
„Architecture is about the important stuff. Whatever that is." Dahinter stehen zwei Aussagen: Architektur ist (1) das **gemeinsame Verständnis** der erfahrenen Entwickler vom Aufbau des Systems und (2) **das, was schwer zu ändern ist** — wovon man möglichst wenig haben sollte.<br>Praktisches Kriterium: **Ist eine Entscheidung später nur mit großem Aufwand revidierbar, ist sie architektonisch.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsx7nzT1a2DzQN0H73q3faW" -->
Welche **vier Dimensionen** von Architektur unterscheiden Richards & Ford?::
1. **Struktur** — welcher Architekturstil (Schichten, Microservices, …)?<br>2. **Architekturmerkmale** — welche „-ilities" muss das System erfüllen?<br>3. **Architekturentscheidungen** — welche **harten Regeln** gelten verbindlich?<br>4. **Entwurfsprinzipien** — welche **Leitlinien** gelten als Empfehlung, von denen man begründet abweichen darf?
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7IMA2Vi3uFHz7LMIIEU5T" -->
Was unterscheidet eine Architektur**entscheidung** von einem Entwurfs**prinzip**?::
Eine **Entscheidung** ist eine harte Regel, gegen die nicht verstoßen werden darf, ohne sie explizit zu ändern (z. B. „der Controller kennt `adapters/` nicht"). Ein **Prinzip** ist eine Leitlinie, von der man begründet abweichen kann (z. B. „Fachlogik gehört nach `core/`").
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0Ht9iVd99nug6pW8hTVMW" -->
Wie grenzt man Architektur von Entwurf (Design) ab?::
**Architektur** betrifft das System als Ganzes, ist teuer und meist systemweit zu ändern und wird von allen Teams/dem Betrieb bemerkt („wir bauen eine REST API statt einer CLI").<br>**Design** betrifft ein Modul/eine Klasse, ist billig und lokal änderbar („diese Klasse bekommt eine Factory-Methode").<br>Die Grenze ist kontextabhängig. Nützliches Kriterium: **Wie viele Leute muss ich fragen, bevor ich das ändere?**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWM24Vjan1CCIT2GdzRE6v" -->
Was sind **Architekturmerkmale** („-ilities") — nenne Beispiele.::
Die **nicht-funktionalen** Anforderungen: nicht *was* das System tut, sondern *wie gut*. Beispiele: **Testbarkeit**, **Modifizierbarkeit/Wartbarkeit**, **Deploybarkeit**, **Performance/Skalierbarkeit**, **Verfügbarkeit/Fehlertoleranz**, **Sicherheit**, **Beobachtbarkeit** (Logging/Monitoring), **Portabilität**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnAtHPa3H6ISRZXZZyLDUs" -->
Warum kann ein System nicht alle Architekturmerkmale gleichzeitig maximieren?::
Weil sie in direkter Konkurrenz zueinander stehen. Beispiele: Mehr Schichten erhöhen Testbarkeit und Modifizierbarkeit, senken aber die Performance. Microservices erhöhen Deploybarkeit und Skalierbarkeit, senken aber Einfachheit und Datenkonsistenz.<br>Genau daraus folgt die Existenz verschiedener Architekturstile: Jeder Stil optimiert einen anderen Satz von Merkmalen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8pJw8fEA2o360miXo3Lwj" -->
Wie lauten die **zwei Gesetze der Softwarearchitektur** (Richards & Ford)?::
1. **Alles in der Softwarearchitektur ist ein Trade-off.** Wenn jemand behauptet, seine Lösung habe keine Nachteile, hat er sie nur noch nicht gefunden.<br>2. **Das *Warum* ist wichtiger als das *Wie*.** Das *Wie* sieht man am Ordnerbaum; warum so geschnitten und welche Alternative verworfen wurde, sieht man nirgends — deshalb muss es aufgeschrieben werden (→ ADR).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsSFtjlnSIuLHLZ1aMUrnjX" -->
Was ist ein **Architecture Decision Record (ADR)** und warum gibt es ihn?::
Ein kurzes, versioniertes Dokument, das **eine** Architekturentscheidung samt **Kontext, Alternativen und Konsequenzen** festhält.<br>Grund: das zweite Gesetz der Architektur — das *Warum* ist im Code nicht sichtbar. Ohne ADR weiß nach einigen Monaten niemand mehr (auch man selbst nicht), warum eine Struktur so gewählt wurde und was dagegen sprach.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBavGcnd95yTYJCLFahLvS" -->
Code-First vs. Design-First — was ist der Unterschied?::
**Code-First:** Der Code ist die Quelle der Wahrheit, die API-Spezifikation wird daraus **generiert** (so macht es FastAPI mit OpenAPI).<br>**Design-First (API-First):** Die **Spezifikation** ist die Quelle der Wahrheit und entsteht *vor* dem Code; die Implementierung hält sich an den vereinbarten Vertrag.<br>Der eigentliche Unterschied ist, **wer der Chef ist**: Bei Code-First folgt die Schnittstelle der Implementierung, bei Design-First folgt die Implementierung der Schnittstelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsm6XsWfYfjOgMeb3FDR4M5" -->
Wann wählt man Code-First, wann Design-First — und was ist der Preis von Code-First?::
**Code-First** bei kleinen Teams, internen APIs und Prototypen: schnell, aber das Client-Team muss warten und die API wächst uneinheitlich.<br>**Design-First** bei öffentlichen APIs, Team-Grenzen und Konzernumfeld: Server- und Client-Team arbeiten parallel gegen den Vertrag, Konsistenz ist per Guidelines erzwingbar.<br>**Preis von Code-First:** Spiegeln die Response-Modelle einfach die ORM-Klassen, hat man keine API entworfen, sondern sein **Datenbankschema veröffentlicht** — und kann es nicht mehr ändern, ohne Clients zu brechen.
<!-- basicblock-end -->

# Architekturmuster

<!-- basicblock-start oid="Obshv6gwMuh80JyYb61Vnrev" -->
Was ist der Unterschied zwischen einem **Entwurfsmuster** und einem **Architekturmuster**?::
Beide sind erprobte Lösungen für wiederkehrende Probleme, unterscheiden sich aber in **Abstraktionsebene und Reichweite**.<br>**Entwurfsmuster:** löst ein Problem *innerhalb* einer Komponente auf Klassen-/Objektebene (Strategy, Factory, Observer), betrifft wenige Klassen, billig zu ändern. Quelle: Gang of Four.<br>**Architekturmuster:** legt die Grundstruktur des *gesamten* Systems fest (Schichten, Microservices, Event-Driven, Pipeline), betrifft alles, teuer zu ändern.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxqV2vZ4usQCxmq24YvBE5" -->
Welche **drei Abstraktionsebenen** von Mustern unterscheidet man klassisch (POSA) — und warum konkurrieren sie nicht?::
1. **Architekturmuster** — Grundstruktur des Systems (Schichten, Pipeline).<br>2. **Entwurfsmuster** — Struktur einzelner Komponenten (Strategy, Factory).<br>3. **Idiome** — sprachspezifische Umsetzungen (Context Manager, Dunder-Methoden).<br>Sie **liegen übereinander statt zu konkurrieren**: Ein System hat gleichzeitig ein Architekturmuster, dutzende Entwurfsmuster und hunderte Idiome.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCGFJnXYxzInPsWEX2OuzQ" -->
Was ist das Muster **Pipeline / Pipes & Filters** — Stärke, Schwäche, typischer Einsatz?::
Daten fließen durch eine Kette unabhängiger Verarbeitungsschritte, jeder Filter macht genau eine Sache.<br>✅ Sehr gut komponierbar und testbar, Schritte einzeln austauschbar.<br>❌ Ungeeignet für interaktive Abläufe oder Anfragen, die hin- und herspringen.<br>Es ist die **Grundform jeder ETL-Strecke** — das zentrale Muster im Data Engineering.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuMNYQI7F3SLexADXe0rTD" -->
Was kennzeichnet eine **Event-Driven**-Architektur, und welche zwei Spielarten gibt es?::
Komponenten reagieren auf Ereignisse, statt sich gegenseitig aufzurufen; die Kommunikation läuft asynchron über einen Broker.<br>✅ Sehr lose gekoppelt, hoch skalierbar, gut erweiterbar. ❌ Der Gesamtablauf steht nirgends vollständig — Debugging und Fehlerbehandlung sind deutlich schwerer.<br>**Broker:** Ereignisse fließen frei, niemand koordiniert. **Mediator:** eine Instanz orchestriert den Ablauf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsddTSQCA1w32nrcVCyKsSR" -->
Was ist das **Microkernel-/Plugin**-Muster?::
Ein minimaler Kern plus Erweiterungen, die sich an klar definierten Punkten einklinken.<br>✅ Neue Funktionalität ohne Eingriff in den Kern.<br>❌ Die Plugin-Schnittstelle muss von Anfang an richtig geschnitten sein, weil sie später kaum änderbar ist.<br>Beispiele: VS Code, Obsidian, pytest.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstMNSG7Sha49zgUmTYOi5o" -->
Warum sind **Microservices** für kleine Projekte fast immer die falsche Wahl?::
Der Gewinn (unabhängige Deploybarkeit, Skalierbarkeit, Teamautonomie) zahlt sich erst ab einer gewissen Team- und Systemgröße aus. Der Preis fällt dagegen sofort an: Man baut ein **verteiltes System** mit allem, was daran hängt — Netzwerklatenz, Timeouts, keine einfachen Transaktionen mehr, aufwendiger Betrieb und Monitoring (vgl. Fallacies of Distributed Computing).<br>Pragmatischer Mittelweg: **service-based** (wenige grobkörnige Dienste, oft mit gemeinsamer Datenbank).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8uxJtwp88vZClqLzFQSlM" -->
In welcher Reihenfolge wählt man ein Architekturmuster aus?::
**Rückwärts, von den Architekturmerkmalen her:**<br>1. Welche Merkmale sind für dieses System wirklich gefordert — und von wem?<br>2. Welche Muster begünstigen genau diese Merkmale?<br>3. Welchen Preis zahlt man dafür, und ist der akzeptabel?<br>Nicht: „Muster X, weil modern." Die Wahl muss begründet sein (2. Gesetz der Architektur).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsfEnYUGj3Skl9Ypuks6jNX" -->
Warum ist die Schichtenarchitektur der übliche **Startpunkt** für ein neues System?::
Weil sie einfach, extrem verbreitet und gut testbar ist — jeder Entwickler versteht sie ohne Erklärung, und sie kostet fast nichts an Betriebskomplexität.<br>Ihre Schwächen (schwer skalier- und deploybar, Sinkhole-Neigung) werden erst bei Größe relevant. Faustregel: Schichten, solange nichts konkret dagegenspricht — die Alternativen erkauft man immer mit Komplexität.
<!-- basicblock-end -->

# HTTP Basics

<!-- basicblock-start oid="Obs6dYfaSKyEO0NLNcxn2Z9f" -->
Wofür stehen die Abkürzungen **REST** und **CRUD**?::
**REST** = **Re**presentational **S**tate **T**ransfer — „Übertragung von Zustand in Form einer Repräsentation": Auf dem Server liegt eine **Ressource**, über die Leitung geht nie die Ressource selbst, sondern eine **Repräsentation** davon (meist JSON). Geprägt von Roy Fielding (Dissertation, 2000).<br>**CRUD** = **C**reate, **R**ead, **U**pdate, **D**elete — die vier Grundoperationen auf gespeicherten Daten. Stammt aus der Datenbankwelt und ist älter als das Web.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsh4PVSdQErSC2T0OegJ1nh" -->
Was ist **HTTP**, und welche Eigenschaften definieren es?::
Ein **Anwendungsprotokoll** auf Basis von TCP. Definierend sind:<br>**Request/Response** — eine Anfrage, genau eine Antwort<br>**Client-initiiert** — nur der Client beginnt; der Server kann von sich aus nichts schicken (dafür bräuchte man WebSockets/SSE)<br>**Zustandslos** — jede Anfrage steht für sich<br>**Textbasiert** (bis HTTP/1.1) — man kann eine Anfrage von Hand tippen und lesen, deshalb ist HTTP so gut debuggbar<br>**Medienunabhängig** — der Body kann JSON, HTML, PDF oder Video sein; `Content-Type` sagt, was es ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscGNVgdOPtLl5PXNxYIIwl" -->
Was ist HTTP ausdrücklich **nicht** — und was folgt daraus?::
**Kein Transportprotokoll** (das ist TCP darunter) · **keine Verschlüsselung** (das ist TLS → HTTPS) · **keine Datenbank-Semantik** · **keine Garantie für irgendetwas**.<br>HTTP definiert **Bedeutung, keine Ausführung**: Dass ein `DELETE` wirklich löscht oder ein `GET` nichts verändert, ist eine Zusage **deiner Anwendung**, nicht des Protokolls.<br>Merksatz zum Verhältnis zu REST: REST ist kein Framework und keine Bibliothek, sondern **die Disziplin, HTTP nicht zu missbrauchen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs95MRwa0UdAGTirEfJWJSF" -->
Aus welchen Teilen besteht eine **HTTP-Request** — und wie unterscheidet sich die Response davon?::
1. **Request-Line**: `<Methode> <Pfad+Query> <HTTP-Version>`<br>2. **Header**: beliebig viele `Name: Wert`-Zeilen (Metadaten)<br>3. **Leerzeile**: trennt Header vom Body<br>4. **Body**: optionale Nutzdaten (bei `GET`/`DELETE` meist leer)<br>Die **Response** ist identisch aufgebaut, hat statt der Request-Line aber eine **Status-Line**: `<HTTP-Version> <Code> <Reason Phrase>`. Auf die Reason Phrase darf sich kein Client verlassen — nur auf die Zahl.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsITtUJ9NwjFFyAk2yrFvpc" -->
Wie ist eine **URL** aufgebaut, und welcher Bestandteil erreicht den Server nie?::
`scheme://host:port/path?query#fragment` — z. B. `https://api.example.com:8000/v1/units/42/prices?limit=10#chart`.<br>Das **Fragment** (alles ab `#`) wird **nie an den Server geschickt**, es bleibt im Browser.<br>Merkregel für REST: **Path identifiziert, Query modifiziert.** Was die Ressource ausmacht (`/units/42`), gehört in den Pfad; was nur die Sicht einschränkt (`?limit=10`), in die Query.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrYwKBBDfUu3d4N6tJEFP5" -->
Nenne die wichtigsten **Request-Header** und ihre Bedeutung.::
`Host` — welcher virtuelle Host gemeint ist (ein Server bedient oft viele Domains)<br>`Content-Type` — Format des **mitgeschickten** Bodys, z. B. `application/json`<br>`Accept` — welches Format der Client **zurückhaben** will (Content Negotiation)<br>`Authorization` — Credentials, z. B. `Bearer <JWT>` oder `Basic <base64>`<br>`Content-Length` — Länge des Bodys in Bytes<br>`User-Agent` — wer fragt (Browser, `curl`, `httpx`)<br>Die Groß-/Kleinschreibung von Header-Namen ist bedeutungslos.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEaOnPD1qnhWa5lxrJXLT5" -->
Nenne die wichtigsten **Response-Header** und ihre Bedeutung.::
`Content-Type` — Format der Antwort<br>`Location` — wo die neu erzeugte Ressource liegt; **gehört zu jedem `201 Created`**<br>`Cache-Control` / `ETag` — Caching-Steuerung bzw. Versionskennung der Repräsentation<br>`Set-Cookie` — setzt ein Cookie im Browser<br>`Retry-After` — wann der Client es erneut versuchen soll (bei `429` und `503`)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshjSJIddSuZWwd8eKhnXz7" -->
Worin unterscheiden sich `GET` und `POST`/`PUT` — über die Absicht hinaus?::
| | `GET` | `POST`/`PUT` |<br>|---|---|---|<br>| Daten | Pfad + Query | **Body** |<br>| Safe | ✅ | ❌ |<br>| Idempotent | ✅ | `PUT` ✅ / `POST` ❌ |<br>| Cacheable | ✅ | ❌ |<br>| Browser-Reload | gefahrlos | „Formular erneut senden?" |<br>Zusätzlich: Sensible Daten gehören nie in eine GET-Query (Logs, Verlauf, Proxies, `Referer`), und Query-Strings sind längenbegrenzt (~2000 Zeichen), Bodys nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxRj0ZBFfjjE5ov0LwhR5B" -->
Was ist der Unterschied zwischen `POST` und `PUT`?::
**`POST`** erzeugt etwas Neues an einer vom **Server** bestimmten Adresse — der Client kennt die ID vorher nicht. Deshalb **nicht idempotent**: Zweimal `POST /prices` = zwei Datensätze.<br>**`PUT`** legt den Zustand an einer vom **Client** bestimmten Adresse fest („diese Ressource soll ab jetzt so aussehen") und ersetzt sie **vollständig**. Deshalb **idempotent**: Zweimal `PUT /prices/1001` = derselbe Zustand.<br>Faustregel: `POST` auf die **Sammlung** (`/prices`), `PUT` auf das **Element** (`/prices/1001`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsehTdyzKLGysEsDnwWdhzT" -->
Was bedeuten **safe** und **idempotent** bei HTTP-Methoden — und welche Methode ist was?::
**Safe** = die Anfrage verändert **nichts** auf dem Server (deshalb dürfen Browser und Crawler sie einfach ausführen).<br>**Idempotent** = *n* identische Aufrufe hinterlassen denselben Zustand wie ein einzelner.<br>`GET`/`HEAD`/`OPTIONS`: safe **und** idempotent · `PUT`/`DELETE`: nicht safe, aber idempotent · `POST`/`PATCH`: weder noch.<br>Praktische Folge: Nach einem **Timeout** darf ein Client nur idempotente Aufrufe gefahrlos wiederholen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMh5A8nVgJkyQsbXXCzoed" -->
Warum ist ein Endpunkt wie `GET /orders/42/delete` ein schwerer Designfehler?::
Weil `GET` als **safe** definiert ist — es darf nichts verändern. Genau darauf verlassen sich Browser, Suchmaschinen-Crawler, Prefetcher, Proxies und Caches: Sie führen `GET`-Anfragen **spekulativ und ungefragt** aus. Der erste Crawler, der die Links durchgeht, räumt damit die Datenbank leer.<br>Zusätzlich verletzt der Endpunkt die REST-Namenskonvention (Verb im Pfad statt Methode). Richtig ist `DELETE /orders/42`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7j8MZ4CQ6Xt8cejWlNBw8" -->
Wofür gibt es die Methoden `HEAD` und `OPTIONS`?::
**`HEAD`** — wie `GET`, aber der Server schickt **nur die Header, keinen Body**. Nützlich, um Existenz, Größe (`Content-Length`) oder Aktualität (`ETag`) einer Ressource zu prüfen, ohne sie herunterzuladen.<br>**`OPTIONS`** — fragt, **welche Methoden** auf einer Ressource erlaubt sind. In der Praxis vor allem als **CORS-Preflight** relevant: Der Browser fragt vorab, ob er die eigentliche Anfrage überhaupt stellen darf.<br>Beide sind **safe und idempotent**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGQF76MzZPknfh6Eh2U0NH" -->
Warum ist `PATCH` nicht zwingend idempotent, `PUT` aber schon?::
`PUT` überträgt den **vollständigen Zielzustand** — egal wie oft man ihn setzt, das Ergebnis ist dasselbe.<br>`PATCH` überträgt eine **Änderungsanweisung**, und die kann relativ sein: `{"price": 129.90}` ist idempotent, `{"increment": 1}` nicht — zweimal ausgeführt steht der Wert um 2 höher.<br>`PATCH` **kann** also idempotent sein, das Protokoll garantiert es aber nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGI7giYKrUmNEno3g2OhwZ" -->
Was ist der Unterschied zwischen den Headern `Content-Type` und `Accept`?::
**`Content-Type`** beschreibt, **was ich mitschicke** (Format des eigenen Bodys, z. B. `application/json`).<br>**`Accept`** beschreibt, **was ich zurückhaben will** (Content Negotiation).<br>`Content-Type` steht daher in Request *und* Response, `Accept` nur in der Request.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssRp4zdXiOfWbE2SF0yzOv" -->
Welche **fünf Statuscode-Klassen** gibt es, und warum ist die Grenze 4xx/5xx die wichtigste?::
**1xx** Informational · **2xx** Erfolg · **3xx** Umleitung · **4xx** Client-Fehler · **5xx** Server-Fehler.<br>4xx vs. 5xx sagt, **wer schuld ist**. Daran hängen zwei praktische Konsequenzen: (1) **Retry** — bei 4xx unverändert nie sinnvoll, bei 5xx ja (mit Backoff); (2) **Monitoring/Alarmierung** — die 5xx-Rate weckt jemanden, die 4xx-Rate normalerweise nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsd3W2IChbSEFf0gJiTJS15" -->
Welche Statuscodes braucht man für eine typische CRUD-API?::
**200 OK** (erfolgreicher `GET`/`PUT`/`PATCH`, mit Body) · **201 Created** (erfolgreicher `POST`, **plus `Location`-Header**) · **204 No Content** (Erfolg ohne Body, typisch für `DELETE`) · **400 Bad Request** · **404 Not Found** · **409 Conflict** (Duplikat/Konflikt) · **422 Unprocessable Content** (Validierungsfehler, FastAPI-Standard) · **500 Internal Server Error**.<br>Dazu bei Auth: **401** und **403**; bei Ausfall nachgelagerter Dienste: **503**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstB8mM00z3lajKX2gdvUlO" -->
Wann antwortet man mit **201**, wann mit **204** — und was gehört jeweils dazu?::
**201 Created** — nach einem erfolgreichen `POST`, wenn eine neue Ressource entstanden ist. Dazu gehört zwingend der **`Location`-Header** mit der Adresse der neuen Ressource (z. B. `Location: /v1/prices/1001`); im Body üblicherweise die erzeugte Ressource selbst.<br>**204 No Content** — Erfolg **ohne** Body, typisch nach `DELETE` (und manchmal nach `PUT`). Der Client weiß: hat geklappt, es gibt nichts zurückzugeben. Ein `204` **darf keinen Body haben**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsvBN8Y34csOnE5c8jQ8lEF" -->
Wofür stehen die Statuscodes **405**, **409** und **429**?::
**405 Method Not Allowed** — der **Pfad existiert**, aber nicht mit dieser Methode (z. B. `DELETE` auf eine Sammlung). Wichtiger Unterschied zu `404`: Dort existiert die Ressource gar nicht.<br>**409 Conflict** — die Anfrage kollidiert mit dem **aktuellen Zustand**: Duplikat, verletzte Eindeutigkeit, gleichzeitige Änderung durch jemand anderen.<br>**429 Too Many Requests** — Rate Limit erreicht; sollte mit dem Header **`Retry-After`** beantwortet werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsD3MIAMd99xdzKTTfxeSSf" -->
Was unterscheidet **502 / 503 / 504** von einem **500**?::
**500 Internal Server Error** = **dein** Fehler — eine unbehandelte Exception in der eigenen Anwendung.<br>Die anderen drei sagen: Ein **nachgelagerter Dienst** (Datenbank, Fremd-API, Upstream-Server) ist das Problem.<br>**502 Bad Gateway** — der Upstream antwortet ungültig · **503 Service Unavailable** — vorübergehend nicht verfügbar/überlastet, oft mit `Retry-After` · **504 Gateway Timeout** — der Upstream antwortet gar nicht rechtzeitig.<br>Praktischer Wert der Unterscheidung: Sie sagt dem Client (und dem Monitoring), ob ein **Retry** Aussicht auf Erfolg hat.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaSuDlJMEL11OfVS7nrQ1f" -->
Wofür ist **304 Not Modified** da, und wie kommt es zustande?::
Es ist der Caching-Fall: „Deine Kopie ist noch aktuell." Der Server schickt beim ersten `GET` einen **`ETag`** (Versionskennung der Repräsentation) mit. Bei der nächsten Anfrage schickt der Client `If-None-Match: <etag>` — ist die Ressource unverändert, antwortet der Server mit **`304` und ohne Body**.<br>Nutzen: Die Anfrage findet statt (Aktualität ist geprüft), aber es werden **keine Nutzdaten** übertragen. Analog mit `Last-Modified` + `If-Modified-Since`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTZhfSNA1RgmhS6TF51MMB" -->
Erkläre die drei klassischen Statuscode-Verwechslungen: 401/403, 400/422, 404/403.::
**401 vs. 403:** 401 = **nicht authentifiziert** („wer bist du?", Ausweis fehlt oder ist ungültig); 403 = **nicht autorisiert** („ich weiß wer du bist, du darfst trotzdem nicht").<br>**400 vs. 422:** 400 = die Anfrage ist **syntaktisch** kaputt (ungültiges JSON); 422 = syntaktisch gültig, aber **inhaltlich** falsch (Pydantic-Validierungsfehler).<br>**404 statt 403:** Bei sensiblen Ressourcen antwortet man bewusst mit 404, um nicht zu verraten, *dass* die Ressource existiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxVkHykR7CzcTI4q7IAXJP" -->
Warum ist eine Antwort `200 OK` mit `{"success": false, "error": "..."}` ein Anti-Pattern?::
Weil der **Statuscode die Antwort ist** und der Body nur das Detail. Client-Bibliotheken, Proxies, Caches, Retry-Logik und Monitoring werten ausschließlich den Code aus — sie halten die Antwort für einen Erfolg, und der Fehler wird erst sichtbar, wenn ein Mensch den Body liest.<br>Richtig: passender 4xx/5xx-Code, Details im Body — idealerweise im Standardformat **RFC 9457 (Problem Details)**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsACM9dD6z1IfHVfFouu9bT" -->
Wenn HTTP zustandslos ist — wie bleibt man dann auf einer Webseite „eingeloggt"?::
Der Zustand wird bei **jeder** Anfrage erneut **mitgeschickt**, statt vom Server erinnert zu werden. Zwei Wege:<br>**Cookies** — der Server schickt `Set-Cookie: session=abc`, der Browser hängt danach automatisch `Cookie: session=abc` an jede Anfrage an dieselbe Domain. Klassisch für Browser-Anwendungen, CSRF-anfällig.<br>**Token** — der Client verwahrt selbst einen Ausweis und schickt ihn als `Authorization: Bearer <JWT>` mit. Der Standard für APIs, weil er ohne Browser funktioniert.<br>In beiden Fällen bleibt der **Server selbst zustandslos** — das Wissen reist in der Anfrage mit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsB9LBVtbv1uqKqISDoV1fJ" -->
Welche drei **HTTP-Versionen** sind relevant, und was ändert sich zwischen ihnen — und was nicht?::
**HTTP/1.1** (1997) — textbasiert, eine Anfrage pro Verbindung zur Zeit, Keep-Alive. Das, was man beim Debuggen sieht; Uvicorn spricht es.<br>**HTTP/2** (2015) — binär, **Multiplexing** (viele Requests parallel über *eine* Verbindung), Header-Kompression. Löst das Head-of-Line-Blocking von 1.1.<br>**HTTP/3** (2022) — über **QUIC/UDP** statt TCP, schnellerer Verbindungsaufbau; vor allem für mobile/instabile Netze.<br>**Unverändert bleibt die Semantik:** Methoden, Statuscodes und Header bedeuten überall dasselbe — nur die Übertragung auf dem Draht unterscheidet sich. (Deshalb wurde die Semantik 2022 separat in RFC 9110 festgeschrieben.) Man schreibt seine API also **nicht** „für HTTP/2".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsG1NhiJcmR7wE2qPKI1iKF" -->
Wie untersucht man den **Inhalt** einer Anfrage im Browser?::
**DevTools öffnen** (`F12` / `Strg+Umschalt+I`) → Reiter **Network**, dann die Anfrage anklicken:<br>*Headers* — URL, Methode, **Statuscode**, alle Request- und Response-Header<br>*Payload* / *Request* — was gesendet wurde (Body bzw. Query-Parameter)<br>*Response* — die rohe Antwort<br>*Preview* — dieselbe Antwort aufbereitet, JSON als aufklappbarer Baum<br>Wichtige Einstellungen: Filter **`Fetch/XHR`** (blendet Bilder/CSS/Fonts aus), **Preserve log** (überlebt Seitenwechsel und Redirects), **Disable cache**. Der nützlichste Handgriff: **Rechtsklick → Copy → Copy as cURL** — damit ist die Anfrage im Terminal reproduzierbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsX2cZUdMSjytMajqRjz6m9" -->
Wie untersucht man die **Performance** einer Anfrage im Browser — und wie liest man das Ergebnis?::
Der **Waterfall** zeigt jede Anfrage als Balken auf einer Zeitachse — sichtbar wird, was seriell läuft, obwohl es parallel laufen könnte. Der Reiter **Timing** zerlegt eine einzelne Anfrage:<br>`Queueing → Stalled → DNS Lookup → Initial connection (TCP) → SSL (TLS) → Request sent → Waiting (TTFB) → Content Download`<br>**Interpretation:**<br>Hoher **TTFB** (Time To First Byte) = **Serverzeit** — deine Anwendung oder deine DB-Abfrage ist langsam.<br>Hoher **Content Download** = die Antwort ist zu groß (es fehlt Paginierung).<br>Auffälliges **DNS/TCP/SSL** = Verbindungs-, kein Anwendungsproblem — beim zweiten Aufruf durch Keep-Alive meist verschwunden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEj0ha2YnAMioaU0EHs9uS" -->
Womit inspiziert man eine API, die gar kein Browser-Frontend hat?::
**Swagger UI** — von FastAPI automatisch unter `/docs` gehostet; Endpunkte direkt im Browser ausprobierbar.<br>**`curl -v`** — zeigt Request- und Response-Header im Terminal; `--trace` noch detaillierter.<br>**HTTPie** — dasselbe mit menschenlesbarer Syntax und Farbe.<br>**Postman / Insomnia / Bruno** — GUI-Clients mit Collections, Umgebungen und gespeicherten Anfragen.<br>**HTTPX** — der Python-Client, den FastAPIs `TestClient` intern nutzt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4HFLDgVX1NVt4wWQAZKuo" -->
Welche Statuscodes gibt das Pricing-Projekt in welcher Situation zurück?::
**404** — die angefragte `unit_id` existiert nicht<br>**422** — Pydantic lehnt den Request-Body ab (passiert automatisch)<br>**500** — eine unbehandelte Exception aus `exceptions.py`; nach außen generische Meldung, im Log der volle Traceback<br>**503** — Postgres ist nicht erreichbar; **nicht** `500`, weil das Problem bei einem **nachgelagerten Dienst** liegt und ein Retry sinnvoll sein kann<br>**201 + `Location`** — neue Buchung/Preisempfehlung angelegt<br>Das ist der Brückenschlag zu Kapitel 1.4: Die dort gebaute **Exception-Hierarchie muss auf Statuscodes abgebildet werden** — diese Zuordnungstabelle ist eine Design-Entscheidung, keine Formalie.
<!-- basicblock-end -->

# CRUD Basics

<!-- basicblock-start oid="ObsKFqyQqFU4RC8pjR7cLT0l" -->
Was ist **CRUD**, woher stammt der Begriff, und was ist die These dahinter?::
**C**reate, **R**ead, **U**pdate, **D**elete — die vier Grundoperationen auf gespeicherten Daten. Geprägt von James Martin (1983) in der **Datenbankwelt**, also deutlich älter als das Web.<br>**Die These:** Alles, was man mit gespeicherten Daten tun kann, lässt sich auf diese vier Operationen zurückführen — der kleinste gemeinsame Nenner jeder Datenhaltung.<br>Und wie jeder kleinste gemeinsame Nenner ist CRUD **vollständig, aber nicht ausdrucksstark**: Fachliche Vorgänge („stornieren", „freigeben") passen nicht hinein.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsK1wAuNouqJd4uitLhfBJo" -->
Ordne CRUD den SQL-Befehlen, SQLAlchemy und den HTTP-Methoden zu.::
**Create** → `INSERT` → `session.add(obj)` → `POST`<br>**Read** → `SELECT` → `session.get()` / `select()` → `GET`<br>**Update** → `UPDATE` → Attribut setzen + `flush` → `PUT` / `PATCH`<br>**Delete** → `DELETE` → `session.delete(obj)` → `DELETE`<br>Der Kern des Kapitels steckt in der letzten Spalte: Die **Presentation-Schicht macht über HTTP dieselben vier Operationen zugänglich**, die die Persistenz-Schicht schon kann.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0l1v3l8TiXQO2WUaIE7Qj" -->
Warum ist es ein Fehler, jede Datenbanktabelle 1:1 als Endpunkt zu veröffentlichen?::
Weil man dann keine API entworfen, sondern sein **Datenbankschema publiziert** hat — und es danach nicht mehr ändern kann, ohne alle Clients zu brechen. Die interne Struktur wird zum öffentlichen Vertrag.<br>Die Zuordnung CRUD ↔ SQL ↔ HTTP ist eine **Übersetzungshilfe, kein Bauplan**. Der Ressourcen-Zuschnitt richtet sich danach, was der Client fachlich braucht, nicht danach, wie die Daten zufällig liegen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEu2BBJHO98Li8XGwGpiVP" -->
Warum hat jede Entität in einer REST-API **zwei** Pfade — und welche Methode gehört an welchen?::
Weil „Read" zwei Bedeutungen hat: die **Sammlung** (`/recommendations`) und das **Element** (`/recommendations/1001`).<br>**Sammlung:** `GET` = Liste lesen · `POST` = neu anlegen<br>**Element:** `GET` = eines lesen · `PUT` = ersetzen · `DELETE` = löschen<br>Die Logik dahinter: `POST` geht an die Sammlung, **weil das neue Element noch keine Adresse hat**; `PUT`/`DELETE` gehen ans Element, weil man wissen muss, welches gemeint ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsitZxWhIpxSgWpoyNhwUhE" -->
Was ist eine **Ressource** — und was ist sie nicht?::
Eine Ressource ist **ein Ding, kein Vorgang** (`/bookings` ✅, `/createBooking` ❌) und alles, was einen Namen verdient — auch eine Beziehung (`/units/42/bookings`) oder eine berechnete Sicht.<br>Sie ist **nicht** identisch mit einer Datenbanktabelle und **nicht** identisch mit einer Klasse.<br>Zentral: Über die Leitung geht **nie die Ressource selbst, sondern eine Repräsentation** davon (JSON, XML, HTML …). Genau das steckt im Namen **Re**presentational **S**tate Transfer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsK0Q773JsRPt5vJuqK8czP" -->
Nenne die **sechs REST-Constraints** von Roy Fielding.::
1. **Client/Server** — Trennung von Darstellung und Datenhaltung<br>2. **Stateless** — der Server hält keinen Sitzungszustand<br>3. **Cacheable** — Antworten sagen, ob sie zwischengespeichert werden dürfen<br>4. **Uniform Interface** — alle Ressourcen werden gleich angesprochen *(das Kern-Constraint)*<br>5. **Layered System** — der Client weiß nicht, ob er mit dem Server oder einem Proxy spricht<br>6. **Code-on-Demand** *(optional)* — der Server darf ausführbaren Code schicken<br>Fielding hat REST 2000 nicht erfunden, sondern **nachträglich beschrieben, warum das Web funktioniert**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsm88hcw9ZOYcKhQ4xkzwoJ" -->
Aus welchen vier Teilen besteht das **Uniform Interface**?::
1. **Identifikation von Ressourcen** — jede Ressource hat eine eigene, stabile URL (daher: Substantive im Pfad)<br>2. **Manipulation über Repräsentationen** — der Client schickt eine veränderte Repräsentation, statt eine Methode fernaufzurufen<br>3. **Selbstbeschreibende Nachrichten** — Methode, Statuscode und `Content-Type` reichen zur Interpretation; auch ein Proxy ohne Fachwissen kann korrekt cachen<br>4. **HATEOAS** — die Antwort enthält Links auf die möglichen nächsten Schritte<br>Aus diesem Constraint leiten sich fast alle praktischen REST-Regeln ab.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObshZ2ens1hT92LJcB7txR4C" -->
Was besagt **HATEOAS**?::
**H**ypermedia **a**s **t**he **E**ngine **o**f **A**pplication **S**tate: Die Antwort enthält **Links auf die möglichen nächsten Schritte** — so wie eine HTML-Seite Links enthält. Der Client muss URLs nicht fest verdrahten, sondern folgt dem, was der Server anbietet.<br>Beispiel: `{"id": 42, "_links": {"cancel": "/bookings/42/cancellation"}}`<br>Es ist der berühmteste und der **am seltensten umgesetzte** Teil von REST.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsoF7F7lUDh1TZ5yFfbdJPB" -->
Welche vier Level kennt das **Richardson Maturity Model** — und wo stehen echte APIs?::
**Level 0 („Swamp of POX")** — ein Endpunkt, alles per `POST`, HTTP nur als Tunnel<br>**Level 1 — Ressourcen** — verschiedene URLs, aber weiterhin alles `POST`<br>**Level 2 — HTTP-Verben & Statuscodes** — Methoden nach ihrer Bedeutung, Statuscodes als Antwort<br>**Level 3 — Hypermedia (HATEOAS)** — Antworten enthalten Links auf Folgeaktionen<br>**Praktisch jede „REST-API" der Industrie steht auf Level 2.** Nach Fieldings strenger Definition ist das gar kein REST — der pragmatische Umgang: Level 2 anstreben, es „REST" nennen wie alle anderen, aber **wissen, dass man eine Abkürzung nimmt und welche**.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs33fLaOPVjMA5Rs4xxSKDC" -->
Warum sind **REST und CRUD nicht dasselbe**?::
**CRUD** ist eine Menge von **Operationen** (aus der Datenbankwelt, 1983) und sagt, *was* man mit Daten tun kann.<br>**REST** ist ein **Architekturstil** (Web-Architektur, 2000) und sagt, *wie* ein verteiltes System aufgebaut sein sollte.<br>Daraus folgen zwei Sätze:<br>**Eine REST-API kann CRUD anbieten — muss aber nicht.**<br>**Eine CRUD-API ist noch lange nicht RESTful** (vgl. Level 0 und 1 im Richardson-Modell).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsATiNtDmu2gyW3oO5N1sxn" -->
Warum ist `PUT /bookings/42` mit `{"status": "cancelled"}` eine schlechte Modellierung für „Buchung stornieren"?::
Weil die fachliche Bedeutung in einem **Datenfeld versteckt** wird. Der Server kann nicht mehr unterscheiden, ob jemand **storniert** oder nur ein Feld korrigiert — und kann daran folglich keine Regeln, Rechte, Validierungen oder Ereignisse hängen.<br>Der Kern des Problems: Stornieren ist kein Create/Update/Delete, sondern ein **fachlicher Vorgang**. Vorgänge passen nicht in vier Verben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEE6847gs9gE0l31y6lGq0" -->
Wie modelliert man Vorgänge, die nicht in CRUD passen (z. B. „stornieren", „Neuberechnung anstoßen")?::
Leitfrage: **„Welches Ding entsteht hier eigentlich?"** — meist gibt es eines. Drei Wege, absteigend nach „Reinheit":<br>1. **Vorgang als eigene Ressource** (REST-konform): `POST /bookings/42/cancellation`, `POST /pricing-runs` → der Lauf bekommt Identität, Status und Abfragbarkeit; bei langer Laufzeit `202 Accepted` + `Location`.<br>2. **Sub-Ressource für den Zustand**: `PUT /bookings/42/status`<br>3. **Action-Endpunkt mit Verb**: `POST /bookings/42/cancel` — unrein, machen aber auch Stripe und GitHub. Muss die **begründete Ausnahme** bleiben, sonst ist man bei Level 0 mit hübscheren URLs.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNknTW9eksVWjXMK0fC8C2" -->
Warum wäre `POST /recommendations` im Pricing-Projekt fachlich fragwürdig — und was macht man stattdessen?::
Weil eine Preisempfehlung **nicht angelegt, sondern berechnet** wird. Sie ist ein **Ergebnis**, kein Eingabedatum — ein vom Client mitgebrachter Empfehlungswert ergibt fachlich keinen Sinn. `/recommendations` bietet daher nur `GET` an (Liste + Element).<br>Für das Anstoßen einer Neuberechnung modelliert man den **Lauf als Ressource**: `POST /pricing-runs` startet `_compute()`, `GET /pricing-runs/{id}` liefert Status und Ergebnis. Das passt gut auf die vorhandene Struktur — `_compute()` *ist* bereits ein Lauf, er hatte nur nie eine Identität.
<!-- basicblock-end -->

# Die 4 zentralen HTTP-Methoden in REST-Controllern

<!-- basicblock-start oid="ObsAodIDhg6wTOjz4GAMt1li" -->
Was ist ein **Controller**, welche fünf Schritte macht er — und was darf er nicht?::
Die Sammlung von Funktionen, die HTTP-Anfragen entgegennimmt (in FastAPI **Router**, in Flask Blueprint, in Django View). Fünf Schritte:<br>1. Anfrage entgegennehmen (Pfad, Query, Body) · 2. Eingabe validieren · 3. an die **Service-Schicht delegieren** · 4. Ergebnis übersetzen (Domain-Objekt → Antwortmodell) · 5. Statuscode setzen.<br>**Nicht erlaubt:** rechnen, SQL absetzen, Sessions verwalten, fachliche Regeln prüfen.<br>Ein Handler ist typisch **3–10 Zeilen**. Lackmustest: *Müsste diese Zeile auch in einem CLI-Kommando stehen?* Dann gehört sie eine Schicht tiefer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgvDB4jrCV9wDNnfNQNZ05" -->
Was liefert `GET /bookings?product_id=GIBTSNICHT` — und warum nicht `404`?::
**`200 OK` mit leerem Array `[]`.**<br>Die **Sammlung** `/bookings` existiert — sie ist nur gerade leer. Ein `404` würde bedeuten, dass es den **Endpunkt** nicht gibt, und zwingt jeden Client, einen Fehlerfall zu behandeln, der keiner ist.<br>**`404` gilt ausschließlich für ein Element**, das es nicht gibt (`GET /bookings/42`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTksN9cgve8tUWUwuIlBeT" -->
Welche zwei Dinge sollte man bei jedem `GET` auf eine Sammlung von Anfang an mitdenken?::
**Ein Default-Limit** — ohne Grenze liefert die Sammlung irgendwann alle Datensätze und wird zum Performance- und Speicherproblem.<br>**Eine stabile Sortierung** — ohne festes `ORDER BY` ist Paginierung bedeutungslos, weil die Reihenfolge zwischen zwei Anfragen wechseln kann und Datensätze dadurch doppelt oder gar nicht erscheinen.<br>Beides ist später teuer nachzurüsten, weil Clients sich auf das bisherige Verhalten verlassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsj9ov5MtD4MtxcowpiWYMv" -->
Was gibt ein erfolgreicher `POST` zurück — und warum jeweils?::
**`201 Created`** + **`Location`-Header** + **Body mit der erzeugten Ressource**.<br>**Warum `Location`:** Der Client hat die ID nicht vergeben, kennt die Adresse der neuen Ressource also nicht.<br>**Warum ein Body**, obwohl der Client die Daten selbst geschickt hat: Der Server hat sie **ergänzt** (ID, Status, Zeitstempel, normalisierte Werte). Der Client soll nicht raten müssen und kein sofortiges `GET` brauchen.<br>Bei einem asynchronen Vorgang stattdessen **`202 Accepted`** + `Location` auf die Vorgangs-Ressource.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obscs1k440ij1QgJsAS9c2X6" -->
Warum ist die fehlende **Idempotenz von `POST`** ein praktisches Problem — und wie löst man es?::
Wegen des Timeout-Problems: Der Client weiß nicht, ob die Buchung angelegt wurde. Sendet er erneut, hat er womöglich **zwei**; sendet er nicht, vielleicht **keine**.<br>Lösung ist der **`Idempotency-Key`**: Der Client erzeugt eine eindeutige ID und schickt sie im Header mit. Der Server erkennt daran eine Wiederholung und liefert die **ursprüngliche Antwort erneut**, statt ein zweites Objekt anzulegen. (Von Stripe bekannt gemacht.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsK0ujRsft37OEfhVrep2B8" -->
Was passiert bei `PUT /bookings/42` mit `{"nights": 5}`, wenn `PUT` korrekt umgesetzt ist?::
Alle **nicht mitgeschickten Felder werden geleert** — `product_id`, `status` usw. sind danach weg.<br>Grund: `PUT` bedeutet **„die Ressource soll ab jetzt genau so aussehen"**, nicht „ändere die Felder, die ich mitschicke". Wer nur ein Feld ändern will, schickt entweder das **komplette** Objekt oder nimmt **`PATCH`**.<br>In der Praxis führt diese Verwechslung zu **stillem Datenverlust**: Das Objekt ist noch da, nur halb leer — und niemand merkt es, bis jemand nachfragt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1YldV6cbY6bS7oWG8loxp" -->
Wann nimmt man `POST`, wann `PUT`, um etwas Neues zu erzeugen?::
**Faustregel: Server vergibt die ID → `POST`. Client kennt die Adresse → `PUT`.**<br>`POST /bookings` erzeugt an einer vom Server bestimmten Adresse (der Client kennt die ID vorher nicht) → nicht idempotent.<br>`PUT /capacity/A1/2026-08-01` setzt den Zustand an einer vom Client bestimmten Adresse — existiert die Ressource noch nicht, ist das ein legitimes **Upsert**; die Antwort ist dann `201 Created` statt `200 OK`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHeJ3khkHJy6v0SolWgmPH" -->
Welche zwei standardisierten Body-Formate gibt es für `PATCH`?::
**JSON Merge Patch (RFC 7396)** — sieht aus wie ein Teilobjekt, `null` löscht ein Feld: `{"nights": 5}`. Das in der Praxis übliche Format, oft ohne Kenntnis des Namens.<br>**JSON Patch (RFC 6902)** — eine Liste von Operationen: `[{"op": "replace", "path": "/nights", "value": 5}]`. Mächtiger (auch `add`, `remove`, `move`, `test`), aber umständlicher.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEKj2YYvlZ5sxCpE95xZNI" -->
Was ist das **Lost-Update-Problem**, und wie löst HTTP es?::
Zwei Clients lesen dasselbe Objekt, beide ändern etwas, beide schreiben zurück — die **zweite Änderung überschreibt die erste unbemerkt**.<br>Die HTTP-Antwort darauf ist **optimistisches Locking** über `ETag`: Der Server liefert beim Lesen einen `ETag` mit; beim Schreiben schickt der Client **`If-Match: <etag>`**. Hat sich die Ressource inzwischen geändert, antwortet der Server mit **`412 Precondition Failed`** statt zu überschreiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsS8s6jeOy0hkKmsDodTXlh" -->
Warum ist `DELETE` **idempotent**, obwohl der zweite Aufruf `404` liefert?::
Weil **Idempotenz eine Aussage über den Zustand des Servers ist, nicht über den Statuscode**. Nach dem ersten `DELETE /bookings/42` ist die Buchung weg; nach dem zweiten ist sie immer noch weg — der Zustand ist identisch.<br>Dass der zweite Aufruf `404` statt `204` antwortet, ändert daran nichts. (Manche APIs antworten deshalb bewusst immer mit `204`. Beides ist vertretbar — nur konsistent muss es sein.)
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstnprCQ5XXZHkJVZsOhAfz" -->
Welche Antworten geben `POST`, `PUT` und `DELETE` nach REST Best Practice zurück?::
**`POST`** → `201 Created` + `Location` + erzeugte Ressource im Body (asynchron: `202 Accepted`)<br>**`PUT`** → `200 OK` mit Body **oder** `204 No Content`; hat es erzeugt: `201` + `Location`<br>**`PATCH`** → `200 OK` mit der aktualisierten Ressource<br>**`DELETE`** → `204 No Content` (oder `200` mit Body, `202` asynchron); ein `204` **darf keinen Body haben**<br>`PUT` darf `204` geben, weil der Client den Inhalt kennt — er hat ihn gerade selbst geschickt.<br>**Die eigentliche Best Practice ist Konsistenz:** Es ist weniger schlimm, `DELETE` mit `200` statt `204` zu beantworten, als es *manchmal* so und *manchmal* anders zu tun.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4KpEUs3OE9Rt7CyTEXjuP" -->
Nenne die wichtigsten **Naming-Konventionen für REST-Endpunkte**.::
**Substantive statt Verben** (`POST /bookings`, nicht `/createBooking`) · **Plural** für Sammlungen · **Kleinschreibung** · **Bindestrich statt Unterstrich** im Pfad (`/price-recommendations`) · **kein abschließender Slash** · **keine Dateiendungen** (Format über `Accept`) · **Hierarchie für Beziehungen** (`/units/42/bookings`) · **Filter in die Query**, nicht in den Pfad (`?status=open`, nicht `/bookings/open`) · **Version am Anfang** (`/v1/bookings`).<br>Im JSON-Body: konsistente Schreibweise (im Python-Umfeld `snake_case`), Zeitstempel als **ISO 8601** mit Zeitzone.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGoKPL0XHBQODzKgoARxLV" -->
Welche zwei Prinzipien stecken hinter allen REST-Naming-Regeln?::
1. **Der Pfad beschreibt ein *Ding*, die Methode die *Handlung*.** Steht ein Verb im Pfad, ist meistens die Methode falsch gewählt.<br>2. **Verschachtelung höchstens eine Ebene tief.** `/units/42/bookings` ✅, `/units/42/bookings/7/items/3` ❌ — ab da nimmt man eine eigene Top-Level-Ressource mit Filter (`/items?booking_id=7`). Sonst hängt jede URL an einer Hierarchie, die sich fachlich ändern kann.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBN9LNv6l9vUVev3sFiiSG" -->
Wann teilt man einen Controller auf, und nach welchem Kriterium ausdrücklich **nicht**?::
**Grundregel: ein Controller pro Ressource** bzw. pro Aggregat (`bookings.py`, `prices.py`, …). Anlässe zum Aufteilen:<br>· Datei > ~200–300 Zeilen oder ~10 Endpunkte<br>· die Endpunkte gehören **verschiedenen fachlichen Themen** an (das eigentliche Kriterium)<br>· unterschiedliche **Zugriffsrechte** oder Lebenszyklen<br>· die Swagger-UI wird unübersichtlich — die Gruppierung dort entsteht aus den Routern (`tags`)<br>**Nicht** nach technischer Ähnlichkeit schneiden (`alle_gets.py`, `alle_posts.py`) — das ist der API-Zwilling von „Package by Layer statt Package by Feature".
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHfY9R5KH38WcnsPHvtFfr" -->
Wie organisiert man mehrere Controller in **FastAPI**?::
Über **`APIRouter`** — ein eigenständiges Bündel von Endpunkten mit gemeinsamem Präfix, Tags und Abhängigkeiten:<br>`router = APIRouter(prefix="/bookings", tags=["bookings"])`<br>`@router.get("")` → `GET /bookings` · `@router.get("/{booking_id}")` → `GET /bookings/42`<br>Die Hauptanwendung setzt sie zusammen: `app.include_router(bookings.router)`.<br>Gewinn: Das **Präfix steht an genau einer Stelle**, die Gruppierung in der Doku ergibt sich von selbst, jeder Router ist einzeln testbar. Gemeinsame Abhängigkeiten (DB-Session, Auth) kommen über **`Depends`** an einen Ort statt in jeden Handler.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obshhjj3H8gtQdmDXuXpRZkv" -->
Warum gehört die Zuordnung Exception → Statuscode **nicht** in die einzelnen Handler?::
Weil sie sonst in **jedem** Handler als `try/except` dupliziert wird — bei elf Endpunkten elfmal dieselbe Fehlerbehandlung, die auseinanderlaufen wird.<br>Der richtige Ort ist ein **zentraler Exception-Handler** (`@app.exception_handler(...)`), der die Hierarchie aus `exceptions.py` **einmal** auf Statuscodes abbildet.<br>Der Nebeneffekt ist der eigentliche Gewinn: **Ein Handler, der nur den Erfolgsfall beschreibt, bleibt kurz genug, um lesbar zu sein.**
<!-- basicblock-end -->

# FastAPI + Uvicorn + Pydantic in Python

<!-- basicblock-start oid="Obsx0psWig1DNdZWXv1R1BfL" -->
Welche Aufgabe hat jede der drei Bibliotheken **Uvicorn**, **FastAPI** und **Pydantic**?::
**Uvicorn** — der **ASGI-Server**: Wie kommen HTTP-Bytes vom Netzwerk in Python? Ohne ihn läuft die Anwendung gar nicht.<br>**FastAPI** — das **Framework**: Welche Funktion bearbeitet welchen Pfad? Ohne es müsste man Routing selbst schreiben.<br>**Pydantic** — die **Datenvalidierung**: Sind diese Daten gültig, und welche Typen sind es? Ohne sie wäre jeder Body ein ungeprüftes `dict`.<br>Merksatz: **FastAPI beschreibt die Anwendung, Uvicorn führt sie aus.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsarN43aETpilKgEXRr1hxJ" -->
Welche vierte Bibliothek steckt unsichtbar im FastAPI-Stack, und was macht sie?::
**Starlette** — das ASGI-Toolkit, auf dem FastAPI aufbaut. Es übernimmt Routing, Middleware, die Request-/Response-Objekte, `TestClient`, WebSockets und statische Dateien.<br>FastAPI ist im Kern eine dünne, sehr clevere Schicht über **Starlette** (HTTP-Mechanik) und **Pydantic** (Daten) — genau deshalb ist es so klein und trotzdem so leistungsfähig.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxllqSrvjYSJjhQaSQDuEN" -->
Was ist der Unterschied zwischen **WSGI** und **ASGI**?::
**WSGI** (1997) — synchron, ein Aufruf pro Anfrage, Nebenläufigkeit über Threads/Prozesse. Kann keine WebSockets oder langlebigen Verbindungen. Server: Gunicorn, uWSGI. Frameworks: Flask, klassisches Django.<br>**ASGI** (ab 2016) — asynchron, ereignisbasiert (`receive`/`send`), Nebenläufigkeit über einen **Event Loop**. Server: **Uvicorn**, Hypercorn, Daphne. Frameworks: FastAPI, Starlette.<br>Der Kern ist nicht „schneller", sondern **wie gewartet wird**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRZ18GxG7fvei0xjGoZkFQ" -->
Warum passt das **asynchrone Modell** ausgerechnet für Webserver so gut?::
Weil eine typische API-Anfrage fast nur **wartet** — auf die Datenbank, auf fremde APIs, auf die Festplatte (**I/O-bound**, kaum Rechenarbeit).<br>Im WSGI-Modell blockiert der Thread währenddessen: 100 gleichzeitige Anfragen brauchen ~100 Threads mit eigenem Stack und Kontextwechselkosten.<br>Im ASGI-Modell **gibt die Funktion beim Warten die Kontrolle an den Event Loop zurück**, der in derselben Zeit andere Anfragen bearbeitet — ein einziger Thread bedient tausende wartende Verbindungen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obst5NjYTdIT8rek1Ds2fzcT" -->
Was ist **Uvicorn** genau — und was ist es nicht?::
Ein **ASGI-Webserver**: das Programm, das auf einem Port lauscht, TCP-Verbindungen annimmt, HTTP parst, die Anfrage in ASGI-Ereignisse übersetzt, deine Anwendung aufruft und die Antwort zurückschreibt.<br>**Kein Framework** — es kennt weder Routen noch Modelle noch deine Fachlichkeit, nur die ASGI-Schnittstelle. Deshalb kann derselbe Uvicorn jede ASGI-App ausführen (FastAPI, Starlette, Django).<br>Schnell durch **`uvloop`** (Event Loop auf libuv, wie Node.js) und **`httptools`** (HTTP-Parser aus nginx-Code) — beides C-Implementierungen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsevf6ZuXqDKzbhXGj1UDis" -->
Was ist der Unterschied zwischen dem Uvicorn-Start in Entwicklung und in Produktion?::
**Entwicklung:** `uvicorn pricing.api.main:app --reload` — Neustart bei jeder Dateiänderung. **`--reload` gehört niemals in Produktion.**<br>**Produktion:** mehrere **Worker** (`--workers 4`, mehrere Prozesse gegen die GIL-Grenze; nicht kombinierbar mit `--reload`) und üblicherweise ein **Reverse Proxy** (nginx, Traefik) davor, der TLS, Kompression und statische Dateien übernimmt — Uvicorn spricht in Produktion oft gar kein TLS.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs7FQgWsS0ohZanWDmBykat" -->
Welche **fünf Dinge** leitet FastAPI aus einer einzigen typannotierten Funktionssignatur ab?::
Aus `@app.post("/bookings", status_code=201)` + `def create_booking(b: BookingCreate) -> BookingRead:` folgt automatisch:<br>1. **Routing** — Pfad und Methode<br>2. **Deserialisierung** — JSON-Body → Python-Objekt<br>3. **Validierung** — falsche Typen ⇒ `422` mit präziser Meldung, ohne eine Zeile Prüfcode<br>4. **Serialisierung** — Rückgabe → JSON, gefiltert auf die Felder von `BookingRead`<br>5. **Dokumentation** — OpenAPI-Schema und Swagger UI<br>Dazu Autovervollständigung und Typprüfung im Editor, weil es echte Typen sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLSS65GHFPpDOW1JBwRSpW" -->
Welches strukturelle Problem klassischer Frameworks löst FastAPI durch Typannotationen?::
Dort steht jedes Detail **dreimal**: im Validierungscode, in der Serialisierung und in der Dokumentation. Diese drei laufen zwangsläufig auseinander — und **die Dokumentation ist zuerst gelogen**.<br>FastAPI benutzt die **Typannotation als einzige Quelle der Wahrheit** und macht Divergenz damit strukturell unmöglich, weil es nur *eine* Definition gibt.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslEDBEsxeJ7nQntGC2YYXq" -->
Warum ist „FastAPI ist schnell" der **am wenigsten wichtige** Grund für seine Beliebtheit?::
Weil in fast jeder realen Anwendung **die Datenbank der Flaschenhals ist, nicht das Framework**. Die Framework-Geschwindigkeit entscheidet selten über die Antwortzeit.<br>Die tatsächlich wichtigen Gründe: Typannotationen als einzige Quelle der Wahrheit (Validierung + Serialisierung + Doku + Editor-Support), **Dependency Injection** (`Depends`), native Async-Unterstützung, **Standards statt Eigenbau** (OpenAPI, JSON Schema, OAuth2) und eine außergewöhnlich gute Dokumentation.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKHdgTJAP8bBW10utnPr9S" -->
Was leistet **Pydantic** konkret?::
**Validierung** — Typen, Pflichtfelder, Wertebereiche, Formate<br>**Konvertierung/Parsing** — `"2026-08-01"` → echtes `date`, `"42"` → `int` (JSON kennt weder `date` noch `Decimal`, irgendjemand muss übersetzen)<br>**Serialisierung** — der Rückweg: Objekt → JSON, mit Feldauswahl und Aliasnamen<br>**Schema-Erzeugung** — daraus baut FastAPI die OpenAPI-Doku<br>**Konfiguration** — `pydantic-settings` liest und prüft Umgebungsvariablen **beim Start** statt beim ersten Zugriff.<br>Pydantic sitzt genau an der **Systemgrenze**: Alles, was über HTTP hereinkommt, ist erst einmal nicht vertrauenswürdig.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsdq1DSs9YDbtYctvagL38C" -->
Was bedeutet der Grundsatz **„Parse, don't validate"**?::
Man prüft Daten nicht nur und arbeitet danach mit dem rohen `dict` weiter, sondern **verwandelt sie in einen Typ, der ungültige Zustände gar nicht mehr darstellen kann**.<br>Danach ist die Prüfung **im Typ enthalten** — man muss nie wieder `if "nights" in data` schreiben, und alles hinter der Grenze arbeitet mit garantiert gültigen Objekten.<br>Ohne diesen Ansatz steht in jedem Handler dieselbe Prüfkaskade: fehleranfällig, uneinheitlich formuliert und ohne jede Dokumentation.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ydbfK5RjARzaNQMcSIGJ" -->
Was ändert sich zwischen **Pydantic v1 und v2** — und warum ist das praktisch wichtig?::
v2 (2023) hat den Kern in **Rust** neu geschrieben (`pydantic-core`) und ist um ein Vielfaches schneller. Die API hat sich aber geändert:<br>`.dict()` → **`.model_dump()`** · `.json()` → **`.model_dump_json()`** · `parse_obj()` → **`model_validate()`** · `@validator` → **`@field_validator`** · `class Config` → **`model_config = ConfigDict(...)`**<br>Praktisch wichtig, weil sehr viele Blogposts und Stack-Overflow-Antworten noch v1 sind — **die häufigste Quelle für Beispiele, die nicht laufen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsWdlm4RnYXYlCMOYtnBPxE" -->
Warum ist eine synchrone Datenbankabfrage in einer `async def`-Route ein Problem?::
Weil ein blockierender Aufruf im Event Loop **den gesamten Loop lahmlegt** — also alle gleichzeitigen Anfragen, nicht nur die eigene. Klassiker: synchrones SQLAlchemy oder `time.sleep()` in einer `async`-Route.<br>**Die Regel:** Solange die Anwendung synchron arbeitet, schreibt man die Endpunkte als normales **`def`** — FastAPI führt diese automatisch in einem **Threadpool** aus, wo Blockieren unschädlich ist. `async def` nimmt man erst, wenn alles darunter asynchron ist.<br>Häufigster Einsteigerfehler: überall `async def` schreiben, weil es modern aussieht, und den Server damit **langsamer** machen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObssP2gsUQRdO33xYdAA2mX2" -->
Wo leben Uvicorn, FastAPI-Handler, Pydantic-Modelle und SQLAlchemy in der Schichtenarchitektur?::
**Uvicorn** — außerhalb, Betriebsebene; weiß nichts von deinem Code (nur ASGI)<br>**FastAPI-Router/Handler** — **Presentation**; kennt die Service-Schicht (`PricingApp`) und die Pydantic-Modelle<br>**Pydantic-Modelle (DTOs)** — **Presentation/Grenze**; kennen nur sich selbst — **keine ORM-Klassen, keine Sessions**<br>**`PricingApp`, `core/`** — Application/Business<br>**SQLAlchemy, `adapters/`** — Persistence<br>Pydantic-Modelle beschreiben den **Vertrag nach außen**, SQLAlchemy-Modelle das **Speicherlayout** — deshalb sind sie nicht dasselbe.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmAHRapA0DT8pYxV4kpngR" -->
Welche Lebensdauer haben SQLAlchemy-`Engine` und `Session` in einer FastAPI-Anwendung?::
Die **`Engine`** gehört **einmal pro Prozess** (sie ist Fabrik und Connection-Pool zugleich) — angelegt im `lifespan`.<br>Die **`Session`** gehört **pro Anfrage** — bereitgestellt über eine `Depends`-Funktion mit `yield`, die nach der Antwort aufräumt.<br>Der klassische Fehler ist eine **globale Session**: Sie ist nach dem ersten Fehler unbrauchbar und vermischt Daten zwischen gleichzeitigen Anfragen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJ7TfpDrIUcolx5BAfwFnC" -->
Welche Probleme lösten **WSGI (2003)** und **ASGI (2016)** jeweils?::
**WSGI:** Vorher brachte jedes Framework seinen eigenen Serveradapter mit — Framework und Server waren fest verdrahtet. WSGI definiert **eine Schnittstelle dazwischen**, seitdem läuft jedes Framework auf jedem Server.<br>**ASGI:** WSGI ist strukturell synchron und kann kein `async`, keine WebSockets, keine langlebigen Verbindungen. ASGI rüstet genau das nach.<br>Die Trennung Server ↔ Framework ist damit eine **bewusste Errungenschaft**, keine Umständlichkeit — derselbe Gedanke wie `ports.py`: eine Schnittstelle in der Mitte, damit beide Seiten austauschbar bleiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBxkqATbXi78xaTkzK060r" -->
Woraus besteht eine **ASGI-Anwendung** minimal — und was folgt daraus?::
Aus **einer einzigen Funktion mit drei Parametern**: `async def app(scope, receive, send)`.<br>`scope` = dict mit den Metadaten der Verbindung (Methode, Pfad, Header) · `receive` = auf eingehende Ereignisse warten · `send` = Ereignisse zurückschicken (erst Status+Header, dann Body).<br>**Alles, was FastAPI tut, mündet in genau diese Funktion** — Routing, Validierung und Serialisierung sind das, was FastAPI davorschaltet. Deshalb muss **Uvicorn über FastAPI nichts wissen**: Es kennt nur die Signatur.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBLPjIPcFAwAwUbXjFWP7w" -->
Prozesse, Threads, Event Loop — wofür ist welches Modell gut, und was hat die **GIL** damit zu tun?::
**Prozesse** (`--workers 4`) — teuer, aber gut für **CPU-Arbeit**, weil sie die GIL umgehen<br>**Threads** (`def`-Handler im Threadpool) — mittlere Kosten, gut für blockierende Bibliotheken<br>**Event Loop** (`async def`) — sehr billig, gut für **I/O-Warten**<br>Die **GIL** (Global Interpreter Lock) erlaubt pro Prozess immer nur *einem* Thread, Python-Bytecode auszuführen. Deshalb bringen Threads beim **Rechnen** nichts — beim **Warten** schon, denn dabei wird die GIL freigegeben.<br>Merksatz: **Der Event Loop skaliert das Warten, Prozesse skalieren das Rechnen.**
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZ67Hiu7Yx0JDXg0V6u7ie" -->
Was ist der Unterschied zwischen einer **Bibliothek** und einem **Framework**?::
**Bibliothek:** *Du* rufst den Code auf — der Kontrollfluss liegt bei dir (Pydantic, SQLAlchemy, `requests`).<br>**Framework:** *Der Code ruft dich* auf — du registrierst Funktionen und übergibst die Kontrolle (FastAPI, Django, **pytest**).<br>Das Prinzip heißt **Inversion of Control**, umgangssprachlich das **Hollywood-Prinzip: „Don't call us, we'll call you."**<br>Konsequenz: Ein Framework ist eine **Architekturentscheidung** (schwer zu ändern, die ganze Presentation-Schicht nimmt seine Form an), eine Bibliothek tauscht man aus. Und: Alles, was im Handler steht, ist an FastAPI gebunden — was in `core/` steht, nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGPCwl81XjRqE0a1PbVnz2" -->
Warum kann Pydantic mit Typannotationen etwas anfangen, obwohl Python sie zur Laufzeit **ignoriert**?::
Python prüft Annotationen nicht — es legt sie nur als Metadaten in **`__annotations__`** ab (`f.__annotations__` → `{'x': <class 'int'>}`). `def f(x: int)` mit `f("text")` wirft **keinen** Laufzeitfehler.<br>Pydantic und FastAPI **lesen dieses Dictionary zur Laufzeit aus** (`inspect.signature`, `typing.get_type_hints`) und bauen daraus Validierungsregeln, JSON-Schemata und Konvertierungen.<br>Das ist der Trick, auf dem der ganze Stack beruht — und der Grund, warum FastAPI 2018 entstand: Die Sprachfunktion lag seit Python 3.5 brach.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsdSlhr0GZzLrgBWGflOu29" -->
Worin unterscheiden sich **mypy** und **Pydantic**, obwohl beide „Typen prüfen"?::
Sie benutzen **dieselbe Schreibweise für zwei völlig verschiedene Dinge**:<br>**mypy** (statisch) sagt dir **vor** dem Start, dass dein **Code** nicht zusammenpasst.<br>**Pydantic** (Laufzeit) sagt dir **während** des Laufs, dass die **Daten** nicht passen.<br>Beides ist nötig, keines ersetzt das andere: Dein Code kann perfekt typgeprüft sein und trotzdem Müll von außen bekommen. Deshalb findet mypy den `422`-Fall nicht — die eingehende JSON-Nachricht existiert für mypy gar nicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXRSybJY0nYcBKAzPiOSWm" -->
Beschreibe den **Lebenszyklus einer Anfrage** durch den FastAPI-Stack.::
1. TCP-Paket trifft ein → 2. **Uvicorn** parst HTTP und baut den `scope` → 3. ruft `app(scope, receive, send)` → 4. **Starlette**-Middleware von außen nach innen → 5. **Routing** (kein Treffer `404`, falsche Methode `405`) → 6. **FastAPI** löst `Depends` auf (DB-Session öffnen) → 7. Parameter einsammeln → 8. **Pydantic** validiert den Body (Fehler ⇒ sofort `422`) → 9. **dein Handler** → 10. `PricingApp`/Persistenz → 11. **Pydantic** serialisiert per Response-Modell → 12. Response bauen → 13. Middleware rückwärts → 14. Dependencies aufräumen (nach `yield`) → 15. Uvicorn schreibt auf den Socket.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYqZ2u1oQtH9pVeXlvtXUK" -->
Welche drei Einsichten stecken im Anfrage-Lebenszyklus von FastAPI?::
**Schritt 8 ist die Systemgrenze** — ab da arbeitet dein Code nur noch mit gültigen Daten; ungültige Anfragen erreichen die Fachlogik nie.<br>**Die Middleware ist eine Zwiebel**, kein Stapel: außen → innen → Handler → innen → außen. Deshalb kann dieselbe Middleware Dauer messen *und* Header an die Antwort hängen.<br>**Aufräumen (14) kommt nach Serialisieren (11)** — deshalb muss die Session in einer `yield`-Dependency liegen und darf nicht im Handler geschlossen werden: Beim Serialisieren kann noch auf die DB zugegriffen werden (Lazy Loading → der berüchtigte Fehler „Session is closed").
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxLTUjkUFdgPYHuuTPhofP" -->
Wann nimmt man **Django**, wann **FastAPI**, wann **Flask**?::
**Django** — „batteries included" (ORM, Admin, Auth, Migrationen, Templates, Forms): schneller bei Anwendungen mit Datenmodell **und** Backoffice/HTML.<br>**FastAPI** — API-fokussiert, Validierung und Doku eingebaut, Async nativ: die klare Wahl für **reine REST-/JSON-APIs**; ORM, Migrationen und Konfiguration bringt man selbst mit.<br>**Flask** — Microframework, maximale Kontrolle, alles selbst zusammenstellen.<br>Merksatz: **Django ist ein Haus, FastAPI ein sehr guter Werkzeugkasten** — der Trade-off lautet Vollständigkeit gegen Passgenauigkeit.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJubolBabOWYvihRk6Vmjg" -->
Nenne die **Nachteile** des FastAPI-Stacks.::
**Viel Implizites** — woher ein Parameter kommt, entscheidet das Framework; Fehlermeldungen zeigen dann in Framework-Interna.<br>**Bindung ans Framework** — alles im Handler ist FastAPI-Code (deshalb ist „Controller dünn halten" keine Kosmetik).<br>**Kein Standardaufbau** — jedes Projekt erfindet seinen `api/`-Ordnerbaum neu.<br>**Versionsdynamik** — Pydantic v1→v2 war ein Bruch, empfohlene Schreibweisen ändern sich (`Query(None)` → `Annotated`).<br>**`async` verleitet zu Fehlern** — leicht schreibbar, nicht leicht verstehbar.<br>**Lücken im Ökosystem** — kein Admin, keine Migrationen, keine Benutzerverwaltung.
<!-- basicblock-end -->

# Validierung von Request/Response (Pydantic)

<!-- basicblock-start oid="ObspQGv1I64V6Oi7nFVrGRKK" -->
Auf welchen **drei Ebenen** wird eine Anfrage validiert?::
**Protokoll** — „Ist das überhaupt eine lesbare Anfrage?" · prüft Starlette/FastAPI · Fehler **`400`**<br>**Form (Schema)** — „Sind die Felder da, richtig typisiert, im erlaubten Bereich?" · prüft **Pydantic** · Fehler **`422`**<br>**Fachlichkeit** — „Ergibt das inhaltlich Sinn: Gibt es Produkt A1, ist noch etwas frei?" · prüft **`core/`**, nicht Pydantic · Fehler `404`, `409`, `422`<br>Validierung ist damit keine Funktion an einer Stelle, sondern eine **Schichtgrenze**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsrTHHtBmfG8zBOZfpvt1HA" -->
Nach welcher Regel entscheidet man, ob eine Prüfung nach **Pydantic** oder in die **Business-Schicht** gehört?::
**Pydantic prüft alles, was sich ohne Datenbank und ohne Fachwissen entscheiden lässt.** Alles, wofür man nachschauen oder rechnen muss, gehört in die Business-Schicht.<br>Pydantic: `nights` ist positiv und ≤ 365 · `start_date` liegt vor `end_date` (struktureller Vergleich)<br>Business: Produkt `A1` existiert · es sind noch Einheiten frei<br>**Warum das hart sein muss:** Müsste ein Validator die DB anfassen, hätte die Presentation-Schicht eine Abhängigkeit nach unten übersprungen — das DTO wäre nicht mehr isoliert testbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObseplExpXEMrfPXuNa3kmee" -->
Was ist ein **DTO**, und wodurch unterscheidet es sich von einem Domain-Objekt?::
**Data Transfer Object** — ein Objekt, dessen einziger Zweck es ist, **Daten über eine Grenze zu transportieren**. Es enthält Felder und **keine Fachlogik**. Begriff von Martin Fowler (PoEAA), ursprünglich, um bei teuren Fernaufrufen alles Nötige in *einem* Objekt zu übertragen.<br>**Unterschied zum Domain-Objekt:** `PriceableUnit` hat Regeln, Invarianten und berechnete Eigenschaften (`occupancy`, `total_capacity`) — es **bedeutet** etwas. Ein DTO bedeutet nichts, es **transportiert** nur.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHfdHtP2UpoY8Val935AHz" -->
Nenne die vier Gründe, warum man **DTOs** benutzt.::
1. **Entkopplung** — der API-Vertrag ist unabhängig von den internen Klassen; `domain.py` und Schema bleiben änderbar, ohne Clients zu brechen.<br>2. **Kontrolle, was rausgeht** — was im DTO nicht steht, kann nicht versehentlich veröffentlicht werden (Fremdschlüssel, Passwort-Hashes, `deleted_at`). Ein **Sicherheitsmerkmal**.<br>3. **Kontrolle, was reinkommt** — der Client darf `id` und `status` gar nicht erst setzen (statt „wird hoffentlich ignoriert").<br>4. **Passender Zuschnitt** — Felder aus mehreren Tabellen zusammenziehen, berechnete Werte ergänzen, Interna weglassen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsgs0te8JsUhtPBcgoeI36L" -->
Mit welcher Bibliothek setzt man DTOs in Python um — und welche Alternativen gibt es?::
**Pydantic** (`BaseModel`) — weil man Validierung, Serialisierung und Schema-Erzeugung geschenkt bekommt und FastAPI daraus zusätzlich die OpenAPI-Doku baut. Für eine API praktisch alternativlos.<br>Reine Datencontainer **ohne** Validierung: **`dataclasses`** (Standardbibliothek), **`attrs`**, `NamedTuple`, `TypedDict`, **`msgspec`** (schneller und schmaler).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsQ3u4LZKfP2M91TQPfiNwx" -->
Warum reicht **ein einziges** `Booking`-Modell für Create, Update und Read nicht?::
Weil dieselbe Entität je nach Richtung **verschiedene Felder** hat:<br>`id` — nicht bei Create (Server vergibt sie), nicht bei Update (steht im Pfad), aber bei Read<br>`status` — nicht bei Create (Server setzt `"open"`), aber bei Update und Read<br>`created_at` — nur bei Read<br>Ein einziges Modell müsste **alle Felder optional** machen, damit alle Fälle passen — und würde damit **nichts mehr validieren**. Der Vertrag wäre „schick irgendwas".
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsk5CtcNpwkhuiC2PD3M0b9" -->
Inwiefern ist **`response_model`** ein Sicherheitsmerkmal?::
FastAPI **filtert die Antwort auf die Felder des Response-Modells**. Gibt ein Handler versehentlich ein vollständiges ORM-Objekt zurück, wird trotzdem nur ausgeliefert, was im Modell steht.<br>Ein Passwort-Hash oder ein interner Fremdschlüssel, der im DTO fehlt, **verlässt das System nicht — auch nicht durch ein Versehen.** Die Sicherheit hängt damit nicht an der Sorgfalt in jedem einzelnen Handler.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsU2UuXamnCLi1efumwkd6f" -->
Warum gehört **`extra="forbid"`** an Eingangsmodelle?::
Standardmäßig **ignoriert Pydantic unbekannte Felder stillschweigend** (`extra="ignore"`). Bei einem Request-Modell ist das die falsche Voreinstellung:<br>Schickt ein Client `nigths: 3` (Tippfehler), würde er wortlos eine Buchung mit **Default-Werten** anlegen statt einen **`422`** zu bekommen. Der Fehler fällt erst auf, wenn die Daten falsch sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGvp3CnZfyYgSn7yb7CVV8" -->
Was muss man bei `PATCH`-Modellen zusätzlich unterscheiden können?::
Zwischen **„Feld nicht geschickt"** und **„Feld ausdrücklich auf `null` gesetzt"** — beides sieht in einem Modell mit optionalen Feldern zunächst gleich aus.<br>Pydantic löst das über **`model_dump(exclude_unset=True)`**: Nur tatsächlich gesetzte Felder landen im Ergebnis.<br>Ohne diese Unterscheidung überschreibt ein `PATCH` unbeabsichtigt Felder mit `None`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZEyCHwcunCmALTVhMzcqi" -->
Warum definiert man Objekte **doppelt** — einmal in SQLAlchemy, einmal in Pydantic?::
Weil sie **verschiedene Fragen beantworten und sich aus verschiedenen Gründen ändern**:<br>**SQLAlchemy-Modell** — wie Daten **gespeichert** werden; Vertragspartner ist die **Datenbank**; ändert sich bei Schema-Optimierungen (Index, Normalisierung); Schicht: Persistence.<br>**Pydantic-Modell (DTO)** — wie Daten **übertragen** werden; Vertragspartner sind die **Clients**; ändert sich bei fachlichem Bedarf; Schicht: Presentation.<br>Merksatz: Es sind nicht zwei Definitionen desselben Dings, sondern **ein Ding und sein Vertrag**. Dass sie sich am Anfang ähneln, ist ein Zufall des ersten Tages.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbpWeaPC5YOAWHjzaj4VCn" -->
Was verliert man, wenn man **ORM-Klassen direkt als API-Modelle** benutzt?::
**Jede Schema-Änderung wird ein Breaking Change** — eine Spalte umzubenennen bricht alle Clients; das Schema ist faktisch eingefroren.<br>**Interna werden veröffentlicht** — Fremdschlüssel, Passwort-Hashes, Löschmarkierungen sind standardmäßig sichtbar.<br>**Mass Assignment** — der Client kann Felder setzen, die ihm nicht gehören (`id`, `status`, `created_at`).<br>**Der Zuschnitt passt nicht** — Clients wollen `product_id`/`calendar_date`, nicht `capacity_id`.<br>**Lazy Loading schlägt durch** — beim Serialisieren werden unbeabsichtigt Nachladeabfragen ausgelöst („Session is closed").
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsecFWrHWubs2CFQEXaeYeJ" -->
Was ist **Mass Assignment**?::
Eine klassische Sicherheitslücke: Der Client schickt Felder mit, die er gar nicht setzen dürfte (`id`, `status`, `role`, `is_admin`), und sie werden **ungefiltert** auf das interne Objekt übertragen.<br>Sie entsteht typischerweise dadurch, dass man das Eingangsobjekt direkt aus dem Request-Body erzeugt, statt ein **eigenes Eingangs-DTO** zu definieren, das nur die erlaubten Felder enthält.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXUMMeR5joiomiN3q2kHYW" -->
Was ist **SQLModel**, und was ist der Preis dafür?::
Eine Bibliothek vom FastAPI-Autor, bei der **eine Klasse gleichzeitig SQLAlchemy-Tabelle und Pydantic-Modell** ist — sie spart die doppelte Definition.<br>**Der Preis ist die Kopplung:** Persistenz und API-Vertrag hängen wieder aneinander, mit allen Folgen (Schema-Änderung = Breaking Change, Interna sichtbar).<br>**Ehrliche Einordnung:** Bei einer kleinen internen CRUD-App mit genau einem Client *ist* die doppelte Definition Overhead und SQLModel vernünftig. In einem Projekt, in dem es **um** Schichten geht, ist die Trennung die richtige Wahl.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs2W1HiyOLQxtzaCgizAv5O" -->
Welche Schicht darf **Pydantic auf keinen Fall kennen** — und warum?::
**`core/` und `domain.py`.** Sonst hängt die Fachlogik an einer Presentation-Bibliothek, und der Batch-Lauf müsste FastAPIs Abhängigkeiten mitschleppen.<br>Der Import zeigt immer nur **nach innen**: JSON → DTO (Pydantic) → Domain → ORM → DB, und zurück.<br>Umgekehrt ist es vertretbar, die Domain-Ebene beim reinen Durchreichen gelegentlich zu überspringen — solange es eine **bewusste** Entscheidung ist (Layer Bridging). Was nie passieren darf, ist ein Pydantic-Modell in `core/`, weil es gerade schneller ging.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTN2RY9AnhaBnedQBfD4HY" -->
Wie sieht eine `422`-Antwort von FastAPI aus, und warum ist dieses Format gut?::
Eine **Liste strukturierter Einzelfehler**: `{"detail": [{"type": "greater_than", "loc": ["body", "nights"], "msg": "Input should be greater than 0", "input": -3}]}`<br>**`loc`** ist der Pfad zum fehlerhaften Feld (`body`/`query`/`path` + Feldname) — ein Client kann den Fehler damit **am richtigen Formularfeld** anzeigen.<br>Es werden **alle** Fehler auf einmal gemeldet, nicht nur der erste — der Nutzer korrigiert einmal statt fünfmal.<br>Eigene Texte oder das RFC-9457-Format bekommt man über einen zentralen Handler für `RequestValidationError`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsq8VggOMBSWKPN1wfzewjw" -->
Warum sollte eine Regel wie `booked_units >= 0` **nicht zusätzlich** in Pydantic stehen, wenn `domain.py` sie schon prüft?::
Weil dieselbe Regel dann an **zwei Stellen** steht, die auseinanderlaufen können — es klingt gründlich und ist gefährlich.<br>Saubere Aufteilung: **Pydantic prüft das Format** (ist es überhaupt ein `int`?), **die Domain prüft die Regel** (darf der Wert das sein?).<br>Wenn die Fachvalidierung erst in `core/` zuschlägt, muss der **zentrale Exception-Handler** die `DomainError` auf einen Statuscode abbilden (`422` bzw. `409`).
<!-- basicblock-end -->

# Swagger & OpenAPI

<!-- basicblock-start oid="ObsMnTBQIK6QLmdyWLxwcSj8" -->
Was ist der Unterschied zwischen **OpenAPI** und **Swagger**?::
**OpenAPI** ist eine **Spezifikation** — ein Dateiformat (JSON/YAML), das beschreibt, wie ein API-Beschreibungsdokument aufgebaut ist. *Die Grammatik.*<br>**Swagger** ist eine **Werkzeugfamilie** — Swagger UI, Swagger Editor, Swagger Codegen. *Die Programme, die die Grammatik lesen.*<br>**Herkunft der Verwirrung:** Swagger war ursprünglich beides. **2015 wurde die Spezifikation an die Linux Foundation gespendet** und in *OpenAPI Specification* umbenannt; die Werkzeuge behielten den Namen.<br>Saubere Sprechweise: **Das Dokument heißt OpenAPI, die Oberfläche heißt Swagger UI.**
<!-- basicblock-end -->

<!-- basicblock-start oid="Obslz5tYJnDpSYDTpEq1VifL" -->
Welche Hauptabschnitte hat ein **OpenAPI-Dokument**?::
`openapi:` — Versionsangabe der Spezifikation (aktuell 3.0.x / 3.1.x; FastAPI erzeugt 3.1)<br>`info:` — Titel, Version, Beschreibung, Kontakt, Lizenz<br>`servers:` — unter welchen Basis-URLs die API erreichbar ist<br>`paths:` — **das Herzstück**: alle Endpunkte mit Methoden, `parameters`, `requestBody`, `responses` pro Statuscode<br>`components:` — `schemas` (die Datenmodelle, einmal definiert und per `$ref` referenziert) und `securitySchemes`<br>Die Struktur bildet exakt die Begriffe dieses Kapitels ab — sie ist deren formale Fassung.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1yTSOpiKAki3H7PLXIz5j" -->
Warum passt **Pydantic verlustfrei** in den `components.schemas`-Teil eines OpenAPI-Dokuments?::
Weil `components.schemas` **JSON Schema** benutzt — und Pydantic genau das **erzeugt**. In OpenAPI **3.1** ist die Kompatibilität sogar vollständig (in 3.0 gab es noch kleine Abweichungen).<br>Deshalb entsteht die Modelldokumentation ohne jede Zusatzarbeit: Dasselbe Modell, das validiert, beschreibt sich auch selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsEROkNWigRynBUrBzANt2y" -->
Woraus erzeugt FastAPI die Inhalte des OpenAPI-Dokuments?::
Pfad + Methode ← dem **Dekorator** · `tags` ← `APIRouter(tags=[...])` · `summary` ← dem **Funktionsnamen** (oder explizit) · **`description` ← dem Docstring** der Handler-Funktion (Markdown!) · `parameters` ← den Funktionsparametern samt `Path`/`Query`-Constraints · `requestBody` ← dem Pydantic-Eingangsmodell · `responses["200"]` ← `response_model` bzw. Rückgabetyp · Erfolgscode ← `status_code=201` · `components.schemas` ← allen benutzten Pydantic-Modellen · Feldbeschreibungen ← `Field(description=..., examples=[...])`.<br>Man verbessert die Doku also, indem man den **Code** verbessert — das ist der Kern des Code-First-Versprechens.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnEBIpX90EZmfAWiUAewFh" -->
Welche drei Endpunkte stellt FastAPI automatisch bereit?::
**`/openapi.json`** — das rohe Dokument, für Werkzeuge<br>**`/docs`** — **Swagger UI**, zum **Ausprobieren** (interaktiv, „Try it out" schickt echte Anfragen, zeigt darunter den fertigen `curl`-Befehl)<br>**`/redoc`** — **ReDoc**, zum **Lesen** (dreispaltig, ruhig, gut für lange APIs; kann keine Anfragen ausführen)<br>Beide Oberflächen zeigen dasselbe Dokument mit entgegengesetzter Absicht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiGxBI5VAVZ3lCVQJJ29Ng" -->
Was steht in einem OpenAPI-Dokument systematisch **nicht**?::
Generierte Doku ist **strukturell korrekt, aber nicht zwangsläufig verständlich**. Es fehlt:<br>· wie man **anfängt** („hol dir einen Token, dann …")<br>· **warum** es einen Endpunkt gibt und wann man ihn benutzt<br>· fachliche **Zusammenhänge** zwischen mehreren Aufrufen<br>· was die Werte **bedeuten** (`status: "open"` — und dann?)<br>Merksatz: **Eine API-Referenz ist eine Landkarte, kein Reiseführer.** Beides wird gebraucht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmBcFNFyo7bksNbPeLpbRa" -->
Welches HTTP-Werkzeug nimmt man wann bei der Entwicklung einer REST-API?::
**Swagger UI** (`/docs`) — sofort da, kennt das Schema, keine Einrichtung; aber keine gespeicherten Abläufe und keine Variablen → **während des Entwickelns**, Endpunkt sofort ausprobieren.<br>**Postman / Insomnia / Bruno** — Collections, Umgebungen (`{{base_url}}`), Variablen, Aufrufketten, Tests, Teilen → **wiederkehrende Abläufe** (Login → anlegen → abfragen).<br>**`curl` / HTTPie** — reproduzierbar, skriptbar, überall vorhanden → **Ticket, README, CI**.<br>**`TestClient` (pytest)** — automatisiert, läuft bei jeder Änderung → **Regression**, das Einzige, was dauerhaft schützt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0Jlsj1IX5xHUf6y9s0cUF" -->
Wie sieht der typische Arbeitsablauf mit HTTP-Clients aus?::
1. Endpunkt schreiben → **Swagger UI** öffnen, „Try it out", Fehler sehen, korrigieren — der schnellste Kreislauf.<br>2. Funktioniert es: den **`curl`-Befehl aus der Swagger UI kopieren** — für Notiz, Ticket, Kollegen.<br>3. Wird der Ablauf wiederkehrend: als **Collection** in Postman/Bruno ablegen — dafür **`/openapi.json` importieren statt abzutippen**.<br>4. Was sich bewährt hat, wandert in einen **automatisierten Test**.<br>Grundsatz: **Die Clients sind zum Erkunden da, nicht zum Absichern.** Wer dieselbe Anfrage zum dritten Mal von Hand schickt, gehört mit ihr in `tests/`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObswaSlFRwcCQsCuB3TqHGaS" -->
Was kann man mit einem OpenAPI-Dokument **maschinell** anfangen?::
**Client-Code generieren** (`openapi-generator`) — fertige Client-Bibliotheken für Python, TypeScript, Java …; ein Frontend-Team programmiert gegen typisierte Funktionen statt gegen URLs.<br>**Mock-Server** (Prism) — sofort eine Fake-API, gegen die andere Teams entwickeln, bevor die Implementierung fertig ist (der praktische Kern von Design-First).<br>**Linting** (**Spectral**) — Style-Regeln automatisch prüfen; macht aus Naming-Konventionen erzwingbare Regeln statt Bitten.<br>**Breaking Changes erkennen** (`oasdiff`) — zwei Versionen vergleichen und die CI rot werden lassen, wenn ein Feld verschwindet.<br>**Import** in Postman/Insomnia/Bruno und API-Gateways.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3K43wQDoVy63hIrfR5bL4" -->
Warum ist eine unübersichtliche Swagger UI ein Hinweis auf ein **Entwurfsproblem**?::
Weil die Oberfläche nichts hinzufügt — sie zeigt nur, was im Code steht. **Wenn die Swagger UI unübersichtlich ist, ist die API unübersichtlich.** Sie ist ein ehrlicher Spiegel des Entwurfs.<br>Typische Befunde und ihre Ursachen: fehlende Gruppierung → keine sinnvollen **Router/`tags`** · generische Namen → schlechte Funktionsnamen · Endpunkte ohne Beschreibung → fehlende Docstrings · nur der Erfolgsfall dokumentiert → **Fehlerfälle nicht deklariert** (`responses={404: ...}`).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsjxJRh4Rct0xmnIT7b05K6" -->
Was gibt `/docs` technisch zurück — eine statische HTML-Seite?::
HTML **ja**, statische Datei **nein**: FastAPI ruft `get_swagger_ui_html()` auf, eine **Python-Funktion**, die ~25 Zeilen HTML als String baut. Die Seite ist **fast leer** — ein `<div id="swagger-ui">`, zwei **CDN**-Verweise (swagger-ui.css/.js) und `SwaggerUIBundle({url: "/openapi.json"})`.<br>Die Kette: Browser holt `/docs` → lädt CSS/JS vom CDN → das JS ruft **`/openapi.json`** auf → FastAPI erzeugt das Dokument aus Routen und Pydantic-Modellen (gecacht in `app.openapi_schema`) → das JS rendert.<br>**Kernaussage: `/docs` ist ein Client deiner eigenen API.** Die Substanz ist JSON, nicht HTML. `/redoc` = dasselbe Dokument, anderer Renderer.<br>Folgen: Das **Internet braucht der Browser**, nicht der Server (isolierter Container → `/docs` geht trotzdem; offline → weiße Seite). Und `/docs`/`/openapi.json` sind **standardmäßig öffentlich**, auch wenn alle Router Auth verlangen → in Produktion `docs_url=None` o. Ä.
<!-- basicblock-end -->
