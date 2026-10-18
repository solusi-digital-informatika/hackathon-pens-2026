# Task: T-010 UI Spec for Auth & Business Setup

## 1. Objective
Author `contracts/ui-auth-business.md` specifying the UX, page layout, components, states, responsive views, motion, and accessibility for:
1. User Registration (`/register`)
2. User Login (`/login`)
3. Business Setup / Onboarding Modal/Page (`/onboarding` or dialog after first login)
4. Business Profile Settings (`/settings`)

## 2. Context and Reference Files
- `contracts/design-direction.md`: Clean Pragmatic Craftsman, Emerald Forest `#0D6E64`, Plus Jakarta Sans & Inter, WCAG AA.
- `contracts/design-tokens.md`: CSS variables for spacing, elevation, type scale, colors.
- `project-map/prd.md`: §5.1 (Auth and Business Setup), §6.1 (First-Time Setup flow).
- `project-map/trd.md`: §5 (TD-06 Auth: email + password, generic auth error messages, session-expired state; TD-05 Tenant isolation).
- Designer SOUL: Specs format (`contracts/ui-<feature>.md`), Anti-Slop Rules (no resting shadows, no decorative AI gradients, visible labels, generic auth errors, session-expired state, touch targets ≥ 44px).

## 3. Expected Output
`contracts/ui-auth-business.md` containing:
- Page specs for Login, Register, Business Setup (Onboarding), and Business Settings.
- Required states: default, hover, focus, active, disabled, loading, empty, validation error, generic auth error.
- Responsive breakdown at 390 / 768 / 1440.
- Motion specs (`trigger -> property -> duration -> easing`).
- A11y and security UX: visible labels, show/hide password toggle, generic auth error message ("Email atau password salah", never revealing account existence), session-expired handling.

## 4. Constraints
- Strict Anti-Slop: NO decorative gradients, NO resting card shadows, plain professional Bahasa Indonesia copy.
- Do not paste long specs into chat. Save to `contracts/ui-auth-business.md`.

## 5. Definition of Done
- `contracts/ui-auth-business.md` created in `contracts/`.
- Update `board/board.md` for T-010 to `done`.
- Send completion report to `inbox/leader/T-010-report.md` and move this task to `inbox/designer/done/`.
