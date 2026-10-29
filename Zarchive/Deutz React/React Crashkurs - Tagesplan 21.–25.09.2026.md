---
tags:
  - documentation
---

# React Crashkurs — Tagesplan 21.–25.09.2026 (archiviert)

> [!info] Archiviert am 20.10.2026
> Der ursprüngliche Fünf-Tage-Plan vor dem Deutz-Gespräch, samt altem Arbeitsmodus („Nick schreibt den Projektcode selbst"). Ab dem 30.09. ist das Projekt im Modus „Claudian baut, Nick reviewt" weitergelaufen. Aktueller Stand: [[React Crashkurs - Übersicht|Übersicht]] · [[Umsetzungsstand (Pricing-Dashboard)]].

## Tagesplan

### Tag 1 — Mo 21.09. · JavaScript-Boden (~3–4 h, angebrochener Tag)

Ohne JS-Basis wird React zum Auswendiglernen. Dieser Tag ist die Investition, die alle folgenden billiger macht.

- [ ] **Werkzeuge aufsetzen** (~30 min) — Node + npm prüfen, Vite-Projekt anlegen, `npm run dev` läuft im Browser. Siehe [[Pricing-Dashboard Plan & Syntax-Tutorial#Schritt 0 — Werkzeuge und ein leeres Projekt|Schritt 0]]
- [ ] **Leseliste Block 0 — Core** (~2 h 30) → [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste#Block 0 — JavaScript für React]]
- [ ] **Exzerpieren** während des Lesens in [[0. JavaScript für React - Vertiefung]]
- [ ] **Kontrollfragen Block 0** dort aus dem Kopf beantworten (~30 min)
- [ ] Claudian: Review + Abschnitt ausschreiben

### Tag 2 — Di 22.09. · React-Grundlagen: die UI beschreiben (~5 h)

- [ ] **Leseliste Block 1 — Core** (~2 h 40) → [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste#Block 1 — Die UI beschreiben]]
- [ ] **Exzerpieren + Kontrollfragen** in [[1. React-Grundlagen - Vertiefung]] (~40 min)
- [ ] **Projekt Schritte 1–4** (~1 h 30): React-Projekt gegen die Pricing-API, erste eigene Komponente, Props, Produktliste aus echten Daten
- [ ] Claudian: Review Antworten + Code

### Tag 3 — Mi 23.09. · State & Interaktivität (~5 h)

Der inhaltlich dichteste Tag — hier liegt das, wonach im Gespräch am ehesten gefragt wird.

- [ ] **Leseliste Block 2 — Core** (~2 h 45) → [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste#Block 2 — State und Interaktivität]]
- [ ] **Exzerpieren + Kontrollfragen** in [[2. State und Interaktivität - Vertiefung]] (~40 min)
- [ ] **Projekt Schritte 5–8** (~1 h 30): Event-Handler, kontrolliertes Suchfeld, Filterung, State nach oben ziehen, Detailansicht
- [ ] Claudian: Review

### Tag 4 — Do 24.09. · Effects, Daten und Architektur (~5 h)

- [ ] **Leseliste Block 3 — Core** (~2 h 20) → [[Documentation/Interview Vorbereitung/Deutz/React/Grundlagen/Leseliste#Block 3 — Effects, Daten und Architektur]]
- [ ] **Exzerpieren + Kontrollfragen** in [[3. Effects, Daten und Architektur - Vertiefung]] (~40 min)
- [ ] **Projekt Schritte 9–12** (~1 h 30): eigener Hook `useApi`, Lade-/Fehlerzustände, Preislauf starten und per Polling verfolgen, API-Key über Context
- [ ] Claudian: Review

### Tag 5 — Fr 25.09. · Konsolidieren & Generalprobe (~5 h)

Kein neuer Stoff. Nur Abrufen, Schließen, Erzählen.

- [ ] **Fragenkatalog laut durchsprechen**, aus dem Kopf (~2 h) → [[Interview-Fragenkatalog]]
- [ ] **Claudian bewertet** Antwort für Antwort; Schwachstellen markieren
- [ ] **Lücken nacharbeiten** — gezielt in den Vertiefungsnotizen (~1 h)
- [ ] **Projekt aufräumen** und die **3-Minuten-Story** dazu schreiben (~1 h) → [[Umsetzungsstand (Pricing-Dashboard)#Die 3-Minuten-Story]]
- [ ] **Vokabel- und Ehrlichkeitsblock** (~30 min): Nicht-Ziele in je zwei Sätzen, plus die Sätze für „das kann ich noch nicht"
- [ ] Optional: Karteikarten für die 20 wichtigsten Begriffe — nur wenn Zeit übrig ist

## Arbeitsmodus (damals)

Gilt wie im Traineeship, mit zwei Abweichungen (deine Vorgabe):

| Punkt | Regelung |
|---|---|
| Lesen & Exzerpieren | **Nick** — roh, unaufgeräumt, während des Lesens, mit den Kontrollfragen im Blick |
| Kontrollfragen | **Claudian** stellt sie vorab in die Vertiefung; **Nick** beantwortet sie aus dem Kopf, ohne Nachschlagen |
| Review | **Claudian** — pro Antwort: was saß, was fehlte, was war falsch. Lücken sind das Diagnosesignal, kein Mangel |
| Vertiefungstext | **Claudian**, nach dem Review; deckt das ganze Stichwort ab, nicht nur die Exzerpte |
| Projektcode | **Nick schreibt selbst.** Claudian gibt Muster und reviewt — außer bei ausdrücklicher Ansage |
| **Anki** | **abgewählt** — bei fünf Tagen bis zum Termin bringt Spaced Repetition nichts mehr. Optionaler Restposten an Tag 5 |
| **Eskalation** | Standard ist ein Denkanstoß, keine Lösung. Bei Zeitdruck darfst du jederzeit „gib mir die Lösung" sagen — anders als im Traineeship ist hier Tempo ein legitimes Ziel |

## Fortschritt (Stand Ende September)

| Tag | Thema | Status |
|---|---|---|
| 1 | JavaScript für React | ✅ fertig — 8 ✅ · 8 🟡 · 3 ❌ · 2 ⏭️ |
| 2 | Die UI beschreiben | ✅ fertig — 11 ✅ · 11 🟡 · 3 ❌ · 3 ⏭️ |
| 3 | State und Interaktivität | ✅ fertig — 13 ✅ · 5 🟡 · 3 ❌ · 3 ⏭️ (bester Block) |
| 4 | Effects, Daten und Architektur | 🔵 gelesen, Kontrollfragen offen |
| 5 | Generalprobe | offen |
