# PRD: Request Tracker

**Status:** Draft
**Owner:** TBD
**Last updated:** 2026-09-03

---

## 1. Assumptions

This draft assumes a lightweight **internal request tracker**: a small team receives
work requests from colleagues (IT help, design asks, data pulls, ops tasks) and needs
one place to see what came in, who owns it, and what's still open. Swap the framing if
the real context is customer support tickets, feature requests from users, or approval
workflows — the structure holds, the details change.

Also assumed:

- Single organization, single shared queue. No multi-tenancy.
- Team size roughly 3–20 fulfillers, up to a few hundred requesters.
- Volume in the low hundreds of open requests, not tens of thousands.
- Web app, desktop-first, usable on a phone.

---

## 2. Problem

Requests arrive over Slack DMs, email, and hallway conversations. Nothing is written
down in a shared place, so:

- Requesters don't know if their ask was seen, or where it stands.
- The team can't tell what's queued, so prioritization happens by whoever shouts loudest.
- Work gets dropped or duplicated.
- Nobody can answer "how much are we actually taking on?" at the end of a quarter.

## 3. Goals

1. Every request has one canonical record with a status.
2. A requester can check status without asking a human.
3. The team can see the full queue and triage it in under five minutes a day.
4. Basic volume and cycle-time numbers are available without exporting anything.

## 4. Non-goals

- Not a project management tool. No subtasks, dependencies, Gantt charts, or sprints.
- Not a chat product. Comments exist for clarification, not for discussion threads.
- No SLA engine, escalation rules, or automation builder in v1.
- No time tracking or billing.
- No public-facing customer portal.

---

## 5. Users and roles

| Role | Who | Can do |
|---|---|---|
| **Requester** | Anyone in the org | Submit requests, view and comment on their own, cancel their own |
| **Fulfiller** | Team member | See all requests, assign, change status, comment, close |
| **Admin** | Team lead | Everything a fulfiller can, plus manage request types, categories, and user roles |

Roles are additive. A fulfiller is also a requester.

---

## 6. Core user stories

**Requester**

- I submit a request in under a minute without knowing who handles it.
- I get told when someone picks it up and when it's done.
- I can look up everything I've submitted and its current state.
- I can add a missing detail after submitting.

**Fulfiller**

- I see everything unassigned so I can pull work.
- I filter the queue by status, assignee, type, and priority.
- I assign a request to myself or a teammate.
- I ask the requester a question and get notified when they answer.
- I close a request with a short resolution note.

**Admin**

- I define the request types and the fields each one asks for.
- I see monthly volume by type and median time-to-close.

---

## 7. Functional requirements

### 7.1 Submitting a request

- Form fields: title (required), description (required), request type (required, from an
  admin-managed list), priority (requester-suggested, fulfiller-adjustable), attachments
  (optional, up to 5 files, 25 MB each).
- Each request type can add up to five extra custom fields (text, number, select, date).
- On submit: request gets a short human-readable ID (`REQ-1042`), status `New`,
  and a timestamp. Requester gets a confirmation.

### 7.2 Statuses

A single linear-ish lifecycle. Any fulfiller can move a request to any status.

```
New → Triaged → In Progress → Done
                    ↕
              Blocked / Waiting on requester
                    ↓
                 Cancelled
```

- `New` — submitted, nobody has looked at it.
- `Triaged` — accepted, priority set, not started.
- `In Progress` — someone is actively working it.
- `Waiting on requester` — blocked pending an answer. Auto-nudges the requester at 3 days.
- `Blocked` — blocked on something else; requires a note.
- `Done` — closed with a resolution note.
- `Cancelled` — closed without work; requires a reason.

Every status change is recorded in an activity log with actor and timestamp.

### 7.3 Queue view

- Default view: all open requests, sorted by priority then age.
- Filters: status, assignee (including "unassigned"), type, priority, date range.
- Free-text search across title, description, and comments.
- Saved views: "My requests," "Unassigned," "Waiting on requester," "Closed this month."
- Bulk action on selected rows: assign, change status.

### 7.4 Request detail

- Header: ID, title, status, priority, type, requester, assignee, created/updated dates.
- Body: description, custom fields, attachments.
- Comments: chronological, plain text plus attachments, with an internal-only toggle
  for fulfillers.
- Activity log: status changes, assignment changes, field edits.

### 7.5 Notifications

Email plus in-app. Each user can mute per-category.

| Event | Notifies |
|---|---|
| Request submitted | All fulfillers (digest option) |
| Assigned to you | Assignee |
| Status changed | Requester |
| New comment | Other participants |
| Waiting on requester for 3 days | Requester |
| Open and untouched for 7 days | Assignee, or team lead if unassigned |

### 7.6 Reporting

One page, no builder:

- Requests opened vs. closed, by week, last 12 weeks.
- Open requests by status and by assignee.
- Median and p90 time-to-close, overall and by type.
- CSV export of any filtered queue view.

---

## 8. Data model (sketch)

```
User        id, name, email, role, notification_prefs
RequestType id, name, description, custom_field_schema, active
Request     id, short_id, title, description, type_id, status, priority,
            requester_id, assignee_id, custom_field_values,
            created_at, updated_at, closed_at, resolution_note
Comment     id, request_id, author_id, body, internal, created_at
Attachment  id, request_id, comment_id?, filename, size, url, uploaded_by
Activity    id, request_id, actor_id, field, old_value, new_value, created_at
```

---

## 9. Non-functional requirements

- **Auth:** SSO via the org's existing identity provider. No separate passwords.
- **Performance:** queue view loads in under 1s at 500 open requests.
- **Permissions:** requesters see only their own requests and non-internal comments.
- **Audit:** activity log is append-only and never deleted, even for cancelled requests.
- **Accessibility:** keyboard-navigable, WCAG 2.1 AA on the submit form and queue view.
- **Data retention:** closed requests stay queryable for 2 years, then archive.

---

## 10. Out of scope for v1 (candidates for later)

- Slack integration for submitting and status updates
- Email-to-request intake
- Recurring requests
- Templates and canned responses
- Custom statuses per request type
- SLA targets with breach alerts
- Requester satisfaction survey on close

---

## 11. Success metrics

Measured 60 days after launch.

| Metric | Target |
|---|---|
| Share of team requests submitted through the tracker | ≥ 80% |
| Requests still in `New` after 48 hours | < 10% |
| "Where is my request?" messages to the team | Down 50% vs. baseline |
| Median time-to-close | Established as a baseline (no target yet) |

---

## 12. Open questions

1. What are the actual request types on day one? The custom-field schema depends on this.
2. Is one shared queue right, or do sub-teams need separate queues?
3. Should requesters see who else's requests are ahead of theirs? (Transparency vs. noise.)
4. Does anything need an approval step before work starts?
5. Build, or configure an off-the-shelf tool? This spec is small enough that the
   build-vs-buy answer isn't obvious.
