# Roadmap: RequestT

## Overview

RequestT is built by first standing up the foundation everyone depends on (SSO, roles, the request record, and a default request type), then delivering the core end-to-end loop — submit a request, triage it in a queue, work it through its audited lifecycle, and close it. Once the loop is complete and trustworthy, we layer on notifications, self-service reporting, and admin configuration. Phases are ordered so each one leaves a coherent, usable slice: after Phase 3 a requester can submit and a fulfiller can triage and close; later phases make it fast, visible, and configurable.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [ ] **Phase 1: Foundation & Access** - SSO auth, roles, the request record, and a default request type
- [ ] **Phase 2: Submit & Capture** - The submit form, custom fields, attachments, and post-submit edits
- [ ] **Phase 3: Triage, Lifecycle & Fulfillment** - Queue, status lifecycle with audit, request detail, assignment, and close
- [ ] **Phase 4: Notifications** - Email + in-app events, per-category mute, and scheduled nudges
- [ ] **Phase 5: Reporting & Export** - Throughput and cycle-time page plus CSV export
- [ ] **Phase 6: Administration** - Request types, custom-field schemas, categories, and role management

## Phase Details

### Phase 1: Foundation & Access
**Goal**: Users sign in via SSO with the right role, the core request data model exists, and a default request type is available so requests can be created.
**Depends on**: Nothing (first phase)
**Requirements**: AUTH-01, AUTH-02, AUTH-03
**Success Criteria** (what must be TRUE):
  1. A user can sign in through the org SSO provider and get a session
  2. The system resolves the user's role (Requester, Fulfiller, or Admin) and enforces it
  3. A requester cannot access another user's request or any internal comment
  4. The request/comment/attachment/activity data model exists with a seeded default request type
**Plans**: TBD

Plans:
- [ ] 01-01: SSO integration and session handling
- [ ] 01-02: Role model and authorization guard with requester scoping
- [ ] 01-03: Core schema + seeded default request type

### Phase 2: Submit & Capture
**Goal**: A requester can submit a well-formed request in under a minute and add detail afterward.
**Depends on**: Phase 1
**Requirements**: SUBMIT-01, SUBMIT-02, SUBMIT-03, SUBMIT-04, SUBMIT-05
**Success Criteria** (what must be TRUE):
  1. A requester can submit a request with title, description, and type and receive a `REQ-####` ID, `New` status, and confirmation
  2. Selecting a type renders that type's custom fields, validated on submit
  3. A requester can attach up to 5 files (≤25 MB each), with clear rejection of oversized/extra files
  4. A requester can add or edit detail on their own open request
**Plans**: TBD

Plans:
- [ ] 02-01: Submit form with type-driven custom fields and validation
- [ ] 02-02: Short-ID generation, confirmation, and attachments
- [ ] 02-03: Post-submit edit on own open request

### Phase 3: Triage, Lifecycle & Fulfillment
**Goal**: A fulfiller can see the queue, work a request through its audited lifecycle, and close it — completing the core end-to-end loop.
**Depends on**: Phase 2
**Requirements**: QUEUE-01, QUEUE-02, QUEUE-03, QUEUE-04, QUEUE-05, LIFE-01, LIFE-02, LIFE-03, LIFE-04, DETAIL-01, DETAIL-02, DETAIL-03, ASSIGN-01, ASSIGN-02
**Success Criteria** (what must be TRUE):
  1. The default queue lists open requests sorted by priority then age and loads under 1s at 500 open requests
  2. A fulfiller can filter, search, use saved views, and bulk assign/change status
  3. A fulfiller can move a request through all statuses, with required notes on Blocked/Cancelled/Done, and every change appends to an append-only activity log
  4. A fulfiller can view full request detail, comment (with internal-only toggle hidden from requesters), assign, and close with a resolution note
**Plans**: TBD

Plans:
- [ ] 03-01: Queue view (sort, filters, search, saved views, bulk actions)
- [ ] 03-02: Status lifecycle with note gating and append-only activity log
- [ ] 03-03: Request detail with comments and internal toggle
- [ ] 03-04: Assignment and close-with-resolution

### Phase 4: Notifications
**Goal**: The loop closes for people — requesters and fulfillers get the right notifications, and stalled work gets nudged.
**Depends on**: Phase 3
**Requirements**: NOTIF-01, NOTIF-02, NOTIF-03
**Success Criteria** (what must be TRUE):
  1. Users receive email + in-app notifications for submit, assignment, status change, and new comment events
  2. Waiting-on-requester (3-day) and untouched (7-day) nudges fire on schedule
  3. Each user can mute notifications per category, and muted categories send nothing
**Plans**: TBD

Plans:
- [ ] 04-01: Event notification fan-out (email + in-app) with per-category mute
- [ ] 04-02: Scheduled nudge worker (3-day, 7-day)

### Phase 5: Reporting & Export
**Goal**: Fulfillers and admins can see volume and cycle time on one page and export any view.
**Depends on**: Phase 3
**Requirements**: REPORT-01, REPORT-02, REPORT-03
**Success Criteria** (what must be TRUE):
  1. The reporting page shows opened vs. closed by week (12 weeks) and open by status and assignee
  2. The reporting page shows median and p90 time-to-close, overall and by type
  3. A user can export any filtered queue view to CSV that mirrors the filter
**Plans**: TBD

Plans:
- [ ] 05-01: Reporting aggregates and page
- [ ] 05-02: CSV export of filtered views

### Phase 6: Administration
**Goal**: Admins configure request types, custom fields, categories, and roles without engineering.
**Depends on**: Phase 3
**Requirements**: ADMIN-01, ADMIN-02, ADMIN-03
**Success Criteria** (what must be TRUE):
  1. An admin can create/edit/deactivate request types with a validated custom-field schema (≤5 fields)
  2. Only active types appear on the submit form; deactivating doesn't affect existing requests
  3. An admin can manage categories and change user roles
**Plans**: TBD

Plans:
- [ ] 06-01: Request type + custom-field schema management
- [ ] 06-02: Categories and role management

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6
(Phases 4, 5, and 6 all depend only on Phase 3 and may be parallelized.)

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Access | 0/3 | Not started | - |
| 2. Submit & Capture | 0/3 | Not started | - |
| 3. Triage, Lifecycle & Fulfillment | 0/4 | Not started | - |
| 4. Notifications | 0/2 | Not started | - |
| 5. Reporting & Export | 0/2 | Not started | - |
| 6. Administration | 0/2 | Not started | - |
