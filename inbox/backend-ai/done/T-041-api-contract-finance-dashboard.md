# Task T-041: API Contract for Financial Summary, Trends & Data Health

- **Owner**: `backend-ai`
- **Milestone**: M4 — Finance Engine, Dashboard & Data Health
- **Size**: Standard
- **Repo**: `office-hub`

## 1. Objective
Menyusun kontrak spesifikasi API RESTful komprehensif untuk modul Finance & Dashboard mencakup endpoint `/dashboard/summary`, `/dashboard/trends`, dan `/dashboard/data-health` sesuai aturan matematis TRD §6.

## 2. Context & Reference Files
- `project-map/trd.md` (§6 Calculation Spec): Rumus deterministik `net_sales`, `costed_net_sales`, `cogs`, `gross_profit`, `gross_margin`, `cost_coverage`, `is_estimate`, `cash_received`, `recorded_fees`.
- `contracts/erd.md`: Skema tabel `transactions`, `transaction_items`, `sales_returns`, `products`, `customers`.
- `contracts/api-transactions-customers.md`: Standar format angka string desimal `NUMERIC(15,2)`.

## 3. Expected Output
Dokumen `contracts/api-finance-dashboard.md` yang memuat:
1. Endpoint `GET /api/v1/businesses/:business_id/dashboard/summary?from=YYYY-MM-DD&to=YYYY-MM-DD`:
   - Agregasi finansial periode: `net_sales`, `costed_net_sales`, `cogs`, `gross_profit`, `gross_margin_pct` (null jika 0), `cost_coverage_pct`, `is_estimate`, `missing_cost_items_count`, `cash_received`, `total_transactions`, `average_ticket_size`.
2. Endpoint `GET /api/v1/businesses/:business_id/dashboard/trends?from=YYYY-MM-DD&to=YYYY-MM-DD&interval=day`:
   - Array per hari: `date`, `net_sales`, `gross_profit`, `transaction_count`.
3. Endpoint `GET /api/v1/businesses/:business_id/dashboard/data-health`:
   - Diagnostik kesehatan data: `cost_coverage_pct`, `missing_cost_product_count`, `unlinked_customer_tx_count`, `unlinked_customer_tx_pct`, `low_stock_products_count`, `health_score` (0-100), `recommendations` (aksi perbaikan data prioritas).
4. Error handling & Validasi query parameter (`from`, `to`).
5. Perlakuan status transaksi: transaksi berstatus `cancelled` WAJIB dikecualikan dari seluruh perhitungan periodik.

## 4. Definition of Done
Dokumen `contracts/api-finance-dashboard.md` selesai ditulis dan dilaporkan di `inbox/leader/T-041-report.md`.
