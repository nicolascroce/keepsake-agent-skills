# Keepsake API Reference

## Authentication

All requests require a Bearer token:

```
Authorization: Bearer ksk_YOUR_API_KEY
```

**Base URL**: `https://app.keepsake.place/api/v1`

**Rate limit**: 60 requests per minute per API key.

## Contacts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contacts` | List all contacts. Query: `?sort=firstName\|lastName\|createdAt&order=asc\|desc` |
| GET | `/contacts/search?q=` | Search contacts (accent-insensitive) |
| GET | `/contacts/:id` | Get a single contact |
| POST | `/contacts` | Create a contact |
| PATCH | `/contacts/:id` | Update a contact |
| DELETE | `/contacts/:id` | Delete a contact |
| GET | `/contacts/:id/timeline` | Get full interaction timeline |

**Create body**: `{ firstName, lastName?, email?, phone?, address?, birthday?, notes?, tag_ids? }`

## Companies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/companies` | List all companies |
| GET | `/companies/search?q=` | Search companies (accent-insensitive) |
| GET | `/companies/:id` | Get company with linked contacts |
| POST | `/companies` | Create a company |
| PATCH | `/companies/:id` | Update a company |
| DELETE | `/companies/:id` | Delete a company |
| POST | `/companies/:id/contacts` | Link a contact (body: `{ contact_id, role? }`) |
| DELETE | `/companies/:id/contacts/:contactId` | Unlink a contact |

**Create body**: `{ name, website?, industry?, notes?, tag_ids? }`

## Entries

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/entries` | List entries. Query: `?from=YYYY-MM-DD&to=YYYY-MM-DD` |
| GET | `/entries/:id` | Get a single entry |
| POST | `/entries` | Create an entry |
| PATCH | `/entries/:id` | Update an entry |
| DELETE | `/entries/:id` | Delete an entry |
| POST | `/entries/:id/contacts/:contactId` | Link a contact |
| DELETE | `/entries/:id/contacts/:contactId` | Unlink a contact |
| POST | `/entries/:id/tags/:tagId` | Link a tag |
| DELETE | `/entries/:id/tags/:tagId` | Unlink a tag |

**Create body**: `{ type, date, content?, contact_ids?, tag_ids? }`

**Entry types**: call, email, meeting, event, gift, letter, message, other

## Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks` | List all tasks |
| GET | `/tasks/today` | Get today's tasks |
| GET | `/tasks/overdue` | Get overdue tasks |
| GET | `/tasks/:id` | Get a single task |
| POST | `/tasks` | Create a task |
| PATCH | `/tasks/:id` | Update a task |
| DELETE | `/tasks/:id` | Delete a task |
| POST | `/tasks/:id/complete` | Complete a task (auto-creates next for recurring) |
| POST | `/tasks/:id/snooze` | Snooze a task (body: `{ until: "YYYY-MM-DD" }`) |
| POST | `/tasks/:id/contacts/:contactId` | Link a contact |
| DELETE | `/tasks/:id/contacts/:contactId` | Unlink a contact |
| POST | `/tasks/:id/tags/:tagId` | Link a tag |
| DELETE | `/tasks/:id/tags/:tagId` | Unlink a tag |

**Create body**: `{ title, description?, date?, date_type?, priority?, recurrence?, primary_contact_id?, contact_ids?, tag_ids?, section_id? }`

**Recurrence**: daily, weekdays, weekly, biweekly, monthly, quarterly, yearly

## Notes (QuickNotes)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notes` | List notes. Query: `?pinned=true&archived=false` |
| GET | `/notes/:id` | Get a single note |
| POST | `/notes` | Create a note |
| PATCH | `/notes/:id` | Update a note |
| DELETE | `/notes/:id` | Delete a note |
| POST | `/notes/:id/pin` | Pin a note |
| POST | `/notes/:id/unpin` | Unpin a note |
| POST | `/notes/:id/archive` | Archive (stays searchable) |
| POST | `/notes/:id/unarchive` | Unarchive |
| POST | `/notes/:id/contacts/:contactId` | Link a contact |
| DELETE | `/notes/:id/contacts/:contactId` | Unlink a contact |
| POST | `/notes/:id/tags/:tagId` | Link a tag |
| DELETE | `/notes/:id/tags/:tagId` | Unlink a tag |

**Create body**: `{ content, pinned?, contact_ids?, tag_ids? }`

## Days

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/days` | List all day entries |
| GET | `/days/:date` | Get a day (date = YYYY-MM-DD) |
| PATCH | `/days/:date` | Update a day (body: `{ content }`) |

## Tags

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tags` | List tags (`?q=` name search, `?limit=&offset=` pagination; ordering arrays omitted — use `?include_orders=true` or GET `/tags/:id`) |
| GET | `/tags/:id` | Get a single tag (includes `tasks_order`, where `"h:<header_id>"` entries mark section separators) |
| GET | `/tags/:id/items` | Get items linked to a tag (`?types=tasks,notes`, `?status=pending\|completed` for tasks, `?summary=true` for lightweight items; response includes task `sections` — `header_id: null` = tasks outside any section) |
| POST | `/tags` | Create a tag |
| PATCH | `/tags/:id` | Update a tag |
| DELETE | `/tags/:id` | Delete a tag |

**Create body**: `{ name, emoji? }`

**Tip**: on large tags, prefer `GET /tags/:id/items?types=tasks&status=pending&summary=true` — the default (no params) returns every linked item in full and can be a very large response.

## Task Headers (Sections)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/task-headers` | List all sections |
| POST | `/task-headers` | Create a section (body: `{ name, position? }`) |
| PATCH | `/task-headers/:id` | Update a section |
| DELETE | `/task-headers/:id` | Delete a section |

## Search

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/search?q=` | Global search across all entity types |

Returns matching contacts, entries, tasks, notes, and tags with highlighted excerpts.

## Changelog

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/changelog?since=ISO_TIMESTAMP` | Changes since the given timestamp |

Returns all created, updated, and deleted entities since the given time. Save the returned `server_time` for the next call.

## Agent Instructions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/agent/instructions` | Get agent best practices and instructions |

Call this at the start of each session to refresh your operational instructions.

## Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad request — missing or invalid parameters |
| 401 | Unauthorized — invalid or missing API key |
| 403 | Forbidden — insufficient permissions |
| 404 | Not found — resource does not exist |
| 429 | Rate limited — 60 requests/minute exceeded |
| 500 | Server error — try again later |
