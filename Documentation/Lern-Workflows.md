---
tags:
  - documentation
---

# Lern-Workflows

Kurzfassung aller Lern-Abläufe, die im Vault definiert sind. Die ausführlichen Regeln stehen in den verlinkten Quellen; diese Notiz ist nur der Spickzettel.

## Inhalt

- [[#1. Traineeship-Kapitel — klassisch (Kapitel 1)]]
- [[#2. Traineeship-Kapitel — Nick produziert (ab Kapitel 2)]]
- [[#3. Übungsaufgabe — klassisch (Kapitel 1)]]
- [[#4. Übungsaufgabe — Hilfe nur auf Nachfrage (ab Kapitel 2)]]
- [[#5. Anki-Karteikarten anlegen]]
- [[#6. Kapitelabschluss]]
- [[#7. Interview-Vorbereitung]]
- [[#8. Übungsprojekt — Claudian baut, Nick reviewt]]
- [[#9. Häppchen-Leseliste mit Quiz (z. B. Git)]]

---

## 1. Traineeship-Kapitel — klassisch (Kapitel 1)

*Quelle: [[CLAUDE.md]] · Vorbild: [[1.3 Testing mit PyTest - Vertiefung]]*

1. Nick sagt: „Lass uns Kapitel X.Y durcharbeiten."
2. Claudian liest den Leitfaden (der bleibt unverändert).
3. Claudian erklärt jedes Stichwort ausführlich in der `Vertiefung`-Notiz: Warum, Konzept, Beispiele, Bezug zu früheren Kapiteln.
4. Claudian beantwortet dort die Kontrollfragen des Leitfadens.
5. Am Ende jedes Abschnitts stehen Self-Check-Fragen.
6. Parallel entstehen pro Stichwort die Anki-Karten (siehe [[#5. Anki-Karteikarten anlegen|Workflow 5]]).

## 2. Traineeship-Kapitel — Nick produziert (ab Kapitel 2)

*Quelle: [[Documentation/Traineeship/Skalierbarkeit & ETL/CLAUDE.md|Bereichs-CLAUDE.md]] · Vorbild Leseliste: [[Documentation/Traineeship/Skalierbarkeit & ETL/2.3 Advanced Python/Leseliste|2.3 Leseliste]]*

Pro Stichwort:

1. **Claudian** schreibt die Leseliste: 2–4 Core-Quellen, genau eingegrenzt, mit Lesezeit (max. ~60–90 Min.) und einem Leseauftrag pro Quelle. Dazu optionale Vertiefungs-Quellen.
2. **Claudian** schreibt gleichzeitig die Kontrollfragen in die `Vertiefung`-Notiz, also *bevor* Nick liest.
3. **Nick** liest und kopiert Wichtiges roh in die Vertiefung, am besten mit einem eigenen Satz dazu. Offene Fragen kommen nach `Q&A/Fragen.md`.
4. **Nick** beantwortet die Kontrollfragen **aus dem Kopf**, ohne nachzuschlagen.
5. **Claudian** reviewt die Antworten: was saß, was fehlte, was falsch war.
6. **Claudian** schreibt daraus den fertigen Vertiefungs-Abschnitt, vollständig, mit Review-Tabelle (✅/🟡/❌/⏭️) und markierten Korrekturen.
7. **Claudian** legt die Anki-Karten an. Was Nick selbst herauskopiert hat, kommt zuerst.
8. **Claudian** beantwortet zum Schluss die gesammelten Fragen aus `Q&A/Fragen.md`.

## 3. Übungsaufgabe — klassisch (Kapitel 1)

*Quelle: [[CLAUDE.md]] · Vorbild: [[Test-Suite Plan & Syntax-Tutorial]]*

1. Projektordner außerhalb des Vaults anlegen: `~/HaegerConsulting/Projects/Traineeship/...`
2. Im Vault `Übungsaufgabe/Umsetzungsstand (<Thema>).md` anlegen: Ziele, Stand, offene Punkte, Lektionen.
3. Bei viel neuer Syntax: einen `Plan & Syntax-Tutorial` mit Kästchen anlegen, von einfach nach schwer, eine neue Technik pro Schritt.
4. **Nick schreibt den Code**, Claudian reviewt, erklärt und leitet an.
5. Claudian schreibt Code nur, wenn Nick es ausdrücklich sagt.

## 4. Übungsaufgabe — Hilfe nur auf Nachfrage (ab Kapitel 2)

*Quelle: [[Documentation/Traineeship/Skalierbarkeit & ETL/CLAUDE.md|Bereichs-CLAUDE.md]]*

1. **Claudian** schreibt einmal einen groben Leitfaden: Zielbild, Reihenfolge der Schritte, woran man merkt, dass ein Schritt fertig ist. Kein Code.
2. **Nick** baut selbst und führt den Umsetzungsstand.
3. Claudian meldet sich **nicht von selbst**.
4. Fragt Nick, hilft Claudian stufenweise:
   1. Denkanstoß (Standard)
   2. Konzept erklären
   3. Syntax an einem fremden Beispiel zeigen
   4. Konkrete Lösung, nur auf ausdrückliche Ansage
5. Bei einem Fehler fragt Claudian zuerst: „Was sagt dir die Fehlermeldung?"
6. Code-Review nur auf Anfrage, dann gründlich.
7. Ausnahme: Bei Gefahr von Datenverlust warnt Claudian sofort.

## 5. Anki-Karteikarten anlegen

*Quelle: [[CLAUDE.md]] · Details: [[Documentation/Services/Obsidian Anki Sync.md|Obsidian Anki Sync]]*

1. In `Anki Karteikarten.md` das `deck` setzen: `Traineeship::<Bereich>::<Unterkapitel>`.
2. Nur Basic-Karten: Frage endet mit `::`, Antwort auf **genau einer Zeile** (Umbrüche mit `<br>`).
3. Neue Blöcke bekommen **kein** `oid`-Attribut.
4. Richtwert: ~10 Karten pro Stichwort, nicht künstlich auffüllen.
5. Anki starten und Sync ausführen.
6. Auf `Failed Created Blocks: 0` achten.
7. Vor dem Verschieben oder Umbenennen einer Karteikarten-Datei: erst Anki-Export mit Lernfortschritt machen.

## 6. Kapitelabschluss

*Vorbilder: [[Kapitelabschluss - Q&A-Ernte und blinde Flecken]] · [[Kapitelabschluss 2.1 - Stand und offene Punkte]]*

1. Festhalten, was fertig ist (Vertiefung, Karten, Leseliste, Übung).
2. Festhalten, was bewusst offen bleibt.
3. Die `Q&A/`-Ordner auswerten: was steht noch nirgends?
4. Bekannte Wissenslücken und blinde Flecken benennen.
5. Muster aus den Selbsttests zusammenfassen.
6. Empfehlungen fürs nächste Kapitel ableiten.

## 7. Interview-Vorbereitung

*Quelle: [[Interview Vorbereitung]]*

**Normale Variante** (wie Workflow 2, nur schneller):

1. Claudian schreibt eine Leseliste mit Zeitbudget.
2. Claudian legt die Kontrollfragen in der Vertiefung an, bevor gelesen wird.
3. Nick liest, exzerpiert und antwortet aus dem Kopf.
4. Claudian reviewt und schreibt den fertigen Abschnitt.
5. Offene Fragen landen in `Q&A/Fragen.md`.
6. Anki ist optional.

**Schnellvariante** (ab Kubernetes):

1. Claudian schreibt einen Überblickstext mit Kontrollfragen.
2. Claudian schreibt eine knappe Leseliste (Minimalpfad).
3. Claudian stellt eine kleine Übung.
4. Nick liest und beantwortet die Kontrollfragen im Überblick.
5. Claudian reviewt.

## 8. Übungsprojekt — Claudian baut, Nick reviewt

*Quelle: [[Umsetzungsstand (Pricing-Dashboard)]] · Vorbild: [[Lückenschluss (Pricing-Dashboard)]]*

1. Claudian baut eine Phase und committet.
2. Claudian schreibt dazu eine Leseaufgabe mit Fragen zum Code.
3. Nick liest den Code wie ein Reviewer und beantwortet die Fragen kurz.
4. Nick trifft die offenen Entscheidungen, bevor die nächste Phase gebaut wird.
5. Claudian bewertet die Antworten (✅/🟡/❌/⏭️).
6. Claudian erklärt die Lücken ausführlich im `Lückenschluss`.

## 9. Häppchen-Leseliste mit Quiz (z. B. Git)

*Quelle: [[Git Leseliste]] · Vorbild: [[Quiz 1 - Commits, Objekte & Rebase]]*

1. Einen Block (~30 Min.) aus der Leseliste wählen.
2. Den Core-Teil lesen.
3. Den Teil *Ausprobieren* in einem Wegwerf-Repo unter `/tmp` durchspielen.
4. Das Quiz **aus dem Kopf** beantworten.
5. Claudian sagen: „Quiz N reviewen."
6. Auf Wunsch macht Claudian aus den Lücken Anki-Karten.

---

## Verwandt

- [[CLAUDE.md]] — Vault-Regeln
- [[Documentation/Traineeship/Skalierbarkeit & ETL/CLAUDE.md|Bereichs-CLAUDE.md Skalierbarkeit & ETL]]
- [[Interview Vorbereitung]]
- [[Documentation/Haeger Consulting.md|Haeger Consulting]]
