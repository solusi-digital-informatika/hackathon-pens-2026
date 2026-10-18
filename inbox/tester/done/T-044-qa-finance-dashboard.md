# Task: T-044 QA Verification for Financial Calculations, Aggregations & Data Health

## 1. Objective
Perform independent end-to-end QA audit and verification for Milestone M4 (Finance Engine, Dashboard & Data Health) across `office-hub/backend` (commit `fb8a4ad`) and `office-hub/frontend` (commit `4ed545b`).

## 2. Context & Reference Files
- Specifications:
  - `project-map/trd.md` (§6 Calculation Spec, §6.4 Golden Test Cases, §6.5 Invariants).
  - `contracts/api-finance-dashboard.md`: API specifications for `/dashboard/summary`, `/dashboard/trends`, `/dashboard/data-health`.
  - `contracts/ui-finance-dashboard.md`: UI specifications for Clarity Card, Metric Cards, Trends, and Data Health diagnostics.
- Code under test:
  - Backend: `C:/Project/ai-office/office-hub/backend` (`internal/finance/...`, routes in `internal/platform/httpserver/server.go`).
  - Frontend: `C:/Project/ai-office/office-hub/frontend` (`src/modules/dashboard/...`).

## 3. Scope of Testing & Verification
1. **Mathematical Accuracy (TRD §6)**:
   - Net Sales formula: `net_sales = gross_sales - discount_amount - refund_amount`.
   - COGS formula: `cogs = sum(qty * snapshot_cost)` (only for items where HPP/cost is recorded).
   - Gross Profit formula: `gross_profit = costed_net_sales - cogs`.
   - Cost Coverage: `cost_coverage = (costed_sales_count / total_sales_count) * 100`.
   - Cash Received: Cash/QRIS payments collected minus cash refunds.
2. **Golden Test Cases Verification (§6.4)**:
   - Case 1: 100% known cost (All items have HPP) -> Laba Kotor Pasti.
   - Case 2: Partial cost (Some items missing HPP) -> Laba Kotor Estimasi + warning banner + item count without HPP.
   - Case 3: Complete return / discount exceeds gross.
   - Case 4: Cancelled transactions -> Must be strictly EXCLUDED from all aggregations.
3. **Data Health Diagnostic Engine**:
   - Health score formula: weighted average (Product HPP 50%, Customer Link 25%, Cash Reconciled 25%).
   - Actionable recommendations triggered when thresholds breached.
4. **Automated Test Suites**:
   - Backend Go test suite: `go test -v ./...` in `office-hub/backend`.
   - Frontend Vitest suite: `npm test -- --run` in `office-hub/frontend`.
   - Frontend build: `npm run build` in `office-hub/frontend`.

## 4. Expected Output
- File: `inbox/leader/T-044-report.md`.
- Content: Executive summary, test matrices, mathematical calculation verification, automated test outputs, edge case analysis, and definitive verdict (`APPROVE` or `REJECT`).
- Move this task to `inbox/tester/done/T-044-qa-finance-dashboard.md`.
- Update `board/board.md` T-044 status to `done`.
