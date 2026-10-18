# Laporan Eksekusi T-032: Backend Customers, Transactions, Idempotency, and Stock Deductions

- **Task ID**: T-032
- **Assignee**: `backend-ai`
- **Pushed Commit Hash**: `ff5c612`
- **Status**: DONE / VERIFIED

## 1. Ringkasan Implementasi
Telah selesai diimplementasikan modul Customers dan Transactions backend dengan integrasi penuh ke database PostgreSQL, mencakup:
1. **Migrasi Database (`000004_create_customers_and_transactions_tables`)**:
   - Skema tabel `customers`: `id`, `business_id`, `display_name`, `phone`, `email`, `notes`, `status`, `created_at`, `updated_at`.
   - Skema tabel `transactions`: `id`, `business_id`, `customer_id`, `cashier_user_id`, `invoice_number`, `transaction_date`, `subtotal_amount`, `discount_amount`, `fee_amount`, `total_amount`, `total_cost_amount`, `payment_method`, `payment_status`, `amount_paid`, `change_amount`, `external_ref`, `status`, `notes`.
   - Skema tabel `transaction_items`: `id`, `transaction_id`, `product_id`, `quantity`, `unit_selling_price`, `recorded_unit_cost`, `subtotal_price`, `discount_amount`, `total_price`, `notes`.
   - Skema tabel `sales_returns`: `id`, `business_id`, `transaction_id`, `transaction_item_id`, `return_number`, `quantity`, `refund_amount`, `reason`, `processed_by_user_id`, `returned_on`, `created_at`.
   - Indeks unik per tenant untuk invoice, return number, dan idempotency key (`business_id`, `external_ref`).

2. **Modul Customer (`internal/customer/`)**:
   - Model, Repository, Service, dan Handler REST API.
   - Endpoint: `POST /customers`, `GET /customers`, `GET /customers/:id`, `PATCH /customers/:id`.
   - Tenant isolation ketat dan audit trail.

3. **Modul Transaction (`internal/transaction/`)**:
   - Pembuatan transaksi atomic dalam DB transaction:
     - Deduplikasi/Idempotensi via `Idempotency-Key` atau `external_ref`.
     - Validasi stok produk: menolak checkout jika stok kurang dari kuantitas beli.
     - Pengurangan stok seketika pada tabel `products`.
     - Snapshot HPP otomatis ke `transaction_items.recorded_unit_cost`.
     - Perhitungan subtotal, diskon, biaya admin, total bayar, kembalian, dan total HPP deterministik.
   - Endpoint: `POST /transactions`, `GET /transactions`, `GET /transactions/:id`, `POST /transactions/:id/cancel`, `POST /transactions/:id/returns`, `GET /transactions/:id/returns`.
   - Pembatalan transaksi (`/cancel`): memulihkan sisa stok barang yang belum diretur kembali ke inventaris.
   - Retur penjualan (`/returns`): validasi kuantitas retur tidak melebihi sisa pembelian, memulihkan kuantitas retur ke stok produk secara otomatis.

4. **Pengujian & Verifikasi**:
   - Unit test `customer_test.go`: CRUD customer & isolasi tenant lintas bisnis lulus.
   - Unit test `transaction_test.go`: helper desimal, formula finansial (subtotal, diskon, laba kotor, kembalian), dan perhitungan pemulihan stok retur lulus.
   - `go test -v ./...`: 100% PASS pada seluruh package backend.
   - `go build -o /dev/null ./cmd/api`: kompilasi biner sukses tanpa error/warning.
