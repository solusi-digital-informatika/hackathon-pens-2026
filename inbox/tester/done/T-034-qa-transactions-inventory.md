# Task T-034: QA Verification for Transaction Idempotency & Inventory Integrity

- **Owner**: `tester`
- **Milestone**: M3 — Customers, Transactions & Inventory Management
- **Size**: Standard
- **References**:
  - `contracts/ui-transactions-customers.md`
  - `contracts/api-transactions-customers.md`
  - Backend commit: `ff5c612`
  - Frontend commit: `d6ffd5a`

## 1. Scope of Audit
1. Verifikasi menyeluruh backend:
   - Kompilasi biner (`go build ./cmd/api`).
   - Eksekusi unit test suite (`go test -v ./...`).
   - Validasi logika idempotensi transaksi: verifikasi header `Idempotency-Key` / `external_ref` mencegah pemotongan stok ganda.
   - Validasi integritas stok: pemotongan saat checkout, penolakan saat stok tidak mencukupi, pemulihan saat transaksi dibatalkan atau diretur.
   - Validasi penguncian HPP snapshot (`recorded_unit_cost`) saat checkout.
   - Validasi isolasi tenant pelanggan dan transaksi lintas bisnis.
2. Verifikasi menyeluruh frontend:
   - Kompilasi build bundle Vite dan pemeriksaan type strict TypeScript (`npm run build`).
   - Eksekusi unit test Vitest (`npm test -- --run`).
   - Validasi antarmuka kasir POS: proteksi stok di keranjang, kalkulasi subtotal, diskon, biaya tambahan, uang kembalian.
   - Validasi antarmuka riwayat transaksi: detail nota invoice, alur konfirmasi pembatalan, dan form retur item.
3. Terbitkan laporan audit resmi QA dengan vonis APPROVE / REJECT di `inbox/leader/T-034-report.md`.
