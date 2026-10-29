---
tags:
  - documentation
---

# Pricing-Dashboard — Plan & Syntax-Tutorial

Abhakbarer Bauplan für ein React-Frontend auf die Pricing-API aus [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/1.5 REST APIs & Layered Architecture - Vertiefung|Kapitel 1.5]]. Teil von [[React Crashkurs - Übersicht]]. Laufender Stand: [[Umsetzungsstand (Pricing-Dashboard)]].

> [!info] Archiviert am 20.10.2026: Plan erledigt
> Alle Schritte sind in den Phasen 0–6 umgesetzt, Stand und Zuordnung in [[Umsetzungsstand (Pricing-Dashboard)]].

> [!warning] Seit 30.09.2026 geänderter Modus: Claudian baut, Nick reviewt
> Die Schritte unten bleiben die inhaltliche Roadmap, werden aber in **Phasen** gebündelt von Claudian umgesetzt; du liest den Code über die [[Code-Review-Aufgaben (Pricing-Dashboard)|Leseaufgaben]]. Außerdem abweichend: Projektordner ist `~/HaegerConsulting/Projects/Interviews/ReactBasics/FrontEndBuchungsSystem/frontend`, Sprache **TypeScript**, und statt CORS (Schritt 0b) leitet der Vite-Proxy weiter. Phasen-Zuordnung und Entscheidungen: [[Umsetzungsstand (Pricing-Dashboard)]].

Jeder Schritt führt **genau eine** neue Technik ein, von einfach nach komplex. Die Syntax-Muster sind bewusst an einem **fremden Datenmodell** (Bücher) gezeigt — die Übertragung auf Produkte, Buchungen und Preisläufe ist der Lerneffekt.

> [!info] Arbeitsmodus
> **Du schreibst den Code.** Claudian gibt das Muster und reviewt — Standard ist ein Denkanstoß, keine Lösung. Anders als im Traineeship darfst du wegen des Zeitdrucks jederzeit eskalieren („gib mir die Lösung"), ohne dich zu rechtfertigen.

> [!important] Warum dieses Projekt und nicht ein Todo-Tutorial
> Im Gespräch wirst du gefragt, was du gebaut hast. „Ein Todo-Tutorial nachgeklickt" ist eine schwache Antwort. „Ich habe eine FastAPI-Anwendung, die ich selbst geschrieben habe, um ein React-Frontend ergänzt — mit Lade- und Fehlerzuständen, einem eigenen Hook fürs Datenholen und Polling auf lang laufende Jobs" ist eine starke. Das ist der eigentliche Grund für dieses Projekt.

## Zielbild

Eine einzelne Seite mit drei Bereichen:

1. **Produktliste** links — aus `GET /v1/products`, mit Suchfeld zum Filtern
2. **Detailbereich** rechts — Empfehlungen zum gewählten Produkt aus `GET /v1/recommendations`
3. **Aktionsleiste** oben — „Preislauf starten" (`POST /v1/pricing-runs`), danach Statusanzeige per Polling auf `GET /v1/pricing-runs/{id}`

Bewusst **eine** Seite: kein Router, keine Authentifizierungsmaske, kein CSS-Framework. Der Wert steckt in den React-Konzepten, nicht im Umfang.

## Das Backend, gegen das du baust

| | |
|---|---|
| Projekt | `~/HaegerConsulting/Projects/Traineeship/GrundlagenSoftwareEntwicklung/Docker` (aktuellster Stand) |
| Basis-URL | `http://localhost:8000/v1` |
| Auth | Header `X-API-Key` bei allem außer `/v1/health` |
| Schlüssel anlegen | `uv run pricing-apikey create frontend` |
| Endpunkte | `/products`, `/bookings`, `/capacity`, `/recommendations`, `/pricing-runs`, `/ingestion-runs`, `/health` |
| API-Doku | `http://localhost:8000/docs` — dein Nachschlagewerk für Feldnamen |

**Frontend-Projekt:** `~/HaegerConsulting/Projects/Interview_Deutz/pricing-frontend` — außerhalb des Vaults, analog zur Traineeship-Regel.

---

## Schritt 0 — Werkzeuge und ein leeres Projekt

**Neu: die Toolchain.** *(Tag 1)*

- [ ] Node prüfen: `node -v` und `npm -v`. Fehlt Node, über den Paketmanager oder <https://nodejs.org> installieren (LTS).
- [ ] Projekt anlegen:
      ```bash
      mkdir -p ~/HaegerConsulting/Projects/Interview_Deutz
      cd ~/HaegerConsulting/Projects/Interview_Deutz
      npm create vite@latest pricing-frontend -- --template react
      cd pricing-frontend && npm install && npm run dev
      ```
- [ ] Im Browser öffnen, in `src/App.jsx` einen Text ändern, zusehen wie die Seite sich ohne Reload aktualisiert (Hot Module Replacement).
- [ ] Alles aus `src/App.css` und `src/index.css` löschen bis auf das Nötigste — Styling ist hier Ablenkung.

> [!tip] Die Einordnung fürs Gespräch
> **Vite** ist der Build- und Dev-Server: es übersetzt JSX nach JavaScript, bündelt die Module und liefert im Dev-Betrieb Hot Reload. **npm** ist der Paketmanager — dein Gegenstück zu `uv`/`pip`, `package.json` das Gegenstück zu `pyproject.toml`, `package-lock.json` zu `uv.lock`. `node_modules/` ist das `.venv/`.

### Schritt 0b — CORS im Backend freischalten

**Das wird dich sonst in Schritt 3 aufhalten.** Dein Browser läuft auf `localhost:5173`, die API auf `localhost:8000` — verschiedene Origins. Ohne Freigabe blockiert der Browser die Antwort, und zwar **nachdem** die API korrekt geantwortet hat. Typisches Symptom: in der Browser-Konsole ein CORS-Fehler, im API-Log ein sauberes `200`.

- [ ] In der App-Factory (`src/pricing/presentation/api/app.py`) `CORSMiddleware` ergänzen, `http://localhost:5173` erlauben, Header `X-API-Key` zulassen.
- [ ] Prüfen: `fetch` aus der Browser-Konsole gegen `/v1/health`.

> [!question] Frage an dich, bevor du das einbaust
> Warum ist CORS eine Sache des **Browsers** und nicht des Servers — und warum funktioniert `curl` gegen dieselbe API ohne jede Anpassung? Beantwortet in [[Lückenschluss (Pricing-Dashboard)#1.4 — Der Weg eines Requests durch den Proxy ❌|Lückenschluss 1.4]].

---

## Schritt 1 — Deine erste eigene Komponente

**Neu: JSX und Komponenten.** *(Tag 2)*

- [ ] `src/components/Kopfzeile.jsx` anlegen, eine Überschrift zurückgeben, in `App.jsx` einbinden.

```jsx
// Muster (fremdes Datenmodell)
export default function Kopfzeile() {
  return (
    <header>
      <h1>Bücherei</h1>
    </header>
  )
}
```

- [ ] Bewusst zwei Fehler provozieren und die Meldung lesen: (a) zwei Elemente nebeneinander ohne Wurzel zurückgeben, (b) die Komponente kleingeschrieben verwenden.

---

## Schritt 2 — Props

**Neu: Daten von außen hereingeben.** *(Tag 2)*

- [ ] `ProduktKarte.jsx` schreiben, die Felder als Props bekommt und anzeigt.
- [ ] In `App.jsx` **drei** Karten mit hart codierten Werten rendern — noch keine API.

```jsx
// Muster
function BuchKarte({ titel, autor, preis }) {
  return (
    <article>
      <h3>{titel}</h3>
      <p>{autor} — {preis.toFixed(2)} €</p>
    </article>
  )
}

// Verwendung
<BuchKarte titel="Dune" autor="Herbert" preis={12.5} />
```

- [ ] Selbsttest: Was passiert, wenn du `preis` als String übergibst? Warum?

---

## Schritt 3 — Echte Daten holen

**Neu: `useState` + `useEffect` + `fetch`.** *(Tag 2)*

- [ ] Backend starten (`docker compose up` oder `uv run uvicorn …`), API-Key erzeugen, in `.env.local` ablegen:
      `VITE_API_KEY=…` und `VITE_API_URL=http://localhost:8000/v1`
- [ ] Produkte laden und als Liste rendern.

```jsx
// Muster
import { useState, useEffect } from 'react'

function BuecherListe() {
  const [buecher, setBuecher] = useState([])

  useEffect(() => {
    async function laden() {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/buecher`, {
        headers: { 'X-API-Key': import.meta.env.VITE_API_KEY },
      })
      const daten = await res.json()
      setBuecher(daten)
    }
    laden()
  }, [])          // leeres Array: einmal beim Einhängen

  return (
    <ul>
      {buecher.map((b) => <li key={b.id}>{b.titel}</li>)}
    </ul>
  )
}
```

- [ ] **Keys richtig setzen** — was ist bei `/v1/products` das stabile Feld? (`/docs` verrät es.)
- [ ] Selbsttest: Was passiert, wenn du das Dependency-Array `[]` weglässt? Probier es aus, schau in den Netzwerk-Tab, und **mach es wieder rückgängig**.

> [!warning] Der API-Key liegt hier im Browser-Bundle
> `VITE_*`-Variablen landen im ausgelieferten JavaScript und sind für jeden lesbar. Für die Übung in Ordnung, in Produktion nicht. **Sag das im Gespräch von dir aus** — die Schwäche selbst zu benennen ist deutlich stärker, als sie erklärt zu bekommen. Die saubere Lösung wäre ein Login mit kurzlebigem Token oder ein Backend-for-Frontend, das den Schlüssel hält.

---

## Schritt 4 — Lade- und Fehlerzustände

**Neu: bedingtes Rendern.** *(Tag 2)*

- [ ] Drei Zustände abbilden: lädt / Fehler / Daten da.
- [ ] Denk an die Falle aus Block 0: `{buecher.length && <Liste />}` — was rendert das bei einer leeren Liste?

```jsx
if (laedt) return <p>Lade …</p>
if (fehler) return <p role="alert">Fehler: {fehler}</p>
if (buecher.length === 0) return <p>Keine Einträge.</p>
return <ul>…</ul>
```

- [ ] `res.ok` prüfen — denk an deine Antwort auf [[0. JavaScript für React - Vertiefung#6. Asynchrones JavaScript und fetch|Block 0, Frage 6.3]]: `fetch` wirft bei 404 **nicht**.

---

## Schritt 5 — Ein Event-Handler

**Neu: `onClick`.** *(Tag 3)*

- [ ] Button „Neu laden", der die Daten erneut holt.
- [ ] Selbsttest: Was passiert bei `onClick={laden()}` statt `onClick={laden}`? Führ es aus und beschreib, was du siehst.

---

## Schritt 6 — Ein kontrolliertes Suchfeld

**Neu: kontrollierte Komponenten.** *(Tag 3)*

```jsx
const [suche, setSuche] = useState('')

<input
  value={suche}
  onChange={(e) => setSuche(e.target.value)}
  placeholder="Suchen …"
/>
```

- [ ] Selbsttest: Lass `onChange` weg. Warum lässt sich das Feld dann nicht mehr tippen? Das ist die beste Erklärung dafür, was „kontrolliert" bedeutet.

---

## Schritt 7 — Filtern **ohne** zweiten State

**Neu: abgeleiteter State.** *(Tag 3) — der lehrreichste Schritt des Projekts.*

- [ ] Gefilterte Liste **während des Renderns berechnen**, nicht in einen zweiten State legen:

```jsx
const gefiltert = buecher.filter((b) =>
  b.titel.toLowerCase().includes(suche.toLowerCase())
)
```

- [ ] Überleg dir die falsche Variante (`const [gefiltert, setGefiltert] = useState([])` plus ein `useEffect`, das bei jeder Suchänderung neu filtert) und schreib in die [[Umsetzungsstand (Pricing-Dashboard)|Umsetzungsnotiz]], **warum** sie falsch ist: was kann dabei auseinanderlaufen?

> [!tip] Interview-Gold
> Genau dieses Muster steht im Zentrum von „You Might Not Need an Effect". Wenn du es an *deinem eigenen* Code erklären kannst, hast du eine fertige, glaubwürdige Antwort auf eine der besten Fragen, die man zu React stellen kann.

---

## Schritt 8 — Auswahl und Detailbereich: State nach oben ziehen

**Neu: Lifting State Up.** *(Tag 3)*

- [ ] `ausgewaehlteId` in die gemeinsame Elternkomponente legen.
- [ ] Liste bekommt `onAuswahl` als Prop, Detailbereich bekommt `produktId`.
- [ ] Detailbereich lädt die Empfehlungen: `GET /v1/recommendations` mit passendem Filter (in `/docs` nachsehen, welcher Query-Parameter gemeint ist).
- [ ] Selbsttest: Warum liegt `ausgewaehlteId` nicht in der Listenkomponente?

---

## Schritt 9 — Ein eigener Hook

**Neu: Custom Hooks.** *(Tag 4)*

- [ ] Die drei Zustände `daten` / `laedt` / `fehler` aus den Komponenten in `src/hooks/useApi.js` herausziehen.

```jsx
// Muster
export function useApi(pfad) {
  const [daten, setDaten] = useState(null)
  const [laedt, setLaedt] = useState(true)
  const [fehler, setFehler] = useState(null)

  useEffect(() => {
    let abgebrochen = false
    setLaedt(true)
    holen(pfad)
      .then((d) => { if (!abgebrochen) setDaten(d) })
      .catch((e) => { if (!abgebrochen) setFehler(e.message) })
      .finally(() => { if (!abgebrochen) setLaedt(false) })
    return () => { abgebrochen = true }   // Cleanup
  }, [pfad])

  return { daten, laedt, fehler }
}
```

- [ ] **Versteh das `abgebrochen`-Flag.** Es verhindert eine Race Condition: klickst du schnell Produkt A, dann B, könnte die langsamere Antwort zu A nach der zu B eintreffen und sie überschreiben. Das ist ein starkes Gesprächsthema — notier den Mechanismus in eigenen Worten.
- [ ] Selbsttest: Zwei Komponenten rufen `useApi` auf. Teilen sie sich die Daten?

---

## Schritt 10 — API-Key über Context

**Neu: Context.** *(Tag 4)*

- [ ] `ApiContext` anlegen, Basis-URL und Schlüssel bereitstellen, per `useApiConfig()` konsumieren.
- [ ] In der Umsetzungsnotiz festhalten: Wie tief wäre der Schlüssel sonst durchgereicht worden? Genau das ist Prop Drilling — an deinem eigenen Beispiel.

---

## Schritt 11 — Preislauf starten und verfolgen

**Neu: POST + Polling mit Cleanup.** *(Tag 4) — der anspruchsvollste Schritt.*

- [ ] Button „Preislauf starten" → `POST /v1/pricing-runs`, Antwort enthält eine Lauf-ID.
- [ ] Solange der Lauf nicht fertig ist, alle 2 Sekunden `GET /v1/pricing-runs/{id}` abfragen.

```jsx
// Muster
useEffect(() => {
  if (!laufId || fertig) return
  const id = setInterval(() => pruefeStatus(laufId), 2000)
  return () => clearInterval(id)     // ohne das läuft das Intervall ewig weiter
}, [laufId, fertig])
```

- [ ] Nach Abschluss die Empfehlungen neu laden.
- [ ] Selbsttest: Lass `clearInterval` weg und beobachte den Netzwerk-Tab, während du mehrfach startest. Was passiert? Das ist die anschaulichste Begründung dafür, wozu Cleanup existiert.

> [!warning] `POST /v1/ingestion-runs` macht `TRUNCATE` auf Buchungen und Kapazität
> Steht so in der API-Beschreibung deines Backends. Für das Dashboard reicht der **Preislauf**; den Ingest-Endpunkt nur bewusst aufrufen.

---

## Schritt 12 — Aufräumen und erzählbar machen

*(Tag 5)*

- [ ] Ordnerstruktur ordnen: `components/`, `hooks/`, `api/`.
- [ ] Tote Konsolen-Ausgaben und ungenutzte Importe raus.
- [ ] Kurze `README.md` im Projekt: was es ist, wie man es startet, was bewusst fehlt.
- [ ] Die **3-Minuten-Story** schreiben → [[Umsetzungsstand (Pricing-Dashboard)#Die 3-Minuten-Story]].

---

## Wenn die Zeit knapp wird

Priorität von oben nach unten. Schritte 0–8 sind das Pflichtprogramm — damit kannst du über Komponenten, Props, State, Listen, Keys, kontrollierte Eingaben und abgeleiteten State an eigenem Code reden. Schritt 9 (Custom Hook) ist der größte Einzelgewinn danach. Schritte 10 und 11 sind Kür: schön, aber im Zweifel erzählst du lieber sicher über acht Schritte als unsicher über zwölf.
