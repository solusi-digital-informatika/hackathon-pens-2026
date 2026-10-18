# Laporan Penyelesaian T-040: UI Specification for Financial Dashboard & Data Health

- **Task ID**: T-040
- **Assignee**: `designer`
- **Deliverable**: `contracts/ui-finance-dashboard.md`
- **Status**: DONE / APPROVED

Telah selesai disusun spesifikasi UI untuk Dashboard Finansial Utama dan Diagnostik Kesehatan Data (*Data Health*), mencakup:
1. Prinsip desain Clean Pragmatic Craftsman dan aksen *Emerald Forest* (`#0D6E64`).
2. Komponen unggulan (*Signature Moment*): **"Kartu Kepastian Laba"** (*Financial Clarity Card*) dengan indikator status kepastian HPP (`100% HPP Tercakup` hijau vs `Estimasi (XX% HPP Tercakup)` amber jika ada produk terjual tanpa HPP).
3. Grid metrik finansial: Penjualan Bersih (*Net Sales*), Total HPP (*COGS*), Kas Diterima (*Cash Received*), dan Rata-rata Nilai Belanja (*Average Ticket Size*).
4. Grafik tren pergerakan penjualan dan laba kotor harian.
5. Diagnostik Kesehatan Data (*Data Health*) mencakup skor kesehatan pembukuan (0-100%), metrik cakupan HPP, rasio transaksi pelanggan, dan peringatan stok menipis.
6. Kepatuhan mutlak terhadap aturan penamaan: selalu menggunakan frasa "Laba Kotor" dan menandai "Estimasi" bila data biaya belum lengkap.
