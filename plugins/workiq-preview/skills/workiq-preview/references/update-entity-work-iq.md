# update_entity_work_iq

Update an existing WorkIQ entity via HTTP PATCH. Only the fields you include in the body are changed — other fields are left untouched.

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `entityUrl` | string | Yes | The entity path including the item's ID (e.g., `/me/events/{id}`). Get the ID from a prior `fetch_work_iq` or `create_entity_work_iq` call. Must be relative to the domain root — start with `/`, do not include a scheme or authority (`https://graph.microsoft.com` ❌, `/me/events/{id}` ✅). URL-encode any special characters in path segments. |
| `jsonBody` | string | Yes | JSON-encoded string containing only the fields to update. Omit fields you don't want to change. |

## When to Use

- Marking an email as read/unread
- Updating the subject, time, or location of a calendar event
- Changing the status or due date of a task
- Updating a document's metadata
- Any partial update to an existing M365 entity

## Gotchas

- **`entityUrl` must address exactly one entity by ID.** A collection or query URL
  (`/me/planner/tasks?$filter=startswith(title,'...')`) is rejected with
  "Write requests are only supported on contained entities" — resolve the ID with
  `fetch_work_iq` first, then PATCH `/.../{id}`.
- The ID must come from a real tool response for the **same entity type** — a directory user ID
  does not work on `/me/contacts/{id}`, and an ID scraped from a search-result URL is not an
  entity ID.
- Updating one entity means one PATCH. If it fails, fix the request and retry once or twice —
  do not loop the same PATCH or fan it out across other entities.
- **Planner writes need an `If-Match` etag** — fetch the task first; on a 412/precondition
  error, re-fetch and retry (see `references/tasks-work-iq.md`).

## Workflow

1. Obtain the entity's `id` from `fetch_work_iq` or `create_entity_work_iq`
2. Optionally call `get_schema_work_iq` with `httpMethod: "patch"` to confirm which fields are updatable
3. Call `update_entity_work_iq` with only the fields you want to change

## Examples

### Mark a message as read
```json
{
  "entityUrl": "/me/messages/{id}",
  "jsonBody": "{\"isRead\":true}"
}
```

### Update a calendar event's subject and location
```json
{
  "entityUrl": "/me/events/{id}",
  "jsonBody": "{\"subject\":\"Updated: Team Sync\",\"location\":{\"displayName\":\"Conference Room B\"}}"
}
```

### Update a To Do task's due date
```json
{
  "entityUrl": "/me/todo/lists/{listId}/tasks/{taskId}",
  "jsonBody": "{\"dueDateTime\":{\"dateTime\":\"2024-06-10T17:00:00\",\"timeZone\":\"UTC\"}}"
}
```

### Mark a task as complete
```json
{
  "entityUrl": "/me/todo/lists/{listId}/tasks/{taskId}",
  "jsonBody": "{\"status\":\"completed\"}"
}
```

### Move a message to a different category
```json
{
  "entityUrl": "/me/messages/{id}",
  "jsonBody": "{\"categories\":[\"Project Alpha\"]}"
}
```
