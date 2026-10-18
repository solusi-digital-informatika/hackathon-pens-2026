# Task T-062: Implement Backend Deterministic Insight Engine, Rule Processors & Audit

- **Task ID**: T-062
- **Assignee**: backend-ai
- **Repo**: `backend` (`https://github.com/solusi-digital-informatika/teman-umkm-backend.git`)
- **Status**: doing

## 1. Objective
Implementasikan modul backend **Insight Engine & Recommendations** (Milestone M6) secara deterministik (tanpa model LLM eksternal/halusinasi, TRD TD-12) di repositori backend Go:
1. Migrasi database versioned (`migrations/000006_create_insight_tables.up.sql` dan `.down.sql`) untuk tabel `analysis_runs`, `insights`, `insight_sources`, `recommended_actions`, indexes, dan unique constraints per `contracts/erd.md`.
2. Pipeline engine analisis deterministik (`internal/insight/engine.go`):
   - Mutex / DB lock agar hanya ada satu run berstatus `running` per bisnis.
   - Pengecekan rate limit manual trigger (cooldown 10 menit paska analisis terakhir selesai).
   - Evaluasi 8 aturan awal sesuai spesifikasi TRD §7 & `contracts/api-insights-recommendations.md`:
     - `CUST_OVERDUE_V1` (Customer overdue interval)
     - `CUST_FREQ_DROP_V1` (Purchase frequency drop)
     - `CUST_VALUE_DROP_V1` (Average ticket value drop)
     - `CUST_COMPLAINT_OPEN_V1` (Unresolved complaint > 3 hari)
     - `PROD_BELOW_COST_V1` (Selling price below active HPP)
     - `PROD_LOW_MARGIN_V1` (Margin 10% di bawah rerata usaha)
     - `DATA_MISSING_COST_V1` (Sales without HPP, cost coverage < 100%)
     - `DATA_UNLINKED_TX_V1` (Unlinked customer transactions > 30%)
3. Logic Prioritisasi & Eskalasi:
   - Hitung Top 20% customer by revenue 180 hari terakhir. Jika pelanggan target masuk Top 20%, eskalasi prioritas wawasan naik 1 level (`low` -> `medium`, `medium` -> `high`).
4. Siklus Hidup Wawasan & Rekomendasi:
   - Pengenalan `dedupe_key`: pemutakhiran in-place jika anomali masih ada pada run berikutnya.
   - Penandaan `superseded_at` jika kondisi anomali teratasi oleh transaksi baru.
   - Dukungan `dismissed` dengan pencatatan `dismissed_reason` dan penegakan jeda 14 hari (*cooldown*) sebelum wawasan serupa boleh dibuat kembali.
   - Pencatatan bukti riwayat data mentah (`insight_sources`) ke transaksi/produk/komplain.
5. REST API Handlers & HTTP Server Wiring:
   - Implementasikan 5 endpoint per `contracts/api-insights-recommendations.md`:
     - `POST /api/v1/businesses/{business_id}/analysis/run`
     - `GET /api/v1/businesses/{business_id}/insights`
     - `GET /api/v1/businesses/{business_id}/insights/{id}`
     - `GET /api/v1/businesses/{business_id}/recommendations`
     - `PATCH /api/v1/businesses/{business_id}/recommendations/{id}`
   - Wire route ke `internal/platform/httpserver/server.go` dan `cmd/api/main.go`.
6. Automated Unit Tests:
   - Buat unit test komprehensif di `internal/insight/insight_test.go` yang memvalidasi setiap aturan (flagged, not flagged, below data gate), prioritas Top 20%, cooldown dismissal, dan lifecycle superseding.

## 2. Reference & Standards
- API Contract: `office-hub/contracts/api-insights-recommendations.md` (Wajib diikuti secara ketat).
- UI Spec: `office-hub/contracts/ui-insights-recommendations.md`.
- TRD Specification: `office-hub/project-map/trd.md` (§7 Insight Engine Spec, §9 API Inventory, §10 Security).
- ERD: `office-hub/contracts/erd.md` (§5 Diagram D).

## 3. Definition of Done
1. Semua migrasi SQL siap dan valid.
2. `go test -v ./internal/insight/...` lulus 100%.
3. `go test -v ./...` dan `go build ./cmd/api` lulus tanpa error atau race warning.
4. `go vet ./...` bersih (0 error).
5. Lakukan git commit dengan Conventional Commits: `feat(insight): implement deterministic insight engine, 8 rules, provenance, and recommendation lifecycle`.
6. Push ke remote `origin main` di repo `backend`.
7. Tulis laporan hasil pengerjaan ke `inbox/leader/T-062-report.md` lengkap dengan commit hash dan bukti tes.
8. Pindahkan file task ini ke `inbox/backend-ai/done/T-062-backend-insights.md`.
