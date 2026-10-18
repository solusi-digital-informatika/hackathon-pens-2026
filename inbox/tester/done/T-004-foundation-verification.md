# Task: T-004 Verify M0 Foundation (Build, Health, Migrations)

## 1. Objective
Perform independent QA verification of Milestone M0 Foundation across both `backend` and `frontend` repositories.

## 2. Context and Reference Files
- TRD: §3 (Stack), §9 (ops endpoints: `/health`, `/ready`), §10 (Security baseline), §12 (Test strategy), §13 (M0 done criteria: App boots, `/health` OK, one migration applies).
- Tester SOUL: "No evidence, no PASS. Status is one of: PASS / FAIL / BLOCKED / NOT VERIFIED / N/A."
- Backend repo: `C:/Project/ai-office/office-hub/backend` (Commit `ad541bd`).
- Frontend repo: `C:/Project/ai-office/office-hub/frontend` (Commit `d7a57e9`).

## 3. Expected Verification Steps
1. **Backend Verification (`backend`)**:
   - Verify working tree is clean on commit `ad541bd`.
   - Run `go vet ./...` and `go test ./...` (PATH includes `/c/Users/ardan/scoop/shims`). Record test outputs.
   - Run `go build ./...` to ensure clean compilation.
   - Run the health handler test or launch server briefly to test `GET /health` returns status 200 with `{"status":"ok"}`.
   - Inspect initial migration files `migrations/000001_init_extensions.up.sql` and `down.sql`.
2. **Frontend Verification (`frontend`)**:
   - Verify working tree is clean on commit `d7a57e9`.
   - Run `npm test -- --run`. Record test outputs.
   - Run `npm run lint`.
   - Run `npm run build` and verify output in `dist/`.
3. **Report**:
   - Write comprehensive verification report to `inbox/leader/T-004-report.md`.
   - Update `board/board.md` row for T-004 to `done`.
   - Move this task to `inbox/tester/done/`.

## 4. Constraints
- Provide real tool execution evidence for all PASS/FAIL verdicts.
- Redact any sensitive credentials if present.

## 5. Definition of Done
- All verification steps executed with real outputs.
- Test report delivered to `inbox/leader/T-004-report.md`.
