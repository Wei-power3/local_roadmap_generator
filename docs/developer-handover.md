# developer handover guide

## what you are building

A single self-contained HTML file (`src/index.html`) that runs in Chrome with no server, no install, and no internet connection. The user uploads a CSV, the tool renders a swimlane Gantt chart on an HTML Canvas, and exports it as a PNG.

There is no backend. There is no build step. There is no framework required. The entire deliverable is one file.

---

## repository structure

```
/
├── README.md                     — project overview
├── docs/
│   ├── user-story.md             — why this exists, who uses it, what good looks like
│   ├── product-plan.md           — product decisions, scope, what is in and out of V1
│   ├── engineering-spec.md       — data model, data flow, edge cases, build order, definition of done
│   ├── csv-format.md             — full CSV column spec, row types, validation rules, examples
│   ├── visual-design-spec.md     — every layout, color, typography, and sizing decision
│   └── developer-handover.md    — this file
├── src/
│   └── index.html               — the tool (your deliverable)
└── examples/
    └── sample.csv               — a working CSV to test with during development
```

Read all docs before writing any code. The engineering spec and visual design spec together contain every decision that has already been made — do not re-make them.

---

## constraints (non-negotiable)

These are not preferences. Do not deviate.

**No internet.** The HTML file must make zero network requests at any point. No CDN scripts, no Google Fonts, no external images, no fetch calls, no analytics. Verify with Chrome DevTools → Network tab — it must be empty when the tool is running.

**No install.** The file opens directly in Chrome by double-clicking. No Node.js, no npm, no Python server, no dependencies to install.

**No admin rights assumed.** The target machine is a managed enterprise Windows machine. The user cannot install anything.

**Single file.** The entire tool — HTML, CSS, JavaScript — lives in `src/index.html`. No separate `.js` or `.css` files. No asset folders.

---

## technical approach

Use the HTML5 Canvas API for chart rendering and PNG export. No charting libraries. No D3. No external dependencies of any kind.

The rendering pipeline:

1. Parse the uploaded CSV into typed data objects (see data model in `engineering-spec.md`)
2. Validate the parsed data (see validation rules in `csv-format.md`)
3. Calculate layout geometry from the data and current config state
4. Draw the chart onto the canvas using the Canvas 2D API
5. On export, redraw at 2× resolution (3840×2160) and trigger `canvas.toBlob()` → file download

---

## build order

Follow this sequence exactly. Each step produces something you can open in a browser and test before moving to the next.

**Step 1 — CSV parser and validator**
Write the parser first, with no UI. Load the sample CSV from `examples/sample.csv` in a `<script>` tag as a hardcoded string. Parse it into the data objects defined in `engineering-spec.md`. Log the output to the console. Verify every row type is correctly identified and every field is correctly typed. Then add validation: log a clear error message for each rule violation defined in `csv-format.md`.

Do not draw anything yet.

**Step 2 — Static chart renderer**
With the parsed sample data hardcoded, draw the chart on a canvas using the layout and color values from `visual-design-spec.md`. Get the full chart rendering correct — dimension headers, activity bars, timeline axis — before adding any interactivity.

Check: does it look correct at 1920×1080? Does the BIOTRONIK Blue dimension header render correctly? Are activity bars the right height and color?

**Step 3 — Floating markers**
Add milestone circles and decision text boxes, positioned at the marker row of their respective dimensions. Test with the sample CSV which includes both marker types.

**Step 4 — Timeline scale toggle**
Add the three-button toggle (Month / Quarter / Year). On click, recalculate the timeline axis and redraw the chart. The rest of the chart (rows, bars, markers) stays the same — only the axis labels and tick positions change.

**Step 5 — PNG export**
Add the Export PNG button. On click, redraw the chart at 3840×2160 (2× scale), call `canvas.toBlob('image/png')`, and trigger a file download named `gantt-export.png`. After download, redraw the display canvas at normal resolution.

Test: open the PNG, paste it into PowerPoint on a widescreen slide, and check that it is sharp and correctly proportioned.

**Step 6 — CSV file upload**
Replace the hardcoded sample data with the real file upload. Add the upload area (button + file drop target) from the visual design spec. On upload, parse the CSV and re-render. On validation error, show the error display and do not render.

**Step 7 — Visibility toggles**
Add checkboxes for each dimension. On toggle, hide or show that dimension's rows and re-render. All dimensions start visible.

**Step 8 — Color overrides from CSV**
Read the optional Color column from dimension rows. If present and a valid hex value, use it as the activity bar color for that dimension. If absent or blank, use the default sequence from `visual-design-spec.md`.

**Step 9 — Error display and edge case handling**
Review the edge cases in `engineering-spec.md`. Add the full validation error display UI. Verify every failure mode produces a readable message and does not crash the app.

---

## definition of done

V1 is complete when every item on this list passes:

- [ ] `index.html` opens in Chrome on a Windows enterprise machine with no installation step
- [ ] Chrome DevTools → Network tab shows zero requests when the tool is running
- [ ] CSV upload works via button click
- [ ] CSV upload works via dragging the file onto the upload area
- [ ] Chart renders correctly with the sample CSV: dimension headers, activity bars, milestone markers, decision markers, timeline axis
- [ ] Month / Quarter / Year toggle redraws the chart correctly
- [ ] Hiding a dimension via the visibility toggle removes it from the chart
- [ ] PNG export produces a file named `gantt-export.png` at 3840×2160px
- [ ] The exported PNG is sharp (not blurry) when pasted into PowerPoint and viewed full-screen on a widescreen slide
- [ ] An invalid CSV row produces a readable, row-specific error message above the upload area
- [ ] The app does not crash on any of the edge cases listed in `engineering-spec.md`
- [ ] A complete chart is produced from scratch (fresh CSV upload to PNG export) in under 10 minutes

---

## testing the tool on a locked-down machine

Before marking V1 done, open `index.html` on the actual target machine (managed Windows, no admin rights, Chrome). Confirm:

- The file opens directly from File Explorer with no prompt to install anything
- The CSV upload and chart render work
- The PNG export downloads to the Downloads folder
- Chrome DevTools → Network tab is empty throughout

Do not rely on testing only on a developer machine. The target environment matters.

---

## what not to build

These are explicitly out of scope for V1. Do not add them, even if they seem easy:

- Drag-and-drop editing of chart elements
- Export to native PowerPoint `.pptx` format
- A color picker UI — color configuration lives in the CSV
- Any server component, even a local one
- Any login, session, or state persistence
- Any external library or CDN dependency
- Multiple file support or project saving

If in doubt about scope, check `product-plan.md` or ask before building.

---

## where to ask questions

Push questions as GitHub Issues in this repository. Tag them with:

- `question` — needs clarification before proceeding
- `decision` — a choice that is not covered by the docs and needs product input
- `bug` — something not working as specified

Do not make undocumented design or architecture decisions independently. If the docs do not cover it, open an issue.
