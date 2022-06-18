---
id: qtutyha7eocui20l6e6hk74
title: Component Presets WBS
desc: ''
updated: 1655352939424
created: 1655343633937
---

Generate using `wbsm report`. Generates `wbs.project.html`.

Feature Definition Example:

```markdown
- Unlinked (group work item's not linked to a story) {story="null"}
- Higher-level user feature/story - [PRJ-001](https://github.com/brainlid/wbs_markdown/issues/1) (Initials) {story="001"}
```

Deliverable Item Examples:

```markdown
- [ ] incomplete item with estimated work size of 3 days {link=001 work=3d}
- [ ] explicitly set confidence value (as percent) {link=001 work=3d confidence=20}
- [ ] item with a note {link=001 work=3d note="shows up in the data table"}
- [x] completed work item, 2 hours estimated {link=001 work=2h}
- [x] completed work item, tracked actual time {link=001 work=2h actual=3.25h}
```

## Stories

chart {#stories-chart}

toggle {#stories-toggle}

- **000**: General admin & project management {story="000"}
- **001**: Icons for elements without presets {story="001"}
- **002**: Add an ‘educational’ link to explain how to find other components that don’t have a preset {story="002"}

### Group 1 - Global access to all components

- **101**: Users can browse components, presets, and preset categories {story=101 group="Group 1"}
- **102**: Users can’t delete non-user presets {story=102 group="Group 1"}
- **103**: If there aren’t saved presets show empty state {story=103 group="Group 1"}
- **104**: The presets label should show the preset category and link to it {story=104 group="Group 1"}
- **105**: Relocation of + to add pages {story="105"}

totals {#stories-total group="Group 1"}

### Group 2 - Contextual access to add a component

- **201**: Only components (and presets if they exist) available in context should be shown {story=201 group="Group 2"}
- **202**: Required fields, if they exist, are not presented to the user until they have chosen a component and a preset or just a component {story=202 group="Group 2"}
- **203**: If presets exist, it should not be possible to add a component without choosing a preset first.  {story=203 group="Group 2"}
- **204**: the add page UI/UX should be changed to use the new modal and flow implemented to add components {story="204"}

totals {#stories-total group="Group 2"}

### Total

totals {#stories-total}

## Component presets global access

filter {#display-filter}

style {#display-style}

level {#detail-level}

### Code

- Project
  - Initial Discovery
    - [x] Presets system implementation notes {work=1d link=000}
    - Estimate
      - [x] Product spec deep dive {work=1d link=000}
      - [ ] Create work breakdown structure {work=1d link=000}
  - New UI Components
    - [ ] Component presets side-panel button / actions {work=4h link="101"}
    - [ ] Component presets layout {work=4h link="101"}
    - [ ] Component presets list {work=1d link="101"}
    - [ ] Component presets filter (Search) {work=1d link="101"}
    - [ ] Component presets gallery (Search) {work=4h link="101"}
  - API Updates {new=true}
    - Database Migrations
  - Misc (uncategorized)
    - [ ] Add an ‘educational’ link to explain how to find other components that don’t have a preset {work=2h link="002"}
    - [ ] Create icon map {work=4h link="001"}
    - [ ] the add page UI/UX should be changed to use the new modal and flow implemented to add components {work=1d link="204"}
    - [ ] If presets exist, it should not be possible to add a component without choosing a preset first {work=4h link="203"}
    - [ ] Required fields, if they exist, are not presented to the user until...   {work=4h link="202"}
    - [ ] Only components (and presets if they exist) available in context should be shown {work=2d link="201"}
    - [ ] Users can’t delete non-user presets {work=2h link="102"}
    - [ ] If there aren’t saved presets show empty state {work=2h link="103"}
    - [ ] The presets label should show the preset category and link to it {work=2h link="104"}

### Administrative

- Administration
  - MR-1
    - Submission
      - [x] **105** pull/1463 {work=2h link=105}
    - Code reviewed and updated
      - [ ] **105** pull/1463 updated and merged {work=2h link=105}

## Raw Table Data

table {#stories-table}
