# UI Specification: F1 Project Screens

## 1. Project List (`/projects`)
- **Purpose**: View accessible projects and initiate creation of new projects.
- **Fields & Labels**:
  - "Project Name" (plain-language text label and heading)
  - "Project ID" (stable identifier badge: e.g. `ID: prj_01`)
  - "Description" (text label, brief project summary)
  - "Status" (dual-coded: icon prefix + text label e.g. `[● Draft]`, never color alone)
  - "Last Updated" (text label with date/timestamp)
- **States**:
  - *Empty*: "No projects found. Create a project to start." with "Create Project" button.
  - *Loading*: "Loading projects..." accessible indicator and placeholder rows.
  - *Error*: "Unable to load projects. Please retry." with "Retry" button.
  - *Success*: Table/list populated with project rows, IDs, and statuses.
- **Key Actions**: "Create Project" (opens creation form), "Open Project" (opens `/projects/{id}`).

## 2. Create Project Form (`/projects/new` or modal dialog)
- **Purpose**: Input required metadata to create a new isolated project.
- **Fields & Labels**:
  - "Project Name" (required text input; label: "Project Name")
  - "Description" (optional multiline text; label: "Description")
  - "Status" (dropdown select; label: "Project Status", defaults to "Draft")
- **States**:
  - *Empty/Default*: Blank name and description inputs; status defaults to "Draft".
  - *Loading*: "Creating project..." with disabled inputs and progress indicator.
  - *Error*: Inline field messages (e.g. "Project Name is required") and error summary banner.
  - *Success*: On 201 Created, confirms creation and immediately opens `/projects/{id}`.
- **Key Actions**: "Create & Open Project" (submits form, opens workspace), "Cancel" (returns to list).

## 3. Project Overview Header (`/projects/{id}`)
- **Purpose**: Workspace header identifying the active project, brief version, and shot list.
- **Fields & Labels**:
  - "Project Title" (heading displaying project name)
  - "Project ID" (stable identifier badge: e.g. `ID: prj_01`)
  - "Project Status" (dual-coded: icon + text e.g. `[● Draft]`, never color alone)
  - "Active Brief Version" (badge/label: e.g. `Active Brief: v1.0` or `No active brief`)
  - "Shot List" (placeholder section container with header: "Shot List" and count: "0 shots")
- **States**:
  - *Empty*: Brand new workspace displays "No active brief" and empty shot list placeholder.
  - *Loading*: "Loading project workspace..." with skeleton placeholder.
  - *Error*: "Project not found or inaccessible (404)." with "Back to Projects" action.
  - *Success*: Header displays project ID, title, status, active brief tag, and shot list.
- **Key Actions**: "Back to Projects" (navigation), "Import Brief" (launch brief workflow).

## 4. FR-01 Acceptance Criteria Coverage
- *Create and open project*: "Create Project Form" creates and navigates to workspace; "Project List" provides "Open Project" action.
- *Stable identifier*: "Project ID" (e.g. `prj_01`) is displayed on both Project List and Overview Header.
- *Project isolation*: Dedicated `/projects/{id}` route; 404 state isolates projects and prevents data leaks.
- *Active brief version and shot list*: Project Overview Header contains "Active Brief Version" indicator and "Shot List" section placeholder.

## 5. Open Questions
- Permitted values for `status` beyond "Draft" (e.g., "Active", "Archived").
- Maximum character limits and whitespace validation rules for Project Name and Description.
- Sorting and pagination rules for the Project List screen.
