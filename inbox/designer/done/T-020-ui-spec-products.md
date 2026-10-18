# Task: T-020 UI Spec for Products, Cost Versions & Pricing Calculator

## 1. Objective
Author `contracts/ui-products.md` specifying the UX, page layout, components, states, responsive views, motion, and accessibility for:
1. Products Catalog & Table View (`/products`)
2. Add / Edit Product Modal or Drawer (with SKU, Name, Category, Unit, Selling Price, Cost Price, Stock)
3. Cost Version & Price History Modal/Drawer (showing timeline of changes with reason)
4. Interactive Pricing Calculator (target margin / markup simulation, showing margin warnings if < 15%)

## 2. Context and Reference Files
- `contracts/design-direction.md`: Clean Pragmatic Craftsman, Emerald Forest `#0D6E64`, Plus Jakarta Sans & Inter, WCAG AA.
- `contracts/design-tokens.md`: CSS tokens for spacing, elevation, type scale, colors, Rupiah `id-ID` formatting.
- `project-map/prd.md`: §5.2 (Products, Cost Versions, Price History, Pricing Calculator), §6.2 (Add Product Flow).
- `project-map/trd.md`: §5 (TD-07 Cost Versioning append-only, TD-08 Price History, TD-12 Pricing formulas).
- Designer SOUL: Specs format (`contracts/ui-<feature>.md`), Anti-Slop Rules (no resting shadows, no decorative AI gradients, visible labels, keyboard accessible, touch targets ≥ 44px).

## 3. Expected Output
`contracts/ui-products.md` containing:
- Page specs for Products list, filters (category, search, stock level), pagination.
- Modal specs for Add/Edit product with currency inputs (`formatRupiah`).
- Cost & Selling Price timeline view showing `effective_from`, previous price, new price, reason.
- Pricing Calculator widget/modal specs with target markup % vs target margin % toggle, instant calculation, and visual warning when margin is unhealthy (< 15%).
- Responsive breakdown at 390 / 768 / 1440.
- Motion and accessibility specs.

## 4. Constraints
- Strict Anti-Slop: NO decorative gradients, NO resting card shadows, clear and concise Bahasa Indonesia copy.
- Save output to `contracts/ui-products.md`.

## 5. Definition of Done
- `contracts/ui-products.md` created in `contracts/`.
- Update `board/board.md` for T-020 to `done`.
- Send report to `inbox/leader/T-020-report.md` and move task to `inbox/designer/done/`.
