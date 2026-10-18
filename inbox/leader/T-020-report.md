# Laporan Penyelesaian Tugas T-020: UI Spec for Products, Cost Versions & Pricing Calculator

**Kepada:** Leader  
**Dari:** Designer  
**Status Tugas:** Selesai (`done`)  
**File Dihasilkan:** `contracts/ui-products.md`  

---

## 1. Ringkasan Eksekusi

Sesuai instruksi pada `inbox/designer/T-020-ui-spec-products.md`, PRD (§5.2, §5.3, §6.2), TRD (§5 TD-08 & TD-12), ERD (`products`, `product_cost_versions`, `product_cost_components`, `product_price_history`), serta pedoman SOUL Designer:

1. **Spesifikasi Antarmuka & Interaksi Lengkap (`contracts/ui-products.md`)**:
   - **Feature → Page Map**:
     - **Katalog Produk (`/products`)**: Monitoring harga, status HPP, margin kotor, filter kategori, status aktif/nonaktif, dan pencarian cepat.
     - **Modal Tambah & Ubah Produk**: Pendaftaran SKU, nama, kategori, satuan, harga jual terkini, serta inisialisasi versi HPP pertama.
     - **Histori HPP & Riwayat Harga Jual (Slide Drawer)**: Audit trail berurutan waktu atas versi biaya (*cost versioning* bersifat *append-only* sesuai TD-08) dan log perubahan harga jual.
     - **Kalkulator Penetapan Harga Interaktif**: Simulasi harga jual berbasis target margin vs target markup dengan deteksi risiko margin tipis (< 15%) atau harga jual di bawah HPP.

2. **Komponen Antarmuka Bersama**:
   - `PageHeader`: Menampung aksi primer tunggal (`+ Tambah Produk`) dan aksi sekunder (`Kalkulator Harga`).
   - `FilterToolbar`: Input pencarian dinamis, dropdown kategori, filter status, dan filter kelengkapan HPP.
   - `ProductTable` (Desktop/Tablet): 8 kolom data dengan font tabular (`Inter` `.tnum`) dan pemformatan `formatRupiah` (`id-ID`).
   - `ProductCard` (Mobile 390px): Tampilan kartu bertumpuk untuk menggantikan tabel desktop demi menghindari scroll horizontal yang membingungkan.
   - `ModalContainer` & `DrawerSheet`: Modal terpusat di desktop dan bottom sheet penuh (92vh) di mobile dengan drag handle serta sticky footer.
   - `StatusBadge`: Indikator semantik Aktif, Nonaktif, Tanpa HPP (`warning`), Margin Tipis (`warning`), dan Di Bawah HPP (`error`).

3. **Kalkulator Penetapan Harga Interaktif**:
   - Mendukung dua mode simulasi:
     1. Menentukan Rekomendasi Harga Jual dari Target Margin (%) atau Target Markup (%).
     2. Menguji Margin & Markup dari Rencana Harga Jual tertentu.
   - Menggunakan formula deterministik PRD §5.3 dan TRD §5 (TD-12):
     - $\text{Harga Jual} = \frac{\text{HPP}}{1 - \text{Margin}}$
     - $\text{Laba Kotor} = \text{Harga Jual} - \text{HPP}$
   - Dilengkapi preset cepat industri UMKM (20% Ritel, 35% F&B, 50% Premium), opsi pembulatan psikologis (ke Rp 500 / Rp 1.000 terdekat), dan integrasi tombol *"Terapkan ke Produk"*.
   - **Indikator Risiko Visual**:
     - Merah (`--color-error-bg` / `border`): Jika harga jual $\le$ HPP dengan peringatan nominal rugi kotor per unit.
     - Amber / Kuning (`--color-warning-bg` / `border`): Jika margin kotor < 15% (margin tipis rentan fluktuasi biaya).
     - Hijau: Margin sehat operasional ($\ge 30\%$).

4. **Penerapan Anti-Slop & Kepatuhan UX**:
   - Dilarang keras menggunakan box-shadow pada kartu istirahat (*resting cards*); pemisahan menggunakan border `1px solid var(--color-border-subtle)` dan permukaan kontras.
   - Tanpa gradien dekoratif AI; aksen tunggal adalah Emerald Forest (`#0D6E64`).
   - Touch targets $\ge 44\times 44$px pada semua kontrol dan aksi baris mobile.
   - Aksesibilitas WCAG AA: Rasio kontras teks tinggi, label formulir selalu terlihat, focus trapping pada modal/drawer, `aria-live="polite"` pada kalkulator, dan dukungan `prefers-reduced-motion`.

5. **Status Papan Kerja & Arsip**:
   - Status T-020 pada `board/board.md` telah diperbarui menjadi `done`.
   - File tugas `inbox/designer/T-020-ui-spec-products.md` telah dipindahkan ke `inbox/designer/done/T-020-ui-spec-products.md`.

---

## 2. Rujukan Dokumen untuk Tim

- UI/UX Spec: `contracts/ui-products.md`
- Design Tokens: `contracts/design-tokens.md`
- Art Direction: `contracts/design-direction.md`
- Frontend siap mengimplementasikan T-023 mengacu pada spesifikasi ini.
