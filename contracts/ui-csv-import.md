# Spesifikasi UI & Alur Interaksi: CSV Import Wizard (M5)

## Metadata
- **ID Kontrak**: UI-M5-001
- **Task ID**: T-050
- **Penulis**: Designer Profile / Product Lead
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED
- **Design System**: Clean Pragmatic Craftsman (`contracts/design-tokens.md`)
- **Primary Color**: Emerald Forest (`#0D6E64` / `var(--color-primary-700)`)

---

## 1. Prinsip Desain & Hirarki Visual
Modul **CSV Import Wizard** (`/imports` atau `/imports/new`) dirancang untuk memandu pemilik UMKM dan staf operasional mengimpor data bisnis dalam volume besar tanpa kebingungan teknis ataupun rasa intimidasi, selaras dengan `contracts/design-direction.md`:

1. **Aksen Tunggal**: *Emerald Forest* (`#0D6E64` / `var(--color-primary-700)`). Aksen fungsional pada langkah aktif stepper, tombol konfirmasi, dan tautan unduh template.
2. **Tipografi Bersih & Data Tabular**:
   - Teks antarmuka, judul, dan label instruksi: **Plus Jakarta Sans** (500/600/700).
   - Pratinjau data baris tabel, angka nominal, nomor SKU, nomor telepon, dan kalkulasi: **Inter** (`tabular-nums` mono/tabular).
3. **Tanpa Bayangan Diam (*No Resting Shadows*)**:
   - Seluruh kartu dropzone, tabel pemetaan, dan panel ringkasan menggunakan diferensiasi permukaan (*surface*) berbatas kontras `1px solid var(--color-border-default)` (`#CBD5D1`) atau `var(--color-border-subtle)` (`#E2E8E5`).
   - Bayangan elevasi (`var(--shadow-lg)`) hanya digunakan pada modal konfirmasi / dialog pembatalan (*elevated layers*).
4. **Pencegahan Kesalahan & Transparansi (*Error Prevention & Clarity*)**:
   - File mentah asli tidak disimpan di server (hanya nilai SHA-256 dan baris hasil *parse* ke tabel sementara `import_rows`).
   - Setiap pesan peringatan (*warning*) dan galat (*error*) disajikan dalam Bahasa Indonesia yang lugas, spesifik, dan menyertakan tindakan perbaikan.
   - Angka ambigu (misalnya `1.500`) tidak ditebak sepihak, melainkan ditandai untuk diputuskan pengguna.
   - Pencocokan nama pelanggan tidak digabung otomatis demi mematuhi aturan privasi dan integritas data (TRD §8).
5. **Kepatuhan Keamanan Formula CSV (*Anti-Formula Injection*)**:
   - Ekspor laporan baris galat wajib menetralkan sel yang diawali karakter `=`, `+`, `-`, atau `@` dengan menambahkan kutip tunggal (`'`).

---

## 2. Struktur Alur Wizard 5-Langkah

Antarmuka disajikan dalam kontainer terstruktur dengan **Stepper Navigasi 5-Langkah** di bagian atas:
```
[1. Unggah File] ──> [2. Pemetaan Kolom] ──> [3. Validasi Data] ──> [4. Konfirmasi & Proses] ──> [5. Ringkasan & Hasil]
```

### Stepper Header Component
- **Indikator Langkah**:
  - *Completed*: Lingkaran latar hijau zamrud dengan ikon centang (`✓`), teks label reguler `var(--color-text-main)`.
  - *Current / Active*: Lingkaran berbingkai tebal `2px solid var(--color-primary-700)` dengan angka langkah tebal, label semibold `var(--color-primary-700)`.
  - *Upcoming*: Lingkaran abu-abu netral latar `var(--color-bg-subtle)`, angka abu-abu, teks `var(--color-text-tertiary)`.
- **Garis Penghubung**: Garis halus `2px solid var(--color-border-subtle)` (berubah hijau saat langkah terlampaui).

---

### 2.1 Langkah 1: Unggah File (`Step 1: Upload`)

#### Tujuan Layar
Memungkinkan pengguna memilih jenis entitas data yang akan diimpor, mengunduh template resmi, memilih berkas CSV (maksimal 5 MB), dan menentukan pembatas pemisah (*delimiter*).

#### Komponen & Tata Letak
1. **Pemilih Jenis Data (Entitas Impor)**:
   - 3 tombol kartu pilihan (*selectable flat cards*):
     - **Katalog Produk** (`products`): Nama produk, SKU, kategori, satuan, harga jual, estimasi HPP awal.
     - **Daftar Pelanggan** (`customers`): Nama, no. WhatsApp/telepon, email, catatan preferensi.
     - **Transaksi Penjualan** (`transactions`): Riwayat pesanan, ref order, pelanggan, SKU item, qty, harga, diskon, biaya admin.
   - Status kartu terpilih: Bingkai `2px solid var(--color-primary-700)` dengan latar `var(--color-primary-50)` dan tanda centang di sudut kanan atas.
2. **Panel Unduh Template CSV Resmi**:
   - Banner informatif: "Gunakan template resmi kami untuk meminimalkan kesalahan pemetaan kolom."
   - Tautan aksi: `Unduh Template Produk (.csv)`, `Unduh Template Pelanggan (.csv)`, `Unduh Template Transaksi (.csv)`.
   - Tombol mengunduh file CSV berkode UTF-8 dengan baris header standar.
3. **Area Dropzone Unggah Berkas**:
   - Area persegi panjang berbatas putus-putus (`border: 2px dashed var(--color-border-default)`), tinggi minimal `200px`.
   - Ikon unggah berkas (arsitektur dokumen monokrom, bukan emoji).
   - Teks instruksi: **"Tarik & lepas file CSV Anda di sini"**, atau tombol teks sekunder **"Pilih Berkas dari Komputer"**.
   - Keterangan batasan teknis: "Hanya format .csv, pengkodean UTF-8, ukuran maksimal 5 MB (hingga 20.000 baris)."
   - **Interaksi Drag & Drop**:
     - Saat kursor membawa file melintas di atas dropzone (*dragover*): batas berubah menjadi `2px dashed var(--color-primary-700)`, latar menjadi `var(--color-primary-50)`, teks berubah menjadi "Lepaskan file CSV untuk mengunggah".
   - **Status Berkas Terpilih**:
     - Kotak informasi berkas: Ikon dokumen CSV, nama file (e.g. `data-penjualan-oktober.csv`), ukuran berkas (e.g. `1.2 MB`), tombol ganti/hapus file (*icon silang*).
4. **Pengaturan Pembatas Kolom (*Delimiter Override*)**:
   - Dropdown pilihan pembatas:
     - `Otomatis Deteksi (Disarankan)` [Default]
     - `Koma (,)` — Standar Google Sheets & internasional
     - `Titik Koma (;)` — Standar ekspor Microsoft Excel format Indonesia
     - `Tab (\t)` — Format teks berjarak tab
   - Petunjuk kontekstual di bawah dropdown: "Microsoft Excel di komputer Indonesia umumnya mengekspor CSV menggunakan titik koma (;)."
5. **Banner Peringatan Berkas Duplikat (Repeat Upload Warning)**:
   - Muncul otomatis jika `sha256(file)` cocok dengan batch impor yang pernah diproses sebelumnya.
   - Varian Amber / Peringatan (`var(--color-warning-bg)`):
     - Teks: "Perhatian: Berkas ini memiliki isi data identik dengan batch impor sebelumnya (#IMP-20261015-002 pada 15 Okt 2026, 14:20 WIB)."
     - Keterangan: "Jika Anda melanjutkan, transaksi dengan `order_ref` yang sama akan diabaikan atau ditandai sebagai duplikat untuk mencegah pencatatan ganda."
     - Kotak centang persetujuan: `Saya memahami dan ingin tetap memproses berkas ini`.
6. **Tombol Navigasi Bawah**:
   - Tombol Kiri: `Batal` (menutup wizard dan kembali ke halaman sebelumnya).
   - Tombol Kanan: `Lanjutkan ke Pemetaan Kolom →` (Tombol Primer Emerald; kondisi dinonaktifkan / *disabled* jika file belum dipilih atau ukuran melebihi 5 MB).

---

### 2.2 Langkah 2: Pemetaan Kolom & Pengaturan Format (`Step 2: Column Mapping & Parsing Settings`)

#### Tujuan Layar
Menghubungkan kolom yang terdeteksi di dalam file CSV pengguna dengan kolom target basis data Teman UMKM, serta menetapkan konvensi format angka dan tanggal.

#### Komponen & Tata Letak
1. **Pengaturan Format Nilai (*Parsing Settings Card*)**:
   - Baris 2 kolom pengaturan cepat di atas tabel pemetaan:
     - **Format Angka & Desimal**:
       - Dropdown opsi:
         - `Format Indonesia (1.500,50)` [Default]: Titik sebagai pemisah ribuan, koma sebagai desimal.
         - `Format Standar Internasional (1,500.50)`: Koma sebagai pemisah ribuan, titik sebagai desimal.
       - Teks pembantu: "Menentukan cara sistem menginterpretasikan harga dan kuantitas."
     - **Format Tanggal**:
       - Dropdown opsi:
         - `DD/MM/YYYY` (Contoh: `18/10/2026`) [Default]
         - `DD-MM-YYYY` (Contoh: `18-10-2026`)
         - `YYYY-MM-DD` (Contoh: `2026-10-18`)
       - Teks pembantu: "Wajib disesuaikan dengan format penulisan tanggal pada berkas CSV Anda."
2. **Tabel Pemetaan Kolom (*Source-to-Target Mapping Grid*)**:
   - Tabel 4 kolom:
     1. **Kolom Target Sistem**:
        - Label field (contoh: "SKU Produk", "Harga Jual Satuan", "Tanggal Transaksi").
        - Badge status: `Wajib` (merah halus `var(--color-error-bg)` + teks merah) atau `Opsional` (abu-abu netral).
     2. **Deskripsi Kolom**: Penjelasan singkat kegunaan data dan aturan validasinya.
     3. **Kolom Sumber CSV**:
        - Dropdown select berisi seluruh header yang ditemukan pada baris pertama CSV pengguna, plus opsi paling atas `(Abaikan / Jangan Impor)`.
        - Indikator otomatis: Jika header CSV sama atau mirip dengan target, dropdown otomatis terpilih dan memunculkan badge hijau mini `Cocok Otomatis`.
     4. **Status Pemetaan**:
        - Badge hijau `Terpetakan` jika valid.
        - Badge merah `Wajib Diisi` jika kolom target berstatus wajib namun pengguna memilih "(Abaikan)".
3. **Komponen Pratinjau Cepat 3 Baris Pertama (*Live Preview Component*)**:
   - Kotak tinjauan dinamis berlatar `var(--color-bg-subtle)` berbingkai halus tepat di bawah tabel pemetaan.
   - Menampilkan tabel mini hasil interpretasi sistem terhadap 3 baris data pertama CSV berdasarkan pengaturan format angka dan pemetaan saat ini.
   - Memberikan kepastian visual seketika (contoh: angka `100.000` berhasil di-parse menjadi nominal Rupiah `Rp 100.000`, dan tanggal `18/10/2026` dibaca dengan benar).
4. **Tombol Navigasi Bawah**:
   - Tombol Kiri: `← Kembali ke Unggah`
   - Tombol Kanan: `Lanjut ke Validasi Data →` (Primer; dinonaktifkan jika masih ada kolom wajib yang belum dipetakan).

---

### 2.3 Langkah 3: Validasi & Pemeriksaan Data (`Step 3: Validation & Error Inspection`)

#### Tujuan Layar
Menjalankan audit integritas baris data tanpa menyimpannya ke database produksi. Memeriksa keberadaan SKU produk, normalisasi telepon pelanggan, format data, dan duplikasi baris.

#### Komponen & Tata Letak
1. **Kartu Rangkuman Hasil Validasi (*Validation Summary Pills*)**:
   - Baris 3 kartu metrik ringkas:
     - **Baris Siap Diimpor (Valid)**: Latar `var(--color-success-bg)`, border `var(--color-success-border)`, angka besar font Inter warna hijau gelap `var(--color-success-text)`, label "Siap masuk sistem".
     - **Peringatan (Warnings)**: Latar `var(--color-warning-bg)`, border `var(--color-warning-border)`, angka besar warna amber `var(--color-warning-text)`, label "Perlu perhatian / tetap dapat diproses".
     - **Baris Galat (Errors)**: Latar `var(--color-error-bg)`, border `var(--color-error-border)`, angka besar warna merah `var(--color-error-text)`, label "Tidak valid / akan dilewati".
2. **Filter Interaktif Tabel Baris Data**:
   - Tab navigasi segmented:
     - `Semua Baris (Total: N)`
     - `Hanya Galat (N)` (Fokus perbaikan masalah)
     - `Hanya Peringatan (N)`
     - `Baris Valid (N)`
   - Input pencarian cepat di sisi kanan untuk mencari nomor baris, SKU, atau nama pelanggan.
3. **Tabel Peninjauan Baris (*Data Inspection Table*)**:
   - Kolom tabel:
     1. **Baris #**: Nomor urut baris asli pada berkas CSV (e.g. `Baris 14`).
     2. **Ringkasan Data**: Cuplikan data utama (e.g. `TRX-1049 | Kopi Susu Aren (SKU-001) | Qty: 2`).
     3. **Status**: Badge status (`Valid` hijau, `Peringatan` amber, `Galat` merah).
     4. **Rincian Masalah & Rekomendasi Tindakan**:
        - Teks penjelasan kesalahan berbahasa Indonesia yang jelas dan spesifik.
        - Contoh galat:
          - *"SKU 'KOP-999' tidak ditemukan pada katalog produk."*
          - *"Nilai harga 'abc' tidak valid untuk format angka rupiah."*
          - *"Kuantitas barang bernilai 0 atau negatif."*
        - Contoh peringatan:
          - *"Nomor WhatsApp dinormalisasi otomatis dari '08123456789' menjadi '+628123456789'."*
          - *"Nilai '1.500' ambigu antara 1.500 atau 1,5. Ditafsirkan sebagai Rp 1.500 sesuai format angka Indonesia."*
4. **Panel Resolusi Isu Khusus (*Resolution Modules*)**:
   - **Resolusi Pencocokan Pelanggan ("Kemungkinan Pelanggan yang Sama")**:
     - Sesuai TRD §8: Sistem dilarang menggabungkan pelanggan hanya berdasarkan kemiripan nama tanpa nomor kontak yang cocok.
     - Tampilan kartu resolusi: Menampilkan baris yang memiliki nama serupa dengan pelanggan lama (misal "Budi Santoso" vs "Budi Santoso - Kantor").
     - Opsi toggle radio untuk pengguna:
       - `(A) Buat Pelanggan Baru Terpisah` [Default rekomendasi sistem].
       - `(B) Tautkan ke Pelanggan Terdaftar: [Pilih Budi Santoso (+62812...)]`.
   - **Resolusi Baris Duplikat**:
     - Jika ditemukan `order_ref` yang sama atau `row_hash` berulang:
     - Opsi aksi: `Lewati Baris Duplikat (Rekomendasi)` atau `Tetap Catat sebagai Transaksi Baru`.
5. **Tombol Navigasi Bawah**:
   - Tombol Kiri: `← Kembali ke Pemetaan`
   - Tombol Kanan: `Lanjutkan ke Konfirmasi →` (Primer).
     - Jika terdapat baris galat: Menampilkan teks peringatan halus di samping tombol: *"N baris galat akan dilewati dan tidak diimpor ke sistem."*
     - Dinonaktifkan jika `0 Baris Valid` (seluruh baris galat).

---

### 2.4 Langkah 4: Konfirmasi & Pemrosesan Latar Belakang (`Step 4: Confirmation & Async Progress`)

#### Tujuan Layar
Menampilkan rekapitulasi final sebelum eksekusi mutasi data, meminta konfirmasi aman pengguna, dan menampilkan progres pemrosesan asinkron (*worker polling*) secara real-time.

#### Komponen & Tata Letak
1. **Modal Konfirmasi Final (*Pre-Execution Confirmation Dialog*)**:
   - Muncul saat pengguna menekan "Lanjutkan ke Konfirmasi":
   - Judul: "Konfirmasi Impor Data"
   - Ringkasan statistik eksekusi:
     - Jumlah data yang akan disimpan: **X data**.
     - Jumlah baris yang dilewati (galat): **Y data**.
     - Entitas terdampak: Produk / Pelanggan / Transaksi.
   - Catatan teknis kepastian:
     - *"Proses berjalan secara aman (*savepoint per row*). Jika terjadi kendala pada satu baris di tengah proses, baris valid lainnya tetap tersimpan secara utuh."*
   - Tombol Aksi Modal:
     - Tombol Sekunder: `Periksa Kembali Data`
     - Tombol Primer: `Mulai Proses Impor Sekarang`
2. **Layar Status Pemrosesan Asinkron (*Async Polling Screen*)**:
   - Setelah konfirmasi ditekan, layar beralih ke mode pemrosesan aktif (State: `processing`).
   - **Kartu Progres Utama**:
     - Judul: "Memproses Impor Data..."
     - Subjudul: "Sistem sedang memvalidasi dan menyimpan data ke pembukuan usaha Anda. Anda dapat tetap berada di layar ini atau membuka halaman lain."
     - **Bilah Kemajuan Asinkron (*Progress Bar*)**:
       - Tinggi `12px`, radius `var(--radius-full)`, latar `var(--color-bg-subtle)`.
       - Isi bilah: `var(--color-primary-700)` dengan animasi transisi halus (`transition: width 300ms ease-out`).
     - **Indikator Metrik Pemrosesan**:
       - Persentase penyelesaian tebal: misal `68%`.
       - Penghitung baris dinamis: **"1.360 dari 2.000 baris selesai diproses"**.
       - Status polling server: Indikator titik berkedip halus (*live heartbeat dot*) dengan status `Mencatat batch transaksi... (interval polling 1,5 detik)`.
   - **Tombol Hentikan Sementara / Batalkan Antrean**:
     - Tombol berbingkai tipis di bawah bilah progres: `Batalkan Proses Impor` (membuka dialog konfirmasi aman pembatalan antrean `POST /imports/{id}/cancel`).

---

### 2.5 Langkah 5: Ringkasan Hasil & Opsi Rollback (`Step 5: Completion Summary & Rollback`)

#### Tujuan Layar
Menyajikan hasil akhir proses impor, menyediakan berkas laporan galat yang aman dari injeksi formula (*formula injection safe*), tautan cepat ke modul terkait, dan mekanisme *rollback* penuh jika pengguna mendeteksi kesalahan data.

#### Komponen & Tata Letak
1. **Status Hasil Akhir (*Outcome Banner & Card*)**:
   - **Varian A: Berhasil Sempurna (`completed` - 100% Berhasil)**:
     - Ikon centang hijau zamrud berlatar bulat lembut `var(--color-success-bg)`.
     - Judul: **"Impor Data Berhasil Sempurna"**.
     - Deskripsi: "Sebanyak **N** data telah berhasil dicatat ke dalam sistem tanpa kendala."
   - **Varian B: Selesai dengan Sebagian Baris Dilewati (`completed_with_errors`)**:
     - Ikon peringatan amber berlatar bulat lembut `var(--color-warning-bg)`.
     - Judul: **"Impor Selesai dengan Catatan"**.
     - Deskripsi: "**X** data berhasil disimpan. Sebanyak **Y** baris memiliki galat dan dilewati oleh sistem."
   - **Varian C: Impor Gagal Total (`failed`)**:
     - Ikon silang merah `var(--color-error-bg)`.
     - Judul: **"Impor Gagal Diproses"**.
     - Deskripsi: Menjelaskan alasan kegagalan teknis (misal kendala jaringan/transaksi database) dengan jaminan data telah di-rollback secara utuh.
2. **Statistik Akhir Impor**:
   - Grid 3 kotak statistik ringkas:
     - Total Baris Diproses: `2.000`
     - Berhasil Disimpan: `1.950`
     - Dilewati / Gagal: `50`
3. **Pusat Unduh Laporan Galat (*Formula-Safe Error Report Download*)**:
   - Ditampilkan aktif jika terdapat baris yang gagal/dilewati.
   - Tombol: `Unduh Laporan Baris Gagal (.csv)`.
   - **Spesifikasi Keamanan Penetrasi Formula CSV (TRD §8)**:
     - Sistem secara otomatis menetralkan karakter berbahaya (`=`, `+`, `-`, `@`) di setiap sel laporan dengan membubuhkan karakter `'` (kutip tunggal) atau spasi di depannya.
     - Teks keterangan keamanan: *"Berkas CSV laporan telah dinetralkan dari potensi injeksi formula spreadsheet."*
4. **Aksi Navigasi Lanjutan**:
   - Tombol Sekunder: `Unggah Berkas CSV Lain` (mereset wizard untuk impor baru).
   - Tombol Primer: `Lihat Data di Sistem →` (Menavigasi ke `/products`, `/customers`, atau `/transactions` sesuai jenis entitas yang diimpor).
5. **Mekanisme Pembatalan Penuh / Rollback (*Import Rollback Module*)**:
   - Area khusus berbatas pemisah halus di bagian dasar halaman untuk mengantisipasi kesalahan entri data massal.
   - Label: "Terjadi kesalahan pada data yang diimpor?"
   - Tombol: `Batalkan Seluruh Batch Impor Ini (Rollback)` (Varian Destruktif / Outline Merah).
   - **Dialog Konfirmasi Pembatalan Bertingkat (*Destructive Rollback Modal*)**:
     - Judul: "Konfirmasi Pembatalan Impor (Rollback)"
     - Peringatan keras: "Tindakan ini akan membatalkan status transaksi penjualan yang dibuat oleh batch impor ini (#IMP-20261018-001) dan memulihkan stok inventaris ke kondisi awal. Catatan audit pembatalan akan disimpan secara permanen."
     - Bidang Verifikasi Ketik Kata Kunci: Pengguna diwajibkan mengetik kata **`BATALKAN`** pada input teks untuk mengaktifkan tombol eksekusi.
     - Tombol: `Ya, Batalkan Batch Impor Ini` (Merah Solid `var(--color-error-bg)` + `var(--color-error-text)`).

---

## 3. Spesifikasi Tata Letak Responsif

Wizard impor CSV dirancang adaptif di seluruh rentang breakpoint standar Teman UMKM:

| Breakpoint | Lebar Layar | Adaptasi Tata Letak |
|---|---|---|
| **Mobile** | `390px` | - Stepper bar horizontal beralih ke indikator langkah ringkas bertumpuk (misal: "Langkah 2 dari 5: Pemetaan Kolom").<br>- Kartu jenis entitas bertumpuk 1 kolom vertikal.<br>- Tabel pemetaan beralih menjadi format kartu vertikal bertumpuk (*stacked mapping cards*): Target field di atas, dropdown kolom CSV di bawahnya.<br>- Pratinjau tabel dan tabel inspeksi validasi memiliki pembungkus scroll horizontal halus (*horizontal scroll wrapper*) dengan kolom nomor baris (#) tetap menempel (*sticky first column*).<br>- Tombol navigasi aksi menempati lebar penuh (100% *full-width stacked buttons*). |
| **Tablet** | `768px` | - Stepper header menampilkan lingkaran nomor langkah dan label ringkas.<br>- Kartu jenis data dan template berjejer 3 kolom proporsional.<br>- Tabel pemetaan 4 kolom standar dengan keterbacaan penuh.<br>- Area dropzone memiliki padding lapang `32px`. |
| **Desktop** | `1440px` | - Stepper bar navigasi penuh dengan deskripsi status tiap langkah.<br>- Layout split/side-by-side: Pengaturan format angka & tanggal di sisi kiri, pratinjau tabel 3 baris di sisi kanan.<br>- Tabel inspeksi baris galat memiliki filter segmented bar di kiri dan search bar di kanan.<br>- Lebar kontainer utama maksimal `1120px` berada di tengah (*centered*) untuk menjaga fokus pembacaan data. |

---

## 4. Keadaan Komponen (*Component States*) & Panduan Gerakan (*Motion*)

### 4.1 Keadaan Komponen (*States*)
- **Dropzone Area**:
  - `Default`: Border `2px dashed var(--color-border-default)`, latar `var(--color-bg-surface)`.
  - `Drag Over`: Border `2px dashed var(--color-primary-700)`, latar `var(--color-primary-50)`, ikon sedikit membesar (skala 1.05).
  - `Uploading / Reading`: Latar `var(--color-bg-subtle)`, indikator spinner berputar halus dengan teks "Membaca berkas CSV...".
  - `File Rejected`: Border merah `2px dashed var(--color-error-border)`, pesan error di bawahnya "Berkas ditolak: Ukuran melebihi batas 5 MB atau bukan format .csv".
- **Dropdown Pemetaan Kolom**:
  - `Unmapped Required`: Border `1px solid var(--color-error-border)`, ikon peringatan kecil di kanan.
  - `Auto-matched`: Badge hijau kecil di samping nama kolom, latar `var(--color-success-bg)`.
  - `Focus`: Outline `2px solid var(--color-border-focus)` tanpa offset bayangan.
- **Bilah Kemajuan Asinkron (*Progress Bar*)**:
  - Berdenyut lembut saat aktif (`animation: pulse 2s infinite`).
  - Berubah menjadi hijau solid saat status mencapai `completed`.

### 4.2 Spesifikasi Gerakan (*Motion Tokens*)
Selaras dengan filosofi desain yang tenang dan tidak berlebihan (*calm, purposeful motion*):
- **Perpindahan Langkah Wizard**:
  - Trigger: Klik tombol "Lanjut" atau "Kembali".
  - Properti: `opacity` (0 ke 1) dan `transform: translateY(8px ke 0)`.
  - Durasi: `250ms`.
  - Easing: `cubic-bezier(0.22, 1, 0.36, 1)`.
- **Ekspansi Baris Detail Galat (*Accordion / Expandable Row*)**:
  - Trigger: Klik baris dengan galat untuk melihat konteks data lengkap.
  - Properti: `opacity` dan `transform: scaleY`.
  - Durasi: `150ms`.
  - Easing: `ease-out`.
- **Dukungan Aksesibilitas Gerak Rendah (*Reduced Motion*)**:
  - Ketika `@media (prefers-reduced-motion: reduce)` aktif, seluruh transisi transformasi spasial dinonaktifkan; hanya menggunakan transisi `opacity` instan atau tanpa animasi.

---

## 5. Aksesibilitas (WCAG AA) & Keamanan Data (Security UX)

1. **Aksesibilitas Antarmuka**:
   - Seluruh elemen interaktif memiliki target sentuh minimal `44x44px`.
   - Rasio kontras teks memenuhi standar AA:
     - Teks utama (`var(--color-text-main)`: kontras `15.2:1` pada putih).
     - Teks label dan petunjuk (`var(--color-text-muted)`: kontras `5.8:1`).
     - Teks galat (`var(--color-error-text)`: kontras `6.1:1`).
   - Seluruh baris progres dan status perubahan dilingkupi atribut `role="status"` dan `aria-live="polite"` agar terbaca oleh *screen reader*.
   - Navigasi penuh menggunakan keyboard (Tab, Shift+Tab, Enter/Spasi, dan Esc untuk menutup dialog konfirmasi).
2. **Keamanan & Integritas Data (Security UX)**:
   - **Tidak Ada Penyimpanan Berkas Mentah**: Sistem tidak menyimpan berkas CSV fisik di storage publik. Berkas dibaca di memori/streaming, dimasukkan ke baris staging terisolasi per `business_id`, dan hanya nilai hash SHA-256 yang disimpan untuk deteksi duplikasi batch.
   - **Pencegahan Injeksi Formula Excel/CSV**: Seluruh nilai teks yang diawali karakter formula berbahaya (`=`, `+`, `-`, `@`) diekstrak murni sebagai teks mentah (*string literal*), dan saat diekspor kembali sebagai laporan galat otomatis diproteksi kutip tunggal (`'`).
   - **Audit Trail Pembatalan (*Rollback Audit*)**: Setiap aksi pembatalan batch dicatat dalam log audit mencakup `business_id`, `user_id`, `timestamp`, jumlah transaksi dibatalkan, dan alasan pembatalan.

---

## 6. Pemetaan Persyaratan Bisnis ke Layar (Feature to Screen Map)

| Kebutuhan TRD §8 / PRD §5.5 | Komponen / Bagian UI | Penanganan Spesifik |
|---|---|---|
| Impor berkas CSV maks 5 MB / 20.000 baris | Langkah 1: Dropzone | Validasi tipe MIME & ukuran berkas di sisi klien & server |
| Peringatan impor berulang (SHA-256) | Langkah 1: Banner Peringatan | Deteksi hash berkas identik dengan batch sebelumnya |
| Format angka `1.500,50` vs `1,500.50` | Langkah 2: Selector Format Angka | Penyesuaian parsing angka Indonesia vs Internasional |
| Format tanggal beragam | Langkah 2: Selector Format Tanggal | Pilihan `DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD` |
| Pratinjau interpretasi data | Langkah 2: Pratinjau 3 Baris Pertama | Tampilan dinamis hasil parse sebelum melangkah ke validasi |
| Resolusi angka ambigu | Langkah 3: Modul Resolusi Galat | Pengguna menentukan tafsir angka ambigu tanpa tebakan liar |
| Larangan auto-merge nama pelanggan serupa | Langkah 3: Modul Resolusi Pelanggan | Penawaran eksplisit: buat pelanggan baru vs tautkan ke ID terverifikasi |
| Eksekusi asinkron batch besar & idempotensi | Langkah 4: Polling Layar Progres | Bilah progres dinamis, counter baris, dan polling interval 1,5 detik |
| Netralisasi formula CSV pada laporan galat | Langkah 5: Tombol Unduh Laporan | Sanitasi sel berawalan `=`, `+`, `-`, `@` dengan kutip tunggal |
| Pembatalan transaksi impor massal | Langkah 5: Dialog Rollback Bertingkat | Pembatalan transaksi terkait + audit trail permanen |
