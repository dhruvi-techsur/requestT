# UX Mockup: RequestT

**Project:** RequestT
**Generated:** 2026-09-03
**Based on:** UserStories-RequestT.md, PRD-RequestT.md, FRD-RequestT.md

---

## Overview

RequestT is a desktop-first internal web app, usable on a phone. The design serves two very different mental models from one shell: a **Requester** who wants a fast form and a status they can check, and a **Fulfiller** who lives in a triage queue. The UX principles:

- **Fast, low-friction submission.** The submit form is short, type-driven, and gives an immediate ID.
- **The queue is the workspace.** Filters, saved views, and bulk actions make five-minute triage possible.
- **Status is always visible.** Every request surfaces its status, assignee, and history without a click chain.
- **Requesters never see internal content.** Internal comments and admin surfaces are role-gated in the UI and the API.
- **Accessible by default.** Submit form and queue are keyboard-navigable and meet WCAG 2.1 AA.

---

## Navigation Map

| Screen | Route | Reached from | Nav element |
|---|---|---|---|
| Sign-in redirect | `/login` | App shell (unauthenticated) | Automatic redirect to IdP |
| Queue | `/requests` | App shell | Sidebar: "Queue" (default landing for fulfillers) |
| Submit request | `/requests/new` | App shell / Queue | Header button: "New request" |
| Request detail | `/requests/:shortId` | Queue (row click) / My requests / notification link | Row click / notification deep-link |
| My requests | `/my-requests` | App shell | Sidebar: "My requests" (default landing for requesters) |
| Reporting | `/reports` | App shell (fulfiller/admin) | Sidebar: "Reports" |
| Admin console | `/admin` | App shell (admin) | Sidebar: "Admin" |
| Notifications tray | (overlay) | App shell | Header bell icon |

**No orphan screens:** every screen above is reachable from the app shell sidebar/header or a reachable parent (Queue → Request detail). Detail is also reachable via notification deep-links, whose parent (the notification tray) traces to the shell.
## User Flows

### Flow 0: Submit a Request

**Trigger:** Requester clicks "New request" from the header.
**User Story:** US-1.1, US-1.2

```
[New request button]
    │
    ▼
[Select request type] ── renders type's custom fields
    │
    ▼
[Fill title, description, priority, custom fields; attach files]
    │
    ├── Valid ──▶ [Submit] ──▶ [Confirmation: REQ-#### created] ──▶ [Request detail]
    │
    └── Invalid ──▶ [Inline validation errors] ──▶ (stay on form)
```

**Steps:**
1. Requester picks a type from a dropdown; the form re-renders that type's up-to-5 custom fields.
2. Requester fills required title + description, suggests a priority, completes custom fields.
3. Optional: drag-drop or browse up to 5 files (≤25 MB each); oversized/extra files rejected inline.
4. On Submit, a success toast shows the `REQ-####` ID and the app routes to the new request's detail.
### Flow 1: Triage the Queue

**Trigger:** Fulfiller lands on the Queue (default landing).
**User Story:** US-3.1, US-3.2, US-3.3, US-5.1

```
[Queue: open requests, priority→age]
    │
    ├── Apply "Unassigned" saved view ──▶ [Filtered list]
    │
    ├── Filter/search ──▶ [Filtered list]
    │
    ├── Select rows ──▶ [Bulk assign / bulk status] ──▶ [Updated rows + per-item result]
    │
    └── Click a row ──▶ [Request detail]
```

**Steps:**
1. Queue loads open requests sorted by priority then age (<1s at 500 open).
2. Fulfiller applies a saved view or ad-hoc filters (status, assignee incl. unassigned, type, priority, date) and/or free-text search.
3. Fulfiller selects rows to bulk-assign or bulk-change status (note-gated per request; 207 partial results surfaced).
4. Clicking a row opens the request detail to work it.

### Flow 2: Work and Close a Request

**Trigger:** Fulfiller opens a request from the queue.
**User Story:** US-2.1, US-4.2, US-5.2

```
[Request detail]
    │
    ├── Assign to me ──▶ [Assignee set, activity logged]
    │
    ├── Change status ──▶ [If Blocked/Cancelled/Done: require note] ──▶ [Logged]
    │
    ├── Comment ──▶ [Optional internal toggle (fulfiller only)]
    │
    └── Set Done + resolution note ──▶ [Closed, requester notified]
```

**Steps:**
1. Fulfiller assigns the request to self; activity log records it.
2. Fulfiller advances status; note/reason/resolution prompted where required.
3. Fulfiller comments to the requester, or marks a comment internal.
4. On Done, a required resolution note closes the request and notifies the requester.

### Flow 3: Configure a Request Type (Admin)

**Trigger:** Admin opens Admin → Request types.
**User Story:** US-8.1

```
[Admin: Request types]
    │
    ▼
[New/Edit type: name, description]
    │
    ▼
[Add custom fields (≤5): text/number/select/date; select needs options]
    │
    ├── Valid ──▶ [Save] ──▶ [Set active] ──▶ [Appears on submit form]
    │
    └── Invalid ──▶ [Schema error message]
```
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
### Screen: Submit Request

**Purpose:** Let a requester log an ask in under a minute.
**User Stories:** US-1.1, US-1.2, US-1.3

#### Layout

```
┌────────────────────────────────────────────┐
│ New request                        [ ✕ ]   │
├────────────────────────────────────────────┤
│ Request type *   [ Select a type      ▾ ]  │
│ Title *          [                       ]  │
│ Description *    [                        ] │
│                  [                        ] │
│ Priority         ( ) Low ( ) Normal (•) Hi  │
│ ── custom fields (per type) ──              │
│ [ field 1 ] [ field 2 ] …                   │
│ Attachments      [ + Add files ] (≤5, 25MB) │
│                                             │
│                      [ Cancel ] [ Submit ]  │
└────────────────────────────────────────────┘
```

#### Information Hierarchy

| Priority | Content | Placement |
|---|---|---|
| Primary | Type, title, description | Top of form |
| Secondary | Priority, custom fields | Middle |
| Tertiary | Attachments | Bottom |

#### States

| State | Appearance | User Feedback |
|---|---|---|
| Default | Empty form, type unselected | N/A |
| Loading | Submit button spinner | "Submitting…" |
| Success | Redirect to detail | Toast: "REQ-#### created" |
| Error | Inline field errors | "Title and description are required" |
| Empty | (initial) custom fields hidden until type chosen | Prompt: "Pick a type to continue" |

#### Interactive Elements

| Element | Type | Behavior |
|---|---|---|
| Type dropdown | Input | Re-renders custom fields for the type |
| Attachments | File input | Enforces ≤5 files, ≤25 MB each |
| Submit | Primary CTA | Validates, creates request, routes to detail |
### Screen: Request Detail

**Purpose:** The full record — metadata, body, comments, and activity — and the place fulfillers work a request.
**User Stories:** US-4.1, US-4.2, US-2.1, US-5.1, US-5.2

#### Layout

```
┌────────────────────────────────────────────────────────────┐
│ REQ-1042  Data pull Q3                    [Status: New ▾]   │
│ Priority: High   Type: Data   Requester: Priya  Assignee:▾ │
│ Created 2h ago · Updated 2h ago                            │
├───────────────────────────────┬────────────────────────────┤
│ Description                    │ Activity log               │
│  …request body…                │  • New (Priya, 2h)         │
│  Custom fields: [range] …      │  • Assigned Frank (Frank)  │
│  Attachments: file1.csv        │  • Status → In Progress    │
│                                │                            │
│ Comments                       │                            │
│  Frank: "What date range?"     │                            │
│  [ Add comment ] [☐ Internal ] │                            │
└───────────────────────────────┴────────────────────────────┘
```

#### Information Hierarchy

| Priority | Content | Placement |
|---|---|---|
| Primary | Status control, title/ID, assignee | Header |
| Secondary | Description, custom fields, comments | Left column |
| Tertiary | Activity log, attachments | Right column |

#### States

| State | Appearance | User Feedback |
|---|---|---|
| Default | Full detail (permission-scoped) | N/A |
| Loading | Skeleton header + panels | "Loading…" |
| Success | Status/assignee update in place | Toast: "Updated" |
| Error | Note-required modal / error banner | "A note is required for this status" |
| Empty | No comments yet | "No comments yet" |

#### Interactive Elements

| Element | Type | Behavior |
|---|---|---|
| Status dropdown | Action | Changes status; prompts note for Blocked/Cancelled/Done (fulfiller only) |
| Assignee dropdown | Action | Assigns/reassigns (fulfiller only); logs + notifies |
| Add comment | Input | Posts comment; Internal toggle visible only to fulfillers |
| Attachment link | Navigation | Opens via presigned URL |
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
## Interaction Patterns

### Pattern: Note-Gated Status Change
**When to use:** Any transition to Blocked, Cancelled, or Done.
**Behavior:** Selecting the status opens an inline prompt requiring a note/reason/resolution; the transition is disabled until the note is provided.
**Examples:** Request detail status dropdown; queue bulk status change.

### Pattern: Type-Driven Form Fields
**When to use:** Submit form and admin schema editor.
**Behavior:** Selecting a request type dynamically renders that type's custom fields; changing type re-renders and preserves compatible values.
**Examples:** Submit request screen.

### Pattern: Saved View Chips
**When to use:** Queue filtering.
**Behavior:** One-click chips apply preset filter sets; the active chip is highlighted; ad-hoc filters can layer on top.
**Examples:** Queue (My, Unassigned, Waiting, Closed this month).

### Pattern: Bulk Action Bar
**When to use:** Multiple queue rows selected.
**Behavior:** A contextual bar appears with Assign/Status actions; results report per-item success/failure (HTTP 207).
**Examples:** Queue.

### Pattern: Internal-Only Toggle
**When to use:** Fulfiller comments.
**Behavior:** A checkbox marks a comment internal; internal comments are visually distinct and never rendered for requesters.
**Examples:** Request detail comments.

### Pattern: Toast + Deep-Link Notifications
**When to use:** Action confirmations and in-app notifications.
**Behavior:** Transient toasts confirm actions; the notification tray lists events, each deep-linking to the relevant request.
**Examples:** Submit confirmation, assignment, status change.
## Responsive Considerations

### Desktop (>1024px)
- Two-column request detail (body left, activity right).
- Queue shows full column set (ID, title, status, priority, assignee, type, age).
- Persistent left sidebar navigation.

### Tablet (768px–1024px)
- Request detail collapses to a single column with a tabbed Body / Activity switch.
- Queue drops the Type column into a row-expand affordance; filters move into a "Filters" popover.
- Sidebar collapses to icons.

### Mobile (<768px)
- Submit form is the priority experience: single-column, large tap targets, sticky Submit.
- Queue becomes a card list (ID + title + status + priority badges); filters in a bottom sheet.
- Bulk actions hidden on mobile (triage is a desktop task); single-row actions remain.
- Request detail is fully stacked; comment composer sticky at the bottom.
## Accessibility Notes

Target: **WCAG 2.1 AA**, with the submit form and queue view as the enforced-priority surfaces (per PRD NFR).

- **Color contrast:** All text and status badges meet ≥4.5:1 (normal) / ≥3:1 (large). Status is never conveyed by color alone — each badge pairs a label with its color (e.g., "Blocked" text + icon).
- **Keyboard navigation:** Full keyboard operability on submit form and queue — logical tab order, visible focus rings, Enter/Space activation. Bulk-select via keyboard; saved-view chips are focusable buttons. No keyboard traps in modals (status-note prompt, field editor).
- **Screen reader:** Queue is a proper table/grid with header associations; each row announces ID, title, status, priority, assignee. The activity log is an ordered list with time stamps read in context. Toasts use an ARIA live region (polite).
- **ARIA labels:** Icon-only controls (bell/notifications, ✕ close, bulk actions) carry `aria-label`. The type-driven form associates each custom field with its `<label>`; validation errors use `aria-describedby` and `aria-invalid`.
- **Forms:** Required fields marked programmatically (not just visually); inline errors are announced and focus moves to the first error on failed submit.
- **Motion/timing:** No essential info conveyed only via animation; the 3-day/7-day nudges are server-driven and never require the user to act within a client-side timer.
