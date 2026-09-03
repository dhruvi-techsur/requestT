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
