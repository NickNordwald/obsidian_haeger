# ressourcen und quellen

*gilt für das gesamte kapitel „grundlagen der software entwicklung", nicht nur für oop.*

## bücher

- **fluent python** (luciano ramalho) – geht tief in pythons objektmodell und „pythonic" idiome; ideal, um python wirklich zu verstehen statt nur zu benutzen.
- **clean code** (robert c. martin) – klassiker zu lesbarem, wartbarem code; sprachunabhängig geschrieben, prinzipien lassen sich gut auf python übertragen.
- **architecture patterns with python** (harry percival & bob gregory) – domain-driven design, test-driven development und event-driven architecture anhand eines durchgehenden python-beispielprojekts. kostenlos online verfügbar unter dem titel „cosmic python": [cosmicpython.com](https://www.cosmicpython.com/).
- **Python Testing with pytest** (Brian Okken) – Das Standardwerk zu pytest; deckt Fixtures, Parametrisierung, Mocking und Coverage praxisnah ab (Pragmatic Bookshelf).
- **Fundamentals of Software Architecture** (Mark Richards & Neal Ford) – Das aktuelle Standardwerk zu Architekturstilen; Kapitel 10 behandelt die Schichtenarchitektur inkl. Sinkhole-Anti-Pattern und der Frage „offene vs. geschlossene Schichten" (O'Reilly).
- **Patterns of Enterprise Application Architecture** (Martin Fowler) – Der Ursprung des Presentation/Domain/Data-Modells und der Service-Layer-Begriffe. Der [Pattern-Katalog](https://martinfowler.com/eaaCatalog/) ist frei online.
- **RESTful Web APIs** (Leonard Richardson & Mike Amundsen) – Der Kanon zu REST-Design jenseits von „CRUD über HTTP"; erklärt Ressourcen, Repräsentationen und Hypermedia sauber her.

## webseiten

- **[offizielle python-dokumentation](https://docs.python.org/3/tutorial/)** – die primärquelle: tutorial, sprachreferenz und doku der standardbibliothek.
- **[real python](https://realpython.com/)** – tutorials und artikel zu nahezu jedem python-thema, von einsteiger- bis fortgeschrittenen-niveau.
- **[refactoring guru](https://refactoring.guru/)** – visuell aufbereitete erklärungen zu design patterns, refactoring-techniken und code smells.
- **microsoft learn** – microsofts kostenlose lernplattform, u. a. mit python- und softwareentwicklungs-lernpfaden.
- **[martin fowler – blog](https://martinfowler.com/)** – artikel zu softwarearchitektur, refactoring und enterprise-patterns von einem der einflussreichsten autoren im software-engineering.
- **[python packaging user guide](https://packaging.python.org/)** – offizieller leitfaden zu paketierung, distribution und dependency-management in python.
- **python enhancement proposals (peps)** – offizielle design-dokumente der sprache:
    - **[pep 8](https://peps.python.org/pep-0008/)** – style guide für python-code (formatierung, namenskonventionen).
    - **[pep 20](https://peps.python.org/pep-0020/)** – „the zen of python", die leitphilosophie hinter pythons designentscheidungen.
    - **[pep 257](https://peps.python.org/pep-0257/)** – konventionen für docstrings.
- **[awesome python](https://awesome-python.com/)** – kuratierte liste von python-frameworks, bibliotheken und tools, nach kategorie sortiert.

## datenbanken

- **[sql bolt](https://sqlbolt.com/)** – interaktives tutorial, das sql-grundlagen schritt für schritt anhand von übungen direkt im browser vermittelt.
- **[postgresql tutorial](https://www.postgresql.org/docs/current/tutorial.html)** – offizielles einführungstutorial aus der postgresql-dokumentation.
- **[postgresql exercises](https://pgexercises.com/)** – sammlung von sql-übungsaufgaben mit lösungen, nach schwierigkeitsgrad aufgebaut.
- **[sqlalchemy tutorial](https://docs.sqlalchemy.org/en/20/tutorial/index.html)** – offizielle einführung in sqlalchemy, pythons verbreitetstes orm- und sql-toolkit.
- **[database systems (berkeley cs186)](https://cs186berkeley.net/notes/note13/)** – Vorlesungsnotizen zu Datenbanksystemen aus dem Berkeley-Kurs CS186.
- **[CMU 15-445](https://15445.courses.cs.cmu.edu/spring2026/)** – bekannter „Database Systems"-Kurs der Carnegie Mellon University, mit Vorlesungen und Materialien frei online.

## Testing

- **[pytest](https://docs.pytest.org/)** – Offizielle Dokumentation des Test-Frameworks (How-to-Guides, Referenz zu Fixtures & Parametrisierung).
    - **[Get Started](https://docs.pytest.org/en/stable/getting-started.html)** – Der offizielle Einstieg („Tutorial"-Teil) der pytest-Doku.
- **[unittest.mock](https://docs.python.org/3/library/unittest.mock.html)** – Offizielle Doku des Mocking-Bordmittels der Standardbibliothek (`Mock`, `patch`, `side_effect`, `autospec`).
- **[coverage.py](https://coverage.readthedocs.io/)** – Offizielle Doku des zugrunde liegenden Coverage-Werkzeugs (inkl. Branch-Coverage).
- **[pytest-cov](https://pytest-cov.readthedocs.io/)** – Plugin, das coverage.py in den pytest-Lauf integriert.
- **[Martin Fowler – Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)** – Kompakte Referenz zur Testpyramide.
- **[Martin Fowler – Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)** – Der kanonische Artikel zu Stub vs. Mock (State- vs. Behavior-Verification).
- **[Martin Fowler – Test Coverage](https://martinfowler.com/bliki/TestCoverage.html)** – Warum Coverage ein Diagnose-Werkzeug ist und kein Ziel.
- **[Mike Cohn – The Forgotten Layer of the Test Automation Pyramid](https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid)** – Der Ursprungstext der Testpyramide vom Urheber selbst.

## Softwarearchitektur & Schichten

- **[Martin Fowler – PresentationDomainDataLayering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)** – Die kompakteste Referenz zur Drei-Schichten-Aufteilung: warum man so schneidet, wo die Grenzen liegen und wann die Aufteilung schadet.
- **[Martin Fowler – Service Layer (PoEAA-Katalog)](https://martinfowler.com/eaaCatalog/serviceLayer.html)** – Definiert die Serviceschicht als Grenze der Anwendung und beantwortet die Kontrollfrage nach der Trennung von Service- und Datenzugriffsschicht.
- **[Microsoft Learn – N-tier architecture style](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/n-tier)** – Saubere Definition von Tiers vs. Layers, mit „closed vs. open layers" (= Strict Layering vs. Layer Bridging) und Best Practices.
- **[Microsoft Learn – Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)** – Zeigt den Weg vom „All-in-One" über die klassische N-Layer- bis zur Clean Architecture; .NET-Kontext, aber die Argumentation ist sprachunabhängig.
- **[Cosmic Python – Kapitel 4: Service Layer](https://www.cosmicpython.com/book/chapter_04_service_layer.html)** – Dasselbe Thema direkt in Python, inklusive der Frage, wann eine Serviceschicht überhaupt nötig ist (Fortsetzung des bereits oben gelisteten Buchs).
- **[Developer to Architect – Free Lessons (Mark Richards)](https://www.developertoarchitect.com/lessons/)** – *Geheimtipp:* über 200 kostenlose Kurzvideos (10–20 Min.) zu Architekturthemen vom Autor von „Fundamentals of Software Architecture" — u. a. zu Layered Architecture, Kopplung und Architekturentscheidungen.

## REST & HTTP

- **[MDN – HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)** – Die Referenz für HTTP-Grundlagen; besonders die [Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview), die [Methoden](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) (inkl. Safe/Idempotent) und die [Statuscodes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status).
- **[RFC 9110 – HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)** – Die Primärquelle: was Methoden, Statuscodes und Header *normativ* bedeuten. Nicht am Stück lesen, sondern gezielt nachschlagen (z. B. §9.2 zu Idempotenz).
- **[Roy Fielding – Dissertation, Kapitel 5 „Representational State Transfer"](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)** – Der Text, in dem REST erfunden wurde; erstaunlich gut lesbar und klärt, dass REST ein Architekturstil ist und kein Framework.
- **[Martin Fowler – Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)** – Die vier Reifegrade einer REST-API (Level 0 bis Hypermedia); ordnet ein, wo eine typische CRUD-API tatsächlich steht.
- **[Microsoft Learn – RESTful web API design](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)** – Sehr praxisnah zu Ressourcen-Naming, HTTP-Methoden-Semantik, Statuscodes und Versionierung.
- **[Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)** – *Geheimtipp:* der öffentliche interne API-Styleguide von Zalando. Hunderte durchnummerierte, begründete MUST/SHOULD-Regeln — die beste frei verfügbare Antwort auf „wie sind die Naming-Konventionen von REST-Endpunkten?".
- **[Google API Improvement Proposals (AIP)](https://google.aip.dev/)** – *Geheimtipp:* Googles Design-Regeln für APIs im PEP-Stil; besonders die AIPs 121–136 zu Ressourcen-Orientierung und Standardmethoden.
- **[RFC 9457 – Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)** – Standardformat für Fehler-Responses; die richtige Antwort auf „wie sieht mein Error-Body aus?" statt selbst erfundener JSON-Strukturen.
- **[Chrome DevTools – Network Reference](https://developer.chrome.com/docs/devtools/network)** – Offizielle Doku zum Netzwerk-Tab: Requests und Responses inspizieren, Timing/Waterfall lesen, Throttling (Kontrollfrage zu Inhalt *und* Performance).
- **[HTTP Cats](https://http.cat/)** – *Geheimtipp der unseriösen Sorte:* jeder Statuscode als Katzenbild. Überraschend wirksam, um sich 409 und 422 endgültig zu merken.

## FastAPI, Pydantic & OpenAPI

- **[FastAPI – Tutorial / User Guide](https://fastapi.tiangolo.com/tutorial/)** – Die offizielle Doku ist gleichzeitig das beste Tutorial zum Framework. Besonders relevant für dieses Kapitel:
    - **[Bigger Applications – Multiple Files](https://fastapi.tiangolo.com/tutorial/bigger-applications/)** – `APIRouter`: die Antwort auf „mehrere kleine Controller statt einem großen".
    - **[Response Model](https://fastapi.tiangolo.com/tutorial/response-model/)** – Response-Validierung und das Trennen von Input- und Output-Modellen (DTOs).
    - **[Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)** – `Depends` als Dependency-Injection-Mechanismus; der saubere Weg, den Service-Layer in den Controller zu reichen.
    - **[Handling Errors](https://fastapi.tiangolo.com/tutorial/handling-errors/)** – `HTTPException` und eigene Exception-Handler; das Bindeglied zu [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]].
    - **[Testing](https://fastapi.tiangolo.com/tutorial/testing/)** – `TestClient`, damit die API-Schicht in der Test-Suite aus [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]] landet.
    - **[SQL Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)** – Zusammenspiel mit SQLAlchemy/SQLModel und die Frage, warum Pydantic- und ORM-Modelle getrennt sind.
- **[Pydantic – Dokumentation](https://docs.pydantic.dev/latest/)** – Offizielle Doku; [Models](https://docs.pydantic.dev/latest/concepts/models/) und [Validators](https://docs.pydantic.dev/latest/concepts/validators/) sind die Kernkapitel, [Why use Pydantic?](https://docs.pydantic.dev/latest/why/) beantwortet die Kontrollfrage direkt.
- **[Uvicorn](https://www.uvicorn.org/)** – Offizielle Doku des ASGI-Servers ([GitHub](https://github.com/encode/uvicorn)); klärt die Arbeitsteilung Server ↔ Framework.
- **[ASGI-Spezifikation](https://asgi.readthedocs.io/en/latest/)** – Der Standard hinter Uvicorn/Starlette/FastAPI und der Nachfolger von WSGI; erklärt, warum „FastAPI + Uvicorn" zwei getrennte Dinge sind.
- **[Starlette](https://www.starlette.io/)** – Das ASGI-Toolkit, auf dem FastAPI aufbaut. Lohnt sich, wenn man wissen will, was FastAPI selbst macht und was es nur weiterreicht.
- **[OpenAPI Specification](https://spec.openapis.org/oas/latest.html)** – Die Spezifikation selbst; zum Nachschlagen, wie das von FastAPI generierte `/openapi.json` aufgebaut ist.
- **[Swagger – What is OpenAPI?](https://swagger.io/docs/specification/v3_0/about/)** – Sortiert die Begriffe Swagger (Tooling) vs. OpenAPI (Spezifikation), die ständig verwechselt werden.
- **[fastapi-best-practices (zhanymkanov)](https://github.com/zhanymkanov/fastapi-best-practices)** – *Geheimtipp:* aus der Produktionserfahrung entstandene Sammlung zu Projektstruktur, Pydantic-Einsatz, async/sync-Fallen und Router-Aufteilung.
- **[Full Stack FastAPI Template](https://github.com/fastapi/full-stack-fastapi-template)** – Das offizielle Projekt-Template des FastAPI-Autors; nützlich als Referenz dafür, wie ein gewachsener API-Ordnerbaum aussieht.

## HTTP-Clients & API-Tooling

- **[Postman – Docs](https://learning.postman.com/docs/introduction/overview/)** – Der Marktstandard unter den HTTP-Clients; Collections, Environments, Tests.
- **[Bruno](https://www.usebruno.com/)** – *Geheimtipp:* schlanke Postman-Alternative, die Requests als Klartextdateien im Repo ablegt (versionierbar via git) und ohne Cloud-Account/Login auskommt.
- **[HTTPie](https://httpie.io/docs/cli)** – HTTP-Client fürs Terminal mit menschenlesbarer Syntax; ideal für schnelle Checks ohne GUI.
- **[HTTPX](https://www.python-httpx.org/)** – Der HTTP-Client, den FastAPIs `TestClient` intern nutzt; relevant sobald man die API aus Python heraus testet oder anspricht.

## Datenbank-Migrationen & Schema-Versionierung

- **[Alembic – Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)** – Der offizielle Einstieg in das Migrationswerkzeug des SQLAlchemy-Autors: Environment anlegen, Revisionen erzeugen, `upgrade`/`downgrade`.
- **[Alembic – Auto Generating Migrations](https://alembic.sqlalchemy.org/en/latest/autogenerate.html)** – Was der Diff zwischen Modell und Datenbank erkennt und (wichtiger) **was nicht** — die Liste der nicht erkannten Änderungen sollte man einmal gelesen haben.
- **[Alembic – The Importance of Naming Constraints](https://alembic.sqlalchemy.org/en/latest/naming.html)** – Warum Constraints Namen brauchen, damit man sie je wieder löschen kann; mit dem hübschen Vergleich der automatisch vergebenen Namen in PostgreSQL vs. Oracle.
- **[Alembic – Changelog](https://alembic.sqlalchemy.org/en/latest/changelog.html)** – Lohnt bei diesem Werkzeug tatsächlich: Die Autogenerate-Fähigkeiten wachsen laufend (CHECK-Constraints z. B. erst seit 1.19.0), Aussagen aus älteren Blogposts sind schnell überholt.
- **[Alembic – Cookbook](https://alembic.sqlalchemy.org/en/latest/cookbook.html)** – Rezepte für die Fälle jenseits des Tutorials: Baseline bestehender Datenbanken, mehrere Datenbanken, Datenmigrationen.
- **[Liquibase – Documentation](https://docs.liquibase.com/)** – Offizielle Doku des datenbank- und sprachunabhängigen Gegenstücks: Changelogs in XML/YAML/JSON/SQL, Changesets, Rollback-Konzept.
- **[Liquibase – GitHub Releases](https://github.com/liquibase/liquibase/releases)** – Die verlässlichste Quelle für den *aktuellen* Stand: Die 5.0-Release-Notes dokumentieren den Lizenzwechsel, die Aufteilung in Community/Secure und den Package Manager. Ältere Blogartikel beschreiben ein anderes Produkt.
- **[Functional Source License (FSL)](https://fsl.software/)** – Die Lizenz, unter der Liquibase Community seit 5.0 steht: „Fair Source" statt Open Source, mit automatischer Umwandlung in Apache 2.0 nach zwei Jahren. Kurz und gut lesbar — eine gute Übung im Lesen von Lizenztexten.
- **[Flyway – Documentation](https://documentation.red-gate.com/flyway)** – Das dritte große Werkzeug (SQL-first, Versionsnummern im Dateinamen). Gute Referenz für den Vergleich der Ordnungsmodelle.
- **[Martin Fowler & Pramod Sadalage – Evolutionary Database Design](https://martinfowler.com/articles/evodb.html)** – Der Grundlagenartikel des ganzen Themas: warum das Schema in die Versionskontrolle gehört und was „Datenbank-Refactoring" bedeutet.
- **[Martin Fowler – ParallelChange](https://martinfowler.com/bliki/ParallelChange.html)** – Das Expand/Migrate/Contract-Muster in seiner allgemeinen Form; die Antwort auf „wie ändere ich etwas Zerstörerisches ohne Ausfall".
- **[Refactoring Databases (Ambler & Sadalage)](https://databaserefactoring.com/)** – Der Katalog der Datenbank-Refactorings (frei online), analog zu Fowlers Refactoring-Katalog für Code.
- **[PostgreSQL – ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html)** – Die Referenz dafür, welche Änderung die Tabelle umschreibt und welche nur den Katalog anfasst; siehe auch die Notes zu `ADD COLUMN` mit Default.
- **[PostgreSQL – Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)** – Die Lock-Matrix; erklärt, warum `ACCESS EXCLUSIVE` alles blockiert und wie die Lock-Warteschlange funktioniert.
- **[Strong Migrations](https://github.com/ankane/strong_migrations)** – *Geheimtipp:* Rails-Werkzeug, dessen README aber die beste kompakte Liste **gefährlicher Migrationen samt sicherer Alternative** enthält — sprachunabhängig lesenswert.

## Docker & Containerisierung

- **[Docker – Get Started / Guides](https://docs.docker.com/get-started/)** – Der offizielle Einstieg: Images, Container, Compose in der von Docker selbst empfohlenen Reihenfolge.
- **[Docker – Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)** – Die Referenz aller Direktiven (`FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`, …). Zum Nachschlagen, nicht zum Durchlesen.
- **[Docker – Building best practices](https://docs.docker.com/build/building/best-practices/)** – Die Kernseite für gute Images: Layer-Caching, Reihenfolge der Anweisungen, Multi-Stage-Builds, `.dockerignore`.
- **[Docker – Compose file reference](https://docs.docker.com/reference/compose-file/)** – Die Spezifikation der `compose.yaml`: `services`, `volumes`, `networks`, `depends_on` mit `condition`, `healthcheck`.
- **[Docker – Storage / Volumes](https://docs.docker.com/engine/storage/volumes/)** – Volumes vs. Bind Mounts vs. tmpfs und wann welches. Die Antwort auf „warum ist mein Zustand nach `docker rm` weg?".
- **[Docker – Networking overview](https://docs.docker.com/engine/network/)** – Warum `localhost` im Container nicht der Host ist und wie die DNS-Auflösung über Dienstnamen in Compose funktioniert.
- **[Docker – Engine release notes](https://docs.docker.com/engine/release-notes/)** – Der verlässliche Stand statt veralteter Blogartikel (Engine 29.x, containerd 2.x, runc 1.4.x).
- **[Docker – Rootless mode](https://docs.docker.com/engine/security/rootless/)** – Docker ohne root-Daemon. Relevant, sobald das Argument „Podman ist rootless, Docker nicht" fällt — es stimmt so nicht mehr.
- **[Open Container Initiative – Specs](https://opencontainers.org/)** – Image-, Runtime- und Distribution-Spec. Der Beleg dafür, dass „Container" ein **Standard** ist und Docker nur eine Implementierung davon.
- **[uv – Working with Docker](https://docs.astral.sh/uv/guides/integration/docker/)** – Direkt einschlägig für unser Projekt: offizielle Beispiele für `uv sync --frozen`, Cache-Mounts und Multi-Stage mit uv.
- **[Docker – Python language guide](https://docs.docker.com/guides/python/)** – Python-spezifischer Leitfaden von Docker selbst: Basis-Image-Wahl, Non-root-User, Layer-Aufteilung.
- **[Testcontainers for Python](https://testcontainers-python.readthedocs.io/)** – Echte PostgreSQL-Instanz pro Integrationstest statt Mock. Die Verbindung zwischen diesem Kapitel und [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.3 Testing mit PyTest/1.3 Testing mit PyTest - Vertiefung|1.3 Testing mit PyTest]].
- **[Julia Evans – Container zines & Artikel](https://jvns.ca/categories/containers/)** – *Geheimtipp:* Namespaces, cgroups und „was ist ein Container eigentlich" von Hand nachgebaut, in ungewöhnlich verständlicher Form.
- **[Liz Rice – Containers From Scratch (Vortrag)](https://www.youtube.com/watch?v=8fi7uSYlOdc)** – *Geheimtipp:* Ein Container wird live in ~100 Zeilen Go gebaut. Nach diesem Vortrag ist „Container = Prozess mit eingeschränkter Sicht" kein Merksatz mehr, sondern offensichtlich.

## YouTube

- **[Visually Explained](https://www.youtube.com/@VisuallyExplainedEducation)** – visuelle Erklärungen zu Mathe-/CS-Konzepten; nicht Python-spezifisch, aber gut für grundlegendes Verständnis.
- **[ArjanCodes](https://www.youtube.com/@ArjanCodes)** – Softwaredesign und -architektur in Python: Design Patterns, Clean Code, Projektstruktur.
- **[Corey Schafer](https://www.youtube.com/@Coreyms)** – sehr verständliche, umfassende Python-Tutorials für Einsteiger bis Fortgeschrittene.
- **[mCoding](https://www.youtube.com/@mCoding)** – kurze, tiefgehende Videos zu Python-Internals und fortgeschrittenen bzw. weniger bekannten Sprachfeatures.
- **[Hussein Nasser](https://www.youtube.com/@hnasr)** – *Geheimtipp:* Backend-Engineering aus der Netzwerk-Perspektive — HTTP/1.1 vs. HTTP/2, TCP, Connection Pooling, Datenbank-Internals. Genau die Ebene unter REST, die man selten erklärt bekommt.
