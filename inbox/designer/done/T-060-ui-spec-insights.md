# Task T-060: UI Spec for Insight Cards, Recommendations, Impact Preview & Detail Modal

- **Task ID**: T-060
- **Assignee**: designer
- **Repo**: `office-hub`
- **Output**: `contracts/ui-insights-recommendations.md`
- **Status**: doing

## 1. Objective
Rancang spesifikasi desain antarmuka lengkap (UI/UX Specification) untuk modul **Insight Engine & Recommendations** (Milestone M6).
Antarmuka ini memberdayakan pemilik UMKM untuk memahami kondisi bisnis dan pelanggan secara proaktif melalui kartu wawasan deterministik berbukti (evidence-based), aksi rekomendasi terpandu, pratinjau dampak (*impact preview*), serta transparansi data (*provenance* & tautan ke riwayat transaksi/pelanggan pendukung).

## 2. Context & Reference Files
- **TRD Spec**: `project-map/trd.md` (§7 Insight Engine Spec, §10 Security & Validation).
- **ERD Spec**: `contracts/erd.md` (§5 Diagram D: `analysis_runs`, `insights`, `insight_sources`, `recommended_actions`).
- **Design Tokens**: `contracts/design-tokens.md` (Clean Pragmatic Craftsman, aksen Emerald Forest `#0D6E64`, neutral clean borders, font Inter, tidak ada resting shadow).
- **Design Direction**: `contracts/design-direction.md`.

## 3. Scope & UI Requirements
Spesifikasi di `contracts/ui-insights-recommendations.md` harus mencakup:
1. **Insight Feed & Overview Layout**:
   - Tab navigasi / filter kategori: `Semua`, `Pelanggan` (`customer_activity`), `Produk & Margin` (`product_profitability`), `Kesehatan Data` (`data_health`).
   - Tombol manual trigger "Analisis Data Terbaru" (`POST /analysis/run`) dengan status cooldown (10 menit) dan indikator status proses analisis running.
   - Status claim-type badges yang tegas dan jujur (TRD §7):
     - `FAKTA` (Emerald / Neutral-solid, misal komplain terbuka > 3 hari, harga di bawah HPP)
     - `ESTIMASI` (Amber / Blue, misal margin di bawah rata-rata)
     - `HIPOTESIS` (Purple / Indigo lembut, misal penurunan frekuensi / overdue pembelian, tidak boleh mengklaim kepastian).
2. **Insight Card Anatomy**:
   - Header: Kategori, badge claim type (`FAKTA` / `ESTIMASI` / `HIPOTESIS`), badge prioritas (`Tinggi`, `Sedang`, `Rendah`), tanggal kalkulasi.
   - Body: Judul lugas Bahasa Indonesia, ringkasan penjelasan (template deterministik), metrik kunci (nilai saat ini vs ambang batas/historis).
   - Footer / Actions: Tombol aksi utama (misal: "Hubungi Pelanggan", "Sesuaikan Harga"), tombol "Lihat Bukti & Detail", tombol "Abaikan" (Dismiss).
3. **Insight Detail & Data Provenance Modal**:
   - Modal transparan yang menampilkan penjelasan lengkap, asumsi (*assumptions*), limitasi data (*limitations*), rule ID & versi (misal `CUST_OVERDUE_V1`), dan periode data observasi.
   - Tabel / Daftar Bukti Riwayat (*Insight Sources*): Tautan langsung ke baris transaksi atau interaksi pelanggan yang menjadi dasar wawasan, sehingga pemilik UMKM dapat memverifikasi sendiri angka tersebut.
4. **Actionable Recommendations Lifecycle**:
   - Kartu rekomendasi dengan status: `Baru`, `Ditinjau`, `Sedang Berjalan`, `Selesai`, `Diabaikan`.
   - Pratinjau Dampak (*Impact Preview*): estimasi pemulihan omzet atau pencegahan kerugian (dinyatakan secara realistis tanpa janji palsu).
   - Dialog konfirmasi abaikan (*Dismiss modal*) dengan pilihan alasan (opsional: misal "Sudah dihubungi di luar sistem", "Pelanggan musiman", dll) dengan catatan cooldown (14 hari).
5. **Responsive Breakdown & Motion Tokens**:
   - Responsive layout di 390px (mobile stack), 768px (tablet), 1440px (desktop 2-column or feed + summary).
   - Empty state informatif jika data belum mencukupi data gate (edukasi pengguna apa data yang kurang).
   - Motion tokens (150ms-250ms ease-out) untuk transisi tab dan pembukaan modal.

## 4. Definition of Done
- Dokumen spesifikasi tersimpan lengkap di `contracts/ui-insights-recommendations.md`.
- Mengikuti aturan anti-slop, token warna, typography, dan McCall's quality factors.
- Tuliskan laporan singkat ke `inbox/leader/T-060-report.md`.
- Pindahkan file task ini ke `inbox/designer/done/T-060-ui-spec-insights.md`.
