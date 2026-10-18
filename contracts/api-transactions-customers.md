# API Contract: Customers, Transactions, Idempotency & Stock Management

## Metadata
- **Task ID**: T-031
- **Version**: 1.0.0
- **Status**: Ratified
- **Base URL**: `/api/v1/businesses/:business_id`
- **Security**: Bearer JWT + `RequireTenant` (scoped to `business_id`)

---

## 1. Modul Pelanggan (Customers)

### 1.1 List Customers
- **Endpoint**: `GET /customers`
- **Query Parameters**:
  - `search` (string, optional): Filter nama pelanggan, nomor telepon, atau email.
  - `status` (string, optional, default: `active`): `active`, `archived`, `all`.
  - `page` (integer, default: 1)
  - `page_size` (integer, default: 20)
- **Response 200 OK**:
  ```json
  {
    "items": [
      {
        "id": "uuid",
        "business_id": "uuid",
        "display_name": "Ibu Siti Rahma",
        "phone": "+6281234567890",
        "email": "siti.rahma@example.com",
        "notes": "Langganan katering harian",
        "status": "active",
        "total_transactions": 14,
        "total_spent": "1450000.00",
        "last_transaction_date": "2026-10-17T14:30:00Z",
        "created_at": "2026-10-01T08:00:00Z",
        "updated_at": "2026-10-17T14:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_items": 1,
      "total_pages": 1
    }
  }
  ```

### 1.2 Create Customer
- **Endpoint**: `POST /customers`
- **Request Body**:
  ```json
  {
    "display_name": "Ibu Siti Rahma",
    "phone": "081234567890",
    "email": "siti.rahma@example.com",
    "notes": "Langganan katering harian"
  }
  ```
- **Response 201 Created**: Objek customer yang baru dibuat.
- **Error Codes**:
  - `400 Bad Request`: `NAME_REQUIRED`, `INVALID_PHONE_FORMAT`.

### 1.3 Get Customer Detail
- **Endpoint**: `GET /customers/:customer_id`
- **Response 200 OK**: Objek customer lengkap dengan ringkasan 5 transaksi terakhir.

### 1.4 Update Customer
- **Endpoint**: `PUT /customers/:customer_id`
- **Request Body**:
  ```json
  {
    "display_name": "Ibu Siti Rahma",
    "phone": "081234567890",
    "email": "siti.rahma@example.com",
    "notes": "Update preferensi alamat katering",
    "status": "active"
  }
  ```
- **Response 200 OK**: Objek customer yang telah diperbarui.

---

## 2. Modul Transaksi Penjualan (Transactions)

### 2.1 List Transactions
- **Endpoint**: `GET /transactions`
- **Query Parameters**:
  - `start_date` (string `YYYY-MM-DD`, optional)
  - `end_date` (string `YYYY-MM-DD`, optional)
  - `customer_id` (uuid, optional)
  - `status` (string, optional): `valid`, `cancelled`, `all` (default: `all`)
  - `payment_status` (string, optional): `paid`, `partial`, `unpaid`
  - `page` (integer, default: 1)
  - `page_size` (integer, default: 20)
- **Response 200 OK**:
  ```json
  {
    "items": [
      {
        "id": "uuid",
        "business_id": "uuid",
        "invoice_number": "TRX-20261018-0001",
        "customer_id": "uuid-or-null",
        "customer_name": "Ibu Siti Rahma",
        "transaction_date": "2026-10-18",
        "status": "valid",
        "payment_status": "paid",
        "payment_method": "qris",
        "subtotal_amount": "120000.00",
        "discount_amount": "10000.00",
        "fee_amount": "0.00",
        "total_amount": "110000.00",
        "total_cost_amount": "60000.00",
        "gross_profit_amount": "50000.00",
        "gross_margin_percentage": 45.45,
        "amount_paid": "110000.00",
        "change_amount": "0.00",
        "source": "manual",
        "notes": "Pesanan makan siang",
        "total_items": 3,
        "created_at": "2026-10-18T10:15:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_items": 1,
      "total_pages": 1
    }
  }
  ```

### 2.2 Create Transaction (POS Checkout)
- **Endpoint**: `POST /transactions`
- **Headers**:
  - `Idempotency-Key` (string, optional tapi sangat disarankan, max 64 chars): Kunci deterministik untuk menjamin transaksi tidak terduplikasi saat network retry. Jika key sudah ada di bisnis terkait, sistem mengembalikan transaksi yang sudah tersimpan tanpa eksekusi ganda atau pengurangan stok berulang.
- **Request Body**:
  ```json
  {
    "customer_id": "uuid-or-null",
    "transaction_date": "2026-10-18",
    "payment_method": "tunai",
    "payment_status": "paid",
    "amount_paid": "150000.00",
    "fee_amount": "0.00",
    "discount_amount": "10000.00",
    "notes": "Catatan transaksi",
    "items": [
      {
        "product_id": "uuid-prod-1",
        "quantity": "2.000",
        "unit_selling_price": "25000.00",
        "discount_amount": "0.00"
      },
      {
        "product_id": "uuid-prod-2",
        "quantity": "1.000",
        "unit_selling_price": "70000.00",
        "discount_amount": "10000.00"
      }
    ]
  }
  ```
- **Logika Internal & Transaksi Basis Data**:
  1. Cek `Idempotency-Key` (atau `external_ref`). Jika sudah pernah tercatat untuk `business_id` tersebut, kembalikan data transaksi yang sudah tersimpan dengan HTTP 200 OK.
  2. Buka DB Transaction.
  3. Validasi setiap `product_id`:
     - Pastikan produk aktif dan milik `business_id`.
     - Validasi stok: Periksa apakah `current_stock >= quantity`. Jika stok tidak mencukupi, rollback dan kembalikan HTTP 400 `INSUFFICIENT_STOCK`.
  4. Ambil HPP produk saat ini (`current_cost_price`) dan snapshot versi HPP (`cost_version_id`), simpan ke `recorded_unit_cost` pada `transaction_items`.
  5. Hitung total biaya HPP transaksi (`total_cost_amount = SUM(quantity * recorded_unit_cost)`).
  6. Kurangi stok produk: `UPDATE products SET current_stock = current_stock - quantity WHERE id = product_id`.
  7. Simpan `transactions` dan seluruh `transaction_items`.
  8. Buat entri audit log: `ACTION_CREATE_TRANSACTION`.
  9. Commit DB Transaction.
- **Response 201 Created**: Objek transaksi lengkap dengan items dan kalkulasi laba.
- **Error Codes**:
  - `400 Bad Request`: `EMPTY_ITEMS`, `INSUFFICIENT_STOCK`, `INVALID_QUANTITY`, `INVALID_PRICE`.
  - `404 Not Found`: `PRODUCT_NOT_FOUND`, `CUSTOMER_NOT_FOUND`.

### 2.3 Get Transaction Detail
- **Endpoint**: `GET /transactions/:transaction_id`
- **Response 200 OK**:
  ```json
  {
    "id": "uuid",
    "business_id": "uuid",
    "invoice_number": "TRX-20261018-0001",
    "customer": {
      "id": "uuid",
      "display_name": "Ibu Siti Rahma",
      "phone": "+6281234567890"
    },
    "transaction_date": "2026-10-18",
    "status": "valid",
    "payment_status": "paid",
    "payment_method": "tunai",
    "subtotal_amount": "120000.00",
    "discount_amount": "10000.00",
    "fee_amount": "0.00",
    "total_amount": "110000.00",
    "total_cost_amount": "60000.00",
    "gross_profit_amount": "50000.00",
    "gross_margin_percentage": 45.45,
    "amount_paid": "150000.00",
    "change_amount": "40000.00",
    "notes": "Pesanan makan siang",
    "items": [
      {
        "id": "uuid-item-1",
        "product_id": "uuid-prod-1",
        "product_name_snapshot": "Ayam Bakar Madu",
        "quantity": "2.000",
        "unit_selling_price": "25000.00",
        "discount_amount": "0.00",
        "subtotal_price": "50000.00",
        "recorded_unit_cost": "15000.00",
        "subtotal_cost": "30000.00",
        "quantity_returned": "0.000"
      }
    ],
    "returns": [],
    "created_at": "2026-10-18T10:15:00Z"
  }
  ```

### 2.4 Cancel / Void Transaction
- **Endpoint**: `POST /transactions/:transaction_id/cancel`
- **Request Body**:
  ```json
  {
    "cancel_reason": "Pelanggan membatalkan seluruh pesanan sebelum diolah"
  }
  ```
- **Logika Internal & Transaksi Basis Data**:
  1. Validasi transaksi milik `business_id` dan status saat ini adalah `valid`. Jika sudah `cancelled`, tolak `ALREADY_CANCELLED`.
  2. Buka DB Transaction.
  3. Untuk setiap item dalam transaksi, kurangi kuantitas yang belum diretur, lalu kembalikan ke stok produk:
     `UPDATE products SET current_stock = current_stock + (item.quantity - item.quantity_returned) WHERE id = item.product_id`.
  4. Ubah status transaksi: `UPDATE transactions SET status = 'cancelled', cancelled_at = NOW(), cancel_reason = :reason WHERE id = :transaction_id`.
  5. Catat audit log: `ACTION_CANCEL_TRANSACTION`.
  6. Commit DB Transaction.
- **Response 200 OK**:
  ```json
  {
    "message": "Transaksi berhasil dibatalkan dan stok produk telah dipulihkan.",
    "transaction_id": "uuid",
    "status": "cancelled",
    "cancelled_at": "2026-10-18T10:30:00Z"
  }
  ```

---

## 3. Modul Retur Penjualan (Sales Returns)

### 3.1 Create Sales Return
- **Endpoint**: `POST /transactions/:transaction_id/returns`
- **Request Body**:
  ```json
  {
    "transaction_item_id": "uuid-item-1",
    "returned_on": "2026-10-18",
    "quantity": "1.000",
    "refund_amount": "25000.00",
    "reason": "Kemasan rusak saat pengiriman"
  }
  ```
- **Logika Internal & Transaksi Basis Data**:
  1. Validasi transaksi status `valid`.
  2. Validasi `transaction_item_id` milik transaksi tersebut.
  3. Validasi kuantitas retur: `quantity <= (item.quantity - item.quantity_returned)`. Jika melebihi, tolak `RETURN_QUANTITY_EXCEEDS_BOUGHT`.
  4. Buka DB Transaction.
  5. Pulihkan stok produk terkait: `UPDATE products SET current_stock = current_stock + :quantity WHERE id = item.product_id`.
  6. Catat record ke tabel `sales_returns`.
  7. Catat audit log: `ACTION_SALES_RETURN`.
  8. Commit DB Transaction.
- **Response 201 Created**: Objek sales return yang berhasil dicatat.

### 3.2 List Sales Returns
- **Endpoint**: `GET /transactions/:transaction_id/returns`
- **Response 200 OK**: List seluruh retur yang tercatat pada transaksi tersebut.
