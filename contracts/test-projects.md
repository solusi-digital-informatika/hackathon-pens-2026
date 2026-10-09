# Test Contract: F1 Project Creation Consistency

## 1. Acceptance Criteria Checks (FR-01)

### AC1: Create and open project
- **Given** a user on the Create Project screen (`/projects/new`),
- **When** valid project metadata (`name`, `description`, `status`) is submitted,
- **Then** the API returns `201 Created` (`Location: /projects/{id}`) and the UI navigates to `/projects/{id}`.

### AC2: Stable identifier
- **Given** a project created with assigned `id` (e.g. `prj_01`),
- **When** the project is retrieved via API or viewed in UI (list or overview),
- **Then** the identifier remains immutable, identical across views, and uniquely references the project.

### AC3: Project data isolation
- **Given** two distinct projects (`prj_01` and `prj_02`),
- **When** accessing `/projects/{id}` with an unknown or unauthorized ID,
- **Then** the API returns `404 Not Found` (UI displays 404 state) and never leaks cross-project records.

### AC4: Workspace active brief and shot list
- **Given** an existing project opened at `/projects/{id}`,
- **When** the project workspace loads successfully,
- **Then** the overview header displays the active brief version indicator and shot list container.

## 2. Contract Mismatches (ui-projects.md vs api-projects.md)
- **Fields (Optionality)**: `ui-projects.md` marks `description` optional; `api-projects.md` requires it on `POST /projects`.
- **Fields (Naming)**: `ui-projects.md` labels header "Project Title"; `api-projects.md` specifies entity field `name`.
- **Fields (Values)**: `ui-projects.md` displays title-case "Draft"; `api-projects.md` schema specifies lowercase `"draft"`.
- **States (Workspace Data)**: `ui-projects.md` displays brief version and shot list in overview; `api-projects.md` `GET /projects/{id}` returns only project metadata without nested brief/shot data.
- **Errors (Error Mapping)**: `api-projects.md` defines structured 400 envelope (`error.fields`); `ui-projects.md` defines inline messages but lacks specification for handling API 500 or non-validation creation errors.

## 3. Open Questions
- What are the allowed enum values and casing for `status` across UI and API?
- What maximum length and whitespace validation rules apply to `name` and `description`?
- How are pagination, sorting, and filter query parameters handled on `GET /projects`?
