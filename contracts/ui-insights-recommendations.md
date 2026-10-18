# Spesifikasi UI & Alur Interaksi: Insight Engine & Recommendations (M6)

## Metadata
- **ID Kontrak**: UI-M6-001
- **Task ID**: T-060
- **Penulis**: Designer Profile / Product Lead
- **Tanggal**: 18 Oktober 2026
- **Status**: APPROVED
- **Design System**: Clean Pragmatic Craftsman (`contracts/design-tokens.md`)
- **Primary Color**: Emerald Forest (`#0D6E64` / `var(--color-primary-700)`)
- **Referensi Teknis**: TRD §7 (Insight Engine Spec), TRD §9 (API Inventory), TRD §10 (Security & Rate Limits), ERD §4 (Diagram C: Customer & Intelligence).

---

## 1. Prinsip Desain & Filosofi "Evidence-First Intelligence"

Modul **Insight Engine & Recommendations** (`/insights`) dirancang khusus untuk pemilik usaha UMKM Indonesia agar dapat mengambil keputusan bisnis preventif tanpa rasa bingung, cemas, atau terjebak dalam halusinasi angka. Sesuai `contracts/design-direction.md`:

1. **Kejujuran Klaim & Epistemik Status (Claim-Type Integrity)**:
   - Setiap wawasan memiliki status klaim eksplisit: **FAKTA** (fakta riil dari data tersimpan), **ESTIMASI** (perhitungan matematis/rata-rata dengan asumsi tertentu), atau **HIPOTESIS** (dugaan berdasarkan deviasi pola historis).
   - Dilarang keras menggunakan klaim kepastian pada jenis hipotesis (misal: dilarang menulis *"Pelanggan ini pasti tidak akan kembali"*; wajib diganti dengan *"Pelanggan melampaui interval belanja biasanya"*).
2. **Transparansi Bukti (*Provenance & Traceability*)**:
   - Pemilik UMKM tidak diminta percaya begitu saja pada angka ringkasan. Setiap kartu wawasan memiliki tautan ke sumber data pembentuk (*insight sources*): nomor faktur transaksi, tanggal komplain, atau versi HPP produk.
3. **Aksen Tunggal & Anti-Slop UI**:
   - Aksen visual utama tetap *Emerald Forest* (`#0D6E64`).
   - Tidak ada gradien dekoratif ungu/biru "AI glow", partikel melayang, atau ikon emoji. Seluruh visual menggunakan kontras permukaan, garis border presisi, dan ikon SVG fungsional monokrom.
   - Tidak ada resting shadows pada kartu wawasan. Bayangan hanya digunakan pada modal layer terangkat (`var(--shadow-modal)`).
4. **Pencegahan Spam & Kelelahan Keputusan (*Actionable & Cooldown*)**:
   - Wawasan yang diabaikan (*dismissed*) memiliki periode jeda (*cooldown*) 14 hari agar tidak membanjiri feed.
   - Pemicuan analisis manual dibatasi kuota 1 kali per 10 menit per entitas bisnis (TRD §10 Rate Limiting) dengan penghitung mundur (*cooldown timer*) transparan di antarmuka.

---

## 2. Desain Token Khusus & Semantic Badge System

### 2.1 Claim-Type Badges (Label Kejujuran Data)
Setiap kartu wawasan wajib menampilkan badge jenis klaim di sudut atas kartu. Seluruh kombinasi warna memenuhi standar kontras WCAG AA (minimal 4.5:1 untuk teks terhadap latar).

```css
/* Token Khusus Claim-Type Badges (contracts/design-tokens.md compliant) */
:root {
  /* 1. FAKTA: Kepastian data transaksi riil (misal komplain > 3 hari, harga < HPP) */
  --badge-claim-fact-bg:     #ECFDF5; /* Emerald subtle */
  --badge-claim-fact-border: #A7F3D0; /* Emerald-200 */
  --badge-claim-fact-text:   #065F46; /* Emerald-800, Kontras: 5.8:1 */

  /* 2. ESTIMASI: Kalkulasi dengan rata-rata/agregasi (misal margin < rata-rata bisnis) */
  --badge-claim-est-bg:      #FFFBEB; /* Amber subtle */
  --badge-claim-est-border:  #FDE68A; /* Amber-200 */
  --badge-claim-est-text:    #92400E; /* Amber-800, Kontras: 5.1:1 */

  /* 3. HIPOTESIS: Pola probabilistik/deviasi (misal keterlambatan beli / overdue) */
  --badge-claim-hypo-bg:     #F5F3FF; /* Indigo-slate subtle */
  --badge-claim-hypo-border: #DDD6FE; /* Indigo-200 */
  --badge-claim-hypo-text:   #5B21B6; /* Indigo-800, Kontras: 6.2:1 */
}
```

| Claim Type | Kode TRD §7 | Latar (`bg`) | Border | Teks (`text`) | Arti & Perlakuan Visual |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FAKTA** | `fact` | `#ECFDF5` | `#A7F3D0` | `#065F46` | Data riil faktual dari baris transaksi/komplain terdaftar. Ikon centang dokumen. |
| **ESTIMASI** | `estimate` | `#FFFBEB` | `#FDE68A` | `#92400E` | Nilai dihitung dari perbandingan rerata periode tertentu. Ikon kalkulasi persentase. |
| **HIPOTESIS** | `hypothesis` | `#F5F3FF` | `#DDD6FE` | `#5B21B6` | Deviasi frekuensi atau kebiasaan belanja pelanggan. Ikon kalender interval. |

### 2.2 Priority Badges (Tingkat Urgensi)
Sesuai TRD §7, prioritas dihitung berdasarkan urgensi aturan ditambah *boost* 1 tingkat jika pelanggan masuk dalam kategori **Top 20% Revenue** 180 hari terakhir:

- **Prioritas Tinggi (`high`)**:
  - Badge: Latar `#FEF2F2`, Border `#FECACA`, Teks `#991B1B` (Kontras: 6.1:1).
  - Indikator: Titik aksen merah `6px` bulat di sebelah teks "Prioritas Tinggi".
- **Prioritas Sedang (`medium`)**:
  - Badge: Latar `#FFFBEB`, Border `#FDE68A`, Teks `#854D0E` (Kontras: 4.8:1).
  - Indikator: Titik aksen oranye `6px` bulat di sebelah teks "Prioritas Sedang".
- **Prioritas Rendah (`low`)**:
  - Badge: Latar `#EFF2F1`, Border `#CBD5D1`, Teks `#4B5E59` (Kontras: 5.8:1).
  - Indikator: Titik aksen abu-abu slate `6px` bulat di sebelah teks "Prioritas Rendah".

### 2.3 Status Siklus Hidup Rekomendasi (`recommended_actions.status`)
- `new` (**Baru**): Tag border biru/info netral (`#F0F9FF` / `#BAE6FD` / `#075985`).
- `reviewed` (**Ditinjau**): Tag netral (`#EFF2F1` / `#CBD5D1` / `#4B5E59`).
- `in_progress` (**Sedang Berjalan**): Tag aktif aksen Emerald (`#F0FDF9` / `#99F6E4` / `#0F766E`).
- `completed` (**Selesai**): Tag sukses hijau zamrud (`#F0FDF4` / `#BBF7D0` / `#166534`).
- `dismissed` (**Diabaikan**): Tag abu-abu muted redup (`#F7F9F8` / `#E2E8E5` / `#6D827C`).
- `superseded` (**Kondisi Sudah Berubah**): Tag peringatan halus dengan catatan kondisi teratasi secara otomatis oleh transaksi baru.

---

## 3. Struktur Layar Utama: Insight Feed & Overview Layout (`/insights`)

### 3.1 Header & Kontrol Analisis Data
- **Judul Halaman**: "Wawasan & Rekomendasi Bisnis"
- **Subjudul**: "Deteksi proaktif berbasis bukti data untuk mengamankan margin keuntungan dan hubungan pelanggan."
- **Blok Tombol Pemicu Analisis Manual (`POST /analysis/run`)**:
  - **Kondisi Normal / Idle**:
    - Tombol Sekunder: `↻ Jalankan Analisis Data Terbaru`
    - Teks bantuan halus di bawah tombol: "Terakhir dihitung: Hari ini, 08:30 WIB (Otomatis)".
  - **Kondisi Sedang Berjalan (`running`)**:
    - Tombol menampilkan *spinner loader* berputar (16px), label teks: "Memeriksa aturan bisnis...", keadaan *disabled*.
    - Banner proses di atas feed: "Analisis sedang mengevaluasi transaksi dan perilaku pelanggan... (Estimasi < 15 detik)".
  - **Kondisi Cooldown (< 10 menit)**:
    - Tombol disabled berlatar `var(--color-bg-subtle)`, border `var(--color-border-subtle)`, teks `var(--color-text-disabled)`.
    - Label tombol: `↻ Jalankan Analisis (Tunggu 07:42)`.
    - Tooltip saat hover: "Pemicuan manual dibatasi sekali per 10 menit untuk menjaga performa sistem."

### 3.2 Tab Kategori & Filter Feed
Navigasi filter diletakkan horizontal di atas feed kartu wawasan:

1. **Tab Kategori (`insight_type`)**:
   - `Semua Wawasan` [Badge total wawasan aktif, misal `8`]
   - `Pelanggan` (`customer_activity`) [Badge misal `4`] — Aturan `CUST_OVERDUE_V1`, `CUST_FREQ_DROP_V1`, `CUST_VALUE_DROP_V1`, `CUST_COMPLAINT_OPEN_V1`.
   - `Produk & Margin` (`product_profitability`) [Badge misal `2`] — Aturan `PROD_BELOW_COST_V1`, `PROD_LOW_MARGIN_V1`.
   - `Kesehatan Data` (`data_health`) [Badge misal `2`] — Aturan `DATA_MISSING_COST_V1`, `DATA_UNLINKED_TX_V1`.
2. **Filter Bar Sekunder (Dropdowns)**:
   - **Filter Klaim**: `Semua Klaim` | `Fakta Saja` | `Estimasi` | `Hipotesis`.
   - **Filter Prioritas**: `Semua Prioritas` | `Tinggi` | `Sedang` | `Rendah`.
   - **Filter Status Rekomendasi**: `Perlu Tindakan` (default: `new`, `reviewed`, `in_progress`) | `Selesai` | `Diabaikan`.

---

## 4. Anatomi Kartu Wawasan (*Insight Card Anatomy*)

Setiap wawasan disajikan dalam kartu datar (*flat surface card*) berlatar putih/netral berbingkai `1px solid var(--color-border-subtle)` dengan pembagian 3 area visual tegas:

```
+-----------------------------------------------------------------------------------+
| [Kategori: Pelanggan]   [HIPOTESIS]   [Prioritas Tinggi]     Dihitung: 18 Okt 14:15|
+-----------------------------------------------------------------------------------+
| Toko Berkah Mandiri melampaui interval belanja biasanya                           |
| Berdasarkan 8 transaksi terakhir, pelanggan biasanya berbelanja setiap 12 hari.  |
| Hingga hari ini sudah 28 hari sejak transaksi terakhir (terlambat 16 hari).       |
|                                                                                   |
| [Metrik Snapshot: Terakhir 20 Sep 2026]  [Nilai Historis: Rp 450.000 / order]     |
| [Top 20% Kontributor Omzet - Pelanggan Inti]                                      |
+-----------------------------------------------------------------------------------+
| [Perkiraan Dampak: Pemulihan omzet belanja berkala ~Rp 900.000 / bln]             |
+-----------------------------------------------------------------------------------+
| [ Tombol: Hubungi via WhatsApp ]  [ Lihat Bukti & Riwayat ]        [ Abaikan... ]|
+-----------------------------------------------------------------------------------+
```

### 4.1 Bagian Header Kartu
- **Kiri**:
  - Kapsul Kategori: Teks kecil tebal `text-xs` (misal: "PELANGGAN", "PRODUK & MARGIN", "KESEHATAN DATA").
  - Claim Type Badge: `FAKTA` / `ESTIMASI` / `HIPOTESIS` (menggunakan token di §2.1).
  - Priority Badge: `Tinggi` / `Sedang` / `Rendah` (menggunakan token di §2.2).
- **Kanan**:
  - Waktu Kalkulasi: "Dihitung 18/10/2026 14:15" (`var(--color-text-tertiary)`, ukuran `12px`).

### 4.2 Bagian Isi / Body Kartu
- **Judul Wawasan (`title`)**:
  - Ukuran `18px` (`--text-md-size`), semibold, warna `var(--color-text-main)`. Maksimal 2 baris pada tampilan desktop.
  - Bahasa lugas, netral, tidak menuduh atau berspekulasi berlebihan.
- **Ringkasan Penjelasan (`explanation`)**:
  - Dihasilkan dari template deterministik (TRD §7), ukuran `14px` (`--text-sm-size`), tinggi baris `1.5`, warna `var(--color-text-muted)`.
  - Contoh template:
    - *Aturan `CUST_OVERDUE_V1`*: "Berdasarkan [N] transaksi pada periode [T], pelanggan rata-rata berbelanja tiap [X] hari. Saat ini sudah [Y] hari sejak pembelian terakhir."
    - *Aturan `PROD_BELOW_COST_V1`*: "Harga jual [Produk] saat ini Rp [X] berada di bawah HPP aktif Rp [Y] (selisih rugi Rp [Z] per unit)."
    - *Aturan `DATA_MISSING_COST_V1`*: "[N] produk terjual dalam 30 hari terakhir belum memiliki data HPP tercatat, menyebabkan nilai laba kotor berstatus estimasi."
- **Grid Metrik Kunci (*Evidence Snapshot Mini-Grid*)**:
  - Kotak metrik berlatar `var(--color-bg-app)` dengan border `1px solid var(--color-border-subtle)` berisi 2-3 kotak metrik:
    - Nilai Saat Ini vs Ambang Batas (misal: `28 Hari Sejak Belanja` vs `Ambang Batas: 19 Hari`).
    - Nominal Rata-rata / Volume Transaksi (misal: `Rp 450.000 / transaksi`, `Total 6 pesanan`).
- **Badge Label Tambahan (Jika Berlaku)**:
  - Pelanggan Inti: Badge khusus bertuliskan `Top 20% Kontributor Omzet Usaha` (Latar emerald tipis `#ECFDF5`, teks `#065F46`).

### 4.3 Bagian Footer Kartu (Action Bar)
- **Tombol Aksi Utama (Rekomendasi Langsung)**:
  - Tombol Primer Emerald: Misal `Hubungi Pelanggan via WhatsApp`, `Perbarui HPP Produk`, `Tinjau Komplain`, `Lengkapi HPP`.
  - Mengubah status rekomendasi menjadi `in_progress` saat diklik dan membuka aksi eksternal/internal.
- **Tombol Aksi Sekunder**:
  - Tombol Outline Netral: `Lihat Bukti & Detail →` (Membuka modal provenance data lengkap).
- **Tombol Sekunder Tambahan**:
  - Tombol Teks Redup di sisi kanan: `Abaikan` (Membuka modal konfirmasi dismiss dengan opsi alasan).

### 4.4 Kartu Kondisi Berubah (*Superseded Card State*)
Jika kondisi data berubah (misal pelanggan yang sempat terlambat tiba-tiba melakukan transaksi pembelian baru):
- Kartu wawasan diberi banner atas berwarna abu-abu netral:
  - "Kondisi wawasan telah teratasi secara otomatis oleh transaksi baru pada [Tanggal]."
- Tombol aksi utama digantikan oleh status badge hijau: `Terselesaikan Otomatis`.
- Tombol `Arsipkan` untuk membersihkan feed.

---

## 5. Siklus Hidup Rekomendasi (*Actionable Recommendations Lifecycle*) & Pratinjau Dampak

### 5.1 Diagram State Machine
Rekomendasi mengikuti alur hidup deterministik yang tersinkronisasi dengan entitas basis data `recommended_actions`:

```
           [ Baru (new) ]
                 │
                 ├───> [ Ditinjau (reviewed) ]
                 │            │
                 │            ▼
                 └───> [ Sedang Berjalan (in_progress) ]
                              │
                              ├───> [ Selesai (completed) ]
                              │
   [ Setiap State ] ─────────┼───> [ Diabaikan (dismissed) ] ── (Cooldown 14 Hari)
                              │
   [ Kondisi Selesai ] ───────└───> [ Kondisi Berubah (superseded) ]
```

1. **Baru (`new`)**: Rekomendasi baru dibuat oleh mesin analisis. Belum disentuh pengguna.
2. **Ditinjau (`reviewed`)**: Pengguna membuka modal detail dan melihat bukti data, tetapi belum memulai eksekusi aksi.
3. **Sedang Berjalan (`in_progress`)**: Pengguna mengklik tombol aksi (misal menekan "Hubungi Pelanggan" yang membuka WhatsApp web/aplikasi atau "Perbarui HPP").
4. **Selesai (`completed`)**: Pengguna menandai tindakan telah selesai dijalankan secara manual, atau sistem mendeteksi penyelesaian (misal HPP produk telah diisi).
5. **Diabaikan (`dismissed`)**: Pengguna memilih untuk tidak menjalankan rekomendasi dengan alasan tertentu. Sistem mencatat waktu dan `dismissed_reason`. **Aturan pencegahan:** Sistem dilarang membuat ulang rekomendasi untuk `dedupe_key` yang sama selama **14 hari (cooldown)**.
6. **Kondisi Berubah (`superseded`)**: Kondisi yang memicu aturan sudah tidak berlaku lagi sebelum pengguna menyelesaikan aksi (misal pelanggan datang sendiri dan bertransaksi).

### 5.2 Komponen Pratinjau Dampak (*Impact Preview Box*)
Diletakkan di atas tombol aksi pada kartu wawasan dan di dalam modal detail:
- **Prinsip Kejujuran Finansial**: Dilarang menjanjikan angka pasti ("Pasti untung Rp 1.000.000"). Wajib menggunakan bahasa estimasi realistis:
  - *Untuk Pelanggan Overdue*: "Perkiraan potensi omzet belanja terlindungi: **~Rp 450.000 / siklus transaksi** (berdasarkan rerata riwayat pembelian pelanggan)."
  - *Untuk Harga di Bawah HPP*: "Pencegahan akumulasi kerugian: **Rp 12.500 per unit terjual** segera setelah harga disesuaikan."
  - *Untuk Komplain Menggantung*: "Mencegah risiko kehilangan pelanggan kategori Top 20% dengan nilai transaksi tahunan **Rp 6.200.000**."
- **Warna Kontainer**:
  - Latar: `var(--color-bg-app)` (`#F7F9F8`).
  - Border Kiri: `3px solid var(--color-primary-700)` (`#0F766E`).
  - Padding: `12px 16px`.

### 5.3 Dialog Konfirmasi Abaikan (*Dismiss Recommendation Modal*)
Ketika pengguna menekan tombol `Abaikan`:
- **Modal Header**: "Abaikan Rekomendasi Ini?"
- **Deskripsi**: "Wawasan ini akan disembunyikan dari daftar utama dan tidak akan dimunculkan kembali selama 14 hari ke depan."
- **Pilihan Alasan Pengabaian (`dismissed_reason`)** (Pilihan Radio Card):
  - [ ] `Sudah dihubungi / diselesaikan di luar sistem aplikasi`
  - [ ] `Pelanggan memiliki jadwal/musim belanja khusus`
  - [ ] `Produk sedang dalam periode promo khusus / cuci gudang`
  - [ ] `Data belum akurat / butuh penyesuaian catatan`
  - [ ] `Lainnya` (disertai input catatan opsional, maks 150 karakter)
- **Tombol Aksi**:
  - Tombol Sekunder: `Kembali`
  - Tombol Primer: `Konfirmasi Abaikan` (Mengirim `PATCH /recommendations/{id}` dengan payload `{ status: "dismissed", dismissed_reason: "..." }`).

---

## 6. Modal Detail Wawasan & Transparansi Bukti (*Insight Detail & Data Provenance Modal*)

Ketika pengguna mengklik `Lihat Bukti & Detail`, modal berukuran lebar terbuka (`max-width: 720px`) berposisi tengah (*centered dialog*).

### 6.1 Struktur Header Modal
- **Judul**: Judul lengkap wawasan (misal: "Evaluasi Frekuensi Belanja: Toko Berkah Mandiri").
- **Kapsul Metadata Teknis**:
  - Rule ID & Versi: `Kode Aturan: CUST_OVERDUE_V1` (monospace font Inter).
  - Status Klaim: Badge `HIPOTESIS`.
  - Periode Observasi Data: `Periode Analisis: 1 Mei 2026 - 18 Okt 2026 (180 hari)`.
  - Kualitas Data: `Data Cukup (8 Transaksi Memenuhi Syarat Data Gate)`.
- **Tombol Tutup**: Ikon silang (`×`) di sudut kanan atas atau tombol keyboard `Esc`.

### 6.2 Seksi Penjelasan Deterministik & Asumsi Algoritma
- **Narasi Lengkap Penjelasan**: Teks komprehensif yang menjabarkan metode kalkulasi.
- **Daftar Asumsi (`assumptions`)**:
  - Format bullet points ringkas:
    - *Asumsi 1*: Interval belanja dihitung dari nilai tengah (*median*) tanggal transaksi untuk mengurangi distorsi belanja mendadak.
    - *Asumsi 2*: Ambang batas keterlambatan ditetapkan pada toleransi `max(1.5 × median, median + 7 hari)`.
- **Daftar Limitasi Data (`limitations`)**:
  - Format bullet points ringkas:
    - *Limitasi 1*: Sistem tidak memantau interaksi atau transaksi pelanggan di luar catatan aplikasi kasir Teman UMKM.
    - *Limitasi 2*: Wawasan ini merupakan analisis pola masa lalu dan bukan jaminan kepastian perilaku mendatang.

### 6.3 Tabel Bukti Riwayat Sumber Data (*Insight Sources Table*)
Komponen inti transparansi (memetakan isi relasi `insight_sources`):

- **Header Tabel**: "Sumber Data yang Digunakan dalam Kalkulasi (8 Transaksi)"
- **Subteks**: "Klik pada nomor faktur untuk memeriksa rincian item transaksi pembentuk wawasan ini."
- **Kolom Tabel**:
  1. **Nomor Referensi**: Tautan teks interaktif (misal `#TRX-20260920-004`), membuka tab baru atau drawer detail transaksi.
  2. **Tanggal Transaksi**: Format `DD/MM/YYYY` (misal `20/09/2026`).
  3. **Total Belanja**: Format Rupiah (misal `Rp 480.000`).
  4. **Item Utama**: Ringkasan item (misal `Kopi Arabika 1kg (x2)`).
  5. **Status Pembayaran**: Lunas (`paid`).
- **Pemberitahuan Audit**: "Seluruh data bersumber dari catatan transaksi terverifikasi pada bisnis Anda. Tidak ada data pihak ketiga yang digunakan."

### 6.4 Footer Modal
- **Kiri**: Status Rekomendasi Terkini (`Status: Baru`).
- **Kanan**:
  - Tombol Outline: `Tutup`
  - Tombol Primer: `Jalankan Rekomendasi: Hubungi via WhatsApp →`

---

## 7. Status Khusus: Data Gate & State Edukatif (Empty, Loading, Cooldown, Error)

Sesuai TRD §7 (Guardrails & Data Gates), mesin analisis tidak akan mengeluarkan wawasan jika data belum memadai, melainkan menampilkan status edukatif:

### 7.1 Empty State: Data Gate Belum Terpenuhi
Ditampilkan pada tab `Pelanggan` jika belum ada pelanggan yang memenuhi syarat analisis (misal belum ada pelanggan dengan ≥ 4 tanggal transaksi dalam 180 hari):
- **Visual**: Ilustrasi diagram garis dokumen monokrom netral (tanpa emoji, tanpa 3D).
- **Judul**: "Data Transaksi Pelanggan Masih Dikumpulkan"
- **Penjelasan**: "Untuk memastikan rekomendasi yang jujur dan dapat diandalkan, sistem memerlukan minimal 4 riwayat transaksi per pelanggan dalam 180 hari terakhir sebelum mengevaluasi pola kebiasaan belanja."
- **Status Saat Ini**: "Saat ini terdapat 12 pelanggan terdaftar, dengan transaksi terbanyak tercatat 2 kali."
- **Aksi Saran**: Tombol Primer `+ Buka Kasir POS` untuk mencatat transaksi baru dengan menautkan nama pelanggan.

### 7.2 Empty State: Seluruh Kondisi Optimal (Zero Issues)
Ditampilkan ketika analisis selesai dijalankan dan tidak ada indikator anomali/risiko:
- **Visual**: Ikon lencana perisai centang hijau zamrud (`#0D6E64`).
- **Judul**: "Semua Operasional Usaha dalam Kondisi Baik"
- **Penjelasan**: "Tidak ditemukan penurunan frekuensi pelanggan, produk di bawah HPP, atau komplain terbuka yang membutuhkan penanganan mendesak."
- **Status Metrik Ringkas**:
  - 100% komplain terselesaikan.
  - Seluruh harga produk berada di atas HPP aktif.
  - Seluruh transaksi 7 hari terakhir terhubung dengan pelanggan.
- **Aksi Saran**: Tombol Sekunder `Lihat Ringkasan Finansial di Dashboard →`.

### 7.3 State Proses Analisis Sedang Berjalan (`Loading / Running State`)
- Saat `POST /analysis/run` dipicu:
  - Header feed menampilkan animasi shimmer / skeleton loading bergaris tipis pada 3 kartu wawasan placeholder.
  - Teks status: "Mengevaluasi transaksi terbaru dan memutakhirkan wawasan... Mohon tunggu."
  - Tidak memblokir pengguna untuk membaca wawasan yang sudah ada.

---

## 8. Tata Letak Responsif (Mobile 390px, Tablet 768px, Desktop 1440px)

### 8.1 Mobile Breakpoint (390px)
- **Header**: Judul dan tombol "Analisis Terbaru" menumpuk secara vertikal (*flex-col*).
- **Filter Tab**: Tab kategori dapat digeser secara horizontal (*horizontal scrollable with hidden scrollbar*).
- **Kartu Wawasan**:
  - Header kartu: Kategori, badge klaim, dan badge prioritas ditata membungkus (*wrap*).
  - Grid metrik bukti: Menjadi 1 kolom tumpuk.
  - Tombol footer kartu: Tombol Aksi Primer menjadi lebar penuh (*width: 100%*), tombol "Bukti" dan "Abaikan" ditata berdampingan di baris bawahnya.
- **Modal Detail Provenance**:
  - Ditampilkan sebagai *Bottom Sheet Drawer* (meluncur dari bawah layar, border-radius atas 16px, tinggi maksimal 85vh dengan area konten *scrollable*).
  - Tabel bukti sumber data disajikan dalam format kartu ringkasan tumpuk (*stacked source cards*).

### 8.2 Tablet Breakpoint (768px)
- **Header**: Sejajar horizontal (*flex-row*) dengan tombol pemicu analisis di sisi kanan.
- **Feed Tata Letak**: 1 kolom feed terpusat (*max-width: 680px*), jarak antar kartu `16px`.
- **Modal Detail**: Modal pop-up di tengah layar dengan lebar `90vw` (maksimal `680px`). Tabel bukti data memiliki scroll horizontal jika kolom lebar.

### 8.3 Desktop Breakpoint (1440px)
- **Tata Letak 2 Kolom Asimetris (Feed 68% + Ringkasan Rekomendasi Prioritas 32%)**:
  - **Kolom Kiri (Feed Utama - 68%)**:
    - Filter kategori dan sub-filter klaim/prioritas.
    - Daftar kartu wawasan terurut berdasarkan prioritas (Tinggi → Sedang → Rendah) dan tanggal kalkulasi.
  - **Kolom Kanan (Sticky Panel Ringkasan Rekomendasi - 32%)**:
    - Kontainer *sticky* (`top: 80px`).
    - Panel "Aksi Cepat Prioritas": Menampilkan 3 rekomendasi prioritas tinggi yang memerlukan tindakan hari ini.
    - Widget "Kesehatan Data Usaha": Ringkasan cakupan HPP dan persentase transaksi terhubung pelanggan dengan tautan langsung ke modul terkait.
- **Modal Detail**: Dialog lebar terpusat (`width: 760px`), tabel bukti data ditampilkan utuh 5 kolom tanpa pemotongan.

---

## 9. Token Gerak (*Motion Specifications*) & Aksesibilitas (WCAG AA & A11y Checklist)

### 9.1 Spesifikasi Gerak Antarmuka (Trigger → Property → Duration → Easing)

| Komponen | Pemicu (Trigger) | Properti Diubah | Durasi | Easing |
| :--- | :--- | :--- | :--- | :--- |
| **Tab Kategori Feed** | Klik Tab | `opacity: 0 → 1`, `transform: translateY(4px → 0)` | 180ms | `cubic-bezier(0.22, 1, 0.36, 1)` |
| **Kartu Wawasan Masuk** | Feed Load / Filter | `opacity: 0 → 1`, `transform: translateY(8px → 0)` | 220ms | `cubic-bezier(0.22, 1, 0.36, 1)` |
| **Kartu Diabaikan (Dismiss)** | Konfirmasi Abaikan | `opacity: 1 → 0`, `transform: scale(1 → 0.96)`, `height: auto → 0` | 240ms | `cubic-bezier(0.4, 0, 1, 1)` |
| **Modal Detail Provenance** | Buka Modal | `opacity: 0 → 1`, `transform: scale(0.96 → 1)` | 240ms | `cubic-bezier(0.22, 1, 0.36, 1)` |
| **Modal Detail Provenance** | Tutup Modal | `opacity: 1 → 0`, `transform: scale(1 → 0.96)` | 180ms | `cubic-bezier(0.4, 0, 1, 1)` |
| **Tombol Cooldown Timer** | Detik Berkurang | `content / innerText` (tanpa animasi pergeseran layout) | 0ms | — |

*Catatan Reduced Motion:* Bila pengguna mengaktifkan mode `prefers-reduced-motion: reduce`, seluruh durasi diset ke `0ms` atau transisi memudar halus murni (*opacity only* tanpa pergeseran `translateY` atau `scale`).

### 9.2 Aksesibilitas & Kompatibilitas Keyboard (WCAG AA)
1. **Navigasi Keyboard**:
   - `Tab` / `Shift+Tab`: Berpindah secara logis dari Header → Filter Tab → Kartu Wawasan → Tombol Aksi di dalam kartu.
   - `Enter` / `Space`: Mengaktifkan filter tab, membuka modal detail wawasan, atau membuka dialog dismiss.
   - `Esc`: Menutup modal detail atau modal dismiss seketika, dan mengembalikan fokus keyboard ke tombol pemicu awal (*return focus trap*).
2. **Atribut ARIA & Screen Reader**:
   - Badge Claim-Type dilengkapi label eksplisit: misal `<span class="badge-claim-hypo" role="status" aria-label="Jenis Klaim: Hipotesis berdasarkan pola kebiasaan">HIPOTESIS</span>`.
   - Area feed wawasan menggunakan `role="region"` dengan `aria-live="polite"` saat proses analisis data selesai memutakhirkan wawasan.
   - Modal detail menggunakan `role="dialog"`, `aria-modal="true"`, dan `aria-labelledby="modal-insight-title"`.
3. **Kontras Teks & Warna Fungsional**:
   - Seluruh teks isi memenuhi rasio kontras ≥ 4.5:1 terhadap permukaan kartu (`#FFFFFF`).
   - Tidak ada informasi yang hanya disampaikan melalui warna semata: seluruh badge prioritas dan claim-type menyertakan teks tertulis dan ikon identifikasi yang berbeda.

---

## 10. Pemetaan Aturan Awal (Rule Engine Mapping & Copy Matrix)

Tabel referensi teks antarmuka baku yang wajib digunakan frontend untuk setiap wawasan deterministik (TRD §7):

| Rule ID | Kategori UI | Claim Type | Judul Kartu Baku | Salinan Aksi Primer | Aksi Saat Diklik |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `CUST_OVERDUE_V1` | Pelanggan | `HIPOTESIS` | "[Nama] melampaui interval belanja biasanya" | "Hubungi via WhatsApp" | Membuka template chat sapaan sopan & tawaran produk favorit |
| `CUST_FREQ_DROP_V1` | Pelanggan | `HIPOTESIS` | "Penurunan frekuensi belanja: [Nama]" | "Tinjau Riwayat Pelanggan" | Menavigasi ke halaman detail profil `/customers/{id}` |
| `CUST_VALUE_DROP_V1` | Pelanggan | `HIPOTESIS` | "Rerata nominal belanja [Nama] menurun" | "Kirim Penawaran Khusus" | Membuka pesan WhatsApp berisi katalog produk |
| `CUST_COMPLAINT_OPEN_V1` | Pelanggan | `FAKTA` | "Komplain belum terselesaikan > 3 hari: [Nama]" | "Selesaikan Komplain" | Membuka detail komplain pelanggan di `/customers/{id}#complaints` |
| `PROD_BELOW_COST_V1` | Produk & Margin | `FAKTA` | "Harga jual [Produk] berada di bawah HPP" | "Sesuaikan Harga Jual" | Membuka dialog kalkulator penetapan harga produk di `/products/{id}` |
| `PROD_LOW_MARGIN_V1` | Produk & Margin | `ESTIMASI` | "Margin [Produk] jauh di bawah rata-rata bisnis" | "Evaluasi HPP & Harga" | Menavigasi ke kalkulator harga produk di `/products/{id}` |
| `DATA_MISSING_COST_V1` | Kesehatan Data | `FAKTA` | "[N] produk terjual belum memiliki data HPP" | "Lengkapi HPP Produk" | Menavigasi ke filter katalog produk tanpa HPP `/products?filter=missing_hpp` |
| `DATA_UNLINKED_TX_V1` | Kesehatan Data | `FAKTA` | "Sebagian besar transaksi belum tertaut pelanggan" | "Buka Kasir POS" | Mengarahkan ke `/pos` dengan panduan memilih pelanggan saat checkout |

---

## 11. Checklist Penerimaan Desain (Sign-Off Criteria)

- [x] Mematuhi palet warna tunggal Emerald Forest (`#0D6E64`) dan slate-sage neutrals tanpa gradien dekoratif slop.
- [x] Sistem badge claim-type (`FAKTA`, `ESTIMASI`, `HIPOTESIS`) memenuhi kontras WCAG AA ≥ 4.5:1.
- [x] Salinan teks wawasan deterministik dan tidak membuat klaim spekulatif palsu.
- [x] Transparansi data terjamin melalui tabel sumber data (*insight sources*) dengan tautan ke catatan riil.
- [x] Siklus hidup rekomendasi mencakup state `new`, `reviewed`, `in_progress`, `completed`, `dismissed`, dan `superseded`.
- [x] Fitur pencegahan spam wawasan dengan cooldown pemicuan manual (10 menit) dan cooldown pengabaian (14 hari).
- [x] Tata letak responsif dirancang lengkap untuk 390px, 768px, dan 1440px.
- [x] Spesifikasi motion lengkap dengan format `trigger → property → duration → easing` dan dukungan reduced-motion.
