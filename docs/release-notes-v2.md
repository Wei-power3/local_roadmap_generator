# Release Notes — V2

## Summary

V2 delivers a complete visual and UX overhaul of the swimlane Gantt tool, building on the V1 foundation. All changes are contained in `src/index.html`. No new dependencies, no build step required.

---

## What Changed

### Marker Visual Disambiguation
Milestones and decisions previously looked identical (both Tangerine circles). They are now visually distinct:

| Marker Type | Shape | Color |
|---|---|---|
| Milestone | Filled circle | Tangerine `#F04E23` |
| Decision | Filled diamond | Yellow `#FFC700` |

Labels sit below each marker. When two markers fall close together, one label moves above its marker automatically to prevent overlap.

### Canvas Legend
A legend is drawn inside the canvas (top-left corner, within the top margin) so it appears in both the browser view and the exported PNG. It shows only the content types currently visible and reflows automatically when items are toggled off.

### Dimension Header Colors
Each swimlane's left-side header now fills with that dimension's assigned palette color instead of always using BIOTRONIK Blue. This creates a clear visual link between the header and its activity bars.

### Content Visibility Toggles
Three new checkboxes appear in the controls area alongside the existing dimension toggles:
- **Show: Activities** — hides/shows all activity bars
- **Show: Milestones** — hides/shows milestone markers and their legend entry
- **Show: Decisions** — hides/shows decision markers and their legend entry

All three are enabled by default.

### Adaptive Activity Label Placement
Activity bar labels are now placed inside the bar when the text fits (measured exactly), and outside when it does not. Previously this used a fixed 60px threshold that ignored actual text length.

### Font Sizes
All canvas text bumped to 12px for comfortable reading at screen and presentation distance:

| Element | V1 | V2 |
|---|---|---|
| Activity labels | 11px | 12px |
| Timeline axis | 10px | 12px |
| Milestone labels | 10px | 12px |
| Decision labels | 10px | 12px |
| Dimension headers | 13px bold | 13px bold (unchanged) |

### CSV Compatibility — macOS Numbers
The parser now handles CSV files exported from Apple Numbers without any manual preprocessing:
- **Auto-detects delimiter** — comma, semicolon, or tab; whichever appears most in the header row
- **Strips UTF-8 BOM** — Numbers sometimes prepends an invisible byte-order mark that previously caused all columns to report as missing

---

## Dimension Color Palette

Default colors assigned in order of appearance (unchanged from V1, documented here for reference):

| Slot | Name | Hex |
|---|---|---|
| 1st | Sky | `#2183CA` |
| 2nd | Green | `#00AC6C` |
| 3rd | Aqua | `#00AFAC` |
| 4th | Berry | `#860C87` |
| 5th | Tangerine | `#F04E23` |
| 6th+ | Blue-Gray | `#587992` |

**Known limitation:** The 5th dimension slot uses Tangerine, which is the same color as milestone markers. If a fifth dimension is needed, assign a custom color via the CSV `Color` column (e.g. `Color;#9B59B6`).

---

## Known Limitations

| # | Issue | Workaround |
|---|---|---|
| 1 | 5th dimension defaults to Tangerine — clashes visually with milestone markers | Add `Color` column in CSV and assign a custom hex |
| 2 | 6th+ dimensions all default to Blue-Gray — indistinguishable from each other | Same workaround as above |
| 3 | No in-app color picker — colors can only be overridden via CSV | Planned for V3 (see below) |
| 4 | Date format must be `YYYY-MM-DD` — Numbers date cells export in locale format | Format date cells as plain text in Numbers before exporting |
| 5 | Marker row height is fixed at 36px — dense marker clusters may still feel tight | Increase `LAYOUT.markerRowHeight` in source if needed |

---

## Planned for V3

- **In-app color configuration** — let users reassign dimension colors from the BIOTRONIK palette directly in the UI, without editing the CSV
- **Palette conflict warning** — alert users when a dimension color would clash with marker colors
- **Color cycle improvement** — generate additional distinct colors beyond the 6 built-in slots for projects with many dimensions

---

## File Reference

| File | Purpose |
|---|---|
| `src/index.html` | Complete self-contained application — open directly in any modern browser |
| `examples/sample.csv` | Test data — 4 dimensions, activities, milestones, decisions |
| `docs/csv-format.md` | Full CSV column reference and row type rules |
| `docs/visual-design-spec.md` | All visual decisions — colors, typography, layout dimensions |
| `docs/engineering-spec.md` | Architecture, edge cases, rendering pipeline |
