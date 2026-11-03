---
tags:
  - karteikarten
erstellt: 2026-11-03
cards-deck: "Traineeship::Grundlagen der Software Entwicklung::FastAPI Tutorial"
---

# FastAPI Tutorial - Karteikarten

Karteikarten zu [[FastAPI Tutorial - Zusammenfassung]]. Syntax, die man auswendig braucht, steht wörtlich auf der Karte, Randthemen nur als Überblick. Format und Abgleich mit Anki beschreibt [[Anki-Karten erstellen]].

- [x] Grundprinzip und erste Schritte
- [x] Parameter
- [x] Pydantic-Modelle und Request Body
- [x] Response-Modelle und Statuscodes
- [x] Fehlerbehandlung
- [x] Dependencies
- [x] Security
- [x] App-Struktur: Router, Middleware, CORS, Lifespan
- [x] Datenbank mit SQLModel
- [x] Testen
- [x] Was es noch gibt

## Grundprinzip und erste Schritte

Welche vier Dinge leitet FastAPI aus einer Typannotation ab?::Konvertierung (`"3"` → `3`), Validierung (sonst 422), OpenAPI-Doku und Editor-Support. Die Annotation ist die einzige Quelle der Wahrheit.

Wann schreibt man einen Endpunkt als `def`, wann als `async def`?::`def` läuft im Threadpool, blockierender Code schadet nicht. `async def` läuft im Event Loop und darf nicht blockieren. Im Zweifel `def`.

Warum muss `/users/me` vor `/users/{user_id}` stehen?::Routen werden in Registrierungsreihenfolge geprüft, die erste passende gewinnt. Sonst landet `"me"` als `user_id` im generischen Handler.

## Parameter

Woher liest FastAPI einen Funktionsparameter?::Name im Pfad → Path-Parameter, singulärer Typ wie `int` → Query, Pydantic-Modell → Body. Ein Marker wie `Query()` überschreibt die Regel.

Wie lautet die moderne Schreibweise für einen validierten Parameter?::`q: Annotated[str | None, Query(max_length=50)] = None`: Typ, Marker mit Regeln, Default hinter dem `=`.

Wodurch wird ein Query-Parameter optional?::Allein durch einen Default. `str | None` ohne `= None` ist Pflicht, darf aber `null` sein. `default=` innerhalb von `Annotated` ist verboten.

Was unterscheidet `Path`, `Query`, `Header`, `Cookie` und `Body`?::Nur die Datenquelle, die Optionen sind gleich. `Path` ist immer Pflicht, `Header` wandelt `_` in `-`, `Body` kennt `embed=True`.

Wie nimmt man einen mehrfach vorkommenden Query-Parameter entgegen?::`tags: Annotated[list[str], Query()] = []`. Ohne `Query()` würde die Liste als Body interpretiert.

Wie bündelt man mehrere Query-Parameter in einem Modell?::Pydantic-Modell definieren und als `Annotated[FilterParams, Query()]` annotieren. Nach außen bleibt jedes Feld ein eigener Parameter.

Wie lehnt man unbekannte Felder in Eingabemodellen ab?::`model_config = {"extra": "forbid"}`. Ein Tippfehler wie `?limt=10` gibt dann 422 statt still den Default.

## Pydantic-Modelle und Request Body

Wie deklariert man einen Request Body?::Ein Parameter mit Pydantic-Modell als Typ, z. B. `async def create_item(item: Item)`. Ein `dict` liefert `item.model_dump()`.

Was passiert bei mehreren Body-Parametern, und was bewirkt `Body(embed=True)`?::Der Body wird nach Parameternamen verschachtelt, z. B. `{"item": {…}, "user": {…}}`. `embed=True` erzwingt das auch bei nur einem Modell.

Wie kommt ein einzelner Wert in den Body statt in die Query?::Mit Marker: `importance: Annotated[int, Body(gt=0)]`. Ohne ihn wäre `int` ein Query-Parameter.

Worauf achtet man bei Listen in Pydantic-Modellen?::Immer den Inhaltstyp angeben: `list[str]` statt `list`. `set[str]` entfernt Duplikate. Modelle lassen sich beliebig tief verschachteln.

## Response-Modelle und Statuscodes

Warum ist das Response-Modell ein Sicherheitsmerkmal?::Es filtert die Antwort auf die deklarierten Felder. Deshalb getrennte Modelle wie `UserIn` (mit Passwort), `UserOut` und `UserInDB`, z. B. per Basisklasse als Rückgabetyp.

Was unterscheidet Rückgabetyp und `response_model`?::Der Rückgabetyp `-> ItemOut` ist der bevorzugte Weg. `response_model=` nutzt man, wenn der Handler etwas anderes zurückgibt, z. B. ein ORM-Objekt, und gewinnt bei beidem.

Wie setzt man den Erfolgs-Statuscode?::Am Dekorator: `@app.post("/items/", status_code=status.HTTP_201_CREATED)`. `204` und `304` haben keinen Body.

Wie macht man ein partielles Update mit PATCH?::`update_data = item.model_dump(exclude_unset=True)`, dann `stored.model_copy(update=update_data)`. Ein `PUT` mit unvollständigem Body setzte fehlende Felder still auf Defaults.

## Fehlerbehandlung

Wie meldet man einen Fehler an den Client?::`raise HTTPException(status_code=404, detail="…")`, nicht `return`. Bricht auch aus verschachtelten Hilfsfunktionen ab, `detail` darf JSON sein.

Wie bildet man eigene Exceptions zentral auf HTTP-Antworten ab?::`@app.exception_handler(NotFoundError)` mit einer Funktion, die eine `JSONResponse` zurückgibt. Endpunkte beschreiben dann nur den Erfolgsfall.

Was unterscheidet einen Validierungsfehler der Anfrage von einem der Antwort?::Ungültige Anfrage: `RequestValidationError`, 422, Client ist schuld. Ungültige Antwort: 500, ein Bug im eigenen Code.

Wie sieht eine 422-Antwort aus?::`{"detail": [...]}` mit `loc` (z. B. `["query", "limit"]`), `msg` und `type` pro Fehler. `str(exc)` nie an Clients geben, es enthält Dateipfade.

## Dependencies

Was ist eine Dependency in FastAPI?::Ein Callable, dessen Signatur FastAPI wie die eines Endpunkts auswertet. Sein Rückgabewert wird injiziert: `Annotated[dict, Depends(common_parameters)]`.

Wie vermeidet man wiederholte Dependency-Deklarationen?::Mit einem Type-Alias: `CommonsDep = Annotated[dict, Depends(common_parameters)]`, dann `commons: CommonsDep`.

Wann hängt man Dependencies in den Dekorator, und wo kann man sie global setzen?::Wenn nur der Effekt zählt, z. B. Auth: `dependencies=[Depends(verify_token)]`. Global an `FastAPI()`, `APIRouter()` oder `include_router()`.

Wie schreibt man eine Dependency mit Setup und Teardown?::Mit genau einem `yield` statt `return`, aufräumen im `finally`, z. B. `db = SessionLocal(); try: yield db; finally: db.close()`.

Warum muss man Exceptions in `yield`-Dependencies neu werfen?::Sonst bekommt der Client 500, aber es entsteht kein Log, der Fehler verschwindet. Also `HTTPException` werfen oder `raise`.

Wann läuft der Teardown einer `yield`-Dependency?::Standardmäßig nach dem Senden der Antwort (`scope="request"`), mit `scope="function"` direkt nach dem Endpunkt. Mehrere Dependencies räumen in umgekehrter Reihenfolge auf.

Was ist der Gewinn von Dependency Injection für Tests?::`app.dependency_overrides[get_db] = lambda: fake_db` ersetzt Abhängigkeiten ohne Codeänderung. Danach mit `.clear()` zurücksetzen.

## Security

Wie läuft der OAuth2 Password Flow ab?::Client schickt `username` und `password` als Form-Data an `/token` und bekommt einen Token. Danach schickt er `Authorization: Bearer <token>` bei jedem Request.

Was macht `OAuth2PasswordBearer(tokenUrl="token")`?::Liest den Bearer-Token aus dem Header (sonst 401) und trägt das Security Scheme in OpenAPI ein. Es prüft den Token nicht und legt keinen `/token`-Endpunkt an.

Wie sieht die typische Auth-Dependency-Kette aus?::`oauth2_scheme` liefert den Token, `get_current_user` prüft ihn und lädt den User, `get_current_active_user` prüft den Status. Endpunkte nutzen nur die letzte.

Wie muss der Token-Endpunkt antworten?::`{"access_token": "<token>", "token_type": "bearer"}`. Eingabe über `form_data: Annotated[OAuth2PasswordRequestForm, Depends()]`, braucht `python-multipart`.

Welcher Header gehört zu 401, und warum ist „Inactive user“ 400?::`WWW-Authenticate: Bearer` nennt das erwartete Verfahren. Bei inaktiven Usern war die Authentifizierung erfolgreich, abgelehnt wird aus fachlichem Grund.

Was ist ein JWT, und was ist der häufigste Denkfehler?::Ein signiertes, kompakt kodiertes JSON. Es ist signiert, nicht verschlüsselt, jeder kann den Inhalt lesen, also keine Geheimnisse hineinschreiben.

Wie erzeugt und prüft man ein JWT mit PyJWT?::`jwt.encode({"sub": name, "exp": …}, SECRET_KEY, algorithm="HS256")` und `jwt.decode(token, SECRET_KEY, algorithms=["HS256"])`. Ungültige Tokens werfen `InvalidTokenError`.

Welche Bibliotheken empfiehlt die FastAPI-Doku für Passwörter, und warum hasht sie auch bei unbekannten Usern?::`pwdlib` mit Argon2. Ein Dummy-Hash bei unbekannten Usern verhindert, dass Antwortzeiten existierende Accounts verraten.

## App-Struktur: Router, Middleware, CORS, Lifespan

Was ist ein `APIRouter`, und welche Fallen gibt es?::Ein einhängbares Mini-FastAPI: `APIRouter(prefix="/items", tags=["items"])` plus `app.include_router()`. `prefix` endet nicht auf `/`, jedes Paket braucht `__init__.py`.

Was unterscheidet `include_router()` von `app.mount()`?::`include_router` macht die Endpunkte Teil der App samt Doku. `mount` hängt eine eigenständige App ein, z. B. `StaticFiles`, ohne Doku.

Wie schreibt man eine HTTP-Middleware, und was bekommt sie nicht mit?::`@app.middleware("http")` mit `response = await call_next(request)`. Teardown von `yield`-Dependencies und Background Tasks laufen erst danach.

Welche Falle hat `allow_credentials=True` bei der `CORSMiddleware`?::Dann darf keine der Listen `allow_origins`, `allow_methods` oder `allow_headers` `["*"]` sein, alles muss explizit stehen.

Wie führt man Code beim Start und Ende der App aus?::Mit einem `@asynccontextmanager`-Lifespan: vor `yield` Start, danach Shutdown, übergeben mit `FastAPI(lifespan=lifespan)`. Ersetzt `@app.on_event`.

## Datenbank mit SQLModel

Was ist SQLModel, und was unterscheidet Table von Data Model?::Bibliothek auf SQLAlchemy und Pydantic. Nur die Klasse mit `table=True` ist eine Tabelle, die anderen sind Pydantic-Modelle für Ein- und Ausgabe.

Warum trennt man auch mit SQLModel in mehrere Modelle?::Kontrolle über Ein- und Ausgabe: `Hero` (Tabelle), `HeroPublic` (ohne Geheimes), `HeroCreate` und `HeroUpdate` mit optionalen Feldern.

Wie sieht die DB-Session als Dependency aus?::`def get_session(): with Session(engine) as session: yield session`, dazu `SessionDep = Annotated[Session, Depends(get_session)]`. Eine Session pro Request.

Warum ist `create_all()` beim Start nur eine Abkürzung?::Es legt Tabellen an, ändert bestehende aber nie. Schemaänderungen gehören in Migrationen mit Alembic.

## Testen

Wie testet man eine FastAPI-App?::`client = TestClient(app)`, dann `response = client.get("/")` und Asserts auf `status_code` und `json()`. Basiert auf HTTPX.

Warum startet der `TestClient` keinen Server, und warum sind Tests `def`?::Er ruft die App direkt über ASGI auf, schnell und ohne Ports. Er kapselt die Asynchronität, pytest braucht kein Plugin.

## Was es noch gibt

Was sind Background Tasks, und wo ist ihre Grenze?::Arbeit nach dem Senden der Antwort per `background_tasks.add_task(...)`. Kein Retry, weg bei Neustart, echte Jobs gehören in Celery.

Wie nimmt man Formulare und Datei-Uploads entgegen?::`Annotated[str, Form()]` und `file: UploadFile`, beides mit `python-multipart`. `UploadFile` lagert große Dateien aus. Form-Daten und JSON lassen sich nicht mischen.

Wie streamt man eine Antwort, und wann SSE statt WebSocket?::`yield` statt `return` mit `-> AsyncIterable[Item]` ergibt JSON Lines. SSE, wenn nur der Server sendet, WebSocket, wenn beide Seiten senden.

Wie verwaltet man Konfiguration in FastAPI?::`pydantic-settings` mit `BaseSettings` und `env_file=".env"`, dazu `@lru_cache` auf `get_settings()` plus `Depends`. Fehlende Pflichtwerte scheitern beim Start.

Wozu dient `responses={}` am Dekorator?::Dokumentiert weitere Statuscodes wie `{404: {"model": Message}}` in OpenAPI. Reine Doku, erzeugt und validiert nichts.

Was gewinnt man durch das OpenAPI-Dokument?::Aus `/openapi.json` lassen sich typisierte Clients generieren, z. B. für TypeScript. Für lesbare Namen `generate_unique_id_function` setzen.
