# Sign-Off Report: Milestone M6 (Insight Engine & Recommendations)

- **Milestone**: M6 — Insight Engine & Recommendations
- **Evaluator**: `leader` (Product Owner & Coordinator)
- **Status**: **ACCEPTED & CLOSED**
- **Date**: 18 Oktober 2026
- **Repositories**:
  - `backend`: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git` (Commit `2a16809` by `backend-ai`, QA suite `91f6d63` by `tester`)
  - `frontend`: `https://github.com/solusi-digital-informatika/teman-umkm-frontend.git` (Commit `5aebead` by `frontend`)
  - `office-hub`: `https://github.com/solusi-digital-informatika/hackathon-pens-2026.git`

---

## 1. Executive Summary
Milestone M6 mengimplementasikan modul inti pembeda sistem Teman UMKM: **Deterministic Insight Engine & Actionable Recommendations**. Seluruh kapabilitas analitik bisnis dijalankan secara 100% deterministik berbasis formula matematis, kueri database terindeks, dan aturan bisnis terstruktur tanpa ketergantungan pada model LLM eksternal sehingga menjamin konsistensi mutlak, kecepatan eksekusi tinggi, nol risiko halusinasi, dan transparansi data provenance penuh hingga ke baris transaksi asli.

Semua pekerjaan dari perancangan antarmuka, kontrak REST API, implementasi backend Go, frontend React TypeScript, hingga audit independen jaminan mutu telah tuntas 100% dan disetujui (Verdict: **PASS / APPROVE**).

---

## 2. Review Deliverables & Verification Checklist

| Task ID | Deskripsi Task | Assignee | Deliverable / Artifact | Status |
| :--- | :--- | :--- | :--- | :--- |
| **T-060** | UI Spec for Insight Cards, Recommendations, Impact Preview & Detail Modal | `designer` | `contracts/ui-insights-recommendations.md`<br>Laporan: `inbox/leader/T-060-report.md` | **DONE** |
| **T-061** | API Contract for Insight Rules Engine, Dismissals & Feedback | `backend-ai` | `contracts/api-insights-recommendations.md`<br>Laporan: `inbox/leader/T-061-report.md` | **DONE** |
| **T-062** | Implement Backend Deterministic Insight Engine, Rule Processors & Audit | `backend-ai` | Migrasi DB 000006, `internal/insight/`<br>Commit: `2a16809`<br>Laporan: `inbox/leader/T-062-report.md` | **DONE** |
| **T-063** | Implement Frontend Insight Feed, Recommendation Actions & Detail Modal | `frontend` | Modul `src/modules/insights/`, routing, modal provenance<br>Commit: `5aebead`<br>Laporan: `inbox/leader/T-063-report.md` | **DONE** |
| **T-064** | QA Verification for Insight Rule Correctness, Edge Cases & Data Provenance | `tester` | Edge cases suite, multi-tenant & boundary tests<br>Commit: `91f6d63`<br>Laporan: `inbox/leader/T-064-report.md` (PASS) | **DONE** |
| **T-065** | M6 Milestone Review & Sign-Off | `leader` | Sign-off report `inbox/leader/T-065-report.md` & update board | **DONE** |

---

## 3. Core Architectural Highlights & Compliance

1. **8 Initial Deterministic Rules TRD §7**:
   - `CUST_OVERDUE_V1` (`hypothesis`): Mendeteksi pelanggan yang melampaui interval belanja biasa dengan formula adaptif `ceil(max(1.5*median, median+7))` dan data gate minimal 4 tanggal transaksi.
   - `CUST_FREQ_DROP_V1` (`hypothesis`): Frekuensi belanja turun $\ge 50\%$ vs kuartal sebelumnya.
   - `CUST_VALUE_DROP_V1` (`hypothesis`): Nilai rata-rata pesanan turun $\ge 40\%$.
   - `CUST_COMPLAINT_OPEN_V1` (`fact`): Keluhan pelanggan belum tuntas $> 3$ hari (prioritas `high`).
   - `PROD_BELOW_COST_V1` (`fact`): Harga jual di bawah modal HPP ($selling < cost$).
   - `PROD_LOW_MARGIN_V1` (`estimate`): Margin kotor produk tertinggal $\ge 10$ poin persentase di bawah rata-rata bisnis.
   - `DATA_MISSING_COST_V1` (`fact`): Transaksi produk tanpa modal HPP terdeteksi.
   - `DATA_UNLINKED_TX_V1` (`fact`): Lebih dari 30% transaksi tidak terhubung ke pelanggan terdaftar.
2. **Prioritization & Top 20% Pareto Escalation**:
   - Sistem secara otomatis menghitung akumulasi omzet pelanggan 180 hari dan mengidentifikasi kelompok Top 20% kontributor omzet.
   - Wawasan yang bersumber dari pelanggan Top 20% secara otomatis dielevasi prioritasnya 1 tingkat lebih tinggi (`low` $\rightarrow$ `medium`, `medium` $\rightarrow$ `high`) untuk perlindungan relasi strategis.
3. **Data Provenance & Audit Trail**:
   - Transparansi 100%: Setiap wawasan terhubung dengan record `insight_sources` yang menyimpan ID transaksi nyata (`TRX-...`), nilai nominal, tanggal, dan bukti kontekstual. Tidak ada angka gaib (*black box*).
4. **Lifecycle & 14-Day Dismissal Cooldown**:
   - Pengabaian rekomendasi (*dismissal*) mewajibkan alasan terstruktur dan mengaktifkan cooldown 14 hari, mencegah wawasan berulang yang mengganggu pengguna.
   - Wawasan yang kondisinya telah membaik secara otomatis ditandai `superseded_at = NOW()`.
5. **Rate Limiting & Safety Safeguards**:
   - Pemicu analisis manual dibatasi cooldown 10 menit antar run per entitas bisnis (HTTP 429 Too Many Requests dengan header `Retry-After`).
   - Concurrency lock mencegah eksekusi ganda yang bertabrakan (HTTP 409 Conflict).

---

## 4. Test Execution Evidence
- **Backend Tests (`go test -v ./internal/insight/...`)**: 16/16 test suites passing (1.74s).
- **Backend Full Regression (`go test ./...`)**: 100% PASS, `go vet` clean (0 warnings).
- **Frontend Unit Tests (`npm test -- --run`)**: 6/6 test files, 27/27 tests passing (5.23s).
- **Frontend Production Build (`npm run build`)**: Lulus bersih tanpa error (`tsc -b && vite build`).

---

## 5. Milestone Verdict
Dengan terpenuhinya seluruh kriteria penerimaan fungsional, arsitektural, keamanan data, dan lolosnya audit jaminan mutu independen, Milestone M6 dinyatakan:

**STATUS: ACCEPTED & CLOSED**
Milestone berikutnya yang siap diaktifkan adalah **Milestone M7 (Financial Reports & Export)**.
