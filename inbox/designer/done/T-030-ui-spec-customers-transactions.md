# Task: T-030 UI Spec for Customers, Transactions POS/Entry & Returns/Cancellation

## Objective
Rancang spesifikasi antarmuka pengguna (UI/UX Specification) yang komprehensif untuk modul Pelanggan (Customers), Kasir / Entri Transaksi Penjualan (Transactions POS/Order Entry), dan Pembatalan/Retur Penjualan (Sales Returns).

## Konteks & Pedoman
- `contracts/design-direction.md`: Clean Pragmatic Craftsman, aksen tunggal *Emerald Forest* (`#0D6E64`), tanpa bayangan kartu istirahat (*no resting shadows*), WCAG AA, bahasa Indonesia, format Rupiah `id-ID`.
- `contracts/erd.md`: `customers`, `customer_identifiers`, `transactions`, `transaction_items`, `sales_returns`.
- Target pengguna: Pemilik UMKM dan staf operasional kasir (desain harus cepat, minim friksi, mendukung barcode/SKU scanner & pencarian cepat).

## Lingkup Spesifikasi UI
1. **Daftar & Detail Pelanggan (`/customers`)**:
   - Pencarian nama/telepon/email pelanggan.
   - Drawer/Modal tambah & edit pelanggan (nama, kontak telepon/WhatsApp, email, catatan).
   - Profil ringkasan pelanggan (total transaksi, tanggal transaksi terakhir).
2. **Entri Transaksi Penjualan / POS (`/transactions/new` atau Drawer POS)**:
   - Pemilihan pelanggan (atau opsi Pelanggan Umum / Walk-in tanpa registrasi).
   - Pencarian dan pemilihan produk (dengan info stok aktual dan HPP untuk estimasi margin seketika).
   - Penyesuaian kuantitas, harga jual, dan diskon per item.
   - Ringkasan pembayaran: subtotal, diskon, biaya admin/pajak (opsional), total akhir, input nominal dibayar, metode pembayaran (`Tunai`, `QRIS`, `Transfer Bank`).
   - Tombol eksekusi transaksi yang jelas dengan proteksi double-submit (loading / disabled state).
3. **Riwayat & Detail Transaksi (`/transactions`)**:
   - Filter tanggal transaksi, status (`Valid`, `Dibatalkan`), status bayar (`Lunas`, `Belum Lunas`).
   - Tampilan rincian invoice/struk: daftar item produk, kuantitas, harga, total HPP vs Total Penjualan, laba kotor transaksi.
4. **Alur Pembatalan Transaksi & Retur Barang (`Sales Returns`)**:
   - Modal konfirmasi pembatalan seluruh transaksi (dengan input alasan pembatalan dan info otomatis pemulihan stok).
   - Modal retur sebagian item: pilih item transaksi, kuantitas yang diretur (maksimal sejumlah kuantitas beli), nominal pengembalian dana (*refund*), dan alasan retur.

## Output yang Diharapkan
Tulis dokumen spesifikasi UI lengkap ke `contracts/ui-transactions-customers.md` dan buat laporan penyelesaian di `inbox/leader/T-030-report.md`.
