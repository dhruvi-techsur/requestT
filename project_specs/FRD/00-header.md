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
