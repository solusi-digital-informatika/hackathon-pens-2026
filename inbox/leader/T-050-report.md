# Laporan Penyelesaian T-050: UI Specification for CSV Import Wizard

- **Task ID**: T-050
- **Assignee**: `designer`
- **Deliverable**: `contracts/ui-csv-import.md`
- **Status**: DONE / APPROVED

Telah selesai disusun spesifikasi UI/UX komprehensif untuk wizard impor berkas CSV 5-langkah (*CSV Import Wizard*) pada `contracts/ui-csv-import.md`, yang sepenuhnya mematuhi prinsip desain Clean Pragmatic Craftsman, Design Tokens (`#0D6E64` Emerald Forest), dan TRD §8 CSV Import Spec:

1. **Alur 5-Langkah Lengkap**:
   - **Langkah 1 (Upload)**: Dropzone drag-and-drop berkas `.csv` (maksimal 5 MB / 20.000 baris), selector kartu jenis entitas (`products`, `customers`, `transactions`), tautan unduh template resmi UTF-8, dropdown penimpaan pembatas (*delimiter override*: auto-detect, koma, titik koma Excel, tab), dan banner deteksi berkas duplikat berbasis hash SHA-256.
   - **Langkah 2 (Mapping & Parsing)**: Tabel pemetaan kolom sumber CSV ke field target sistem dengan indikator wajib vs opsional, selector format angka Indonesia (`1.500,50`) vs internasional (`1,500.50`), selector format tanggal (`DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`), dan komponen pratinjau dinamis 3 baris pertama (*live preview*).
   - **Langkah 3 (Validation & Inspection)**: Kartu rangkuman baris valid (hijau), peringatan (amber), dan galat (merah); filter interaktif segmented baris data; penjelasan galat spesifik berbahasa Indonesia; modul resolusi angka ambigu; serta modul penanganan kecocokan nama pelanggan ("Kemungkinan Pelanggan yang Sama") dengan larangan penggabungan otomatis tanpa kecocokan nomor kontak sesuai TRD §8.
   - **Langkah 4 (Confirmation & Progress)**: Modal konfirmasi eksekusi transaksi terisolasi (*savepoint per row*), bilah kemajuan asinkron (*progress bar*) dengan penghitung dinamis `X dari Y baris`, indikator status polling pekerja latar belakang (interval 1,5 detik), dan tombol pembatalan antrean aman.
   - **Langkah 5 (Summary & Rollback)**: Tampilan status akhir (`completed`, `completed_with_errors`, `failed`), statistik baris, tombol unduh laporan galat dengan proteksi penetrasi formula CSV (*formula injection neutralization* untuk `=`, `+`, `-`, `@`), tombol navigasi cepat ke modul data terkait, serta mekanisme *rollback* penuh dengan dialog konfirmasi verifikasi ketik kata kunci `BATALKAN` yang mencatat log audit permanen.
2. **Responsivitas & Anti-Slop**:
   - Tata letak adaptif untuk Mobile (390px - *stacked mapping cards* & *sticky first-column scroll*), Tablet (768px - 3-kolom grid), dan Desktop (1440px - *split mapping & live preview*).
   - Seluruh kartu berstatus diam (*resting*) menggunakan kontras batas permukaan tanpa bayangan dekoratif, kontras warna memenuhi standar WCAG AA, dan tipografi memadukan Plus Jakarta Sans (UI) serta Inter (data tabular/angka).
   - Mendukung penuh navigasi keyboard dan preferensi pengguna *reduced motion*.
