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
| BIOTRONIK Blue | `#00234C` | dimension header background, primary text |
| White | `#FFFFFF` | chart background, text on dark backgrounds |
| Tangerine | `#F04E23` | decision markers, export button, warning border, today line |
| Tangerine Dark | `#BE3C1C` | export button hover state |
| Blue-Gray | `#587992` | timeline axis, separator lines, secondary labels |
| Gray | `#8E9295` | upload area border, disabled states, muted labels |
| Green | `#00AC6C` | upload area border on successful file drop |
| Alert Red | `#EE0000` | validation error display border and text |

### dimension color defaults (in order of appearance)

When no per-dimension color override is provided in the CSV, assign colors in this sequence:

| dimension slot | activity bar color | hex |
|---|---|---|
| 1st dimension | Sky | `#2183CA` |
| 2nd dimension | Green | `#00AC6C` |
| 3rd dimension | Aqua | `#00AFAC` |
| 4th dimension | Berry | `#860C87` |
| 5th dimension | Tangerine | `#F04E23` |
| 6th dimension+ | Blue-Gray | `#587992` |

Activity bar fill uses the dimension color at ~80% opacity (hex suffix `CC`). Activity bar border (stroke) uses the dimension color at 100% opacity.

### marker colors

| marker type | fill | text |
|---|---|---|
| milestone circle | White | BIOTRONIK Blue `#00234C` |
| milestone circle border | BIOTRONIK Blue `#00234C` | — |
| decision box fill | Tangerine `#F04E23` at 15% opacity | — |
| decision box border | Tangerine `#F04E23` | — |
| decision box text | BIOTRONIK Blue `#00234C` | — |

---

## typography

The tool targets PowerPoint output. Use Verdana as the primary font — it is universally available on Windows enterprise machines and is the BIOTRONIK Office standard.

| element | font | weight | size (at 1920px canvas width) |
|---|---|---|---|
| Dimension header label | Verdana | Bold | 13px |
| Activity label | Verdana | Regular | 11px |
| Timeline axis label (month/quarter/year) | Verdana | Regular | 10px |
| Milestone label | Verdana | Regular | 10px |
| Decision box text | Verdana | Regular | 10px |
| Chart title (if shown) | Verdana | Bold | 15px |
| UI controls (outside canvas) | system-ui, sans-serif | Regular | 14px |

All text on white background: BIOTRONIK Blue `#00234C`.
All text on BIOTRONIK Blue background: White `#FFFFFF`.

---

## chart layout

### overall structure

```
┌─────────────────────────────────────────────────────────────────┐
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

### dimension header

- Background: BIOTRONIK Blue `#00234C`
- Text: White `#FFFFFF`, Verdana Bold 13px, vertically centered, 12px left padding
- Spans full height of the dimension (header row + all activity rows + marker row)
- The dimension header column is on the left; the timeline area is to the right

### activity bars

- Height: 18px, vertically centered within the 28px activity row
- Corner radius: 3px
- Fill: dimension color at ~80% opacity (hex suffix `CC`)
- Stroke: dimension color at 100% opacity, 1px
- Label: Verdana Regular 11px, BIOTRONIK Blue `#00234C`, positioned inside the bar if bar width >60px; otherwise positioned to the right of the bar
- Minimum bar width: 4px (for very short activities — label goes to the right)

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

- Shape: circle, 14px diameter (7px radius)
- Fill: White `#FFFFFF`
- Stroke: BIOTRONIK Blue `#00234C`, 1.5px
- Position: centered on the milestone date, vertically centered within the marker row
- Label: Verdana Regular 10px, BIOTRONIK Blue `#00234C`, positioned below the circle, center-aligned, max 2 lines before truncating with ellipsis

### decision (text box)

- Shape: rounded rectangle, corner radius 4px
- Fill: Tangerine `#F04E23` at 15% opacity
- Stroke: Tangerine `#F04E23`, 1px
- Position: left edge aligned to the decision date on the timeline, top-aligned to marker row
- Width: auto — expands to fit text content, maximum 220px, wraps to next line if longer
- Minimum height: 36px
- Text: Verdana Regular 10px, BIOTRONIK Blue `#00234C`, 6px padding all sides, line height 12px
- Multi-line text: line breaks inserted at `\n` in the CSV Notes field (stored as `\\n` in raw CSV)

### marker row positioning

Each dimension has one marker row, sitting below its last activity row and above the dimension separator line. Milestone and decision markers for the same dimension share this row. If two markers have dates within 24px of each other on the timeline, the second marker is offset 8px downward.

---

## UI controls (outside the canvas)

These are HTML elements above the canvas. They are not part of the exported PNG.

### validation error display

Shown above all other controls if the CSV fails validation. Hidden when no errors exist.

- Background: Alert Red `#EE0000` at 10% opacity
- Border: 1px solid Alert Red `#EE0000`
- Border radius: 4px
- Padding: 12px
- Font: system-ui Regular 13px, Alert Red `#EE0000`
- White-space: pre-wrap (each error on its own line)
- Example: `Row 4: invalid date format — expected YYYY-MM-DD`

### scale compression warning display

Shown below the error display (or at the top if no errors) when the timeline spans >3 years and Month scale is active. Hidden otherwise.

- Background: Tangerine `#F04E23` at 12% opacity
- Border: 1px solid Tangerine `#F04E23`
- Border radius: 4px
- Padding: 12px
- Font: system-ui Regular 13px, BIOTRONIK Blue `#00234C`
- Message: `Timeline spans over 3 years. Month scale may be compressed — consider Quarter or Year.`

### upload area

- Dashed border, 2px, Gray `#8E9295`
- Corner radius: 8px
- Background: White
- Height: 80px
- Text: "Upload CSV or drop file here" — system-ui Regular 14px, Gray `#8E9295`, centered
- On hover: border color changes to BIOTRONIK Blue `#00234C` (class `hover`)
- On successful file load: border color and text color change to Green `#00AC6C`, text changes to filename (class `drop-ok`, reverts after 1 second)

### controls row

A flex row between the upload area and visibility toggles:

```
[scale toggle — left] [   spacer   ] [export button — right]
```

### timeline scale toggle

Three buttons: Month | Quarter | Year

- Default state: White background, BIOTRONIK Blue border `#00234C` 1px, BIOTRONIK Blue text, system-ui 14px
- Active state: BIOTRONIK Blue `#00234C` background, White text
- Button height: 32px, padding: 0 16px
- Grouped with shared borders (left button: rounds top-left and bottom-left only; right button: rounds top-right and bottom-right only; middle: no border radius). Adjacent buttons share a single 1px border (no double border).

### visibility toggles

One toggle per dimension, rendered after CSV is loaded. Shown in a wrapping flex row, 24px gap between items.

- Custom checkbox: 16×16px square, BIOTRONIK Blue `#00234C` border 1.5px, corner radius 3px
- Checked state: BIOTRONIK Blue fill, white ✓ checkmark (11px, centered)
- Label: system-ui Regular 13px, BIOTRONIK Blue `#00234C`
- All dimensions start visible (checked)

### export button

- Label: "Export PNG"
- Background: Tangerine `#F04E23`
- Text: White `#FFFFFF`, system-ui Bold 13px
- Height: 36px, padding: 0 24px
- Border radius: 4px
- On hover: Tangerine Dark `#BE3C1C`

---

## spacing and layout of UI controls

```
[error display — full width, hidden if no errors]
[warning display — full width, hidden if no warning]
[16px gap]
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
- Any animation or transition effect
- Any external font loading (no Google Fonts, no CDN fonts)
- Any chart type other than the horizontal swimlane Gantt described here
