---
epoch: 1793710679737
mode: agent
backendId: claude
sessionId: "e24ba18c-6e82-41a2-ae98-1b25754afbc5"
agentLabel: "Ich würde gerne in diesem Vault die folgenden Schriften nutz…"
tags:
  - copilot-conversation
usage: '{"usedTokens":48057,"contextWindow":1000000,"inputTokens":2,"outputTokens":2,"cacheReadTokens":46491,"cacheWriteTokens":1562,"updatedAt":1793710847505}'
---

**user**: Ich würde gerne in diesem Vault die folgenden Schriften nutzen: Lexend für text. Inter für interface und recursive monospace für monospace. kannst du mir diese runterladen und in Resources/Fonts legen? ich habe bereits das Plugin Custom Font Loader
[Timestamp: 2026/11/03 13:57:59]

**ai**: Ich lade jetzt die Schriften herunter (Lexend und Inter aus dem Google-Fonts-Repo, Recursive Mono aus dem offiziellen Release).Ich wähle statische Schnitte (Regular, Bold usw.), weil das Plugin variable Fonts nur mit Gewicht 400 registriert. Lade jetzt die statischen Lexend-Dateien.Kopiere die Dateien jetzt nach `Resources/Fonts`.Die drei Schriften liegen jetzt in `Resources/Fonts/` und sind committet. In Obsidian zugewiesen sind sie noch nicht, das musst du in den Plugin-Einstellungen selbst machen.

**Was in `Resources/Fonts/` liegt:**

| Rolle | Schrift | Schnitte | Quelle |
|---|---|---|---|
| Text | Lexend (`.ttf`) | Regular, Medium, SemiBold, Bold | Offizielles Repo `googlefonts/lexend` |
| Interface | Inter 4.1 (`.woff2`) | Regular, Medium, SemiBold, Bold, jeweils mit Italic | Offizielles Release `rsms/inter` |
| Monospace | Recursive Mono 1.085, Variante „Linear“ (`.woff2`) | Regular, Italic, Bold, BoldItalic | Offizielles Release `arrowtype/recursive` |

Dazu kommt zu jeder Schrift die Lizenz als `<Schrift> OFL.txt`. Alle drei stehen unter der freien Open Font License.

**Warum ich keine variablen Fonts genommen habe:** Das Plugin liest Gewicht und Stil aus dem Dateinamen (z. B. `-Bold`, `-Italic`) und registriert eine variable Schrift nur als Regular. Fett wäre damit nicht richtig dargestellt worden. Deshalb habe ich einzelne Dateien pro Schnitt genommen. Die Recursive-Dateien habe ich dafür von `RecursiveMonoLnrSt-…` in `RecursiveMono-…` umbenannt. Lexend gibt es nicht kursiv, kursiver Text wird also schräg berechnet.

**So richtest du es ein:**

1. Trag in den Einstellungen von Custom Font Loader als Font-Ordner `Resources/Fonts` ein.
2. Wähl pro Rolle die Dateien aus:
	- Text: alle `Lexend-*`
	- Interface: alle `Inter-*`
	- Monospace: alle `RecursiveMono-*`
3. Lade das Plugin neu.

Hätte ich die Einstellungsdatei des Plugins direkt bearbeitet, hätte das laufende Obsidian sie überschreiben können.

Recursive Mono gibt es auch in der verspielteren Variante „Casual“. Die kann ich dir statt „Linear“ dazulegen, wenn du sie lieber magst.

Commit: `Ergaenze Schriften Lexend, Inter und Recursive Mono` (19 neue Dateien, rund 1,8 MB).
[Timestamp: 2026/11/03 13:57:59]

**user**: was meinst du mit wähle alle Lexend-* aus? Warum? Und sicher muss ich dann irgendwie eine Reihenfolge beachten oder so?
[Timestamp: 2026/11/03 14:00:31]

**ai**: Ich habe im Code des Plugins nachgesehen. Die Reihenfolge ist in deinem Fall egal.

**Warum alle Dateien einer Schrift?** Jede Datei enthält genau einen Schnitt. `Lexend-Regular.ttf` hat nur normale Buchstaben, `Lexend-Bold.ttf` nur fette. Das Plugin lädt nur die Dateien, die du einer Rolle zuweist. Es meldet jede davon unter demselben Schriftnamen `lexend` an und liest dabei Gewicht und Stil aus dem Dateinamen.

- **Nur Regular ausgewählt:** Für `**fett**` oder Überschriften gibt es keine echte fette Lexend. Der Browser verdickt dann die Regular-Buchstaben künstlich, und das sieht meist matschig aus.
- **Alle `Lexend-*` ausgewählt:** Der Browser nimmt für jeden Text den passenden echten Schnitt, also Regular für Fließtext, Bold für Fettes und SemiBold, wo das Theme es nutzt.

Bei Inter und Recursive Mono gilt dasselbe, dort kommen noch die kursiven Schnitte dazu.

**Reihenfolge:** In den Einstellungen siehst du die Einträge nummeriert. Der erste heißt „Preferred“, die übrigen „Fallback“. Das ist für den Fall gedacht, dass du verschiedene Schriften mischst, z. B. Lexend und als Ersatz eine andere Schrift für Zeichen, die Lexend nicht hat. Das Plugin fasst Dateien derselben Schrift aber zu einem einzigen Namen zusammen. Ob `Lexend-Bold` vor oder nach `Lexend-Regular` steht, ändert also nichts. Achte nur darauf, dass du pro Rolle nicht aus Versehen Dateien einer anderen Schrift mit auswählst.

Praktisch heißt das: Klick unter „Text font“ die vier `Lexend`-Dateien in beliebiger Reihenfolge an, unter „Interface font“ die acht `Inter`-Dateien und unter „Monospace font“ die vier `RecursiveMono`-Dateien.

Wenn dir das zu viel Klickerei ist, kann ich die Zuweisung auch direkt in die Einstellungsdatei des Plugins schreiben. Dafür solltest du Obsidian vorher schließen, sonst kann es die Datei überschreiben.
[Timestamp: 2026/11/03 14:00:31]