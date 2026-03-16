# 📱 Field Operator Usage Guide
## GPS Field Logger v2.0 — Step-by-Step Instructions

This guide is for **field team members** collecting data. No technical background needed.

---

## Before You Go to the Field

### 1. Get the App on Your Phone
1. Open this link in your phone's browser:
   `https://ShaswatiB.github.io/gps-field-logger/gps-field-logger.html`
2. **Save it to your home screen** for offline use:
   - **iPhone/iPad:** Tap the Share button (box with arrow) → "Add to Home Screen"
   - **Android:** Tap the three-dot menu → "Add to Home Screen" or "Install App"
3. The app now works **without internet** in the field

### 2. Charge Your Phone
The app keeps the screen active during collection — bring a power bank for long sessions.

---

## Starting a Session

1. Open the app on your phone
2. On the home screen, **type a session name**
   - Recommended format: `YYYY-MM-DD_SiteName_AM` (e.g. `2025-06-15_NorthField_AM`)
3. Tap **"+ New Session"**

---

## Setup Tab (Setup)

Complete **all steps** before beginning data collection:

---

### Step 1 — Fetch Weather (optional)
Weather is recorded once per session and included in your data export. Three options are available:

- **📡 Use My Current Location (GPS)** — tap to auto-detect your device's location. You will be prompted to allow location access; this works anywhere in the world.
- **Search by place name** — type any city, region, or country (e.g. "Ames Iowa", "Punjab India", "Nairobi Kenya") and tap **Search**. If multiple matches are found, a dropdown lets you pick the exact location.
- **Enter coordinates manually** — expand the "Enter coordinates manually" section and type latitude/longitude directly. Useful when you already have site coordinates from a GPS unit or map.

> 💡 Skip this step entirely if you have no internet signal in the field — weather is optional.

---

### Step 2 — Log GPS Receivers
1. Type your receiver's name (e.g. "Swift Duro #1") and tap **+**
2. When you physically **power on** the receiver, tap **"Power On"** — timestamps the power-on event
3. When the receiver shows **RTK Fixed** on its display, tap **"RTK Fixed"** — logs the Time-To-First-Fix (TTFF)

> 💡 TTFF is automatically calculated and exported. It tells you how long the receiver took to achieve RTK lock from cold start.

---

### Step 3 — Add Your Measurement Scenarios
Scenarios are the **types of points** you will collect.

1. Type the scenario name (e.g. "Under Canopy", "Open Sky", "Near Post")
2. Optionally add a short description
3. Pick an emoji icon from the selector
4. Tap **"+ Add Scenario"**
5. Repeat for all scenario types in your study

**Examples by study type:**

| Study | Example Scenarios |
|---|---|
| GPS accuracy (agrivoltaic) | Under Panel Center, Between Rows, Panel Edge, Near Post |
| Soil sampling | 0–15 cm depth, 15–30 cm depth, 30–60 cm depth |
| Vegetation survey | Canopy cover, Ground cover, Bare soil, Litter |
| Irrigation | Emitter zone, Mid-row, Row end, Control |

---

### Step 4 — Add Your Study Blocks
Blocks are the **spatial or treatment divisions** of your field.

1. Type the block name (e.g. "North Field", "Zone A", "Block 1")
2. Optionally add notes about that block's conditions
3. Tap **"+ Add Block"**
4. Repeat for all blocks

**Examples:**
- Field quadrants: North, South, East, West
- Treatment groups: High Input, Low Input, Control
- Transects: Transect 1, Transect 2, Transect 3

> 💡 The app will create **one point per block per scenario** (e.g. 4 blocks × 6 scenarios = 24 total points), randomised in a balanced order.

---

### Step 5 — Review Collection Settings
- **Static Duration** — minutes you occupy each point (default: 5 min)
- **Travel+Setup** — travel time between points (default: 3 min)
- **GPS Height** — antenna height above ground in meters
- The **time estimate** shows expected total session duration

---

### Step 6 — Review the Schedule Preview
The dark panel shows the randomised point order for your session. Each session uses a different random order to prevent time-of-day bias across blocks. Tap **↻ Re-shuffle** if you want a different order.

---

### Step 7 — Add Field Notes
Record observer names, equipment serial numbers, anomalies, or anything relevant.

---

### Step 8 — Begin Collection
Once GPS receivers show RTK Fixed and you have scenarios + blocks defined, the green **"BEGIN DATA COLLECTION"** button will appear. Tap it to start.

---

## Collect Tab (Collect)

### Working Through Points

Each point card shows:
- **Scenario type** (e.g. "Under Panel Center")
- **Block name** (e.g. "Block NW")
- **Visit order** within the block
- **Point ID** (block + scenario code)
- **Block notes** (conditions you entered during setup)

To collect a point:
1. **Navigate** to the location for this block/scenario combination
2. **Set up** your equipment
3. Tap **▶ START COLLECTION** — starts the timer
4. Wait for the configured static duration (timer counts up on screen)
5. Tap **⏹ END COLLECTION** when done
6. The app automatically advances to the next point

### If You Need to Re-Collect a Point
Tap **RE-COLLECT THIS POINT** on any completed point, select the reason. The re-collection is logged and the original timestamps are preserved.

### Progress Bar
The bar at the top shows how many points are done out of total.

---

## Exporting Your Data

When all points are complete, two export buttons appear:

- **JSON** — Full data export including all metadata (recommended for archiving)
- **CSV** — Spreadsheet-friendly format (open directly in Excel or Google Sheets)

**Always export both formats** and transfer to your computer the same day.

---

## Viewing Past Sessions

Tap **History** to see all sessions stored on this device. You can re-export any past session at any time.

> ⚠️ **Important:** Session data is stored in your phone's browser storage. Clearing browser data will erase sessions. Always export before clearing.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| App won't load | Check internet; once loaded it works offline |
| Weather fetch fails | Try a different option: GPS → place name search → manual coordinates. Or skip weather entirely. |
| GPS button greyed out | Power on the receiver first, then log RTK |
| Data seems lost | Check History tab — data auto-saves every 300ms |
| "Begin" button not showing | Check hint text below it — it says exactly what's missing |
| Phone screen goes dark | Increase screen timeout in phone Settings |

---

## Quick Reference

```
SESSION NAME:   YYYY-MM-DD_SiteName_AM/PM
TOTAL POINTS:   (# of blocks) × (# of scenarios)
POINT ID:       BlockID_ScenarioID
EXPORTS:        JSON (archive) + CSV (analysis)
BACKUP RULE:    Export after EVERY session before leaving the field
```
