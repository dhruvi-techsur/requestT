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
