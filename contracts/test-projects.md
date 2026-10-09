# Test Contract: F1 Project Creation Consistency

## 1. Acceptance Criteria Checks (FR-01)

### AC1: Create and open project
- **Execution**: Automatable (API POST + UI form submission)
- **Given** a user on the Create Project screen (`/projects/new`),
- **When** valid project metadata is submitted,
- **Then** the API returns `201 Created` (`Location: /projects/{id}`) and the UI navigates to `/projects/{id}`.

### AC2: Stable identifier
- **Execution**: Automatable (API GET persistence + UI badge verification)
- **Given** a project created with assigned `id` (e.g. `prj_01`),
- **When** the project is retrieved via API or viewed in UI (list or overview),
- **Then** the identifier remains immutable, identical across views, and uniquely references the project.

### AC3: Project data isolation
- **Execution**: Automatable (API 404 response on unknown ID + UI 404 state verification)
- **Given** two distinct projects (`prj_01` and `prj_02`),
- **When** accessing `/projects/{id}` with an unknown or unauthorized ID,
- **Then** the API returns `404 Not Found` (UI displays 404 state) and never leaks cross-project records.

### AC4: Workspace active brief and shot list
- **Execution**: Automatable (UI component render test) / Manual (visual layout check)
- **Given** an existing project opened at `/projects/{id}`,
- **When** the project workspace loads successfully,
- **Then** the overview header displays static placeholder indicators "No brief yet" and "No shots yet".

## 2. Contract Mismatches (ui-projects.md vs api-projects.md)
- **None** — All specifications, UI contracts, and API contracts are 100% reconciled and aligned.
  - *Mismatches resolved per D-001 & automated reconciliation*:
    - Project Status on Creation: resolved (UI Create Project Form omits status input; server defaults to `draft` per D-001 #6 and `api-projects.md`).
    - Description optionality: resolved (optional in both UI and API per D-001 #1).
    - Header naming: resolved ("Project Name" in UI, `name` in API per D-001 #2).
    - Status casing: resolved (Title-case in UI, lowercase in API per D-001 #3).
    - Workspace placeholders: resolved (static placeholders per D-001 #4).
    - Error handling: resolved (400 inline errors, 500/network fallback banner per D-001 #5).

## 3. Open Questions
- None. All questions and mismatches are fully resolved. Automated execution proceeds without interruption.
