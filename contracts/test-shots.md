# Test Contract: F2 Shot CRUD and Shot Board

## 1. Acceptance Criteria Checks (FR-04)

### AC1: Create Shot
- **Execution**: Automatable (API POST + UI Modal)
- **Given** an existing project `/projects/{project_id}`,
- **When** valid shot data (`title`, optional `description`) is submitted,
- **Then** the API returns `201 Created` with unique stable `id`, auto-assigned `sequence_order`, and `draft` status; UI renders the shot card on the Shot Board.

### AC2: List and Sequence Ordering
- **Execution**: Automatable (API GET)
- **Given** multiple shots in a project,
- **When** retrieving shots via `GET /projects/{project_id}/shots`,
- **Then** shots are returned sorted by `sequence_order` ascending.

### AC3: Update and Delete Shot
- **Execution**: Automatable (API PATCH & DELETE)
- **When** updating shot attributes or deleting a shot,
- **Then** changes persist; deletion removes the shot (`204 No Content`) and returns `404` on subsequent get.

### AC4: Project Scoping and Isolation
- **Execution**: Automatable (API 404 test)
- **When** accessing shots with an invalid `project_id`,
- **Then** API returns `404 Not Found`.

### AC5: Shot Board UI & Placeholders
- **Execution**: Automatable (Component test) / Manual (Visual)
- **Given** an existing project overview `/projects/{id}`,
- **When** shots exist,
- **Then** the "No shots yet" placeholder is replaced by the Shot Board displaying dual-coded statuses and sequence badges.

## 2. Test Dataset
- Validated against the 6 ground-truth shots in `contracts/test-scenario.md` (`SHOT-01` to `SHOT-06`).
