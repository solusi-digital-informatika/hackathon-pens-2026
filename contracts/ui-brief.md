# UI Specification: Brief Ingestion & Structured Review

## 1. Overview Screen Integration (`/projects/:id/overview`)
- Replaces "No brief yet" placeholder with **Brief Summary Card**:
  - Displays: Objective, Visual Style, Lighting/Mood, Extraction status (`Pending Review` / `Approved`).
  - Dual-coded status badge: `[● Approved]` or `[○ Pending Review]`.
  - Action button: "Open Brief Workspace" -> navigates to `/projects/:id/brief`.

## 2. Brief Workspace Screen (`/projects/:id/brief`)
### A. Ingestion Mode (when no brief exists)
- Large textarea for pasting raw brief text or markdown.
- Source Name input (default: "Creative Brief v1").
- Primary button: "Ingest & Extract Brief" (submits ingestion, triggers extraction, switches to Review Mode).

### B. Review & Edit Mode (when brief exists)
- Split view or stacked sections:
  1. **Source Document Panel**: Collapsible drawer/card showing original raw input.
  2. **Structured Form Fields (Editable by Human)**:
     - Visual Style (text input).
     - Lighting & Atmosphere (text input).
     - Characters (list of tags/cards with name & description, editable).
     - Key Props & Objects (list of tags/cards).
     - Constraints (list of items, add/remove/edit).
     - Unresolved Questions / Ambiguities (highlighted alert box with question list).
  3. **Action Bar**:
     - "Save Draft" (updates fields with `pending_review`).
     - "Approve Structured Brief" (saves and updates status to `approved`).
