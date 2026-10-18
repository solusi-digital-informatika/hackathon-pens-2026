# Task: T-001 Design Direction and Tokens

## 1. Objective
Create `contracts/design-direction.md` and `contracts/design-tokens.md` for Teman UMKM based on PRD and TRD.

## 2. Context and Reference Files
- `project-map/prd.md`: §1 (Product Overview), §2 (Target Users: UMKM Owner, Operational Staff), §7 (Information Architecture, Dashboard Sections), §8.5 (Usability: Indonesian language, Rupiah format)
- `project-map/trd.md`: §0 (How agents use TRD), §3 (Frontend Stack), §5 (TD-13 API & UI conventions)
- Designer SOUL: Art Direction, Anti-Slop Rules, Design Tokens spec

## 3. Expected Output
1. `contracts/design-direction.md`:
   - Audience and context: Indonesian UMKM owners on desktop and mobile web.
   - Direction: consider 3 options in one line each, pick one, explain why it fits Teman UMKM.
   - Typography: max 2 families (e.g., Plus Jakarta Sans / Inter), scale, weights.
   - Palette: semantic roles, ONE accent color (e.g., emerald / forest green symbolizing growth and Indonesian business trust), neutrals tinted toward the accent, WCAG AA compliant.
   - Imagery: clean, purpose-driven UI elements, no generic stock or fake dashboards.
   - Signature moment: clear financial clarity view (e.g., deterministic profit breakdown cards with cost coverage badges).
   - Density: compact dashboard and transaction tables; calm, clear onboarding.
2. `contracts/design-tokens.md`:
   - CSS variables definition:
     - Spacing 4-pt scale (4, 8, 12, 16, 24, 32, 48, 64)
     - Radius scale
     - Type scale (sizes, line heights, font weights)
     - Semantic colors (background, surface, text, muted, border, primary accent, success, warning, error, info)
     - Motion tokens (micro 100-150ms, standard 200-300ms, page ≤ 400ms, easing curves)
     - z-index scale

## 4. Constraints
- Anti-slop: NO decorative gradients, NO resting shadows (surface/border contrast only), NO eyebrow labels everywhere, max 1 accent color, max 2 font families.
- All interface copy in natural, professional Bahasa Indonesia.
- Currency formatting standard: Indonesian Rupiah (`Rp ...`).

## 5. Definition of Done
- `contracts/design-direction.md` and `contracts/design-tokens.md` created in `contracts/`.
- Update your row status in `board/board.md` to `done`.
- Send completion report to `inbox/leader/T-001-report.md` and move this task to `inbox/designer/done/`.
