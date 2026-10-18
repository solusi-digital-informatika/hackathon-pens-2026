# Task: T-012 Implement Backend Auth, Business, Tenant Middleware & Audit

## 1. Objective
Implement the backend modules for Authentication, Business Setup, Membership, Tenant Middleware, and Audit Trail in `office-hub/backend`, along with database migrations and automated unit + cross-tenant tests.

## 2. Context and Reference Files
- Repo local path: `C:/Project/ai-office/office-hub/backend`
- Remote: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
- Hub contracts & specifications:
  - `contracts/api-auth.md`: Authoritative API contract for Auth, Business, and Tenant Middleware.
  - `contracts/erd.md`: §2 (Diagram A: `users`, `businesses`, `business_memberships`, `refresh_tokens`, `audit_logs`), §6 (Enums), §7 (Constraints & Composite FKs).
  - `project-map/trd.md`: §4 (Backend Modules: `auth`, `business`, `audit`, `platform`), §5 (TD-04 UUIDv7, TD-05 Tenant Isolation, TD-06 Auth, TD-11 Audit).
  - `project-map/git-workflow.md`: Commit conventions and branch verification.
- Backend-AI SOUL: Go clean architecture (`internal/<module>/` with handler, service, repository, dto; thin handlers, no GORM leaks, `argon2id`, constant-time comparison, table-driven tests).

## 3. Expected Output
In repository `C:/Project/ai-office/office-hub/backend`:
1. **Migrations**:
   - `migrations/000002_create_auth_and_business_tables.up.sql` and `down.sql`:
     - `users` (id UUID PK, email citext unique, password_hash text, display_name text, status text, last_login_at, timestamps).
     - `businesses` (id UUID PK, name text, business_type text, currency text default 'IDR', timezone text default 'Asia/Jakarta', report_period_start_day smallint default 1, timestamps).
     - `business_memberships` (id UUID PK, business_id UUID FK, user_id UUID FK, role text check in ('owner', 'staff'), unique(business_id, user_id), timestamps).
     - `refresh_tokens` (id UUID PK, user_id UUID FK, family_id UUID, token_hash text unique, expires_at, used_at, revoked_at, created_at).
     - `audit_logs` (id UUID PK, business_id UUID FK, actor_user_id UUID, entity_type text, entity_id UUID, action text, before_snapshot jsonb, after_snapshot jsonb, request_id text, created_at).
2. **Modules**:
   - `internal/auth/`: password hashing with argon2id, JWT access token generation & validation (`golang-jwt`), refresh token rotation with family reuse revocation, handler, service, repository.
   - `internal/business/`: business CRUD, membership service, repository.
   - `internal/audit/`: append-only audit logger writing to `audit_logs` within caller's DB transaction.
   - `internal/platform/middleware/`:
     - Auth middleware: validates JWT Bearer token, populates user claims in context.
     - Tenant middleware: extracts `business_id` from route, verifies caller membership in `business_memberships`, injects business context and role.
3. **Route Wiring**:
   - Wire all routes from `contracts/api-auth.md` in `internal/platform/httpserver/server.go`.
4. **Automated Tests**:
   - Unit tests for password hashing, JWT claims, token family rotation.
   - Integration / handler tests for register, login, refresh, me, logout.
   - **Cross-tenant isolation tests**: User A cannot read or patch User B's business (verifying 403 / 404).

## 4. Constraints
- Strict adherence to `contracts/api-auth.md`.
- No plain text passwords or raw refresh tokens saved in DB.
- Generic error on auth failure (`INVALID_CREDENTIALS`).
- `go test ./...` and `go build ./...` must pass with zero failures.
- Conventional Commits (`feat(auth): implement authentication, business setup, tenant middleware, and audit trail`).
- Push to `origin main` and verify with `git status -sb`.

## 5. Definition of Done
- Migrations apply cleanly.
- All unit and cross-tenant tests pass.
- Pushed commit hash reported to `inbox/leader/T-012-report.md`.
- Update `board/board.md` for T-012 to `done`.
- Move this task to `inbox/backend-ai/done/`.
