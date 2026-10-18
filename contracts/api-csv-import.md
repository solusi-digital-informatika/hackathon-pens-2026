# Kontrak REST API: CSV Import, Column Mapping, Row Validation & Async Worker Execution (M5)

## Metadata
- **ID Kontrak**: API-M5-001
- **Task ID**: T-051
- **Penulis**: Backend & AI Engineer (`backend-ai`) / Systems Architect
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED
- **Target Audience**: Frontend Engineer (`frontend`), Backend & AI Engineer (`backend-ai`), QA Engineer (`tester`)
- **Dokumen Terkait**: 
  - `project-map/trd.md` (§8 CSV Import Spec, §9 API Inventory, §10 Security, TD-07 Async Work, TD-11 Audit)
  - `contracts/erd.md` (§5 Diagram D: `import_batches`, `import_rows`, enums, composite foreign keys)
  - `contracts/ui-csv-import.md` (UI-M5-001: 5-Step CSV Import Wizard)
- **Base Prefix**: `/api/v1/businesses/{business_id}/imports`

---

## 1. Aturan Dasar & Keamanan Multi-Tenant

1. **Multi-Tenant Boundary**:
   - Seluruh endpoint berada di bawah prefix `/api/v1/businesses/{business_id}/imports`.
   - Middleware `RequireAuth` (memvalidasi JWT access token) dan `RequireTenant` (memvalidasi keanggotaan aktif pengguna pada `business_id`) **WAJIB** aktif di setiap rute.
   - Semua operasi database wajib menyertakan filter `business_id = :business_id` (menggunakan composite foreign keys sesuai `contracts/erd.md` §7: `import_rows (business_id, import_batch_id)`).

2. **Siklus Hidup Status Batch (Batch Lifecycle States)**:
   ```
   uploaded ──> mapped ──> validated ──> confirmed ──> processing ──> completed | completed_with_errors | failed
      │            │            │            │
      └───[cancel]─┴───[cancel]─┴───[cancel]─┴───[cancel]──> cancelled
                                                                  ▲
      completed | completed_with_errors ───────[rollback]─────────┘
   ```
   - `uploaded`: File CSV telah diunggah, di-stream, dan baris mentah disimpan ke `import_rows` dengan status `pending`.
   - `mapped`: Pengguna telah mengirimkan pemetaan kolom dan preferensi format parsing.
   - `validated`: Validasi baris telah selesai dijalankan; jumlah valid, warning, dan error telah dihitung.
   - `confirmed`: Pengguna telah menyetujui impor; batch siap diambil oleh async background worker.
   - `processing`: Worker sedang mengeksekusi mutasi baris data ke database produksi (chunk-by-chunk).
   - `completed`: Seluruh baris valid berhasil diimpor tanpa galat (100% sukses).
   - `completed_with_errors`: Sebagian baris valid berhasil diimpor, baris galat dilewati (*skipped*).
   - `failed`: Terjadi kendala fatal sistem/database yang menggagalkan proses impor.
   - `cancelled`: Impor dibatalkan oleh pengguna (sebelum diproses atau di-rollback setelah selesai).

3. **Siklus Hidup Status Baris (`import_rows.status`)**:
   - `pending`: Baris baru diunggah, belum divalidasi dengan pemetaan final.
   - `valid`: Baris lulus validasi dan siap diimpor.
   - `warning`: Baris memiliki catatan/penyesuaian (misal nomor telepon dinormalisasi atau kemungkinan duplikat), tetapi tetap dapat diimpor.
   - `error`: Baris gagal validasi bisnis/skema; wajib dilewati saat eksekusi.
   - `imported`: Baris berhasil dikonversi menjadi entitas bisnis (`entity_type`, `entity_id`).
   - `skipped`: Baris berstatus error atau duplikat yang diabaikan saat eksekusi final.

4. **Keamanan Berkas Mentah (*Zero Raw File Storage*)**:
   - File fisik CSV **TIDAK DISIMPAN** pada disk server maupun public cloud storage (S3/GCS) guna mencegah kebocoran data dan eksekusi file berbahaya.
   - File di-parse secara streaming di memori, disimpan dalam bentuk JSONB pada tabel staging `import_rows (raw_data)`, lalu ringkasan hash SHA-256 dicatat pada `import_batches (file_sha256)`.
   - Batas file streaming (diperiksa saat pembacaan byte): **Maksimal 5 MB** (`5 * 1024 * 1024` bytes) dan **Maksimal 20.000 baris**. Jika melebihi batas, streaming dihentikan seketika dan request ditolak dengan kode `FILE_TOO_LARGE` atau `ROW_LIMIT_EXCEEDED`.

5. **Penanganan UTF-8 BOM & Pembatas Kolom (*Delimiter*)**:
   - Sistem membaca UTF-8 saja. Byte Order Mark (BOM `0xEF, 0xBB, 0xBF`) dihapus otomatis (*stripped*) pada awal stream.
   - Deteksi delimiter otomatis menganalisis baris pertama untuk memilih antara koma (`,`), titik koma (`;`), atau tab (`\t`). Microsoft Excel pada konfigurasi regional Indonesia umumnya mengekspor CSV menggunakan pembatas titik koma (`;`). Pengguna dapat melakukan override pembatas pada saat upload maupun pemetaan.

6. **Netralisasi Injeksi Formula CSV (*Anti-Formula Injection / CSV DDE Injection*)**:
   - Sel pada file CSV diperlakukan sebagai teks mentah literal, tidak pernah dievaluasi sebagai ekspresi.
   - Pada endpoint pengunduhan laporan baris galat (`GET .../errors/download`), seluruh sel teks yang diawali karakter berbahaya:
     - `=` (Formula Excel)
     - `+` (Operator Tambah)
     - `-` (Operator Kurang)
     - `@` (Deklarasi Fungsi / Decorator)
     - `\t` (Tab) atau `\r` (Carriage return)
     **Wajib dinetralkan** dengan menambahkan tanda kutip tunggal (`'`) di depan karakter tersebut sebelum diekspor.

---

## 2. Katalog Entitas & Spesifikasi Kolom Template

### 2.1 Katalog Produk (`products`)
- Kolom Template:
  1. `name` (string, **Wajib**): Nama produk (maks 255 karakter).
  2. `sku` (string, Opsional): SKU unik produk per bisnis.
  3. `category` (string, Opsional): Kategori produk (default: `"Umum"` jika kosong).
  4. `unit` (string, Opsional): Satuan unit (contoh: `"Pcs"`, `"Cup"`, `"Kg"`). Default: `"Pcs"`.
  5. `selling_price` (angka/desimal, **Wajib**): Harga jual satuan (harus $\ge 0$).
  6. `hpp_per_unit` (angka/desimal, Opsional): Estimasi modal/HPP awal. Jika diisi $\ge 0$, sistem otomatis membuat `product_cost_versions` perdana dengan 1 komponen `HPP (impor)`.

### 2.2 Daftar Pelanggan (`customers`)
- Kolom Template:
  1. `name` (string, **Wajib**): Nama lengkap / display name pelanggan.
  2. `phone` (string, Opsional): Nomor telepon/WhatsApp. Dinormalisasi ke format E.164 (`08...` $\rightarrow$ `+628...`).
  3. `email` (string, Opsional): Alamat email valid.
  4. `notes` (string, Opsional): Catatan preferensi atau riwayat pelanggan.

### 2.3 Riwayat Transaksi Penjualan (`transactions`)
- Format: Satu baris per item transaksi (*flat transaction item*). Baris yang memiliki nilai `order_ref` yang sama dikelompokkan ke dalam satu transaksi penjualan tunggal (`transactions`). Jika `order_ref` kosong, setiap baris dianggap sebagai 1 transaksi terpisah.
- Kolom Template:
  1. `order_ref` (string, Opsional): Kode referensi transaksi eksternal (contoh: `"TRX-202610-001"`). Baris dengan `order_ref` sama dalam 1 file akan digabung menjadi 1 invoice transaksi.
  2. `date` (string tanggal, **Wajib**): Tanggal/waktu transaksi sesuai format yang dipilih (`DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`).
  3. `customer_name` (string, Opsional): Nama pelanggan.
  4. `customer_phone` (string, Opsional): Nomor telepon pelanggan. Digunakan sebagai kunci unik identifikasi pelanggan (`+628...`).
  5. `product_sku` (string, Opsional): SKU produk yang terdaftar di katalog.
  6. `product_name` (string, **Wajib** jika SKU kosong): Nama produk jika SKU tidak tersedia.
  7. `quantity` (angka/desimal, **Wajib**): Jumlah barang (harus $> 0$).
  8. `unit_price` (angka/desimal, **Wajib**): Harga jual satuan (harus $\ge 0$).
  9. `discount` (angka/desimal, Opsional): Nilai diskon item (harus $\ge 0$ dan $\le quantity \times unit\_price$). Default: `0`.
  10. `payment_status` (string, Opsional): `paid`, `partial`, `unpaid`, `unknown`. Default: `paid`.
  11. `fee` (angka/desimal, Opsional): Biaya transaksi/ongkir/admin level order (hanya dialokasikan sekali per `order_ref`). Default: `0`.

---

## 3. Aturan Validasi Bisnis & Katalog Kode Galat

Setiap baris divalidasi secara independen. Hasil validasi menghasilkan array `errors` dan `warnings` berformat:
`{ "code": "STRING_CODE", "field": "nama_kolom", "message": "Pesan dalam Bahasa Indonesia" }`.

### 3.1 Aturan Pencocokan Produk (*Product Matching Rule*)
1. Pencocokan dilakukan pertama kali melalui `product_sku` (pencocokan persis / *exact match*, case-insensitive).
2. Jika SKU tidak diisi atau tidak cocok, sistem mencocokkan melalui `product_name` (pencocokan persis hasil *trimmed string*, case-insensitive).
3. **DILARANG menggunakan pencocokan fuzzy (Fuzzy Matching Dilarang)**: Untuk mencegah kekeliruan varian produk dan selisih HPP, ketidakcocokan nama persis menghasilkan galat `PRODUCT_NOT_FOUND`.

### 3.2 Aturan Pencocokan Pelanggan (*Customer Matching Rule*)
1. Pencocokan pelanggan lama **HANYA** dilakukan berdasarkan nomor telepon yang telah dinormalisasi ke standar internasional E.164 (`+62...`).
2. **DILARANG melakukan auto-merge berdasarkan kemiripan nama saja**:
   - Jika `customer_phone` cocok dengan pelanggan terdaftar, transaksi otomatis ditautkan ke pelanggan tersebut.
   - Jika `customer_phone` kosong atau berbeda, namun `customer_name` serupa/identik dengan pelanggan terdaftar, sistem **TIDAK MENGGABUNGKANNYA**. Sistem akan membuat entitas pelanggan baru dan menerbitkan peringatan `CUSTOMER_NAME_SIMILAR` (*"Kemungkinan pelanggan yang sama dengan [Nama Terdaftar]. Dibuat sebagai pelanggan terpisah untuk mencegah percampuran data"*).

### 3.3 Katalog Kode Galat (*Error Codes*)
| Kode Galat | Field | Pesan Bahasa Indonesia | Status Baris |
|---|---|---|---|
| `FIELD_REQUIRED` | `[nama_field]` | "Kolom wajib ini tidak boleh kosong." | `error` |
| `INVALID_NUMBER_FORMAT` | `selling_price`, `quantity`, dll | "Format angka tidak valid sesuai pengaturan format yang dipilih." | `error` |
| `NUMBER_OUT_OF_RANGE` | `quantity`, `selling_price` | "Nilai angka harus lebih besar dari 0." | `error` |
| `DISCOUNT_EXCEEDS_PRICE` | `discount` | "Nilai diskon tidak boleh melebihi total harga item." | `error` |
| `INVALID_DATE_FORMAT` | `date` | "Format tanggal tidak sesuai dengan pola yang ditentukan ({format})." | `error` |
| `PRODUCT_NOT_FOUND` | `product_sku` / `product_name` | "Produk dengan SKU '{sku}' atau nama '{name}' tidak ditemukan di katalog." | `error` |
| `DUPLICATE_ORDER_REF_IN_DB` | `order_ref` | "Kode referensi transaksi '{order_ref}' sudah pernah dicatat sebelumnya di sistem." | `error` |
| `INVALID_PAYMENT_STATUS` | `payment_status` | "Status pembayaran tidak valid. Pilihan: paid, partial, unpaid, unknown." | `error` |
| `INVALID_EMAIL_FORMAT` | `email` | "Format alamat email tidak valid." | `error` |
| `DUPLICATE_SKU_IN_FILE` | `sku` | "SKU '{sku}' terulang lebih dari satu kali dalam berkas ini." | `error` |

### 3.4 Katalog Kode Peringatan (*Warning Codes*)
| Kode Peringatan | Field | Pesan Bahasa Indonesia | Dampak Eksekusi |
|---|---|---|---|
| `PHONE_NORMALIZED` | `customer_phone` | "Nomor telepon dinormalisasi otomatis ke format internasional (+62...)." | Tetap diimpor |
| `NUMBER_FORMAT_AMBIGUOUS` | `[field]` | "Nilai '{value}' ambigu antara ribuan atau desimal. Ditafsirkan sesuai format {format}." | Tetap diimpor |
| `CUSTOMER_NAME_SIMILAR` | `customer_name` | "Nama pelanggan mirip dengan kontak terdaftar. Dibuat pelanggan baru secara terpisah." | Tetap diimpor (buat baru) |
| `HPP_NOT_PROVIDED` | `hpp_per_unit` | "HPP tidak dicantumkan. Laba kotor produk ini akan dihitung estimasi sampai HPP diisi." | Tetap diimpor |
| `CATEGORY_DEFAULTED` | `category` | "Kategori kosong, dialokasikan ke kategori 'Umum'." | Tetap diimpor |

---

## 4. Protokol Eksekusi Asinkron & Basis Data sebagai Antrean (TD-07)

Pemrosesan batch berukuran besar (hingga 20.000 baris) dijalankan secara asinkron di latar belakang menggunakan PostgreSQL sebagai antrean (*Database as Queue*) sesuai arsitektur **TRD TD-07**:

### 4.1 Mekanisme Pengambilan Job (*Worker Claiming with SKIP LOCKED*)
Worker mengambil batch yang siap diproses menggunakan query atomik:
```sql
BEGIN;
SELECT id, business_id, import_type, column_mapping, format_options
FROM import_batches
WHERE status = 'confirmed'
ORDER BY created_at ASC
LIMIT 1
FOR UPDATE SKIP LOCKED;

-- Jika batch ditemukan:
UPDATE import_batches
SET status = 'processing',
    started_at = NOW()
WHERE id = :batch_id;

COMMIT;
```
- Multi-instance safety: `SKIP LOCKED` menjamin tidak ada 2 worker yang memproses batch yang sama secara bersamaan.
- PostgreSQL Advisory Lock: `SELECT pg_try_advisory_lock(hashtext('import_worker_scheduler'))` digunakan pada cron loop scheduler untuk mencegah duplikasi pemanggilan rutin scheduler.

### 4.2 Pemrosesan per Chunk & Savepoint per Baris (*Row-level Savepoint*)
Worker memproses baris dalam chunk berukuran terukur (default: **200 baris per iterasi**):
1. Mengambil baris:
   ```sql
   SELECT id, row_number, raw_data
   FROM import_rows
   WHERE import_batch_id = :batch_id AND status = 'valid'
   ORDER BY row_number ASC
   LIMIT 200;
   ```
2. Setiap baris dieksekusi di dalam `SAVEPOINT`:
   ```sql
   SAVEPOINT import_row_savepoint;

   -- Eksekusi mutasi entitas (INSERT transactions, INSERT products, dll)
   -- Jika berhasil:
   UPDATE import_rows 
   SET status = 'imported', entity_type = :entity_type, entity_id = :entity_id
   WHERE id = :row_id;
   RELEASE SAVEPOINT import_row_savepoint;

   -- Jika gagal karena constraint/runtime error:
   ROLLBACK TO SAVEPOINT import_row_savepoint;
   UPDATE import_rows 
   SET status = 'error', errors = :error_json
   WHERE id = :row_id;
   ```
3. Menjamin isolasi kegagalan: Galat pada satu baris **TIDAK PERNAH** membatalkan baris-baris valid lainnya dalam batch yang sama.

### 4.3 Penyelesaian Batch (*Finalizing Batch*)
Setelah seluruh baris selesai diproses:
- Jika `error_rows == 0`: batch diubah menjadi `status = 'completed'`.
- Jika `error_rows > 0` dan `imported_rows > 0`: batch diubah menjadi `status = 'completed_with_errors'`.
- Jika `imported_rows == 0` dan seluruh baris gagal: batch diubah menjadi `status = 'failed'`.
- Menyetel `completed_at = NOW()`, `imported_rows = :count`.
- Mencatat entri audit log permanen pada `audit_logs` (TD-11).

### 4.4 Graceful Shutdown
Worker menangkap sinyal OS (`SIGINT`, `SIGTERM`). Ketika sinyal diterima:
- Worker menyelesaikan chunk yang sedang aktif berjalan hingga commit savepoint terakhir.
- Tidak mengambil chunk baru.
- Melepaskan advisory lock dan keluar secara bersih tanpa meninggalkan state database menggantung (*dangling processing lock*).

---

## 5. Spesifikasi Mekanisme Pembatalan & Rollback

### 5.1 Pembatalan Sebelum Eksekusi (*Pre-Execution Cancellation*)
Dapat dipanggil ketika batch berada pada status: `uploaded`, `mapped`, `validated`, atau `confirmed`.
- Status batch langsung diubah menjadi `cancelled`.
- Baris-baris pada `import_rows` tidak diproses lebih lanjut oleh worker.
- Jika batch sudah berada dalam antrean `confirmed`, worker akan melewati (*skip*) batch berstatus `cancelled`.

### 5.2 Pembatalan & Rollback Paska Eksekusi (*Post-Execution Rollback*)
Dapat dipanggil ketika batch berada pada status: `completed` atau `completed_with_errors`.
- Sesuai **TRD §8**: Pembatalan batch transaksi penjualan akan membatalkan seluruh transaksi yang dibuat oleh batch tersebut dan memulihkan stok barang.
- **Langkah Mutasi Atomik Rollback (Single Transaction)**:
  1. Cari seluruh transaksi yang dibuat oleh batch ini:
     `SELECT id FROM transactions WHERE business_id = :bid AND import_batch_id = :batch_id AND status = 'valid' FOR UPDATE;`
  2. Untuk setiap transaksi, pulihkan stok produk dari setiap item transaksi:
     ```sql
     UPDATE products p
     SET current_stock = p.current_stock + ti.quantity - ti.quantity_returned
     FROM transaction_items ti
     WHERE ti.transaction_id = :trx_id AND p.id = ti.product_id;
     ```
  3. Ubah status transaksi menjadi dibatalkan:
     ```sql
     UPDATE transactions
     SET status = 'cancelled',
         cancelled_at = NOW(),
         cancel_reason = 'Rollback import batch ' || :batch_id
     WHERE id IN (:transaction_ids);
     ```
  4. Perbarui status batch menjadi `cancelled`.
  5. Catat log audit aksi destruktif: `ACTION_IMPORT_ROLLBACK` mencakup jumlah transaksi yang divoid dan user ID pelaku.
- Persyaratan Keamanan Eksekusi: Klien wajib menyertakan payload `{ "confirmation_phrase": "BATALKAN" }` untuk mengeksekusi rollback paska-eksekusi.

---

## 6. Spesifikasi Endpoint

### 6.1 Unduh Template CSV Resmi
`GET /api/v1/businesses/{business_id}/imports/templates/{type}`

Mengunduh berkas template CSV resmi dengan header terstandar untuk memandu pengisian data pengguna.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `type` (string, wajib): Jenis entitas. Nilai yang valid:
  - `products`
  - `customers`
  - `transactions`

#### Query Parameters:
- `delimiter` (string, opsional, default: `comma`): Pilihan format pembatas file template:
  - `comma` (Koma `,`)
  - `semicolon` (Titik koma `;`)

#### Response 200 OK:
- Header:
  - `Content-Type: text/csv; charset=utf-8`
  - `Content-Disposition: attachment; filename="template-{type}.csv"`
- Payload Body (Contoh `type = products`, delimiter koma):
  ```csv
  name,sku,category,unit,selling_price,hpp_per_unit
  Kopi Susu Aren,KOP-001,Minuman,Cup,18000,8500
  Roti Bakar Cokelat,ROT-001,Makanan,Porsi,15000,7000
  ```

#### Error Responses:
- `400 Bad Request`:
  ```json
  {
    "error": {
      "code": "INVALID_TEMPLATE_TYPE",
      "message": "Jenis template tidak valid. Pilihan yang tersedia: products, customers, transactions."
    }
  }
  ```

---

### 6.2 Unggah Berkas CSV Impor
`POST /api/v1/businesses/{business_id}/imports`

Menerima berkas CSV secara streaming, melakukan parsing baris mentah, mendeteksi header & pembatas kolom, menghitung hash SHA-256 untuk mendeteksi unggahan berulang, dan menyimpan baris staging awal ke `import_rows`.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.

#### Request Headers:
- `Content-Type: multipart/form-data`

#### Form-Data Fields:
- `file` (file binary, wajib): Berkas CSV (Maks 5 MB, maks 20.000 baris).
- `import_type` (string, wajib): `products`, `customers`, atau `transactions`.
- `delimiter` (string, opsional, default: `auto`): `auto`, `comma` (`,`), `semicolon` (`;`), `tab` (`\t`).

#### Response 201 Created:
```json
{
  "id": "0192a000-7000-7000-8000-000000000001",
  "business_id": "018f23a0-0000-7000-8000-000000000000",
  "import_type": "products",
  "original_filename": "katalog_produk_oktober.csv",
  "file_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "file_size_bytes": 1048576,
  "status": "uploaded",
  "detected_delimiter": ";",
  "detected_headers": [
    "Nama Barang",
    "Kode SKU",
    "Kategori Produk",
    "Satuan",
    "Harga Jual",
    "Estimasi Modal HPP"
  ],
  "suggested_mapping": {
    "name": "Nama Barang",
    "sku": "Kode SKU",
    "category": "Kategori Produk",
    "unit": "Satuan",
    "selling_price": "Harga Jual",
    "hpp_per_unit": "Estimasi Modal HPP"
  },
  "is_duplicate_hash": false,
  "previous_batch": null,
  "total_rows": 150,
  "sample_rows": [
    {
      "row_number": 2,
      "raw_values": {
        "Nama Barang": "Kopi Susu Aren",
        "Kode SKU": "KOP-001",
        "Kategori Produk": "Minuman",
        "Satuan": "Cup",
        "Harga Jual": "18.000",
        "Estimasi Modal HPP": "8.500"
      }
    },
    {
      "row_number": 3,
      "raw_values": {
        "Nama Barang": "Croissant Butter",
        "Kode SKU": "BAK-002",
        "Kategori Produk": "Pastry",
        "Satuan": "Pcs",
        "Harga Jual": "25.000",
        "Estimasi Modal HPP": "12.000"
      }
    }
  ],
  "created_at": "2026-10-18T10:00:00Z"
}
```

*Kondisi Peringatan File Duplikat (SHA-256 Identik)*:
Jika hash cocok dengan batch yang pernah diproses sebelumnya pada bisnis yang sama, `is_duplicate_hash` bernilai `true` dan menyertakan rincian `previous_batch`:
```json
{
  "is_duplicate_hash": true,
  "previous_batch": {
    "id": "01929000-5000-7000-8000-000000000099",
    "status": "completed",
    "original_filename": "katalog_produk_lama.csv",
    "created_at": "2026-10-15T14:20:00Z"
  }
}
```

#### Error Responses:
- `400 Bad Request` — Ukuran file melebihi batas atau bukan CSV:
  ```json
  {
    "error": {
      "code": "FILE_TOO_LARGE",
      "message": "Ukuran file melebihi batas maksimal 5 MB."
    }
  }
  ```
- `400 Bad Request` — Melebihi batas jumlah baris:
  ```json
  {
    "error": {
      "code": "ROW_LIMIT_EXCEEDED",
      "message": "Jumlah baris melebihi batas maksimal 20.000 baris."
    }
  }
  ```
- `422 Unprocessable Entity` — Header file CSV tidak ditemukan:
  ```json
  {
    "error": {
      "code": "EMPTY_OR_INVALID_CSV",
      "message": "Berkas CSV kosong atau tidak memiliki baris header yang valid."
    }
  }
  ```

---

### 6.3 Ambil Status & Rincian Batch Impor
`GET /api/v1/businesses/{business_id}/imports/{id}`

Mendapatkan progres pemrosesan, ringkasan jumlah baris valid/galat, dan konfigurasi batch saat ini. Endpoint ini digunakan frontend untuk polling kemajuan asinkron.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Response 200 OK:
```json
{
  "id": "0192a000-7000-7000-8000-000000000001",
  "business_id": "018f23a0-0000-7000-8000-000000000000",
  "import_type": "transactions",
  "original_filename": "penjualan_pos_minggu3.csv",
  "file_sha256": "f4b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b123",
  "file_size_bytes": 2450120,
  "status": "processing",
  "column_mapping": {
    "order_ref": "No Pesanan",
    "date": "Tanggal Transaksi",
    "customer_name": "Nama Pembeli",
    "customer_phone": "No WhatsApp",
    "product_sku": "SKU Item",
    "product_name": "Nama Item",
    "quantity": "Jumlah",
    "unit_price": "Harga Satuan",
    "discount": "Diskon",
    "payment_status": "Status Bayar",
    "fee": "Biaya Layanan"
  },
  "format_options": {
    "delimiter": ";",
    "number_format": "id_ID",
    "date_format": "DD/MM/YYYY"
  },
  "total_rows": 2000,
  "valid_rows": 1950,
  "warning_rows": 120,
  "error_rows": 50,
  "imported_rows": 1360,
  "progress_pct": 68.0,
  "started_at": "2026-10-18T10:15:00Z",
  "completed_at": null,
  "error_summary": null,
  "created_at": "2026-10-18T10:00:00Z"
}
```

#### Error Responses:
- `404 Not Found`:
  ```json
  {
    "error": {
      "code": "IMPORT_BATCH_NOT_FOUND",
      "message": "Batch impor tidak ditemukan pada bisnis ini."
    }
  }
  ```

---

### 6.4 Simpan Pemetaan Kolom & Jalankan Validasi Baris
`PUT /api/v1/businesses/{business_id}/imports/{id}/mapping`

Menyimpan konfigurasi pemetaan kolom sumber CSV ke kolom target sistem, menentukan preferensi parsing angka dan tanggal, serta memicu proses audit validasi terhadap seluruh baris staging `import_rows`.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Request Body:
```json
{
  "column_mapping": {
    "order_ref": "No Pesanan",
    "date": "Tanggal Transaksi",
    "customer_name": "Nama Pembeli",
    "customer_phone": "No WhatsApp",
    "product_sku": "SKU Item",
    "product_name": "Nama Item",
    "quantity": "Jumlah",
    "unit_price": "Harga Satuan",
    "discount": "Diskon",
    "payment_status": "Status Bayar",
    "fee": null
  },
  "format_options": {
    "delimiter": ";",
    "number_format": "id_ID",
    "date_format": "DD/MM/YYYY"
  }
}
```

*Keterangan Format Options*:
- `number_format`:
  - `id_ID`: Titik (`.`) adalah pemisah ribuan, koma (`,`) adalah desimal (contoh: `1.500,50`).
  - `en_US`: Koma (`,`) adalah pemisah ribuan, titik (`.`) adalah desimal (contoh: `1,500.50`).
- `date_format`: `DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`.

#### Logika Eksekusi Server:
1. Validasi bahwa seluruh kolom target yang bertanda **Wajib** telah dipetakan (tidak bernilai `null` atau kosong).
2. Perbarui `import_batches.column_mapping` dan `import_batches.format_options`.
3. Jalankan audit validasi baris pada `import_rows`:
   - Lakukan interpretasi nilai angka dan tanggal berdasarkan `format_options`.
   - Cek keberadaan SKU produk / kecocokan nama persis di tabel `products`.
   - Normalisasi nomor telepon ke format E.164.
   - Deteksi potensi duplikasi `order_ref` dan format ambigu.
   - Simpan hasil temuan ke `import_rows.errors` dan `import_rows.warnings`.
   - Perbarui status baris: `valid`, `warning`, atau `error`.
4. Hitung ulang metrik `total_rows`, `valid_rows`, `warning_rows`, `error_rows`.
5. Ubah status batch menjadi `validated`.

#### Response 200 OK:
```json
{
  "id": "0192a000-7000-7000-8000-000000000001",
  "status": "validated",
  "total_rows": 2000,
  "valid_rows": 1950,
  "warning_rows": 120,
  "error_rows": 50,
  "sample_validated_rows": [
    {
      "row_number": 2,
      "status": "valid",
      "summary": "TRX-1001 | Kopi Susu Aren (SKU-001) | Qty: 2",
      "errors": [],
      "warnings": [
        {
          "code": "PHONE_NORMALIZED",
          "field": "customer_phone",
          "message": "Nomor WhatsApp dinormalisasi otomatis dari '08123456789' menjadi '+628123456789'."
        }
      ]
    },
    {
      "row_number": 14,
      "status": "error",
      "summary": "TRX-1014 | Matcha Latte (MAT-999) | Qty: 1",
      "errors": [
        {
          "code": "PRODUCT_NOT_FOUND",
          "field": "product_sku",
          "message": "SKU 'MAT-999' tidak ditemukan pada katalog produk."
        }
      ],
      "warnings": []
    }
  ]
}
```

#### Error Responses:
- `422 Unprocessable Entity` — Kolom wajib belum dipetakan:
  ```json
  {
    "error": {
      "code": "REQUIRED_COLUMNS_UNMAPPED",
      "message": "Beberapa kolom wajib belum dipetakan.",
      "details": [
        {
          "field": "date",
          "message": "Kolom tanggal transaksi wajib dipetakan."
        },
        {
          "field": "unit_price",
          "message": "Kolom harga satuan wajib dipetakan."
        }
      ]
    }
  }
  ```

---

### 6.5 Daftar Baris Data Impor Terpaginasi
`GET /api/v1/businesses/{business_id}/imports/{id}/rows`

Menampilkan daftar baris hasil validasi dengan dukungan filter status (`valid`, `warning`, `error`, `imported`, `skipped`), pencarian teks, dan paginasi standar.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Query Parameters:
- `status` (string, opsional, default: `all`): Filter status baris (`all`, `valid`, `warning`, `error`, `pending`, `imported`, `skipped`).
- `search` (string, opsional): Pencarian berbasis teks pada nomor baris, SKU, nama produk, nomor telepon, atau pesan galat.
- `page` (integer, opsional, default: `1`): Nomor halaman.
- `page_size` (integer, opsional, default: `20`, maks: `100`): Jumlah data per halaman.

#### Response 200 OK:
```json
{
  "items": [
    {
      "id": "0192a000-8000-7000-8000-000000000014",
      "import_batch_id": "0192a000-7000-7000-8000-000000000001",
      "row_number": 14,
      "raw_data": {
        "No Pesanan": "TRX-1014",
        "Tanggal Transaksi": "18/10/2026",
        "Nama Pembeli": "Budi Santoso",
        "No WhatsApp": "081298765432",
        "SKU Item": "MAT-999",
        "Nama Item": "Matcha Latte",
        "Jumlah": "1",
        "Harga Satuan": "24.000",
        "Diskon": "0"
      },
      "parsed_data": {
        "order_ref": "TRX-1014",
        "date": "2026-10-18T00:00:00Z",
        "customer_name": "Budi Santoso",
        "customer_phone": "+6281298765432",
        "product_sku": "MAT-999",
        "product_name": "Matcha Latte",
        "quantity": "1.000",
        "unit_price": "24000.00",
        "discount": "0.00"
      },
      "status": "error",
      "errors": [
        {
          "code": "PRODUCT_NOT_FOUND",
          "field": "product_sku",
          "message": "SKU 'MAT-999' tidak ditemukan pada katalog produk."
        }
      ],
      "warnings": [],
      "row_hash": "a1b2c3d4e5f67890...",
      "entity_type": null,
      "entity_id": null
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 50,
    "total_pages": 3
  }
}
```

---

### 6.6 Konfirmasi & Pemicu Eksekusi Asinkron
`POST /api/v1/businesses/{business_id}/imports/{id}/confirm`

Mengonfirmasi bahwa pengguna menyetujui hasil validasi dan memerintahkan worker latar belakang untuk memproses baris yang valid ke dalam sistem pembukuan.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Request Headers:
- `Idempotency-Key` (string UUID/teks unik, opsional namun direkomendasikan): Mencegah submit ganda akibat kendala jaringan.

#### Request Body:
```json
{
  "skip_errors": true
}
```
- `skip_errors` (boolean, opsional, default: `true`): Menyatakan persetujuan bahwa baris-baris berstatus `error` akan dilewati (*skipped*) dan tidak menghalangi impor baris valid lainnya.

#### Logika Internal:
1. Periksa `idempotency_keys` jika header disertakan.
2. Buka transaksi database dengan row lock:
   `SELECT status, valid_rows FROM import_batches WHERE id = :id AND business_id = :bid FOR UPDATE;`
3. Validasi status batch:
   - Harus berstatus `validated` (atau sudah `confirmed` jika merupakan retry idempoten).
   - `valid_rows` harus $> 0$. Jika `valid_rows == 0`, tolak dengan kode `NO_VALID_ROWS_TO_IMPORT`.
4. Ubah status batch: `UPDATE import_batches SET status = 'confirmed' WHERE id = :id;`.
5. Catat audit log permanen: `ACTION_IMPORT_CONFIRMED`.
6. Commit transaksi. Worker akan mengambil batch ini pada siklus polling berikutnya.

#### Response 202 Accepted:
```json
{
  "message": "Impor data berhasil dikonfirmasi dan sedang dimasukkan ke antrean pemrosesan sistem.",
  "id": "0192a000-7000-7000-8000-000000000001",
  "status": "confirmed",
  "confirmed_rows_count": 1950,
  "skipped_error_rows_count": 50
}
```

#### Error Responses:
- `409 Conflict` — Status tidak valid untuk konfirmasi:
  ```json
  {
    "error": {
      "code": "INVALID_BATCH_STATE",
      "message": "Batch impor tidak berada dalam status yang dapat dikonfirmasi (status saat ini: processing)."
    }
  }
  ```
- `422 Unprocessable Entity` — Tidak ada baris yang valid:
  ```json
  {
    "error": {
      "code": "NO_VALID_ROWS_TO_IMPORT",
      "message": "Tidak ada baris valid yang dapat diimpor. Seluruh baris memiliki galat."
    }
  }
  ```

---

### 6.7 Pembatalan Batch atau Rollback Transaksi
`POST /api/v1/businesses/{business_id}/imports/{id}/cancel`

Membatalkan proses impor:
- Jika dipanggil sebelum selesai (`uploaded`, `mapped`, `validated`, `confirmed`): Membatalkan antrean dan menghentikan pemrosesan batch.
- Jika dipanggil setelah selesai (`completed`, `completed_with_errors`): Melakukan *destructive rollback* terhadap seluruh transaksi penjualan yang dibuat oleh batch tersebut dan memulihkan stok inventaris ke kondisi awal.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Request Body:
```json
{
  "reason": "Salah memasukkan file rekap transaksi cabang lain",
  "confirmation_phrase": "BATALKAN"
}
```
- `reason` (string, wajib): Alasan pembatalan (disimpan dalam log audit).
- `confirmation_phrase` (string, wajib jika batch sudah `completed` / `completed_with_errors`): Harus bernilai `"BATALKAN"`.

#### Response 200 OK:
```json
{
  "message": "Batch impor berhasil dibatalkan.",
  "id": "0192a000-7000-7000-8000-000000000001",
  "status": "cancelled",
  "rolled_back_transactions_count": 1950,
  "restored_inventory_items_count": 2100,
  "cancelled_at": "2026-10-18T10:35:00Z"
}
```

#### Error Responses:
- `400 Bad Request` — Frasa konfirmasi salah:
  ```json
  {
    "error": {
      "code": "CONFIRMATION_PHRASE_MISMATCH",
      "message": "Ketik kata 'BATALKAN' dengan benar untuk menyetujui pembatalan massal ini."
    }
  }
  ```
- `409 Conflict` — Batch sedang aktif diproses oleh worker:
  ```json
  {
    "error": {
      "code": "CANNOT_CANCEL_PROCESSING_BATCH",
      "message": "Batch sedang aktif ditulis ke basis data. Silakan tunggu hingga proses selesai sebelum melakukan rollback."
    }
  }
  ```

---

### 6.8 Unduh Laporan Baris Galat (CSV Aman Formula)
`GET /api/v1/businesses/{business_id}/imports/{id}/errors/download`

Mengunduh berkas CSV berisi seluruh baris yang gagal diimpor (`status = 'error'`) beserta kolom penjelasan penyebab galat. Berkas telah diproteksi dari potensi eksploitasi injeksi formula spreadsheet.

#### Path Parameters:
- `business_id` (string UUID, wajib): ID bisnis aktif.
- `id` (string UUID, wajib): ID batch impor.

#### Query Parameters:
- `delimiter` (string, opsional, default: `comma`): `comma` (`,`) atau `semicolon` (`;`).

#### Spesifikasi Keamanan Formula (Anti-Formula Injection Rule):
Setiap nilai string pada seluruh sel diperiksa:
```go
func NeutralizeCSVCell(val string) string {
    trimmed := strings.TrimSpace(val)
    if len(trimmed) == 0 {
        return val
    }
    firstChar := trimmed[0]
    if firstChar == '=' || firstChar == '+' || firstChar == '-' || firstChar == '@' || firstChar == '\t' || firstChar == '\r' {
        return "'" + val
    }
    return val
}
```

#### Format Kolom CSV yang Dihasilkan:
1. Kolom 1..N: Kolom asli berkas CSV pengguna (disanitasi).
2. Kolom Tambahan:
   - `_nomor_baris_asli`: Posisi baris asli pada berkas pengguna.
   - `_status_validasi`: Nilai `error`.
   - `_rincian_galat`: Gabungan pesan kesalahan (dipisahkan tanda titik koma `;`).

#### Response 200 OK:
- Headers:
  - `Content-Type: text/csv; charset=utf-8`
  - `Content-Disposition: attachment; filename="laporan-galat-impor-{id}.csv"`
- Payload CSV (Contoh baris galat):
  ```csv
  No Pesanan,Tanggal Transaksi,Nama Pembeli,No WhatsApp,SKU Item,Nama Item,Jumlah,Harga Satuan,Diskon,_nomor_baris_asli,_status_validasi,_rincian_galat
  TRX-1014,18/10/2026,Budi Santoso,081298765432,MAT-999,Matcha Latte,1,24.000,0,14,error,SKU 'MAT-999' tidak ditemukan pada katalog produk.
  '=cmd|' /C calc'!A0,18/10/2026,Hacker,0811111111,KOP-001,Kopi Susu,1,18000,0,15,error,Format nomor pesanan tidak valid.
  ```
  *(Perhatikan baris ke-15: Nilai formula `=cmd|' /C calc'!A0` otomatis dinetralkan menjadi `'=cmd|' /C calc'!A0`)*.

#### Error Responses:
- `404 Not Found` — Tidak ada baris galat:
  ```json
  {
    "error": {
      "code": "NO_ERROR_ROWS_FOUND",
      "message": "Tidak ada baris galat pada batch impor ini."
    }
  }
  ```

---

## 7. Rangkuman Pemetaan Model Data (`contracts/erd.md` Alignment)

| ERD Column | Representasi API DTO | Keterangan & Validasi |
|---|---|---|
| `import_batches.id` | `id` (UUIDv7) | Primary Key unik. |
| `import_batches.business_id` | `business_id` (UUIDv7) | Tenant isolation key. |
| `import_batches.import_type` | `import_type` (string) | Enum: `products`, `customers`, `transactions`. |
| `import_batches.original_filename` | `original_filename` (string) | Nama file asli saat diunggah (display only). |
| `import_batches.file_sha256` | `file_sha256` (string hex) | Hash SHA-256 untuk deteksi duplikasi. |
| `import_batches.file_size_bytes` | `file_size_bytes` (integer) | Ukuran file maksimal `5242880` (5 MB). |
| `import_batches.status` | `status` (string) | Enum siklus batch (lihat §1.2). |
| `import_batches.column_mapping` | `column_mapping` (object/JSONB) | Key-value mapping field target $\rightarrow$ header CSV. |
| `import_batches.format_options` | `format_options` (object/JSONB) | `delimiter`, `number_format`, `date_format`. |
| `import_batches.total_rows` | `total_rows` (integer) | Maksimal `20000`. |
| `import_batches.valid_rows` | `valid_rows` (integer) | Jumlah baris berstatus `valid`. |
| `import_batches.warning_rows` | `warning_rows` (integer) | Jumlah baris berstatus `warning`. |
| `import_batches.error_rows` | `error_rows` (integer) | Jumlah baris berstatus `error`. |
| `import_batches.imported_rows` | `imported_rows` (integer) | Jumlah baris yang sukses dimutasi ke entitas aktif. |
| `import_rows.raw_data` | `raw_data` (object/JSONB) | Baris mentah sebelum divalidasi. |
| `import_rows.status` | `status` (string) | Enum: `pending`, `valid`, `warning`, `error`, `imported`, `skipped`. |
| `import_rows.errors` | `errors` (array of objects) | Rincian `[{code, field, message}]`. |
| `import_rows.warnings` | `warnings` (array of objects) | Rincian `[{code, field, message}]`. |
| `import_rows.entity_type` | `entity_type` (string/null) | Tipe entitas yang terbentuk (`product`, `customer`, `transaction`). |
| `import_rows.entity_id` | `entity_id` (UUID/null) | ID entitas yang terbentuk di database. |
