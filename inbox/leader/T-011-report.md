# Report: T-011 API Contract for Auth, Business & Tenant Middleware

**From:** backend-ai  
**To:** leader  
**Task ID:** T-011  
**Status:** Done  
**Contract Path:** `contracts/api-auth.md`  
**Repository:** `office-hub`  

---

## 1. Algorithm
Request → JSON validation & DTO binding → Authentication middleware (`RequireAuth`, JWT verification) → Tenant middleware (`RequireTenantMembership`, UUIDv7 validation & membership lookup) → Role guard (`RequireBusinessRole`, owner/staff check) → Service logic & DB transaction (GORM + audit writer) → Safe DTO response / standard error mapping (`{error: {code, message, details?}}`).

## 2. Completed
- Authored comprehensive API contract in `contracts/api-auth.md` adhering strictly to TRD TD-04, TD-05, TD-06, TD-11, TD-13, ERD Diagram A, and UI spec `contracts/ui-auth-business.md`.
- Documented global conventions: `/api/v1` prefix, snake_case JSON, RFC 3339 timestamps, decimal strings for currency, and standard error format with user-safe Indonesian messages and stable machine-readable codes.
- Documented full Authentication endpoint specifications:
  - `POST /api/v1/auth/register`: body `{email, password, display_name}`, argon2id password requirements, 201 Created with user + access_token, httpOnly refresh token cookie.
  - `POST /api/v1/auth/login`: body `{email, password}`, returns 200 OK with user + access_token + businesses list + httpOnly refresh token cookie; generic `INVALID_CREDENTIALS` error on bad email or password (prevents account enumeration).
  - `POST /api/v1/auth/refresh`: reads httpOnly cookie, rotates refresh token, issues new access token, implements token family reuse detection (revokes entire family upon reuse).
  - `POST /api/v1/auth/logout`: revokes refresh token family and clears httpOnly cookie (204 No Content).
  - `GET /api/v1/auth/me`: Bearer token required, returns user profile and affiliated businesses with roles.
- Documented Business Management endpoint specifications:
  - `POST /api/v1/businesses`: body `{name, business_type, timezone?, report_period_start_day?}`, creates business and assigns creator as `owner` in `business_memberships` in an atomic transaction with audit log.
  - `GET /api/v1/businesses/{business_id}`: validates UUIDv7, tenant isolation checks, returns business profile and caller membership.
  - `PATCH /api/v1/businesses/{business_id}`: partial update with role guard (owner only; staff receives 403 Forbidden).
- Detailed Tenant Middleware architecture (`RequireTenantMembership`):
  - Path scoping convention `/api/v1/businesses/{business_id}/...`.
  - Context injection (`business_id`, `membership_role`).
  - Distinct status codes: 401 unauthenticated, 403 `FORBIDDEN` if not a member, 404 `BUSINESS_NOT_FOUND` if business does not exist.
- Defined rate limiting specifications (10 req/15 min for login/refresh, 5 req/15 min for register, 120 req/min for authenticated endpoints) with standard rate-limiting headers.
- Specified audit logging (`audit_logs`) behavior and schema for business mutations.
- Provided a 14-scenario contract verification matrix for Tester (T-014) and Frontend (T-013).
- Updated `board/board.md` row for T-011 to `done`.
- Archived task to `inbox/backend-ai/done/T-011-api-contract-auth-business.md`.

## 3. Verification
- Checked file creation: `contracts/api-auth.md` exists and verified (33.5 KB).
- Verified alignment with PRD §5.1, §8.1; TRD §4, §5 (TD-04, TD-05, TD-06, TD-11, TD-13); ERD `erd.md` §2, §6, §7; and UI specification `ui-auth-business.md`.
- Verified task move to `inbox/backend-ai/done/` and task board status in `board/board.md`.

## 4. Issues & Blockers
None. Ready for backend implementation (T-012), frontend implementation (T-013), and test scenario preparation (T-014).

## 5. Manual Task
None.
