# Task T-070: Hardening, Security UI Audit & UX Edge-Case Specification

- **Task ID**: T-070
- **Assignee**: designer
- **Repo**: `office-hub`
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: doing

## 1. Objective
Susun spesifikasi desain antarmuka, UX hardening, dan audit visual keamanan untuk kesiapan rilis produksi (Milestone M7) Teman UMKM:
1. **Audit Permukaan Akses & Sesi (Session & Auth UX)**:
   - Status session timeout dan auto-refresh token (UX state saat token kedaluwarsa: inline toast pemberitahuan re-autentikasi tanpa kehilangan input form / draft uncommitted).
   - Penanganan status multi-tenant: indikator nama bisnis aktif, badge peran pengguna (`Owner` / `Admin`), dan proteksi visual terhadap kebocoran data antar entitas bisnis.
2. **Penanganan Error Global & Network Degradation**:
   - Spesifikasi komponen `GlobalErrorBoundary` (tampilan fallback bersih bernuansa *Emerald Forest*, tanpa stack trace bocor ke pengguna awam, tombol "Muat Ulang Halaman" dan "Laporkan Kendala").
   - Status offline / koneksi terputus (*Offline Banner* non-intrusif di bagian atas).
   - Feedback respons lambat (Skeleton loading konsisten, tombol submit disable dengan spinner untuk mencegah double submit).
3. **Audit Ekspor & Formula Injection Protection UX**:
   - Peringatan visual edukatif saat mengekspor data transaksi atau CSV: konfirmasi pengunduhan dan kepastian format aman (tidak memicu eksekusi macro/formula berbahaya di Excel).
4. **Verifikasi Kontras, Aksesibilitas WCAG 2.1 AA & Dark Mode Readiness**:
   - Verifikasi kontras warna token `#0D6E64` (Emerald Forest) pada tombol primer dan badge klaim (`FAKTA`, `ESTIMASI`, `HIPOTESIS`).

## 2. Deliverables
- Simpan dokumen spesifikasi di `contracts/ui-hardening-security.md`.
- Tuliskan laporan hasil pekerjaan di `inbox/leader/T-070-report.md`.
- Pindahkan file task ini ke `inbox/designer/done/T-070-ui-hardening.md`.
