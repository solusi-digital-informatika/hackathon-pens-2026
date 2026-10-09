# Test Report: F2 Shot CRUD & Board Integration Check

## 1. Commands Run
- Backend Tests: `cd C:/project/ai-office/backend && .venv/Scripts/python.exe -m pytest -q` (57 passed)
- Frontend Tests: `cd C:/project/ai-office/frontend && npm test` (50 passed in 10 files)
- Frontend Build: `cd C:/project/ai-office/frontend && npm run build` (tsc -b && vite build: 0 errors)
- Backend Server: `uvicorn app.main:app --host 127.0.0.1 --port 8000` (started and cleanly terminated)
- Live HTTP: Exercised full Shot CRUD and 6-shot demo dataset against live backend

## 2. Table of Checks
| Check ID | Description | Type | Status | Evidence / Notes |
|---|---|---|---|---|
| BE-UNIT | Backend test suite | Automated | PASS | 57 passed (33 shot tests + 24 project tests) |
| FE-UNIT | Frontend test suite | Automated | PASS | 50 passed in 10 test files (Vitest) |
| FE-BUILD | Frontend production build | Automated | PASS | Clean tsc & vite production build |
| AC1-API | AC1: POST shot creation | HTTP | PASS | 201 Created, auto seq 1 & 2, draft status, Location header |
| AC1-UI | AC1: Shot creation modal | Automated | PASS | Vitest ShotModal: validates title, submits payload |
| AC2-API | AC2: GET list & sequence order | HTTP | PASS | 200 OK, items returned sorted by sequence_order asc |
| AC3-API | AC3: PATCH update & DELETE shot | HTTP | PASS | 200 OK update persists; 204 DELETE; subsequent GET 404 |
| AC3-UI | AC3: Delete confirmation modal | Automated | PASS | Vitest DeleteShotModal: triggers delete callback |
| AC4-ISO | AC4: Project scoping & isolation | HTTP | PASS | 404 on invalid project_id, shot_id, & cross-project GET |
| AC4-VAL | Input validation error handling | HTTP | PASS | 400 Bad Request with validation_error code & fields |
| AC5-UI | AC5: Board renders & badges | Automated | PASS | Vitest ShotBoard: dual-coded status & sequence badges |
| AC5-VIS | AC5: Visual layout in browser | Visual | MANUAL | Visual check: board card/list layout in live browser |
| DATASET | 6-shot demo dataset verification | HTTP | PASS | Verified 6 ground-truth shots (SHOT-01..06) in sequence |

## 3. Failing Output
None. All automated backend (57) and frontend (50) tests passed; live HTTP suite completed with 0 errors.

## 4. Status Summary
- Automated / HTTP Checks: 12 PASS, 0 FAIL
- Visual / Browser Checks: 1 MANUAL (live browser UX review)
- Server process stopped cleanly after verification.
