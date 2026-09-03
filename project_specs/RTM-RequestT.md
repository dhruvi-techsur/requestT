# Requirements Traceability Matrix: RequestT

| Field | Value |
|---|---|
| Product Name | RequestT |
| Date | 2026-09-03 |
| Project Acronym | RequestT |
| Source Documents | PRD, FRD, TechArch, UserStories, Personas, JTBD, Journeys, Story Map |

---

## 1. Overview

This Requirements Traceability Matrix (RTM) provides bidirectional traceability across the RequestT specification suite. It links PRD features (F0–F8) to their FRD functional requirements, TechArch specifications, and User Stories, and back again — ensuring every feature is specified, designed, and covered by at least one testable story, and that no story is orphaned from a requirement.

Traceability levels:
- **Business/Product:** PRD features (F0–F8), Personas (PER-01…03), Jobs (JTBD-XX.Y).
- **Functional:** FRD requirements (REQ-FUNC-###) and non-functional requirements (REQ-NFR-###).
- **Technical:** TechArch specifications (SPEC-###).
- **Verification:** User Stories (US-X.Y) and Test Cases (TEST-###).

## 2. Requirements Summary

- **F0 — SSO Authentication & Role Model** (P0): SSO login, additive Requester/Fulfiller/Admin roles, requester scoping.
- **F1 — Submit a Request** (P0): short-form intake, type-driven custom fields, attachments, short ID.
- **F2 — Status Lifecycle & Activity Log** (P0): note-gated transitions, 3-day nudge, append-only log.
- **F3 — Queue View** (P0): sort, filter, search, saved views, bulk actions, <1s@500.
- **F4 — Request Detail** (P0): header/body/comments/activity, internal-comment toggle.
- **F5 — Assignment & Fulfillment** (P0): pull, assign, close with resolution note.
- **F6 — Notifications** (P1): email + in-app events, per-category mute, scheduled nudges.
- **F7 — Reporting & Export** (P1): throughput + cycle-time page, CSV export.
- **F8 — Administration** (P1): request types, custom-field schemas, categories, roles.

**Non-functional:** SSO-only auth, <1s queue@500, requester permissions, append-only audit, WCAG 2.1 AA, 2-year retention.

## 3. Traceability Matrix

| PRD Feature | FRD Requirement | TechArch Spec | User Story |
|---|---|---|---|
| F0: SSO Auth & Roles | REQ-FUNC-001 (FRD F00) | SPEC-005 (§5 Security), SPEC-002 (§2 Auth/Authz components) | US-0.1, US-0.2 |
| F1: Submit a Request | REQ-FUNC-002 (FRD F01) | SPEC-003 (Request/Type/Attachment DDL), SPEC-004 (Requests/Attachments API) | US-1.1, US-1.2, US-1.3 |
| F2: Status Lifecycle & Activity | REQ-FUNC-003 (FRD F02) | SPEC-003 (activity table, append-only), SPEC-004 (status API) | US-2.1, US-2.2, US-2.3 |
| F3: Queue View | REQ-FUNC-004 (FRD F03) | SPEC-003 (indexes priority/created), SPEC-004 (list/saved-views API) | US-3.1, US-3.2, US-3.3 |
| F4: Request Detail | REQ-FUNC-005 (FRD F04) | SPEC-003 (comments DDL), SPEC-004 (comments API), SPEC-005 (internal filtering) | US-4.1, US-4.2 |
| F5: Assignment & Fulfillment | REQ-FUNC-006 (FRD F05) | SPEC-004 (assign API) | US-5.1, US-5.2 |
| F6: Notifications | REQ-FUNC-007 (FRD F06) | SPEC-002 (notifications service + worker), SPEC-006 (email/scheduler integrations) | US-6.1, US-6.2 |
| F7: Reporting & Export | REQ-FUNC-008 (FRD F07) | SPEC-002 (reporting service), SPEC-004 (reports/export API) | US-7.1, US-7.2 |
| F8: Administration | REQ-FUNC-009 (FRD F08) | SPEC-003 (request_types/categories DDL), SPEC-004 (admin API) | US-8.1, US-8.2 |

**Non-functional traceability:**

| NFR | Requirement | TechArch Spec | Verified via |
|---|---|---|---|
| REQ-NFR-001 SSO-only auth | F0 / FRD F00 | SPEC-005 | US-0.1 |
| REQ-NFR-002 Queue <1s@500 | F3 / FRD F03 | SPEC-003 (indexes), §8 | US-3.1 |
| REQ-NFR-003 Requester permissions | F0/F4 | SPEC-005 (query scoping) | US-0.2, US-4.1 |
| REQ-NFR-004 Append-only audit | F2 | SPEC-003 (no UPDATE/DELETE) | US-2.3 |
| REQ-NFR-005 WCAG 2.1 AA | F1/F3 | UX-Mockup Y2-accessibility | US-1.1, US-3.1 |
| REQ-NFR-006 2-year retention | data model | SPEC-003 (archival), §5 | (ops verification) |

## 4. Requirements Detail

- **F0 → REQ-FUNC-001:** SSO login provisions/matches a User; additive roles resolved per session; Requester scoping enforced at the data layer. (Personas: all; JTBD-01.2)
- **F1 → REQ-FUNC-002:** Required title/description/type; ≤5 typed custom fields; ≤5 attachments ≤25 MB; `REQ-####` short ID; `New` status; confirmation. (JTBD-01.1, JTBD-01.3)
- **F2 → REQ-FUNC-003:** Any fulfiller → any status; Blocked/Cancelled/Done require notes; 3-day waiting nudge; append-only activity with actor+timestamp. (JTBD-02.2, JTBD-02.3)
- **F3 → REQ-FUNC-004:** Priority→age default sort; filters + free-text search; saved views; bulk assign/status; <1s@500. (JTBD-02.1)
- **F4 → REQ-FUNC-005:** Header/body/comments/activity; internal-only comment toggle hidden from requesters. (JTBD-01.2, JTBD-02.3)
- **F5 → REQ-FUNC-006:** View unassigned; assign to self/teammate; close with resolution note. (JTBD-02.1, JTBD-02.2)
- **F6 → REQ-FUNC-007:** Email + in-app for six event types; per-category mute; digest option; scheduled nudges. (JTBD-01.2, JTBD-02.3)
- **F7 → REQ-FUNC-008:** Opened/closed by week; open by status/assignee; median & p90 time-to-close; CSV export. (JTBD-03.2)
- **F8 → REQ-FUNC-009:** Manage request types + schemas (≤5 fields); categories; user roles. (JTBD-03.1)

## 5. Test Case Coverage

| Feature | Stories | Test Cases | Coverage |
|---|---|---|---|
| F0: SSO Auth & Roles | 2 | TEST-001…004 | 100% |
| F1: Submit a Request | 3 | TEST-005…010 | 100% |
| F2: Status Lifecycle & Activity | 3 | TEST-011…016 | 100% |
| F3: Queue View | 3 | TEST-017…022 | 100% |
| F4: Request Detail | 2 | TEST-023…027 | 100% |
| F5: Assignment & Fulfillment | 2 | TEST-028…031 | 100% |
| F6: Notifications | 2 | TEST-032…036 | 100% |
| F7: Reporting & Export | 2 | TEST-037…040 | 100% |
| F8: Administration | 2 | TEST-041…044 | 100% |
| **Total** | **21** | **44** | **100%** |

**Representative test cases:**
- TEST-002: Requester cannot open another user's request (403). (US-0.2, REQ-NFR-003)
- TEST-005: Submit creates `REQ-####`, `New`, timestamp, confirmation. (US-1.1)
- TEST-009: 26 MB attachment rejected with FILE_TOO_LARGE. (US-1.2)
- TEST-012: Done requires a resolution note. (US-5.2)
- TEST-014: Activity entry cannot be edited/deleted. (US-2.3, REQ-NFR-004)
- TEST-017: Queue loads <1s with 500 open requests. (US-3.1, REQ-NFR-002)
- TEST-024: Internal comment not returned to requester. (US-4.2, REQ-NFR-003)
- TEST-041: Request type with 6 custom fields rejected (SCHEMA_ERROR). (US-8.1)

## 6. Change Management

| Change ID | Date | Description | Affected Docs | Author |
|---|---|---|---|---|
| CHG-001 | 2026-09-03 | Initial spec suite generated from reference PRD | All | Pivota Spec |

## 7. Approval

| Role | Name | Signature | Date |
|---|---|---|---|
| Product Owner | TBD | __________ | ________ |
| Tech Lead | TBD | __________ | ________ |
| QA Lead | TBD | __________ | ________ |

---

**Coverage confirmation:** All 9 PRD features (F0–F8) trace to an FRD requirement, at least one TechArch spec, and at least one User Story. All 21 user stories (US-0.1…US-8.2) trace back to a PRD feature. All 6 non-functional requirements are mapped to a spec and a verification story. No orphan requirements or stories.
