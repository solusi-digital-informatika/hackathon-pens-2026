# UI Specification: F1 Project Screens

## 1. Project List (`/projects`)
- **Purpose**: View accessible projects (newest first, 20/page) and initiate creation.
- **Fields & Labels**:
  - "Project Name" (text heading and column label)
  - "Project ID" (stable identifier badge: e.g. `ID: prj_01`)
  - "Description" (text label, optional project summary)
  - "Status" (dual-coded icon + Title-case label: `[● Draft]`, `[● Active]`, `[● Archived]`, never color alone)
  - "Last Updated" (text label with date/timestamp)
- **Pagination & Sorting**: Default limit 20 (max 100), offset navigation using `total`, sorted by `created_at` desc.
- **States**:
  - *Empty*: "No projects found. Create a project to start." with "Create Project" button.
  - *Loading*: "Loading projects..." accessible indicator and placeholder rows.
  - *Error*: Inline/banner message for 400; fallback banner for 500 or network errors with "Retry" action.
  - *Success*: Table/list populated with project rows, IDs, statuses, and pagination controls.
- **Key Actions**: "Create Project" (opens creation form), "Open Project" (opens `/projects/{id}`).

## 2. Create Project Form (`/projects/new` or modal dialog)
- **Purpose**: Input metadata to create a new isolated project (plain CSS, browser fetch).
- **Fields & Labels**:
  - "Project Name" (required text input; label: "Project Name"; trimmed, 1–100 chars, whitespace-only rejected; duplicates allowed).
  - "Description" (optional multiline text; label: "Description"; trimmed, max 500 chars).
  - *(Status field omitted on creation: server automatically defaults to `draft` per D-001 #6 and `api-projects.md`).*
- **States**:
  - *Empty/Default*: Blank name and description inputs.
  - *Loading*: "Creating project..." with disabled inputs and progress indicator.
  - *Error*: Inline field messages for 400 validation; fallback banner for 500 or network errors.
  - *Success*: On 201 Created, confirms creation and immediately navigates to `/projects/{id}`.
- **Key Actions**: "Create & Open Project" (submits form, opens workspace), "Cancel" (returns to list).

## 3. Project Overview Header (`/projects/{id}`)
- **Purpose**: Workspace header identifying active project and static placeholders for milestone data.
- **Fields & Labels**:
  - "Project Name" (heading displaying project name).
  - "Project ID" (stable identifier badge: e.g. `ID: prj_01`; scoped isolation).
  - "Project Status" (dual-coded icon + Title-case label: Draft / Active / Archived).
  - "Active Brief": Static placeholder text "No brief yet" (brief endpoints out of scope for F1).
  - "Shot List": Static placeholder container "No shots yet" (shot endpoints out of scope for F1).
- **States**:
  - *Empty*: Workspace displays "No brief yet" and "No shots yet" static placeholders.
  - *Loading*: "Loading project workspace..." with skeleton placeholder.
  - *Error*: "Project not found or inaccessible" (404); fallback banner for 500 or network error.
  - *Success*: Header displays project ID, name, status, and static placeholders.
- **Key Actions**: "Back to Projects" (navigation).

## 4. FR-01 Acceptance Criteria Coverage
- *Create and open project*: "Create Project Form" creates and navigates to `/projects/{id}`; "Project List" provides "Open Project".
- *Stable identifier*: Immutable "Project ID" displayed on Project List and Overview Header.
- *Project isolation*: Dedicated `/projects/{id}` route; 404 state isolates projects; no auth UI in F1.
- *Active brief & shot placeholders*: Overview Header contains static "No brief yet" and "No shots yet".

## 5. Open Questions
- None (all prior questions resolved by D-001).

## 6. Changelog (D-001 Reconciliation)
- Item 1 (`description` optional): No change needed (already specified optional in UI).
- Item 2 (UI label "Project Name"): Changed Overview header label from "Project Title" to "Project Name".
- Item 3 (Status Title-case mapping): Changed; UI displays Title-case mapped to lowercase API values (`draft`, `active`, `archived`).
- Item 4 (Brief and shot static placeholders): Changed; set static placeholders "No brief yet" and "No shots yet" (endpoints out of scope).
- Item 5 (Error fallback and inline messages): Changed; added fallback banner for 500/network errors and inline messages for 400 errors.
- Item 6 (`status` on creation): Reconciled with `api-projects.md` and D-001 #6; removed status dropdown from create form so projects always initialize as `draft`.
- Item 7 (Field validation): Changed; name trimmed 1–100 chars (whitespace rejected); description trimmed max 500 chars.
- Item 8 (Pagination & sorting): Changed; added default limit 20, offset pagination, `total` handling, and `created_at` desc sorting.
- Item 9 (Duplicate names): Changed; confirmed duplicate names allowed without client-side blocking.
- Item 10 (Single user / 404): No change needed (404 state already defined; no auth UI in F1).
- Item 11 (Isolation scoping): No change needed (application-level scoping by `project_id` retained).
