# Changelog

All notable changes to the Keepsake Agent Skill will be documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/).

## [1.5.0] - 2026-09-29

### Added
- **Tasks, notes and entries linked to companies.** `company_ids` on create/update
  (separate from `contact_ids`), returned on every task, note and entry; `company_id`
  filter on the three lists; `link_`/`unlink_` + `task`/`note`/`entry` + `_company`
  tools (keepsake-mcp 1.12.0).
- `get_task`: read one task with its tags, contacts, companies and linked notes.

## [1.4.0] - 2026-09-29

### Added
- **Companies as records, everywhere.** A contact's company is a linked company
  record (`companies: [{ id, name, role }]`), never free text. `company: "Name"` on
  `create_contact` / `update_contact` finds or creates the record and links it.
- New tools: `link_contact_company` (with role), `unlink_contact_company`,
  `merge_companies` (keepsake-mcp 1.11.0).
- `list_contacts`: `fields` for lighter responses, filters `company`,
  `has_company`, `updated_since`.

### Fixed
- SKILL.md named a non-existent tool (`link_company_contact`).
- API reference: contact and company fields now use the API's snake_case names;
  unlink endpoint is `DELETE /companies/:id/contacts?contact_id=`; company fields
  are `website, email, phone, address, notes` (there is no `industry`).

## [1.3.0] - 2026-09-06

### Added
- **Notes on a day.** A note can be attached to one or more calendar days:
  `dates` on `create_note` / `update_note`, `link_note_date` / `unlink_note_date`,
  `list_notes` filtered by `date` / `date_from` / `date_to`, and `get_day` now
  returns the day's linked notes (`notes`) next to its intention (`note`).
  Workflow added to SKILL.md ("note for tomorrow" is a dated note, not a task,
  and not the day's intention). API reference and data model updated
  (keepsake-mcp 1.10.0).

## [1.2.1] - 2026-09-03

### Changed
- **Days are the intention or question of the day, not a journal.** The daily
  ritual no longer tells agents to write an end-of-day summary via `update_day`
  (it overwrote the user's one-line intention). What happened during the day
  belongs in entries. API reference fixed: `/days` body is `{ note }`, with the
  `POST /days` upsert documented; data model updated (keepsake-mcp 1.9.1).

## [1.2.0] - 2026-08-11

### Added
- **Note comments (marginalia)** — work in the margin of a note without touching
  its text: `list_note_comments`, `create_note_comment` (anchor to a passage by
  quoting it verbatim, or comment note-wide), `update_note_comment`,
  `delete_note_comment`. Comments are never published with the note and are
  temporary by design — deletion is the normal exit. Agent comments render in
  blue ink in the app, the user's in red.
- API reference and data model updated accordingly (keepsake-mcp 1.7.0, 67 tools).


## [1.1.0] - 2026-08-01

### Changed
- Tag endpoints documentation: `GET /tags` now supports name search (`q`), pagination, and omits ordering arrays by default; `GET /tags/:id/items` supports `types`, `status` and `summary` filters and returns explicit task `sections`. Added guidance to prefer filtered summary requests on large tags.

## [1.0.0] - 2026-03-08

### Added
- Initial release
- SKILL.md with 3-dimension model (Time, People, Themes)
- Workflows: capture interactions, daily ritual, knowledge base, cross-dimensional search
- API reference documentation
- Data model reference
