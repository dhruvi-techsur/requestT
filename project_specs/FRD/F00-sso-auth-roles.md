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
