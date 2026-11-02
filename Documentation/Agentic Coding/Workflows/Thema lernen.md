---
tags:
  - workflow
erstellt: 2026-10-27
---

# Thema lernen

Dieser Workflow beschreibt, wie ich Themen lerne. Er läuft über mehrere Sessions. Den Stand hältst du in der Lernnotiz fest.

## Lernnotiz

Lege die Lernnotiz nach Schritt 1 in dem passenden Unterordner von `Documentation` an. Gibt es keinen passenden Unterordner, schlage einen neuen vor. Die Notiz hat diesen Aufbau:

```yaml
---
tags:
erstellt:
lernphase: discovery
---
```

Der Wert von `lernphase` ist immer der aktuelle Schritt: `discovery`, `recherche`, `lesen`, `feedback`, `aufbereitung` oder `fertig`.

## Unterbrechungen

- Aktualisiere am Ende jedes Schritts `lernphase` und den Abschnitt `Stand`. `Stand` enthält die Antworten aus der Discovery, den aktuellen Schritt und offene Punkte.
- Lies beim Fortsetzen zuerst `lernphase` und `Stand` und mache beim aktuellen Schritt weiter.

## Schritte

1. **Discovery:** Frage nach:
	- Anlass: konkretes Projekt oder allgemeines Interesse
	- Vorwissen
	- Zeitbudget in Gesamtstunden
	- Ziel: verstehen, anwenden oder mitreden können
	- Bevorzugte Medien
2. **Recherche:** Recherchiere das Thema breit, ohne einzelne Perspektiven überzugewichten. Bevorzuge bewährte Quellen, die nicht veraltet sind.
	- Gehe davon aus, dass ich Zugriff auf alle Bücher habe.
	- Nimm Videos, Kurse und Online-Dokumentation gleichberechtigt auf, wenn sie ohne Account und ohne Paywall zugänglich sind.
	- Nimm nur Quellen auf, deren Existenz du per Recherche bestätigt hast.
3. **Leseliste:** Erstelle die Leseliste.
	- Bis 4 h Zeitbudget: Overview, der die relevanten Informationen für den Anlass vermittelt.
	- Über 4 h: angemessene Vertiefung.
	- Jeder Eintrag enthält: Quelle mit Link oder ISBN, die konkreten Kapitel oder Abschnitte, die geschätzte Lesezeit und eine kurze Begründung.
	- Die Summe der Lesezeiten liegt im Zeitbudget.
4. **Lernkontrolle:** Erstelle zehn bis 25 Fragen im Abschnitt `Lernkontrolle`.
	- Die meisten Fragen prüfen Wissen, also ob die Leseliste konzentriert gelesen wurde.
	- Bis zu fünf Fragen prüfen Verständnis.
	- Jede Frage ist mit der Leseliste beantwortbar und nennt ihre Quelle.
	- Lass unter jeder Frage Platz für meine Antwort.
5. **Lesen:** Ich lese, sammle Fragen und Auszüge in der Notiz und beantworte die Lernkontrolle aus dem Kopf. Warte, bis ich melde, dass ich fertig bin.
6. **Feedback:** Bewerte jede Antwort als richtig oder falsch und erkläre in einem Satz, warum. Bleib kurz, sachlich und auf den Punkt. Ändere meine Antworten nicht.
7. **Aufbereitung:**
	- Fasse die Quellen über `Stand` zusammen, sodass die Notiz zum Wiederholen und Nachschlagen taugt.
	- Beantworte die offenen Fragen aus `Fragen und Auszüge` mit Quellenangabe.
	- Verlinke passende bestehende Notizen.
	- Die Lernkontrolle bleibt unten in der Datei.
	- Setze `lernphase: fertig`.
