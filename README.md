# Achievement Automator Plugin for ASF

**Aktuelle Version:** `1.1.9.0` · Ziel: [.NET 10](https://dotnet.microsoft.com/) / neuestes [ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm)-Release

Dieses Plugin ermöglicht das automatische Freischalten fehlender Achievements für definierte Bots in ArchiSteamFarm.

> **Dokumentation:** Diese `README.md` ist die **Single Source of Truth**. Änderungen nur hier (privates Repo); [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub) wird per CI automatisch synchronisiert (`.github/workflows/sync-pub-readme.yml`). Jules-Agenten (z. B. Scribe) pflegen dieselbe Datei auf Branch `jules`.

| Repo | Zweck |
|------|--------|
| [ASF-Archivements](https://github.com/Teufel0406/ASF-Archivements) (privat) | Quellcode, Issues, CI, **README.md** |
| [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub) (öffentlich) | Releases (`AchievementAutomator.zip`) + **gespiegelte** README/SECURITY |

## Kernfunktionen

- **Automatisierung**: Prüft alle 60 Minuten die Bibliothek aktivierter Bots auf fehlende Achievements.
- **Event-basiert**: Sofortige Prüfung, wenn neue Spiele zur Bibliothek hinzugefügt werden.
- **Spamschutz**: Implementiert Verzögerungen zwischen den Anfragen, um Steam-Limits einzuhalten.
- **Konsole-Befehle**:
  - `!achieve [BotNames]` (Alias: `!aa`): Aktiviert/Deaktiviert das Farming für die Bots (Umschalten). Bei mehreren Bots werden die Antworten nach Status (**Enabled (N)**, **Disabled (N)**, **Failed (N)**) gruppiert. Ein einzelner Bot-Toggle bestätigt den Namen: `Farming enabled for bot BotName`. Um Steam-Limits einzuhalten, wird bei sehr vielen Bots eine zufällige Stichprobe (Random Sampling) angezeigt.
  - `!achieve enable/disable/on/off/start/stop [BotNames]` (Alias: `!aaenable`, `!aadisable`, …): Explizites Aktivieren oder Deaktivieren. **Ohne Bot-Namen** gilt der aufrufende Bot. Gruppiert Ergebnisse mit Mengenangaben und Zufallsauswahl bei Listen-Kürzung.
  - `!achieve status [BotNames]` (Alias: `!aastatus`): Zeigt den Session-Startzeitpunkt, eine Zusammenfassung der aktivierten Bots (X von Y), alle Bots mit aktivem Farming (wenn keine Namen angegeben) oder den Status spezifischer Bots (**Enabled (N)**/**Disabled (N)**) und den aktuellen Rate-Limit-Status an. Status-Indikatoren wie `(Disconnected)`, `(Testing...)`, `(Checking...)`, `(Self-Test pending...)` oder `(1/3 fails)` / `(Circuit Broken)` zeigen den aktuellen Zustand, Fortschritt und Fehlerrate an. Gruppierte Listen nutzen Zufallsauswahl bei Überlänge.
  - `!achieve stats [BotNames]` (Alias: `!aastats`): Zeigt Session-Statistiken (abgeschlossene/fehlgeschlagene Spiele) und den Startzeitpunkt für Bots an.
  - `!achieve app <AppID> [BotNames]` (Alias: `!aaapp`): Prüft den Status einer AppID (Globaler Skip-Cache und Session-Caches der Bots).
  - `!achieve reset <BotNames>|all|global|app <AppID> [BotNames]` (Alias: `!aareset`): Setzt Session-Caches zurück (Bot-spezifisch, alles, globale Skip-Liste oder spezifische AppID chirurgisch für bestimmte oder alle Bots).
  - `!achieve reload` (Alias: `!aareload`): Lädt die Plugin-Konfiguration neu von der Festplatte (Owner-Zugriff erforderlich).
  - `!achieve interval <Minuten>` (Alias: `!aainterval`): Zeigt das globale Farming-Intervall an oder ändert es (1–1440).
  - `!achieve version` (Alias: `!aaversion`): Zeigt die installierte Plugin-Version an.
  - `!achieve test` (Alias: `!aatest`): Startet einen Selbsttest — siehe unten.
  - `!achieve e2eteardown [Bots]` (Alias: `!aae2eteardown`): Nur bei aktiviertem `E2ETestMode`: sperrt alle Bibliotheks-Achievements wieder und fährt ASF herunter (Owner-Zugriff erforderlich).
  - `!achieve help` (Alias: `!aahelp`): Zeigt eine strukturierte Hilfe zu den Befehlen an.
- **Auto-Update**: ASF lädt Releases von [Teufel0406/ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub) (nur ZIP/DLL; Quellcode bleibt im privaten Repo).

## Architektur und Laufzeitverhalten

AchievementAutomator ist ein ASF-Plugin (`IPlugin`) und registriert sich pro Bot als Steam-Client-Handler. Aktivierte Bots werden in `config/AchievementAutomator.config` gespeichert; der Inhalt ist JSON, obwohl die Datei auf `.config` endet.

Der Farming-Ablauf ist bewusst gedrosselt:

1. ASF lädt das Plugin und liest die Konfiguration.
2. Beim ASF-Start wird ein Timer angelegt, der standardmäßig alle 60 Minuten läuft (konfigurierbar über `FarmingIntervalMinutes`). Der erste Lauf startet nach einer kurzen Verzögerung (10 Sekunden bei Intervallen ≤ 5 Min., ansonsten ca. 1 Minute mit Jitter).
3. Wenn Steam eine neue Lizenzliste für einen aktivierten Bot meldet, startet zusätzlich eine Prüfung für diesen Bot. Um Ressourcen zu schonen, ist hier ein **5-sekündiges Debouncing** aktiv: Mehrere Lizenz-Events in kurzer Folge lösen nur einen Check aus.
4. Pro Prozess läuft nur ein Farming-Check gleichzeitig. Wenn die globale Semaphore nach **90 Minuten** nicht frei ist (`MaxSemaphoreWaitMinutes`), wird der neue Check übersprungen.
5. Pro Zyklus werden maximal **100 aktivierte Bots** (in zufälliger Reihenfolge) und pro Bot maximal **40 passende AppIDs** geprüft. Apps ohne User-Stats, vollständig erledigte Apps und Apps mit komplett fehlgeschlagenem Unlock werden bis zum nächsten ASF-Neustart übersprungen.
6. Nach normalen Prüfungen wartet das Plugin 1 Sekunde, nach Unlock-Versuchen 5 Sekunden. Alle Verzögerungen beinhalten einen **zufälligen Jitter von 0–20%**, um synchrone Anfragemuster zu vermeiden.
7. **Fairness:** Zwischen der Bearbeitung verschiedener Bots wird eine Verzögerung von 5 Sekunden (plus Jitter) eingehalten, um die Last auf die Steam-Infrastruktur zu verteilen.

Beim Speichern nutzt das Plugin Steam `ClientStoreUserStats2`. Es setzt nicht nur Achievement-Bits, sondern auch abhängige INT-Stats aus dem Steam-Schema, wenn ein Achievement Fortschritts- oder `max_val`-Abhängigkeiten hat. Restricted Achievements werden nicht freigeschaltet.

### Technische Limits & Sicherheit

Um die Stabilität von ASF und die Einhaltung von Steam-Limits zu gewährleisten, implementiert das Plugin verschiedene harte Obergrenzen:

- **Bots:** Maximal 500 aktivierte Bots in der Konfiguration.
- **Konfiguration:** Die Datei `AchievementAutomator.config` darf maximal 1 MB groß sein.
- **Steam-Schemas:**
  - Maximale Größe des binären Schemas pro App: 20 MB.
  - Maximale Anzahl an Statistiken pro Schema: 5.000.
  - Maximale Anzahl an Achievements pro Schema: 10.000.
  - Maximale Anzahl an User-Stats pro App: 5.000.
- **Farming-Durchsatz:**
  - Maximal 100 Bots pro Farming-Zyklus (zufällig ausgewählt).
  - Maximal 40 Apps pro Bot und Farming-Zyklus (zufällig ausgewählt).
  - Maximal 2000 Achievements pro Bot-Zyklus.
  - Maximal 200 Achievements pro App und Zyklus.
  - Maximal 20 individuelle Retries pro App; **Circuit-Breaker** nach 3 aufeinanderfolgenden fehlgeschlagenen individuellen Fehlern bei Retries.
  - **Fairness:** Bots werden pro Zyklus geshuffelt; globale Semaphore (max. **90** Min. Wartezeit).
- **Circuit-Breaker (Bot):** Dauerhafte Fehlermarkierung und Überspringen eines Bots nach 3 aufeinanderfolgenden fehlgeschlagenen Zyklen (Reset via `!achieve reset`).
- **Circuit-Breaker (App):** Sofortiger Abbruch des aktuellen Bot-Zyklus nach 3 aufeinanderfolgenden Steam-Fehlern bei App-Checks oder fehlgeschlagenen individuellen Retries.
  - **Global Cooldown:** 30 Minuten Pause für alle Bots bei `RateLimitExceeded`.
  - **Befehls-Spamschutz:** Globaler 5-Sekunden-Cooldown pro Nutzer/Instanz für Plugin-Befehle (Owner und lokale Konsole/IPC umgehen dieses Limit).
  - **Eingabe-Limit:** Maximal 2048 Zeichen für Befehlsnachrichten und 512 Zeichen für Bot-Namen in Befehlen.
  - **Self-Test Log:** Maximal 32 Einträge, maximal 512 Zeichen pro Eintrag, maximal 3500 Zeichen Gesamtlänge.
- **Caches:**
  - Schema-Cache: 2.500 Apps.
  - Skip-Liste (technisch fehlerhafte Apps): 25.000 Einträge.
  - Session-Caches (fehlgeschlagene/erledigte Apps): bis zu 1.000.000 Einträge.

## Auto-Update (ASF)

Das Plugin implementiert `IGitHubPluginUpdates` mit `RepositoryName = Teufel0406/ASF-Archivements-pub`. In `config/ASF.json`:

```json
"PluginsUpdateMode": 1
```

| Repo | Zweck |
|------|--------|
| [ASF-Archivements](https://github.com/Teufel0406/ASF-Archivements) (privat) | Quellcode, Issues, CI-Build |
| [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub) (öffentlich) | Nur **GitHub Releases** mit `AchievementAutomator.zip` |

**CI (privates Repo):** GitHub Secret `RELEASES_REPO_PAT` = Fine-grained PAT mit **Contents: Read and write** nur auf `ASF-Archivements-pub`. Bei Git-Tag (z. B. `1.0.3.0`) baut `.github/workflows/publish.yml` die ZIP und legt das Release im öffentlichen Repo an.

Voraussetzungen auf dem ASF-Server:

1. Release-Asset **`AchievementAutomator.zip`** im öffentlichen Repo (Tag = Versionsnummer).
2. Installierte Plugin-Version älter als der Release-Tag.
3. ASF-Neustart erlaubt (kein `--no-restart` im Docker-Entrypoint).

Manuell: `!update` oder ASF-Neustart nach Update. **Hinweis:** `POST /Api/ASF/Restart` beendet ASF — in Docker mit `RestartPolicy=no` startet der Container danach **nicht** von selbst neu (siehe E2E unten).

### Öffentliche README synchronisieren

Bei Änderungen an `README.md` oder `SECURITY.md` auf `main` oder `jules` pusht der Workflow **Sync pub README** die Dateien nach [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub). Secret: `RELEASES_REPO_PAT` (wie bei `publish.yml`).

### Release-Runbook

1. `AchievementAutomator.csproj` `<Version>` auf die Zielversion setzen.
2. `dotnet restore AchievementAutomator.csproj`
3. `dotnet build AchievementAutomator.csproj -c Release`
4. Committen und einen Git-Tag mit derselben Versionsnummer setzen, z. B. `1.1.0.0`.
5. Den Tag auf **`main`** pushen (nicht auf `jules`). Der Workflow `Plugin-publish` reagiert nur auf `main` und Tags — Bot-Merges nach `jules` starten ihn nicht.
6. Im öffentlichen Repo prüfen, dass der Release-Tag existiert und das Asset exakt `AchievementAutomator.zip` heißt.

Für lokale oder manuelle Releases erzeugt `./build.sh` zusätzlich zu `out/AchievementAutomator.dll` auch `out/AchievementAutomator.zip`. ASF erwartet für Plugin-Updates ein ZIP-Asset, nicht nur eine DLL im Repository.

## Kompilierung

Das Plugin kann mit dem mitgelieferten Skript vollautomatisch kompiliert werden. Das Skript lädt sowohl die erforderliche `ArchiSteamFarm.dll` als auch das **.NET SDK** selbstständig herunter, falls diese nicht vorhanden sind:

1. Öffnen Sie ein Terminal im Verzeichnis `AchievementAutomator/`.
2. Machen Sie das Build-Skript ausführbar und führen Sie es aus:
   ```bash
   chmod +x build.sh
   ./build.sh
   ```
3. Die resultierende Datei `AchievementAutomator.dll` befindet sich im Ordner `out/`.

## Installation

1. Erstellen Sie im ASF-Ordner `plugins/` ein neues Unterverzeichnis namens `AchievementAutomator`.
2. Kopieren Sie die Datei `AchievementAutomator.dll` aus dem `out/`-Ordner in dieses Verzeichnis.
3. Starten Sie ASF neu.
4. Das Plugin legt automatisch `config/AchievementAutomator.config` (JSON-Inhalt) für die aktivierten Bots an; eine ältere `AchievementAutomator.json` wird beim ersten Start migriert.

## Nutzung

Senden Sie die Befehle über die ASF-Konsole oder per Steam-Chat (Master-Berechtigung erforderlich):

- `!achieve` (Alias: `!aa`): Aktiviert/Deaktiviert Farming für den aktuellen Bot. Antwortet mit `Farming enabled/disabled for bot BotName`.
- `!achieve Bot1,Bot2`: Umschalten für spezifische Bots. Gruppiert Antworten nach Status mit Mengenangaben (z.B. `Enabled (1)`).
- `!achieve enable/disable/on/off/start/stop [BotNames]` (Alias: `!aaenable`, …): Explizites Setzen des Status (ohne Namen: aktueller Bot).
- `!achieve status` (Alias: `!aastatus`): Liste aktiver Bots + Rate-Limit-Status. Nutzt Zufallsauswahl bei Listen-Kürzung.
- `!achieve stats [Bots]` (Alias: `!aastats`): Zeigt abgeschlossene/fehlgeschlagene Spiele der Session. Nutzt Zufallsauswahl bei Listen-Kürzung.
- `!achieve app <AppID> [Bots]` (Alias: `!aaapp`): Status einer AppID prüfen (Globaler Skip-Cache und Session-Caches).
- `!achieve reset <Bots>|all|global|app <AppID> [Bots]` (Alias: `!aareset`): Löscht Caches (Bot-spezifisch, Alles, Global oder spezifische AppID chirurgisch für bestimmte Bots), um Spiele/Apps erneut zu prüfen.
- `!achieve reload` (Alias: `!aareload`): Lädt die Konfiguration neu von der Festplatte.
- `!achieve interval <Minuten>` (Alias: `!aainterval`): Intervall anzeigen oder setzen.
- `!achieve version` (Alias: `!aaversion`): Aktuelle Plugin-Version anzeigen.
- `!achieve help` (Alias: `!aahelp`): Strukturierte Befehlsübersicht anzeigen.
- `!achievetest [Bot]` (Alias: `!achieve test [Bot]`, `!aa test [Bot]`, `!aatest [Bot]`): Führt einen **Selbsttest** für einen Bot aus (Master erforderlich; Bot muss per `!achieve` aktiviert und mit Steam verbunden sein). **Nach `!achieve enable` mindestens 5 s warten** (License-Debounce). Ablauf: (1) sofortiger Achievement-Check, (2) Farming-Timer **einmalig** 2 Min., (3) zweiter Check, (4) strukturiertes Ergebnis mit `[Result] PASS/FAIL` in ASF/Bot-Log. Bei `E2ETestMode: true` werden freigeschaltete Achievements danach auf Steam wieder gesperrt.
- `!achieve e2eteardown [Bots]` (Alias: `!aae2eteardown`): Nur bei aktiviertem `E2ETestMode`: sperrt alle Bibliotheks-Achievements wieder und fährt ASF herunter (Owner-Zugriff erforderlich).

Die Konfiguration wird automatisch geschrieben:

```json
{
  "EnabledBots": [
    "Bot1",
    "Bot2"
  ]
}
```

Botnamen werden beim Lesen und Umschalten case-insensitiv verglichen. Bearbeiten Sie die Datei möglichst nicht parallel zu laufenden ASF-Kommandos, da das Plugin bei `!achieve` speichert. Eine alte `config/AchievementAutomator.json` wird beim ersten Laden nach `AchievementAutomator.config` migriert und nach erfolgreichem Speichern gelöscht.

## Fehlerbehebung

| Symptom | Prüfen / Ursache | Lösung |
|---------|------------------|--------|
| ASF findet kein Plugin-Update oder meldet `PluginUpdateNoAssetFound` | Release liegt nicht im öffentlichen Repo, Asset heißt nicht `AchievementAutomator.zip`, `PluginsUpdateMode` ist deaktiviert oder der Release-Tag ist nicht neuer als die installierte Version. | Öffentliches Release-Repo, ZIP-Asset und Versions-Tag prüfen; ASF-Neustart bzw. `!update` auslösen. |
| `!achieve BotName` wird nicht akzeptiert | Der Befehl benötigt Master-Zugriff. Alte Plugin-Versionen hatten außerdem Probleme mit unterschiedlich tokenisierten ASF-Konsole-/IPC-Argumenten. | Aktuelle Plugin-Version installieren und Befehl als Master über ASF-Konsole, IPC oder Steam-Chat senden. Anführungszeichen in Bot-Namen werden nun korrekt unterstützt. |
| `ClientGetUserStats` liefert `Fail`, `AccessDenied`, `InvalidParam`, `Banned`, `AccountDisabled` oder `Blocked` | Viele Steam-Bibliothekseinträge haben keine User-Stats oder sind zugriffsgeschützt; das ist normal. | Keine Aktion erforderlich. Das Plugin cached solche AppIDs (Silent Fail) bis zum ASF-Neustart und reduziert weitere Anfragen. |
| `ClientStoreUserStats` schlägt mit `InvalidParam`, `AccessDenied`, `Fail`, `Banned`, `AccountDisabled` oder `Blocked` fehl | Steam-Schemas unterscheiden sich; manche Spiele erlauben kein Setzen bestimmter Werte oder haben zusätzliche Einschränkungen. | Das Plugin markiert solche Apps als fehlerhaft, cached sie (Silent Fail) und überspringt sie bis zum ASF-Neustart. |
| `RateLimitExceeded` oder globaler Cooldown | Zu viele Anfragen in kurzer Zeit. | Das Plugin aktiviert einen **30-minütigen globalen Cooldown** für alle Bots. Abwarten; das Plugin setzt die Arbeit danach automatisch fort. |
| `Timeout` bei Steam-Anfragen | Steam reagiert nicht rechtzeitig auf die Anfrage (Netzwerkprobleme oder Steam-Wartung). | Keine Aktion erforderlich; die App wird im aktuellen Zyklus übersprungen und beim nächsten Mal erneut geprüft. |
| `WebProxy`-/HTTP-Fehler im ASF-Log | Diese Fehler stammen typischerweise vom ASF-HTTP-Stack, Proxy-Umgebung oder anderen Plugins, nicht vom Achievement-Handler. | Proxy-Variablen, andere Plugins und fremde `System.*.dll` im Plugin-Ordner prüfen. |
| Installierte Plugin-Version ist unklar | Die Version wird im Log beim Start ausgegeben oder kann per Befehl abgefragt werden. | Befehl `!achieve version` (Alias: `!aaversion`) nutzen. |

## Entwicklerhinweise

- Das Projekt zielt auf `net10.0`, passend zur referenzierten `libs/ArchiSteamFarm.dll` (immer **neuestes ASF-Release**, nie downgraden).
- `libs/ArchiSteamFarm.dll` wird nicht in das Plugin kopiert (`Private=false`); ASF stellt die Runtime-Abhängigkeit bereit.
- **Tests:** `dotnet test Tests/AchievementAutomator.Tests.csproj -c Release -- RunConfiguration.MaxParallelThreads=1`
- **E2E (Home-AtF):** Test-Instanz `ArchitestFarm` (Docker, IPC **1245**, AppData `/mnt/user/appdata/AtF`). Deploy: `scripts/test-deploy-atf.sh` — nutzt **`docker stop` / DLL-Copy / `docker start`** (nicht IPC-Restart; Container hat `RestartPolicy=no`). Optional `RUN_ACHIEVE_TEST=1` für `!achievetest`. Produktions-ASF auf Port **1242** nicht für E2E nutzen.
- Für ASF-Kompatibilität nutzt das Plugin konservative BCL-APIs: Timeouts über `CancellationTokenSource.CancelAfter`, JSON über `System.Text.Json` Source Generation, AppID-Reihenfolge über Fisher-Yates.
- Sicherheitsmeldungen: siehe [SECURITY.md](SECURITY.md).
