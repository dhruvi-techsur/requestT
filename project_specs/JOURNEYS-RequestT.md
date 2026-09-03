# User Journeys: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Related Personas | project_specs/PERSONAS-RequestT.md |
| Related JTBD | project_specs/JTBD-RequestT.md |
| Related PRD | project_specs/PRD-RequestT.md |

## Journey Index

| JRN-ID | Persona | Scenario | Key JTBD | Stages |
|---|---|---|---|---|
| JRN-01.1 | PER-01 Requester | Submit a request and track it to completion | JTBD-01.1, JTBD-01.2 | 5 |
| JRN-02.1 | PER-02 Fulfiller | Morning queue triage and work a request | JTBD-02.1, JTBD-02.2 | 5 |
| JRN-03.1 | PER-03 Admin | Configure a new request type and review the month | JTBD-03.1, JTBD-03.2 | 5 |

---

## PER-01: Priya Requester

### JRN-01.1: Submit a request and track it to completion

**Persona:** PER-01 (Priya Requester)
**Scenario:** Priya needs a data pull from another team. She wants to log the ask quickly and know where it stands without pinging anyone until it's done.

**Related Jobs:** JTBD-01.1, JTBD-01.2

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| Discover | Opens RequestT, signs in via SSO | Auth (F0) | "Do I even need a login? Oh, it's just SSO." | Neutral | Uncertain this is the right place | One-click SSO, no new password |
| Submit | Picks a type, fills title/description, sets priority | Submit form (F1) | "Which type is this? I hope I'm giving enough detail." | Slightly anxious | Doesn't know who handles it or what's needed | Type-driven fields guide required detail |
| Confirm | Reads the confirmation with `REQ-####` | Confirmation (F1) | "Good, it's on record. I have an ID now." | Relieved | Wonders if anyone will actually see it | Immediate ID + "fulfillers notified" reassurance |
| Track | Checks status a day later; sees it's In Progress | Queue/detail (F3/F4) | "Someone picked it up — I don't have to chase it." | Reassured | Might miss a status change | In-app + email notification on pickup |
| Complete | Gets a "Done" notification with resolution note | Notification (F6) | "Done, and here's what they did. Nice." | Satisfied | Could still have a follow-up question | Reopen/comment path from the closed request |

#### Key Moments
- **Decision Point:** Submit stage — picking the right type shapes the whole request.
- **Risk of Abandonment:** Submit stage — if the form feels long, Priya falls back to Slack.
- **Delight Opportunity:** Complete stage — a clear resolution note closes the loop without a message.

#### Success Outcome
Priya submits in under a minute and never messages the team to ask "where is my request?" (JTBD-01.1, JTBD-01.2 success measures).

#### Feature Touchpoints

| Stage | Features |
|---|---|
| Discover | F0 |
| Submit | F1 |
| Confirm | F1, F6 |
| Track | F3, F4, F6 |
| Complete | F2, F6 |

---

## PER-02: Frank Fulfiller

### JRN-02.1: Morning queue triage and work a request

**Persona:** PER-02 (Frank Fulfiller)
**Scenario:** Frank starts his day and needs to see what came in overnight, pull the most important item, and move it forward with a clear trail.

**Related Jobs:** JTBD-02.1, JTBD-02.2

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| Orient | Opens the queue, applies "Unassigned" saved view | Queue (F3) | "What's new and nobody owns yet?" | Focused | Could be a wall of rows | Priority-then-age sort surfaces top items |
| Prioritize | Scans by priority; picks the urgent data pull | Queue (F3) | "This one's high and it's been sitting 6 hours." | Slightly pressed | Hard to tell true urgency at a glance | Clear priority + age columns |
| Claim | Assigns the request to himself | Detail (F5) | "Mine now. Let me get context." | Determined | Risk of two people grabbing the same one | Assignment logged + assignee notified |
| Work | Moves to In Progress; comments a question to requester | Detail (F2/F4) | "Missing a date range — I'll ask." | Engaged | Waiting on requester could stall silently | 3-day auto-nudge on Waiting-on-requester |
| Close | Requester replies; Frank finishes and closes with note | Detail (F2/F5) | "Answered, delivered, done. Note it." | Relieved | Forgetting the resolution note | Required resolution note on Done |

#### Key Moments
- **Decision Point:** Prioritize stage — choosing the right item sets the day's order.
- **Risk of Abandonment:** Work stage — a blocked request with no nudge gets forgotten.
- **Delight Opportunity:** Orient stage — a full triage in under five minutes.

#### Success Outcome
Frank triages the full queue in under five minutes and closes with an auditable note; <10% of requests linger in New past 48h (JTBD-02.1, JTBD-02.2).

#### Feature Touchpoints

| Stage | Features |
|---|---|
| Orient | F3 |
| Prioritize | F3 |
| Claim | F5 |
| Work | F2, F4, F6 |
| Close | F2, F5, F6 |

---

## PER-03: Ada Admin

### JRN-03.1: Configure a new request type and review the month

**Persona:** PER-03 (Ada Admin)
**Scenario:** A new kind of ask is coming in often. Ada wants to add a structured request type with the right fields, then check month-end volume and cycle time.

**Related Jobs:** JTBD-03.1, JTBD-03.2

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| Identify | Notices repeated free-text asks of one kind | Queue/reports (F3/F7) | "We keep getting these — let's structure it." | Motivated | No signal that a pattern exists | Reports surface common untyped asks |
| Configure | Creates a request type, adds custom fields | Admin console (F8) | "Text, a select, a date — under five fields." | Confident | Fear of needing engineering | Self-serve schema editor, ≤5 fields |
| Activate | Sets the type active | Admin console (F8) | "It's live on the form now." | Satisfied | Worry it breaks existing requests | Deactivation/activation doesn't touch old requests |
| Measure | Opens the reporting page at month end | Reporting (F7) | "How much did we take on, how fast did we close?" | Curious | Would otherwise export and build charts | On-page opened/closed + median/p90 |
| Share | Exports a filtered view to CSV for the team | Export (F7) | "I'll send the closed-this-month set around." | Accomplished | Export could be unwieldy | Export mirrors the current filter exactly |

#### Key Moments
- **Decision Point:** Configure stage — the field schema determines data quality going forward.
- **Risk of Abandonment:** Configure stage — if it seems to need a developer, Ada gives up.
- **Delight Opportunity:** Measure stage — answering the quarterly question on one page.

#### Success Outcome
Ada stands up a typed request without engineering and reads a median-time-to-close baseline on one page (JTBD-03.1, JTBD-03.2).

#### Feature Touchpoints

| Stage | Features |
|---|---|
| Identify | F3, F7 |
| Configure | F8 |
| Activate | F8 |
| Measure | F7 |
| Share | F7 |

---

## Cross-Journey Patterns

- **Notifications close loops for everyone.** Priya relies on pickup/done alerts, Frank on the requester-reply nudge; a single, mutable notification system serves both (F6).
- **The queue is the shared surface.** Both the fulfiller (triage) and the admin (spotting patterns) start from the queue/reporting views (F3/F7).
- **Required notes create the audit trail all three trust.** Resolution/blocked/cancel notes feed the append-only activity log (F2) that underpins reporting.
- **Friction concentrates at the first structured step.** Priya's Submit and Ada's Configure are both the make-or-break moments where users might fall back to Slack or engineering — investing in guided forms pays off twice.

## Journey-to-JTBD Traceability

| Journey Stage | JTBD-ID | Expected Outcome |
|---|---|---|
| JRN-01.1 Submit | JTBD-01.1 | Sub-minute submission on the record |
| JRN-01.1 Track/Complete | JTBD-01.2 | Self-service status; no "where is it?" pings |
| JRN-02.1 Orient/Prioritize | JTBD-02.1 | Fast, prioritized triage from one queue |
| JRN-02.1 Work/Close | JTBD-02.2 | Auditable lifecycle with required notes |
| JRN-03.1 Configure/Activate | JTBD-03.1 | Self-serve request-type configuration |
| JRN-03.1 Measure/Share | JTBD-03.2 | On-page volume and cycle-time visibility |
