## Y0: Database Schema (DDL)

Illustrative relational DDL (PostgreSQL dialect). Concrete engine and types finalized in TechArch.

```sql
CREATE TYPE user_role AS ENUM ('requester', 'fulfiller', 'admin');
CREATE TYPE request_status AS ENUM (
  'new', 'triaged', 'in_progress', 'waiting_on_requester',
  'blocked', 'done', 'cancelled'
);
CREATE TYPE request_priority AS ENUM ('low', 'normal', 'high', 'urgent');
CREATE TYPE field_type AS ENUM ('text', 'number', 'select', 'date');

CREATE TABLE users (
  id                 UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name               TEXT NOT NULL,
  email              TEXT NOT NULL UNIQUE,
  role               user_role NOT NULL DEFAULT 'requester',
  notification_prefs JSONB NOT NULL DEFAULT '{}',
  created_at         TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE categories (
  id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name   TEXT NOT NULL,
  active BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE request_types (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                TEXT NOT NULL,
  description         TEXT,
  custom_field_schema JSONB NOT NULL DEFAULT '[]',  -- <=5 field defs
  active              BOOLEAN NOT NULL DEFAULT true,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT max_five_fields CHECK (jsonb_array_length(custom_field_schema) <= 5)
);

CREATE SEQUENCE request_short_id_seq START 1000;

CREATE TABLE requests (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  short_id            TEXT NOT NULL UNIQUE,            -- e.g. REQ-1042
  title               TEXT NOT NULL,
  description         TEXT NOT NULL,
  type_id             UUID NOT NULL REFERENCES request_types(id),
  status              request_status NOT NULL DEFAULT 'new',
  priority            request_priority NOT NULL DEFAULT 'normal',
  requester_id        UUID NOT NULL REFERENCES users(id),
  assignee_id         UUID REFERENCES users(id),
  custom_field_values JSONB NOT NULL DEFAULT '{}',
  resolution_note     TEXT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  closed_at           TIMESTAMPTZ
);
CREATE INDEX idx_requests_status   ON requests(status);
CREATE INDEX idx_requests_assignee ON requests(assignee_id);
CREATE INDEX idx_requests_type     ON requests(type_id);
CREATE INDEX idx_requests_priority_created ON requests(priority, created_at);

CREATE TABLE comments (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  author_id  UUID NOT NULL REFERENCES users(id),
  body       TEXT NOT NULL,
  internal   BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_comments_request ON comments(request_id);

CREATE TABLE attachments (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id  UUID NOT NULL REFERENCES requests(id),
  comment_id  UUID REFERENCES comments(id),
  filename    TEXT NOT NULL,
  size_bytes  BIGINT NOT NULL CHECK (size_bytes <= 26214400), -- 25 MB
  url         TEXT NOT NULL,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Append-only; no UPDATE/DELETE by policy.
CREATE TABLE activity (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL REFERENCES requests(id),
  actor_id   UUID NOT NULL REFERENCES users(id),
  field      TEXT NOT NULL,          -- 'status' | 'assignee' | field name
  old_value  TEXT,
  new_value  TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_activity_request ON activity(request_id, created_at);

CREATE TABLE saved_views (
  id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES users(id),  -- null = system view
  name    TEXT NOT NULL,
  filters JSONB NOT NULL DEFAULT '{}'
);

CREATE TABLE notifications (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES users(id),
  request_id UUID REFERENCES requests(id),
  category   TEXT NOT NULL,
  body       TEXT NOT NULL,
  read_at    TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_notifications_user ON notifications(user_id, read_at);
```

**Retention note:** closed requests remain queryable for 2 years, then archive. Activity is append-only and exempt from deletion.
