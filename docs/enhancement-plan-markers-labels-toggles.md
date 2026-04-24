# Enhancement Plan: Marker Disambiguation, Label Handling, and Content Visibility Toggles

## Overview
This enhancement addresses UX friction in the swimlane Gantt chart: making marker types visually distinct, improving label readability when markers cluster, enhancing legend visibility, optimizing activity label placement, and adding granular content filters.

---

## 1. Marker Visual Distinction

### Current State
- Both milestone and decision markers render as filled Tangerine circles
- Legend differentiates them, but in-chart visuals are indistinguishable
- Users cannot quickly identify marker type without reading the label

### Proposed Solution
Create semantic visual distinction while maintaining design cohesion:

| Marker Type | Shape | Color | Fill | Stroke |
|---|---|---|---|---|
| **Milestone** | Circle (radius 7px) | Tangerine `#F04E23` | Filled | None |
| **Decision** | Diamond (rotated square, 14×14px) | Yellow `#FFC700` | Filled | None |

### Implementation Details
- **Milestone**: Keep existing circle rendering (no change)
- **Decision**: Replace circle with diamond
  - Render as a rotated square: `context.rotate(Math.PI/4)` to create 45° rotation
  - Dimensions: 14×14px (when rotated, creates 10px visual diagonal)
  - Centered on marker date, vertically centered in marker row
  - Same label positioning below shape as milestones (`y_label = cy + 10`)

### Design Rationale
- Diamond is a universally recognized decision point symbol (flowchart standard)
- Yellow provides strong color contrast without introducing new brand colors
- Both shapes are 7px in effective "radius," maintaining consistent sizing

---

## 2. Marker Label Crowding Improvement

### Current State
- Collision detection: markers within 24px horizontally trigger a vertical offset of 8px
- All colliding markers offset in the same direction (downward)
- Results in label stacking and readability issues when 3+ markers cluster

### Proposed Solution

#### Increase Collision Threshold & Offset
- **Collision distance**: 24px → 40px (triggers earlier, gives more breathing room)
- **Vertical offset**: 8px → 16px (more separation between labels)

#### Smart Directional Offset
- When a marker would collide, check available space above and below
- Offset in the direction with more room (minimize overlap with timeline/dimension rows)
- Algorithm:
  1. Check space below (from marker row bottom to bottom margin): if ample, offset +16px down
  2. Else check space above (from marker row top to timeline header): if ample, offset -16px up
  3. Fallback: offset +16px (current behavior)

### Implementation Details
- Modify collision detection in `drawChart()`, marker rendering block
- Add helper function to calculate available vertical space at marker's y position
- Track both collision X-position and applied offset direction for visual debugging

---

## 3. Legend Background Enhancement

### Current State
- Legend rendered directly on canvas with text labels only
- Blends visually with timeline header and dimension rows
- Lacks visual hierarchy/separation

### Proposed Solution
Add a subtle background container behind legend items:

- **Background**: White (`#FFFFFF`) with 1px border in Blue-Gray (`#587992`, 40% opacity)
- **Padding**: 8px horizontal, 6px vertical around legend content
- **Position**: Drawn before legend items, same coordinates, expanded for padding
- **Effect**: Subtle but intentional—stands out without competing with content

### Implementation Details
- Render rounded rectangle (3px corners) before legend text/icons
- Dimensions: measure all legend items, add 8px padding per side
- Z-order: background drawn first, then legend items on top

---

## 4. Activity Label Placement Logic

### Current State
- Fixed threshold: if `barWidth > 60px`, label inside; else label outside
- Does not account for actual label text length
- Results in narrow bars with short labels having outside placement, while longer labels in wider bars fit outside

### Proposed Solution
Adaptive placement based on measured text width:

1. Measure activity label text width at 11px font size
2. Check if label + padding (6px on each side) fits within bar width
3. If `textWidth + 12 <= barWidth`: place label inside (`x = barLeft + 6`)
4. Else: place label outside (`x = barRight + 6`)

### Implementation Details
- Use `renderCtx.measureText(label).width` to get actual width
- Account for font metrics and padding
- Handles dynamic label lengths without manual threshold tuning
- Better utilizes whitespace on wider bars

---

## 5. Content Visibility Toggles

### Current State
- Dimension visibility toggles exist (checkboxes for each dimension)
- Users can show/hide swimlanes but cannot filter activity bars, milestones, or decisions individually
- All content types always rendered

### Proposed Solution
Add three top-level toggle checkboxes for content filtering.

#### UI Placement
- **Location**: Top-left corner, below dimension toggles (same container)
- **Style**: Consistent with dimension toggles (checkbox + label)
- **Order**:
  1. Dimension toggles (existing)
  2. Content type toggles (new):
     - ☐ Show Activities
     - ☐ Show Milestones
     - ☐ Show Decisions

#### Default State
- All three enabled by default (full chart visibility)

#### Behavior
- Clicking toggle updates `state` object
- `drawChart()` skips rendering based on visibility flags:
  - `state.showActivities`: if false, skip all activity bars
  - `state.showMilestones`: if false, skip milestone markers
  - `state.showDecisions`: if false, skip decision markers
- Legend dynamically updates to show only visible content types

#### Implementation Details
- Add to `state` object: `showActivities: true`, `showMilestones: true`, `showDecisions: true`
- Create HTML checkboxes in `refreshVisibilityToggles()` function
- Add event listeners to update state and call `render()`
- In `drawChart()`, conditionally render:
  - Activities: `if (state.showActivities) { /* draw bars */ }`
  - Milestones: `if (state.showMilestones) { /* draw circles */ }`
  - Decisions: `if (state.showDecisions) { /* draw diamonds */ }`
- Update legend rendering to skip items not in visibility state

---

## 6. Legend Updates

### Legend Content Updates
- Legend now shows three distinct items:
  - Activity bar sample (existing dimension color)
  - Milestone circle (Tangerine `#F04E23`) + "Milestone" label
  - Decision diamond (Yellow `#FFC700`) + "Decision point" label

### Legend Visibility
- If `state.showActivities === false`, hide activity bar sample and label
- If `state.showMilestones === false`, hide milestone circle and label
- If `state.showDecisions === false`, hide decision diamond and label
- Legend dynamically reflows to show only visible items (recalculate x positions)

---

## Testing Checklist

- [ ] Upload `sample.csv`; verify milestone (Tangerine circle) and decision (yellow diamond) render correctly
- [ ] Verify marker label offsets when 3+ markers cluster in same row
- [ ] Confirm legend background is subtle and doesn’t obscure content
- [ ] Test activity label placement: short labels in wide bars → inside; long labels → outside
- [ ] Toggle each content type off; verify rendering stops and legend updates
- [ ] Test export (PNG): confirm all markers, labels, and legend appear correctly at 2× resolution
- [ ] Verify colors meet accessibility contrast ratios (especially yellow on white)

---

## Accessibility & Brand Notes

- **Yellow (`#FFC700`)**: Verify WCAG contrast ratio against white background (minimum 4.5:1 for text)
- **Diamond shape**: Validate against users with color blindness (shape distinction + color distinction)
- **Legend background**: Subtle styling maintains design hierarchy without competing for attention

---

## Rollout Notes

- All changes are contained in `drawChart()` and state management
- No CSV format changes required
- Backward compatible: existing charts render with new visuals
- Feature flag consideration: could wrap content toggles behind a user preference if complexity is a concern
