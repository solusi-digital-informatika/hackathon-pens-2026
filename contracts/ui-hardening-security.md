# Spesifikasi UI & Hardening Keamanan: Global Error Boundary, Offline UX, Session Timeout, & WCAG AA (M7)

## Metadata
- **ID Kontrak**: UI-M7-001
- **Task ID**: T-070
- **Penulis**: Designer Profile / Product Lead
- **Tanggal**: 19 Oktober 2026
- **Status**: APPROVED
- **Design System**: Clean Pragmatic Craftsman (`contracts/design-tokens.md`, `contracts/design-direction.md`)
- **Primary Color**: Emerald Forest (`#0D6E64` / `var(--color-primary-700)`)
- **Referensi Teknis**: TRD §8 (CSV Spec), TRD §9 (API Inventory), TRD §10 (Security & Rate Limits), `contracts/api-auth.md`, `contracts/api-csv-import.md`.

---

## 1. Filosofi Hardening Antarmuka & Keamanan Pragmatis

Menuju kesiapan rilis produksi (Milestone M7), antarmuka Teman UMKM dirancang dengan prinsip **keterandalan tanpa kompromi (*resilient by default*)** dan **perlindungan data non-intrusif**. Pemilik UMKM di lapangan sering kali menghadapi jaringan seluler tidak stabil, baterai perangkat menipis, atau sesi aplikasi yang terhenti di tengah pencatatan transaksi kasir.

1. **Integritas Formulir & Tanpa Kehilangan Data (*Zero Data Loss on Auth Failure*)**:
   - Jika sesi login kedaluwarsa saat pengguna sedang mengetik rincian transaksi atau mengonfigurasi katalog produk, sistem **tidak boleh** melakukan *hard reload* atau *redirect* paksa yang menghapus draf uncommitted pengguna.
   - Sesi dipulihkan lewat dialog re-autentikasi *inline* atau *background refresh*, mempertahankan seluruh *state* formulir di memori/storage lokal.
2. **Kerahasiaan Diagnostik (*Zero Stack Trace Leakage*)**:
   - Pengguna non-teknis tidak boleh dihadapkan pada galat sintaksis mentah, jejak stack trace server/database, query SQL, atau variabel internal yang membuka celah intelijen penyerang.
   - Layar galat menyediakan *Safe Incident Code* yang dapat disalin untuk tiket dukungan tanpa mengekspos rahasia sistem.
3. **Pemisahan Konteks Multi-Tenant yang Tegas (*Visual Tenant Isolation*)**:
   - Mencegah kekeliruan operator yang mengelola lebih dari satu entitas UMKM (misal toko cabang atau unit usaha terpisah). Header antarmuka selalu menyajikan identitas bisnis aktif dan peran pengguna secara persisten.
4. **Pencegahan Injeksi Formula Spreadsheet (*CSV DDE Neutralization*)**:
   - Data ekspor yang diunduh ke Excel atau Google Sheets diproteksi secara deterministik agar sel yang diawali karakter formula (`=`, `+`, `-`, `@`) tidak mengeksekusi perintah berbahaya.
5. **Kepatuhan Aksesibilitas Nyata (WCAG 2.1 AA Compliant)**:
   - Seluruh rasio kontras warna teks terhadap latar diverifikasi matematis (≥ 4.5:1 untuk teks normal, ≥ 3.0:1 untuk komponen UI grafis), navigasi papan ketik penuh, dan pembaca layar (*screen reader*) terintegrasi.

---

## 2. Audit Permukaan Akses & Sesi (Session & Multi-Tenant UX)

### 2.1 Siklus Sesi, Auto-Refresh Token, & Dialog Re-Autentikasi Non-Destruktif

```
[Request API 401 Unauthorized]
        │
        ├── 1. Interceptor HTTP menghentikan antrean request sementara
        │
        ├── 2. Coba Silent Refresh: POST /api/v1/auth/refresh (Cookie HttpOnly)
        │       ├── [Sukses (200 OK)] ──> Simpan Access Token Baru ──> Ulangi Request Tertunda (Seamless)
        │       └── [Gagal / Refresh Token Expired (401)]
        │               │
        │               └── 3. Buka Modal Re-Autentikasi Inline (Draft Form Tetap Utuh)
        │                       ├── [Pengguna Masukkan Password] ──> Verifikasi Sukses ──> Lanjutkan Form
        │                       └── [Pengguna Klik Batal / Keluar] ──> Simpan Draft ke LocalStorage ──> Arahkan ke /login
```

#### Komponen: Modal Re-Autentikasi Inline (`SessionTimeoutModal`)
- **Tujuan**: Memulihkan sesi pengguna yang kedaluwarsa tanpa mereset formulir yang sedang diisi (misal: kasir sedang memasukkan 12 item transaksi).
- **Triggers**: Respons HTTP `401 Unauthorized` pada endpoint bisnis saat silent refresh gagal atau token keluarga dicabut (`INVALID_REFRESH_TOKEN`).
- **Anatomi Visual**:
  - **Latar Belakang Backdrop**: `rgba(19, 31, 28, 0.6)` dengan `backdrop-filter: blur(4px)`.
  - **Kontainer Modal**: Lebar `420px`, latar `var(--color-bg-surface)` (`#FFFFFF`), border `1px solid var(--color-border-subtle)`, sudut `var(--radius-lg)` (`8px`), bayangan terangkat `var(--shadow-modal)`.
  - **Header Keamanan**:
    - Ikon gembok pelindung dalam lingkaran `36x36px` berlatar `var(--color-primary-50)` dengan warna ikon `var(--color-primary-700)`.
    - Judul (Heading 4, 18px / 600, `var(--color-text-main)`): **"Sesi Anda Telah Berakhir"**.
    - Deskripsi (Text Regular, 14px, `var(--color-text-muted)`): *"Untuk melindungi data usaha Anda, silakan masukkan kata sandi akun **budi@warunglestari.id** untuk melanjutkan tanpa kehilangan draf yang sedang diisi."*
  - **Field Input Kata Sandi**:
    - Label eksplisit: **"Kata Sandi"**.
    - Input tipe `password` dengan tombol aksi samping *Tampilkan/Sembunyikan Kata Sandi* (ikon mata SVG, label aksesibilitas `aria-label="Tampilkan kata sandi"`).
    - Status validasi inline jika password salah: Latar `#FEF2F2`, border `#FECACA`, teks `#991B1B` (*"Kata sandi salah. Silakan coba lagi."*).
  - **Action Bar**:
    - Tombol Sekunder (Teks netral, 14px): *"Batal & Simpan Draf ke Perangkat"*. Menyimpan isi formulir saat ini ke `localStorage` berenkripsi ringan berbasis timestamp, lalu mengarahkan pengguna ke `/login`.
    - Tombol Primer (`var(--color-primary-700)`, teks putih, 14px / 600): *"Masuk Kembali"*. Menampilkan spinner mini saat memverifikasi.
- **Aksesibilitas & Focus Trap**:
  - Fokus otomatis (*auto-focus*) diarahkan langsung ke input kata sandi saat modal terbuka.
  - Tombol keyboard `Tab` terkunci di dalam modal (*focus trap*).
  - Penekanan tombol `Escape` memicu konfirmasi pembatalan penyimpanan draf.
  - Atribut ARIA: `role="dialog"`, `aria-modal="true"`, `aria-labelledby="session-timeout-title"`, `aria-describedby="session-timeout-desc"`.

### 2.2 Indikator Status Multi-Tenant & Proteksi Visual Kebocoran Data

Dalam aplikasi multi-tenant, kesalahan memilih cabang bisnis dapat mengakibatkan salah input stok, transaksi, atau bocornya informasi finansial antar unit usaha.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo Teman UMKM]  [Kapsul Usaha: Warung Lestari Jaya ▾] [Role: Pemilik Usaha]         │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

#### 1. Kapsul Pemilih Entitas Bisnis Aktif (`TenantSelectorPill`)
- **Posisi**: Terpasang permanen di *Top Navigation Bar* sebelah kiri (tepat di samping logo aplikasi).
- **Anatomi Visual**:
  - Latar: `var(--color-bg-subtle)` (`#EFF2F1`), border `1px solid var(--color-border-default)` (`#CBD5D1`), padding `6px 12px`, radius `var(--radius-full)`.
  - Ikon Toko Fisik (Storefront SVG, `16x16px`, warna `var(--color-primary-700)`).
  - Nama Bisnis Aktif: Font 14px / 600, warna `var(--color-text-main)`, elipsis jika melebihi `200px` di desktop (`130px` di mobile).
  - Panah dropdown chevron bawah (`14x14px`, `var(--color-text-muted)`).
- **Menu Dropdown Pemilih Bisnis**:
  - Daftar seluruh bisnis yang diasosiasikan dengan akun pengguna (`GET /api/v1/auth/me`).
  - Item bisnis aktif ditandai dengan ikon centang zamrud dan label teks *"Aktif"*.
  - **Pemberitahuan Proteksi Pindah Tenant**:
    - Jika pengguna sedang berada di halaman yang memiliki formulir belum tersimpan (dirty state), pergantian bisnis memunculkan dialog konfirmasi:
      *"Anda sedang mengubah data di Warung Lestari Jaya. Berpindah ke Toko Cabang Dua akan membatalkan perubahan yang belum disimpan. Lanjutkan perpindahan?"*
    - Tindakan pindah tenant melakukan *clean slate*: cache query direset, URL diperbarui ke `/businesses/{new_business_id}/...`, dan state memori dibersihkan guna mencegah kontaminasi state antar bisnis.

#### 2. Badge Peran Pengguna Aktif (`UserRoleBadge`)
- **Posisi**: Menempel pada avatar / menu profil di sudut kanan atas antarmuka.
- **Peran Pemilik (`owner`)**:
  - Badge: Latar `var(--color-primary-50)` (`#F0FDF9`), border `1px solid var(--color-primary-200)` (`#99F6E4`), teks `var(--color-primary-800)` (`#115E59`, kontras 7.58:1).
  - Label: **"Pemilik Usaha"**.
- **Peran Staf / Kasir (`staff`)**:
  - Badge: Latar `#F1F5F9`, border `1px solid #CBD5E1`, teks `#334155` (kontras 7.12:1).
  - Label: **"Staf Operasional"**.
- **Perlakuan Menu Terbatas untuk Staf**:
  - Elemen navigasi yang memerlukan izin pemilik (misal `/settings/business` atau penghapusan data masal) diberi penanda gembok mini dan tooltip: *"Fitur ini hanya dapat dikonfigurasi oleh Pemilik Usaha (Owner)."*
  - Jika staf mencoba membuka URL langsung, tampil halaman `403 Forbidden` elegan dengan tombol kembali ke dasbor operasional.

### 2.3 Feedback Rate Limiting & Cooldown Protection (HTTP 429)

Sesuai TRD §10, endpoint seperti `POST /analysis/run` memiliki limit 1 request per 10 menit, dan endpoint auth dibatasi per IP.

- **Banner Pemberitahuan Rate Limit (`RateLimitAlert`)**:
  - Latar `#FFFBEB` (Amber-50), border `1px solid #FDE68A` (Amber-200), teks `#92400E` (Amber-800, kontras 6.84:1).
  - Ikon jam pasir / timer SVG `18x18px`.
  - Teks eksplisit: *"Batas frekuensi tercapai. Sistem sedang mengistirahatkan kalkulasi data. Silakan coba kembali dalam **08:42** menit."*
  - Tombol pemicu otomatis dinonaktifkan (`disabled`) dengan atribut `aria-disabled="true"` selama hitungan mundur berlangsung.

---

## 3. Penanganan Error Global & Network Degradation UX

### 3.1 Spesifikasi Komponen `GlobalErrorBoundary`

Komponen ini menangkap *unhandled runtime exception* di layer React tanpa menyebabkan layar putih kosong (*White Screen of Death*).

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                      [ Perisai Zamrud SVG ]                     │
│                                                                 │
│                Terjadi Kendala pada Halaman Ini                 │
│   Sistem kami telah mengamankan data Anda. Tidak ada data       │
│   transaksi yang terhapus akibat kendala teknis ini.            │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ Kode Insiden: ERR_M7_20261019_88F2A    [Salin Kode]     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│          [ Muat Ulang Halaman ]   [ Kembali ke Dasbor ]         │
│                                                                 │
│   Pusat Bantuan: Butuh bantuan mendesak? Hubungi Tim Dukungan   │
└─────────────────────────────────────────────────────────────────┘
```

#### Spesifikasi Elemen Fallback:
1. **Visual Pusat (Center Graphic)**:
   - Kontainer melingkar `64x64px` berlatar `var(--color-primary-50)` (`#F0FDF9`), border `1.5px solid var(--color-primary-200)` (`#99F6E4`).
   - Ikon perisai pelindung dengan tanda seru halus di dalam (*Shield Alert SVG*), warna `var(--color-primary-700)` (`#0F766E`).
   - **Aturan Anti-Slop**: Tidak ada ilustrasi robot menangis, kabel putus kartun, atau gambar 3D melayang.
2. **Tipografi & Pesan Penenang**:
   - Judul (Heading 2, 24px / 700, `var(--color-text-main)`): **"Terjadi Kendala pada Tampilan Ini"**.
   - Isi Pesan (14px / 400, `var(--color-text-muted)`, line-height 1.6):
     *"Aplikasi mengalami kendala tak terduga saat memproses tampilan. Seluruh data transaksi dan laporan keuangan Anda tetap aman tersimpan di peladen."*
3. **Penyembunyian Jejak Teknis (Anti-Leakage Security)**:
   - **DILARANG KERAS** menampilkan: `error.stack`, rincian komponen React, query GraphQL/SQL, token JWT, path direktori lokal server/klien, atau pesan mentah seperti `"TypeError: Cannot read properties of undefined (reading 'map')"`.
   - **Kotak Kode Tiket Insiden**:
     - Latar `var(--color-bg-subtle)` (`#EFF2F1`), border `1px solid var(--color-border-default)` (`#CBD5D1`), padding `8px 14px`, radius `var(--radius-sm)` (`4px`).
     - Teks format: `Kode Referensi: ERR_CLIENT_20261019_9B1C` (font monospaced Inter Tabular, 13px, `var(--color-text-main)`).
     - Tombol inline *"Salin Kode"* dengan umpan balik visual instan (*"Kode Tersalin!"* selama 2 detik).
4. **Tombol Tindakan Pemulihan**:
   - **Tombol Primer**: *"Muat Ulang Halaman"* (`var(--color-primary-700)`, teks putih). Memicu `window.location.reload()`.
   - **Tombol Sekunder**: *"Kembali ke Dasbor Utama"* (Latar putih, border `var(--color-border-default)`, teks `var(--color-text-main)`). Mengarahkan ke rute root bisnis `/businesses/{business_id}/dashboard`.
   - **Aksi Dukungan**: Tautan teks *"Laporkan Kendala ke WhatsApp Dukungan"*, membuka pesan WhatsApp terformat otomatis dengan template kode referensi dan timestamp tanpa rahasia pengguna.

### 3.2 Status Offline & Degradasi Jaringan (Network Disconnect UX)

Pengguna di toko fisik sering mengalami interupsi Wi-Fi atau kehilangan sinyal seluler saat cuaca buruk.

#### 1. Banner Status Offline Sticky (`OfflineBanner`)
- **Penempatan**: Menempel di bagian paling atas viewport (*sticky top*, `z-index: 1000`), mendorong konten utama ke bawah secara mulus sebesar `40px` (bukan menimpa header).
- **Triggers**: Event browser `window.addEventListener('offline')` atau kegagalan beruntun 3x HTTP probe heartbeat.
- **Anatomi Visual**:
  - Latar: `#1E293B` (Slate gelap berwibawa, bukan merah panik yang mencemaskan).
  - Border bawah: `1px solid #334155`.
  - Teks: Font 13px / 500, warna `#F8FAFC` (kontras 14.8:1).
  - Ikon: Sinyal nirkabel terputus (*Wifi Off SVG*, `16x16px`, warna `#94A3B8`).
  - Pesan teks: **"Koneksi internet terputus. Anda masih dapat melihat data yang telah dimuat. Tindakan penyimpanan baru dinonaktifkan sementara."**
  - Indikator Status: Dot pulsing kuning amber (`#F59E0B`, `8x8px`) di samping teks *"Mencoba menyambung kembali..."*.
- **Perilaku Tombol & Formulir Saat Offline**:
  - Seluruh tombol mutasi data (e.g. *"Simpan Transaksi"*, *"Tambah Produk"*, *"Mulai Analisis"*) otomatis beralih ke state `disabled` dengan atribut `aria-disabled="true"`.
  - Tooltip kontekstual pada tombol non-aktif: *"Koneksi terputus. Hubungkan kembali internet untuk menyimpan data."*
  - Data yang sedang diketik pada formulir **dilarang dihapus**; form tetap dapat diedit secara visual dan disimpan sementara di draf browser.
- **Pemulihan Koneksi (Reconnected Toast)**:
  - Saat event `online` terpicu dan probe heartbeat berhasil (`GET /api/v1/health` 200 OK):
  - Banner offline bertransisi mulus (slide up 200ms) dan digantikan oleh *Toast Pulih* di sudut kanan bawah:
    - Latar: `var(--color-success-bg)` (`#F0FDF4`), border `1px solid var(--color-success-border)` (`#BBF7D0`), teks `var(--color-success-text)` (`#166534`, kontras 6.81:1).
    - Ikon centang hijau: *"Koneksi pulih. Sistem siap digunakan kembali."*
    - Toast hilang otomatis setelah 3.5 detik.

### 3.3 Feedback Respons Lambat & Pencegahan Double Submit

#### 1. Pencegahan Double Submit pada Seluruh Aksi Mutasi
- Begitu tombol submit (misal *"Simpan Transaksi"* atau *"Konfirmasi Impor"*) diklik:
  1. Tombol seketika (`< 50ms`) beralih ke keadaan `disabled` untuk mencegah klik ganda yang dapat memicu duplikasi record transaksi di database.
  2. Label teks tombol digantikan oleh spinner ring melingkar `16x16px` (warna putih, border 2px, animasi putar `600ms linear infinite`) dan teks progres: *"Menyimpan..."*.
  3. Lebar tombol dipertahankan tetap (*fixed width* berbasis ukuran semula) agar tidak menimbulkan pergeseran tata letak (*layout shift / CLS = 0*).
  4. Tombol pembatalan di sebelahnya dinonaktifkan agar pengguna tidak menutup dialog di tengah proses penulisan basis data.

#### 2. Skeleton Loading Terpadu (> 400ms)
- Jika data memerlukan waktu pemuatan lebih dari 400ms (Hukum Doherty):
  - Dilarang menggunakan spinner layar penuh yang membingungkan layout.
  - Wajib menggunakan blok skeleton abu-abu netral (`#E2E8E5`) dengan animasi kilau halus (*shimmer wave*, durasi `1.5s ease-in-out infinite`).
  - Bentuk skeleton mencerminkan bentuk visual target:
    - Baris tabel: 5 baris skeleton persegi panjang tinggi `20px` dengan sudut radius `4px`.
    - Kartu ringkasan finansial: 3 kotak skeleton dengan tinggi `96px`.

#### 3. Rollback State pada Optimistic Update
- Pada aksi seperti *Tandai Rekomendasi Selesai* atau *Hapus Item Keranjang*:
  - Antarmuka memperbarui status secara optimistik seketika untuk responsivitas prima.
  - Jika server mengembalikan kegagalan (misal `500 Internal Server Error` atau timeout):
    - Antarmuka mengembalikan (*rollback*) elemen ke posisi dan status semula secara halus.
    - Menampilkan toast galat berwarna merah: *"Gagal memperbarui status. Perubahan telah dikembalikan. Silakan coba lagi."*.

---

## 4. Audit Ekspor & Formula Injection Protection UX

### 4.1 Dialog Konfirmasi Ekspor Aman (`SafeExportModal`)

Saat pengguna mengekspor laporan transaksi harian, katalog produk, atau ringkasan laba rugi ke dalam format CSV/Excel, sistem menyajikan modal konfirmasi terproteksi.

```
┌─────────────────────────────────────────────────────────────────┐
│ Unduh Data Transaksi (Ekspor Aman)                       [ × ] │
├─────────────────────────────────────────────────────────────────┤
│ Format Berkas:                                                  │
│ (●) CSV untuk Microsoft Excel (Pemisah Titik Koma ';') - Rekomendasi │
│ ( ) CSV Standar Internasional (Pemisah Koma ',')                │
│                                                                 │
│ [i] Proteksi Formula Spreadsheet Aktif:                         │
│ Seluruh karakter rentan (=, +, -, @) pada catatan transaksi     │
│ dan nama pelanggan telah dinetralkan dengan tanda kutip tunggal │
│ (') agar aman saat dibuka di Microsoft Excel atau Google Sheets. │
│                                                                 │
│ Rentang Data: 01 Okt 2026 - 19 Okt 2026 (142 Transaksi)        │
├─────────────────────────────────────────────────────────────────┤
│                       [ Batal ]   [ Unduh Berkas CSV (142 KB) ] │
└─────────────────────────────────────────────────────────────────┘
```

#### Spesifikasi Elemen Ekspor Aman:
1. **Opsi Pembatas Kolom (Delimiter Selector)**:
   - **Opsi 1 (Direkomendasikan di Indonesia)**: *"CSV untuk Microsoft Excel (Pemisah Titik Koma ';')"*. Ditandai badge mini hijau: *"Rekomendasi Excel Indonesia"*.
   - **Opsi 2**: *"CSV Standar Internasional (Pemisah Koma ',')"* untuk pengguna Google Sheets atau perkakas analisis data berbasis Python/R.
2. **Badge Penjaminan Keamanan Formula (Anti-Formula Injection Guarantee)**:
   - Kotak informasi edukatif berlatar `var(--color-info-bg)` (`#F0F9FF`), border `1px solid var(--color-info-border)` (`#BAE6FD`), teks `var(--color-info-text)` (`#075985`, kontras 7.09:1).
   - Ikon perisai centang info SVG `16x16px`.
   - Teks panduan transparan:
     *"Fitur Sanitasi Formula Aktif: Nilai teks yang diawali karakter formula (`=`, `+`, `-`, `@`) secara otomatis dinetralkan dengan awalan tanda kutip tunggal (`'`). Berkas aman dari eksekusi perintah makro berbahaya di komputer Anda."*
3. **Format Pengkodean Berkas (UTF-8 with BOM)**:
   - Berkas CSV disajikan dengan byte order mark `\uFEFF` di awal aliran berkas. Hal ini menjamin huruf non-ASCII, aksen, dan simbol Rupiah (`Rp`) terbuka sempurna tanpa karakter rusak (*garbled text / mojibake*) di Microsoft Excel versi apa pun.
4. **Indikator Progres Pengunduhan**:
   - Tombol unduh menampilkan spinner dan teks *"Menyiapkan Berkas..."* hingga browser memicu dialog simpan berkas asli dari OS.

---

## 5. Verifikasi Kontras, Aksesibilitas WCAG 2.1 AA & Dark Mode Readiness

### 5.1 Matriks Audit Kontras Warna Aksesibilitas (WCAG 2.1 AA)

Seluruh token warna dievaluasi menggunakan formula luminance standar WCAG 2.1:
$$\text{Contrast Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05}$$
Standar minimum WCAG 2.1 Level AA:
- **Teks Normal (< 18pt / < 14pt tebal)**: Minimal **4.5:1**
- **Teks Besar (≥ 18pt / ≥ 14pt tebal)**: Minimal **3.0:1**
- **Komponen Antarmuka Grafis & State Kontrol**: Minimal **3.0:1**

#### Tabel Hasil Verifikasi Kontras:

| Elemen / Token UI | Warna Teks / Foreground | Warna Latar / Background | Rasio Kontras Riil | Status WCAG AA (Teks) | Status WCAG AA (UI) | Catatan Desain |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Tombol Primer (Resting)** | `#FFFFFF` | `#0D6E64` (Emerald Forest) | **6.12:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Lulus level AA & AAA Large |
| **Tombol Primer (Hover)** | `#FFFFFF` | `#115E59` (Primary-800) | **7.58:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Kontras sangat tinggi & tegas |
| **Tombol Primer (Interactive)** | `#FFFFFF` | `#0F766E` (Primary-700) | **5.47:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Default interactive button |
| **Teks Utama Aplikasi** | `#131F1C` (Text Main) | `#FFFFFF` (Surface) | **16.93:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Lulus AAA maksimal |
| **Teks Sekunder / Label** | `#4B5E59` (Text Muted) | `#FFFFFF` (Surface) | **6.90:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Jelas & mudah dibaca kasir |
| **Badge Klaim: FAKTA** | `#065F46` (Emerald-800) | `#ECFDF5` (Emerald Subtle) | **7.29:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Bukti transaksi riil teruji |
| **Badge Klaim: ESTIMASI** | `#92400E` (Amber-800) | `#FFFBEB` (Amber Subtle) | **6.84:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Angka kalkulasi rata-rata |
| **Badge Klaim: HIPOTESIS**| `#5B21B6` (Indigo-800) | `#F5F3FF` (Indigo Subtle) | **8.19:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Deviasi pola belanja |
| **Status Bahaya / Error** | `#991B1B` (Red-800) | `#FEF2F2` (Red Subtle) | **7.60:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Notifikasi galat validasi |
| **Status Sukses** | `#166534` (Green-800) | `#F0FDF4` (Green Subtle) | **6.81:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Konfirmasi data tersimpan |
| **Status Info / Panduan** | `#075985` (Sky-800) | `#F0F9FF` (Sky Subtle) | **7.09:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | Edukasi keamanan ekspor |
| **Teks Tertiary Lama** | `#6D827C` | `#FFFFFF` | **4.09:1** | *GAGAL* (< 4.5:1) | **PASS** (≥ 3.0:1) | Hanya boleh untuk placeholder |
| **Teks Tertiary Hardened**| `#5C6F69` (Slate Dipertegas)| `#FFFFFF` | **5.34:1** | **PASS** (≥ 4.5:1) | **PASS** (≥ 3.0:1) | **Pembaruan Token Resmi M7** |
| **Focus Ring Kontrol** | `#0F766E` (Focus Ring) | `#FFFFFF` (Surface) | **5.47:1** | — | **PASS** (≥ 3.0:1) | Memenuhi kriteria SC 2.4.11 |

### 5.2 Audit Temuan & Penyesuaian Token (Design Token Update)

Dalam audit kontras terhadap token awal di `contracts/design-tokens.md`:
- Nilai `--color-text-tertiary: #6D827C` menghasilkan rasio **4.09:1** pada latar putih (`#FFFFFF`). Nilai ini memenuhi batas 3.0:1 untuk placeholder formulir yang tidak esensial, namun tidak memenuhi standar 4.5:1 jika dipakai untuk teks instruksi pendukung atau timestamp transaksi.
- **Rekomendasi & Penetapan Token Hardened**:
  ```css
  /* contracts/ui-hardening-security.md — Token Refinement */
  --color-text-tertiary:          #5C6F69; /* Rasio kontras 5.34:1 terhadap #FFFFFF (WCAG AA Compliant) */
  --color-placeholder:            #6D827C; /* Khusus teks bayangan input (placeholder non-kritis) */
  --color-focus-ring:             #0F766E; /* 2px solid offset 2px, rasio kontras 5.47:1 */
  ```

### 5.3 Aksesibilitas Navigasi Keyboard, Focus State, & Pembaca Layar

1. **Urutan Tabulasi Logis & Fokus Terlihat (*Visible Focus Indicator*)**:
   - Seluruh elemen interaktif (`button`, `a`, `input`, `select`, `textarea`, custom tabs) memiliki outline fokus tegas:
     `outline: 2px solid var(--color-primary-700); outline-offset: 2px;`
   - Dilarang keras menggunakan `outline: none` tanpa pengganti selektor visual yang setara.
2. **Pintasan Langsung Konten (*Skip to Main Content*)**:
   - Elemen pertama pada dokumen HTML:
     ```html
     <a href="#main-content" class="skip-link">Lewati ke Konten Utama</a>
     ```
   - Tersembunyi di luar layar (`top: -100px`) dan muncul di posisi `top: 12px; left: 12px; z-index: 9999` saat menerima fokus keyboard pertama kali.
3. **Komunikasi Dinamis Screen Reader (*ARIA Live Regions*)**:
   - Indikator koneksi offline/online menggunakan: `aria-live="polite"` dan `role="status"`.
   - Dialog session timeout dan pesan kegagalan mutasi menggunakan: `aria-live="assertive"` dan `role="alert"`.
   - Seluruh tombol hanya-ikon (e.g. ikon tutup modal, ganti password, salin kode) wajib memiliki atribut `aria-label` berbahasa Indonesia yang jelas.

### 5.4 Fondasi Kesiapan Mode Gelap (*Dark Mode Readiness Foundation*)

Meskipun antarmuka utama rilis MVP berfokus pada mode terang (*Light Mode*) bernuansa slate-sage, sistem token telah disiapkan secara semantik untuk mendukung mode gelap tanpa perombakan struktur komponen:

```css
/* Dark Mode Semantic Mapping Foundation (Future-Ready) */
@media (prefers-color-scheme: dark) {
  :root[data-theme="dark"] {
    /* Surfaces */
    --color-bg-app:        #0B1311; /* Deep slate emerald */
    --color-bg-surface:    #131F1C; /* Kontainer kartu */
    --color-bg-subtle:     #1A2B27; /* Baris zebra tabel */
    --color-bg-hover:      #223732; /* Hover state */
    
    /* Borders */
    --color-border-subtle:  #1F332E;
    --color-border-default: #2E4A43;
    --color-border-strong:  #4B7066;
    
    /* Typography */
    --color-text-main:      #ECF4F1; /* Kontras 13.8:1 terhadap #131F1C */
    --color-text-muted:     #9EADA8; /* Kontras 6.2:1 */
    --color-text-tertiary:  #7B918B; /* Kontras 4.6:1 */
    
    /* Primary Accent (Sedikit diangkat luminansinya untuk kontras gelap) */
    --color-primary-700:    #14B8A6; /* Emerald terang untuk tombol gelap */
    --color-primary-text:   #042F2E; /* Teks gelap di atas tombol terang */
  }
}
```

---

## 6. Matriks Spesifikasi Gerak (Motion Specs) & Anti-Slop Checklist

### 6.1 Spesifikasi Gerak Terstandar

| Komponen & Pemicu | Sifat yang Dianimasikan | Durasi | Easing Function | Fallback `prefers-reduced-motion` |
| :--- | :--- | :--- | :--- | :--- |
| **Offline Banner (Muncul)** | `transform: translateY(0)`, `opacity` | 240ms | `cubic-bezier(0.16, 1, 0.3, 1)` | `opacity` instan (0ms) |
| **Offline Banner (Selesai)**| `transform: translateY(-100%)`, `opacity` | 180ms | `cubic-bezier(0.7, 0, 0.84, 0)` | `opacity` instan (0ms) |
| **Modal Timeout (Enter)** | `transform: scale(0.96) -> scale(1)`, `opacity` | 200ms | `cubic-bezier(0.16, 1, 0.3, 1)` | `opacity` 100ms |
| **Toast Pulih (Slide In)** | `transform: translateY(12px) -> translateY(0)`, `opacity`| 220ms | `cubic-bezier(0.16, 1, 0.3, 1)` | `opacity` 100ms |
| **Spinner Submit Button** | `transform: rotate(0deg) -> rotate(360deg)` | 600ms | `linear infinite` | Teks statis *"Menyimpan..."* |
| **Shimmer Skeleton** | `background-position: -200% -> 200%` | 1500ms| `ease-in-out infinite` | Latar statis abu-abu `#E2E8E5` |

### 6.2 Anti-Slop Design Audit Checklist

- [x] **Tanpa Gradien Dekoratif**: Tidak ada gradien ungu/biru "AI glow", partikel berkilau, atau efek kaca berlebihan.
- [x] **Tanpa Bayangan pada Kartu Istirahat**: Kartu resting murni menggunakan batas border `1px solid var(--color-border-subtle)` dan diferensiasi warna latar.
- [x] **Tanpa Eyebrow Uppercase Berlebih**: Judul lugas tanpa label kecil huruf besar yang mengotori ritme baca.
- [x] **Maksimal 2 Baris Judul**: Seluruh headline halaman diuji tidak melebihi 2 baris pada desktop 1440px.
- [x] **Tanpa Emoji Pengganti Ikon**: Seluruh simbol menggunakan ikon SVG monokromatik terstandarisasi.
- [x] **Tanpa Teks Generik / Klise**: Seluruh salinan teks berbahasa Indonesia lugas, hangat, dan mudah dipahami pelaku UMKM (e.g. tidak menggunakan jargon abstrak seperti "Sinergikan Alur Kerja Anda").
- [x] **Penanganan State Kosong Berdaya Guna**: Error boundary dan offline state selalu menyertakan tindakan konkret (*Call to Action*).
- [x] **Satu Warna Aksen Tunggal**: Hanya menggunakan Emerald Forest (`#0D6E64` / `#0F766E`) sebagai penuntun visual aksi primer.

---

## 7. Pemetaan Komponen Antarmuka ke Kebutuhan Sistem

| Kebutuhan M7 | Nama Komponen UI | Rute / Penempatan | State yang Dikelola |
| :--- | :--- | :--- | :--- |
| **Error Handling Global** | `GlobalErrorBoundary` | Root App Wrapper (`<App />`) | Normal, Runtime Error, Incident Code Copied |
| **Konektivitas Jaringan** | `OfflineBanner` | Global Layout Header (Sticky) | Online, Offline, Reconnecting, Reconnected Toast |
| **Manajemen Sesi** | `SessionTimeoutModal` | Global Portal (`#modal-root`) | Hidden, Open, Verifying, Password Error, Draft Saved |
| **Isolasi Multi-Tenant** | `TenantSelectorPill` & `UserRoleBadge` | Main Top Navigation Bar | Active Tenant, Dropdown Open, Switching Warn, Role Badge |
| **Ekspor Aman Data** | `SafeExportModal` | Modul Transaksi, Produk, & Laporan | Format Choice, Formula Warning, Generating, Downloaded |
| **Rate Limiting UX** | `RateLimitAlert` | Header Halaman Analisis & Auth | Active Cooldown, Countdown Timer, Expired |
