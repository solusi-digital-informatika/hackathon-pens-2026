# Task T-054: QA Verification for CSV Injection, Malformed Rows, Async Recovery & Wizard

- **Task ID**: T-054
- **Assignee**: tester
- **Status**: doing

## 1. Objective
Lakukan verifikasi jaminan kualitas independen (QA & Security Verification) secara menyeluruh terhadap deliverable Milestone M5 (Asynchronous CSV Import Wizard):
- Backend: Commit `d007c69404d6433891a8d8fe863536a2b0e54753` di `teman-umkm-backend`.
- Frontend: Commit `15d0c57820602b2c0c552359adf8c63937040197` di `teman-umkm-frontend`.

## 2. Reference & Standards
- TRD Specification: `office-hub/project-map/trd.md` (§8 CSV Import Specification, §10 Security & Validation).
- UI Spec: `office-hub/contracts/ui-csv-import.md`.
- API Contract: `office-hub/contracts/api-csv-import.md`.
- Backend Implementation Report: `office-hub/inbox/leader/T-052-report.md`.
- Frontend Implementation Report: `office-hub/inbox/leader/T-053-report.md`.

## 3. Test Matrix & Scope
1. **Security & Formula Injection Neutralization (TRD §10)**:
   - Verifikasi karakter berbahaya `=`, `+`, `-`, `@`, `\t`, `\r` pada baris CSV.
   - Pastikan ekspor baris galat (error report CSV) menyisipkan prefix quote (`'`) sehingga formula dinetralisasi dan tidak dapat dieksekusi di aplikasi spreadsheet (Excel/LibreOffice/Google Sheets).
   - Verifikasi batas unggah file (maks 5 MB) dan batas baris (maks 20.000 baris).
   - Verifikasi checksum SHA-256 untuk mendeteksi berkas duplikat.
2. **Localization & Number/Date Formats**:
   - Pemformatan angka Indonesia (`id_ID`: `1.250.000,50`) vs internasional (`en_US`: `1,250,000.50`).
   - Format tanggal Indonesia: `DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`.
   - Normalisasi nomor telepon ke standar E.164 (`+628...`).
3. **Data Integrity & Row-Level Isolation**:
   - Batch chunk processing (200 baris per transaksi DB).
   - Mode `skip_errors`: Baris valid tetap diproses menjadi entitas aktual (`products`, `customers`, `transactions`), sedangkan baris galat dilewati tanpa menggagalkan seluruh batch.
   - Rollback capability: Validasi bahwa proses pembatalan batch menghapus relasi transaksi/entitas yang dibuat oleh batch tersebut dan mengembalikan status ke `cancelled`.
4. **Frontend Integration & UX Flows**:
   - 5-step Stepper: Upload -> Mapping -> Validation -> Polling -> Summary.
   - Polling interaktif: Verifikasi penanganan status progress bar dan penanganan kegagalan.
   - Modal Rollback: Verifikasi bahwa tombol konfirmasi rollback dinonaktifkan hingga pengguna mengetik frasa persis `BATALKAN`.
5. **Execution Verification**:
   - Jalankan `go test -v ./...` dan `go build ./cmd/api` pada backend.
   - Jalankan `npm test -- --run` dan `npm run build` pada frontend.

## 4. Definition of Done
1. Seluruh rangkaian unit test backend & frontend berhasil dieksekusi (100% Pass).
2. Tidak ada kerentanan formula injection yang lolos tanpa netralisasi.
3. Tulis laporan audit independen ke `inbox/leader/T-054-report.md` berisi:
   - Summary audit
   - Test cases & results matrix
   - Verdict (APPROVE / REJECT)
4. Pindahkan task ini ke `inbox/tester/done/T-054-qa-csv-import.md`.
