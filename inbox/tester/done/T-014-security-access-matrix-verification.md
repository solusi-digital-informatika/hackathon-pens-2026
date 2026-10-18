# Task: T-014 Security & Access Matrix Verification (BOLA, Auth, Tenant Isolation)

## 1. Objective
Perform independent QA and security matrix verification of Milestone M1 across both `backend` (commit `e25c943`) and `frontend` (commit `8f20051`).

## 2. Context and Reference Files
- TRD: §4 (Backend Modules), §5 (TD-04 UUIDv7, TD-05 Tenant Isolation, TD-06 Auth), §10 (Security Baseline: OWASP API1 BOLA, API2 Broken Auth, generic auth errors).
- `contracts/api-auth.md`: Section 6 (Contract Verification Matrix: 14 test scenarios).
- `contracts/ui-auth-business.md`: Security UX (generic error messages, show/hide password toggle).
- Tester SOUL: "No evidence, no PASS. Status is one of: PASS / FAIL / BLOCKED / NOT VERIFIED / N/A. Matrix format: Scenario | Role | Endpoint | Expected | Actual | Status."

## 3. Expected Verification Steps
1. **Backend Verification (`office-hub/backend` at commit `e25c943`)**:
   - Verify `git status -sb` is clean on `e25c943`.
   - Run `go vet ./...` (using `/c/Users/ardan/scoop/shims/go`).
   - Run `go test -v ./...` and capture evidence for:
     - `TestHashAndVerifyPassword` and `TestVerifyInvalidHashFormat` (Argon2id + constant time)
     - `TestJWTGenerateAndValidate` and `TestJWTInvalidSecretOrTampered`
     - `TestTenantIsolationCrossAccess` (User A vs User B cross-tenant access rejection)
   - Run `go build ./...` to confirm clean compilation.
   - Inspect migration SQL files `migrations/000002_create_auth_and_business_tables.up.sql` and `down.sql`.
2. **Frontend Verification (`office-hub/frontend` at commit `8f20051`)**:
   - Verify `git status -sb` is clean on `8f20051`.
   - Run `npm test -- --run` and verify test assertions pass.
   - Run `npm run lint`.
   - Run `npm run build` and verify output bundle in `dist/`.
   - Inspect `src/modules/auth/pages/LoginPage.tsx` for generic auth error (*"Email atau password salah."*).
   - Inspect `src/core/network/client.ts` for `credentials: 'include'`.
3. **Security & Access Matrix**:
   - Construct the verification matrix comparing actual results against `contracts/api-auth.md`.
4. **Report & Board**:
   - Write comprehensive verification report to `inbox/leader/T-014-report.md`.
   - Update `board/board.md` for T-014 to `done`.
   - Move this task to `inbox/tester/done/`.

## 4. Constraints
- Strict evidence-based reporting with actual command outputs.
- Never invent data.

## 5. Definition of Done
- Complete verification report delivered to `inbox/leader/T-014-report.md`.
- Status updated on `board/board.md`.
