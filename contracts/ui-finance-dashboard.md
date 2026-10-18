# Spesifikasi UI & Alur Interaksi: Dashboard Finansial & Kesehatan Data (M4)

- **ID Kontrak**: UI-M4-001
- **Task ID**: T-040
- **Penulis**: Designer Profile / Product Lead
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED

---

## 1. Prinsip Desain & Hirarki Visual
Sesuai `contracts/design-direction.md`, modul Dashboard Finansial dirancang menggunakan prinsip **Clean Pragmatic Craftsman**:
- **Aksen Tunggal**: *Emerald Forest* (`#0D6E64` / `var(--color-primary-700)`).
- **Tipografi**: Inter, ukuran `18px` bold untuk kartu metrik, `24px` untuk judul halaman.
- **Tanpa Bayangan Diam**: Kartu data datar (*flat cards*) berbatas `1px solid var(--color-border-default)`.
- **Naming Rule Ketat**: Wajib menggunakan sebutan **"Laba Kotor"**, dilarang menyebut hanya "Laba". Jika ada item tanpa HPP, wajib membubuhkan label **"Estimasi"** disertai jumlah item yang belum memiliki HPP.
- **Signature Moment**: **"Kartu Kepastian Laba"** (*Financial Clarity Card*) menjadi komponen sentral di puncak halaman.

---

## 2. Struktur Komponen Dashboard (`/`)

### 2.1 Header & Filter Periode
- **Judul**: "Ringkasan Finansial Usaha"
- **Subjudul**: "Pantau penjualan bersih, laba kotor pasti, dan kesehatan pembukuan usaha Anda."
- **Tombol Aksi Cepat**:
  - Tombol Primer: `+ Buka Kasir POS` (menavigasi ke `/pos`).
  - Tombol Sekunder: `+ Produk Baru` (menavigasi ke `/products`).
- **Selector Rentang Waktu (Tabs/Pills)**:
  - `Hari Ini` (`today`)
  - `7 Hari Terakhir` (`7d` - default)
  - `30 Hari Terakhir` (`30d`)
  - `Bulan Ini` (`month`)

---

### 2.2 Signature Component: "Kartu Kepastian Laba" (*Financial Clarity Card*)
Kartu berlatar permukaan putih/netral berbingkai khusus dengan aksen *Emerald Forest*:
- **Badge Kepastian Status**:
  - Jika `cost_coverage === 100%`:
    - Badge Hijau (*Success*): `100% HPP Tercakup`
    - Keterangan teks halus: "Perhitungan laba kotor terverifikasi dari seluruh HPP transaksi."
  - Jika `cost_coverage < 100%`:
    - Badge Oranye/Amber (*Warning*): `Estimasi (XX% HPP Tercakup)`
    - Peringatan: "Terdapat N item produk terjual tanpa HPP tercatat. Nilai laba kotor dihitung dari transaksi yang memiliki data biaya."
    - Tombol mini inline: `Lengkapi HPP Produk →` (membuka filter produk tanpa HPP).
- **Angka Laba Kotor Utama**:
  - Teks: `Laba Kotor (Periode Terpilih)`
  - Nilai: format Rupiah besar (misal: `Rp 4.850.000`)
  - Margin Persentase: `Margin 34.8%` dalam kapsul badge kontras.

---

### 2.3 Grid Kartu Metrik Finansial
Grid 4 kolom responsif (1 kolom pada mobile, 2 kolom pada tablet, 4 kolom pada desktop):
1. **Penjualan Bersih (*Net Sales*)**:
   - Total nominal setelah dikurangi diskon & retur.
   - Subteks: "Jumlah Transaksi: N penjualan".
2. **Total HPP (*COGS*)**:
   - Total modal pokok produk terjual.
   - Subteks: "Berdasarkan snapshot HPP saat checkout".
3. **Kas Diterima (*Cash Received*)**:
   - Total pembayaran tunai, transfer, dan QRIS yang sudah lunas.
   - Subteks: "Total uang riil masuk".
4. **Rata-rata Nilai Belanja (*Average Ticket Size*)**:
   - Rata-rata nominal per transaksi invoice.
   - Subteks: "Net Sales / Jumlah Transaksi".

---

### 2.4 Grafik Tren Penjualan & Laba Harian
- **Visualisasi**: Bar chart atau multi-line chart membandingkan *Penjualan Bersih* (batang abu-abu kebiruan) vs *Laba Kotor* (batang hijau zamrud `#0D6E64`).
- **Sumbu X**: Tanggal (format `DD MMM`).
- **Sumbu Y**: Nominal Rupiah (format ringkas `1jt`, `2jt`).
- **Interaksi**: Tooltip saat hover menampilkan Tanggal, Penjualan Bersih, Laba Kotor, dan Margin %.

---

### 2.5 Kartu Diagnostik Kesehatan Data (*Data Health Card*)
Panel diagnostik kelengkapan data pembukuan UMKM untuk mendorong disiplin pencatatan:
- **Skor Kesehatan Data Usaha**: Indikator persentase 0-100% dengan status:
  - `Sehat` (≥ 85%): Warna hijau emerald.
  - `Perlu Perhatian` (60-84%): Warna amber.
  - `Kritis` (< 60%): Warna merah bata.
- **Rincian Parameter Diagnostik**:
  - Cakupan HPP Produk: `XX%` produk aktif telah memiliki data HPP.
  - Keterhubungan Pelanggan: `XX%` transaksi terhubung ke nama pelanggan.
  - Ketersediaan Stok: `N` produk memiliki stok menipis di bawah stok minimum.
- **Daftar Rekomendasi Aksi Perbaikan**:
  - Item dapat diklik langsung mengarahkan pengguna ke halaman terkait (misal: "Input HPP untuk 3 produk", "Restock 2 produk menipis").

---

## 3. Responsive Breakpoints
- **Mobile (390px)**:
  - Header menumpuk secara vertikal.
  - Pills rentang waktu dapat digeser horizontal (*scrollable*).
  - Kartu metrik ditata 1 kolom.
  - Grafik tren menyesuaikan lebar layar dengan label tanggal interval 2 hari.
- **Desktop (1024px+)**:
  - Grid 4 kolom untuk metrik utama.
  - Layout 2 panel: Grafik Tren di sisi kiri (65%), Diagnostik Kesehatan Data di sisi kanan (35%).
