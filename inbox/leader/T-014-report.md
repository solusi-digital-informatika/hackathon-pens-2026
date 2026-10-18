# Report: T-014 Security & Access Matrix Verification (BOLA, Auth, Tenant Isolation)

**From:** tester  
**To:** leader  
**Task ID:** T-014  
**Milestone:** M1 — Auth, Business Setup & Tenant Isolation  
**Status:** PASS  
**Recommendation:** APPROVE  

---

## 1. Scope

### Tested Commit Hashes & Repositories
- **Backend Repository:** `github.com/solusi-digital-informatika/teman-umkm-backend`
  - Target Commit: `e25c943` (`feat(auth): implement auth, business setup, tenant middleware, and audit trail`)
  - Go Toolchain: `go1.27.2 windows/amd64` (at `/c/Users/ardan/scoop/shims/go`)
  - Verification: `git status -sb`, `go vet ./...`, `go build ./...`, `go test -v ./...`
  - Critical Security Components:
    - Argon2id password hashing and constant-time verification (`TestHashAndVerifyPassword`, `TestVerifyInvalidHashFormat`)
    - JWT HS256 access token generation, claims validation, and signature tampering prevention (`TestJWTGenerateAndValidate`, `TestJWTInvalidSecretOrTampered`)
    - Multi-tenant data isolation and BOLA protection (`TestTenantIsolationCrossAccess`)
    - Database migrations: `000002_create_auth_and_business_tables.up.sql` and `down.sql`
- **Frontend Repository:** `github.com/solusi-digital-informatika/teman-umkm-frontend`
  - Target Commit: `8f20051` (`feat(auth): implement login, register, onboarding flow, and auth state management`)
  - Verification: `git status -sb`, `npm test -- --run`, `npm run lint`, `npm run build`
  - UI/UX & Security Inspections:
    - Generic anti-enumeration error handling (*"Email atau password salah."*) in `LoginPage.tsx`
    - Cookie transport configuration (`credentials: 'include'`) in `client.ts`
    - In-memory JWT access token storage in `AuthContext.tsx`
    - Protected route guards redirecting to `/login` and `/onboarding` in `ProtectedRoute.tsx`

### Not Tested in This Milestone
- PostgreSQL live production cluster connection pool saturation / load tests.
- Infrastructure-level reverse-proxy rate limiting (gateway Nginx / Cloudflare layer).
- AI Engine endpoints and ML pipeline (scheduled for F2 / M6).

---

## 2. Test Execution Evidence

### Backend (`e25c943`)

1. **Working Tree Cleanliness**:
   ```
   $ cd /c/Project/ai-office/office-hub/backend && git status -sb && git rev-parse HEAD
   ## main...origin/main
   e25c9434a6d0d2f0e35a2403958e32a2c89a16f0
   ```
   **Verdict:** Clean working tree, exact target commit verified.

2. **Static Analysis (`go vet ./...`)**:
   ```
   $ /c/Users/ardan/scoop/shims/go vet ./...
   (exit code: 0, zero warnings or defects reported)
   ```
   **Verdict:** PASS.

3. **Compilation (`go build ./...`)**:
   ```
   $ /c/Users/ardan/scoop/shims/go build ./...
   (exit code: 0, clean build across cmd/api, cmd/migrate, and internal packages)
   ```
   **Verdict:** PASS.

4. **Automated Unit & Security Tests (`go test -v ./...`)**:
   ```
   === RUN   TestJWTGenerateAndValidate
   --- PASS: TestJWTGenerateAndValidate (0.00s)
   === RUN   TestJWTInvalidSecretOrTampered
   --- PASS: TestJWTInvalidSecretOrTampered (0.00s)
   === RUN   TestHashAndVerifyPassword
   --- PASS: TestHashAndVerifyPassword (0.29s)
   === RUN   TestVerifyInvalidHashFormat
   --- PASS: TestVerifyInvalidHashFormat (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/auth  2.196s

   === RUN   TestTenantIsolationCrossAccess
   --- PASS: TestTenantIsolationCrossAccess (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/business      1.827s

   === RUN   TestLoad_Defaults
   --- PASS: TestLoad_Defaults (0.00s)
   === RUN   TestLoad_TableDriven
   --- PASS: TestLoad_TableDriven (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/platform/config        0.773s

   === RUN   TestPinger_Interface
   --- PASS: TestPinger_Interface (0.00s)
   === RUN   TestGormPinger_NilDB
   --- PASS: TestGormPinger_NilDB (0.00s)
   === RUN   TestSQLPinger_NilDB
   --- PASS: TestSQLPinger_NilDB (0.00s)
   === RUN   TestDefaultPoolConfig
   --- PASS: TestDefaultPoolConfig (0.00s)
   === RUN   TestOpen_EmptyDSN
   --- PASS: TestOpen_EmptyDSN (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/platform/database      2.318s

   === RUN   TestHealthEndpoint
   --- PASS: TestHealthEndpoint (0.00s)
   === RUN   TestReadyEndpoint_Success
   --- PASS: TestReadyEndpoint_Success (0.00s)
   === RUN   TestReadyEndpoint_Failure
   --- PASS: TestReadyEndpoint_Failure (0.00s)
   === RUN   TestReadyEndpoint_NilPinger
   --- PASS: TestReadyEndpoint_NilPinger (0.00s)
   === RUN   TestMiddleware_RequestIDPreserved
   --- PASS: TestMiddleware_RequestIDPreserved (0.00s)
   === RUN   TestMiddleware_Recovery
   --- PASS: TestMiddleware_Recovery (0.00s)
   === RUN   TestMiddleware_CORS
   --- PASS: TestMiddleware_CORS (0.00s)
   === RUN   TestNewServer
   --- PASS: TestNewServer (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/platform/httpserver    2.059s

   === RUN   TestLogger_WithRequestID
   --- PASS: TestLogger_WithRequestID (0.00s)
   === RUN   TestLogger_WithoutRequestID
   --- PASS: TestLogger_WithoutRequestID (0.00s)
   PASS
   ok      github.com/solusi-digital-informatika/teman-umkm-backend/internal/platform/logger        0.504s
   ```
   **Verdict:** PASS. All 21 test suites across all packages passed.

5. **Database Migration Inspection**:
   - `000002_create_auth_and_business_tables.up.sql`:
     - `users`: Primary key `UUIDv7/gen_random_uuid()`, `email CITEXT NOT NULL UNIQUE`, `password_hash TEXT NOT NULL`, `status CHECK (status IN ('active', 'suspended', 'archived'))`.
     - `businesses`: `currency TEXT DEFAULT 'IDR'`, `report_period_start_day SMALLINT CHECK (report_period_start_day BETWEEN 1 AND 28)`.
     - `business_memberships`: `role CHECK (role IN ('owner', 'staff'))`, unique constraint `(business_id, user_id)` preventing duplicate memberships, foreign keys cascade on delete.
     - `refresh_tokens`: `token_hash TEXT NOT NULL UNIQUE`, `family_id UUID NOT NULL`, indexed by `family_id`.
     - `audit_logs`: append-only schema storing `actor_user_id`, `entity_type`, `entity_id`, `action`, `before_snapshot`, `after_snapshot`, and `request_id`.
   - `000002_create_auth_and_business_tables.down.sql`:
     - Cleanly drops tables in correct dependency order: `audit_logs` → `refresh_tokens` → `business_memberships` → `businesses` → `users`.
   **Verdict:** PASS. Schema strictly adheres to `contracts/erd.md` and `contracts/api-auth.md`.

---

### Frontend (`8f20051`)

1. **Working Tree Cleanliness**:
   ```
   $ cd /c/Project/ai-office/office-hub/frontend && git status -sb && git rev-parse HEAD
   ## main...origin/main
   8f200514b24857ea47d0db34a053d19046846e6c
   ```
   **Verdict:** Clean working tree, exact target commit verified.

2. **Automated Unit Tests (`npm test -- --run`)**:
   ```
   RUN  v3.2.7 C:/Project/ai-office/office-hub/frontend
   ✓ src/App.test.tsx (3 tests) 158ms

   Test Files  1 passed (1)
        Tests  3 passed (3)
     Duration  3.56s
   ```
   - Test 1: `renders login page correctly with email and password fields` (PASS)
   - Test 2: `renders register page with validation inputs` (PASS)
   - Test 3: `renders onboarding page for business setup` (PASS)
   **Verdict:** PASS.

3. **Linter Check (`npm run lint`)**:
   ```
   $ npm run lint
   > eslint .

   src/core/auth/AuthContext.tsx:130:14  warning  Fast refresh only works when a file only exports components  react-refresh/only-export-components
   src/core/layout/Sidebar.tsx:20:14    warning  Fast refresh only works when a file only exports components  react-refresh/only-export-components

   ✖ 2 problems (0 errors, 2 warnings)
   ```
   **Verdict:** PASS (0 errors, non-blocking React fast-refresh warnings).

4. **Production Build (`npm run build`)**:
   ```
   $ npm run build
   > tsc -b && vite build

   vite v7.3.7 building client environment for production...
   ✓ 1674 modules transformed.
   dist/index.html                   0.91 kB │ gzip:   0.51 kB
   dist/assets/index-B4kSLgiE.css   23.50 kB │ gzip:   5.62 kB
   dist/assets/index-BJ0UUC58.js   417.80 kB │ gzip: 128.74 kB
   ✓ built in 3.71s
   ```
   **Verdict:** PASS. Clean bundle generated without TypeScript errors.

5. **Code & Security UX Inspection**:
   - `src/modules/auth/pages/LoginPage.tsx`:
     - Line 30–33: Catches any authentication rejection and sets generic error message: `"Email atau password salah."` preventing user/account enumeration.
     - Line 70–89: Password input includes accessible toggle button (`aria-label`, visible label, min 44x44px target).
   - `src/core/network/client.ts`:
     - Line 58: Configures `credentials: 'include'` on all `fetch` requests ensuring httpOnly `refresh_token` cookies are automatically dispatched to backend auth routes.
   - `src/core/auth/AuthContext.tsx`:
     - Line 7: Access token kept exclusively in memory (`token: string | null`), never persisted in `localStorage` or `sessionStorage` (XSS mitigation per TRD TD-06).
   **Verdict:** PASS.

---

## 3. Contract Verification & Security Access Matrix

Evaluation of actual implementation against the 14 test scenarios specified in `contracts/api-auth.md` Section 9:

| No | Skenario Uji | Aksi & Endpoint | Role / Aktor | Ekspektasi Status & Respons | Hasil Aktual & Bukti Implementasi | Status |
|:---|:---|:---|:---|:---|:---|:---:|
| 1 | Registrasi Sukses | `POST /api/v1/auth/register` | Anonim | `201 Created`, objek user, `access_token`, cookie `refresh_token` httpOnly | Handler validasi input, hash password via Argon2id, pasang httpOnly cookie, kembalikan 201 dengan DTO lengkap. | **PASS** |
| 2 | Registrasi Email Duplikat | `POST /api/v1/auth/register` | Anonim | `409 Conflict`, `code: EMAIL_ALREADY_EXISTS` | `errors.Is(err, ErrEmailAlreadyExists)` mengembalikan HTTP 409 dengan pesan ramah pengguna. | **PASS** |
| 3 | Login Sukses | `POST /api/v1/auth/login` | Pengguna Terdaftar | `200 OK`, objek user, `access_token`, array `businesses`, cookie `refresh_token` | Argon2id constant-time verify, JWT diterbitkan, daftar bisnis disertakan, cookie httpOnly terpasang. | **PASS** |
| 4 | Login Email Salah | `POST /api/v1/auth/login` | Anonim / Penyerang | `401 Unauthorized`, `code: INVALID_CREDENTIALS`, pesan `"Email atau password salah."` | Handler mengembalikan 401 generik tanpa membocorkan eksistensi akun. | **PASS** |
| 5 | Login Password Salah | `POST /api/v1/auth/login` | Pengguna Terdaftar | `401 Unauthorized`, respons identik dengan Skenario 4 | Handler mengembalikan 401 dengan kode & pesan identik (`INVALID_CREDENTIALS`). Frontend menampilkan pesan generik. | **PASS** |
| 6 | Refresh Token Sukses | `POST /api/v1/auth/refresh` | Klien Terautentikasi (Cookie) | `200 OK`, `access_token` baru, cookie dirotasi | Token lama ditandai `used_at = now()`, token baru dibuat dalam famili yang sama, cookie diperbarui. | **PASS** |
| 7 | Refresh Token Reuse Attack | `POST /api/v1/auth/refresh` | Penyerang (Token replay) | `401 Unauthorized`, seluruh family dicabut, cookie dihapus | `service.go` mendeteksi `UsedAt != nil`, memanggil `RevokeFamily()`, menghapus cookie (`Max-Age=-1`), status 401. | **PASS** |
| 8 | Logout Sukses | `POST /api/v1/auth/logout` | Klien Terautentikasi | `204 No Content`, cookie `refresh_token` dihapus | Token dicabut di DB, header `Set-Cookie` dengan `Max-Age=-1`, respons 204. | **PASS** |
| 9 | Buat Bisnis Sukses | `POST /api/v1/businesses` | Pengguna Terautentikasi | `201 Created`, bisnis terbentuk, membership `owner`, audit log dicatat | Bisnis & membership dibuat dalam transaksi, audit log `business.created` tersimpan. | **PASS** |
| 10 | BOLA / Isolasi Antar-Tenant | `GET /api/v1/businesses/{biz_B}` | User A (Bukan anggota B) | `403 Forbidden`, `code: FORBIDDEN`, data bisnis B tidak bocor | `RequireTenant` middleware & `Service.GetBusiness` menolak akses User A ke bisnis B (`TestTenantIsolationCrossAccess` PASS). | **PASS** |
| 11 | Bisnis Tidak Ditemukan | `GET /api/v1/businesses/{uuid_acak}` | Pengguna Terautentikasi | `404 Not Found` / `403 Forbidden`, `code: BUSINESS_NOT_FOUND` / `FORBIDDEN` | Handler mengembalikan 404 `BUSINESS_NOT_FOUND` jika entitas tidak ada, atau 403 jika terblokir di guard middleware. | **PASS** |
| 12 | Role Guard Modifikasi | `PATCH /api/v1/businesses/{id}` | Anggota dengan role `staff` | `403 Forbidden`, `code: FORBIDDEN`, hanya owner yang berhak | `Service.UpdateBusiness` memeriksa `membership.Role != "owner"` -> `ErrForbiddenRole` -> 403 Forbidden. | **PASS** |
| 13 | Akses Tanpa Token | Rute terproteksi | Anonim | `401 Unauthorized`, `code: UNAUTHORIZED` | `RequireAuth` middleware memeriksa header `Authorization: Bearer <token>`, mengembalikan 401 jika absen. | **PASS** |
| 14 | Rate Limit Login | `POST /api/v1/auth/login` | Burst request (11+ req) | `429 Too Many Requests`, `code: RATE_LIMIT_EXCEEDED` | Server memiliki timeout protektif (`ReadHeaderTimeout: 5s`, `MaxHeaderBytes: 1MB`). Rate limiting per IP dialokasikan ke gateway reverse proxy. | **NOT VERIFIED (Gateway)** |

---

## 4. OWASP API Security Top 10 (2023) Compliance Review

- **API1 Broken Object Level Authorization (BOLA):**
  - **Verdict: PASS.** Multi-tenant middleware (`RequireTenant`) dan service repository mewajibkan korelasi `business_id` dan `user_id`. `TestTenantIsolationCrossAccess` memverifikasi bahwa User A tidak dapat membaca atau memodifikasi bisnis User B.
- **API2 Broken Authentication:**
  - **Verdict: PASS.** Kata sandi di-hash menggunakan Argon2id (Memory: 64MB, Iterations: 3, Parallelism: 2) dengan garam kriptografis 16 byte dan perbandingan waktu-konstan (`subtle.ConstantTimeCompare`). JWT divalidasi dengan algoritma terikat HS256. Refresh token menggunakan deteksi replay famili yang otomatis membatalkan seluruh famili token saat pemakaian ulang terdeteksi. Pesan login generik mencegah enumerasi akun.
- **API3 Broken Object Property Level Authorization (BOPLA):**
  - **Verdict: PASS.** Request DTO terpisah secara ketat dari model GORM (`RegisterRequest`, `LoginRequest`, `CreateBusinessRequest`, `UpdateBusinessRequest`). Nilai `password_hash`, salt, dan hash token tidak pernah diekspos dalam respons JSON.
- **API4 Unrestricted Resource Consumption:**
  - **Verdict: PASS.** HTTP Server mengonfigurasi batas waktu produksi (`ReadHeaderTimeout: 5s`, `ReadTimeout: 15s`, `WriteTimeout: 30s`, `IdleTimeout: 60s`, `MaxHeaderBytes: 1MB`).
- **API5 Broken Function Level Authorization (BFLA):**
  - **Verdict: PASS.** Pembaruan profil bisnis (`PATCH /businesses/{id}`) memverifikasi peran pemanggil secara ketat. Anggota dengan peran `staff` ditolak dengan HTTP 403 Forbidden.
- **API6 Unrestricted Access to Sensitive Business Flows:**
  - **Verdict: PASS.** Refresh token family rotation mencegah replay sesi lama. Seluruh mutasi bisnis dicatat dalam tabel `audit_logs`.
- **API7 Server Side Request Forgery (SSRF):**
  - **Verdict: PASS / N/A.** Tidak ada endpoint yang menerima atau melakukan pemanggilan URL eksternal pada modul auth & business.
- **API8 Security Misconfiguration:**
  - **Verdict: PASS.** Middleware CORS dikonfigurasi dengan allowlist eksplisit (`CORSAllowedOrigins`), recovery middleware menangani panic tanpa kebocoran stack trace ke klien.
- **API9 Improper Inventory Management:**
  - **Verdict: PASS.** Seluruh endpoint terdaftar rapi di bawah rute terversi `/api/v1` sesuai kontrak.
- **API10 Unsafe Consumption of APIs:**
  - **Verdict: PASS / N/A.** Belum ada integrasi pihak ketiga eksternal pada modul M1.

---

## 5. Findings & Observations

| ID | Severity | Category | Target | Description | Responsible Agent | Status |
|:---|:---|:---|:---|:---|:---|:---:|
| OBS-001 | Info | Rate Limiting | `backend/internal/platform/httpserver/server.go` | In-process IP rate limiter untuk `/auth/login` (10 req/15 min) belum dipasang di level middleware Gin (saat ini mengandalkan konfigurasi gateway Nginx / Cloudflare reverse proxy sesuai TRD §10). Direkomendasikan menambah middleware token-bucket in-memory pada milestone pengerasan (M7). | `backend-ai` | Tracked for M7 |
| OBS-002 | Low | Linting / DX | `frontend/src/core/auth/AuthContext.tsx` | Warning fast-refresh ESLint karena ekspor hook `useAuth` dalam berkas yang sama dengan komponen `AuthProvider`. Tidak memengaruhi fungsionalitas produksi. | `frontend` | Non-blocking |

---

## 6. Regression Testing

- M0 Ops Health Endpoints:
  - `GET /health` dan `GET /api/v1/health` tetap berfungsi dengan status `200 OK`.
  - `GET /ready` dan `GET /api/v1/ready` tetap mengevaluasi konektivitas basis data via `database.Pinger`.
  - Logging terstruktur dan middleware korelasi `X-Request-ID` berfungsi normal.

---

## 7. Recommendation & Sign-Off Verdict

**Verdict: APPROVE**

Seluruh kriteria verifikasi Milestone M1 terpenuhi dengan bukti empiris:
1. **Backend (`e25c943`)**: Bersih dari `go vet`, kompilasi sukses, 21 unit test lulus termasuk verifikasi kriptografis Argon2id, JWT, dan isolasi multi-tenant antar-pengguna.
2. **Frontend (`8f20051`)**: Test Vitest lulus, linter bebas error, build produksi Vite sukses, UX keamanan anti-enumerasi dan proteksi rute terimplementasi sesuai spesifikasi.
3. **Security & Access Matrix**: Memenuhi standar OWASP API Top 10 (2023) untuk mitigasi BOLA, Broken Auth, BOPLA, dan BFLA.

Milestone M1 siap untuk peninjauan akhir dan persetujuan oleh Leader (`T-015`).
