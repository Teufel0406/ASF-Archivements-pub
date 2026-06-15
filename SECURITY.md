# Security Policy — AchievementAutomator

**Last updated:** 2026-06-15

AchievementAutomator ist ein [ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm)-Plugin. Releases (ZIP/DLL) liegen öffentlich unter [Teufel0406/ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub); Quellcode und Jules-Automation in diesem privaten Repo.

## Unterstützte Versionen

| Version | Unterstützt | Hinweis |
| ------- | ----------- | ------- |
| **1.1.x** | ✅ | Aktuelle Release-Linie (`AchievementAutomator.csproj`) |
| 1.0.x | ❌ | Bitte auf neuestes Release von `-pub` aktualisieren |
| < 1.0 | ❌ | Nicht mehr gepflegt |

ASF selbst muss zur Plugin-Version passen (aktuell **.NET 10** / neuestes ASF-Release). Kein Downgrade von ASF, um ein älteres Plugin zu betreiben — siehe `AGENTS.md` (ASF version policy).

## Was wir als Sicherheitsrelevant betrachten

- Unbefugtes Freischalten von Achievements oder Manipulation von Steam-Stats über das Plugin
- Ausführung von beliebigem Code / Path-Traversal über Plugin-Konfiguration oder Befehle
- Leak von **Steam-Credentials**, Shared Secrets, IPC-Passwörtern oder Tokens in Logs/Responses
- Denial-of-Service gegen die ASF-Instanz (Ressourcen-Exhaustion, Deadlocks)
- Schwachstellen in **Abhängigkeiten** (NuGet, `apps/web`, GitHub Actions) — Bearbeitung über Dependabot + manuelle Review

**Nicht im Scope:** Allgemeine Steam-/Valve-Sicherheit, ASF-Kernbugs (bitte an [JustArchiNET/ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm/issues) melden).

## Meldung einer Schwachstelle

**Bitte keine öffentlichen Issues** für ungepatchte Sicherheitslücken.

1. **Bevorzugt:** [GitHub Security Advisory](https://github.com/Teufel0406/ASF-Archivements/security/advisories/new) (privat) in diesem Repo  
2. **Alternativ:** GitHub-Issue mit Prefix `[SECURITY]` nur wenn keine sensiblen Exploit-Details nötig sind

**Nicht senden:** Live-Passwörter, `STEAM_SHARED_SECRET`, volle Bot-Configs — nur minimal reproduzierbare, anonymisierte Beispiele.

### Was wir erwarten können

| Schritt | Ziel |
| ------- | ---- |
| Eingangsbestätigung | innerhalb von **7 Tagen** |
| Erste Einschätzung (Severity/Repro) | innerhalb von **14 Tagen** |
| Fix oder begründete Ablehnung | abhängig von Schwere; kritische Plugin-Fixes priorisiert auf `jules`, Release nach Review auf `main` / `-pub` |

## Abhängigkeiten & Automation

- **Dependabot:** `.github/dependabot.yml` — PRs gegen `main` (Plugin-NuGet, `apps/web`, Actions)
- **Dashboard:** `apps/web` — CI [React Doctor](.github/workflows/react-doctor.yml)
- **Jules-KI-Agenten:** bearbeiten **keine** Security-Meldungen; Meldungen nur an den Maintainer (dieses Dokument)

## Bekannte Grenzen (kein Bug)

- Plugin benötigt Steam-Login der konfigurierten Bots — Betrieb nur auf vertrauenswürdigen Hosts
- IPC/ASF-Konsole gelten als vertrauenswürdig (`steamID == 0` / Owner) — siehe `AGENTS.md`
- Logs enthalten absichtlich nur Bot-Namen und AppIDs, keine Secrets
