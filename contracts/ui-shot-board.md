# UI Specification: Project Overview & Shot Board
**Document ID:** `UI-SPEC-001`  
**Milestone:** M1 (Structured Project Data)  
**Target Roles:** Frontend (`frontend`), Backend (`backend-ai`), Tester (`tester`)  
**Status:** Implementation-Ready Contract  
**References:** 
- `project-map/PRD.md` (§9.1, §9.2, FR-01, FR-04, FR-05, FR-06, FR-11, NFR-01, NFR-07)
- `project-map/product-brief.md`
- `board/board.md` (Task T-003)

---

## 1. Executive Summary & Design Principles

This specification defines the visual hierarchy, component architecture, responsive layout, interaction states, and accessibility standards for two foundational screens:
1. **Project Overview** (PRD §9.1 item 2, FR-01)
2. **Shot Board** (PRD §9.1 item 4, PRD §9.2, FR-04, FR-11)

### Non-Negotiable Design Directives
- **Zero AI-Slop Aesthetics:** Clean, functional, information-dense production UI inspired by professional creative pipelines (e.g., Linear, DaVinci Resolve, Figma). No gratuitous glowing gradients, decorative floating shapes, unreadable glassmorphism, or fake metric widgets.
- **Strict Accessibility (NFR-07):** Every status, direction state, and action recommendation **must be dual-coded**. Never communicate meaning via color alone. Every badge and indicator couples a distinct geometric icon, explicit textual label, and WCAG 2.1 AA compliant contrast (≥ 4.5:1 text, ≥ 3:1 graphical elements).
- **Explicit Version Traceability:** The active direction version and each shot's version must remain persistently visible across all views. When a shot was authored under an older direction version, its desynchronized state must be visibly announced.
- **Human-in-the-Loop Authority:** AI recommendations and human-approved statuses are visually separated and never conflated.

---

## 2. Information Architecture & Shell Navigation

The application uses a persistent top shell or collapsible left rail navigation reflecting the 7 core functional areas outlined in PRD §9.1.

### 2.1 Global Navigation Map
```
[App Shell]
 ├── Global Header
 │    ├── Project Selector & Workspace Breadcrumb ("Projects / [Project Name]")
 │    ├── Stable Project Identifier ("PRJ-001")
 │    ├── Active Creative Direction Badge ("Dir: v1.0 [Active]")
 │    └── Team Timezone Indicator (Tokyo UTC+9 · Surabaya UTC+7)
 └── Navigation Menu (PRD §9.1)
      ├── 1. Projects (Directory & Creation)
      ├── 2. Project Overview [SPECIFIED HERE]
      ├── 3. Brief & Knowledge (Ingestion, Structured Requirements)
      ├── 4. Shot Board [SPECIFIED HERE]
      ├── 5. Direction Changes (Comparison & Impact Review)
      ├── 6. Version History (Traceability Matrix)
      └── 7. Quality & Delivery (Checklists & Approvals)
```

---

## 3. Screen Specification 1: Project Overview

### 3.1 Page Metadata
- **Route:** `/projects/:projectId/overview`
- **Purpose:** Provide an at-a-glance health check of the production pipeline, active creative direction, outstanding approvals, and quick entry into the Shot Board.
- **Primary Users:** Creative Director (Persona A), AI VFX Producer (Persona B).
- **Primary Action:** Review outstanding actions and transition to Shot Board.

### 3.2 Layout Structure
A responsive 12-column grid (`max-w-7xl mx-auto px-6 py-8`):
- **Desktop (≥ 1024px):** 8 columns (Main Feed) + 4 columns (Sidebar Context).
- **Tablet / Mobile (< 1024px):** Stacks vertically (12 columns single stack).

```
+---------------------------------------------------------------------------------+
| Project Header: Title | PRJ-001 | Status: Active | Dir: v1.0 (Active) | Timezone|
+-------------------------------------------------------+-------------------------+
| [MAIN CONTENT - 8 cols]                               | [SIDEBAR - 4 cols]      |
|                                                       |                         |
| 1. Project Summary & Brief Snapshot                   | 4. Shot Pipeline Status |
|    - Title, Description, Creation & Modified metadata |    - Draft: 2           |
|    - Source document link & extraction status         |    - In Progress: 1     |
|                                                       |    - In Review: 2       |
| 2. Active Creative Direction Card                     |    - Approved: 1        |
|    - Version: v1.0 (Active)                           |                         |
|    - Visual Style: Realistic Cinematic                +-------------------------+
|    - Lighting/Mood: Cool Blue Lighting                | 5. Quick Actions        |
|    - Primary Constraints & Target Aspect Ratio        |    - [Go to Shot Board] |
|    - [View Direction Details] [Compare Changes]       |    - [+ Add Shot]       |
|                                                       |    - [Review Brief]     |
| 3. Outstanding Actions & Production Health (PRD §14)  +-------------------------+
|    - Retain: 3 | Adapt: 1 | Regenerate: 1 | Review: 1 | 6. Recent Audit Events  |
|    - Critical Alerts (e.g. 2 shots pending review)    |    - Timestamped log    |
+-------------------------------------------------------+-------------------------+
```

### 3.3 Field & Component Specification for Project Overview

#### A. Project Summary Card
- **Project Title:** `H1`, Font Size `1.5rem / 24px`, Semi-bold (`font-semibold text-neutral-900`).
- **Stable Project ID:** Badge `font-mono text-xs px-2 py-0.5 rounded bg-neutral-100 text-neutral-700 border border-neutral-300`.
- **Description:** Body text, `text-sm text-neutral-600 line-clamp-3`.
- **Status Badge:** Dual-coded badge (see §5 for token definitions).
- **Metadata Strip:** Created timestamp, last updated timestamp, total shot count.

#### B. Active Creative Direction Banner
Displays the authoritative creative reference (FR-06, NFR-07):
- **Version Identifier:** `v1.0` (with green active pill: `[● Active Direction]`).
- **Style Classification:** Text label, e.g., "Realistic Cinematic · Cool Blue Lighting".
- **Summary Excerpt:** Extracted core direction summary (max 180 chars).
- **Direction Age / Last Modified:** e.g., "Approved 2 hours ago by Director".
- **Action Links:**
  - Button `View Direction Details` (Ghost button).
  - Button `Propose Direction Change` (Secondary button).

#### C. Production Health & Change Impact Summary
Summarizes shot distribution and impact states (FR-08, FR-14):
- **Action Breakdown Counters:** 4 metric cards displaying:
  1. **Retain:** `[🛡️ Retain: 3]` (Work remains compatible).
  2. **Adapt:** `[✏️ Adapt: 1]` (Prompts/assets need minor adjustments).
  3. **Regenerate:** `[🔄 Regenerate: 1]` (Incompatible with active direction).
  4. **Needs Review:** `[⚠️ Needs Review: 1]` (Uncertainty or missing dependency).
- **Outstanding Approval Alert:** If unreviewed shots > 0, an alert box displays:
  `⚠️ Attention Required: 2 shots require human review before production advances.`

---

## 4. Screen Specification 2: Shot Board (Key Screen)

### 4.1 Page Metadata
- **Route:** `/projects/:projectId/shots`
- **Purpose:** Primary production workspace for managing shot sequences, inspecting thumbnails, prompts, versions, dependencies, and approval states (PRD §9.2, FR-04, FR-11).
- **Primary Users:** AI VFX Producer (Persona B), AI Artist (Persona C), Creative Director (Persona A).
- **Primary Action:** Sequence review, shot prompt inspection, status update, shot reordering.

### 4.2 Board Toolbar & Filter Controls
The top toolbar provides essential operational controls:
- **Sequence View Switcher:** Toggle between **Storyboard Cards** (visual thumbnail grid) and **List / Table** (data-dense tabular view).
- **Status Filter:** Multi-select dropdown (`All`, `Draft`, `In Progress`, `In Review`, `Approved`).
- **Direction Alignment Filter:** Filter by `All`, `Aligned with Active Direction`, `Outdated / Direction Mismatch`.
- **Impact Action Filter:** Filter by `Retain`, `Adapt`, `Regenerate`, `Needs Review`.
- **Search Bar:** Real-time filter by Shot ID (`SHT-xxx`) or Title keyword.
- **Add Shot Button:** Primary button `+ Add Shot` (`bg-primary-600 text-white hover:bg-primary-700`).

---

### 4.3 Shot Card Specification (PRD §9.2 Exhaustive Field Mapping)

Every shot card in the Storyboard View displays all mandatory fields mandated by PRD §9.2 and FR-04:

```
+-------------------------------------------------------------------------------+
| [::] #01  SHT-001 : Opening Neon Street Wide               [...] (Action Menu)|
+-------------------------------------------------------------------------------+
|                                                                               |
|                   [ 16:9 Thumbnail / Asset Preview ]                          |
|                   (Or Placeholder: "No Asset Selected")                       |
|                                                                               |
| [Asset v1 (Primary)]                                [3 Alternatives Available]|
+-------------------------------------------------------------------------------+
| STATUS & AUDIT STRIP                                                          |
| [✓ Approved]  [Dir: v1.0 Aligned]  [Shot: v2]       [⚠️ Needs Review: Adapt]  |
+-------------------------------------------------------------------------------+
| DESCRIPTION & CONSTRAINTS                                                     |
| "Wide establishing shot of rain-soaked neon street at midnight..."            |
+-------------------------------------------------------------------------------+
| RELEVANT REQUIREMENTS (PRD §9.2)                                              |
| [REQ-01: Cyberpunk Mood]  [REQ-04: Cool Blue Palette]  [+1 more]              |
+-------------------------------------------------------------------------------+
| ACTIVE PROMPT SNIPPET (PRD §9.2, FR-05)                                       |
| Prompt: "Cinematic 8k wide shot of wet asphalt reflecting cyan neon..."       |
| [Copy Prompt]  [Expand Prompt]                                                |
+-------------------------------------------------------------------------------+
| DEPENDENCIES & SEQUENCE (FR-04)                                               |
| Dependencies: SHT-000 (Intro Beat), Asset #A12                                 |
+-------------------------------------------------------------------------------+
| CARD FOOTER CONTROLS                                                          |
| [↑ Move Up]  [↓ Move Down]                    [Edit Details]  [New Version]   |
+-------------------------------------------------------------------------------+
```

#### Detailed Field Requirements on Each Shot Card:

| # | Field Name | PRD Clause | Component Representation | Validation & Interaction Rules |
|---|------------|------------|--------------------------|--------------------------------|
| 1 | **Shot Sequence Number** | FR-04 | `#01`, `#02`, `#03` badge | Auto-increments. Updated when reordered. |
| 2 | **Stable Shot ID** | PRD §9.2, FR-04 | `SHT-001` monospace chip | **Immutable across versions.** Never changes when prompts, assets, or direction change. |
| 3 | **Shot Title** | PRD §9.2, FR-04 | `H3` semi-bold text (`text-base`) | Editable in shot detail drawer. Max 80 chars. |
| 4 | **Reorder Handles** | FR-04, FR-11 | Drag grip `::` + Accessible `Move Up` / `Move Down` buttons | Supports drag-and-drop AND keyboard navigation (Alt+Arrow). |
| 5 | **Thumbnail / Asset Preview** | PRD §9.2, FR-05 | 16:9 aspect ratio image container | If image exists: renders thumbnail with zoom click. If empty: renders dashed placeholder with picture icon and "+ Attach Asset" action. |
| 6 | **Asset Selector Tag** | FR-05 | `Asset v1 (Selected)` pill | Displays active asset version and count of alternatives (e.g. "+2 alts"). |
| 7 | **Production Status** | PRD §9.2, NFR-07 | Dual-coded badge (Icon + Text) | Values: `Draft`, `In Progress`, `In Review`, `Approved`. |
| 8 | **Direction Version** | PRD §9.2, NFR-07 | Explicit version pill | Shows linked direction version (e.g., `Dir v1.0`). Must indicate if it matches or is outdated relative to Active Project Direction. |
| 9 | **Shot Version** | PRD §9.2, NFR-07 | Monospace version tag | e.g. `Shot v2 (Active)`. Shows previous versions exist in drawer. |
| 10 | **Outstanding Review Indicator** | PRD §9.2, FR-08, FR-09 | High-visibility warning banner on card | Displays when human review is required. Shows recommended action: `Retain`, `Adapt`, `Regenerate`, or `Needs Review`. |
| 11 | **Relevant Requirements** | PRD §9.2, FR-04 | Chip list with requirement IDs | e.g. `[REQ-01: Cyberpunk Mood]`. Hover shows tooltip with source excerpt. |
| 12 | **Active Prompt Preview** | PRD §9.2, FR-05 | Collapsible code/quote block | Truncated to 2 lines. Includes `Copy Prompt` icon button and `Expand` trigger. |
| 13 | **Known Dependencies** | FR-04, PRD §12.3 | Dependency pill list | Links to predecessor shot or character reference assets. If unverified, flags `⚠️ Unresolved Dependency`. |
| 14 | **Card Action Overflow** | FR-04 | 3-dot dropdown menu | `Edit Shot`, `Create New Revision`, `Duplicate`, `View Version History`, `Delete Shot`. |

---

### 4.4 List / Table View Alternative (High-Density Scanning)
When toggled to List View, the board displays a structured table with sticky headers:
- **Columns:**
  1. `Seq` (`#01`)
  2. `Shot ID` (`SHT-001`)
  3. `Thumbnail` (Small 48x27px thumbnail)
  4. `Title & Description`
  5. `Status` (Dual-coded badge)
  6. `Shot Ver` (`v2`)
  7. `Dir Ver` (`v1.0 Aligned` vs `v1.0 Outdated`)
  8. `Action Rec` (`Retain`, `Adapt`, `Regen`, `Needs Review`)
  9. `Requirements` (Count & Chips)
  10. `Actions` (Edit, Move, Delete)

---

## 5. Non-Color Accessibility System (NFR-07 Strict Compliance)

To satisfy **NFR-07** ("Avoid using color alone to communicate status") and WCAG 2.1 AA:
Every status and indicator is expressed through **three simultaneous channels**:
1. **Geometric Shape / Icon:** A distinct SVG symbol.
2. **Explicit Text Label:** Readable plain text (never just an icon or dot).
3. **High-Contrast Color Token:** Meets contrast ratio ≥ 4.5:1 against the card background.

### 5.1 Production Status Matrix

| Status Key | Icon Symbol | Rendered Label | Background Token | Text Token | Border Token | ARIA Semantics |
|------------|-------------|----------------|------------------|------------|--------------|----------------|
| `draft` | ○ (Circle Outline) | `[○ Draft]` | `bg-neutral-100` | `text-neutral-800` | `border-neutral-400` | `aria-label="Status: Draft"` |
| `in_progress` | ⏳ (Hourglass / Clock) | `[⏳ In Progress]` | `bg-amber-50` | `text-amber-900` | `border-amber-400` | `aria-label="Status: In Progress"` |
| `in_review` | 👁️ (Open Eye) | `[👁️ In Review]` | `bg-sky-50` | `text-sky-900` | `border-sky-400` | `aria-label="Status: In Review"` |
| `approved` | ✓ (Check Circle) | `[✓ Approved]` | `bg-emerald-50` | `text-emerald-900` | `border-emerald-500` | `aria-label="Status: Approved"` |

### 5.2 Direction Version Synchronization Matrix (NFR-07)

A shot can be aligned with the active project direction or authored under a prior direction version:

| Synchronization State | Visual Indicator | Icon | Text Label | Explanation & Context |
|-----------------------|------------------|------|------------|-----------------------|
| **Aligned** | `[✓ Dir: v1.0 Aligned]` | Checkmark inside shield | `Dir: v{N} Aligned` | Shot was authored or reviewed against current active direction. |
| **Outdated / Desynced** | `[⚠️ Dir: v1.0 (Active: v2.0)]` | Warning Triangle | `Dir: v{N} (Outdated)` | Active project direction has evolved. Shot requires change impact assessment. |
| **Unlinked / Draft** | `[? Dir: Unassigned]` | Question mark | `Dir: Unassigned` | Shot has not been evaluated against any direction version. |

### 5.3 Change Impact Recommendation Matrix (FR-08, FR-09)

When direction changes are evaluated, recommendations must be explicitly labeled:

| Action Category | Icon | Visible Text | Human Review Badge | Interaction Required |
|-----------------|------|--------------|---------------------|----------------------|
| **Retain** | 🛡️ Shield Check | `[🛡️ Retain]` | `AI Suggested: Retain` | Human confirmation button: `[Confirm Retain]` |
| **Adapt** | ✏️ Pencil/Edit | `[✏️ Adapt]` | `AI Suggested: Adapt` | Human action: `[Review & Adapt Prompt]` |
| **Regenerate** | 🔄 Circular Arrow | `[🔄 Regenerate]` | `AI Suggested: Regenerate` | Human action: `[Approve Regeneration]` |
| **Needs Review** | ❓ Alert Circle | `[❓ Needs Review]` | `Action Required: Review` | Human action: `[Inspect Dependencies]` |

> **Crucial Rule:** The card displays both the **AI Suggestion** (e.g. `Suggested: Adapt`) AND the **Human Approval State** (e.g. `Pending Human Approval` or `Human Approved`). They are never merged into a single ambiguous flag (FR-09).

---

## 6. Interaction & Operational Workflows

### 6.1 Shot Reordering Workflow (FR-04, FR-11)
- **User Goal:** Change the narrative flow and sequential delivery order of shots.
- **Mouse / Touch:**
  - Dragging the grip handle (`::`) reorders cards in the grid.
  - Drop placeholder highlights destination slot with a dashed boundary.
- **Keyboard / Screen Reader Accessible Alternative:**
  - Every card includes `[↑ Move Up]` and `[↓ Move Down]` accessible button controls.
  - Keyboard shortcut: `Alt + ArrowUp` / `Alt + ArrowDown` when card is focused.
  - Immediate update of sequence numbers (`#01`, `#02`, etc.).
  - An `aria-live="polite"` region announces: `"Shot SHT-002 moved to position 1"`.
- **Backend Sync:** PATCH payload to `/api/v1/projects/:projectId/shots/reorder` with updated order array. Rollback with toast notice if network fails.

### 6.2 Shot Creation Workflow (FR-01, FR-04)
- **Trigger:** Clicking `+ Add Shot` in toolbar or empty state.
- **Modal / Slide-over Drawer:**
  - Fields:
    1. Title (Required, string, 1-80 chars).
    2. Description (Optional, multiline textarea).
    3. Initial Status (Default: `draft`).
    4. Relevant Requirements (Multi-select dropdown populated from extracted PRD requirements).
    5. Prompt / Instructions (Optional multiline code textarea).
    6. Direction Version (Defaults to current active project direction, e.g. `v1.0`).
  - Validation: Title cannot be empty. If requirements are not linked, warning displays: `"Warning: Creating a shot without requirements increases uncertainty in impact analysis."`
  - On Submit: Stable ID generated (`SHT-{NNN}`), appended to end of sequence.

### 6.3 Shot Editing & Version Creation Workflow (FR-06, FR-10)
- **Trigger:** Clicking card `Edit Details` or `New Version`.
- **Slide-Over Drawer (Right 480px):**
  - **Tab 1: Current Active Version (v2):** Edit title, description, prompt, reference links.
  - **Tab 2: Version History:** List of prior versions:
    - `v1.0 (Authored 2026-10-09 under Dir v1.0)` -> Read-only inspection with "Restore as new revision" action.
  - **Rule (FR-06, NFR-02):** Editing prompt or description allows saving as:
    1. `Update Current Version (Minor patch)`
    2. `Create New Shot Version (v3)` (Preserves prior version intact).

### 6.4 Shot Deletion Workflow (FR-04)
- **Trigger:** Clicking `Delete Shot` in card action menu.
- **Destructive Confirmation Dialog:**
  - Title: `Delete Shot SHT-001?`
  - Body: `"This action will remove shot '#01: Opening Neon Street' from the storyboard sequence. All associated prompt versions and asset links will be archived. Remaining shots will be re-sequenced automatically."`
  - Confirm Button: `Delete Shot` (Destructive red token).
  - Cancel Button: `Cancel` (Neutral ghost).

---

## 7. Reusable Component Inventory & Props Specification

To ensure frictionless frontend implementation in React/Tailwind, the following component interfaces are defined.

### 7.1 Component: `ShotStatusBadge`
```typescript
interface ShotStatusBadgeProps {
  status: 'draft' | 'in_progress' | 'in_review' | 'approved';
  size?: 'sm' | 'md';
}
```
- **Renders:** Dual-coded badge containing SVG icon + localized text label.
- **Styling:** Pill shape (`rounded-full px-2.5 py-0.5 text-xs font-medium border`).

### 7.2 Component: `DirectionVersionBadge`
```typescript
interface DirectionVersionBadgeProps {
  shotDirectionVersionId: string;      // e.g. "v1.0"
  activeProjectDirectionVersionId: string; // e.g. "v2.0"
  showDetailedComparison?: boolean;
}
```
- **Renders:**
  - If equal: Green/neutral pill `[✓ Dir: v1.0 Aligned]`
  - If not equal: Amber pill `[⚠️ Dir: v1.0 (Active: v2.0)]` with tooltip indicating desynchronization.

### 7.3 Component: `ShotCard`
```typescript
interface ShotCardProps {
  shot: {
    id: string;                      // Stable ID e.g. "SHT-001"
    sequenceOrder: number;           // 1-indexed sequence order
    title: string;
    description?: string;
    status: 'draft' | 'in_progress' | 'in_review' | 'approved';
    currentVersionNumber: number;    // e.g. 1
    directionVersionId: string;      // e.g. "v1.0"
    thumbnailUrl?: string | null;
    prompt?: string;
    requirementIds: string[];        // e.g. ["REQ-01", "REQ-04"]
    dependencies?: string[];         // e.g. ["SHT-000"]
    reviewRecommendation?: {
      action: 'retain' | 'adapt' | 'regenerate' | 'needs_review';
      reason: string;
      isHumanApproved: boolean;
    } | null;
  };
  activeDirectionVersionId: string;
  onEdit: (shotId: string) => void;
  onNewVersion: (shotId: string) => void;
  onMoveUp: (shotId: string) => void;
  onMoveDown: (shotId: string) => void;
  onDelete: (shotId: string) => void;
  isFirst: boolean;
  isLast: boolean;
}
```

### 7.4 Component: `OutstandingReviewBanner`
```typescript
interface OutstandingReviewBannerProps {
  action: 'retain' | 'adapt' | 'regenerate' | 'needs_review';
  reason: string;
  isHumanApproved: boolean;
  onApproveAction: () => void;
  onOverrideAction: () => void;
}
```
- **Renders:** High-contrast notification strip across top of card with explicit recommendation and reason excerpt.

---

## 8. Design Tokens Specification

The design utilizes a restrained, high-contrast, professional palette avoiding muddy gradients and artificial translucency.

### 8.1 Color Palette Tokens

```css
/* Core Neutrals */
--color-bg-canvas: #f8fafc;        /* slate-50 */
--color-bg-surface: #ffffff;       /* pure white */
--color-bg-subtle: #f1f5f9;        /* slate-100 */
--color-border-subtle: #e2e8f0;    /* slate-200 */
--color-border-strong: #94a3b8;    /* slate-400 */
--color-text-main: #0f172a;        /* slate-900 (High contrast) */
--color-text-muted: #475569;       /* slate-600 */
--color-text-caption: #64748b;     /* slate-500 */

/* Semantic Status Tokens (Non-Color Compliant Pairings) */
/* Draft (Neutral) */
--status-draft-bg: #f1f5f9;
--status-draft-text: #1e293b;
--status-draft-border: #94a3b8;

/* In Progress (Amber) */
--status-progress-bg: #fef3c7;
--status-progress-text: #78350f;
--status-progress-border: #f59e0b;

/* In Review (Sky Blue) */
--status-review-bg: #e0f2fe;
--status-review-text: #0369a1;
--status-review-border: #38bdf8;

/* Approved (Emerald Green) */
--status-approved-bg: #d1fae5;
--status-approved-text: #065f46;
--status-approved-border: #10b981;

/* Recommendation Alert Tokens */
/* Retain (Blue-Slate) */
--rec-retain-bg: #f0fdf4;
--rec-retain-text: #166534;
--rec-retain-border: #86efac;

/* Adapt (Orange) */
--rec-adapt-bg: #fff7ed;
--rec-adapt-text: #9a3412;
--rec-adapt-border: #fdba74;

/* Regenerate (Rose) */
--rec-regen-bg: #fff1f2;
--rec-regen-text: #9f1239;
--rec-regen-border: #fda4af;

/* Needs Review (Purple) */
--rec-needsreview-bg: #faf5ff;
--rec-needsreview-text: #581c87;
--rec-needsreview-border: #d8b4fe;
```

### 8.2 Typography Scale
- **H1 (Page Titles):** `1.5rem (24px)`, Line-height `2rem (32px)`, Weight `600`.
- **H2 (Section Headers):** `1.25rem (20px)`, Line-height `1.75rem (28px)`, Weight `600`.
- **H3 (Card Titles):** `1.0rem (16px)`, Line-height `1.5rem (24px)`, Weight `600`.
- **Body Regular:** `0.875rem (14px)`, Line-height `1.25rem (20px)`, Weight `400`.
- **Caption / Meta:** `0.75rem (12px)`, Line-height `1.0rem (16px)`, Weight `500`.
- **Monospace (IDs, Prompts):** `0.8125rem (13px)`, Font Family `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas`.

### 8.3 Spacing & Radius Scales
- **Spacing Scale:** `4px (1)`, `8px (2)`, `12px (3)`, `16px (4)`, `24px (6)`, `32px (8)`.
- **Radius Scale:** Cards: `rounded-lg (8px)`; Badges/Pills: `rounded-full (9999px)`; Buttons: `rounded-md (6px)`.
- **Elevation / Shadows:** Restrained micro-shadows only (`shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05)`). No intense glowing blur shadows.

---

## 9. Responsive Breakpoints & Viewport Rules

| Viewport | Width | Layout Rules |
|----------|-------|--------------|
| **Desktop Wide** | ≥ 1280px | 3-column Shot Card grid; Slide-over drawer opens as non-modal right panel (380px) preserving board visibility. |
| **Desktop Standard** | 1024px – 1279px | 2-column Shot Card grid; Project overview stacks 8 / 4 cols. |
| **Tablet** | 768px – 1023px | 2-column Shot Card grid; Filters collapse into filter drawer; Slide-over drawer acts as modal overlay with backdrop. |
| **Mobile** | < 768px | Single column stacked cards; List View converted to mobile card list; Reordering handles use explicit `[Move Up]` / `[Move Down]` buttons only (drag disabled on touch). |

---

## 10. Empty, Loading, and Error States

### 10.1 Shot Board Empty State
- **Condition:** Project exists with 0 shots.
- **Illustration:** Clean line-art film slate / storyboard frame icon (no 3D blobs).
- **Heading:** `No shots created yet`
- **Body:** `Start your storyboard by adding shots manually or extracting them automatically from your project brief.`
- **Action Buttons:**
  - Primary: `[+ Create First Shot]`
  - Secondary: `[Extract Shots from Brief]` (Links to `/brief`).

### 10.2 Shot Card Skeleton Loading State
- **Structure:** 6 skeleton cards mimicking the exact 16:9 thumbnail and text layout.
- **Animation:** Subtle pulse (`animate-pulse bg-neutral-200`), not shimmering gradient sweep.

### 10.3 Error State (API Failure or Desync)
- **Banner:** Persistent alert banner at top of board:
  `⚠️ Unable to sync board changes with server. [Retry Connection]`
- **Local Persistence:** Changes kept in local UI state without discarding user edits until retry succeeds.

---

## 11. Frontend Implementation Checklist & Test Hooks (`data-testid`)

To facilitate smooth testing and verification by `tester` (Task T-008):

| Element | `data-testid` | Description |
|---------|---------------|-------------|
| Project Overview Container | `project-overview-container` | Main container for overview screen |
| Active Direction Header Badge | `active-direction-badge` | Displays active direction version string |
| Shot Board Toolbar | `shot-board-toolbar` | Toolbar container |
| View Toggle (Cards) | `btn-view-cards` | Switch to storyboard card grid |
| View Toggle (List) | `btn-view-list` | Switch to list view |
| Add Shot Button | `btn-add-shot` | Trigger for shot creation drawer/modal |
| Shot Card Item | `shot-card-{shotId}` | Individual shot card container (e.g. `shot-card-SHT-001`) |
| Shot Sequence Badge | `shot-seq-{shotId}` | Displays `#01` sequence number |
| Shot Stable ID | `shot-id-{shotId}` | Monospace ID container |
| Shot Thumbnail Container | `shot-thumbnail-{shotId}` | Container for 16:9 image or fallback |
| Shot Status Badge | `shot-status-badge-{shotId}` | Dual-coded production status badge |
| Shot Direction Badge | `shot-direction-badge-{shotId}` | Direction version & sync state badge |
| Shot Version Tag | `shot-version-tag-{shotId}` | Shot revision number (e.g. `v1`) |
| Outstanding Review Alert | `shot-review-alert-{shotId}` | Review recommendation banner on card |
| Shot Move Up Button | `btn-move-up-{shotId}` | Accessible reorder up button |
| Shot Move Down Button | `btn-move-down-{shotId}` | Accessible reorder down button |
| Shot Detail Drawer | `shot-detail-drawer` | Drawer container for editing shot details |

---

## 12. Verification Against PRD & Acceptance Criteria

1. **All PRD §9.2 Fields Covered:**
   - Shot ID & title (`SHT-001`, title header) — **Covered (§4.3 items 2, 3)**
   - Current thumbnail if available (`16:9 preview + fallback`) — **Covered (§4.3 item 5)**
   - Current status (`Draft`, `In Progress`, etc.) — **Covered (§4.3 item 7, §5.1)**
   - Relevant requirements (`REQ-xx` chips) — **Covered (§4.3 item 11)**
   - Active prompt / asset version (`Shot v2`, prompt snippet, asset pill) — **Covered (§4.3 items 6, 9, 12)**
   - Direction version (`Dir v1.0 Aligned` vs `Outdated`) — **Covered (§4.3 item 8, §5.2)**
   - Outstanding review indicator (`Needs Review` banner with action recommendation) — **Covered (§4.3 item 10, §5.3)**
2. **Accessibility (NFR-07):** Status and direction version are never distinguished by color alone; every indicator combines an SVG icon, explicit text string, and WCAG AA contrast.
3. **Traceability (FR-06, NFR-07):** Both active direction version and shot version numbers remain persistently displayed.
