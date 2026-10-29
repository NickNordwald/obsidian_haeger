Sammlung von Fäden, die in der [[1.5 REST APIs & Layered Architecture - Vertiefung|Vertiefung]] bewusst nur gestreift oder ganz weggelassen wurden — als Landkarte für einen späteren, tieferen Durchgang. **Kein Pflichtstoff für das Kapitel.**

Legende: 🔴 lohnt sich sehr / berufsrelevant · 🟡 gut zu wissen · ⚪ Bonus, wenn Zeit und Lust da sind
Quellen zu den meisten Punkten: [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]].

---

# n-tier Architektur

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Package by Layer vs. Package by Feature** — Der Ordnerbaum kann nach *technischer* Schicht (`controllers/`, `services/`, `repositories/`) oder nach *fachlichem* Feature (`pricing/`, `booking/`, jeweils mit eigenen Schichten darin) geschnitten sein. Die Vertiefung setzt stillschweigend Package-by-Layer voraus; Package-by-Feature („vertikale Schnitte") skaliert bei wachsenden Systemen oft besser. Stichwort auch: *Screaming Architecture* (Robert C. Martin).
- 🔴 **Dependency Inversion vs. Dependency Injection** — In der Vertiefung nur als Einzeiler erwähnt. Wert der Vertiefung: Warum ist DI das *Mittel* und DIP das *Prinzip*? Wo liegt der Unterschied zwischen Constructor Injection (unser `PricingApp.__init__`), Setter Injection und einem DI-Container? Braucht Python überhaupt einen Container, wenn `Depends` in FastAPI existiert?
- 🔴 **Transaktionsgrenzen über Schichten hinweg** — Die praktisch wichtigste offene Frage: Wer öffnet und committet die Datenbank-Transaktion — Controller, Service oder Repository? Standardantwort ist die Serviceschicht („Unit of Work"-Muster), weil nur sie den fachlichen Anwendungsfall kennt. Cosmic Python hat dazu ein eigenes Kapitel.
- 🟡 **Mapping zwischen Schichten** — Jede Schichtgrenze bedeutet potenziell ein eigenes Modell: ORM-Entity ↔ Domain-Objekt ↔ DTO. Wann lohnt sich diese Dreifach-Definition, wann ist sie Overengineering? (Hängt eng an der Kontrollfrage „müssen wir Objekte wirklich doppelt definieren?" aus dem Python-Teil des Leitfadens.)
- 🟡 **Anemic Domain Model** — Fowlers Kritik an Domain-Objekten, die nur Daten halten, während die gesamte Logik in Services liegt. Direkt relevant für unser Projekt: Ist `domain.py` anämisch? Ist das schlimm? (Es gibt starke Gegenpositionen.)
- 🟡 **Sinkhole-Anti-Pattern quantifiziert** — In der Vertiefung nur als Begriff genannt. Richards' Faustregel: Wenn mehr als ~20 % der Anfragen eine Schicht ohne Beitrag durchlaufen, stimmt der Schnitt nicht.
- 🟡 **Architekturmerkmale („-ilities")** — Warum man Architektur überhaupt bewertet: Performance, Skalierbarkeit, Testbarkeit, Deploybarkeit, Modifizierbarkeit. Die Schichtenarchitektur schneidet bei manchen davon systematisch schlecht ab — das ist der eigentliche Grund, warum es andere Stile gibt.
- ⚪ **Conway's Law** — Die These, dass die Systemarchitektur die Kommunikationsstruktur der Organisation abbildet. Erklärt überraschend viel darüber, warum Architekturen in Firmen so aussehen, wie sie aussehen.

## Explizit out of scope laut Leitfaden (später relevant)

- 🔴 **Hexagonale Architektur / Ports & Adapters** — Steckt faktisch schon in unserem `ports.py`, ohne dass der Leitfaden es benennt. Kernidee: Nicht „oben/unten", sondern „innen/außen" — der Kern kennt niemanden, alle Abhängigkeiten zeigen nach innen. Verwandt: Onion Architecture, Clean Architecture. Alle drei sind im Kern dasselbe mit anderem Vokabular.
- 🟡 **Visualisierung von Architekturen** — Das C4-Modell (Context, Container, Component, Code) von Simon Brown ist der aktuelle Praxisstandard; dazu Diagrams-as-Code-Werkzeuge (Structurizr, Mermaid, PlantUML). Mermaid rendert Obsidian nativ — für dieses Vault also besonders praktisch.
- 🟡 **Architecture Decision Records (ADR)** — Kurze, versionierte Textdokumente, die *eine* Architekturentscheidung samt Kontext und Konsequenzen festhalten. Passt inhaltlich genau zu den `Decisions/`-Ordnern, die dieses Vault für gewachsene Projekte unter `Projects/` vorsieht.
- ⚪ **Modularer Monolith vs. Microservices** — Die Frage „ein Deployment oder viele" ist die Tier-Seite der Layer/Tier-Unterscheidung. Wichtige Gegenposition zum Microservice-Hype: der modulare Monolith.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Ist `PricingApp` wirklich eine Serviceschicht — oder eher ein *Application Service*, der zu viel auf einmal tut (`_compute()` lädt, baut und rechnet)?
- Wo gehört das Logging-Setup aus [[1.4 Logging & Exceptionhandling - Vertiefung|1.4]] architektonisch hin? (Antwort vorweg: Composition Root, nicht Bibliothekscode — aber warum genau?)
- Ist `ingest.py` mit seinen 293 Zeilen noch ein Adapter, oder hat sich dort Fachlogik eingeschlichen, die eigentlich in `core/` gehört?
- Wenn die REST API dazukommt: Braucht das Projekt zwei Composition Roots (`main.py` für den Batch-Lauf, ein zweiter für die API)? Wie vermeidet man Duplikation?

---

# Client/Server

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Nebenläufigkeit im Server** — In der Vertiefung nur als „Anfragen kommen gleichzeitig" angedeutet. Der ganze Themenkomplex dahinter: Was passiert, wenn zwei Requests gleichzeitig dieselben Daten lesen und schreiben? Race Conditions, Locking (optimistisch vs. pessimistisch), Isolation Levels in der Datenbank. Für unser `_compute()` sehr real.
- 🔴 **async/await und der Event Loop in Python** — Der Grund, warum FastAPI überhaupt schnell ist, und die häufigste Fehlerquelle bei Einsteigern: eine blockierende (synchrone) DB-Abfrage in einer `async def`-Route legt den gesamten Server lahm. Verwandt: warum SQLAlchemy zwei Varianten hat (sync/async). Wird beim Stichwort FastAPI kurz auftauchen, verdient aber einen eigenen Durchgang.
- 🔴 **Timeouts, Retries und Backoff** — Die Vertiefung erklärt, *warum* ein Timeout mehrdeutig ist, aber nicht, was man dagegen tut: Exponential Backoff, Jitter, Retry-Budget, Circuit Breaker. Sehr berufsrelevant, sobald man fremde APIs oder Datenquellen anzapft. Stichwort auch: „Retry Storm".
- 🟡 **Wie Verbindungen technisch entstehen** — TCP-Handshake, TLS-Handshake, Keep-Alive, Connection Pooling. Erklärt, warum die erste Anfrage an einen Server oft deutlich langsamer ist als die zehnte, und warum ein DB-Connection-Pool existiert. (Hussein Nassers YouTube-Kanal aus den Quellen ist hierfür genau richtig.)
- 🟡 **Alternativen zu Request/Response** — Polling, Long Polling, Server-Sent Events, WebSockets, Message Queues (RabbitMQ/Kafka), Publish/Subscribe. Als Data Engineer sind Queues und Kafka mittelfristig relevanter als REST.
- 🟡 **Load Balancing & Sticky Sessions** — Der praktische Grund, warum Statuslosigkeit sich auszahlt. Wie verteilt ein Load Balancer Anfragen (Round Robin, Least Connections), und was genau bricht, wenn der Server doch Sitzungszustand im Speicher hält?
- 🟡 **Sicherheit an der Serverschnittstelle** — Authentifizierung vs. Autorisierung, API Keys vs. OAuth2/JWT, Rate Limiting, CORS. Der Leitfaden lässt das aus, aber jede API, die tatsächlich erreichbar ist, braucht davon etwas.
- ⚪ **Idempotency Keys** — Die industrietaugliche Lösung des Timeout-Problems bei nicht-idempotenten Operationen (Stripe hat das Muster bekannt gemacht): Der Client schickt eine selbst erzeugte ID mit, der Server erkennt Wiederholungen daran.
- ⚪ **Peer-to-Peer als Gegenmodell** — Systeme ohne feste Rollenverteilung; nützlich zum Kontrast, um zu verstehen, was Client/Server eigentlich vereinfacht.
- ⚪ **CAP-Theorem** — Sobald Zustand über mehrere Knoten verteilt ist: Consistency, Availability, Partition Tolerance — zwei von drei. Der theoretische Unterbau vieler Datenbank-Entscheidungen.

## Fragen zum eigenen Projekt, die offen geblieben sind

- `_compute()` lädt bei jedem Aufruf alle Daten neu. Als Batch egal — als HTTP-Endpunkt: Caching, Vorberechnung oder einfach akzeptieren? Woran macht man diese Entscheidung fest?
- Der Batch-Lauf und der API-Server brauchen dieselben Objekte, aber mit unterschiedlicher Lebensdauer (Batch: einmal; Server: pro Request oder pro Prozess?). Was davon darf ein Singleton sein — Engine, Session, `PricingApp`?
- Was passiert eigentlich, wenn Postgres während einer laufenden HTTP-Anfrage nicht erreichbar ist? Welchen Statuscode bekommt der Client, und was steht dann im Log?

---

# Softwarearchitektur

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Architekturmerkmale konkret messbar machen** — Die Vertiefung listet die „-ilities" nur auf. Der eigentliche Handwerksteil fehlt: Ein Merkmal ist erst brauchbar, wenn es *messbar* formuliert ist („99,5 % Verfügbarkeit im Monat", „p95-Antwortzeit < 200 ms") statt als Wunsch („soll schnell sein"). Stichworte: Service Level Objectives (SLO), Fitness Functions (automatisierte Tests *für Architekturmerkmale*, z. B. „kein Import von `adapters` in `api`" als CI-Check — das könntest du in deinem Projekt tatsächlich bauen).
- 🔴 **ADRs praktisch schreiben** — In der Vertiefung nur als Begriff. Zum Vertiefen: das Standardformat von Michael Nygard (Titel, Status, Kontext, Entscheidung, Konsequenzen), Werkzeuge wie `adr-tools`, und die Frage, wo ADRs leben (im Repo neben dem Code vs. im Wiki — starke Argumente für „im Repo").
- 🟡 **Kopplung und Kohäsion** — Das eigentliche Vokabular, mit dem man Struktur *bewertet*, statt sie nur zu beschreiben. Afferente vs. efferente Kopplung, Instabilitätsmetrik, „Connascence" als feingliedrigere Alternative zum groben Begriff Kopplung. Hierdurch wird aus Bauchgefühl ein Argument.
- 🟡 **Technische Schulden** — Der ökonomische Blick auf Architektur: Warum eine bewusst eingegangene Abkürzung etwas anderes ist als Schlamperei, und wie man Zinsen darauf sichtbar macht. Fowlers Technical Debt Quadrant ist der Einstieg.
- 🟡 **Nicht-funktionale Anforderungen im Standard** — ISO/IEC 25010 definiert eine vollständige Taxonomie der Softwarequalität. Trocken, aber nützlich als Checkliste, wenn man in einem Projekt fragt „woran haben wir nicht gedacht?".
- 🟡 **Der Architekt als Rolle** — Was macht diese Person eigentlich den ganzen Tag? Technische Breite statt Tiefe, Entscheidungen treffen *und* begründen, Architektur „durchsetzen" ohne Weisungsbefugnis. Relevant, weil es die realistische nächste Karrierestufe nach Senior Engineer ist.
- ⚪ **Evolutionary Architecture** — Die Gegenposition zum großen Vorabentwurf: Architektur so bauen, dass geführte, inkrementelle Veränderung möglich bleibt. Von denselben Autoren (Ford/Parsons/Kua).
- ⚪ **Der „Big Ball of Mud"** — Das meistverbreitete Architekturmuster der Welt, halb ironisch beschrieben in einem berühmten Paper von Foote & Yoder. Lesenswert, weil es erklärt, *warum* Systeme so enden — die Gründe sind meist rational, nicht dumm.

## Zum Kontrollfragen-Thema Code-First vs. Design-First

- 🔴 **Contract Testing** — Wie stellt man sicher, dass Client und Server nicht auseinanderlaufen, wenn beide unabhängig deployt werden? Stichwort Consumer-Driven Contracts (Pact). Die praktische Antwort auf das Problem, das Design-First adressiert.
- 🟡 **API-Versionierung und Breaking Changes** — Was ist überhaupt ein Breaking Change? Versionierung per URL-Pfad (`/v1/`), Header oder Media Type — und warum die meisten großen Anbieter widerwillig den URL-Pfad nehmen. Direkt anschlussfähig an den in der Vertiefung genannten „Preis von Code-First".
- 🟡 **Spec-Linting** — Werkzeuge wie Spectral prüfen ein OpenAPI-Dokument automatisch gegen Style-Regeln (z. B. die Zalando-Guidelines). So wird aus einem Design-First-Prozess etwas Erzwingbares statt einer Bitte.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Welche Architekturmerkmale sind für dein Pricing-Projekt eigentlich *tatsächlich* gefordert — und von wem? (Der Leitfaden sagt es nicht; im echten Projekt müsste man fragen. Genau das ist die Diskussion, die die Aufgabe mit den Trainern vorsieht.)
- Ließe sich die Architekturentscheidung „die API-Schicht kennt `adapters/` nicht" als automatisierter Test formulieren, der bei Verstoß rot wird?
- Wenn du drei ADRs für den bisherigen Projektstand nachträglich schreiben müsstest — welche drei Entscheidungen wären es? (Kandidaten: Ports & Adapters statt direktem DB-Zugriff, SQLAlchemy als ORM, Logging-Konfiguration ausschließlich in der Composition Root.)

---

# Architekturmuster

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Entwurfsmuster systematisch nachholen** — Die Vertiefung nennt Strategy, Factory, Observer, Adapter und Repository nur als Namen. Für den Alltag lohnen sich vor allem: **Strategy** (steckt in deinen Ports), **Adapter** (steckt in `adapters/`), **Factory** (Objekterzeugung in der Composition Root), **Decorator** (Python-Dekoratoren sind genau das — und FastAPI-Routen werden so registriert), **Observer** (die Grundidee hinter Event-Driven). Refactoring Guru aus den Quellen ist dafür die beste Anlaufstelle.
- 🔴 **Pipeline-Muster vertiefen** — Für dein Berufsfeld das wichtigste Muster überhaupt und in der Vertiefung nur in fünf Zeilen abgehandelt. Weiterführend: Batch vs. Streaming, Idempotenz und Wiederaufsetzbarkeit einzelner Schritte, Backpressure, Orchestrierung (Airflow, Dagster, Prefect) vs. Choreografie, DAGs als Modell. Das ist faktisch der Stoff der späteren Traineeship-Kapitel.
- 🟡 **CQRS (Command Query Responsibility Segregation)** — Lese- und Schreibpfad trennen, weil sie unterschiedliche Anforderungen haben. Der saubere, benannte Ausweg aus dem Layer-Bridging-Dilemma aus dem ersten Stichwort: Der Lesepfad darf die Business-Schicht überspringen, *weil* er ein anderes Modell benutzt.
- 🟡 **Saga-Muster** — Wie führt man einen fachlichen Vorgang über mehrere Dienste hinweg aus, wenn es keine gemeinsame Transaktion mehr gibt? Kompensierende Aktionen statt Rollback. Der Preis von Microservices, konkret gemacht.
- 🟡 **Circuit Breaker, Bulkhead, Retry** — Die „Stabilitätsmuster" aus *Release It!* (Michael Nygard). Was tun, wenn ein aufgerufener Dienst langsam oder tot ist, ohne das eigene System mitzureißen? Sehr praxisnah, sobald man fremde APIs anzapft.
- 🟡 **Backends for Frontends (BFF)** — Ein eigener API-Zuschnitt pro Client-Typ (Web, Mobile, Partner). Die pragmatische Antwort auf „ein Endpunkt kann nicht allen passen" und eine Alternative zu GraphQL.
- ⚪ **Serverless / FaaS** — Als weiteres Muster in der Liste; interessant vor allem wegen der Trade-offs (Kaltstart, kein Zustand, Kostenmodell nach Aufruf).
- ⚪ **Muster-Kritik** — Es gibt eine ernstzunehmende Gegenposition, dass Entwurfsmuster teilweise nur fehlende Sprachfeatures kompensieren (in Python braucht man für Strategy oft keine Klassenhierarchie, sondern übergibt eine Funktion). Peter Norvigs Vortrag „Design Patterns in Dynamic Languages" ist der klassische Text dazu — nützlich, um Muster nicht dogmatisch anzuwenden.

## Explizit out of scope laut Leitfaden (später relevant)

- 🟡 **GraphQL** — Der prominenteste Gegenentwurf zu REST: Der Client bestimmt, welche Felder er bekommt. Löst Over-/Under-Fetching, handelt sich dafür Caching- und Komplexitätsprobleme ein.
- 🟡 **gRPC** — Binäres RPC-Protokoll mit Protobuf-Schema, Design-First per Definition. Im Data-Engineering-Umfeld und zwischen internen Diensten sehr verbreitet, weil deutlich schneller als JSON über HTTP.
- ⚪ **Vergleichsraster REST vs. GraphQL vs. gRPC vs. Messaging** — Wann nimmt man was? Eine Tabelle, die man einmal selbst aufstellen sollte; sie schärft das Verständnis für REST mehr als jede weitere REST-Lektüre.

## Fragen zum eigenen Projekt, die offen geblieben sind

- `PricingApp._compute()` ist faktisch eine Pipeline aus vier Schritten. Wäre es sinnvoll, das *explizit* zu machen (Schritte als eigene Objekte, komponierbar) — oder wäre das Overengineering bei vier Schritten?
- `ports.py` nutzt `Protocol` (strukturelle Typisierung) statt abstrakter Basisklassen (nominale Typisierung). Was genau ist der Unterschied, und was wäre der Preis der jeweils anderen Wahl?
- Kommt in deinem Projekt später ein zweiter Lesepfad hinzu (z. B. „gib mir alle Empfehlungen als Liste"), der eigentlich kein Business-Layer braucht — wäre das der Moment für CQRS im Kleinen?

---

# HTTP Basics

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **REST als Architekturstil (Fielding-Constraints)** — Die Vertiefung behandelt HTTP als Protokoll, nicht REST als Stil. Fieldings sechs Constraints (Client/Server, Stateless, Cacheable, Uniform Interface, Layered System, Code-on-Demand) sind der eigentliche Kern — insbesondere das *Uniform Interface*, aus dem sich fast alle REST-Regeln ableiten. Dazu passend: das **Richardson Maturity Model** (Level 0 „HTTP als Tunnel" → 1 Ressourcen → 2 HTTP-Verben & Statuscodes → 3 Hypermedia) und die ernüchternde Erkenntnis, dass praktisch jede „REST-API" der Industrie auf Level 2 stehenbleibt.
- 🔴 **HTTP-Caching richtig verstanden** — In der Vertiefung nur als „`GET` ist cacheable" gestreift. Der ganze Mechanismus dahinter: `Cache-Control` (`max-age`, `no-store`, `private`/`public`), **ETag** + `If-None-Match` und `Last-Modified` + `If-Modified-Since`, die Rolle von `304 Not Modified`, sowie der Unterschied zwischen Browser-Cache, Proxy/CDN und Anwendungs-Cache. Praktischer Nebeneffekt: ETags sind auch die Basis für **optimistisches Locking** (`If-Match` → `412 Precondition Failed`) und damit die HTTP-Antwort auf Race Conditions.
- 🔴 **Fehler-Bodys nach RFC 9457 (Problem Details)** — In der Vertiefung nur als Link. Wert der Vertiefung: das konkrete Schema (`type`, `title`, `status`, `detail`, `instance`), wie man es in FastAPI als eigenen Exception-Handler umsetzt und warum ein einheitliches Fehlerformat über alle Endpunkte hinweg mehr wert ist als schöne Einzelmeldungen.
- 🟡 **Content Negotiation im Detail** — `Accept` mit Quality-Werten (`application/json;q=0.9, text/html;q=0.8`), `Accept-Language`, `Accept-Encoding` (gzip/br) und `Vary` als Antwort-Header. Erklärt, warum eine API überhaupt mehrere Repräsentationen derselben Ressource anbieten kann und wie ein Cache das auseinanderhält.
- 🟡 **CORS (Cross-Origin Resource Sharing)** — Warum ein Browser-Frontend auf `localhost:3000` deine API auf `localhost:8000` *nicht* aufrufen darf, was ein **Preflight** mit `OPTIONS` ist, und wie FastAPIs `CORSMiddleware` das löst. Praktisch der erste Stolperstein, sobald ein echtes Frontend dazukommt — und ein Fehler, der ausschließlich im Browser auftritt, nie in Postman.
- 🟡 **Redirects und ihre Tücken** — 301 vs. 302 vs. 307 vs. 308: Der entscheidende Unterschied ist, ob die **Methode erhalten bleibt** (307/308) oder stillschweigend zu `GET` wird (301/302 in der Praxis). Erklärt, warum ein `POST` auf `/units` gelegentlich als `GET` beim Server ankommt, wenn ein Slash am Pfadende fehlt — genau das passiert mit FastAPIs `redirect_slashes`.
- 🟡 **Content-Types jenseits von JSON** — `application/x-www-form-urlencoded` (HTML-Formulare, OAuth2-Token-Endpunkt), `multipart/form-data` (Datei-Uploads), `text/event-stream` (Server-Sent Events), `application/octet-stream`. Sobald ein Upload oder ein Login-Formular dazukommt, ist das relevant.
- 🟡 **HTTPS/TLS konkret** — Was beim Handshake wirklich passiert, was ein Zertifikat beweist (und was nicht), Certificate Authorities, Let's Encrypt, TLS-Terminierung am Reverse Proxy (nginx/Traefik) — der Grund, warum Uvicorn in Produktion meistens gar kein TLS spricht.
- ⚪ **HTTP von Hand sprechen** — Einmal `openssl s_client` oder `telnet` aufmachen und eine Request selbst tippen. Didaktisch extrem wirksam: Danach ist HTTP endgültig entmystifiziert.
- ⚪ **Header-Kuriositäten mit Praxisrelevanz** — `Referer` (mit dem historischen Rechtschreibfehler im Standard), `X-Forwarded-For` und `X-Forwarded-Proto` (was der Reverse Proxy an deine App weiterreicht — wichtig, damit `Location`-Header hinter einem Proxy nicht auf `http://localhost:8000` zeigen), `Retry-After`, `Idempotency-Key`.

## Cookies & Sessions im Detail

*Auf ausdrücklichen Wunsch als eigener Block — die Vertiefung streift Cookies nur in fünf Zeilen.*

- 🔴 **Die Attribute von `Set-Cookie`** — Hier steckt die eigentliche Substanz, und jedes Attribut löst ein konkretes Sicherheitsproblem:
    - `Expires` / `Max-Age` — **Session-Cookie** (stirbt mit dem Browser) vs. **persistentes** Cookie.
    - `Domain` / `Path` — an welche Anfragen der Browser das Cookie anhängt. `Domain=example.com` gilt auch für alle Subdomains, was mehr Reichweite ist, als man meist will.
    - `Secure` — nur über HTTPS senden.
    - `HttpOnly` — **für JavaScript unsichtbar**. Das ist die zentrale Verteidigung gegen **XSS**-Diebstahl der Session.
    - `SameSite=Strict | Lax | None` — ob das Cookie bei Anfragen von *fremden* Seiten mitgeschickt wird. Seit dieses Attribut auf `Lax` als Default umgestellt wurde, ist CSRF für die meisten Anwendungen weitgehend entschärft. `None` erfordert zwingend `Secure`.
    - `__Host-` / `__Secure-` **Präfixe** — wenig bekannt, aber elegant: Ein Cookie mit `__Host-`-Präfix akzeptiert der Browser nur, wenn es `Secure` ist, keinen `Domain`-Wert hat und `Path=/` setzt. Damit kann eine kompromittierte Subdomain es nicht überschreiben.
- 🔴 **CSRF verstehen, nicht nur abwehren** — Warum ein Angriff funktioniert, obwohl der Angreifer die Session gar nicht kennt: Der Browser hängt Cookies **automatisch** an *jede* Anfrage an die Ziel-Domain, egal wer sie ausgelöst hat. Verteidigungen: `SameSite`, CSRF-Token (Synchronizer Token), Double-Submit-Cookie, Prüfung von `Origin`/`Referer`. Der Grund, warum token-basierte APIs davon gar nicht erst betroffen sind: Ein `Authorization`-Header wird nie automatisch mitgeschickt.
- 🔴 **Cookie vs. Token — die Abwägung, die niemand sauber erklärt** — „JWT im `localStorage`" gilt als modern, ist aber für **XSS** angreifbar (JavaScript kommt dran). „Session-Cookie mit `HttpOnly` + `SameSite`" gilt als altmodisch, ist gegen XSS-Diebstahl aber robuster — dafür braucht der Server Sitzungsspeicher. Es gibt hier keine eindeutig richtige Antwort, sondern zwei verschiedene Bedrohungsmodelle. Verwandt: **Refresh Tokens**, Token-Rotation, das ungelöste Problem des **Token-Widerrufs** bei zustandslosen JWTs (ein ausgestelltes JWT kann man nicht zurückrufen — Denylist oder kurze Laufzeiten sind die üblichen Notlösungen).
- 🟡 **Server-side Sessions vs. Stateless** — Wo liegt der Sitzungszustand: im Serverspeicher (skaliert nicht), in Redis (skaliert, aber zusätzliche Infrastruktur) oder im signierten Cookie selbst (skaliert, aber begrenzt und nicht widerrufbar)? Direkter Bezug zur Statuslosigkeit aus [[1.5 REST APIs & Layered Architecture - Vertiefung#Client/Server|Client/Server]] und zu Sticky Sessions.
- 🟡 **Cookies in den DevTools ansehen** — Reiter **Application** (Chrome) bzw. **Storage** (Firefox) → *Cookies*: alle Attribute pro Cookie sichtbar und editierbar. Zusätzlich zeigt der Network-Reiter pro Anfrage einen eigenen **Cookies**-Unterreiter mit „gesendet" und „empfangen". Sehr lehrreich, einmal zu beobachten, wie ein Login-Vorgang das Session-Cookie setzt.
- 🟡 **Grenzen von Cookies** — ~4 KB pro Cookie, begrenzte Anzahl pro Domain, und sie werden bei **jeder** Anfrage mitgeschickt (auch für Bilder und CSS) — ein Grund, warum große Cookies echte Performance-Kosten haben und warum APIs sie meiden.
- ⚪ **Third-Party-Cookies, Tracking und das Danach** — Warum Werbenetzwerke sie brauchten, warum Browser sie inzwischen blockieren, und was an ihre Stelle tritt (Privacy Sandbox, serverseitiges Tracking, Fingerprinting). Erklärt auch, warum es seit der DSGVO überall Cookie-Banner gibt — und warum die meisten davon rechtlich fragwürdig sind.

## Netzwerkanalyse in der Tiefe

*Ebenfalls auf Wunsch ausgebaut — die Vertiefung zeigt nur die Grundhandgriffe.*

- 🔴 **Die Spalten des Network-Reiters vollständig lesen** — Über *Status* und *Time* hinaus lohnen sich vor allem:
    - **Initiator** — *wer* hat diese Anfrage ausgelöst? Klickbar bis in die Zeile JavaScript. Das beste Werkzeug gegen die Frage „wo zum Teufel kommt dieser Request her?".
    - **Size vs. Transferred** — dekomprimierte Größe vs. tatsächlich übertragene Bytes. Die Differenz ist die Wirkung von gzip/brotli; steht dort `(from disk cache)`, ging gar nichts über die Leitung.
    - **Type** — `fetch`/`xhr`/`document`/`preflight`. Ein separat gelisteter `preflight`-Eintrag ist der Beweis, dass CORS im Spiel ist.
    - **Priority** — wie der Browser die Anfrage einordnet.
- 🔴 **Throttling und Fehlerfälle simulieren** — Der am meisten unterschätzte Teil: Im Network-Reiter lässt sich die Verbindung drosseln (*Slow 3G*) oder auf **Offline** stellen, und einzelne Anfragen bzw. ganze URL-Muster lassen sich **blockieren** (Rechtsklick → *Block request URL*). Damit testet man genau die Fälle aus den [[1.5 REST APIs & Layered Architecture - Vertiefung#Client/Server|Fallacies of Distributed Computing]], die sonst nur in Produktion auftreten. Ergänzend: **Local Overrides** (Chrome) — Antworten lokal überschreiben, um einen Fehlerfall im Frontend nachzustellen, ohne den Server anzufassen.
- 🔴 **Beobachtbarkeit auf der Serverseite** — Der Browser zeigt nur *eine* Seite. Die andere: Zugriffslogs von Uvicorn, eine **Timing-Middleware** in FastAPI (Dauer pro Request loggen), und vor allem **Correlation-/Request-IDs** — eine pro Anfrage erzeugte ID, die im Response-Header (`X-Request-ID`) *und* in jeder Logzeile auftaucht. Damit lässt sich eine im Browser gesehene Anfrage im Serverlog exakt wiederfinden. Das ist die logische Fortsetzung von [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]] und in deinem Projekt tatsächlich baubar.
- 🟡 **HAR-Export** — Der gesamte Netzwerkmitschnitt als JSON-Datei (*Export HAR*). Das Standardformat, um einen Fehler an jemand anderen weiterzugeben oder später erneut zu analysieren. **Vorsicht:** Eine HAR-Datei enthält Cookies und Auth-Header im Klartext — vor dem Verschicken bereinigen („Export HAR (sanitized)" nutzen).
- 🟡 **`curl` als Analysewerkzeug statt nur als Aufrufer** — `-v` zeigt Header, `--trace-ascii` alles, und mit `-w` bekommt man exakte Zeitmessungen ohne Browser: `curl -w '%{time_namelookup} %{time_connect} %{time_starttransfer} %{time_total}\n' -o /dev/null -s <url>`. Das ist die kommandozeilentaugliche Version des Timing-Reiters — und automatisierbar.
- 🟡 **Network-Panel vs. Performance-Panel** — Zwei verschiedene Werkzeuge: Network zeigt *Anfragen*, Performance zeigt, was der **Browser-Hauptthread** tut (Rendering, JavaScript, Layout). Eine Seite kann trotz schneller API langsam sein — und umgekehrt. Stichworte: Core Web Vitals (LCP, INP, CLS), Lighthouse.
- 🟡 **`chrome://net-export` und `about:networking`** — Mitschnitt auf einer Ebene *unter* den DevTools: DNS-Auflösung, Socket-Pools, HTTP/2-Sessions. Der richtige Ort, wenn das Problem nicht in der Anwendung liegt.
- 🟡 **mitmproxy** — Ein lokaler Proxy, der HTTPS-Verkehr aufbricht und anzeigt (mit eigenem Zertifikat). Damit inspiziert man Anfragen, die **kein Browser** stellt: eine Mobile App, ein Python-Skript, ein Docker-Container. Das Werkzeug für „was schickt diese Bibliothek da eigentlich?".
- ⚪ **Wireshark / tcpdump** — Die Ebene darunter: einzelne TCP-Pakete, Handshakes, Retransmissions. Selten nötig, aber lehrreich — einmal einen TCP- und einen TLS-Handshake im Mitschnitt sehen, macht die Timing-Phasen aus dem DevTools-Reiter erst wirklich verständlich. Hussein Nassers Kanal aus den [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Quellen]] arbeitet genau auf dieser Ebene.
- ⚪ **Warum HTTPS die Analyse erschwert** — Ein Paketmitschnitt zeigt bei TLS nur verschlüsselten Kram. Die drei Auswege: im Client mitlesen (DevTools), einen Proxy dazwischenhängen (mitmproxy), oder die Session Keys exportieren (`SSLKEYLOGFILE` → Wireshark).
- ⚪ **Öffentliche Hilfsdienste** — `webhook.site` und `requestbin` zeigen, was ein fremdes System *tatsächlich* an dich schickt; `ngrok` macht deinen lokalen Uvicorn von außen erreichbar (nützlich, sobald jemand einen Webhook an dich schicken will).

## Fragen zum eigenen Projekt, die offen geblieben sind

- Welche Exception aus `exceptions.py` wird auf welchen Statuscode abgebildet? Die Zuordnungstabelle ist noch nicht geschrieben — und sie ist eine Design-Entscheidung, keine Formalie.
- Ist ein Datenbankausfall wirklich ein `503` oder doch ein `500`? (Argument für `503`: Das Problem ist nachgelagert und vorübergehend, ein Retry ist sinnvoll. Argument für `500`: Für den Client ist deine API die Blackbox.) Was schickst du in `Retry-After`?
- `_compute()` schreibt Empfehlungen weg. Wenn dieser Vorgang hinter einem Endpunkt hängt: Ist das noch ein `GET` (safe!) oder muss es ein `POST` sein? Das ist genau der Fall, in dem die Safe-Eigenschaft eine Design-Entscheidung erzwingt.
- Sollen die Pfade eine Version tragen (`/v1/...`)? Für ein Traineeship-Projekt Overhead — aber die Entscheidung ist später teuer nachzuholen.
- Lohnt sich eine **Request-ID-Middleware** in der API (ID erzeugen, in `X-Request-ID` zurückgeben, in jede Logzeile hängen)? Wenig Code, und sie macht das Logging aus 1.4 erst wirklich nutzbar, sobald mehrere Anfragen gleichzeitig laufen.
- Wie sähe ein `curl`-Einzeiler aus, der die Antwortzeit deiner Endpunkte misst — und ab welchem TTFB würdest du dir Sorgen machen? (Die Frage zwingt dazu, ein Architekturmerkmal *messbar* zu formulieren, statt „soll schnell sein".)

---

# CRUD Basics

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **HATEOAS praktisch — und warum es fast niemand macht** — Die Vertiefung erklärt das Konzept, nicht die Umsetzung. Es gibt konkrete Medientypen dafür: **HAL** (`_links`), **JSON:API**, **Siren**, **Collection+JSON**. Die interessantere Frage ist die ökonomische: Der Nutzen entsteht erst, wenn Clients Links wirklich *folgen*, statt URLs zu verdrahten — und das tun sie praktisch nie, weil jeder Client ohnehin gegen eine bekannte API programmiert wird. Lesenswert dazu Fieldings Blogpost „REST APIs must be hypertext-driven" (2008), in dem er sich genau darüber ärgert.
- 🔴 **Paginierung** — Der wichtigste ungelöste Punkt an jedem `GET` auf eine Sammlung: Ohne Grenze liefert `GET /bookings` irgendwann eine Million Datensätze. Zwei Verfahren mit klarem Trade-off: **Offset/Limit** (`?limit=50&offset=100`) ist trivial, aber langsam bei großen Offsets und liefert bei parallelen Einfügungen Duplikate oder Lücken. **Cursor-/Keyset-Pagination** (`?after=<cursor>`) ist stabil und schnell, aber kein wahlfreier Zugriff. Für dich als Data Engineer die praktisch relevantere Variante. Dazu: Wo stehen die Metadaten — im Body oder im `Link`-Header (GitHub-Stil)?
- 🔴 **Filtern, Sortieren, Suchen** — Ebenfalls ausgespart. Wie drückt man „alle Buchungen für Produkt X zwischen zwei Daten, sortiert nach Datum" in einer URL aus? Konventionen: `?product_id=X&from=...&to=...&sort=-calendar_date`. Und die Grenze: Ab wann ist die Query so komplex, dass sie in einen `POST /bookings/search`-Body gehört (technisch unrein, weil nicht cacheable und nicht safe — aber der übliche Ausweg)? Das ist einer der Punkte, an denen GraphQL seine Existenzberechtigung herleitet.
- 🔴 **Bulk-Operationen** — CRUD ist auf *ein* Element pro Anfrage ausgelegt. Was, wenn 10.000 Datensätze anzulegen sind (dein Ingest-Fall!)? Optionen: `POST` mit Array im Body, ein eigener `/batch`-Endpunkt, oder gar keine API dafür, sondern ein Bulk-Load an der API vorbei. Direkt relevant, sobald jemand fragt „kann ich die CSV auch hochladen?". Verwandt: **Teilerfolge** — was ist der Statuscode, wenn 9.998 von 10.000 Datensätzen durchgehen? (`207 Multi-Status` ist die formale, selten benutzte Antwort.)
- 🟡 **Soft Delete vs. Hard Delete** — `DELETE` bedeutet in vielen Systemen nicht „weg", sondern „`deleted_at` gesetzt". Das hat Folgen bis in die API hinein: Was liefert ein `GET` auf eine soft-gelöschte Ressource — `404` oder `410 Gone`? Darf sie wiederhergestellt werden (und über welchen Endpunkt)? Als Data Engineer besonders relevant, weil gelöschte Daten in Auswertungen wieder auftauchen können.
- 🟡 **Ressourcen-Zuschnitt: Aggregate und Verschachtelung** — Wie tief darf `/units/42/bookings/7/items/3` gehen? (Faustregel: maximal eine Verschachtelungsebene, danach eigene Top-Level-Ressource mit Filter.) Und wann gehören zwei Dinge in **eine** Ressource statt in zwei? Der saubere Begriff dafür kommt aus Domain-Driven Design: das **Aggregate** mit seiner Wurzel als natürliche Ressourcengrenze.
- 🟡 **Over-Fetching und Under-Fetching** — Das strukturelle Problem fester Repräsentationen: Der Client bekommt zu viele Felder (Over-Fetching) oder muss fünf Anfragen stellen, um eine Ansicht zu füllen (Under-Fetching, „N+1 über HTTP"). Antworten in REST: Sparse Fieldsets (`?fields=id,price`), Embedding (`?include=bookings`), oder ein eigener Endpunkt pro Ansicht (**Backends for Frontends**). Das ist exakt der Punkt, an dem GraphQL ansetzt.
- 🟡 **Konsistenz zwischen Endpunkten** — Heißt es `calendar_date` oder `date`? `snake_case` oder `camelCase`? Werden Zeitstempel als ISO-8601 mit Zeitzone geliefert? Einzeln banal, in Summe der Unterschied zwischen einer API, die man versteht, und einer, die man nachschlagen muss. Die [Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/) beantworten genau solche Fragen durchnummeriert und begründet.
- ⚪ **CRUD als Kritikbegriff** — „CRUD-App" wird in manchen Kreisen abwertend benutzt: eine Anwendung ohne eigene Fachlogik, die nur Formulare auf Tabellen legt. Die Gegenposition (u. a. aus Domain-Driven Design): Sobald echte fachliche Regeln existieren, ist CRUD die falsche Abstraktion — man modelliert dann **Kommandos** und **Ereignisse** statt Zustandsänderungen. Verwandt: **Event Sourcing**, bei dem nicht der Zustand gespeichert wird, sondern die Folge von Ereignissen, die zu ihm führten.
- ⚪ **`202 Accepted` und lang laufende Vorgänge** — In der Vertiefung nur gestreift. Das vollständige Muster: `202` + `Location` auf eine Vorgangs-Ressource, Client pollt deren Status, am Ende ein Verweis auf das Ergebnis. Die HTTP-Antwort auf „das dauert länger als eine Anfrage" — und für einen Pricing-Lauf genau der richtige Ansatz, sobald er nicht mehr in Millisekunden fertig ist.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Wird `/pricing-runs` als Ressource modelliert oder reicht ein pragmatisches `POST /recommendations/compute`? Was gewinnt man mit der ersten Variante konkret — und ab wann zahlt sich das aus?
- `GET /recommendations` ohne Grenze liefert alle Empfehlungen aller Produkte und Tage. Welche Filter sind **verpflichtend** (z. B. Datum), und was ist ein sinnvolles Default-Limit?
- Was ist die fachliche Identität einer Preisempfehlung nach außen — die interne `id`, oder das Paar `(product_id, calendar_date)`? Davon hängt ab, wie die Element-URL aussieht.
- Braucht `capacity` überhaupt Schreiboperationen über die API, oder kommen diese Daten ausschließlich über `ingest.py` ins System? (Eine API anzubieten, die niemand nutzen soll, ist auch eine Entscheidung — nur selten eine gute.)
- Die zwei Endpunkte aus der Bewerberaufgabe: Sind das Ressourcen im REST-Sinn oder Auswertungen/Reports? Falls Letzteres — ist das der Moment für den Lesepfad, der die Business-Schicht überspringen dürfte (vgl. **CQRS** im Abschnitt *Architekturmuster*)?

---

# Die 4 zentralen HTTP-Methoden in REST-Controllern

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Conditional Requests vollständig** — Die Vertiefung nennt `ETag` + `If-Match` nur als Lösung des Lost-Update-Problems. Das ganze Bild: `If-None-Match` (Caching, `304`), `If-Match` (optimistisches Locking, `412`), starke vs. schwache ETags (`W/"..."`), und wie man einen ETag überhaupt bildet (Hash der Repräsentation? Versionsspalte? `updated_at`?). Sehr elegant, weil es Nebenläufigkeitskontrolle ohne serverseitigen Zustand ermöglicht — passt genau zur Statuslosigkeit.
- 🔴 **Idempotency Keys serverseitig implementieren** — In der Vertiefung nur als Konzept. Die Umsetzung wirft echte Fragen auf: Wo wird der Schlüssel gespeichert, wie lange, was passiert bei einem *parallelen* zweiten Versuch (bevor der erste fertig ist), und wie unterscheidet man „gleiche Anfrage erneut" von „gleicher Schlüssel, anderer Inhalt" (→ `422`)? Stripes öffentliche Dokumentation ist hier die beste Referenz.
- 🔴 **Nebenläufigkeit im Schreibpfad** — Der Themenkomplex unter dem Lost-Update-Problem: optimistisches vs. pessimistisches Locking, `SELECT ... FOR UPDATE`, Transaktions-Isolationsstufen, Deadlocks. Die Frage „wer gewinnt, wenn zwei Requests gleichzeitig dieselbe Zeile ändern?" beantwortet nicht HTTP, sondern die Datenbank — und als Data Engineer ist das die Ebene, auf der du sie wirklich verstehen solltest.
- 🟡 **`HEAD` und `OPTIONS` praktisch nutzen** — In der Vertiefung nur als Randnotiz: `HEAD` für günstige Existenz-/Aktualitätsprüfungen (Ist die Datei größer geworden? Hat sich der ETag geändert?), `OPTIONS` + `Allow`-Header als Selbstauskunft. FastAPI erzeugt beide teilweise automatisch — nachzuschauen, was tatsächlich registriert ist, lohnt sich.
- 🟡 **Statuscodes, die man selten sieht, aber gelegentlich braucht** — `202 Accepted` (asynchron, in der Vertiefung gestreift), `207 Multi-Status` (Teilerfolge bei Bulk), `410 Gone` (bewusst gelöscht, im Gegensatz zu „nie existiert"), `412 Precondition Failed`, `415 Unsupported Media Type`, `428 Precondition Required` (der Server *verlangt* ein `If-Match` — die saubere Art, Lost Updates zu verhindern), `431`, `451`.
- 🟡 **Response-Envelope: ja oder nein?** — Liefert man `[{...}, {...}]` direkt oder verpackt in `{"data": [...], "meta": {...}}`? Der Envelope erleichtert Paginierungs-Metadaten und Erweiterbarkeit, kostet aber Direktheit und verleitet zum `200`-mit-Fehler-Anti-Pattern. JSON:API schreibt ihn vor, die meisten APIs verzichten. Eine Entscheidung, die man einmal trifft und dann überall durchhält.
- 🟡 **Zusammengesetzte und alternative Schlüssel in URLs** — `/recommendations/{id}` vs. `/recommendations/{product_id}/{date}` (die offene Frage aus deinem Projekt). Verwandt: Warum viele APIs **opake IDs** (UUID, ULID, Hash) statt fortlaufender Integer verwenden — fortlaufende IDs verraten Geschäftszahlen und laden zum Durchprobieren ein (**IDOR**, eine der häufigsten realen Sicherheitslücken).
- 🟡 **Wie ein Controller getestet wird** — Die Fortsetzung von [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]] auf der API-Ebene: FastAPIs `TestClient` (bzw. `httpx.AsyncClient`), Dependency Overrides zum Einsetzen von Fakes, und die Frage, welche Testebene was prüft — Unit-Test der Service-Schicht vs. API-Test des Controllers vs. Contract-Test. Wichtige Faustregel: Wenn ein Controller-Test *Fachlogik* prüft, ist die Fachlogik am falschen Ort.
- ⚪ **Long Polling, Webhooks und Callbacks** — Wenn `202 Accepted` + Polling zu ineffizient wird: Wie meldet ein Server *aktiv*, dass ein Vorgang fertig ist? Webhooks sind faktisch der umgekehrte Request/Response-Fall und bringen ihre eigene Problemwelt mit (Retries, Signaturen, Reihenfolge).
- ⚪ **RPC-Denke vs. Ressourcen-Denke** — Der Grund, warum `POST /doSomething` sich für viele Entwickler natürlicher anfühlt: Sie denken in Funktionsaufrufen, nicht in Zuständen. Lesenswert als bewusster Perspektivwechsel — und als Vorbereitung auf gRPC, wo RPC-Denke die *richtige* ist.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Bekommt `/recommendations` eine Element-URL über die interne `id` oder über den fachlichen Schlüssel `(product_id, calendar_date)`? Und wenn Letzteres: Ist das Paar wirklich eindeutig?
- Braucht `capacity` Schreibendpunkte, wenn die Daten über `ingest.py` kommen? Falls ja — was passiert bei einem Konflikt zwischen API-Schreibzugriff und laufendem Ingest?
- Wie sieht der zentrale Exception-Handler konkret aus, und welche Exception-Klassen aus `exceptions.py` brauchen eine eigene Zuordnung, welche fallen in den generischen `500`-Fall?
- Sollen `PUT`-Endpunkte ein `If-Match` **verlangen** (`428`), sobald mehr als ein Client schreibt — oder ist das für dieses Projekt Überbau?
- Lohnt sich `PATCH` überhaupt, oder reicht `PUT`? (Weniger Endpunkte, aber Clients müssen immer das ganze Objekt schicken — und das setzt voraus, dass sie es vorher gelesen haben.)

---

# FastAPI + Uvicorn + Pydantic in Python

*Syntax zu allen drei Bibliotheken: [[FastAPI, Uvicorn & Pydantic - Syntax|FastAPI, Uvicorn & Pydantic – Syntax]].*

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **`async`/`await` und der Event Loop richtig verstehen** — Die Vertiefung gibt nur die Faustregel („synchron ⇒ `def`"). Der Unterbau verdient einen eigenen Durchgang: Coroutinen vs. Threads vs. Prozesse, was `await` tatsächlich tut (Kontrolle abgeben, nicht „warten"), `asyncio.gather` für parallele I/O, `run_in_executor`/`asyncio.to_thread` für blockierenden Code, und die **GIL** als Grund, warum Threads in Python bei Rechenarbeit nichts bringen. Praktisch relevant, sobald ein Endpunkt drei Datenquellen gleichzeitig abfragen soll.
- 🔴 **Async SQLAlchemy** — Die logische Fortsetzung: `create_async_engine`, `AsyncSession`, `await session.execute(...)`, und die Fallen (Lazy Loading funktioniert nicht mehr wie gewohnt, `selectinload` wird Pflicht). Die ehrliche Frage dahinter: Lohnt sich der Umbau für dein Projekt überhaupt, oder ist synchron + Threadpool die richtige Wahl? (Für die meisten Anwendungen: ja, synchron reicht.)
- 🔴 **Dependency Injection in FastAPI ausreizen** — In der Vertiefung nur als Stichwort. Weiterführend: verkettete Dependencies, `yield`-Dependencies als Kontextmanager (Transaktionsgrenzen!), Sub-Dependencies mit Caching pro Request, klassenbasierte Dependencies, Dependencies auf Router- und App-Ebene, und `dependency_overrides` als Testwerkzeug. Direkt anschlussfähig an die offene Frage aus dem Abschnitt *n-tier*: **Wer öffnet und committet die Transaktion?**
- 🔴 **Deployment des Stacks** — Der Teil, den Tutorials auslassen: Uvicorn hinter nginx/Traefik, Worker-Anzahl richtig wählen (`2 × CPU + 1` ist die alte Faustregel und für I/O-lastige ASGI-Apps meist falsch), Graceful Shutdown, Health-/Readiness-Endpunkte, Containerisierung, und warum `gunicorn -k uvicorn.workers.UvicornWorker` lange der Standard war (und es heute nicht mehr sein muss).
- 🟡 **Was Starlette selbst kann** — FastAPI reicht vieles nur weiter: Middleware, `BackgroundTasks`, `StaticFiles`, WebSockets, `TestClient`, Streaming-Responses. Wer weiß, was von Starlette kommt, findet die Doku dafür an der richtigen Stelle — und versteht, warum manche FastAPI-Frage in der Starlette-Doku beantwortet wird.
- 🟡 **`BackgroundTasks` und ihre Grenzen** — FastAPI kann nach der Antwort noch Code ausführen (`BackgroundTasks`). Verlockend für „Pricing-Lauf im Hintergrund starten" — aber: kein Retry, kein Persistieren, weg bei einem Neustart. Die Grenze zu echten Task-Queues (**Celery**, **RQ**, **Dramatiq**, **arq**) sollte man kennen, bevor man sie überschreitet.
- 🟡 **Pydantic jenseits von FastAPI** — `TypeAdapter` für Nicht-Modell-Typen, `computed_field`, Serialisierungs-Aliase (`camelCase` nach außen, `snake_case` innen), `Discriminated Unions` für polymorphe Payloads, `Strict Mode` (keine automatische Typkonvertierung), eigene Typen via `Annotated` + `AfterValidator`. Und: Pydantic taugt auch für **Datenvalidierung in Pipelines** — für dich als Data Engineer der eigentlich interessante Einsatz.
- 🟡 **Structured Logging in der API** — Die Fortsetzung von [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]] im Serverkontext: Uvicorns eigenes Logging in die vorhandene Konfiguration integrieren (die beiden beißen sich gern), Zugriffslogs als JSON, `contextvars` für die Request-ID über alle Logzeilen hinweg, und die Frage, was **nicht** ins Log darf (Bodys mit personenbezogenen Daten, Tokens).
- 🟡 **Authentifizierung mit FastAPI** — `OAuth2PasswordBearer`, `Security`-Dependencies, Scopes, API-Keys. Der Leitfaden lässt es aus, aber es ist der offensichtliche nächste Schritt für jede API, die tatsächlich erreichbar ist — und FastAPI hat dafür überraschend elegante Bordmittel.
- ⚪ **Alternativen im Ökosystem** — **Litestar** (ähnlicher Ansatz, anderes DI-Modell), **Django Ninja** (FastAPI-Stil auf Django), **Flask + marshmallow** (der klassische Weg), **msgspec** (deutlich schneller als Pydantic, dafür schmaler). Nützlich, um zu sehen, welche Entwurfsentscheidungen von FastAPI *nicht* alternativlos sind.
- ⚪ **Wie FastAPI intern funktioniert** — Wie werden aus Typannotationen zur Laufzeit Validatoren? (`typing.get_type_hints`, `inspect.signature`, Pydantics `TypeAdapter`.) Lehrreich als Anwendung von Python-Introspektion — und entzaubert die „Magie" des Frameworks.

## Fragen zum eigenen Projekt, die offen geblieben sind

- **Zwei Composition Roots:** `main.py` (Batch) und `api/main.py` (Server) bauen dieselben Objekte. Gemeinsame Factory-Funktion, oder bewusst getrennt halten? Was gehört in die Factory, was bleibt spezifisch?
- Wer öffnet und schließt die **Transaktion** — die `Depends`-Funktion mit `yield`, die Service-Schicht, oder der Adapter? (Standardantwort der Literatur: die Serviceschicht, „Unit of Work". Passt das zu `PricingApp`?)
- Wird der Pricing-Lauf synchron im Request ausgeführt oder als `BackgroundTask`? Und ab welcher Laufzeit ist die Antwort „keins von beidem, das braucht eine Queue"?
- Kommt `pydantic-settings` zum Einsatz, oder bleibt die Konfiguration wie bisher? Der Gewinn wäre, dass eine fehlende `DATABASE_URL` den Prozess **beim Start** stoppt statt bei der ersten Anfrage.
- Uvicorns Logging und die Konfiguration aus 1.4 gleichzeitig — wie verhindert man doppelte oder verschluckte Logzeilen?

---

# Validierung von Request/Response (Pydantic)

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **Mapping zwischen den Modellebenen automatisieren** — Die Vertiefung sagt, *dass* zwischen DTO, Domain und ORM übersetzt werden muss, aber nicht, wie man die Handarbeit klein hält. Ansätze: Mapper-Funktionen pro Richtung (explizit, langweilig, robust), Klassenmethoden (`Booking.from_dto()`), `model_validate(obj, from_attributes=True)` für den Rückweg, oder Bibliotheken wie `dataclass-mapper`/`automapper` (bequem, aber die Fehler wandern in die Laufzeit). Die ehrliche Faustregel: **Explizites Mapping ist langweilig und fast immer richtig** — bis es hundert Felder werden.
- 🔴 **Wo Validierung noch stattfindet: die Datenbank** — Ein dritter Prüfort, den die Vertiefung ausspart: `NOT NULL`, `CHECK`, `UNIQUE`, Fremdschlüssel. Die Frage, wie viel Validierung ins Schema gehört und wie viel in den Code, hat starke Argumente auf beiden Seiten — und für dich als Data Engineer die berufsrelevante Fassung: **Die Datenbank ist der letzte Ort, an dem Datenqualität noch durchsetzbar ist, wenn mehrere Systeme schreiben.** Verwandt: Wie übersetzt man eine `IntegrityError` in einen sinnvollen `409`?
- 🔴 **Fehlermeldungen, die Clients wirklich helfen** — In der Vertiefung nur der Standardfall. Weiterführend: eigene `RequestValidationError`-Handler, RFC 9457 als einheitliches Format über *alle* Fehlerarten hinweg, maschinenlesbare Fehlercodes (`"code": "booking.dates_overlap"`) statt englischer Prosa, Mehrsprachigkeit, und die Sicherheitsfrage — wie viel darf eine Fehlermeldung über die Interna verraten?
- 🟡 **Serialisierungs-Aliase und Feldnamen** — `alias`, `serialization_alias`, `populate_by_name`, `alias_generator` (z. B. automatisch `camelCase` nach außen bei `snake_case` innen). Die saubere Antwort auf die Namenskonventions-Frage aus dem Methoden-Stichwort, ohne die Python-Seite zu verbiegen.
- 🟡 **Pydantic-Typen jenseits der Standardtypen** — `EmailStr`, `HttpUrl`, `SecretStr` (verhindert versehentliches Loggen von Passwörtern!), `AwareDatetime`, `PositiveInt`, `constr`/`conint` (veraltet zugunsten von `Annotated`), sowie eigene Typen per `Annotated[str, AfterValidator(...)]`. Besonders `SecretStr` ist ein Beispiel dafür, wie Typen Sicherheitsprobleme *strukturell* lösen.
- 🟡 **Discriminated Unions** — Wenn ein Feld verschiedene Formen annehmen kann (`{"type": "fixed", ...}` vs. `{"type": "percentage", ...}`), erzeugt Pydantic mit `Field(discriminator="type")` präzise Fehlermeldungen statt „passt zu keiner Variante". Auch in der OpenAPI-Doku sauber abgebildet.
- 🟡 **Strict Mode und Typkonvertierung** — Pydantic konvertiert standardmäßig (`"42"` → `42`). Das ist bequem und gelegentlich gefährlich (`"1"` → `True`). `strict=True` schaltet es ab. Die Grundsatzfrage dahinter: Wie viel Nachsicht schuldet eine API ihren Clients? (Verwandt: Postels Gesetz „be liberal in what you accept" und die berechtigte moderne Kritik daran.)
- 🟡 **Validierung großer und verschachtelter Daten** — Was passiert bei einem 50-MB-JSON-Body? Größenlimits, `max_length` auf Listen, Rekursionstiefe, und **Denial of Service durch teure Validierung** (etwa katastrophales Backtracking in einem `pattern`-Regex, „ReDoS").
- ⚪ **Pydantic ohne FastAPI** — Für dich beruflich der eigentlich spannende Einsatz: Validierung von CSV-Zeilen beim Ingest, Schema-Prüfung von Konfigurationsdateien, Verträge zwischen Pipeline-Schritten. `TypeAdapter` macht das auch ohne `BaseModel` möglich. Vergleich lohnt sich mit **Pandera** (DataFrame-Validierung) und **Great Expectations** (Datenqualität als eigenständige Disziplin).
- ⚪ **Alternativen zu Pydantic** — `msgspec` (deutlich schneller, schmaler), `attrs` + `cattrs`, `marshmallow` (der Klassiker aus der Flask-Welt), `dataclasses` mit `__post_init__`. Nützlich, um zu sehen, welche Entwurfsentscheidungen von Pydantic nicht alternativlos sind.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Bekommt `RecommendationRead` wirklich `product_id` und `calendar_date` statt `capacity_id`? Dann muss der Adapter beim Lesen joinen — wo genau passiert dieser Join, und gehört er in `adapters/` oder in die Service-Schicht?
- Werden Geldbeträge als JSON-Zahl oder als String übertragen? (`Decimal` → Zahl ist bequem, aber JavaScript-Clients rechnen damit in `float` weiter. String ist unbequem und korrekt.)
- Braucht das Projekt eine eigene Domain-Ebene *zwischen* DTO und ORM, oder ist der direkte Weg DTO ↔ ORM hier vertretbares Layer Bridging? Woran machst du diese Entscheidung fest?
- Wie viele der Prüfungen aus `domain.py` sind eigentlich Formatprüfungen, die nach Pydantic gehören — und wie viele echte Fachregeln?
- Soll `extra="forbid"` für alle Eingangsmodelle gelten, oder nimmt man Erweiterbarkeit in Kauf? (Bei einer öffentlichen API kann strenges Ablehnen künftige Client-Versionen brechen.)

---

# Swagger & OpenAPI

## Nicht behandelt, aber direkt anschlussfähig

- 🔴 **API-Versionierung und Breaking Changes konkret** — Die Vertiefung nennt `oasdiff` als Werkzeug, aber nicht die Regeln: Was *ist* ein Breaking Change? (Feld entfernen, Pflichtfeld hinzufügen, Typ verengen, Statuscode ändern — dagegen ist ein *optionales* Feld hinzuzufügen harmlos.) Dazu die Verfahren: Versionierung per URL-Pfad, Header oder Media Type; Deprecation-Fristen und der `Deprecation`/`Sunset`-Header; `deprecated=True` in FastAPI. **Der berufsrelevanteste Punkt dieses Abschnitts.**
- 🔴 **Contract Testing** — Die Antwort auf „was, wenn Client und Server unabhängig deployt werden und auseinanderlaufen?". Zwei Spielarten: Schema-Validierung gegen das OpenAPI-Dokument in der CI (`schemathesis` prüft eine laufende API automatisch gegen ihre Spezifikation — inklusive generierter Zufallseingaben, sehr lehrreich) und Consumer-Driven Contracts (**Pact**). Schließt die Lücke, die Code-First hinterlässt.
- 🔴 **Authentifizierung im Dokument beschreiben** — `securitySchemes` und wie FastAPI daraus den „Authorize"-Knopf in der Swagger UI baut. Ohne das ist jede geschützte API in der Oberfläche unbenutzbar — und es ist der Punkt, an dem die meisten Leute die Swagger UI aufgeben und zu Postman wechseln, obwohl es unnötig wäre.
- 🟡 **Die Doku gestalten, ohne sie zu schreiben** — `openapi_tags` mit Beschreibungen pro Gruppe, `summary`/`description` pro Endpunkt, `examples` in `Field` und `openapi_examples` für vollständige Beispielanfragen, `deprecated`, `include_in_schema=False` für interne Endpunkte, und das Überschreiben von `app.openapi()` für Sonderfälle. Kleine Handgriffe mit großer Wirkung auf die Benutzbarkeit.
- 🟡 **Style-Guides erzwingen** — **Spectral** mit den [Zalando-Regeln](https://opensource.zalando.com/restful-api-guidelines/) als CI-Schritt. Damit werden die Naming-Konventionen aus dem Methoden-Stichwort von einer Empfehlung zu einer **Fitness Function** im Sinne von Richards & Ford — messbar und automatisiert. In deinem Projekt tatsächlich baubar.
- 🟡 **Alternative Dokumentationsoberflächen** — Scalar, Stoplight Elements, RapiDoc; alle lesen dasselbe `/openapi.json`. Nützlich zu wissen, dass Swagger UI und ReDoc nicht die einzigen Optionen sind, wenn die Doku öffentlich wird.
- 🟡 **Das Dokument statisch exportieren** — `app.openapi()` in eine Datei schreiben, ins Repo legen und dort versionieren. Ermöglicht Diffs im Pull Request, Doku-Deployment ohne laufenden Server und Client-Generierung in der CI. Der Standardweg, um Code-First-Dokumente überprüfbar zu machen.
- ⚪ **AsyncAPI** — Das Gegenstück zu OpenAPI für **ereignisgetriebene** Systeme (Kafka, MQTT, WebSockets): Wie beschreibt man einen Vertrag, wenn es keine Request/Response-Paare gibt? Für dich als Data Engineer mittelfristig relevanter als jede weitere REST-Vertiefung.
- ⚪ **Schema Registry als verwandtes Konzept** — In der Datenwelt macht eine Schema Registry (Avro/Protobuf bei Kafka) genau das, was OpenAPI für HTTP macht: den Vertrag maschinenlesbar und versioniert halten, inklusive Kompatibilitätsprüfung beim Veröffentlichen. Dieselbe Idee, anderes Ökosystem.

## Fragen zum eigenen Projekt, die offen geblieben sind

- Wird `/openapi.json` als Datei ins Repo exportiert, damit man Vertragsänderungen im Diff sieht? (Wenig Aufwand, hoher Lerneffekt.)
- Welche Fehlerfälle sind pro Endpunkt zu deklarieren, damit die Doku nicht nur den Erfolgsfall zeigt — und lässt sich das zentral lösen statt pro Endpunkt?
- Sollen `/docs` und `/redoc` in einer echten Produktionsumgebung überhaupt erreichbar sein? (Bei internen APIs meist ja, bei öffentlichen eine bewusste Entscheidung — die Doku verrät auch Angreifern die gesamte Angriffsfläche.)
- Lohnt sich `schemathesis` als zusätzlicher Test? Es prüft die laufende API automatisch gegen ihr eigenes Schema und findet dabei Fälle, an die man nicht denkt.
