# Task: T-021 API Contract for Products, Cost Versions & Pricing Calculator

## 1. Objective
Author `contracts/api-products.md` detailing the HTTP API contract for Products management, Cost Versions, Price History, and the Pricing Calculator.

## 2. Context and Reference Files
- `project-map/prd.md`: §5.2 (Products, Cost Versions, Price History, Pricing Calculator).
- `project-map/trd.md`:
  - §4 (Backend Modules: `product`, `platform`).
  - §5 (Key Decisions: TD-04 UUIDv7, TD-05 Tenant Isolation via path-scoped `/businesses/{bid}/...`, TD-07 Append-only Cost Versions, TD-08 Price History, TD-12 Pricing Calculator formulas, TD-13 API conventions).
  - §9 (API Inventory: `products`, `cost_versions`, `price_history`, `pricing-calculator`).
- `contracts/erd.md`:
  - §3 (Diagram B: `products`, `cost_versions`, `price_history`).
  - §7 (Constraints: composite unique `(business_id, sku)`, composite FKs).
  - §8 (Precision: `NUMERIC(15,2)` for currency, `NUMERIC(12,3)` for stock).
- `contracts/api-auth.md`: Conventions (`/api/v1`, error responses `{error: {code, message}}`, decimal strings in JSON).

## 3. Expected Output
`contracts/api-products.md` detailing:
1. Endpoints:
   - `GET /api/v1/businesses/{business_id}/products`: query params `search`, `category`, `status`, `page`, `page_size`. Returns paginated product list with `items`, `total_items`, `page`, `page_size`.
   - `POST /api/v1/businesses/{business_id}/products`: body `{sku, name, category, unit, current_selling_price, current_cost_price, min_stock, current_stock}`. Creates product + initial cost version + initial price history + audit log atomically.
   - `GET /api/v1/businesses/{business_id}/products/{product_id}`: returns full product details.
   - `PATCH /api/v1/businesses/{business_id}/products/{product_id}`: body `{name?, category?, unit?, min_stock?, current_stock?, status?, new_selling_price?, new_cost_price?, change_reason?}`. If new cost or selling price provided, appends new cost version or price history record.
   - `GET /api/v1/businesses/{business_id}/products/{product_id}/cost-versions`: returns list of cost versions ordered by `effective_from DESC`.
   - `GET /api/v1/businesses/{business_id}/products/{product_id}/price-history`: returns list of price history ordered by `effective_from DESC`.
   - `POST /api/v1/businesses/{business_id}/pricing-calculator`: body `{cost_price, method: 'margin' | 'markup', target_percentage}`. Returns `{recommended_selling_price, profit_per_unit, margin_percentage, markup_percentage, warnings: []}`.
2. Decimal Precision & Validation rules:
   - Currency formatted as decimal strings (`"15000.00"`).
   - Stock quantities formatted as strings (`"10.000"`).
   - Selling price cannot be negative.
3. Verification matrix with test cases for QA/Tester.

## 4. Constraints
- Strict tenant scoping on all routes (`/api/v1/businesses/{business_id}/...`).
- Append-only immutability for `cost_versions` and `price_history` (no DELETE or UPDATE).

## 5. Definition of Done
- `contracts/api-products.md` created in `contracts/`.
- Update your row in `board/board.md` for T-021 to `done`.
- Send report to `inbox/leader/T-021-report.md` and move task to `inbox/backend-ai/done/`.
