# UI/UX Specification — Products, Cost Versions & Pricing Calculator

> Dokumen spesifikasi antarmuka dan interaksi pengguna untuk Katalog Produk, Formulir Tambah/Ubah Produk, Histori Versi HPP & Harga Jual, serta Kalkulator Penetapan Harga Interaktif.
> Mengacu pada: `contracts/design-direction.md`, `contracts/design-tokens.md`, `project-map/prd.md` (§5.2, §5.3, §6.2), `project-map/trd.md` (§5 TD-08, TD-12), dan `contracts/erd.md` (Entitas `products`, `product_cost_versions`, `product_cost_components`, `product_price_history`).

---

## 1. Feature → Page Map

| Fitur | Rute / Permukaan | Akses / Guard | Tujuan Utama |
| :--- | :--- | :--- | :--- |
| **Katalog Produk** | `/products` | Terautentikasi (Owner / Staff) | Melihat daftar seluruh produk usaha, status HPP, margin kotor, filter kategori/status, dan pencarian cepat. |
| **Tambah Produk** | Modal / Drawer pada `/products` | Terautentikasi (Owner / Staff) | Mendaftarkan SKU, nama, kategori, satuan, harga jual terkini, serta inisialisasi versi HPP pertama. |
| **Ubah Produk** | Modal / Drawer pada `/products` | Terautentikasi (Owner / Staff) | Memperbarui informasi umum produk (nama, SKU, kategori, satuan, status) atau memperbarui harga jual terkini. |
| **Histori HPP & Harga** | Modal / Slide Drawer (`/products?drawer=history&id={id}`) | Terautentikasi (Owner / Staff) | Menampilkan audit trail berurutan waktu atas versi biaya (*cost versioning* append-only) dan riwayat perubahan harga jual. |
| **Kalkulator Harga** | Modal Dialog / Widget Tersemat (`/products?drawer=calculator`) | Terautentikasi (Owner / Staff) | Simulasi penetapan harga jual berbasis target margin vs markup dengan peringatan visual jika margin < 15% atau jual < HPP. |

---

## 2. Global Anti-Slop & UX Rules

1. **Resting Surfaces & Borders**: Dilarang menggunakan box-shadow pada kartu atau baris tabel dalam keadaan diam (*resting cards*). Pemisahan visual mengandalkan kontras permukaan `var(--color-bg-surface)` terhadap kanvas `var(--color-bg-app)` (`#F7F9F8`) serta garis batas halus `1px solid var(--color-border-subtle)`. Shadow hanya diterapkan pada lapisan terangkat: popover dropdown (`var(--shadow-dropdown)`) dan dialog modal/drawer (`var(--shadow-modal)`).
2. **Tanpa Gradien Dekoratif**: Tidak ada gradien warna-warni, efek ambient glow, maupun glassmorphism. Aksen tunggal adalah **Emerald Forest** (`var(--color-primary-700)` = `#0F766E`), dengan netral slate-sage bertinting emerald.
3. **Format Angka & Mata Uang Monospace**: Seluruh nominal Rupiah dan angka persentase wajib menggunakan font tabular `var(--font-family-tabular)` (`Inter`) dengan class `.tnum` (`font-variant-numeric: tabular-nums`) dan diformat via `formatRupiah` (`id-ID`, misal: `Rp 25.000`).
4. **Kaidah Terminologi Finansial Deterministik**:
   - Selalu gunakan istilah **"Laba Kotor"** (dilarang hanya menuliskan "Laba" atau mengklaimnya sebagai "Laba Bersih").
   - HPP disajikan sebagai **"HPP per Unit"** dengan rincian biaya yang transparan.
   - Jika produk belum memiliki versi HPP, status ditampilkan sebagai badge **"Tanpa HPP"** (`var(--color-warning-bg)` + `var(--color-warning-text)`), margin kotor diberi tanda `-` (bukan 0% spekulatif).
5. **Aksesibilitas & Fitts**:
   - Touch target seluruh kontrol, tombol baris, dan aksi minimal 44×44px di mobile (390px).
   - Kontras warna teks memenuhi WCAG AA: teks utama `#131F1C` (15.2:1), teks sekunder `#4B5E59` (5.8:1), tombol primer terhadap putih (5.6:1).
   - Label formulir wajib selalu terlihat (*visible label*), dilarang menggunakan placeholder sebagai pengganti label.
6. **Hukum Hick & Doherty**:
   - Satu aksi primer per tampilan: tombol `+ Tambah Produk` di kanan atas header halaman.
   - Respon penekanan tombol (*press feedback*) ≤ 100ms (`transform: scale(0.98)`).
   - Tampilan data memuat state skeleton saat proses fetch > 400ms.

---

## 3. Komponen Antarmuka Bersama

### 3.1. PageHeader
- **Struktur**: 
  - Kiri: Judul Halaman `var(--text-xl-size)` (Plus Jakarta Sans Bold), Subtitle `var(--text-sm-size)` (Inter Regular, warna `var(--color-text-muted)`).
  - Kanan: Kelompok aksi horizontal dengan jarak `var(--space-3)`. Aksi sekunder: tombol `"Kalkulator Harga"` (ikon kalkulator + teks, secondary outline). Aksi primer: tombol `"+ Tambah Produk"` (`var(--color-primary-700)`).
- **Dimensi**: Padding vertikal `var(--space-6)` (desktop) / `var(--space-4)` (mobile).

### 3.2. FilterToolbar
- **Struktur**: Bar horizontal di atas tabel berisi elemen pencarian dan filter status/kategori.
  1. *Input Cari*: Placeholder `"Cari nama produk atau SKU..."`, ikon kaca pembesar di sisi kiri, clear button di sisi kanan saat terisi. Lebar fleksibel (min 280px di desktop).
  2. *Dropdown Kategori*: Opsi filter kategori produk (`Semua Kategori`, `Makanan`, `Minuman`, dll.), tinggi 44px.
  3. *Dropdown Status*: Opsi `Semua Status`, `Aktif`, `Nonaktif`.
  4. *Dropdown Status HPP*: Opsi `Semua HPP`, `Memiliki HPP`, `Tanpa HPP`.
- **Responsive**: Di desktop/tablet sejajar satu baris; di mobile (390px) bertumpuk vertikal dengan input cari di atas dan deretan chip horizontal di bawahnya.

### 3.3. ProductTable (Desktop & Tablet View)
- **Struktur Kolom**:
  1. **Produk & SKU**: Nama Produk (`var(--font-weight-semibold)`, `var(--text-sm-size)`) dan SKU di bawahnya (`var(--text-xs-size)`, `var(--color-text-muted)`).
  2. **Kategori**: Badge kategori netral (`var(--color-bg-subtle)`, border `var(--color-border-subtle)`).
  3. **Satuan**: Teks ringkas (misal: `pcs`, `botol`, `kg`).
  4. **Harga Jual**: Angka nominal `formatRupiah`, rata kanan, font tabular.
  5. **HPP per Unit**: Angka nominal `formatRupiah` atau badge peringatan `"Belum diatur"` jika NULL. Rata kanan.
  6. **Margin Kotor**: Persentase `((Harga - HPP) / Harga * 100%)`. Rata kanan. Jika Margin < 15%: badge kuning/amber `"X.X% (Tipis)"`. Jika Margin ≥ 30%: badge hijau muda. Jika Tanpa HPP: `-`.
  7. **Status**: Badge `Aktif` (hijau) atau `Nonaktif` (abu-abu).
  8. **Aksi**: Tombol ikon menu titik tiga (`...`) atau tombol aksi cepat (`Histori`, `Edit`) dengan touch target min 44×44px.
- **Gaya**: Header `var(--color-bg-subtle)`, teks header huruf kapital kecil `var(--text-xs-size)` tebal medium. Baris berganti warna hover `var(--color-bg-hover)` (durasi 120ms). Border baris `1px solid var(--color-border-subtle)`.

### 3.4. ProductCard (Mobile 390px View)
- **Peran**: Pengganti tabel pada layar smartphone untuk menghindari scrolling horizontal yang membingungkan.
- **Struktur**:
  - Baris Atas: Nama Produk (tebal 14px), badge status (`Aktif`/`Nonaktif`), dan tombol menu aksi (`...`).
  - Baris Tengah: SKU dan Kategori (teks 12px muted).
  - Baris Bawah: Grid 2 kolom finansial:
    - Kiri: Harga Jual (`Rp 25.000`) & HPP per Unit (`Rp 15.000` atau `"Tanpa HPP"`).
    - Kanan: Laba Kotor per unit & Persentase Margin Kotor dengan indikator visual.
- **Gaya**: Latar `var(--color-bg-surface)`, border `1px solid var(--color-border-subtle)`, radius `var(--radius-lg)`, padding `var(--space-4)`, margin bawah `var(--space-3)`. Tanpa box shadow (*strict anti-slop*).

### 3.5. StatusBadge
- `success` (Aktif / Margin Sehat ≥ 30%): Background `var(--color-success-bg)` (`#F0FDF4`), border `1px solid var(--color-success-border)`, teks `var(--color-success-text)` (`#166534`).
- `warning` (Tanpa HPP / Margin Tipis < 15%): Background `var(--color-warning-bg)` (`#FFFBEB`), border `1px solid var(--color-warning-border)`, teks `var(--color-warning-text)` (`#854D0E`).
- `error` (Harga Jual < HPP): Background `var(--color-error-bg)` (`#FEF2F2`), border `1px solid var(--color-error-border)`, teks `var(--color-error-text)` (`#991B1B`).
- `muted` (Nonaktif): Background `var(--color-bg-subtle)`, border `1px solid var(--color-border-subtle)`, teks `var(--color-text-muted)`.

### 3.6. ModalContainer & DrawerSheet
- **Desktop (1440px / 768px)**: Dialog modal melayang di tengah layar (*centered modal*) dengan backdrop gelap transparan `rgba(19, 31, 28, 0.4)`, lebar `560px` (Add/Edit) atau `680px` (Timeline & Kalkulator), radius `var(--radius-xl)`, bayangan `var(--shadow-modal)`.
- **Mobile (390px)**: Bottom Sheet / Slide-over Drawer penuh dari bawah, sudut atas melengkung `var(--radius-xl)`, lebar 100%, tinggi maksimal `90vh` dengan scroll area vertikal mandiri. Area pegangan tarikan (*drag handle*) di bagian atas sheet (tinggi 4px, lebar 36px, radius pill, warna `var(--color-border-strong)`).

---

## 4. Spesifikasi Halaman & Alur Pengguna

### 4.1. Halaman Utama: Katalog Produk (`/products`)

#### Metadata & Rangkuman Halaman (15 Baris Ringkas)
- **Tujuan Halaman**: Katalog terpusat untuk memantau harga jual, struktur biaya (HPP), kelengkapan data margin, dan ketersediaan produk.
- **Pengguna & Tugas Utama**: Pemilik UMKM dan staf operasional; mencari produk, mengevaluasi margin kotor, mengelola status jual, dan membuka kalkulator harga.
- **Aksi Primer**: Tombol `"+ Tambah Produk"` di header utama.
- **Aksi Sekunder**: Tombol `"Kalkulator Harga"` di header, baris filter kategori, dan tombol `"Histori"` pada setiap produk.
- **Entri**: Navigasi sidebar/navbar item `"Produk"` (`/products`).
- **Keluar**: Buka modal Tambah/Ubah Produk, buka drawer Histori/Kalkulator, atau navigasi ke Transaksi/Dashboard.
- **Data Ditampilkan**: Daftar produk (nama, SKU, kategori, unit, harga jual, HPP terakhir, margin kotor, status), total produk, jumlah produk tanpa HPP.
- **Layout Desktop (1440px)**: Sidebar kiri (240px) + Header + Metrik Ringkas 3 kolom + FilterToolbar + ProductTable (8 kolom) + Pagination bar.
- **Layout Tablet (768px)**: Header + Metrik Ringkas 3 kolom + FilterToolbar (wrap 2 baris) + ProductTable (sembunyikan kolom satuan & tanggal) + Pagination.
- **Layout Mobile (390px)**: Header kompak + Banner ringkasan HPP + Search bar + Chip filter + Daftar ProductCard vertikal + Tombol sticky FAB `+ Produk`.
- **Keadaan (States)**: Loading (skeleton table 5 baris), Empty (tanpa produk), Filtered Empty (hasil pencarian nihil), Populated, Error (gagal memuat data).
- **Motion**: Skeleton fade pulse (1.5s), row hover color (120ms), dropdown open (200ms cubic-bezier), tab page enter (250ms translateY 6px).
- **Aksesibilitas**: Table berlabel `aria-label="Katalog Produk"`, header kolom bertipe `th scope="col"`, focus ring 2px pada seluruh aksi interaktif, keyboard navigasi `Tab`/`Enter`.

#### Ringkasan Metrik Produk (Product Health Strip)
Sebelum tabel utama, terdapat 3 kartu ringkas (*resting cards*, tanpa bayangan, border `var(--color-border-subtle)`):
1. **Total Produk Aktif**: Angka besar `var(--text-2xl-size)` font tabular, label `"Produk siap dijual"`.
2. **Cakupan HPP (Cost Coverage)**: Persentase produk yang telah memiliki versi biaya. Jika < 100%, teks pendukung: `"X produk belum memiliki HPP — laba kotor dihitung estimasi"`.
3. **Rata-rata Margin Kotor**: Rata-rata tertimbang margin produk aktif dengan indikator warna status sehat.

#### Keadaan Khusus (Edge Cases & States)
1. **Empty State (Belum ada produk)**:
   - Ilustrasi fungsional: Kotak garis putus-putus (*dashed border*) dengan ikon paket box `32×32px` warna `var(--color-text-muted)`.
   - Judul: `"Belum Ada Produk Terdaftar"`
   - Deskripsi: `"Tambahkan produk pertama Anda beserta harga jual dan HPP untuk mulai memantau laba kotor secara presisi."`
   - Tombol Aksi: `"+ Tambah Produk Pertama"` (fokus otomatis ke tombol ini).
2. **Filtered Empty State (Pencarian tidak ditemukan)**:
   - Teks: `"Tidak ada produk yang cocok dengan pencarian '{query}'"`.
   - Tombol Aksi: `"Reset Filter"` untuk mengembalikan tabel ke daftar penuh.
3. **Error State**:
   - AlertBanner merah dengan pesan: `"Gagal memuat daftar produk. Periksa koneksi internet Anda."` dan tombol `"Coba Lagi"`.

---

### 4.2. Modal Tambah & Ubah Produk (`AddEditProductModal`)

#### Metadata & Rangkuman Komponen
- **Tujuan**: Mendaftarkan produk baru atau memperbarui informasi produk lama, termasuk SKU opsional, satuan, harga jual, dan inisialisasi HPP pertama.
- **Dimensi**: Modal dialog `560px` (desktop/tablet), Full Bottom Sheet (mobile 390px).
- **Perilaku Khusus (Append-Only Logic)**:
  - Mengubah informasi umum (Nama, Kategori, SKU, Satuan, Status) memperbarui baris produk (`products`).
  - Mengubah **Harga Jual** pada mode Edit memicu pencatatan otomatis ke `product_price_history` dengan field `effective_from` hari ini dan `price` baru (TD-08).
  - Mengubah HPP pada formulir ini dialihkan melalui tab/seksi terpisah `"Buat Versi HPP Baru"` agar tidak menimpa versi biaya yang telah digunakan pada transaksi masa lalu.

#### Struktur Kolom Formulir
1. **Nama Produk** (Wajib):
   - Label: `"Nama Produk *"`
   - Input: Teks biasa, autofocus pada modal Tambah, placeholder: `"Contoh: Kopi Susu Gula Aren 250ml"`.
   - Validasi: Minimal 2 karakter, maksimal 150 karakter. Error jika kosong: `"Nama produk wajib diisi."`
2. **SKU / Kode Produk** (Opsional):
   - Label: `"SKU / Kode Produk"`
   - Input: Teks uppercase/alphanumeric, placeholder: `"KOP-AREN-01"`.
   - Helper Text: `"Opsional. Harus unik per produk jika diisi."`
   - Validasi: Jika diisi, periksa keunikan tenant. Error jika duplikat: `"SKU sudah digunakan oleh produk lain."`
3. **Kategori Produk** (Wajib):
   - Label: `"Kategori *"`
   - Input: Dropdown combo-box dengan opsi default (Makanan, Minuman, Bahan Mentah, Jasa, Lainnya) atau ketik kategori kustom baru.
4. **Satuan Ukuran** (Wajib):
   - Label: `"Satuan *"`
   - Input: Dropdown pilihan (`pcs`, `porsi`, `botol`, `cup`, `kg`, `gram`, `liter`, `box`). Default: `pcs`.
5. **Harga Jual Saat Ini** (Wajib):
   - Label: `"Harga Jual per Unit *"`
   - Input: Format Rupiah dengan prefix `Rp` terpatri (*fixed addon prefix*), input rata kanan, `font-variant-numeric: tabular-nums`.
   - Helper Text: `"Harga jual standar yang berlaku untuk transaksi pelanggan."`
   - Validasi: Wajib angka positif (`> 0`). Error jika 0 atau negatif: `"Harga jual harus lebih dari Rp 0."`
6. **Inisialisasi HPP (Harga Pokok Penjualan) — Khusus Modal Tambah**:
   - Pilihan Accordion / Toggle: `[x] Tetapkan HPP awal sekarang` (default: tercentang).
   - Saat tercentang, muncul opsi:
     - Opsi A: *Input Langsung HPP per Unit* (cocok untuk reseller/toko ritel sederhana). Input nominal `formatRupiah`.
     - Opsi B: *Hitung dari Komponen Biaya* (membuka sub-panel komponen bahan, kemasan, tenaga kerja; yield default = 1).
   - Helper Text: `"HPP digunakan untuk menghitung laba kotor deterministik. Jika belum tahu pasti, Anda dapat mengaturnya nanti."`
7. **Status Produk**:
   - Toggle Switch: `Aktif` (dapat dipilih di transaksi) vs `Nonaktif` (disembunyikan dari transaksi baru, histori lama tetap aman).

#### Aksi Footer Modal
- Kiri: Tombol `"Batal"` (Secondary neutral, menutup modal tanpa menyimpan data).
- Kanan: Tombol `"Simpan Produk"` (Primary Emerald, menampilkan spinner loading saat `isSubmitting = true`).
- Keyboard Shortcut: `Esc` untuk menutup (dengan dialog konfirmasi jika formulir sudah terisi), `Ctrl+Enter` / `Cmd+Enter` untuk submit instan.

---

### 4.3. Drawer Histori Biaya (HPP) & Riwayat Harga Jual

#### Metadata & Rangkuman
- **Tujuan**: Membuka audit trail kronologis yang tidak dapat diubah (*immutable append-only history*, TD-08) atas setiap perubahan HPP dan fluktuasi harga jual produk terkait.
- **Dimensi**: Right Slide Drawer selebar `640px` (desktop), Full Modal/Sheet (mobile 390px).
- **Header Drawer**:
  - Judul: `"Riwayat Biaya & Harga: {product_name}"`
  - Subjudul: `"SKU: {sku} · Kategori: {category} · Satuan: {unit}"`
  - Tab Switcher: Tab 1 `"Versi Biaya (HPP)"` (aktif), Tab 2 `"Riwayat Harga Jual"`.

#### Tab 1: Histori Versi HPP (`product_cost_versions`)
- **Tombol Aksi Atas**: `"+ Buat Versi HPP Baru"` (membuka formulir penambahan versi biaya baru).
- **Struktur Node Timeline** (Urutan dari versi terbaru di atas ke versi terlama di bawah):
  - **Badge Versi**: `"Versi #{version_no}"` (Versi aktif saat ini memiliki penanda tambahan chip hijau `"Sedang Berlaku"`).
  - **Tanggal Efektif**: Tanggal mulai berlakunya biaya (`effective_from`, format `DD/MM/YYYY`, misal: `01/10/2026`).
  - **Metrik Utama Baris**:
    - Total Biaya Produksi: `formatRupiah(total_cost)`
    - Hasil Produksi (*Yield*): `{yield_units} {unit}`
    - **HPP per Unit**: `formatRupiah(hpp_per_unit)` (Teks tebal 16px, font tabular).
    - Selisih vs Versi Sebelumnya: Badge indikator naik/turun, misal: `+Rp 1.200 (+8.5%)` warna merah (jika biaya naik) atau hijau (jika biaya turun).
  - **Catatan / Alasan Perubahan (`note`)**:
    - Teks kutipan di dalam latar `var(--color-bg-subtle)`: *"Penyesuaian kenaikan harga biji kopi robusta dari supplier utama."*
  - **Accordion Komponen Biaya (`product_cost_components`)**:
    - Toggle `"Lihat Rincian Biaya (X komponen)"` yang dapat dibuka (*collapsible*).
    - Tabel mini komponen:
      | Tipe Biaya | Nama Komponen | Nominal Biaya | Porsi (%) |
      | :--- | :--- | :--- | :--- |
      | Bahan Baku (`ingredient`) | Biji Kopi Robusta 1kg | Rp 85.000 | 56.7% |
      | Bahan Baku (`ingredient`) | Susu UHT 5 Liter | Rp 45.000 | 30.0% |
      | Kemasan (`packaging`) | Cup 250ml + Lid + Sedotan | Rp 15.000 | 10.0% |
      | Tenaga Kerja (`direct_labor`) | Upah barista per batch | Rp 5.000 | 3.3% |
      | **Total Biaya Batch** | **Yield: 25 cup** | **Rp 150.000** | **HPP: Rp 6.000/cup** |

#### Sub-Alur: Formulir "Buat Versi HPP Baru"
- **Prinsip**: Append-only. Versi lama tidak pernah ditimpa (TD-08). Transaksi historis tetap mengacu pada `cost_version_id` yang berlaku pada tanggal transaksi tersebut.
- **Field**:
  1. Tanggal Berlaku (`effective_from`): Default hari ini, dapat diisi tanggal lampau untuk data susulan.
  2. Jumlah Hasil Produksi (*Yield Units*): Input angka positif, default `1`.
  3. Tabel Komponen Biaya Dinamis:
     - Tombol `"+ Tambah Komponen Biaya"`
     - Baris komponen: Dropdown Tipe (`ingredient`, `packaging`, `direct_labor`, `other_direct`, `allocated_overhead`), Input Nama Komponen, Input Nominal Rupiah, Tombol Hapus baris.
     - Live Kalkulasi: Total Biaya Batch dihitung otomatis, HPP per Unit = `Total Biaya / Yield Units`.
  4. Alasan / Catatan Perubahan (`note`): Wajib diisi minimal 5 karakter (misal: *"Kenaikan harga susu kemasan"*).
  5. Tombol `"Simpan Versi HPP Baru"`.

#### Tab 2: Riwayat Harga Jual (`product_price_history`)
- **Tampilan Tabel Kronologis**:
  | Tanggal Berlaku | Harga Jual | Perubahan | Margin Teoretis saat itu | Diubah Oleh |
  | :--- | :--- | :--- | :--- | :--- |
  | 15/10/2026 | Rp 18.000 | +Rp 2.000 (+12.5%) | 66.7% (HPP Rp 6.000) | Budi (Owner) |
  | 01/08/2026 | Rp 16.000 | Harga Awal | 62.5% (HPP Rp 6.000) | Budi (Owner) |
- **Catatan Audit**: Setiap kali pengguna mengubah harga jual produk di modal edit, baris baru ditambahkan ke riwayat ini secara otomatis.

---

### 4.4. Kalkulator Penetapan Harga Interaktif (`PricingCalculator`)

#### Metadata & Rangkuman Komponen
- **Tujuan**: Alat simulasi keuangan deterministik bagi pemilik usaha untuk menentukan harga jual ideal berdasarkan target margin laba kotor atau markup atas HPP, lengkap dengan deteksi risiko margin tipis.
- **Dimensi**: Modal dialog terpusat `600px` (desktop) atau Full Bottom Sheet (mobile).
- **Mode Integrasi**:
  1. *Mode Tersemat / Kontekstual*: Dibuka langsung dari baris produk tertentu (otomatis mengisi data HPP dan Harga Jual produk tersebut). Memiliki tombol `"Terapkan ke Harga Jual Produk"`.
  2. *Mode Bebas (Sandbox)*: Dibuka dari header utama `/products`. Pengguna bebas memasukkan angka HPP sembarang untuk merancang produk baru.

#### Dua Mode Simulasi (Toggle Tab Pilihan)

##### Mode A: "Tentukan Harga dari Target Margin / Markup" (Rekomendasi Utama)
1. **Input HPP per Unit**:
   - Input format Rupiah (contoh: `Rp 12.000`). Jika dibuka dari produk tertentu, field ini terisi otomatis dan dapat di-override.
2. **Pilihan Model Penetapan**:
   - Radio / Segmented Control: `[ Target Margin Kotor (%) ]` vs `[ Target Markup (%) ]`.
   - *Bantuan Edukasi Transparan (Sesuai PRD §5.3)*:
     - Tooltip Margin: *"Margin dihitung dari persentase Laba Kotor terhadap Harga Jual akhir. Formula: Laba / Harga Jual."*
     - Tooltip Markup: *"Markup dihitung dari persentase penambahan di atas modal HPP. Formula: Laba / HPP."*
3. **Slider & Input Target Persentase**:
   - Slider visual rentang `5%` hingga `80%` bergradasi step 1%, berdampingan dengan kolom input teks numerik `[ 30 ] %`.
   - Preset Cepat: Tombol chip rekomendasi UMKM: `[ 20% (Ritel) ]`, `[ 35% (F&B Standar) ]`, `[ 50% (F&B Premium / Fashion) ]`.
4. **Rumus Deterministik Sistem (TRD TD-12)**:
   - Jika memilih Target Margin ($M$):
     $$\text{Harga Jual Rekomendasi} = \frac{\text{HPP}}{1 - M}$$
     $$\text{Laba Kotor per Unit} = \text{Harga Jual} - \text{HPP}$$
     $$\text{Markup Terbentuk} = \left(\frac{\text{Laba Kotor}}{\text{HPP}}\right) \times 100\%$$
   - Jika memilih Target Markup ($K$):
     $$\text{Harga Jual Rekomendasi} = \text{HPP} \times (1 + K)$$
     $$\text{Laba Kotor per Unit} = \text{Harga Jual} - \text{HPP}$$
     $$\text{Margin Terbentuk} = \left(\frac{\text{Laba Kotor}}{\text{Harga Jual}}\right) \times 100\%$$
5. **Panel Hasil Kalkulasi Instan**:
   - Kotak ringkasan berlatar `var(--color-bg-subtle)` berbatas `1px solid var(--color-border-default)`:
     - **Rekomendasi Harga Jual**: `var(--text-3xl-size)` font tabular, warna `var(--color-text-main)`. Dilengkapi opsi tombol pembulatan psikologis: `[ Bulatkan ke Rp 500 terdekat ]` atau `[ Bulatkan ke Rp 1.000 terdekat ]`.
     - **Laba Kotor per Unit**: `+Rp 5.143` (warna hijau emerald `var(--color-primary-700)`).
     - **Markup Ekuivalen**: `42.8%`.

##### Mode B: "Uji Margin dari Rencana Harga Jual"
1. **Input HPP per Unit**: Input Rupiah (misal: `Rp 12.000`).
2. **Input Rencana Harga Jual**: Input Rupiah (misal: `Rp 15.000`).
3. **Hasil Perhitungan**:
   - Laba Kotor per Unit = `Rp 3.000`
   - Margin Kotor = `20.0%`
   - Markup = `25.0%`

#### Indikator Risiko & Peringatan Visual (Kondisi Khusus)
Sistem secara proaktif memeriksa kesehatan margin hasil simulasi (Sesuai Anti-Slop & PRD §5.3):

1. **Kasus Bahaya: Harga Jual di Bawah HPP (Rugi Kotor)**:
   - Kondisi: `Harga Jual <= HPP` atau `Margin <= 0%`.
   - Visual: AlertBanner merah menyala (`var(--color-error-bg)`, border `var(--color-error-border)`).
   - Teks Pesan: `"Harga Jual di bawah HPP! Anda akan mengalami kerugian sebesar {formatRupiah(HPP - Harga)} per unit yang terjual."`
   - Aksi: Tombol `"Terapkan ke Produk"` dinonaktifkan (*disabled*).
2. **Kasus Peringatan: Margin Sangat Tipis (< 15%)**:
   - Kondisi: `0% < Margin < 15%`.
   - Visual: AlertBanner amber / kuning hangat (`var(--color-warning-bg)`, border `var(--color-warning-border)`).
   - Teks Pesan: `"Peringatan: Margin Kotor sangat tipis ({margin}%). Rentan tergerus jika terjadi kenaikan harga bahan baku, biaya kemasan, atau potongan komisi transaksi."`
3. **Kasus Sehat (15% - 50%)**:
   - Visual: Badge hijau emerald lembut (`var(--color-success-bg)`).
   - Teks Pesan: `"Margin berada dalam batas aman operasional UMKM."`
4. **Catatan Disclaimer Edukatif**:
   - Teks mikro di bawah kalkulator (`var(--text-xs-size)`, `var(--color-text-muted)`):
     *"Perhitungan di atas adalah Laba Kotor (belum dikurangi biaya sewa, listrik, promosi, dan biaya operasional tidak langsung lainnya). Pastikan margin kotor Anda cukup untuk menutupi beban tetap bulanan."*

---

## 5. Responsive Breakdowns (Detail Ukuran Layar)

### 5.1. Desktop (1440px)
- **Katalog (`/products`)**:
  - Sidebar aplikasi permanen di sisi kiri (lebar 240px).
  - Konten utama memiliki `max-width: 1200px`, padding horizontal `var(--space-8)`.
  - FilterToolbar sejajar dalam satu baris fleksibel: Input Cari (lebar 360px), Dropdown Kategori (180px), Dropdown Status (150px), Dropdown HPP (160px), dan Tombol Reset jika aktif.
  - ProductTable menampilkan 8 kolom penuh dengan pagination bar di bagian kaki tabel (`1-20 dari 85 produk`).
- **Modal Add/Edit**: Lebar tetap `560px`, terpusat secara vertikal dan horizontal.
- **Drawer Histori**: Slide panel dari sisi kanan selebar `640px` dengan backdrop overlay gelap.
- **Kalkulator**: Dialog modal terpusat selebar `600px`.

### 5.2. Tablet (768px)
- **Katalog (`/products`)**:
  - Sidebar aplikasi dapat diciutkan (*collapsible hamburger menu*). Padding konten `var(--space-6)`.
  - Strip Metrik HPP bertumpuk 2 baris (kartu 1 & 2 di baris pertama, kartu 3 di baris kedua lebar penuh).
  - FilterToolbar membungkus menjadi 2 baris: Baris 1 Input Cari (100%), Baris 2 ketiga dropdown filter berdampingan.
  - ProductTable menyembunyikan kolom sekunder: Kolom `Satuan` digabungkan ke bawah nama produk, menyisakan 6 kolom utama.
- **Modal & Drawer**:
  - Modal Add/Edit: Lebar `90%` (maksimal 540px).
  - Drawer Histori: Menempati lebar `80%` layar tablet dari kanan.

### 5.3. Mobile (390px — Smartphone Viewport)
- **Katalog (`/products`)**:
  - Padding konten `var(--space-4)`. Tidak ada sidebar fisik; navigasi berpindah ke top app bar + bottom nav bar.
  - Header halaman kompak: Judul `"Produk"` berdampingan dengan tombol icon kalkulator harga.
  - Tombol Primer Aksi: Menggunakan Floating Action Button (FAB) berlabel `"+ Produk"` di sudut kanan bawah layar (`position: fixed`, `bottom: 72px`, `right: 16px`, tinggi 48px, radius pill, z-index `var(--z-header)`).
  - FilterToolbar: Input cari selebar 100%, di bawahnya terdapat deretan horizontal scrollable chip filter (`Semua`, `Makanan`, `Minuman`, `Tanpa HPP`).
  - Tampilan Data: **ProductTable dihilangkan, digantikan sepenuhnya oleh deretan ProductCard vertikal** yang ramah sentuhan (touch target minimal 48px pada setiap tombol aksi).
- **Semua Modal & Drawer**:
  - Add/Edit Modal, Drawer Histori, dan Kalkulator bertransformasi menjadi **Full-Width Bottom Sheet** (`height: 92vh`, radius atas 16px, dengan drag handle untuk menutup ke bawah).
  - Tombol aksi simpan/terapkan ditempatkan pada sticky footer di bagian paling bawah bottom sheet sehingga selalu terlihat di atas keyboard virtual smartphone.

---

## 6. Spesifikasi Motion & Transisi Antarmuka

Mengikuti aturan ketat: *Motion calm, responsive, and purposeful. Never blocks input. No decorative bounce.*

| Elemen Antarmuka | Pemicu (Trigger) | Properti Diubah | Durasi | Kurva Easing | Fallback Reduced Motion |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Baris Tabel Produk** | Hover mouse | `background-color` (`--color-bg-surface` → `--color-bg-hover`) | 120ms | `--ease-standard` | Instan tanpa transisi |
| **Tombol Aksi / FAB** | Press (Active) | `transform: scale(0.98)` | 100ms | `--ease-exit` | Tanpa scaling |
| **Modal Tambah/Ubah** | Enter (Buka) | `opacity: 0 → 1`, `transform: scale(0.96 → 1)` | 240ms | `--ease-enter` | Hanya `opacity: 0 → 1` |
| **Modal Tambah/Ubah** | Exit (Tutup) | `opacity: 1 → 0`, `transform: scale(1 → 0.96)` | 180ms | `--ease-exit` | Hanya `opacity: 1 → 0` |
| **Drawer Histori Biaya** | Enter (Slide in) | `transform: translateX(100% → 0)`, `opacity: 0.5 → 1` | 280ms | `--ease-enter` | `opacity: 0 → 1` tanpa geser |
| **Drawer Histori Biaya** | Exit (Slide out) | `transform: translateX(0 → 100%)`, `opacity: 1 → 0` | 200ms | `--ease-exit` | `opacity: 1 → 0` tanpa geser |
| **Bottom Sheet Mobile** | Enter (Slide up) | `transform: translateY(100% → 0)` | 260ms | `--ease-enter` | `opacity: 0 → 1` |
| **Accordion Rincian HPP** | Expand / Collapse| `max-height`, `opacity` | 200ms | `--ease-standard` | Instan buka/tutup |
| **Warning Banner Margin**| Muncul saat simulasi | `opacity: 0 → 1`, `transform: translateY(-4px → 0)` | 160ms | `--ease-enter` | Instan |
| **Skeleton Loading Data**| Kondisi Fetching | `background-position` shimmer atau pulse opacity | 1.4s loop | Linear | Statis warna subtle |

---

## 7. Aksesibilitas (A11y Checklist & Keyboard Matrix)

### 7.1. Navigasi Keyboard & Focus Trapping
- **Modal & Bottom Sheet**:
  - Saat modal terbuka, fokus otomatis diarahkan ke input pertama (Input Nama Produk pada modal Add, atau Input HPP pada kalkulator).
  - *Focus Trap*: Menekan tombol `Tab` pada elemen terakhir di dalam modal akan memutar fokus kembali ke elemen interaktif pertama (tombol tutup / input pertama).
  - Tombol `Esc` segera menutup modal/drawer dan mengembalikan fokus ke elemen pemicu (*trigger button* pada tabel).
- **ProductTable**:
  - Baris tabel dapat dijelajahi dengan tombol panah atau tombol `Tab`.
  - Tombol menu aksi `...` memiliki atribut `aria-haspopup="menu"` dan `aria-expanded="false"`.

### 7.2. ARIA Attributes & Live Regions
- **Tabel**:
  - Elemen tabel memiliki atribut `role="table"` dengan `aria-label="Daftar Produk dan Harga Pokok Penjualan"`.
  - Kolom sortable (jika ada) dilengkapi `aria-sort="ascending|descending|none"`.
- **Kalkulator Harga**:
  - Kotak hasil rekomendasi harga jual memiliki atribut `aria-live="polite"` sehingga setiap pergeseran slider persentase margin akan dibacakan oleh pembaca layar tanpa menginterupsi input aktif.
  - AlertBanner risiko margin tipis menggunakan `role="alert"` dan `aria-live="assertive"`.
- **Status Badges**:
  - Seluruh status menggunakan teks eksplisit (bukan sekadar indikator warna). Pembaca layar akan membaca: `"Status: Aktif"`, `"Peringatan: Margin Tipis 12 persen"`, `"HPP Belum Diatur"`.

### 7.3. Touch Targets & Form Inputs
- Seluruh input teks, dropdown, tombol aksi baris, dan chip filter memiliki tinggi minimal `44px` dan lebar minimal `44px`.
- Seluruh label input terhubung secara semantik ke input kontrol menggunakan atribut `for="{input_id}"` dan `id="{input_id}"`.
- Pesan validasi error memiliki id `id="{input_id}-error"` dan dihubungkan ke input via `aria-describedby="{input_id}-error"`.

---

## 8. Anti-Slop Self-Check Verification

| Prinsip Anti-Slop | Implementasi pada Spesifikasi Ini | Status |
| :--- | :--- | :--- |
| **Tanpa gradien dekoratif AI** | Seluruh kanvas menggunakan `#F7F9F8`, aksen tunggal `#0D6E64` Emerald Forest. | TERPENUHI |
| **Tanpa bayangan pada resting card** | Kartu metrik, kartu produk mobile, dan tabel menggunakan border `1px solid var(--color-border-subtle)`. Bayangan hanya pada modal/drawer. | TERPENUHI |
| **Tanpa label eyebrow berlebihan** | Judul halaman langsung to the point tanpa uppercase micro-badge yang tidak fungsional. | TERPENUHI |
| **Kaidah Bahasa Indonesia Lugas** | Menggunakan istilah bisnis UMKM lokal yang tepat: "Laba Kotor", "HPP per Unit", "Margin Kotor", "Satuan". Tanpa jargon asing berlebihan. | TERPENUHI |
| **Informasi Deterministik & Transparan** | Menolak estimasi kosong; menampilkan HPP `NULL` sebagai "Belum diatur", bukan 0 Rupiah yang memalsukan data. | TERPENUHI |
| **Mobile-First UX** | Mengganti tabel horizontal yang rumit dengan kartu bertumpuk (`ProductCard`) di resolusi 390px. | TERPENUHI |
| **Audit Trail Abadi (TD-08)** | Versi HPP bersifat append-only; harga jual dicatat dalam log riwayat tanpa merusak rekaman masa lalu. | TERPENUHI |
