## Y1: API Endpoints

All endpoints require an authenticated SSO session. Authorization is role-scoped per F00. JSON over HTTPS.

### §Auth
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/auth/login` | Redirect to IdP | any |
| GET | `/auth/callback` | IdP callback, establish session | any |
| POST | `/auth/logout` | End session | any |
| GET | `/me` | Current user + effective role | any |

### §Requests
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests` | List/filter/search (paginated, sorted priority→age) | any (scoped) |
| POST | `/requests` | Create a request | any |
| GET | `/requests/{id}` | Request detail | owner or fulfiller |
| PATCH | `/requests/{id}` | Edit fields / add detail | owner (own) or fulfiller |
| POST | `/requests/{id}/status` | Change status (note-gated) | fulfiller |
| POST | `/requests/{id}/assign` | Assign / reassign / unassign | fulfiller |
| POST | `/requests/bulk` | Bulk assign / status change | fulfiller |

`GET /requests` query params: `status`, `assignee`, `type`, `priority`, `date_from`, `date_to`, `q`, `saved_view`, `page`, `page_size`.

### §Comments
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests/{id}/comments` | List comments (internal filtered for requesters) | owner or fulfiller |
| POST | `/requests/{id}/comments` | Add comment (`internal` fulfiller-only) | owner or fulfiller |

### §Activity
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/requests/{id}/activity` | Append-only activity log | owner or fulfiller |

### §SavedViews
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/saved-views` | List system + own views | any |
| POST | `/saved-views` | Create a personal saved view | any |

### §Notifications
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/notifications` | In-app notifications | any |
| POST | `/notifications/{id}/read` | Mark read | any |
| PATCH | `/me/notification-prefs` | Update per-category mutes | any |

### §Reports
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET | `/reports` | Throughput + cycle-time aggregates | fulfiller |
| GET | `/requests/export` | CSV export of a filtered view | fulfiller |

### §Admin
| Method | Path | Purpose | Roles |
|---|---|---|---|
| GET/POST | `/admin/request-types` | List/create request types | admin |
| PATCH | `/admin/request-types/{id}` | Edit schema / active flag | admin |
| GET/POST | `/admin/categories` | Manage categories | admin |
| PATCH | `/admin/users/{id}/role` | Change a user's role | admin |
