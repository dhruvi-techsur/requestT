## Responsive Considerations

### Desktop (>1024px)
- Two-column request detail (body left, activity right).
- Queue shows full column set (ID, title, status, priority, assignee, type, age).
- Persistent left sidebar navigation.

### Tablet (768px–1024px)
- Request detail collapses to a single column with a tabbed Body / Activity switch.
- Queue drops the Type column into a row-expand affordance; filters move into a "Filters" popover.
- Sidebar collapses to icons.

### Mobile (<768px)
- Submit form is the priority experience: single-column, large tap targets, sticky Submit.
- Queue becomes a card list (ID + title + status + priority badges); filters in a bottom sheet.
- Bulk actions hidden on mobile (triage is a desktop task); single-row actions remain.
- Request detail is fully stacked; comment composer sticky at the bottom.
