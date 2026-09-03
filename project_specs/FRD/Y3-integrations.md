## Y3: Integration Points

| Integration | Direction | Purpose | Notes |
|---|---|---|---|
| Org Identity Provider (OIDC/SAML SSO) | Inbound auth | Authenticate users, no local passwords | Required for F0; provisions User on first login |
| Email delivery service (SMTP/API) | Outbound | Send notification emails and digests | F6; retried on failure (EMAIL_FAILED) |
| Object storage | Outbound/Inbound | Store and serve request/comment attachments | F1/F4; ≤25 MB per file, ≤5 per request |
| Scheduled job runner | Internal | Fire 3-day waiting and 7-day untouched nudges | F2/F6; runs periodically over open requests |

**Explicitly not integrated in v1** (see PRD Non-goals / Out of Scope): Slack, email-to-request intake, external SLA/automation engines, billing/time-tracking systems.
