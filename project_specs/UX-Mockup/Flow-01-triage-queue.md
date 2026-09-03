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
