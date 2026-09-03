## F02: Status Lifecycle & Activity Log

**Description:** Requests move through a single linear-ish lifecycle. Any fulfiller can move a request to any status. Some transitions require a note. Every change is recorded in an append-only activity log.

**Terminology:**
- **Status:** One of New, Triaged, In Progress, Waiting on requester, Blocked, Done, Cancelled.
- **Append-only:** Activity entries are never edited or deleted, even for cancelled requests.

**Sub-features:**
- Status transitions (any fulfiller → any status)
- Required notes on Blocked (note), Cancelled (reason), Done (resolution note)
- 3-day auto-nudge when in Waiting on requester
- Append-only activity log with actor + timestamp

**Process:**
1. Fulfiller opens a request and selects a new status.
2. If target status requires a note/reason/resolution, system prompts for it and blocks the transition until provided.
3. System updates `status`, sets `closed_at` on Done/Cancelled, writes an activity entry (actor, old→new, timestamp).
4. System fires the "status changed" notification to the requester.
5. A scheduled job checks Waiting-on-requester requests; at 3 days it fires the requester nudge.

**Status definitions:**
- `New` — submitted, nobody has looked at it.
- `Triaged` — accepted, priority set, not started.
- `In Progress` — someone is actively working it.
- `Waiting on requester` — blocked pending an answer; auto-nudges requester at 3 days.
- `Blocked` — blocked on something else; requires a note.
- `Done` — closed with a resolution note.
- `Cancelled` — closed without work; requires a reason.

**Inputs:**
- `request_id` (uuid, required)
- `new_status` (enum, required)
- `note` (string, conditionally required for Blocked/Cancelled/Done)

**Outputs:**
- Updated request status (+ `closed_at` when terminal)
- Activity log entry
- Status-changed notification

**Validation:**
- Actor must be a Fulfiller or Admin.
- `new_status` must be a valid status.
- Blocked → note required; Cancelled → reason required; Done → resolution note required.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-fulfiller changes status | 403 | FORBIDDEN | "Only fulfillers can change status" |
| Missing required note | 422 | NOTE_REQUIRED | "A note is required for this status" |
| Invalid status value | 422 | INVALID_STATUS | "Unknown status" |

**API Surface (this feature):** see `Y1-api.md` §Requests, §Activity.

**Schema Surface (this feature):** uses `requests`, `activity` — see `Y0-schema.md`.
