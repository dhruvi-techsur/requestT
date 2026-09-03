### Screen: Reporting

**Purpose:** One-page volume and cycle-time view for fulfillers/admins.
**User Stories:** US-7.1, US-7.2

#### Layout

```
┌────────────────────────────────────────────────────────────┐
│ Reports                                    [ Export CSV ]   │
├────────────────────────────────────────────────────────────┤
│ Opened vs Closed (12 wks)   ▇▇▅▆▇▅▆▇▇▆▅▇                     │
│ Open by status  [New 4][Triaged 3][In Prog 6][Waiting 2]…   │
│ Open by assignee [Frank 5][Ada 3][Unassigned 7]             │
│ Time-to-close   Median 2.1d · p90 6.4d   (by type ▾)        │
└────────────────────────────────────────────────────────────┘
```

#### Information Hierarchy

| Priority | Content | Placement |
|---|---|---|
| Primary | Opened vs closed trend; time-to-close | Top |
| Secondary | Open by status / assignee | Middle |
| Tertiary | Export action, by-type breakdown | Header / toggle |

#### States

| State | Appearance | User Feedback |
|---|---|---|
| Default | Charts + stats rendered | N/A |
| Loading | Chart skeletons | "Crunching numbers…" |
| Success | Export downloads | Toast: "CSV downloaded" |
| Error | Error banner | "Couldn't build report" / "Narrow the filter" (export too large) |
| Empty | No closed requests yet | "No data yet — close some requests to see cycle time" |

#### Interactive Elements

| Element | Type | Behavior |
|---|---|---|
| Export CSV | Action | Streams the current filtered view; requester-forbidden |
| By-type toggle | Input | Recomputes time-to-close per type |

### Screen: Admin Console

**Purpose:** Manage request types, custom-field schemas, categories, and user roles.
**User Stories:** US-8.1, US-8.2

#### Layout

```
┌────────────────────────────────────────────────────────────┐
│ Admin   [ Request types | Categories | Users ]              │
├────────────────────────────────────────────────────────────┤
│ Request types           [ + New type ]                     │
│  • Data pull      (active)   fields: 3   [Edit]            │
│  • Design ask     (active)   fields: 2   [Edit]            │
│  • Access request (inactive) fields: 4   [Edit]           │
│  Field editor: [name][type▾][required☐][options…] (≤5)     │
└────────────────────────────────────────────────────────────┘
```

#### States

| State | Appearance | User Feedback |
|---|---|---|
| Default | Type list | N/A |
| Success | Type saved/activated | Toast: "Saved" |
| Error | Schema validation | "A type can have at most 5 custom fields" |
| Empty | No types yet | "Create your first request type" |

#### Interactive Elements

| Element | Type | Behavior |
|---|---|---|
| New type | Primary CTA | Opens field editor |
| Type row Edit | Navigation | Edits schema / active flag |
| User role select | Input | Changes role (admin only) |
