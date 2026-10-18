# Laporan Penyelesaian T-061: API Contract for Insight Rules Engine, Recommendations, Provenance & Cooldown

- **Task ID**: T-061
- **Assignee**: `backend-ai`
- **Deliverable**: `contracts/api-insights-recommendations.md` (API-M6-001)
- **Status**: DONE / APPROVED
- **Referensi Terkait**: `project-map/trd.md` (§7 Insight Engine Spec, §9 API Inventory, §10 Security & Rate Limits, TD-05, TD-09), `contracts/erd.md` (§5 Diagram D), `contracts/ui-insights-recommendations.md` (UI-M6-001)

---

## 1. Ringkasan Eksekutif
Telah selesai dirancang dan didokumentasikan kontrak antarmuka REST API komprehensif pada `contracts/api-insights-recommendations.md` untuk modul **Insight Engine & Recommendations** (Milestone M6). Kontrak ini mendefinisikan secara tuntas 5 endpoint RESTful, pemisahan DTO request/response dari model GORM, katalog kode galat, protokol pengabaian wawasan ber-jeda (*dismissal with 14-day cooldown*), pembatasan pemicuan manual (10 menit per bisnis), serta spesifikasi teknis 8 aturan deterministik awal sesuai TRD §7.

---

## 2. Rincian Ruang Lingkup yang Telah Ditetapkan

1. **5 Endpoint RESTful Komprehensif**:
   - `POST /api/v1/businesses/{business_id}/analysis/run`: Memicu eksekusi pipeline analisis deterministik manual dengan batas rate limit 1 run per 10 menit dan pencegahan konkurensi ganda (`409 Conflict` jika run berstatus `running` masih aktif; `429 Too Many Requests` dengan header `Retry-After: <detik>` jika jeda cooldown belum terpenuhi).
   - `GET /api/v1/businesses/{business_id}/insights`: Mengambil daftar wawasan terpaginasi dengan filter `type`, `claim_type`, `priority`, `is_active` (`superseded_at IS NULL`), `subject_type`, dan `subject_id`.
   - `GET /api/v1/businesses/{business_id}/insights/{id}`: Menyajikan rincian lengkap satu wawasan, asumsi metodologi, limitasi, data rekomendasi tertaut, serta daftar bukti riwayat data mentah pembentuk (*insight sources*) yang mencakup ID, nomor referensi transaksi/komplain/produk, tanggal, nominal, dan ringkasan.
   - `GET /api/v1/businesses/{business_id}/recommendations`: Mengambil daftar seluruh rekomendasi tindakan terpaginasi dengan filter `status` (`new`, `reviewed`, `in_progress`, `completed`, `dismissed`), `priority`, dan `insight_type`.
   - `PATCH /api/v1/businesses/{business_id}/recommendations/{id}`: Memperbarui status rekomendasi. Jika status diubah menjadi `dismissed`, field `dismissed_reason` wajib diisi, dan sistem otomatis mengaktifkan jeda 14 hari (*cooldown*) untuk mencegah pembuatan ulang wawasan dengan `dedupe_key` yang sama selama periode tersebut.

2. **Spesifikasi 8 Aturan Deterministik Awal (TRD §7)**:
   - `CUST_OVERDUE_V1`: Mendeteksi pelanggan aktif (≥ 4 tanggal belanja dalam 180 hari) yang melampaui interval waktu belanja biasanya (`days_since_last > max(1.5 × median, median + 7)`). Status klaim: `hypothesis`, base priority: `medium`.
   - `CUST_FREQ_DROP_V1`: Mendeteksi penurunan frekuensi transaksi belanja pelanggan (≥ 4 transaksi pada H-91 s/d H-180, dan transaksi 90 hari terakhir ≤ 0.5 × periode sebelumnya). Status klaim: `hypothesis`, base priority: `medium`.
   - `CUST_VALUE_DROP_V1`: Mendeteksi penurunan nilai rata-rata pesanan pelanggan (≥ 4 transaksi sebelumnya dan ≥ 2 transaksi baru-baru ini, dengan rata-rata nominal belanja recent ≤ 0.6 × previous). Status klaim: `hypothesis`, base priority: `low`.
   - `CUST_COMPLAINT_OPEN_V1`: Mendeteksi komplain pelanggan berstatus `open` atau `in_progress` yang belum terselesaikan lebih dari 3 hari. Status klaim: `fact`, base priority: `high`.
   - `PROD_BELOW_COST_V1`: Mendeteksi produk aktif dengan harga jual saat ini berada di bawah HPP per unit aktif (`current_selling_price < hpp_per_unit`). Status klaim: `fact`, base priority: `high`.
   - `PROD_LOW_MARGIN_V1`: Mendeteksi produk dengan margin penjualan tertinggal ≥ 10 poin persentase di bawah margin rata-rata usaha. Status klaim: `estimate`, base priority: `medium`.
   - `DATA_MISSING_COST_V1`: Mendeteksi penjualan produk yang belum dilengkapi catatan HPP (`cost_coverage < 100%`). Status klaim: `fact`, base priority: `medium`.
   - `DATA_UNLINKED_TX_V1`: Mendeteksi transaksi tanpa tautan pelanggan yang melebihi 30% dari total transaksi periode 30 hari. Status klaim: `fact`, base priority: `low`.

3. **Aturan Prioritas & Eskalasi (Top 20% Revenue Customer)**:
   - Pelanggan yang masuk kategori kontributor omzet 20% teratas usaha dalam 180 hari terakhir otomatis mendapatkan eskalasi prioritas wawasan naik 1 tingkat (`low` → `medium`, `medium` → `high`).

4. **Siklus Hidup Wawasan, Dedupe Key, Cooldown & Superseding**:
   - Pemutakhiran in-place wawasan jika kondisi anomali masih sama pada run berikutnya.
   - Penandaan `superseded_at = NOW()` jika kondisi anomali telah teratasi secara otomatis oleh transaksi baru.
   - Penegakan jeda 14 hari (*14-day cooldown*) paska aksi `dismissed` berbasis `dedupe_key`.

5. **Pemisahan DTO & Keamanan Data**:
   - DTO Go terdefinisi lengkap dengan validator tags Gin (`binding:"required,oneof=..."`).
   - Format moneter konsisten string desimal (`NUMERIC(15,2)`), tanggal ISO 8601 `YYYY-MM-DD`, waktu RFC 3339 UTC.
   - Isolasi tenant ketat di setiap layer (`business_id` scoping dan composite foreign keys).

Dokumen kontrak `contracts/api-insights-recommendations.md` telah siap menjadi acuan implementasi modul backend (T-062) dan modul frontend (T-063).
