# user story

## the user

Product and strategy manager at a privately held cardiac medical device company. Responsible for cross-functional project management, product launches, and executive alignment. Builds Gantt and swimlane charts repeatedly across project cycles — for strategy reviews, milestone tracking, and informing leadership on decision points.

Works on a managed enterprise machine with no admin rights and no access to internet-connected external tools.

---

## the current situation

Gantt charts are built in PowerPoint using native elements: rectangles for activity bars, circles for milestones, diamonds for decision points, text boxes for labels, lines for separators. Each element is placed and sized manually by dragging.

A typical chart session breaks down as follows:

- Setting up the frame, headers, and swimlane structure: 30–60 minutes
- Defining and validating the content (which activities, milestones, and decision points belong): 30–60 minutes  
- Each subsequent update iteration: 10–20 minutes, depending on the number of changes

The mechanical assembly — dragging boxes, adjusting sizes, changing colors, retyping labels — is the part that causes procrastination and delegation. It is time that should go toward the judgment work: what belongs on the chart, why this milestone, what decision is needed here, which teams are involved.

---

## tools tried and why they failed

**Online applications** — work well but are blocked by data policy. Project data cannot leave the machine.

**PowerPoint plugins** — technically allow chart creation and data upload, but the drag-and-drop interaction inside the plugin sandbox is too slow and unresponsive to be usable. Templates are rigid and hard to configure.

---

## what good looks like

The user opens a single file in Chrome. No installation. She has already done the thinking: the swimlanes, the activities, the milestones, the decision points, the dates. She enters that data into a CSV — the format is simple and consistent. She uploads the CSV. The chart appears. She adjusts the timeline scale if needed, toggles a swimlane's visibility, and exports a PNG. She pastes the PNG into PowerPoint. The whole process takes under 10 minutes.

The judgment stays with the user. The assembly is gone.

---

## chart structure (observed from real usage)

Charts have two levels of row hierarchy:

**Dimension** — a named group header, equivalent to a swimlane. Examples from real usage: Business Strategy, Product/Solution, Development via Partners, Go-to-Market.

**Activity** — a work item within a dimension. Has a start date and end date. Renders as a horizontal bar on the timeline.

In addition to activities, charts carry **floating markers** that sit at the dimension level rather than on a specific activity row. These are pinned to a date and carry a text label. Two types appear in practice:

- **Decision** — a highlighted text box, often with multi-line content noting which teams or regions are involved (e.g. "Approve US partnership; FR: entry decision after policy update; IT: identify KOLs")
- **Milestone** — a circle marker, single date, shorter label

Timeline scale varies by project: some charts run month by month, others quarter by quarter or year by year.

---

## constraints

- No admin rights on the work machine — cannot install software
- No internet access permitted for tools handling project data
- Drag-and-drop editing of chart elements is not required
- Export to native PowerPoint format is not required — PNG is sufficient
