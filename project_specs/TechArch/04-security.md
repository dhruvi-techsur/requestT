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
