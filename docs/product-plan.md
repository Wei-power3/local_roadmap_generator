# product plan

## problem statement

A product/strategy manager at a regulated medical device company builds executive-facing swimlane Gantt charts multiple times per project cycle. The current method — native PowerPoint drag-and-drop — consumes 1–3 hours per chart and 10–20 minutes per update. External tools are blocked by data policy. PowerPoint plugins are too slow to be usable. The wedge: a zero-install, offline-capable HTML file that takes structured CSV input and renders a clean swimlane chart exportable as PNG.

---

## what we are building

A single `.html` file. The user opens it in Chrome, uploads a CSV, and gets a rendered swimlane Gantt chart. She adjusts scale and visibility, exports a PNG, and pastes it into PowerPoint.

Nothing is installed. Nothing leaves the machine.

---

## locked decisions

| decision | answer |
|---|---|
| Delivery format | Single `.html` file, opened in Chrome |
| Input method | CSV upload via button or drag-drop of the file onto the page |
| Row hierarchy | Dimension (swimlane group) → Activity (bar), with floating markers at dimension level |
| Marker types | `bar`, `milestone` (circle), `decision` (highlighted text box) |
| Timeline scales | Month / Quarter / Year — toggle in the UI |
| Color default | BIOTRONIK brand palette |
| Color override | Per-dimension, specified in CSV |
| Export format | PNG |
| Export size | 1920×1080px (standard widescreen PowerPoint slide) |
| Network | Zero requests — fully offline |
| Admin rights required | None |

---

## scope: V1

Include:

- CSV upload (button and file drop)
- Swimlane chart rendering with dimension headers, activity bars, floating markers
- Month / Quarter / Year timeline scale toggle
- Swimlane visibility toggle
- PNG export at 1920×1080px, rendered at 2× for sharpness
- Row-level CSV validation with readable error messages

Exclude from V1:

- Drag-and-drop editing of chart elements
- Native PowerPoint (`.pptx`) export
- Color picker UI — color config lives in the CSV
- Multi-user or shared state of any kind
- Any server component

---

## why this approach wins

Every existing alternative fails on one of two dimensions: internet dependency (blocked by policy) or PowerPoint plugin sandbox lag (too slow to use). A standalone HTML file sidesteps both. It requires no install, makes no network calls, and runs entirely in the browser's rendering engine — which is fast.

---

## kill assumptions

These three assumptions, if wrong, stop the project:

1. Chrome on the work machine supports the HTML Canvas API. This is required for PNG export. Test: open a minimal canvas test HTML file in Chrome on the actual work machine before building anything.

2. CSV date format can be standardized to `YYYY-MM-DD`. If users cannot commit to a consistent format, date parsing becomes fragile. Fix: define the format in the CSV template and document it clearly.

3. PNG quality at 1920×1080px is sufficient for executive slides viewed on a projector or large screen. Test: export one chart, paste into PowerPoint, and view at full size before V1 sign-off.

---

## success metric

The user completes a real project chart from scratch — including data entry — in under 10 minutes. The exported PNG is paste-ready for a PowerPoint executive slide with no further editing.
