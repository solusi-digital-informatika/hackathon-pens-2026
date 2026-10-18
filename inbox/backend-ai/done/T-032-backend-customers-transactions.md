# Task: T-032 Implement Backend Customers, Transactions, Idempotency & Stock Deductions

## Objective
Implementasikan seluruh modul backend untuk Pelanggan (Customers), Transaksi Penjualan (Transactions & Items), Idempotency Key, Pembatalan Transaksi (Void/Cancel), Retur Penjualan (Sales Returns), dan deduksi/pemulihan stok otomatis dalam transaksi database.

## Referensi & Kontrak
- `contracts/api-transactions-customers.md`
- `contracts/erd.md` (Diagram B & C)
- `backend/migrations/`
- `backend/internal/platform/`

## Rincian Implementasi
1. **Migrasi Database (`migrations/000004_create_customers_and_transactions_tables.up.sql` & `.down.sql`)**:
   - Tabel `customers`: `id UUID PRIMARY KEY`, `business_id UUID NOT NULL`, `display_name TEXT NOT NULL`, `phone TEXT`, `email TEXT`, `notes TEXT`, `status TEXT NOT NULL DEFAULT 'active'`, `row_version INT NOT NULL DEFAULT 1`, `created_at TIMESTAMPTZ`, `updated_at TIMESTAMPTZ`.
   - Tabel `transactions`: `id UUID PRIMARY KEY`, `business_id UUID NOT NULL`, `customer_id UUID NULL`, `invoice_number TEXT NOT NULL`, `transaction_date DATE NOT NULL`, `status TEXT NOT NULL DEFAULT 'valid'`, `payment_status TEXT NOT NULL DEFAULT 'paid'`, `payment_method TEXT NOT NULL DEFAULT 'tunai'`, `subtotal_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `discount_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `fee_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `total_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `total_cost_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `amount_paid NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `change_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `source TEXT NOT NULL DEFAULT 'manual'`, `external_ref TEXT NULL`, `cancelled_at TIMESTAMPTZ NULL`, `cancel_reason TEXT NULL`, `notes TEXT NULL`, `row_version INT NOT NULL DEFAULT 1`, `created_by UUID NULL`, `created_at TIMESTAMPTZ`, `updated_at TIMESTAMPTZ`.
   - Tabel `transaction_items`: `id UUID PRIMARY KEY`, `business_id UUID NOT NULL`, `transaction_id UUID NOT NULL`, `product_id UUID NOT NULL`, `product_name_snapshot TEXT NOT NULL`, `quantity NUMERIC(12,3) NOT NULL`, `unit_selling_price NUMERIC(15,2) NOT NULL`, `discount_amount NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `subtotal_price NUMERIC(15,2) NOT NULL`, `recorded_unit_cost NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `subtotal_cost NUMERIC(15,2) NOT NULL DEFAULT 0.00`, `cost_version_id UUID NULL`, `quantity_returned NUMERIC(12,3) NOT NULL DEFAULT 0.000`, `created_at TIMESTAMPTZ`.
   - Tabel `sales_returns`: `id UUID PRIMARY KEY`, `business_id UUID NOT NULL`, `transaction_id UUID NOT NULL`, `transaction_item_id UUID NOT NULL`, `returned_on DATE NOT NULL`, `quantity NUMERIC(12,3) NOT NULL`, `refund_amount NUMERIC(15,2) NOT NULL`, `reason TEXT NOT NULL`, `created_by UUID NULL`, `created_at TIMESTAMPTZ`.
   - Indexes: `idx_customers_business`, `idx_transactions_business_date`, `idx_transactions_external_ref`, `idx_transaction_items_trx`, `idx_sales_returns_trx`.
2. **Paket Go (`internal/customer/` dan `internal/transaction/`)**:
   - `model.go`: Struct GORM dengan tag JSON string untuk desimal.
   - `repository.go`: Operasi database dengan parameter scoped `business_id`.
   - `service.go`: Business logic:
     - `CreateTransaction`: Validasi stok, snapshot HPP ke `recorded_unit_cost`, deduksi `current_stock = current_stock - quantity`, cek `Idempotency-Key` / `external_ref`.
     - `CancelTransaction`: Pemulihan seluruh stok produk (`current_stock = current_stock + (quantity - quantity_returned)`).
     - `CreateSalesReturn`: Pemulihan stok parsial barang yang diretur (`current_stock = current_stock + return_qty`).
   - `handler.go`: Gin router handlers.
3. **Pendaftaran Rute**:
   - Daftarkan grup rute `/customers` dan `/transactions` di `internal/platform/httpserver/server.go`.
4. **Unit & Integration Tests**:
   - `customer_test.go`: CRUD customer dan isolasi tenant.
   - `transaction_test.go`: Pembuatan transaksi, deduksi stok otomatis, pencegahan minus stok, proteksi `Idempotency-Key`, pembatalan void dan pemulihan stok, serta retur sebagian.

## Output
Jalankan tes Go, pastikan pass 100%, commit dengan pesan konvensional, dan laporkan hash commit.
