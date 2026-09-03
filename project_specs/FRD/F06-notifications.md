## F06: Notifications

**Description:** Email plus in-app notifications for the events that matter, with per-category muting per user. Two time-triggered nudges (3-day waiting, 7-day untouched) run on a schedule.

**Terminology:**
- **Notification category:** The event class a user can mute (submitted, assigned, status, comment, nudges).
- **Digest:** Optional batched delivery of "request submitted" events for fulfillers.

**Sub-features:**
- Event notifications (email + in-app)
- Per-category mute preferences
- Digest option for "request submitted"
- Scheduled nudges (3-day waiting-on-requester, 7-day untouched)

**Event table:**
| Event | Notifies |
|---|---|
| Request submitted | All fulfillers (digest option) |
| Assigned to you | Assignee |
| Status changed | Requester |
| New comment | Other participants (internal comments to fulfillers only) |
| Waiting on requester for 3 days | Requester |
| Open and untouched for 7 days | Assignee, or team lead if unassigned |

**Process:**
1. A feature event (submit, assign, status change, comment) fires a notification request.
2. System resolves recipients and filters out those who muted that category.
3. System delivers in-app + email (or queues into a digest).
4. A scheduled job scans requests: Waiting-on-requester ≥3 days → nudge requester; open + untouched ≥7 days → nudge assignee (or team lead if unassigned).

**Inputs:**
- Event payload (type, request_id, actor, participants)
- User `notification_prefs`

**Outputs:**
- In-app notification records
- Email deliveries (or digest queue entries)

**Validation:**
- Never deliver internal-comment notifications to Requesters.
- Respect per-category mute settings.

**Error States:**
| Scenario | HTTP Status | Error Code | Message |
|---|---|---|---|
| Email delivery failure | 502 | EMAIL_FAILED | "Notification email could not be sent" (retried) |
| Invalid prefs update | 422 | VALIDATION_ERROR | "Invalid notification preferences" |

**API Surface (this feature):** see `Y1-api.md` §Notifications.

**Schema Surface (this feature):** uses `notifications`, `users.notification_prefs` — see `Y0-schema.md`.
