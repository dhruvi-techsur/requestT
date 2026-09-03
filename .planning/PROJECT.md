# RequestT

## What This Is

RequestT is a lightweight internal request tracker for a small team. Colleagues across the org submit work requests (IT help, design asks, data pulls, ops tasks) into one shared queue, and the fulfilling team sees what came in, who owns it, and what's still open. It replaces scattered Slack DMs, emails, and hallway conversations with a single canonical record per request.

## Core Value

Every request has one canonical, status-tracked record that a requester can check without asking a human and the team can triage in under five minutes a day.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

(None yet — ship to validate)

### Active

<!-- Current scope. Building toward these. -->

**Submitting requests**
- [ ] Submit a request with title, description, request type, and suggested priority in under a minute
- [ ] Attach up to 5 files (25 MB each) to a request
- [ ] Request types carry up to 5 admin-defined custom fields (text, number, select, date)
- [ ] On submit, request gets a human-readable ID (`REQ-1042`), `New` status, timestamp, and confirmation
- [ ] Requester can add a missing detail after submitting

**Statuses & lifecycle**
- [ ] Requests move through New → Triaged → In Progress → Done, with Waiting-on-requester, Blocked, and Cancelled branches
- [ ] Any fulfiller can move a request to any status
- [ ] Blocked requires a note; Cancelled requires a reason; Done requires a resolution note
- [ ] Waiting-on-requester auto-nudges the requester at 3 days
- [ ] Every status change is recorded in an append-only activity log with actor and timestamp

**Queue view**
- [ ] Default view lists all open requests sorted by priority then age
- [ ] Filter by status, assignee (including "unassigned"), type, priority, and date range
- [ ] Free-text search across title, description, and comments
- [ ] Saved views: My requests, Unassigned, Waiting on requester, Closed this month
- [ ] Bulk assign and bulk status change on selected rows

**Request detail**
- [ ] Header shows ID, title, status, priority, type, requester, assignee, created/updated dates
- [ ] Body shows description, custom fields, and attachments
- [ ] Chronological comments (plain text + attachments) with an internal-only toggle for fulfillers
- [ ] Activity log of status changes, assignment changes, and field edits

**Assignment & fulfillment**
- [ ] Fulfiller sees all unassigned requests to pull work
- [ ] Fulfiller assigns a request to self or a teammate
- [ ] Fulfiller closes a request with a short resolution note

**Notifications**
- [ ] Email + in-app notifications for: request submitted, assigned to you, status changed, new comment, waiting-on-requester 3-day nudge, untouched-7-day nudge
- [ ] Each user can mute notifications per category

**Reporting**
- [ ] Single reporting page: opened vs. closed by week (last 12 weeks), open by status and by assignee, median and p90 time-to-close overall and by type
- [ ] CSV export of any filtered queue view

**Administration**
- [ ] Admin manages request types and their custom-field schemas
- [ ] Admin manages categories and user roles

**Access & roles**
- [ ] Three additive roles: Requester, Fulfiller, Admin
- [ ] Requesters see only their own requests and non-internal comments

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Project management features (subtasks, dependencies, Gantt, sprints) — this is a tracker, not a PM tool
- Discussion-thread chat — comments exist for clarification, not conversation
- SLA engine, escalation rules, automation builder — deferred to keep v1 lightweight
- Time tracking / billing — not part of the tracking problem
- Public-facing customer portal — internal tool only
- Slack integration for submit/status updates — later candidate
- Email-to-request intake — later candidate
- Recurring requests — later candidate
- Templates and canned responses — later candidate
- Custom statuses per request type — later candidate
- SLA targets with breach alerts — later candidate
- Requester satisfaction survey on close — later candidate

## Context

- **Environment:** Internal web app, desktop-first, usable on a phone. Single organization, single shared queue — no multi-tenancy.
- **Scale:** ~3–20 fulfillers, up to a few hundred requesters, low-hundreds of open requests (not tens of thousands).
- **Problem it solves:** Requests arrive over Slack DMs, email, and hallway conversations with nothing written down, so requesters don't know status, the team can't see the queue, work gets dropped or duplicated, and quarterly volume is unanswerable.
- **Data model sketch:** User, RequestType, Request, Comment, Attachment, Activity — see reference PRD at `project_specs/ref_docs/request-tracker-prd.md`.
- **Success metrics (60 days post-launch):** ≥80% of team requests submitted through the tracker; <10% still in `New` after 48h; "where is my request?" messages down 50% vs. baseline; establish a median time-to-close baseline.

## Constraints

- **Security**: SSO via the org's existing identity provider — no separate passwords — because the tool is internal and must not add a credential surface.
- **Performance**: Queue view loads in under 1s at 500 open requests — triage must stay fast to be adopted.
- **Permissions**: Requesters see only their own requests and non-internal comments — internal fulfiller notes must never leak to requesters.
- **Audit**: Activity log is append-only and never deleted, even for cancelled requests — for accountability and quarterly reporting.
- **Accessibility**: Keyboard-navigable, WCAG 2.1 AA on the submit form and queue view — the whole org uses the submit path.
- **Data retention**: Closed requests stay queryable for 2 years, then archive — balances history against storage.

## Key Decisions

<!-- Decisions that constrain future work. Add throughout project lifecycle. -->

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single shared queue, no multi-tenancy | Small single-org team; keeps v1 simple | — Pending (open question: do sub-teams need separate queues?) |
| SSO-only authentication | Internal tool; avoid a separate password surface | — Pending |
| Linear-ish status lifecycle, any fulfiller can move to any status | Small team trusts itself; avoids rigid workflow rules | — Pending |
| Build custom vs. buy off-the-shelf | Spec is small enough that build-vs-buy is genuinely open | — Pending (open question 5) |

## Open Questions

1. What are the actual request types on day one? The custom-field schema depends on this.
2. Is one shared queue right, or do sub-teams need separate queues?
3. Should requesters see who else's requests are ahead of theirs? (Transparency vs. noise.)
4. Does anything need an approval step before work starts?
5. Build, or configure an off-the-shelf tool?

---
*Last updated: 2026-09-03 after initialization*
