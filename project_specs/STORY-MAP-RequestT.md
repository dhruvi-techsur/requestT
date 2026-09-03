# Story Map: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Related Personas | project_specs/PERSONAS-RequestT.md |
| Related Journeys | project_specs/JOURNEYS-RequestT.md |
| Related JTBD | project_specs/JTBD-RequestT.md |
| Related UserStories | project_specs/UserStories-RequestT.md |
| Related PRD | project_specs/PRD-RequestT.md |

## Overview

This story map organizes RequestT's user stories along the two primary journeys — a requester submitting and tracking, and a fulfiller triaging and working — plus admin configuration. Lanes follow journey stages. Each story carries a Natural Acceptance Criterion (NaC) derived from the intersection of a JTBD outcome, a journey stage, and the story itself. The Release column groups delivery: **R1 (Core Workflow)** completes end-to-end submit→triage→close; **R2 (Visibility & Config)** adds notifications, reporting, and administration; **R3 (Refinements)** adds convenience features.

## Story Map Matrix

### Access & Entry

| Activity | Persona | Epic | Stories | NaC | Release |
|---|---|---|---|---|---|
| Sign in via SSO | PER-01/02/03 | Epic 0 (F0) | US-0.1 | JTBD-01.1 → JRN-01.1:Discover → SSO login establishes a session without a separate password | R1 |
| Enforce role scope | PER-01/02/03 | Epic 0 (F0) | US-0.2 | JTBD-01.2 → JRN-01.1:Track → A requester sees only their own requests and non-internal comments | R1 |

### Submit & Capture

| Activity | Persona | Epic | Stories | NaC | Release |
|---|---|---|---|---|---|
| Submit a request | PER-01 | Epic 1 (F1) | US-1.1 | JTBD-01.1 → JRN-01.1:Submit → A valid request is created in <60s with a `REQ-####` ID and `New` status | R1 |
| Attach files | PER-01 | Epic 1 (F1) | US-1.2 | JTBD-01.1 → JRN-01.1:Submit → Up to 5 files ≤25 MB attach; oversized/extra files rejected clearly | R2 |
| Add a detail later | PER-01 | Epic 1 (F1) | US-1.3 | JTBD-01.3 → JRN-01.1:Submit → A requester adds detail to their own open request without a duplicate | R2 |

### Triage & Assign

| Activity | Persona | Epic | Stories | NaC | Release |
|---|---|---|---|---|---|
| See & sort the queue | PER-02 | Epic 3 (F3) | US-3.1 | JTBD-02.1 → JRN-02.1:Orient → Open requests sort by priority then age and load <1s at 500 open | R1 |
| Filter & search | PER-02 | Epic 3 (F3) | US-3.2 | JTBD-02.1 → JRN-02.1:Prioritize → Filters (status/assignee/type/priority/date) and search combine correctly | R1 |
| Saved views & bulk | PER-02 | Epic 3 (F3) | US-3.3 | JTBD-02.1 → JRN-02.1:Orient → Saved views and bulk assign/status act on multiple rows | R2 |
| Pull & assign work | PER-02 | Epic 5 (F5) | US-5.1 | JTBD-02.1 → JRN-02.1:Claim → Assigning writes activity and notifies the assignee | R1 |

### Work & Resolve

| Activity | Persona | Epic | Stories | NaC | Release |
|---|---|---|---|---|---|
| Move through lifecycle | PER-02 | Epic 2 (F2) | US-2.1 | JTBD-02.2 → JRN-02.1:Work → Any status reachable; Blocked/Cancelled/Done require a note; each logs actor+timestamp | R1 |
| Auto-nudge on waiting | PER-01 | Epic 2 (F2) | US-2.2 | JTBD-02.3 → JRN-02.1:Work → Waiting-on-requester ≥3 days nudges the requester | R2 |
| Append-only audit | PER-03 | Epic 2 (F2) | US-2.3 | JTBD-02.2 → JRN-02.1:Close → Activity entries can never be edited or deleted | R1 |
| View request detail | PER-01/02 | Epic 4 (F4) | US-4.1 | JTBD-01.2 → JRN-01.1:Track → Detail shows header, body, activity; requester sees only non-internal comments | R1 |
| Comment (internal toggle) | PER-02 | Epic 4 (F4) | US-4.2 | JTBD-02.3 → JRN-02.1:Work → Only fulfillers post internal comments; requesters never see them | R1 |
| Close with resolution | PER-02 | Epic 5 (F5) | US-5.2 | JTBD-02.2 → JRN-02.1:Close → Closing sets Done, requires a resolution note, records `closed_at` | R1 |

### Notify, Measure & Configure

| Activity | Persona | Epic | Stories | NaC | Release |
|---|---|---|---|---|---|
| Receive notifications | PER-01 | Epic 6 (F6) | US-6.1 | JTBD-01.2 → JRN-01.1:Complete → Requester notified on pickup and completion, in-app and email | R2 |
| Mute by category | PER-02 | Epic 6 (F6) | US-6.2 | JTBD-02.3 → JRN-02.1:Work → Muted categories send nothing; 7-day nudge routes to assignee/lead | R3 |
| View reporting page | PER-03 | Epic 7 (F7) | US-7.1 | JTBD-03.2 → JRN-03.1:Measure → One page shows opened/closed by week and median & p90 time-to-close | R2 |
| Export CSV | PER-02 | Epic 7 (F7) | US-7.2 | JTBD-03.2 → JRN-03.1:Share → Export mirrors the current filter exactly | R3 |
| Manage types & fields | PER-03 | Epic 8 (F8) | US-8.1 | JTBD-03.1 → JRN-03.1:Configure → Admin creates a type with ≤5 typed custom fields and activates it | R2 |
| Manage user roles | PER-03 | Epic 8 (F8) | US-8.2 | JTBD-03.1 → JRN-03.1:Configure → Admin sets a user's role; effective on next auth check | R2 |

## NaC Derivation Table

| JTBD ID | Outcome | Journey Stage | NaC | Story |
|---|---|---|---|---|
| JTBD-01.1 | Sub-minute submission on record | JRN-01.1:Submit | Valid request created <60s with `REQ-####` + `New` | US-1.1 |
| JTBD-01.2 | Self-service status | JRN-01.1:Track | Requester sees own requests + status; scoped access | US-0.2, US-4.1 |
| JTBD-01.3 | Add detail without duplicates | JRN-01.1:Submit | Edit own open request; no duplicate | US-1.3 |
| JTBD-02.1 | Prioritized triage | JRN-02.1:Orient/Prioritize | Priority→age sort, <1s@500, filters/search, assign | US-3.1, US-3.2, US-5.1 |
| JTBD-02.2 | Auditable lifecycle | JRN-02.1:Work/Close | Note-gated transitions; append-only log; resolution note | US-2.1, US-2.3, US-5.2 |
| JTBD-02.3 | No stalled blocked work | JRN-02.1:Work | 3-day nudge; internal-safe comments | US-2.2, US-4.2, US-6.2 |
| JTBD-03.1 | Self-serve config | JRN-03.1:Configure | ≤5 typed custom fields; role management | US-8.1, US-8.2 |
| JTBD-03.2 | Cycle-time visibility | JRN-03.1:Measure/Share | On-page metrics; filter-accurate export | US-7.1, US-7.2 |

## Release Planning

### R1 — Core Workflow (MVP)
- **Stories:** US-0.1, US-0.2, US-1.1, US-3.1, US-3.2, US-5.1, US-2.1, US-2.3, US-4.1, US-4.2, US-5.2
- **Personas served:** Requester (submit + track), Fulfiller (triage + work + close)
- **JTBD addressed:** JTBD-01.1, JTBD-01.2, JTBD-02.1, JTBD-02.2
- **Why:** Completes the end-to-end submit → triage → close journey — the core value.

### R2 — Visibility & Configuration
- **Stories:** US-1.2, US-1.3, US-3.3, US-2.2, US-6.1, US-7.1, US-8.1, US-8.2
- **Personas served:** adds Admin (config + reporting); enriches Requester/Fulfiller
- **JTBD addressed:** JTBD-01.3, JTBD-02.3, JTBD-03.1, JTBD-03.2
- **Why:** Closes the loop with notifications and gives admins configuration + insight.

### R3 — Refinements
- **Stories:** US-6.2, US-7.2
- **Personas served:** Fulfiller/Admin convenience
- **JTBD addressed:** deepens JTBD-02.3, JTBD-03.2
- **Why:** Mute controls and CSV export are valuable polish, not blocking.

## Coverage Analysis

**Persona coverage:**
- PER-01 Requester: served from R1 (submit, track, detail); enriched R2 (attach, add detail, notifications).
- PER-02 Fulfiller: fully served R1 (triage, assign, work, close); R2/R3 add saved views, nudges, export/mute.
- PER-03 Admin: served R2 (types, roles, reporting); R3 adds export.

**JTBD coverage:** All 8 JTBD jobs map to at least one story with a derived NaC. No unaddressed outcomes.

**Gap analysis:**
- No journey stage lacks a mapped story — every stage in JRN-01.1/02.1/03.1 has coverage.
- No orphan stories — all 21 user stories (US-0.1 … US-8.2) appear in the matrix.
- Admin has no R1 stories by design (config/reporting is not on the MVP critical path); R1 seeds a default request type so submission works before F8 lands.

## NaC-to-Acceptance Criteria Mapping

| Story | NaC (this map) | Aligns with UserStories AC |
|---|---|---|
| US-1.1 | Valid request <60s, `REQ-####`, `New` | "On submit I get a `REQ-####` ID, `New` status, timestamp, confirmation" |
| US-3.1 | Priority→age sort, <1s@500 | "Sorted by priority then age"; "loads in under 1 second with 500 open" |
| US-2.1 | Note-gated transitions, logged | "Blocked/Cancelled/Done require a note"; "appends actor+timestamp" |
| US-4.2 | Internal comments hidden from requester | "Internal comments are never shown to the requester" |
| US-5.2 | Done requires resolution note, `closed_at` | "Closing sets status Done and requires a resolution note; `closed_at` recorded" |
| US-8.1 | ≤5 typed custom fields, activate | "Up to 5 custom fields"; "only active types appear on the submit form" |
| US-7.1 | On-page median & p90 time-to-close | "Shows median and p90 time-to-close, overall and by type" |
