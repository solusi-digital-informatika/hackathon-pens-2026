# Laporan Penyelesaian T-062: Backend Implementation for Insight Engine, Recommendations, Provenance & Cooldown

- **Task ID**: T-062
- **Assignee**: `backend-ai`
- **Scope**: Deterministic Insight Engine, Migrations 000006, 8 Rules per TRD §7, Provenance Sources, Recommendation Lifecycle with 14-day Dismissal Cooldown, and API Handlers.
- **Status**: DONE / PUSHED

---

## 1. Algorithm
```
HTTP Request → Tenant/Auth Middleware → Handler (parse/validate DTO) → Service (cooldown check / dedupe / lifecycle) → Deterministic Engine (load evaluation context → evaluate 8 rules & top 20% spend escalation → compute dedupe keys → save runs, insights, provenance sources, recommendations & handle superseding) → DB (PostgreSQL / GORM) → Response DTO
```

---

## 2. Completed

1. **Database Migration (`migrations/000006_create_insight_tables.up.sql` & `.down.sql`)**:
   - Dibuat tabel `complaints` untuk pencatatan keluhan pelanggan.
   - Dibuat tabel `analysis_runs` dengan status enum (`running`, `completed`, `failed`), tracking durasi (`started_at`, `finished_at`), dan metadata `rules_version`.
   - Dibuat tabel `insights` dengan skema `dedupe_key`, `insight_type`, `claim_type`, `priority`, `confidence_score`, `evidence_snapshot` (JSONB), `limitations`, `superseded_at`.
   - Dibuat tabel `insight_sources` yang menyimpan relasi bukti data mentah (sumber transaksi, komplain, atau produk) beserta snapshot referensi, tanggal, nominal, dan ringkasan teks.
   - Dibuat tabel `recommended_actions` dengan lifecycle enum (`new`, `reviewed`, `in_progress`, `completed`, `dismissed`), `cooldown_until` (14 hari setelah dismiss), dan `dismissed_reason`.
   - Composite foreign keys dengan `business_id` dan indeks performa untuk pencarian dedupe key, status aktif, dan filter tenant.

2. **8 Aturan Deterministik Awal (TRD §7 & `internal/insight/rules.go`)**:
   - `CUST_OVERDUE_V1`: Mendeteksi pelanggan aktif (≥ 4 tanggal belanja dalam 180 hari) yang telat belanja melampaui interval kebiasaannya (`days_since_last > max(1.5 * median, median + 7)`).
   - `CUST_FREQ_DROP_V1`: Mendeteksi penurunan transaksi pelanggan drastis (≥ 4 transaksi pada H-91..H-180 dan transaksi 90 hari terakhir ≤ 0.5 * periode sebelumnya).
   - `CUST_VALUE_DROP_V1`: Mendeteksi penurunan rata-rata keranjang belanja pelanggan (≥ 4 tx periode sebelumnya, ≥ 2 tx periode recent, dan nilai rata-rata recent ≤ 0.6 * previous).
   - `CUST_COMPLAINT_OPEN_V1`: Mendeteksi keluhan pelanggan berstatus `open` atau `in_progress` yang belum selesai > 3 hari.
   - `PROD_BELOW_COST_V1`: Mendeteksi produk aktif dengan harga jual berada di bawah HPP (`selling_price < cost_price`).
   - `PROD_LOW_MARGIN_V1`: Mendeteksi produk dengan margin penjualan tertinggal ≥ 10 poin persentase di bawah margin rata-rata usaha (gate: minimal 10 unit terjual).
   - `DATA_MISSING_COST_V1`: Mendeteksi penjualan produk yang belum tercatat HPP-nya (`cost_coverage < 100%`).
   - `DATA_UNLINKED_TX_V1`: Mendeteksi transaksi tanpa tautan pelanggan yang melebihi 30% dari total transaksi periode 30 hari terakhir.

3. **Eskalasi Prioritas Pelanggan 20% Teratas (Top 20% Revenue Spenders)**:
   - Menghitung persentil ke-80 kontributor omzet pelanggan selama 180 hari terakhir.
   - Wawasan yang bersubjek pelanggan pada kuantil 20% teratas otomatis dieskalasikan prioritasnya satu tingkat (`low` → `medium`, `medium` → `high`), dan field `priority_boosted` diset `true`.

4. **Siklus Hidup Rekomendasi, Cooldown 14 Hari & Superseding**:
   - Status transisi tervalidasi: `new` → `reviewed` / `in_progress` / `dismissed`, `reviewed` → `in_progress` / `dismissed`, `in_progress` → `completed` / `dismissed`.
   - Aksi `dismissed` mewajibkan `dismissed_reason`, mencatat waktu `status_changed_at`, dan menetapkan `cooldown_until = status_changed_at + 14 hari`.
   - Engine mencegah pembuatan ulang atau pengubahan rekomendasi berstatus `dismissed` selama masa cooldown 14 hari masih aktif. Jika kondisi anomali masih terjadi setelah 14 hari berlalu, engine mereset status kembali menjadi `new`.
   - Mekanisme superseding otomatis: wawasan aktif yang kondisinya sudah terselesaikan/tidak terpicu lagi pada run berikutnya ditandai `superseded_at = NOW()`.

5. **API Handlers & Routing (`internal/insight/handler.go`, `cmd/api/main.go`)**:
   - `POST /api/v1/businesses/:business_id/analysis/run`: Memicu analisis deterministik manual dengan cooldown 10 menit (bisa di-bypass via `?force=true`), locking per-bisnis untuk mencegah eksekusi ganda, dan proteksi status 409 Conflict.
   - `GET /api/v1/businesses/:business_id/insights`: Daftar wawasan terpaginasi dengan filter tipe wawasan, claim type, prioritas, dan status aktif.
   - `GET /api/v1/businesses/:business_id/insights/:id`: Detail wawasan lengkap beserta asumsi metodologi, limitasi, objek rekomendasi, dan riwayat sumber pembentuk (*provenance sources*).
   - `GET /api/v1/businesses/:business_id/recommendations`: Daftar tindakan rekomendasi terpaginasi dengan filter status, prioritas, dan tipe wawasan.
   - `PATCH /api/v1/businesses/:business_id/recommendations/:id`: Pembaruan status rekomendasi beserta validasi alasan dismiss.

6. **Git Commit & Push**:
   - **Commit**: `2a16809` (`feat(insight): implement deterministic insight engine, 8 rules, provenance, and recommendation lifecycle`)
   - **Repository URL**: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
   - **Branch**: `main`

---

## 3. Verification

Semua verifikasi dieksekusi secara nyata pada lingkungan backend Go:
- `go test -v ./internal/insight/...`: **PASS** (100% test passing)
  - `TestRule_CustOverdue` (PASS)
  - `TestRule_CustFreqDrop` (PASS)
  - `TestRule_CustValueDrop` (PASS)
  - `TestRule_CustComplaintOpen` (PASS)
  - `TestRule_ProdBelowCost` (PASS)
  - `TestRule_ProdLowMargin` (PASS)
  - `TestRule_DataMissingCost` (PASS)
  - `TestRule_DataUnlinkedTx` (PASS)
  - `TestTop20PercentPriorityEscalation` (PASS)
  - `TestEngine_Lifecycle_DismissalCooldownAndSuperseding` (PASS)
  - `TestService_RecommendationLifecycleAndValidation` (PASS)
  - `TestService_AnalysisRunCooldownAndConflict` (PASS)
- `go test ./...`: **PASS** (semua unit test di seluruh package repository lulus tanpa regresi)
- `go vet ./...`: **PASS** (bersih tanpa galat atau peringatan)
- `go build ./cmd/api`: **PASS** (binary server terkompilasi sukses)

---

## 4. Issues & Limitations
- Tidak ada kendala atau konflik spesifikasi yang ditemukan.
- Implementasi engine murni deterministik berbasis aturan bisnis tanpa ketergantungan pada model machine learning eksternal, sehingga menjamin latensi rendah, reproduktifitas data, serta auditabilitas (*traceability*) 100% dari raw data ke insight.

---

## 5. Manual Task
Tidak ada tindakan manual yang diperlukan untuk kode program. Saat migrasi dijalankan ke database target (staging/production), jalankan `migrate -path migrations -database "$DATABASE_URL" up`.
