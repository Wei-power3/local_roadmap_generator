# CSV format

## columns

```
Row Type, Dimension, Activity, Label, Start Date, End Date, Type, Notes
```

| column | required | description |
|---|---|---|
| Row Type | yes | one of: `dimension`, `activity`, `marker` |
| Dimension | yes for activity and marker rows | must match the name of a dimension row defined earlier in the file |
| Activity | yes for activity rows | the label shown on the chart bar |
| Label | yes for marker rows | the text shown in the milestone or decision marker |
| Start Date | yes for activity and marker rows | format: `YYYY-MM-DD` |
| End Date | yes for activity rows | format: `YYYY-MM-DD` — must be on or after Start Date |
| Type | yes | one of: `bar`, `milestone`, `decision` |
| Notes | optional | additional text shown in decision markers; supports line breaks using `\n` |

---

## row types

**`dimension`** — defines a swimlane group header. Must appear in the file before any activity or marker rows that reference it. The Dimension column and the Activity/Label columns are left blank. A Color column can be added to override the default palette for this swimlane.

**`activity`** — a work item with a date span. Renders as a horizontal bar within the named dimension. Requires Start Date and End Date. Type must be `bar`.

**`marker`** — a point-in-time annotation that floats at the dimension level, not on a specific activity row. Pinned to Start Date. End Date is ignored. Two types:
- `milestone` — a circle marker with a short label
- `decision` — a highlighted text box, supports multi-line notes

---

## date format

All dates must use `YYYY-MM-DD`. Example: `2025-04-01`.

Mixed formats in the same file will produce a validation error.

---

## color override (optional)

To override the default BIOTRONIK color palette for a specific dimension, add a `Color` column to the dimension row and provide a hex value.

```
Row Type, Dimension, Activity, Label, Start Date, End Date, Type, Notes, Color
dimension, Business Strategy, , , , , , , #00234C
```

If the Color column is absent or blank for a dimension row, the default palette applies.

---

## full example

```csv
Row Type,Dimension,Activity,Label,Start Date,End Date,Type,Notes
dimension,Business Strategy,,,,,,
activity,Business Strategy,vision and ambition,,2025-01-01,2025-06-30,bar,
activity,Business Strategy,market prioritization,,2025-02-01,2025-07-31,bar,
activity,Business Strategy,business model,,2025-03-01,2025-09-30,bar,
marker,Business Strategy,,,2025-01-15,,decision,"Approve HF telemonitoring strategy and allocate resources"
dimension,Product/Solution,,,,,,
activity,Product/Solution,customer problem discovery,,2025-01-01,2025-03-31,bar,
activity,Product/Solution,solution concept and design,,2025-02-01,2025-05-31,bar,
activity,Product/Solution,offering definition,,2025-04-01,2025-06-30,bar,
activity,Product/Solution,product roadmap,,2025-05-01,2025-09-30,bar,
activity,Product/Solution,validation,,2025-06-01,2025-10-31,bar,
marker,Product/Solution,,,2025-04-01,,milestone,Offering definition locked
dimension,Development via Partners,,,,,,
activity,Development via Partners,solution development or adjustment,,2025-04-01,2025-12-31,bar,
marker,Development via Partners,,,2025-04-01,,decision,"Approve US partnership\nFR: entry decision after policy update\nIT: identify KOLs"
marker,Development via Partners,,,2025-04-01,,milestone,US platform development starts
dimension,Go-to-Market,,,,,,
activity,Go-to-Market,Sales,,2025-09-01,2025-12-31,bar,
activity,Go-to-Market,Marketing,,2025-08-01,2025-12-31,bar,
```

---

## validation rules

The parser checks these before rendering. Any failure produces a row-level error message and blocks rendering.

- `Row Type` must be one of `dimension`, `activity`, `marker`
- `activity` and `marker` rows must reference a Dimension that was defined earlier in the file
- `Start Date` and `End Date` must be in `YYYY-MM-DD` format
- `End Date` must be on or after `Start Date` for activity rows
- `marker` rows must have a `Start Date`
- `Type` on activity rows must be `bar`
- `Type` on marker rows must be `milestone` or `decision`
- File must have at least one activity row to render

---

## common mistakes

**Dimension name mismatch** — if an activity row says `Dimension = Business strategy` (lowercase s) but the dimension row says `Business Strategy`, the activity will be skipped. Names are case-sensitive.

**Missing End Date on activity** — activity rows require both Start Date and End Date. Milestone and decision markers use only Start Date.

**Wrong date format** — `01/15/2025` or `15-01-2025` will fail. Use `2025-01-15`.
