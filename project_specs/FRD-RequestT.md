# Functional Requirements Document: RequestT

**Version:** 1.0
**Project:** RequestT
**Generated:** 2026-09-03
**Source:** Derived from `project_specs/PRD-RequestT.md`

## Scope

This FRD specifies the functional behavior of RequestT, a lightweight internal request tracker. Each PRD feature (F0–F8) is detailed with its process, inputs, outputs, validation rules, and error states. Consolidated database schema, API endpoints, error catalog, and integration points appear in the `Y*` chunks.

## Conventions

- **Feature IDs** map directly to PRD feature IDs: `F0`–`F8`.
- **Roles:** Requester, Fulfiller, Admin (additive — see F0).
- **Entities** follow the PRD data model: User, RequestType, Request, Comment, Attachment, Activity.
- Error states list HTTP status, error code, and message.

## Table of Contents

- F00 — SSO Authentication & Role Model
- F01 — Submit a Request
- F02 — Status Lifecycle & Activity Log
- F03 — Queue View
- F04 — Request Detail
- F05 — Assignment & Fulfillment
- F06 — Notifications
- F07 — Reporting & Export
- F08 — Administration
- Y0 — Database Schema (DDL)
- Y1 — API Endpoints
- Y2 — Error Catalog
- Y3 — Integration Points

## Shared Terminology

- **Request:** The canonical record of a single ask, identified by a human-readable short ID (`REQ-1042`).
- **Queue:** The single shared list of all requests (no multi-tenancy).
- **Activity log:** Append-only record of every status change, assignment change, and field edit.
- **Internal comment:** A comment visible only to Fulfillers/Admins, never to the Requester.
- **Custom field:** An admin-defined field on a RequestType (text, number, select, date), max 5 per type.
## F00: SSO Authentication & Role Model

**Description:** Users authenticate through the organization's existing identity provider via SSO — no separate passwords. Three additive roles (Requester, Fulfiller, Admin) govern authorization across every feature.

**Terminology:**
- **IdP:** The org's identity provider (OIDC or SAML).
- **Additive role:** A higher role inherits all lower-role permissions (Fulfiller ⊇ Requester; Admin ⊇ Fulfiller).

**Sub-features:**
- SSO login/logout via org IdP
- Session establishment and expiry
- Role resolution (Requester / Fulfiller / Admin)
- Role-based authorization enforced on every request

**Process:**
1. Unauthenticated user hits the app and is redirected to the IdP.
2. IdP authenticates and returns identity assertion.
3. System provisions or matches a User record by email.
4. System resolves the user's role(s) and establishes a session.
5. Every subsequent action is authorized against the user's effective role.

**Inputs:**
- IdP assertion (from provider): subject, email, display name
- Requested resource + action (implicit per API call)

**Outputs:**
- Authenticated session
- Effective role for the user
- 401/403 on failed auth/authz

**Validation:**
- Email from assertion must be present and well-formed.
- Role must be one of Requester, Fulfiller, Admin.
- A Requester may only read/write their own requests and non-internal comments.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Not authenticated | 401 | AUTH_REQUIRED | "Sign in to continue" |
| IdP unavailable | 503 | IDP_UNAVAILABLE | "Sign-in service unavailable" |
| Insufficient role | 403 | FORBIDDEN | "You don't have access to this" |
| Requester accessing other's request | 403 | FORBIDDEN | "You don't have access to this request" |

**API Surface (this feature):** see `Y1-api.md` §Auth.

**Schema Surface (this feature):** uses `users` — see `Y0-schema.md`.
## F01: Submit a Request

**Description:** Any org member submits a request through a short form in under a minute. On submit the request receives a human-readable ID, `New` status, and a timestamp, and the requester gets a confirmation.

**Terminology:**
- **Short ID:** Human-readable request identifier, e.g. `REQ-1042`.
- **Custom field values:** Values for the RequestType's admin-defined fields.

**Sub-features:**
- Core submit form (title, description, type, suggested priority)
- Type-driven custom fields (up to 5: text, number, select, date)
- Attachments (up to 5 files, 25 MB each)
- Post-submit edit to add a missing detail

**Process:**
1. Requester opens the submit form.
2. Requester selects a request type; the form renders that type's custom fields.
3. Requester fills title, description, suggested priority, custom fields; optionally attaches files.
4. On submit, system validates, assigns the next short ID, sets status `New`, stamps `created_at`.
5. System records a creation entry in the activity log.
6. System sends confirmation to the requester and fires the "request submitted" notification to fulfillers.
7. Requester may later edit fields or add custom-field values on their own open request.

**Inputs:**
- `title` (string, required)
- `description` (string, required)
- `type_id` (uuid, required — from active RequestTypes)
- `priority` (enum, required — requester-suggested)
- `custom_field_values` (object, optional — validated against type schema)
- `attachments` (file[], optional — ≤5 files, ≤25 MB each)

**Outputs:**
- Created Request with `short_id`, `status = New`, `created_at`
- Confirmation to requester
- Activity log entry (create)

**Validation:**
- `title` and `description` required, non-empty.
- `type_id` must reference an active RequestType.
- `priority` must be a valid value.
- Custom field values must match the type's schema (types, required, select options).
- Attachments: max 5, each ≤25 MB.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Missing required field | 422 | VALIDATION_ERROR | "Title and description are required" |
| Inactive/unknown type | 422 | INVALID_TYPE | "Select a valid request type" |
| Custom field mismatch | 422 | CUSTOM_FIELD_ERROR | "One or more fields are invalid" |
| Attachment too large | 413 | FILE_TOO_LARGE | "Files must be 25 MB or smaller" |
| Too many attachments | 422 | TOO_MANY_FILES | "Attach at most 5 files" |

**API Surface (this feature):** see `Y1-api.md` §Requests.

**Schema Surface (this feature):** uses `requests`, `request_types`, `attachments`, `activity` — see `Y0-schema.md`.
## F02: Status Lifecycle & Activity Log

**Description:** Requests move through a single linear-ish lifecycle. Any fulfiller can move a request to any status. Some transitions require a note. Every change is recorded in an append-only activity log.

**Terminology:**
- **Status:** One of New, Triaged, In Progress, Waiting on requester, Blocked, Done, Cancelled.
- **Append-only:** Activity entries are never edited or deleted, even for cancelled requests.

**Sub-features:**
- Status transitions (any fulfiller → any status)
- Required notes on Blocked (note), Cancelled (reason), Done (resolution note)
- 3-day auto-nudge when in Waiting on requester
- Append-only activity log with actor + timestamp

**Process:**
1. Fulfiller opens a request and selects a new status.
2. If target status requires a note/reason/resolution, system prompts for it and blocks the transition until provided.
3. System updates `status`, sets `closed_at` on Done/Cancelled, writes an activity entry (actor, old→new, timestamp).
4. System fires the "status changed" notification to the requester.
5. A scheduled job checks Waiting-on-requester requests; at 3 days it fires the requester nudge.

**Status definitions:**
- `New` — submitted, nobody has looked at it.
- `Triaged` — accepted, priority set, not started.
- `In Progress` — someone is actively working it.
- `Waiting on requester` — blocked pending an answer; auto-nudges requester at 3 days.
- `Blocked` — blocked on something else; requires a note.
- `Done` — closed with a resolution note.
- `Cancelled` — closed without work; requires a reason.

**Inputs:**
- `request_id` (uuid, required)
- `new_status` (enum, required)
- `note` (string, conditionally required for Blocked/Cancelled/Done)

**Outputs:**
- Updated request status (+ `closed_at` when terminal)
- Activity log entry
- Status-changed notification

**Validation:**
- Actor must be a Fulfiller or Admin.
- `new_status` must be a valid status.
- Blocked → note required; Cancelled → reason required; Done → resolution note required.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-fulfiller changes status | 403 | FORBIDDEN | "Only fulfillers can change status" |
| Missing required note | 422 | NOTE_REQUIRED | "A note is required for this status" |
| Invalid status value | 422 | INVALID_STATUS | "Unknown status" |

**API Surface (this feature):** see `Y1-api.md` §Requests, §Activity.

**Schema Surface (this feature):** uses `requests`, `activity` — see `Y0-schema.md`.
## F03: Queue View

**Description:** The team's working surface. Shows all open requests, sortable, filterable, and searchable, with saved views and bulk actions. Must load in under 1s at 500 open requests.

**Terminology:**
- **Saved view:** A named, reusable filter set (My requests, Unassigned, Waiting on requester, Closed this month).
- **Bulk action:** Assign or status-change applied to multiple selected rows at once.

**Sub-features:**
- Default view: all open requests, sorted by priority then age
- Filters: status, assignee (incl. unassigned), type, priority, date range
- Free-text search across title, description, comments
- Saved views (system + per-user)
- Bulk assign / bulk status change

**Process:**
1. User opens the queue; system loads open requests sorted by priority then age.
2. User applies filters and/or search; system returns matching rows (permission-scoped for Requesters).
3. User optionally selects a saved view to apply a preset filter set.
4. Fulfiller selects rows and performs a bulk assign or bulk status change.
5. Bulk actions write an activity entry per affected request and fire notifications.

**Inputs:**
- `filters` (object, optional: status, assignee, type, priority, date_range)
- `q` (string, optional — free-text search)
- `saved_view` (id, optional)
- `selection` + `bulk_action` (for bulk operations)

**Outputs:**
- Paginated, sorted list of requests
- Applied filter state
- Bulk-action results (per-request success/failure)

**Validation:**
- Requester queries are always scoped to their own requests.
- Bulk status change obeys F02 note requirements per request.
- Date range must be a valid interval.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Invalid filter value | 422 | VALIDATION_ERROR | "Invalid filter" |
| Bulk action partial failure | 207 | BULK_PARTIAL | "Some requests could not be updated" |
| Requester bulk action | 403 | FORBIDDEN | "Only fulfillers can bulk-update" |

**API Surface (this feature):** see `Y1-api.md` §Requests (list), §SavedViews.

**Schema Surface (this feature):** uses `requests`, `saved_views` — see `Y0-schema.md`.
## F04: Request Detail

**Description:** The full record for a single request: header metadata, body, comments (with an internal-only toggle for fulfillers), and the activity log.

**Terminology:**
- **Participant:** Requester, assignee, and any commenter on a request — recipients of "new comment" notifications.
- **Internal comment:** Comment flagged `internal = true`, hidden from the Requester.

**Sub-features:**
- Header (ID, title, status, priority, type, requester, assignee, created/updated)
- Body (description, custom fields, attachments)
- Comments (chronological, plain text + attachments, internal-only toggle)
- Activity log (status changes, assignment changes, field edits)

**Process:**
1. User opens a request by ID.
2. System authorizes: Requesters only see their own requests and non-internal comments.
3. System renders header, body, permitted comments, and activity log.
4. A participant adds a comment; fulfillers may mark it internal.
5. System stores the comment, fires "new comment" to other participants (respecting internal visibility), and appends nothing to activity for comments (comments are their own stream).

**Inputs:**
- `request_id` (uuid, required)
- Comment: `body` (string, required), `internal` (bool, fulfiller-only), `attachments` (optional)

**Outputs:**
- Full request detail (permission-scoped)
- Stored comment + notification

**Validation:**
- Requester cannot view internal comments or others' requests.
- `internal = true` only settable by Fulfiller/Admin.
- Comment body non-empty.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Unauthorized request access | 403 | FORBIDDEN | "You don't have access to this request" |
| Requester sets internal flag | 403 | FORBIDDEN | "Only fulfillers can post internal comments" |
| Empty comment | 422 | VALIDATION_ERROR | "Comment can't be empty" |
| Request not found | 404 | NOT_FOUND | "Request not found" |

**API Surface (this feature):** see `Y1-api.md` §Requests, §Comments.

**Schema Surface (this feature):** uses `requests`, `comments`, `attachments`, `activity` — see `Y0-schema.md`.
## F05: Assignment & Fulfillment

**Description:** Fulfillers pull work from the unassigned pool, assign requests to themselves or teammates, and close requests with a resolution note.

**Terminology:**
- **Unassigned:** A request with `assignee_id = null`.
- **Assignment change:** Setting or reassigning `assignee_id`, recorded in the activity log.

**Sub-features:**
- View unassigned requests
- Assign to self or a teammate
- Reassign
- Close with resolution note (via F02 Done transition)

**Process:**
1. Fulfiller filters the queue to unassigned (F03).
2. Fulfiller assigns a request to self or a teammate.
3. System sets `assignee_id`, writes an activity entry, fires "assigned to you" to the assignee.
4. To close, fulfiller sets status to Done (F02) with a resolution note.

**Inputs:**
- `request_id` (uuid, required)
- `assignee_id` (uuid | null, required for assign/unassign)

**Outputs:**
- Updated `assignee_id`
- Activity log entry
- "Assigned to you" notification

**Validation:**
- Actor must be Fulfiller/Admin.
- `assignee_id` must reference a Fulfiller/Admin (or null to unassign).

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-fulfiller assigns | 403 | FORBIDDEN | "Only fulfillers can assign" |
| Assignee not a fulfiller | 422 | INVALID_ASSIGNEE | "Assignee must be a team member" |
| Request not found | 404 | NOT_FOUND | "Request not found" |

**API Surface (this feature):** see `Y1-api.md` §Requests (assign).

**Schema Surface (this feature):** uses `requests`, `activity` — see `Y0-schema.md`.
## F06: Notifications

**Description:** Email plus in-app notifications for the events that matter, with per-category muting per user. Two time-triggered nudges (3-day waiting, 7-day untouched) run on a schedule.

**Terminology:**
- **Notification category:** The event class a user can mute (submitted, assigned, status, comment, nudges).
- **Digest:** Optional batched delivery of "request submitted" events for fulfillers.

**Sub-features:**
- Event notifications (email + in-app)
- Per-category mute preferences
- Digest option for "request submitted"
- Scheduled nudges (3-day waiting-on-requester, 7-day untouched)

**Event table:**
| Event | Notifies |
|---|---|
| Request submitted | All fulfillers (digest option) |
| Assigned to you | Assignee |
| Status changed | Requester |
| New comment | Other participants (internal comments to fulfillers only) |
| Waiting on requester for 3 days | Requester |
| Open and untouched for 7 days | Assignee, or team lead if unassigned |

**Process:**
1. A feature event (submit, assign, status change, comment) fires a notification request.
2. System resolves recipients and filters out those who muted that category.
3. System delivers in-app + email (or queues into a digest).
4. A scheduled job scans requests: Waiting-on-requester ≥3 days → nudge requester; open + untouched ≥7 days → nudge assignee (or team lead if unassigned).

**Inputs:**
- Event payload (type, request_id, actor, participants)
- User `notification_prefs`

**Outputs:**
- In-app notification records
- Email deliveries (or digest queue entries)

**Validation:**
- Never deliver internal-comment notifications to Requesters.
- Respect per-category mute settings.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Email delivery failure | 502 | EMAIL_FAILED | "Notification email could not be sent" (retried) |
| Invalid prefs update | 422 | VALIDATION_ERROR | "Invalid notification preferences" |

**API Surface (this feature):** see `Y1-api.md` §Notifications.

**Schema Surface (this feature):** uses `notifications`, `users.notification_prefs` — see `Y0-schema.md`.
## F07: Reporting & Export

**Description:** A single reporting page (no builder) showing throughput and cycle-time metrics, plus CSV export of any filtered queue view.

**Terminology:**
- **Time-to-close:** `closed_at - created_at` for Done requests.
- **p90:** 90th percentile of time-to-close.

**Sub-features:**
- Opened vs. closed by week (last 12 weeks)
- Open requests by status and by assignee
- Median and p90 time-to-close, overall and by type
- CSV export of a filtered queue view

**Process:**
1. Fulfiller/Admin opens the reporting page.
2. System computes aggregates from requests and activity data.
3. System renders the four report sections.
4. From any queue view (F03), user clicks Export; system streams a CSV of the current filtered set.

**Inputs:**
- Reporting page: none (uses full dataset, permission-scoped)
- Export: current `filters` + `q` from the queue view

**Outputs:**
- Rendered report widgets
- CSV file (columns match the queue view)

**Validation:**
- Reporting visible to Fulfiller/Admin only.
- Export reflects exactly the applied filters.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Requester opens reporting | 403 | FORBIDDEN | "Reporting is for team members" |
| Export too large | 413 | EXPORT_TOO_LARGE | "Narrow the filter and try again" |

**API Surface (this feature):** see `Y1-api.md` §Reports.

**Schema Surface (this feature):** reads `requests`, `activity` — see `Y0-schema.md`.
## F08: Administration

**Description:** Admins configure request types and their custom-field schemas, manage categories, and manage user roles.

**Terminology:**
- **Custom-field schema:** The definition of a RequestType's up-to-5 custom fields (name, type, required, options).
- **Active flag:** Whether a RequestType is offered on the submit form.

**Sub-features:**
- Create/edit/deactivate request types
- Define custom-field schema per type (text, number, select, date; max 5)
- Manage categories
- Manage user roles (Requester/Fulfiller/Admin)

**Process:**
1. Admin opens administration.
2. Admin creates or edits a request type, defining its custom-field schema.
3. System validates the schema (≤5 fields, valid types, select options present).
4. Admin sets a type active/inactive; only active types appear on the submit form.
5. Admin assigns or changes a user's role; change takes effect on next authorization check.

**Inputs:**
- RequestType: `name`, `description`, `custom_field_schema`, `active`
- Role change: `user_id`, `role`

**Outputs:**
- Persisted RequestType / category / role
- Activity/audit entry for role and type changes

**Validation:**
- Admin-only.
- Custom-field schema: ≤5 fields; each has name + valid type; select fields have ≥1 option.
- Role must be Requester/Fulfiller/Admin.
- Deactivating a type does not affect existing requests of that type.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-admin access | 403 | FORBIDDEN | "Admin access required" |
| >5 custom fields | 422 | SCHEMA_ERROR | "A type can have at most 5 custom fields" |
| Invalid field type | 422 | SCHEMA_ERROR | "Unsupported field type" |
| Select without options | 422 | SCHEMA_ERROR | "Select fields need at least one option" |

**API Surface (this feature):** see `Y1-api.md` §Admin.

**Schema Surface (this feature):** uses `request_types`, `users`, `categories` — see `Y0-schema.md`.
## Y0: Database Schema (DDL)

Illustrative relational DDL (PostgreSQL dialect). Concrete engine and types finalized in TechArch.

```sql
CREATE TYPE user_role AS ENUM ('requester', 'fulfiller', 'admin');
CREATE TYPE request_status AS ENUM (
  'new', 'triaged', 'in_progress', 'waiting_on_requester',
  'blocked', 'done', 'cancelled'
);
CREATE TYPE request_priority AS ENUM ('low', 'normal', 'high', 'urgent');
CREATE TYPE field_type AS ENUM ('text', 'number', 'select', 'date');

CREATE TABLE users (
  id                 UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name               TEXT NOT NULL,
  email              TEXT NOT NULL UNIQUE,
  role               user_role NOT NULL DEFAULT 'requester',
  notification_prefs JSONB NOT NULL DEFAULT '{}',
  created_at         TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE categories (
  id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name   TEXT NOT NULL,
  active BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE request_types (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                TEXT NOT NULL,
  description         TEXT,
  custom_field_schema JSONB NOT NULL DEFAULT '[]',  -- <=5 field defs
  active              BOOLEAN NOT NULL DEFAULT true,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT max_five_fields CHECK (jsonb_array_length(custom_field_schema) <= 5)
);

CREATE SEQUENCE request_short_id_seq START 1000;

CREATE TABLE requests (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  short_id            TEXT NOT NULL UNIQUE,            -- e.g. REQ-1042
  title               TEXT NOT NULL,
  description         TEXT NOT NULL,
  type_id             UUID NOT NULL REFERENCES request_types(id),
  status              request_status NOT NULL DEFAULT 'new',
  priority            request_priority NOT NULL DEFAULT 'normal',
  requester_id        UUID NOT NULL REFERENCES users(id),
  assignee_id         UUID REFERENCES users(id),
  custom_field_values JSONB NOT NULL DEFAULT '{}',
  resolution_note     TEXT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  closed_at           TIMESTAMPTZ
);
CREATE INDEX idx_requests_status   ON requests(status);
CREATE INDEX idx_requests_assignee ON requests(assignee_id);
CREATE INDEX idx_requests_type     ON requests(type_id);
CREATE INDEX idx_requests_priority_created ON requests(priority, created_at);

CREATE TABLE comments (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  author_id  UUID NOT NULL REFERENCES users(id),
  body       TEXT NOT NULL,
  internal   BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_comments_request ON comments(request_id);

CREATE TABLE attachments (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id  UUID NOT NULL REFERENCES requests(id),
  comment_id  UUID REFERENCES comments(id),
  filename    TEXT NOT NULL,
  size_bytes  BIGINT NOT NULL CHECK (size_bytes <= 26214400), -- 25 MB
  url         TEXT NOT NULL,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Append-only; no UPDATE/DELETE by policy.
CREATE TABLE activity (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  actor_id   UUID NOT NULL REFERENCES users(id),
  field      TEXT NOT NULL,          -- 'status' | 'assignee' | field name
  old_value  TEXT,
  new_value  TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_activity_request ON activity(request_id, created_at);

CREATE TABLE saved_views (
  id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES users(id),  -- null = system view
  name    TEXT NOT NULL,
  filters JSONB NOT NULL DEFAULT '{}'
);

CREATE TABLE notifications (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES users(id),
  request_id UUID REFERENCES requests(id),
  category   TEXT NOT NULL,
  body       TEXT NOT NULL,
  read_at    TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_notifications_user ON notifications(user_id, read_at);
```

**Retention note:** closed requests remain queryable for 2 years, then archive. Activity is append-only and exempt from deletion.
## Y1: API Endpoints

All endpoints require an authenticated SSO session. Authorization is role-scoped per F00. JSON over HTTPS.

### §Auth
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/auth/login` | Redirect to IdP | any |
| GET | `/auth/callback` | IdP callback, establish session | any |
| POST | `/auth/logout` | End session | any |
| GET | `/me` | Current user + effective role | any |

### §Requests
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests` | List/filter/search (paginated, sorted priority→age) | any (scoped) |
| POST | `/requests` | Create a request | any |
| GET | `/requests/{id}` | Request detail | owner or fulfiller |
| PATCH | `/requests/{id}` | Edit fields / add detail | owner (own) or fulfiller |
| POST | `/requests/{id}/status` | Change status (note-gated) | fulfiller |
| POST | `/requests/{id}/assign` | Assign / reassign / unassign | fulfiller |
| POST | `/requests/bulk` | Bulk assign / status change | fulfiller |

`GET /requests` query params: `status`, `assignee`, `type`, `priority`, `date_from`, `date_to`, `q`, `saved_view`, `page`, `page_size`.

### §Comments
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests/{id}/comments` | List comments (internal filtered for requesters) | owner or fulfiller |
| POST | `/requests/{id}/comments` | Add comment (`internal` fulfiller-only) | owner or fulfiller |

### §Activity
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests/{id}/activity` | Append-only activity log | owner or fulfiller |

### §SavedViews
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/saved-views` | List system + own views | any |
| POST | `/saved-views` | Create a personal saved view | any |

### §Notifications
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/notifications` | In-app notifications | any |
| POST | `/notifications/{id}/read` | Mark read | any |
| PATCH | `/me/notification-prefs` | Update per-category mutes | any |

### §Reports
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/reports` | Throughput + cycle-time aggregates | fulfiller |
| GET | `/requests/export` | CSV export of a filtered view | fulfiller |

### §Admin
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET/POST | `/admin/request-types` | List/create request types | admin |
| PATCH | `/admin/request-types/{id}` | Edit schema / active flag | admin |
| GET/POST | `/admin/categories` | Manage categories | admin |
| PATCH | `/admin/users/{id}/role` | Change a user's role | admin |
## Y2: Error Catalog

Cross-feature error scenarios with HTTP status and retry guidance.

| Error Code | HTTP | Meaning | Retry? |
|---|---|---|---|
| AUTH_REQUIRED | 401 | No valid session | Re-authenticate |
| IDP_UNAVAILABLE | 503 | Identity provider down | Retry with backoff |
| FORBIDDEN | 403 | Role/ownership check failed | No |
| NOT_FOUND | 404 | Resource does not exist | No |
| VALIDATION_ERROR | 422 | Generic input validation failure | No — fix input |
| INVALID_TYPE | 422 | Request type inactive/unknown | No |
| CUSTOM_FIELD_ERROR | 422 | Custom field values fail type schema | No |
| FILE_TOO_LARGE | 413 | Attachment > 25 MB | No |
| TOO_MANY_FILES | 422 | > 5 attachments | No |
| NOTE_REQUIRED | 422 | Status transition needs a note/reason | No |
| INVALID_STATUS | 422 | Unknown status value | No |
| INVALID_ASSIGNEE | 422 | Assignee not a fulfiller | No |
| BULK_PARTIAL | 207 | Some bulk items failed | Per-item |
| SCHEMA_ERROR | 422 | Invalid custom-field schema | No |
| EXPORT_TOO_LARGE | 413 | Export result set too large | Narrow filter |
| EMAIL_FAILED | 502 | Notification email delivery failed | Auto-retried |
| RATE_LIMITED | 429 | Too many requests | Retry after delay |
## Y3: Integration Points

| Integration | Direction | Purpose | Notes |
|---|---|---|---|
| Org Identity Provider (OIDC/SAML SSO) | Inbound auth | Authenticate users, no local passwords | Required for F0; provisions User on first login |
| Email delivery service (SMTP/API) | Outbound | Send notification emails and digests | F6; retried on failure (EMAIL_FAILED) |
| Object storage | Outbound/Inbound | Store and serve request/comment attachments | F1/F4; ≤25 MB per file, ≤5 per request |
| Scheduled job runner | Internal | Fire 3-day waiting and 7-day untouched nudges | F2/F6; runs periodically over open requests |

**Explicitly not integrated in v1** (see PRD Non-goals / Out of Scope): Slack, email-to-request intake, external SLA/automation engines, billing/time-tracking systems.
