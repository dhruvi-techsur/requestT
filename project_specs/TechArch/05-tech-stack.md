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
