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
  color: string (hex — either from CSV override or assigned from default palette)

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
```

---

## data flow

```
User opens index.html in Chrome
        ↓
Upload CSV (button click or file drop)
        ↓
Browser reads file via FileReader API (no server involved)
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
User adjusts config (scale toggle, visibility toggles)
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
| CSV row has an invalid date format | Chart renders incorrectly or crashes | Validate all dates before rendering; show "Row N: invalid date — expected YYYY-MM-DD" |
| Activity end date is before start date | Bar renders at zero width or backwards | Detect and surface as a validation error before rendering |
| Marker has no date | Cannot place on timeline | Flag as validation error: "Row N: marker requires a Start Date" |
| Dimension name in an activity row does not match any defined dimension | Activity silently disappears | Error: "Row N: dimension 'X' not found — must be defined earlier" |
| Timeline scale set to Month on a >3-year project | Chart is compressed | Auto-show warning banner: "Timeline spans over 3 years. Month scale may be compressed — consider Quarter or Year." |
| PNG export on a high-DPI (Retina) display | Export looks blurry when printed | Render offscreen canvas at 3840×2160 (2× scale) via `setTransform(2,0,0,2,0,0)` |
| CSV is empty or has only a header row | Nothing to render | Show: "No data found — check that your CSV has at least one activity row" |
| User uploads a non-CSV file | Parser fails | Show: "File must be a .csv" — do not attempt to parse |
| Two markers with dates <24px apart on timeline | Overlap | Second marker offset 8px downward |

---

## security

This tool has no authentication surface, no backend, and no network calls. The security surface is effectively zero.

The one consideration: if the `.html` file is shared internally with colleagues, any data a colleague enters stays on their own machine. The file contains no tracking, no telemetry, and no external script dependencies.

---

## build order

Build in this sequence. Each step produces something testable before the next begins.

1. **CSV parser** — define the column format, parse rows into typed objects, surface validation errors with row-level messages
2. **Chart renderer** — draw dimension headers, activity bars, and the timeline axis on an HTML Canvas
3. **Floating markers** — place milestone circles and decision text boxes at dimension level, pinned to date position
4. **Timeline scale toggle** — redraw the chart when the user switches between month, quarter, and year
5. **PNG export** — render offscreen canvas at 2× resolution and trigger a file download
6. **Swimlane visibility toggle** — hide/show individual dimensions and re-render
7. **Color configuration** — apply BIOTRONIK palette as default; read per-dimension color overrides from CSV
8. **Error and warning display** — show validation messages above the chart; show scale compression warning when applicable

---

## definition of done (V1)

All of the following must pass before V1 is complete:

- [x] `index.html` opens in Chrome on a locked-down work machine with no installation step
- [x] CSV upload works via button click and via dragging the file onto the page
- [x] Chart renders correctly: dimension headers, activity bars, milestone markers, decision markers, timeline axis
- [x] Month / Quarter / Year toggle redraws the chart without page reload
- [x] Swimlane visibility toggle hides and shows individual dimensions correctly
- [x] PNG export produces a 3840×2160px file (`gantt-export.png`) that is sharp when pasted into PowerPoint and viewed full-screen
- [x] Invalid CSV rows produce a readable, row-specific error message — the app does not crash
- [x] No network requests are made at any point (verify in Chrome DevTools → Network tab)
- [ ] A real project chart is completed from scratch, including data entry, in under 10 minutes *(to be validated on target machine)*
