# Task T-040: UI Specification for Financial Dashboard, Metrics & Data Health Indicators

- **Owner**: `designer`
- **Milestone**: M4 — Finance Engine, Dashboard & Data Health
- **Size**: Standard
- **Repo**: `office-hub`

## 1. Objective
Menyusun spesifikasi UI komprehensif untuk Halaman Dashboard Finansial Utama (`/`) dan Diagnostik Kesehatan Data (*Data Health*), termasuk komponen unggulan (*signature moment*) "Kartu Kepastian Laba" (*Financial Clarity Card*), kartu ringkasan omzet, laba kotor, margin, tren penjualan, dan peringatan cakupan data biaya.

## 2. Context & Reference Files
- `contracts/design-direction.md`: Prinsip Clean Pragmatic Craftsman, Signature Moment "Kartu Kepastian Laba", larangan kata "laba" tanpa keterangan kotor ("laba kotor"), status cakupan biaya (100% Tercakup vs Estimasi XX% Tercakup).
- `contracts/design-tokens.md`: Desain sistem token (Emerald Forest `#0D6E64`, neutral palette, no resting shadows).
- `project-map/trd.md` (§6 Calculation Spec): Aturan laba kotor, cakupan HPP (`cost_coverage`), estimasi vs pasti.

## 3. Expected Output
Dokumen `contracts/ui-finance-dashboard.md` yang memuat:
1. **Layout & Komponen Dashboard Finansial**:
   - Filter Periode: Hari Ini, 7 Hari Terakhir, 30 Hari Terakhir, Bulan Ini, Kustom (From - To).
   - Kartu Signature: "Kartu Kepastian Laba" (Menampilkan Laba Kotor, Status `100% Tercakup` hijau atau `Estimasi (XX% Tercakup)` amber dengan tooltip/badge rincian produk tanpa HPP).
   - Metrik Utama: Total Penjualan Bersih (*Net Sales*), Total HPP (*COGS*), Margin Laba Kotor (%), Total Kas Diterima (*Cash Received*).
   - Grafik Tren Harian: Bar chart / Area chart pergerakan penjualan dan laba kotor.
   - Diagnostik Kesehatan Data (*Data Health Card*): Skor kelengkapan data usaha (cakupan HPP produk terjual, persentase transaksi terhubung pelanggan, status inventaris stok rendah).
   - Pintasan Cepat: Tombol aksi langsung ke "Kasir POS", "Tambah Produk", "Input HPP".
2. **Kepatuhan Naming Rule**: Wajib menggunakan frasa "Laba Kotor" (tidak pernah hanya "Laba"). Jika ada item tanpa HPP, wajib membubuhkan label "Estimasi" dan jumlah item yang belum memiliki HPP.
3. Responsivitas: Breakpoint mobile (390px), tablet, dan desktop (1280px+).

## 4. Definition of Done
Dokumen `contracts/ui-finance-dashboard.md` diterbitkan dan dilaporkan di `inbox/leader/T-040-report.md`.
