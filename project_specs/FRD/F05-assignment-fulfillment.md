## F05: Assignment & Fulfillment

**Description:** Fulfillers pull work from the unassigned pool, assign requests to themselves or teammates, and close requests with a resolution note.

**Terminology:**
- **Unassigned:** A request with `assignee_id = null`.
- **Assignment change:** Setting or reassigning `assignee_id`, recorded in the activity log.

**Sub-features:**
- View unassigned requests
- Assign to self or a teammate
- Reassign
- Close with resolution note (via F02 Done transition)

**Process:**
1. Fulfiller filters the queue to unassigned (F03).
2. Fulfiller assigns a request to self or a teammate.
3. System sets `assignee_id`, writes an activity entry, fires "assigned to you" to the assignee.
4. To close, fulfiller sets status to Done (F02) with a resolution note.

**Inputs:**
- `request_id` (uuid, required)
- `assignee_id` (uuid | null, required for assign/unassign)

**Outputs:**
- Updated `assignee_id`
- Activity log entry
- "Assigned to you" notification

**Validation:**
- Actor must be Fulfiller/Admin.
- `assignee_id` must reference a Fulfiller/Admin (or null to unassign).

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-fulfiller assigns | 403 | FORBIDDEN | "Only fulfillers can assign" |
| Assignee not a fulfiller | 422 | INVALID_ASSIGNEE | "Assignee must be a team member" |
| Request not found | 404 | NOT_FOUND | "Request not found" |

**API Surface (this feature):** see `Y1-api.md` §Requests (assign).

**Schema Surface (this feature):** uses `requests`, `activity` — see `Y0-schema.md`.
