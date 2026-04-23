# swimlane-gantt

A self-contained browser tool for building executive swimlane Gantt charts from CSV input, with PNG export. No install. No internet. Runs in Chrome on a locked-down enterprise machine.

---

## Why this exists

Building Gantt and swimlane charts in PowerPoint — dragging boxes, resizing bars, repositioning text — takes 1–3 hours per chart and 10–20 minutes per update iteration. The work that actually matters (deciding what goes on the chart, defining milestones, communicating decision points to executives) gets less time because the assembly gets more. External tools are blocked by data policy. PowerPoint plugins are too slow to be usable.

This tool takes a structured CSV and renders the chart. The user's job is the data, not the boxes.

---

## Repository structure

```
/
├── README.md               — this file
├── docs/
│   ├── user-story.md       — problem framing and user context
│   ├── product-plan.md     — full product plan (CEO review output)
│   ├── engineering-spec.md — architecture, data model, edge cases, build order
│   └── csv-format.md       — CSV specification and examples
├── src/
│   └── index.html          — the tool (single file, fully self-contained)
└── examples/
    └── sample.csv          — a working example CSV to test with
```

---

## Constraints (non-negotiable)

- Runs as a single `.html` file opened in Chrome — no install, no server
- Zero network requests at any point — all processing happens in the browser
- No admin rights required
- Data never leaves the machine

---

## V1 success metric

A real project chart completed from scratch in under 10 minutes including data entry, producing a PNG that is paste-ready for a PowerPoint executive slide.
