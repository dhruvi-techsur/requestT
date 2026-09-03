# User Stories: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Related PRD | project_specs/PRD-RequestT.md |
| Related FRD | project_specs/FRD-RequestT.md |
| Personas | Priya Requester (PER-01), Frank Fulfiller (PER-02), Ada Admin (PER-03) |

---

## Epic 0: SSO Authentication & Roles (F0)

### US-0.1: Sign in with SSO
**As a** Requester, **I want to** sign in through my company's single sign-on, **so that** I can use the tracker without another password.

**Acceptance Criteria:**
- [ ] Visiting a protected page redirects to the org identity provider
- [ ] A successful SSO login establishes a session and returns me to the app
- [ ] A first-time login provisions my user record from the IdP
- [ ] When the identity provider is unavailable, I see a clear "sign-in service unavailable" message

**Priority:** P0 | **Feature Ref:** F0

### US-0.2: Role-scoped access
**As a** Requester, **I want to** see only my own requests and non-internal comments, **so that** other people's requests and internal notes stay private.

**Acceptance Criteria:**
- [ ] A requester cannot open another user's request (403)
- [ ] A requester never sees comments marked internal
- [ ] A fulfiller can see all requests and internal comments
- [ ] An admin has all fulfiller access plus administration

**Priority:** P0 | **Feature Ref:** F0

---

## Epic 1: Submit a Request (F1)

### US-1.1: Submit a request quickly
**As a** Requester, **I want to** submit a request with a title, description, type, and suggested priority, **so that** my ask is on record in under a minute.

**Acceptance Criteria:**
- [ ] Title, description, and request type are required
- [ ] Selecting a type renders that type's custom fields
- [ ] On submit I get a `REQ-####` ID, `New` status, timestamp, and a confirmation
- [ ] Missing required fields show inline validation errors

**Priority:** P0 | **Feature Ref:** F1

### US-1.2: Attach files
**As a** Requester, **I want to** attach up to 5 files, **so that** I can include supporting material.

**Acceptance Criteria:**
- [ ] I can attach up to 5 files, each up to 25 MB
- [ ] A file over 25 MB is rejected with a clear message
- [ ] A 6th file is rejected with a clear message

**Priority:** P1 | **Feature Ref:** F1

### US-1.3: Add a detail after submitting
**As a** Requester, **I want to** add a missing detail to my open request, **so that** I don't create a duplicate.

**Acceptance Criteria:**
- [ ] I can edit fields / add custom-field values on my own open request
- [ ] The edit is recorded in the activity log
- [ ] I cannot edit a request I don't own

**Priority:** P1 | **Feature Ref:** F1

---

## Epic 2: Status Lifecycle & Activity Log (F2)

### US-2.1: Move a request through its lifecycle
**As a** Fulfiller, **I want to** change a request's status, **so that** its state reflects reality.

**Acceptance Criteria:**
- [ ] I can move a request to any status (New, Triaged, In Progress, Waiting on requester, Blocked, Done, Cancelled)
- [ ] Blocked requires a note; Cancelled requires a reason; Done requires a resolution note
- [ ] Each change appends an actor + timestamp entry to the activity log
- [ ] A requester cannot change status

**Priority:** P0 | **Feature Ref:** F2

### US-2.2: Auto-nudge on waiting
**As a** Requester, **I want to** be nudged when my input has been pending for 3 days, **so that** blocked work doesn't stall silently.

**Acceptance Criteria:**
- [ ] A request in Waiting on requester for 3 days triggers a nudge to the requester
- [ ] The nudge respects the requester's notification mute settings

**Priority:** P1 | **Feature Ref:** F2

### US-2.3: Append-only audit trail
**As an** Admin, **I want to** trust that the activity log is never altered, **so that** the audit trail is reliable.

**Acceptance Criteria:**
- [ ] Activity entries cannot be edited or deleted, even for cancelled requests
- [ ] Every status change, assignment change, and field edit appears in the log

**Priority:** P0 | **Feature Ref:** F2

---

## Epic 3: Queue View (F3)

### US-3.1: See and sort the open queue
**As a** Fulfiller, **I want to** see all open requests sorted by priority then age, **so that** I work the most important items first.

**Acceptance Criteria:**
- [ ] Default view lists open requests sorted by priority then age
- [ ] The queue loads in under 1 second with 500 open requests
- [ ] Each row shows ID, title, status, priority, type, assignee, age

**Priority:** P0 | **Feature Ref:** F3

### US-3.2: Filter and search the queue
**As a** Fulfiller, **I want to** filter and search the queue, **so that** I can focus on a subset.

**Acceptance Criteria:**
- [ ] I can filter by status, assignee (including "unassigned"), type, priority, and date range
- [ ] Free-text search matches title, description, and comments
- [ ] Filters combine correctly

**Priority:** P0 | **Feature Ref:** F3

### US-3.3: Saved views and bulk actions
**As a** Fulfiller, **I want to** use saved views and bulk actions, **so that** I triage faster.

**Acceptance Criteria:**
- [ ] Saved views exist for My requests, Unassigned, Waiting on requester, Closed this month
- [ ] I can select multiple rows and bulk-assign or bulk-change status
- [ ] A bulk change enforces per-request note requirements and returns per-item results

**Priority:** P1 | **Feature Ref:** F3

---

## Epic 4: Request Detail (F4)

### US-4.1: View request detail
**As a** Requester, **I want to** open a request and see its full detail, **so that** I know its state and history.

**Acceptance Criteria:**
- [ ] Header shows ID, title, status, priority, type, requester, assignee, created/updated dates
- [ ] Body shows description, custom fields, and attachments
- [ ] Activity log shows status, assignment, and field changes
- [ ] A requester sees only non-internal comments

**Priority:** P0 | **Feature Ref:** F4

### US-4.2: Comment with internal toggle
**As a** Fulfiller, **I want to** comment on a request and optionally mark it internal, **so that** I can clarify with the requester or note things privately.

**Acceptance Criteria:**
- [ ] Comments are chronological, plain text, and can include attachments
- [ ] Only fulfillers can mark a comment internal
- [ ] Internal comments are never shown to the requester
- [ ] New comments notify other participants

**Priority:** P0 | **Feature Ref:** F4

---

## Epic 5: Assignment & Fulfillment (F5)

### US-5.1: Pull and assign work
**As a** Fulfiller, **I want to** assign requests to myself or a teammate, **so that** ownership is clear.

**Acceptance Criteria:**
- [ ] I can filter to unassigned requests
- [ ] I can assign a request to myself or another team member
- [ ] Assignment writes an activity entry and notifies the assignee
- [ ] Only fulfillers/admins can be assignees

**Priority:** P0 | **Feature Ref:** F5

### US-5.2: Close with a resolution note
**As a** Fulfiller, **I want to** close a request with a short resolution note, **so that** the outcome is on record.

**Acceptance Criteria:**
- [ ] Closing sets status Done and requires a resolution note
- [ ] `closed_at` is recorded
- [ ] The requester is notified of the status change

**Priority:** P0 | **Feature Ref:** F5

---

## Epic 6: Notifications (F6)

### US-6.1: Receive relevant notifications
**As a** Requester, **I want to** be notified when my request is picked up and completed, **so that** I don't have to ask.

**Acceptance Criteria:**
- [ ] I'm notified on assignment (as assignee), status change (as requester), and new comments (as participant)
- [ ] Notifications arrive in-app and by email
- [ ] Fulfillers are notified of new submissions (with a digest option)

**Priority:** P1 | **Feature Ref:** F6

### US-6.2: Mute by category
**As a** Fulfiller, **I want to** mute notification categories, **so that** I only get what I care about.

**Acceptance Criteria:**
- [ ] I can mute per category
- [ ] Muted categories send no email or in-app notification
- [ ] The 7-day untouched nudge goes to the assignee, or the team lead if unassigned

**Priority:** P2 | **Feature Ref:** F6

---

## Epic 7: Reporting & Export (F7)

### US-7.1: View the reporting page
**As an** Admin, **I want to** see volume and cycle-time on one page, **so that** I can answer how much we're handling.

**Acceptance Criteria:**
- [ ] Shows opened vs. closed by week for the last 12 weeks
- [ ] Shows open requests by status and by assignee
- [ ] Shows median and p90 time-to-close, overall and by type
- [ ] Reporting is not visible to requesters

**Priority:** P1 | **Feature Ref:** F7

### US-7.2: Export a filtered view to CSV
**As a** Fulfiller, **I want to** export any filtered queue view to CSV, **so that** I can share or analyze it.

**Acceptance Criteria:**
- [ ] Export reflects exactly the current filters and search
- [ ] CSV columns match the queue view
- [ ] An oversized export prompts me to narrow the filter

**Priority:** P2 | **Feature Ref:** F7

---

## Epic 8: Administration (F8)

### US-8.1: Manage request types and custom fields
**As an** Admin, **I want to** define request types and their custom fields, **so that** intake is standardized without engineering.

**Acceptance Criteria:**
- [ ] I can create/edit/deactivate request types
- [ ] Each type has up to 5 custom fields (text, number, select, date)
- [ ] Select fields require at least one option
- [ ] Only active types appear on the submit form; deactivating doesn't affect existing requests

**Priority:** P1 | **Feature Ref:** F8

### US-8.2: Manage user roles
**As an** Admin, **I want to** change a user's role, **so that** access matches responsibilities.

**Acceptance Criteria:**
- [ ] I can set a user's role to Requester, Fulfiller, or Admin
- [ ] Role changes take effect on the next authorization check
- [ ] Only admins can change roles

**Priority:** P1 | **Feature Ref:** F8

---

## Story Index

| Story | Title | Priority | Feature |
|---|---|---|---|
| US-0.1 | Sign in with SSO | P0 | F0 |
| US-0.2 | Role-scoped access | P0 | F0 |
| US-1.1 | Submit a request quickly | P0 | F1 |
| US-1.2 | Attach files | P1 | F1 |
| US-1.3 | Add a detail after submitting | P1 | F1 |
| US-2.1 | Move a request through its lifecycle | P0 | F2 |
| US-2.2 | Auto-nudge on waiting | P1 | F2 |
| US-2.3 | Append-only audit trail | P0 | F2 |
| US-3.1 | See and sort the open queue | P0 | F3 |
| US-3.2 | Filter and search the queue | P0 | F3 |
| US-3.3 | Saved views and bulk actions | P1 | F3 |
| US-4.1 | View request detail | P0 | F4 |
| US-4.2 | Comment with internal toggle | P0 | F4 |
| US-5.1 | Pull and assign work | P0 | F5 |
| US-5.2 | Close with a resolution note | P0 | F5 |
| US-6.1 | Receive relevant notifications | P1 | F6 |
| US-6.2 | Mute by category | P2 | F6 |
| US-7.1 | View the reporting page | P1 | F7 |
| US-7.2 | Export a filtered view to CSV | P2 | F7 |
| US-8.1 | Manage request types and custom fields | P1 | F8 |
| US-8.2 | Manage user roles | P1 | F8 |

---

## Priority Definitions

- **P0 — Critical:** MVP requirement; the product is not usable without it.
- **P1 — High:** Important for a complete v1; strongly expected by users.
- **P2 — Medium:** Valuable refinement; can follow shortly after launch.
- **P3 — Low:** Nice to have; deferred without impact to core value.
