---
deck: "Traineeship::Grundlagen der Software Entwicklung::FastAPI Tutorial"
---

Karteikarten zu [[FastAPI Tutorial - Zusammenfassung|FastAPI Tutorial – Zusammenfassung]]. Wird über das Plugin **[Anki Sync](https://github.com/debanjandhar12/Obsidian-Anki-Sync)** (+ AnkiConnect-Addon in Anki) synchronisiert: Anki im Hintergrund laufen lassen, dann in Obsidian den Befehl **„Start Obsidian Anki Sync"** ausführen (oder auf das Anki-Symbol links in der Ribbon-Leiste klicken).

Ziel-Deck ist oben im Frontmatter als `deck` gesetzt: `Traineeship::Grundlagen der Software Entwicklung::FastAPI Tutorial` — auf derselben Ebene wie die Kapitel-Decks. Die `oid`-Attribute in den Block-Kommentaren werden beim ersten Sync automatisch ergänzt.

**Konvention:** nur Basic-Karten (keine Cloze); **~10 Karten pro Stichwort als Richtwert**. **Syntax, die man auswendig braucht, steht wörtlich auf der Karte**; Randthemen bekommen nur einen High-Level-Überblick — Details stehen in der Zusammenfassung.

# Fortschritt

- [x] Grundprinzip & erste Schritte
- [x] Parameter — das gemeinsame Muster
- [x] Pydantic-Modelle & Request Body
- [x] Response-Modelle & Statuscodes
- [x] Fehlerbehandlung
- [x] Dependencies
- [x] Security
- [x] App-Struktur: Router, Middleware, CORS, Lifespan
- [x] Datenbank (SQLModel)
- [x] Testen
- [x] Was es noch gibt

---

# Grundprinzip & erste Schritte

<!-- basicblock-start oid="ObsKYBvPLqz3ZQI58xxHYMX4" -->
Welche **vier** Dinge leitet FastAPI aus einer einzigen Typannotation ab?::
1. **Parsing/Konvertierung** — `"3"` wird zu `int` 3<br>2. **Validierung** — `"foo"` als `int` ergibt automatisch **422**<br>3. **Dokumentation** — OpenAPI-Schema, daraus Swagger UI und ReDoc<br>4. **Editor-Support** — Autovervollständigung und Typprüfung<br>Kernidee: Man schreibt weder Parser noch Validierung noch Doku, sondern **eine Annotation** — sie ist die einzige Quelle der Wahrheit.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs876g1mh1YZjfv1EJPWvFX" -->
Was ist eine **Path Operation** und was eine **Path Operation Function**?::
**Path Operation** = Pfad + HTTP-Methode zusammen, also das, was der Dekorator registriert (`@app.get("/items/{item_id}")`).<br>**Path Operation Function** = die darunter dekorierte Python-Funktion, die die Anfrage bearbeitet.<br>Ihr Name ist frei wählbar, landet aber als `operationId` im OpenAPI-Dokument.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaeHwzhOtuIb8iG9dUAA8X" -->
Was ist der Unterschied zwischen **API-Schema** und **Daten-Schema**?::
**API-Schema** = **OpenAPI**: Welche Pfade, Methoden und Parameter gibt es? Liegt unter `/openapi.json`.<br>**Daten-Schema** = **JSON Schema**: Wie sieht ein einzelnes Objekt aus (Felder, Typen, Constraints)? Steckt *innerhalb* des OpenAPI-Dokuments.<br>Merksatz: OpenAPI beschreibt die **Schnittstelle**, JSON Schema die **Daten**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxygaSz9d5o9qgiGgd69b1" -->
Wie sieht die minimale FastAPI-Anwendung aus, und wie startet man sie?::
`from fastapi import FastAPI`<br>`app = FastAPI()`<br>`@app.get("/")`<br>`async def root():`<br>`    return {"message": "Hello World"}`<br>Start: `fastapi dev main.py` (neues CLI) oder klassisch `uvicorn main:app --reload`.<br>Doku dann unter `/docs` (Swagger UI) und `/redoc` (ReDoc).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsW7K7ZR7TYqT0fe0KuWEIB" -->
Wann schreibt man einen Endpunkt als `def` und wann als `async def`?::
**`def`** — FastAPI führt die Funktion in einem **Threadpool** aus; blockierender Code (synchrones SQLAlchemy, `requests`, Dateizugriff) ist dort unschädlich.<br>**`async def`** — läuft direkt im **Event Loop**; darin darf **nichts blockieren**, sonst steht der ganze Server.<br>Im Zweifel `def`. Beides ist frei mischbar, auch zwischen Endpunkt und Dependency.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsmq3eYLf1BvvrvWiFyv2gO" -->
Warum muss `/users/me` **vor** `/users/{user_id}` deklariert werden?::
Weil Routen in der **Reihenfolge ihrer Registrierung** geprüft werden und die **erste passende gewinnt**.<br>Steht `/users/{user_id}` zuerst, matcht es auch die URL `/users/me` — der Handler bekommt dann den String `"me"` als `user_id`, statt dass der Spezialfall greift.<br>Regel: **spezifische Pfade vor generischen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqEuM4gx0p0UPhsa3Y6xqG" -->
Wie schränkt man einen Pfadparameter auf **feste Auswahlwerte** ein?::
Über ein Enum, das **von `str` erbt** — dann serialisiert und dokumentiert es sauber als String:<br>`class ModelName(str, Enum):`<br>`    alexnet = "alexnet"`<br>`    resnet = "resnet"`<br>Dann `async def get_model(model_name: ModelName)`. Im Code vergleicht man mit `model_name is ModelName.alexnet`, den Rohwert liefert `model_name.value`.<br>In OpenAPI erscheint das als `enum`, in der Swagger UI als Dropdown.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsswdDNAvOQ4kaYoWqRPTvl" -->
Wie deklariert man einen Pfadparameter, der selbst **einen Pfad mit Schrägstrichen** enthält?::
Mit dem Starlette-Konverter `:path`:<br>`@app.get("/files/{file_path:path}")`<br>Ohne ihn würde ein `/` im Wert das Routing beenden.<br>Haken: In OpenAPI ist das nicht ausdrückbar, und ein führender Slash im Wert braucht in der URL einen doppelten Slash (`/files//home/nick/x.txt`).
<!-- basicblock-end -->

---

# Parameter — das gemeinsame Muster

<!-- basicblock-start oid="ObsXswn0I6INj1PfZQ2Bq07Z" -->
Nach welcher Regel entscheidet FastAPI, **woher** ein Funktionsparameter gelesen wird?::
In dieser Reihenfolge:<br>1. Der Name kommt im **Pfad-String** vor → **Path-Parameter**<br>2. Der Typ ist **singulär** (`int`, `str`, `bool`, `float`, `UUID`, `datetime` …) → **Query-Parameter**<br>3. Der Typ ist ein **Pydantic-Modell** → **Body**<br>Ein expliziter **Marker** im `Annotated` (`Query()`, `Header()`, `Body()` …) überschreibt diese Regel immer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUg8wzswuFcgW5nKEA5lto" -->
Wie lautet die moderne Schreibweise für einen Parameter mit Validierungsregeln?::
`q: Annotated[str | None, Query(max_length=50)] = None`<br>Aufbau: **Typ** im ersten Slot, **Marker mit Regeln** im zweiten, **Default** ganz normal hinter dem `=`.<br>`Annotated` gibt es seit FastAPI 0.95; der alte Stil `q: str | None = Query(default=None, max_length=50)` läuft noch, ist aber überholt.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1ay0zbz10KMXQBzG6PCJH" -->
Wie macht man einen Query-Parameter zur **Pflicht** bzw. **optional**?::
Allein über den **Defaultwert**:<br>`q: Annotated[str, Query()]` → **Pflicht** (kein Default)<br>`q: Annotated[str | None, Query()] = None` → optional<br>`q: Annotated[str, Query()] = "fixed"` → optional mit Default<br>`q: Annotated[str | None, Query()]` → **Pflicht**, aber `null` ist ein zulässiger Wert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxHiT6GZIvGe0EpmGvgek9" -->
Warum macht `str | None` einen Parameter **nicht** optional?::
Weil die Union nur sagt, dass `None` ein zulässiger **Wert** ist — optional wird ein Parameter ausschließlich durch einen **Defaultwert**.<br>`q: Annotated[str | None, Query()]` ohne `= None` ist deshalb ein **Pflichtparameter**, der explizit `null` sein darf.<br>Das ist eine der häufigsten Verwechslungen im Tutorial.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsNbo0wahAfIoN3p7dWhwBA" -->
Warum darf man innerhalb von `Annotated` kein `default=` verwenden?::
Weil es dann zwei widersprüchliche Quellen für den Default gäbe:<br>`q: Annotated[str, Query(default="rick")] = "morty"` → **Fehler**<br>Richtig ist `q: Annotated[str, Query()] = "rick"` — der echte Python-Default steht an der echten Default-Stelle.<br>Das ist auch der Vorteil des `Annotated`-Stils: Die Funktion bleibt außerhalb von FastAPI ganz normal aufrufbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsgr9aTQ6Hbn7UafCPLvLsS" -->
Was unterscheidet `Path`, `Query`, `Header`, `Cookie` und `Body` voneinander?::
Mechanisch **nichts** — alle nehmen dieselben Optionen; sie legen nur die **Datenquelle** fest:<br>`Path()` — Pfadsegment, **immer Pflicht**<br>`Query()` — Query-String<br>`Header()` — HTTP-Header, wandelt `_` automatisch in `-`<br>`Cookie()` — Cookie-Header<br>`Body()` — JSON-Body, kennt zusätzlich `embed=True`<br>Dazu `Form()` und `File()` für Formulardaten bzw. Uploads.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPvzdJU9dOMxsFVS2uUMCk" -->
Welche Optionen kann man allen Parameter-Markern mitgeben?::
**Zahlen:** `gt`, `ge`, `lt`, `le`, `multiple_of`, `max_digits`, `decimal_places`<br>**Strings/Sequenzen:** `min_length`, `max_length`, `pattern` (früher `regex`, veraltet)<br>**Doku:** `title`, `description`, `examples`, `openapi_examples`, `deprecated`, `include_in_schema`<br>**Namen:** `alias` — externer Name, wenn er kein gültiger Python-Bezeichner ist, z. B. `Query(alias="item-query")`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXpmvM4enoePsvjPI16cpG" -->
Wie nimmt man einen Query-Parameter entgegen, der **mehrfach** vorkommt (`?tag=a&tag=b`)?::
`tags: Annotated[list[str], Query()] = []`<br>Das Ergebnis ist `["a", "b"]`.<br>Das explizite `Query()` ist hier **Pflicht**: Ohne den Marker würde FastAPI eine `list`-Annotation als **Body** interpretieren.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1AC1bXj96eU3DQhcPiSN0" -->
Welche Besonderheit hat `Header()` gegenüber den anderen Markern?::
Es wandelt **Unterstriche automatisch in Bindestriche** um: `user_agent: Annotated[str | None, Header()] = None` liest den Header `user-agent`.<br>Grund: HTTP-Header sind mit Bindestrich geschrieben und case-insensitiv, Python-Bezeichner nicht.<br>Abschalten mit `Header(convert_underscores=False)`. Mehrfach gesendete Header nimmt man mit `list[str]` entgegen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspQmHrnjCI44O3DbQBoAbX" -->
Wie bündelt man mehrere zusammengehörige Query-Parameter in einem Pydantic-Modell?::
Modell definieren und den Parameter mit `Query()` annotieren (seit FastAPI 0.115):<br>`class FilterParams(BaseModel):`<br>`    limit: int = Field(100, gt=0, le=100)`<br>`    offset: int = Field(0, ge=0)`<br>dann `async def read_items(f: Annotated[FilterParams, Query()])`.<br>Jedes Feld bleibt nach außen ein **eigener** Query-Parameter. Analog für `Cookie()`, `Header()` und `Form()`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1gBc6txrnvILPSWasumXJ" -->
Wie lehnt man unbekannte Felder in einem Parameter- oder Body-Modell ab?::
`model_config = {"extra": "forbid"}` im Modell.<br>Ein unbekannter Parameter erzeugt dann **422** mit `"type": "extra_forbidden"` statt still ignoriert zu werden.<br>Sehr sinnvoll für Eingabemodelle: Ein Tippfehler des Clients (`?limt=10`) fällt sofort auf, statt kommentarlos den Default zu verwenden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXN934UntmwdI0nA1S9wrW" -->
Wann nutzt man Pydantics `Field` und wann FastAPIs `Query`/`Path`/`Body`?::
**`Field`** (aus `pydantic`) — Regeln und Metadaten für **Attribute innerhalb eines Modells**.<br>**`Query`, `Path`, `Body` …** (aus `fastapi`) — für **Funktionsparameter**; sie legen zusätzlich die **Datenquelle** fest.<br>Technisch dieselbe Maschinerie: `Query` & Co. erben über FastAPIs `Param` von Pydantics `FieldInfo` — deshalb sind die Optionen identisch.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsXzswkuPeiozdhtXPkI0x4" -->
Spielt die **Reihenfolge** der Funktionsparameter eine Rolle?::
Mit `Annotated` **nein** — FastAPI ordnet nach Name, Typ und Marker zu, nicht nach Position.<br>Der alte Trick mit dem führenden `*` (`def f(*, item_id: int = Path(...), q: str)`) war nur nötig, weil im alten Stil `= Path(...)` ein echter Python-Default war und Python keinen Parameter ohne Default danach erlaubt.<br>Mit `Annotated` entfällt das Problem komplett.
<!-- basicblock-end -->

---

# Pydantic-Modelle & Request Body

<!-- basicblock-start oid="Obs19GvuCLCHLJPTxb2Lqdr1" -->
Wie deklariert man einen Request Body?::
Als Funktionsparameter mit einem Pydantic-Modell als Typ — mehr braucht es nicht:<br>`class Item(BaseModel):`<br>`    name: str`<br>`    price: float`<br>`    description: str | None = None`<br>dann `async def create_item(item: Item)`.<br>Im Handler kommt man mit `item.model_dump()` an ein `dict`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObspWIQ9c94nIauRCyYvFjOJ" -->
Was passiert, wenn eine Endpunktfunktion **mehrere** Body-Parameter hat?::
FastAPI verschachtelt den Body dann **nach den Parameternamen**:<br>Bei `async def update(item: Item, user: User, importance: Annotated[int, Body()])` erwartet die API<br>`{"item": {…}, "user": {…}, "importance": 5}`.<br>Bei nur **einem** Modell liegen dessen Felder dagegen direkt auf oberster Ebene.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsLnzQbwj9nSTJtoU0JXzKR" -->
Was bewirkt `Body(embed=True)`?::
Es erzwingt die Verschachtelung auch dann, wenn es **nur ein** Body-Modell gibt.<br>`item: Annotated[Item, Body(embed=True)]` erwartet `{"item": {"name": "Foo", …}}` statt `{"name": "Foo", …}`.<br>Nützlich, wenn der Body später um weitere Felder wachsen soll, ohne dass sich das Format der bestehenden ändert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsaYvN2WPDKkinQ2hOBalnj" -->
Wie bekommt man einen **einzelnen Wert** (keinen Modell-Parameter) in den Body statt in die Query?::
Mit dem `Body()`-Marker:<br>`importance: Annotated[int, Body(gt=0)]`<br>Ohne den Marker wäre `importance: int` nach der Standardregel ein **Query-Parameter**, weil `int` ein singulärer Typ ist.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsxO4vB3vPaRBQk3TXEXPbT" -->
Worauf muss man bei Listen-Annotationen in Pydantic-Modellen achten?::
Immer den **Inhaltstyp** angeben: `tags: list` lässt den Inhalt **ungeprüft**, `tags: list[str]` validiert ihn.<br>`set[str]` entfernt zusätzlich Duplikate und erzeugt im JSON Schema `uniqueItems`.<br>Für unbekannte Schlüssel gibt es `dict[int, float]` — JSON-Keys kommen als String an und werden konvertiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4BiLHqlkeKWnuQX1j7n4x" -->
Wie tief darf man Pydantic-Modelle verschachteln?::
Beliebig tief — mit voller Validierung, Konvertierung und Doku auf **jeder** Ebene:<br>`class Image(BaseModel): url: HttpUrl; name: str`<br>`class Item(BaseModel): images: list[Image] | None = None`<br>`class Offer(BaseModel): items: list[Item]`<br>Auch ein reines Listen-Body ist erlaubt: `async def f(images: list[Image])`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4g24OCjQwKB7dR7RSbciu" -->
Welche Datentypen jenseits von `int`, `str`, `float` und `bool` versteht FastAPI direkt?::
`UUID`, `datetime`, `date`, `time`, `timedelta`, `Decimal`, `bytes`, `set` und `frozenset`.<br>Übertragen werden sie als String bzw. Zahl (`datetime` als ISO 8601, `timedelta` als Sekunden), im Handler bekommt man aber das **echte Python-Objekt** und kann direkt damit rechnen:<br>`start_process = start_datetime + process_after`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsFQhoRxLFZ2rP6t77Wj4a7" -->
Wie hinterlegt man Beispieldaten für die Doku?::
Vier Möglichkeiten, gleiche Wirkung, unterschiedlicher Ort:<br>Im Modell: `model_config = {"json_schema_extra": {"examples": [...]}}`<br>Am Feld: `name: str = Field(examples=["Foo"])`<br>Am Parameter: `Body(examples=[{...}])` — eine **Liste**, landet im JSON Schema<br>Am Parameter: `Body(openapi_examples={...})` — ein **Dict** mit `summary`/`description`/`value`, ergibt ein Dropdown in der Swagger UI.<br>Faustregel: `examples` reicht fast immer.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZnEbpMb4hUUoD2I3jtONY" -->
Welche Pydantic-v1-Schreibweisen begegnen einem im Netz, und wie heißen sie in v2?::
`.dict()` → `.model_dump()`<br>`.json()` → `.model_dump_json()`<br>`parse_obj()` → `.model_validate()`<br>`@validator` → `@field_validator`<br>`class Config` → `model_config`<br>`.copy(update=…)` → `.model_copy(update=…)`<br>Ältere Blogposts und Stack-Overflow-Antworten sind fast immer v1 und laufen so nicht mehr.
<!-- basicblock-end -->

---

# Response-Modelle & Statuscodes

<!-- basicblock-start oid="ObsQKKYb0m6iMlr8VOImxd0h" -->
Warum ist das Response-Modell ein **Sicherheitsmerkmal** und nicht nur Komfort?::
Weil es die Antwort **filtert**: Was im Modell nicht deklariert ist, verlässt den Server nicht — auch dann nicht, wenn der Handler es versehentlich zurückgibt.<br>Klassischer Fall: `UserIn` enthält das Passwort. Nutzt man dasselbe Modell für Ein- und Ausgabe, gibt die API Passwörter zurück.<br>Deshalb das Trio **`UserIn`** (mit Passwort), **`UserOut`** (ohne), **`UserInDB`** (mit `hashed_password`, nie nach außen).
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszbYVCASnlxQN8FeFoSD77" -->
Was ist der Unterschied zwischen dem Rückgabetyp und dem Parameter `response_model`?::
`async def read_item() -> ItemOut` — der **Rückgabetyp**, der bevorzugte Weg.<br>`@app.get("/x", response_model=ItemOut)` — der **Dekorator-Parameter**; nötig, wenn der tatsächlich zurückgegebene Typ **abweicht** (ORM-Objekt, `dict`), damit der Typechecker nicht zu Recht meckert.<br>Sind **beide** gesetzt, gewinnt `response_model`. `response_model=None` schaltet Filterung, Validierung und Doku ab.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKcFUCapNhvhDkaBOwAJfT" -->
Wie filtert man Ausgabefelder elegant über **Vererbung**?::
Basisklasse als Rückgabetyp annotieren, Subklasse zurückgeben:<br>`class BaseUser(BaseModel): username: str; email: EmailStr`<br>`class UserIn(BaseUser): password: str`<br>`async def create_user(user: UserIn) -> BaseUser: return user`<br>Der Typechecker ist zufrieden (eine Subklasse **ist** ein `BaseUser`), FastAPI filtert trotzdem auf die Felder der Basis — `password` fliegt raus.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsPvUZ5EQbacuWg7eB84liw" -->
Was bewirkt `response_model_exclude_unset=True`?::
Es liefert nur die Felder, die **tatsächlich gesetzt** wurden — Defaultwerte, die niemand angefasst hat, fallen aus der Antwort raus.<br>Nützlich bei Modellen mit vielen optionalen Feldern, damit die Antwort nicht mit `null` und Defaults zugemüllt wird.<br>Verwandt: `response_model_exclude_defaults` und `response_model_exclude_none`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsiJ2zC180fO8wQnOxqdO2I" -->
Welchen Haken haben `response_model_include` und `response_model_exclude`?::
Sie nehmen ein `set[str]` und schneiden Felder aus der **Antwort** — aber das **OpenAPI-Schema zeigt weiterhin das vollständige Modell**.<br>Die Doku lügt damit über die tatsächliche Antwort.<br>Sauberer ist deshalb fast immer ein eigenes, kleineres Modell.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHvopm5J9oSAJhuSpqnvgI" -->
Wie setzt man den Erfolgs-Statuscode eines Endpunkts?::
Über den Dekorator: `@app.post("/items/", status_code=201)`<br>Lesbarer mit den Konstanten: `from fastapi import status` und dann `status_code=status.HTTP_201_CREATED`.<br>Die Konstanten sind reine Lesbarkeit — `201` tut exakt dasselbe. `fastapi.status` ist identisch mit `starlette.status`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsALtTZYpRWrCe9talcL7Ct" -->
Welche Statuscodes dürfen **keinen** Body haben?::
**204 No Content** und **304 Not Modified** — außerdem generell die gesamte **1xx**-Klasse.<br>FastAPI weiß das und dokumentiert diese Operationen entsprechend ohne Response-Body.<br>Typischer Einsatz: `@app.delete("/items/{id}", status_code=status.HTTP_204_NO_CONTENT)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsINWaPXLB0CeT5BTihFFvo" -->
Warum ist ein `PUT` mit unvollständigem Body gefährlich?::
Weil `PUT` **vollständig ersetzt**: Felder, die der Client wegläßt, fallen auf die Modell-Defaults zurück — **stiller Datenverlust**.<br>Beispiel: Ein gespeichertes `"tax": 20.2` wird durch den Modell-Default `10.5` überschrieben, nur weil der Client `tax` nicht mitgeschickt hat.<br>Für partielle Updates ist `PATCH` mit `exclude_unset=True` gedacht.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsBx7h96TVMSgObT3nsiLze" -->
Wie lautet das Rezept für ein **partielles Update** (PATCH)?::
`stored_model = Item(**stored_data)`<br>`update_data = item.model_dump(exclude_unset=True)`<br>`updated = stored_model.model_copy(update=update_data)`<br>Entscheidend ist **`exclude_unset=True`** — nur damit lässt sich „Feld weggelassen" von „Feld auf Default gesetzt" unterscheiden.<br>Zusätzlich muss das Update-Modell alle Felder optional haben (`str | None = None`), sonst schlägt schon die Validierung fehl.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsu033kSt0WLpqvChtInx4r" -->
Was macht `jsonable_encoder` und wann braucht man es?::
Es wandelt beliebige Objekte (Pydantic-Modelle, `datetime`, `UUID` …) in **JSON-kompatible Python-Datenstrukturen** um — also in `dict`/`list`/`str`, **nicht** in einen JSON-String.<br>`from fastapi.encoders import jsonable_encoder`<br>Nötig, wenn man an FastAPIs automatischer Serialisierung **vorbei** arbeitet: Schreiben in eine JSON-Spalte oder NoSQL-DB, oder beim direkten Zurückgeben einer `JSONResponse`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsN5pdFqYyFPt8N00ArK4K4" -->
Welche Metadaten kann man einem Endpunkt am Dekorator mitgeben?::
`tags=["items"]` — Gruppierung in der Swagger UI (auch als Enum, gegen Tippfehler)<br>`summary="…"` — kurze Überschrift<br>`description="…"` — oder einfach die **Docstring** der Funktion, Markdown wird gerendert<br>`response_description="…"` — beschreibt speziell die Antwort<br>`deprecated=True` — markiert den Endpunkt als veraltet, er funktioniert weiter.
<!-- basicblock-end -->

---

# Fehlerbehandlung

<!-- basicblock-start oid="Obs4QRwiJ6wGWWHfdjo4Grtd" -->
Wie meldet man einen Fehler an den Client?::
Mit `raise`, nicht mit `return`:<br>`raise HTTPException(status_code=404, detail="Item nicht gefunden")`<br>Als Exception bricht sie die Verarbeitung sofort ab — auch aus tief verschachtelten Hilfsfunktionen heraus.<br>`detail` darf alles JSON-Fähige sein (`dict`, `list`), nicht nur ein String. Optional: `headers={"X-Error": "…"}`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsepfQETGnC47EgUeqeXVi9" -->
Wie bildet man **eigene** Exceptions zentral auf HTTP-Antworten ab?::
Mit einem Exception-Handler:<br>`@app.exception_handler(NotFoundError)`<br>`def handle(request: Request, exc: NotFoundError):`<br>`    return JSONResponse(status_code=404, content={"detail": str(exc)})`<br>So bleibt die Zuordnung Domänenfehler → Statuscode an **einer** Stelle, und die Endpunkte beschreiben nur den Erfolgsfall.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZDWipgdW4mfaPq9EF7hCJ" -->
Was ist der Unterschied zwischen `RequestValidationError` und einem Pydantic-`ValidationError` in der Antwort?::
**`RequestValidationError`** — die **Anfrage** war ungültig, der Client ist schuld → **422**.<br>Ein **`ValidationError` beim Bauen der Antwort** — der Server hätte kaputte Daten ausgeliefert, also ist der eigene Code schuld → **500**.<br>Merksatz: 422 zeigt nach außen, 500 zeigt auf einen Bug bei einem selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsbksh7CyFpo2B5iLuxzMk1" -->
Worauf muss man beim Überschreiben des `HTTPException`-Handlers achten?::
Werfen sollte man weiterhin FastAPIs `HTTPException` (deren `detail` beliebiges JSON erlaubt, Starlettes nur Strings) — den **Handler** aber für die Starlette-Variante registrieren:<br>`from starlette.exceptions import HTTPException as StarletteHTTPException`<br>Sonst greift er nicht bei Exceptions, die aus Starlette-internem Code oder aus Plugins kommen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsmIyWltA3nSHuA1b40RtwM" -->
Warum darf man `str(exc)` einer `RequestValidationError` nicht an den Client geben?::
Weil die Darstellung **Dateiname und Zeilennummer** enthält — ein Informationsleck über die interne Struktur des Servers.<br>Stattdessen `exc.errors()` gezielt auswerten und nur Feldpfad und Meldung zurückgeben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsVbOtBkYaQYKeuCsphczpA" -->
Wie ergänzt man den Standard-Fehlerhandler, ohne ihn komplett zu ersetzen?::
Die Original-Handler importieren und am Ende aufrufen:<br>`from fastapi.exception_handlers import http_exception_handler`<br>`@app.exception_handler(StarletteHTTPException)`<br>`async def custom(request, exc):`<br>`    logger.warning(...)`<br>`    return await http_exception_handler(request, exc)`<br>So loggt man zusätzlich, behält aber das Standardverhalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsUG9JHSm0ZqhRKKvaXiXYq" -->
Wie sieht eine 422-Antwort von FastAPI inhaltlich aus?::
Ein JSON-Objekt mit dem Schlüssel `detail`, darin eine **Liste** von Fehlern — pro Fehler:<br>`loc` — wo (z. B. `["query", "limit"]` oder `["body", "price"]`)<br>`msg` — was ist falsch<br>`type` — die Fehlerart (z. B. `extra_forbidden`, `int_parsing`)<br>Der `loc`-Pfad zeigt dabei auch die Quelle (`query`, `body`, `header`, `cookie`).
<!-- basicblock-end -->

---

# Dependencies

<!-- basicblock-start oid="ObsyJQLHXi4O5VXscJ7gPPnS" -->
Was ist eine Dependency in FastAPI?::
Ein beliebiges **Callable**, dessen Signatur FastAPI **genauso auswertet wie die einer Endpunktfunktion** — mit Parametern, Validierung, Konvertierung und OpenAPI-Doku.<br>Die Doku nennt es „eine Path Operation Function ohne Dekorator".<br>Der Rückgabewert wird injiziert:<br>`async def read_items(commons: Annotated[dict, Depends(common_parameters)])`
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnEui0YOCPmuk8kpOVHLNr" -->
Warum schreibt man `Depends(func)` und nicht `Depends(func())`?::
Weil `Depends` das **Callable selbst** bekommt — FastAPI ruft es pro Request auf.<br>Mit Klammern würde man das *Ergebnis* eines einmaligen Aufrufs übergeben.<br>Zusatz: Nur das Argument **innerhalb** von `Depends()` zählt für FastAPI; der Typ davor im `Annotated` ist reine Editor-Information.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObstQwLMpwaam6vQQSNDwxPJ" -->
Wie vermeidet man, dieselbe Dependency-Deklaration in jedem Endpunkt zu wiederholen?::
Mit einem **Type-Alias** — reines Standard-Python, kein FastAPI-Feature:<br>`CommonsDep = Annotated[dict, Depends(common_parameters)]`<br>Danach genügt `async def read_items(commons: CommonsDep)`.<br>Das ist der empfohlene Stil; die Typinformation bleibt für Editor und mypy erhalten.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsYZfzeEKCicWcgAROH19yt" -->
Wie nutzt man eine **Klasse** als Dependency, und was bringt das?::
Eine Klasse ist callable, also inspiziert FastAPI die Parameter ihrer `__init__`:<br>`commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]`<br>Kurzform, wenn Typ und Dependency identisch sind: `commons: Annotated[CommonQueryParams, Depends()]`<br>Vorteil gegenüber einem `dict`: **Autovervollständigung und Typprüfung** auf dem Ergebnis.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsCBzAkmMa8axo34aJLmaxT" -->
Was passiert, wenn dieselbe Dependency in einem Request mehrfach gebraucht wird?::
FastAPI ruft sie **nur einmal pro Request** auf und **cached** den Rückgabewert für alle Abhängigen desselben Requests.<br>Wichtig, wenn eine Sub-Dependency von mehreren Dependencies gebraucht wird — die teure DB-Abfrage läuft trotzdem nur einmal.<br>Abschalten mit `Depends(get_value, use_cache=False)`.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6pUo9ukmsUXANAlu9J8nZ" -->
Wann hängt man eine Dependency in den **Dekorator** statt in die Signatur?::
Wenn man den **Rückgabewert nicht braucht**, sondern nur den Effekt — typisch Auth- und Header-Prüfungen:<br>`@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])`<br>`dependencies=` ist eine **Liste** von `Depends()`. Diese Dependencies dürfen Parameter deklarieren und Exceptions werfen; etwaige Rückgabewerte werden schlicht verworfen.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs0QmcGqRXcp0Jeh0ogjZwj" -->
Auf welchen drei Ebenen kann man Dependencies global setzen?::
**App-weit:** `app = FastAPI(dependencies=[Depends(verify_token)])`<br>**Pro Router:** `APIRouter(prefix="/items", dependencies=[Depends(get_token_header)])`<br>**Beim Einbinden:** `app.include_router(admin.router, dependencies=[Depends(...)])` — lässt den Router selbst unverändert.<br>Reihenfolge der Ausführung: **Router → Dekorator → Parameter**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZk8qIeJFmUCrVhuJZYrWT" -->
Wie schreibt man eine Dependency mit Setup und Teardown?::
Mit `yield` statt `return` — genau **ein** `yield` pro Dependency:<br>`def get_db():`<br>`    db = SessionLocal()`<br>`    try:`<br>`        yield db`<br>`    finally:`<br>`        db.close()`<br>FastAPI baut daraus intern einen Context Manager — den `@contextmanager`-Dekorator setzt man deshalb **nicht** selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs1khyJ517lnA1HD5ikW7OU" -->
Was passiert mit einer Exception, die **nach** dem `yield` einer Dependency ankommt?::
Sie kommt genau an der `yield`-Stelle wieder heraus und ist dort abfangbar — die Stelle, um technische Fehler in HTTP-Antworten zu übersetzen:<br>`try:`<br>`    yield "Rick"`<br>`except OwnerError as e:`<br>`    raise HTTPException(status_code=400, detail=f"Owner error: {e}")`<br>Das gilt auch für Exceptions aus der Endpunktfunktion selbst.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6Hteytd0pNZEf3Wh7UeHR" -->
Warum muss man eine abgefangene Exception in einer `yield`-Dependency **immer neu werfen**?::
Weil FastAPI sonst nichts davon merkt: Der Client bekommt zwar 500, aber im Server steht **kein einziges Log** — der Fehler verschwindet spurlos.<br>Also entweder eine `HTTPException` werfen oder mit nacktem `raise` weiterreichen.<br>Bis FastAPI 0.110.0 wurde die Exception automatisch weitergereicht; seitdem verhält es sich wie normales Python.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obscp1PKx6lJjaZByJ389Byy" -->
Wann läuft der Teardown-Code einer `yield`-Dependency, und wie steuert man das?::
Default ist **nach dem Senden der Antwort**. Seit FastAPI 0.121.0 über `scope` steuerbar:<br>`scope="request"` (Default) — Teardown nach der Antwort; nötig z. B. für eine DB-Session in einer `StreamingResponse`<br>`scope="function"` — Teardown direkt nach der Endpunktfunktion, **vor** dem Senden; gibt Ressourcen früher frei<br>Syntax: `Annotated[str, Depends(get_username, scope="function")]`
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6QErYsLZOt1VLn3LZeybK" -->
In welcher Reihenfolge wird bei verketteten `yield`-Dependencies aufgeräumt?::
In **umgekehrter** Reihenfolge zum Setup (LIFO, wie geschachtelte `with`-Blöcke): `c` → `b` → `a`.<br>FastAPI garantiert dabei, dass beim Teardown von `c` der Wert von `b` noch verfügbar ist — man kann also beim Aufräumen noch auf die Ressourcen der übergeordneten Dependency zugreifen.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsezB0cwqMPeLtr1G79jnvp" -->
Was ist der eigentliche Gewinn von Dependency Injection für die **Tests**?::
Dependencies lassen sich austauschen, ohne den Produktivcode anzufassen:<br>`app.dependency_overrides[get_db] = lambda: fake_db`<br>`app.dependency_overrides.clear()`<br>Damit laufen API-Tests komplett ohne echte Datenbank oder fremde API.<br>Wichtig: nach dem Test **zurücksetzen**, sonst leckt der Override in andere Tests.
<!-- basicblock-end -->

---

# Security

<!-- basicblock-start oid="ObsEZiTMan5W9DV0xrkKzUUx" -->
Wie läuft der **OAuth2 Password Flow** in FastAPI ab?::
1. Client schickt `username` und `password` als **Form-Data** an `/token`<br>2. Server prüft die Credentials und antwortet mit einem **Token**<br>3. Client speichert ihn und schickt bei jedem weiteren Request den Header `Authorization: Bearer <token>`<br>4. Eine Dependency-Kette prüft den Token und lädt den User<br>Das Passwort wandert also **nur einmal** über die Leitung.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsH6GIr71CTAB7C9WJZYnNl" -->
Was macht `OAuth2PasswordBearer(tokenUrl="token")` konkret?::
Drei Dinge:<br>1. Es liest den `Authorization`-Header, prüft ihn auf `Bearer <token>` und gibt den Token als **`str`** zurück — fehlt er, kommt sofort **401**, die eigene Funktion läuft gar nicht erst.<br>2. Es trägt das **Security Scheme** ins OpenAPI-Dokument ein → daher der **„Authorize"-Button** und die Schloss-Symbole in der Swagger UI.<br>3. Es **validiert den Token noch nicht** — das ist die eigene Aufgabe.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs3LMmSz1oabcDzL2guRVyW" -->
Erzeugt `tokenUrl="token"` einen Endpunkt?::
**Nein.** Es dokumentiert nur, **wohin** der Client seine Credentials schicken soll — den `/token`-Endpunkt schreibt man selbst.<br>Die **relative** URL (`"token"`) ist wichtig für den Betrieb hinter einem Reverse Proxy, damit der Pfad nicht am Proxy vorbei absolut wird.<br>Die camelCase-Schreibweise stammt aus der OpenAPI-Spezifikation.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsuuDoaqIvhs66UHk2ziQXG" -->
Wie sieht die typische **Dependency-Kette** für Authentifizierung aus?::
Drei Stufen, jede mit einer eigenen Verantwortung:<br>`oauth2_scheme` → liefert den rohen Token-String<br>`get_current_user` → dekodiert und prüft den Token, lädt den User<br>`get_current_active_user` → prüft, ob der User aktiv ist<br>Im Endpunkt steht dann nur noch `current_user: Annotated[User, Depends(get_current_active_user)]`.<br>Die Sicherheitslogik schreibt man **einmal**, tausende Endpunkte bleiben Dreizeiler.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6fiRwJRqk3hzYqsIumGz5" -->
Welche Felder hat `OAuth2PasswordRequestForm`?::
`username` und `password` (Pflicht — die Namen schreibt die Spezifikation vor, `email` wäre nicht konform), dazu optional `scope`, `grant_type`, `client_id`, `client_secret`.<br>Übertragen wird das als **Form-Data, nicht als JSON** — deshalb braucht es `python-multipart`.<br>Feinheit: `scope` ist **ein String** mit leerzeichengetrennten Werten, das Attribut `scopes` dagegen eine **Liste**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJfKFU2FQuMaUlP8DDeYDP" -->
Wie muss die Antwort des Token-Endpunkts aussehen?::
Exakt so, die Schlüsselnamen sind von der Spezifikation vorgeschrieben:<br>`{"access_token": "<token>", "token_type": "bearer"}`<br>Die Doku nennt das „fast das Einzige, was man selbst korrekt hinbekommen muss".<br>Deklariert wird der Endpunkt als `@app.post("/token")` mit `form_data: Annotated[OAuth2PasswordRequestForm, Depends()]`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsKa3pdpGkOFsGjUT3ztO0S" -->
Welcher Header gehört zu einer **401**-Antwort, und warum?::
`WWW-Authenticate: Bearer`<br>Die HTTP-Spezifikation verlangt, dass eine 401 mitteilt, **welches Authentifizierungsverfahren** erwartet wird; bei Bearer-Tokens ist der Wert `Bearer`.<br>`raise HTTPException(status_code=401, detail="…", headers={"WWW-Authenticate": "Bearer"})`<br>Technisch funktioniert es auch ohne, ist aber nicht standardkonform.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsAXUKhovIfZKUlu4c1gXWf" -->
Warum ist „Inactive user" **400** und nicht 401?::
Weil die **Authentifizierung erfolgreich war** — der Token ist gültig und der User identifiziert.<br>401 heißt „ich weiß nicht, wer du bist"; hier weiß der Server es sehr wohl, lehnt die Anfrage aber aus einem fachlichen Grund ab.<br>Das ist ein gutes Beispiel dafür, dass Statuscodes eine Bedeutung transportieren und nicht beliebig austauschbar sind.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRhvSgy3cR4lS44IA0tbRs" -->
Was ist ein **JWT**, und was ist der häufigste Denkfehler dabei?::
Ein JSON-Objekt, kompakt kodiert und **signiert** — Manipulation (z. B. am Ablaufdatum) wird dadurch erkannt.<br>Der Denkfehler: JWTs sind **signiert, nicht verschlüsselt**. Der Inhalt ist für jeden lesbar, der den Token in die Finger bekommt.<br>Also **niemals Geheimnisse** in die Claims schreiben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsS2tpnBkUoCTuWk2nXHlWC" -->
Welche Bibliotheken nutzt die aktuelle FastAPI-Doku für JWT und Passwort-Hashing?::
**PyJWT** für die Token (`uv add pyjwt`, dann `import jwt`).<br>**pwdlib** mit **Argon2** für Passwörter (`uv add "pwdlib[argon2]"`), erzeugt über `PasswordHash.recommended()`.<br>Das ältere `passlib`/bcrypt findet man noch in vielen Blogposts; die Doku empfiehlt es nur noch für **Alt-Hashes**, die man weiter prüfen muss.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsnZa3SC3sX8jJt8dNkTpPi" -->
Wie erzeugt und prüft man ein JWT?::
**Erzeugen:**<br>`to_encode = {"sub": username, "exp": datetime.now(timezone.utc) + timedelta(minutes=30)}`<br>`token = jwt.encode(to_encode, SECRET_KEY, algorithm="HS256")`<br>**Prüfen:**<br>`payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])`<br>Achtung: beim Dekodieren heißt es `algorithms` als **Liste**. Ein abgelaufener oder manipulierter Token wirft `InvalidTokenError` → 401.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsqhhqvEBEsjbrbPZIBSreg" -->
Was ist der `sub`-Claim, und welche Regel gilt für ihn?::
„Subject" — das **Wer** des Tokens, also die Identifikation des Nutzers.<br>Er muss ein **String** und **app-weit eindeutig** sein.<br>Bei mehreren Entitätstypen deshalb präfixen: `"username:johndoe"` statt nur `"johndoe"` — sonst kollidiert der User `foo` mit dem Auto `foo` und ein Token gilt plötzlich für die falsche Entität.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsZKRQ0D17PwD7fP2cyZC5C" -->
Warum hasht die Doku ein Passwort auch dann, wenn der User gar nicht existiert?::
Wegen **Timing-Angriffen**: Ohne den Dummy-Hash würde der Server bei unbekannten Nutzern messbar schneller antworten und damit verraten, **welche Accounts existieren** (User-Enumeration).<br>`if not user:`<br>`    verify_password(password, DUMMY_HASH)`<br>`    return False`<br>Beide Pfade brauchen so ungefähr gleich lang.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsA9JfF7vjkf3dvvQIdyz0W" -->
Wie funktionieren **Scopes**, und was unterscheidet `Security()` von `Depends()`?::
Scopes sind feingranulare Berechtigungen (`"items:read"`, `"me"`), die im Token stehen und pro Endpunkt gefordert werden.<br>`Security()` ist eine Unterklasse von `Depends` mit dem einen Extra-Parameter `scopes=` — dadurch landet die Anforderung auch in der OpenAPI-Doku:<br>`current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])]`<br>In der Auth-Dependency liest man sie über das Objekt `SecurityScopes` aus.
<!-- basicblock-end -->

---

# App-Struktur: Router, Middleware, CORS, Lifespan

<!-- basicblock-start oid="ObseKPMHjzliq4gTnhOQCMsG" -->
Was ist ein `APIRouter`?::
Ein „Mini-FastAPI" für Teildateien — dieselben Dekoratoren und Optionen, aber einhängbar:<br>`router = APIRouter(prefix="/items", tags=["items"], dependencies=[Depends(get_token_header)])`<br>`@router.get("/")` ergibt dann `GET /items/`.<br>Eingebunden wird er mit `app.include_router(items.router)`. In Flask heißt das Konzept **Blueprints**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRrpAn9eb2q7wZzpYmthi9" -->
Welche Stolperfallen gibt es beim Aufbau einer größeren Anwendung mit Routern?::
`prefix` darf **nicht** auf `/` enden.<br>In **jedem** Ordner braucht es eine `__init__.py`, sonst funktionieren die Imports nicht.<br>Immer das **Submodul** importieren (`from .routers import items, users`), nicht `router` direkt — sonst kollidieren die gleichnamigen Objekte.<br>Relative Imports: `.` gleiches Paket, `..` übergeordnet.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGMluav7OhCGtFP77WsrqD" -->
Was passiert mit `tags`, `responses` und `dependencies`, wenn sie sowohl am Router als auch am Endpunkt stehen?::
Sie werden **kombiniert**, nicht ersetzt — ein Endpunkt mit `tags=["custom"]` in einem Router mit `tags=["items"]` bekommt beide.<br>Ausführungsreihenfolge der Dependencies: **Router → Dekorator → Parameter**.<br>Setzt man Prefix und Dependencies erst beim `include_router()`, bleibt der Router selbst unverändert und ist mit anderer Auth mehrfach einbindbar.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs4K8GXaVjdhQVKekPP0MGZ" -->
Was ist der Unterschied zwischen `include_router()` und `app.mount()`?::
**`include_router()`** — die Endpunkte werden Teil derselben Anwendung und bleiben im **OpenAPI-Schema** sichtbar; FastAPI kombiniert Prefixe, Tags und Dependencies.<br>**`app.mount()`** — hängt eine **eigenständige** Anwendung unter einem Pfad ein (z. B. `StaticFiles`); sie ist isoliert und taucht **nicht** in der Doku auf.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsRULWBWZg0e5jZL00283Ok" -->
Wie schreibt man eine HTTP-Middleware?::
`@app.middleware("http")`<br>`async def add_process_time(request: Request, call_next):`<br>`    start = time.perf_counter()`<br>`    response = await call_next(request)`<br>`    response.headers["X-Process-Time"] = str(time.perf_counter() - start)`<br>`    return response`<br>Alles vor `await call_next(...)` ist die Request-Phase, alles danach die Response-Phase — und die Response **muss** zurückgegeben werden.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsgU2AW9IBKqemKmyh6hg7M" -->
Was bekommt eine Middleware **nicht** mit?::
Den Teardown von `yield`-Dependencies und **Background Tasks** — beide laufen **nach** der Middleware.<br>Eine Timing-Middleware misst also nicht mit, wie lange das Schließen der DB-Session gedauert hat.<br>Bei mehreren Middlewares gilt außerdem: **zuletzt hinzugefügt = außen**.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObszJmE8EhSXWZ8haKAvvAd3" -->
Was ist eine **Origin**, und was ist ein **Preflight**?::
Eine Origin ist die Kombination aus **Protokoll + Domain + Port** — `http://localhost`, `https://localhost` und `http://localhost:8080` sind drei **verschiedene** Origins.<br>Ein **Preflight** ist eine vorgeschaltete `OPTIONS`-Anfrage des Browsers mit den Headern `Origin` und `Access-Control-Request-Method`. Antwortet der Server nicht mit den passenden CORS-Headern, blockiert der Browser die eigentliche Anfrage.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsDlMSPamTqfRXzZMWrKlkG" -->
Wie konfiguriert man CORS, und welche Falle lauert bei `allow_credentials`?::
`app.add_middleware(CORSMiddleware, allow_origins=[...], allow_credentials=True, allow_methods=["*"], allow_headers=["*"])`<br>**Falle:** Sobald `allow_credentials=True` gesetzt ist (Cookies, `Authorization`-Header), darf **keines** von `allow_origins`/`allow_methods`/`allow_headers` `["*"]` sein — alles muss explizit aufgezählt werden.<br>Genau deshalb funktioniert eine CORS-Konfiguration oft lokal, aber mit Login plötzlich nicht mehr.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsGWD46CfJdyTWJcvRGktWX" -->
Wie führt man Code beim **Start** und beim **Herunterfahren** der Anwendung aus?::
Mit einem Lifespan-Context-Manager:<br>`@asynccontextmanager`<br>`async def lifespan(app: FastAPI):`<br>`    ml_models["x"] = load_model()`<br>`    yield`<br>`    ml_models.clear()`<br>`app = FastAPI(lifespan=lifespan)`<br>Alles vor `yield` läuft beim Start, alles danach beim Beenden. Ersetzt die veralteten `@app.on_event("startup"/"shutdown")`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsABG65QD6vRbO4RBSj9ahD" -->
Welche Metadaten kann man der App selbst mitgeben?::
`title`, `summary`, `description` (Markdown), `version` (die **eigene**, nicht die von OpenAPI), `terms_of_service`, `contact`, `license_info`.<br>Dazu `openapi_tags=[...]` für Beschreibungen der Tag-Gruppen — **die Reihenfolge dieser Liste bestimmt die Anzeigereihenfolge in der Doku**.<br>Doku-URLs sind konfigurierbar bzw. abschaltbar: `docs_url`, `redoc_url`, `openapi_url=None` (schaltet auch beide Oberflächen ab, weil sie darauf aufbauen).
<!-- basicblock-end -->

---

# Datenbank (SQLModel)

<!-- basicblock-start oid="ObsggGk4E6di8blYjqOuHZYw" -->
Welche Bibliothek nutzt das FastAPI-Tutorial inzwischen für Datenbanken?::
**SQLModel** — vom selben Autor wie FastAPI, gebaut auf **SQLAlchemy** (DB-Zugriff) und **Pydantic** (Validierung).<br>Der Clou: Eine Klasse kann gleichzeitig SQLAlchemy-Tabelle **und** Pydantic-Modell sein.<br>Zur Einordnung: Im eigenen Projekt nutzen wir SQLAlchemy direkt — die Muster (Session-Dependency, Modelltrennung) sind aber dieselben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHrFdOF73qOjw25GXafrnZ" -->
Was unterscheidet in SQLModel ein **Table Model** von einem **Data Model**?::
Allein das Argument `table=True`:<br>`class Hero(HeroBase, table=True)` → echte Datenbanktabelle<br>`class HeroPublic(HeroBase)` → ohne `table=True`, also **ein ganz normales Pydantic-Modell** für die API<br>Nur **eine** Klasse pro Entität bekommt `table=True`; die anderen sind Ein-/Ausgabemodelle.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsJCiQ7GH7xM3uGFUAbeWus" -->
Warum trennt man auch mit SQLModel in mehrere Modelle?::
Aus demselben Grund wie beim Response-Modell — **Kontrolle darüber, was rein- und rausgeht**:<br>`Hero` (Tabelle, enthält `secret_name`), `HeroPublic` (Ausgabe, **ohne** `secret_name`), `HeroCreate` (Eingabe), `HeroUpdate` (alle Felder optional für PATCH).<br>So kann der Client weder die `id` selbst setzen noch gelangt `secret_name` je in eine Antwort.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslBDq7hI2vFnu5F25gzss4" -->
Wie sieht das Standardmuster für die DB-Session als Dependency aus?::
`def get_session():`<br>`    with Session(engine) as session:`<br>`        yield session`<br>`SessionDep = Annotated[Session, Depends(get_session)]`<br>Danach genügt in jedem Endpunkt `session: SessionDep`.<br>Der `with`-Block plus `yield` garantieren **genau eine Session pro Request**, die danach zuverlässig geschlossen wird.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObslqpIsahFBsY9FE7AM628P" -->
Warum braucht SQLite bei FastAPI `connect_args={"check_same_thread": False}`?::
Weil SQLite standardmäßig verbietet, dass eine Verbindung aus mehreren Threads benutzt wird — und **ein einzelner Request kann bei FastAPI mehrere Threads berühren** (Endpunkt und Dependencies laufen ggf. im Threadpool).<br>`engine = create_engine(sqlite_url, connect_args={"check_same_thread": False})`<br>Unbedenklich ist das, weil die `yield`-Dependency ohnehin **eine Session pro Request** garantiert.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscOJ7PyHA2EO3vjUX78bSC" -->
Warum nutzt das SQLModel-Kapitel `response_model=HeroPublic` statt der Rückgabetyp-Annotation?::
Weil der Handler tatsächlich ein `Hero` (das Tabellenmodell) zurückgibt, nicht ein `HeroPublic`.<br>Mit `-> HeroPublic` würde der Typechecker zu Recht meckern; mit `response_model=` bleibt die Signatur ehrlich, und FastAPI validiert und filtert trotzdem auf `HeroPublic`.<br>Das ist der Standardfall für „Rückgabetyp und Response-Modell weichen ab".
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8t004Mr3hDgy2gFNWtCUO" -->
Warum ist `SQLModel.metadata.create_all(engine)` beim Start nur eine Abkürzung?::
Weil es Tabellen nur **anlegt**, aber bestehende nie **verändert** — jede spätere Schemaänderung (neue Spalte, geänderter Typ) wird stillschweigend ignoriert.<br>In Produktion gehören Schemaänderungen deshalb in **Migrationen** (Alembic), nicht in den Anwendungsstart.<br>Für ein Tutorial-Beispiel ist die Abkürzung in Ordnung, für ein echtes Projekt nicht.
<!-- basicblock-end -->

---

# Testen

<!-- basicblock-start oid="Obsu6nOB03GYbO9OuiP4OllP" -->
Wie testet man eine FastAPI-Anwendung?::
`from fastapi.testclient import TestClient`<br>`client = TestClient(app)`<br>`def test_read_main():`<br>`    response = client.get("/")`<br>`    assert response.status_code == 200`<br>`    assert response.json() == {"msg": "Hello World"}`<br>Der `TestClient` basiert auf **HTTPX** und hat eine Requests-ähnliche API.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsM76Dyh8MKZeRT4mIcL5S4" -->
Warum startet der `TestClient` keinen Server, und warum ist das gut?::
Er ruft die Anwendung **direkt über die ASGI-Schnittstelle** auf, ohne Netzwerk, Port oder Prozess.<br>Dadurch sind API-Tests schnell genug, um wie Unit-Tests behandelt zu werden — kein Warten auf einen Serverstart, keine Portkonflikte, keine Flakiness.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs6L5TKtJMRkWOMJxWJRHOt" -->
Warum schreibt man Testfunktionen als `def` und nicht als `async def`, obwohl die Endpunkte `async` sind?::
Weil der `TestClient` die Asynchronität selbst kapselt — die Aufrufe werden **nicht** `await`ed.<br>Genau dadurch funktioniert pytest ohne jedes Zusatz-Plugin.<br>Für echte `async def`-Tests braucht man stattdessen `httpx.AsyncClient` mit `ASGITransport(app=app)` und `@pytest.mark.anyio`.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsMQLkgmFcVzRNxokeiVl7b" -->
Wie übergibt man dem `TestClient` Body, Header und Cookies?::
Alles direkt wie bei HTTPX:<br>Query-/Pfadparameter → in den URL-String<br>JSON-Body → `json={"name": "Foo"}`<br>Formulardaten → `data={...}`<br>Header → `headers={"X-Token": "..."}`<br>Cookies → `cookies={...}`<br>Wichtig: Der Client erwartet **JSON-fähige Daten, keine Pydantic-Modelle** — vorher durch `jsonable_encoder` schicken.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs74VR5xE1EN6nO6sLNtnQZ" -->
Wie testet man einen Endpunkt, der eine Datenbank braucht, ohne echte Datenbank?::
Über `dependency_overrides` — das ist der praktische Hauptgewinn der Dependency Injection:<br>`app.dependency_overrides[get_session] = lambda: fake_session`<br>Nach dem Test wieder aufräumen: `app.dependency_overrides.clear()`<br>Als pytest-Fixture in `conftest.py` ist das der natürliche Ort dafür — Bezug zu Kapitel 1.3.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsbbbgYZu25rHOQd5rhS2Sq" -->
Warum startet man Uvicorn zum Debuggen aus der eigenen Datei heraus?::
`if __name__ == "__main__":`<br>`    uvicorn.run(app, host="0.0.0.0", port=8000)`<br>Weil der Debugger so **das eigene Programm** startet statt eines externen Serverprozesses — nur dann greifen Breakpoints.<br>Der `__name__`-Test verhindert zugleich, dass beim bloßen Importieren des Moduls versehentlich ein Server hochfährt.
<!-- basicblock-end -->

---

# Was es noch gibt

<!-- basicblock-start oid="ObsSERE1cWos2aytwsDlyeD6" -->
Was sind **Background Tasks**, und wo ist ihre Grenze?::
Arbeit, die **nach** dem Senden der Antwort im selben Prozess läuft — der Client muss nicht warten:<br>`async def notify(email: str, background_tasks: BackgroundTasks):`<br>`    background_tasks.add_task(write_notification, email)`<br>Gut für Leichtgewichtiges (E-Mail, Log). **Grenze:** kein eigener Worker, kein Retry, weg bei Neustart — für echte Jobs nimmt man **Celery** mit Redis/RabbitMQ.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs8FSl5x9jaDnLhsDwxQ4Dr" -->
Wie nimmt man Formulardaten und Datei-Uploads entgegen?::
`username: Annotated[str, Form()]` für Formularfelder, `file: UploadFile` für Dateien (`list[UploadFile]` für mehrere).<br>Beides braucht **`python-multipart`**.<br>`UploadFile` ist besser als `bytes`: Es liegt gespoolt im Speicher und wird bei großen Dateien auf die Platte ausgelagert, statt alles komplett einzulesen. Es bietet `.filename`, `.content_type` und `await .read()`.<br>**Form-Daten und JSON-Body lassen sich nicht mischen** — das ist eine HTTP-Beschränkung, keine von FastAPI.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obssy0Os3wvzGwTy8GnM9S3N" -->
Wie streamt man eine Antwort schrittweise statt am Stück?::
Überraschend einfach — **kein `StreamingResponse` nötig**, man tauscht `return` gegen `yield` und annotiert den Rückgabetyp:<br>`@app.get("/items/stream")`<br>`async def stream_items() -> AsyncIterable[Item]:`<br>`    for item in items:`<br>`        yield item`<br>Das ergibt **JSON Lines** (`application/jsonl`): ein Objekt pro Zeile, der Client verarbeitet Element *n*, während der Server *n+1* baut.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obs9ze7IHKboTGz3XyUx1D9j" -->
Was sind **Server-Sent Events**, und wie unterscheiden sie sich von WebSockets?::
**SSE** — `text/event-stream`, **unidirektional** Server → Client, vom Browser per `EventSource` direkt konsumierbar. In FastAPI über `response_class=EventSourceResponse` aus `fastapi.sse` plus `yield`. Keep-Alive-Ping alle 15 s ist eingebaut.<br>**WebSockets** — **bidirektionale** Dauerverbindung über `@app.websocket(...)`, mit `await websocket.accept()` und einer Schleife aus `receive_text()`/`send_text()`.<br>Faustregel: Nur Server sendet → SSE. Beide Seiten senden → WebSocket.
<!-- basicblock-end -->

<!-- basicblock-start oid="Obss5jgOV4fXGVJgaCaD5Muo" -->
Wie liefert man ein gebautes Frontend oder statische Dateien aus?::
Statische Dateien: `app.mount("/static", StaticFiles(directory="static"), name="static")` — eine eigenständige, **isolierte** App, die nicht in der Doku auftaucht.<br>Für ein echtes Frontend gibt es inzwischen `app.frontend("/", directory="dist", fallback="index.html")` — nutzt intern `StaticFiles`, unterstützt aber zusätzlich **Client-seitiges Routing**.<br>Endpunkte werden dabei immer **zuerst** geprüft.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsTy6TcBSv5iFRa6Vz9ZOyJ" -->
Wie verwaltet man Konfiguration und Secrets?::
Mit `pydantic-settings`:<br>`class Settings(BaseSettings):`<br>`    model_config = SettingsConfigDict(env_file=".env")`<br>`    database_url: str`<br>Dazu `@lru_cache` auf einer `get_settings()`-Funktion plus `Depends` — dann wird das Objekt nur einmal gebaut und bleibt trotzdem **im Test überschreibbar**.<br>Größter Gewinn: Fehlt eine Pflichtvariable, scheitert der Prozess **sofort beim Start** statt mitten in der ersten Anfrage.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObsHn8yzfRWoWMXlq7bguUxn" -->
Wozu dient `response_class=` und wann gibt man eine `Response` direkt zurück?::
`response_class=` legt fest, in welchen Antworttyp der Rückgabewert verpackt wird, und setzt zugleich den `media_type` in der Doku: `HTMLResponse`, `PlainTextResponse`, `FileResponse`, `StreamingResponse`, `RedirectResponse`.<br>Gibt man ein `Response`-Objekt **direkt** zurück, reicht FastAPI es unverändert durch — **ohne** Validierung, Filterung und Doku.<br>Also nur für Sonderfälle wie eigene Header oder Cookies, und der Inhalt muss selbst JSON-tauglich gemacht werden (`jsonable_encoder`).
<!-- basicblock-end -->

<!-- basicblock-start oid="Obsp9pvDLAGMbqYvUToDpISC" -->
Wozu dient der Parameter `responses={}` am Dekorator?::
Er dokumentiert **weitere Statuscodes** samt Antwortmodell in OpenAPI:<br>`@app.get("/items/{id}", response_model=Item, responses={404: {"model": Message}})`<br>**Wichtig:** Das ist **reine Doku** — FastAPI validiert diese Zusatzantworten nicht und erzeugt sie auch nicht; man muss sie im Code selbst zurückgeben.
<!-- basicblock-end -->

<!-- basicblock-start oid="ObscPlHP7pdyXwqHR3ehG6LH" -->
Was gewinnt man dadurch, dass FastAPI ein vollständiges OpenAPI-Dokument erzeugt?::
Aus `/openapi.json` lässt sich ein **vollständig typisierter Client** generieren — z. B. TypeScript über Hey API (`npx @hey-api/openapi-ts`).<br>Das Frontend bekommt dadurch Autovervollständigung für Methoden, Payloads und Antworten, und Typfehler fallen schon beim Build auf statt in Produktion.<br>Damit die Methodennamen lesbar werden, lohnt sich `generate_unique_id_function` — sonst heißen sie `createItemItemsPost`.
<!-- basicblock-end -->
