## F07: Reporting & Export

**Description:** A single reporting page (no builder) showing throughput and cycle-time metrics, plus CSV export of any filtered queue view.

**Terminology:**
- **Time-to-close:** `closed_at - created_at` for Done requests.
- **p90:** 90th percentile of time-to-close.

**Sub-features:**
- Opened vs. closed by week (last 12 weeks)
- Open requests by status and by assignee
- Median and p90 time-to-close, overall and by type
- CSV export of a filtered queue view

**Process:**
1. Fulfiller/Admin opens the reporting page.
2. System computes aggregates from requests and activity data.
3. System renders the four report sections.
4. From any queue view (F03), user clicks Export; system streams a CSV of the current filtered set.

**Inputs:**
- Reporting page: none (uses full dataset, permission-scoped)
- Export: current `filters` + `q` from the queue view

**Outputs:**
- Rendered report widgets
- CSV file (columns match the queue view)

**Validation:**
- Reporting visible to Fulfiller/Admin only.
- Export reflects exactly the applied filters.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Requester opens reporting | 403 | FORBIDDEN | "Reporting is for team members" |
| Export too large | 413 | EXPORT_TOO_LARGE | "Narrow the filter and try again" |

**API Surface (this feature):** see `Y1-api.md` §Reports.

**Schema Surface (this feature):** reads `requests`, `activity` — see `Y0-schema.md`.
