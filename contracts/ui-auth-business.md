# UI/UX Specification — Authentication & Business Setup

> Dokumen spesifikasi antarmuka dan interaksi pengguna untuk fitur Autentikasi (Registrasi, Login, Sesi Kedaluwarsa) dan Setup/Pengaturan Bisnis (Onboarding & Pengaturan Profil Usaha).
> Mengacu pada: `contracts/design-direction.md`, `contracts/design-tokens.md`, `project-map/prd.md` (§5.1, §6.1), `project-map/trd.md` (§5 TD-05, TD-06), dan `contracts/erd.md`.

---

## 1. Feature → Page Map

| Fitur | Rute URL | Akses / Guard | Tujuan Utama |
| :--- | :--- | :--- | :--- |
| **Registrasi** | `/register` | Publik (Anonymous only; jika sudah login dialihkan ke `/`) | Pendaftaran akun pemilik usaha baru (email, nama tampilan, kata sandi). |
| **Login** | `/login` | Publik (Anonymous only; jika sudah login dialihkan ke `/`) | Masuk ke akun menggunakan email & kata sandi dengan keamanan generik. |
| **Onboarding Bisnis** | `/onboarding` | Terautentikasi (Wajib bagi akun tanpa bisnis terasosiasi) | Setup profil usaha pertama (nama, kategori, zona waktu, periode buku). |
| **Pengaturan Bisnis** | `/settings` | Terautentikasi (Role: `owner` atau `staff` dengan pembatasan hak) | Pembaruan profil usaha, preferensi laporan, dan inspeksi identitas tenant. |
| **Sesi Kedaluwarsa** | Modal/Banner di rute aktif | Terautentikasi (Saat token kedaluwarsa / 401 Unauthorized) | Notifikasi pengalihan sesi aman tanpa kehilangan konteks halaman. |

---

## 2. Global Anti-Slop & UX Rules

1. **Resting Surfaces & Borders**: Tidak ada box-shadow pada kartu istirahat (*resting cards*). Gunakan kontras permukaan `var(--color-bg-surface)` dan border `1px solid var(--color-border-subtle)`. Shadow hanya berlaku pada dialog modal dan dropdown (`var(--shadow-modal)`, `var(--shadow-dropdown)`).
2. **Tanpa Gradien Dekoratif**: Seluruh latar belakang kanvas menggunakan warna datar `var(--color-bg-app)` (`#F7F9F8`). Tidak ada efek glow, ambient blur, atau glassmorphism.
3. **Penyampaian Pesan Keamanan (Security UX)**:
   - Kesalahan login menggunakan pesan generik: `"Email atau password salah."` Tidak boleh mengonfirmasi keberadaan email tertentu di sistem.
   - Kolom kata sandi wajib menyertakan toggle *Tampilkan/Sembunyikan Kata Sandi* dengan tombol berukuran minimal 44×44px.
   - Sesi habis (*Session Expired*) menampilkan peringatan transparan: `"Sesi Anda telah berakhir. Silakan masuk kembali."`
4. **Sentuhan & Aksesibilitas (Fitts & WCAG AA)**:
   - Target sentuh kontrol formulir dan tombol minimal 44×44px di mobile (390px).
   - Label formulir selalu terlihat (*visible labels*), dilarang mengandalkan placeholder sebagai label.
   - Rasio kontras teks minimal 4.5:1 terhadap latar belakang (teks utama: `#131F1C` dengan kontras 15.2:1; teks sekunder: `#4B5E59` dengan kontras 5.8:1).
5. **Hukum Hick & Tesler**:
   - Satu aksi primer per layar.
   - Nilai default cerdas: Zona waktu otomatis `Asia/Jakarta` (WIB), Mata Uang `IDR (Rp)`, Hari Awal Periode Laporan `Tanggal 1`.

---

## 3. Komponen Antarmuka Bersama

### 3.1. AuthCard (Kontainer Formulir Autentikasi)
- **Peran**: Membungkus formulir autentikasi `/login`, `/register`, dan `/onboarding`.
- **Dimensi**: Lebar maksimal `440px` (desktop), `100%` dengan padding tepi `var(--space-4)` (mobile).
- **Gaya**:
  - Background: `var(--color-bg-surface)`
  - Border: `1px solid var(--color-border-default)`
  - Border-radius: `var(--radius-lg)` (8px)
  - Box-shadow: `none` (*strict anti-slop*)
  - Padding dalam: `var(--space-8)` (32px) pada desktop/tablet, `var(--space-6)` (24px) pada mobile.

### 3.2. FormInput (Input Teks & Password)
- **Struktur**:
  1. Label: Teks tebal ukuran `var(--text-sm-size)`, warna `var(--color-text-main)`, `margin-bottom: var(--space-2)`. Label wajib memiliki atribut `for="id"`.
  2. Input Container: Tinggi `44px`, border `1px solid var(--color-border-default)`, background `var(--color-bg-surface)`, radius `var(--radius-md)` (6px), padding `0 var(--space-3)`.
  3. Action Icon (opsional): Tombol show/hide password dengan ukuran `44×44px` tepat di sisi kanan input.
  4. Helper Text / Error Text: Teks `var(--text-xs-size)`. Helper warna `var(--color-text-muted)`. Error warna `var(--color-error-text)` dengan ikon peringatan `14×14px`.
- **States**:
  - *Default*: Border `var(--color-border-default)`, teks `var(--color-text-main)`.
  - *Hover*: Border `var(--color-border-strong)`.
  - *Focus-visible*: Border `var(--color-border-focus)` (`#0F766E`), outline `2px solid var(--color-primary-100)`.
  - *Disabled*: Background `var(--color-bg-subtle)`, border `var(--color-border-subtle)`, teks `var(--color-text-disabled)`, cursor `not-allowed`.
  - *Validation Error*: Border `var(--color-error-icon)` (`#DC2626`), background `var(--color-error-bg)` (saat fokus).

### 3.3. SelectDropdown (Dropdown Kategori & Pengaturan)
- **Struktur**: Label terlihat + kontainer pilihan native `<select>` atau custom select beralas keyboard.
- **Tinggi**: `44px`, padding `0 var(--space-3)`, border `1px solid var(--color-border-default)`.
- **Focus state**: Outline ring `var(--color-border-focus)` dengan transisi `120ms`.

### 3.4. PrimaryButton & SecondaryButton
- **Primary**: Background `var(--color-primary-700)` (`#0F766E`), teks `#FFFFFF`, tinggi `44px`, radius `var(--radius-md)`.
  - *Hover*: Background `var(--color-primary-800)` (`#115E59`).
  - *Active*: `transform: scale(0.98)` (durasi `100ms`).
  - *Loading*: Teks disembunyikan/digantikan spinner SVG `18×18px` berwarna putih dengan label aksesibilitas `aria-label="Memproses..."`, `aria-busy="true"`.
  - *Disabled*: Background `var(--color-bg-subtle)`, border `1px solid var(--color-border-subtle)`, teks `var(--color-text-disabled)`, cursor `not-allowed`.
- **Secondary / Link**: Teks `var(--color-primary-700)`, tanpa background, padding `var(--space-2) var(--space-3)`, hover underline atau background `var(--color-bg-hover)`.

### 3.5. AlertBanner (Pesan Status Sistem)
- **Variasi**:
  - `error`: Background `var(--color-error-bg)`, border `1px solid var(--color-error-border)`, teks `var(--color-error-text)`.
  - `warning`: Background `var(--color-warning-bg)`, border `1px solid var(--color-warning-border)`, teks `var(--color-warning-text)`.
  - `success`: Background `var(--color-success-bg)`, border `1px solid var(--color-success-border)`, teks `var(--color-success-text)`.
- **Struktur**: Icon di sisi kiri (`16×16px`), teks pesan di kanan, radius `var(--radius-md)`, padding `var(--space-3) var(--space-4)`. Atribut aksesibilitas: `role="alert"`, `aria-live="polite"`.

---

## 4. Spesifikasi Halaman

### 4.1. Halaman Login (`/login`)

#### Metadata & Rangkuman
- **Tujuan Halaman**: Mengautentikasi pengguna terdaftar secara aman dan mengarahkan ke Dashboard atau rute tujuan sebelumnya (*intended destination*).
- **Pengguna & Tugas Utama**: Pemilik usaha atau staf operasional; memasukkan email dan kata sandi untuk mengakses dasbor bisnis.
- **Aksi Primer**: Tombol `"Masuk"` (`type="submit"`).
- **Aksi Sekunder**: Tautan `"Belum punya akun? Daftar"` mengarah ke `/register`.
- **Entri**: Akses langsung ke `/login` atau pengalihan otomatis saat mengakses rute terproteksi tanpa token.
- **Keluar**: Redirect ke `/` (jika sudah memiliki bisnis) atau `/onboarding` (jika belum memiliki bisnis), atau kembali ke query parameter `redirect_to`.
- **Data Ditampilkan**:
  - Judul: `"Masuk ke Teman UMKM"`
  - Subjudul: `"Kelola transaksi dan pantau kepastian laba usaha Anda."`
  - Kolom Email (tipe: `email`, autocomplete: `username`)
  - Kolom Kata Sandi (tipe: `password`, autocomplete: `current-password`) dengan toggle buka-tutup
  - AlertBanner generik (saat terjadi kegagalan autentikasi atau sesi berakhir)

#### Layout Responsif
- **Desktop (1440px)**:
  - Kanvas di tengah layar (*centered canvas*), background `var(--color-bg-app)`.
  - AuthCard selebar `440px` berada di titik tengah vertikal & horizontal (`min-height: 100vh`, flex center).
  - Logo aplikasi dan nama brand `"Teman UMKM"` di bagian atas kartu (`margin-bottom: var(--space-6)`).
- **Tablet (768px)**:
  - AuthCard selebar `440px`, margin horizontal otomatis, padding atas `var(--space-12)`.
- **Mobile (390px)**:
  - Tampilan penuh (*full width*). Margin luar `var(--space-4)`.
  - AuthCard memenuhi lebar layar dengan batas tepi `var(--space-4)`, padding dalam `var(--space-6)`.
  - Tombol `"Masuk"` memiliki lebar penuh (`width: 100%`, tinggi `44px`).

#### Keadaan (States)
1. **Default**:
   - Kolom email kosong, kolom kata sandi kosong, tombol `"Masuk"` aktif.
2. **Hover / Focus**:
   - Hover pada input mengubah border menjadi `var(--color-border-strong)`.
   - Focus pada input menampilkan ring `var(--color-border-focus)`.
3. **Loading**:
   - Tombol `"Masuk"` menampilkan spinner, atribut `disabled`, `aria-busy="true"`. Seluruh input dibuat `readonly` sementara.
4. **Validation Error (Client-side)**:
   - Email tidak valid formatnya: Teks bantuan merah di bawah kolom `"Format email tidak valid (contoh: nama@domain.com)"`.
   - Kata sandi kosong: Teks bantuan merah `"Kata sandi wajib diisi"`.
5. **Generic Auth Error (Server-side 401)**:
   - AlertBanner di atas kolom email muncul dengan teks: `"Email atau password salah."`
   - *Catatan Anti-Slop Keamanan*: Tidak ada pesan seperti "Email tidak terdaftar" atau "Kata sandi salah untuk email ini". Selalu gunakan pesan generik identik.
6. **Session-Expired State (Redirect dengan Flag `?reason=session_expired`)**:
   - AlertBanner kuning/info di atas formulir: `"Sesi Anda telah berakhir demi keamanan. Silakan masuk kembali."`

#### Motion Specs
- *Form Card Enter*: `Page mount → opacity: 0 to 1, transform: translateY(8px to 0) → 240ms → cubic-bezier(0.22, 1, 0.36, 1)`.
- *Error Banner Reveal*: `Submit failure → opacity: 0 to 1, transform: translateY(-4px to 0) → 180ms → cubic-bezier(0.22, 1, 0.36, 1)`.
- *Button Press*: `Mousedown / Touchstart → transform: scale(0.98) → 100ms → cubic-bezier(0.4, 0, 1, 1)`.

#### Aksesibilitas (A11y)
- Label eksplisit: `<label htmlFor="email">Email</label>` dan `<label htmlFor="password">Kata Sandi</label>`.
- Keyboard Tab Order: `Email` → `Kata Sandi` → `Toggle Password` → `Tombol Masuk` → `Tautan Daftar`.
- Atribut form: `aria-invalid="true"` disematkan pada input bila terjadi error, dengan `aria-describedby="email-error"`.

---

### 4.2. Halaman Registrasi (`/register`)

#### Metadata & Rangkuman
- **Tujuan Halaman**: Mendaftarkan identitas pemilik usaha baru untuk membuat akun sistem.
- **Pengguna & Tugas Utama**: Pemilik UMKM yang baru pertama kali bergabung; memasukkan nama lengkap/tampilan, email aktif, dan membuat kata sandi yang aman.
- **Aksi Primer**: Tombol `"Daftar Akun"` (`type="submit"`).
- **Aksi Sekunder**: Tautan `"Sudah memiliki akun? Masuk"` mengarah ke `/login`.
- **Entri**: Akses langsung ke `/register` atau tautan dari halaman login.
- **Keluar**: Redirect otomatis langsung menuju alur Onboarding Bisnis (`/onboarding`) setelah token akun berhasil diterbitkan.
- **Data Ditampilkan**:
  - Judul: `"Buat Akun Pemilik Usaha"`
  - Subjudul: `"Mulai pencatatan rapi dan analisis keuntungan usaha Anda."`
  - Kolom Nama Lengkap / Tampilan (tipe: `text`, autocomplete: `name`)
  - Kolom Email (tipe: `email`, autocomplete: `email`)
  - Kolom Kata Sandi (tipe: `password`, autocomplete: `new-password`)
  - Kolom Konfirmasi Kata Sandi (tipe: `password`, autocomplete: `new-password`)
  - Teks Petunjuk Kata Sandi: `"Minimal 8 karakter, mengandung huruf dan angka."`

#### Layout Responsif
- **Desktop (1440px)**:
  - AuthCard selebar `480px` di tengah layar (`min-height: 100vh`, flex center).
  - Padding dalam kartu `var(--space-8)` (32px).
- **Tablet (768px)**:
  - AuthCard selebar `460px`, margin horizontal otomatis.
- **Mobile (390px)**:
  - Lebar penuh dengan margin horizontal `var(--space-4)`, padding dalam `var(--space-6)`.
  - Jarak antar elemen formulir menggunakan `var(--space-4)` untuk kemudahan scrolling vertikal.

#### Keadaan (States)
1. **Default**:
   - Seluruh kolom kosong, tombol pendaftaran aktif.
2. **Validation Error (Client-side)**:
   - Nama kurang dari 2 karakter: `"Nama tampilan minimal 2 karakter."`
   - Email format salah: `"Format email tidak valid."`
   - Kata sandi < 8 karakter: `"Kata sandi minimal 8 karakter."`
   - Konfirmasi kata sandi tidak cocok: `"Konfirmasi kata sandi tidak cocok."`
3. **Conflict Error (Server-side 409)**:
   - Jika email telah terdaftar sebelumnya, tampilkan AlertBanner: `"Email ini sudah terdaftar. Silakan gunakan email lain atau masuk ke akun Anda."`
4. **Loading State**:
   - Tombol `"Daftar Akun"` menampilkan spinner, status `aria-busy="true"`.
5. **Success State**:
   - Transisi mulus menuju rute `/onboarding` tanpa jeda buatan.

#### Motion Specs
- *Field Error Shake / Appear*: `Validation blur → opacity: 0 to 1, transform: translateY(-2px to 0) → 120ms → cubic-bezier(0.2, 0, 0, 1)`.
- *Password Match Indicator*: `Input event → color change → 120ms → ease-in-out`.

#### Aksesibilitas (A11y)
- Petunjuk kata sandi terhubung via `aria-describedby="password-rules"`.
- Validasi instan dievaluasi saat *blur* (keluar dari kolom input) agar tidak mengganggu pengetikan pengguna.

---

### 4.3. Halaman / Modal Onboarding Bisnis (`/onboarding`)

#### Metadata & Rangkuman
- **Tujuan Halaman**: Membangun entitas usaha pertama (`business`) dan menetapkan kepemilikan tenant (`owner`) sebelum pengguna diarahkan ke dasbor utama.
- **Pengguna & Tugas Utama**: Pemilik akun yang baru terdaftar; mengisi identitas usaha, tipe bisnis, dan preferensi perhitungan.
- **Aksi Primer**: Tombol `"Simpan & Lanjutkan ke Dasbor"`.
- **Aksi Sekunder**: Tidak ada tombol skip/batal karena profil bisnis merupakan syarat mutlak isolasi tenant (TD-05).
- **Entri**: Pengalihan paksa (*guarded route*) jika akun login belum memiliki relasi dalam tabel `business_memberships`.
- **Keluar**: Redirect ke Dasbor Utama (`/`) dengan notifikasi selamat datang.
- **Data Ditampilkan**:
  - Judul: `"Setup Profil Usaha"`
  - Subjudul: `"Langkah awal untuk memulai isolasi data dan laporan finansial Anda."`
  - Kolom Nama Usaha (wajib, tipe: `text`, placeholder: `"Contoh: Kopi Seduh Santai"`)
  - Pilihan Jenis Usaha / Kategori (dropdown/radio group):
    - `Kuliner & F&B`
    - `Ritel & Reseller`
    - `Jasa & Servis`
    - `Produksi Sendiri / Manufaktur Ringan`
    - `Lainnya`
  - Pilihan Zona Waktu Usaha (default: `Asia/Jakarta (WIB)`, opsi: `Asia/Makassar (WITA)`, `Asia/Jayapura (WIT)`)
  - Mata Uang (tetap: `IDR (Rp)` - dikunci dengan teks pembantu `"Mata uang default MVP adalah Rupiah (IDR)"`)
  - Awal Periode Laporan Buku (pilihan angka 1-28, default: `1` - awal bulan kalender)

#### Layout Responsif
- **Desktop (1440px)**:
  - Form Wizard terstruktur lapang (*airy* sesuai Art Direction), lebar maksimal `640px` di tengah kanvas.
  - Border kartu `1px solid var(--color-border-subtle)`, background `var(--color-bg-surface)`.
  - Pengelompokan visual menggunakan jarak `var(--space-6)` (24px) antar blok data.
- **Tablet (768px)**:
  - Lebar kartu `600px`, padding `var(--space-6)`.
- **Mobile (390px)**:
  - Lebar penuh (`width: 100%`), padding layar `var(--space-4)`.
  - Tombol aksi primer melekat di bagian bawah formulir (*sticky bottom bar* atau tombol full width).

#### Keadaan (States)
1. **Default**:
   - Nama Usaha kosong (fokus otomatis ke kolom ini).
   - Jenis Usaha: Belum dipilih atau placeholder `"Pilih kategori usaha"`.
   - Zona Waktu: Terisi otomatis `Asia/Jakarta`.
   - Awal Periode: Terisi otomatis `1`.
2. **Validation Error (Client-side)**:
   - Nama Usaha kosong: `"Nama usaha wajib diisi."`
   - Kategori Usaha belum dipilih: `"Pilih salah satu kategori usaha."`
3. **Loading State**:
   - Tombol utama menampilkan spinner: `"Menyiapkan ruang kerja bisnis..."`, `aria-busy="true"`.
4. **Server Error (500 / 400)**:
   - AlertBanner: `"Gagal membuat profil bisnis. Silakan periksa kembali isian Anda atau coba sesaat lagi."`

#### Motion Specs
- *Card Transition*: `Route enter → opacity: 0 to 1, transform: translateY(12px to 0) → 300ms → cubic-bezier(0.22, 1, 0.36, 1)`.
- *Selection Radio/Button Change*: `Click → border-color & background-color → 120ms → cubic-bezier(0.2, 0, 0, 1)`.

#### Aksesibilitas (A11y)
- Seluruh opsi kategori dapat dinavigasikan menggunakan tombol panah keyboard (`ArrowDown` / `ArrowUp`).
- `aria-required="true"` disematkan pada Nama Usaha dan Jenis Usaha.

---

### 4.4. Halaman Pengaturan Bisnis (`/settings`)

#### Metadata & Rangkuman
- **Tujuan Halaman**: Meninjau dan memperbarui informasi identitas usaha, zona waktu operasional, dan parameter perhitungan periode laporan.
- **Pengguna & Tugas Utama**: Pemilik usaha (`role: owner`) yang ingin mengubah konfigurasi operasional, atau staf (`role: staff`) yang melihat konfigurasi dalam status baca-saja (*read-only*).
- **Aksi Primer**: Tombol `"Simpan Perubahan"` (aktif bila ada perubahan nilai input).
- **Aksi Sekunder**: Tombol `"Batal / Kembalikan"` (mengembalikan isian ke kondisi data tersimpan).
- **Entri**: Melalui menu navigasi utama Sidebar/TopBar (`Pengaturan`).
- **Keluar**: Tetap di halaman dengan status konfirmasi tersimpan, atau beralih ke tab navigasi lain.
- **Data Ditampilkan**:
  - Header: `"Pengaturan Usaha"` dengan deskripsi `"Konfigurasi profil bisnis dan preferensi operasional."`
  - Badge Status Peran: Menampilkan badge `Owner` atau `Staff` di sebelah nama bisnis.
  - Kartu Profil Bisnis:
    - ID Tenant / Bisnis: Teks UUID read-only dengan tombol satu-klik `"Salin ID"` (berguna untuk audit & bantuan teknis).
    - Kolom Nama Usaha (Input teks).
    - Kolom Kategori Usaha (Select dropdown).
    - Kolom Zona Waktu (Select dropdown: `Asia/Jakarta`, `Asia/Makassar`, `Asia/Jayapura`).
    - Kolom Tanggal Awal Siklus Laporan (Input angka / select 1 s.d. 28).
  - Seksi Keamanan & Akun Pengguna:
    - Informasi Email Login (read-only).
    - Tombol `"Keluar dari Sesi"` (Logout).

#### Layout Responsif
- **Desktop (1440px)**:
  - Terintegrasi dalam layout `AppShell` (Sidebar kiri lebar `260px`, area konten `max-width: 960px`).
  - Form diatur dalam grid 2-kolom untuk label dan input: Label & deskripsi di kolom kiri (lebar `320px`), kontrol input di kolom kanan (lebar `560px`).
  - Bar aksi simpan (*Action Bar*) berada di bawah form dengan penyelarasan kanan (*right-aligned*).
- **Tablet (768px)**:
  - Kolom vertikal bertumpuk (*stacked layout*). Label berada tepat di atas kontrol input.
  - Padding area kerja `var(--space-6)`.
- **Mobile (390px)**:
  - Tampilan bertumpuk 1 kolom penuh.
  - Action bar menempel di dasar layar (*sticky bottom container*) dengan border atas tipis `1px solid var(--color-border-subtle)` dan tombol `"Simpan Perubahan"` selebar layar.

#### Keadaan (States)
1. **Default (Unmodified)**:
   - Data bisnis terisi sesuai database.
   - Tombol `"Simpan Perubahan"` dalam status `disabled` (mencegah klik sia-sia tanpa mutasi data).
2. **Dirty (Modified)**:
   - Pengguna mengubah satu atau lebih kolom.
   - Tombol `"Simpan Perubahan"` beralih ke status `active` (hijau emerald `var(--color-primary-700)`).
   - Tombol `"Batal"` muncul di sebelah tombol simpan.
3. **Read-Only / Staff Permission**:
   - Jika pengguna memiliki `role: staff`, seluruh input dalam status `disabled` / `readonly`.
   - Tombol simpan ditiadakan, digantikan notifikasi info: `"Hanya pemilik usaha (owner) yang dapat memperbarui pengaturan profil bisnis."`
4. **Saving / Loading**:
   - Tombol simpan menampilkan spinner: `"Menyimpan..."`, `aria-busy="true"`.
5. **Success Toast**:
   - Toast notifikasi melayang di pojok kanan atas desktop atau bawah tengah mobile:
   - Background `var(--color-success-bg)`, teks `"Pengaturan usaha berhasil diperbarui."`, durasi tampil 4 detik.
6. **Validation / API Error**:
   - Error banner di atas kartu: `"Gagal memperbarui pengaturan. Pastikan nama usaha tidak kosong."`

#### Motion Specs
- *Action Bar Reveal*: `Form dirty → opacity: 0 to 1, transform: translateY(6px to 0) → 200ms → cubic-bezier(0.22, 1, 0.36, 1)`.
- *Toast Slide*: `Save success → opacity: 0 to 1, transform: translateY(8px to 0) → 240ms → cubic-bezier(0.22, 1, 0.36, 1)`.

#### Aksesibilitas (A11y)
- Atribut `disabled` pada tombol simpan disertai tooltip penjelasan saat pengguna hover jika form belum diubah.
- Notifikasi sukses menggunakan `role="status"` dan `aria-live="polite"`.

---

## 5. Security & Session UX Specification

### 5.1. Alur Penanganan Sesi Kedaluwarsa (*Session Expired*)
Mengacu pada TRD TD-06, access token berlaku selama 15 menit dan refresh token diperbarui otomatis melalui cookie httpOnly. Jika terjadi kegagalan refresh token (misal setelah 14 hari tidak aktif atau revokasi keluarga token):
1. **Intersepsi 401 Unauthorized**:
   - Lapisan jaringan frontend (`apiClient`) menangkap respon status `401`.
   - Menghentikan proses background dan menghapus status autentikasi lokal di memori.
2. **Penyimpanan URL Terakhir (*Preserve Return URL*)**:
   - Aplikasi menyimpan rute aktif saat itu ke dalam query string: `/login?redirect_to=${encodeURIComponent(currentPath)}&reason=session_expired`.
3. **Tampilan Halaman Login Pasca-Kedaluwarsa**:
   - Menampilkan AlertBanner ramah:
     > **Sesi Berakhir**
     > Sesi masuk Anda telah selesai. Silakan masukkan kata sandi kembali untuk melanjutkan pekerjaan Anda di halaman sebelumnya.
4. **Redirect Kembali Pasca-Login**:
   - Setelah login berhasil, pengguna langsung dikembalikan ke halaman asal tanpa kehilangan konteks.

### 5.2. Penanganan Kegagalan Login (Proteksi Enumerasi Akun)
- **Aturan**: Sistem tidak boleh memberikan indikasi apakah email terdaftar atau tidak pada endpoint `/auth/login`.
- **Tampilan Kesalahan UI**:
  - `401 Unauthorized` atau `400 Bad Request` selalu memicu tampilan:
    `"Email atau password salah."`
  - Tidak ada perbedaan visual maupun teks untuk kondisi email salah vs kata sandi salah.

---

## 6. Tabel Matriks Status Komponen Formulir (Component State Matrix)

| Komponen | Default | Hover | Focus-Visible | Active / Press | Disabled | Validation Error |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Input Teks / Password** | Border `#CBD5D1`, Bg `#FFFFFF` | Border `#94A39F` | Border `#0F766E`, Ring 2px `#CCFBF1` | Sesuai fokus | Bg `#EFF2F1`, Border `#E2E8E5`, Teks `#9EADA8` | Border `#DC2626`, Teks error `#991B1B` |
| **Tombol Primer** | Bg `#0F766E`, Teks `#FFFFFF` | Bg `#115E59` | Outline ring 2px `#0F766E`, offset 2px | `transform: scale(0.98)` | Bg `#EFF2F1`, Teks `#9EADA8`, Border `#E2E8E5` | N/A |
| **Tombol Sekunder** | Teks `#0F766E`, Bg transparan | Bg `#E6EBE9` | Outline ring 2px `#0F766E` | `transform: scale(0.98)` | Teks `#9EADA8`, Bg transparan | N/A |
| **Select Dropdown** | Border `#CBD5D1`, Ikon chevron | Border `#94A39F` | Border `#0F766E`, Ring 2px `#CCFBF1` | Sesuai fokus | Bg `#EFF2F1`, Ikon redup | Border `#DC2626` |
| **Checkbox / Radio** | Border `#CBD5D1`, Bg `#FFFFFF` | Border `#94A39F` | Ring 2px `#CCFBF1` | Skala mikro (0.95) | Bg `#EFF2F1`, Border `#E2E8E5` | Border `#DC2626` |

---

## 7. Verifikasi Kepatuhan Desain Anti-Slop & UX Laws

- [x] **Tanpa Bayangan Kartu Istirahat**: AuthCard dan Card Settings murni menggunakan batas tepi `1px solid var(--color-border-subtle)` / `var(--color-border-default)`.
- [x] **Tanpa Gradien Dekoratif**: Semua permukaan solid netral (`#F7F9F8` dan `#FFFFFF`).
- [x] **Bahasa Indonesia Lugas**: Bebas dari jargon kosong seperti "Elevate your business", "Seamless integration", atau "Smart AI Magic".
- [x] **Hick's Law**: 1 tombol aksi primer yang dominan di setiap layar auth dan form setup.
- [x] **Fitts's Law**: Seluruh input dan tombol aksi memenuhi target sentuh minimum 44×44px di mobile 390px.
- [x] **Jakob's Law**: Penataan layout login/register menggunakan konvensi web yang umum diakui pengguna (input email, password dengan toggle mata, tombol submit penuh).
- [x] **Doherty Threshold**: Respon visual tombol ditekan (*press feedback*) berjalan instan dalam 100ms.
- [x] **Tesler's Law**: Nilai default cerdas (WIB, IDR, Periode tanggal 1) menyerap kompleksitas konfigurasi awal pemilik UMKM.
- [x] **Aksesibilitas Kontras**: Memenuhi standar WCAG AA (rasio kontras teks utama 15.2:1, teks sekunder 5.8:1, warna interaktif primer 5.6:1).
