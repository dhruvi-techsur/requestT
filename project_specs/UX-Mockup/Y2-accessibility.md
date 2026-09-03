## Accessibility Notes

Target: **WCAG 2.1 AA**, with the submit form and queue view as the enforced-priority surfaces (per PRD NFR).

- **Color contrast:** All text and status badges meet ≥4.5:1 (normal) / ≥3:1 (large). Status is never conveyed by color alone — each badge pairs a label with its color (e.g., "Blocked" text + icon).
- **Keyboard navigation:** Full keyboard operability on submit form and queue — logical tab order, visible focus rings, Enter/Space activation. Bulk-select via keyboard; saved-view chips are focusable buttons. No keyboard traps in modals (status-note prompt, field editor).
- **Screen reader:** Queue is a proper table/grid with header associations; each row announces ID, title, status, priority, assignee. The activity log is an ordered list with time stamps read in context. Toasts use an ARIA live region (polite).
- **ARIA labels:** Icon-only controls (bell/notifications, ✕ close, bulk actions) carry `aria-label`. The type-driven form associates each custom field with its `<label>`; validation errors use `aria-describedby` and `aria-invalid`.
- **Forms:** Required fields marked programmatically (not just visually); inline errors are announced and focus moves to the first error on failed submit.
- **Motion/timing:** No essential info conveyed only via animation; the 3-day/7-day nudges are server-driven and never require the user to act within a client-side timer.
