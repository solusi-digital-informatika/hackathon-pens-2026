# Kontrak REST API: Insight Rules Engine, Recommendations, Provenance & Cooldown (M6)

## Metadata
- **ID Kontrak**: API-M6-001
- **Task ID**: T-061
- **Penulis**: Backend & AI Engineer (`backend-ai`) / Systems Architect
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED
- **Target Audience**: Frontend Engineer (`frontend`), Backend & AI Engineer (`backend-ai`), QA Engineer (`tester`), Systems Architect
- **Dokumen Terkait**:
  - `project-map/trd.md` (§7 Insight Engine Spec, §9 API Inventory, §10 Security & Rate Limits, TD-05 Tenant Isolation, TD-09 Unknown vs Zero)
  - `contracts/erd.md` (§5 Diagram D, §7 Rules & Enums: `analysis_runs`, `insights`, `insight_sources`, `recommended_actions`)
  - `contracts/ui-insights-recommendations.md` (UI-M6-001: Insight Cards, Impact Preview, Provenance Modal, Dismissal Dialog)
- **Base Prefix**: `/api/v1/businesses/{business_id}`

---

## 1. Aturan Dasar & Keamanan Multi-Tenant

1. **Multi-Tenant Boundary**:
   - Seluruh endpoint berada di bawah prefix `/api/v1/businesses/{business_id}`.
   - Middleware `RequireAuth` (memverifikasi Bearer JWT access token) dan `RequireTenant` (memverifikasi kepemilikan/keanggotaan user pada `business_id`) **WAJIB** aktif di setiap rute.
   - Setiap repository query wajib memfilter data dengan klausul `business_id = :business_id`.
   - Menggunakan composite foreign keys sesuai `contracts/erd.md` §7: `insights (business_id, analysis_run_id)`, `insight_sources (business_id, insight_id)`, `recommended_actions (business_id, insight_id)`.

2. **Idempotensi & Rate Limiting Pemicuan Analisis (`POST /analysis/run`)**:
   - Pemicuan analisis manual dibatasi maksimal **1 kali per 10 menit per entitas bisnis** (TRD §10 Rate Limiting).
   - Menjaga keutuhan proses: Hanya 1 analisis berstatus `running` yang diizinkan berjalan dalam waktu bersamaan per bisnis (dijamin oleh partial unique index: `analysis_runs (business_id) WHERE status = 'running'`).
   - Jika terdapat run yang sedang berjalan: sistem mengembalikan `409 Conflict` dengan kode `ANALYSIS_ALREADY_RUNNING`.
   - Jika analisis terakhir selesai kurang dari 10 menit yang lalu: sistem mengembalikan `429 Too Many Requests` dengan kode `ANALYSIS_COOLDOWN_ACTIVE` dan header `Retry-After: <detik>`.

3. **Format Data & Tipe Presisi**:
   - Nilai moneter (rupiah/omzet/rugi) dikirimkan dalam format string numerik desimal `NUMERIC(15,2)` (misal: `"450000.00"`). Dilarang representasi float.
   - Persentase atau rasio dikirimkan sebagai desimal/float berpresisi 2 angka di belakang koma (misal: `15.50` untuk 15.5%).
   - Tanggal format `YYYY-MM-DD` (ISO 8601).
   - Waktu stempel format RFC 3339 UTC (misal: `2026-10-18T14:30:00Z`).
   - Identitas baris (ID) menggunakan UUIDv7 string.

4. **Siklus Hidup Wawasan & Rekomendasi (Dedupe Key, Cooldown, & Superseding)**:
   - Setiap wawasan diidentifikasi unik secara semantik melalui `dedupe_key` (contoh: `CUST_OVERDUE_V1:cust_0192e10a-4b92-7000-8000-000000000001`).
   - **Kondisi Tetap Sama**: Jika pipeline analisis mengevaluasi ulang dan kondisi anomali masih berlaku, baris wawasan diperbarui di tempat (`calculated_at` dimutakhirkan, `evidence_snapshot` diperbarui), status rekomendasi yang sedang berjalan (`in_progress` / `reviewed`) tetap dipertahankan.
   - **Kondisi Selesai / Teratasi (Superseded)**: Jika kondisi aturan sudah tidak lagi terpenuhi (misal pelanggan yang terlambat telah melakukan transaksi baru), kolom `superseded_at` pada wawasan diisi `NOW()`. Rekomendasi terkait diberi label penanda bahwa kondisi telah teratasi otomatis.
   - **Pengabaian (Dismissal) & Jeda 14 Hari (Cooldown)**:
     - Pengguna dapat mengabaikan wawasan dengan mengirimkan `PATCH /recommendations/{id}` berstatus `dismissed` disertai `dismissed_reason`.
     - Ketika diabaikan, mesin analisis dilarang membuat ulang wawasan atau rekomendasi untuk `dedupe_key` tersebut selama **14 hari (14 × 24 jam)** sejak saat pengabaian (`status_changed_at`).
     - Setelah jeda 14 hari terlampaui dan kondisi data masih memicu anomali, wawasan baru dapat dimunculkan kembali dengan status rekomendasi `new`.

5. **Integritas Status Klaim (Claim-Type Integrity - TRD §7 & PRD §5.9)**:
   - `fact`: Fakta pasti yang bersumber langsung dari baris data tersimpan (contoh: komplain terbuka > 3 hari, harga produk di bawah HPP tercatat).
   - `estimate`: Nilai kalkulasi agregat atau rata-rata perbandingan periode tertentu (contoh: margin produk berada di bawah rata-rata margin bisnis).
   - `hypothesis`: Pola dugaan berbasis deviasi historis (contoh: keterlambatan siklus belanja pelanggan). Dilarang menyimpulkan kepastian (dilarang menggunakan kata "pasti berhenti").

---

## 2. Spesifikasi 8 Aturan Deterministik Awal (TRD §7)

Seluruh parameter numerik merupakan target awal terkalibrasi (*initial defaults*) yang tersimpan dalam konfigurasi versi aturan (`rule_version: "v1.0"`).

### 2.1 Ringkasan Katalog Aturan

| Rule ID | Kategori (`insight_type`) | Status Klaim (`claim_type`) | Data Gate (Syarat Minimal Data) | Rumus Pemicu (Trigger Condition) | Base Priority | Target Entitas (`subject_type`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `CUST_OVERDUE_V1` | `customer_activity` | `hypothesis` | Memiliki ≥ 4 tanggal transaksi pembelian berbeda dalam 180 hari terakhir | `days_since_last > max(1.5 × median_interval, median_interval + 7)` | `medium` | `customer` |
| `CUST_FREQ_DROP_V1` | `customer_activity` | `hypothesis` | Memiliki ≥ 4 transaksi dalam periode 90 hari sebelumnya (H-91 s/d H-180) | `tx_count_last_90d ≤ 0.5 × tx_count_prev_90d` | `medium` | `customer` |
| `CUST_VALUE_DROP_V1` | `customer_activity` | `hypothesis` | Memiliki ≥ 4 transaksi periode sebelumnya (H-91 s/d H-180) dan ≥ 2 transaksi 90 hari terakhir (H-1 s/d H-90) | `avg_order_value_recent ≤ 0.6 × avg_order_value_prev` | `low` | `customer` |
| `CUST_COMPLAINT_OPEN_V1` | `customer_activity` | `fact` | Terdapat komplain berstatus `open` atau `in_progress` | Durasi komplain terbuka > 3 hari kalender (`NOW() - created_at > 3 days`) | `high` | `customer` |
| `PROD_BELOW_COST_V1` | `product_profitability` | `fact` | Produk aktif (`is_active = true`) dan memiliki versi HPP aktif (`current_cost_version_id IS NOT NULL`) | `current_selling_price < hpp_per_unit` | `high` | `product` |
| `PROD_LOW_MARGIN_V1` | `product_profitability` | `estimate` | Produk memiliki penjualan dengan HPP tercatat ≥ 10 unit dalam 30 hari terakhir dan total penjualan bisnis ber-HPP > 0 | `product_margin_pct ≤ business_wide_margin_pct - 10.0` (selisih minimal 10 poin persentase) | `medium` | `product` |
| `DATA_MISSING_COST_V1` | `data_health` | `fact` | Terdapat ≥ 1 transaksi penjualan item dalam 30 hari terakhir | `cost_coverage_pct < 100.0` (terdapat penjualan tanpa HPP tercatat) | `medium` | `business` |
| `DATA_UNLINKED_TX_V1` | `data_health` | `fact` | Terdapat ≥ 20 transaksi penjualan dalam 30 hari terakhir | `(unlinked_tx_count / total_tx_count) > 0.30` (transaksi tanpa pelanggan > 30%) | `low` | `business` |

---

### 2.2 Aturan Prioritas & Eskalasi (*Priority Boost Rule*)
- **Base Priority**: Ditentukan oleh kolom `Base Priority` pada masing-masing aturan (`high`, `medium`, `low`).
- **Revenue Contribution Boost (Top 20% Rule)**:
  - Pelanggan yang masuk ke dalam kategori **Top 20% Kontributor Omzet Usaha** (berdasarkan total belanja transaksi tidak dibatalkan dalam 180 hari terakhir) mendapatkan eskalasi prioritas **naik 1 tingkat**:
    - `low` → `medium`
    - `medium` → `high`
    - `high` → tetap `high` (dengan label penanda nasabah inti).
  - Wawasan dengan subject selain pelanggan (`product`, `business`) tidak mendapatkan boost ini.

---

### 2.3 Rincian Logika, Template, & Pembuktian 8 Aturan

#### 1. Aturan `CUST_OVERDUE_V1`
- **Signal**: Pelanggan aktif melampaui interval waktu belanja biasanya.
- **Format Dedupe Key**: `CUST_OVERDUE_V1:cust_{customer_id}`
- **Data Gate**: `COUNT(DISTINCT transaction_date) >= 4` dalam rentang waktu `[NOW() - 180 days, NOW()]`.
- **Kalkulasi**:
  - Ambil selisih hari antar transaksi berurutan: `intervals = [d2 - d1, d3 - d2, ...]`.
  - Hitung median interval: `median_interval = median(intervals)`.
  - Hitung hari sejak belanja terakhir: `days_since_last = CURRENT_DATE - last_transaction_date`.
  - Ambang batas: `threshold_days = CEIL(GREATEST(1.5 * median_interval, median_interval + 7))`.
  - Kondisi terpicu: `days_since_last > threshold_days`.
- **Template Judul**: `"{customer_name} melampaui interval belanja biasanya"`
- **Template Narasi Penjelasan**:
  `"Berdasarkan {tx_count} transaksi dalam 180 hari terakhir, pelanggan biasanya berbelanja setiap {median_interval} hari. Hingga saat ini sudah {days_since_last} hari sejak transaksi terakhir ({overdue_days} hari melampaui ambang batas aman)."`
- **Asumsi Algoritma**:
  - "Interval belanja dihitung menggunakan nilai median tanggal belanja untuk meredam anomali pembelian mendadak."
  - "Toleransi keterlambatan ditetapkan pada nilai terbesar antara 1.5 kali median atau median ditambah 7 hari."
- **Limitasi**:
  - "Sistem tidak memantau transaksi atau interaksi pelanggan di luar catatan kasir aplikasi."
  - "Analisis ini merupakan evaluasi pola masa lalu dan bukan ramalan kepastian perilaku masa depan."
- **Rekomendasi Tindakan**:
  - Judul: `"Hubungi pelanggan untuk menanyakan kebutuhan stok produk"`
  - Tindakan: `"Kirimkan sapaan berkala via WhatsApp atau tawarkan pemesanan ulang produk favorit."`
  - Reasoning: `"Pelanggan dengan interval teratur berisiko beralih ke pemasok lain jika tidak segera disapa saat pola belanjanya terhenti."`
  - Pratinjau Dampak: `"Peluang pengamanan omzet belanja berkala ~Rp {avg_ticket_size} per siklus transaksi."`
- **Sumber Bukti (`insight_sources`)**:
  - Seluruh baris transaksi pelanggan dalam 180 hari terakhir (`entity_type: "transaction"`).
  - Profil pelanggan (`entity_type: "customer"`).
- **Kondisi Supersede**: Transaksi baru dibuat untuk pelanggan ini (`transaction_date > last_transaction_date`), sehingga `days_since_last` kembali bernilai 0.

#### 2. Aturan `CUST_FREQ_DROP_V1`
- **Signal**: Penurunan frekuensi jumlah transaksi pembelian pelanggan secara signifikan.
- **Format Dedupe Key**: `CUST_FREQ_DROP_V1:cust_{customer_id}`
- **Data Gate**: Memiliki `tx_count_prev >= 4` transaksi pada rentang H-91 s/d H-180.
- **Kalkulasi**:
  - `tx_count_recent`: Total transaksi non-cancelled rentang H-1 s/d H-90.
  - `tx_count_prev`: Total transaksi non-cancelled rentang H-91 s/d H-180.
  - Kondisi terpicu: `tx_count_recent <= 0.5 * tx_count_prev`.
- **Template Judul**: `"Frekuensi belanja {customer_name} menurun drastis"`
- **Template Narasi Penjelasan**:
  `"Pelanggan hanya melakukan {tx_count_recent} transaksi dalam 90 hari terakhir, turun lebih dari 50% dibandingkan 90 hari sebelumnya ({tx_count_prev} transaksi)."`
- **Sumber Bukti (`insight_sources`)**: Transaksi periode 180 hari terakhir.
- **Kondisi Supersede**: Pada run berikutnya, rasio frekuensi transaksi 90 hari terakhir terhadap 90 hari sebelumnya > 0.5.

#### 3. Aturan `CUST_VALUE_DROP_V1`
- **Signal**: Rata-rata nominal belanja per transaksi pelanggan turun tajam.
- **Format Dedupe Key**: `CUST_VALUE_DROP_V1:cust_{customer_id}`
- **Data Gate**: Memiliki `tx_count_prev >= 4` pada H-91 s/d H-180 dan `tx_count_recent >= 2` pada H-1 s/d H-90.
- **Kalkulasi**:
  - `avg_val_recent = net_sales_recent / tx_count_recent`
  - `avg_val_prev = net_sales_prev / tx_count_prev`
  - Kondisi terpicu: `avg_val_recent <= 0.6 * avg_val_prev`.
- **Template Judul**: `"Nilai rata-rata pesanan {customer_name} turun signifikan"`
- **Template Narasi Penjelasan**:
  `"Rata-rata nilai belanja per transaksi pelanggan turun menjadi Rp {avg_val_recent} dalam 90 hari terakhir, dibandingkan Rp {avg_val_prev} pada periode 90 hari sebelumnya."`
- **Sumber Bukti (`insight_sources`)**: Transaksi pelanggan pada kedua periode.
- **Kondisi Supersede**: Nilai rata-rata transaksi 90 hari terakhir kembali melampaui 60% dari nilai sebelumnya.

#### 4. Aturan `CUST_COMPLAINT_OPEN_V1`
- **Signal**: Terdapat komplain pelanggan yang belum diselesaikan lebih dari 3 hari.
- **Format Dedupe Key**: `CUST_COMPLAINT_OPEN_V1:complaint_{complaint_id}`
- **Data Gate**: Baris komplain dengan status `open` atau `in_progress`.
- **Kalkulasi**: `days_open = CURRENT_DATE - DATE(complaints.created_at) > 3`.
- **Template Judul**: `"Komplain pelanggan #{complaint_id_short} belum terselesaikan selama {days_open} hari"`
- **Template Narasi Penjelasan**:
  `"Komplain dari {customer_name} mengenai '{complaint_title}' telah terbuka sejak {created_date} ({days_open} hari lalu) dan belum terselesaikan."`
- **Sumber Bukti (`insight_sources`)**:
  - Entitas komplain (`entity_type: "complaint"`).
  - Entitas transaksi tertaut jika ada (`entity_type: "transaction"`).
  - Entitas pelanggan (`entity_type: "customer"`).
- **Kondisi Supersede**: Status komplain berubah menjadi `resolved` atau `dismissed`.

#### 5. Aturan `PROD_BELOW_COST_V1`
- **Signal**: Harga jual produk aktif berada di bawah nilai HPP (Harga Pokok Penjualan) per unit aktif.
- **Format Dedupe Key**: `PROD_BELOW_COST_V1:prod_{product_id}`
- **Data Gate**: Produk berstatus aktif (`products.is_active = true`) dan memiliki versi HPP aktif (`current_cost_version_id IS NOT NULL`).
- **Kalkulasi**: `current_selling_price < hpp_per_unit`.
  - `loss_per_unit = hpp_per_unit - current_selling_price`.
- **Template Judul**: `"Harga jual {product_name} berada di bawah HPP"`
- **Template Narasi Penjelasan**:
  `"Harga jual aktif produk saat ini (Rp {current_selling_price}) lebih rendah daripada nilai HPP per unit (Rp {hpp_per_unit}). Bisnis mengalami potensi kerugian sebesar Rp {loss_per_unit} untuk setiap 1 unit yang terjual."`
- **Rekomendasi Tindakan**:
  - Judul: `"Sesuaikan harga jual produk atau efisiensikan komponen biaya HPP"`
  - Tindakan: `"Naikkan harga jual minimal menjadi Rp {hpp_per_unit} atau perbarui rincian bahan baku pada versi HPP terbaru."`
  - Pratinjau Dampak: `"Mencegah akumulasi kerugian langsung sebesar Rp {loss_per_unit} per unit penjualan."`
- **Sumber Bukti (`insight_sources`)**:
  - Entitas produk (`entity_type: "product"`).
  - Versi HPP aktif (`entity_type: "product_cost_version"`).
- **Kondisi Supersede**: Harga jual dinaikkan sehingga `current_selling_price >= hpp_per_unit` ATAU versi HPP baru diterbitkan dengan nilai HPP yang lebih rendah.

#### 6. Aturan `PROD_LOW_MARGIN_V1`
- **Signal**: Margin keuntungan produk jauh lebih rendah dibandingkan rata-rata margin usaha.
- **Format Dedupe Key**: `PROD_LOW_MARGIN_V1:prod_{product_id}`
- **Data Gate**: Produk memiliki penjualan ber-HPP (`costed_sales_units >= 10`) dalam 30 hari terakhir dan total penjualan bisnis ber-HPP > 0.
- **Kalkulasi**:
  - `product_margin_pct = (net_sales - cogs) / net_sales * 100`
  - `business_margin_pct = (biz_net_sales - biz_cogs) / biz_net_sales * 100`
  - Kondisi terpicu: `product_margin_pct <= business_margin_pct - 10.0`.
- **Template Judul**: `"Margin {product_name} tertinggal {margin_gap_pct}% di bawah rata-rata bisnis"`
- **Template Narasi Penjelasan**:
  `"Dalam 30 hari terakhir, margin keuntungan produk ini hanya {product_margin_pct}%, sementara margin rata-rata seluruh produk usaha Anda adalah {business_margin_pct}% (selisih {margin_gap_pct} poin persentase)."`
- **Sumber Bukti (`insight_sources`)**: Produk dan item transaksi penjualan 30 hari terakhir.
- **Kondisi Supersede**: Margin produk naik sehingga berada di atas `business_margin_pct - 10.0`.

#### 7. Aturan `DATA_MISSING_COST_V1`
- **Signal**: Terdapat penjualan produk yang belum dilengkapi data HPP, mendistorsi angka laba bersih.
- **Format Dedupe Key**: `DATA_MISSING_COST_V1:biz_{business_id}`
- **Data Gate**: Terdapat transaksi penjualan pada rentang 30 hari terakhir.
- **Kalkulasi**: `cost_coverage_pct < 100.0` (dihitung dari `costed_net_sales / net_sales * 100`).
- **Template Judul**: `"{missing_items_count} transaksi memiliki produk tanpa catatan HPP"`
- **Template Narasi Penjelasan**:
  `"Sebesar {uncovered_sales_pct}% dari total omzet 30 hari terakhir (Rp {uncovered_sales_amount}) belum memiliki komponen HPP. Akibatnya, nilai laba kotor bisnis Anda berstatus estimasi."`
- **Rekomendasi Tindakan**:
  - Judul: `"Lengkapi rincian biaya HPP pada produk yang belum memiliki biaya"`
  - Tindakan: `"Buka modul Katalog Produk dan masukkan komponen HPP untuk memastikan laporan laba 100% akurat."`
  - Pratinjau Dampak: `"Mengubah status kalkulasi laba kotor dari estimasi menjadi fakta riil."`
- **Sumber Bukti (`insight_sources`)**: Produk-produk yang terjual tanpa versi HPP aktif.
- **Kondisi Supersede**: `cost_coverage_pct == 100.0`.

#### 8. Aturan `DATA_UNLINKED_TX_V1`
- **Signal**: Proporsi transaksi kasir yang tidak ditautkan ke akun pelanggan melebihi ambang toleransi.
- **Format Dedupe Key**: `DATA_UNLINKED_TX_V1:biz_{business_id}`
- **Data Gate**: Total transaksi 30 hari terakhir `total_tx >= 20`.
- **Kalkulasi**: `(unlinked_tx_count / total_tx) > 0.30`.
- **Template Judul**: `"{unlinked_tx_pct}% transaksi tidak terhubung dengan data pelanggan"`
- **Template Narasi Penjelasan**:
  `"Dari {total_tx} transaksi dalam 30 hari terakhir, sebanyak {unlinked_tx_count} transaksi dicatat tanpa memilih atau mendaftarkan pelanggan. Hal ini membatasi akurasi deteksi pelanggan yang berisiko berhenti berbelanja."`
- **Rekomendasi Tindakan**:
  - Judul: `"Biasakan memilih atau mendaftarkan nomor kontak pelanggan di POS Kasir"`
  - Tindakan: `"Tanyakan nomor WhatsApp pelanggan saat pembayaran untuk mengaktifkan riwayat belanja otomatis."`
- **Sumber Bukti (`insight_sources`)**: Transaksi-transaksi non-customer dalam 30 hari terakhir.
- **Kondisi Supersede**: Proporsi transaksi tanpa pelanggan dalam 30 hari terakhir turun menjadi ≤ 30%.

---

## 3. Spesifikasi 5 Endpoint RESTful

### 3.1 Trigger Manual Analysis Run
`POST /api/v1/businesses/{business_id}/analysis/run`

Memicu eksekusi pipeline evaluasi 8 aturan deterministik secara manual dengan perlindungan cooldown & konkurensi.

#### Headers:
- `Authorization: Bearer <jwt_access_token>` (Wajib)
- `Idempotency-Key: <uuid/string>` (Opsional)

#### Request Body (JSON, Opsional):
```json
{
  "force": false
}
```
*Catatan:* Parameter `force: true` hanya dapat digunakan jika diizinkan konfigurasi tenant (secara default `false` dan tunduk pada cooldown 10 menit).

#### Response 202 Accepted:
```json
{
  "id": "0192e10a-4b92-7100-8000-000000000001",
  "business_id": "0192e10a-3a11-7000-8000-000000000001",
  "trigger": "manual",
  "status": "running",
  "rules_version": "v1.0",
  "started_at": "2026-10-18T14:30:00Z",
  "finished_at": null,
  "stats": null,
  "error_summary": null
}
```

#### Response Galat Khusus:
- **409 Conflict** (`ANALYSIS_ALREADY_RUNNING`):
  ```json
  {
    "error": {
      "code": "ANALYSIS_ALREADY_RUNNING",
      "message": "Analisis sedang berjalan untuk bisnis ini. Silakan tunggu hingga proses selesai.",
      "details": {
        "active_run_id": "0192e10a-4b92-7100-8000-000000000001",
        "started_at": "2026-10-18T14:29:45Z"
      }
    }
  }
  ```
- **429 Too Many Requests** (`ANALYSIS_COOLDOWN_ACTIVE`):
  Header tambahan: `Retry-After: 480`
  ```json
  {
    "error": {
      "code": "ANALYSIS_COOLDOWN_ACTIVE",
      "message": "Pemicuan analisis manual dibatasi 1 kali setiap 10 menit. Silakan tunggu jeda selesai.",
      "details": {
        "retry_after_seconds": 480,
        "cooldown_until": "2026-10-18T14:38:00Z",
        "last_run_finished_at": "2026-10-18T14:28:00Z"
      }
    }
  }
  ```

---

### 3.2 List Insights
`GET /api/v1/businesses/{business_id}/insights`

Mengambil daftar wawasan terpaginasi dengan penyaringan kategori, klaim, prioritas, dan status aktif.

#### Query Parameters:
| Parameter | Tipe | Wajib | Default | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| `type` | string | Tidak | - | Filter tipe: `customer_activity`, `product_profitability`, `data_health`, `period_comparison`. |
| `claim_type` | string | Tidak | - | Filter klaim: `fact`, `estimate`, `hypothesis`. |
| `priority` | string | Tidak | - | Filter prioritas: `high`, `medium`, `low`. |
| `is_active` | boolean| Tidak | `true` | `true`: hanya wawasan aktif (`superseded_at IS NULL`); `false`: termasuk yang sudah superseded. |
| `subject_type`| string | Tidak | - | Filter target: `customer`, `product`, `business`. |
| `subject_id` | uuid | Tidak | - | Filter ID spesifik entitas (misal UUID pelanggan tertentu). |
| `page` | integer| Tidak | `1` | Nomor halaman (minimal 1). |
| `page_size` | integer| Tidak | `20` | Jumlah baris per halaman (maksimal 100). |

#### Response 200 OK:
```json
{
  "items": [
    {
      "id": "0192e10a-4b92-7200-8000-000000000010",
      "rule_id": "CUST_OVERDUE_V1",
      "rule_version": "v1.0",
      "insight_type": "customer_activity",
      "claim_type": "hypothesis",
      "title": "Toko Berkah Mandiri melampaui interval belanja biasanya",
      "explanation": "Berdasarkan 8 transaksi dalam 180 hari terakhir, pelanggan biasanya berbelanja setiap 12 hari. Hingga saat ini sudah 28 hari sejak transaksi terakhir (terlambat 16 hari).",
      "data_quality": "sufficient",
      "subject_type": "customer",
      "subject_id": "0192e10a-3a11-7000-8000-000000000101",
      "subject_display": {
        "name": "Toko Berkah Mandiri",
        "identifier": "+6281234567890",
        "is_top_contributor": true
      },
      "evidence_summary": {
        "days_since_last": 28,
        "median_interval_days": 12,
        "threshold_days": 19,
        "last_transaction_date": "2026-09-20",
        "avg_ticket_size": "450000.00"
      },
      "priority": "high",
      "priority_boosted": true,
      "recommendation": {
        "id": "0192e10a-4b92-7300-8000-000000000020",
        "title": "Hubungi Toko Berkah Mandiri via WhatsApp",
        "suggested_action": "Sapa pelanggan dan tanyakan ketersediaan stok atau berikan promo belanja berkala.",
        "status": "new",
        "impact_preview": "Perkiraan potensi omzet belanja terlindungi: ~Rp 450.000 / siklus transaksi."
      },
      "calculated_at": "2026-10-18T14:15:00Z",
      "is_superseded": false,
      "superseded_at": null
    },
    {
      "id": "0192e10a-4b92-7200-8000-000000000011",
      "rule_id": "PROD_BELOW_COST_V1",
      "rule_version": "v1.0",
      "insight_type": "product_profitability",
      "claim_type": "fact",
      "title": "Harga jual Kopi Arabika 250g berada di bawah HPP",
      "explanation": "Harga jual aktif produk saat ini (Rp 45.000,00) lebih rendah daripada nilai HPP per unit (Rp 52.500,00). Bisnis mengalami potensi kerugian sebesar Rp 7.500,00 untuk setiap 1 unit yang terjual.",
      "data_quality": "sufficient",
      "subject_type": "product",
      "subject_id": "0192e10a-3a11-7000-8000-000000000201",
      "subject_display": {
        "name": "Kopi Arabika 250g",
        "identifier": "SKU-KOP-001",
        "is_top_contributor": false
      },
      "evidence_summary": {
        "selling_price": "45000.00",
        "hpp_per_unit": "52500.00",
        "loss_per_unit": "7500.00"
      },
      "priority": "high",
      "priority_boosted": false,
      "recommendation": {
        "id": "0192e10a-4b92-7300-8000-000000000021",
        "title": "Sesuaikan harga jual atau perbarui HPP produk",
        "suggested_action": "Naikkan harga jual minimal menjadi Rp 52.500,00 atau tinjau kembali rincian bahan baku HPP.",
        "status": "in_progress",
        "impact_preview": "Mencegah akumulasi kerugian langsung sebesar Rp 7.500,00 per unit penjualan."
      },
      "calculated_at": "2026-10-18T14:15:00Z",
      "is_superseded": false,
      "superseded_at": null
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 2
}
```

---

### 3.3 Get Insight Detail & Provenance
`GET /api/v1/businesses/{business_id}/insights/{id}`

Mengembalikan rincian lengkap satu wawasan, mencakup asumsi, limitasi metodologi, data rekomendasi, dan daftar bukti riwayat data mentah pembentuk (*insight sources*).

#### Response 200 OK:
```json
{
  "id": "0192e10a-4b92-7200-8000-000000000010",
  "business_id": "0192e10a-3a11-7000-8000-000000000001",
  "analysis_run_id": "0192e10a-4b92-7100-8000-000000000001",
  "rule_id": "CUST_OVERDUE_V1",
  "rule_version": "v1.0",
  "insight_type": "customer_activity",
  "claim_type": "hypothesis",
  "title": "Toko Berkah Mandiri melampaui interval belanja biasanya",
  "explanation": "Berdasarkan 8 transaksi dalam 180 hari terakhir, pelanggan biasanya berbelanja setiap 12 hari. Hingga saat ini sudah 28 hari sejak transaksi terakhir (terlambat 16 hari).",
  "evidence_snapshot": {
    "days_since_last": 28,
    "median_interval_days": 12,
    "threshold_days": 19,
    "last_transaction_date": "2026-09-20",
    "observation_period_days": 180,
    "transactions_analyzed_count": 8,
    "avg_ticket_size": "450000.00",
    "total_period_spent": "3600000.00",
    "is_top_20_pct_revenue": true
  },
  "assumptions": [
    "Interval belanja dihitung dari nilai tengah (median) tanggal transaksi untuk mengurangi distorsi belanja mendadak.",
    "Ambang batas keterlambatan ditetapkan pada toleransi max(1.5 × median, median + 7 hari)."
  ],
  "limitations": [
    "Sistem tidak memantau transaksi atau interaksi pelanggan di luar catatan aplikasi kasir Teman UMKM.",
    "Wawasan ini merupakan evaluasi tren masa lalu dan bukan jaminan kepastian perilaku masa mendatang."
  ],
  "period_start": "2026-04-21",
  "period_end": "2026-10-18",
  "data_quality": "sufficient",
  "subject_type": "customer",
  "subject_id": "0192e10a-3a11-7000-8000-000000000101",
  "subject_display": {
    "name": "Toko Berkah Mandiri",
    "identifier": "+6281234567890",
    "is_top_contributor": true
  },
  "dedupe_key": "CUST_OVERDUE_V1:cust_0192e10a-3a11-7000-8000-000000000101",
  "calculated_at": "2026-10-18T14:15:00Z",
  "superseded_at": null,
  "created_at": "2026-10-18T08:00:00Z",
  "recommendation": {
    "id": "0192e10a-4b92-7300-8000-000000000020",
    "title": "Hubungi Toko Berkah Mandiri via WhatsApp",
    "description": "Pelanggan inti telah melewati ambang batas 19 hari tanpa pesanan baru. Segera hubungi pengelola toko untuk memastikan kelancaran suplai produk.",
    "suggested_action": "Kirimkan sapaan hangat, tanyakan persediaan barang di toko mereka, dan tawarkan pengiriman jadwal reguler.",
    "reasoning": "Toko Berkah Mandiri merupakan kontributor omzet Top 20%. Mencegah jeda pemesanan lebih lanjut akan melindungi omzet berkala rata-rata Rp 450.000,00 per pesanan.",
    "priority": "high",
    "status": "new",
    "status_changed_at": "2026-10-18T08:00:00Z",
    "dismissed_reason": null,
    "impact_preview": "Perkiraan potensi omzet belanja terlindungi: ~Rp 450.000 / siklus transaksi.",
    "created_at": "2026-10-18T08:00:00Z",
    "updated_at": "2026-10-18T08:00:00Z"
  },
  "sources": [
    {
      "id": "0192e10a-4b92-7400-8000-000000000001",
      "entity_type": "transaction",
      "entity_id": "0192e10a-3a11-7000-8000-000000001001",
      "reference_no": "TRX-20260920-004",
      "date": "2026-09-20",
      "amount": "480000.00",
      "summary": "Kopi Arabika 1kg (x2)",
      "status": "paid"
    },
    {
      "id": "0192e10a-4b92-7400-8000-000000000002",
      "entity_type": "transaction",
      "entity_id": "0192e10a-3a11-7000-8000-000000001002",
      "reference_no": "TRX-20260908-012",
      "date": "2026-09-08",
      "amount": "450000.00",
      "summary": "Kopi Arabika 1kg (x1), Sirup Karamel (x2)",
      "status": "paid"
    },
    {
      "id": "0192e10a-4b92-7400-8000-000000000003",
      "entity_type": "transaction",
      "entity_id": "0192e10a-3a11-7000-8000-000000001003",
      "reference_no": "TRX-20260827-009",
      "date": "2026-08-27",
      "amount": "420000.00",
      "summary": "Kopi Arabika 1kg (x1), Kopi Robusta 1kg (x1)",
      "status": "paid"
    }
  ]
}
```

#### Response Galat Khusus:
- **404 Not Found** (`INSIGHT_NOT_FOUND`): Wawasan tidak ditemukan atau tidak berada dalam bisnis yang diakses.

---

### 3.4 List Recommendations
`GET /api/v1/businesses/{business_id}/recommendations`

Mengambil daftar seluruh rekomendasi tindakan terpaginasi untuk memfasilitasi panel *Action Items* atau *To-Do Feed*.

#### Query Parameters:
| Parameter | Tipe | Wajib | Default | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| `status` | string | Tidak | - | Filter status: `new`, `reviewed`, `in_progress`, `completed`, `dismissed`. |
| `priority` | string | Tidak | - | Filter prioritas: `high`, `medium`, `low`. |
| `insight_type`| string | Tidak | - | Filter tipe: `customer_activity`, `product_profitability`, `data_health`. |
| `page` | integer| Tidak | `1` | Nomor halaman (minimal 1). |
| `page_size` | integer| Tidak | `20` | Jumlah baris per halaman (maksimal 100). |

#### Response 200 OK:
```json
{
  "items": [
    {
      "id": "0192e10a-4b92-7300-8000-000000000020",
      "insight_id": "0192e10a-4b92-7200-8000-000000000010",
      "rule_id": "CUST_OVERDUE_V1",
      "insight_type": "customer_activity",
      "claim_type": "hypothesis",
      "subject_type": "customer",
      "subject_id": "0192e10a-3a11-7000-8000-000000000101",
      "subject_name": "Toko Berkah Mandiri",
      "title": "Hubungi Toko Berkah Mandiri via WhatsApp",
      "description": "Pelanggan inti telah melewati ambang batas 19 hari tanpa pesanan baru. Segera hubungi pengelola toko untuk memastikan kelancaran suplai produk.",
      "suggested_action": "Kirimkan sapaan hangat, tanyakan persediaan barang di toko mereka, dan tawarkan pengiriman jadwal reguler.",
      "reasoning": "Toko Berkah Mandiri merupakan kontributor omzet Top 20%. Mencegah jeda pemesanan lebih lanjut akan melindungi omzet berkala rata-rata Rp 450.000,00 per pesanan.",
      "priority": "high",
      "status": "new",
      "status_changed_at": "2026-10-18T08:00:00Z",
      "dismissed_reason": null,
      "impact_preview": "Perkiraan potensi omzet belanja terlindungi: ~Rp 450.000 / siklus transaksi.",
      "created_at": "2026-10-18T08:00:00Z",
      "updated_at": "2026-10-18T08:00:00Z"
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

---

### 3.5 Update Recommendation Status & Dismissal
`PATCH /api/v1/businesses/{business_id}/recommendations/{id}`

Memperbarui status tindakan rekomendasi (misal memulai tindakan, menandai selesai, atau mengabaikan wawasan).

#### Aturan Transisi Status (*Status State Machine*):
- `new` → `reviewed`, `in_progress`, `completed`, `dismissed`
- `reviewed` → `in_progress`, `completed`, `dismissed`
- `in_progress` → `completed`, `dismissed`
- `completed` → `in_progress` (reopen jika tindakan butuh perbaikan)
- `dismissed` → jika berstatus `dismissed`, wajib menyertakan `dismissed_reason`. Sistem otomatis mengaktifkan jeda pengabaian (*14-day cooldown*) pada `dedupe_key` wawasan terkait.

#### Pilihan Alasan Pengabaian Resmi (`dismissed_reason`):
Frontend dapat mengirimkan salah satu dari teks alasan baku atau kustom (maks 255 karakter):
1. `"Sudah dihubungi / diselesaikan di luar sistem aplikasi"`
2. `"Pelanggan memiliki jadwal/musim belanja khusus"`
3. `"Produk sedang dalam periode promo khusus / cuci gudang"`
4. `"Data belum akurat / butuh penyesuaian catatan"`
5. `"Lainnya: [catatan kustom]"`

#### Request Body (JSON) - Contoh Menandai Tindakan Sedang Berjalan:
```json
{
  "status": "in_progress"
}
```

#### Request Body (JSON) - Contoh Mengabaikan Rekomendasi (Dismissal):
```json
{
  "status": "dismissed",
  "dismissed_reason": "Sudah dihubungi / diselesaikan di luar sistem aplikasi"
}
```

#### Response 200 OK:
```json
{
  "id": "0192e10a-4b92-7300-8000-000000000020",
  "insight_id": "0192e10a-4b92-7200-8000-000000000010",
  "title": "Hubungi Toko Berkah Mandiri via WhatsApp",
  "description": "Pelanggan inti telah melewati ambang batas 19 hari tanpa pesanan baru.",
  "suggested_action": "Kirimkan sapaan hangat, tanyakan persediaan barang di toko mereka.",
  "reasoning": "Toko Berkah Mandiri merupakan kontributor omzet Top 20%.",
  "priority": "high",
  "status": "dismissed",
  "status_changed_at": "2026-10-18T14:35:00Z",
  "dismissed_reason": "Sudah dihubungi / diselesaikan di luar sistem aplikasi",
  "cooldown_until": "2026-11-01T14:35:00Z",
  "impact_preview": "Perkiraan potensi omzet belanja terlindungi: ~Rp 450.000 / siklus transaksi.",
  "created_at": "2026-10-18T08:00:00Z",
  "updated_at": "2026-10-18T14:35:00Z"
}
```

#### Response Galat Khusus:
- **400 Bad Request** (`DISMISSAL_REASON_REQUIRED`):
  ```json
  {
    "error": {
      "code": "DISMISSAL_REASON_REQUIRED",
      "message": "Alasan pengabaian wajib diisi saat status diubah menjadi dismissed."
    }
  }
  ```
- **400 Bad Request** (`INVALID_STATUS_TRANSITION`):
  ```json
  {
    "error": {
      "code": "INVALID_STATUS_TRANSITION",
      "message": "Perubahan status dari dismissed ke reviewed tidak diperbolehkan."
    }
  }
  ```
- **404 Not Found** (`RECOMMENDATION_NOT_FOUND`): Rekomendasi tidak ditemukan pada bisnis ini.

---

## 4. Definisi Go DTOs (Data Transfer Objects)

Struktur DTO Go di bawah ini diletakkan pada layer transport (`internal/insight/dto/`) untuk memastikan pemisahan tegas dari entitas model GORM.

```go
package dto

import "time"

// --- Request DTOs ---

type TriggerAnalysisRunRequest struct {
	Force bool `json:"force"`
}

type UpdateRecommendationStatusRequest struct {
	Status          string  `json:"status" binding:"required,oneof=reviewed in_progress completed dismissed"`
	DismissedReason *string `json:"dismissed_reason" binding:"required_if=Status dismissed,omitempty,max=255"`
}

type ListInsightsQuery struct {
	Type        string `form:"type" binding:"omitempty,oneof=customer_activity product_profitability data_health period_comparison"`
	ClaimType   string `form:"claim_type" binding:"omitempty,oneof=fact estimate hypothesis"`
	Priority    string `form:"priority" binding:"omitempty,oneof=high medium low"`
	IsActive    *bool  `form:"is_active"`
	SubjectType string `form:"subject_type" binding:"omitempty,oneof=customer product business"`
	SubjectID   string `form:"subject_id" binding:"omitempty,uuid"`
	Page        int    `form:"page,default=1" binding:"min=1"`
	PageSize    int    `form:"page_size,default=20" binding:"min=1,max=100"`
}

type ListRecommendationsQuery struct {
	Status      string `form:"status" binding:"omitempty,oneof=new reviewed in_progress completed dismissed"`
	Priority    string `form:"priority" binding:"omitempty,oneof=high medium low"`
	InsightType string `form:"insight_type" binding:"omitempty,oneof=customer_activity product_profitability data_health"`
	Page        int    `form:"page,default=1" binding:"min=1"`
	PageSize    int    `form:"page_size,default=20" binding:"min=1,max=100"`
}

// --- Response DTOs ---

type AnalysisRunResponse struct {
	ID           string                 `json:"id"`
	BusinessID   string                 `json:"business_id"`
	Trigger      string                 `json:"trigger"`
	Status       string                 `json:"status"`
	RulesVersion string                 `json:"rules_version"`
	StartedAt    time.Time              `json:"started_at"`
	FinishedAt   *time.Time             `json:"finished_at"`
	Stats        map[string]interface{} `json:"stats,omitempty"`
	ErrorSummary *string                `json:"error_summary,omitempty"`
}

type SubjectDisplay struct {
	Name             string `json:"name"`
	Identifier       string `json:"identifier,omitempty"`
	IsTopContributor bool   `json:"is_top_contributor"`
}

type RecommendationSummaryResponse struct {
	ID              string `json:"id"`
	Title           string `json:"title"`
	SuggestedAction string `json:"suggested_action"`
	Status          string `json:"status"`
	ImpactPreview   string `json:"impact_preview"`
}

type InsightListItemResponse struct {
	ID              string                         `json:"id"`
	RuleID          string                         `json:"rule_id"`
	RuleVersion     string                         `json:"rule_version"`
	InsightType     string                         `json:"insight_type"`
	ClaimType       string                         `json:"claim_type"`
	Title           string                         `json:"title"`
	Explanation     string                         `json:"explanation"`
	DataQuality     string                         `json:"data_quality"`
	SubjectType     string                         `json:"subject_type"`
	SubjectID       string                         `json:"subject_id"`
	SubjectDisplay  SubjectDisplay                 `json:"subject_display"`
	EvidenceSummary map[string]interface{}         `json:"evidence_summary"`
	Priority        string                         `json:"priority"`
	PriorityBoosted bool                           `json:"priority_boosted"`
	Recommendation  *RecommendationSummaryResponse `json:"recommendation,omitempty"`
	CalculatedAt    time.Time                      `json:"calculated_at"`
	IsSuperseded    bool                           `json:"is_superseded"`
	SupersededAt    *time.Time                     `json:"superseded_at"`
}

type RecommendationDetailResponse struct {
	ID              string     `json:"id"`
	InsightID       *string    `json:"insight_id,omitempty"`
	Title           string     `json:"title"`
	Description     string     `json:"description"`
	SuggestedAction string     `json:"suggested_action"`
	Reasoning       string     `json:"reasoning"`
	Priority        string     `json:"priority"`
	Status          string     `json:"status"`
	StatusChangedAt time.Time  `json:"status_changed_at"`
	DismissedReason *string    `json:"dismissed_reason"`
	CooldownUntil   *time.Time `json:"cooldown_until,omitempty"`
	ImpactPreview   string     `json:"impact_preview"`
	CreatedAt       time.Time  `json:"created_at"`
	UpdatedAt       time.Time  `json:"updated_at"`
}

type InsightSourceItemResponse struct {
	ID          string                 `json:"id"`
	EntityType  string                 `json:"entity_type"`
	EntityID    string                 `json:"entity_id"`
	ReferenceNo string                 `json:"reference_no,omitempty"`
	Date        *string                `json:"date,omitempty"`
	Amount      *string                `json:"amount,omitempty"`
	Summary     string                 `json:"summary,omitempty"`
	Status      string                 `json:"status,omitempty"`
	Metadata    map[string]interface{} `json:"metadata,omitempty"`
}

type InsightDetailResponse struct {
	ID               string                        `json:"id"`
	BusinessID       string                        `json:"business_id"`
	AnalysisRunID    string                        `json:"analysis_run_id"`
	RuleID           string                        `json:"rule_id"`
	RuleVersion      string                        `json:"rule_version"`
	InsightType      string                        `json:"insight_type"`
	ClaimType        string                        `json:"claim_type"`
	Title            string                        `json:"title"`
	Explanation      string                        `json:"explanation"`
	EvidenceSnapshot map[string]interface{}        `json:"evidence_snapshot"`
	Assumptions      []string                      `json:"assumptions"`
	Limitations      []string                      `json:"limitations"`
	PeriodStart      string                        `json:"period_start"`
	PeriodEnd        string                        `json:"period_end"`
	DataQuality      string                        `json:"data_quality"`
	SubjectType      string                        `json:"subject_type"`
	SubjectID        string                        `json:"subject_id"`
	SubjectDisplay   SubjectDisplay                `json:"subject_display"`
	DedupeKey        string                        `json:"dedupe_key"`
	CalculatedAt     time.Time                     `json:"calculated_at"`
	SupersededAt     *time.Time                    `json:"superseded_at"`
	CreatedAt        time.Time                     `json:"created_at"`
	Recommendation   *RecommendationDetailResponse `json:"recommendation,omitempty"`
	Sources          []InsightSourceItemResponse   `json:"sources"`
}

type RecommendationListItemResponse struct {
	ID              string     `json:"id"`
	InsightID       string     `json:"insight_id"`
	RuleID          string     `json:"rule_id"`
	InsightType     string     `json:"insight_type"`
	ClaimType       string     `json:"claim_type"`
	SubjectType     string     `json:"subject_type"`
	SubjectID       string     `json:"subject_id"`
	SubjectName     string     `json:"subject_name"`
	Title           string     `json:"title"`
	Description     string     `json:"description"`
	SuggestedAction string     `json:"suggested_action"`
	Reasoning       string     `json:"reasoning"`
	Priority        string     `json:"priority"`
	Status          string     `json:"status"`
	StatusChangedAt time.Time  `json:"status_changed_at"`
	DismissedReason *string    `json:"dismissed_reason"`
	ImpactPreview   string     `json:"impact_preview"`
	CreatedAt       time.Time  `json:"created_at"`
	UpdatedAt       time.Time  `json:"updated_at"`
}
```

---

## 5. Rangkuman Pemetaan Model Basis Data (`contracts/erd.md` Alignment)

Spesifikasi DTO ini selaras 100% dengan skema PostgreSQL pada `contracts/erd.md` (§5 Diagram D) untuk tabel modul intelejen:

1. **`analysis_runs`**:
   - `id` (UUID PK), `business_id` (UUID FK), `trigger` (TEXT: `schedule`, `manual`, `post_import`), `status` (TEXT: `running`, `completed`, `failed`), `rules_version` (TEXT), `started_at` (TIMESTAMPTZ), `finished_at` (TIMESTAMPTZ), `stats` (JSONB), `error_summary` (TEXT).
   - Indeks unik: `analysis_runs (business_id) WHERE status = 'running'`.

2. **`insights`**:
   - `id` (UUID PK), `business_id` (UUID FK), `analysis_run_id` (UUID FK), `rule_id` (TEXT), `rule_version` (TEXT), `insight_type` (TEXT: `customer_activity`, `product_profitability`, `data_health`, `period_comparison`), `claim_type` (TEXT: `fact`, `estimate`, `hypothesis`), `title` (TEXT), `explanation` (TEXT), `evidence_snapshot` (JSONB), `assumptions` (JSONB), `limitations` (JSONB), `period_start` (DATE), `period_end` (DATE), `data_quality` (TEXT: `sufficient`, `limited`), `subject_type` (TEXT: `customer`, `product`, `business`), `subject_id` (UUID), `dedupe_key` (TEXT), `calculated_at` (TIMESTAMPTZ), `superseded_at` (TIMESTAMPTZ), `created_at` (TIMESTAMPTZ).
   - Indeks unik: `insights (business_id, dedupe_key) WHERE superseded_at IS NULL`.

3. **`insight_sources`**:
   - `id` (UUID PK), `business_id` (UUID FK), `insight_id` (UUID FK), `entity_type` (TEXT: `transaction`, `customer`, `product`, `complaint`, `product_cost_version`), `entity_id` (UUID).
   - Menghubungkan wawasan langsung ke baris transaksi atau produk riil pembentuk bukti.

4. **`recommended_actions`**:
   - `id` (UUID PK), `business_id` (UUID FK), `insight_id` (UUID FK), `title` (TEXT), `description` (TEXT), `suggested_action` (TEXT), `reasoning` (TEXT), `priority` (TEXT: `high`, `medium`, `low`), `status` (TEXT: `new`, `reviewed`, `in_progress`, `completed`, `dismissed`), `status_changed_at` (TIMESTAMPTZ), `dismissed_reason` (TEXT), `created_at` (TIMESTAMPTZ), `updated_at` (TIMESTAMPTZ).

---

## 6. Katalog Kode Galat Mesin Wawasan (*Error Catalogue*)

Seluruh galat dikembalikan menggunakan format standar `{error: {code, message, details?}}`:

| Status HTTP | Error Code | Deskripsi Penyebab | Penanganan Klien / Frontend |
| :--- | :--- | :--- | :--- |
| `400 Bad Request` | `DISMISSAL_REASON_REQUIRED` | Status diubah menjadi `dismissed` tanpa menyertakan `dismissed_reason`. | Tampilkan validasi form agar pengguna memilih salah satu alasan. |
| `400 Bad Request` | `INVALID_STATUS_TRANSITION` | Mencoba melakukan transisi status rekomendasi yang tidak diizinkan. | Perbarui state UI dan refresh status data rekomendasi terbaru. |
| `401 Unauthorized` | `UNAUTHORIZED` | Token JWT tidak ada, kedaluwarsa, atau tidak valid. | Alihkan ke halaman masuk (`/login`). |
| `403 Forbidden` | `FORBIDDEN_TENANT_ACCESS` | Pengguna tidak memiliki akses ke `business_id` target. | Tampilkan peringatan otorisasi dan cegah akses. |
| `404 Not Found` | `INSIGHT_NOT_FOUND` | Wawasan dengan ID tersebut tidak ditemukan pada bisnis ini. | Tampilkan pesan wawasan tidak ditemukan / kembali ke daftar. |
| `404 Not Found` | `RECOMMENDATION_NOT_FOUND` | Rekomendasi dengan ID tersebut tidak ditemukan. | Tampilkan pesan tidak ditemukan. |
| `409 Conflict` | `ANALYSIS_ALREADY_RUNNING` | Sedang ada analisis aktif yang belum selesai untuk bisnis ini. | Tampilkan indikator proses evaluasi sedang berlangsung. |
| `422 Unprocessable`| `VALIDATION_ERROR` | Format parameter query atau JSON body tidak sesuai schema DTO. | Periksa rincian kolom galat pada objek `details`. |
| `429 Too Many Req` | `ANALYSIS_COOLDOWN_ACTIVE` | Pemicuan analisis manual dilakukan sebelum jeda 10 menit selesai. | Tampilkan timer hitung mundur sesuai nilai `Retry-After`. |
| `500 Internal Error`| `INTERNAL_SERVER_ERROR` | Terjadi galat internal pada evaluasi aturan atau database. | Tampilkan pesan ramah pengguna tanpa membocorkan SQL / stack trace. |
