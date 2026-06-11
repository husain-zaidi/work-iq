# Tasks & To-Do (Microsoft To Do + Planner)

Use the WorkIQ **entity tools** for task/to-do/follow-up requests whose data lives in
Microsoft 365 — **not** the CLI's local todo files, the session SQL `todos` table, or any
other on-disk task tracker. If the user says "add a task", "remind me to…", "follow up
with…", "mark … done", or "list my tasks", that is M365 data: route it to WorkIQ.

> **⚠️ Do not fall back to local/builtin task storage.** Creating a markdown todo file or
> inserting into a local database does **not** satisfy an M365 task request and is not
> recoverable by the user in Outlook/Planner. If a WorkIQ task call fails, report the
> failure — do not silently substitute a local todo.

## Which surface? To Do vs Planner

| Situation | Surface | Path root |
|-----------|---------|-----------|
| Personal task, "my tasks", reminders, "follow up", no team/plan named | **Microsoft To Do** (default) | `/me/todo/...` |
| Task in a shared team plan, "the X plan", assigned to others, buckets/boards | **Planner** | `/planner/...` |

When the request is ambiguous ("add a task to follow up with finance"), **default to
Microsoft To Do** (`/me/todo/lists/{listId}/tasks`) — it is the user's personal task store.
Do not run repeated `search_paths` to "decide"; the canonical paths below are stable.

## Microsoft To Do — canonical paths

| Operation | Tool | Path |
|-----------|------|------|
| List task lists | `fetch_work_iq` | `/me/todo/lists` |
| Create a task list | `create_entity_work_iq` | parentUrl `/me/todo/lists` |
| Rename / delete a list | `update_entity_work_iq` / `delete_entity_work_iq` | `/me/todo/lists/{listId}` |
| List tasks in a list | `fetch_work_iq` | `/me/todo/lists/{listId}/tasks` |
| Create a task | `create_entity_work_iq` | parentUrl `/me/todo/lists/{listId}/tasks` |
| Update / rename / set due / complete a task | `update_entity_work_iq` | `/me/todo/lists/{listId}/tasks/{taskId}` |
| Delete a task | `delete_entity_work_iq` | `/me/todo/lists/{listId}/tasks/{taskId}` |

To Do task body fields: `title`, `body` (`{ "content": "...", "contentType": "text" }`),
`dueDateTime` (`{ "dateTime": "2026-06-12T17:00:00", "timeZone": "UTC" }`),
`status` (`notStarted` | `inProgress` | `completed`), `importance`, `reminderDateTime`.

- **Mark a task done:** `update_entity_work_iq` on the task with `{"status":"completed"}`.
- The default list has a well-known id of `Tasks`; you can also resolve a named list via
  `fetch_work_iq` on `/me/todo/lists` and match `displayName`.

## Planner — canonical paths

| Operation | Tool | Path |
|-----------|------|------|
| List my plans | `fetch_work_iq` | `/me/planner/plans` |
| List tasks in a plan | `fetch_work_iq` | `/planner/plans/{planId}/tasks` |
| Create a task | `create_entity_work_iq` | parentUrl `/planner/tasks` (body includes `planId`) |
| Update / complete a task | `update_entity_work_iq` | `/planner/tasks/{taskId}` |
| Delete a task | `delete_entity_work_iq` | `/planner/tasks/{taskId}` |

Planner task body fields: `planId`, `title`, `bucketId`, `assignments`, `dueDateTime`,
`percentComplete` (`0` = not started, `50` = in progress, `100` = complete).

- **Mark a Planner task done:** `update_entity_work_iq` with `{"percentComplete":100}`.
- **Planner gotcha:** `update_entity_work_iq` / `delete_entity_work_iq` on Planner resources
  require the current `@odata.etag` (an `If-Match` precondition). Fetch the task first to
  read its etag; if a Planner write returns a `412`/precondition error, re-fetch and retry.

## Resolve-then-act (do not loop)

1. Resolve the target with **one** `fetch_work_iq` (To Do list/task) — match by `displayName`/`title`.
2. If the first fetch does not find it, try **one** `ask_work_iq` to locate it semantically.
3. If still not found, **stop and report "not found"** — do not fire 10+ more `fetch`/`search_paths`/`ask` calls.
4. Once you have the id, call the mutation (`create_entity` / `update_entity` / `delete_entity`).

## Examples

### Create a To Do task (default surface)
```json
{ "parentUrl": "/me/todo/lists/Tasks/tasks",
  "jsonBody": "{\"title\":\"Follow up with finance\"}" }
```

### Mark a To Do task complete
```json
{ "entityUrl": "/me/todo/lists/{listId}/tasks/{taskId}",
  "jsonBody": "{\"status\":\"completed\"}" }
```

### Create a To Do list
```json
{ "parentUrl": "/me/todo/lists",
  "jsonBody": "{\"displayName\":\"Eval prep\"}" }
```

### Mark a Planner task complete
```json
{ "entityUrl": "/planner/tasks/{taskId}",
  "jsonBody": "{\"percentComplete\":100}" }
```
