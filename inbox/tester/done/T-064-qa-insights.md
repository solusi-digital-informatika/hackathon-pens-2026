# Task T-064: QA Verification for Insight Rule Correctness, Edge Cases & Data Provenance

- **Task ID**: T-064
- **Assignee**: tester
- **Status**: doing
- **Context**: Milestone M6 (Insight Engine & Recommendations)
- **Reference Contracts**:
  - `contracts/ui-insights-recommendations.md`
  - `contracts/api-insights-recommendations.md`
  - `contracts/erd.md` (§5 Diagram D)
  - `project-map/trd.md` (§7 Insight Engine Spec & TRD §9)
  - Backend implementation report: `inbox/leader/T-062-report.md` (commit `2a16809`)
  - Frontend implementation report: `inbox/leader/T-063-report.md` (commit `5aebead`)

## 1. Objective
Lakukan audit jaminan mutu independen (QA Verification) menyeluruh terhadap modul **Insight Engine & Recommendations** (Milestone M6):
1. **Rule Logic & Correctness (8 Deterministik Rules TRD §7)**:
   - Verifikasi ketepatan formula, ambang batas matematis (*threshold*), dan claim types:
     - `CUST_OVERDUE_V1` (`FAKTA`): Invoice overdue >30 hari, saldo >0.
     - `CUST_FREQ_DROP_V1` (`HIPOTESIS`): Interval pembelian drop >2x vs 90 hari terakhir.
     - `PROD_DEAD_STOCK_V1` (`FAKTA`): Stok >0, tanpa penjualan >45 hari.
     - `PROD_LOW_STOCK_V1` (`ESTIMASI`): Stok habis <7 hari berdasarkan run-rate 30 hari.
     - `PROFIT_MARGIN_DROP_V1` (`FAKTA`): Margin laba kotor drop >15% vs kuartal lalu.
     - `DISC_EROSION_V1` (`FAKTA`): Diskon rata-rata >20% mengikis margin kotor.
     - `PARETO_CONCENTRATION_V1` (`FAKTA`): Top 20% pelanggan menyumbang >70% omzet.
     - `EXPENSE_SPIKE_V1` (`FAKTA`): Pengeluaran kategori tertentu melonjak >30% MoM.
2. **Prioritization & Top 20% Spender Escalation**:
   - Uji mekanisme eskalasi otomatis: Wawasan terkait pelanggan dalam kelompok Top 20% kontributor omzet otomatis dinaikkan level urgensinya ke prioritas `HIGH`.
3. **Data Provenance & Audit Trail**:
   - Verifikasi relasi `insight_sources` dengan data transaksi asli (`TRX-` / invoice / produk).
   - Pastikan tidak ada "angka gaib" (semua angka wawasan dapat dilacak kembali ke baris sumber).
4. **Lifecycle & 14-Day Cooldown on Dismissal**:
   - Verifikasi pengabaian (*dismissal*) wawasan menyimpan alasan (`dismissed_reason`), mencatat waktu, dan menerapkan cooldown 14 hari sehingga aturan yang sama tidak membuat wawasan duplikat yang mengganggu UMKM.
5. **Rate Limiting / Manual Trigger Cooldown (10 Menit)**:
   - Verifikasi endpoint `/analysis/run` menerapkan jeda 10 menit antar run manual per bisnis untuk melindungi server dari serangan spam/DoS.
6. **Frontend UI/UX Verification**:
   - Verifikasi visual badges (`FAKTA`, `ESTIMASI`, `HIPOTESIS`), impact preview box, kelancaran filter kategori tab, modal data provenance, dan modal konfirmasi abaikan wawasan.
   - Jalankan `npm test -- --run` dan `npm run build` di direktori frontend.
   - Jalankan `go test -v ./internal/insight/...` dan `go test ./...` di direktori backend.

## 2. Deliverables
1. Jalankan test suite dan tambahkan test cases tambahan jika ditemukan celah edge case (misal: divide-by-zero pada produk tanpa penjualan, transaksi Rp 0, atau tanggal cutoff pergantian bulan).
2. Tuliskan laporan audit QA lengkap di `inbox/leader/T-064-report.md` mencakup:
   - Test execution results (Backend Go tests & Frontend Vitest).
   - Verifikasi 8 aturan deterministik & bukti matematis.
   - Verifikasi eskalasi Top 20% Pareto.
   - Verifikasi cooldown 14 hari dan rate limit 10 menit.
   - Security & integrity review (SQL injection, boundary testing, zero-division).
   - Rekomendasi / Vonis akhir (PASS / FAIL).
3. Pindahkan task ini ke `inbox/tester/done/T-064-qa-insights.md`.
