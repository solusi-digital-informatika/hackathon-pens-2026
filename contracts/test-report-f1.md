# Test Report: F1 Integration Check

## 1. Commands Run
- Backend Tests: `cd C:/project/ai-office/backend && .venv/Scripts/python.exe -m pytest -q`
- Frontend Tests: `cd C:/project/ai-office/frontend && npm test`
- Backend Server: `cd C:/project/ai-office/backend && .venv/Scripts/python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000`
- HTTP Verification: Real HTTP queries against 127.0.0.1:8000 (POST, GET, pagination, error schemas)

## 2. Table of Checks
| Check ID | Description | Type | Status | Evidence / Notes |
|---|---|---|---|---|
| BE-UNIT | Backend test suite | Automated | PASS | 24 tests passed in 0.97s |
| FE-UNIT | Frontend test suite | Automated | PASS | 27 tests passed in 6.49s (Vitest) |
| API-POST | POST /projects (201 Created) | HTTP | PASS | Returns 201, Location header, default status `draft` |
| API-GET-ID | GET /projects/{id} (200 OK) | HTTP | PASS | Fetches created project by persistent ID |
| API-LIST | GET /projects pagination & sort | HTTP | PASS | Returns 200, `items`, `total`, sorted created_at desc |
| API-404 | Unknown project ID isolation | HTTP | PASS | Returns 404 with standard error body schema |
| API-400 | Validation error on blank name | HTTP | PASS | Returns 400 with `validation_error` code & fields |
| API-CORS | CORS headers verification | HTTP | PASS | `access-control-allow-origin: http://127.0.0.1:5173` |
| AC1-API | AC1: API project creation | Automated | PASS | 201 Created + Location header verified |
| AC1-UI | AC1: UI creation form & navigation | Visual | MANUAL | Visual check: form submit navigates to /projects/{id} |
| AC2-API | AC2: Stable identifier persistence | Automated | PASS | Assigned ID persists across GET and list |
| AC2-UI | AC2: UI identifier display | Visual | MANUAL | Visual check: ID badge rendered consistently |
| AC3-API | AC3: Data isolation & 404 | Automated | PASS | 404 returned on unknown ID, no data leak |
| AC3-UI | AC3: UI 404 error state | Visual | MANUAL | Visual check: UI displays 404 not found state |
| AC4-UNIT | AC4: Brief & shot placeholders | Automated | PASS | Vitest checks render static placeholders |
| AC4-UI | AC4: UI header layout | Visual | MANUAL | Visual check: "No brief yet" & "No shots yet" displayed |

## 3. Failing Output
None. All automated backend and frontend tests passed with zero failures.

## 4. Status Summary
- Automated Checks: 11 PASS, 0 FAIL
- Visual / Browser Checks: 5 MANUAL (no browser in test environment)
- Backend process stopped cleanly after execution.
