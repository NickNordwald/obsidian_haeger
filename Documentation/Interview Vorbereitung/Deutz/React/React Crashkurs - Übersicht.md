---
tags:
  - documentation
---

# React Crashkurs — Übersicht (Deutz)

Einstieg in alles zu React. Ursprünglich ein Fünf-Tage-Plan (21.–25.09.2026) für das Gespräch am 26./27.09.; seither als Lernthema mit Übungsprojekt weitergeführt.

Übergeordnet: [[Interview Vorbereitung]] · Quellen: [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste]] · Projekt: [[Umsetzungsstand (Pricing-Dashboard)]] · Generalprobe: [[Interview-Fragenkatalog]]

## Ausgangslage

| | |
|---|---|
| **Vorwissen** | Python/Backend solide (siehe [[Documentation/Traineeship/Grundlagen der Software Entwicklung/1.5 REST APIs & Layered Architecture/1.5 REST APIs & Layered Architecture - Vertiefung\|Kapitel 1.5]]), JavaScript/Frontend praktisch keins |
| **Interviewformat** | Konzeptfragen im Gespräch — **erklären, nicht tippen** |
| **Rolle** | noch unklar → Plan zielt auf solide Grundlagen mit ehrlicher Tiefe im Kern |
| **Zeit** | ~22–24 h netto |

> [!important] Das Ziel ist nicht „React können", sondern „über React sprechen können"
> In vier Tagen wird niemand React-Entwickler. Erreichbar ist etwas anderes und für ein Konzeptgespräch Wertvolleres: das **mentale Modell** von React sauber erklären können, die Fachbegriffe richtig verwenden, an einem selbst gebauten Beispiel erzählen — und die eigenen Grenzen präzise benennen statt sie zu überspielen. Ein Kandidat, der sagt „Effects habe ich benutzt, aber wann man sie *nicht* braucht, habe ich erst halb verstanden — hier ist mein Stand" wirkt stärker als einer, der auswendig Gelerntes abspult und bei der zweiten Nachfrage kippt.

## Was am Ende sitzen muss

- **Das Rendering-Modell**: Komponente = Funktion von State/Props auf UI-Beschreibung. Render → Commit. Warum React deklarativ ist und was der Unterschied zu imperativer DOM-Manipulation ist.
- **State**: was State ist, warum er unveränderlich behandelt wird, warum ein Render einen *Snapshot* sieht, wie Batching funktioniert.
- **Props & Datenfluss**: einseitiger Datenfluss von oben nach unten, Lifting State Up, wann Context sinnvoll wird.
- **Hooks**: `useState`, `useEffect`, `useRef`, `useContext` — und die Regeln, warum es Regeln gibt.
- **Listen & Keys**: warum Keys existieren und was ein Index-Key kaputt macht.
- **Effects**: wofür sie da sind (Synchronisation mit der Außenwelt) und wofür ausdrücklich nicht.
- **Ein eigenes Projekt**, über das du drei Minuten frei reden kannst.

## Nicht-Ziele (bewusst gestrichen)

Diese Themen tauchen nur als **Vokabel** auf — du sollst sagen können, was es ist und wofür, nicht es beherrschen. Das offenzulegen ist Teil des Plans:

Redux/Zustand · Next.js, SSR, Server Components · TypeScript *(im Übungsprojekt dann doch verwendet)* · Testing (Vitest/RTL) · CSS-Frameworks · React Compiler · Performance-Optimierung (`memo`, `useMemo`, `useCallback` — nur Grundidee) · Animationen · Accessibility-Tiefe

> [!tip] Vokabel-Karte
> [[3. Effects, Daten und Architektur - Vertiefung#7. Einordnung: was es sonst noch gibt|Block 3, Abschnitt 7]] enthält genau diese Begriffe in je zwei Sätzen. Damit kannst du auf „Haben Sie schon mit Next.js gearbeitet?" sauber antworten: *was* es ist, *warum* man es nimmt, und dass du es nicht benutzt hast.

---

## Inhalte und Stand

| Bereich | Notiz | Stand |
|---|---|---|
| Quellen | [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste]] | Blöcke 0–3 gelesen |
| Block 0 | [[0. JavaScript für React - Vertiefung]] | ✅ fertig — 8 ✅ · 8 🟡 · 3 ❌ · 2 ⏭️ |
| Block 1 | [[1. React-Grundlagen - Vertiefung]] | ✅ fertig — 11 ✅ · 11 🟡 · 3 ❌ · 3 ⏭️ |
| Block 2 | [[2. State und Interaktivität - Vertiefung]] | ✅ fertig — 13 ✅ · 5 🟡 · 3 ❌ · 3 ⏭️ |
| Block 3 | [[3. Effects, Daten und Architektur - Vertiefung]] | 🔵 gelesen, die acht ⭐-Kontrollfragen offen |
| Generalprobe | [[Interview-Fragenkatalog]] | offen |
| Übungsprojekt | [[Umsetzungsstand (Pricing-Dashboard)]] → [[Code-Review-Aufgaben (Pricing-Dashboard)\|Leseaufgaben]] · [[Lückenschluss (Pricing-Dashboard)\|Lückenschluss]] · [[Frontend-Projektaufbau erklärt]] | ✅ Phasen 0–7 gebaut, Leseaufgaben 1–8 bearbeitet · läuft weiter als Lernprojekt, [[Umsetzungsstand (Pricing-Dashboard)#Roadmap (ab 20.10.2026)\|Roadmap]] bis Phase 15 |

> [!tip] Block 3 lohnt sich jetzt doppelt
> Effects, Custom Hooks, Regeln der Hooks und Cleanup hast du im Übungsprojekt an eigenem Code gesehen (Leseaufgaben 3, 5, 6, 7). Die ⭐-Fragen in [[3. Effects, Daten und Architektur - Vertiefung|Block 3]] sollten sich damit deutlich leichter beantworten lassen als im September.

## Arbeitsmodus

| | |
|---|---|
| **Lesen & Vertiefung** | Nick liest und beantwortet die Kontrollfragen aus dem Kopf; Claudian reviewt und schreibt den Abschnitt aus. Anki abgewählt. |
| **Übungsprojekt** (seit 30.09.) | **Claudian baut** Phase für Phase und committet, **Nick reviewt** über Leseaufgaben und trifft die Entscheidungen; Lücken schließt Claudian im Lückenschluss. Details: [[Umsetzungsstand (Pricing-Dashboard)#Arbeitsmodus (seit 30.09.2026)]] |

Der ursprüngliche Tagesplan für 21.–25.09. mit dem damaligen Arbeitsmodus liegt im Archiv: [[React Crashkurs - Tagesplan 21.–25.09.2026]].
