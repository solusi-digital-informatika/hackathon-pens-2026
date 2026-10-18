# Laporan Penyelesaian T-070: Hardening, Security UI Audit & UX Edge-Case Specification

- **Task ID**: T-070
- **Assignee**: `designer`
- **Deliverable**: `contracts/ui-hardening-security.md`
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: DONE / APPROVED

Telah selesai disusun dan didokumentasikan spesifikasi antarmuka, UX hardening, dan audit visual keamanan komprehensif untuk kesiapan rilis produksi (Milestone M7) pada `contracts/ui-hardening-security.md`. Dokumen ini sepenuhnya mematuhi prinsip desain Clean Pragmatic Craftsman, Design Tokens (`contracts/design-tokens.md`), Art Direction (`contracts/design-direction.md`), serta referensi arsitektur keamanan multi-tenant (TD-05, TRD §8, TRD §9, TRD §10, `contracts/api-auth.md`, dan `contracts/api-csv-import.md`).

---

### Rincian Cakupan Hasil Pekerjaan:

1. **Audit Permukaan Akses & Sesi (Session & Auth UX)**:
   - **Siklus Sesi & Silent Refresh**: Menguraikan alur penanganan `401 Unauthorized` dengan *silent refresh* via cookie httpOnly (`POST /api/v1/auth/refresh`).
   - **Dialog Re-Autentikasi Non-Destruktif (`SessionTimeoutModal`)**: Spesifikasi modal inline jika refresh token kedaluwarsa/batal, memungkinkan pengguna memasukkan kata sandi tanpa mereset formulir atau kehilangan draf transaksi yang sedang diketik (*Zero Data Loss*). Menyediakan opsi penyimpanan draf ke `localStorage` jika pengguna memilih keluar.
   - **Status Multi-Tenant & Proteksi Kebocoran Data**:
     - Kapsul pemilih bisnis aktif (`TenantSelectorPill`) di top bar dengan penanda visual status toko aktif.
     - Proteksi perpindahan tenant: konfirmasi pembersihan state saat form dalam status belum tersimpan (*dirty state*) dan isolasi rute terlingkup `/businesses/:business_id/...`.
     - Badge peran pengguna (`UserRoleBadge`) membedakan hak akses `owner` (*Pemilik Usaha*, aksen emerald) dan `staff` (*Staf Operasional*, slate netral), dilengkapi penanda gembok dan pencegahan navigasi tak berwenang (HTTP 403 fallback).
   - **Feedback Rate Limiting & Cooldown Protection (HTTP 429)**: Komponen `RateLimitAlert` dengan hitungan mundur dinamis transparan dan penonaktifan tombol pemicu selama masa jeda pendinginan (*cooldown*).

2. **Penanganan Error Global & Network Degradation**:
   - **Komponen `GlobalErrorBoundary`**: Fallback antarmuka bernuansa *Emerald Forest* dengan perisai SVG pelindung. Menerapkan aturan kerahasiaan sistem ketat (*Zero Stack Trace Leakage*): dilarang mengekspos stack trace, query SQL, atau token ke pengguna awam. Menyediakan kode tiket insiden terstandarisasi (e.g. `ERR_CLIENT_20261019_9B1C`) dengan tombol salin cepat, tombol muat ulang halaman, dan tombol kembali ke dasbor.
   - **Banner Offline Non-Intrusif (`OfflineBanner`)**: Sticky banner atas viewport saat event `offline` aktif atau heartbeat terputus, menonaktifkan aksi mutasi dengan tooltip informatif tanpa menghapus input formulir. Dilengkapi transisi pemulihan koneksi (*Reconnected Toast* hijau zamrud otomatis hilang dalam 3.5 detik).
   - **Pencegahan Double Submit & Skeleton Loading**: Tombol submit seketika terkunci (*disabled*) dengan spinner animasi ring melingkar (< 50ms) dan mempertahankan lebar tetap (*layout shift prevention*). Skeleton shimmer terpadu untuk pemuatan > 400ms (Hukum Doherty) dan mekanisme rollback visual pada *optimistic updates*.

3. **Audit Ekspor & Formula Injection Protection UX**:
   - **Modal Ekspor Aman (`SafeExportModal`)**: Antarmuka konfirmasi sebelum pengunduhan data transaksi atau katalog produk.
   - **Penetralan Injeksi Formula CSV**: Penjelasan edukatif transparan bahwa karakter berisiko formula spreadsheet (`=`, `+`, `-`, `@`) secara otomatis dinetralkan dengan awalan tanda kutip tunggal (`'`).
   - **Kesesuaian Regional & UTF-8 BOM**: Pilihan pembatas titik koma (`;`) yang direkomendasikan untuk pengguna Microsoft Excel di Indonesia, serta penyertaan byte order mark (`\uFEFF`) guna memastikan simbol Rupiah (`Rp`) dan karakter khusus tampil tanpa cacat (*mojibake*).

4. **Verifikasi Kontras, Aksesibilitas WCAG 2.1 AA & Dark Mode Readiness**:
   - **Matriks Audit Kontras Warna Riil**:
     - Tombol Primer Emerald Forest (`#FFFFFF` di atas `#0D6E64`): **6.12:1** (LULUS WCAG AA & AAA Large).
     - Tombol Primer Hover (`#FFFFFF` di atas `#115E59`): **7.58:1** (LULUS WCAG AA).
     - Tombol Primer Default Interactive (`#FFFFFF` di atas `#0F766E`): **5.47:1** (LULUS WCAG AA).
     - Teks Utama (`#131F1C` di atas `#FFFFFF`): **16.93:1** (LULUS WCAG AAA).
     - Teks Sekunder (`#4B5E59` di atas `#FFFFFF`): **6.90:1** (LULUS WCAG AA).
     - Claim-Type Badges: `FAKTA` (**7.29:1**), `ESTIMASI` (**6.84:1**), `HIPOTESIS` (**8.19:1**) — seluruhnya LULUS WCAG AA.
   - **Temuan Audit & Penetapan Token Hardened**:
     - Token `--color-text-tertiary: #6D827C` memiliki kontras 4.09:1 (hanya lulus untuk placeholder non-kritis atau komponen grafis). Ditetapkan pembaruan token `--color-text-tertiary: #5C6F69` (**5.34:1**, LULUS WCAG AA) untuk seluruh teks pendukung esensial.
     - Ring fokus aksesibilitas (`#0F766E`) memiliki kontras **5.47:1** terhadap latar (memenuhi SC 2.4.11).
   - **Aksesibilitas Terapan**: Focus trap pada modal, skip-to-content link, `aria-live="polite"` untuk notifikasi konektivitas, dan `aria-live="assertive"` untuk sesi timeout.
   - **Kesiapan Mode Gelap (*Dark Mode Foundation*)**: Pemetaan token semantik berbasis CSS variables dan atribut data `data-theme="dark"` untuk konsistensi di masa mendatang.

---

### Verifikasi Kelengkapan Berkas:
- Dokumen Kontrak: `contracts/ui-hardening-security.md` (Tersimpan & Valid)
- Task Berpindah: `inbox/designer/done/T-070-ui-hardening.md`
