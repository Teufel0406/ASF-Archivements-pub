# Achievement Automator für ArchiSteamFarm

ASF-Plugin zum **automatischen Freischalten fehlender Steam-Achievements** für ausgewählte Bots. Releases und Auto-Updates werden über dieses Repository bereitgestellt; der Quellcode wird separat gepflegt.

**Aktuelle Version:** [1.0.4.0](https://github.com/Teufel0406/ASF-Archivements-pub/releases/tag/1.0.4.0)  
**ASF:** 6.3.x mit **.NET 10**
---

## Funktionen

- **Periodischer Check** — standardmäßig alle **60 Minuten** (über `FarmingIntervalMinutes` in der Plugin-Config anpassbar)
- **Sofort-Check** bei neuen Spielen in der Steam-Bibliothek (Lizenzliste)
- **Drosselung** — Verzögerungen zwischen Anfragen, max. 20 AppIDs pro Zyklus, globale Semaphore
- **Auto-Update** — ASF lädt neuere Releases von diesem Repo (`AchievementAutomator.zip`)
- **Selbsttest** — eingebauter Diagnose-Befehl mit strukturiertem Log

---

## Installation

### Variante A: Auto-Update (empfohlen)

1. In `config/ASF.json` Plugin-Updates aktivieren:

   ```json
   "PluginsUpdateMode": 1
   ```

2. ASF neu starten (oder `!update` ausführen). Das Plugin registriert sich für Updates aus **diesem** Repository.

3. Nach dem ersten Update liegt die DLL unter `plugins/AchievementAutomator/`.

### Variante B: Manuelles Release

1. Auf der Seite [Releases](https://github.com/Teufel0406/ASF-Archivements-pub/releases) die Datei **`AchievementAutomator.zip`** der gewünschten Version herunterladen.
2. ZIP entpacken und den Inhalt nach `plugins/AchievementAutomator/` im ASF-Verzeichnis kopieren (mindestens `AchievementAutomator.dll` und `AchievementAutomator.deps.json`).
3. ASF neu starten.

> **Hinweis:** Für Plugin-Updates erwartet ASF ein **ZIP-Release-Asset**, nicht nur eine lose DLL im Git-Tree.

---

## Nutzung

Befehle in der **ASF-Konsole**, per **IPC** oder **Steam-Chat** — jeweils mit **Master**-Berechtigung:

| Befehl | Beschreibung |
|--------|----------------|
| `!achieve` | Farming für den aktuellen Bot ein-/ausschalten |
| `!achieve Bot1,Bot2` | Farming für benannte Bots umschalten |
| `!achievestatus` | Liste aller Bots mit aktivem Farming |
| `!achievetest` | Selbsttest (siehe unten) |
| `!achieve test` | Alias für `!achievetest` |
| `!aa test` | Kurzalias für `!achievetest` |

Der Bot muss **mit Steam verbunden** sein. Farming muss für den Bot zuvor mit `!achieve` aktiviert worden sein.

### Konfiguration

Beim ersten Einsatz legt das Plugin `config/AchievementAutomator.config` an (Inhalt = **JSON**, Dateiendung `.config`):

```json
{
  "EnabledBots": [
    "MeinBot"
  ],
  "FarmingIntervalMinutes": 60
}
```

| Feld | Bedeutung |
|------|-----------|
| `EnabledBots` | Botnamen mit aktivem Achievement-Farming |
| `FarmingIntervalMinutes` | Intervall des Farming-Timers in Minuten (1–1440, Standard **60**) |

Eine ältere `AchievementAutomator.json` wird beim ersten Start nach `.config` migriert.

---

## Selbsttest (`!achievetest`)

Prüft Plugin, Timer und Achievement-Pipeline **ohne** das Produktions-Intervall dauerhaft zu ändern:

1. **Zyklus 1** — sofortiger Check wie im normalen Farming-Lauf  
2. **Timer** — einmalig **2 Minuten** (danach wieder das konfigurierte Intervall, z. B. 60 min)  
3. **Zyklus 2** — Verifikations-Check nach Timer-Auslösung  
4. **Log** — zusammengefasste Meldung in ASF-/Bot-Log (`=== AchievementAutomator Self-Test … ===`, Ergebnis **PASS** / **FAIL**)

Nur **ein** Selbsttest gleichzeitig. Bei laufendem Test meldet der Befehl, dass bereits ein Test aktiv ist.

---

## Auto-Update — Voraussetzungen

Damit ASF ein Update findet und einspielt:

1. Release mit Tag = Versionsnummer (z. B. `1.0.4.0`) existiert hier auf GitHub.
2. Release-Asset heißt exakt **`AchievementAutomator.zip`**.
3. Installierte Plugin-Version ist **älter** als der Release-Tag.
4. `PluginsUpdateMode` ≥ 1 in `ASF.json`.
5. ASF darf nach dem Download neu starten (kein `--no-restart` im Docker-Entrypoint).

Manuell auslösen: `!update` oder ASF-Neustart.

---

## Fehlerbehebung

| Symptom | Was prüfen |
|---------|------------|
| Kein Plugin-Update / `PluginUpdateNoAssetFound` | Release-Tag neuer als installierte Version? Asset `AchievementAutomator.zip`? `PluginsUpdateMode` aktiv? |
| `!achieve` wird abgelehnt | Master-Recht? Aktuelle Plugin-Version? |
| Self-test: „nicht verbunden“ | Bot eingeloggt? |
| Self-test: „Farming nicht aktiviert“ | Zuerst `!achieve` für den Bot |
| Viele `ClientGetUserStats` / `Fail` | Normal bei Spielen **ohne** User-Stats — werden gecacht |
| `ClientStoreUserStats` schlägt fehl | Spiel-spezifische Steam-Limits; Plugin überspringt die AppID bis zum ASF-Neustart |
| `WebProxy` / HTTP-Fehler im ASF-Log | Meist ASF, Proxy oder **andere** Plugins — nicht dieses Achievement-Plugin |

---

## Lizenz / Haftung

Nutzung auf eigenes Risiko. Steam-Nutzungsbedingungen und Bot-Limits beachten. Kein Anspruch auf vollständige Achievement-Abdeckung bei allen Spielen.
