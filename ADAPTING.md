# 🔧 Adapting GPS Field Logger to New Studies

GPS Field Logger is designed to work **without any code changes** for most studies — you define blocks and scenarios through the app's UI. This guide covers everything beyond that: removing GPS-specific features entirely, replacing them with other recording methods, creating new data categories, and building custom collection workflows.

---

## Contents

1. [No-Code Customisation (Built-In)](#1-no-code-customisation-built-in)
2. [Pre-loading Default Scenarios and Blocks](#2-pre-loading-default-scenarios-and-blocks)
3. [Pre-loading a Default Weather Location](#3-pre-loading-a-default-weather-location)
4. [Renaming the App for Your Study](#4-renaming-the-app-for-your-study)
5. [Removing GPS Receiver Tracking Entirely](#5-removing-gps-receiver-tracking-entirely)
6. [Replacing GPS Tracking with Other Recording Methods](#6-replacing-gps-tracking-with-other-recording-methods)
7. [Adding Custom Data Fields to Each Collection Point](#7-adding-custom-data-fields-to-each-collection-point)
8. [Creating New Data Categories Beyond Scenarios and Blocks](#8-creating-new-data-categories-beyond-scenarios-and-blocks)
9. [Changing the Randomisation Structure](#9-changing-the-randomisation-structure)
10. [Adjusting Re-collection Reasons](#10-adjusting-re-collection-reasons)
11. [Full Worked Examples](#11-full-worked-examples)
12. [Testing Your Changes](#12-testing-your-changes)
13. [Contributing Back](#13-contributing-back)

---

## 1. No-Code Customisation (Built-In)

At session start, you can define through the UI — no code editing needed:

- Any number of **scenarios** (observation or measurement types) with names, descriptions, and emoji icons
- Any number of **blocks** (spatial zones, treatment groups, transects, plots) with names and condition notes
- Collection settings: duration per point, travel time, antenna height (rename this to whatever unit makes sense for your study)

These are saved with the session and exported in both CSV and JSON.

**The app works for any structured field study**, not just GPS. The "GPS Receiver" section is the only part that is truly GPS-specific — everything else is general-purpose.

---

## 2. Pre-loading Default Scenarios and Blocks

If your team always uses the same scenarios or blocks, pre-populate them so field staff don't need to type them each session.

Open `gps-field-logger.html` in a text editor (VS Code recommended). Find the `startNewSession()` function and locate:

```javascript
scenarios: [],
blocks: [],
```

### Pre-loading scenarios
```javascript
scenarios: [
  { id:'CR', label:'Crop Row',      icon:'🌽', desc:'Center of plant row' },
  { id:'RM', label:'Row Midpoint',  icon:'↔',  desc:'Between two crop rows' },
  { id:'FE', label:'Field Edge',    icon:'🌿', desc:'Within 5 ft of field boundary' },
  { id:'DZ', label:'Drainage Zone', icon:'💧', desc:'Low-lying drainage area' },
],
```

### Pre-loading blocks
```javascript
blocks: [
  { id:'B1', label:'North Field', notes:'Full sun, well-drained loam' },
  { id:'B2', label:'South Field', notes:'Partial shade, heavier clay' },
  { id:'B3', label:'East Strip',  notes:'Sandy soil, drip irrigation' },
],
```

Users can still add or remove items during setup even with defaults loaded.

---

## 3. Pre-loading a Default Weather Location

Weather location is fully universal — no hardcoded coordinates exist. Users choose their site at session time via GPS, place-name search, or manual lat/lon.

To skip the prompt for a fixed-site team, auto-fetch weather in `startNewSession()`:

```javascript
// Add after: state.session = sanitiseSession({...});
fetchWeatherByCoords(YOUR_LAT, YOUR_LON, 'Your Site Name')
  .then(w => { state.session.weather = w; autoSave(); render(); });
```

To find your coordinates: Google Maps → right-click your site → copy the lat/lon shown.

---

## 4. Renaming the App for Your Study

In the `<head>` section:
```html
<title>Soil Sampling Logger v1.0</title>
```

In the `rHome()` function:
```javascript
<div class="app-title">Soil Sampling Logger</div>
<div class="app-sub">Watershed Nutrient Study — Spring 2026</div>
```

---

## 5. Removing GPS Receiver Tracking Entirely

If your study has nothing to do with GPS receivers — vegetation surveys, soil sampling, insect counts, water quality, crop scouting, phenology, yield estimation — remove the GPS receiver section completely.

### Step 1 — Remove the GPS Receivers UI

In `rSetup()`, delete everything between and including these two HTML comments:

```
<!-- GPS Receivers -->
   ... (all GPS card HTML) ...
<!-- Scenarios -->
```

Keep the `<!-- Scenarios -->` comment and everything after it.

### Step 2 — Remove the GPS condition from the "Begin" check

Find this line in `rSetup()`:
```javascript
const canBegin = gpsRx.length > 0 && scenarios.length > 0 && blocks.length > 0 && !locked && gpsRx.some(g=>g.rtk_fixed_at);
```

Replace with:
```javascript
const canBegin = scenarios.length > 0 && blocks.length > 0 && !locked;
```

Also simplify the hint text line just below it:
```javascript
// Change the hint to only mention what is still missing:
${!canBegin && !locked ? `<p class="hint-text">${scenarios.length===0 ? 'Add scenarios · ' : blocks.length===0 ? 'Add blocks · ' : ''}then Begin</p>` : ''}
```

### Step 3 — Remove GPS fields from the session object

In `startNewSession()`, remove:
```javascript
gps_receivers: [],
```

In `sanitiseSession()`, remove:
```javascript
s.gps_receivers = Array.isArray(s.gps_receivers) ? s.gps_receivers : [];
```

### Step 4 — Remove GPS columns from CSV export

In `exportCSV()`, delete the GPS receivers block:
```javascript
// Delete these lines entirely:
c += '\nGPS_Receivers\nName,Powered_On_UTC,RTK_Fixed_UTC,TTFF_s\n';
s.gps_receivers.forEach(g => { ... });
```

### Step 5 — Repurpose or remove the "GPS Height" setting

The `gps_height_m` setting can be repurposed for any numeric field relevant to your study, or removed. In `rSetup()`:

```javascript
// Before (GPS-specific):
<label class="setting-label">GPS Height (m) <input ...></label>

// After (soil sampling example):
<label class="setting-label">Sampling Depth (cm) <input ...></label>
```

Update the CSV metadata label from `GPS_Height_m` to your new field name.

---

## 6. Replacing GPS Tracking with Other Recording Methods

The GPS receiver section is essentially a **two-event timestamped logger**: power-on time, then fix time. You can repurpose this exact pattern — or replace it with something else entirely — for any instrument or process that needs event-based logging at the session level.

---

### Option A — Timed Observation Window (e.g. Bird Count, Insect Emergence, Phenology)

Replace each GPS card with a start/stop observation timer per observer or transect.

In `rSetup()`, replace the GPS Receivers section with:

```javascript
<!-- Observation Windows -->
<div class="section"><div class="section-title">🔭 Observation Windows</div>
  <div class="add-row">
    <input type="text" id="obs-name-input" class="input" placeholder="Observer name or transect ID">
    <button class="add-btn" onclick="addObserver()">+</button>
  </div>
  ${(s.observers||[]).map(o => `
    <div class="gps-card">
      <div class="gps-header"><span class="gps-name">${o.name}</span></div>
      <div class="gps-actions">
        <button class="gps-btn ${o.start_at ? 'powered' : ''}" onclick="logObsStart('${o.id}')">
          ▶ ${o.start_at ? 'Started ' + fmt(o.start_at) : 'Start Observation'}
        </button>
        <button class="gps-btn ${o.end_at ? 'rtk' : ''}" onclick="logObsEnd('${o.id}')"
          ${!o.start_at ? 'disabled' : ''}>
          ⏹ ${o.end_at ? 'Ended ' + fmt(o.end_at) : 'End Observation'}
        </button>
      </div>
    </div>`).join('')}
</div>
```

Add these functions near the other GPS functions:

```javascript
function addObserver() {
  const el = document.getElementById('obs-name-input'), n = el?.value?.trim();
  if (!n) return;
  US(s => ({...s, observers: [...(s.observers||[]), {id:genId(), name:n, start_at:null, end_at:null}]}));
}
function logObsStart(id) {
  US(s => ({...s, observers: (s.observers||[]).map(o => o.id===id ? {...o, start_at:new Date().toISOString()} : o)}));
}
function logObsEnd(id) {
  US(s => ({...s, observers: (s.observers||[]).map(o => o.id===id ? {...o, end_at:new Date().toISOString()} : o)}));
}
```

Update `sanitiseSession()` to include `s.observers = s.observers || [];`

---

### Option B — Instrument Power / Calibration Log (e.g. Spectroradiometer, Soil Probe, Water Quality Meter)

Keep the exact two-event card structure but rename the events to match your instrument's workflow. In `rSetup()`:

```javascript
// Rename labels:
'⏻ Power On'   →  '⚡ Sensor On'
'🎯 RTK Fixed' →  '✓ Calibrated'

// Rename the TTFF display:
'TTFF → RTK:'  →  'Warm-up time:'
```

Update the CSV export column headers:
```javascript
// Change:
'Name,Powered_On_UTC,RTK_Fixed_UTC,TTFF_s'
// To:
'Instrument,Power_On_UTC,Calibrated_UTC,Warmup_s'
```

---

### Option C — Count Tally per Point (e.g. Plant Stand Count, Pest Scouting, Animal Survey)

Add a large tap-friendly counter to the collection card that the observer can increment or decrement at each point.

In `rLocs()`, find the section with the START / END buttons and add below:

```javascript
${c.status === 'collecting' ? `
  <div style="margin-top:12px;text-align:center">
    <div style="font-size:13px;font-weight:700;color:#666;margin-bottom:6px">COUNT</div>
    <div style="display:flex;align-items:center;justify-content:center;gap:16px">
      <button onclick="adjustCount(${i}, -1)"
        style="width:52px;height:52px;font-size:28px;background:#fff;
               border:3px solid #000;border-radius:12px;cursor:pointer;font-weight:900">−</button>
      <span style="font-size:40px;font-weight:900;min-width:60px;text-align:center">
        ${c.count || 0}
      </span>
      <button onclick="adjustCount(${i}, 1)"
        style="width:52px;height:52px;font-size:28px;background:#0055ff;color:#fff;
               border:3px solid #003399;border-radius:12px;cursor:pointer;font-weight:900">+</button>
    </div>
  </div>` : ''}
```

Add the function:
```javascript
function adjustCount(i, delta) {
  US(s => {
    const locs = [...s.locations];
    locs[i] = {...locs[i], count: Math.max(0, (locs[i].count || 0) + delta)};
    return {...s, locations: locs};
  });
}
```

Add `Count` to the CSV export header and row data.

---

### Option D — Scored Rating Scale (e.g. Crop Damage 0–5, Weed Pressure, Visual Assessment)

Replace the timer workflow with a tap-to-select rating that immediately records and advances to the next point — no start/stop needed.

In `rLocs()`, replace the START / END buttons with:

```javascript
${c.status === 'pending' ? `
  <div style="margin-top:12px">
    <div style="font-size:13px;font-weight:700;color:#444;margin-bottom:10px">DAMAGE SCORE (0–5)</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px">
      ${[0,1,2,3,4,5].map(v => `
        <button onclick="setScore(${i}, ${v})"
          style="flex:1;min-width:52px;padding:14px 0;font-size:20px;font-weight:900;
                 background:${c.score===v ? '#0055ff' : '#eee'};
                 color:${c.score===v ? '#fff' : '#000'};
                 border:3px solid ${c.score===v ? '#003399' : '#ccc'};
                 border-radius:10px;cursor:pointer">${v}</button>`).join('')}
    </div>
    ${c.score !== undefined ? `
      <button onclick="confirmScore(${i})"
        style="display:block;width:100%;margin-top:12px;padding:14px;
               background:#00aa00;color:#fff;border:none;border-radius:10px;
               font-size:16px;font-weight:900;cursor:pointer;font-family:inherit">
        ✓ Record Score ${c.score} and Continue
      </button>` : ''}
  </div>` : ''}
```

Add functions:
```javascript
function setScore(i, v) {
  US(s => { const l = [...s.locations]; l[i] = {...l[i], score:v}; return {...s, locations:l}; });
}
function confirmScore(i) {
  US(s => {
    const l = [...s.locations];
    const now = new Date().toISOString();
    l[i] = {...l[i], status:'completed', start_time:now, end_time:now};
    const n = i+1 < l.length ? i+1 : i;
    return {...s, locations:l, current_location_index:n};
  });
}
```

Add `Score` to the CSV export.

---

### Option E — Dropdown Selection per Point (e.g. Soil Texture Class, Growth Stage, Cover Type, Phenological Stage)

Add a labelled dropdown to the collection card for selecting from a fixed list of categories:

```javascript
${c.status === 'collecting' ? `
  <div style="margin-top:12px">
    <label style="font-size:13px;font-weight:700;color:#444;display:block;margin-bottom:6px">
      GROWTH STAGE
    </label>
    <select onchange="setDropdown(${i}, 'growth_stage', this.value)"
      style="width:100%;padding:12px;font-size:15px;font-weight:700;
             border:3px solid #888;border-radius:10px;background:#fff;font-family:inherit">
      <option value="">— select —</option>
      <option value="V2" ${c.growth_stage==='V2'?'selected':''}>V2  — 2nd leaf collar</option>
      <option value="V6" ${c.growth_stage==='V6'?'selected':''}>V6  — 6th leaf collar</option>
      <option value="VT" ${c.growth_stage==='VT'?'selected':''}>VT  — Tasseling</option>
      <option value="R1" ${c.growth_stage==='R1'?'selected':''}>R1  — Silking</option>
      <option value="R6" ${c.growth_stage==='R6'?'selected':''}>R6  — Physiological maturity</option>
    </select>
  </div>` : ''}
```

Add the generic function (works for any dropdown field):
```javascript
function setDropdown(i, fieldName, value) {
  US(s => { const l = [...s.locations]; l[i] = {...l[i], [fieldName]:value}; return {...s, locations:l}; });
}
```

Add the column to CSV export. You can stack multiple dropdowns per point by calling `setDropdown()` with different `fieldName` values.

---

## 7. Adding Custom Data Fields to Each Collection Point

Any per-point data field follows the same three-step pattern: **UI input → state update → CSV export**.

### Step 1 — Add the input in `rLocs()`

Place it inside the collection card, after the stop button:

```javascript
// Free-text note
`<input type="text" class="input" placeholder="Observer note"
  style="margin-top:8px"
  onchange="state.session.locations[${i}].obs_note = this.value; autoSave()"
  value="${c.obs_note || ''}">`

// Numeric measurement  
`<input type="number" class="input" placeholder="Canopy cover (%)"
  style="margin-top:8px"
  onchange="state.session.locations[${i}].canopy_pct = this.value; autoSave()"
  value="${c.canopy_pct || ''}">`
```

### Step 2 — Add the column in `exportCSV()`

In the header string:
```javascript
let c = 'Point,...,Recollect_Reason,Obs_Note,Canopy_Pct\n';
```

In each row:
```javascript
c += `...,${l.recollect_count||0},"${(l.recollect_reason||'').replace(/"/g,'""')}","${(l.obs_note||'').replace(/"/g,'""')}",${l.canopy_pct||''}\n`;
```

### Step 3 — Verify in a test export

Create a test session, fill in your new fields at a few points, export CSV, and confirm the columns appear with the correct values.

---

## 8. Creating New Data Categories Beyond Scenarios and Blocks

The built-in structure is **Blocks × Scenarios**, which covers most factorial field designs. For more complex designs, you can add a third dimension or replace the category system entirely.

---

### Adding a Third Category (e.g. Depth Layer, Replicate, Time Period)

The cleanest approach is to expand each scenario into multiple sub-entries inside `generateSchedule()`.

For a soil study with three sampling depths:

```javascript
// In startNewSession(), add to settings:
depth_layers: ['0-15cm', '15-30cm', '30-60cm'],

// In generateSchedule(), before the main loop, expand scenarios:
const expandedScenarios = [];
scenarios.forEach(sc => {
  (settings.depth_layers || ['']).forEach(depth => {
    expandedScenarios.push({
      ...sc,
      id:    sc.id + (depth ? '_' + depth : ''),
      label: sc.label + (depth ? ' ' + depth : ''),
      desc:  (sc.desc || '') + (depth ? ' — ' + depth : ''),
    });
  });
});
// Then replace 'scenarios' with 'expandedScenarios' in the schedule loop
```

This generates one point per **Block × Scenario × Depth** without changing any other part of the app.

---

### Replacing Scenarios with a Per-Point Checklist (e.g. Multi-Attribute Survey)

For surveys where each point requires recording multiple attributes simultaneously — rather than visiting one scenario type per stop — replace the single scenario card with a checklist rendered from a `checklist_items` array stored in `settings`.

**Define the checklist in `startNewSession()`:**
```javascript
settings: {
  static_duration_min: 5,
  travel_setup_min: 3,
  gps_height_m: 1.5,
  random_seed: null,
  checklist_items: [
    { id:'weed',    label:'Weed presence',     type:'boolean'   },
    { id:'pest',    label:'Pest damage',        type:'boolean'   },
    { id:'stand',   label:'Stand density (1-5)',type:'score_1_5' },
    { id:'height',  label:'Plant height (cm)',  type:'number'    },
  ],
},
```

**Render the checklist in `rLocs()`:**
```javascript
${c.status === 'collecting' ? `
  <div style="margin-top:12px">
    ${(s.settings.checklist_items||[]).map(item => `
      <div style="display:flex;justify-content:space-between;align-items:center;
                  padding:10px 12px;background:#fff;border-radius:8px;
                  border:2px solid #ccc;margin-bottom:6px">
        <span style="font-weight:700;font-size:14px">${item.label}</span>
        ${item.type === 'boolean' ? `
          <input type="checkbox"
            onchange="setChecklist(${i}, '${item.id}', this.checked)"
            ${(c.checklist||{})[item.id] ? 'checked' : ''}
            style="width:24px;height:24px;cursor:pointer">` : ''}
        ${item.type === 'number' ? `
          <input type="number" style="width:80px;padding:6px 8px;font-size:14px;
                  font-weight:700;border:2px solid #888;border-radius:8px;text-align:center"
            onchange="setChecklist(${i}, '${item.id}', parseFloat(this.value))"
            value="${(c.checklist||{})[item.id] || ''}">` : ''}
        ${item.type === 'score_1_5' ? `
          <select onchange="setChecklist(${i}, '${item.id}', parseInt(this.value))"
            style="padding:8px;font-size:14px;font-weight:700;
                   border:2px solid #888;border-radius:8px">
            <option value="">-</option>
            ${[1,2,3,4,5].map(v => `
              <option value="${v}" ${(c.checklist||{})[item.id]===v ? 'selected':''}>${v}</option>
            `).join('')}
          </select>` : ''}
      </div>`).join('')}
  </div>` : ''}
```

**Add the function:**
```javascript
function setChecklist(i, key, value) {
  US(s => {
    const locs = [...s.locations];
    locs[i] = {...locs[i], checklist: {...(locs[i].checklist||{}), [key]: value}};
    return {...s, locations: locs};
  });
}
```

**Flatten the checklist in `exportCSV()`:**
```javascript
const items = s.settings.checklist_items || [];
// Header — add item labels as columns:
c = 'Point,...,Recollect_Reason,' + items.map(it => it.label.replace(/[,"]/g,'_')).join(',') + '\n';
// Each row — append checklist values:
c += `...,"${(l.recollect_reason||'')}",` + items.map(it => (l.checklist||{})[it.id] ?? '').join(',') + '\n';
```

---

## 9. Changing the Randomisation Structure

The default creates one point per block per scenario, with block order and scenario-within-block order both randomised. Find `generateSchedule()` to modify.

**Multiple replicates per block** — wrap the existing schedule loop in `for (let rep = 1; rep <= numReps; rep++)` and append `_R${rep}` to the point ID.

**Fixed (non-random) visit order** — remove the Fisher-Yates shuffle calls from `generateSchedule()` so points are always visited in definition order.

**Latin Square** — build an N×N grid where each scenario appears exactly once per row-block and once per column-block. Useful when both rows and columns represent a spatial or treatment gradient.

**Stratified sampling** — call `generateSchedule()` separately for each stratum with a different seed, then concatenate the resulting arrays before returning.

The seeded RNG (`mulberry32`) guarantees reproducibility — the same seed always produces the same schedule. Seeds are recorded in both CSV and JSON exports.

---

## 10. Adjusting Re-collection Reasons

Find the `REASONS` array inside `rLocs()`:

```javascript
const REASONS = [
  'Equipment issue / receiver lost fix',
  'Tripod bumped or disturbed',
  'Wrong location measured',
  'Obstructed by vehicle or person',
  'Correction link dropped',
  'Weather interference',
  'Other (see field notes)',
];
```

Replace with reasons relevant to your study. For a vegetation survey:

```javascript
const REASONS = [
  'Plot boundary incorrectly located',
  'Quadrat disturbed by livestock or wildlife',
  'Incorrect subplot selected',
  'Observer identification error',
  'Equipment malfunction',
  'Other (see field notes)',
];
```

For crop scouting:

```javascript
const REASONS = [
  'Row boundary unclear',
  'Plot partially harvested or damaged',
  'Incorrect row counted',
  'Equipment malfunction (densiometer, handheld sensor)',
  'Other (see field notes)',
];
```

---

## 11. Full Worked Examples

### Example A — Crop Scouting App (No GPS, Count Tally + Dropdown)

**Goal:** Walk crop rows, record pest count and growth stage at each plot. No GPS receivers.

**Changes needed:**
1. Remove GPS Receivers section and update `canBegin` (Section 5)
2. Rename `GPS Height` setting to `Row spacing (m)`
3. Add count tally to collection card (Section 6, Option C)
4. Add growth stage dropdown (Section 6, Option E)
5. Pre-load scenarios: `Aphids`, `Spider Mites`, `Corn Borer`, `Earworm` (Section 2)
6. Rename app to "Crop Scout Logger" (Section 4)
7. Update re-collection reasons to scouting context (Section 10)
8. Add `Count`, `Growth_Stage` columns to CSV export (Section 7)

**Result:** A dedicated scouting tool — a 40-acre field with 4 blocks × 4 scenarios takes roughly 15 minutes per session.

---

### Example B — Soil Sampling App (No GPS, Depth Layers + Texture Notes)

**Goal:** Collect soil at multiple depths per plot; record texture class and visible features.

**Changes needed:**
1. Remove GPS Receivers section (Section 5)
2. Rename `GPS Height` setting to `Core length (cm)`
3. Add `depth_layers` to settings and expand in `generateSchedule()` (Section 8)
4. Add soil texture class dropdown (Section 6, Option E)
5. Add free-text "Visual features" field (Section 7)
6. Rename app to "Soil Core Logger" (Section 4)
7. Adjust re-collection reasons to soil context (Section 10)

---

### Example C — Vegetation Survey App (Multi-Attribute Checklist)

**Goal:** At each quadrat record canopy cover %, litter cover %, bare ground %, weed presence, and observer notes — all in a single stop.

**Changes needed:**
1. Remove GPS Receivers section (Section 5)
2. Replace scenario-per-point with a per-point checklist (Section 8)
3. Define `checklist_items` in settings with types `number`, `boolean`, `score_1_5` as needed
4. Add `obs_note` free-text field (Section 7)
5. Export all checklist attributes as flattened CSV columns (Section 8)
6. Rename app to "Quadrat Survey Logger" (Section 4)
7. Rename `GPS Height` to `Quadrat area (m²)` (Section 5)

---

### Example D — Phenology Observation App (Timed Windows + Rating)

**Goal:** Multiple observers each watch a fixed plot for 10 minutes; record flowering intensity (0–4) per observation window per species.

**Changes needed:**
1. Replace GPS Receivers with timed observation windows per observer (Section 6, Option A)
2. Replace START / END collection with a scored rating (Section 6, Option D, scale 0–4)
3. Pre-load scenarios as species names: `Corn`, `Soybean`, `Cover Crop Mix`
4. Blocks = transects or spatial zones
5. Rename app to "Phenology Logger" (Section 4)

---

## 12. Testing Your Changes

1. Open `gps-field-logger.html` in Chrome on your desktop
2. Press **F12 → device toolbar** to simulate a phone screen (375×812 for iPhone)
3. Create a test session and walk through the complete workflow end to end
4. Export CSV and open in Excel or Google Sheets — verify all columns appear with the correct values
5. Reload the page and confirm the session restores correctly from local storage
6. Test edge cases: zero count, unselected dropdown, empty checklist items
7. Test on an actual phone browser before deploying to the field

---

## 13. Contributing Back

If you adapt this tool for a new study type, consider sharing it:
- Fork the repo and document your adaptation in your fork's README
- Open a GitHub Issue with **"Adaptation: [Study Type]"** so others can find it
- Submit a Pull Request if your changes would benefit all users
