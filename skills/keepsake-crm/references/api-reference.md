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
| GET | `/contacts` | List contacts. Query: `sort` (`first_name`, `last_name`, `created_at`, `updated_at`…), `order=asc\|desc`, `fields` (comma-separated: columns + `companies`, `last_interaction_date`), `company` (company id or part of its name), `has_company=true\|false`, `updated_since` (ISO), `include_last_interaction=true` |
| GET | `/contacts/search?q=` | Search contacts (accent-insensitive: names, email, notes, phone, linked company names) |
| GET | `/contacts/:id` | Get a single contact (with recent entries, tags, companies) |
| POST | `/contacts` | Create a contact |
| PATCH | `/contacts/:id` | Update a contact |
| DELETE | `/contacts/:id` | Delete a contact (`?permanent=true` for hard delete) |
| GET | `/contacts/:id/timeline` | Get full interaction timeline |

**Create body**: `{ first_name, last_name?, email?, phone?, job_title?, address?, birthday? (YYYY-MM-DD), notes?, company? }`

`company` is a shortcut, not a stored field: it links the contact to that company record (found by name ignoring case and accents, created if missing). It adds a link and never removes one. Contacts are returned with `companies: [{ id, name, role }]`.

## Companies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/companies` | List all companies |
| GET | `/companies/search?q=` | Search companies (accent-insensitive) |
| GET | `/companies/:id` | Get company with linked contacts (and their role) and tags |
| POST | `/companies` | Create a company |
| PATCH | `/companies/:id` | Update a company. Renaming onto another company's name returns 409 (use merge) |
| DELETE | `/companies/:id` | Delete a company (`?permanent=true` for hard delete) |
| POST | `/companies/:id/contacts` | Link a contact (body: `{ contact_id, role? }`), idempotent |
| DELETE | `/companies/:id/contacts?contact_id=` | Unlink a contact |
| POST | `/companies/:id/merge` | Merge this company into `{ target_id }`: contacts, entries and tags move over, empty details are filled, notes appended, then this company is deleted |

**Create body**: `{ name, website?, email?, phone?, address?, notes? }`

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
| GET | `/notes` | List notes. Query: `?pinned=true&archived=false`, `?date=YYYY-MM-DD` or `?date_from=…&date_to=…` (notes linked to a day / range). Each note carries `dates` |
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
| POST | `/notes/:id/dates/:date` | Attach the note to a day (YYYY-MM-DD, idempotent) |
| DELETE | `/notes/:id/dates/:date` | Detach the note from a day (the note survives) |

**Create body**: `{ content, pinned?, contact_ids?, tag_ids?, dates? }` — `dates` is an array of `YYYY-MM-DD`.

**Update body**: `{ content?, contact_ids?, tag_ids?, dates? }` — `dates` **replaces** the full set of days (the way to move a note to another day; `[]` unlinks from every day).

### Note comments (marginalia)

Working material kept alongside a note without entering its text — an idea, a
reference, an excerpt. Never published, temporary by design (anything worth
keeping becomes a note or a linked task). Comments created via the API are
marked `author_type: "agent"` and shown in blue ink in the app.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notes/:id/comments` | List a note's comments |
| POST | `/notes/:id/comments` | Attach a comment. Body: `{ body, quote? }` |
| PATCH | `/notes/:id/comments/:commentId` | Edit content. Body: `{ body }` |
| DELETE | `/notes/:id/comments/:commentId` | Delete (the normal way to retire one) |

To anchor a comment to a passage, pass `quote` with that passage copied
**verbatim** from the note content — the server locates it and stores the
surrounding context so the anchor survives later edits. If the quote is not
found verbatim, the call is refused (`QUOTE_NOT_FOUND`) rather than attached to
the wrong place. Omit `quote` for a note-wide comment.

## Days

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/days` | List days with their intention or question of the day |
| GET | `/days/:date` | Get a day (date = YYYY-MM-DD). Always 200: `note` (intention, may be null), `exists`, and `notes[]` — the notes linked to that day |
| POST | `/days` | Create or update a day (upsert, body: `{ date, note }`) |
| PATCH | `/days/:date` | Update a day (body: `{ note }`) |

`note` is the **intention or question of the day**: one short line (mantra, intention, single priority, or a question to keep in mind). Not a journal — never write a summary of the day here, and never write a note's content here: to put a note on a day, use `POST /notes/:id/dates/:date` (or `dates` on the note).

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
