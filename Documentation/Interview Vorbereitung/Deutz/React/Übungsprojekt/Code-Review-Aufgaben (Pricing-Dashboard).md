---
tags:
  - documentation
---

# Code-Review-Aufgaben — Pricing-Dashboard

Claudian baut, du liest. Nach jeder Bauphase kommt hier eine **Leseaufgabe** dazu: welche Dateien, in welcher Reihenfolge, worauf achten — plus ein paar Fragen, die du **kurz** beantwortest (ein, zwei Sätze reichen). Claudian bewertet jede Antwort kurz im Feedback-Kasten darunter; die ausführlichen Erklärungen zu allen Lücken stehen in [[Lückenschluss (Pricing-Dashboard)]].

Zurück zu: [[Umsetzungsstand (Pricing-Dashboard)]] · Hintergrund zu allen Dateien: [[Frontend-Projektaufbau erklärt]]

> [!info] So liest du am besten
> - **Code und Notiz nebeneinander**: VS Code links, diese Notiz rechts.
> - **Erst lesen, dann antworten, dann nachschlagen.** Lücken sind das Diagnosesignal, kein Mangel — lieber „weiß ich nicht" schreiben als raten.
> - **„Ausprobieren"-Punkte wirklich ausprobieren.** Eine Minute Herumspielen erklärt mehr als zehn Minuten Lesen. Danach alles wieder rückgängig machen.
> - Kommentare im Code lesen — die sind für dich geschrieben.

> [!tip] Ab Leseaufgabe 4: eine Phase = ein Commit
> Jede Bauphase ist ein Commit auf dem Branch `frontend`. Wie man einen Commit liest: [[#So liest du einen Commit]]. Leseaufgaben 1–3 beziehen sich auf den Baseline-Commit `0734487`.

## So liest du einen Commit

Ein Commit zeigt nur, **was sich geändert hat**, nicht die ganze Datei. Deshalb immer in zwei Durchgängen lesen:

1. **Überblick:** Welche Dateien hat der Commit angefasst, und wie viel? Das zeigt, wo der Schwerpunkt liegt.
2. **Diff pro Datei:** Links vorher, rechts nachher. Rot ist weggefallen, grün neu. Neue Dateien sind komplett grün, also ganz normal von oben nach unten lesen.

Danach die wichtigste Datei **einmal ganz** öffnen, ohne Diff. Ein Diff zeigt nur Ausschnitte, den Zusammenhang sieht man erst in der vollständigen Datei.

### Weg 1 — VS Code mit GitLens (Standard)

Den Ordner `FrontEndBuchungsSystem/` in VS Code öffnen (nicht nur `frontend/`, sonst findet VS Code das Git-Repo nicht).

**Einen Commit lesen:**
1. Links das Symbol **Quellcodeverwaltung** (drittes von oben, die Verzweigung).
2. Im Abschnitt **GitLens** auf **Commits** → den Commit aufklappen, z. B. „Phase 3: eigener Hook useApi".
3. Erst die **Commit-Nachricht** lesen: Sie sagt, was und warum.
4. Datei anklicken → öffnet die **Vorher/Nachher-Ansicht**. Zur nächsten Änderung springen: `Alt+F5`, zurück: `Shift+Alt+F5`.
5. Oben rechts im Diff-Fenster gibt es ein Symbol **„Datei öffnen"**, das öffnet die vollständige aktuelle Fassung (Schritt „einmal ganz lesen").

**Alternativ, mit Commit-Details:** Das GitLens-Symbol mit der **Lupe** („GitLens Inspect") zeigt zum gerade gewählten Commit Nachricht, Autor und Dateiliste in der Seitenleiste.

**Mehrere Commits auf einmal** (z. B. die ganze Leseaufgabe 5 = drei Commits): GitLens Inspect → **Search & Compare** → *Compare References…* → zuerst den Commit **vor** dem Bereich wählen (`7a53e5d`), dann den letzten (`c0a335f`). Ergebnis: alle Änderungen dazwischen als eine Dateiliste.

> [!tip] Warum VS Code die erste Wahl ist
> Im Diff funktioniert alles, was im Editor geht: Maus über eine Variable zeigt ihren **Typ**, `F12` (bzw. `gd` mit Vim) springt zur **Definition**. Gerade bei TypeScript liest man so viel schneller als im Terminal.

### Weg 2 — Terminal (schnell, für den Überblick)

Im Repo-Ordner `FrontEndBuchungsSystem/`:

| Befehl | Zeigt |
|---|---|
| `git log --oneline -10` | die letzten Commits, einer pro Zeile |
| `git show --stat c0a335f` | Nachricht + geänderte Dateien mit Anzahl Zeilen |
| `git show c0a335f` | den kompletten Diff |
| `git show c0a335f -- frontend/src/hooks/useApi.ts` | nur eine Datei daraus |
| `git show -w c0a335f` | wie oben, aber **Einrückungsänderungen ignoriert** |
| `git show --word-diff c0a335f` | wortweise statt zeilenweise, gut bei kleinen Änderungen in langen Zeilen |
| `git diff 7a53e5d c0a335f -- frontend/src` | mehrere Commits zusammen (alles zwischen den beiden) |
| `git show c0a335f:frontend/src/components/ApiStatus.tsx` | die Datei **so, wie sie in diesem Commit war**, ohne Diff |

Die Ausgabe läuft durch `less`: `Leertaste` blättert, `/useApi` sucht, `n` nächster Treffer, `q` beendet.

### Weg 3 — kitty als Seite-an-Seite-Ansicht (optional)

kitty bringt mit `kitten diff` einen eigenen Diff-Betrachter mit: zweispaltig, mit Syntaxhervorhebung, direkt im Terminal. Einmal in Git einrichten (in `~/.gitconfig`):

```ini
[diff]
    tool = kitty
[difftool]
    prompt = false
    trustExitCode = true
[difftool "kitty"]
    cmd = kitten diff $LOCAL $REMOTE
```

Dann:

```bash
git difftool --no-symlinks --dir-diff c0a335f~1 c0a335f
```

`c0a335f~1` heißt „der Commit davor". Bedienung laut kitty-Doku: `j`/`k` scrollen, `n`/`p` nächste/vorige Änderung, `+`/`-` mehr/weniger Kontextzeilen, `q` beenden. *(Von Claudian nicht getestet, kitty ist aus Obsidian heraus nicht erreichbar.)*

## Übersicht

| #   | Thema                                                                                                    | Dateien                                           | Dauer   | Status                                  |
| --- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------- | --------------------------------------- |
| 1   | [[#Leseaufgabe 1 — Das Projektgerüst\|Das Projektgerüst]]                                                | Konfiguration, `index.html`, `main.tsx`           | ~30 min | ✅ beantwortet (1 ✅ · 3 🟡 · 1 ❌)        |
| 2   | [[#Leseaufgabe 2 — Die API-Schicht\|Die API-Schicht]]                                                    | `src/api/*`                                       | ~20 min | ✅ beantwortet (3 ✅ · 3 🟡)              |
| 3   | [[#Leseaufgabe 3 — Die erste Komponente\|Die erste Komponente]]                                          | `App.tsx`, `components/ApiStatus.tsx`             | ~15 min | ✅ beantwortet (2 ✅ · 3 ❌)               |
| 4   | [[#Leseaufgabe 4 — Phase 2 Produktliste, Suche, Auswahl\|Phase 2: Produktliste, Suche, Auswahl]]         | Commit `7a53e5d`                                  | ~25 min | ✅ beantwortet (1 ✅ · 3 🟡 · 3 ❌)        |
| 5   | [[#Leseaufgabe 5 — Phase 3 Der eigene Hook\|Phase 3: Der eigene Hook]]                                   | Commits `3cc4fda`, `30cceab`, `c0a335f`           | ~25 min | ✅ beantwortet (1 ✅ · 3 🟡 · 2 ❌)        |
| 6   | [[#Leseaufgabe 6 — Phase 4 Backend-Filter und Detailbereich\|Phase 4: Backend-Filter und Detailbereich]] | Commits `18ac2a7` (Backend), `9a675dc` (Frontend) | ~35 min | ✅ beantwortet (1 ✅ · 2 🟡 · 3 ❌ · 1 ⏭️) |
| 7   | [[#Leseaufgabe 7 — Phase 5 Preislauf und Polling\|Phase 5: Preislauf und Polling]]                       | Commit `8a051f1`                                  | ~35 min | ✅ beantwortet (2 ✅ · 3 🟡 · 1 ❌ · 1 ⏭️) |
| 8 | [[#Leseaufgabe 8 — Phase 6 Aufräumen und Gesamtbild\|Phase 6: Aufräumen und Gesamtbild]] | Commit `a650ffd` + README | ~30 min | ✅ beantwortet (1 ✅ · 2 🟡 · 1 ⏭️) |
| 9 | [[#Leseaufgabe 9 — Phase 7 Ingest und ein Hook für zwei Läufe\|Phase 7: Ingest, generischer Hook]] | Commits `aaedccd`, `504466c` | ~30 min | offen |
| 10 | [[#Leseaufgabe 10 — Phase 8 Tailwind CSS\|Phase 8: Tailwind CSS]] | Commits `aebd5c1`, Phase 8b | ~25 min | offen |

---

## Leseaufgabe 1 — Das Projektgerüst

**Ziel:** Du weißt, welche Datei wofür da ist, und kannst den Weg vom Browser bis zu deiner FastAPI erklären.

**Vorbereitung:** [[Frontend-Projektaufbau erklärt]] einmal quer lesen (die Tabellen genügen fürs Erste).

**Lesereihenfolge** (alle im Ordner `frontend/`):

- [x] `package.json` — Skripte und die zwei Abhängigkeitsgruppen
- [x] `index.html` — der Einstieg
- [x] `src/main.tsx` — wie React an die Seite kommt
- [x] `vite.config.ts` — der Proxy, inklusive Kommentare
- [x] `tsconfig.app.json` — nur überfliegen, mit der Tabelle aus der Erklärnotiz daneben
- [x] `eslint.config.js` — überfliegen

**Ausprobieren:**

- [x] `npm run dev`, Seite öffnen. Oben rechts sollte „API erreichbar" stehen.
- [x] In `App.tsx` den Titel ändern und speichern — die Seite aktualisiert sich ohne Neuladen (Hot Module Replacement).
- [x] In `App.tsx` einen **Typfehler** einbauen, z. B. `const x: number = 'hallo'` in der Komponente. Speichern, Browser ansehen. Dann `npm run build` im Terminal. Danach wieder entfernen.

**Fragen:**

1.1 Ein Kollege klont das Repo und tippt sofort `npm run dev`. Was fehlt ihm, und was sieht er jeweils, wenn es fehlt?
> Antwort: Der API-KEY, weil dieser sich nicht im Repo befindet und er sieht dann zwar API erreichabar, aber links in der produktleiste ein http error

1.2 `react` steht unter `dependencies`, `typescript` unter `devDependencies`. Was davon kommt am Ende im Browser des Nutzers an — und warum das andere nicht?
> Antwort: es kommt pures javascript an, typescript ist wie der name schon sagt eine developer dependency

1.3 Beim Typfehler-Experiment: Was ist im Browser passiert, was bei `npm run build`? Warum der Unterschied?
> Antwort:Im Browser passiert nichts, aber npm run build failes wegen syntax error

1.4 Beschreib den Weg eines Aufrufs `fetch('/v1/health')` bis zur FastAPI. Welche Adresse sieht der Browser, welche die API? Und warum brauchst du deshalb keine CORS-Freigabe?
> Antwort: der browse sieht 127.0.0.1:8000/v1/health und schickt es dann an den uvicorn server

1.5 Was würde passieren, wenn die Variable in `.env.local` nur `API_KEY` statt `VITE_API_KEY` hieße?
> Antwort: Man wäre nicht mehr autorisiert, weil das in client.ts gehardcoded ist

> [!note]- Feedback von Claudian
> 1.3 saß (nur: **Typ**fehler, nicht Syntaxfehler). Lücken: 1.1 `node_modules` und Backend fehlten · 1.2 React kommt sehr wohl im Browser an · 1.4 ❌ der Browser sieht nur `localhost:5173`, den Rest macht der Proxy · 1.5 nicht hardcodiert, sondern Vite gibt nur `VITE_`-Variablen weiter.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 1 — Projektgerüst]]

---

## Leseaufgabe 2 — Die API-Schicht

**Ziel:** Du verstehst, warum Komponenten nie selbst `fetch` aufrufen, und kennst die Stolperstellen zwischen Pydantic-JSON und TypeScript.

**Hintergrund:** Diese Schicht war schon vorbereitet, bevor die Bauphasen losgingen. Seit dem 30.09. kam nur `getHealth()` und der Typ `Health` dazu.

**Lesereihenfolge:**

- [x] `src/api/types.ts` — zuerst die Datenformen, mit dem Kopfkommentar
- [x] `src/api/client.ts` — `ApiError`, `readDetail`, `apiFetch`, `withQuery`
- [x] `src/api/endpoints.ts` — eine Funktion pro Endpunkt

**Worauf achten:** Vergleich mit deinem Backend. `types.ts` ↔ `presentation/api/schemas/`, `endpoints.ts` ↔ `presentation/api/routers/`, `client.ts` ↔ die Fehlerbehandlung in `errors.py`.

**Fragen:**

2.1 `fetch` wirft bei einem 404 keinen Fehler. Wo im Code wird das abgefangen, und was bekommt eine Komponente stattdessen in die Hand?
> Antwort: Das wird immer dort wo das promise eingelöst wird aufgefangen und dann sagt die komponente selber was doert stattdessen stehen soll (also weil apiFetch weiterwirft)

2.2 Warum ist `recommended_adjustment_pct` ein `string` und `occupancy` eine `number`? Was käme bei `empfehlung.recommended_adjustment_pct + 1` heraus?
> Antwort:  pydantic macht aus decimal string aber aus float eine zahl. er händt 1 an den string an wenn man rechnen will braucht man Number()

2.3 `apiFetch<T>(...)` endet mit `return (await response.json()) as T`. Prüft irgendjemand **zur Laufzeit**, dass die Antwort wirklich die Form `T` hat? Was ist das Gegenstück dazu in deinem Backend?
> Antwort: Ich denke nein. es sind wie in python typehints

2.4 Warum ist `RunStatus` ein Union-Typ (`'queued' | 'running' | …`) statt einfach `string`?
> Antwort: damit ein switch darüber vollständig sein muss 

2.5 In `apiFetch` steht `...init?.headers` **nach** dem `X-API-Key`. Was bewirkt diese Reihenfolge?
> Antwort: wer später schreibt gewinnt. so könnte der nutzer also den apikey noch überschreiben

2.6 **Suchauftrag im Backend:** Gibt es einen Endpunkt, der *alle* Empfehlungen zu *einem* Produkt liefert? Schau in `routers/recommendations.py`. Was heißt das für den geplanten Detailbereich?
> Antwort: nein. aktuell nicht

> [!note]- Feedback von Claudian
> 2.2, 2.5, 2.6 saßen. Lücken: 2.1 die Stelle `if (!response.ok) throw new ApiError(…)` fehlte · 2.3 das Backend-Gegenstück ist Pydantic (prüft zur Laufzeit) · 2.4 Vollständigkeit erzwingt der Union-Typ allein **nicht**. Das war ein Fehler in meinem Kommentar, inzwischen korrigiert.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 2 — API-Schicht]]

---

## Leseaufgabe 3 — Die erste Komponente

**Ziel:** Du kannst `useState` + `useEffect` + Cleanup an echtem Code erklären — und weißt, warum React in der Entwicklung alles doppelt macht.

**Lesereihenfolge:**

- [x] `src/App.tsx` — im Baseline-Commit reines Layout ohne State (`git show 0734487:frontend/src/App.tsx`); seit Phase 2 steckt dort die Auswahl, das ist Leseaufgabe 4
- [x] `src/components/ApiStatus.tsx` — die eigentliche Aufgabe
- [x] `src/index.css` — nur die Klassennamen aus `App.tsx` wiederfinden

**Ausprobieren:**

- [x] Browser-DevTools (F12) → Tab **Netzwerk** → Seite neu laden. Wie viele Anfragen an `health` siehst du, und welchen Status hat die erste?
- [x] Backend stoppen, Seite neu laden. Was zeigt die Ampel? Mit der Maus darüberfahren.
- [x] In `ApiStatus.tsx` das leere Dependency-Array `[]` durch `[zustand]` ersetzen, speichern, Netzwerk-Tab beobachten. **Danach sofort zurückändern.**

**Fragen:**

3.1 Im Netzwerk-Tab: Warum zwei Anfragen, und warum ist die erste abgebrochen? (Stichworte: `StrictMode` in `main.tsx`, Cleanup.)
> Antwort: es liegt am stricktmode. react macht absichtlich manche dinge doppelt damit fehler sichtbar werden. so fällt zum beispiel auf wenn rendern nicht rein ist + man sieht ob aufgeräumt wird. deshalb auch abgebrochen

3.2 Was ist mit `[zustand]` im Dependency-Array passiert — und warum? (Tipp: `{ art: 'ok' } === { art: 'ok' }` ist in JavaScript `false`.)
> Antwort: es ist in eine endlose schleife geangen und wurde wieder gesetzt was einen rerender getriggert hat was wiederum einen effect getriggerd hat usw. es liegt daran, dass es niccht das exakt gleiche objekt ist und deshalb nach j

3.3 Wozu dient `if (controller.signal.aborted) return` im `catch`? Was würdest du ohne diese Zeile auf der Seite sehen?
> Antwort: Ich weiß es nicht ich nehmen an gar nichts

3.4 `Zustand` ist eine *discriminated union*. Was sagt der Compiler, wenn du ganz oben in der Komponente `console.log(zustand.meldung)` schreibst — und warum geht es im `case 'fehler'` problemlos?
> Antwort: weiß ich nicht

3.5 **Bonus:** Du ergänzt `Zustand` um eine vierte Variante `{ art: 'langsam' }`, passt den `switch` aber nicht an. Meldet der Compiler das? Probier es aus.
> Antwort: ja

> [!note]- Feedback von Claudian
> 3.1 und 3.2 sehr gut. Lücken: 3.3 ❌ ein Abbruch landet als Fehler im `catch` → ohne die Zeile kurzes rotes Flackern · 3.4 ❌ Narrowing · 3.5 ❌ **nein**, der Compiler meldet nichts. Deshalb gibt es jetzt `assertNever`.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 3 — Erste Komponente]]

---

## Leseaufgabe 4 — Phase 2 Produktliste, Suche, Auswahl

**Commit:** `7a53e5d` — `git show 7a53e5d`

**Ziel:** Du kannst an deinem eigenen Code erklären, wo State wohnt und warum, was eine kontrollierte Komponente ist, und warum die gefilterte Liste **kein** State ist. Das sind die Plan-Schritte 2–8 und damit das Pflichtprogramm aus dem [[Pricing-Dashboard Plan & Syntax-Tutorial#Wenn die Zeit knapp wird|Plan]].

**Lesereihenfolge:**

- [x] `components/ProduktEintrag.tsx` — die einfachste Komponente: nur Props, kein State
- [x] `components/Produktliste.tsx` — der Kern der Phase: Laden, Suche, abgeleitete Liste
- [x] `App.tsx` — nur der Diff: wo die Auswahl jetzt liegt und wie sie verteilt wird
- [x] `components/Detailbereich.tsx` — kurz, zeigt nur die Auswahl
- [x] `index.css` — überfliegen

**Ausprobieren:**

- [x] Nach `4 bed` suchen, ein Produkt anklicken, dann per Tastatur: mit Tab durch die Liste, Enter wählt aus.
- [x] Ein Produkt auswählen, dann so suchen, dass es aus der Liste verschwindet. Was zeigt der Detailbereich?
- [x] In `Produktliste.tsx` die Zeile `onChange={…}` am `<input>` entfernen. Versuch zu tippen, und schau in die Browser-Konsole. **Danach zurückändern.**
- [x] In `ProduktEintrag.tsx` `onClick={() => onAuswahl(produkt.product_id)}` durch `onClick={onAuswahl(produkt.product_id)}` ersetzen (ohne `() =>`). Seite und Konsole ansehen. **Danach zurückändern.**

**Fragen:**

4.1 Es gibt drei Stück State: die geladenen Produkte, den Suchbegriff und die Auswahl. Wo liegt jeder, und warum dort? Angenommen, die Kopfzeile soll „12 Treffer" anzeigen: Was müsste umziehen?
> Antwort: Auswahl lebt in App.tsx, Produkte leben in Produktliste (in dem Typ Laden) und Suchbegriff lebt in Produktliste

4.2 `gefiltert` ist eine einfache Konstante, kein `useState`. Beschreib die „falsche" Variante mit zweitem State plus `useEffect` — und was dabei auseinanderlaufen kann.
> Antwort: Ich weiß nicht genau was du meisnt.

4.3 Warum steht `key` an `<ProduktEintrag>` in `Produktliste` und nicht am `<li>` in `ProduktEintrag`? Was wäre mit `key={index}` schiefgegangen, sobald man sucht?
> Antwort: Auch das ist mir unklar

4.4 Warum lässt sich das Feld ohne `onChange` nicht mehr tippen? Was heißt also „kontrolliert"?
> Antwort: Weil das onChange dafür verantworlich ist was dabei ausgelöst wird und eben dafür den entsprechenden State zu verändern

4.5 Was ist beim `onClick` ohne `() =>` passiert, und warum?
> Antwort: es würde auch nicht funktionieren, weil man ihm eine ausführbare Funktion geben muss

4.6 In `App` steht `onAuswahl={setAusgewaehlteId}` — die Setter-Funktion wird direkt weitergereicht. Warum passt die, obwohl `onAuswahl` als `(produktId: string) => void` getypt ist?
> Antwort: weiß ich nicht

4.7 **Deine Meinung:** Die Auswahl bleibt bestehen, auch wenn die Suche das gewählte Produkt ausblendet. Bug oder Feature? Wenn Bug: Wie würdest du es lösen, **ohne** einen Effect zu benutzen?
> Antwort: Ich finde in dem Fall ist es eher ein feature. Aus meiner sicht gibt es keinen anlass das produkt auszublenben, da ja noch kein neues angeklickt wurde.

> [!note]- Feedback von Claudian
> 4.7 gut begründet, als Entscheidung übernommen. Wichtigste Lücken: **4.2** (abgeleiteter State) und **4.3** (Keys), beide Gesprächsklassiker. Außerdem 4.6 Funktionstypen und Details zu 4.1/4.4/4.5. Bei 4.5 bitte nachtragen, was du im Browser gesehen hast.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 4 — Produktliste, Suche, Auswahl]]

---

## Leseaufgabe 5 — Phase 3 Der eigene Hook

**Commits** (in dieser Reihenfolge):

| Commit    | Inhalt                                                                    | Wie lesen                                                 |
| --------- | ------------------------------------------------------------------------- | --------------------------------------------------------- |
| `3cc4fda` | Prettier als Formatierer, alle Dateien einmal durchformatiert             | nur `git show --stat 3cc4fda`, Inhalt ist rein kosmetisch |
| `30cceab` | `assertNever`, Ergebnis deiner Frage 3.5                                  | `git show 30cceab`                                        |
| `c0a335f` | **Phase 3:** `hooks/useApi.ts`, `ApiStatus` und `Produktliste` nutzen ihn | `git show c0a335f`                                        |

**Ziel:** Du kannst erklären, was ein Custom Hook ist, was zwei Komponenten mit demselben Hook teilen, und wie der Hook die Race Condition verhindert. Das ist Plan-Schritt 9, laut Plan „der größte Einzelgewinn" fürs Gespräch.

**Vorbereitung:** [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 3 — Erste Komponente|Lückenschluss zu Leseaufgabe 3]] lesen. 3.2 bis 3.5 sind die Grundlage für diesen Hook.

**Lesereihenfolge:**

- [x] `src/lib/assertNever.ts`, kurz, Kommentar lesen
- [x] `src/hooks/useApi.ts`, der Kern; alle Kommentare lesen
- [x] Diff von `ApiStatus.tsx`: was weggefallen ist
- [x] Diff von `Produktliste.tsx`: dasselbe

**Ausprobieren:**

- [x] In `ApiStatus.tsx` den `case "fertig":` samt Zeile darunter auskommentieren. Was meldet der Editor? Zurückändern.
- [x] Backend stoppen, Seite neu laden, mit der Maus über die rote Ampel fahren. Welcher Statuscode taucht im Netzwerk-Tab auf?
- [x] In `Produktliste.tsx` `useApi(listProducts)` durch `useApi((signal) => listProducts(signal))` ersetzen. Netzwerk-Tab beobachten. **Sofort zurückändern.**

**Fragen:**

5.1 Zwei Komponenten rufen `useApi(listProducts)` auf. Teilen sie sich die Daten? Was teilen sie sich?
> Antwort: Nein sie teilen sich keine Daten. Wenn man Hooks verwendet wird der State in den Komponenten gespeichert und nicht geteilt. Sie teilen sich also nur den ausgelargerten Code

5.2 Was ist bei `useApi((signal) => listProducts(signal))` passiert, und warum? Welche Leseaufgabe-Frage war dasselbe Problem in anderer Form?
> Antwort: Ich weiß nicht genau warum, aber er ist in eine schleife gegangen wo er dern get produktes endpoint gespammt hat. In 3.2 ist das schonmal passiert

5.3 Am Ende des Hooks steht `if (ergebnis === null || ergebnis.quelle !== laden) return { art: "laedt" }`. Warum merkt sich das Ergebnis seine `quelle`, statt dass der Effekt am Anfang einfach `setErgebnis(null)` aufruft?
> Antwort: Die Quelle wird sich gemerkt damit man irgendwie später besser debuggen kann i guess

5.4 **Race Condition:** Du klickst Produkt A an (Antwort dauert 2 s), dann sofort B (Antwort dauert 0,2 s). Was verhindert, dass am Ende As Daten angezeigt werden? Es sind zwei Mechanismen.
> Antwort: Es gibt auf jeden fall den abort controller damit alte anfragen neuer nicht überschreiben

5.5 Backend aus: Welcher Statuscode kam an? Warum gibt es überhaupt einen Statuscode, obwohl der Server, der ihn liefern müsste, gar nicht läuft?
> Antwort:  502 bad gateway und weil die webseite ja nicht über den server gehostet wird sondern auf dem npm dev server und ich denke mal dass der so etwas wie bad gaterway bereit stellt, wenn die datenbank nicht erraichbar ist

5.6 Warum ist die Prettier-Formatierung ein eigener Commit, statt mit Phase 3 zusammen?
> Antwort: das weiß ich nicht.

> [!note]- Feedback von Claudian
> 5.1 saß. 5.2 Beobachtung richtig, das Warum fehlte (neue Funktion bei jedem Rendern = 3.2 in anderer Form) · 5.3 ❌ nicht Debugging: Ohne `quelle` stünden für einen Frame die Daten des alten Produkts unter dem neuen Namen · 5.4 zweiter Mechanismus fehlte (`quelle`-Vergleich) · 5.5 nicht die Datenbank, sondern die API ist weg; 502 = der Proxy hat niemanden hinter sich · 5.6 ❌ atomare Commits.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 5 — Eigener Hook]]

---

## Leseaufgabe 6 — Phase 4 Backend-Filter und Detailbereich

**Commits:**

| Commit | Inhalt |
|---|---|
| `18ac2a7` | **Backend:** `GET /v1/recommendations?product_id=…`, Repository, Fake, drei neue Tests |
| `9a675dc` | **Frontend:** `Empfehlungen.tsx` (neu), `Detailbereich.tsx`, `lib/format.ts`, Endpunkt erweitert |

**Ziel:** Du kannst erklären, wozu `useCallback` da ist, warum Hooks nicht bedingt aufgerufen werden dürfen, und was ein `key` außerhalb von Listen bewirkt. Und du hast zum ersten Mal eine Änderung gelesen, die durch **beide** Schichten geht.

**Vorbereitung:** [[Lückenschluss (Pricing-Dashboard)#5.2 — Die Pfeilfunktion als Ladefunktion 🟡|Lückenschluss 5.2]] lesen, darauf baut `useCallback` direkt auf.

> [!warning] Zuerst: Dein Docker-Container läuft noch mit dem alten Backend
> Der Container wurde vor der Änderung gebaut. Er **ignoriert** `product_id` einfach und liefert die ersten 500 Empfehlungen aller Produkte. Das ist Teil der ersten Übung unten. **Danach** neu bauen:
> ```bash
> cd ~/HaegerConsulting/Projects/Interviews/ReactBasics/FrontEndBuchungsSystem
> docker compose up -d --build
> ```
> Am 14.10. hat Claudian außerdem einen Preislauf gestartet (Lauf 1, 82 866 Empfehlungen). Nach deinem Ingest am 13.10. war die Tabelle leer.

**Lesereihenfolge:**

- [x] Backend, Diff `18ac2a7`: `persistence/repositories.py` → `interfaces.py` → `routers/recommendations.py` → `tests/test_api_routing.py` + `tests/conftest.py`
- [x] `frontend/src/api/endpoints.ts`, nur der Diff
- [x] `frontend/src/components/Detailbereich.tsx`
- [x] `frontend/src/components/Empfehlungen.tsx`, der Kern; komplett lesen
- [x] `frontend/src/lib/format.ts`

**Ausprobieren:**

- [x] **Vor dem Neubau:** im Dashboard drei verschiedene Produkte anklicken. Was fällt an der Tabelle und den Kennzahlen auf?
- [x] Neu bauen (siehe oben), dann dasselbe noch einmal. `http://localhost:8000/docs` → `GET /v1/recommendations`: Der neue Parameter steht mit Beschreibung da.
- [x] Netzwerk-Tab: ein Produkt anklicken, dann ein anderes. Wie viele Anfragen pro Klick?
- [x] In `Empfehlungen.tsx` die Abhängigkeiten von `useCallback` von `[produktId]` auf `[]` ändern. Was meldet der Editor? Zwischen Produkten wechseln: Funktioniert es trotzdem?
- [x] Zusätzlich in `Detailbereich.tsx` das `key={produktId}` entfernen. Jetzt wechseln. Was passiert? **Danach beides zurückändern.**

**Fragen:**

6.1 Was hat das Dashboard mit dem alten Backend angezeigt, und warum gab es **keinen** Fehler, obwohl das Backend `product_id` gar nicht kannte? Was lernst du daraus über den „Vertrag" zwischen Frontend und Backend?
> Antwort: Ich weiß es leider nicht, weil ich das Backend ausversehen schon neu gebaut hatte :(

6.2 **Backend:** `SqlAlchemyCapacityRepository.list_all` filtert ohne Join, `SqlAlchemyRecommendationRepository.list_all` braucht einen. Warum? Und warum liefert ein unbekanntes Produkt `[]` statt 404?
> Antwort: Weil wir informationen aus capacity brauchen, welche wir nur über den join erhalten. Weil es eine Liste zurück gibt und die korekte antwort auf die frage welche emphelungen es gibt zu einem produkt das nicht existiert ist keine empfehlungen für dieses produkt

6.3 **Backend:** Der Router ruft jetzt `repository.list_all(product_id=…, limit=…, offset=…)` mit Namen auf statt wie vorher `list_all(limit, offset)`. Was wäre mit dem alten Aufruf passiert, nachdem `product_id` als **erster** Parameter dazukam? Wer hätte es bemerkt?
> Antwort: ich nehme mal an python und dann weitergeletiett an pydantic 

6.4 Wozu dient `useCallback` in `Empfehlungen.tsx`? Warum braucht `Produktliste` für `useApi(listProducts)` keins?
> Antwort: callback sorgt dafür, dass zwei arrow funktioinen von react als gleich erkannt werden und nicht zu einem neu laden führen. was wichtig ist, weil man sonst in eine endlosschleife geht. das ist vor allem relevant, weil

6.5 Das `[]`-Experiment: Warum hat es mit `key` trotzdem funktioniert und ohne `key` nicht? Was genau macht `key={produktId}` an einer einzelnen Komponente, die gar nicht in einer Liste steht?
> Antwort: Ich weiß es tatsächlich nicht

6.6 Warum ruft `Detailbereich` nicht selbst `useApi` auf, sondern lagert das in `Empfehlungen` aus? Was wäre an folgendem Code falsch?
```tsx
if (produktId === null) return <p>Wähle ein Produkt</p>;
const empfehlungen = useApi(laden);
```
> Antwort: Ich weiß es nicht.

6.7 Wie viele Anfragen pro Klick hast du im Netzwerk-Tab gesehen, und warum genau so viele?
> Antwort: 3.

> [!note]- Feedback von Claudian
> 6.2 saß. Dass du 6.5 und 6.6 trotz Lesen des ganzen Codes nicht beantworten konntest, liegt nicht an dir: Die Antwort steht nicht im Code, sondern in Reacts Innenleben (wo der State aufbewahrt wird). Das fehlende Denkmodell steht jetzt am Anfang des Lückenschlusses. 6.3: Nicht Pydantic, sondern **mypy** fängt es; nachgeprüft. 6.4: `useCallback` macht Funktionen nicht „gleich“, sondern gibt **dieselbe** zurück. 6.7: erwartet sind 2, bitte nachtragen, was die dritte Anfrage war.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 6 — Backend-Filter und Detailbereich]]

---

## Leseaufgabe 7 — Phase 5 Preislauf und Polling

**Commit:** `8a051f1`

**Ziel:** Du kannst erklären, wie man in React etwas **wiederholt im Hintergrund** tut (Polling), warum dafür ein Effect richtig ist, das Starten aber **kein** Effect ist, und was ohne Cleanup passiert. Das ist Plan-Schritt 11, laut Plan „der anspruchsvollste Schritt“.

### Was du vorher wissen musst

Fünf Dinge, die nicht im Code stehen, aber alles erklären:

**1. Ein Effect hat eine Lebensdauer.** Er startet nach einem Rendern, wenn sich seine Abhängigkeiten geändert haben. Sein Cleanup läuft, **bevor** er das nächste Mal startet, und wenn die Komponente verschwindet. Man kann einen Effect deshalb als „solange X gilt, halte Y am Laufen" lesen. Der Polling-Effect heißt übersetzt: *„Solange es einen aktiven Lauf gibt, frag alle 2 s nach."* Wird `aktiv` false, läuft das Cleanup und das Polling ist vorbei.

**2. Event-Handler vs. Effect.** Die Faustregel aus der React-Doku: Passiert etwas, **weil der Nutzer etwas getan hat** (Klick), gehört es in einen **Event-Handler**. Passiert es, **weil die Komponente gerade so aussieht** (es gibt einen aktiven Lauf → abfragen), gehört es in einen **Effect**. Deshalb ist `starten()` eine normale Funktion und kein `useEffect`.

**3. Timer in JavaScript.** `setTimeout(f, 2000)` ruft `f` einmal nach 2 s auf und gibt eine Nummer zurück; `clearTimeout(nummer)` sagt den Aufruf ab. `setInterval(f, 2000)` ruft `f` **alle** 2 s auf, bis `clearInterval`. Wichtig: Timer laufen **unabhängig von React** weiter. React weiß nichts von ihnen, und wenn die Komponente verschwindet, tickt der Timer trotzdem weiter. Nur ein Cleanup stoppt ihn.

**4. Die Updater-Funktion bei `setState`.** `setLauf(neu)` setzt einen Wert. `setLauf((vorher) => …)` bekommt den **aktuellen** Wert hereingereicht und gibt den neuen zurück. Das braucht man, wenn der neue Wert vom aktuellen abhängt und man nicht sicher ist, ob die eigene Variable noch aktuell ist, zum Beispiel in einem `.then`, das erst viel später ausgeführt wird.

**5. Datenbank-Sperre durch `TRUNCATE`.** Im Backend leert der Preislauf die Tabelle mit `TRUNCATE` und füllt sie in **derselben Transaktion** neu. `TRUNCATE` sperrt die Tabelle bis zum Commit komplett, auch für Leser. Eine Anfrage auf `/v1/recommendations` während des Laufs bekommt also keinen Fehler, sie **wartet**, bis zu 40 s.

### Lesereihenfolge

- [x] `src/hooks/usePreislauf.ts`, der Kern; die drei nummerierten Abschnitte einzeln lesen
- [x] `src/App.tsx`, Diff: wo der Hook aufgerufen wird und wohin die Werte gehen
- [x] `src/components/Aktionsleiste.tsx`
- [x] `src/components/Detailbereich.tsx`, Diff: der neue `if (laufAktiv)`-Zweig mit Kommentar
- [x] `src/lib/format.ts` und `src/api/types.ts`, nur die Diffs

### Ausprobieren

Jeder Preislauf dauert 20–45 s und berechnet die Empfehlungen neu; das ist gefahrlos.

- [x] Ein Produkt wählen, **Preislauf starten**. Was passiert mit Knopf, Statuszeile und Detailbereich? Netzwerk-Tab mit Filter „Fetch/XHR“ offen lassen.
- [x] Während der Lauf läuft: Seite **neu laden** (F5). Was zeigt die Statuszeile, und läuft das Polling weiter?
- [x] Zweiten Browser-Tab mit dem Dashboard öffnen, in Tab 1 starten. Was zeigt Tab 2, und warum ist sein Knopf gesperrt?
- [ ] In `usePreislauf.ts` im Polling-Effect die ganze `return () => { … }`-Cleanup-Funktion auskommentieren. Einen Lauf starten und den Netzwerk-Tab **nach** Laufende 20 s beobachten. **Danach zurückändern.**

### Fragen

7.1 Der Lauf-State liegt in `App` (über `usePreislauf`), nicht in der `Aktionsleiste`, obwohl dort der Knopf ist. Warum?
> Antwort: Weil der Detailbereich ebenfalls wissen muss, ob ein Preislauf aktiv ist.

7.2 Während eines Laufs zeigt der Detailbereich einen Hinweis statt der Tabelle. Was würde ohne diesen Zweig passieren, wenn du während des Laufs ein Produkt anklickst? Und wie kommt die Tabelle **nach** dem Lauf zurück, obwohl nirgends „jetzt neu laden" steht?
> Antwort: Weil dann der Effekt mit dem Polling es nicht mehr überschreibt und das letzte neu rendern den effekt des detailbereichs triggeret (also der Detailbereich weiß ja ob ein lauf aktiv ist oder nicht). Dann würde man noch die Ergebnisse des vorigen laufens sehen

7.3 Das Experiment ohne Cleanup: Was hast du nach Laufende im Netzwerk-Tab gesehen? Warum hört die Kette nicht von selbst auf, obwohl `aktiv` false geworden ist?
> Antwort: Weiß ich nicht

7.4 Warum ist `starten()` eine normale Funktion, die der Knopf aufruft, und kein `useEffect`? Versuch zu skizzieren, wie es als Effect aussehen müsste, und was daran umständlich wäre.
> Antwort: WEil es ein Event Handler ist, welcher durch einen Klick des benutzers ausgelöst wird und nicht durch "was zu sehen ist". Übrigens habe ich mich gefragt, warum man den Event handler nicht einfach in der Aktionsleiste schreien kann.

7.5 Im ersten Effect steht `setLauf((vorher) => vorher ?? letzter)` statt `setLauf(letzter)`. Beschreibe die Situation, in der der Unterschied zählt.
> Antwort: Ich weiß es nicht

7.6 Warum eine Kette aus `setTimeout` statt einem einfachen `setInterval`? (Steht im Code, versuch es aber in eigenen Worten.)
> Antwort: WEil sich so keine Läufe stappeln bei verzhögerungen

7.7 `STATUS_TEXT` ist ein `Record<RunStatus, string>`. Was passiert, wenn in `types.ts` ein fünfter Status `"cancelled"` dazukommt? Vergleich mit `assertNever`: Wann nimmt man was?
> Antwort: dann liefert der compiler einen error

> [!note]- Feedback von Claudian
> 7.1 und 7.6 saßen. 7.2: Die Tabelle kommt nicht übers Polling zurück, sondern durch Aus- und Wiedereinhängen; dass man ohne den Zweig alte Zahlen sähe, stimmt aber. 7.3 nachgestellt: ohne Cleanup alle 2 s weiter, auch nach dem Aushängen. 7.4: deine Zusatzfrage ist beantwortet. 7.5 ❌: Race Condition beim Laden + Closure (wie 6.5). 7.7: zusätzlich ein Fehler, den der Compiler *nicht* findet.
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 7 — Preislauf und Polling]]

---

## Leseaufgabe 8 — Phase 6 Aufräumen und Gesamtbild

**Commit:** `a650ffd`

**Ziel:** Der Commit selbst ist klein. Wichtiger ist der zweite Teil: das **ganze Projekt einmal von oben** sehen, so wie du es im Gespräch erklären würdest.

### Was du vorher wissen musst

**1. Kommentare beschreiben den Code, nicht seine Geschichte.** „Vor Phase 3 stand hier …" hilft beim Lernen, aber niemandem, der den Code später liest. Die Geschichte steht in den Commits (`git log`), der Kommentar sagt, **warum der Code jetzt so ist**.

**2. Toter Code ist nicht harmlos.** Eine Funktion, die nie aufgerufen wird, muss trotzdem gelesen, verstanden und bei Änderungen mitgepflegt werden, und sie täuscht vor, dass sie gebraucht wird. Wer sie braucht, holt sie aus der Git-Historie zurück.

**3. Das README ist die Visitenkarte.** Wer ein Repo öffnet, liest zuerst das README. Es sollte in zwei Minuten beantworten: Was ist das, wie starte ich es, wie ist es aufgebaut, was fehlt bewusst?

### Lesereihenfolge

- [ ] `git show --stat a650ffd`, dann den Diff von `src/api/types.ts` (`istFinal`) und `src/api/endpoints.ts`
- [ ] `frontend/README.md` **komplett**, als ob du das Projekt zum ersten Mal siehst
- [ ] Die 3-Minuten-Story in [[Umsetzungsstand (Pricing-Dashboard)#Die 3-Minuten-Story]] einmal laut lesen

### Ausprobieren

- [x] Die Story **ohne Vorlage** frei erzählen, Stoppuhr an. Wie lange hat es gedauert? Wo bist du hängen geblieben?
- [x] Die Abschnitte der Story, die nicht nach dir klingen, in eigenen Worten neu schreiben.

### Fragen

8.1 Die Funktionen `getMaxRecommendation` und `findRecommendation` waren fertig, getestet gegen die API, und kosteten nichts. Warum trotzdem löschen? Gibt es ein Gegenargument?
> Antwort: Sie werden nicht verwendet und auch toter Code hat kosten. Muss gewartet und verstanden werden. Und kann potenzielle Mitarbeiter verwirren (weil man erstmal davon ausgeht, dass der meiste Code gebraucht wird)

8.2 `istFinal` nutzt jetzt `Record<RunStatus, boolean>` statt einer Liste. Erklär in zwei Sätzen, welchen Fehler das verhindert. (Wiederholung aus 7.7.)
> Antwort: Sorgt dafür, dass der Compiler einen Error wirft, wenn man probiert andere Status zu nutzen

8.3 Zeichne den Weg eines Klicks auf ein Produkt bis zur fertigen Tabelle, als Liste der beteiligten Dateien in Reihenfolge. Nutze die „Aufbau"-Übersicht im README.
> Antwort: Als erstes leitet der Browser es an den Event handler weiter. Dieser wird von React resolved und löst einen useApi und damit ein fetch aus, welches den Endpunkt aufruft. Diese wird dann von der starlette middleware verarbeitet und dann von pydantic angenommen. Von dort wird es durch die schichten zu der persistence schicht gereicht, welche die relevanten informationen liefert bzw aus der datenbank holt. Das wird dann zurück gereicht von pydantic zu json serialisiert und dann von react passend verarbietet. dann wird der entsprechende state upgedated und es wird neu gerendert und effekte werden verwendet

8.4 Welche Stelle der 3-Minuten-Story fiel dir beim freien Erzählen am schwersten, und welche Nachfrage dazu würdest du am meisten fürchten?
> Antwort: - Ist nicht so wichtig. Wurde schon gehired das ist mehr ein Fun Übungsprojekt

> [!note]- Feedback von Claudian
> 8.1 saß. 8.2: Es geht um einen *neu hinzugefügten* Status, nicht ums Benutzen. 8.3: Backend-Weg gut, aber der Klick ändert nur State; der Fetch kommt erst nach Render und Commit aus dem Effect. Die Dateien in Reihenfolge stehen im Lückenschluss. 8.4: Glückwunsch zur Stelle!
>
> → Ausführlich: [[Lückenschluss (Pricing-Dashboard)#Leseaufgabe 8 — Aufräumen und Gesamtbild]]

---

## Leseaufgabe 9 — Phase 7 Ingest und ein Hook für zwei Läufe

**Commits:** `aaedccd` (Backend), `504466c` (Frontend). Tipp: `git show -M 504466c` zeigt `usePreislauf.ts → useLauf.ts` als **Umbenennung mit Änderungen**, statt als gelöschte und neue Datei.

**Ziel:** Du verstehst, wie man zwei fast gleiche Abläufe in **einen** generischen Hook zieht, und warum die Sperre zusätzlich im Backend sitzt.

### Was du vorher wissen musst

**1. Generics in TypeScript.** `function useLauf<T extends Hintergrundlauf>(quelle: LaufQuelle<T>): Lauf<T>` heißt: „`T` ist ein Platzhalter für einen Typ, den der Aufrufer festlegt, und der mindestens die Felder von `Hintergrundlauf` hat." Bei `useLauf(PREISLAUF)` ist `T` = `PricingRun`, bei `useLauf(INGEST)` ist `T` = `IngestionRun`. Der Hook selbst darf nur die gemeinsamen Felder benutzen (`id`, `status` …), gibt aber den **konkreten** Typ zurück. In Python wäre das `TypeVar("T", bound=Hintergrundlauf)`.

**2. `interface B extends A`** übernimmt alle Felder von `A` und fügt eigene hinzu. `PricingRun` und `IngestionRun` erben so von `Hintergrundlauf`.

**3. Daten statt Code unterscheiden lassen.** Statt zwei Hooks mit `if (art === "ingest")` steckt der Unterschied in einem **Objekt mit drei Funktionen** (`LaufQuelle`). Der Hook ruft immer `quelle.starten()` auf und weiß nicht, was dahinter passiert. Das ist das Strategy-Muster, das du aus dem Backend kennst (`RecommendationStrategy`).

**4. Eine Prüfung im Frontend ist keine Sicherheit.** Das Frontend ist nur *ein* Weg zur API. Swagger, curl, ein zweiter Tab: Alles umgeht gesperrte Knöpfe. Eine Regel, die immer gelten soll, gehört deshalb ins Backend; das Frontend macht sie nur sichtbar.

### Lesereihenfolge

- [ ] Backend: `git show aaedccd`, die beiden Router und `tests/test_api_laeufe.py` (wie werden die Hintergrundjobs im Test ausgeschaltet?)
- [ ] `src/api/types.ts`: `Hintergrundlauf`, `extends`
- [ ] `src/hooks/useLauf.ts`, mit `git show -M 504466c -- frontend/src/hooks/` als Diff gegen den alten `usePreislauf`
- [ ] `src/components/Aktionsleiste.tsx`: `brauchtPreislauf`, die generische `LaufSteuerung` mit `details`
- [ ] `src/App.tsx`: zwei Aufrufe desselben Hooks, `sperrgrundEmpfehlungen`

### Ausprobieren

Den Container vorher neu bauen (`docker compose up -d --build`), sonst fehlt die Backend-Sperre.

- [ ] Ingest starten. Was sieht man in Produktliste, Detailbereich und an beiden Knöpfen? Danach: Was zeigt der gelbe Hinweis, und was passiert nach F5?
- [ ] Während der Ingest läuft, in Swagger (`/docs`) `POST /v1/pricing-runs` ausführen.

### Fragen

9.1 Warum stehen `PREISLAUF` und `INGEST` als Konstanten **außerhalb** jeder Komponente? Was würde passieren, wenn man in `App` `useLauf({ letzter: …, abfragen: …, starten: … })` schriebe? (Denk an 5.2.)
> Antwort:

9.2 `details` in `LaufSteuerung` ist eine Funktion `(lauf: T) => ReactNode`. Warum kann die Preislauf-Zeile darin `lauf.recommendation_count` lesen, obwohl `LaufSteuerung` selbst nur `Hintergrundlauf` kennt?
> Antwort:

9.3 Der Hinweis „Empfehlungen leer" ist kein State, sondern wird aus den beiden letzten Läufen berechnet. Welchen Vorteil hat das gegenüber einem `useState(false)`, das man nach dem Ingest auf `true` setzt?
> Antwort:

9.4 Warum reicht es nicht, im Frontend beide Knöpfe zu sperren? Nenne zwei Wege, auf denen trotzdem beide Läufe gleichzeitig starten könnten.
> Antwort:

9.5 Im Test werden `execute_run` und `execute_ingestion_run` per `monkeypatch` ersetzt. Was würde ohne das passieren?
> Antwort:

> [!note]- Feedback von Claudian
> *(folgt nach deinen Antworten)*

---

## Leseaufgabe 10 — Phase 8 Tailwind CSS

**Commits:** `aebd5c1` (Tailwind), danach Phase 8b (mehr Hellblau, Symbole, Balken; `git log --oneline -3`). Zuerst `git show --stat aebd5c1`: Fast jede Komponente ist betroffen, die Logik aber nirgends. Achte im Diff darauf, dass sich nur `className` und Markup ändern.

![[Pricing-Dashboard Design.png]]

### Was du vorher wissen musst

**1. Utility-First.** Statt einer Klasse `.karte` mit fünf Regeln in einer CSS-Datei schreibt man fünf kleine Klassen direkt ans Element: `rounded-xl bg-neutral-100 p-6`. Jede macht genau eine Sache (`p-6` = Innenabstand 1,5 rem). Vorteil: Man sieht im JSX, wie etwas aussieht, und es gibt keine verwaisten CSS-Regeln. Nachteil: lange Klassenlisten, deshalb zieht man Wiederholungen in Komponenten (`ui.tsx`).

**2. Präfixe sind Bedingungen.** `hover:bg-white` gilt nur beim Überfahren, `disabled:bg-neutral-200` nur bei gesperrtem Knopf, `lg:grid-cols-[22rem_1fr]` erst ab großen Bildschirmen (Mobile-First: ohne Präfix gilt es überall, mit Präfix ab der Breite). `[22rem_1fr]` ist ein **beliebiger Wert**, der Unterstrich steht für ein Leerzeichen.

**3. Tailwind liest deinen Code als Text.** Beim Bauen durchsucht es alle Dateien nach Wörtern, die wie Klassen aussehen, und erzeugt **nur dafür** CSS. Ein Name, der erst zur Laufzeit zusammengesetzt wird (`` `text-${farbe}-700` ``), steht nirgends wörtlich im Code und bekommt kein CSS. Deshalb `STATUS_FARBE` mit vollständigen Namen.

**4. `@theme` erweitert das System.** `--color-haeger-400: #5bb5d1` in `index.css` erzeugt automatisch `bg-haeger-400`, `text-haeger-400`, `ring-haeger-400` usw. Eine `tailwind.config.js` braucht Version 4 dafür nicht mehr.

### Lesereihenfolge

- [ ] `src/index.css`: der ganze Rest vom alten CSS
- [ ] `vite.config.ts`: das Plugin
- [ ] `src/components/ui.tsx`
- [ ] `src/App.tsx`: Kopfleiste, Wortmarke, Grid
- [ ] `src/components/Aktionsleiste.tsx`: `STATUS_FARBE`, `KNOPF_VARIANTE`
- [ ] `src/components/Empfehlungen.tsx`: Tabelle, `TH`/`TD`/`ZAHL`, `Kennzahl`

### Ausprobieren

- [ ] Browserfenster schmal ziehen (oder DevTools → Gerätesymbol). Ab welcher Breite springen die zwei Spalten untereinander, und welche Klasse entscheidet das?
- [ ] In `Aktionsleiste.tsx` bei `succeeded` die Farbe zu `` `text-${"emerald"}-700` `` umschreiben. Was passiert mit der Farbe? **Zurückändern.**
- [ ] In `index.css` den Wert von `--color-haeger-400` ändern und speichern. Was ändert sich überall?

### Fragen

10.1 Warum stehen wiederholte Stile als **Komponenten** (`Karte`) in `ui.tsx` und nicht als CSS-Klasse mit `@apply`?
> Antwort:

10.2 Erkläre mit eigenen Worten, warum `` `text-${farbe}-700` `` nicht funktioniert, obwohl im Browser der richtige Klassenname am Element steht.
> Antwort:

10.3 In `LaufSteuerung` steht ``className={`w-44 ${KNOPF_VARIANTE[variante]}`}``. Was ist `keyof typeof KNOPF_VARIANTE`, und welchen Fehler verhindert dieser Typ?
> Antwort:

10.4 In Phase 8 war der Hauptknopf dunkler als das Logo-Blau, in Phase 8b hat er genau das Logo-Blau, aber **dunkle** Schrift. Warum ging beides nicht mit weißer Schrift auf dem hellen Blau?
> Antwort:

10.5 In `AuslastungsBalken` steht die Breite in `style={{ width: … }}` statt in einer Klasse wie `w-1/3`. Warum?
> Antwort:

> [!note]- Feedback von Claudian
> *(folgt nach deinen Antworten)*
