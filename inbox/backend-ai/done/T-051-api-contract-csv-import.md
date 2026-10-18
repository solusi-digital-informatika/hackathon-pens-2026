# Task: T-051 API Contract for CSV Import, Mapping, Row Validation & Async Worker

## 1. Objective
Author complete and authoritative REST API contract in `contracts/api-csv-import.md` covering all import endpoints, schemas, validation errors, and asynchronous worker execution protocol based on TRD §8, TRD §9, and `contracts/erd.md`.

## 2. Context & Reference Files
- `project-map/trd.md` (§8 CSV Import Spec, §9 API Inventory, §10 Security, TD-07 Async Work, TD-11 Audit).
- `contracts/erd.md` (§4 Diagram C: `import_batches`, `import_rows`, enums, composite foreign keys).
- Endpoints to specify:
  - `GET /api/v1/businesses/{business_id}/imports/templates/{type}`
  - `POST /api/v1/businesses/{business_id}/imports` (Multipart CSV upload, size/type check, SHA-256 calculation, auto delimiter detection)
  - `GET /api/v1/businesses/{business_id}/imports/{id}` (Batch status, summary counts, parsing settings)
  - `PUT /api/v1/businesses/{business_id}/imports/{id}/mapping` (Column mapping & format options: number format, date format)
  - `GET /api/v1/businesses/{business_id}/imports/{id}/rows` (Paginated rows, filter by status: valid, warning, error, pending)
  - `POST /api/v1/businesses/{business_id}/imports/{id}/confirm` (Idempotent trigger to async worker)
  - `POST /api/v1/businesses/{business_id}/imports/{id}/cancel` (Cancel batch or rollback created transactions)
  - `GET /api/v1/businesses/{business_id}/imports/{id}/errors/download` (Download CSV of errored rows with neutralized formulas)

## 3. Scope of API Contract
1. **Request & Response DTOs**:
   - Exact JSON schemas, data types, and enum values for batch statuses:
     `uploaded`, `mapped`, `validated`, `confirmed`, `processing`, `completed`, `completed_with_errors`, `failed`, `cancelled`.
   - Row statuses: `pending`, `valid`, `warning`, `error`, `imported`, `skipped`.
2. **Parsing & Security Guardrails**:
   - Max file size 5 MB, max 20,000 rows.
   - Streaming text parser with UTF-8 BOM stripping.
   - SHA-256 hash detection for warning on duplicate upload.
   - Strict formula injection neutralization for error export (`=`, `+`, `-`, `@`).
3. **Validation Error Catalog**:
   - Structured error objects `{code, field, message, row_number}` with clear Indonesian messages.
   - Product match by exact SKU or exact trimmed name (no fuzzy match).
   - Customer match only by normalized E.164 phone (`08...` -> `+628...`).
4. **Async Execution Protocol (TD-07)**:
   - Database as queue: `SELECT ... FOR UPDATE SKIP LOCKED` on `import_batches`.
   - Chunk processing with row savepoints.
   - Audit trail on confirmation and cancellation.

## 4. Expected Output
- Author `contracts/api-csv-import.md`.
- Update `board/board.md` T-051 status to `done`.
- Write completion report to `inbox/leader/T-051-report.md`.
- Move this task to `inbox/backend-ai/done/T-051-api-contract-csv-import.md`.
