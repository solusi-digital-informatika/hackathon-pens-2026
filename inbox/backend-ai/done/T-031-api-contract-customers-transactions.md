# Task: T-031 API Contract for Customers, Transactions, Idempotency & Stock Management

## Objective
Rancang kontrak API (REST API Contract) yang komprehensif untuk modul Pelanggan, Transaksi Penjualan, Idempotency, Retur, dan Manajemen Pengurangan/Pemulihan Stok Otomatis.

## Konteks & Pedoman
- `contracts/erd.md`: Bagian Diagram B (Transactions) dan Diagram C (Customers).
- `project-map/trd.md`:
  - `Idempotency-Key` (atau `external_ref`) untuk mencegah duplicate order pada network retry.
  - Perhitungan laba kotor transaksi tersimpan secara deterministik melalui snapshot `recorded_unit_cost` pada `transaction_items`.
  - Deduksi stok otomatis pada pembuatan transaksi dan pemulihan stok otomatis saat retur atau pembatalan transaksi (`sales_returns` / `cancelled`).
  - Ketahanan presisi desimal: `NUMERIC(15,2)` untuk harga/uang, `NUMERIC(12,3)` untuk stok.

## Lingkup Endpoint API
1. **Pelanggan (`/api/v1/businesses/:business_id/customers`)**:
   - `GET /` — List pelanggan (search, pagination, sort by latest).
   - `POST /` — Buat pelanggan baru (nama, telepon/identifikasi, catatan).
   - `GET /:customer_id` — Detail pelanggan beserta riwayat transaksi ringkas.
   - `PUT /:customer_id` — Update data pelanggan.
2. **Transaksi Penjualan (`/api/v1/businesses/:business_id/transactions`)**:
   - `GET /` — List transaksi (filter by date range, customer_id, status, pagination).
   - `POST /` — Buat transaksi baru.
     - Header: `Idempotency-Key: <uuid-or-string>` (opsional, jika tidak ada generate internal uuid).
     - Payload: `customer_id` (opsional), `transaction_date`, `items: [{product_id, quantity, unit_selling_price, discount_amount}]`, `payment_status`, `amount_paid`, `fee_amount`, `notes`.
     - Logika internal: Snapshot snapshot HPP terkini produk ke `recorded_unit_cost`, validasi stok mencukupi, deduksi stok `current_stock -= quantity`, buat transaksi & items dalam 1 DB transaction.
   - `GET /:transaction_id` — Rincian detail transaksi beserta items, total penjualan, total HPP, laba kotor, dan riwayat retur.
   - `POST /:transaction_id/cancel` — Batalkan transaksi (`status = 'cancelled'`). Pulihkan seluruh stok produk yang terkait.
3. **Retur Penjualan (`/api/v1/businesses/:business_id/transactions/:transaction_id/returns`)**:
   - `POST /` — Retur item transaksi (`transaction_item_id`, `quantity`, `refund_amount`, `reason`).
     - Logika internal: Validasi total quantity retur tidak melebihi quantity beli, pulihkan stok produk terkait `current_stock += quantity`, catat ke tabel `sales_returns`.
   - `GET /` — List seluruh retur pada transaksi tersebut.

## Output yang Diharapkan
Tulis kontrak API detail ke `contracts/api-transactions-customers.md` dan buat laporan penyelesaian di `inbox/leader/T-031-report.md`.
