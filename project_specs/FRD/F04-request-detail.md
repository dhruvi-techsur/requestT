## F04: Request Detail

**Description:** The full record for a single request: header metadata, body, comments (with an internal-only toggle for fulfillers), and the activity log.

**Terminology:**
- **Participant:** Requester, assignee, and any commenter on a request — recipients of "new comment" notifications.
- **Internal comment:** Comment flagged `internal = true`, hidden from the Requester.

**Sub-features:**
- Header (ID, title, status, priority, type, requester, assignee, created/updated)
- Body (description, custom fields, attachments)
- Comments (chronological, plain text + attachments, internal-only toggle)
- Activity log (status changes, assignment changes, field edits)

**Process:**
1. User opens a request by ID.
2. System authorizes: Requesters only see their own requests and non-internal comments.
3. System renders header, body, permitted comments, and activity log.
4. A participant adds a comment; fulfillers may mark it internal.
5. System stores the comment, fires "new comment" to other participants (respecting internal visibility), and appends nothing to activity for comments (comments are their own stream).

**Inputs:**
- `request_id` (uuid, required)
- Comment: `body` (string, required), `internal` (bool, fulfiller-only), `attachments` (optional)

**Outputs:**
- Full request detail (permission-scoped)
- Stored comment + notification

**Validation:**
- Requester cannot view internal comments or others' requests.
- `internal = true` only settable by Fulfiller/Admin.
- Comment body non-empty.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Unauthorized request access | 403 | FORBIDDEN | "You don't have access to this request" |
| Requester sets internal flag | 403 | FORBIDDEN | "Only fulfillers can post internal comments" |
| Empty comment | 422 | VALIDATION_ERROR | "Comment can't be empty" |
| Request not found | 404 | NOT_FOUND | "Request not found" |

**API Surface (this feature):** see `Y1-api.md` §Requests, §Comments.

**Schema Surface (this feature):** uses `requests`, `comments`, `attachments`, `activity` — see `Y0-schema.md`.
