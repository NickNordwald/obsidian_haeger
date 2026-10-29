---
tags:
  - documentation
---

# Umsetzungsstand — Pricing-Dashboard

Laufendes Statusdokument zum React-Frontend. Plan: [[Pricing-Dashboard Plan & Syntax-Tutorial]] · Lernplan: [[React Crashkurs - Übersicht]] · Dateien erklärt: [[Frontend-Projektaufbau erklärt]] · Leseaufgaben: [[Code-Review-Aufgaben (Pricing-Dashboard)]] · Lücken erklärt: [[Lückenschluss (Pricing-Dashboard)]]

| | |
|---|---|
| Projektordner | `~/HaegerConsulting/Projects/Interviews/ReactBasics/FrontEndBuchungsSystem/frontend` |
| Backend | derselbe Repo-Ordner eine Ebene höher (`FrontEndBuchungsSystem/`, Stand der Docker-Version), API unter `127.0.0.1:8000` |
| Stack | React 19, TypeScript 6, Vite 8, Tailwind CSS 4, ESLint, Prettier — noch kein Router |
| Stand | Phasen 0–8 fertig · Leseaufgaben 1–8 beantwortet, 9–10 offen · nächste Phase: 9 (Routing) |
| Git | Branch `frontend`, ein Commit pro Phase (siehe Entscheidungen) |

## Arbeitsmodus (seit 30.09.2026)

Geändert gegenüber dem ursprünglichen Plan, weil die Zeit fehlt:

| Wer | Was |
|---|---|
| **Claudian** | Baut Phase für Phase, erklärt im Code (Kommentare) und im Vault, führt diese Notiz |
| **Nick** | Liest den Code als Reviewer anhand der [[Code-Review-Aufgaben (Pricing-Dashboard)\|Leseaufgaben]], beantwortet die Fragen kurz, **trifft die Entscheidungen** |
| **Claudian, nach jeder Leseaufgabe** | Bewertet die Antworten und schließt die Lücken ausführlich in [[Lückenschluss (Pricing-Dashboard)]] (seit 14.10.) |
| **Zusammen** | Offene Entscheidungen stehen unten, bevor die betroffene Phase gebaut wird |

Der ursprüngliche [[Pricing-Dashboard Plan & Syntax-Tutorial|Plan & Syntax-Tutorial]] (Schritte 0–12, Nick schreibt selbst) ist erledigt und archiviert; die Spalte „Plan-Schritte" unten zeigt, welche Phase welche Schritte abgedeckt hat.

## Fortschritt

| Phase | Inhalt | Plan-Schritte | Status |
|---|---|---|---|
| 0 | Gerüst: Vite-Projekt, Proxy statt CORS, `.env`, Template aufgeräumt, Seitenlayout, API-Status-Anzeige | 0, 0b, 1 | ✅ 30.09. — Review: Leseaufgabe 1 + 3 |
| 1 | API-Schicht: `client.ts`, `endpoints.ts`, `types.ts` | (Vorgriff auf 9/10) | ✅ war vorbereitet — Review: Leseaufgabe 2 |
| 2 | Produktliste: laden, Lade-/Fehler-/Leer-Zustand, Keys, Suchfeld, abgeleiteter State, Auswahl nach oben ziehen | 2–8 | ✅ 30.09. `7a53e5d` — Review: Leseaufgabe 4 |
| 3 | Eigener Hook `useApi` mit Abbruch (Race Condition); vorab Prettier (`3cc4fda`) und `assertNever` (`30cceab`) | 9 | ✅ 14.10. `c0a335f` — Review: Leseaufgabe 5 |
| 4 | Backend: `product_id`-Filter an `GET /v1/recommendations` (+ Test), dann Detailbereich mit den Empfehlungen zum gewählten Produkt | 8 | ✅ 14.10. `18ac2a7` (Backend), `9a675dc` (Frontend) — Review: Leseaufgabe 6 |
| 5 | Preislauf starten, Status per Polling verfolgen, danach neu laden | 11 | ✅ 15.10. `8a051f1` — Review: Leseaufgabe 7 |
| 6 | Aufräumen (toter Code, Kommentare, `istFinal` als `Record`), README, Entwurf der 3-Minuten-Story | 12 | ✅ 20.10. `a650ffd` — Review: Leseaufgabe 8; Story: Nick |
| 7 | Ingest aus der Web-App: Backend sperrt Preislauf ⇄ Ingest gegenseitig (409, +4 Tests), generischer Hook `useLauf`, Hinweis „Empfehlungen leer → Preislauf“ | — | ✅ 20.10. `aaedccd` (Backend), `504466c` (Frontend) — Review: Leseaufgabe 9 |
| 8 | Tailwind CSS 4, Auftritt im haeger-Stil („Hotelbuchungssystem — a haeger product“), UI-Bausteine in `ui.tsx` | — | ✅ 20.10. `aebd5c1` — Review: Leseaufgabe 10 |
| 8b | Mehr haeger-Blau: Symbole (lucide-react), Aktions-Kacheln, Balken für Auslastung und Preisanpassung, Hinweis-Box hellblau | — | ✅ 20.10. — Review: Leseaufgabe 10 (ergänzt) |
| 8c | Akzentfarbe #98F5FF (Nicks Wunsch), Skala neu, kontrastschwache Stellen auf dunklere Töne derselben Farbe umgestellt | — | ✅ 20.10. |

## Aktueller Stand

![[Pricing-Dashboard Design.png]]

## Roadmap (ab 20.10.2026)

Das Projekt läuft als Lernprojekt weiter. Nicks Auswahl vom 20.10., in sinnvoller Reihenfolge:

| Phase | Inhalt | Neu zu lernen | Warum an dieser Stelle |
|---|---|---|---|
| 8 | **Tailwind CSS** + Branding „Hotelbuchungssystem — a Haeger product“, Optik nach Nicks Screenshot | Utility-First-CSS, Tailwind v4 mit Vite | Alles Spätere entsteht direkt im neuen Stil, statt zweimal gestylt zu werden |
| 9 | **Routing**: echte Seiten (Dashboard, Produkt-Detail mit URL wie `/produkte/RFJA3D`, Läufe) | React Router, URL als State | Gibt den Rahmen, in den Historie und Diagramm als Seiten passen |
| 10 | **Laufhistorie** als eigene Seite | Tabellen mit Paginierung | Endpunkte existieren schon, schneller Gewinn |
| 11 | **Diagramm** im Produkt-Detail: Preisanpassung und Auslastung über das Reisedatum | Chart-Bibliothek (z. B. Recharts) | Macht die Empfehlungen anschaulich |
| 12 | **Sortieren & Filtern**: Spalten sortierbar, Zimmertyp-Filter, Zeitraum | viel abgeleiteter State, ggf. in der URL | Baut auf Routing auf (Filter teilbar per Link) |
| 13 | **Toasts & Dialoge**: „Preislauf fertig", eigener Bestätigungsdialog statt `window.confirm` | Portale, Fokus-Management | Ersetzt das Provisorium aus Phase 7 |
| 14 | **Tests**: Vitest + React Testing Library für `useApi`, `useLauf`, Komponenten | Testen mit Fake-Timern und gemocktem `fetch` | Sichert das Verhalten ab, **bevor** Phase 15 die Hooks austauscht |
| 15 | **TanStack Query** statt der eigenen Hooks, Vergleich vorher/nachher | Server-State-Bibliothek: Cache, Refetch, Polling | Lehrreich, weil du die Innereien schon selbst gebaut hast; die Tests aus 14 zeigen, dass sich nichts verändert |

## Entscheidungen

*(Was bewusst so und nicht anders gemacht wurde — mit Begründung. Das sind die Stellen, an denen ein Gespräch interessant wird.)*

- **Review per Git-Commit** (Nick, 30.09.): Baseline-Commit `0734487` auf Branch `frontend`, danach ein Commit pro Phase. Jede Phase wird als Diff gelesen (`git show <commit>`), wie in einem echten Code-Review.
- **Backend wird für den Detailbereich erweitert** (Nick, 30.09.): `GET /v1/recommendations` bekommt einen optionalen Query-Parameter `product_id` statt eines reinen Frontend-Workarounds über `by-unit`. So zeigt der Detailbereich alle Empfehlungen eines Produkts, und das Projekt reicht über Frontend und Backend.
- **Zustand einer Anfrage als Discriminated Union** (`{ art: 'laedt' } | { art: 'fehler', … } | …`) statt drei getrennter `useState` für `daten`/`laedt`/`fehler`: Unmögliche Kombinationen („lädt **und** Fehler **und** Daten") lassen sich gar nicht erst ausdrücken.
- **Produkte werden in der Liste geladen, nicht in `App`**: Nur die Liste braucht sie. Hochgezogen wird allein die Auswahl, weil Liste und Detail sie teilen.
- **Auswahl bleibt bestehen, wenn die Suche das Produkt ausblendet** (Nick, 14.10., Leseaufgabe 4.7): Die Auswahl ist eine bewusste Handlung, die Suche nur ein Blick auf die Liste. Die Alternative wäre ohne Effect, als abgeleiteter Wert lösbar, siehe [[Lückenschluss (Pricing-Dashboard)#4.7 — Auswahl bei ausgeblendetem Produkt ✅]].
- **Prettier als Projektformatierer** (14.10.): VS Code formatierte ohnehin beim Speichern mit Prettier, die übrigen Dateien aber nicht → laute Diffs. Jetzt Projektabhängigkeit mit Standardeinstellungen, alles einmal in einem eigenen Commit durchformatiert.
- **`assertNever` für vollständige `switch`-Anweisungen** (14.10.): Ein Union-Typ allein meldet keinen vergessenen `case` (Leseaufgabe 3.5, nachgeprüft).
- **`useApi(laden)` nimmt eine Funktion aus `endpoints.ts`, keinen URL-Pfad** (14.10.): So bleibt die Schichtung erhalten, Komponenten kennen weiterhin keine URLs. Preis: Die Funktion muss stabil sein; für Funktionen mit Parametern (Phase 4) braucht es `useCallback`.
- **„lädt" im Hook abgeleitet statt per `setState` im Effekt** (14.10.): Das Ergebnis merkt sich seine Ladefunktion; passt sie nicht mehr, gilt „lädt". Spart einen Render und vermeidet die ESLint-Regel `set-state-in-effect`.
- **TypeScript statt JavaScript** (abweichend vom Plan, beim Anlegen mit `--template react-ts` gewählt): Die API-Typen machen Stolperstellen sichtbar, die in JS erst zur Laufzeit auffallen — z. B. dass Pydantic `Decimal` als String schickt.
- **Vite-Proxy statt CORS im Backend** (ersetzt Plan-Schritt 0b): Browser spricht nur mit `localhost:5173`, also gleiche Herkunft, kein Backend-Eingriff nötig. Nachteil: gilt nur im Dev-Server.
- **Eigene API-Schicht** (`src/api/`): Komponenten kennen weder URLs noch `fetch` noch Header — gleiche Schichtidee wie im Backend.
- **API-Schlüssel im Bundle** (`VITE_API_KEY`): bewusst in Kauf genommen, nur lokal. Im Gespräch selbst ansprechen.
- **Kein Context für den API-Schlüssel** (Plan-Schritt 10 entfällt in dieser Form): Der Schlüssel ist eine Konstante in `client.ts`, keine Komponente braucht ihn — es gibt also nichts durchzureichen und damit kein Prop Drilling, das Context lösen müsste. Context kommt nur, falls wir später echten geteilten State haben.
- **Template aufgeräumt** (30.09.): Demo-Seite, Logos, Demo-CSS und Template-README entfernt; Details in [[Frontend-Projektaufbau erklärt#Was aus dem Template gelöscht wurde]].

- **Filter nach Produkt sortiert nach Reisedatum** (14.10.): Ohne Filter bleibt die Reihenfolge nach `id` (bestehende Aufrufe unverändert), mit Filter liest man die Empfehlungen als Verlauf. Der Unique-Index `(product_id, calendar_date)` auf `capacity` deckt Filter und Sortierung ab.
- **Eine Anfrage mit `limit=500` statt Blättern** (14.10.): Gegen echte Daten geprüft, pro Produkt höchstens ~430 Empfehlungen. Erreicht ein Produkt 500, zeigt die Oberfläche einen Hinweis statt still abzuschneiden.
- **`Empfehlungen` als eigene Komponente mit `key={produktId}`** (14.10.): `useApi` darf nicht hinter einem `if (produktId === null) return` stehen (Rules of Hooks). Der `key` sorgt dafür, dass jedes Produkt eine frische Instanz bekommt.

- **Preislauf-State in `App` über den Hook `usePreislauf`** (15.10.): Nicht nur die Aktionsleiste braucht ihn, auch der Detailbereich. Grund: Der Lauf sperrt die Empfehlungstabelle per `TRUNCATE` in einer Transaktion, Leseanfragen hängen bis zu 40 s.
- **Detailbereich hängt `Empfehlungen` während eines Laufs aus** (15.10.): Statt einer hängenden Anfrage ein Hinweis. Das Wiedereinhängen nach dem Lauf lädt automatisch frisch, ein eigenes „neu laden"-Signal entfällt.
- **Polling per `setTimeout`-Kette statt `setInterval`** (15.10.): Die nächste Abfrage wird erst nach der Antwort geplant, Anfragen können sich nicht stapeln oder überholen.
- **Beim Laden den letzten Lauf holen, bei 409 den laufenden übernehmen** (15.10.): Ein Lauf aus einem anderen Tab oder vor einem Neuladen wird angezeigt und verfolgt, statt dass die Seite „kein Lauf" behauptet.
- **Preislauf-Texte per `Record<RunStatus, string>`** (15.10.): Reine Zuordnung Wert → Text; der Compiler verlangt für jeden Status einen Eintrag.

- **Preislauf und Ingest sperren sich gegenseitig, im Backend** (Nick, 20.10.): Das Backend ist die Quelle der Wahrheit; auch Swagger, curl und andere Tabs bekommen 409. Das Frontend sperrt zusätzlich beide Knöpfe. Bekannte Grenze: Prüfen und Anlegen sind nicht atomar.
- **Nach einem Ingest: Hinweis + Knopf statt automatischem Preislauf** (Nick, 20.10.): volle Kontrolle, einfacher Zustand. Der Hinweis ist abgeleitet (erfolgreicher Ingest ohne späteren erfolgreichen Preislauf) und stimmt deshalb auch nach einem Neuladen.
- **Ein generischer Hook `useLauf` statt zwei Kopien** (20.10.): Ingest und Preislauf unterscheiden sich nur in drei API-Aufrufen, gebündelt als stabile `LaufQuelle`-Konstanten.
- **`window.confirm` vor dem Ingest als Provisorium** (20.10.): ein eigener Dialog folgt in Phase 13.

- **Tailwind v4 ohne Konfigurationsdatei** (20.10.): Farben und Schrift stehen im `@theme`-Block von `index.css`; die eigenen CSS-Regeln sind vollständig ersetzt.
- **Wiederholte Stile als Komponenten, nicht als `@apply`-Klassen** (20.10.): `Karte`, `Kartentitel`, `Seitentitel`, `Hinweis` in `components/ui.tsx`. Das Aussehen bleibt im JSX sichtbar und steht trotzdem nur einmal da.
- **Hauptknopf in `haeger-600` statt im hellen Markenton** (20.10.): Weiß auf `haeger-400` hat 2,3 : 1 Kontrast, WCAG AA verlangt 4,5 : 1. Das helle Petrol bleibt für Akzente.
- **Schrift selbst ausgeliefert** (`@fontsource-variable/open-sans`, 20.10.): keine Abhängigkeit von Google-Servern, kein Datenschutzthema, funktioniert offline.

- **Hauptknopf in vollem haeger-Blau mit dunkler Schrift** (Nick: „viel mehr Hellblau“, 20.10.): `tinte` auf `haeger-400` hat 5,0 : 1 Kontrast, Weiß nur 2,3 : 1. Ersetzt den dunkleren `haeger-600`-Knopf aus Phase 8. Faustregel in `ui.tsx` notiert.
- **Symbole aus `lucide-react`** (20.10.): einzeln importierbare SVG-Komponenten, nur die benutzten landen im Bundle; Optik wie die Linien-Icons auf haeger.de.
- **Akzentfarbe #98F5FF** (Nick, 20.10.): als `haeger-400`, Skala 50–900 darum gebaut. Darauf dunkle Schrift (9,4 : 1); auf Weiß nur dekorativ (1,25 : 1). Bedeutungsträger auf Weiß deshalb in `haeger-600` (Ränder, Symbole, Fokus, 3,5 : 1) bzw. `haeger-700/800` (Schrift).
- **Datenabhängige Breiten über `style`, alles Feste über Klassen** (20.10.): Balkenbreiten wie `33 %` lassen sich nicht als Tailwind-Klasse vorab erzeugen.

## Gelernte Lektionen

- **Manches steht nicht im Code** (15.10.): Leseaufgabe 6 war trotz vollständigem Lesen nicht beantwortbar, weil die Antworten in Reacts Laufzeitverhalten liegen (Hook-Speicherplätze nach Reihenfolge, Instanz-Identität über Typ + Position + Key). Konsequenz: Ab Leseaufgabe 7 bekommt jede Aufgabe einen kurzen Block „Was du vorher wissen musst“.
- **Keyword-Argumente schützen vor still verschobenen Parametern** (15.10., Leseaufgabe 6.3): Der alte Positionsaufruf hätte `product_id=50, limit=0` ergeben; gefangen von mypy und zwei Tests, zur Laufzeit ein 500 von Postgres. Pydantic war daran nicht beteiligt.
- **FastAPI ignoriert unbekannte Query-Parameter still.** Der alte Docker-Container lieferte bei `?product_id=…` einfach alle Empfehlungen, ohne Fehler, und das Dashboard zeigte für jedes Produkt dieselbe Tabelle. Nach Backend-Änderungen den Container neu bauen (`docker compose up -d --build`).
- **Tailwind erzeugt nur Klassen, die wörtlich im Quelltext stehen** (20.10.): `status--${x}` oder `text-${farbe}-700` bekommen kein CSS. Lösung: vollständige Klassennamen in einem `Record`.
- **Ein Union-Typ erzwingt keine vollständigen `switch`-Anweisungen.** Fehlt ein `case`, liefert die Komponente still `undefined`. Abhilfe: `assertNever`.
- **Typfehler stoppen den Dev-Server nicht.** Vite streicht Typen nur weg; geprüft wird im Editor und bei `npm run build`. Siehe [[Frontend-Projektaufbau erklärt#Warum Vite die Typen nicht prüft]].

## Die 3-Minuten-Story

*(Vier Absätze, laut sprechbar. So beantwortest du „Erzählen Sie mal, was Sie gebaut haben".)*

> [!warning] Entwurf von Claudian, 20.10. — in deine eigenen Worte bringen
> Die Fakten stimmen, die Formulierung ist meine. Fremde, auswendig gelernte Sätze hört man im Gespräch sofort heraus. Vorgehen: einmal laut vorlesen, dann **ohne Vorlage** frei nacherzählen, dabei deine eigenen Formulierungen hier hineinschreiben. Ziel ~3 Minuten, gestoppt.

**1. Der Ausgangspunkt.** In der ersten Phase meines Traineeships habe ich eine Backend-Anwendung für Hotel-Preisempfehlungen gebaut. Sie liest Buchungen und Kapazitäten aus CSV-Dateien in eine Postgres-Datenbank und berechnet für jedes Produkt und jeden künftigen Reisetag eine Preisanpassung: Bei mehr als halber Auslastung geht der Preis nach oben, darunter nach unten, und je weiter das Reisedatum entfernt ist, desto stärker. Das Ganze ist eine FastAPI-Anwendung in Schichtenarchitektur, mit SQLAlchemy, Alembic-Migrationen, API-Keys und Docker Compose. Ein Preislauf rechnet gut 80 000 Empfehlungen und läuft als Hintergrund-Job: Die API antwortet sofort mit 202, den Fortschritt fragt man über einen Status-Endpunkt ab.

**2. Was ich gebaut habe.** Um React zu lernen, habe ich dafür ein Dashboard gebaut, mit React 19, TypeScript und Vite. Links eine durchsuchbare Produktliste, rechts die Empfehlungen zum gewählten Produkt als Tabelle mit ein paar Kennzahlen, und oben ein Knopf, der einen Preislauf startet und seinen Status live verfolgt, bis er fertig ist. Danach lädt die Tabelle automatisch die neuen Zahlen. Dafür musste ich auch das Backend erweitern: Der Empfehlungs-Endpunkt konnte nicht nach Produkt filtern, also habe ich einen Filter mit Join auf die Kapazitätstabelle und Tests ergänzt.

**3. Woran ich etwas gelernt habe.** Am meisten habe ich beim Preislauf gelernt. Der Status wird alle zwei Sekunden abgefragt, und das läuft in einem `useEffect`. Wichtig war das Aufräumen: Der Effect startet eine Kette von Timern, und die gehören nicht React. Ohne Cleanup läuft die Abfrage nach Laufende einfach weiter, auch wenn die Komponente längst weg ist; das habe ich bewusst ausprobiert. Und dabei bin ich auf etwas im Backend gestoßen: Der Preislauf leert die Tabelle per `TRUNCATE` in einer Transaktion, und so lange hängt jede Leseanfrage. Deshalb zeigt der Detailbereich während eines Laufs einen Hinweis und hängt die Tabelle aus. Wenn der Lauf fertig ist, wird sie neu eingehängt und lädt dadurch von selbst frisch, ohne einen eigenen „neu laden"-Mechanismus. Ähnlich beim Laden generell: Mein eigener Hook bricht veraltete Anfragen ab und merkt sich, zu welcher Anfrage ein Ergebnis gehört, sodass bei schnellem Klicken nie die Daten des falschen Produkts erscheinen.

**4. Was ich bewusst weggelassen habe.** Es gibt keine Frontend-Tests. Das Verhalten habe ich gegen die echte API geprüft, aber der nächste Schritt wären Vitest und React Testing Library für die beiden Hooks. Der API-Schlüssel steckt im Browser-Bundle und ist damit lesbar; für ein lokales Projekt in Ordnung, in Produktion bräuchte es ein Login oder ein Backend-for-Frontend. Die API-Antworten werden im Frontend nicht zur Laufzeit geprüft, anders als mit Pydantic im Backend; dafür gäbe es Zod oder aus OpenAPI erzeugte Typen. Und ich habe auf Router, State-Bibliotheken und Caching wie TanStack Query verzichtet, weil ich die Grundlagen erst ohne Abstraktion verstehen wollte.

### Wahrscheinliche Nachfragen

Zu jeder Story-Aussage die Stelle, an der die Antwort steht:

| Nachfrage | Nachlesen |
|---|---|
| „Warum nicht einfach `setInterval`?" | [[Lückenschluss (Pricing-Dashboard)#7.6 — `setTimeout`-Kette statt `setInterval` ✅\|7.6]] |
| „Was genau passiert ohne Cleanup?" | [[Lückenschluss (Pricing-Dashboard)#7.3 — Polling ohne Cleanup ⏭️\|7.3]] |
| „Wie verhindert Ihr Hook die Race Condition?" | [[Lückenschluss (Pricing-Dashboard)#5.4 — Die zwei Schutzmechanismen gegen die Race Condition 🟡\|5.4]] |
| „Warum wird die Tabelle durch Einhängen neu geladen?" | [[Lückenschluss (Pricing-Dashboard)#Das Denkmodell — wo React den State aufbewahrt\|Denkmodell]], [[Lückenschluss (Pricing-Dashboard)#7.2 — Der Hinweis im Detailbereich, und wie die Tabelle zurückkommt 🟡\|7.2]] |
| „Wann braucht man keinen Effect?" | [[Lückenschluss (Pricing-Dashboard)#4.2 — Die falsche Variante mit zweitem State ❌\|4.2]] |
| „Wofür `useCallback`?" | [[Lückenschluss (Pricing-Dashboard)#6.4 — Wozu `useCallback`? 🟡\|6.4]] |
| „Wie redet das Frontend mit der API, CORS?" | [[Lückenschluss (Pricing-Dashboard)#1.4 — Der Weg eines Requests durch den Proxy ❌\|1.4]] |
| „Was haben Sie im Backend geändert?" | [[Lückenschluss (Pricing-Dashboard)#Exkurs — `relationship`, `joinedload` und `join` im Backend\|Exkurs Join]] |
