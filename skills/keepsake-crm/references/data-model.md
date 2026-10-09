# Keepsake Data Model

## Decision Tree — Which Entity to Use?

1. Is it an action to do? → **Task**
2. Is it something that happened on a specific date involving contacts? → **Entry**
3. Is it reference info or an idea to keep? → **Note** (QuickNote)
4. Need to group elements by theme or project? → **Tag**

## Entities

### Contact

A person in the user's network.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| first_name | string | yes | First name |
| last_name | string | no | Last name |
| email | string | no | Email address |
| phone | string | no | Phone number |
| job_title | string | no | Job title |
| address | string | no | Physical address |
| birth_day / birth_month / birth_year | integer | no | Birthday (write it with `birthday: "YYYY-MM-DD"`) |
| notes | text | no | Free-form notes (Markdown) |
| companies | array | computed | Linked company records: `[{ id, name, role }]` |
| created_at | timestamp | auto | Creation date |
| updated_at | timestamp | auto | Last update |

A contact's company is always a **company record**, never free text. `company: "Name"` on create/update is a shortcut that links (or creates) the record.

### Company

An organization or business.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| name | string | yes | Company name |
| website | string | no | Website URL |
| email | string | no | Email address |
| phone | string | no | Phone number |
| address | string | no | Address |
| notes | text | no | Free-form notes (Markdown) |
| created_at | timestamp | auto | Creation date |
| updated_at | timestamp | auto | Last update |

Contacts can be linked to companies with an optional role (e.g. "CEO", "Designer"). A contact can belong to several companies. Duplicate companies are merged, not deleted, so nothing is lost.

### Entry

A dated interaction log tied to one or more contacts.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| type | enum | yes | One of: call, email, meeting, event, gift, letter, message, other |
| date | string | yes | Date (YYYY-MM-DD) |
| content | text | no | Description (Markdown) |
| contactIds | uuid[] | no | Linked contacts |
| tagIds | uuid[] | no | Linked tags |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

### Task

An action item to accomplish.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| title | string | yes | Task title |
| description | text | no | Details (Markdown) |
| date | string | no | Due date (YYYY-MM-DD) |
| dateType | enum | no | "specific" or "no_date" |
| priority | integer | no | 1 (highest) to 4 (lowest) |
| completed | boolean | auto | Whether task is done |
| completedAt | timestamp | auto | When completed |
| recurrence | string | no | Recurrence pattern |
| snoozedUntil | string | no | Snoozed until date |
| primaryContactId | uuid | no | Main contact |
| contactIds | uuid[] | no | Additional contacts |
| tagIds | uuid[] | no | Linked tags |
| sectionId | uuid | no | Task section/header |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

**Recurrence types**: daily, weekdays, weekly, biweekly, monthly, quarterly, yearly.
Completing a recurring task auto-creates the next occurrence.

### QuickNote (Note)

A durable text document — like a digital index card. Intentional capture, short, reformulated. Can be attached to one or more calendar days (it then surfaces in that day's view) — neither a task nor the day's intention.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| content | text | yes | Note content (Markdown) |
| pinned | boolean | no | Pinned for quick access |
| archived | boolean | no | Archived (stays searchable) |
| contactIds | uuid[] | no | Linked contacts |
| tagIds | uuid[] | no | Linked tags |
| dates | string[] | no | Days the note is linked to (YYYY-MM-DD) |
| status | object \| null | no | Stage in the publication flow (`id`, `key`, `name`, `color`, `category`), or null when the note is outside the flow. Set with a stage name, key or id from `list_note_statuses` |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

### Day

A calendar day carrying the user's **intention or question of the day** — one short line shown at the top of the Today view (a mantra, an intention, a single priority, or a question to keep in mind). Not a journal: what happened during the day lives in entries.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| date | string | yes | Date (YYYY-MM-DD) |
| note | text | no | Intention or question of the day (one short line) |
| notes | Note[] | read-only | Notes linked to this day (via `dates` on notes) |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

### Tag

A thematic grouping space / project page.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| name | string | yes | Tag name |
| emoji | string | no | Display emoji |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

**Syntax in content**: `#tag name#` or `[[tag name]]` — auto-creates and links.

### TaskHeader

A section header for organizing tasks into groups.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| name | string | yes | Section name |
| position | integer | no | Sort order |
| createdAt | timestamp | auto | Creation date |

## Relations

```
Contact <──many-to-many──> Entry
Contact <──many-to-many──> Task (one can be primary)
Contact <──many-to-many──> Note
Contact <──many-to-many──> Company (with role)
Tag     <──many-to-many──> Entry
Tag     <──many-to-many──> Task
Tag     <──many-to-many──> Note
Tag     <──many-to-many──> Contact
Note    <──many-to-many──> Day (dated note: surfaces in the Day view)
Task    ──many-to-one───> TaskHeader (optional section)
Comment ──many-to-one───> Note (marginalia, anchored to a passage or note-wide)
```

A **note comment** (marginalia) is working material kept alongside a note
without entering its text: `body` (markdown), an optional anchor (`quote` — the
exact passage, plus server-derived context), and `author_type` (`user` or
`agent`; API-created comments are always `agent` and render in blue ink).
Comments are temporary by design — there is no resolved state, deletion is the
normal exit; anything worth keeping becomes a note or a linked task. They never
appear in published notes.

Every note, entry, and task can be linked to 0-N contacts and 0-N tags. Use dedicated link/unlink endpoints for granular control, or pass `contact_ids`/`tag_ids` arrays at creation time.

## Entry Types

| Type | Use for |
|------|---------|
| call | Phone or video call |
| email | Email exchange |
| meeting | In-person or virtual meeting |
| event | Social event, conference, etc. |
| gift | Gift given or received |
| letter | Physical letter or card |
| message | Text message, chat, DM |
| other | Anything else |
