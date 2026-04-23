# swimlane-gantt

A self-contained browser tool for building executive swimlane Gantt charts from CSV input, with PNG export. No install. No internet. Runs in Chrome on a locked-down enterprise machine.

---

## Why this exists

Building Gantt and swimlane charts in PowerPoint — dragging boxes, resizing bars, repositioning text — takes 1–3 hours per chart and 10–20 minutes per update iteration. The work that actually matters (deciding what goes on the chart, defining milestones, communicating decision points to executives) gets less time because the assembly gets more. External tools are blocked by data policy. PowerPoint plugins are too slow to be usable.

This tool takes a structured CSV and renders the chart. The user’s job is the data, not the boxes.

---

## Quick start

1. Open `src/index.html` in Chrome (double-click from File Explorer — no install needed)
2. Upload a CSV using the upload area or drag-and-drop the file onto it
3. Adjust the timeline scale (Month / Quarter / Year) and toggle swimlane visibility as needed
4. Click **Export PNG** to download a 3840×2160px file ready to paste into PowerPoint

See `examples/sample.csv` for a working CSV to test with. See `docs/csv-format.md` for the full column specification.

---

## Repository structure

```
/
├── README.md                     — this file
├── docs/
│   ├── user-story.md             — problem framing and user context
│   ├── product-plan.md           — full product plan and locked decisions
│   ├── engineering-spec.md       — architecture, data model, edge cases, build order
│   ├── csv-format.md             — CSV specification and examples
│   ├── visual-design-spec.md     — layout, color, typography, and sizing decisions
│   └── developer-handover.md     — build instructions and definition of done
├── src/
│   └── index.html                — the tool (single file, fully self-contained)
└── examples/
    └── sample.csv                — a working example CSV to test with
```

---

## Status

**V1 complete.** `src/index.html` is a fully working implementation covering:

- CSV upload (button click and drag-and-drop)
- Row-level CSV validation with readable error messages
- Swimlane chart rendering: dimension headers, activity bars, milestone circles, decision text boxes, timeline axis
- Month / Quarter / Year timeline scale toggle
- Swimlane visibility toggles
- Today line (shown when today falls within the chart date range)
- Scale compression warning (shown when Month scale is used on a >3-year project)
- PNG export at 3840×2160px (2× resolution) via `canvas.toBlob()`
- Per-dimension color overrides via optional `Color` column in CSV
- Zero network requests — fully offline

---

## Constraints (non-negotiable)

- Runs as a single `.html` file opened in Chrome — no install, no server
- Zero network requests at any point — all processing happens in the browser
- No admin rights required
- Data never leaves the machine

---

## V1 success metric

A real project chart completed from scratch in under 10 minutes including data entry, producing a PNG that is paste-ready for a PowerPoint executive slide.
