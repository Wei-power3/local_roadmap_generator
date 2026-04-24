# engineering spec

## system overview

A single self-contained `.html` file that runs entirely in the browser. The user uploads a CSV, the app parses it and renders a swimlane Gantt chart on an HTML Canvas element, and the user exports the chart as a PNG. No server. No network calls. No install.

---

## data model

```
Project
  name: string
  start_date: date
  end_date: date
  timelineScale: "month" | "quarter" | "year"

Dimension
  id: string
  name: string
  order: integer
  visible: boolean
  color: string (hex — from CSV override, UI swatch picker, or default palette)

Activity
  id: string
  dimensionId: string
  label: string
  startDate: date
  endDate: date
  type: "bar"

Marker
  id: string
  dimensionId: string
  label: string
  date: date
  type: "milestone" | "decision"
  notes: string (optional; \\n in CSV is converted to actual newline)

Config (runtime state)
  scale: "month" | "quarter" | "year"
  visibleDimensions: Set<string> (dimension ids)
  showActivities: boolean (default true)
  showMilestones: boolean (default true)
  showDecisions: boolean (default true)
```

---

## data flow

```
User opens index.html in browser
        ↓
Upload CSV (button click or file drop)
        ↓
Browser reads file via FileReader API (no server involved)
        ↓
Strip UTF-8 BOM if present; auto-detect delimiter (comma / semicolon / tab)
        ↓
Validation: check row types, dates, required fields
        ↓
  ┌─────────────────────┐
  │  Parse error?        │
  │  → show row-level    │
  │    error messages    │
  │  → block rendering   │
  └─────────────────────┘
        ↓ (clean parse)
Chart renders on HTML Canvas
        ↓
User adjusts config (scale toggle, visibility toggles, dimension color swatches)
        ↓
Canvas re-renders on each config change
        ↓
User clicks Export PNG
        ↓
Offscreen canvas drawn at 2× resolution (3840×2160) → canvas.toBlob() → file download
```

Everything in this flow happens in the browser. No data is transmitted anywhere.

---

## app state

```
EMPTY → CSV_UPLOADED → PARSE_ERROR (if invalid) → EMPTY
EMPTY → CSV_UPLOADED → CHART_RENDERED → CONFIG_ADJUSTED → PNG_EXPORTED
```

State transitions are driven by user actions only. There is no async processing, background jobs, or server round-trips.

---

## edge cases and failure handling

| scenario | impact | handling |
|---|---|---|
| CSV row has an invalid date format | Cannot place on timeline | Validate all dates before rendering; show "Row N: invalid date — expected YYYY-MM-DD" |
| Activity end date is before start date | Bar renders at zero width or backwards | Detect and surface as a validation error before rendering |
| Marker has no date | Cannot place on timeline | Flag as validation error: "Row N: marker requires a Start Date" |
| Dimension name in an activity row does not match any defined dimension | Activity silently disappears | Error: "Row N: dimension 'X' not found — must be defined earlier" |
| Timeline scale set to Month on a >3-year project | Chart is compressed | Auto-show warning banner: "Timeline spans over 3 years. Month scale may be compressed — consider Quarter or Year." |
| PNG export on a high-DPI (Retina) display | Export looks blurry when printed | Render offscreen canvas at 3840×2160 (2× scale) via `setTransform(2,0,0,2,0,0)` |
| CSV is empty or has only a header row | Nothing to render | Show: "No data found — check that your CSV has at least one activity row" |
| User uploads a non-CSV file | Parser fails | Show: "File must be a .csv" — do not attempt to parse |
| Two markers with dates within 40px of each other on timeline | Overlap | Second marker offset ±16px; direction chosen by which side has fewer placed markers; label drawn above or below accordingly |
| UTF-8 BOM at start of file | Corrupts first column header | Strip `﻿` before parsing |
| CSV uses semicolons or tabs instead of commas | All columns report as missing | Auto-detect delimiter from header row character counts |
| 5th dimension assigned default Tangerine color | Visually clashes with milestone markers | No automatic fix — user should assign a custom color via CSV `Color` column or UI swatch picker |
| 6th+ dimensions all default to Blue-Gray | Multiple dimensions look identical | Same workaround as above |

---

## security

This tool has no authentication surface, no backend, and no network calls. The security surface is effectively zero.

The one consideration: if the `.html` file is shared internally with colleagues, any data a colleague enters stays on their own machine. The file contains no tracking, no telemetry, and no external script dependencies.

---

## build order

Build in this sequence. Each step produces something testable before the next begins.

**V1**
1. **CSV parser** — define the column format, parse rows into typed objects, surface validation errors with row-level messages
2. **Chart renderer** — draw dimension headers, activity bars, and the timeline axis on an HTML Canvas
3. **Floating markers** — place milestone circles and decision markers at dimension level, pinned to date position
4. **Timeline scale toggle** — redraw the chart when the user switches between month, quarter, and year
5. **PNG export** — render offscreen canvas at 2× resolution and trigger a file download
6. **Swimlane visibility toggle** — hide/show individual dimensions and re-render
7. **Color configuration** — apply BIOTRONIK palette as default; read per-dimension color overrides from CSV
8. **Error and warning display** — show validation messages above the chart; show scale compression warning when applicable

**V2**
9. **Marker visual disambiguation** — milestone as Tangerine circle, decision as Yellow diamond; unified label positioning
10. **Canvas legend** — draw legend inside topMargin using Canvas 2D API; visibility-aware; background box
11. **Dimension header colors** — fill left column with dim.color instead of fixed BIOTRONIK Blue
12. **Adaptive activity label placement** — use measureText() to place label inside or outside bar
13. **Content visibility toggles** — Activities / Milestones / Decisions checkboxes; canvas updates instantly
14. **Smart marker collision** — 40px threshold, ±16px offset, smart directional logic, label above/below
15. **CSV compatibility** — strip BOM, auto-detect delimiter (comma/semicolon/tab)
16. **In-app color swatch picker** — 7 BIOTRONIK color families × 4 HSL shades per dimension; click to reassign

---

## definition of done (V2)

- [x] `index.html` opens in Chrome with no installation step
- [x] CSV upload works via button click and via file drop
- [x] Chart renders correctly: dimension headers in palette color, activity bars, milestone circles, decision diamonds, timeline axis
- [x] Canvas legend appears in browser view and in exported PNG
- [x] Month / Quarter / Year toggle redraws the chart without page reload
- [x] Swimlane visibility toggle hides and shows individual dimensions correctly
- [x] Activities / Milestones / Decisions toggles filter content and update legend
- [x] Color swatch picker reassigns dimension color instantly without CSV change
- [x] PNG export produces a 3840×2160px file (`gantt-export.png`) sharp in PowerPoint
- [x] Invalid CSV rows produce readable, row-specific error messages
- [x] App handles semicolon and tab-delimited CSV files and UTF-8 BOM without error
- [x] No network requests are made at any point
- [ ] A real project chart is completed from scratch in under 10 minutes *(validate on target machine)*
