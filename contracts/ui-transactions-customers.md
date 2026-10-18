# UI Specification: Customers, Transactions (POS) & Sales Returns

## Metadata
- **Task ID**: T-030
- **Version**: 1.0.0
- **Status**: Ratified
- **Design System**: Clean Pragmatic Craftsman (`contracts/design-tokens.md`)
- **Primary Color**: Emerald Forest (`#0D6E64`)

---

## 1. Overview & User Flows

Modul ini memfasilitasi operasional penjualan sehari-hari bagi pemilik usaha dan kasir:
1. **Manajemen Pelanggan (`/customers`)**: Registrasi pelanggan, pencarian cepat kontak, dan histori loyalitas transaksi.
2. **Kasir / Entri Transaksi Baru (`/transactions/new` atau POS Modal)**: Antarmuka kasir cepat dengan pencarian produk berbasis autocomplete, seleksi pelanggan walk-in / terdaftar, pengaturan diskon per item, kalkulasi otomatis laba kotor seketika, dan dukungan metode pembayaran beragam.
3. **Riwayat & Detail Transaksi (`/transactions`)**: Audit penjualan, status pembayaran, filter rentang tanggal, cetak/lihat invoice struk.
4. **Pembatalan & Retur Barang (`Sales Returns`)**: Mekanisme pembatalan penuh (dengan pemulihan stok otomatis) atau retur sebagian per item barang.

---

## 2. Struktur Layar & Komponen UI

### 2.1 Halaman Pelanggan (`/customers`)
- **Header**:
  - Judul: "Daftar Pelanggan"
  - Deskripsi: "Kelola data kontak pelanggan, riwayat belanja, dan preferensi transaksi."
  - Tombol Aksi: `+ Tambah Pelanggan` (Primary button).
- **Filter Bar**:
  - Input pencarian: Search nama pelanggan, nomor WhatsApp/telepon, atau email.
  - Quick badge stats: Total Pelanggan Aktif.
- **Tabel Pelanggan**:
  - Kolom:
    1. **Nama Pelanggan** (Nama lengkap + badge "Pelanggan Baru" / "Loyal").
    2. **Kontak** (Nomor telepon/WhatsApp + email).
    3. **Total Transaksi** (Jumlah frekuensi transaksi).
    4. **Total Belanja** (Akumulasi nominal rupiah `id-ID`).
    5. **Transaksi Terakhir** (Tanggal transaksi terakhir).
    6. **Aksi** (Lihat Detail, Edit).
- **Drawer / Modal Tambah & Edit Pelanggan**:
  - Field:
    - Nama Lengkap (Required)
    - Nomor WhatsApp/Telepon (Required, format normalisasi e.g. `08...` atau `+62...`)
    - Email (Opsional)
    - Catatan / Preferensi Khusus (Opsional, textarea)

---

### 2.2 Entri Transaksi Baru / Layar POS (`/transactions/new`)
Layout split-screen responsif (Kiri: Pemilihan Produk & Cart; Kanan: Ringkasan Pembayaran & Checkout).

- **Panel Kiri — Keranjang Belanja & Pemilihan Produk**:
  - **Pilih Pelanggan**:
    - Input autocomplete pencarian pelanggan.
    - Opsi default cepat: "Pelanggan Umum (Walk-in / Tanpa Nama)".
    - Shortcut tombol mini: `+ Pelanggan Baru` (buka modal kilat jika belum terdaftar).
  - **Pencarian Produk**:
    - Autocomplete cepat: Cari berdasarkan SKU atau Nama Produk.
    - Kartu item produk cepat: Nama produk, SKU, stok tersedia saat ini (dengan penanda visual jika stok menipis), harga jual.
  - **Tabel Keranjang Belanja (Cart Items)**:
    - Kolom:
      - Item & SKU
      - Harga Satuan (Bisa di-override jika ada diskon khusus, default dari `current_selling_price`)
      - Kuantitas (Input angka dengan tombol stepper `+` dan `-`, validasi tidak melebihi stok tersedia)
      - Diskon Item (Nominal potongan rupiah)
      - Subtotal Item ($Subtotal = (Harga \times Qty) - Diskon$)
      - Tombol Hapus item (Icon trash)
    - Empty State: Ilustrasi keranjang kosong dengan instruksi "Pilih produk di atas untuk menambahkan ke transaksi."

- **Panel Kanan — Ringkasan Pembayaran & Eksekusi**:
  - **Kartu Ringkasan Finansial**:
    - Subtotal Belanja
    - Diskon Keseluruhan
    - Biaya Admin / Layanan (Opsional)
    - **Total Pembayaran** (Font mono tebal, ukuran `text-xl`, aksen Emerald Forest).
    - **Estimasi Laba Kotor Seketika**:
      - Ditampilkan kepada kasir/pemilik: "Estimasi Laba Kotor: Rp XX.XXX (XX.X%)".
      - Dihitung secara real-time dari selisih Harga Jual vs Snapshot HPP Terkini.
  - **Form Pembayaran**:
    - Metode Pembayaran: Pilihan toggle button (`Tunai`, `QRIS`, `Transfer Bank`).
    - Nominal Diterima (Khusus Tunai): Input rupiah + tombol pecahan cepat (Uang Pas, 50.000, 100.000, 200.000).
    - Nominal Kembalian: Otomatis dihitung ($Kembalian = Nominal Diterima - Total$).
    - Catatan Transaksi: Input teks opsional.
  - **Tombol Eksekusi**:
    - Tombol Primary Lebar Penuh: `Selesaikan Transaksi (Rp XXX.XXX)`.
    - Loading indicator & proteksi tombol disable untuk mencegah *double billing*.

---

### 2.3 Riwayat Transaksi (`/transactions`)
- **Filter Bar**:
  - Pemilih Rentang Tanggal (Hari ini, 7 hari terakhir, 30 hari terakhir, Kustom).
  - Filter Status Transaksi: `Semua`, `Valid`, `Dibatalkan`.
  - Filter Metode Pembayaran: `Semua`, `Tunai`, `QRIS`, `Transfer`.
  - Filter Pelanggan.
- **Tabel Transaksi**:
  - Kolom:
    1. **No. Invoice / Ref** (e.g. `TRX-20261018-001`, font mono tebal).
    2. **Tanggal & Waktu**.
    3. **Pelanggan** (Nama atau `Pelanggan Umum`).
    4. **Metode & Status Bayar** (Badge `Tunai` / `QRIS` / `Transfer`, `Lunas`).
    5. **Total Penjualan** (Nominal rupiah).
    6. **Total HPP & Margin** (HPP: Rp XX.XXX, Margin: XX%).
    7. **Status** (Badge hijau `Valid` atau merah `Dibatalkan`).
    8. **Aksi** (Lihat Detail / Invoice, Batalkan / Retur).

---

### 2.4 Modal Detail Transaksi & Retur / Pembatalan

#### A. Modal Rincian Invoice
- Ringkasan header invoice, identitas toko, nama pelanggan, waktu transaksi, kasir pembuat.
- Tabel rincian produk yang dibeli: Qty, Harga Satuan, Diskon, Subtotal, serta snapshot HPP unit.
- Total penjualan, total HPP, dan laba kotor yang terealisasi.
- Riwayat retur (jika ada barang yang telah diretur sebelumnya).

#### B. Modal Pembatalan Transaksi (Void / Cancel)
- Banner Peringatan: "Pembatalan transaksi bersifat permanen. Seluruh stok produk (X item) akan dikembalikan secara otomatis ke inventaris toko."
- Field: Alasan Pembatalan (Required, e.g. "Salah input kasir", "Pelanggan membatalkan pesanan").
- Konfirmasi: Tombol `Batalkan Transaksi` (Warna Danger).

#### C. Modal Retur Penjualan (Sales Return)
- Tabel pemilihan item yang ingin diretur:
  - Checkbox pilih item.
  - Kuantitas yang dibeli vs Kuantitas yang sudah pernah diretur sebelumnya.
  - Input kuantitas retur baru (validasi: $\le Qty\_Beli - Qty\_Retur\_Lama$).
  - Nominal pengembalian dana (*refund amount*).
- Field: Alasan Retur (e.g. "Kemasan rusak", "Produk kedaluwarsa / cacat", "Tukar varian").
- Banner info: "Stok produk yang diretur akan otomatis ditambahkan kembali ke inventaris."
- Tombol aksi: `Proses Retur & Kembalikan Dana`.

---

## 3. Ketentuan Desain & Aksesibilitas
1. **No Resting Shadows**: Menggunakan border `border-subtle` (`#E5E5E5`) dan background `bg-surface` (`#FFFFFF`).
2. **Warna Status**:
   - Status Valid: Hijau Emerald / Forest (`#0D6E64`).
   - Status Dibatalkan / Void: Merah Terracotta / Danger (`#DC2626`).
   - Status Pending / Partial: Amber (`#D97706`).
3. **Format Angka**: Seluruh nilai rupiah diformat via `formatRupiah()` (`id-ID`).
4. **Presisi Stok**: Kuantitas menampilkan unit yang tepat (e.g. `2 Pcs`, `1.500 Kg`).
