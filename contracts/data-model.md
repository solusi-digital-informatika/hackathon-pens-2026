# Project data model

## Scope
This contract defines only the Project entity needed by FR-01. Brief and shot
records are separate project-owned entities; their schemas are outside this file.

## Project
| Field | Type | Required | Mutable | Source |
|---|---|---:|---:|---|
| `id` | stable opaque string | yes | no | PRD §11 Project; FR-01 stable identifier |
| `name` | string | yes | yes | PRD §11 Project; FR-01 |
| `description` | string | no | yes | D-001 #1: optional in API and UI |
| `status` | enum string | yes | yes | PRD §11 Project; FR-01; D-001 #6 |
| `created_at` | timestamp | yes | no | PRD §11 Project |
| `updated_at` | timestamp | yes | system-managed | PRD §11 Project |

## Status enum (D-001 #6)
Valid values: `draft`, `active`, `archived`. Default on create: `draft`.
The API stores lowercase; the UI shows Title-case labels via a client-side mapping.

## Validation rules (D-001 #7)
- `name`: trimmed, 1–100 characters; whitespace-only is rejected.
- `description`: trimmed, at most 500 characters; may be absent or empty string.

## Invariants
- `id` is generated once and remains unchanged (FR-01).
- Every brief, shot, and other project-owned record is associated with exactly one
  `project_id`; reads through one project must not include another project's data
  (FR-01; D-001 #11 application-level scoping).

## Assumptions
- Creating a Project does not implicitly create a brief or shots (PRD §6).
- Timestamps are interoperable date-time strings (e.g. ISO 8601 UTC) in API responses.
- IDs are opaque to clients; generation format is an implementation detail.
- `created_at` and `updated_at` are system-assigned, not accepted on create.
- No authentication in F1; a single implicit user owns all projects (D-001 #10).
- Duplicate project names are allowed; identity is the unique ID (D-001 #9).
- Transport-level error fields are not Project domain fields.

## Changelog
| Date | Change |
|---|---|
| 2026-10-09 | `description` changed to optional (D-001 #1). |
| 2026-10-09 | `status` enum values added: draft/active/archived, default draft (D-001 #6). |
| 2026-10-09 | Validation rules for `name` and `description` added (D-001 #7). |
| 2026-10-09 | Isolation clarified as application-level scoping (D-001 #11). |
| 2026-10-09 | No-auth note added; duplicate names allowed noted (D-001 #9, #10). |
| 2026-10-09 | All open questions resolved; section removed. |
