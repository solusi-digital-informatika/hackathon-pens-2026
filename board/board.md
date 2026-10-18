# Task Board — Teman UMKM

Status values: `todo` · `doing` · `review` · `testing` · `done` · `blocked`

Current milestone: **M7 — Hardening, Security Pass, Seed CLI & Production Readiness (COMPLETED)**

## Project Status: 100% COMPLETE & PRODUCTION READY

All MVP milestones (M0–M7) have been implemented, tested, verified, and signed off.

| ID | Title | Owner | Size | Status | Repo | Commit |
|---|---|---|---|---|---|---|
| T-070 | Hardening & Security Audit Specification (Access matrix, rate limits, audit verification) | designer | Small | done | office-hub | — |
| T-071 | Security and Hardening Contract (Rate-limits, route inventory, headers, DB drill script) | backend-ai | Standard | done | office-hub | — |
| T-072 | Implement backend cmd/seed, route inventory authorization tests, and security hardening | backend-ai | Large | done | backend | `c4a7389` |
| T-073 | Frontend security polish, build optimization, CSP verification, error boundary hardening | frontend | Standard | done | frontend | `ec4e1ee` |
| T-074 | Independent QA audit: full security pass, tenant isolation matrix, perf benchmarks & restore drill | tester | Large | done | hub | `fcb4f0d` |
| T-075 | Final Project Review, M7 sign-off, and Production Readiness Assessment | leader | Small | done | hub | — |

## Completed Milestones

### M7 — Hardening, Security Pass, Seed CLI & Production Readiness (Accepted & Closed)
- T-070: Hardening & Security Audit Specification (designer) — `done`
- T-071: Security and Hardening Contract (backend-ai) — `done`
- T-072: Implement backend cmd/seed, route inventory authorization tests, and security hardening (backend-ai) — `c4a7389` (`done`)
- T-073: Frontend security polish, build optimization, CSP verification, error boundary hardening (frontend) — `ec4e1ee` (`done`)
- T-074: Independent QA audit: full security pass, tenant isolation matrix, perf benchmarks & restore drill (tester) — `fcb4f0d / ec4e1ee` (`done`, Verdict: APPROVE)
- T-075: Final Project Review, M7 sign-off, and Production Readiness Assessment (leader) — `done`

### M6 — Insight Engine & Recommendations (Accepted)
- T-060: UI spec for Insight Cards, Recommendations, Impact Preview & Detail Modal (designer) — `done`
- T-061: API contract for Insight Rules Engine, Dismissals & Feedback (backend-ai) — `done`
- T-062: Implement backend Deterministic Insight Engine, Rule Processors & Audit (backend-ai) — `2a16809` (`done`)
- T-063: Implement frontend Insight Feed, Recommendation Actions & Detail Modal (frontend) — `5aebead` (`done`)
- T-064: QA verification for Insight Rule Correctness, Edge Cases & Data Provenance (tester) — `91f6d63 / 5aebead` (`done`, Verdict: APPROVE)
- T-065: M6 Milestone review and sign-off (leader) — `done`

### M5 — CSV Import (Async) (Accepted)
- T-050: UI spec for CSV Import Wizard (Upload, Mapping, Validation & Progress) (designer) — `done`
- T-051: API contract for CSV Import, Mapping, Row Validation & Async Worker (backend-ai) — `done`
- T-052: Implement backend CSV Parser, Async Batch Worker & Import Rollbacks (backend-ai) — `d007c69` (`done`)
- T-053: Implement frontend CSV Import Wizard, Mapping & Progress Polling (frontend) — `15d0c57` (`done`)
- T-054: QA verification for CSV Injection, Malformed Rows, Async Recovery (tester) — `0747414 / 15d0c57` (`done`, Verdict: APPROVE)
- T-055: M5 Milestone review and sign-off (leader) — `done`

### M4 — Finance Engine, Dashboard & Data Health (Accepted)
- T-040: UI spec for Financial Dashboard, Metrics & Data Health Indicators (designer) — `done`
- T-041: API contract for Financial Summary, Daily Trends & Data Health (backend-ai) — `done`
- T-042: Implement backend Finance Engine, Summary Aggregations & Data Health (backend-ai) — `fb8a4ad` (`done`)
- T-043: Implement frontend Dashboard, Financial Metric Cards & Health Badges (frontend) — `4ed545b` (`done`)
- T-044: QA verification for Financial Metric Calculations & Aggregations (tester) — `fb8a4ad / 4ed545b` (`done`, Verdict: APPROVE)
- T-045: M4 Milestone review and sign-off (leader) — `done`

### M3 — Customers, Transactions & Inventory Management (Accepted)
- T-030: UI spec for Customers, Transactions & Returns/Cancellations (designer) — `done`
- T-031: API contract for Customers, Transactions, Idempotency & Stock (backend-ai) — `done`
- T-032: Implement backend Customers, Transactions, Idempotency & Stock Deductions (backend-ai) — `ff5c612` (`done`)
- T-033: Implement frontend Customers & Transaction/POS flow (frontend) — `d6ffd5a` (`done`)
- T-034: QA verification for Transaction Idempotency & Inventory Integrity (tester) — `ff5c612 / d6ffd5a` (`done`, Verdict: APPROVE)
- T-035: M3 Milestone review and sign-off (leader) — `done`

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
