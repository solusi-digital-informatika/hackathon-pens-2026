# Task: T-022 Implement Backend Products, Cost Versions, Calculator & Migrations

## 1. Objective
Implement the backend modules for Products, Cost Versions (append-only), Price History (append-only), and the Pricing Calculator in `office-hub/backend` according to `contracts/api-products.md`, `contracts/erd.md`, and TRD TD-07, TD-08, TD-12.

## 2. Context and Reference Files
- Repo local path: `C:/Project/ai-office/office-hub/backend`
- Remote: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
- `contracts/api-products.md`: Authoritative API contract for products and calculator.
- `contracts/erd.md`: §3 (Diagram B: `products`, `cost_versions`, `price_history`), §7 (Constraints: composite unique `(business_id, sku)`, composite FKs).
- `project-map/trd.md`: §4 (`product`), §5 (TD-07 Cost Versioning, TD-08 Price History, TD-12 Pricing formulas).

## 3. Expected Output
In repository `C:/Project/ai-office/office-hub/backend`:
1. **Migration**:
   - `migrations/000003_create_products_and_pricing_tables.up.sql` and `down.sql`:
     - `products`: `id UUID PK`, `business_id UUID FK`, `sku text`, `name text`, `category text`, `unit text`, `current_selling_price numeric(15,2)`, `current_cost_price numeric(15,2)`, `min_stock numeric(12,3)`, `current_stock numeric(12,3)`, `status text check in ('active', 'archived')`, `created_at`, `updated_at`.
     - Unique constraint: `(business_id, sku)`.
     - `cost_versions`: `id UUID PK`, `business_id UUID FK`, `product_id UUID FK`, `cost_price numeric(15,2)`, `reason text`, `effective_from timestamptz`, `created_at timestamptz`.
     - `price_history`: `id UUID PK`, `business_id UUID FK`, `product_id UUID FK`, `selling_price numeric(15,2)`, `reason text`, `effective_from timestamptz`, `created_at timestamptz`.
2. **Modul `internal/product/`**:
   - `model.go`, `repository.go`, `service.go`, `handler.go`, `calculator.go`.
   - Pricing calculator: implements margin and markup calculation formulas with warning detection.
   - Enforce composite unique SKU per business and atomic insertion of initial cost/price versions on create.
3. **Route Wiring**:
   - Wire all routes from `contracts/api-products.md` in `internal/platform/httpserver/server.go` under `scopedBiz`.
4. **Unit & Isolation Tests**:
   - Calculator formula tests (margin, markup, warning triggers).
   - Product repository & service unit tests.
   - Cross-tenant product isolation tests.

## 4. Constraints
- Strict tenant isolation: Caller can only view/mutate products belonging to their authenticated business.
- `go vet ./...`, `go test ./...`, and `go build ./...` must pass cleanly.
- Conventional Commits: `feat(product): implement products catalog, cost versions, price history, and calculator`.
- Push to `origin main`.

## 5. Definition of Done
- All migrations created and tested.
- Unit and tenant tests pass.
- Commit hash reported to `inbox/leader/T-022-report.md`.
- Update `board/board.md` for T-022 to `done`.
- Move this task to `inbox/backend-ai/done/`.
