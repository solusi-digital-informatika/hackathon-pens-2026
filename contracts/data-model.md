# Project & Shot data model

## Scope
This contract defines the Project (FR-01) and Shot (FR-04) entities.

## 1. Project
| Field | Type | Required | Mutable | Source |
|---|---|---:|---:|---|
| `id` | stable opaque string | yes | no | PRD §11 Project; FR-01 stable identifier |
| `name` | string | yes | yes | PRD §11 Project; FR-01 |
| `description` | string | no | yes | D-001 #1: optional in API and UI |
| `status` | enum string | yes | yes | PRD §11 Project; FR-01; D-001 #6 |
| `created_at` | timestamp | yes | no | PRD §11 Project |
| `updated_at` | timestamp | yes | system-managed | PRD §11 Project |

### Status enum (Project)
Valid values: `draft`, `active`, `archived`. Default on create: `draft`.
The API stores lowercase; the UI shows Title-case labels via a client-side mapping.

## 2. Shot
| Field | Type | Required | Mutable | Source |
|---|---|---:|---:|---|
| `id` | stable opaque string | yes | no | PRD §11 Shot; FR-04 stable ID |
| `project_id` | string (FK to project) | yes | no | PRD §11 Shot; FR-01 isolation |
| `sequence_order` | integer (>= 1) | yes | yes | PRD §11 Shot; FR-04 ordering |
| `title` | string (1-100 chars) | yes | yes | PRD §11 Shot; FR-04 |
| `description` | string (<= 1000 chars) | no | yes | PRD §11 Shot; FR-04 |
| `status` | enum string | yes | yes | PRD §11 Shot |
| `created_at` | timestamp | yes | no | System-assigned |
| `updated_at` | timestamp | yes | system-managed | System-assigned |

### Status enum (Shot)
Valid values: `draft`, `in_progress`, `in_review`, `approved`. Default on create: `draft`.

## Validation rules (D-001 #7, D-003)
- Project `name`: trimmed, 1–100 characters; whitespace-only is rejected.
- Project `description`: trimmed, at most 500 characters; may be absent or empty string.
- Shot `title`: trimmed, 1–100 characters; whitespace-only is rejected.
- Shot `description`: trimmed, at most 1000 characters; optional.

## Invariants
- `id` for projects and shots are stable and immutable (FR-01, FR-04).
- Every shot is associated with exactly one `project_id`; reads through one project must not include another project's data (FR-01; D-001 #11 application-level scoping).

## Changelog
| Date | Change |
|---|---|
| 2026-10-09 | F1: Reconciled Project data model per D-001. |
| 2026-10-09 | F2: Added Shot entity and status enum per D-003 and PRD §11 (FR-04). |

