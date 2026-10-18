# Task T-061: API Contract for Insight Rules Engine, Dismissals & Feedback

- **Task ID**: T-061
- **Assignee**: backend-ai
- **Repo**: `office-hub`
- **Output**: `contracts/api-insights-recommendations.md`
- **Status**: doing

## 1. Objective
Rancang kontrak antarmuka API lengkap (`contracts/api-insights-recommendations.md`) untuk modul **Insight Engine & Recommendations** (Milestone M6).
Kontrak ini mendefinisikan skema REST endpoint untuk eksekusi pipeline analisis deterministik, pengambilan daftar insight dan rekomendasi, detail insight beserta sumber bukti data mentah (*provenance*), serta aksi pembaruan status rekomendasi dan pengabaian (*dismissal with cooldown*).

## 2. Context & Reference Files
- **TRD Spec**: `project-map/trd.md` (§7 Insight Engine Spec, §9 API Inventory, §10 Security & Validation).
- **ERD Spec**: `contracts/erd.md` (§5 Diagram D: tabel `analysis_runs`, `insights`, `insight_sources`, `recommended_actions`).
- **Conventions**:
  - URL Prefix: `/api/v1/businesses/{business_id}/...`.
  - JSON snake_case, standard list response `{items, page, page_size, total}`.
  - Standard error format `{error: {code, message, details?}}`.
  - Money sebagai string representasi desimal, tanggal `YYYY-MM-DD`, waktu RFC 3339 UTC.

## 3. Scope & Endpoints to Specify
1. **Trigger Analysis Run**:
   - `POST /api/v1/businesses/{business_id}/analysis/run`
   - Trigger manual dengan idempotensi & cooldown guard (maksimal 1 run per 10 menit per bisnis jika data tidak berubah).
   - Menghasilkan record `analysis_runs` dengan status `running` -> `completed` / `failed`.
2. **List Insights**:
   - `GET /api/v1/businesses/{business_id}/insights`
   - Filter query params: `type` (`customer_activity`, `product_profitability`, `data_health`), `claim_type` (`fact`, `estimate`, `hypothesis`), `is_active` (boolean, default true: `superseded_at IS NULL`), pagination.
   - Response DTO mencakup snapshot ringkas nilai metriks, prioritas, status rekomendasi tertaut.
3. **Get Insight Detail & Provenance**:
   - `GET /api/v1/businesses/{business_id}/insights/{id}`
   - Mengembalikan detail lengkap insight: title, explanation, assumptions, limitations, rule_id, rule_version, calculated_at.
   - Mengembalikan array `sources` (`insight_sources`): entity_type (`transaction`, `customer`, `product`, `complaint`), entity_id, dan ringkasan data sumber terkait.
4. **List Recommendations**:
   - `GET /api/v1/businesses/{business_id}/recommendations`
   - Filter: `status` (`new`, `reviewed`, `in_progress`, `completed`, `dismissed`), `priority` (`high`, `medium`, `low`).
5. **Update Recommendation Status**:
   - `PATCH /api/v1/businesses/{business_id}/recommendations/{id}`
   - Mengubah status (misal: menandai `in_progress`, `completed`, atau `dismissed`).
   - Jika `dismissed`: wajib mencatat `reason` dan mengaktifkan cooldown dedupe key (default 14 hari).
6. **Insight Rule Engine Specifications**:
   - Sertakan definisi 8 aturan deterministik awal TRD §7 (`CUST_OVERDUE_V1`, `CUST_FREQ_DROP_V1`, `CUST_VALUE_DROP_V1`, `CUST_COMPLAINT_OPEN_V1`, `PROD_BELOW_COST_V1`, `PROD_LOW_MARGIN_V1`, `DATA_MISSING_COST_V1`, `DATA_UNLINKED_TX_V1`).
   - Aturan penentuan prioritas (Top 20% customer revenue naik 1 tingkat).
   - Aturan superseding wawasan ketika kondisi tidak lagi terpenuhi.

## 4. Definition of Done
- Dokumen spesifikasi tersimpan lengkap di `contracts/api-insights-recommendations.md`.
- Sesuai kaidah REST, DTO terpisah dari model GORM, user-safe error messages, dan isolasi tenant ketat.
- Tuliskan laporan singkat ke `inbox/leader/T-061-report.md`.
- Pindahkan file task ini ke `inbox/backend-ai/done/T-061-api-contract-insights.md`.
