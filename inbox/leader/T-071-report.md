# Laporan Penyelesaian T-071: Security Hardening Contract, Route Authorization Inventory, Seed CLI & DB Backup Drill Spec

- **Task ID**: T-071
- **Assignee**: `backend-ai`
- **Deliverable**: `contracts/api-hardening-security.md` (API-M7-001)
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: DONE / APPROVED
- **Referensi Terkait**: `project-map/trd.md` (§8, §9, §10, §12, §14, TD-05, TD-06, TD-11, TD-13), `contracts/erd.md`, `contracts/ui-hardening-security.md` (UI-M7-001)

---

## 1. Ringkasan Eksekutif

Telah selesai dirancang dan didokumentasikan kontrak arsitektur keamanan, inventarisasi otorisasi rute menyeluruh, spesifikasi tooling seeding mandiri (`cmd/seed`), hardening HTTP/transport, serta prosedur backup/restore drill database PostgreSQL pada `contracts/api-hardening-security.md` untuk kesiapan rilis produksi (Milestone M7). Dokumen ini menjadi acuan implementasi langsung bagi Backend-AI pada task T-072 dan audit independen oleh QA/Tester pada task T-074.

---

## 2. Rincian Ruang Lingkup yang Telah Ditetapkan

1. **Inventarisasi Otorisasi Rute (Route Inventory Contract)**:
   - Mengklasifikasikan secara tuntas **46 rute HTTP backend** ke dalam 4 level proteksi:
     - **Level 0 (Public)**: Endpoint ops (`/health`, `/ready`) dan endpoint autentikasi publik (`/auth/register`, `/auth/login`, `/auth/refresh`, `/auth/logout`).
     - **Level 1 (Authenticated User)**: Profil pengguna (`/auth/me`) dan registrasi entitas bisnis baru (`POST /businesses`).
     - **Level 2 (Tenant Scoped - Member/Owner)**: Seluruh rute operasional produk, pelanggan, transaksi, retur, ringkasan finansial, impor CSV, wawasan deterministik, dan rekomendasi di bawah `/businesses/:business_id/...`. Wajib lolos validasi keanggotaan tenant (`RequireTenant`).
     - **Level 3 (Tenant Scoped - Owner Only)**: Rute administratif destruktif/kritis (`PATCH /businesses/:business_id`, `POST /transactions/:transaction_id/cancel`, `POST /products/:product_id/deactivate`). Pengguna berstatus `staff` ditolak dengan `403 Forbidden`.
   - **Spesifikasi Automated Route Inventory Test**:
     - Pengujian berbasis refleksi langsung terhadap engine router Gin (`router.Routes()`).
     - Prinsip *Fail-closed*: Jika ada rute baru ditambahkan ke router tanpa didaftarkan pada matriks otorisasi test suite, test suite **WAJIB GAGAL SEKETIKA** (*Unregistered route detected*).
     - Menjalankan matriks pengujian lintas-tenant (*Cross-Tenant Matrix Test*): Memanggil endpoint Level 2/3 dengan token Bisnis B terhadap ID Bisnis A wajib menghasilkan `403 Forbidden`.
     - Menjalankan matriks unauthenticated: Memanggil endpoint berproteksi tanpa Bearer token wajib menghasilkan `401 Unauthorized`.

2. **Spesifikasi Tooling Seeding Mandiri (`cmd/seed`)**:
   - Utilitas CLI mandiri: `go run ./cmd/seed [flags]` dengan flag `--clean` (pembersihan transaksional aman), `--scale` (`standard` ~10k tx vs `minimal` ~100 tx), dan `--database-url`.
   - **Bisnis A (Primary Realistic UMKM - "Warung Sembako & Kopi Lestari")**:
     - Kredensial siap pakai: `owner.lestari@temanumkm.id` dan `staff.lestari@temanumkm.id` (kata sandi: `Password123!`, hash: argon2id).
     - 200 Produk: 160 ber-HPP aktif lengkap dengan komponen biaya (Bahan Baku, Kemasan, Tenaga Kerja), 40 tanpa HPP (`NULL` cost version) untuk memicu `DATA_MISSING_COST_V1`, produk negatif margin (`PROD_BELOW_COST_V1`), dan produk low margin (`PROD_LOW_MARGIN_V1`).
     - 2.000 Pelanggan: Distribusi Pareto (Top 20% pelanggan menghasilkan 80% omzet), pelanggan overdue (`CUST_OVERDUE_V1`), penurunan frekuensi (`CUST_FREQ_DROP_V1`), penurunan nominal belanja (`CUST_VALUE_DROP_V1`), dan komplain terbuka > 3 hari (`CUST_COMPLAINT_OPEN_V1`).
     - ~10.000 Transaksi (~50.000 item): Terdistribusi realistis dalam 180 hari ke belakang. 70% tertaut pelanggan, 30% guest/anonim (`DATA_UNLINKED_TX_V1`). Merekam snapshot `recorded_unit_cost` yang presisi, 2% transaksi batal, transaksi retur (`transaction_returns`), serta riwayat wawasan dan rekomendasi.
   - **Bisnis B (Isolation Benchmark - "Toko Elektronik Makmur Jaya")**:
     - Kredensial siap pakai: `owner.makmur@temanumkm.id` dan `staff.makmur@temanumkm.id`.
     - 50 produk, 200 pelanggan, dan 500 transaksi terisolasi tanpa irisan identifier dengan Bisnis A.
     - Entitas pembanding untuk membuktikan secara otomatis bahwa akses silang dari akun Bisnis B ke Bisnis A ditolak mutlak.

3. **Spesifikasi Keamanan HTTP & Transport Hardening**:
   - **Header Keamanan Standar Produksi**:
     - `Content-Security-Policy`: Default `'self'`, isolasi skrip/frame, `frame-ancestors 'none'`.
     - `X-Content-Type-Options: nosniff` (mencegah eksploitasi MIME type).
     - `X-Frame-Options: DENY` (anti-clickjacking mutlak).
     - `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` (enforced HTTPS 1 tahun).
     - `Referrer-Policy: strict-origin-when-cross-origin`.
     - `Permissions-Policy: geolocation=(), camera=(), microphone=(), payment=()`.
     - `Cache-Control: no-store` pada seluruh endpoint bisnis dan identitas.
   - **Penegakan Rate Limiting (TRD §10)**:
     - Login & Refresh: 10 request / 15 menit per IP + akun identifier.
     - Unggah CSV: 5 unggahan / 1 jam per entitas bisnis.
     - Analisis Wawasan Manual: 1 eksekusi / 10 menit per entitas bisnis (didukung pencegahan konkurensi database: `409 Conflict` jika analisis sedang berjalan).
     - Respons seragam HTTP 429 (`RATE_LIMIT_EXCEEDED`) menyertakan header `Retry-After: <detik>`.
   - **Batas Payload & Server Timeout**:
     - Maksimal 1 MB (`http.MaxBytesReader`) untuk seluruh request JSON standar.
     - Pengecualian 10 MB khusus untuk endpoint upload multipart CSV (`POST /imports`).
     - Timeout server diperketat: `ReadHeaderTimeout: 5s` (mitigasi Slowloris), `ReadTimeout: 15s`, `WriteTimeout: 30s`, `IdleTimeout: 60s`.
     - CORS ketat: Larangan wildcard `*` dengan credentials, allowlist eksplisit dari konfigurasi environment.

4. **Prosedur Backup & Restore Drill Database (TRD §10, §14)**:
   - Target Disaster Recovery SLA:
     - Target RPO: < 1 jam.
     - Target RTO: < 15 menit pada dataset 10.000 transaksi.
     - Integritas Paritas Data: 100% Zero Data Loss.
   - Format: PostgreSQL Custom Archive (`-Fc`) terkompresi level 9 dengan metadata integritas.
   - Runbook Langkah Demi Langkah:
     1. Perekaman snapshot baris & checksum finansial database sumber (`teman_umkm`).
     2. Pembuatan database sandbox drill terisolasi (`teman_umkm_drill_verify`).
     3. Eksekusi `pg_restore` dengan opsi `--clean --if-exists --no-acl --no-owner`.
     4. Audit otomatis paritas baris 21 tabel, paritas saldo moneter (`SUM(total_amount)`), integritas kunci asing komposit, dan probe aplikasi (`/health`, `/ready`, `/dashboard/summary`).
     5. Pembersihan sandbox drill (*safe teardown*).

5. **Kesesuaian McCall's Quality Factors & Manual Tasks**:
   - Memenuhi faktor Correctness, Integrity, Reliability, Efficiency, Usability, Maintainability, Testability, Flexibility, Portability, Reusability, dan Interoperability.
   - Mendokumentasikan 4 langkah manual sebelum produksi riil (konfigurasi snapshot cloud, pelaksanaan drill berkala, registrasi HSTS preload, dan tinjauan UU PDP).

---

## 3. Status Berkas & Tindak Lanjut

- **Dokumen Kontrak**: `contracts/api-hardening-security.md` (Tersimpan & Valid)
- **Task Board**: Baris T-071 diperbarui ke status `done`.
- **Langkah Berikutnya**: Backend-AI siap melanjutkan implementasi kode pada **Task T-072** (`cmd/seed`, middleware headers & rate limiting, serta automated route inventory tests).
