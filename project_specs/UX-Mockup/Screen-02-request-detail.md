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
