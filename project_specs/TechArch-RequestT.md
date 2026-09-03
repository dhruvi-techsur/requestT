# Technical Architecture: RequestT

**Version:** 1.0
**Project:** RequestT
**Generated:** 2026-09-03
**Source:** Derived from `project_specs/PRD-RequestT.md` and `project_specs/FRD-RequestT.md`

## 1. Architectural Overview

RequestT is a single-tenant internal web application: a browser SPA talking to a REST API backed by a relational database and object storage, with SSO delegated to the org's identity provider and a scheduled worker for time-based nudges. The scale is modest (low-hundreds of open requests, ≤20 fulfillers), so a straightforward monolithic API with a relational store is the right pattern — no microservices, no event bus, no multi-tenancy.

**Key architectural decisions:**
- **Monolithic API + SPA** — smallest moving-part count for a small team; matches the sub-1s queue requirement without distributed complexity.
- **Relational database** — the data is highly relational (requests ↔ comments ↔ activity) and reporting needs aggregate queries.
- **Append-only activity table** — enforced by application policy (no UPDATE/DELETE) to satisfy the audit NFR.
- **SSO-only auth** — no local password store; the app never holds a credential.
- **Scheduled worker** — a periodic job drives the 3-day and 7-day nudges; no realtime infrastructure needed.

```
                       ┌──────────────────────────┐
                       │   Org Identity Provider  │
                       │      (OIDC / SAML)       │
                       └────────────┬─────────────┘
                                    │ SSO
┌──────────────┐   HTTPS   ┌────────▼─────────┐        ┌──────────────────┐
│   Browser    │◀────────▶ │   API Server     │◀──────▶│ Relational DB    │
│   SPA        │   REST    │  (monolith,      │  SQL   │ (requests, users,│
│  (queue,     │           │   RBAC-enforced) │        │  comments,       │
│  detail,     │           │                  │        │  activity, ...)  │
│  reports)    │           │                  │        └──────────────────┘
└──────────────┘           │                  │        ┌──────────────────┐
                           │                  │◀──────▶│ Object Storage   │
                           │                  │        │ (attachments)    │
                           │                  │        └──────────────────┘
                           │                  │        ┌──────────────────┐
                           │   ┌──────────┐   │        │ Email Service    │
                           │   │Scheduled │───┼───────▶│ (notifications)  │
                           │   │ Worker   │   │        └──────────────────┘
                           │   └──────────┘   │
                           └──────────────────┘
```

## Deployment Topology

- Single application deployment (API + served SPA assets) behind HTTPS.
- One primary relational database instance; managed backups.
- Object storage bucket for attachments.
- A scheduled worker (cron-style) co-deployed or as a separate task, running the nudge scans.
- All internal; access gated by SSO. No public endpoints beyond the SSO callback.
## 2. Component Architecture

### Frontend (SPA)

| Component | Responsibility |
|---|---|
| Auth shell | Redirects to IdP, holds session, gates routes by role |
| Submit form | Renders type-driven fields, validates, uploads attachments (F1) |
| Queue view | List with filters, search, saved views, bulk actions (F3) |
| Request detail | Header, body, comments (internal toggle), activity log (F4) |
| Reporting page | Renders throughput + cycle-time widgets, CSV export button (F7) |
| Admin console | Request types, custom-field schema editor, categories, roles (F8) |
| Notification tray | In-app notifications, mark-read, preferences (F6) |

### Backend (API monolith)

| Component | Responsibility |
|---|---|
| Auth/Session middleware | Validates SSO session, resolves effective role (F0) |
| Authorization guard | Enforces role + ownership on every route (F0) |
| Requests service | Create, read, list/filter/search, edit, status, assign, bulk (F1–F5) |
| Comments service | Add/list comments; enforce internal-comment visibility (F4) |
| Activity service | Append-only writes; activity log reads (F2) |
| Attachments service | Presigned upload/download to object storage; size/count limits (F1) |
| Notifications service | Fan-out to in-app + email; per-category mute (F6) |
| Scheduled worker | Scans for 3-day waiting and 7-day untouched nudges (F2/F6) |
| Reporting service | Aggregate queries; CSV export streaming (F7) |
| Admin service | Request types, schemas, categories, roles (F8) |

### Cross-cutting

- **RBAC** is enforced centrally in the authorization guard; services assume an authorized principal.
- **Requester scoping** is applied at the query layer so Requesters can never read others' requests or internal comments.
- **Short ID generation** uses a database sequence rendered as `REQ-<n>`.
## 3. Data Model

### ER Diagram (ASCII)

```
 users ──< requests >── request_types
   │           │  │
   │           │  └──< activity
   │           ├──< comments ──< attachments
   │           └──< attachments
   │
   ├──< saved_views
   └──< notifications >── requests
 categories (standalone, managed by admin)
```

- A `user` submits many `requests` (requester) and may be assigned many (assignee).
- A `request` belongs to one `request_type`, has many `comments`, `attachments`, and `activity` entries.
- An `attachment` belongs to a `request`, optionally to a `comment`.
- `notifications` reference a `user` and optionally a `request`.

### DDL

Authoritative DDL matches `project_specs/FRD/Y0-schema.md`. Target engine: PostgreSQL.

```sql
CREATE TYPE user_role AS ENUM ('requester', 'fulfiller', 'admin');
CREATE TYPE request_status AS ENUM (
  'new','triaged','in_progress','waiting_on_requester','blocked','done','cancelled');
CREATE TYPE request_priority AS ENUM ('low','normal','high','urgent');

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  role user_role NOT NULL DEFAULT 'requester',
  notification_prefs JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE request_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  custom_field_schema JSONB NOT NULL DEFAULT '[]',
  active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT max_five_fields CHECK (jsonb_array_length(custom_field_schema) <= 5)
);

CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  active BOOLEAN NOT NULL DEFAULT true
);

CREATE SEQUENCE request_short_id_seq START 1000;

CREATE TABLE requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  short_id TEXT NOT NULL UNIQUE,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  type_id UUID NOT NULL REFERENCES request_types(id),
  status request_status NOT NULL DEFAULT 'new',
  priority request_priority NOT NULL DEFAULT 'normal',
  requester_id UUID NOT NULL REFERENCES users(id),
  assignee_id UUID REFERENCES users(id),
  custom_field_values JSONB NOT NULL DEFAULT '{}',
  resolution_note TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  closed_at TIMESTAMPTZ
);
CREATE INDEX idx_requests_status ON requests(status);
CREATE INDEX idx_requests_assignee ON requests(assignee_id);
CREATE INDEX idx_requests_type ON requests(type_id);
CREATE INDEX idx_requests_priority_created ON requests(priority, created_at);

CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  author_id UUID NOT NULL REFERENCES users(id),
  body TEXT NOT NULL,
  internal BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_comments_request ON comments(request_id);

CREATE TABLE attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  comment_id UUID REFERENCES comments(id),
  filename TEXT NOT NULL,
  size_bytes BIGINT NOT NULL CHECK (size_bytes <= 26214400),
  url TEXT NOT NULL,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_attachments_request ON attachments(request_id);

CREATE TABLE activity (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  actor_id UUID NOT NULL REFERENCES users(id),
  field TEXT NOT NULL,
  old_value TEXT,
  new_value TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_activity_request ON activity(request_id, created_at);

CREATE TABLE saved_views (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES users(id),
  name TEXT NOT NULL,
  filters JSONB NOT NULL DEFAULT '{}'
);

CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  request_id UUID REFERENCES requests(id),
  category TEXT NOT NULL,
  body TEXT NOT NULL,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_notifications_user ON notifications(user_id, read_at);
```

**Append-only enforcement:** the application never issues `UPDATE`/`DELETE` on `activity`; optionally enforced with a DB trigger or restricted role grants. Retention: closed requests remain queryable for 2 years, then archived out of the hot table.
## 4. API Design

REST over HTTPS, JSON bodies. Every route requires an authenticated SSO session; authorization is role-scoped. Full endpoint catalog lives in `project_specs/FRD/Y1-api.md`; core TypeScript interfaces follow.

```typescript
type UserRole = 'requester' | 'fulfiller' | 'admin';

type RequestStatus =
  | 'new' | 'triaged' | 'in_progress'
  | 'waiting_on_requester' | 'blocked' | 'done' | 'cancelled';

type Priority = 'low' | 'normal' | 'high' | 'urgent';
type FieldType = 'text' | 'number' | 'select' | 'date';

interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  notificationPrefs: Record<string, boolean>;
}

interface CustomFieldDef {
  key: string;
  label: string;
  type: FieldType;
  required: boolean;
  options?: string[]; // for 'select'
}

interface RequestType {
  id: string;
  name: string;
  description?: string;
  customFieldSchema: CustomFieldDef[]; // max 5
  active: boolean;
}

interface RequestRecord {
  id: string;
  shortId: string;            // "REQ-1042"
  title: string;
  description: string;
  typeId: string;
  status: RequestStatus;
  priority: Priority;
  requesterId: string;
  assigneeId: string | null;
  customFieldValues: Record<string, unknown>;
  resolutionNote?: string;
  createdAt: string;
  updatedAt: string;
  closedAt?: string;
}

interface Comment {
  id: string;
  requestId: string;
  authorId: string;
  body: string;
  internal: boolean;
  createdAt: string;
}

interface ActivityEntry {
  id: string;
  requestId: string;
  actorId: string;
  field: string;        // 'status' | 'assignee' | custom field key
  oldValue: string | null;
  newValue: string | null;
  createdAt: string;
}

// Request bodies
interface CreateRequestBody {
  title: string;
  description: string;
  typeId: string;
  priority: Priority;
  customFieldValues?: Record<string, unknown>;
}

interface ChangeStatusBody {
  newStatus: RequestStatus;
  note?: string; // required for blocked/cancelled/done
}

interface AssignBody {
  assigneeId: string | null;
}

interface RequestListQuery {
  status?: RequestStatus;
  assignee?: string | 'unassigned';
  type?: string;
  priority?: Priority;
  dateFrom?: string;
  dateTo?: string;
  q?: string;
  savedView?: string;
  page?: number;
  pageSize?: number;
}
```

**Contract notes:**
- List responses are paginated and default-sorted by priority then age.
- `POST /requests/bulk` returns HTTP 207 with per-item results.
- Requester principals get server-side query scoping; internal comments are stripped from their responses.
## 5. Security Architecture

### Authentication
- SSO via the org's identity provider (OIDC or SAML). The app holds no passwords.
- On first login a `users` row is provisioned from the IdP assertion (email as unique key).
- Sessions are established server-side (secure, httpOnly cookie) with a sensible expiry; re-auth via the IdP on expiry.

### Authorization
- Three additive roles: `requester` ⊂ `fulfiller` ⊂ `admin`.
- A central authorization guard runs on every route:
  - **Requester:** may create requests; may read/edit only their own requests; sees only non-internal comments.
  - **Fulfiller:** all Requester rights plus read all requests, assign, change status, comment (incl. internal), close, view reports, export.
  - **Admin:** all Fulfiller rights plus manage request types, schemas, categories, and user roles.
- Requester scoping is enforced at the data-access layer, not just the UI, so ownership can't be bypassed by crafting requests.

### Data Protection
- **Internal comments** (`internal = true`) are filtered out of all Requester-facing responses server-side.
- **Attachments** are served via short-lived presigned URLs; direct bucket access is not exposed.
- **Audit:** the `activity` table is append-only; no UPDATE/DELETE path exists, satisfying the audit NFR even for cancelled requests.
- **Transport:** HTTPS only.
- **Retention:** closed requests remain queryable for 2 years, then archived; activity is retained per the audit policy.

### Accessibility (security-adjacent NFR)
- Submit form and queue view are keyboard-navigable and meet WCAG 2.1 AA; enforced in component tests.
## 6. Technology Stack

Concrete choices are recommendations consistent with the PRD constraints; any equivalent stack that meets the NFRs is acceptable. The binding constraints are: SSO-only auth, <1s queue at 500 open requests, append-only audit, WCAG 2.1 AA on submit + queue, and 2-year retention.

| Layer | Technology (recommended) | Purpose |
|---|---|---|
| Frontend | SPA framework (e.g., React + TypeScript) | Queue, detail, submit, reporting, admin UIs |
| Styling/A11y | Accessible component library | WCAG 2.1 AA on submit + queue |
| API | Node.js + TypeScript (e.g., Express/Fastify/NestJS) | REST API, RBAC, services |
| Database | PostgreSQL | Relational store, JSONB for custom fields, aggregates for reporting |
| ORM/Query | Type-safe query layer (e.g., Prisma/Drizzle) | Schema + query scoping |
| Object storage | S3-compatible bucket | Attachments (≤25 MB × 5) |
| Auth | Org IdP via OIDC/SAML library | SSO, session issuance |
| Email | Transactional email service | Notification + digest delivery |
| Scheduler | Cron/worker runner | 3-day & 7-day nudges |
| Reporting | SQL aggregates + CSV streaming | Reporting page + export |

**Rationale highlights:**
- **PostgreSQL** covers relational integrity, JSONB custom fields, and the percentile/aggregate queries reporting needs — one store, no analytics pipeline for this scale.
- **TypeScript end-to-end** shares the interface definitions in §4 between client and server.
- **JSONB custom fields** avoid schema migrations every time an admin adds a field.
## 7. Integration Points

Mirrors `project_specs/FRD/Y3-integrations.md`.

| Integration | Direction | Purpose | Contract / Notes |
|---|---|---|---|
| Org Identity Provider (OIDC/SAML) | Inbound auth | SSO login; no local passwords | Standard OIDC/SAML flow; provisions User on first login |
| Transactional email service | Outbound | Notification emails + digests | API-key auth; retried on failure (EMAIL_FAILED); bounces logged |
| Object storage (S3-compatible) | Bidirectional | Store/serve attachments | Presigned PUT/GET; ≤25 MB/file, ≤5/request; server validates size/count |
| Scheduled worker | Internal | Fire 3-day waiting & 7-day untouched nudges | Periodic scan over open requests; idempotent per (request, nudge-type, day) |

**Explicitly out of scope for v1 integrations** (PRD Non-goals / Out of Scope): Slack submit/status, email-to-request intake, external SLA/automation engines, billing/time-tracking. The architecture keeps notification fan-out behind a service boundary so a Slack channel could be added later without touching feature code.

## 8. Non-Functional Alignment

| NFR | How the architecture meets it |
|---|---|
| Queue <1s at 500 open | Indexed `(priority, created_at)`, `status`, `assignee`, `type`; server-side pagination |
| SSO-only auth | OIDC/SAML integration; no password store |
| Requester permissions | Data-layer query scoping + server-side internal-comment filtering |
| Append-only audit | `activity` table with no UPDATE/DELETE path (+ optional trigger) |
| Accessibility (AA) | Accessible component library; keyboard-nav tests on submit + queue |
| 2-year retention | Retention/archival job moves aged closed requests out of the hot table |
