## 4. API Design

REST over HTTPS, JSON bodies. Every route requires an authenticated SSO session; authorization is role-scoped. Full endpoint catalog lives in `project_specs/FRD/Y1-api.md`; core TypeScript interfaces follow.

```typescript
type UserRole = 'requester' | 'fulfiller' | 'admin';

type RequestStatus =
  | 'new' | 'triaged' | 'in_progress'
  | 'waiting_on_requester' | 'blocked' | 'done' | 'cancelled';

type Priority = 'low' | 'normal' | 'high' | 'urgent';
type FieldType = 'text' | 'number' | 'select' | 'date';

interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  notificationPrefs: Record<string, boolean>;
}

interface CustomFieldDef {
  key: string;
  label: string;
  type: FieldType;
  required: boolean;
  options?: string[]; // for 'select'
}

interface RequestType {
  id: string;
  name: string;
  description?: string;
  customFieldSchema: CustomFieldDef[]; // max 5
  active: boolean;
}

interface RequestRecord {
  id: string;
  shortId: string;            // "REQ-1042"
  title: string;
  description: string;
  typeId: string;
  status: RequestStatus;
  priority: Priority;
  requesterId: string;
  assigneeId: string | null;
  customFieldValues: Record<string, unknown>;
  resolutionNote?: string;
  createdAt: string;
  updatedAt: string;
  closedAt?: string;
}

interface Comment {
  id: string;
  requestId: string;
  authorId: string;
  body: string;
  internal: boolean;
  createdAt: string;
}

interface ActivityEntry {
  id: string;
  requestId: string;
  actorId: string;
  field: string;        // 'status' | 'assignee' | custom field key
  oldValue: string | null;
  newValue: string | null;
  createdAt: string;
}

// Request bodies
interface CreateRequestBody {
  title: string;
  description: string;
  typeId: string;
  priority: Priority;
  customFieldValues?: Record<string, unknown>;
}

interface ChangeStatusBody {
  newStatus: RequestStatus;
  note?: string; // required for blocked/cancelled/done
}

interface AssignBody {
  assigneeId: string | null;
}

interface RequestListQuery {
  status?: RequestStatus;
  assignee?: string | 'unassigned';
  type?: string;
  priority?: Priority;
  dateFrom?: string;
  dateTo?: string;
  q?: string;
  savedView?: string;
  page?: number;
  pageSize?: number;
}
```

**Contract notes:**
- List responses are paginated and default-sorted by priority then age.
- `POST /requests/bulk` returns HTTP 207 with per-item results.
- Requester principals get server-side query scoping; internal comments are stripped from their responses.
