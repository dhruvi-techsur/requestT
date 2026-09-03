## User Flows

### Flow 0: Submit a Request

**Trigger:** Requester clicks "New request" from the header.
**User Story:** US-1.1, US-1.2

```
[New request button]
    │
    ▼
[Select request type] ── renders type's custom fields
    │
    ▼
[Fill title, description, priority, custom fields; attach files]
    │
    ├── Valid ──▶ [Submit] ──▶ [Confirmation: REQ-#### created] ──▶ [Request detail]
    │
    └── Invalid ──▶ [Inline validation errors] ──▶ (stay on form)
```

**Steps:**
1. Requester picks a type from a dropdown; the form re-renders that type's up-to-5 custom fields.
2. Requester fills required title + description, suggests a priority, completes custom fields.
3. Optional: drag-drop or browse up to 5 files (≤25 MB each); oversized/extra files rejected inline.
4. On Submit, a success toast shows the `REQ-####` ID and the app routes to the new request's detail.
