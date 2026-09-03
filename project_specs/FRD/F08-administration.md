## F08: Administration

**Description:** Admins configure request types and their custom-field schemas, manage categories, and manage user roles.

**Terminology:**
- **Custom-field schema:** The definition of a RequestType's up-to-5 custom fields (name, type, required, options).
- **Active flag:** Whether a RequestType is offered on the submit form.

**Sub-features:**
- Create/edit/deactivate request types
- Define custom-field schema per type (text, number, select, date; max 5)
- Manage categories
- Manage user roles (Requester/Fulfiller/Admin)

**Process:**
1. Admin opens administration.
2. Admin creates or edits a request type, defining its custom-field schema.
3. System validates the schema (≤5 fields, valid types, select options present).
4. Admin sets a type active/inactive; only active types appear on the submit form.
5. Admin assigns or changes a user's role; change takes effect on next authorization check.

**Inputs:**
- RequestType: `name`, `description`, `custom_field_schema`, `active`
- Role change: `user_id`, `role`

**Outputs:**
- Persisted RequestType / category / role
- Activity/audit entry for role and type changes

**Validation:**
- Admin-only.
- Custom-field schema: ≤5 fields; each has name + valid type; select fields have ≥1 option.
- Role must be Requester/Fulfiller/Admin.
- Deactivating a type does not affect existing requests of that type.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Non-admin access | 403 | FORBIDDEN | "Admin access required" |
| >5 custom fields | 422 | SCHEMA_ERROR | "A type can have at most 5 custom fields" |
| Invalid field type | 422 | SCHEMA_ERROR | "Unsupported field type" |
| Select without options | 422 | SCHEMA_ERROR | "Select fields need at least one option" |

**API Surface (this feature):** see `Y1-api.md` §Admin.

**Schema Surface (this feature):** uses `request_types`, `users`, `categories` — see `Y0-schema.md`.
