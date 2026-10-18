# Laporan Penyelesaian T-051: API Contract for CSV Import, Mapping, Row Validation & Async Worker

- **Task ID**: T-051
- **Assignee**: `backend-ai`
- **Deliverable**: `contracts/api-csv-import.md` (API-M5-001)
- **Status**: DONE / APPROVED
- **Referensi Terkait**: `project-map/trd.md` (§8, §9, §10, TD-07, TD-11), `contracts/erd.md` (§5 Diagram D), `contracts/ui-csv-import.md` (UI-M5-001)

---

## Ringkasan Eksekutif
Telah selesai disusun dan disahkan kontrak arsitektur REST API komprehensif pada `contracts/api-csv-import.md` untuk Modul CSV Import Wizard (M5). Kontrak ini mendefinisikan secara tuntas 8 endpoint RESTful, skema DTO request/response, katalog kode galat, protokol pekerja asinkron (*Database as Queue* via `SELECT ... FOR UPDATE SKIP LOCKED`), mekanisme pembatalan/rollback atomik, serta perlindungan injeksi formula spreadsheet.

## Rincian Ruang Lingkup yang Telah Ditetapkan

1. **8 Endpoint RESTful Komprehensif**:
   - `GET /api/v1/businesses/{business_id}/imports/templates/{type}`: Mengunduh template resmi CSV (`products`, `customers`, `transactions`) dengan pilihan pembatas koma/titik-koma.
   - `POST /api/v1/businesses/{business_id}/imports`: Menerima multipart CSV stream (maks 5 MB, maks 20.000 baris), parsing tanpa penyimpanan file mentah, pembersihan UTF-8 BOM, deteksi otomatis delimiter (koma, titik koma Excel, tab), kalkulasi hash SHA-256 (`file_sha256`) untuk deteksi unggah berulang, dan staging ke `import_rows`.
   - `GET /api/v1/businesses/{business_id}/imports/{id}`: Menyajikan rincian status batch (`uploaded`, `mapped`, `validated`, `confirmed`, `processing`, `completed`, `completed_with_errors`, `failed`, `cancelled`), metrik hitungan baris (`total_rows`, `valid_rows`, `warning_rows`, `error_rows`, `imported_rows`), dan progres polling asinkron.
   - `PUT /api/v1/businesses/{business_id}/imports/{id}/mapping`: Menyimpan pemetaan kolom target sistem, preferensi parsing angka (`id_ID` vs `en_US`) dan tanggal (`DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`), sekaligus memicu audit validasi baris ke tabel staging.
   - `GET /api/v1/businesses/{business_id}/imports/{id}/rows`: Mengembalikan daftar baris data terpaginasi dengan filter status (`valid`, `warning`, `error`, `pending`, `imported`, `skipped`) dan fitur pencarian.
   - `POST /api/v1/businesses/{business_id}/imports/{id}/confirm`: Pemicu eksekusi idempoten ke antrean pekerja latar belakang dengan verifikasi status row-lock.
   - `POST /api/v1/businesses/{business_id}/imports/{id}/cancel`: Pembatalan sebelum eksekusi atau rollback destruktif paska eksekusi (membatalkan transaksi penjualan yang dibuat batch tersebut, memulihkan stok inventaris produk, dan mencatat log audit permanen dengan verifikasi kata kunci `BATALKAN`).
   - `GET /api/v1/businesses/{business_id}/imports/{id}/errors/download`: Mengunduh berkas CSV berisi seluruh baris galat dengan netralisasi injeksi formula Excel (`=`, `+`, `-`, `@`, `\t`, `\r` otomatis diproteksi kutip tunggal `'`).

2. **Mesin Validasi & Aturan Bisnis Ketat (TRD §8)**:
   - **Pencocokan Produk**: Hanya berdasarkan SKU persis atau nama produk persis (*case-insensitive trimmed*). **Dilarang fuzzy matching** untuk mencegah kesalahan varian dan selisih HPP.
   - **Pencocokan Pelanggan**: Hanya berdasarkan nomor telepon terstandarisasi internasional E.164 (`+62...`). **Dilarang auto-merge berdasarkan kemiripan nama saja**; jika hanya nama yang mirip, sistem otomatis membuat kontak baru dan menerbitkan peringatan `CUSTOMER_NAME_SIMILAR`.
   - **Resolusi Angka Ambigu**: Format ambigu (seperti `1.500`) diproses tegas berdasarkan preferensi format yang dipilih pengguna pada langkah pemetaan.

3. **Protokol Pekerja Asinkron (*Database as Queue* - TD-07 & TD-11)**:
   - Worker mengambil pekerjaan menggunakan kueri atomik `SELECT ... FOR UPDATE SKIP LOCKED` pada `import_batches WHERE status = 'confirmed'`.
   - Scheduler multi-instance diamankan menggunakan PostgreSQL Advisory Lock `pg_try_advisory_lock`.
   - Pemrosesan chunk per 200 baris dengan isolasi savepoint per baris (`SAVEPOINT import_row_savepoint`). Kegagalan satu baris tidak menggagalkan baris valid lainnya.
   - Dukungan penutupan bersih (*graceful shutdown*) saat menerima sinyal `SIGINT`/`SIGTERM`.
   - Audit trail permanen pada tabel `audit_logs` untuk seluruh aksi status perubahan: konfirmasi, penyelesaian, pembatalan, dan rollback transaksi.

Dokumen kontrak `contracts/api-csv-import.md` siap dijadikan acuan implementasi backend (T-052) dan frontend (T-053).
