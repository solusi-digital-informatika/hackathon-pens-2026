# Kontrak REST API: Dashboard Finansial & Kesehatan Data (M4)

- **ID Kontrak**: API-M4-001
- **Task ID**: T-041
- **Penulis**: Backend-AI Profile / Systems Architect
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED

---

## 1. Aturan Dasar & Keamanan Multi-Tenant
1. Semua rute berada di bawah path ber-prefix `/api/v1/businesses/:business_id/dashboard`.
2. Middleware `RequireAuth` dan `RequireTenant` wajib aktif untuk memvalidasi akses token dan membatasi data hanya untuk `business_id` aktif.
3. Seluruh nilai moneter direpresentasikan dalam format string berpresisi tetap `NUMERIC(15,2)` pada payload JSON (contoh: `"1250000.00"`).
4. **Pengecualian Transaksi Dibatalkan**: Transaksi dengan status `cancelled` **WAJIB DIKECUALIKAN** dari seluruh agregasi finansial (omzet, HPP, laba kotor, kas diterima, dan volume transaksi).
5. **Penanganan Retur Barang**: Retur penjualan (`sales_returns`) mengurangi nilai penjualan bersih (`net_sales`), namun tidak mengurangi HPP yang telah keluar (`cogs`) sesuai spesifikasi TRD §6 (A3).

---

## 2. Endpoint Specifications

### 2.1 Ringkasan Finansial Periode
`GET /api/v1/businesses/:business_id/dashboard/summary`

#### Query Parameters:
- `from` (string, opsional, format `YYYY-MM-DD`): Awal rentang tanggal (inklusif). Default: 7 hari yang lalu.
- `to` (string, opsional, format `YYYY-MM-DD`): Akhir rentang tanggal (inklusif). Default: hari ini.

#### Response 200 OK:
```json
{
  "period": {
    "from": "2026-10-12",
    "to": "2026-10-18"
  },
  "net_sales": "4500000.00",
  "costed_net_sales": "4500000.00",
  "cogs": "2850000.00",
  "gross_profit": "1650000.00",
  "gross_margin_pct": 36.67,
  "cost_coverage_pct": 100.00,
  "is_estimate": false,
  "missing_cost_items_count": 0,
  "cash_received": "4500000.00",
  "recorded_fees": "0.00",
  "total_transactions": 28,
  "average_ticket_size": "160714.29"
}
```

*Catatan Khusus:*
- Jika `costed_net_sales == 0`, `gross_margin_pct` bernilai `null` (mencegah *division by zero*).
- Jika ada item produk terjual tanpa HPP tercatat (`recorded_unit_cost == 0` atau tidak ada versi HPP), `cost_coverage_pct` < 100, `is_estimate` bernilai `true`, dan `missing_cost_items_count` mencatat jumlah item tersebut.

---

### 2.2 Grafik Tren Finansial Harian
`GET /api/v1/businesses/:business_id/dashboard/trends`

#### Query Parameters:
- `from` (string, opsional, format `YYYY-MM-DD`): Tanggal mulai.
- `to` (string, opsional, format `YYYY-MM-DD`): Tanggal akhir.

#### Response 200 OK:
```json
{
  "period": {
    "from": "2026-10-12",
    "to": "2026-10-18"
  },
  "data_points": [
    {
      "date": "2026-10-12",
      "net_sales": "650000.00",
      "gross_profit": "240000.00",
      "cogs": "410000.00",
      "transaction_count": 4
    },
    {
      "date": "2026-10-13",
      "net_sales": "720000.00",
      "gross_profit": "280000.00",
      "cogs": "440000.00",
      "transaction_count": 5
    }
  ]
}
```

---

### 2.3 Diagnostik Kesehatan Data (*Data Health*)
`GET /api/v1/businesses/:business_id/dashboard/data-health`

Mengukur kelengkapan dan kedisiplinan pembukuan usaha pengguna untuk menghasilkan analisis yang akurat.

#### Response 200 OK:
```json
{
  "health_score": 92,
  "status": "healthy",
  "metrics": {
    "cost_coverage_pct": 94.50,
    "missing_cost_products_count": 1,
    "total_active_products": 18,
    "unlinked_customer_tx_count": 5,
    "total_transactions": 50,
    "unlinked_customer_tx_pct": 10.00,
    "low_stock_products_count": 2
  },
  "action_items": [
    {
      "id": "missing_cost_alert",
      "type": "warning",
      "title": "1 Produk Belum Memiliki HPP",
      "description": "Lengkapi komponen HPP pada produk agar perhitungan laba kotor 100% pasti.",
      "action_url": "/products?filter=missing_cost"
    },
    {
      "id": "low_stock_alert",
      "type": "info",
      "title": "2 Produk Mendekati Batas Minimum Stok",
      "description": "Periksa persediaan barang dan lakukan pemesanan ulang ke pemasok.",
      "action_url": "/products?filter=low_stock"
    }
  ]
}
```

---

## 3. Rumus Matematis & Kasus Uji Emas (*Golden Test Cases*)
Mengacu pada TRD §6:
1. `gross = quantity * unit_selling_price`
2. `refunds = Σ sales_returns.refund_amount`
3. `net = gross - discount_amount - refunds`
4. `cogs = quantity * recorded_unit_cost` (jika HPP diketahui, jika tidak maka *unknown*)
5. `gross_profit = costed_net_sales - cogs`
6. `gross_margin = gross_profit / costed_net_sales`
7. `cost_coverage = costed_net_sales / net_sales`
8. `is_estimate = cost_coverage < 100%`
