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
  timeline_scale: "month" | "quarter" | "year"

Dimension
  id: string
  name: string
  order: integer
  visible: boolean
  color: string (hex, optional — falls back to BIOTRONIK palette default)

Activity
  id: string
  dimension_id: string
  label: string
  start_date: date
  end_date: date
  type: "bar"

Marker
  id: string
  dimension_id: string
  label: string
  date: date
  type: "milestone" | "decision"
  notes: string (optional)

Config
  timeline_scale: "month" | "quarter" | "year"
  visible_dimensions: string[] (list of dimension ids)
```

---

## data flow

```
User opens index.html in Chrome
        ↓
Upload CSV (button click or file drop)
        ↓
Browser parses CSV (no server involved)
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
Canvas drawn at 2× resolution → saved as PNG to downloads folder
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
| Dimension name in an activity row does not match any defined dimension | Activity silently disappears | Warn: "Row N: dimension 'X' not found — activity skipped" |
| Timeline scale set to Month on a 3-year project | Chart is unreadably compressed | Auto-suggest scale based on date range; show a warning in the UI |
| PNG export on a high-DPI (Retina) display | Export looks blurry when printed | Render canvas at devicePixelRatio × 1920 width, then scale down for display |
| CSV is empty or has only a header row | Nothing to render | Show: "No data found — check that your CSV has at least one activity row" |
| User uploads a non-CSV file | Parser fails | Show: "File must be a .csv" — do not attempt to parse |

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
5. **PNG export** — render at 2× resolution and trigger a file download
6. **Swimlane visibility toggle** — hide/show individual dimensions and re-render
7. **Color configuration** — apply BIOTRONIK palette as default; read per-dimension color overrides from CSV
8. **Error display** — show validation messages above the chart, list problem rows, do not render if errors are present

---

## definition of done (V1)

All of the following must pass before V1 is complete:

- [ ] `index.html` opens in Chrome on a locked-down work machine with no installation step
- [ ] CSV upload works via button click and via dragging the file onto the page
- [ ] Chart renders correctly: dimension headers, activity bars, milestone markers, decision markers, timeline axis
- [ ] Month / Quarter / Year toggle redraws the chart without page reload
- [ ] Swimlane visibility toggle hides and shows individual dimensions correctly
- [ ] PNG export produces a 1920×1080px file that is sharp when pasted into PowerPoint and viewed full-screen
- [ ] Invalid CSV rows produce a readable, row-specific error message — the app does not crash
- [ ] No network requests are made at any point (verify in Chrome DevTools → Network tab)
- [ ] A real project chart is completed from scratch, including data entry, in under 10 minutes
