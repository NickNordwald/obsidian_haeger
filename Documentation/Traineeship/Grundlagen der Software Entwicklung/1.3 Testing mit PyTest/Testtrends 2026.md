---
tags:
  - documentation
---
Ergänzung zu [[1.3 Testing mit PyTest - Vertiefung|1]], speziell zum Abschnitt [[1.3 Testing mit PyTest - Vertiefung#Mike Cohns Testpyramide|Mike Cohns Testpyramide]]. Ordnet ein, wie die Branche 2026 tatsächlich über Unit-Tests denkt — der Leitfaden fokussiert bewusst auf Modultests, diese Notiz liefert den Kontext drumherum.

# Testtrends 2026 — Ist der Trend weg von Unit-Tests?

> [!abstract] Kurzantwort
> **Nein — nicht weg von Unit-Tests, aber weg von deren Alleinherrschaft.** Der Backlash richtet sich nicht gegen den Modultest als solchen, sondern gegen *mock-lastige, an die Implementierung gekoppelte* Unit-Tests und gegen Coverage als Selbstzweck. Die klassische 70/20/10-Verteilung ist 2026 weiterhin die Mehrheitsmeinung, steht aber unter deutlich mehr Rechtfertigungsdruck als noch vor zehn Jahren.

# Inhaltsverzeichnis

- [[#Was sich verschoben hat]]
- [[#Was unverändert für Unit-Tests spricht]]
- [[#Der KI-Faktor — zwei gegenläufige Kräfte]]
- [[#Data Engineering: der eigentliche Sonderfall]]
- [[#Fazit für die eigene Praxis]]
- [[#Quellen]]

---

# Was sich verschoben hat

## 1. Von der Pyramide zur „Testing Trophy"

Das im Leitfaden erwähnte Modell von Kent C. Dodds (breite Integrationsschicht statt breiter Unit-Basis) ist in Teilen der Branche vom Gegenentwurf zum Default geworden. Der Grund ist rein ökonomisch: **Integrationstests sind billig geworden.**

- **Testcontainers** — echte Datenbank, echter Broker, echter Cache im Docker-Container, in Sekunden hochgefahren
- **In-Memory-Backends** (SQLite, DuckDB, fakeredis) für schnelle, realistische Persistenzschicht-Tests
- **Schnelle Runner** und Parallelisierung (`pytest-xdist`, Vitest) plus CI-Runner mit vielen Kernen

Das klassische Argument *„Unit-Tests, weil alles andere zu langsam und zu unzuverlässig ist"* hat damit an Kraft verloren. Wenn ein Integrationstest 200 ms statt 20 s braucht, verschiebt sich der optimale Punkt auf der Kosten-/Nutzenkurve nach oben.

## 2. Kritik am Over-Mocking

Der schärfste Vorwurf gegen Unit-Tests: Ein Test mit fünf Mocks testet nicht mehr das *Verhalten*, sondern die *Struktur* des Codes. Folgen:

- Jedes Refactoring bricht Tests, obwohl sich das Verhalten nicht geändert hat → Tests werden zur Bremse statt zum Sicherheitsnetz
- Der Test kann grün sein, während die echte Integration bricht (Mock und Realität driften auseinander)
- Es entsteht trügerische Confidence bei hoher Coverage-Zahl

Die Standardantwort der Community lautet **„Test behavior, not implementation."** Siehe dazu bereits [[1.3 Testing mit PyTest - Vertiefung#Wann man *nicht* mocken sollte (Over-Mocking)|Wann man nicht mocken sollte]] — das ist genau dieser Punkt.

## 3. Die „Eistüte" wird aus Kostengründen korrigiert

Interessanter Nebenbefund 2026: Viele Teams hatten jahrelang über die Pyramide *geredet*, aber faktisch eine [[1.3 Testing mit PyTest - Vertiefung#Anti-Muster: Woran man erkennt, dass etwas schiefläuft|Eistüte]] gebaut. Was die Korrektur jetzt erzwingt, ist nicht Einsicht, sondern die **CI-Rechnung**: Minutenpreise in der Cloud-CI und die messbaren Kosten flaky Tests machen die langsame E2E-Spitze schlicht zu teuer. Unit-Tests gewinnen hier also gerade wieder an Boden — aus Kostengründen.

## 4. TDD-Purismus ist vorbei

Seit DHHs „TDD is dead" (2014) ist Test-First keine unhinterfragte Doktrin mehr. Testen selbst ist unstrittig; die Frage ist nur *wann* und *auf welcher Ebene* — nicht *ob*.

---

# Was unverändert für Unit-Tests spricht

Unangefochten bleibt der Modultest überall dort, wo **reine Logik ohne I/O** steckt:

| Anwendungsfall | Warum hier der Unit-Test gewinnt |
|---|---|
| Algorithmen, Parser, State Machines | Viele Eingabekombinationen, kein sinnvoller Integrations-Ersatz |
| Preis-, Steuer-, Zinsberechnungen | Korrektheit ist geschäftskritisch, Randfälle zahlreich |
| Fehler- und Randpfade | Über Integrationstests oft gar nicht gezielt erreichbar |
| Validierungs- und Transformationslogik | Schnelles Feedback direkt beim Schreiben |

Zusätzlich **wächst** eine Unit-Test-Spielart klar: **Property-based Testing** (in Python `hypothesis`, in JS `fast-check`). Statt einzelner Beispiele beschreibt man eine *Eigenschaft*, die immer gelten muss („Sortieren ändert die Länge nicht"), und das Framework sucht selbst Gegenbeispiele. Das ist die konsequente Weiterentwicklung der [[1.3 Testing mit PyTest - Vertiefung#Dieselbe Logik, viele Eingaben: parametrisierte Tests (Kontrollfrage)|parametrisierten Tests]] — ein sinnvoller nächster Lernschritt nach diesem Kapitel.

---

# Der KI-Faktor — zwei gegenläufige Kräfte

> [!warning] Der spannendste Punkt 2026
> KI wirkt gleichzeitig **gegen** und **für** Unit-Tests.

**Gegen die Unit-Ebene:** KI schreibt heute ganze Features statt einzelner Funktionen. Wenn der generierte Codeblock größer ist als die „Unit", passt die Testgranularität nicht mehr zur Entstehungsgranularität — Integrations- und Contract-Tests treffen den generierten Umfang besser. Dazu kommt die Gefahr, dass KI massenhaft triviale Unit-Tests erzeugt, die nur die Coverage-Metrik füttern (siehe [[1.3 Testing mit PyTest - Vertiefung#Wie viel Coverage macht Sinn? (Kontrollfrage)|Wie viel Coverage macht Sinn?]]).

**Für Tests generell:** Ein Coding-Agent braucht ein **automatisiertes Verifikationssignal**, um überhaupt sinnvoll iterieren zu können. Die Testsuite ist genau dieses Signal — sie ist die Leitplanke, an der sich autonome Änderungen messen lassen. In KI-gestützten Workflows werden Tests dadurch eher *wichtiger*, nicht unwichtiger. Und: Schnelle Unit-Tests sind für einen Agent, der zwanzig Iterationen fährt, deutlich brauchbarer als eine 15-minütige E2E-Suite.

---

# Data Engineering: der eigentliche Sonderfall

Für die berufliche Praxis bei [[Documentation/Haeger Consulting.md|Haeger Consulting]] relevant: Im Data Engineering ist die Verschiebung am deutlichsten — dort testet man überwiegend **Daten**, nicht **Code**.

| Ebene | Werkzeuge | Was geprüft wird |
|---|---|---|
| **Unit-Tests** | `dbt` Unit Tests (seit v1.8), `pytest` | SQL-/Transformationslogik gegen Mock-Inputs, *vor* der Materialisierung |
| **Data Tests** | dbt tests, Great Expectations, Soda | Assertions über den *tatsächlichen* Datenbestand *nach* dem Lauf (not null, unique, referentielle Integrität) |
| **Model Contracts / Data Contracts** | dbt contracts, YAML-Schemas | Schema und Typen zwischen Producer und Consumer — verhindert strukturellen Drift |
| **Pipeline-Integrationstests** | Sample-Datensätze, Staging-Umgebung | Zusammenspiel der Schritte end-to-end |

Bemerkenswert: dbt hat 2024 mit v1.8 *nachträglich* echte Unit-Tests eingeführt — die Bewegung geht in diesem Feld also sogar **hin zu** Unit-Tests, weil die reine Datenprüfung im Nachhinein zu spät kommt. Wer im Data-Umfeld den Eindruck hat, klassische Unit-Tests spielten kaum eine Rolle, liegt für den Status quo richtig — aber das ist ein Zustand im Wandel, keine Endstufe.

---

# Fazit für die eigene Praxis

1. **Der Leitfaden-Fokus auf Modultests ist nicht veraltet.** Die Unit-Ebene ist das Fundament, auf dem alles andere aufbaut — und die Ebene, auf der man Testdesign überhaupt erst lernt.
2. **Die eigentliche Lektion ist nicht die Zahl 70 %, sondern die Frage: Was gibt mir dieser Test an Confidence pro Wartungskosten?** Ein Test, der bei jedem Refactoring bricht, ohne je einen echten Bug gefunden zu haben, war ein schlechtes Investment — egal auf welcher Ebene.
3. **Faustregel für die Ebenenwahl:** Reine Logik → Unit. Zusammenspiel mit DB/API/Dateisystem → Integration (mit echter Abhängigkeit im Container, nicht mit Mock). Kritische Nutzerpfade → wenige E2E.
4. **Sparsam mocken.** Mocken an Systemgrenzen (Netzwerk, Zeit, Zufall, externe Dienste), nicht zwischen den eigenen Klassen — dafür lieber [[Dependency Injection und Inversion of Control|Dependency Injection]] nutzen.

---

# Quellen

Recherchiert am 2026-07-26:

- [The Test Pyramid in 2026: Still Relevant, Still Necessary](https://www.onpathtesting.com/blog/the-test-pyramid-in-2026-still-relevant-still-necessary/)
- [Testing Pyramid: Strategy, Layers & Best Practices 2026 Guide](https://testomat.io/blog/testing-pyramid-role-in-modern-software-testing-strategies/)
- [The Testing Pyramid: Why 70% Unit, 20% Integration, 10% E2E Still Wins](https://medium.com/@yashbatra11111/the-testing-pyramid-why-70-unit-20-integration-10-e2e-still-wins-fb25df39c18c)
- [The Testing Pyramid Is Upside Down: E2E Tests First](https://getautonoma.com/blog/testing-pyramid) — die Gegenposition
- [Test Automation Strategy for AI-First Teams (2026)](https://getautonoma.com/blog/test-automation-strategy-ai-teams)
- [Software Testing Strategy 2026: The Engineering Guide](https://www.digitalapplied.com/blog/software-testing-strategy-2026-engineering-reference)
- [dbt Data Quality: A Complete Guide for 2026](https://atlan.com/dbt-data-quality/)
- [Your dbt Tests Are Not Data Contracts. Here's What Is.](https://medium.com/@ludovic.schmetz/your-dbt-tests-are-not-data-contracts-heres-what-is-9121052b7d29)
