# GPS Field Logger

[![Hits](https://hits.dwyl.com/ShaswatiB/gps-field-logger.svg)](https://hits.dwyl.com/ShaswatiB/gps-field-logger)
A **mobile-first, offline-capable** field data collection tool — built as a single HTML file. No installation, no server, no internet required in the field.

Fully configurable: you define your own **blocks** and **measurement scenarios** at the start of each session, making it adaptable to **any field study** — GPS accuracy, soil sampling, vegetation surveys, agrivoltaics, irrigation, precision agriculture, and more.

> **⭐ If this tool helped your research, please star this repo!**

---

## Quick Start

1. **Download** `gps-field-logger.html`
2. **Open** it in any mobile browser (Chrome, Safari, Firefox)
3. **No setup needed** — works offline, saves to your browser's local storage
4. Export your data as **CSV or JSON** when done

Or use the live hosted version:
```
https://ShaswatiB.github.io/gps-field-logger/gps-field-logger.html
```

---

## What It Does

| Feature | Description |
|---|---|
| **User-defined scenarios** | Type in your own measurement types at session start — no code editing needed |
| **User-defined blocks** | Define your own spatial/treatment blocks — any names, any number |
| **RCBD schedule generation** | Randomized Complete Block Design with seeded RNG for reproducibility |
| **GPS receiver tracking** | Log power-on time, RTK fix time, and TTFF per receiver |
| **Weather logging** | Auto-fetch weather for any location worldwide — via device GPS, place-name search, or manual coordinates |
| **Point-by-point collection** | Guided workflow with start/stop timers per measurement point |
| **Re-collection support** | Flag and re-collect any point with a documented reason |
| **Export** | Download session data as CSV or JSON |
| **Session history** | Browse and re-export past sessions |
| **Offline-capable** | Works without internet once loaded; saves to device storage |

---

## How to Adapt to Your Study

No code editing required! At the start of each session:

1. **Add your scenarios** — e.g. "Under Panel", "Open Sky", "Near Fence" for a GPS study; or "0–15 cm", "15–30 cm", "30–60 cm" for soil sampling
2. **Add your blocks** — e.g. "North Field", "South Field"; or "Block A", "Block B", "Block C"
3. The app generates a randomized RCBD schedule from your inputs automatically

For researchers who want to **pre-set** scenarios/blocks or add custom fields, see [`ADAPTING.md`](ADAPTING.md).

---

## Data Export Format

### CSV columns
`Point, Block, Block_Label, Block_Notes, Visit_Order, Point_ID, Scenario, Scenario_Desc, Start_Local, Start_UTC, End_Local, End_UTC, Duration_s, Recollect_Count, Recollect_Reason`

Plus separate sections for GPS receivers, scenario definitions, block definitions, and session metadata.

### JSON
Full session object including all metadata, weather, GPS receiver logs, scenario/block definitions, and all location records.


---

## Repository Structure

```
gps-field-logger/
├── gps-field-logger.html     ← The entire app (single file, v2.1)
├── README.md                 ← This file
├── USAGE_GUIDE.md            ← Step-by-step field operator instructions
├── ADAPTING.md               ← Guide for pre-configuring or extending the tool
├── CHANGELOG.md              ← Version history
├── GITHUB_SETUP_GUIDE.md     ← Full GitHub walkthrough for first-time users
└── LICENSE                   ← MIT License
```

---

## 📄 Citation

If you use this tool in published research, please cite:

```
[Your Name] (2025). GPS Field Logger [Software].
GitHub. https://github.com/ShaswatiB/gps-field-logger
```

---

## Contact

Questions or suggestions? Open a [GitHub Issue](../../issues).

---

## 📝 License

MIT License — free to use, modify, and distribute with attribution.
