# Requirements: RequestT

**Defined:** 2026-09-03
**Core Value:** Every request has one canonical, status-tracked record that a requester can check without asking a human and the team can triage in under five minutes a day.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Authentication & Roles (F0)

- [ ] **AUTH-01**: User can sign in via the org's SSO identity provider (no separate password)
- [ ] **AUTH-02**: System resolves additive roles (Requester ⊂ Fulfiller ⊂ Admin)
- [ ] **AUTH-03**: Requester can access only their own requests and non-internal comments

### Submit (F1)

- [ ] **SUBMIT-01**: Requester can submit a request with required title, description, and type in under a minute
- [ ] **SUBMIT-02**: On submit, the request gets a `REQ-####` ID, `New` status, timestamp, and confirmation
- [ ] **SUBMIT-03**: Request type renders up to 5 admin-defined custom fields (text, number, select, date)
- [ ] **SUBMIT-04**: Requester can attach up to 5 files (25 MB each) to a request
- [ ] **SUBMIT-05**: Requester can add or edit detail on their own open request

### Lifecycle & Audit (F2)

- [ ] **LIFE-01**: Fulfiller can move a request to any status (New, Triaged, In Progress, Waiting on requester, Blocked, Done, Cancelled)
- [ ] **LIFE-02**: Blocked requires a note, Cancelled requires a reason, Done requires a resolution note
- [ ] **LIFE-03**: A request in Waiting on requester for 3 days auto-nudges the requester
- [ ] **LIFE-04**: Every status/assignment/field change appends an actor+timestamp entry to an append-only activity log

### Queue (F3)

- [ ] **QUEUE-01**: Default view lists open requests sorted by priority then age, loading under 1s at 500 open
- [ ] **QUEUE-02**: User can filter by status, assignee (incl. unassigned), type, priority, and date range
- [ ] **QUEUE-03**: User can free-text search across title, description, and comments
- [ ] **QUEUE-04**: Saved views exist for My requests, Unassigned, Waiting on requester, Closed this month
- [ ] **QUEUE-05**: Fulfiller can bulk-assign and bulk-change status on selected rows

### Request Detail (F4)

- [ ] **DETAIL-01**: Request detail shows header (ID, title, status, priority, type, requester, assignee, dates), body, and activity log
- [ ] **DETAIL-02**: Participants can add chronological comments with attachments; fulfillers can mark a comment internal
- [ ] **DETAIL-03**: Internal comments are never shown to requesters

### Assignment (F5)

- [ ] **ASSIGN-01**: Fulfiller can view unassigned requests and assign to self or a teammate
- [ ] **ASSIGN-02**: Fulfiller can close a request with a resolution note (sets Done, records closed_at)

### Notifications (F6)

- [ ] **NOTIF-01**: Users receive email + in-app notifications for submit, assignment, status change, and new comment events
- [ ] **NOTIF-02**: Scheduled nudges fire for 3-day waiting-on-requester and 7-day untouched requests
- [ ] **NOTIF-03**: Each user can mute notifications per category

### Reporting (F7)

- [ ] **REPORT-01**: Reporting page shows opened vs. closed by week (12 weeks) and open by status and assignee
- [ ] **REPORT-02**: Reporting page shows median and p90 time-to-close, overall and by type
- [ ] **REPORT-03**: User can export any filtered queue view to CSV

### Administration (F8)

- [ ] **ADMIN-01**: Admin can create, edit, and deactivate request types with their custom-field schemas (≤5 fields)
- [ ] **ADMIN-02**: Admin can manage categories
- [ ] **ADMIN-03**: Admin can manage user roles

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Intake & Automation

- **V2-01**: Slack integration for submitting and status updates
- **V2-02**: Email-to-request intake
- **V2-03**: Recurring requests
- **V2-04**: Templates and canned responses
- **V2-05**: Custom statuses per request type
- **V2-06**: SLA targets with breach alerts
- **V2-07**: Requester satisfaction survey on close

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Project management (subtasks, dependencies, Gantt, sprints) | Tracker, not a PM tool |
| Discussion-thread chat | Comments clarify, not converse |
| SLA engine / escalation / automation builder | Keep v1 lightweight |
| Time tracking / billing | Not part of the tracking problem |
| Public-facing customer portal | Internal tool only |
| Multi-tenancy / multiple queues | Single org, single shared queue for v1 |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Pending |
| AUTH-02 | Phase 1 | Pending |
| AUTH-03 | Phase 1 | Pending |
| SUBMIT-01 | Phase 2 | Pending |
| SUBMIT-02 | Phase 2 | Pending |
| SUBMIT-03 | Phase 2 | Pending |
| SUBMIT-04 | Phase 2 | Pending |
| SUBMIT-05 | Phase 2 | Pending |
| LIFE-01 | Phase 3 | Pending |
| LIFE-02 | Phase 3 | Pending |
| LIFE-03 | Phase 3 | Pending |
| LIFE-04 | Phase 3 | Pending |
| QUEUE-01 | Phase 3 | Pending |
| QUEUE-02 | Phase 3 | Pending |
| QUEUE-03 | Phase 3 | Pending |
| QUEUE-04 | Phase 3 | Pending |
| QUEUE-05 | Phase 3 | Pending |
| DETAIL-01 | Phase 3 | Pending |
| DETAIL-02 | Phase 3 | Pending |
| DETAIL-03 | Phase 3 | Pending |
| ASSIGN-01 | Phase 3 | Pending |
| ASSIGN-02 | Phase 3 | Pending |
| NOTIF-01 | Phase 4 | Pending |
| NOTIF-02 | Phase 4 | Pending |
| NOTIF-03 | Phase 4 | Pending |
| REPORT-01 | Phase 5 | Pending |
| REPORT-02 | Phase 5 | Pending |
| REPORT-03 | Phase 5 | Pending |
| ADMIN-01 | Phase 6 | Pending |
| ADMIN-02 | Phase 6 | Pending |
| ADMIN-03 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 31 total
- Mapped to phases: 31
- Unmapped: 0 ✓

---
*Requirements defined: 2026-09-03*
*Last updated: 2026-09-03 after initial definition*
