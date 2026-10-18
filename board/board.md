# Task Board — Teman UMKM

Status values: `todo` · `doing` · `review` · `testing` · `done` · `blocked`

Current milestone: **M3 — Customers, Transactions & Inventory Management**

## M3 — Customers, Transactions & Inventory Management

| ID | Title | Owner | Size | Status | Repo | Commit |
|---|---|---|---|---|---|---|
| T-030 | UI spec for Customers, Transactions & Returns/Cancellations | designer | Standard | todo | office-hub | — |
| T-031 | API contract for Customers, Transactions, Idempotency & Stock | backend-ai | Standard | todo | office-hub | — |
| T-032 | Implement backend Customers, Transactions, Idempotency & Stock Deductions | backend-ai | Standard | todo | backend | — |
| T-033 | Implement frontend Customers & Transaction/POS flow | frontend | Standard | todo | frontend | — |
| T-034 | QA verification for Transaction Idempotency & Inventory Integrity | tester | Standard | todo | — | — |
| T-035 | M3 Milestone review and sign-off | leader | Small | todo | — | — |

## Completed Milestones

### M2 — Products, Cost Versions & Pricing Calculator (Accepted)
- T-020: UI spec for Products, Cost Versions & Pricing Calculator (designer) — `done`
- T-021: API contract for Products, Cost Versions & Calculator (backend-ai) — `done`
- T-022: Implement backend Products, Cost Versions, Calculator & Migrations (backend-ai) — `7abb990` (`done`)
- T-023: Implement frontend Products catalog, history & calculator (frontend) — `9ce21cb` (`done`)
- T-024: QA verification for Products, Decimal precision & Calculator (tester) — `7abb990 / 9ce21cb` (`done`, Verdict: APPROVE)
- T-025: M2 Milestone review and sign-off (leader) — `done`

### M1 — Auth, Business Setup & Tenant Isolation (Accepted)
- T-010: UI spec for Auth & Business Setup (designer) — `done`
- T-011: API contract for Auth, Business & Tenant Middleware (backend-ai) — `done`
- T-012: Implement backend Auth, Business, Tenant & Audit (backend-ai) — `e25c943` (`done`)
- T-013: Implement frontend Auth & Business Onboarding flow (frontend) — `8f20051` (`done`)
- T-014: Security & Access Matrix verification (tester) — `e25c943 / 8f20051` (`done`, Verdict: APPROVE)
- T-015: M1 Milestone review and sign-off (leader) — `done`

### M0 — Foundation (Accepted)
- T-001: Design direction and tokens (designer) — `done`
- T-002: Backend scaffold, DB, migrations, health check (backend-ai) — `ad541bd` (`done`)
- T-003: Frontend scaffold, routing, design token setup (frontend) — `d7a57e9` (`done`)
- T-004: Verify M0 foundation (tester) — `done` (Verdict: APPROVE)
- T-005: M0 Milestone review and sign-off (leader) — `done`

## Upcoming Milestones

- **M3**: Customers, transactions (items, returns, cancel), idempotency (Standard)
- **M4**: Finance engine, dashboard, data-health (Standard)
- **M5**: CSV import (async) (Large)
- **M6**: Insight engine, recommendations, insight detail and provenance (Large)
- **M7**: Hardening: security, performance, backup drill (Standard)
