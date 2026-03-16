# Changelog

All notable changes to GPS Field Logger are documented here.
Format follows [Semantic Versioning](https://semver.org/): Major.Minor.Patch

---

## [2.1.0] — 2025

### Changed
- **Weather location is now fully universal** — the hardcoded "Lincoln, NE" default has been removed. Users now choose their field site at session time via three options: (1) device GPS, (2) place-name search powered by the Open-Meteo geocoding API with a disambiguation picker for multiple matches, or (3) manual lat/lon entry. The app works correctly from any location in the world.

### Fixed
- `TypeError: Cannot read properties of undefined (reading 'length')` on load when a session was saved without `scenarios`, `blocks`, `locations`, or `gps_receivers` fields (old session format compatibility)
- `current_location_index` out-of-range crash in the Collect tab after loading a saved session
- All session state is now sanitised on load, on new session creation, and after every state update

---

## [2.0.0] — 2025

### Changed — Breaking
- **Scenarios are now fully user-defined** via the app UI. The hardcoded agrivoltaic solar farm scenarios (Perimeter, Between Rows, Under Panel Edge, etc.) have been removed. Users type in their own scenario names, descriptions, and icons at session start.
- **Blocks are now fully user-defined** via the app UI. The hardcoded solar farm block structure (NW/NE/SW/SE with row numbers and N-S distances) has been removed. Users define any number of blocks with any names.
- **N-S distance and row number fields removed** from all data structures, exports, and UI. Point IDs are now `BlockID_ScenarioID`.
- Schedule generation now works off user-defined blocks and scenarios (any count), replacing the fixed 4×6 structure.

### Added
- Emoji icon picker for scenarios (30 options)
- Optional description field for scenarios
- Optional notes field for blocks
- Dynamic hint text on Setup tab showing exactly what is still needed before collection can begin
- Re-shuffle button for schedule preview regenerates with a new seed
- Block notes displayed in the collection point card during data collection

### Improved
- Setup tab hint is now specific: tells you whether you need to add receivers, scenarios, blocks, or achieve RTK fix
- Past session viewer now shows scenario and block definitions for the session

---

## [1.1.0] — 2025

### Added
- Version number displayed in app footer
- GitHub and Help links on home screen
- Session naming tip in placeholder text
- Export reminder warning on home screen

---

## [1.0.0] — 2025

### Initial Release
- Single-file HTML app, no dependencies or server required
- Session management with auto-save to browser local storage
- GPS receiver tracking: power-on and RTK fix timestamps, TTFF calculation
- Weather auto-fetch via Open-Meteo API
- RCBD schedule generation with seeded RNG (4 blocks × 6 scenarios)
- Hardcoded agrivoltaic solar farm study design (NW/NE/SW/SE blocks, 6 GPS scenarios)
- Point-by-point guided collection with start/stop timers
- Re-collection workflow with reason logging
- Export to CSV and JSON
- Session history with re-export
- Mobile-optimised UI (iOS Safari and Android Chrome)

---

## How to Add a Version Entry

```markdown
## [X.Y.Z] — YYYY-MM-DD

### Added
- New feature description

### Changed
- What was modified and why

### Fixed
- Bug that was resolved
```
