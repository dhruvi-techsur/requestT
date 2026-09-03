## Y2: Error Catalog

Cross-feature error scenarios with HTTP status and retry guidance.

| Error Code | HTTP | Meaning | Retry? |
|---|---|---|---|
| AUTH_REQUIRED | 401 | No valid session | Re-authenticate |
| IDP_UNAVAILABLE | 503 | Identity provider down | Retry with backoff |
| FORBIDDEN | 403 | Role/ownership check failed | No |
| NOT_FOUND | 404 | Resource does not exist | No |
| VALIDATION_ERROR | 422 | Generic input validation failure | No — fix input |
| INVALID_TYPE | 422 | Request type inactive/unknown | No |
| CUSTOM_FIELD_ERROR | 422 | Custom field values fail type schema | No |
| FILE_TOO_LARGE | 413 | Attachment > 25 MB | No |
| TOO_MANY_FILES | 422 | > 5 attachments | No |
| NOTE_REQUIRED | 422 | Status transition needs a note/reason | No |
| INVALID_STATUS | 422 | Unknown status value | No |
| INVALID_ASSIGNEE | 422 | Assignee not a fulfiller | No |
| BULK_PARTIAL | 207 | Some bulk items failed | Per-item |
| SCHEMA_ERROR | 422 | Invalid custom-field schema | No |
| EXPORT_TOO_LARGE | 413 | Export result set too large | Narrow filter |
| EMAIL_FAILED | 502 | Notification email delivery failed | Auto-retried |
| RATE_LIMITED | 429 | Too many requests | Retry after delay |
