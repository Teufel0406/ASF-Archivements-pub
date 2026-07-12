# Achievement Automator Plugin for ASF

**Version:** `1.1.9.0` * **Last updated:** 2026-07-11 * Benoetigt [.NET 10](https://dotnet.microsoft.com/) und eine aktuelle [ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm)-Installation

Achievement Automator ist ein Plugin fuer [ArchiSteamFarm (ASF)](https://github.com/JustArchiNET/ArchiSteamFarm). Es prueft die Steam-Bibliothek konfigurierter Bots und schaltet fehlende Achievements automatisch frei - mit Drosselung und Limits, damit Steam-Anfragen im Rahmen bleiben.

**Releases & Updates:** [Teufel0406/ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub)

## Funktionen

- **Zeitgesteuert:** Standardmaessig alle 60 Minuten (einstellbar) Pruefung aller aktivierten Bots
- **Event-basiert:** Sofortige Pruefung, wenn neue Spiele zur Bibliothek hinzugefuegt werden (mit 5 s Debouncing)
- **Stat-Abhaengigkeiten:** Setzt bei Bedarf abhaengige Steam-Stats aus dem Spielschema mit
- **Spamschutz:** Verzoegerungen, Jitter, Rate-Limit-Cooldown und Circuit-Breaker
- **ASF-Plugin-Updates:** Automatische Updates ueber das oeffentliche Release-Repo (optional)

## Installation

### Variante A - Release (empfohlen)

1. Neuestes Release auf [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub/releases) oeffnen
2. **`AchievementAutomator.zip`** herunterladen
3. ZIP entpacken - die Datei `AchievementAutomator.dll` (und ggf. `AchievementAutomator.deps.json`) in einen Ordner `plugins/AchievementAutomator/` im ASF-Verzeichnis legen
4. ASF neu starten
5. Beim ersten Start legt das Plugin `config/AchievementAutomator.config` an (JSON-Inhalt)

### Variante B - Manuelles Kopieren

1. Ordner `plugins/AchievementAutomator/` im ASF-Verzeichnis anlegen
2. `AchievementAutomator.dll` aus dem Release dort ablegen
3. ASF neu starten

> Eine aeltere `config/AchievementAutomator.json` wird beim ersten Laden automatisch nach `AchievementAutomator.config` migriert.

## Auto-Update (ASF)

In `config/ASF.json`:

```json
"PluginsUpdateMode": 1
```

ASF laedt dann neuere Versionen von [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub), wenn ein Release-Tag neuer ist als die installierte Plugin-Version. Voraussetzungen:

- Release-Asset heisst **`AchievementAutomator.zip`**
- ASF-Neustart ist erlaubt (kein dauerhaftes `--no-restart` ohne manuelles Update)

Manuell: ASF-Befehl `!update` oder Neustart nach Update.

## Erste Schritte

1. ASF starten und Bot einloggen
2. Farming aktivieren, z. B. in der ASF-Konsole oder per Steam-Chat (Master):

   ```
   !achieve enable MeinBot
   ```

   Kurzform: `!aa enable MeinBot`

3. Status pruefen: `!achieve status` (`!aastatus`)

Die aktivierten Bots werden in `config/AchievementAutomator.config` gespeichert.

### Beispiel-Konfiguration

```json
{
  "EnabledBots": [
    "Bot1",
    "Bot2"
  ],
  "FarmingIntervalMinutes": 60
}
```

Botnamen sind case-insensitiv. Bearbeiten Sie die Datei moeglichst nicht parallel zu laufenden `!achieve`-Befehlen, da das Plugin bei Umschaltungen speichert.

## Befehle

Alle Befehle ueber ASF-Konsole, IPC oder Steam-Chat (in der Regel **Master**-Berechtigung). Aliase in Klammern.

| Befehl | Beschreibung |
|--------|----------------|
| `!achieve [BotNames]` (`!aa`) | Farming fuer Bots **umschalten** (an/aus). Mehrere Bots: gruppierte Antwort (Enabled/Disabled/Failed) |
| `!achieve enable/disable/on/off/start/stop [BotNames]` (`!aaenable`, `!aadisable`, `!aaon`, `!aaoff`, `!aastart`, `!aastop`) | Explizit an oder aus. **Ohne Namen:** aktueller Bot |
| `!achieve status/info [BotNames]` (`!aastatus`, `!aainfo`) | Aktivierte Bots, Rate-Limit, Status-Indikatoren |
| `!achieve stats [BotNames]` (`!aastats`) | Session-Statistik (abgeschlossen/fehlgeschlagen) |
| `!achieve app/apps <AppID> [BotNames]` (`!aaapp`, `!aaapps`) | Status einer AppID (Globaler Skip-Cache & Session-Caches) |
| `!achieve reset <Bots>\|all\|global\|app <AppID> [Bots]` (`!aareset`) | Session- oder Skip-Caches zuruecksetzen. `app` entfernt AppID chirurgisch aus allen Caches. |
| `!achieve reload` (`!aareload`) | Konfiguration von der Festplatte neu laden (**Owner**) |
| `!achieve interval [Minuten]` (`!aainterval`) | Farming-Intervall anzeigen oder setzen (1-1440) |
| `!achieve version` (`!aaversion`) | Installierte Plugin-Version |
| `!achieve help` (`!aahelp`) | Kurzuebersicht aller Befehle |
| `!achievetest [Bot]` (`!achieve test`, `!aatest`) | Selbsttest - siehe unten |

### Selbsttest (`!achievetest`)

Prueft die Plugin-Pipeline fuer einen Bot (muss aktiviert und mit Steam verbunden sein):

1. Sofortiger Achievement-Check (wie im Farming-Zyklus)
2. Farming-Timer **einmalig** auf 2 Minuten
3. Zweiter Check nach Timer
4. Zusammenfassung im ASF-/Bot-Log (`[Result] PASS` oder `FAIL`)

Nach `!achieve enable` **mindestens 5 Sekunden** warten, bevor der Test startet (License-Debounce).

## Laufzeitverhalten (Kurz)

1. Timer prueft aktivierte Bots (Standard: 60 Min., erster Lauf nach kurzer Verzoegerung)
2. Neue Steam-Lizenzen loesen zusaetzlich eine bot-spezifische Pruefung aus
3. Pro ASF-Prozess laeuft hoechstens **ein** Farming-Check gleichzeitig
4. Pro Zyklus: max. **100 Bots**, pro Bot max. **40 AppIDs** (zufaellige Auswahl bei mehr)
5. Verzoegerungen zwischen Apps und Bots schuetzen vor Steam-Limits
6. Restricted Achievements werden nicht freigeschaltet

Technische Obergrenzen (Auszug):

| Bereich | Limit |
|---------|--------|
| Aktivierte Bots in Config | 500 |
| BotName-Laenge | 64 |
| Config-Dateigroesse | 1 MB |
| Schema-Dateigroesse (Cache) | 20 MB |
| Max. Stats im Schema | 5.000 |
| Max. Achievements im Schema | 10.000 |
| Max. User-Stats zu verarbeiten | 5.000 |
| Max. Bits pro Stat | 1.000 |
| Max. Kinder pro Achievement | 100 |
| Bots pro Zyklus | 100 |
| Apps pro Bot und Zyklus | 40 |
| Achievements pro Bot-Zyklus | 2.000 |
| Achievements pro AppID und Zyklus | 200 |
| Individuelle Retries pro App | 20 |
| Konsekutive Fehler Circuit-Breaker (App) | 3 |
| Konsekutive Fehler Circuit-Breaker (Bot) | 3 |
| Schema-Cache (Eintraege) | 2.500 |
| Globaler Skip-Cache (Eintraege) | 25.000 |
| Persistente Skips (Eintraege) | 10.000 |
| Skip-Dateigroesse | 1 MB |
| Session-Cache (Eintraege) | 1.000.000 |
| Globaler Cooldown bei Rate-Limit | 30 Min. |
| Befehls-Cooldown | 5 s (Owner/Konsole/lokal ausgenommen) |
| Semaphore-Timeout | 90 Min. |
| Maximale Antwortlaenge (Gesamt) | 1.600 Zeichen |
| Maximale Statistik-Liste | 1.000 Zeichen |
| Maximale App-Status-Liste | 400 Zeichen |
| Maximale Individual-Status-Liste | 400 Zeichen |
| Maximale Multi-Bot-Umschalt-Zusammenfassung | 400 Zeichen |
| Maximale Zusammenfassungs-Laenge | 400 Zeichen |
| Maximale Befehlslaenge (Eingabe) | 2.048 Zeichen |
| Maximale Botnamen-Liste (Befehl) | 512 Zeichen |

## Fehlerbehebung

| Symptom | Was tun |
|---------|---------|
| Kein Plugin-Update / `PluginUpdateNoAssetFound` | Release auf [ASF-Archivements-pub](https://github.com/Teufel0406/ASF-Archivements-pub) pruefen: Tag neuer als installierte Version, Asset `AchievementAutomator.zip`, `PluginsUpdateMode` >= 1, ASF neu starten |
| Befehl wird abgelehnt | Master-Berechtigung noetig; aktuelle Plugin-Version verwenden |
| Viele Apps werden uebersprungen (`Fail`, `AccessDenied`, ...) | Normal bei Spielen ohne User-Stats oder mit Zugriffsbeschraenkung - Plugin cached diese Apps bis zum ASF-Neustart |
| Unlock schlaegt fehl (`InvalidParam`, ...) | Schema-/Fortschrittsabhaengigkeit - App wird uebersprungen und gecacht |
| `RateLimitExceeded`, `IPBanned` | 30 Min. globaler Cooldown fuer alle Bots - abwarten |
| `Timeout` | App wird im aktuellen Zyklus uebersprungen, spaeter erneut versucht |
| Version unbekannt | `!achieve version` |

Weitere Logs: ASF-Log und Bot-Log nach `AchievementAutomator` durchsuchen.

## Sicherheit

Sicherheitsluecken bitte gemaess [SECURITY.md](SECURITY.md) melden - nicht als oeffentliches Issue mit Exploit-Details.

## Lizenz & Haftung

Nutzen auf eigenes Risiko. Steam-Nutzungsbedingungen und ASF-Richtlinien beachten. Das Plugin ist kein offizielles Valve- oder ASF-Produkt.
