---
tags:
  - workflow
erstellt: 2026-10-27
---

# Codebase verstehen

Dieser Workflow beschreibt, wie ich mich in eine Codebase einarbeite. Anlass ist entweder die Übernahme eines Projekts oder ein Code-Review. Er läuft über mehrere Sessions.

## Regeln

- Lies den Code nur. Ändere ihn nie.
- Recherchiere im Web nur Technologien und Libraries, nie den Code selbst.
- Lies eine `AGENTS.md` im Repo als Informationsquelle. Für deine Arbeit im Vault gilt weiterhin die `AGENTS.md` des Vaults.

## Notizen

Schreibe die Projektdoku in `Documentation/<Projekt>/` nach [[Code dokumentieren]]. Lege zusätzlich die Notiz `<Projekt> - Einarbeitung` mit den Abschnitten `Lesereihenfolge`, `Fragen und Auszüge` und `Lernkontrolle` an.

Die Überblick-Notiz bekommt zusätzlich die Property `einarbeitungsphase` und den Abschnitt `Stand`. Der Wert von `einarbeitungsphase` ist immer der aktuelle Schritt: `discovery`, `scan`, `einschätzung`, `analyse`, `lesen`, `feedback`, `aufbereitung` oder `fertig`.

## Unterbrechungen

- Aktualisiere am Ende jedes Schritts `einarbeitungsphase` und den Abschnitt `Stand` der Überblick-Notiz. `Stand` enthält die Antworten aus der Discovery, den aktuellen Schritt und offene Punkte.
- Lies beim Fortsetzen zuerst `einarbeitungsphase` und `Stand` und mache beim aktuellen Schritt weiter.

## Schritte

1. **Discovery:** Frage nach:
	- Anlass: Übernahme oder Code-Review. Bei einem Review zusätzlich: welche Änderungen oder welcher Bereich.
	- Pfad zum Repo
	- Zeitbudget in Gesamtstunden
2. **Scan:** Verschaffe dir einen Überblick über README, `AGENTS.md`, Verzeichnisstruktur, Build- und Dependency-Dateien und die Commit-Historie. Die Historie zeigt, welche Bereiche aktiv sind.
3. **Einschätzung:** Lege mir die gefundenen Technologien, Libraries und Hauptfunktionalitäten vor. Ich markiere sie als bekannt, teilweise bekannt oder unbekannt. Schlage für relevante unbekannte Technologien den Workflow [[Thema lernen]] vor.
4. **Analyse:** Schreibe die Projektdoku.
	- Bis 4 h Zeitbudget: Overview. Über 4 h: angemessene Vertiefung.
	- Bei einem Review: Konzentriere dich auf den betroffenen Bereich und sein Umfeld.
	- Lege den Schwerpunkt auf das, was ich als unbekannt markiert habe.
5. **Lesereihenfolge und Lernkontrolle:** Erstelle in der Einarbeitungs-Notiz:
	- Eine Lesereihenfolge entlang konkreter Funktionalitäten. Jeder Eintrag enthält die Dateien oder Funktionen, die geschätzte Lesezeit und eine kurze Begründung. Die Summe liegt im Zeitbudget.
	- Eine Lernkontrolle mit zehn bis 25 Fragen. Die meisten prüfen Wissen, also ob die Lesereihenfolge konzentriert gelesen wurde. Bis zu fünf prüfen Verständnis. Jede Frage ist mit der Lesereihenfolge beantwortbar und nennt die betroffene Datei. Lass unter jeder Frage Platz für meine Antwort.
6. **Lesen:** Ich lese den Code, baue und starte das Projekt, sammle Fragen und Auszüge und beantworte die Lernkontrolle. Warte, bis ich melde, dass ich fertig bin.
7. **Feedback:** Bewerte jede Antwort als richtig oder falsch und erkläre in einem Satz, warum. Bleib kurz, sachlich und auf den Punkt. Ändere meine Antworten nicht.
8. **Aufbereitung:**
	- Beantworte die offenen Fragen aus `Fragen und Auszüge` mit Belegen.
	- Ergänze die Projektdoku um neue Erkenntnisse.
	- Verlinke passende bestehende Notizen.
	- Setze `einarbeitungsphase: fertig`.
