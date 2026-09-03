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
