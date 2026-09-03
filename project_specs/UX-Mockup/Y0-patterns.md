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
