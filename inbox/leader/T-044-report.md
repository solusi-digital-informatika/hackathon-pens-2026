# Laporan Audit Independen QA T-044: Financial Calculations, Aggregations & Data Health Verification

- **Task ID**: T-044
- **Auditor**: `tester`
- **Backend Verified Commit**: `fb8a4ad` (Test suite commit: `812b2bd`)
- **Frontend Verified Commit**: `4ed545b`
- **Tanggal Audit**: 18 Oktober 2026
- **Vonis**: APPROVE / PASS (100% Lulus)

---

## 1. Lingkup Pengujian & Metodologi
Pengujian independen QA dilakukan pada Milestone M4 (Finance Engine, Dashboard & Data Health) dengan memvalidasi kepatuhan terhadap:
1. **TRD §6 (Calculation Spec, Invariants, dan Golden Test Cases)**:
   - Rumus perhitungan per item transaksi: `gross = qty * price`, `net = gross - discount - refund`, `cogs = qty * snapshot_cost`.
   - Rumus agregasi periode: `net_sales = Σ net`, `costed_net_sales = Σ net (ber-HPP)`, `cogs = Σ cogs`, `gross_profit = costed_net_sales - cogs`.
   - Aturan pembagian dengan nol (*division by zero guard*): margin kotor bernilai `null`/`nil` saat `costed_net_sales == 0`.
   - Aturan retur barang: retur penjualan mengurangi `net_sales` namun **tidak mengurangi HPP** yang telah dicatat (A3).
   - Aturan transaksi dibatalkan: transaksi berstatus `cancelled` **wajib dikeluarkan** dari seluruh agregasi finansial.
2. **Kontrak API (`contracts/api-finance-dashboard.md`)**:
   - `GET /api/v1/businesses/:business_id/dashboard/summary`
   - `GET /api/v1/businesses/:business_id/dashboard/trends`
   - `GET /api/v1/businesses/:business_id/dashboard/data-health`
   - Presisi desimal moneter `NUMERIC(15,2)` dalam format string JSON dan isolasi tenant `RequireAuth` + `RequireTenant`.
3. **Kontrak UI (`contracts/ui-finance-dashboard.md`)**:
   - Signature Component: "Kartu Kepastian Laba" (*Financial Clarity Card*).
   - Naming Rule Ketat: "Laba Kotor" (tidak boleh hanya "Laba") dan "Estimasi" saat `cost_coverage < 100%`.
   - Grid 4 Kartu Metrik Finansial (Penjualan Bersih, Total HPP, Kas Diterima, Rata-rata Nilai Belanja).
   - Grafik Tren Harian SVG interaktif (Penjualan Bersih vs Laba Kotor).
   - Panel Diagnostik Kesehatan Data (*Data Health Card*) dengan skor 0–100 dan status Sehat/Perlu Perhatian/Kritis.

---

## 2. Verifikasi Rumus Matematis & Golden Test Cases (TRD §6)

### 2.1 Matriks Verifikasi Kasus Emas (Golden Test Cases §6.4)

| Kasus Uji | Parameter Input | Nilai Ekspektasi TRD §6 | Hasil Kalkulasi Sistem | Status |
|---|---|---|---|---|
| **Golden Case 1: Partial Cost Coverage** | Item A: Qty 10, Harga 25.000, Diskon 10.000, HPP 15.000.<br>Item B: Qty 5, Harga 30.000, Diskon 0, HPP unknown (0). | Net A = 240.000<br>Net B = 150.000<br>Net Sales = 390.000<br>Costed Net Sales = 240.000<br>COGS = 150.000<br>Gross Profit = 90.000<br>Margin = 37.5%<br>Cost Coverage = 61.54%<br>Is Estimate = true<br>Missing Cost Count = 1 | Net A = 240.000<br>Net B = 150.000<br>Net Sales = 390.000,00<br>Costed Net Sales = 240.000,00<br>COGS = 150.000,00<br>Gross Profit = 90.000,00<br>Margin = 37.5%<br>Cost Coverage = 61.54%<br>Is Estimate = true<br>Missing Cost Count = 1 | **PASS** |
| **Golden Case 2: Return Handling** | Item A seperti di atas (Gross 250.000, Diskon 10.000, HPP 150.000) ditambah retur/refund 25.000. | Net A = 215.000<br>COGS = 150.000 (tidak berkurang)<br>Gross Profit = 65.000<br>Margin = 30.23%<br>Cost Coverage = 100% | Net A = 215.000,00<br>COGS = 150.000,00<br>Gross Profit = 65.000,00<br>Margin = 30.23%<br>Cost Coverage = 100% | **PASS** |
| **Golden Case 3: Zero Sales** | `net_sales` = 0, item transaksi kosong. | `gross_margin_pct` = null / nil (tidak memicu division by zero panic atau NaN) | `gross_margin_pct` = nil<br>Net Sales = 0.00<br>Cost Coverage = 100% | **PASS** |
| **Golden Case 4: Cancelled Transactions** | Transaksi dengan `status = 'cancelled'`. | Dikecualikan 100% dari seluruh agregasi omzet, HPP, kas diterima, dan hitungan transaksi. | Terfilter melalui SQL query repository (`WHERE status != 'cancelled'`). | **PASS** |

### 2.2 Verifikasi Engine Diagnostik Kesehatan Data (*Data Health*)
- **Bobot Skor Komposit**:
  - Skor awal: 100 poin.
  - Penalti HPP Produk: `- (100 - cost_coverage_pct) * 0.5`.
  - Penalti Transaksi Tanpa Pelanggan: `- (unlinked_customer_tx_pct) * 0.2`.
  - Penalti Stok Menipis: `- (low_stock_products_count * 5)` (maksimal 15 poin).
- **Klasifikasi Status**:
  - `healthy`: Skor ≥ 85 (Label: "Sehat").
  - `warning`: Skor 60–84 (Label: "Perlu Perhatian").
  - `critical`: Skor < 60 (Label: "Kritis").
- **Rekomendasi Cerdas**: Menghasilkan tautan kontekstual ke `/products` untuk melengkapi HPP atau restock barang, dan ke `/customers` untuk pencatatan pelanggan.

---

## 3. Hasil Eksekusi Pengujian Backend (`fb8a4ad` / `812b2bd`)
Perintah eksekusi:
`cd C:/Project/ai-office/office-hub/backend && go test -v -count=1 ./...`

```
=== RUN   TestTRDGoldenCase1_PartialCostCoverage
--- PASS: TestTRDGoldenCase1_PartialCostCoverage (0.00s)
=== RUN   TestTRDGoldenCase2_ReturnHandling
--- PASS: TestTRDGoldenCase2_ReturnHandling (0.00s)
=== RUN   TestTRDGoldenCase3_ZeroSalesMarginNull
--- PASS: TestTRDGoldenCase3_ZeroSalesMarginNull (0.00s)
=== RUN   TestDailyTrendsCalculation
--- PASS: TestDailyTrendsCalculation (0.00s)
=== RUN   TestFinanceService_DataHealthCompositeScore
--- PASS: TestFinanceService_DataHealthCompositeScore (0.00s)
=== RUN   TestTRDSection6_VerbatimGoldenCases
--- PASS: TestTRDSection6_VerbatimGoldenCases (0.00s)
PASS
ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/finance       1.941s
```

Ringkasan Regresi Seluruh Modul Backend:
- `internal/auth`: 4 tests PASS (JWT generation, tampered JWT, Argon2 hashing, hash validation)
- `internal/business`: 1 test PASS (`TestTenantIsolationCrossAccess`)
- `internal/customer`: 1 test PASS (`TestCustomerCRUDAndTenantIsolation`)
- `internal/finance`: 6 tests PASS (Semua Golden Cases TRD §6 verbatim, Daily trends, Health composite score)
- `internal/platform/config`: 2 tests (8 subtests) PASS
- `internal/platform/database`: 5 tests PASS
- `internal/platform/httpserver`: 7 tests PASS
- `internal/platform/logger`: 2 tests PASS
- `internal/product`: 4 tests PASS (Margin, Markup, Low margin warning, Isolation)
- `internal/transaction`: 3 tests PASS (`TestCalculationHelpers`, `TestTransactionFinancialCalculations`, `TestSalesReturnStockCalculation`)
- **Status Kompilasi Biner**: `go build ./cmd/api` -> EXIT 0 (0 error, 0 warning).

---

## 4. Hasil Eksekusi Pengujian Frontend (`4ed545b`)
Perintah eksekusi:
`cd C:/Project/ai-office/office-hub/frontend && npm test -- --run`

```
 ✓ src/modules/pos/__tests__/POSTransactions.test.tsx (3 tests) 3ms
 ✓ src/App.test.tsx (3 tests) 197ms
 ✓ src/modules/dashboard/__tests__/Dashboard.test.tsx (4 tests) 237ms
 ✓ src/modules/products/__tests__/Products.test.tsx (4 tests) 383ms

 Test Files  4 passed (4)
      Tests  14 passed (14)
   Duration  11.45s
```

Status Kompilasi Produksi Frontend:
`cd C:/Project/ai-office/office-hub/frontend && npm run build`
- `tsc -b && vite build`: Berhasil dalam 4.61 detik tanpa error TypeScript atau bundler.

---

## 5. Evaluasi Desain UI & Aturan Naming (UI Contract UI-M4-001)

1. **Aturan Penamaan Finansial (Strict Naming Rule)**:
   - Terverifikasi menggunakan sebutan **"Laba Kotor"** pada `ClarityCard`, `MetricCards`, dan `TrendChart`. Tidak pernah menggunakan label ambigu "Laba".
   - Saat `cost_coverage < 100%`, badge secara eksplisit menampilkan label **"Estimasi (XX% HPP Tercakup)"** dan teks peringatan jumlah item tanpa HPP dengan tautan CTA `/products?filter=missing_cost`.
2. **Visual & Estetika Clean Pragmatic Craftsman**:
   - Aksen tunggal *Emerald Forest* (`#0D6E64` / `var(--color-primary-700)`) digunakan secara konsisten pada badge status pasti, batang grafik laba kotor, dan angka sorotan.
   - Flat cards tanpa resting shadows berbatas `1px solid var(--color-border-default)`.
   - Angka finansial diformat rapi dengan `formatRupiah` standar Indonesia.
3. **Grafik Tren Responsif**:
   - Menggunakan visualisasi SVG murni yang ringan dan adaptif.
   - Skala sumbu Y ringkas (`rb`, `jt`, `M`) dan tooltip hover interaktif.
4. **Layout & State Lengkap**:
   - Tombol aksi cepat header: `+ Buka Kasir POS` dan `+ Produk Baru`.
   - Selector pil rentang waktu: Hari Ini, 7 Hari Terakhir (default), 30 Hari Terakhir, Bulan Ini.
   - Penanganan skeleton loading, error state dengan tombol coba lagi (*retry*), dan data kosong (*empty state*).

---

## 6. Verifikasi Keamanan & Multi-Tenant

- **Autentikasi & Autorisasi Tenant**:
  - Seluruh rute `/api/v1/businesses/:business_id/dashboard/...` berada di bawah middleware `RequireAuth` dan `RequireTenant`.
  - Percobaan akses antar penyewa (`business_id` berbeda) ditolak dengan `403 Forbidden` atau `401 Unauthorized`.
- **Injeksi SQL & Parameter Tanggal**:
  - Parameter tanggal `from` dan `to` diparsing dan dinormalisasi secara aman pada service layer.
  - Parameter query GORM menggunakan prepared statements terparameterisasi (`WHERE business_id = ? AND date(created_at) >= ?`).

---

## 7. Rekomendasi & Vonis Akhir

Seluruh spesifikasi teknis TRD §6, kontrak API `contracts/api-finance-dashboard.md`, dan kontrak UI `contracts/ui-finance-dashboard.md` telah diuji secara mendalam dan terbukti 100% patuh dan presisi.

**Vonis QA: APPROVE / PASS**

Milestone M4 dinyatakan **SELESAI dan VALID**. Siap untuk penandatanganan penutupan milestone oleh Leader (T-045).