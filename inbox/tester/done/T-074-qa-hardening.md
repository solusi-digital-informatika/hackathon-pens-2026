# Task T-074: Full Security Pass, Tenant Isolation Matrix, Performance Benchmarks & Restore Drill Audit

- **Task ID**: T-074
- **Assignee**: tester
- **Status**: doing
- **Milestone**: M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Reference Contracts**:
  - `contracts/ui-hardening-security.md` (UI-M7-001)
  - `contracts/api-hardening-security.md` (API-M7-001)
  - `project-map/trd.md` (§10 Security, §12 Non-Functional Targets)
  - Backend Report: `inbox/leader/T-072-report.md` (Commit `c4a7389`)
  - Frontend Report: `inbox/leader/T-073-report.md` (Commit `ec4e1ee`)

## 1. Objective
Lakukan audit jaminan mutu independen (*independent QA audit*) komprehensif untuk pengujian akhir kesiapan rilis produksi (Milestone M7):
1. **Full Security Pass & Route Inventory Verification**:
   - Jalankan automated route inventory test di backend (`go test -v ./internal/platform/httpserver -run TestRouteInventory`).
   - Verifikasi 46 rute: pastikan tidak ada rute tanpa pengujian otorisasi tenant (fail-closed test).
   - Verifikasi penolakan cross-tenant access: user Bisnis B memanggil rute Bisnis A wajib mengembalikan HTTP 403 Forbidden.
   - Verifikasi route owner-only: user dengan peran `staff` memanggil rute khusus owner wajib mengembalikan HTTP 403 Forbidden.
   - Verifikasi HTTP security headers (`CSP`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `HSTS`, `Referrer-Policy`).
   - Verifikasi rate limiting (429 Too Many Requests dengan `Retry-After`) dan payload size limiter (413 Payload Too Large).
2. **Seed CLI & Performance Benchmarks (TRD §12)**:
   - Jalankan `go build ./cmd/seed` dan uji eksekusi seeding di backend.
   - Ukur waktu eksekusi benchmark data reference (Target PRD §8.3: standard CRUD p95 < 500ms, dashboard p95 < 2s).
3. **Database Backup & Restore Drill Verification (TRD §10, §14)**:
   - Verifikasi prosedur runbook backup & restore di sandbox testing:
     - Dump database skema + data via `pg_dump`.
     - Restore ke target database uji via `pg_restore`.
     - Validasi row count parity dan zero financial data drift.
4. **Frontend Hardening & Integrity Verification**:
   - Jalankan `npm test -- --run` dan `npm run build` di direktori frontend.
   - Verifikasi unit tests untuk `GlobalErrorBoundary`, `OfflineBanner`, `SafeExportModal`, dan netralisasi CSV injection (`'`, `=`, `+`, `-`, `@`).
   - Verifikasi ketiadaan kebocoran stack trace atau error internal pada UI fallback.

## 2. Deliverables
- Tuliskan laporan audit QA independen lengkap di `inbox/leader/T-074-report.md` mencakup hasil uji keamanan, matriks isolasi tenant, performa, hasil restore drill, dan vonis akhir (**PASS / FAIL**).
- Pindahkan file task ini ke `inbox/tester/done/T-074-qa-hardening.md`.
