---
tags:
  - documentation
erstellt: 2026-11-05
---
# Boulder App - Routenerkennung

Teil der Discovery für die [[Boulder App]]. Idee: Man lädt ein Foto einer Wand hoch, und ein Vision-Modell erkennt die Routen automatisch. Die Annahme dahinter ist, dass die Griffe einer Route meist dieselbe Farbe haben und sich ähnlich sehen. Das Feature ist das größte technische Risiko des Projekts. Deshalb soll es früh und unabhängig vom Rest als kleines Experiment getestet werden, bevor es in die Planung der [[Boulder App - Features|Features]] eingeht.

## Discovery-Fragen

### Ziel des Features

- Was genau soll erkannt werden: nur die Griffe, die Zugehörigkeit der Griffe zu einer Route, Start- und Topgriff oder auch der Schwierigkeitsgrad?
- Reicht ein Vorschlag, den man danach von Hand korrigiert, oder muss es vollautomatisch sein?
- Wie markiert deine Halle Routen? Nur über die Griff-Farbe, oder zusätzlich über Tape, Schilder oder Startmarkierungen?

### Schwierigkeiten zum Prüfen

- Mehrere Routen derselben Farbe an einer Wand
- Volumen und Strukturen, die keine Griffe sind
- Beleuchtung, Schatten und Kreidereste, die Farben verfälschen
- Perspektive, wenn das Foto nicht genau frontal ist

### Ansatz

- Willst du dabei Computer Vision lernen und ein eigenes Modell trainieren, oder ein fertiges Modell über eine API nutzen?
- Wärst du bereit, Fotos zu labeln, um Trainingsdaten zu erzeugen?
- Soll die Erkennung auf dem Server, auf dem Handy oder über einen externen Dienst laufen? Das beeinflusst [[Boulder App - Hosting und Kosten|Hosting und Kosten]].

## Experiment

Noch offen. Erster Schritt: einige Fotos deiner Halle sammeln und verschiedene Ansätze daran ausprobieren.

## Ergebnis

Noch offen.
