# Task: T-024 QA Verification for Products, Cost Versions & Pricing Calculator

## Objective
Lakukan audit kualitas komprehensif dan pengujian verifikasi independen untuk Milestone M2 (Katalog Produk, Riwayat HPP & Harga, Presisi Desimal, Formula Kalkulator Harga, dan Isolasi Akses Tenant).

## Referensi & Kontrak
- `contracts/ui-products.md`
- `contracts/api-products.md`
- `contracts/design-tokens.md`
- `backend/migrations/000003_create_products_and_pricing_tables.up.sql`
- `backend/internal/product/`
- `frontend/src/modules/products/`

## Matriks Uji QA (Harus Diuji & Diverifikasi)
1. **Presisi Desimal & Ketahanan Tipe Data**:
   - Nilai mata uang (harga jual, HPP, laba) menggunakan `NUMERIC(15,2)` di database dan string di JSON API untuk mencegah distorsi floating point.
   - Stok barang menggunakan `NUMERIC(12,3)` (misalnya mendukung unit pecahan seperti 0.750 kg).
2. **Integritas Append-Only Riwayat HPP & Harga Jual**:
   - Pembuatan produk awal membuat rekaman awal di `cost_versions` dan `price_history`.
   - Update produk dengan harga baru menambah rekaman baru di `price_history` tanpa menghapus riwayat lama.
   - Update produk dengan HPP baru menambah rekaman baru di `cost_versions` tanpa menghapus riwayat lama.
   - Perubahan tanpa mengubah nominal harga/HPP tidak boleh membuat duplikasi riwayat palsu.
3. **Akurasi Formula Kalkulator Harga (TRD TD-12)**:
   - Margin: $P = \frac{C}{1 - M}$
   - Markup: $P = C \times (1 + U)$
   - Warning margin berisiko ketika margin $< 15\%$ atau harga $\le$ HPP.
4. **Isolasi Keamanan & Tenant (OWASP API1 / BOLA)**:
   - Tenant A tidak dapat melihat, mengubah, atau mengambil riwayat produk Tenant B.
   - Rute kalkulator di bawah bisnis hanya dapat diakses oleh member bisnis terkait.
5. **Kualitas Frontend & Desain**:
   - Kesesuaian UI dengan Clean Pragmatic Craftsman (tanpa bayangan kartu istirahat, aksen Emerald Forest `#0D6E64`, format Rupiah `id-ID`).
   - Linting TypeScript bebas error dan build production bersih.

## Output yang Diharapkan
Tulis laporan hasil verifikasi lengkap ke `inbox/leader/T-024-report.md` dengan status akhir (APPROVE / REVISE) beserta bukti eksekusi test runner (Go test & Vitest).
