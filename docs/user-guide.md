# User Guide — Swimlane Roadmap Generator

## What this tool does

You upload a CSV file. The tool draws a swimlane Gantt chart from it and lets you export it as a PNG ready for PowerPoint. Everything runs in your browser — no login, no cloud, no internet required.

---

## Step 1 — Set up your spreadsheet

Open a new spreadsheet in Excel, Google Sheets, or Apple Numbers. The first row must be exactly these column headers, in this order:

```
Row Type | Dimension | Activity | Label | Start Date | End Date | Type | Notes
```

Every row after that is either a **dimension**, an **activity**, or a **marker**. Each is explained below.

---

## Step 2 — Understand the three row types

### Dimension row
A dimension is a swimlane — a horizontal band that groups related activities. Think of it as a chapter heading.

| Column | What to fill in |
|---|---|
| Row Type | `dimension` |
| Dimension | The name of the swimlane (e.g. `Business Strategy`) |
| Everything else | Leave blank |

> A dimension row must appear **before** any activities or markers that belong to it.

---

### Activity row
An activity is a task with a start and end date. It appears as a colored horizontal bar in the chart.

| Column | What to fill in |
|---|---|
| Row Type | `activity` |
| Dimension | Must match a dimension name defined above |
| Activity | The label shown on the bar (e.g. `Market research`) |
| Start Date | `YYYY-MM-DD` format (e.g. `2025-03-01`) |
| End Date | `YYYY-MM-DD` format — must be same day or later than Start Date |
| Type | `bar` |
| Label, Notes | Leave blank |

---

### Marker row
A marker is a point-in-time event — a milestone or a decision. It appears as a symbol on the chart at the given date.

| Column | What to fill in |
|---|---|
| Row Type | `marker` |
| Dimension | Must match a dimension name defined above |
| Start Date | The date the event occurs (`YYYY-MM-DD`) |
| Type | `milestone` or `decision` |
| Label | Short name shown below the marker (e.g. `Go-live`) |
| Notes | Optional longer text — only used if you want extra context |
| Activity, End Date | Leave blank |

**Milestone** → appears as a filled orange circle  
**Decision** → appears as a filled yellow diamond

---

## Step 3 — Date format

All dates must be typed as `YYYY-MM-DD`. This means:
- Year first, then month, then day
- Always use four digits for the year, two for the month, two for the day
- Separate with hyphens

| ✓ Correct | ✗ Wrong |
|---|---|
| `2025-03-15` | `15/03/2025` |
| `2025-11-01` | `Nov 1, 2025` |
| `2026-01-07` | `1/7/26` |

> **Apple Numbers tip:** Numbers formats date cells automatically using your system locale, which will export in the wrong format. To avoid this, format your date columns as **plain text** before typing dates, or type an apostrophe before the date (`'2025-03-15`) to force Numbers to treat it as text.

---

## Step 4 — A complete example

Here is a small but complete CSV showing the structure:

```
Row Type,Dimension,Activity,Label,Start Date,End Date,Type,Notes
dimension,Business Strategy,,,,,,
activity,Business Strategy,Vision and ambition,,2025-01-01,2025-06-30,bar,
activity,Business Strategy,Market prioritization,,2025-02-01,2025-07-31,bar,
marker,Business Strategy,,,2025-01-15,,decision,Approve strategy and allocate budget
dimension,Product,,,,,,
activity,Product,Customer discovery,,2025-01-01,2025-03-31,bar,
activity,Product,Solution design,,2025-02-01,2025-05-31,bar,
marker,Product,,,2025-04-01,,milestone,Concept locked
```

Notice:
- Each `dimension` row comes before the activities and markers that reference it
- The `Dimension` column on activity and marker rows matches the dimension name exactly (including capitalisation)
- All dates use `YYYY-MM-DD`
- Activity rows have both Start Date and End Date; marker rows only have Start Date

---

## Step 5 — Export as CSV from your app

### Apple Numbers (Mac)
1. `File → Export To → CSV…`
2. Set Text Encoding to **Unicode (UTF-8)**
3. Click **Next** and save

### Microsoft Excel
1. `File → Save As`
2. Choose **CSV UTF-8 (Comma delimited) (.csv)**

### Google Sheets
1. `File → Download → Comma Separated Values (.csv)`

> The tool automatically detects whether your file uses commas, semicolons, or tabs as separators — all three work.

---

## Step 6 — Upload and view

1. Open `src/index.html` in your browser
2. Drag your CSV file onto the upload area, or click it to browse
3. If there are errors, a red box appears with the row number and what went wrong — fix those rows in your spreadsheet and re-upload
4. Use the **Month / Quarter / Year** buttons to change the timeline scale
5. Use the **Dimensions** checkboxes to show or hide individual swimlanes
6. Use the **Show** checkboxes to show or hide Activities, Milestones, or Decisions across the whole chart

---

## Step 7 — Change dimension colors

Below each dimension's checkbox in the control panel, a grid of color swatches appears. Click any swatch to instantly change that dimension's bar and header color. Colors are organised in families (Sky, Green, Aqua, Tangerine, Yellow, Blue-Gray), each with four shades from light to dark.

Color changes apply immediately to the canvas and will appear in the exported PNG.

---

## Step 8 — Export as PNG

Click **Export PNG** in the top-right corner. The file saves as `gantt-export.png` at 3840 × 2160 px (2× resolution), sharp enough for projected presentations and PowerPoint slides.

---

## Common mistakes

**"Header: missing required column"**  
Your column headers don't match what the tool expects. Check for typos, extra spaces, or a different separator (e.g. semicolons instead of commas). Copy the header row from the example above to be safe.

**"dimension 'X' not found — must be defined earlier"**  
An activity or marker references a dimension that hasn't been defined yet, or the name is spelled differently. Check that the dimension row appears before its activities, and that the spelling and capitalisation match exactly.

**"invalid date format — expected YYYY-MM-DD"**  
The date in that row is not in the right format. Numbers or Excel may have exported it in your local format (e.g. `15/03/2025`). Re-enter the dates as plain text in `YYYY-MM-DD` format.

**"activity requires End Date"**  
Activity rows need both a start and an end date. Marker rows only need a start date.

**"No data found"**  
The file is empty or the tool couldn't read it. Try re-exporting, making sure to choose UTF-8 encoding.

---

## Optional — Custom dimension color via CSV

If you want a dimension to use a specific color that isn't in the swatch palette, add a `Color` column to your spreadsheet header and put a hex color code on the dimension row:

```
Row Type,Dimension,Activity,Label,Start Date,End Date,Type,Notes,Color
dimension,Special Initiative,,,,,,, #9B59B6
activity,Special Initiative,Phase one,,2025-06-01,2025-09-30,bar,
```

Hex colors must be in `#RRGGBB` format (e.g. `#9B59B6`). If the value is invalid, the default palette color is used instead.
