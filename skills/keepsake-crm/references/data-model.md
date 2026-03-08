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
| firstName | string | yes | First name |
| lastName | string | no | Last name |
| email | string | no | Email address |
| phone | string | no | Phone number |
| address | string | no | Physical address |
| birthday | string | no | Birthday (YYYY-MM-DD) |
| notes | text | no | Free-form notes (Markdown) |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

### Company

An organization or business.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| name | string | yes | Company name |
| website | string | no | Website URL |
| industry | string | no | Industry/sector |
| notes | text | no | Free-form notes (Markdown) |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

Contacts can be linked to companies with an optional role (e.g. "CEO", "Designer").

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

A durable text document — like a digital index card. Intentional capture, short, reformulated.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| content | text | yes | Note content (Markdown) |
| pinned | boolean | no | Pinned for quick access |
| archived | boolean | no | Archived (stays searchable) |
| contactIds | uuid[] | no | Linked contacts |
| tagIds | uuid[] | no | Linked tags |
| createdAt | timestamp | auto | Creation date |
| updatedAt | timestamp | auto | Last update |

### Day

A daily journal entry / summary.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | uuid | auto | Unique identifier |
| date | string | yes | Date (YYYY-MM-DD) |
| content | text | no | Journal content (Markdown) |
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
Task    ──many-to-one───> TaskHeader (optional section)
```

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
