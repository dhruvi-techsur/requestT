## F01: Submit a Request

**Description:** Any org member submits a request through a short form in under a minute. On submit the request receives a human-readable ID, `New` status, and a timestamp, and the requester gets a confirmation.

**Terminology:**
- **Short ID:** Human-readable request identifier, e.g. `REQ-1042`.
- **Custom field values:** Values for the RequestType's admin-defined fields.

**Sub-features:**
- Core submit form (title, description, type, suggested priority)
- Type-driven custom fields (up to 5: text, number, select, date)
- Attachments (up to 5 files, 25 MB each)
- Post-submit edit to add a missing detail

**Process:**
1. Requester opens the submit form.
2. Requester selects a request type; the form renders that type's custom fields.
3. Requester fills title, description, suggested priority, custom fields; optionally attaches files.
4. On submit, system validates, assigns the next short ID, sets status `New`, stamps `created_at`.
5. System records a creation entry in the activity log.
6. System sends confirmation to the requester and fires the "request submitted" notification to fulfillers.
7. Requester may later edit fields or add custom-field values on their own open request.

**Inputs:**
- `title` (string, required)
- `description` (string, required)
- `type_id` (uuid, required — from active RequestTypes)
- `priority` (enum, required — requester-suggested)
- `custom_field_values` (object, optional — validated against type schema)
- `attachments` (file[], optional — ≤5 files, ≤25 MB each)

**Outputs:**
- Created Request with `short_id`, `status = New`, `created_at`
- Confirmation to requester
- Activity log entry (create)

**Validation:**
- `title` and `description` required, non-empty.
- `type_id` must reference an active RequestType.
- `priority` must be a valid value.
- Custom field values must match the type's schema (types, required, select options).
- Attachments: max 5, each ≤25 MB.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Missing required field | 422 | VALIDATION_ERROR | "Title and description are required" |
| Inactive/unknown type | 422 | INVALID_TYPE | "Select a valid request type" |
| Custom field mismatch | 422 | CUSTOM_FIELD_ERROR | "One or more fields are invalid" |
| Attachment too large | 413 | FILE_TOO_LARGE | "Files must be 25 MB or smaller" |
| Too many attachments | 422 | TOO_MANY_FILES | "Attach at most 5 files" |

**API Surface (this feature):** see `Y1-api.md` §Requests.

**Schema Surface (this feature):** uses `requests`, `request_types`, `attachments`, `activity` — see `Y0-schema.md`.
