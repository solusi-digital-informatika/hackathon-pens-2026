# Project data model

## Scope
This contract defines only the Project entity needed by FR-01. Related brief and shot
records remain separate project-owned entities; their schemas are outside this file.

## Project
| Field | Type | Required | Mutable | Source |
|---|---|---:|---:|---|
| `id` | stable opaque string | yes | no | PRD §11 Project; FR-01 stable identifier |
| `name` | string | yes | yes | PRD §11 Project; FR-01 |
| `description` | string | yes | yes | PRD §11 Project; FR-01 short description |
| `status` | string | yes | yes | PRD §11 Project; FR-01 |
| `created_at` | timestamp | yes | no | PRD §11 Project |
| `updated_at` | timestamp | yes | system-managed | PRD §11 Project |

## Invariants
- `id` is generated once and remains unchanged (FR-01).
- Every brief, shot, and other project-owned record is associated with exactly one
  `project_id`; reads through one project must not include another project's data
  (FR-01 project-data separation; PRD §11 related entities).

## Assumptions
- Creating a Project does not implicitly create a brief or shots; project creation
  precedes brief import in the PRD primary journey (§6).
- Timestamps use an interoperable date-time string in API representations.
- IDs are opaque to clients; their generation format is an implementation detail.
- `created_at` and `updated_at` are assigned by the system, not accepted on create.
- Transport-level error fields are not Project domain fields.

## Open questions
- What values and state transitions are valid for `status`?
- Are whitespace-only names/descriptions invalid, and what length limits apply?
- May `description` be empty despite being required on creation?
- Which identity and authorization rules control project access?
- Does project isolation require storage-level isolation or only access scoping?
