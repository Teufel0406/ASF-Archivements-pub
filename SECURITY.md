# Security Policy - AchievementAutomator

**Last updated:** 2026-07-25

AchievementAutomator ist ein Plugin fuer [ArchiSteamFarm (ASF)](https://github.com/JustArchiNET/ArchiSteamFarm). Veroeffentlichte Builds (ZIP/DLL) stammen von [Teufel0406/ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub).

## Unterstuetzte Versionen

| Version | Unterstuetzt | Hinweis |
| ------- | ----------- | ------- |
| **1.1.x** | [OK] | Aktuelle Release-Linie |
| 1.0.x | [NO] | Bitte auf das neueste Release von `-pub` aktualisieren |
| < 1.0 | [NO] | Nicht mehr gepflegt |

ASF sollte zur Plugin-Version passen (aktuell **.NET 10** und eine aktuelle ASF-Installation). Aeltere ASF-Versionen nicht verwenden, um ein veraltetes Plugin weiter zu betreiben - stattdessen Plugin und ASF aktualisieren.

## Was wir als sicherheitsrelevant betrachten

- Unbefugtes Freischalten von Achievements oder Manipulation von Steam-Stats ueber das Plugin
- Ausfuehrung von beliebigem Code oder Path-Traversal ueber Plugin-Konfiguration oder Befehle
- Leak von **Steam-Credentials**, Shared Secrets, IPC-Passwoertern oder Tokens in Logs oder Antworten
- Denial-of-Service gegen die ASF-Instanz (Ressourcen-Exhaustion, Deadlocks)
- Schwachstellen in **Plugin-Abhaengigkeiten** (z. B. NuGet-Pakete im Release-Build)

**Nicht im Scope:** Allgemeine Steam-/Valve-Sicherheit und ASF-Kernbugs - bitte an [JustArchiNET/ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm/issues) melden.

## Meldung einer Schwachstelle

**Bitte keine oeffentlichen Issues** mit ungepatchten Exploit-Details.

### So melden Sie (empfohlen)

Dieses Repository ([ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub)) hat **Private Vulnerability Reporting** aktiviert:

1. Oeffnen Sie die [Security-Seite von ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub/security)
2. Klicken Sie auf **Report a vulnerability** (bzw. *Schwachstelle melden*)
3. Formular ausfuellen und absenden - die Meldung ist nur fuer Maintainer sichtbar

Alternativ: Tab **Security and quality** -> **Report a vulnerability**.

### Was danach passiert

- Meldungen werden vom Maintainer im oeffentlichen Release-Repository entgegengenommen
- Zur Entwicklung und Behebung werden sie ins **private Quellcode-Repository** weitergeleitet
- Gepatchte Versionen erscheinen als neues Release auf [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub/releases)

**Nicht senden:** Live-Passwoerter, `STEAM_SHARED_SECRET`, vollstaendige Bot-Configs - nur minimal reproduzierbare, anonymisierte Beispiele.

### Erwartbare Antwortzeiten

| Schritt | Ziel |
| ------- | ---- |
| Eingangsbestaetigung | innerhalb von **7 Tagen** |
| Erste Einschaetzung (Schweregrad/Reproduktion) | innerhalb von **14 Tagen** |
| Fix oder begruendete Ablehnung | abhaengig von Schweregrad; kritische Plugin-Fixes haben Prioritaet |

Nach Freigabe eines Fixes kann eine oeffentliche Security Advisory auf `-pub` veroeffentlicht werden.

## Bekannte Grenzen (kein Bug)

- Das Plugin benoetigt Steam-Login der konfigurierten Bots - Betrieb nur auf vertrauenswuerdigen Hosts
- ASF-Konsole und IPC gelten als vertrauenswuerdige Steuerungskanaele (lokaler Zugriff / Owner)
- Logs enthalten absichtlich nur Bot-Namen und AppIDs, keine Secrets
