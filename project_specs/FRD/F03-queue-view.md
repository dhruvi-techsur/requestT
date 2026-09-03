## F03: Queue View

**Description:** The team's working surface. Shows all open requests, sortable, filterable, and searchable, with saved views and bulk actions. Must load in under 1s at 500 open requests.

**Terminology:**
- **Saved view:** A named, reusable filter set (My requests, Unassigned, Waiting on requester, Closed this month).
- **Bulk action:** Assign or status-change applied to multiple selected rows at once.

**Sub-features:**
- Default view: all open requests, sorted by priority then age
- Filters: status, assignee (incl. unassigned), type, priority, date range
- Free-text search across title, description, comments
- Saved views (system + per-user)
- Bulk assign / bulk status change

**Process:**
1. User opens the queue; system loads open requests sorted by priority then age.
2. User applies filters and/or search; system returns matching rows (permission-scoped for Requesters).
3. User optionally selects a saved view to apply a preset filter set.
4. Fulfiller selects rows and performs a bulk assign or bulk status change.
5. Bulk actions write an activity entry per affected request and fire notifications.

**Inputs:**
- `filters` (object, optional: status, assignee, type, priority, date_range)
- `q` (string, optional — free-text search)
- `saved_view` (id, optional)
- `selection` + `bulk_action` (for bulk operations)

**Outputs:**
- Paginated, sorted list of requests
- Applied filter state
- Bulk-action results (per-request success/failure)

**Validation:**
- Requester queries are always scoped to their own requests.
- Bulk status change obeys F02 note requirements per request.
- Date range must be a valid interval.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Invalid filter value | 422 | VALIDATION_ERROR | "Invalid filter" |
| Bulk action partial failure | 207 | BULK_PARTIAL | "Some requests could not be updated" |
| Requester bulk action | 403 | FORBIDDEN | "Only fulfillers can bulk-update" |

**API Surface (this feature):** see `Y1-api.md` §Requests (list), §SavedViews.

**Schema Surface (this feature):** uses `requests`, `saved_views` — see `Y0-schema.md`.
