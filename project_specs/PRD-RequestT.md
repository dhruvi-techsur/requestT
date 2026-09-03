# PRD: RequestT

**Status:** Draft
**Owner:** TBD
**Last updated:** 2026-09-03
**Source:** Synthesized from `project_specs/ref_docs/request-tracker-prd.md` and `.planning/PROJECT.md`

---

## 1. Executive Summary

RequestT is a lightweight internal request tracker for a small team. Colleagues across the organization submit work requests (IT help, design asks, data pulls, ops tasks) into one shared queue, and the fulfilling team sees what came in, who owns it, and what remains open. It replaces scattered Slack DMs, emails, and hallway conversations with a single canonical, status-tracked record per request, giving requesters self-service visibility and giving the team a triageable queue plus basic volume and cycle-time reporting.

---

## 2. Problem Statement

Requests arrive over Slack DMs, email, and hallway conversations. Nothing is written down in a shared place, which creates four recurring pains:

- **No visibility for requesters.** People don't know if their ask was seen or where it stands, so they re-ping the team.
- **No shared queue.** The team can't tell what's queued, so prioritization defaults to whoever shouts loudest.
- **Dropped and duplicated work.** Without a canonical record, items fall through the cracks or get worked twice.
- **No measurement.** Nobody can answer "how much are we actually taking on?" at the end of a quarter.

The organization is a single team (roughly 3–20 fulfillers serving up to a few hundred requesters) handling low-hundreds of open requests. The scale is small; the missing piece is a single source of truth.

---

## 3. Product Vision

**Vision:** One place where every internal request has a canonical, status-tracked record — checkable by the requester without asking a human, and triageable by the team in under five minutes a day.

**Strategic goals:**

- Every request has one canonical record with a status.
- A requester can check status without asking a human.
- The team can see the full queue and triage it in under five minutes a day.
- Basic volume and cycle-time numbers are available without exporting anything.

**Non-goals (explicitly not this product):**

- Not a project management tool — no subtasks, dependencies, Gantt charts, or sprints.
- Not a chat product — comments clarify, they don't host discussion threads.
- No SLA engine, escalation rules, or automation builder in v1.
- No time tracking or billing.
- No public-facing customer portal.

---

## 4. Technical Architecture

| Layer | Choice | Rationale |
|---|---|---|
| Client | Web app, desktop-first, responsive to mobile | Whole org submits; team triages at a desk |
| Auth | SSO via org identity provider (OIDC/SAML) | Internal tool; no separate password surface |
| Backend | Application server with role-based access control | Requester/Fulfiller/Admin permission model |
| Data store | Relational database | Structured entities with an append-only activity log |
| File storage | Object storage for attachments | Up to 5 files × 25 MB per request |
| Notifications | Email + in-app; async job for scheduled nudges | 3-day and 7-day nudges are time-triggered |
| Reporting | Pre-built aggregation queries + CSV export | One reporting page, no builder |

*Concrete stack (framework, DB engine, hosting) is deferred to TechArch-RequestT.md; the constraints above are binding.*

---

## 5. Feature Requirements

### F0: SSO Authentication & Role Model
**Description:** Users authenticate through the organization's existing identity provider — no separate passwords. Three additive roles (Requester, Fulfiller, Admin) govern what each user can do.

**Capabilities:**
- SSO login via org IdP
- Additive roles: Fulfiller is also a Requester; Admin has all Fulfiller powers plus administration
- Requesters restricted to their own requests and non-internal comments

**Priority:** P0 (Critical — MVP requirement)

### F1: Submit a Request
**Description:** Any org member submits a request in under a minute without knowing who handles it.

**Capabilities:**
- Required fields: title, description, request type; requester-suggested priority
- Up to 5 attachments, 25 MB each
- Request type may add up to 5 custom fields (text, number, select, date)
- On submit: human-readable ID (`REQ-1042`), status `New`, timestamp, and confirmation to requester
- Requester can add a missing detail after submitting

**Priority:** P0 (Critical — MVP requirement)

### F2: Status Lifecycle & Activity Log
**Description:** A single linear-ish lifecycle that any fulfiller can advance, with an append-only activity log.

**Capabilities:**
- States: New → Triaged → In Progress → Done, with Waiting-on-requester, Blocked, and Cancelled branches
- Any fulfiller can move a request to any status
- Blocked requires a note; Cancelled requires a reason; Done requires a resolution note
- Waiting-on-requester auto-nudges the requester at 3 days
- Every status change logged with actor and timestamp (append-only)

**Priority:** P0 (Critical — MVP requirement)

### F3: Queue View
**Description:** The team's working surface — all open requests, filterable and searchable, with saved views and bulk actions.

**Capabilities:**
- Default: all open requests sorted by priority then age
- Filters: status, assignee (incl. "unassigned"), type, priority, date range
- Free-text search across title, description, and comments
- Saved views: My requests, Unassigned, Waiting on requester, Closed this month
- Bulk assign and bulk status change on selected rows

**Priority:** P0 (Critical — MVP requirement)

### F4: Request Detail
**Description:** The full record for a single request: header metadata, body, comments, and activity log.

**Capabilities:**
- Header: ID, title, status, priority, type, requester, assignee, created/updated dates
- Body: description, custom fields, attachments
- Comments: chronological, plain text + attachments, with an internal-only toggle for fulfillers
- Activity log: status changes, assignment changes, field edits

**Priority:** P0 (Critical — MVP requirement)

### F5: Assignment & Fulfillment
**Description:** Fulfillers pull, assign, and close work.

**Capabilities:**
- See all unassigned requests to pull work
- Assign to self or a teammate
- Close with a short resolution note

**Priority:** P0 (Critical — MVP requirement)

### F6: Notifications
**Description:** Email plus in-app notifications for the events that matter, with per-category muting.

**Capabilities:**
- Events: request submitted (all fulfillers, digest option), assigned to you, status changed (requester), new comment (participants), waiting-on-requester 3-day nudge, open-and-untouched 7-day nudge
- Each user can mute per category

**Priority:** P1 (High)

### F7: Reporting & Export
**Description:** A single reporting page plus CSV export — no report builder.

**Capabilities:**
- Opened vs. closed by week (last 12 weeks)
- Open requests by status and by assignee
- Median and p90 time-to-close, overall and by type
- CSV export of any filtered queue view

**Priority:** P1 (High)

### F8: Administration
**Description:** Admins configure the system.

**Capabilities:**
- Manage request types and their custom-field schemas
- Manage categories
- Manage user roles

**Priority:** P1 (High)

---

## 6. Non-Functional Requirements

- **Authentication:** SSO via the org's existing identity provider. No separate passwords.
- **Performance:** Queue view loads in under 1s at 500 open requests.
- **Permissions:** Requesters see only their own requests and non-internal comments.
- **Audit:** Activity log is append-only and never deleted, even for cancelled requests.
- **Accessibility:** Keyboard-navigable; WCAG 2.1 AA on the submit form and queue view.
- **Data retention:** Closed requests stay queryable for 2 years, then archive.
- **Scale:** ~3–20 fulfillers, up to a few hundred requesters, low-hundreds of open requests. Single org, single shared queue — no multi-tenancy.
- **Platform:** Web, desktop-first, usable on a phone.

---

## 7. Success Metrics

Measured 60 days after launch.

| Metric | Target |
|---|---|
| Share of team requests submitted through the tracker | ≥ 80% |
| Requests still in `New` after 48 hours | < 10% |
| "Where is my request?" messages to the team | Down 50% vs. baseline |
| Median time-to-close | Establish a baseline (no target yet) |

---

## 8. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Requesters keep using Slack/email instead of the tracker | Low adoption; problem unsolved | Fast <1-minute submit, self-service status, notifications close the loop |
| Unknown day-one request types block custom-field design | Delivery delay | Ship a small default type set; admin-managed schemas allow later additions |
| Internal comments leak to requesters | Trust/privacy breach | Permission model enforces requester visibility; internal-only comment toggle |
| Single shared queue doesn't fit sub-teams | Rework | Validate with team before build; queue architecture kept extensible (open question 2) |
| Build-vs-buy uncertainty | Wasted effort if off-the-shelf suffices | Resolve build-vs-buy decision before Phase 1 build (open question 5) |
| Activity log growth over 2-year retention | Storage/performance | Archive policy after 2 years; append-only log indexed for reporting |

---

## 9. Feature Index

| ID | Feature | Priority | Category |
|---|---|---|---|
| F0 | SSO Authentication & Role Model | P0 | Platform |
| F1 | Submit a Request | P0 | Intake |
| F2 | Status Lifecycle & Activity Log | P0 | Workflow |
| F3 | Queue View | P0 | Triage |
| F4 | Request Detail | P0 | Workflow |
| F5 | Assignment & Fulfillment | P0 | Workflow |
| F6 | Notifications | P1 | Engagement |
| F7 | Reporting & Export | P1 | Insight |
| F8 | Administration | P1 | Platform |

**Priority summary:** 6 × P0 (MVP core), 3 × P1 (high). No P2/P3 in v1 scope.

---

## Open Questions

1. What are the actual request types on day one? The custom-field schema depends on this.
2. Is one shared queue right, or do sub-teams need separate queues?
3. Should requesters see who else's requests are ahead of theirs? (Transparency vs. noise.)
4. Does anything need an approval step before work starts?
5. Build, or configure an off-the-shelf tool?
