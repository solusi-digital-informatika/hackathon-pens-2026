# Laporan Penyelesaian Tugas T-001: Design Direction and Tokens

**Kepada:** Leader  
**Dari:** Designer  
**Status Tugas:** Selesai (`done`)  
**File Dihasilkan:**
1. `contracts/design-direction.md`
2. `contracts/design-tokens.md`

---

## 1. Ringkasan Eksekusi

Sesuai arahan tugas T-001 serta ketentuan PRD, TRD, dan SOUL Designer:

1. **Art Direction (`contracts/design-direction.md`)**:
   - Memenuhi batas ketat (maksimal 10 baris), merangkum target audiens pemilik UMKM fisik di Indonesia, pertimbangan 3 opsi gaya dan pemilihan arah *Clean Pragmatic Craftsman*, 2 keluarga huruf (*Plus Jakarta Sans* untuk display/UI dan *Inter* untuk tabular figures numerik), 1 warna aksen utama (*Emerald Forest* `#0D6E64`) dengan palet netral bertinting sage, aturan visual tanpa stok generik, momen khas (*Kartu Kepastian Laba* dengan status cakupan biaya), serta aturan kepadatan antarmuka (kompak untuk tabel/kasir, lapang untuk onboarding).

2. **Design Tokens (`contracts/design-tokens.md`)**:
   - Didefinisikan dalam format CSS Custom Properties (`:root`) untuk diimplementasikan langsung oleh tim Frontend di T-003.
   - **Skala Spasi**: 4-pt grid (4, 8, 12, 16, 24, 32, 48, 64px).
   - **Radius**: Skala konsisten dan lugas (0, 2, 4, 6, 8, 12, 9999px) tanpa bentuk pill sembarangan.
   - **Tipografi**: Skala modular 1.25 (xs 12px hingga 3xl 36px) dengan konfigurasi font numerik tabular (`tnum`).
   - **Warna Semantik & Kontras**: Menjamin kepatuhan standar WCAG AA (kontras teks ≥ 4.5:1, elemen grafis/UI ≥ 3:1).
   - **Anti-Slop Elevation**: Tidak ada bayangan (*no shadows*) pada resting card; pemisahan visual menggunakan kontras border halus dan warna permukaan. Bayangan dikhususkan hanya untuk lapisan terangkat (*dropdown, modal, toast, drag*).
   - **Motion Tokens**: Durasi terukur (micro 120ms, standard 240ms, page 350ms), kurva easing masuk/keluar yang fungsional tanpa efek membal, serta deklarasi wajib `@media (prefers-reduced-motion: reduce)`.
   - **Z-Index Scale**: Terstandarisasi dari `-1` hingga `70`.
   - **Format Finansial & Standar Istilah**: Panduan `formatRupiah` (`Intl.NumberFormat('id-ID')`), keharusan penulisan label "Laba Kotor" (bukan hanya "Laba"), badge "Estimasi" saat `cost_coverage < 100%`, serta penanganan "Tidak tersedia" untuk laba bersih MVP.

3. **Status Papan Kerja & Arsip**:
   - Status T-001 pada `board/board.md` telah diperbarui menjadi `done`.
   - File tugas `T-001-design-direction-and-tokens.md` telah dipindahkan ke `inbox/designer/done/`.

Tokens dan Design Direction kini siap digunakan oleh agen Frontend untuk pengerjaan T-003.
