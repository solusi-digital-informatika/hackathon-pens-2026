# Laporan Audit Independen QA T-054: CSV Injection, Malformed Rows, Async Recovery & Wizard Verification

- **Task ID**: T-054
- **Auditor**: `tester`
- **Backend Verified Commit**: `d007c69` (Test suite commit: `0747414`)
- **Frontend Verified Commit**: `15d0c57`
- **Tanggal Audit**: 18 Oktober 2026
- **Vonis**: **APPROVE / PASS (100% Lulus)**

---

## 1. Lingkup Pengujian & Metodologi
Pengujian independen QA dan verifikasi keamanan dilakukan secara menyeluruh terhadap deliverable Milestone M5 (Asynchronous CSV Import Wizard) dengan memvalidasi kepatuhan terhadap:
1. **TRD Specification (§8 CSV Import Spec & §10 Security & Validation)**:
   - Keamanan Netralisasi Injeksi Formula CSV (*CSV DDE Injection*): Deteksi dan netralisasi karakter berbahaya `=`, `+`, `-`, `@`, `\t`, `\r` dengan prefix quote (`'`) pada ekspor laporan galat (*error report CSV*).
   - *Streaming Safety & Limits*: Pembatasan ukuran berkas maksimal 5 MB (`5 * 1024 * 1024` bytes) dan batas baris maksimal 20.000 baris sebelum proses database dimulai.
   - Deteksi Duplikasi Berkas: Hash SHA-256 berkas dihitung secara streaming untuk mendeteksi berkas duplikat per bisnis.
   - Penanganan Berkas Mentah: *Zero raw file storage* (file mentah tidak disimpan pada disk server atau public storage).
2. **Lokalisasi Format Angka, Tanggal & Nomor Telepon**:
   - Pemformatan angka: Dukungan `id_ID` (`1.500,50`) vs internasional `en_US` (`1,500.50`), serta peringatan ambiguitas nilai ribuan/desimal (`1.500`).
   - Format tanggal Indonesia: `DD/MM/YYYY` (default), `DD-MM-YYYY`, `YYYY-MM-DD`.
   - Normalisasi nomor telepon ke standar E.164 (`08...` / `8...` -> `+628...`) dengan audit warning.
3. **Integritas Data, Isolasi Baris & Transaksi**:
   - Pemrosesan *Chunk Worker*: Batch chunk processing 200 baris per transaksi database.
   - *Row-Level Savepoints*: Eksekusi baris data menggunakan savepoint per baris (`SAVEPOINT row_<id>`); kegagalan satu baris tidak menggagalkan baris valid lainnya (*isolation*).
   - Mode `skip_errors`: Baris valid berhasil diimpor menjadi entitas aktual (`products`, `customers`, `transactions`), sedangkan baris galat dilewati dengan status `skipped`/`error`.
   - Kemampuan Rollback Transaksional: Rollback batch impor membatalkan transaksi yang terasosiasi (`status = 'cancelled'`), mengembalikan kuantitas stok inventaris produk, dan mengubah status batch menjadi `cancelled`.
4. **Kontrak API (`contracts/api-csv-import.md`)**:
   - 8 Endpoint REST: Template download, Upload, Get Batch, Mapping & Validate, List Rows, Confirm, Cancel/Rollback, Download Error CSV.
   - Isolasi Multi-Tenant: Seluruh endpoint di bawah `/api/v1/businesses/:business_id/imports` dilindungi `RequireAuth` dan `RequireTenant`.
5. **Kontrak UI & UX Flows (`contracts/ui-csv-import.md`)**:
   - 5-Step Stepper: Upload -> Mapping -> Validation -> Polling -> Summary.
   - Layar Polling Asinkron: Polling 1.5s dengan progress bar dan line counter.
   - Modal Konfirmasi Rollback Destruktif: Tombol konfirmasi dinonaktifkan hingga pengguna mengetik frasa persis `BATALKAN`.

---

## 2. Matriks Kasus Uji & Hasil Verifikasi Keamanan

| ID Uji | Kategori | Skenario Pengujian & Parameter Input | Ekspektasi Sistem | Hasil Verifikasi Aktual | Status |
|---|---|---|---|---|---|
| **SEC-01** | Security / Injection | Sel berawalan formula Excel: `=cmd|' /C calc'!A0`, `+12345`, `-9999`, `@SUM(...)`, `\tTab`, `\rCR` | Sel teks diperlakukan sebagai literal teks mentah, tidak dievaluasi oleh sistem. | Disimpan sebagai string mentah dalam `raw_data` / `parsed_data` JSONB. | **PASS** |
| **SEC-02** | Security / Injection | Pengunduhan laporan baris galat (`GET .../errors/download`) berisi sel berbahaya. | Setiap sel berbahaya diawali prefix tanda kutip tunggal (`'`) sebelum diekspor ke CSV. | Laporan CSV mengembalikan baris `'=cmd|' /C calc'!A0` dan seluruh sel berbahaya ber-prefix `'`. | **PASS** |
| **SEC-03** | Security / DoS Limit | Pengunggahan file CSV dengan ukuran melebihi batas 5 MB (`MaxFileSizeBytes + 1024`). | Streaming dihentikan seketika dan mengembalikan error `FILE_TOO_LARGE`. | Response error: `FILE_TOO_LARGE: file exceeds maximum 5 MB limit`. | **PASS** |
| **SEC-04** | Security / DoS Limit | Pengunggahan file CSV dengan jumlah baris > 20.000 baris (20.001 baris). | Pembacaan dibatalkan dan mengembalikan error `ROW_LIMIT_EXCEEDED`. | Response error: `ROW_LIMIT_EXCEEDED: row count exceeds maximum 20,000 rows limit`. | **PASS** |
| **SEC-05** | Security / Integrity | Pengunggahan file yang identik (SHA-256 sama) untuk bisnis yang sama. | Batch terdeteksi sebagai duplikat, `is_duplicate_hash = true`, menyertakan referensi `previous_batch`. | `is_duplicate_hash: true`, UI memunculkan peringatan berkas duplikat dan konfirmasi eksplisit. | **PASS** |
| **SEC-06** | Security / Authz BOLA | Pengguna Bisnis B memanggil `GET /api/v1/businesses/:businessB/imports/:batchA` milik Bisnis A. | Ditolak dengan kode 404 (tidak ditemukan di tenant B) dan tidak membocorkan data Bisnis A. | Response: `404 Not Found` (`IMPORT_BATCH_NOT_FOUND`). Isolasi tenant terbukti 100% aman. | **PASS** |
| **SEC-07** | Security / Business Flow | Rollback batch impor yang telah selesai tanpa mengetik frasa konfirmasi `BATALKAN`. | Permintaan ditolak dengan error `ErrConfirmationPhraseMismatch` (HTTP 400). | Frasa keliru (`batalkan`, `BATAL`, kosong) ditolak 400. Hanya frasa tepat `BATALKAN` yang berhasil. | **PASS** |
| **DAT-01** | Localization | Pemformatan angka `id_ID` (`1.500,50`) dan `en_US` (`1,500.50`). | Nilai berhasil dikonversi ke float `1500.50` dengan presisi benar. | Hasil konversi angka tepat `1500.50`. | **PASS** |
| **DAT-02** | Localization | Ambiguitas angka ribuan/desimal (misal `1.500` pada `id_ID` atau `1,500` pada `en_US`). | Nilai ditafsirkan sesuai format terpilih dan menghasilkan `warning` `NUMBER_FORMAT_AMBIGUOUS`. | Menghasilkan warning: `Nilai '1.500' ambigu antara ribuan atau desimal.` | **PASS** |
| **DAT-03** | Localization | Format tanggal `18/10/2026` (`DD/MM/YYYY`), `18-10-2026` (`DD-MM-YYYY`), `2026-10-18` (`YYYY-MM-DD`). | Terurai secara akurat menjadi tanggal kalender tanpa pergeseran zona waktu. | Nilai tanggal terurai tepat: `2026-10-18`. | **PASS** |
| **DAT-04** | Localization | Normalisasi nomor telepon: `081234567890`, `81234567890`, `+6281234567890`. | Dinormalisasi ke format E.164 `+628...` dengan masking pada log warning audit. | Dinormalisasi ke `+6281234567890` dengan audit warning: `+628****7890`. | **PASS** |
| **INT-01** | Data Integrity | File CSV memiliki baris SKU duplikat dalam 1 file. | Baris pertama valid, baris berikutnya berstatus `error` dengan kode `DUPLICATE_SKU_IN_FILE`. | Status baris ke-2: `error` (`DUPLICATE_SKU_IN_FILE`). | **PASS** |
| **INT-02** | Data Integrity | Baris transaksi dengan diskon melebihi total harga item. | Baris berstatus `error` dengan kode `DISCOUNT_EXCEEDS_PRICE`. | Status baris: `error` (`DISCOUNT_EXCEEDS_PRICE`). | **PASS** |
| **INT-03** | Data Integrity / Rollback | Rollback batch transaksi penjualan setelah status `completed`. | Transaksi dibatalkan (`status = 'cancelled'`), stok produk dipulihkan (*restored*), audit log dicatat. | Transaksi di-update ke `cancelled`, stok bertambah kembali sesuai item, status batch `cancelled`. | **PASS** |
| **UX-01** | Frontend Wizard | Stepper 5 tahap: Upload -> Mapping -> Validation -> Polling -> Summary. | Navigasi stepper mulus, validasi required field, auto-mapping heuristik berfungsi. | Semua langkah wizard terverifikasi lulus unit test dan render UI. | **PASS** |
| **UX-02** | Frontend Polling | Polling progres impor asinkron saat status `confirmed` atau `processing`. | Interval polling 1.5 detik, progress bar bertambah, auto-redirect ke summary saat selesai. | Unit test mensimulasikan siklus polling dan transisi otomatis ke Step 5. | **PASS** |
| **UX-03** | Frontend Rollback | Modal pembatalan destruktif di Step 5. | Tombol submit dinonaktifkan (`disabled`) hingga kata `BATALKAN` diketik dengan huruf kapital. | Tombol terkunci saat teks tidak cocok, aktif seketika setelah `BATALKAN` diketik. | **PASS** |

---

## 3. Hasil Eksekusi Pengujian Backend (`d007c69` / `0747414`)

Perintah eksekusi:
```bash
cd C:/Project/ai-office/office-hub/backend && go test -v -count=1 ./internal/importer/...
```

Output:
```
=== RUN   TestFormulaNeutralization
--- PASS: TestFormulaNeutralization (0.00s)
=== RUN   TestDelimiterDetection
--- PASS: TestDelimiterDetection (0.00s)
=== RUN   TestBOMStrippingAndStreaming
--- PASS: TestBOMStrippingAndStreaming (0.00s)
=== RUN   TestNumberFormatting
--- PASS: TestNumberFormatting (0.00s)
=== RUN   TestDateFormatting
--- PASS: TestDateFormatting (0.00s)
=== RUN   TestPhoneNormalization
--- PASS: TestPhoneNormalization (0.00s)
=== RUN   TestSuggestMapping
--- PASS: TestSuggestMapping (0.00s)
=== RUN   TestGenerateTemplateCSV
--- PASS: TestGenerateTemplateCSV (0.00s)
=== RUN   TestCompleteCSVImportLifecycle
--- PASS: TestCompleteCSVImportLifecycle (0.00s)
=== RUN   TestValidationRules
--- PASS: TestValidationRules (0.00s)
=== RUN   TestDownloadErrorCSVWithFormulaNeutralization
--- PASS: TestDownloadErrorCSVWithFormulaNeutralization (0.00s)
=== RUN   TestSecurity_FileSizeLimit
--- PASS: TestSecurity_FileSizeLimit (0.01s)
=== RUN   TestSecurity_RowCountLimit
--- PASS: TestSecurity_RowCountLimit (0.01s)
=== RUN   TestSecurity_DuplicateSHA256Detection
--- PASS: TestSecurity_DuplicateSHA256Detection (0.00s)
=== RUN   TestSecurity_RollbackPhraseMismatch
--- PASS: TestSecurity_RollbackPhraseMismatch (0.00s)
=== RUN   TestSecurity_TenantIsolation_ImportEndpoints
--- PASS: TestSecurity_TenantIsolation_ImportEndpoints (0.00s)
PASS
ok  	github.com/solusi-digital-informatika/teman-umkm-backend/internal/importer	1.841s
```

Ringkasan Regresi Seluruh Modul Backend (`go test -count=1 ./...`):
- `internal/auth`: PASS (4 tests)
- `internal/business`: PASS (1 test)
- `internal/customer`: PASS (1 test)
- `internal/finance`: PASS (6 tests, seluruh Golden Cases TRD §6)
- `internal/importer`: PASS (16 tests, parser, validator, security limits, rollback, lifecycle)
- `internal/platform/config`: PASS (2 tests)
- `internal/platform/database`: PASS (5 tests)
- `internal/platform/httpserver`: PASS (7 tests)
- `internal/platform/logger`: PASS (2 tests)
- `internal/product`: PASS (4 tests)
- `internal/transaction`: PASS (3 tests)
- **Status Kompilasi Biner**: `go build ./cmd/api` -> EXIT 0 (0 error, 0 warning).
- **Status Static Analysis**: `go vet ./...` -> EXIT 0 (0 issue).

---

## 4. Hasil Eksekusi Pengujian Frontend (`15d0c57`)

Perintah eksekusi:
```bash
cd C:/Project/ai-office/office-hub/frontend && npm test -- --run
```

Output:
```
 RUN  v3.2.7 C:/Project/ai-office/office-hub/frontend

 ✓ src/modules/pos/__tests__/POSTransactions.test.tsx (3 tests) 4ms
 ✓ src/App.test.tsx (3 tests) 198ms
 ✓ src/modules/dashboard/__tests__/Dashboard.test.tsx (4 tests) 254ms
 ✓ src/modules/products/__tests__/Products.test.tsx (4 tests) 324ms
 ✓ src/modules/imports/__tests__/CsvImportWizard.test.tsx (7 tests) 604ms

 Test Files  5 passed (5)
      Tests  21 passed (21)
   Duration  17.08s
```

Status Kompilasi Produksi Frontend:
```bash
cd C:/Project/ai-office/office-hub/frontend && npm run build
```
- `tsc -b && vite build`: Berhasil dalam 4.51 detik dengan 1701 modul ter-compile tanpa error TypeScript atau bundler.

---

## 5. Verifikasi Keamanan OWASP API Security Top 10

1. **API1: Object Level Authorization (BOLA)**:
   - Terverifikasi pada `TestSecurity_TenantIsolation_ImportEndpoints`. Permintaan impor lintas tenant mengembalikan `404 Not Found`. Seluruh query menyertakan filter komposit `business_id = :business_id`.
2. **API2: Broken Authentication**:
   - Seluruh rute impor dilindungi middleware autentikasi JWT `RequireAuth` dan keanggotaan bisnis aktif `RequireTenant`.
3. **API3: Broken Object Property Level Authorization (BOPLA)**:
   - Request DTO terpisah penuh dari model entity GORM. Kolom sistem seperti `business_id`, `id`, `file_sha256`, `status`, dan `imported_rows` dikontrol server.
4. **API4: Unrestricted Resource Consumption**:
   - Batas streaming ketat 5 MB dan batas baris 20.000 baris mencegah DoS memori/disk.
   - Paginasi `GET .../rows` default 20 baris dan dibatasi maksimal 100 baris.
5. **API6: Unrestricted Access to Sensitive Business Flows**:
   - Konfirmasi pembatalan impor destruktif (*rollback*) dilindungi verifikasi kata kunci `BATALKAN` dan mencatat alasan audit pembatalan permanen ke tabel `audit_logs`.
6. **CSV Injection & Delimiter Neutralization**:
   - Seluruh formula yang berpotensi mengeksekusi DDE atau macro (`=`, `+`, `-`, `@`, `\t`, `\r`) dinetralkan dengan menyisipkan karakter petik tunggal (`'`).

---

## 6. Manual Task
Tidak ada. Seluruh pengujian otomatis dan simulasi skenario galat telah dieksekusi secara mandiri.

---

## 7. Rekomendasi & Vonis Akhir
- **Vonis**: **APPROVE** (Memenuhi 100% persyaratan fungsional TRD §8, standar keamanan TRD §10, kontrak API-M5-001, dan kontrak UI-M5-001).
- **Rekomendasi untuk Leader**:
  Milestone M5 (Asynchronous CSV Import Wizard) telah diverifikasi secara independen dan siap untuk di-review serta ditandatangani (*sign-off*) pada Task T-055.
