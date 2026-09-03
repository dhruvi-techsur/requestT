# Personas: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Related PRD | project_specs/PRD-RequestT.md |

---

## Persona Summary

| PER-ID | Name | Role | Primary Goal |
|---|---|---|---|
| PER-01 | Priya Requester | Requester (org colleague) | Submit an ask fast and know where it stands without pinging anyone |
| PER-02 | Frank Fulfiller | Fulfiller (team member) | See the queue, pull work, and close requests with a clear record |
| PER-03 | Ada Admin | Admin (team lead) | Configure request types and see monthly volume and cycle time |

---

## PER-01: Priya Requester

**Role & Context:**
Priya works in marketing at the same company as the fulfilling team. A few times a month she needs something from another team — a data pull, a design asset, an access grant. Today she asks over Slack or email and then waits, unsure whether her message was even seen. She is not technical about ticketing systems; she wants a form, a confirmation, and a way to check status later. She uses the tool sporadically, often on her phone, so the submit path has to be obvious and fast.

**Goals:**
- Submit a request in under a minute without knowing who handles it (F1)
- Get told when someone picks it up and when it's done (F6)
- Look up everything she's submitted and its current state (F3, F4)
- Add a missing detail after submitting (F1)

**Pain Points:**
- Doesn't know if her ask was seen or where it stands
- Has to re-ping the team to get a status update
- Requests she sends over chat get dropped or forgotten

**Technical Expertise:** Basic — comfortable with web forms and email; no interest in ticketing jargon or admin screens.

**Top Tasks:**
1. Submit a new request with the right type and details (a few times/month, high)
2. Check the status of an open request (as-needed, high)
3. Answer a fulfiller's clarifying question (occasional, medium)

**Success Criteria:**
- Can submit a complete request in under a minute
- Never has to message the team to ask "where is my request?"

## PER-02: Frank Fulfiller

**Role & Context:**
Frank is on the small team that receives requests. He spends part of each day triaging and working the queue. He needs to see everything unassigned so he can pull work, filter down to what's his, and move requests through their lifecycle with a note trail others can follow. He works at a desk on a dual-monitor setup and expects the queue to load instantly even when a few hundred requests are open. Frank is also a Requester when he needs something from another team.

**Goals:**
- See everything unassigned so he can pull work (F3, F5)
- Filter the queue by status, assignee, type, and priority (F3)
- Assign a request to himself or a teammate (F5)
- Ask the requester a question and get notified when they answer (F4, F6)
- Close a request with a short resolution note (F5, F2)

**Pain Points:**
- Can't tell what's queued, so prioritization goes to whoever shouts loudest
- Work gets duplicated because there's no shared record
- Internal notes risk leaking to requesters in ad-hoc channels

**Technical Expertise:** Intermediate — lives in web tools all day, comfortable with filters, saved views, and bulk actions.

**Top Tasks:**
1. Triage the queue and pull or assign work (daily, critical)
2. Move requests through statuses with required notes (daily, high)
3. Comment to requesters and internally, using the internal-only toggle (daily, high)
4. Close requests with a resolution note (daily, high)

**Success Criteria:**
- Can triage the full queue in under five minutes a day
- Queue view loads in under 1s at 500 open requests
- Zero internal comments visible to requesters

## PER-03: Ada Admin

**Role & Context:**
Ada leads the fulfilling team. She does everything a fulfiller does, plus she owns the configuration: which request types exist, what custom fields each asks for, and who has which role. At the end of each month she wants to answer "how much did we take on and how fast did we close it?" without exporting to a spreadsheet and building charts by hand.

**Goals:**
- Define request types and the fields each one asks for (F8)
- Manage categories and user roles (F8, F0)
- See monthly volume by type and median time-to-close (F7)

**Pain Points:**
- Nobody can answer quarterly "how much are we taking on?"
- No baseline for cycle time to improve against
- Request types and their fields aren't standardized

**Technical Expertise:** Intermediate-to-advanced — comfortable configuring schemas and reading a reporting page; not a developer.

**Top Tasks:**
1. Create and edit request types and their custom-field schemas (occasional, high)
2. Assign and adjust user roles (occasional, medium)
3. Review the reporting page for monthly volume and cycle time (monthly, high)

**Success Criteria:**
- Can stand up a new request type with custom fields without engineering help
- Can read monthly volume and median/p90 time-to-close on one page, no export

---

## Persona Relationships

| From | To | Interaction |
|---|---|---|
| PER-01 Requester | PER-02 Fulfiller | Submits requests; answers clarifying questions; receives status notifications |
| PER-02 Fulfiller | PER-01 Requester | Triages, assigns, comments, closes; the 3-day/7-day nudges involve both |
| PER-03 Admin | PER-02 Fulfiller | Configures the request types and roles that shape the fulfiller's queue |
| PER-03 Admin | PER-01 Requester | Defines the types and fields requesters see on the submit form |
| PER-02 Fulfiller | PER-02 Fulfiller | Assign to a teammate; internal-only comments between fulfillers |

---

## Feature-Persona Matrix

| Feature | PER-01 Requester | PER-02 Fulfiller | PER-03 Admin |
|---|---|---|---|
| F0 SSO Auth & Roles | Primary | Primary | Primary |
| F1 Submit a Request | Primary | Secondary | Secondary |
| F2 Status Lifecycle & Activity Log | Secondary | Primary | Secondary |
| F3 Queue View | Secondary | Primary | Secondary |
| F4 Request Detail | Primary | Primary | Secondary |
| F5 Assignment & Fulfillment | None | Primary | Secondary |
| F6 Notifications | Primary | Primary | Secondary |
| F7 Reporting & Export | None | Secondary | Primary |
| F8 Administration | None | None | Primary |
