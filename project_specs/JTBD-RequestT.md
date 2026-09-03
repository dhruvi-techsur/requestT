# Jobs-to-be-Done: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Related Personas | project_specs/PERSONAS-RequestT.md |
| Related PRD | project_specs/PRD-RequestT.md |

---

## JTBD Summary

| JTBD-ID | Persona | Job Statement (short) | Priority |
|---|---|---|---|
| JTBD-01.1 | PER-01 Requester | Submit an ask fast without knowing who handles it | P0 |
| JTBD-01.2 | PER-01 Requester | Know where my request stands without asking a human | P0 |
| JTBD-01.3 | PER-01 Requester | Add a missing detail after submitting | P1 |
| JTBD-02.1 | PER-02 Fulfiller | See what's queued and pull the right work | P0 |
| JTBD-02.2 | PER-02 Fulfiller | Move work through its lifecycle with a clear trail | P0 |
| JTBD-02.3 | PER-02 Fulfiller | Ask the requester a question and know when they answer | P1 |
| JTBD-03.1 | PER-03 Admin | Configure request types without engineering help | P1 |
| JTBD-03.2 | PER-03 Admin | Know monthly volume and cycle time at a glance | P1 |

---

## PER-01: Priya Requester — Jobs

### JTBD-01.1: Submit an ask fast

**Job Statement:**
When I need something from another team, I want to write it down in one place in under a minute without figuring out who owns it, so I can get back to my own work knowing the ask is on record.

**Current Alternatives:**
- Sends a Slack DM to whoever she thinks handles it
- Sends an email and hopes it's seen
- Catches someone in the hallway

**Hiring Criteria:**
- A single form with title, description, and type takes under a minute
- No need to know or pick the assignee
- Immediate confirmation with a reference ID

**Success Measure:** ≥80% of team requests come through the tracker within 60 days; a request can be submitted in under a minute.

**Related Features:** F1, F0
**Priority:** P0

### JTBD-01.2: Know where my request stands

**Job Statement:**
When I'm waiting on a request, I want to check its current status myself, so I can stop pinging the team to ask "where is it?"

**Current Alternatives:**
- Re-pings the team on Slack
- Waits and assumes it's forgotten

**Hiring Criteria:**
- Can look up every request she's submitted and its current state
- Gets notified when it's picked up and when it's done
- Never has to message a human for status

**Success Measure:** "Where is my request?" messages to the team drop 50% vs. baseline.

**Related Features:** F3, F4, F6
**Priority:** P0

### JTBD-01.3: Add a missing detail later

**Job Statement:**
When I realize I left out information after submitting, I want to add it to the existing request, so I don't have to create a duplicate or hunt down the fulfiller.

**Current Alternatives:**
- Sends a follow-up Slack message that gets detached from the original ask
- Submits a second, duplicate request

**Hiring Criteria:**
- Can edit or add detail to her own open request
- The addition is visible to the fulfiller in context

**Success Measure:** Requesters resolve missing-detail cases without creating duplicates.

**Related Features:** F1, F4
**Priority:** P1

---

## PER-02: Frank Fulfiller — Jobs

### JTBD-02.1: See what's queued and pull the right work

**Job Statement:**
When I start my day, I want to see every open and unassigned request in priority order, so I can pull the most important work instead of reacting to whoever shouted loudest.

**Current Alternatives:**
- Scans Slack and email for asks
- Relies on memory and hallway prioritization

**Hiring Criteria:**
- One queue sorted by priority then age
- Filter by status, assignee, type, priority; "unassigned" is one click
- Queue loads in under 1s at 500 open requests

**Success Measure:** The full queue can be triaged in under five minutes a day.

**Related Features:** F3, F5
**Priority:** P0

### JTBD-02.2: Move work through its lifecycle with a clear trail

**Job Statement:**
When I work a request, I want to move it through statuses and record why, so anyone can see what happened without asking me.

**Current Alternatives:**
- Tracks status in his head or a personal note
- Explains status verbally when asked

**Hiring Criteria:**
- Any status reachable in one action; required notes on Blocked/Cancelled/Done
- Every change logged with actor and timestamp, append-only
- Closing takes a short resolution note

**Success Measure:** <10% of requests remain in `New` after 48 hours; every closed request has a resolution note.

**Related Features:** F2, F5
**Priority:** P0

### JTBD-02.3: Ask the requester and know when they answer

**Job Statement:**
When a request is missing information, I want to ask the requester and be notified when they reply, so blocked work doesn't sit silently.

**Current Alternatives:**
- DMs the requester and forgets to check back
- Lets the request stall

**Hiring Criteria:**
- Comment to the requester from the request itself
- Waiting-on-requester auto-nudges at 3 days
- Notified when the requester responds

**Success Measure:** Waiting-on-requester requests get a response or nudge within 3 days.

**Related Features:** F4, F6, F2
**Priority:** P1

---

## PER-03: Ada Admin — Jobs

### JTBD-03.1: Configure request types without engineering help

**Job Statement:**
When the team's intake needs change, I want to define request types and their fields myself, so we can standardize asks without waiting on a developer.

**Current Alternatives:**
- Files a ticket with engineering to change a form
- Uses a generic free-text ask with no structure

**Hiring Criteria:**
- Create/edit request types with up to 5 custom fields (text, number, select, date)
- Activate/deactivate types
- No code required

**Success Measure:** A new request type with custom fields is live without engineering involvement.

**Related Features:** F8
**Priority:** P1

### JTBD-03.2: Know monthly volume and cycle time at a glance

**Job Statement:**
When the month ends, I want to see how much we took on and how fast we closed it, so I can answer "how much are we handling?" without building a spreadsheet.

**Current Alternatives:**
- Manually counts Slack/email asks
- Cannot answer the question at all

**Hiring Criteria:**
- One reporting page: opened vs. closed by week, open by status/assignee, median and p90 time-to-close
- CSV export of any filtered view
- No report builder needed

**Success Measure:** A median time-to-close baseline is established and readable on one page.

**Related Features:** F7
**Priority:** P1

---

## Outcome-to-Feature Traceability

| JTBD-ID | Feature(s) | Expected Outcome |
|---|---|---|
| JTBD-01.1 | F1, F0 | Sub-minute submission on the record |
| JTBD-01.2 | F3, F4, F6 | Self-service status, fewer "where is it?" pings |
| JTBD-01.3 | F1, F4 | Missing details added without duplicates |
| JTBD-02.1 | F3, F5 | Fast, prioritized triage from one queue |
| JTBD-02.2 | F2, F5 | Auditable lifecycle with required notes |
| JTBD-02.3 | F4, F6, F2 | No silently stalled blocked work |
| JTBD-03.1 | F8 | Self-serve request-type configuration |
| JTBD-03.2 | F7 | On-page volume and cycle-time visibility |

---

## NaC Preview

| JTBD-ID | Outcome | Candidate Natural Acceptance Criteria |
|---|---|---|
| JTBD-01.1 | Sub-minute submission | Given the submit form, a requester can create a valid request in under 60 seconds and receives a `REQ-####` confirmation |
| JTBD-01.2 | Self-service status | A requester can view the current status of every request they submitted, and is notified on pickup and completion |
| JTBD-01.3 | Add detail later | A requester can add detail to their own open request without creating a new one |
| JTBD-02.1 | Prioritized triage | The queue defaults to open requests sorted by priority then age and loads <1s at 500 open |
| JTBD-02.2 | Auditable lifecycle | Every status change requires the appropriate note and appends an actor+timestamp entry to an append-only log |
| JTBD-02.3 | No stalled blocked work | Waiting-on-requester auto-nudges at 3 days; the fulfiller is notified on the requester's reply |
| JTBD-03.1 | Self-serve config | An admin can create a request type with ≤5 typed custom fields and activate it |
| JTBD-03.2 | Cycle-time visibility | The reporting page shows opened/closed by week and median & p90 time-to-close without export |
