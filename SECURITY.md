# Security Policy - AchievementAutomator

**Last updated:** 2026-06-15

AchievementAutomator ist ein Plugin für [ArchiSteamFarm (ASF)](https://github.com/JustArchiNET/ArchiSteamFarm). Veröffentlichte Builds (ZIP/DLL) stammen von [Teufel0406/ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub).

## Unterstützte Versionen

| Version | Unterstützt | Hinweis |
| ------- | ----------- | ------- |
| **1.1.x** | [OK] | Aktuelle Release-Linie |
| 1.0.x | [NO] | Bitte auf das neueste Release von `-pub` aktualisieren |
| < 1.0 | [NO] | Nicht mehr gepflegt |

ASF sollte zur Plugin-Version passen (aktuell **.NET 10** und eine aktuelle ASF-Installation). Ältere ASF-Versionen nicht verwenden, um ein veraltetes Plugin weiter zu betreiben - stattdessen Plugin und ASF aktualisieren.

## Was wir als sicherheitsrelevant betrachten

- Unbefugtes Freischalten von Achievements oder Manipulation von Steam-Stats über das Plugin
- Ausführung von beliebigem Code oder Path-Traversal über Plugin-Konfiguration oder Befehle
- Leak von **Steam-Credentials**, Shared Secrets, IPC-Passwörtern oder Tokens in Logs oder Antworten
- Denial-of-Service gegen die ASF-Instanz (Ressourcen-Exhaustion, Deadlocks)
- Schwachstellen in **Plugin-Abhängigkeiten** (z. B. NuGet-Pakete im Release-Build)

**Nicht im Scope:** Allgemeine Steam-/Valve-Sicherheit und ASF-Kernbugs - bitte an [JustArchiNET/ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm/issues) melden.

## Meldung einer Schwachstelle

**Bitte keine öffentlichen Issues** mit ungepatchten Exploit-Details.

### So melden Sie (empfohlen)

Dieses Repository ([ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub)) hat **Private Vulnerability Reporting** aktiviert:

1. Öffnen Sie die [Security-Seite von ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub/security)
2. Klicken Sie auf **Report a vulnerability** (bzw. *Schwachstelle melden*)
3. Formular ausfüllen und absenden - die Meldung ist nur für Maintainer sichtbar

Alternativ: Tab **Security and quality** -> **Report a vulnerability**.

### Was danach passiert

- Meldungen werden vom Maintainer im öffentlichen Release-Repository entgegengenommen
- Zur Entwicklung und Behebung werden sie ins **private Quellcode-Repository** weitergeleitet
- Gepatchte Versionen erscheinen als neues Release auf [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub/releases)

**Nicht senden:** Live-Passwörter, `STEAM_SHARED_SECRET`, vollständige Bot-Configs - nur minimal reproduzierbare, anonymisierte Beispiele.

### Erwartbare Antwortzeiten

| Schritt | Ziel |
| ------- | ---- |
| Eingangsbestätigung | innerhalb von **7 Tagen** |
| Erste Einschätzung (Schweregrad/Reproduktion) | innerhalb von **14 Tagen** |
| Fix oder begründete Ablehnung | abhängig von Schweregrad; kritische Plugin-Fixes haben Priorität |

Nach Freigabe eines Fixes kann eine öffentliche Security Advisory auf `-pub` veröffentlicht werden.

## Bekannte Grenzen (kein Bug)

- Das Plugin benötigt Steam-Login der konfigurierten Bots - Betrieb nur auf vertrauenswürdigen Hosts
- ASF-Konsole und IPC gelten als vertrauenswürdige Steuerungskanäle (lokaler Zugriff / Owner)
- Logs enthalten absichtlich nur Bot-Namen und AppIDs, keine Secrets
