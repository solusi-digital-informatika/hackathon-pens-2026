# Task T-072: Implement Backend cmd/seed, Route Inventory Authorization Tests & Security Hardening

- **Task ID**: T-072
- **Assignee**: backend-ai
- **Repo**: `backend` (`https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`)
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: doing

## 1. Objective
Implementasikan tooling seeding mandiri, audit otorisasi rute otomatis (*route inventory test*), dan hardening keamanan tingkat produksi di repositori `backend` mengacu pada `contracts/api-hardening-security.md` (API-M7-001) dan TRD §10, §12:
1. **Tooling Seeding Mandiri (`cmd/seed/main.go`)**:
   - Bangun CLI command mandiri `cmd/seed/main.go` yang mendukung flag:
     - `--clean`: Membersihkan data lama sebelum insert (khusus environment lokal/testing).
     - `--scale`: `minimal` (untuk test cepat) atau `standard` (10.000 transaksi realistis).
     - `--database-url`: Koneksi string Postgres target (fallback ke `DATABASE_URL` env).
   - Generasi Bisnis A (Toko Berkah Mandiri):
     - Akun owner & staff terdaftar (password: `Password123!`).
     - Katalog produk ber-HPP & produk tanpa HPP (untuk menguji `DATA_MISSING_COST_V1`).
     - 2.000 pelanggan dengan segmentasi Pareto 80/20 (Top 20% Spender).
     - Transaksi historis (180 hari) termasuk variasi tunai, QRIS, piutang tempo, dan retur item.
     - Riwayat keluhan pelanggan (status `open` > 3 hari untuk menguji `CUST_COMPLAINT_OPEN_V1`).
   - Generasi Bisnis B (Warung Sejahtera):
     - Entitas bisnis kedua yang terisolasi total untuk menguji isolasi multi-tenant.
2. **Automated Route Inventory Authorization Test (`internal/httpserver/security_inventory_test.go`)**:
   - Tulis unit/integration test yang menginspeksi seluruh rute yang terdaftar pada engine HTTP router (`Server.Engine().Routes()`).
   - Verifikasi bahwa setiap rute non-public terikat pada middleware otorisasi tenant / auth.
   - Uji penolakan akses (HTTP 403 Forbidden / 401 Unauthorized) jika token Bisnis B mengakses endpoint milik Bisnis A.
   - Sifat tes: *Fail-closed* (akan GAGAL jika ada rute baru ditambahkan tanpa terdaftar di matriks keamanan).
3. **HTTP Security Headers & Rate Limiting Hardening**:
   - Pastikan middleware menambahkan header produksi: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`.
   - Pastikan request body limiter aktif (1MB default, 10MB untuk upload CSV multipart).
   - Pastikan rate limiter menolak request melebihi kuota dengan HTTP 429 dan header `Retry-After`.

## 2. Verification & Constraints
- Jalankan `go test -v ./...` dan `go vet ./...` di repositori backend; semua package harus 100% PASS tanpa regression.
- Jalankan `go run ./cmd/seed --scale minimal` untuk memastikan proses seeding berjalan lancar.
- Lakukan git commit dengan Conventional Commits: `feat(security): implement cmd/seed, route inventory authorization tests, and security hardening`.
- Push ke remote origin `main` pada repositori backend.
- Tuliskan laporan hasil pekerjaan di `inbox/leader/T-072-report.md`.
- Pindahkan file ini ke `inbox/backend-ai/done/T-072-backend-hardening.md`.
