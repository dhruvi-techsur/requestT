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
