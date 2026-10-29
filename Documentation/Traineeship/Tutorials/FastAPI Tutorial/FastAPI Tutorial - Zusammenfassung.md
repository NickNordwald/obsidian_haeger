---
tags:
  - documentation
---

Verdichtete Zusammenfassung des offiziellen **[FastAPI Tutorial / User Guide](https://fastapi.tiangolo.com/tutorial/)** (42 Kapitel), durchgearbeitet im August 2026. Ziel: das Gelernte behalten, ohne die Doku nochmal komplett lesen zu müssen.

> [!info] Wie diese Notiz gedacht ist
> **Bewusst gekürzt.** Wiederkehrende Muster werden **einmal** erklärt und danach nur noch als Abweichung notiert — `Path`, `Query`, `Header`, `Cookie`, `Body`, `Form`, `File` sind mechanisch dasselbe und stehen deshalb in *einem* Abschnitt statt in sieben. Randthemen bekommen einen Überblick und einen Doku-Link statt eines Beispiels.
>
> **Abgrenzung zu den Nachbarnotizen:**
> - [[FastAPI, Uvicorn & Pydantic - Syntax|FastAPI, Uvicorn & Pydantic – Syntax]] → Spickzettel für das eigene Projekt (Projektstruktur, SQLAlchemy-Session, pydantic-settings).
> - [[1.5 REST APIs & Layered Architecture - Vertiefung#FastAPI + Uvicorn + Pydantic in Python|Vertiefung 1.5]] → Einordnung und Konzepte (ASGI, Starlette, Schichtenarchitektur, REST-Theorie).
> - **Diese Notiz** → was das Tutorial selbst durchgeht, in der Reihenfolge des Frameworks.
>
> Stand: **Pydantic v2**, **FastAPI ≥ 0.115**, Python 3.10+ (`X | None` statt `Optional[X]`).

# Inhalt

- [[#1. Das Grundprinzip — die Typannotation ist die Spezifikation]]
- [[#2. Parameter — ein Muster für alles]]
- [[#3. Pydantic-Modelle als Request Body]]
- [[#4. Der Rückweg — Response-Modelle und Statuscodes]]
- [[#5. Fehlerbehandlung]]
- [[#6. Dependencies — das Herzstück]]
- [[#7. Security]]
- [[#8. Struktur einer größeren Anwendung]]
- [[#9. Datenbank — SQLModel]]
- [[#10. Testen]]
- [[#11. Kurz erwähnt]]
- [[#12. Spickzettel]]

---

# 1. Das Grundprinzip — die Typannotation ist die Spezifikation

Wenn man **eine** Sache aus dem Tutorial mitnimmt, dann diese: In FastAPI schreibt man keine Validierung, keine Parser und keine Doku. Man schreibt eine **Typannotation**, und daraus zieht das Framework vier Dinge gleichzeitig:

```
        def read_item(item_id: int, q: str | None = None)
                              └─┬─┘
                                │
        ┌───────────────┬───────┴────────┬─────────────────┐
        ▼               ▼                ▼                 ▼
    Parsing         Validierung      Doku (OpenAPI)   Editor-Support
  "3" → 3          "foo" → 422      type: integer     Autocomplete
```

Das ist der Grund, warum FastAPI-Code so kurz ist — und warum ein Tippfehler in einer Annotation weiter reicht, als man denkt.

**Die Vokabeln des Tutorials:**

| Begriff | Bedeutung |
|---|---|
| **Path** | der Pfadteil der URL, z. B. `/items/{item_id}` |
| **Operation** | die HTTP-Methode: `get`, `post`, `put`, `delete`, … |
| **Path Operation** | Pfad + Methode zusammen — das, was der Dekorator registriert |
| **Path Operation Function** | die dekorierte Python-Funktion darunter |
| **API-Schema** | OpenAPI — welche Pfade/Parameter gibt es (`/openapi.json`) |
| **Daten-Schema** | JSON Schema — wie sieht ein einzelnes Objekt aus (steckt im OpenAPI-Dokument) |

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

Starten mit `fastapi dev main.py` (das neue CLI) oder klassisch `uvicorn main:app --reload`. Doku dann unter `/docs` (Swagger UI) und `/redoc`.

> [!tip] `def` oder `async def`?
> Beides erlaubt. **`def`** wird von FastAPI in einen Threadpool ausgelagert — blockierender Code (synchrones SQLAlchemy!) ist dort unschädlich. **`async def`** läuft direkt im Event Loop, dort darf nichts blockieren. Im Zweifel `def`.

**Zwei Fallen bei Pfadparametern:**

1. **Die Reihenfolge der Registrierung entscheidet.** Routen werden von oben nach unten geprüft, die erste passende gewinnt. `/users/me` muss **vor** `/users/{user_id}` stehen, sonst landet `"me"` als `user_id` im Handler.
2. **Pfade in Pfaden** brauchen den Starlette-Konverter `:path` — `@app.get("/files/{file_path:path}")`.

Feste Auswahlwerte über ein `str`-Enum, das dann auch in der Doku als `enum` auftaucht:

```python
from enum import Enum

class ModelName(str, Enum):      # von str erben → sauber serialisiert & dokumentiert
    alexnet = "alexnet"
    resnet = "resnet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet: ...
```

---

# 2. Parameter — ein Muster für alles

Das Tutorial verteilt das auf ~12 Kapitel. Es ist aber **ein** Mechanismus.

## 2.1 Woher kommt ein Parameter? — die Entscheidungsregel

FastAPI schaut sich jeden Funktionsparameter an und entscheidet in dieser Reihenfolge:

```
1. Name kommt im Pfad-String vor?          →  PATH-Parameter
2. Typ ist "singulär" (int, str, bool,     →  QUERY-Parameter
   float, UUID, datetime, …)?
3. Typ ist ein Pydantic-Modell?            →  BODY
                                    ┕━ außer ein Marker sagt etwas anderes
```

Der **Marker** im `Annotated` überschreibt diese Regel immer. Und genau daher kommen die Sonderfälle, die im Tutorial wie eigene Themen aussehen:

- Ein *singulärer* Typ + `Body()` → landet im Body statt in der Query.
- Ein *Pydantic-Modell* + `Query()`/`Header()`/`Cookie()` → wird zu einer **Gruppe** von Einzelparametern (siehe [[#2.4 Parameter-Modelle]]).
- Eine `list` ohne Marker wäre ein Body — für Mehrfach-Query-Parameter ist `Query()` deshalb **Pflicht**.

## 2.2 Die Schreibweise

```python
from typing import Annotated
from fastapi import Query

q: Annotated[str | None, Query(max_length=50)] = None
#            └───┬────┘  └────────┬──────────┘   └─┬─┘
#              Typ            Marker + Regeln    Default
```

**Die Default-Regel ist der ganze Trick:**

| Schreibweise | Bedeutung |
|---|---|
| `q: Annotated[str, Query()]` | **Pflicht** (kein Default) |
| `q: Annotated[str \| None, Query()] = None` | optional |
| `q: Annotated[str, Query()] = "fixed"` | optional mit Default |
| `q: Annotated[str \| None, Query()]` | **Pflicht**, darf aber `null` sein |

> [!warning] Häufige Verwechslung
> `str | None` macht einen Parameter **nicht** optional — das tut allein der Defaultwert. Die Union sagt nur: `None` ist ein zulässiger *Wert*.

> [!warning] Kein `default=` innerhalb von `Annotated`
> ```python
> q: Annotated[str, Query(default="rick")] = "morty"   # ✗ Fehler
> q: Annotated[str, Query()] = "rick"                  # ✓
> q: str = Query(default="rick")                       # alter Stil (vor 0.95), läuft noch
> ```
> Der `Annotated`-Stil ist seit 0.95 empfohlen: Der echte Python-Default steht an der echten Default-Stelle, die Funktion bleibt außerhalb von FastAPI normal aufrufbar, und `Annotated` kann Metadaten für andere Tools tragen. Ebenso überholt: `Query(...)` mit Ellipsis für „Pflicht" — heute schreibt man einfach keinen Default hin.

## 2.3 Die sieben Marker — nur die Unterschiede

Alle kommen aus `fastapi`, alle nehmen dieselben Optionen. Was sie unterscheiden:

| Marker | Liest aus | Besonderheit |
|---|---|---|
| `Path()` | Pfadsegment | **immer Pflicht** (ein Pfad hat das Segment ja) |
| `Query()` | Query-String | der Default für einfache Typen; `list[str]` für `?t=a&t=b` |
| `Header()` | HTTP-Header | `_` → `-` automatisch (`user_agent` liest `user-agent`); `convert_underscores=False` schaltet es ab; `list[str]` für mehrfach gesendete Header |
| `Cookie()` | Cookie-Header | Marker zwingend, sonst wäre es Query |
| `Body()` | JSON-Body | `embed=True` verpackt ein einzelnes Modell unter seinem Parameternamen |
| `Form()` | Formulardaten | braucht `python-multipart`; **nicht** mit JSON-Body kombinierbar |
| `File()` | Multipart-Upload | braucht `python-multipart`; besser `UploadFile` als `bytes` |

**Die gemeinsame Optionsliste** (gilt für alle sieben):

| Gruppe | Optionen |
|---|---|
| Zahlen | `gt`, `ge`, `lt`, `le`, `multiple_of`, `max_digits`, `decimal_places` |
| Strings/Sequenzen | `min_length`, `max_length`, `pattern` (früher `regex`, deprecated) |
| Doku | `title`, `description`, `examples`, `openapi_examples`, `deprecated`, `include_in_schema` |
| Namen | `alias` (externer Name, z. B. `item-query`), `validation_alias`, `serialization_alias` |

```python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="Item-ID", ge=1)],
    q: Annotated[str | None, Query(alias="item-query", max_length=50,
                                   pattern="^fixedquery$")] = None,
    tags: Annotated[list[str], Query()] = [],          # ?tags=a&tags=b
    user_agent: Annotated[str | None, Header()] = None,
    session_id: Annotated[str | None, Cookie()] = None,
): ...
```

> [!note] Reihenfolge der Parameter
> Mit `Annotated` **egal**. Der alte Trick mit dem führenden `*` (`def f(*, item_id: int = Path(...), q: str)`) war nur nötig, weil Python keinen Parameter ohne Default nach einem mit Default erlaubt — im alten Stil war `= Path(...)` ja ein Default.

## 2.4 Parameter-Modelle

Seit **0.115**: mehrere zusammengehörige Query-/Cookie-/Header-Parameter als Pydantic-Modell bündeln — wiederverwendbar und an einer Stelle validiert. Jedes Feld bleibt nach außen ein **eigener** Parameter.

```python
from pydantic import BaseModel, Field
from typing import Literal

class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}      # unbekannte Parameter → 422 statt still ignoriert

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"

@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]): ...
```

Beachte: Die Regeln stehen hier in Pydantics `Field`, nicht in `Query` — siehe unten.

## 2.5 `Field` vs. `Query`/`Path`/`Body`

Die Frage, die im Tutorial leicht untergeht:

| | Woher | Wofür |
|---|---|---|
| **`Field`** | `pydantic` | Regeln für **Attribute *innerhalb* eines Modells** |
| **`Query`, `Path`, …** | `fastapi` | Regeln für **Funktionsparameter** — *und* sie legen die Datenquelle fest |

Technisch ist es dieselbe Maschinerie: `Query` & Co. liefern Objekte, die von FastAPIs `Param` erben, das wiederum von Pydantics `FieldInfo`. Die Optionen sind deshalb identisch — nur der Ort und die Zusatzbedeutung „woher kommt der Wert" unterscheiden sich.

---

# 3. Pydantic-Modelle als Request Body

Ein Parameter, dessen Typ ein `BaseModel` ist, ist der Body. Mehr braucht es nicht.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item):
    return item.model_dump()
```

## 3.1 Verschachtelung

Beliebig tief, mit voller Validierung und Doku auf jeder Ebene:

```python
from pydantic import BaseModel, HttpUrl

class Image(BaseModel):
    url: HttpUrl              # wird als URL validiert
    name: str

class Item(BaseModel):
    tags: set[str] = set()    # Duplikate raus, JSON Schema: uniqueItems
    images: list[Image] | None = None

class Offer(BaseModel):
    items: list[Item]         # Item enthält list[Image] — beliebig tief
```

- `list` ohne Typparameter → Inhalt wird **nicht** geprüft. Immer `list[str]` schreiben.
- Der Body darf auch eine reine Liste sein: `async def f(images: list[Image])`.
- Für unbekannte Keys: `dict[int, float]` — JSON-Keys kommen als String und werden konvertiert.

## 3.2 Mehrere Body-Parameter

Sobald **mehr als ein** Body-Parameter da ist, verschachtelt FastAPI nach Parameternamen:

```python
async def update_item(item_id: int, item: Item, user: User,
                      importance: Annotated[int, Body()]): ...
```
```json
{ "item": { ... }, "user": { ... }, "importance": 5 }
```

Bei nur einem Modell liegen dessen Felder direkt auf oberster Ebene — `Body(embed=True)` erzwingt auch dort die Verschachtelung.

## 3.3 Weitere Datentypen

`UUID`, `datetime`, `date`, `time`, `timedelta`, `Decimal`, `bytes`, `frozenset` funktionieren direkt. Im Handler bekommt man das echte Python-Objekt und kann damit rechnen:

```python
async def read_items(
    start_datetime: Annotated[datetime, Body()],
    process_after: Annotated[timedelta, Body()],
):
    start_process = start_datetime + process_after   # echte datetime-Arithmetik
```

## 3.4 Beispiele in der Doku

Drei Ebenen, gleiche Wirkung, unterschiedlicher Ort:

```python
# im Modell
model_config = {"json_schema_extra": {"examples": [{"name": "Foo", "price": 35.4}]}}

# am Feld
name: str = Field(examples=["Foo"])

# am Parameter — Liste, landet im JSON Schema
item: Annotated[Item, Body(examples=[{"name": "Foo", "price": 35.4}])]

# am Parameter — Dict mit Metadaten, ergibt ein Dropdown in der Swagger UI
item: Annotated[Item, Body(openapi_examples={
    "normal": {"summary": "Normalfall", "value": {"name": "Foo", "price": 35.4}},
    "invalid": {"summary": "Ungültig", "value": {"name": "Baz", "price": "viel"}},
})]
```

Faustregel: `examples` reicht fast immer; `openapi_examples` nur, wenn man **mehrere benannte** Beispiele in der Swagger UI durchklicken will.

---

# 4. Der Rückweg — Response-Modelle und Statuscodes

## 4.1 Das Response-Modell filtert — und das ist Sicherheit, kein Komfort

Der wichtigste Gedanke des Tutorials in diesem Bereich: Das Response-Modell ist ein **Filter**. Was dort nicht deklariert ist, verlässt den Server nicht — auch nicht, wenn der Handler es versehentlich zurückgibt.

```python
class UserIn(BaseModel):     # rein: mit Passwort
    username: str
    password: str
    email: EmailStr

class UserOut(BaseModel):    # raus: ohne Passwort
    username: str
    email: EmailStr

@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user              # password wird herausgefiltert
```

Das ist das klassische Trio **`UserIn` / `UserOut` / `UserInDB`** (letzteres mit `hashed_password`, nie nach außen). Gemeinsame Felder kommen in eine `UserBase`, von der die drei erben.

**Zwei Schreibweisen:**

```python
async def read_item() -> ItemOut: ...                       # Rückgabetyp — bevorzugt
@app.get("/x", response_model=ItemOut)                       # Parameter
```

Der Rückgabetyp reicht meistens. `response_model=` braucht man, wenn beides **abweichen** soll — typisch beim Zurückgeben eines ORM-Objekts, das kein Pydantic-Modell ist (sonst meckert der Typechecker zu Recht). Sind **beide** gesetzt, gewinnt `response_model`. `response_model=None` schaltet Filterung, Validierung und Doku ganz ab.

Eleganter als zwei getrennte Modelle ist oft **Vererbung**: die Basisklasse als Rückgabetyp annotieren und die Subklasse zurückgeben — der Typechecker ist zufrieden (eine Subklasse *ist* ein `BaseUser`), FastAPI filtert trotzdem auf die Felder der Basis:

```python
class BaseUser(BaseModel):
    username: str
    email: EmailStr

class UserIn(BaseUser):
    password: str

@app.post("/user/")
async def create_user(user: UserIn) -> BaseUser:   # password fliegt raus
    return user
```

**Feinsteuerung:**

| Parameter | Wirkung |
|---|---|
| `response_model_exclude_unset=True` | nur Felder, die tatsächlich gesetzt wurden — keine Default-Rauschwerte |
| `response_model_exclude_defaults` / `_exclude_none` | analog |
| `response_model_include` / `_exclude` | einzelne Felder als `set[str]` ein-/ausschließen — **Abkürzung mit Haken:** das OpenAPI-Schema zeigt weiterhin das *vollständige* Modell. Sauberer ist ein eigenes Modell. |

## 4.2 Statuscodes

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
@app.delete("/items/{id}", status_code=status.HTTP_204_NO_CONTENT)
```

Die `status`-Konstanten sind reine Lesbarkeit (`201` tut dasselbe). **204** und **304** dürfen keinen Body haben.

## 4.3 PUT vs. PATCH — das Update-Rezept

`PUT` **ersetzt** vollständig: Felder, die der Client wegläßt, werden auf ihren Default zurückgesetzt. Das ist fast nie gewollt. Für partielle Updates:

```python
stored_item = Item(**stored_item_data)
update_data = item.model_dump(exclude_unset=True)   # ← nur wirklich gesendete Felder
updated_item = stored_item.model_copy(update=update_data)
```

`exclude_unset=True` ist hier das Entscheidende — ohne es kann man „Feld weggelassen" nicht von „Feld auf Default gesetzt" unterscheiden.

## 4.4 `jsonable_encoder`

Wandelt beliebige Objekte (Pydantic-Modelle, `datetime`, `UUID`, …) in reine JSON-taugliche Python-Typen um. Braucht man immer dann, wenn man an FastAPIs automatischer Serialisierung **vorbei** arbeitet — z. B. beim Schreiben in eine JSON-Spalte oder beim direkten Zurückgeben einer `JSONResponse`.

## 4.5 Metadaten am Endpunkt

```python
@app.post(
    "/items/",
    tags=["items"],                       # Gruppierung in der Swagger UI
    summary="Ein Item anlegen",
    response_description="Das angelegte Item",
    deprecated=True,
)
async def create_item(item: Item):
    """Diese Docstring wird zur **Beschreibung** — Markdown erlaubt."""
```

---

# 5. Fehlerbehandlung

**Punktuell** — `HTTPException` mit Statuscode, `detail` (beliebiges JSON-taugliches Objekt) und optionalen Headern:

```python
raise HTTPException(status_code=404, detail="Item nicht gefunden",
                    headers={"X-Error": "..."})
```

**Zentral** — eigene Exceptions auf HTTP-Antworten abbilden. Das ist die Stelle, an der die Domänenfehler aus [[1.4 Logging & Exceptionhandling - Vertiefung|Kapitel 1.4]] an der Schichtgrenze übersetzt werden:

```python
@app.exception_handler(NotFoundError)
def handle_not_found(request: Request, exc: NotFoundError):
    return JSONResponse(status_code=404, content={"detail": str(exc)})
```

Ebenso überschreibbar sind die eingebauten Handler für `RequestValidationError` (die 422er) und `HTTPException`. Die Original-Handler lassen sich dabei aus `fastapi.exception_handlers` importieren und weiterverwenden (z. B. um vorher zu loggen).

> [!tip] Feinheit beim Überschreiben
> Werfen sollte man weiterhin FastAPIs `HTTPException` (deren `detail` beliebiges JSON erlaubt, Starlettes nur Strings) — den **Handler** aber für `StarletteHTTPException` registrieren:
> ```python
> from starlette.exceptions import HTTPException as StarletteHTTPException
> ```
> Sonst greift er nicht bei Exceptions, die aus Starlette-internem Code kommen.
>
> Und: `str(exc)` einer `RequestValidationError` enthält Dateiname und Zeilennummer — nicht ungefiltert an den Client geben, sondern `exc.errors()` gezielt auswerten.

> [!note] `RequestValidationError` vs. `ValidationError`
> `RequestValidationError` → die **Anfrage** war ungültig, der Client ist schuld → **422**.
> Ein Pydantic-`ValidationError` beim Bauen der **Antwort** → der Server ist schuld → **500**, weil die API sonst kaputte Daten ausliefern würde.

---

# 6. Dependencies — das Herzstück

Der konzeptionell wertvollste Teil des Tutorials. Eine Dependency ist eine Funktion (oder irgendein Callable), deren Signatur FastAPI **genauso auswertet wie die eines Endpunkts** — mit Parametern, Validierung und Doku. Das Ergebnis wird injiziert.

```python
from fastapi import Depends

async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

CommonsDep = Annotated[dict, Depends(common_parameters)]    # wiederverwendbarer Alias

@app.get("/items/")
async def read_items(commons: CommonsDep): ...
```

Der Type-Alias ist der empfohlene Stil — er spart die Wiederholung in jedem Handler.

> [!warning] Die Funktion wird **nicht** aufgerufen
> `Depends(common_parameters)` — **ohne** Klammern. `Depends(common_parameters())` ist falsch: `Depends` bekommt das Callable selbst, FastAPI ruft es pro Request auf. Nur das Argument *innerhalb* von `Depends()` zählt; der Typ davor im `Annotated` ist reine Editor-Information.

**Die Varianten in Kürze:**

| Variante | Wozu |
|---|---|
| **Klasse als Dependency** | `Annotated[CommonQueryParams, Depends(CommonQueryParams)]`, kurz `Depends()` — man bekommt ein typisiertes Objekt statt eines `dict` |
| **Sub-Dependencies** | Dependencies dürfen selbst welche haben. Ergebnisse werden **pro Request gecacht**; `Depends(f, use_cache=False)` erzwingt Neuberechnung |
| **Im Dekorator** | `@app.get("/", dependencies=[Depends(verify_token)])` — wenn man den Rückgabewert nicht braucht, nur den Effekt (Auth, Rate Limit) |
| **Global** | `FastAPI(dependencies=[...])` oder `APIRouter(dependencies=[...])` — für alle Endpunkte |

## 6.1 Dependencies mit `yield`

Setup vor dem `yield`, Teardown danach — Sessions, Transaktionen, Dateien:

```python
def get_db():
    db = SessionLocal()
    try:
        yield db          # ← wird injiziert
    finally:
        db.close()        # ← Teardown
```

Genau **ein** `yield` pro Dependency. FastAPI baut daraus intern einen Context Manager (`@contextlib.contextmanager`) — den Dekorator setzt man deshalb **nicht** selbst. Bei verketteten Dependencies wird in umgekehrter Reihenfolge aufgeräumt (LIFO, wie geschachtelte `with`-Blöcke), und beim Teardown von `c` ist der Wert von `b` noch verfügbar.

**Exceptions kommen am `yield` wieder heraus.** Wirft die Endpunktfunktion (oder eine spätere Dependency) etwas, landet das an der `yield`-Stelle und ist dort abfangbar — die Stelle, um technische Fehler in HTTP-Antworten zu übersetzen:

```python
def get_username():
    try:
        yield "Rick"
    except OwnerError as e:
        raise HTTPException(status_code=400, detail=f"Owner error: {e}")
```

> [!warning] Immer neu werfen
> Fängt man eine Exception ab und wirft **nicht** neu, merkt FastAPI nichts davon — der Client bekommt zwar 500, aber im Server steht **kein Log**. Also entweder eine `HTTPException` werfen oder mit nacktem `raise` weiterreichen. (Bis 0.110.0 hat FastAPI das automatisch getan, seitdem verhält es sich wie normales Python.)

**Wann der Teardown läuft** — Default ist **nach dem Senden der Antwort**. Seit 0.121.0 steuerbar:

| `scope` | Teardown |
|---|---|
| `"request"` (Default) | nach dem Senden der Antwort — nötig z. B. für eine DB-Session in einer `StreamingResponse` |
| `"function"` | direkt nach der Endpunktfunktion, **vor** dem Senden — gibt Ressourcen früher frei |

```python
username: Annotated[str, Depends(get_username, scope="function")]
```

## 6.2 Warum das der eigentliche Gewinn ist

Weil sich Dependencies in Tests austauschen lassen, ohne den Produktivcode anzufassen — direkter Bezug zu [[1.3 Testing mit PyTest - Vertiefung|Kapitel 1.3]]:

```python
app.dependency_overrides[get_db] = lambda: fake_db
# ... Test ...
app.dependency_overrides.clear()      # sonst leckt es in andere Tests
```

---

# 7. Security

Das Tutorial baut in vier Schritten einen kompletten OAuth2-Password-Flow mit JWT. Die Bausteine:

```
Client                        FastAPI
  │  POST /token               │
  │  username, password  ─────►│  OAuth2PasswordRequestForm
  │                            │  ├─ User laden, Passwort verifizieren
  │  ◄──── {"access_token":    │  └─ JWT signieren
  │         "token_type":      │
  │          "bearer"}         │
  │                            │
  │  GET /users/me             │
  │  Authorization: Bearer …  ►│  oauth2_scheme  → Token als str
  │                            │  get_current_user → JWT prüfen, User laden
  │                            │  get_current_active_user → aktiv?
```

**Das Schema-Objekt** — es holt nicht nur den Token aus dem Header, es trägt auch das Sicherheitsschema in OpenAPI ein und aktiviert damit den **„Authorize"-Button** in der Swagger UI:

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")   # liefert den rohen Token-String
```

**Die Depends-Kette** — jede Stufe eine eigene, testbare Verantwortung:

```python
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]) -> User:
    credentials_exception = HTTPException(
        status_code=401, detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},        # bei 401 vorgeschrieben
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")                  # "sub" = Subject = wer
    except InvalidTokenError:
        raise credentials_exception
    ...

async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)]
) -> User:
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user
```

**Der Token-Endpunkt** — `OAuth2PasswordRequestForm` ist ein Formular (kein JSON!) mit `username`, `password`, `scope`, `grant_type`, `client_id`/`client_secret`. Die Antwort **muss** exakt so aussehen, das schreibt die Spezifikation vor:

```python
@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    ...
    return {"access_token": token, "token_type": "bearer"}
```

**JWT und Passwörter** — die aktuelle Doku nutzt **PyJWT** (`uv add pyjwt`) und **pwdlib** mit Argon2 (`uv add "pwdlib[argon2]"`; das ältere `passlib`/bcrypt findet man noch in vielen Blogposts):

```python
to_encode = {"sub": username, "exp": datetime.now(timezone.utc) + timedelta(minutes=30)}
token = jwt.encode(to_encode, SECRET_KEY, algorithm="HS256")   # HS256, SECRET_KEY aus env!
```

> [!important] Die Kernpunkte, falls man den Rest vergisst
> - Passwörter werden **nie** gespeichert, nur **gehasht** — und Hashing ist absichtlich langsam.
> - Der Token gehört in den Header `Authorization: Bearer <token>`.
> - Ein JWT ist **signiert, nicht verschlüsselt** — jeder kann den Inhalt lesen. Keine Geheimnisse hineinschreiben.
> - `SECRET_KEY` gehört in die Konfiguration, nie in den Code (`openssl rand -hex 32`).
> - `sub` muss ein **String** und **app-weit eindeutig** sein — bei mehreren Entitätstypen präfixen (`"username:johndoe"`), sonst kollidiert User `foo` mit Auto `foo`.
> - Feingranulare Rechte laufen über **Scopes** mit `Security()` statt `Depends()` (Advanced Guide).

Zwei Details, die im Tutorial leicht überliest:

- **`tokenUrl="token"` erzeugt keinen Endpunkt.** Es dokumentiert nur, wohin der Client seine Credentials schicken soll — den `/token`-Endpunkt schreibt man selbst. Die relative URL ist wichtig für den Betrieb hinter einem Proxy.
- **Timing-Angriffe:** Existiert der User nicht, hasht die Doku trotzdem gegen einen Dummy-Hash. Sonst antwortet der Server bei unbekannten Nutzern messbar schneller — und verrät damit, welche Accounts es gibt.

```python
def authenticate_user(db, username: str, password: str):
    user = get_user(db, username)
    if not user:
        verify_password(password, DUMMY_HASH)   # gleiche Laufzeit
        return False
    ...
```

---

# 8. Struktur einer größeren Anwendung

`APIRouter` ist das „`FastAPI` für Teildateien" — dieselben Dekoratoren, aber einhängbar:

```
app/
├── main.py            # FastAPI(), include_router(...)
├── dependencies.py
├── internal/
│   └── admin.py
└── routers/
    ├── items.py
    └── users.py
```

```python
# routers/items.py
router = APIRouter(
    prefix="/items",                          # darf NICHT auf "/" enden
    tags=["items"],
    dependencies=[Depends(get_token_header)], # gilt für alle Endpunkte des Routers
    responses={404: {"description": "Not found"}},
)

@router.get("/")            # → GET /items/
async def read_items(): ...

# main.py
app.include_router(items.router)
app.include_router(admin.router, prefix="/admin", tags=["admin"])   # Prefix nachträglich
```

- Immer das **Submodul** importieren (`from .routers import items, users`), nicht `router` direkt — sonst kollidieren die gleichnamigen Objekte.
- `prefix`/`tags`/`dependencies` erst beim `include_router()` anzuhängen lässt den Router selbst unverändert — derselbe Router ist so mit unterschiedlicher Auth mehrfach einbindbar.
- Tags und `responses` von Router und Endpunkt werden **kombiniert**, nicht ersetzt.
- Reihenfolge der Ausführung: **Router-Dependencies → Dekorator-`dependencies` → Parameter-Dependencies.**
- Ein Router ist **kein** Mount: er bleibt Teil des OpenAPI-Schemas (anders als `app.mount(...)`).

**Middleware** — läuft um *jede* Anfrage herum:

```python
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.perf_counter() - start)
    return response
```

**CORS** — nötig, sobald ein Browser-Frontend von einer anderen Origin (Schema + Host + Port!) zugreift. Der Browser schickt bei nicht-trivialen Anfragen erst einen **Preflight** (`OPTIONS`); fehlen die Header, blockiert er die eigentliche Anfrage:

```python
app.add_middleware(CORSMiddleware, allow_origins=["http://localhost:3000"],
                   allow_credentials=True, allow_methods=["*"], allow_headers=["*"])
```

> [!warning] Wildcard und Credentials schließen sich aus
> Sobald `allow_credentials=True` gesetzt ist (Cookies, `Authorization`-Header), darf **keines** von `allow_origins`/`allow_methods`/`allow_headers` `["*"]` sein — alles muss explizit aufgezählt werden. Genau das ist der Grund, warum eine CORS-Konfiguration „lokal funktioniert und mit Login plötzlich nicht mehr".

> [!note] Was Middleware *nicht* sieht
> Der Teardown von `yield`-Dependencies und Background Tasks läuft **nach** der Middleware. Eine Timing-Middleware misst also nicht mit, wie lange das Schließen der DB-Session gedauert hat. Bei mehreren Middlewares gilt: zuletzt hinzugefügt = außen.

**Lifespan** — Start-/Shutdown-Code (ersetzt die veralteten `@app.on_event(...)`):

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    ml_models["x"] = load_model()     # Startup
    yield
    ml_models.clear()                 # Shutdown

app = FastAPI(lifespan=lifespan)
```

**API-Metadaten** — `title`, `summary`, `description`, `version`, `contact`, `license_info`, dazu `openapi_tags` für Beschreibungen der Tag-Gruppen. Die Doku-URLs sind konfigurierbar bzw. abschaltbar: `docs_url`, `redoc_url`, `openapi_url=None`.

**Statische Dateien:**
```python
app.mount("/static", StaticFiles(directory="static"), name="static")
```

---

# 9. Datenbank — SQLModel

Das Tutorial-Kapitel „SQL (Relational) Databases" nutzt inzwischen **SQLModel** (vom selben Autor), nicht mehr pures SQLAlchemy. Der Clou: eine Klasse ist gleichzeitig SQLAlchemy-Tabelle *und* Pydantic-Modell.

```python
class HeroBase(SQLModel):
    name: str = Field(index=True)

class Hero(HeroBase, table=True):          # table=True → echte Tabelle
    id: int | None = Field(default=None, primary_key=True)
    secret_name: str

class HeroPublic(HeroBase):                # ohne table=True = normales Pydantic-Modell
    id: int

engine = create_engine(sqlite_url, connect_args={"check_same_thread": False})

def get_session():
    with Session(engine) as session:
        yield session

SessionDep = Annotated[Session, Depends(get_session)]
```

Die Modelltrennung ist dieselbe Idee wie in [[#4.1 Das Response-Modell filtert — und das ist Sicherheit, kein Komfort]]: `Hero` (Tabelle, enthält `secret_name`), `HeroPublic` (Ausgabe, ohne), `HeroCreate` (Eingabe), `HeroUpdate` (alle Felder optional).

> [!note] Für das eigene Projekt
> Wir nutzen **SQLAlchemy** direkt (Kapitel 1.2), nicht SQLModel. Übertragbar ist trotzdem alles Wesentliche: das `SessionDep`-Muster, `connect_args={"check_same_thread": False}` bei SQLite, und die Trennung Tabellen-/DTO-Modelle. Bei SQLAlchemy braucht das Ausgabemodell dafür `ConfigDict(from_attributes=True)`.

---

# 10. Testen

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_main():                      # normales def, kein async
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}

def test_create_item():
    response = client.post("/items/", json={"name": "Foo"},
                           headers={"X-Token": "secret"})
    assert response.status_code == 201
```

- `TestClient` startet **keinen Server** — er spricht die App direkt über ASGI an (intern `httpx`). Deshalb schnell genug für Unit-Test-Tempo.
- Testfunktionen sind ganz normale `def`, auch wenn die Endpunkte `async def` sind.
- Zusammen mit `dependency_overrides` laufen API-Tests komplett ohne Datenbank.
- In `async def`-Tests funktioniert `TestClient` nicht — dort `httpx.AsyncClient(transport=ASGITransport(app=app))` mit `@pytest.mark.anyio`.
- **Debugging:** `uvicorn.run(app)` unter `if __name__ == "__main__":`, damit der Debugger den Prozess selbst startet.

---

# 11. Kurz erwähnt

Für den Rest reicht zu wissen, **dass** es das gibt:

| Thema | Kern in einem Satz |
|---|---|
| **Background Tasks** | `BackgroundTasks` als Parameter, `.add_task(fn, args)` — läuft *nach* der Antwort, im selben Prozess. Für Leichtgewichtiges (E-Mail, Log); für echte Jobs → Celery. |
| **Form-/File-Uploads** | `Form()` und `UploadFile` (mit `.filename`, `.content_type`, `await .read()`), `list[UploadFile]` für mehrere. `python-multipart` nötig. |
| **Stream JSON Lines** (0.134+) | Ein JSON-Objekt pro Zeile (`application/jsonl`) statt eines großen Arrays — der Client verarbeitet Element *n*, während der Server *n+1* baut. Für LLM-Ausgaben, Logs, Telemetrie. |
| **Server-Sent Events** (0.135+) | `text/event-stream`, unidirektional Server→Client, vom Browser per `EventSource` direkt konsumierbar. Keep-Alive-Ping alle 15 s ist eingebaut. |
| **`app.frontend()`** | Ein gebautes Frontend (React/Vite/Astro …) ausliefern: `app.frontend("/", directory="dist", fallback="index.html")` — mit Unterstützung für Client-seitiges Routing. Endpunkte werden zuerst geprüft. |
| **WebSockets** | `@app.websocket(...)`, `await websocket.accept()`, dann Schleife aus `receive_text()`/`send_text()`; `WebSocketDisconnect` abfangen. Bidirektional, kein HTTP-Status. |
| **Custom Responses** | `response_class=HTMLResponse` / `PlainTextResponse` / `FileResponse` / `StreamingResponse` / `RedirectResponse`. |
| **Response direkt zurückgeben** | Ein `Response`-Objekt zurückgeben umgeht Validierung *und* Doku — nur für Sonderfälle (eigene Header, Cookies). |
| **`responses={}`** | Weitere Statuscodes in OpenAPI dokumentieren. Reine Doku, FastAPI validiert sie nicht. |
| **Settings** | `pydantic-settings` `BaseSettings` + `@lru_cache` + `Depends` — konfigurierbar und im Test überschreibbar. |
| **OAuth2 Scopes** | `Security()` statt `Depends()`, `SecurityScopes` für feingranulare Rechte. |
| **SDK-Generierung** | Aus `/openapi.json` typisierte Clients bauen (z. B. TypeScript via Hey API). |

Das Streaming ist eine Erwähnung wert, weil es überraschend einfach ist — **kein `StreamingResponse` nötig**, man tauscht nur `return` gegen `yield` und annotiert den Rückgabetyp:

```python
from collections.abc import AsyncIterable
from fastapi.sse import EventSourceResponse

@app.get("/items/stream")                                   # JSON Lines
async def stream_items() -> AsyncIterable[Item]:
    for item in items:
        yield item

@app.get("/items/sse", response_class=EventSourceResponse)  # Server-Sent Events
async def sse_items() -> AsyncIterable[Item]:
    for item in items:
        yield item
```

Die Typannotation lohnt sich auch hier: mit ihr validiert, filtert und serialisiert Pydantic (in Rust) — ohne sie greift der langsamere `jsonable_encoder`.

---

# 12. Spickzettel

| Ich will … | Syntax |
|---|---|
| Endpunkt | `@app.get("/pfad")` / `@router.post("")` |
| Pfadparameter | `/{id}` + `id: int` |
| Query, optional | `q: str \| None = None` |
| Query, Pflicht + Regeln | `q: Annotated[str, Query(min_length=3)]` |
| Query-Liste | `Annotated[list[str], Query()] = []` |
| Header | `Annotated[str \| None, Header()] = None` (`_` → `-`) |
| Body | Pydantic-Modell als Parameter |
| Einzelwert in den Body | `Annotated[int, Body()]` |
| Body-Modell einbetten | `Body(embed=True)` |
| Parameter gruppieren | `Annotated[MeinModell, Query()]` |
| Regeln im Modell | `Field(gt=0, max_length=20)` |
| Unbekannte Felder ablehnen | `model_config = {"extra": "forbid"}` |
| Antwort filtern | `-> ItemOut` oder `response_model=ItemOut` |
| Statuscode | `status_code=status.HTTP_201_CREATED` |
| Partielles Update | `.model_dump(exclude_unset=True)` + `.model_copy(update=...)` |
| Fehler | `raise HTTPException(404, "…")` |
| Eigene Exception abbilden | `@app.exception_handler(MeinFehler)` |
| Abhängigkeit | `Annotated[T, Depends(func)]` |
| Aufräumen nach Request | Dependency mit `yield` |
| Auth-Token | `Depends(OAuth2PasswordBearer(tokenUrl="token"))` |
| Router | `APIRouter(prefix=…, tags=…)` + `app.include_router(r)` |
| Start/Shutdown | `lifespan=` |
| Testen | `TestClient(app)` + `app.dependency_overrides` |
| Doku | `/docs` · `/redoc` · `/openapi.json` |

# Weiterführend

- [FastAPI Tutorial / User Guide](https://fastapi.tiangolo.com/tutorial/) — das Original
- [FastAPI Advanced User Guide](https://fastapi.tiangolo.com/advanced/) — der Rest aus [[#11. Kurz erwähnt]]
- [[FastAPI, Uvicorn & Pydantic - Syntax|FastAPI, Uvicorn & Pydantic – Syntax]] — projektbezogener Spickzettel
- [[1.5 REST APIs & Layered Architecture - Vertiefung|Vertiefung 1.5]] — Konzepte und Einordnung
- [[Documentation/Traineeship/Grundlagen der Software Entwicklung/Ressourcen und Quellen.md|Ressourcen und Quellen]], Abschnitt *FastAPI, Pydantic & OpenAPI*
