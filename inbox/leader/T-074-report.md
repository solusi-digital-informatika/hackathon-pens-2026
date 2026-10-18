# QA Audit & Security Hardening Verification Report: Milestone M7

- **Task ID**: T-074
- **Assignee**: `tester` (QA & Security Tester)
- **Target Milestone**: Milestone M7 (Hardening, Security Pass, Seed CLI & Production Readiness)
- **Status**: COMPLETED
- **Final Verdict**: **PASS / APPROVE**
- **Date**: 19 Oktober 2026

---

## 1. Scope & Tested Commits

### 1.1 Repositories and Commits Under Test
- **Backend Repository**: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
  - Coder Pushed Commit (`backend-ai`): `c4a7389` (`feat(security): implement cmd/seed, route inventory authorization tests, and security hardening`)
  - QA Verified & Bug-Fixed Commit (`tester`): `fcb4f0d` (`fix(finance): fix postgresql status and uuid column syntax in data health query`)
- **Frontend Repository**: `https://github.com/solusi-digital-informatika/teman-umkm-frontend.git`
  - Coder Pushed Commit (`frontend`): `ec4e1ee` (`feat(security): implement global error boundary, offline detector, and safe export dialog`)
- **Reference Contracts & Specifications**:
  - `contracts/api-hardening-security.md` (API-M7-001)
  - `contracts/ui-hardening-security.md` (UI-M7-001)
  - `project-map/trd.md` (§10 Security, §12 Non-Functional Targets & Performance, §14 Manual Tasks)

### 1.2 Coverage Scope
- **Covered**:
  - Automated Fail-Closed Route Inventory Test (46 declared HTTP routes against Gin router).
  - Multi-tenant isolation verification (Anonymous -> 401, Cross-Tenant -> 403 Forbidden).
  - Role-based authorization enforcement (Staff -> 403 on Owner-only endpoints).
  - Production HTTP security headers, request body size limiter (413 >1MB), and IP/account sliding window rate limiter (429 with `Retry-After`).
  - Standalone Seed CLI tool (`cmd/seed` minimal & standard scale up to 10,000 transactions).
  - Performance benchmarks on 10,000 transactions dataset (Standard CRUD p95 and Dashboard aggregations p95).
  - PostgreSQL Database Disaster Recovery Backup & Restore Drill (`pg_dump` + `pg_restore` to sandbox database with 100% row count and zero financial drift parity check).
  - Frontend security features (Global Error Boundary zero-leakage, offline detection banner, safe export CSV DDE injection neutralization, production build).
- **Not Covered**:
  - Distributed multi-node Kubernetes cluster failover (tested on local containerized PostgreSQL 16 engine).
  - Third-party payment gateway callbacks (mocked per M3 specification).

---

## 2. Test Execution Summary

| Area | Scope / Target | Tests Run | Result | Evidence |
| :--- | :--- | :---: | :---: | :--- |
| **Backend Unit & Integration** | `go test ./...` | All packages | **PASS** | 100% passed across all 15 testable packages |
| **Route Inventory Test** | `TestRouteInventory_FailClosed` | 46 routes | **PASS** | 0 undeclared/unprotected routes detected |
| **Auth & Tenant Isolation** | `TestRouteInventory_AuthorizationEnforcement` | 46 routes | **PASS** | Anon: 401, Cross-Tenant: 403, Staff on Owner: 403 |
| **HTTP Security Headers** | `TestSecurityHeaders` | 9 headers | **PASS** | CSP, HSTS, X-Content-Type-Options, DENY, etc. |
| **Request Body Limiter** | `TestRequestBodyLimiter` (>1MB) | 1 payload | **PASS** | HTTP 413 `REQUEST_BODY_TOO_LARGE` |
| **Rate Limiter** | `TestRateLimiting` (Auth abuse) | 12 reqs | **PASS** | HTTP 429 `RATE_LIMIT_EXCEEDED` + `Retry-After` |
| **Seed CLI Execution** | `cmd/seed` minimal & standard | 2 scales | **PASS** | Minimal: 2.45s, Standard: 17.87s (10k txs) |
| **Performance Benchmarks** | Seeded dataset (10k txs, 2k cust, 200 prod) | 8 endpoints | **PASS** | CRUD p95 < 10ms (<500ms), Dash p95 < 20ms (<2s) |
| **DB Backup & Restore Drill** | `pg_dump` + `pg_restore` drill | 20 tables | **PASS** | 100% row parity, 0.00 financial drift |
| **Frontend Tests** | `npm test -- --run` | 46 tests | **PASS** | 10/10 files passed in 6.26s |
| **Frontend Production Build**| `npm run build` (`tsc -b && vite build`)| 1,716 modules | **PASS** | Build successful in 4.41s, 0 TS errors |

---

## 3. Deep-Dive Audit Findings & Results

### 3.1 Route Authorization Inventory (46 Routes Fail-Closed)
Automated route inventory test `TestRouteInventory_FailClosed` and `TestRouteInventory_AuthorizationEnforcement` verified that all 46 endpoints strictly enforce their protection levels:
- **Level 0 (Public - 8 routes)**: `/health`, `/ready`, `/api/v1/health`, `/api/v1/ready`, `/api/v1/auth/register`, `/api/v1/auth/login`, `/api/v1/auth/refresh`, `/api/v1/auth/logout`.
- **Level 1 (Authenticated User - 2 routes)**: `/api/v1/auth/me`, `/api/v1/businesses`.
- **Level 2 (Tenant Scoped - 33 routes)**: All products, customers, transactions, returns, dashboard, imports, and insight endpoints.
- **Level 3 (Owner Only - 3 routes)**:
  - `PATCH /api/v1/businesses/:business_id`
  - `POST /api/v1/businesses/:business_id/transactions/:transaction_id/cancel`
  - `POST /api/v1/businesses/:business_id/products/:product_id/deactivate`

**Live Access Matrix Verification (Sample Table)**:
```
Endpoint & Method                                  | Anon   | OwnerA | OwnerB | StaffA | Verdict
---------------------------------------------------------------------------------------------------
GET   /api/v1/businesses/{BIZ_A}/products          | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/customers         | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/transactions      | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/dashboard/summary | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/dashboard/trends  | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/dashboard/data-health | 401 | 200  | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/insights          | 401    | 200    | 403    | 200    | PASS
GET   /api/v1/businesses/{BIZ_A}/recommendations   | 401    | 200    | 403    | 200    | PASS
PATCH /api/v1/businesses/{BIZ_A}                   | 401    | 200    | 403    | 403    | PASS
```

### 3.2 HTTP Security Headers & Abuse Prevention
Every live HTTP response emits full production security headers:
- `Content-Security-Policy`: `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'`
- `Strict-Transport-Security`: `max-age=31536000; includeSubDomains; preload`
- `X-Content-Type-Options`: `nosniff`
- `X-Frame-Options`: `DENY`
- `Referrer-Policy`: `strict-origin-when-cross-origin`
- `Permissions-Policy`: `geolocation=(), camera=(), microphone=(), payment=()`
- `Cache-Control`: `no-store, no-cache, must-revalidate, proxy-revalidate`
- **Request Body Limiter**: Sending a 1,049,600-byte payload to `/api/v1/businesses/:id/products` returned `HTTP 413 Payload Too Large` with contract error code `REQUEST_BODY_TOO_LARGE`.
- **Sliding-Window Rate Limiter**: 12 rapid login attempts from the same IP triggered `HTTP 429 Too Many Requests` on attempt 7, with contract error code `RATE_LIMIT_EXCEEDED` and `Retry-After: 872`.

---

## 4. Performance Benchmarks on Seeded Dataset (10,000 Transactions)

Executed against live backend connected to PostgreSQL 16 seeded with realistic data for Business A (10,000 transactions over 180 days, 2,000 customers, 200 products) and Business B:

```
Endpoint                                   | Min     | Avg     | p50     | p90     | p95     | Max     | Target  | Verdict
-------------------------------------------------------------------------------------------------------------------
CRUD: Products List (200 items)            | 2.3ms   | 3.1ms   | 3.2ms   | 3.8ms   | 3.9ms   | 4.1ms   | <500ms  | PASS
CRUD: Customers List (2,000 items)         | 2.3ms   | 2.5ms   | 2.5ms   | 2.7ms   | 2.8ms   | 2.9ms   | <500ms  | PASS
CRUD: Transactions List (10,000 items)     | 4.7ms   | 6.3ms   | 6.0ms   | 7.6ms   | 8.0ms   | 8.6ms   | <500ms  | PASS
Dashboard: Summary (10k txs aggregations)  | 12.6ms  | 15.5ms  | 14.7ms  | 18.7ms  | 19.1ms  | 19.3ms  | <2000ms | PASS
Dashboard: Daily Trends (180 days)         | 10.4ms  | 13.0ms  | 12.8ms  | 15.8ms  | 16.0ms  | 16.9ms  | <2000ms | PASS
Dashboard: Data Health Composite Score     | 3.8ms   | 4.9ms   | 4.8ms   | 6.3ms   | 6.5ms   | 6.8ms   | <2000ms | PASS
Insight: Active Insights List              | 4.1ms   | 5.2ms   | 5.0ms   | 5.9ms   | 6.5ms   | 11.9ms  | <1000ms | PASS
Insight: Recommendations List              | 6.8ms   | 9.3ms   | 8.5ms   | 11.9ms  | 13.8ms  | 15.1ms  | <1000ms | PASS
```

**Analysis**:
- All standard CRUD queries (paginated 50 items) execute at p95 under **10ms**, outperforming the 500ms requirement by >50x.
- Heavy financial aggregations across 10,000 transactions execute at p95 under **20ms**, outperforming the 2,000ms requirement by >100x.

---

## 5. Database Backup & Restore Drill Verification (TRD §10, §14)

### 5.1 Drill Parameters & Execution Timings
- **Tooling**: `pg_dump` (custom binary format `-Fc -Z9`) and `pg_restore` (PostgreSQL 16.15).
- **Dump Archive**: Size **1.9 MB** (compressed), execution time: **780 ms**.
- **Sandbox Database**: `teman_umkm_drill_verify`.
- **Restore Duration**: **1,490 ms** (1.49s). Target SLA RTO < 900s (15 minutes) -> **Exceeded by >600x**.

### 5.2 Row Count Parity Across All 20 Tables
```
TABLE                     | SOURCE     | RESTORED   | STATUS
------------------------------------------------------------
analysis_runs             | 1          | 1          | MATCH
audit_logs                | 0          | 0          | MATCH
business_memberships      | 4          | 4          | MATCH
businesses                | 2          | 2          | MATCH
complaints                | 5          | 5          | MATCH
cost_versions             | 210        | 210        | MATCH
customers                 | 2200       | 2200       | MATCH
import_batches            | 0          | 0          | MATCH
import_rows               | 0          | 0          | MATCH
insight_sources           | 0          | 0          | MATCH
insights                  | 3          | 3          | MATCH
price_history             | 250        | 250        | MATCH
products                  | 250        | 250        | MATCH
recommended_actions       | 3          | 3          | MATCH
refresh_tokens            | 0          | 0          | MATCH
sales_returns             | 95         | 95         | MATCH
schema_migrations         | 1          | 1          | MATCH
transaction_items         | 20671      | 20671      | MATCH
transactions              | 10500      | 10500      | MATCH
users                     | 4          | 4          | MATCH
```
**Row Count Parity**: **100.0% (Zero Missing Rows)**.

### 5.3 Financial Aggregate Parity
- **Source Database (`teman_umkm`)**:
  - Total Paid Transactions: `10,162`
  - Total Gross Sales: `Rp 1.555.855.500,00`
- **Restored Sandbox Database (`teman_umkm_drill_verify`)**:
  - Total Paid Transactions: `10,162`
  - Total Gross Sales: `Rp 1.555.855.500,00`
- **Financial Drift**: **Rp 0,00 (100% Zero Drift Match)**.

### 5.4 Sandbox Application Probe & Safe Teardown
- Connected test server instance to `teman_umkm_drill_verify`:
  - `GET /health` -> `200 OK` (`{"status":"ok"}`)
  - `GET /ready` -> `200 OK` (`{"database":"connected","status":"ready"}`)
  - `GET /api/v1/businesses/0192e000-0000-7000-8000-000000000001/dashboard/summary` -> `200 OK` (Revenue matching source exactly).
- Sandbox database was dropped and temporary dump file purged cleanly.

---

## 6. Frontend Hardening & Integrity Verification

1. **Vitest Unit Test Suite (`npm test -- --run`)**:
   - 10 test files, 46 tests passed in 6.26s.
   - Tested: `GlobalErrorBoundary.test.tsx` (5 tests), `OfflineBanner.test.tsx` (4 tests), `SafeExportModal.test.tsx` (4 tests), `csv.test.ts` (6 tests), modules tests (POS, Dashboard, Products, Imports, Insights).
2. **CSV Formula Injection Mitigation**:
   - Verified prefix neutralization (`'`) for cells starting with `=`, `+`, `-`, `@`.
   - Verified `\uFEFF` UTF-8 BOM injection for Microsoft Excel Indonesian locale compatibility without mojibake.
3. **Zero Information Leakage**:
   - Verified that unhandled exceptions caught by `GlobalErrorBoundary` display user-friendly empathetic copy, incident reference codes (`ERR_CLIENT_YYYYMMDD_XXXX`), and never expose component stack traces or internal errors.
4. **Production Build**:
   - `npm run build` (`tsc -b && vite build`) completed cleanly with 0 TypeScript/compilation errors.

---

## 7. QA Findings & Immediate Fixes

### Finding QAF-074-001 (Resolved)
- **Severity**: Medium
- **Component**: `internal/finance/repository.go` (`GetDataHealthMetrics`)
- **Symptom**: Calling `GET /api/v1/businesses/:id/dashboard/data-health` against PostgreSQL 16 returned HTTP 500 (`ERROR: column "is_active" does not exist` and `invalid input syntax for type uuid: ""`).
- **Root Cause**: The raw SQL query checked `is_active = true` (the actual column in PostgreSQL is `status = 'active'`) and checked `(customer_id IS NULL OR customer_id = '')` (comparing a UUID column to empty string is invalid in PostgreSQL).
- **Remediation**: Patched `internal/finance/repository.go` to use `status = 'active'` and `customer_id IS NULL`. Re-tested and verified all tests pass (`go test -v ./internal/finance/...`). Pushed to `backend` repository as commit `fcb4f0d`.

---

## 8. Final Verdict & Recommendation

- **Verdict**: **PASS / APPROVE**
- **Recommendation**:
  - Milestone M7 hardening, security pass, route inventory, seed CLI, and backup drill meet all production readiness requirements specified in `project-map/trd.md` (§10, §12, §14) and contracts `contracts/api-hardening-security.md` and `contracts/ui-hardening-security.md`.
  - System is **READY** for Milestone M7 final review and sign-off by Leader (Task T-075).
