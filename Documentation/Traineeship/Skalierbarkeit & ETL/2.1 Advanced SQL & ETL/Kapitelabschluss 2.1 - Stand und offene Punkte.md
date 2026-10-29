---
tags:
  - documentation
---

# Kapitelabschluss 2.1 — Stand und offene Punkte

Abschlussnotiz zu [[2.1 Advanced SQL & ETL - Leitfaden]], analog zu [[Kapitelabschluss - Q&A-Ernte und blinde Flecken|der Notiz aus Kapitel 1]]. Zweck: festhalten, was steht und was bewusst offen bleibt — damit später niemand rät.

## Was fertig ist

| Artefakt | Stand |
|---|---|
| [[2.1 Advanced SQL & ETL - Vertiefung]] | ✅ Alle **9 Stichworte** ausgearbeitet, 9 Hauptabschnitte |
| [[Anki Karteikarten]] | ✅ **112 Karten** über alle Stichworte |
| [[SQL-Werkzeugkasten]] | ✅ Nachschlagenotiz: `EXPLAIN` lesen · `ON CONFLICT`/`RETURNING` · `NOT IN`-NULL-Falle · Kleinzeug · `GROUPING SETS` |
| [[Documentation/Traineeship/Skalierbarkeit & ETL/2.1 Advanced SQL & ETL/Leseliste]] | ✅ Alle Core-Quellen gelesen |
| [[Q&A/Fragen\|Q&A]] | ✅ Beantwortet |
| Übungsaufgabe Schritte 0–6 | ✅ Äquivalenz bewiesen — siehe [[Umsetzungsstand (SQL-Portierung)]] |

## Was offen bleibt

### In der Übungsaufgabe

Details im [[Umsetzungsstand (SQL-Portierung)#Offene Punkte|Umsetzungsstand]]. Kurzfassung:

- **Schritt 7 unvollständig** — eine Messung statt einer Reihe, keine Staffelung über Datenmengen. Der Leitfaden verlangt „diverse Benchmarks". Bewusst so belassen: Die Optimierung war nicht der Zweck der Aufgabe, und die vorhandene Messung plus Plananalyse trägt die inhaltlichen Aussagen bereits.
- **Schritt 8 offen** — Code-Review und Fazit. Das Fragegerüst steht, die Bewertung fehlt.
- Zwei identifizierte, aber ungemessene Optimierungen: Arithmetik hinter den Filter · `double precision` statt `numeric`.

### Übung

Die Übung fand an zwei Stellen statt: in der **Übungsaufgabe** — eine mehrstufige SQL-Pipeline von Grund auf, über mehrere Tage, mit Review nach jedem Schritt — und über die **PGExercises-Sektionen** aus [[Documentation/Traineeship/Skalierbarkeit & ETL/2.1 Advanced SQL & ETL/Leseliste#Nachtrag B — Übungsaufgaben zum Festigen|Nachtrag B]], die zu großen Teilen bearbeitet wurden. Der aktuelle Stand ist in der Leseliste abgehakt.

Damit ist der Praxisteil des Kapitels abgedeckt.

### Nicht gelesen

- [ ] **Kimball vs. Inmon** ([[Documentation/Traineeship/Skalierbarkeit & ETL/2.1 Advanced SQL & ETL/Leseliste#Nachtrag A — Kimball vs. Inmon (30–60 Min.)|Nachtrag A]]) — die Gegenposition zur dimensionalen Modellierung fehlt
- [ ] Optionale Vertiefungsquellen der Blöcke A–D

### Bekannte Wissenslücken

Aus dem Selbsttest zu Block D: Die **dimensionale Modellierung** (Fakten, Dimensionen, Grain, Star vs. Snowflake) blieb im Selbsttest weitgehend unbeantwortet. Der Stoff ist in [[2.1 Advanced SQL & ETL - Vertiefung#9.7 Dimensionale Modellierung: Fakten, Dimensionen, Grain|Abschnitt 9.7–9.10]] vollständig ausgearbeitet und in fünf Karten erfasst — **gelesen ist aber nicht gelernt.** Star/Snowflake stand schon in Kapitel 1 als blinder Fleck und ist jetzt zum zweiten Mal durchgerutscht. Für Data Engineering zentral.

## Muster über das Kapitel

Vier Selbsttests, ein durchgehender Befund:

| Block | Ergebnis | Muster |
|---|---|---|
| 1 — Pipelines/ETL | 3 · 3 · 7 | Antworten standen in den eigenen Exzerpten → Enkodierungsproblem |
| 2 — SQL | 5 · 5 · 2 | Konzepte gut, **getippte Queries übersprungen** |
| A — Paradigmen | 5 · 8 · 2 | Zweite Hälfte zweiteiliger Fragen fehlte · **Querverweise nicht hergestellt** |
| B — Testing | 5 · 9 · 3 | Ermüdung (zweiter Block am selben Tag) |
| C — Datentypen | 8 · 9 · 2 | Beste Quote; Schwachpunkt **Syntax**, nicht Verständnis |
| D2 — Materialisierung | 6 · 7 · 4 | Postgres stark, **Modellierung leer** |

**Zwei wirksame Maßnahmen:** Die `(a)`/`(b)`-Nummerierung ab Block B hat das „zweite Hälfte fehlt"-Muster beseitigt. Rückbezugsfragen haben fehlende Verknüpfungen sichtbar gemacht.

**Zur Einordnung der Zahlen:** Wo Antworten knapper ausfielen, war das überwiegend **Priorisierung unter Zeitdruck**, nicht nachlassendes Verständnis — in denselben Blöcken standen fachlich starke Antworten daneben. Ein festes Zeitbudget ist eine Rahmenbedingung, keine Abweichung: Wenn zwei Blöcke an einem Tag nötig sind, ist die Ausführlichkeit der Antworten das, was nachgibt. Das ist die richtige Stellschraube.

**Die Lernkurve bei der Syntax** ist im Verlauf sichtbar: Die Fehler häufen sich in Stichwort 2 und Block C; das Abschlussskript der Übungsaufgabe ist korrekt strukturiert, mit mehrstufigen CTEs, Klassifizierungsspalte und sauberer Transaktionsklammer. Die anfängliche Lücke zwischen Verstehen und Formulieren hat sich über die Übungsaufgabe geschlossen.

## Empfehlung fürs nächste Kapitel

Der Arbeitsmodus aus [[Skalierbarkeit & ETL/CLAUDE.md|der Bereichs-CLAUDE.md]] hat funktioniert und bleibt. Zwei Kalibrierungen:

- **Zeitbudget ist Vorgabe, nicht Verhandlungsmasse.** Claudians Schätzungen lagen zweimal deutlich zu niedrig (Stichwort 2, Block D). Leselisten sind am Budget auszurichten, nicht umgekehrt — und wenn zwei Blöcke an einem Tag nötig sind, wird das Fragenset gekürzt statt der Antworten.
- **Die Übungsaufgabe ist die Übung.** Zusätzliche Aufgabenlisten sind Auffrischung, kein Pflichtteil. Der Lerneffekt kam aus dem schrittweisen Bauen mit Review.
