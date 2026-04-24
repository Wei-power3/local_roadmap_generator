# visual design spec

This document defines every visual decision for `index.html`. The developer should not make independent layout or styling choices — if something is not covered here, ask before deciding.

---

## canvas and export dimensions

The chart renders on an HTML Canvas element.

| property | value |
|---|---|
| Display width | 100% of browser window width |
| Display height | auto — scales to content |
| Export resolution | 3840×2160px (2× of 1920×1080) |
| Export format | PNG |
| Export filename | `gantt-export.png` |
| Aspect ratio | 16:9 (widescreen PowerPoint slide) |

Render at 2× (3840×2160) using an offscreen canvas with `setTransform(2,0,0,2,0,0)` and `canvas.toBlob()`. The display canvas scales to fit the browser window via CSS (`width: 100%; height: auto`). This ensures the exported PNG is sharp on high-DPI screens and when projected.

---

## color palette

All colors come from the BIOTRONIK brand palette. Do not introduce colors outside this set.

### primary colors

| name | hex | use |
|---|---|---|
| BIOTRONIK Blue | `#00234C` | primary text, dimension header text, legend text |
| White | `#FFFFFF` | chart background, text on colored backgrounds |
| Tangerine | `#F04E23` | milestone markers, export button, warning border, today line |
| Tangerine Dark | `#BE3C1C` | export button hover state |
| Yellow | `#FFC700` | decision markers |
| Blue-Gray | `#587992` | timeline axis, separator lines, secondary labels, legend border |
| Gray | `#8E9295` | upload area border, disabled states, muted labels |
| Green | `#00AC6C` | upload area border on successful file drop |
| Alert Red | `#EE0000` | validation error display border and text |

### dimension color defaults (in order of appearance)

When no per-dimension color override is provided in the CSV or set via the UI, assign colors in this sequence:

| dimension slot | color name | hex |
|---|---|---|
| 1st dimension | Sky | `#2183CA` |
| 2nd dimension | Green | `#00AC6C` |
| 3rd dimension | Aqua | `#00AFAC` |
| 4th dimension | Berry | `#860C87` |
| 5th dimension | Tangerine | `#F04E23` |
| 6th dimension+ | Blue-Gray | `#587992` |

Activity bar fill uses the dimension color at ~80% opacity (hex suffix `CC`). Activity bar stroke uses the dimension color at 100% opacity, 1px.

> **Known conflict:** The 5th slot uses Tangerine, which is the same color as milestone markers. If a fifth or later dimension is added, assign a custom color via the CSV `Color` column or via the in-app swatch picker.

### marker colors

| marker type | shape | fill | stroke | label color |
|---|---|---|---|---|
| Milestone | Circle, 7px radius | Tangerine `#F04E23` | None | BIOTRONIK Blue `#00234C` |
| Decision | Diamond (rotated square, 7px) | Yellow `#FFC700` | None | BIOTRONIK Blue `#00234C` |

---

## typography

The tool targets PowerPoint output. Use Verdana as the primary font — it is universally available on Windows enterprise machines and is the BIOTRONIK Office standard.

| element | font | weight | size (at 1920px canvas width) |
|---|---|---|---|
| Dimension header label | Verdana | Bold | 13px |
| Activity label | Verdana | Regular | 12px |
| Timeline axis label (month/quarter/year) | Verdana | Regular | 12px |
| Milestone label | Verdana | Regular | 12px |
| Decision label | Verdana | Regular | 12px |
| Legend label | Verdana | Regular | 12px |
| Chart title (if shown) | Verdana | Bold | 15px |
| UI controls (outside canvas) | system-ui, sans-serif | Regular | 14px |

All text on white background: BIOTRONIK Blue `#00234C`.
All text on dimension-colored background: White `#FFFFFF`.

---

## chart layout

### overall structure

```
┌─────────────────────────────────────────────────────────────────┐
│  [legend — top margin area, left-aligned]                       │
├─────────────────────────────────────────────────────────────────┤
│  [timeline axis header — months or quarters or years]           │
├──────────────────┬──────────────────────────────────────────────┤
│  Dimension A     │  [activity bars and floating markers]        │
│  header          │  activity row 1                              │
│                  │  activity row 2                              │
│                  │  activity row 3                              │
│                  │  [marker row]                                │
├──────────────────┼──────────────────────────────────────────────┤
│  Dimension B     │  [activity bars and floating markers]        │
│  header          │  activity row 1                              │
│                  │  activity row 2                              │
│                  │  [marker row]                                │
└──────────────────┴──────────────────────────────────────────────┘
```

### measurements (at 1920px canvas width)

| element | value |
|---|---|
| Left label column width | 200px |
| Timeline area width | 1720px (1920 − 200) |
| Canvas top margin | 40px |
| Canvas bottom margin | 40px |
| Timeline axis header height | 36px |
| Dimension header row height | 32px |
| Activity row height | 28px |
| Marker row height | 36px |
| Activity bar height | 18px (vertically centered in 28px row) |
| Row vertical padding | 4px top and bottom |
| Separator line between dimensions | 1px, Blue-Gray `#587992` at 40% opacity |
| Separator line between activities within a dimension | 0.5px, Gray `#8E9295` at 30% opacity |
| Minimum canvas height | 1080px |

### legend

Drawn inside the canvas using the Canvas 2D API, within `topMargin` (40px). Not an HTML element — appears in both the browser view and the exported PNG.

- Position: top-left corner, 12px from left edge, vertically centered within topMargin (y = 20px)
- Layout: three items in a horizontal row — activity bar sample, milestone sample, decision sample
- Background: white fill with 1px Blue-Gray border, 3px corner radius, 8px horizontal padding, 6px vertical padding
- Font: Verdana Regular 12px, BIOTRONIK Blue `#00234C`, textBaseline middle
- Spacing between items: 24px

| item | sample shape | label |
|---|---|---|
| Activity bar | Rounded rectangle 28×12px, dimension color at 80% fill + 100% stroke | "Activity bar" |
| Milestone | Filled Tangerine circle, 7px radius | "Milestone" |
| Decision | Filled Yellow diamond, 7px | "Decision point" |

The legend dynamically hides items when the corresponding content type is toggled off. Remaining items reflow to close the gap.

### dimension header

- Background: the dimension's assigned palette color (same color as its activity bars)
- Text: White `#FFFFFF`, Verdana Bold 13px, vertically centered, 12px left padding
- Spans full height of the dimension section (header row + all activity rows + marker row)

### activity bars

- Height: 18px, vertically centered within the 28px activity row
- Corner radius: 3px
- Fill: dimension color at ~80% opacity (hex suffix `CC`)
- Stroke: dimension color at 100% opacity, 1px
- Label: Verdana Regular 12px, BIOTRONIK Blue `#00234C`
  - Placed inside the bar if `textWidth + 12px ≤ barWidth`; otherwise placed to the right of the bar
  - Text width is measured at runtime using `ctx.measureText()`
- Minimum bar width: 4px (label always goes to the right for very short activities)

### timeline axis

- Header row background: White `#FFFFFF`
- Header row border-bottom: 1.5px solid BIOTRONIK Blue `#00234C`

| scale | label format | tick interval |
|---|---|---|
| Month | MMM YYYY (e.g. Jan 2025) | 1 month |
| Quarter | Q1 2025, Q2 2025 etc. | 1 quarter |
| Year | 2025, 2026 etc. | 1 year |

- Tick lines: 0.5px, Blue-Gray `#587992` at 30% opacity, run full height of the chart
- Today line: 1px dashed (6px on, 4px off), Tangerine `#F04E23` at 60% opacity — only shown if today falls within the chart date range

---

## markers

### milestone (circle)

- Shape: filled circle, 7px radius
- Fill: Tangerine `#F04E23`
- Stroke: none
- Position: centered on the milestone date, vertically centered within the marker row
- Label: Verdana Regular 12px, BIOTRONIK Blue `#00234C`, center-aligned, positioned below the circle (3px gap), max 2 lines before truncating with ellipsis

### decision (diamond)

- Shape: filled diamond (square rotated 45°), 7px point-to-center
- Fill: Yellow `#FFC700`
- Stroke: none
- Position: centered on the decision date, vertically centered within the marker row
- Label: Verdana Regular 12px, BIOTRONIK Blue `#00234C`, center-aligned, positioned below the diamond (3px gap), max 2 lines before truncating with ellipsis

### marker row positioning

Each dimension has one marker row, sitting below its last activity row and above the dimension separator line. Milestone and decision markers for the same dimension share this row.

**Collision handling:** if two markers have dates within 40px of each other on the timeline:
- The second marker is offset vertically by ±16px
- Direction is chosen based on which side (above/below center) has fewer placed markers
- When offset is negative (upward), the label is drawn above the marker shape
- When offset is zero or positive (downward), the label is drawn below the marker shape

---

## UI controls (outside the canvas)

These are HTML elements above the canvas. They are not part of the exported PNG.

### validation error display

- Background: Alert Red `#EE0000` at 10% opacity
- Border: 1px solid Alert Red `#EE0000`
- Border radius: 4px, Padding: 12px
- Font: system-ui Regular 13px, Alert Red `#EE0000`
- White-space: pre-wrap (each error on its own line)

### scale compression warning display

Shown when the timeline spans >3 years and Month scale is active.

- Background: Tangerine `#F04E23` at 12% opacity
- Border: 1px solid Tangerine `#F04E23`
- Border radius: 4px, Padding: 12px
- Font: system-ui Regular 13px, BIOTRONIK Blue `#00234C`

### upload area

- Dashed border, 2px, Gray `#8E9295`, corner radius 8px, height 80px
- Text: "Upload CSV or drop file here" — system-ui Regular 14px, Gray `#8E9295`, centered
- On hover: border color → BIOTRONIK Blue `#00234C`
- On successful load: border and text → Green `#00AC6C`, text → filename (reverts after 1s)

### controls row

```
[scale toggle — left] [   spacer   ] [export button — right]
```

### timeline scale toggle

Three buttons: Month | Quarter | Year

- Default: White background, BIOTRONIK Blue border 1px, BIOTRONIK Blue text, system-ui 14px
- Active: BIOTRONIK Blue background, White text
- Height: 32px, padding: 0 16px
- Grouped with shared borders, no double borders

### visibility toggles

Rendered after CSV is loaded. Shown in a wrapping flex row.

**Dimensions section** — one toggle per dimension, labelled "Dimensions:":
- Each dimension item shows a checkbox + name, with a 7×4 color swatch grid below it
- Swatch grid: 7 color families (Sky, Green, Aqua, Berry, Tangerine, Yellow, Blue-Gray) × 4 shades (lightest → light → base → dark), generated via HSL offsets
- Clicking a swatch instantly reassigns that dimension's color and re-renders
- Active swatch highlighted with a BIOTRONIK Blue border

**Content section** — three checkboxes labelled "Show:", separated from dimensions by a vertical divider:
- Show: Activities
- Show: Milestones
- Show: Decisions
- All enabled by default
- Toggling off hides that content type from the canvas and removes it from the legend

### export button

- Label: "Export PNG", Background: Tangerine `#F04E23`, Text: White Bold system-ui 13px
- Height: 36px, padding: 0 24px, border radius: 4px
- On hover: Tangerine Dark `#BE3C1C`

---

## spacing and layout of UI controls

```
[error display — full width, hidden if no errors]
[warning display — full width, hidden if no warning]
[upload area — full width]
[16px gap]
[controls row: scale toggle left | spacer | export button right]
[16px gap]
[visibility toggles — wrapping flex row, left aligned]
[24px gap]
[canvas — full width, border: 1px rgba(88,121,146,0.25)]
```

---

## what the developer should not decide independently

- Any color not in the BIOTRONIK palette above
- Font choices other than Verdana (for canvas text) and system-ui (for UI controls)
- Layout dimensions other than those specified
- Any animation or transition effect (hover scale on swatches excepted)
- Any external font loading (no Google Fonts, no CDN fonts)
- Any chart type other than the horizontal swimlane Gantt described here
