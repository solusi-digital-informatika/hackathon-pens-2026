# Task: T-002 Backend Scaffold, DB, Migrations, Health Check

## 1. Objective
Scaffold the Go backend repository with Gin, PostgreSQL database connection with GORM, initial versioned migration, Docker Compose, configuration loading from env, structured logging with `slog`, health endpoints (`/health` and `/ready`), and GitHub Actions CI.

## 2. Context and Reference Files
- Repo local path: `C:/Project/ai-office/office-hub/backend`
- Remote: `https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git`
- Hub docs:
  - `project-map/trd.md`: §2 (Architecture), §3 (Stack: Go, Gin, GORM, shopspring/decimal, google/uuid, golang-jwt, argon2id, slog, migrations), §4 (Backend Modules: `internal/platform/`), §5 (Key Decisions: TD-02, TD-04, TD-13, TD-14), §9 (ops endpoints: `GET /health`, `GET /ready`)
  - `contracts/erd.md`: §1 (Conventions), §14 (Migration order)
  - `project-map/git-workflow.md`: commit conventions, `.gitignore`, `.gitattributes`
- Backend-AI SOUL: layout `cmd/api/main.go`, `internal/platform/`, thin handlers, security baseline, `govulncheck`.

## 3. Expected Output
In repository `C:/Project/ai-office/office-hub/backend`:
1. `.gitignore` (ignoring `.env`, `*.exe`, `/bin`, `/tmp`, `coverage.out`) and `.gitattributes` (`* text=auto eol=lf`).
2. `go.mod` (module e.g. `github.com/solusi-digital-informatika/teman-umkm-backend` or `teman-umkm-backend`) and `go.sum`.
3. `docker-compose.yml` defining PostgreSQL (e.g. postgres:16-alpine with port mapping e.g. 5432, default db `teman_umkm`, user `teman_umkm`, password `secret`).
4. `.env.example` with standard config keys (`PORT=8080`, `DATABASE_URL=...`, `JWT_SECRET=...`, `APP_ENV=development`).
5. `internal/platform/`:
   - `config/`: validated struct from env.
   - `logger/`: `slog` JSON structured logger with request ID propagation.
   - `database/`: GORM PostgreSQL connection pool setup.
   - `httpserver/`: Gin engine setup, recovery middleware, CORS allowlist, request logging middleware.
6. `cmd/api/main.go`: clean wiring and graceful shutdown on SIGINT/SIGTERM.
7. Health routes under root or `/api/v1`:
   - `GET /health`: liveness (returns 200 `{"status":"ok"}`).
   - `GET /ready`: readiness checking database ping (returns 200 `{"status":"ready","database":"connected"}` or 503 if DB down).
8. `migrations/`: initial migration script (e.g., `000001_init_extensions.up.sql` and `000001_init_extensions.down.sql` enabling `uuid-ossp` or `pgcrypto`, `citext`, `pg_trgm`).
9. Migration runner tool/helper or CLI integration.
10. `.github/workflows/ci.yml`: runs `go test ./...`, `go vet ./...`, and `go build ./...`.

## 4. Constraints
- Strict Go idioms: composition over inheritance, small interfaces, no globals, no `init()` magic, context first.
- No `AutoMigrate` in production.
- Use Conventional Commits (`feat(platform): scaffold backend service, db, and health endpoints`).
- Push to `origin main` and verify with `git status -sb`.

## 5. Definition of Done
- `go build ./...` succeeds with zero errors.
- `go test ./...` passes.
- Server boots and responds to `GET /health` with 200 OK.
- Changes committed and pushed to `origin main`.
- Report commit hash and details to `inbox/leader/T-002-report.md`, update row in `board/board.md` to `done`, and move this task to `inbox/backend-ai/done/`.
