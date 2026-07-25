# PureMac ↔ CleanMac — Synthesis v5.2

Questo repository (PureMac, app nativa SwiftUI) è stato analizzato insieme a **CleanMac**
(tool bash + dashboard web) per produrre **una versione unificata di sintesi**.

## Dove vive la sintesi

La versione unificata che integra **tutte** le funzionalità delle due repo è
**CleanMac (Synthesis Edition)**, scelta come base perché possiede già sia un
motore CLI sia una GUI (web) e il set di operazioni più ampio.

**Stato: fusione completata.** Ogni sorgente funzionale di PureMac ha ora un
equivalente in CleanMac:

| Sorgente PureMac | Destinazione CleanMac |
|---|---|
| `AppPathFinder.swift` | `appPathFinder.js` — matching a 9 livelli, entitlements, Team ID |
| `Conditions.swift` | `conditions.js` — 25 regole per-app + skip lists |
| `StringNormalization.swift` | `stringNormalization.js` |
| `Locations.swift` | `conditions.js` (appSearch + reverseSearch) |
| `AppInfoFetcher.swift` | `appInventory.js` |
| `AppState.findOrphans()` | `orphanFinder.js` + operazione bash `op33` |
| `FullDiskAccessManager.swift` | probe TCC bash + `GET /api/fda-status` + banner web |
| `SchedulerService.swift` | `schedule.command` (LaunchAgent) |
| `ScanEngine.swift` | operazioni bash `op01`–`op34` |
| `CleaningEngine.swift` | `safe_remove` + guard-rail server |
| `AppListView` / `AppFilesView` | pannello Uninstaller della dashboard |
| `OrphanListView` | pannello Residui della dashboard |
| `SmartScanView` | flusso dry-run → selezione → pulizia |

Restano fuori solo i file specifici di SwiftUI (`Theme.swift`, `MainWindow.swift`,
`OnboardingView.swift`, `Models.swift`, `Logger.swift`): la loro funzione è coperta
dal CSS, dalla dashboard e dai log dello script.

Vedi `SYNTHESIS.md` nel repository CleanMac per la matrice funzionale completa,
le scelte di merge e i guard-rail di sicurezza.

## Cosa cambia in PureMac (direzione reciproca)

Dove le due si sovrappongono si adotta l'impostazione migliore, in entrambe le direzioni.

### v5.0 — whitelist app di sistema

La whitelist di app di sistema Apple era più completa in CleanMac (più utility protette).
Perciò `AppInfoFetcher.protectedBundleIDs` è stata estesa per allinearla:

- Aggiunti: Books, Dictionary, Automator, Script Editor, Disk Utility, Keychain Access,
  Font Book, Image Capture, Migration Assistant, Freeform, Voice Memos, Shortcuts.

Questo riduce il rischio che utility di sistema compaiano nell'elenco disinstallabili.
È una modifica solo-dati (nessun cambiamento di logica), compatibile con la build esistente.

### v5.2 — fix falsi positivi sugli orfani Apple

Durante il porting di `findOrphans()` in Node, i test di regressione hanno
evidenziato un difetto **presente anche in questo sorgente Swift**:

```swift
if skipReverse.contains(where: { normalized.hasPrefix($0) }) { continue }
```

`skipReverse` confronta **prefissi del nome normalizzato**. Un item chiamato
`com.apple.something` viene normalizzato in `comapplesomething`, che **non** ha il
prefisso `apple`: la guardia non scattava e ogni file di sistema Apple presente
nelle location di ricerca inversa veniva classificato come orfano.

Correzione applicata in `AppState`: aggiunta `systemNamePrefixes`
(`comapple`, `apple`), verificata prima di `skipReverse`. La stessa guardia esiste
in CleanMac come `SYSTEM_NAME_PREFIXES` (`orphanFinder.js`), coperta da test
unitario, e come `case com.apple.*` nell'operazione bash `op33`.

## Funzionalità di PureMac superiori (mantenute qui)

- Esperienza nativa SwiftUI (NavigationSplitView, Table, Form) e onboarding Full Disk Access.
- Rilevamento purgeable APFS via `URLResourceValues`, più preciso del parsing `diskutil`
  usato in bash (che resta l'unica via disponibile senza Foundation).
- Scansione parallela delle location (`findPathsAsync` con `DispatchGroup`).
