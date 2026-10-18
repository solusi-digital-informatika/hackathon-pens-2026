# Task T-071: Security Hardening Contract, Route Authorization Inventory & DB Backup Drill Spec

- **Task ID**: T-071
- **Assignee**: backend-ai
- **Repo**: `office-hub`
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: doing

## 1. Objective
Rancang kontrak arsitektur keamanan, inventarisasi otorisasi rute menyeluruh, dan prosedur backup/restore database untuk kesiapan produksi (Milestone M7) mengacu pada TRD §10, §12, & TD-05:
1. **Inventarisasi Otorisasi Rute (Route Inventory Contract)**:
   - Daftarkan seluruh rute REST API di backend (`/auth/*`, `/businesses/*`, `/products/*`, `/transactions/*`, `/customers/*`, `/finance/*`, `/imports/*`, `/insights/*`, `/recommendations/*`, `/health`, `/ready`).
   - Tentukan level proteksi tiap endpoint: `Public`, `Authenticated User`, `Tenant Scoped (Member/Owner Only)`.
   - Wajibkan mekanisme automated route inventory test: pengujian otomatis yang akan GAGAL jika ada rute baru ditambahkan ke router tanpa pengujian otorisasi tenant di test suite.
2. **Spesifikasi Tooling Seeding Mandiri (`cmd/seed`)**:
   - Spesifikasi CLI command `go run ./cmd/seed` yang menghasilkan 2 bisnis independen:
     - Bisnis A: ~10.000 transaksi realistis, produk ber-HPP & tanpa HPP, pelanggan reguler & Top 20% Pareto, keluhan terbuka, dan riwayat wawasan.
     - Bisnis B: Bisnis pembanding untuk uji isolasi tenant mutlak (verifikasi kebocoran data silang).
3. **Spesifikasi Keamanan HTTP & Transport Hardening**:
   - Pengaturan header keamanan standar produksi (`Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security`, `Referrer-Policy: strict-origin-when-cross-origin`).
   - Rate limit enforcement: 10 req/15min untuk login/refresh, 5 upload/jam untuk batch CSV, 1 run/10min untuk trigger wawasan manual.
4. **Prosedur Backup & Restore Drill (TRD §10)**:
   - Prosedur backup berbasis PostgreSQL CLI (`pg_dump` dan `pg_restore`) teruji mandiri dengan verifikasi integritas data dan checksum baris.

## 2. Deliverables
- Simpan dokumen kontrak di `contracts/api-hardening-security.md`.
- Tuliskan laporan hasil pekerjaan di `inbox/leader/T-071-report.md`.
- Pindahkan file task ini ke `inbox/backend-ai/done/T-071-api-hardening.md`.
