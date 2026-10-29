---
tags:
  - documentation
---

# Leseliste — React Crashkurs (Deutz)

Kuratierte Leseaufgaben zu [[React Crashkurs - Übersicht]]. Pro Block ein **Core**-Teil (Pflicht) und ein **Vertiefung**-Teil (optional, jeweils markiert als *andere Perspektive* oder *weiterführend*).

Ablauf: lesen → beim Lesen roh in die jeweilige Vertiefungsnotiz exzerpieren → Kontrollfragen dort **aus dem Kopf** beantworten → Review durch Claudian → Claudian arbeitet den Abschnitt aus.

> [!warning] Diese Liste ist auf Kürze kuratiert, nicht auf Vollständigkeit
> Anders als im Traineeship ist Zeit hier **der** Engpass. Jeder Block ist bewusst beschnitten; was rausgeflogen ist, steht am Blockende unter *Was hier fehlt*. Die Vertiefungsnotizen decken den Stoff trotzdem vollständig ab — die Verknappung gilt nur fürs Lesen.
>
> Alle URLs unten wurden am 21.09.2026 auf Erreichbarkeit geprüft. Sprache: englisch, weil `react.dev` und `javascript.info` die Primärquellen sind; deine Notizen bleiben deutsch.

## Die vier Blöcke

| Block | Thema | Core-Umfang | Tag | Vertiefungsnotiz | Status |
|---|---|---|---|---|---|
| **[[#Block 0 — JavaScript für React\|0]]** | JS-Syntax, die React voraussetzt | ~2 h 30 | Mo | [[0. JavaScript für React - Vertiefung]] | ✅ fertig |
| **[[#Block 1 — Die UI beschreiben\|1]]** | Komponenten, JSX, Props, Listen, Rendering | ~2 h 40 | Di | [[1. React-Grundlagen - Vertiefung]] | ✅ fertig |
| **[[#Block 2 — State und Interaktivität\|2]]** | Events, State, Snapshot, Lifting State Up | ~2 h 45 | Mi | [[2. State und Interaktivität - Vertiefung]] | ✅ fertig |
| **[[#Block 3 — Effects, Daten und Architektur\|3]]** | Refs, Effects, Custom Hooks, Context | ~2 h 20 | Do | [[3. Effects, Daten und Architektur - Vertiefung]] | 🔵 gelesen, Fragen offen |

## Die zwei Quellen, die alles tragen

| Kürzel | Quelle | Wofür |
|---|---|---|
| **JS.info** | Ilya Kantor, *The Modern JavaScript Tutorial* — <https://javascript.info> | Block 0. Kostenlos, präzise, mit Aufgaben. Kein Buch nötig |
| **react.dev** | React, *Learn React* — <https://react.dev/learn> | Block 1–3. Die offizielle Doku ist seit dem 2023er-Relaunch **das** Lehrbuch; jede Seite hat interaktive Sandboxes und „Challenges" am Ende |

> [!tip] Die Sandboxes sind der eigentliche Wert
> Auf `react.dev` steht neben fast jedem Absatz ein lauffähiges Beispiel. **Ändere darin etwas und schau, was kaputtgeht** — das kostet 30 Sekunden und ersetzt eine halbe Seite Lesen. Besonders bei „State as a Snapshot" und „Rendering Lists".

Versionsstand heute: React **19.3**, Vite **8.3**. Beides brauchst du im Gespräch nicht auswendig, aber „React 19, aktuell seit Ende 2024" ist eine gute Antwort auf „welche Version?".

---

## Block 0 — JavaScript für React

**Tag 1 (Mo) · Core ~2 h 30 · Notiz: [[0. JavaScript für React - Vertiefung]]**

Warum das zählt: Fast jede React-Zeile ist gewöhnliches JavaScript. Wer `map` nicht kennt, hält das Rendern von Listen für React-Magie; wer Referenzsemantik nicht kennt, versteht nie, warum man State unveränderlich behandelt. Genau diese zwei Punkte sind die häufigsten Stolperstellen von Backend-Leuten beim Umstieg.

### Core (in dieser Reihenfolge)

- [x] **Arrow functions, the basics** (~10 min) — <https://javascript.info/arrow-functions-basics>
      *Leseauftrag: Wann darfst du die geschweiften Klammern und das `return` weglassen? Warum sieht man in React fast nur Arrow Functions als Event-Handler?*
- [x] **Template literals** (~10 min) — <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals>
      *Leseauftrag: Wie interpoliert man einen Ausdruck? Das ist das JS-Gegenstück zu Pythons f-Strings.*
- [x] **Destructuring assignment** (~25 min) — <https://javascript.info/destructuring-assignment>
      Nur „Array destructuring", „Object destructuring" und „Smart function parameters". Rest überspringen.
      *Leseauftrag: Was bedeutet `const [count, setCount] = useState(0)` rein sprachlich? Und `function Card({ title, price })`?*
- [x] **Array methods** (~35 min) — <https://javascript.info/array-methods>
      Fokus: `map`, `filter`, `find`, `reduce`, `slice` vs. `splice`, `sort`. Den Rest quer lesen.
      *Leseauftrag: Welche dieser Methoden verändern das Original, welche geben ein neues Array zurück? Schreib dir die Liste auf — sie entscheidet später, was du in React benutzen darfst.*
- [x] **Object references and copying** (~20 min) — <https://javascript.info/object-copy>
      **Der wichtigste Artikel des Blocks.**
      *Leseauftrag: Warum ist `obj2 = obj1` keine Kopie? Was macht `{...obj}` und wo hört es auf (flach vs. tief)?*
- [x] **Optional chaining `?.`** (~10 min) — <https://javascript.info/optional-chaining>
      *Leseauftrag: Wie rettet dich `data?.items?.length` vor einem Absturz, wenn die API noch nichts geliefert hat?*
- [x] **Logical operators** (~15 min) — <https://javascript.info/logical-operators>
      Fokus: die Abschnitte zu `&&` und `||` und „short-circuit evaluation".
      *Leseauftrag: Was gibt `0 && "hallo"` zurück — `false` oder `0`? Merk dir die Antwort, sie erzeugt in React einen berühmten Bug.*
- [x] **Nullish coalescing `??`** (~10 min) — <https://javascript.info/nullish-coalescing-operator>
      *Leseauftrag: Wann ist `??` besser als `||`?*
- [x] **Promise basics** (~15 min) — <https://javascript.info/promise-basics> — nur Konzept, nicht Chaining
- [x] **Async/await** (~20 min) — <https://javascript.info/async-await>
      *Leseauftrag: Vergleich zu Python: was ist gleich, was anders? (Du kennst `async def`/`await` aus FastAPI.)*
- [x] **Fetch** (~15 min) — <https://javascript.info/fetch>
      Nur bis einschließlich „Response headers".
      *Leseauftrag: Warum braucht man **zwei** `await` für ein JSON-Ergebnis? Und: löst `fetch` bei HTTP 404 einen Fehler aus?*
- [x] **Modules, import/export** (~15 min) — <https://javascript.info/import-export>
      *Leseauftrag: Unterschied Default- vs. Named-Export — und warum du beides in React-Dateien sehen wirst.*

### Vertiefung (optional)

- **Closure** (~40 min, *weiterführend*) — <https://javascript.info/closure>
  Nur wenn Zeit bleibt. Zahlt sich in Block 3 aus (Stichwort „stale closure" in Effects). Sonst kommt es dort nochmal.
- **Object methods, `this`** (~25 min, *andere Perspektive*) — <https://javascript.info/object-methods>
  Interessant für das Verständnis, warum moderne React-Komponenten Funktionen und keine Klassen mehr sind.

### Was hier fehlte — nachgetragen ✅

Die aus Zeitgründen aus der Leseliste geschnittenen Themen stehen ausgearbeitet in der Vertiefung:
`var`/`let`/`const`, Hoisting und Block-Scope · `==` vs. `===` · die Truthy/Falsy-Tabelle · `null` vs. `undefined` · Klassen, `this` und Prototypen (nur als Einordnung) · das DOM und was React davon übernimmt.
→ [[0. JavaScript für React - Vertiefung#5. Kurzschluss-Operatoren, Vergleiche und sicherer Zugriff|Abschnitt 5]] und [[0. JavaScript für React - Vertiefung#8. Die Ränder: Scope, this und das DOM|Abschnitt 8]]

---

## Block 1 — Die UI beschreiben

**Tag 2 (Di) · Core ~2 h 40 · Notiz: [[1. React-Grundlagen - Vertiefung]]**

Warum das zählt: Hier entsteht das mentale Modell, auf das alle Konzeptfragen zurückgeführt werden. Wenn du „Komponente = Funktion, die eine Beschreibung der UI zurückgibt" wirklich verinnerlicht hast, beantwortest du die Hälfte der typischen Interviewfragen aus dem Prinzip statt aus dem Gedächtnis.

### Core (in dieser Reihenfolge)

- [x] **Your First Component** (~15 min) — <https://react.dev/learn/your-first-component>
      *Leseauftrag: Warum fängt ein Komponentenname mit einem Großbuchstaben an — was passiert sonst?*
- [x] **Writing Markup with JSX** (~15 min) — <https://react.dev/learn/writing-markup-with-jsx>
      *Leseauftrag: JSX ist kein HTML. Nenne drei konkrete Unterschiede. Warum `className` statt `class`?*
- [x] **JavaScript in JSX with Curly Braces** (~10 min) — <https://react.dev/learn/javascript-in-jsx-with-curly-braces>
      *Leseauftrag: Wann `{}` und wann `{{}}`?*
- [x] **Passing Props to a Component** (~20 min) — <https://react.dev/learn/passing-props-to-a-component>
      *Leseauftrag: Wie verhält sich ein Prop zu einem Funktionsargument? Was ist `children`? Warum darf eine Komponente ihre Props nicht ändern?*
- [x] **Conditional Rendering** (~15 min) — <https://react.dev/learn/conditional-rendering>
      *Leseauftrag: Drei Schreibweisen (if, `? :`, `&&`). Der Abschnitt „Logical AND operator" hängt direkt an deiner Frage aus Block 0 zu `0 && "hallo"` — schlag den Bogen.*
- [x] **Rendering Lists** (~25 min) — <https://react.dev/learn/rendering-lists>
      **Interview-Klassiker.**
      *Leseauftrag: Wozu dient `key`? Warum ist der Array-Index als Key meistens falsch, und was genau geht dabei kaputt?*
- [x] **Keeping Components Pure** (~20 min) — <https://react.dev/learn/keeping-components-pure>
      *Leseauftrag: Was heißt „rein" hier konkret? Was darf eine Komponente während des Renderns **nicht** tun — und warum ist das für React nicht bloß Stilfrage?*
- [x] **Render and Commit** (~15 min) — <https://react.dev/learn/render-and-commit>
      *Leseauftrag: Die drei Phasen Trigger → Render → Commit in eigenen Worten. Wer ruft deine Komponentenfunktion eigentlich auf?*
- [x] **Thinking in React** (~25 min) — <https://react.dev/learn/thinking-in-react>
      **Der beste Einzeltext für Konzeptfragen.** Die fünf Schritte sind eine fertige Antwort auf „Wie gehen Sie an eine neue Oberfläche heran?"
      *Leseauftrag: Merk dir die fünf Schritte. Und: nach welchem Kriterium entscheidet man, ob etwas State ist oder nicht?*

### Vertiefung (optional)

- **Tutorial: Tic-Tac-Toe** (~1 h 30, *andere Perspektive*) — <https://react.dev/learn/tutorial-tic-tac-toe>
  Derselbe Stoff als durchgehendes Bauprojekt. Nimm es, wenn das Lesen abstrakt bleibt — aber nur, wenn du mit dem eigenen Projekt gut in der Zeit liegst, sonst baust du zwei Dinge parallel.
- **Importing and Exporting Components** (~15 min, *andere Perspektive*) — <https://react.dev/learn/importing-and-exporting-components>
- **Creating a React App** (~10 min, *weiterführend*) — <https://react.dev/learn/creating-a-react-app> und <https://react.dev/learn/build-a-react-app-from-scratch>
  Ordnet ein, wofür Next.js/Remix stehen und wann ein blankes Vite-Setup reicht. Liefert dir die Vokabel-Antwort auf „Warum kein Next.js?".

### Was hier fehlte — nachgetragen ✅

Alle aus Zeitgründen geschnittenen Themen stehen ausgearbeitet in der Vertiefung:
Fragments und ihre Begründung · die JSX-Transform (was React wirklich aus JSX macht) · Virtual DOM und Reconciliation inkl. der Einordnung, warum das React-Team den Begriff meidet · deklarativ vs. imperativ an einem Code-Gegenbeispiel · einseitiger Datenfluss · der Komponentenbaum.
→ [[1. React-Grundlagen - Vertiefung#2.4 Was mit JSX wirklich passiert|2.4]], [[1. React-Grundlagen - Vertiefung#7. Render und Commit — was React eigentlich tut|Abschnitt 7]]

---

## Block 2 — State und Interaktivität

**Tag 3 (Mi) · Core ~2 h 45 · Notiz: [[2. State und Interaktivität - Vertiefung]]**

Warum das zählt: Der dichteste Block und der mit der höchsten Interview-Trefferquote. „Erklären Sie mir State" ist die Standardfrage — und die Standardantwort („da speichert man Daten") ist zu dünn. Der Unterschied liegt in Snapshot-Semantik, Unveränderlichkeit und der Frage, *wo* State hingehört.

### Core (in dieser Reihenfolge)

- [x] **Responding to Events** (~20 min) — <https://react.dev/learn/responding-to-events>
      *Leseauftrag: Warum `onClick={handleClick}` und nicht `onClick={handleClick()}`? Was ist ein Event-Handler-Prop?*
- [x] **State: A Component's Memory** (~25 min) — <https://react.dev/learn/state-a-components-memory>
      *Leseauftrag: Warum reicht eine normale lokale Variable nicht? Zwei Gründe. Warum ist State „isoliert und privat" pro Komponenteninstanz?*
- [x] **State as a Snapshot** (~20 min) — <https://react.dev/learn/state-as-a-snapshot>
      **Der Schlüsseltext.** Lies langsam und spiel die Sandbox durch.
      *Leseauftrag: Warum zeigt `console.log(count)` direkt nach `setCount(count + 1)` noch den alten Wert? Das ist keine Verzögerung — was ist es stattdessen?*
- [x] **Queueing a Series of State Updates** (~20 min) — <https://react.dev/learn/queueing-a-series-of-state-updates>
      *Leseauftrag: Was ist Batching? Wann brauchst du `setCount(c => c + 1)` statt `setCount(count + 1)`?*
- [x] **Updating Objects in State** (~25 min) — <https://react.dev/learn/updating-objects-in-state>
      Knüpft direkt an „Object references and copying" aus Block 0 an.
      *Leseauftrag: Warum darf man State nie direkt mutieren? Was genau merkt React dabei nicht?*
- [x] **Updating Arrays in State** (~25 min) — <https://react.dev/learn/updating-arrays-in-state>
      *Leseauftrag: Die Tabelle „avoid / prefer" ist die Antwort auf deine `push` vs. `concat`-Frage aus Block 0. Übertrag sie in die Notiz.*
- [x] **Choosing the State Structure** (~25 min) — <https://react.dev/learn/choosing-the-state-structure>
      *Leseauftrag: Fünf Prinzipien. Besonders: warum ist redundanter State ein Bug-Generator, und was heißt „don't mirror props in state"?*
- [x] **Sharing State Between Components** (~25 min) — <https://react.dev/learn/sharing-state-between-components>
      **Interview-Klassiker: „Lifting State Up".**
      *Leseauftrag: Was ist eine kontrollierte vs. unkontrollierte Komponente? Wohin gehört State, wenn zwei Geschwister ihn brauchen?*

### Vertiefung (optional)

- **Preserving and Resetting State** (~25 min, *weiterführend*) — <https://react.dev/learn/preserving-and-resetting-state>
  Erklärt, warum ein geänderter `key` State zurücksetzt — die elegante Antwort auf eine typische Nachfrage zu Keys. **Wenn du nur eine Vertiefung in diesem Block liest, diese.**
- **Reacting to Input with State** (~20 min, *andere Perspektive*) — <https://react.dev/learn/reacting-to-input-with-state>
  Deklarativ vs. imperativ an einem Formular durchdekliniert.
- **Closure** (~40 min, *weiterführend*) — <https://javascript.info/closure>, falls in Block 0 übersprungen.

### Was hier fehlte — Stand

Nachgetragen in der Vertiefung: warum Updates asynchron *wirken* ([[2. State und Interaktivität - Vertiefung#3.1 Warum console.log den alten Wert zeigt|3.1]] und [[2. State und Interaktivität - Vertiefung#3.3 „Ein Render sieht einen eingefrorenen Snapshot"|3.3]]) · `useState` mit Initialisierungsfunktion ([[2. State und Interaktivität - Vertiefung#2.2 Was useState zurückgibt|2.2]]) · kontrollierte Komponenten ([[2. State und Interaktivität - Vertiefung#7.2 Kontrolliert vs. unkontrolliert|7.2]]).

> [!todo] Bewusst offen geblieben — Zeitgründe, nicht vergessen
> - **`useReducer`** → steht als Vokabel in [[3. Effects, Daten und Architektur - Vertiefung#7.1 — `useReducer`|Block 3, Abschnitt 7.1]], inklusive Abgrenzung zu Redux
> - **Formulare im Detail** (mehrere Felder, Validierung, `onSubmit`/`preventDefault`) — für ein Konzeptgespräch entbehrlich, für das Projekt nicht nötig. Bei mehr Zeit nachzuholen

---

## Block 3 — Effects, Daten und Architektur

**Tag 4 (Do) · Core ~2 h 20 · Notiz: [[3. Effects, Daten und Architektur - Vertiefung]]**

Warum das zählt: `useEffect` ist der Hook, bei dem am meisten falsch gemacht und am schärfsten nachgefragt wird. Wer im Gespräch sagen kann *wofür ein Effect nicht da ist*, hebt sich sofort ab — genau das ist der Inhalt des Texts „You Might Not Need an Effect".

### Core (in dieser Reihenfolge)

- [x] **Referencing Values with Refs** (~15 min) — <https://react.dev/learn/referencing-values-with-refs>
      *Leseauftrag: Wann Ref statt State? Was passiert (bzw. passiert nicht), wenn du eine Ref änderst?*
- [x] **Synchronizing with Effects** (~30 min) — <https://react.dev/learn/synchronizing-with-effects>
      *Leseauftrag: Wozu das Dependency-Array? Was ist die Cleanup-Funktion, wann läuft sie — und warum feuert ein Effect im Development zweimal?*
- [x] **You Might Not Need an Effect** (~30 min) — <https://react.dev/learn/you-might-not-need-an-effect>
      **Der mit Abstand wertvollste Text für ein Konzeptgespräch.**
      *Leseauftrag: Zwei Fälle, in denen man fälschlich einen Effect nimmt — und was stattdessen richtig ist. Was ist „abgeleiteter State"?*
- [x] **Reusing Logic with Custom Hooks** (~30 min) — <https://react.dev/learn/reusing-logic-with-custom-hooks>
      *Leseauftrag: Was macht eine Funktion zu einem Hook? Teilen zwei Komponenten, die denselben Custom Hook benutzen, den State — ja oder nein, und warum?*
- [x] **Rules of Hooks** (~10 min) — <https://react.dev/reference/rules/rules-of-hooks>
      *Leseauftrag: Die zwei Regeln. Und die eigentliche Frage: **warum** gibt es sie — was macht React intern, das eine stabile Aufrufreihenfolge braucht?*
- [x] **Passing Data Deeply with Context** (~25 min) — <https://react.dev/learn/passing-data-deeply-with-context>
      *Leseauftrag: Was ist „Prop Drilling"? Drei Fälle, in denen Context passt — und wann man es lieber lässt.*

### Vertiefung (optional)

- **Extracting State Logic into a Reducer** (~25 min, *weiterführend*) — <https://react.dev/learn/extracting-state-logic-into-a-reducer>
  Plus **Scaling Up with Reducer and Context** (~25 min) — <https://react.dev/learn/scaling-up-with-reducer-and-context>. Zusammen deine Antwort auf „Wie würden Sie größeren State organisieren?".
- **`useMemo` / `memo`** (~30 min, *weiterführend*) — <https://react.dev/reference/react/useMemo> und <https://react.dev/reference/react/memo>
  Nur die Einleitung plus „Troubleshooting" überfliegen. Ziel ist die Vokabel, nicht die Beherrschung.
- **React 19 Release-Blog** (~20 min, *weiterführend*) — <https://react.dev/blog/2024/12/05/react-19>
  Actions, `use`, `useActionState`, ref als normales Prop. Gibt dir eine aktuelle, informierte Bemerkung fürs Gespräch.
- **TanStack Query** (~20 min, *weiterführend*) — <https://tanstack.com/query/latest>
  Nur „Overview". Antwort auf „Wie holen Sie Daten?": selbst gebauter Hook für die Übung, in echt nimmt man so etwas.
- **React Router** (~15 min, *weiterführend*) — <https://reactrouter.com/start/declarative/installation>
  Nur falls dein Projekt mehr als eine Seite bekommt.

### Was hier fehlt

> [!todo] Lücken schließen — beim Ausschreiben von Block 3 mitbehandeln
> - Race Conditions beim Datenholen und wie Cleanup sie entschärft (`ignore`-Flag / `AbortController`)
> - `useEffect` vs. `useLayoutEffect` — nur als Einordnung
> - Der Lebenszyklus-Vergleich zu Klassenkomponenten (`componentDidMount` & Co.) — kommt in Interviews von älteren Entwicklern oft vor
> - Error Boundaries und `Suspense` — als Vokabel
> - Warum Context kein State-Management-Tool ist und wo Redux/Zustand ansetzen
> - Die Grundidee des React Compiler (React 19) in drei Sätzen
