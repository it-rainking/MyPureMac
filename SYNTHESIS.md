# PureMac ↔ CleanMac — Synthesis v5.0

Questo repository (PureMac, app nativa SwiftUI) è stato analizzato insieme a **CleanMac**
(tool bash + dashboard web) per produrre **una versione unificata di sintesi**.

## Dove vive la sintesi

La versione unificata che integra **tutte** le funzionalità delle due repo è
**CleanMac v5.0 (Synthesis Edition)**, scelta come base perché possiede già sia un
motore CLI sia una GUI (web) e il set di operazioni più ampio. Da PureMac sono state
portate lì le capacità mancanti:

- **AppPathFinder** → `appPathFinder.js` (uninstaller euristico multi-livello nel server web)
- **Boot Optimization** → operazione bash `op32`
- **Orphaned File Finder** → operazione bash `op33`
- **Discovery dinamica cache** e **HOMEBREW_CACHE personalizzato** → miglioramenti `op02`/`op26`

Vedi `SYNTHESIS.md` nel repository CleanMac per la matrice funzionale completa.

## Cosa cambia in PureMac (direzione reciproca)

Dove le due si sovrappongono, si adotta l'impostazione migliore. La whitelist di app
di sistema Apple era più completa in CleanMac (più utility protette). Perciò
`AppInfoFetcher.protectedBundleIDs` è stata estesa per allinearla:

- Aggiunti: Books, Dictionary, Automator, Script Editor, Disk Utility, Keychain Access,
  Font Book, Image Capture, Migration Assistant, Freeform, Voice Memos, Shortcuts.

Questo riduce il rischio che utility di sistema compaiano nell'elenco disinstallabili.
È una modifica solo-dati (nessun cambiamento di logica), compatibile con la build esistente.

## Funzionalità di PureMac superiori (mantenute qui)

- Motore euristico `AppPathFinder` a 10 livelli (bundle id, entitlements, team id, container).
- Scan engine con discovery dinamica delle cache (nessuna lista hardcoded).
- Rilevamento purgeable APFS via `URLResourceValues` (più preciso del parsing `diskutil`).
- Esperienza nativa SwiftUI (NavigationSplitView, Table, Form) e onboarding Full Disk Access.
