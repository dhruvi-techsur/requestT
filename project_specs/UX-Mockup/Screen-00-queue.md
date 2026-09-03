## Screen Designs

### Screen: Queue

**Purpose:** The fulfiller's primary workspace — see, filter, and act on all open requests.
**User Stories:** US-3.1, US-3.2, US-3.3, US-5.1

#### Layout

```
┌────────────────────────────────────────────────────────────┐
│ RequestT   [🔔]  [ + New request ]            [User ▾]      │
├───────────┬────────────────────────────────────────────────┤
│ Queue     │ Saved views: [My] [Unassigned] [Waiting] [Closed│
│ My reqs   │ Filters: Status▾ Assignee▾ Type▾ Priority▾ Date │
│ Reports   │ Search: [ 🔍 title, description, comments      ] │
│ Admin     ├────────────────────────────────────────────────┤
│           │ ☐  ID       Title            Status  Pri  Asgn  │
│           │ ☐ REQ-1042  Data pull Q3     New     High  —    │
│           │ ☐ REQ-1041  Design asset     In Prog Norm  Frank│
│           │ ☐ REQ-1039  Access request   Waiting High  Ada  │
│           │ [ Bulk: Assign▾  Status▾ ]   (n selected)       │
└───────────┴────────────────────────────────────────────────┘
```

#### Information Hierarchy

| Priority | Content | Placement |
|---|---|---|
| Primary | Request rows (ID, title, status, priority, assignee, age) | Main list |
| Secondary | Filters, saved views, search | Above the list |
| Tertiary | Bulk action bar | Appears on selection |

#### States

| State | Appearance | User Feedback |
|---|---|---|
| Default | Sorted list, priority→age | N/A |
| Loading | Row skeletons | "Loading queue…" |
| Success | Rows render <1s@500 | N/A |
| Error | Error banner | "Couldn't load the queue — retry" |
| Empty | Illustration + text | "No open requests match these filters" |

#### Interactive Elements

| Element | Type | Behavior |
|---|---|---|
| New request | Primary CTA | → `/requests/new` |
| Row | Navigation | → `/requests/:shortId` |
| Filter dropdowns | Input | Refine the list; combine |
| Saved view chip | Toggle | Applies a preset filter set |
| Bulk Assign/Status | Action | Applies to selected rows (note-gated) |
