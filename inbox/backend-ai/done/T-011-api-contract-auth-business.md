# Task: T-011 API Contract for Auth, Business & Tenant Middleware

## 1. Objective
Author `contracts/api-auth.md` detailing the HTTP API contract for Authentication, Business Setup, Membership, and Tenant Middleware for Teman UMKM.

## 2. Context and Reference Files
- `project-map/prd.md`: §5.1 (Authentication and Business Setup), §8.1 (Security: isolation by Business ID, generic auth errors, no secrets leaked).
- `project-map/trd.md`:
  - §4 (Backend Modules: `auth`, `business`, `platform`).
  - §5 (Key Decisions: TD-04 UUIDv7, TD-05 Tenant Isolation, TD-06 Auth with argon2id, JWT access token, httpOnly refresh token rotated on every use with family reuse detection, TD-11 Audit Writer, TD-13 API conventions).
  - §9 (API Inventory: `auth` and `business` endpoints).
  - §10 (Security and Privacy Requirements: rate limits on login/refresh).
- `contracts/erd.md`:
  - §2 (Diagram A: `users`, `businesses`, `business_memberships`, `refresh_tokens`, `audit_logs`).
  - §6 (Enum values: `users.status`, `business_memberships.role`).
  - §7 (Constraints: composite FKs, unique constraints).
- Backend-AI SOUL: API Contract conventions (`contracts/api-<feature>.md`).

## 3. Expected Output
`contracts/api-auth.md` detailing:
1. Conventions: prefix `/api/v1`, standard error format `{error: {code, message, details?}}`, Indonesian user-facing error messages, snake_case JSON.
2. Authentication Endpoints:
   - `POST /api/v1/auth/register`: body `{email, password, display_name}` -> 201 Created `{user: {id, email, display_name}, access_token}` + set httpOnly refresh token cookie.
   - `POST /api/v1/auth/login`: body `{email, password}` -> 200 OK `{user, access_token, businesses}` + set httpOnly refresh token cookie. Generic error `INVALID_CREDENTIALS` on bad email or password.
   - `POST /api/v1/auth/refresh`: uses refresh token cookie -> 200 OK `{access_token}` + new rotated refresh token cookie. Revoke family on reuse.
   - `POST /api/v1/auth/logout`: revokes refresh token family, clears cookie -> 204 No Content.
   - `GET /api/v1/auth/me`: Bearer token -> 200 OK `{user, businesses}`.
3. Business Setup & Management Endpoints:
   - `POST /api/v1/businesses`: body `{name, business_type}` -> 201 Created `{business, role}` (assigns creator as `owner` in `business_memberships`).
   - `GET /api/v1/businesses/{business_id}`: header Authorization -> 200 OK `{business, membership}`.
   - `PATCH /api/v1/businesses/{business_id}`: body `{name?, business_type?, report_period_start_day?}` -> 200 OK `{business}`.
4. Tenant Middleware Specification:
   - Scoped route convention: `/api/v1/businesses/{business_id}/...`.
   - Checks caller's membership in `{business_id}`.
   - Returns 401 if unauthenticated, 403 `FORBIDDEN` if not a member, 404 `BUSINESS_NOT_FOUND` if business does not exist.
5. Error definitions & Rate limiting specs (e.g. 10 requests per 15 min on login/refresh).

## 4. Constraints
- Strict adherence to TRD TD-05 and TD-06.
- Generic auth error messages (never reveal whether an email exists).
- Do not paste code in contract; this is the schema and behavioral contract for frontend and tester.

## 5. Definition of Done
- `contracts/api-auth.md` created in `contracts/`.
- Update your row status in `board/board.md` to `done`.
- Send report to `inbox/leader/T-011-report.md` and move this task to `inbox/backend-ai/done/`.
