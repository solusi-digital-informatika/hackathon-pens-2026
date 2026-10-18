# Task: T-052 Implement Backend CSV Parser, Async Batch Worker & Import Rollbacks

## 1. Objective
Implement the complete backend CSV Import module in `office-hub/backend` according to `contracts/api-csv-import.md`, `contracts/erd.md`, and TRD §8/TD-07/TD-11. Include migrations for `import_batches` and `import_rows`, streaming CSV parser, column mapping, row validation engine, async background worker (`SELECT ... FOR UPDATE SKIP LOCKED` with row savepoints), rollback support, and error CSV export with formula neutralization.

## 2. Context & Reference Files
- Repo path: `C:/Project/ai-office/office-hub/backend`
- Remote: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
- Hub Specifications:
  - `contracts/api-csv-import.md`: Complete API specification for all 8 endpoints and schemas.
  - `contracts/erd.md`: §4 Diagram C (`import_batches`, `import_rows`, enums, composite foreign keys).
  - `project-map/trd.md`: §8 CSV Import Spec, §9 API Inventory, §10 Security, TD-07 Async Work, TD-11 Audit.
- Backend-AI SOUL: Go clean architecture (`internal/importer/` with handler, service, repository, parser, validator, worker; thin handlers, no raw CSV file storage on disk, row-level savepoints, table-driven unit & integration tests).

## 3. Expected Deliverables
In repository `C:/Project/ai-office/office-hub/backend`:
1. **Migrations**:
   - `migrations/000004_create_import_tables.up.sql` and `down.sql`:
     - `import_batches`: id (UUIDv7), business_id (FK), user_id (FK), import_type (products, customers, transactions), status (uploaded, mapped, validated, confirmed, processing, completed, completed_with_errors, failed, cancelled), file_name, file_size_bytes, file_sha256, delimiter, number_format, date_format, column_mapping (jsonb), total_rows, valid_rows, warning_rows, error_rows, imported_rows, processed_rows, error_message, started_at, completed_at, timestamps.
     - `import_rows`: id (UUIDv7), import_batch_id (FK), row_number (int), raw_data (jsonb), parsed_data (jsonb), status (pending, valid, warning, error, imported, skipped), validation_errors (jsonb), entity_id (UUID), timestamps.
     - Composite indexes per `contracts/erd.md`.
2. **Streaming Parser & Security (`internal/importer/parser.go`)**:
   - Streaming reader: limits enforcement (max 5 MB, max 20,000 rows).
   - Zero physical file storage: stream directly into hash computation and staging rows.
   - UTF-8 BOM stripping.
   - Auto-detection of delimiter (comma, semicolon, tab).
   - Anti-formula injection on error CSV export: prepending `'` to cells starting with `=`, `+`, `-`, `@`, `\t`, `\r`.
3. **Validation Engine (`internal/importer/validator.go`)**:
   - Indonesian number format (`1.500,50`) and international (`1,500.50`) parsing.
   - Date format parsing (`DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`).
   - Exact product matching by SKU or exact trimmed name (NO fuzzy matching).
   - Exact customer matching by normalized E.164 phone (`+62...`). Name-only similarity emits warning `CUSTOMER_NAME_SIMILAR`, never auto-merges.
   - Products import: optional `hpp_per_unit` creates cost version with component "HPP (impor)".
   - Transactions import: rows sharing `order_ref` group into one transaction.
4. **Async Worker Engine (`internal/importer/worker.go`)**:
   - Background worker claiming batches via `SELECT ... FOR UPDATE SKIP LOCKED` on `import_batches WHERE status = 'confirmed'`.
   - Chunked processing (200 rows) with row-level savepoints (`SAVEPOINT import_row_savepoint`).
   - Graceful shutdown handling (`context.WithCancel`).
5. **Rollback & Cancellation (`internal/importer/service.go`)**:
   - Cancel prior to processing marks status `cancelled`.
   - Destructive rollback for completed transaction imports: cancels created transactions, restores inventory, and logs audit trail. Requires verification string `BATALKAN`.
6. **HTTP Handlers & Server Wiring**:
   - Register routes in `internal/platform/httpserver/server.go` under `/api/v1/businesses/:business_id/imports/...` protected by `RequireAuth` and `RequireTenant`.
7. **Automated Tests**:
   - Unit tests for delimiter detection, BOM strip, number/date formatting, and formula neutralization.
   - Integration / handler tests for upload, mapping, validation, confirm, cancel, and error CSV download.
   - `go test -v ./...` and `go build ./cmd/api` must pass cleanly.

## 4. Constraints
- Strict adherence to `contracts/api-csv-import.md`.
- Never store raw uploaded CSV files on disk or in DB (only SHA-256 and parsed rows).
- Commit with Conventional Commits: `feat(importer): implement streaming CSV parser, async worker, and rollback`.
- Push to `origin main` and verify with `git status -sb`.

## 5. Definition of Done
- Migrations apply cleanly.
- Unit and integration tests pass 100%.
- Biner compiles successfully (`go build ./cmd/api`).
- Pushed commit hash reported to `inbox/leader/T-052-report.md`.
- Move this task to `inbox/backend-ai/done/`.
