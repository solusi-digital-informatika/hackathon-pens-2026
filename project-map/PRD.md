# Product Requirements Document (PRD)

# Teman UMKM

**Product type:** AI-Powered Business Decision Support Platform  
**Platform:** Web Application  
**Target market:** UMKM di Indonesia  
**Initial segment:** UMKM penjual barang fisik, terutama bisnis makanan dan reseller  
**Document status:** Draft v1.0  
**Product stage:** MVP  
**Primary language:** Bahasa Indonesia

---

## 1. Product Overview

### 1.1. Product Vision

Teman UMKM adalah platform pendukung keputusan bisnis berbasis data yang membantu pemilik UMKM memahami kondisi keuangan, profitabilitas produk, perilaku pelanggan, dan prioritas tindakan bisnis melalui satu sistem yang sederhana dan mudah dipahami.

Produk ini dirancang untuk membantu pemilik usaha beralih dari pengambilan keputusan berdasarkan intuisi semata menuju keputusan yang didukung oleh data transaksi, perhitungan keuangan yang transparan, dan analisis kontekstual.

Dalam jangka panjang, Teman UMKM akan berkembang menjadi AI Business Operator yang bukan hanya menganalisis bisnis, tetapi juga membantu pemilik usaha menjalankan pekerjaan operasional secara lebih efisien.

### 1.2. Problem Statement

Banyak pemilik UMKM menjalankan bisnis melalui kombinasi spreadsheet, buku catatan, aplikasi kasir, marketplace, dan percakapan dengan pelanggan. Informasi tersebut sering kali tersebar dan tidak membentuk gambaran bisnis yang utuh.

Akibatnya, pemilik usaha dapat mengalami kesulitan dalam:

- Menentukan harga jual berdasarkan biaya dan margin yang diinginkan.
- Mengetahui keuntungan aktual dari setiap produk.
- Membedakan omzet, laba, dan arus kas.
- Mengidentifikasi pelanggan yang mulai tidak aktif.
- Menentukan produk atau pelanggan yang perlu diprioritaskan.
- Mengubah data transaksi menjadi tindakan bisnis yang konkret.

Pencatatan transaksi saja belum cukup. Pemilik usaha membutuhkan sistem yang mampu menjelaskan apa yang sedang terjadi, mengapa hal tersebut penting, dan tindakan apa yang dapat dipertimbangkan.

### 1.3. Product Goal

Tujuan utama Teman UMKM adalah mengurangi kesenjangan antara pencatatan bisnis dan pengambilan keputusan bisnis.

Produk harus mampu:

1. Mengubah data usaha yang tersebar menjadi informasi terstruktur.
2. Menghitung metrik keuangan dengan formula yang transparan.
3. Menghubungkan data pelanggan, transaksi, produk, interaksi, dan keluhan.
4. Mengidentifikasi risiko atau peluang yang didukung oleh bukti.
5. Memberikan rekomendasi tindakan yang dapat diperiksa dan diputuskan oleh pemilik usaha.

### 1.4. Product Principles

**Accuracy before intelligence**

Perhitungan keuangan harus menggunakan formula deterministik yang dapat diuji. AI tidak boleh menjadi sumber utama perhitungan angka keuangan.

**Explainable by default**

Setiap insight harus menjelaskan bukti, periode data, asumsi, dan keterbatasannya.

**Owner stays in control**

Pemilik usaha tetap memegang kendali atas keputusan harga, promosi, pengeluaran, komunikasi pelanggan, dan tindakan bisnis lainnya.

**Simple for non-technical users**

Pengguna tidak harus memahami akuntansi, data analytics, atau prompt engineering untuk mendapatkan manfaat produk.

**Progressive automation**

Produk dimulai dari pencatatan dan analisis, kemudian berkembang menuju rekomendasi dan eksekusi tindakan yang membutuhkan persetujuan pengguna.

---

## 2. Target Users and Personas

### 2.1. Primary Persona: UMKM Owner

Karakteristik:

- Mengelola bisnis sendiri atau dengan tim kecil.
- Menjual produk fisik, seperti makanan, minuman, atau barang reseller.
- Mencatat transaksi menggunakan spreadsheet, buku, atau aplikasi sederhana.
- Memiliki pelanggan berulang.
- Tidak selalu mempunyai staf keuangan atau analis bisnis.

Kebutuhan utama:

- Mengetahui laba dan margin produk.
- Memahami kondisi keuangan usaha.
- Mengetahui pelanggan yang masih aktif dan yang mulai tidak aktif.
- Mendapatkan rekomendasi bisnis yang praktis.
- Menghemat waktu dalam mengolah data.

### 2.2. Secondary Persona: Operational Staff

Karakteristik:

- Membantu memasukkan transaksi atau memperbarui data usaha.
- Membutuhkan alur input yang cepat.
- Tidak selalu memiliki kewenangan mengambil keputusan finansial.

Kebutuhan utama:

- Memasukkan data secara konsisten.
- Meminimalkan kesalahan input.
- Mengetahui transaksi yang belum lengkap.
- Bekerja tanpa harus memahami seluruh analisis bisnis.

### 2.3. Initial Customer Profile

MVP memprioritaskan UMKM yang:

- Menjual barang fisik.
- Memiliki data transaksi yang dapat dikumpulkan.
- Memiliki produk dengan biaya yang bisa diestimasi.
- Memiliki kemungkinan pelanggan melakukan pembelian berulang.
- Bersedia mencoba pencatatan yang lebih terstruktur.

Teman UMKM tidak ditujukan untuk semua jenis bisnis pada tahap awal. Validasi pertama harus dilakukan pada satu segmen yang cukup homogen agar kebutuhan dan pola transaksi dapat dipahami dengan baik.

---

## 3. Jobs to Be Done

Ketika mengelola bisnis, pemilik usaha ingin:

1. Mengetahui apakah penjualan yang meningkat benar-benar meningkatkan keuntungan.
2. Mengetahui produk mana yang menghasilkan margin lebih baik.
3. Memahami berapa harga jual yang masuk akal berdasarkan biaya.
4. Mengetahui pelanggan mana yang menunjukkan perubahan pola pembelian.
5. Memahami penyebab perubahan performa bisnis berdasarkan data yang tersedia.
6. Mendapatkan langkah berikutnya tanpa harus menganalisis spreadsheet secara manual.

### Core Job

"Ketika saya mengelola usaha, bantu saya memahami kondisi bisnis berdasarkan data yang saya miliki dan menentukan tindakan yang paling masuk akal, tanpa harus menjadi analis keuangan atau data."

---

## 4. Product Scope

### 4.1. MVP — In Scope

MVP mencakup:

1. Authentication dan business profile.
2. Product management.
3. Cost of Goods Sold (COGS/HPP) dan kalkulasi harga.
4. Sales transaction recording.
5. Data import melalui CSV atau spreadsheet.
6. Financial summary dan profit analysis.
7. Customer-linked transaction history.
8. Simple business context graph.
9. Customer activity dan risk insight.
10. Explainable recommendation engine.
11. Dashboard dan insight detail.
12. Data validation dan import error handling.

### 4.2. Out of Scope for MVP

Fitur berikut tidak menjadi bagian dari MVP:

- Integrasi langsung dengan WhatsApp Business API.
- Integrasi otomatis dengan marketplace.
- Integrasi POS secara langsung.
- Sinkronisasi otomatis rekening bank.
- Akuntansi dan pelaporan pajak lengkap.
- Inventory management yang kompleks.
- Supplier management.
- Payroll dan human resources.
- Forecasting berbasis machine learning.
- Autonomous AI yang dapat melakukan transaksi tanpa persetujuan.
- Multi-branch management.
- Multi-user role management yang kompleks.
- Pengiriman promosi otomatis kepada pelanggan.
- Sistem kredit atau pemberian pinjaman.

Fitur-fitur tersebut dapat dipertimbangkan setelah kebutuhan utama tervalidasi.

---

## 5. Functional Requirements

Priority definitions:

- **P0:** Wajib tersedia untuk MVP.
- **P1:** Penting, tetapi dapat ditambahkan setelah alur utama berjalan.
- **P2:** Pengembangan lanjutan.

### 5.1. Authentication and Business Setup

**Priority: P0**

#### Requirements

- Pengguna dapat membuat akun dan login.
- Pengguna dapat membuat profil usaha.
- Sistem menyimpan nama usaha, kategori, mata uang, zona waktu, dan preferensi dasar.
- Pengguna dapat mengatur periode laporan.
- Data setiap usaha harus terisolasi dari usaha lainnya.

#### Acceptance Criteria

- Pengguna yang belum login tidak dapat mengakses data privat.
- Pengguna hanya dapat mengakses data bisnis yang menjadi haknya.
- Sistem menggunakan zona waktu Asia/Jakarta sebagai default untuk bisnis Indonesia.
- Periode laporan ditampilkan secara konsisten.

### 5.2. Product Management

**Priority: P0**

#### Requirements

Pengguna dapat:

- Menambahkan produk.
- Mengubah informasi produk.
- Menonaktifkan produk.
- Menentukan SKU atau kode produk opsional.
- Menentukan satuan produk.
- Menentukan harga jual.
- Menentukan komponen biaya produk.
- Melihat riwayat perubahan biaya dan harga yang relevan.

#### Product Data

- Product ID.
- Business ID.
- Product name.
- SKU.
- Category.
- Unit of measure.
- Selling price.
- Status.
- Created at.
- Updated at.

Produk yang sudah memiliki transaksi tidak boleh dihapus secara permanen jika penghapusan tersebut merusak histori transaksi.

### 5.3. HPP and Pricing Calculator

**Priority: P0**

#### Requirements

Pengguna dapat memasukkan komponen biaya untuk menghitung estimasi HPP produk.

Komponen dapat meliputi:

- Bahan baku atau harga pembelian.
- Kemasan.
- Biaya produksi langsung.
- Biaya langsung lainnya.
- Alokasi biaya tambahan yang relevan.

Sistem harus memperlihatkan komponen perhitungan dan hasilnya secara transparan.

#### Core Formulas

HPP per unit:

`HPP per unit = Total biaya yang dialokasikan / Jumlah unit yang dihasilkan`

Laba kotor per unit:

`Laba kotor per unit = Harga jual per unit - HPP per unit`

Gross margin:

`Gross margin (%) = (Laba kotor per unit / Harga jual per unit) × 100%`

Harga jual berdasarkan target margin:

`Harga jual = HPP per unit / (1 - Target margin)`

Target margin harus berada di antara 0% dan 100%.

Catatan: target margin berbeda dari markup. Biaya platform, diskon, biaya pembayaran, pajak, dan biaya lain harus diperhitungkan jika relevan. Sistem tidak boleh mengklaim bahwa laba kotor adalah laba bersih.

#### Acceptance Criteria

- Hasil perhitungan dapat direproduksi menggunakan formula yang sama.
- Sistem menolak input biaya atau kuantitas yang tidak valid.
- Sistem menampilkan komponen biaya yang digunakan.
- Sistem memperingatkan pengguna jika harga jual berada di bawah HPP.
- Perubahan HPP tidak mengubah histori transaksi lama secara diam-diam.

### 5.4. Sales Transaction Recording

**Priority: P0**

#### Requirements

Pengguna dapat:

- Membuat transaksi penjualan.
- Memilih satu atau lebih produk.
- Menentukan kuantitas.
- Mencatat harga aktual.
- Mencatat diskon dan biaya transaksi jika tersedia.
- Menghubungkan transaksi dengan pelanggan.
- Mencatat tanggal transaksi.
- Mengubah atau membatalkan transaksi dengan audit trail.
- Melihat histori transaksi.

Pelanggan baru dapat dibuat ketika transaksi dicatat.

#### Transaction Data

- Transaction ID.
- Business ID.
- Customer ID, optional.
- Transaction date.
- Transaction status.
- Line items.
- Quantity.
- Unit selling price.
- Discount.
- Additional fees, if applicable.
- Recorded cost snapshot.
- Payment status, when captured.
- Source.
- Created at.
- Updated at.

#### Acceptance Criteria

- Transaksi harus memiliki minimal satu item.
- Kuantitas harus lebih besar dari nol.
- Harga dan nilai transaksi harus divalidasi.
- Sistem menyimpan snapshot biaya yang digunakan untuk analisis transaksi.
- Transaksi yang dibatalkan tidak dihitung sebagai penjualan valid.
- Perubahan transaksi harus dapat ditelusuri.

### 5.5. CSV and Spreadsheet Import

**Priority: P0**

Impor data menjadi jalur utama untuk membawa data usaha yang telah dimiliki pengguna ke dalam sistem tanpa membutuhkan integrasi eksternal.

#### Requirements

- Pengguna dapat mengunggah file CSV.
- Sistem menyediakan contoh template CSV.
- Sistem menampilkan preview data sebelum impor.
- Sistem memungkinkan pemetaan kolom.
- Sistem memvalidasi format dan nilai.
- Sistem menampilkan baris yang gagal diproses beserta alasannya.
- Pengguna dapat mengonfirmasi proses impor.
- Sistem memberikan ringkasan jumlah data yang berhasil dan gagal.

#### Import Flow

1. Upload file.
2. Parse file.
3. Preview data.
4. Map columns.
5. Validate rows.
6. Display warnings and errors.
7. Confirm import.
8. Process valid rows.
9. Display import summary.

#### Acceptance Criteria

- File yang tidak valid tidak boleh menyebabkan data bisnis rusak.
- Kesalahan pada satu baris tidak boleh menyebabkan seluruh impor gagal tanpa penjelasan.
- Sistem mendeteksi potensi duplikasi.
- Sistem tidak melakukan penggabungan pelanggan secara otomatis jika identitasnya ambigu.
- Pengguna mengetahui data mana yang berhasil diproses.

### 5.6. Financial Dashboard

**Priority: P0**

#### Requirements

Dashboard menampilkan:

- Total penjualan.
- Jumlah transaksi.
- Total diskon jika tersedia.
- Estimasi HPP penjualan.
- Estimasi laba kotor.
- Gross margin.
- Produk dengan penjualan tertinggi.
- Produk dengan margin tertinggi.
- Perbandingan antarperiode.
- Status kelengkapan data.

Dashboard harus menyediakan filter periode.

#### Calculation Rules

Total penjualan bersih:

`Net sales = Gross sales - Discounts - Returns, when recorded`

Pendapatan penjualan tidak boleh otomatis dianggap sebagai kas yang sudah diterima.

Estimasi laba kotor:

`Gross profit = Net sales - COGS of valid sales`

Gross margin:

`Gross margin (%) = Gross profit / Net sales × 100%`

Jika net sales bernilai nol, margin harus ditampilkan sebagai tidak tersedia, bukan menghasilkan pembagian dengan nol.

Laporan harus membedakan:

- Penjualan.
- Pembayaran yang diterima, jika datanya tersedia.
- Laba kotor.
- Estimasi biaya lainnya.
- Laba bersih, hanya jika komponen biaya yang dibutuhkan telah tercatat secara memadai.

#### Acceptance Criteria

- Angka dashboard konsisten dengan transaksi valid.
- Perubahan filter memperbarui seluruh metrik yang relevan.
- Data yang belum lengkap ditandai.
- Sistem tidak menampilkan estimasi laba sebagai angka pasti tanpa penjelasan asumsi.

### 5.7. Customer Management and Transaction History

**Priority: P0**

#### Requirements

Pengguna dapat:

- Membuat dan memperbarui profil pelanggan.
- Melihat transaksi pelanggan.
- Melihat tanggal pembelian terakhir.
- Melihat frekuensi transaksi.
- Melihat total nilai pembelian dalam periode tertentu.
- Melihat produk yang pernah dibeli.
- Melihat catatan interaksi atau keluhan jika tersedia.

#### Customer Data

- Customer ID.
- Business ID.
- Display name.
- Contact identifier, optional.
- Identity mapping.
- First transaction date.
- Last transaction date.
- Transaction count.
- Customer notes, optional.

Sistem harus mendukung pelanggan tanpa nomor telepon, karena tidak semua data historis mempunyai identitas lengkap.

#### Acceptance Criteria

- Pelanggan dapat ditautkan ke beberapa transaksi.
- Data transaksi tetap dapat digunakan walaupun identitas pelanggan belum diketahui.
- Penggabungan dua profil pelanggan memerlukan konfirmasi jika terdapat ambiguitas.
- Data pelanggan tidak boleh digabung hanya berdasarkan kemiripan nama.

### 5.8. Business Context Graph

**Priority: P0**

Teman UMKM membangun hubungan antarentitas bisnis untuk mendukung analisis kontekstual.

#### Core Entities

- Business.
- Product.
- Customer.
- Transaction.
- Transaction Item.
- Interaction.
- Complaint.
- Insight.
- Recommended Action.

#### Core Relationships

- Customer melakukan Transaction.
- Transaction memiliki Transaction Item.
- Transaction Item mengacu pada Product.
- Customer memiliki Interaction.
- Customer dapat memiliki Complaint.
- Insight dihasilkan dari data transaksi dan hubungan terkait.
- Recommended Action terkait dengan Insight.

#### Requirements

- Hubungan data harus memiliki referensi yang konsisten.
- Analisis harus dapat menelusuri data sumber yang mendukungnya.
- Data yang tidak memiliki identitas pasti tidak boleh dihubungkan secara spekulatif.
- Model data dapat dikembangkan tanpa mewajibkan penggunaan graph database pada MVP.

Implementasi awal dapat menggunakan relational database dengan foreign key dan query yang sesuai. Graph database hanya dipertimbangkan jika kebutuhan teknis dan skala sudah membenarkannya.

### 5.9. Customer Activity and Risk Insight

**Priority: P0**

#### Objective

Mengidentifikasi perubahan pola pembelian pelanggan yang mungkin memerlukan perhatian pemilik usaha.

#### Initial Signals

- Pelanggan yang biasanya membeli secara berkala tetapi belum membeli kembali.
- Penurunan frekuensi pembelian.
- Penurunan nilai transaksi.
- Keluhan yang belum ditindaklanjuti.
- Pelanggan dengan nilai pembelian tinggi yang menunjukkan perubahan aktivitas.

#### Requirements

- Sistem menganalisis histori transaksi pelanggan.
- Sistem mempertimbangkan jarak antartransaksi jika datanya memadai.
- Sistem menampilkan bukti dan periode analisis.
- Sistem menjelaskan keterbatasan data.
- Pengguna dapat membuka histori yang mendukung suatu insight.

#### Example Insight

"Rina biasanya melakukan pembelian setiap 14–21 hari. Transaksi terakhir tercatat 35 hari lalu. Pelanggan ini mungkin memerlukan tindak lanjut."

Insight tersebut harus menampilkan:

- Tanggal transaksi terakhir.
- Jumlah transaksi yang menjadi dasar analisis.
- Pola pembelian historis.
- Periode pengamatan.
- Penjelasan bahwa perubahan aktivitas tidak membuktikan pelanggan telah berhenti membeli.

#### Guardrails

- Sistem tidak boleh menyatakan bahwa pelanggan pasti akan berhenti membeli.
- Sistem tidak boleh membuat klaim tentang alasan pelanggan tanpa bukti.
- Pelanggan baru dengan histori terbatas tidak boleh diberi label risiko yang tidak berdasar.
- Ambang batas analisis harus dapat dijelaskan dan dievaluasi.
- Risiko hubungan pelanggan harus dibedakan dari risiko keuangan usaha.

### 5.10. Explainable Recommendation Engine

**Priority: P0**

#### Objective

Mengubah metrik dan insight menjadi rekomendasi tindakan yang dapat dipertimbangkan pemilik usaha.

#### Recommendation Examples

- Meninjau kembali harga produk dengan margin rendah.
- Memeriksa komponen biaya yang belum tercatat.
- Menindaklanjuti pelanggan yang melewati pola pembelian biasanya.
- Memeriksa keluhan pelanggan yang belum diselesaikan.
- Membandingkan performa produk antarperiode.
- Memvalidasi data sebelum mengambil keputusan.

#### Recommendation Structure

Setiap rekomendasi harus memiliki:

- Title.
- Description.
- Supporting evidence.
- Reasoning.
- Data period.
- Assumptions.
- Limitations.
- Suggested action.
- Priority.
- Status.
- Creation timestamp.

#### Recommendation Status

- New.
- Reviewed.
- In Progress.
- Completed.
- Dismissed.

#### Requirements

- Rekomendasi memiliki referensi terhadap data yang mendukungnya.
- Rekomendasi dapat diabaikan oleh pemilik usaha.
- Rekomendasi tidak otomatis menjalankan transaksi finansial.
- Sistem membedakan fakta, estimasi, dan hipotesis.
- Rekomendasi tidak dibuat jika data pendukung tidak memadai.

### 5.11. Insight Detail and Data Provenance

**Priority: P0**

Pengguna harus dapat memahami dari mana sebuah angka atau insight berasal.

#### Requirements

Setiap insight atau perhitungan penting harus dapat menampilkan:

- Sumber data.
- Periode yang digunakan.
- Formula atau aturan yang diterapkan.
- Asumsi.
- Data yang tidak tersedia.
- Waktu terakhir perhitungan.
- Versi aturan analisis jika relevan.

Fitur ini menjadi fondasi kepercayaan terhadap produk.

### 5.12. Natural Language AI Assistant

**Priority: P1**

AI assistant dapat membantu pengguna memahami data dengan bahasa sehari-hari.

Contoh:

- "Produk mana yang marginnya paling rendah bulan ini?"
- "Pelanggan mana yang belum membeli kembali seperti biasanya?"
- "Mengapa laba kotor saya turun dibanding bulan lalu?"
- "Apa yang perlu saya periksa terlebih dahulu?"

#### Requirements

- AI menggunakan data bisnis yang telah diotorisasi.
- Perhitungan angka menggunakan sumber data dan formula yang dapat diverifikasi.
- Jawaban harus menyertakan periode dan sumber data.
- AI menyatakan jika informasi tidak tersedia.
- AI tidak mengarang transaksi atau metrik.
- AI tidak boleh mengakses data bisnis pengguna lain.
- Pertanyaan yang memerlukan data tidak lengkap harus menghasilkan penjelasan keterbatasan.

AI assistant tidak boleh menjadi prasyarat agar fitur inti MVP dapat digunakan.

---

## 6. Core User Flows

### 6.1. First-Time Setup

1. Pengguna membuat akun.
2. Pengguna mengisi profil bisnis.
3. Pengguna menambahkan produk atau mengimpor data.
4. Pengguna memvalidasi HPP dan harga.
5. Pengguna memasukkan atau mengimpor transaksi.
6. Sistem menghitung metrik dasar.
7. Pengguna melihat dashboard dan kelengkapan data.
8. Sistem menampilkan insight pertama jika data telah memadai.

### 6.2. Product Profitability Analysis

1. Pengguna membuka daftar produk.
2. Pengguna memilih produk.
3. Sistem menampilkan harga jual dan komponen HPP.
4. Sistem menghitung laba kotor dan margin.
5. Sistem menampilkan histori performa penjualan.
6. Pengguna meninjau data dan menentukan tindakan.

### 6.3. Customer Follow-Up

1. Sistem mengidentifikasi perubahan pola pembelian.
2. Sistem menghasilkan insight beserta bukti.
3. Pemilik usaha membuka detail insight.
4. Pemilik usaha memeriksa histori transaksi.
5. Pemilik usaha memutuskan apakah tindak lanjut diperlukan.
6. Status tindakan diperbarui secara manual.

### 6.4. CSV Import

1. Pengguna mengunduh template.
2. Pengguna mengisi data.
3. Pengguna mengunggah file.
4. Sistem memvalidasi data.
5. Pengguna memperbaiki error atau melanjutkan dengan baris valid.
6. Pengguna mengonfirmasi impor.
7. Sistem menyimpan data dan menghasilkan ringkasan.
8. Dashboard diperbarui berdasarkan data yang berhasil diproses.

---

## 7. Information Architecture

### Primary Navigation

1. Dashboard.
2. Transactions.
3. Products.
4. Customers.
5. Insights.
6. Reports.
7. Settings.

### Dashboard Sections

**Business Overview**

- Net sales.
- Gross profit.
- Gross margin.
- Transaction count.

**Product Performance**

- Top products by sales.
- Products with low gross margin.
- Product-level performance.

**Customer Activity**

- Recent purchases.
- Customers whose purchasing patterns have changed.
- Outstanding follow-up items.

**Recommended Actions**

- Priority.
- Supporting evidence.
- Action status.

**Data Health**

- Missing product costs.
- Unlinked customer transactions.
- Import errors.
- Incomplete transaction records.

Dashboard harus memprioritaskan informasi yang membantu keputusan, bukan sekadar menampilkan sebanyak mungkin grafik.

---

## 8. Non-Functional Requirements

### 8.1. Security

- Seluruh endpoint privat memerlukan autentikasi.
- Setiap operasi harus memvalidasi otorisasi berdasarkan Business ID.
- Sistem harus mencegah akses lintas bisnis.
- Data sensitif harus dikirim melalui HTTPS.
- Password tidak boleh disimpan dalam bentuk plaintext.
- Secret dan API key tidak boleh ditanamkan dalam frontend.
- Perubahan penting harus memiliki audit trail.
- File impor harus divalidasi dan diproses secara aman.
- Backup dan prosedur pemulihan harus disiapkan sebelum penggunaan produksi.

### 8.2. Privacy

- Data pelanggan hanya dikumpulkan jika dibutuhkan.
- Informasi pribadi yang tidak diperlukan tidak boleh diwajibkan.
- Data bisnis tidak boleh digunakan untuk melatih model eksternal tanpa dasar yang sah dan pengungkapan yang sesuai.
- Pengguna harus mendapatkan penjelasan mengenai data yang disimpan.
- Retensi dan penghapusan data harus mengikuti kebijakan produk serta ketentuan hukum yang berlaku.

### 8.3. Performance

Target awal yang perlu divalidasi melalui pengujian:

- Dashboard standar: p95 di bawah 2 detik untuk dataset MVP yang disepakati.
- Endpoint CRUD standar: p95 di bawah 500 ms pada kondisi pengujian yang ditentukan.
- Proses impor: asynchronous jika ukuran file melewati ambang batas yang ditentukan.
- Pengguna mendapatkan feedback ketika proses membutuhkan waktu lebih lama.

Angka ini merupakan target rekayasa awal, bukan klaim performa yang sudah terukur.

### 8.4. Reliability

- Proses impor tidak boleh menggandakan data secara tidak sengaja ketika diulang.
- Kegagalan satu proses tidak boleh merusak transaksi yang telah tersimpan.
- Perhitungan harus konsisten dengan sumber data.
- Operasi yang mengubah data harus menangani kegagalan dan retry secara aman.
- Error harus dicatat tanpa membocorkan informasi sensitif.

### 8.5. Usability and Accessibility

- Bahasa antarmuka menggunakan Bahasa Indonesia.
- Angka uang menggunakan format Rupiah.
- Formulir menampilkan validasi yang jelas.
- Tabel harus tetap mudah digunakan pada layar kecil.
- Status data kosong dan data tidak lengkap harus dijelaskan.
- Informasi penting tidak boleh hanya dibedakan menggunakan warna.

---

## 9. Data Model Overview

Model data awal menggunakan relational database.

### Core Tables

**businesses**

- id
- name
- business_type
- timezone
- created_at
- updated_at

**users**

- id
- email
- password_hash_or_auth_provider
- created_at
- updated_at

**business_memberships**

- id
- business_id
- user_id
- role
- created_at

**products**

- id
- business_id
- name
- sku
- unit
- current_selling_price
- status
- created_at
- updated_at

**product_cost_components**

- id
- product_id
- component_name
- cost_per_unit
- effective_from
- created_at

**customers**

- id
- business_id
- display_name
- created_at
- updated_at

**customer_identifiers**

- id
- customer_id
- identifier_type
- identifier_value
- source
- verified_at

**transactions**

- id
- business_id
- customer_id
- transaction_date
- status
- source
- created_at
- updated_at

**transaction_items**

- id
- transaction_id
- product_id
- quantity
- unit_selling_price
- discount_amount
- recorded_unit_cost
- created_at

**interactions**

- id
- business_id
- customer_id
- interaction_type
- occurred_at
- notes
- source

**complaints**

- id
- business_id
- customer_id
- transaction_id
- description
- status
- created_at
- resolved_at

**insights**

- id
- business_id
- insight_type
- title
- explanation
- evidence_snapshot
- period_start
- period_end
- confidence_or_data_quality
- created_at

**recommended_actions**

- id
- business_id
- insight_id
- title
- description
- status
- created_at
- updated_at

**audit_logs**

- id
- business_id
- actor_user_id
- entity_type
- entity_id
- action
- before_snapshot
- after_snapshot
- created_at

### Data Model Rules

- Seluruh data bisnis harus memiliki batas kepemilikan yang jelas.
- Transaksi menyimpan snapshot biaya yang digunakan dalam perhitungan.
- Perubahan harga dan biaya tidak boleh mengubah laporan historis tanpa aturan penyesuaian yang eksplisit.
- Identitas pelanggan harus dipisahkan dari asumsi pencocokan identitas.
- Bukti insight harus cukup untuk merekonstruksi dasar analisis.
- Data yang tidak diketahui harus dibedakan dari nilai nol.

---

## 10. System Architecture

### 10.1. Architecture Overview

MVP menggunakan arsitektur web client dan backend API dengan satu sumber data utama.

Komponen:

1. Web frontend.
2. Backend application API.
3. Relational database.
4. File import and processing module.
5. Deterministic financial calculation engine.
6. Rule-based insight engine.
7. Optional AI integration layer.
8. Logging and audit trail.

### 10.2. Architectural Principles

- Financial calculations tidak bergantung pada output LLM.
- Business rules berada di backend.
- Frontend tidak menjadi sumber kebenaran untuk perhitungan.
- Semua akses data mengikuti otorisasi bisnis.
- Proses impor menggunakan validasi dan transaksi database yang sesuai.
- Insight menyimpan referensi data dan versi aturan.
- AI diintegrasikan melalui lapisan terpisah agar provider dapat diganti.
- Modul bisnis dipisahkan berdasarkan tanggung jawab, bukan hanya berdasarkan jenis file.

### 10.3. AI Strategy

MVP menggunakan kombinasi:

**Deterministic calculations**

Untuk HPP, penjualan, laba kotor, dan margin.

**Rule-based analysis**

Untuk mendeteksi perubahan pola transaksi, data tidak lengkap, dan kondisi bisnis yang dapat dijelaskan.

**LLM-assisted explanation**

Untuk menerjemahkan hasil analisis menjadi bahasa yang lebih mudah dipahami dan membantu pengguna menanyakan data bisnis.

LLM tidak boleh menciptakan fakta yang tidak ada di sumber data. Rekomendasi yang berdampak pada transaksi atau komunikasi eksternal memerlukan kontrol pengguna.

### 10.4. Future WhatsApp Integration

WhatsApp merupakan kandidat kanal interaksi untuk fase berikutnya.

Arsitektur yang direncanakan:

- Satu backend bisnis sebagai sumber kebenaran.
- Web dashboard untuk konfigurasi, laporan, dan analisis.
- WhatsApp Business Platform sebagai kanal input dan query.
- Webhook untuk menerima pesan.
- Parser untuk mengubah pesan menjadi data terstruktur.
- Validasi dan konfirmasi sebelum penyimpanan.
- Idempotency untuk mencegah pesan yang sama diproses berulang.

Contoh interaksi masa depan:

Pemilik: "Hari ini terjual 12 kotak brownies, harga 25 ribu per kotak."

Sistem:

- Mengidentifikasi produk dan kuantitas.
- Memeriksa apakah harga dan identitas produk jelas.
- Menampilkan ringkasan transaksi.
- Meminta konfirmasi jika terdapat ambiguitas.
- Menyimpan transaksi setelah konfirmasi.

Integrasi ini tidak termasuk MVP dan harus menggunakan mekanisme akses resmi yang sesuai.

---

## 11. Analytics and Success Metrics

### 11.1. North Star Metric

**Weekly Active Businesses Receiving Verified Business Insights**

Jumlah bisnis aktif mingguan yang menerima setidaknya satu insight yang dapat ditelusuri ke data sumber yang valid.

Metrik ini menggabungkan penggunaan berulang dan manfaat inti produk. Namun, metrik ini harus dibaca bersama metrik tindakan pengguna agar sistem tidak sekadar menghasilkan banyak insight yang tidak berguna.

### 11.2. Activation Metrics

- Persentase pengguna yang menyelesaikan profil bisnis.
- Persentase bisnis yang berhasil mengimpor atau memasukkan transaksi.
- Persentase bisnis yang memiliki biaya produk memadai.
- Persentase bisnis yang mencapai dashboard dengan metrik valid.
- Waktu dari pendaftaran hingga insight pertama yang valid.

### 11.3. Engagement Metrics

- Weekly active businesses.
- Jumlah insight yang dibuka.
- Persentase insight yang ditinjau.
- Persentase rekomendasi yang ditindaklanjuti.
- Frekuensi penggunaan laporan.
- Retensi bisnis pada minggu ke-4 dan bulan ke-3.

### 11.4. Quality Metrics

- Akurasi perhitungan keuangan terhadap test cases.
- Tingkat kesalahan impor.
- Tingkat duplikasi transaksi.
- Persentase insight dengan bukti lengkap.
- Persentase insight yang dinilai berguna oleh pengguna.
- Jumlah rekomendasi yang tidak didukung data.
- Tingkat kesalahan pencocokan pelanggan.

### 11.5. Business Metrics

- Conversion to paid.
- Monthly Recurring Revenue (MRR).
- Customer Acquisition Cost (CAC).
- Customer Lifetime Value (LTV).
- Monthly customer churn.
- Gross margin.
- AI cost per active business, apabila AI digunakan.

Target numerik harus ditetapkan setelah pengujian awal dan pengumpulan baseline. Target tidak boleh dipilih hanya karena terlihat menarik.

---

## 12. MVP Validation Plan

### Phase 1: Problem Validation

Durasi indikatif: 1–2 minggu.

Aktivitas:

- Wawancara 20–30 pemilik usaha.
- Mengamati metode pencatatan yang digunakan.
- Memeriksa contoh data transaksi yang telah dianonimkan.
- Mengidentifikasi masalah biaya, laba, dan pelanggan.
- Menguji kesediaan pengguna untuk membayar.

Output:

- Segmen pelanggan awal.
- Daftar masalah yang paling sering terjadi.
- Peta alur kerja saat ini.
- Hipotesis manfaat produk.
- Daftar kebutuhan data.

### Phase 2: Prototype Validation

Durasi indikatif: 1–2 minggu.

Aktivitas:

- Membuat prototipe dashboard.
- Menguji input produk dan biaya.
- Menguji impor data.
- Menampilkan insight pelanggan.
- Menguji pemahaman pengguna terhadap penjelasan insight.

Output:

- Feedback usability.
- Daftar masalah prioritas.
- Revisi alur produk.
- Validasi awal terhadap manfaat.

### Phase 3: MVP Development

Durasi indikatif: 4–8 minggu, tergantung tim dan cakupan teknis.

Aktivitas:

- Membangun authentication dan business isolation.
- Membangun product management.
- Membangun transaction management.
- Membangun CSV import.
- Membangun financial calculation engine.
- Membangun dashboard.
- Membangun customer insight engine.
- Menambahkan audit trail dan pengujian.

### Phase 4: Pilot

Durasi indikatif: 2–4 minggu.

Aktivitas:

- Menguji produk bersama sejumlah kecil UMKM.
- Mengukur kelengkapan data.
- Memvalidasi perhitungan.
- Menguji apakah insight dapat dipahami.
- Mengukur penggunaan berulang.
- Menguji kesediaan membayar.

### Exit Criteria

MVP layak dikembangkan lebih lanjut jika:

- Perhitungan inti lulus seluruh test cases yang ditetapkan.
- Isolasi data antarbisnis telah diuji.
- Impor data memiliki penanganan error yang dapat dipahami.
- Insight dapat ditelusuri ke sumber data.
- Pengguna dapat menyelesaikan alur inti tanpa bantuan intensif.
- Ada bukti bahwa pengguna memperoleh manfaat yang cukup untuk kembali menggunakan produk.
- Terdapat indikasi kesediaan membayar yang dapat diuji lebih lanjut.

---

## 13. Risks and Mitigations

### 13.1. Poor Data Quality

**Risk:** Data historis tidak lengkap atau formatnya tidak konsisten.

**Mitigation:**

- Menyediakan template.
- Memvalidasi kolom dan nilai.
- Menampilkan tingkat kelengkapan.
- Menghindari kesimpulan yang membutuhkan data yang tidak tersedia.

### 13.2. Incorrect Financial Interpretation

**Risk:** Pengguna menganggap omzet sebagai laba atau menganggap laba kotor sebagai laba bersih.

**Mitigation:**

- Memisahkan metrik keuangan.
- Menampilkan formula.
- Menjelaskan biaya yang belum dimasukkan.
- Menggunakan label "estimasi" jika diperlukan.

### 13.3. False Customer Risk Signals

**Risk:** Pelanggan diberi label berisiko meskipun pola pembeliannya memang tidak teratur.

**Mitigation:**

- Menggunakan histori individual jika memadai.
- Menampilkan dasar analisis.
- Menghindari label pasti.
- Menguji aturan pada data nyata yang telah dianonimkan.

### 13.4. Low Willingness to Pay

**Risk:** Pengguna menyukai produk tetapi tidak merasa manfaatnya cukup besar untuk membayar.

**Mitigation:**

- Menguji willingness to pay sebelum memperluas fitur.
- Mengukur waktu yang dihemat dan keputusan yang terbantu.
- Menawarkan pilot berbayar jika memungkinkan.
- Memprioritaskan satu segmen bisnis.

### 13.5. Excessive AI Costs

**Risk:** Biaya penggunaan model lebih tinggi daripada nilai langganan.

**Mitigation:**

- Menggunakan formula dan rules untuk tugas deterministik.
- Memanggil LLM hanya jika diperlukan.
- Membatasi konteks yang dikirim.
- Memantau biaya per bisnis.
- Menyediakan batas penggunaan yang transparan.

### 13.6. Privacy and Security

**Risk:** Data keuangan dan informasi pelanggan bocor atau diakses oleh pihak yang tidak berwenang.

**Mitigation:**

- Business-level authorization.
- Least-privilege access.
- Audit trail.
- Enkripsi saat transmisi.
- Pengujian isolasi data.
- Kebijakan retensi dan penghapusan data.

---

## 14. Product Roadmap

### Stage 1: Business Intelligence Foundation

Fokus:

- Data produk dan transaksi.
- HPP dan kalkulasi harga.
- Dashboard keuangan.
- Riwayat transaksi pelanggan.
- Insight berbasis aturan.
- CSV import.

### Stage 2: AI Business Assistant

Fokus:

- Pertanyaan dalam bahasa alami.
- Penjelasan performa bisnis.
- Rekomendasi tindakan yang lebih kontekstual.
- Ringkasan bisnis berkala.
- Perbaikan kualitas insight.

### Stage 3: Conversational Business Input

Fokus:

- Integrasi WhatsApp Business Platform.
- Input transaksi melalui percakapan.
- Validasi dan konfirmasi.
- Ringkasan dan query bisnis melalui WhatsApp.

### Stage 4: Operational Automation

Fokus:

- Integrasi dengan kanal penjualan yang didukung.
- Tindak lanjut pelanggan.
- Pengelolaan tugas operasional.
- Notifikasi dan reminder.
- Approval workflow.

### Stage 5: AI Business Operator

Fokus:

- Mengusulkan rencana tindakan berdasarkan tujuan bisnis.
- Menyusun prioritas pekerjaan.
- Menjalankan tindakan yang diizinkan.
- Memantau hasil dan memberi laporan.
- Memperluas otomasi dengan audit trail dan kontrol pengguna.

Setiap tahap harus bergantung pada bukti kebutuhan pengguna, kelayakan integrasi, keamanan, dan hasil tahap sebelumnya.

---

## 15. Definition of Done

Sebuah fitur dianggap selesai jika:

- Acceptance criteria terpenuhi.
- Validasi input dan error handling tersedia.
- Otorisasi dan isolasi data telah diuji.
- Unit test dan integration test yang relevan lulus.
- Perubahan database memiliki migration yang jelas.
- Perubahan penting memiliki audit trail jika diperlukan.
- Dokumentasi penggunaan dan API diperbarui.
- Tidak ada secret yang tertanam dalam source code.
- UI menampilkan loading, empty, success, dan error states.
- Perhitungan dan insight dapat ditelusuri ke data sumber.
- Tidak ada klaim bisnis yang tidak didukung data.

---

## 16. Open Questions

Keputusan berikut harus divalidasi sebelum pengembangan penuh:

1. Segmen awal: usaha makanan, reseller, atau salah satu subsegmen yang lebih spesifik?
2. Format data transaksi apa yang paling umum digunakan oleh segmen tersebut?
3. Seberapa lengkap data HPP yang dimiliki calon pengguna?
4. Apakah pelanggan lebih membutuhkan analisis profitabilitas atau tindak lanjut pelanggan?
5. Seberapa besar kesediaan membayar untuk produk?
6. Apakah pengguna membutuhkan pencatatan manual, impor spreadsheet, atau integrasi kanal sejak awal?
7. Apa batas minimum data agar insight pelanggan dapat dianggap layak?
8. Model harga apa yang sesuai dengan nilai yang diberikan?
9. Bagaimana menangani transaksi tunai, kredit, retur, diskon, dan biaya tambahan?
10. Apakah AI assistant dibutuhkan sejak MVP atau baru setelah pengguna memahami dashboard dasar?

---

## 17. Final Product Definition

Teman UMKM adalah platform pendukung keputusan bisnis yang membantu pemilik usaha memahami profitabilitas, aktivitas pelanggan, dan prioritas tindakan berdasarkan data operasional yang tersedia.

MVP berfokus pada fondasi data yang akurat, perhitungan yang transparan, insight yang dapat dijelaskan, dan rekomendasi yang tetap berada di bawah kendali pemilik usaha.

Produk tidak dimulai sebagai chatbot serba bisa. Produk dimulai sebagai sistem yang dapat dipercaya, kemudian berkembang menjadi asisten AI yang mampu membantu pekerjaan bisnis secara bertahap.

**Prinsip utama produk:**

> Teman UMKM tidak hanya membantu pemilik usaha mengetahui apa yang terjadi dalam bisnisnya, tetapi juga membantu mereka memahami alasannya dan menentukan langkah berikutnya berdasarkan bukti.
