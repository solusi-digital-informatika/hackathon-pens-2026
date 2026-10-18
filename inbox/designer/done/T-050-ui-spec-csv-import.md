# Task: T-050 UI Specification for CSV Import Wizard

## 1. Objective
Author comprehensive UI/UX specifications for the 5-step CSV Import Wizard (`/imports` or contextual modal/page) in `contracts/ui-csv-import.md`, adhering to the Clean Pragmatic Craftsman design system and TRD §8 CSV Import Spec.

## 2. Context & Reference Files
- `project-map/trd.md` (§8 CSV Import Spec, §9 API Inventory, §10 Security).
- `project-map/prd.md` (§5.5 CSV and Spreadsheet Import).
- `contracts/design-direction.md`: Clean Pragmatic Craftsman, Emerald Forest (`#0D6E64`), flat cards, no resting shadows, clear density.
- `contracts/design-tokens.md`: Standard color tokens, typography, spacing, radius, and motion tokens.

## 3. Scope of UI Specification
1. **5-Step Import Wizard Flow**:
   - **Step 1: Upload**:
     - File dropzone (drag & drop, max 5 MB, `.csv` only).
     - Template download links (`products`, `customers`, `transactions`).
     - Delimiter override dropdown (Auto-detect [default], Koma `,`, Titik Koma `;`, Tab `\t`).
     - Repeat upload warning banner if file hash matches previous batch.
   - **Step 2: Column Mapping & Parsing Settings**:
     - Source column -> Target field mapping table (Required vs Optional indicators).
     - Format select: Number format (`1.500,50` [id-ID] vs `1,500.50` [en]), Date format (`DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`).
     - Live preview of first 3 rows showing mapped interpretation.
   - **Step 3: Validation & Error Inspection**:
     - Summary pills: Valid rows count (green), Warnings count (amber), Error rows count (red).
     - Interactive table of rows with filters (`All`, `Only Errors`, `Only Warnings`).
     - Inline badge indicators and clear Indonesian explanation per row/field.
     - Ambiguous value resolution UI and "Kemungkinan Pelanggan yang Sama" resolution toggle.
   - **Step 4: Confirmation & Processing Progress**:
     - Final summary stats (Total items to import, items to skip, unlinked warnings).
     - Confirm button with modal confirmation.
     - Asynchronous progress bar with percentage, row counter (`X dari Y baris diproses`), and live status polling.
   - **Step 5: Completion Summary & Rollback**:
     - Final outcome state: `completed` (100% success) or `completed_with_errors`.
     - Button to download error report CSV (with formula injection neutralization).
     - Rollback / Cancel Import action with confirmation dialog (cancels created transactions with audit log).
2. **Responsive Breakdown**:
   - Mobile (390px): stacked cards for mapping, horizontal scroll on preview tables with fixed indicator.
   - Tablet (768px): 2-column layout for upload & templates.
   - Desktop (1440px): spacious grid, side-by-side mapping and live preview.
3. **Motion & States**:
   - Loading skeletons, upload progress bar, dropzone drag hover state, error banners, accessibility focus states.

## 4. Expected Output
- Author `contracts/ui-csv-import.md`.
- Update `board/board.md` T-050 status to `done`.
- Write completion report to `inbox/leader/T-050-report.md`.
- Move this task to `inbox/designer/done/T-050-ui-spec-csv-import.md`.
