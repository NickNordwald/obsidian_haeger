---
tags:
  - documentation
---
# PGP-Verschlüsselung (Gpg4win / Kleopatra)

PGP (Pretty Good Privacy) wird bei [[Documentation/Haeger Consulting.md|Haeger Consulting]] zum Verschlüsseln sensibler E-Mails eingesetzt. Unter Windows kommt dafür **Gpg4win** mit der Zertifikatsverwaltung **Kleopatra** zum Einsatz.

Bereitgestellt von: **GnuPG-Projekt** (freie/Open-Source-Software, Windows-Installer über [gpg4win.de](https://www.gpg4win.de)) — anders als die übrigen Services kein kommerzieller Anbieter.

Genutzt bei [[Documentation/Haeger Consulting.md|Haeger Consulting]], um sensible Daten verschlüsselt per E-Mail zu versenden. Jede:r Mitarbeitende erstellt dafür einen eigenen PGP-Key für die dienstliche E-Mail-Adresse (siehe auch [[Documentation/Onboarding.md|Onboarding-Basics]]).

Quelle: [[Resources/Startdokumente/PGP Quick Guide1.0.pdf|PGP Quick Guide 1.0]] (Original mit bebilderter Schritt-für-Schritt-Anleitung, Autor: [[Documentation/People/David Haeger.md|David Haeger]])

## Grundkonzept
PGP arbeitet mit einem **Schlüsselpaar**:
- **Privater Key (Secret Key)**: wird zum Entschlüsseln empfangener und Signieren eigener Nachrichten benötigt — niemals an Dritte weitergeben.
- **Öffentlicher Key (Public Key)**: wird geteilt — andere nutzen ihn, um dir verschlüsselte Nachrichten zu schicken bzw. deine Signatur zu prüfen.

## Kurzanleitung: PGP-Key erstellen (Windows)
1. **Gpg4win** von [gpg4win.de](https://www.gpg4win.de) herunterladen und installieren. Bei der Komponentenauswahl **GpgOL** mit installieren, falls Outlook genutzt wird (bei Thunderbird nicht nötig).
2. **Kleopatra** öffnen → **Neues Schlüsselpaar** → Name + dienstliche E-Mail-Adresse eingeben (ausgeschriebene Form, z. B. `max.mustermann@haeger-consulting.de`, nicht `mmustermann@...`).
3. Haken bei **„Den generierten Schlüssel mit einer Passphrase schützen"** setzen.
4. Unter **Erweiterte Einstellungen**: **kein Ablaufdatum** für den Key setzen (empfohlener Standard) → **OK** → **Erstellen** → Passphrase festlegen.
5. **Sicherheitskopie** des Schlüsselpaares erstellen; den **Secret Key** an einem eigenen, sicheren Ort speichern — niemals weitergeben.
6. **Public Key exportieren** und an `verwaltung@haeger-consulting.de` schicken. Es kommt danach eine verschlüsselte Testnachricht zur Verifikation zurück — lässt sie sich nicht entschlüsseln, [[Documentation/People/David Haeger.md|David Haeger]] kontaktieren.

## Nutzung im Mail-Client
- **Outlook**: Add-in wird bei der Gpg4win-Installation automatisch eingerichtet (sofern Haken gesetzt). Entschlüsseln = E-Mail anklicken + Passphrase eingeben (danach ca. 30 Min. Cooldown, in dem ohne erneute Eingabe ver-/entschlüsselt werden kann). Verschlüsseln = vor dem Senden auf **„Absichern"** klicken, dann Passphrase eingeben.
- **Thunderbird**: Key über **☰ → Extras → OpenPGP-Schlüssel verwalten → Datei → Geheime(n) Schlüssel aus Datei importieren** einbinden, Passphrase eingeben.
- **macOS / Linux**: externe Tutorials im Original-PDF verlinkt.

## Key verlängern
Standardmäßig kein Ablaufdatum, kann aber nachträglich angepasst werden:
- **Kleopatra**: Secret Key → Rechtsklick → **Ablaufdatum ändern**. Wichtig: Haken „auch für Unterschlüssel aktualisieren" setzen, sonst wird nur ein Teil des Keys verlängert und die verschlüsselte Kommunikation bricht ab.
- **Konsole**: `gpg --edit-key max.mustermann@haeger-consulting.de` → `expire` → Anweisungen folgen → mit `quit` und `j` speichern.

Nach jeder Verlängerung den aktualisierten Key erneut an `verwaltung@haeger-consulting.de` schicken.

---
Übersicht: [[Documentation/Websites und Services.md]]
