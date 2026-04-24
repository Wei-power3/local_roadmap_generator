# developer handover guide

## what this tool is

A single self-contained HTML file (`src/index.html`) that runs in a browser with no server, no install, and no internet connection. The user uploads a CSV, the tool renders a swimlane Gantt chart on an HTML Canvas, and exports it as a PNG.

There is no backend. There is no build step. There is no framework required. The entire deliverable is one file.

---

## repository structure

```
/
├── README.md
├── docs/
│   ├── user-story.md                            — why this exists, who uses it
│   ├── product-plan.md                          — product decisions, scope
│   ├── engineering-spec.md                      — data model, data flow, edge cases, build order
│   ├── csv-format.md                            — CSV column spec, row types, validation rules
│   ├── visual-design-spec.md                    — every layout, color, typography, and sizing decision
│   ├── enhancement-plan-markers-labels-toggles.md — V2 design decisions (implemented)
│   ├── release-notes-v2.md                      — summary of all V2 changes
│   ├── user-guide.md                            — end-user instructions
│   └── developer-handover.md                   — this file
├── src/
│   └── index.html                              — the complete tool
└── examples/
    └── sample.csv                              — working test data
```

Read all docs before writing any code. `engineering-spec.md` and `visual-design-spec.md` together contain every decision that has already been made — do not re-make them.

---

## constraints (non-negotiable)

**No internet.** Zero network requests at any point. No CDN scripts, no Google Fonts, no external images, no fetch calls, no analytics. Verify with Chrome DevTools → Network tab — it must be empty.

**No install.** The file opens directly in a browser by double-clicking. No Node.js, no npm, no Python server, no dependencies.

**No admin rights assumed.** The target machine is a managed enterprise Windows machine.

**Single file.** All HTML, CSS, and JavaScript live in `src/index.html`. No separate files or asset folders.

---

## technical approach

Use the HTML5 Canvas API for all chart rendering and PNG export. No charting libraries. No D3. No external dependencies.

The rendering pipeline:
1. Parse the uploaded CSV into typed data objects (see `engineering-spec.md`)
2. Validate the parsed data (see `csv-format.md`)
3. Calculate layout geometry from data and current config state
4. Draw the chart onto the canvas using the Canvas 2D API
5. On export, redraw at 2× resolution (3840×2160) and trigger `canvas.toBlob()` → download

---

## key implementation notes (V2)

**CSV parsing**
- Strip UTF-8 BOM (`﻿`) from the start of the raw text before parsing
- Auto-detect delimiter by counting commas, semicolons, and tabs in the header row; use whichever appears most
- Pass the detected delimiter into `splitCsvFields()` for every row

**Color swatch picker**
- `SWATCH_PALETTE` is a 7×4 array: 7 BIOTRONIK color families × 4 HSL-derived shades (lightest → light → base → dark)
- Shades are generated at runtime using `hexToHsl()` / `hslToHex()` helpers with lightness offsets (+30, +15, base, −18)
- Clicking a swatch sets `dim.color` directly on the dimension object and calls `render()` + `refreshVisibilityToggles()`
- Color changes persist for the session only — re-uploading a CSV resets to defaults

**Marker rendering**
- Milestone: `arc()` filled with Tangerine `#F04E23`, no stroke
- Decision: four-point polygon (diamond) filled with Yellow `#FFC700`, no stroke
- Collision: markers within 40px on the x-axis get a ±16px y-offset; direction chosen by which side has fewer placed markers
- Label above marker when `yOffset < 0`: start y = `cy − markerR − 3 − 24` (fixed 2-line reservation)
- Label below marker when `yOffset >= 0`: start y = `cy + markerR + 3`

**Legend**
- Drawn by Canvas 2D API within `LAYOUT.topMargin` before the timeline header
- Measures total item width dynamically using `ctx.measureText()` to size the background box
- Skips hidden content types; remaining items reflow with recalculated x positions

**Content visibility state**
- Three boolean flags on state: `showActivities`, `showMilestones`, `showDecisions`
- `drawChart()` checks these flags before rendering each content type
- Legend also reads these flags to decide which items to draw

---

## build order for future changes

When adding new features, follow the V1/V2 pattern from `engineering-spec.md`:
1. Update `visual-design-spec.md` with the visual decisions first
2. Implement in `src/index.html`
3. Update `release-notes-v2.md` (or create `release-notes-v3.md`)
4. Update `user-guide.md` if the feature is user-facing

---

## definition of done (current — V2)

- [x] `index.html` opens in Chrome with no installation step
- [x] Chrome DevTools → Network tab shows zero requests
- [x] CSV upload works via button click and file drop
- [x] Chart renders: dimension headers in palette color, activity bars, milestone circles (Tangerine), decision diamonds (Yellow), timeline axis, canvas legend
- [x] Month / Quarter / Year toggle redraws correctly
- [x] Dimension visibility toggle hides/shows swimlanes
- [x] Activities / Milestones / Decisions toggles filter content and update legend
- [x] Color swatch picker reassigns dimension color without CSV change
- [x] PNG export: `gantt-export.png` at 3840×2160px, sharp in PowerPoint
- [x] Invalid CSV rows produce readable row-specific error messages
- [x] Files from Numbers (semicolons/BOM), Excel, and Google Sheets all parse without error
- [ ] Complete chart produced from scratch in under 10 minutes *(validate on target machine)*

---

## known limitations and planned V3 work

| limitation | planned fix |
|---|---|
| 5th dimension default color (Tangerine) clashes with milestone markers | Palette conflict warning in V3 |
| 6th+ dimensions all default to Blue-Gray — visually identical | Auto-generate additional distinct colors in V3 |
| Color swatch selections are session-only — lost on CSV re-upload | State persistence (localStorage) in V3 |
| No in-app editing of activity dates or labels | Out of scope — use CSV |

---

## testing the tool

Before marking any release done, open `index.html` on the actual target machine (managed Windows, Chrome). Confirm:

- Opens from File Explorer with no prompts
- CSV upload and chart render work with `examples/sample.csv`
- PNG export downloads correctly
- Network tab is empty throughout

Do not rely on developer machine testing only.

---

## what not to build without discussion

- Export to native PowerPoint `.pptx` format
- Any server component, even a local one
- Any login, session, or state persistence beyond what is already implemented
- Any external library or CDN dependency
- Multiple file support or project saving

If in doubt about scope, open a GitHub Issue tagged `decision` before building.
