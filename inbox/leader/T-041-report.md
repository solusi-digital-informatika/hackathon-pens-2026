# Laporan Penyelesaian T-041: API Contract for Financial Summary, Trends & Data Health

- **Task ID**: T-041
- **Assignee**: `backend-ai`
- **Deliverable**: `contracts/api-finance-dashboard.md`
- **Status**: DONE / APPROVED

Telah selesai disusun kontrak spesifikasi API RESTful untuk Finance & Dashboard Engine, mencakup:
1. Endpoint `GET /api/v1/businesses/:business_id/dashboard/summary` yang mengembalikan agregasi deterministik TRD §6:
   - `net_sales`, `costed_net_sales`, `cogs`, `gross_profit`, `gross_margin_pct`, `cost_coverage_pct`, `is_estimate`, `missing_cost_items_count`, `cash_received`, `recorded_fees`, `total_transactions`, `average_ticket_size`.
2. Endpoint `GET /api/v1/businesses/:business_id/dashboard/trends` yang mengembalikan data series harian per tanggal: omzet, laba kotor, HPP, dan frekuensi transaksi.
3. Endpoint `GET /api/v1/businesses/:business_id/dashboard/data-health` yang mengevaluasi skor kesehatan data pembukuan usaha (persentase produk ber-HPP, rasio transaksi pelanggan terhubung, produk stok menipis, dan rekomendasi aksi).
4. Aturan pengabaian transaksi `cancelled` dari agregasi moneter serta penanganan desimal `NUMERIC(15,2)` yang presisi.
