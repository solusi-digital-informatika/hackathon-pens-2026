# Laporan Penyelesaian T-060: UI Specification for Insight Engine & Recommendations

- **Task ID**: T-060
- **Assignee**: `designer`
- **Deliverable**: `contracts/ui-insights-recommendations.md`
- **Status**: DONE / APPROVED

Telah selesai dirancang dan didokumentasikan spesifikasi antarmuka dan interaksi komprehensif untuk modul **Insight Engine & Recommendations** (Milestone M6) pada `contracts/ui-insights-recommendations.md`, yang sepenuhnya mematuhi prinsip desain Clean Pragmatic Craftsman, Design Tokens (`contracts/design-tokens.md`), TRD §7 (Insight Engine Spec), TRD §9 (API Inventory), TRD §10 (Rate Limiting), dan ERD §4 (Diagram C: Customer & Intelligence).

### Rincian Cakupan Spesifikasi:

1. **Filosofi "Evidence-First Intelligence" & Integritas Klaim**:
   - Menghilangkan halusinasi angka dan kepalsuan prediksi AI: wawasan dirancang deterministik berbasis template terverifikasi.
   - Larangan klaim kepastian mutlak pada wawasan bertipe hipotesis (misal dilarang menyatakan "Pelanggan pasti berhenti", wajib dinyatakan secara jujur sebagai deviasi pola historis).
   - Penolakan terhadap slop visual (tanpa gradien ungu/biru "AI glow", tanpa ilustrasi mengambang, tanpa resting shadow pada kartu diam).

2. **Sistem Token Badge Semantik & Kontras WCAG AA**:
   - **Claim-Type Badges**:
     - `FAKTA` (Emerald solid `#ECFDF5` / `#065F46` / `#A7F3D0`): data riil faktual dari baris transaksi/komplain terdaftar (rasio kontras 5.8:1).
     - `ESTIMASI` (Amber `#FFFBEB` / `#92400E` / `#FDE68A`): nilai dihitung dari kalkulasi rerata/agregasi periode tertentu (rasio kontras 5.1:1).
     - `HIPOTESIS` (Indigo lembut `#F5F3FF` / `#5B21B6` / `#DDD6FE`): deviasi interval belanja atau tren frekuensi (rasio kontras 6.2:1).
   - **Priority Badges**: `Tinggi` (merah bata), `Sedang` (amber), `Rendah` (slate netral) dengan perhitungan *boost* 1 tingkat untuk pelanggan kategori Top 20% omzet 180 hari terakhir.
   - **Status Badge Rekomendasi**: `new`, `reviewed`, `in_progress`, `completed`, `dismissed`, dan `superseded`.

3. **Struktur Feed & Kontrol Analisis Manual**:
   - Tab navigasi kategori: `Semua Wawasan`, `Pelanggan` (`customer_activity`), `Produk & Margin` (`product_profitability`), dan `Kesehatan Data` (`data_health`).
   - Kontrol pemicu analisis manual `POST /analysis/run` dengan state: *idle*, *running/evaluating* (animasi loader tanpa memblokir pembacaan wawasan aktif), dan status *cooldown* 10 menit dengan penghitung waktu mundur transparan.
   - Filter bar sekunder: filter jenis klaim, tingkat prioritas, dan status penyelesaian rekomendasi.

4. **Anatomi Kartu Wawasan (*Insight Card Anatomy*)**:
   - Header terstruktur: Kapsul kategori, badge klaim, badge prioritas, waktu kalkulasi, dan badge pelanggan inti (Top 20%).
   - Body: Judul lugas, ringkasan penjelasan template deterministik, serta grid metrik kunci (*Evidence Snapshot Mini-Grid*: nilai terkini vs ambang batas).
   - Pratinjau Dampak (*Impact Preview Box*): Perkiraan pemulihan omzet atau pencegahan kerugian yang disajikan secara realistis tanpa janji palsu.
   - Action Bar: Tombol aksi utama (e.g. "Hubungi via WhatsApp", "Sesuaikan Harga Jual"), tombol sekunder "Lihat Bukti & Detail", serta tombol "Abaikan".
   - State Khusus *Superseded*: Penandaan otomatis kartu yang kondisinya sudah terselesaikan oleh transaksi baru.

5. **Modal Detail Transparansi Data & Bukti Riwayat (*Data Provenance Modal*)**:
   - Menampilkan Rule ID & Versi (e.g. `CUST_OVERDUE_V1`), jenis klaim, periode observasi, dan status kualitas data.
   - Transparansi metodologi: Seksi asumsi (*assumptions*) dan batasan analisis (*limitations*).
   - Tabel Bukti Riwayat Sumber Data (*Insight Sources Table*): Pemetaan langsung ke entitas data riil (`transactions`, `customers`, `products`, `complaints`) dengan tautan klik langsung ke detail transaksi untuk verifikasi mandiri pengguna.

6. **Siklus Hidup Rekomendasi & Dialog Pengabaian (*Dismiss Modal*)**:
   - Alur state machine rekomendasi: `new` → `reviewed` → `in_progress` → `completed` / `dismissed` / `superseded`.
   - Dialog pengabaian berpenjelasan: Pilihan alasan dismiss tersimpan ke `dismissed_reason` dan penerapan jeda kemunculan kembali (*cooldown*) 14 hari untuk `dedupe_key` yang sama.

7. **Status Khusus Data Gate & Edukasi Pengguna**:
   - Empty state informatif ketika data transaksi belum mencukupi batas minimal data gate (misal: belum ada pelanggan dengan ≥ 4 tanggal pembelian dalam 180 hari) dengan rekomendasi aksi pencatatan di POS.
   - State konfirmasi ketika seluruh kondisi bisnis sehat (zero issues) dengan lencana perisai centang hijau zamrud.

8. **Tata Letak Responsif & Spesifikasi Gerak**:
   - Penyesuaian tata letak untuk Mobile 390px (stack & bottom-sheet drawer), Tablet 768px, dan Desktop 1440px (asimetris: 68% feed utama + 32% sticky panel ringkasan aksi prioritas).
   - Motion tokens terinci dengan format `trigger → property → duration → easing` (durasi 180ms–240ms) serta kepatuhan penuh terhadap `prefers-reduced-motion` dan navigasi keyboard / ARIA accessibility.
