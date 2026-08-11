# Changelog

All notable changes to the Keepsake Agent Skill will be documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/).

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
