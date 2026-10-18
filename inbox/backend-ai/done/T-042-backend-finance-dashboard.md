# Task T-042: Implement Backend Finance Engine, Summary Aggregations & Data Health

- **Owner**: `backend-ai`
- **Milestone**: M4 — Finance Engine, Dashboard & Data Health
- **Size**: Standard
- **Repo**: `backend`

## 1. Objective
Mengimplementasikan modul `internal/finance` di backend Go untuk menghitung ringkasan finansial, grafik tren harian, dan diagnostik kesehatan data (*data health*) sesuai kontrak `contracts/api-finance-dashboard.md` dan spesifikasi perhitungan matematis TRD §6.

## 2. Context & Reference Files
- `contracts/api-finance-dashboard.md`: Kontrak endpoint `/dashboard/summary`, `/dashboard/trends`, `/dashboard/data-health`.
- `project-map/trd.md` (§6 Calculation Spec): Rumus deterministik `net_sales`, `costed_net_sales`, `cogs`, `gross_profit`, `gross_margin`, `cost_coverage`, `is_estimate`, `cash_received`, `recorded_fees`.
- `Golden Test Cases` (TRD §6.4): Wajib diimplementasikan sebagai unit test di `internal/finance/finance_test.go`.

## 3. Scope of Implementation
1. `internal/finance/model.go`: Struct data untuk `DashboardSummary`, `DailyTrendPoint`, `DashboardTrends`, `DataHealthReport`, `HealthMetrics`, `ActionItem`.
2. `internal/finance/calculator.go`: Pure calculation engine tanpa ketergantungan DB atau HTTP untuk mengagregasi item transaksi menjadi ringkasan finansial yang akurat.
3. `internal/finance/repository.go`: Query ke database untuk mengambil data transaksi (kecuali `cancelled`), item, retur, dan inventaris produk dalam rentang `from` - `to`.
4. `internal/finance/service.go`: Business logic untuk summary, trends, dan data health scoring.
5. `internal/finance/handler.go`: HTTP handler Gin untuk ketiga rute dashboard.
6. `internal/platform/httpserver/server.go` & `cmd/api/main.go`: Wire-up modul finance ke HTTP server.
7. `internal/finance/finance_test.go`: Unit test mencakup seluruh Golden Test Cases TRD §6.

## 4. Constraints
- Pure calculation functions stay decoupled from DB and HTTP.
- Transaksi `cancelled` tidak boleh masuk ke agregasi finansial.
- Hindari division by zero: jika `costed_net_sales == 0`, `gross_margin_pct` bernilai `nil`/`null`.
- Push commit ke branch `main` repositori `teman-umkm-bakcend` dengan Conventional Commits.
