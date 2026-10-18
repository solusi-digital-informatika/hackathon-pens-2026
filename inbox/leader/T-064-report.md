# QA Audit & Verification Report: Milestone M6 (Insight Engine & Recommendations)

- **Task ID**: T-064
- **Assignee**: `tester` (QA & Security Tester)
- **Target Milestone**: Milestone M6 (Insight Engine & Recommendations)
- **Status**: COMPLETED
- **Final Verdict**: **PASS / APPROVE**
- **Date**: 18 Oktober 2026

---

## 1. Scope & Tested Commits

### 1.1 Repositories and Commits Under Test
- **Backend Repository**: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
  - Coder Pushed Commit (`backend-ai`): `2a16809` (`feat(insight): implement deterministic insight engine, 8 rules, provenance, and recommendation lifecycle`)
  - QA Verified & Test-Augmented Commit (`tester`): `91f6d63` (`test(insight): add edge case, boundary threshold, and tenant isolation tests`)
- **Frontend Repository**: `https://github.com/solusi-digital-informatika/teman-umkm-frontend.git`
  - Coder Pushed Commit (`frontend`): `5aebead33dce4692bb9febe8eead2fb735bdaa9b` (`feat(insights): implement insight feed, provenance modal, and recommendation lifecycle`)
- **Reference Contracts**:
  - `contracts/api-insights-recommendations.md` (API-M6-001)
  - `contracts/ui-insights-recommendations.md` (UI-M6-001)
  - `contracts/erd.md` (§5 Diagram D, §7 Rules & Enums)
  - `project-map/trd.md` (§7 Insight Engine Spec, §9 API Inventory, §10 Security & Rate Limits)

### 1.2 Coverage Matrix
- **Covered**:
  - 8 Initial Deterministic Rules per TRD §7 (formula correctness, thresholds, data gates, and claim types).
  - Priority boost and Top 20% Pareto revenue spender escalation.
  - Transactional data provenance (`insight_sources` table and audit traceability).
  - Recommendation lifecycle, 14-day dismissal cooldown, and condition superseding.
  - 10-minute analysis run cooldown, concurrency locking, and 409/429 HTTP protections.
  - Multi-tenant isolation and data protection.
  - Edge cases (divide-by-zero, Rp 0 transactions, boundary values, empty context).
  - Frontend UI/UX components (claim badges, impact previews, provenance modal, dismissal dialog, empty states).
- **Not Covered**:
  - Production database load under millions of concurrent rows (evaluated up to standard local test volumes).
  - External ML services (TRD §7 mandates 100% deterministic rule engine for M6; no external ML dependency).

---

## 2. Test Execution Results

### 2.1 Backend Unit & Integration Tests (Go)
Command: `go test -v ./internal/insight/...`
- **Result**: **PASS** (16/16 test suites passing in 1.74s)
  ```
  === RUN   TestEdgeCases_ZeroDivisionAndEmptyData
  --- PASS: TestEdgeCases_ZeroDivisionAndEmptyData (0.00s)
  === RUN   TestEdgeCases_BoundaryThresholds
  --- PASS: TestEdgeCases_BoundaryThresholds (0.00s)
  === RUN   TestEdgeCases_TenantIsolation
  --- PASS: TestEdgeCases_TenantIsolation (0.00s)
  === RUN   TestEdgeCases_Top20PercentEscalationNuances
  --- PASS: TestEdgeCases_Top20PercentEscalationNuances (0.00s)
  === RUN   TestRule_CustOverdue
  --- PASS: TestRule_CustOverdue (0.00s)
  === RUN   TestRule_CustFreqDrop
  --- PASS: TestRule_CustFreqDrop (0.00s)
  === RUN   TestRule_CustValueDrop
  --- PASS: TestRule_CustValueDrop (0.00s)
  === RUN   TestRule_CustComplaintOpen
  --- PASS: TestRule_CustComplaintOpen (0.00s)
  === RUN   TestRule_ProdBelowCost
  --- PASS: TestRule_ProdBelowCost (0.00s)
  === RUN   TestRule_ProdLowMargin
  --- PASS: TestRule_ProdLowMargin (0.00s)
  === RUN   TestRule_DataMissingCost
  --- PASS: TestRule_DataMissingCost (0.00s)
  === RUN   TestRule_DataUnlinkedTx
  --- PASS: TestRule_DataUnlinkedTx (0.00s)
  === RUN   TestTop20PercentPriorityEscalation
  --- PASS: TestTop20PercentPriorityEscalation (0.00s)
  === RUN   TestEngine_Lifecycle_DismissalCooldownAndSuperseding
  --- PASS: TestEngine_Lifecycle_DismissalCooldownAndSuperseding (0.00s)
  === RUN   TestService_RecommendationLifecycleAndValidation
  --- PASS: TestService_RecommendationLifecycleAndValidation (0.00s)
  === RUN   TestService_AnalysisRunCooldownAndConflict
  --- PASS: TestService_AnalysisRunCooldownAndConflict (0.00s)
  PASS
  ok  	github.com/solusi-digital-informatika/teman-umkm-backend/internal/insight	1.744s
  ```

Full Backend Suite Regression: `go test ./...`
- **Result**: **PASS** (all packages pass: auth, business, customer, finance, importer, insight, product, transaction, platform).
- Static Analysis: `go vet ./...` completed with 0 errors and 0 warnings.

### 2.2 Frontend Unit Tests & Build Verification (React + Vite)
Command: `npm test -- --run`
- **Result**: **PASS** (6/6 test files passed, 27/27 tests passed)
  ```
  Test Files  6 passed (6)
       Tests  27 passed (27)
    Duration  5.23s
  ```
Command: `npm run build` (`tsc -b && vite build`)
- **Result**: **PASS** (Production bundle generated cleanly with zero TypeScript errors).

---

## 3. Mathematical & Rule Logic Verification (8 Deterministic Rules TRD §7)

| Rule ID | Signal | Data Gate | Trigger Condition | Claim Type | Base Priority | QA Verification Result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `CUST_OVERDUE_V1` | Pelanggan melampaui interval belanja biasa | $\ge 4$ tanggal transaksi berbeda dalam 180 hari | `days_since_last > ceil(max(1.5*median, median+7))` | `hypothesis` | `medium` | **PASS**: Formula median terurut, toleransi adaptif terverifikasi, boundary test (17 vs 18 hari) tepat. |
| `CUST_FREQ_DROP_V1` | Frekuensi belanja turun signifikan | $\ge 4$ transaksi periode H-91 s/d H-180 | `tx_recent_90d <= 0.5 * tx_prev_90d` | `hypothesis` | `medium` | **PASS**: Gate mencegah pelanggan baru terlabeli salah; rasio 50% terverifikasi. |
| `CUST_VALUE_DROP_V1` | Nilai belanja rata-rata turun | $\ge 4$ tx prev dan $\ge 2$ tx recent | `avg_order_recent <= 0.6 * avg_order_prev` | `hypothesis` | `low` | **PASS**: Zero-division dicegah oleh data gate (divisor $\ge 2$ dan $\ge 4$); penurunan 40% teruji. |
| `CUST_COMPLAINT_OPEN_V1` | Keluhan pelanggan belum terselesaikan | Status `open` / `in_progress` | `days_open > 3` hari kalender | `fact` | `high` | **PASS**: Tepat 3 hari tidak memicu wawasan; $\ge 4$ hari memicu wawasan prioritas tinggi. |
| `PROD_BELOW_COST_V1` | Harga jual di bawah modal HPP | Status `active` & memiliki HPP $> 0$ | `selling_price < cost_price` | `fact` | `high` | **PASS**: $selling = cost$ tidak memicu; $selling < cost$ memicu penghitungan kerugian per unit. |
| `PROD_LOW_MARGIN_V1` | Margin produk tertinggal di bawah rata-rata bisnis | Penjualan ber-HPP $\ge 10$ unit, penjualan bisnis ber-HPP $> 0$ | `product_margin_pct <= biz_margin_pct - 10.0` | `estimate` | `medium` | **PASS**: Selisih 10 poin persentase terverifikasi; proteksi pembagian nol bila total penjualan nol. |
| `DATA_MISSING_COST_V1` | Terdapat penjualan produk tanpa HPP | $\ge 1$ item transaksi dalam 30 hari | `cost_coverage_pct < 100.0` | `fact` | `medium` | **PASS**: Deteksi coverage omzet terverifikasi; snapshot memuat persentase dan nominal omzet estimasi. |
| `DATA_UNLINKED_TX_V1` | Banyak transaksi tanpa tautan pelanggan | $\ge 20$ transaksi dalam 30 hari | `unlinked_count / total_tx > 0.30` | `fact` | `low` | **PASS**: Tepat 30.0% (6/20) tidak memicu; 35.0% (7/20) memicu peringatan data health. |

---

## 4. Prioritization & Top 20% Pareto Escalation

1. **Kalkulasi Kontribusi Omzet (TRD §7 & PRD §5.9)**:
   - Sistem mengakumulasi total belanja non-batal dalam rentang 180 hari.
   - Peringkat pelanggan dihitung dengan persentil $TopCount = \lceil 0.20 \times N \rceil$ (minimal 1 jika ada pelanggan).
2. **Penerapan Eskalasi (Boost)**:
   - Pelanggan dalam kelompok 20% teratas otomatis dieskalasikan 1 tingkat:
     - `low` $\rightarrow$ `medium`
     - `medium` $\rightarrow$ `high`
     - `high` $\rightarrow$ `high` (dengan penanda `priority_boosted = true`).
3. **Pemisahan Subject (Safety Check)**:
   - Terverifikasi dalam `TestEdgeCases_Top20PercentEscalationNuances`: Subjek `product` (`PROD_LOW_MARGIN_V1`) dan `business` (`DATA_UNLINKED_TX_V1`) **tidak pernah** dinaikkan statusnya oleh aturan Pareto pelanggan.

---

## 5. Data Provenance & Audit Trail Verification

1. **Relasi Entitas (`insight_sources`)**:
   - Seluruh baris wawasan terhubung dengan record asli melalui tabel `insight_sources` (`entity_type: "transaction" | "customer" | "product" | "complaint"`).
   - Detail wawasan pada endpoint `GET /insights/:id` memperkaya entitas transaksi dengan `reference_no` (contoh: `TRX-20261001-001`), `date`, `amount`, `status`, dan ringkasan faktur.
2. **Eliminasi "Angka Gaib"**:
   - Nilai pada `evidence_snapshot` (seperti `days_since_last`, `median_interval_days`, `unlinked_tx_pct`, `cost_coverage_pct`) terbukti 100% dapat dilacak kembali ke baris data mentah yang tersimpan.

---

## 6. Lifecycle, 14-Day Dismissal Cooldown & Superseding

1. **Validasi Alasan Pengabaian (`dismissed_reason`)**:
   - Status transisi ke `dismissed` mewajibkan pengiriman `dismissed_reason`. Request tanpa alasan ditolak dengan `400 DISMISSAL_REASON_REQUIRED`.
2. **Jeda Cooldown 14 Hari**:
   - Saat diabaikan, `cooldown_until` diset menjadi `status_changed_at + 14 hari`.
   - Engine analisis terbukti **tidak membuat wawasan duplikat** selama masa 14 hari aktif (`cooldown_skipped_count` bertambah).
   - Setelah 14 hari berlalu dan kondisi anomali tetap berlanjut, rekomendasi otomatis direset kembali ke status `new` untuk ditinjau ulang.
3. **Mekanisme Superseding Otomatis**:
   - Wawasan aktif yang kondisinya sudah tidak lagi terpenuhi (misal komplain telah berstatus `resolved` atau HPP telah dilengkapi) otomatis ditandai `superseded_at = NOW()`.

---

## 7. Rate Limiting & 10-Minute Analysis Run Cooldown

1. **Proteksi Concurrency (`409 Conflict`)**:
   - Jika terdapat analisis dengan status `running` untuk bisnis yang sama, pemanggilan baru mengembalikan `409 Conflict` (`ANALYSIS_ALREADY_RUNNING`) disertai metadata `active_run_id` dan `started_at`.
2. **Jeda 10 Menit Manual Run (`429 Too Many Requests`)**:
   - Pemanggilan manual sebelum jeda 10 menit selesai ditolak dengan `429 Too Many Requests` (`ANALYSIS_COOLDOWN_ACTIVE`), header `Retry-After: <detik>`, serta rincian waktu cooldown.
3. **Bypass Flag (`force=true`)**:
   - Penggunaan parameter `force=true` berhasil mem-bypass cooldown 10 menit (diperlukan untuk sinkronisasi pasca-impor data besar).

---

## 8. Security & Multi-Tenant Isolation Review

| Kontrol Keamanan | Status | Catatan Audit |
| :--- | :--- | :--- |
| **BOLA / Multi-Tenant Isolation** | **PASS** | Akses wawasan/rekomendasi milik bisnis lain menghasilkan `ErrInsightNotFound` / `404 Not Found`. Query repositori selalu menyertakan `business_id`. |
| **SQL Injection** | **PASS** | Semua query database menggunakan parameter binding GORM (`?` placeholders). Tidak ada penggabungan string SQL mentah. |
| **Zero Division / Arithmetic Safety** | **PASS** | Seluruh pembagian persentase dan rata-rata terlindungi oleh pengecekan nilai nol (`netSales > 0`, `totalTx >= 20`, data gates). Teruji bersih dari nilai `NaN` atau `Inf`. |
| **BOPLA / Mass Assignment** | **PASS** | Payload pembaruan rekomendasi strictly divalidasi lewat DTO (`UpdateRecommendationStatusRequest`). Field internal (`business_id`, `insight_id`) tidak dapat ditimpa user. |

---

## 9. Frontend UI/UX Verification

1. **Visual Badges & Tone**:
   - Badge `FAKTA` (Emerald `#ECFDF5`), `ESTIMASI` (Amber `#FFFBEB`), dan `HIPOTESIS` (Purple `#F5F3FF`) tampil akurat sesuai panduan desain.
   - Narasi penjelasan pada hipotesis tidak menggunakan kata kepastian ("pasti berhenti"), menjaga kepatuhan PRD §5.9.
2. **Data Provenance Modal**:
   - Modal menampilkan transparansi kalkulasi, asumsi sistem, limitasi analisis, serta tabel audit trail sumber transaksi berformat `TRX-...`.
3. **Dismiss Dialog & Cooldown Notice**:
   - Dialog abaikan wawasan menyediakan 5 pilihan alasan standar, konfirmasi jeda 14 hari, dan feedback toast sukses.
4. **Impact Preview**:
   - Kotak preview dampak finansial dengan aksen garis kiri hijau emerald merender nilai moneter secara defensif baik dalam bentuk string maupun objek terstruktur.

---

## 10. Summary & Recommendation

- **Defects Found**: 0 blocker, 0 high, 0 medium.
- **Regression Tests Added**: `internal/insight/edge_cases_test.go` (335 baris kode pengujian mencakup pembagian nol, batas ambang, isolasi tenant, dan nuansa prioritas Pareto).
- **Recommendation**: **APPROVE** untuk Milestone M6. Seluruh fungsionalitas siap untuk Milestone Review & Sign-Off oleh Leader (Task T-065).
