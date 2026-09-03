# UX Mockup: RequestT

**Project:** RequestT
**Generated:** 2026-09-03
**Based on:** UserStories-RequestT.md, PRD-RequestT.md, FRD-RequestT.md

---

## Overview

RequestT is a desktop-first internal web app, usable on a phone. The design serves two very different mental models from one shell: a **Requester** who wants a fast form and a status they can check, and a **Fulfiller** who lives in a triage queue. The UX principles:

- **Fast, low-friction submission.** The submit form is short, type-driven, and gives an immediate ID.
- **The queue is the workspace.** Filters, saved views, and bulk actions make five-minute triage possible.
- **Status is always visible.** Every request surfaces its status, assignee, and history without a click chain.
- **Requesters never see internal content.** Internal comments and admin surfaces are role-gated in the UI and the API.
- **Accessible by default.** Submit form and queue are keyboard-navigable and meet WCAG 2.1 AA.

---

## Navigation Map

| Screen | Route | Reached from | Nav element |
|---|---|---|---|
| Sign-in redirect | `/login` | App shell (unauthenticated) | Automatic redirect to IdP |
| Queue | `/requests` | App shell | Sidebar: "Queue" (default landing for fulfillers) |
| Submit request | `/requests/new` | App shell / Queue | Header button: "New request" |
| Request detail | `/requests/:shortId` | Queue (row click) / My requests / notification link | Row click / notification deep-link |
| My requests | `/my-requests` | App shell | Sidebar: "My requests" (default landing for requesters) |
| Reporting | `/reports` | App shell (fulfiller/admin) | Sidebar: "Reports" |
| Admin console | `/admin` | App shell (admin) | Sidebar: "Admin" |
| Notifications tray | (overlay) | App shell | Header bell icon |

**No orphan screens:** every screen above is reachable from the app shell sidebar/header or a reachable parent (Queue → Request detail). Detail is also reachable via notification deep-links, whose parent (the notification tray) traces to the shell.
