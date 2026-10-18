# Kontrak Arsitektur Keamanan, Hardening API, Tooling Seed CLI & DB Backup Drill (M7)

## Metadata
- **ID Kontrak**: API-M7-001
- **Task ID**: T-071
- **Penulis**: Backend & AI Engineer (`backend-ai`) / Systems Architect
- **Tanggal**: 19 Oktober 2026
- **Status**: APPROVED
- **Target Audience**: Backend & AI Engineer (`backend-ai`), Frontend Engineer (`frontend`), QA Engineer (`tester`), Systems Lead
- **Dokumen Terkait**:
  - `project-map/trd.md` (§8 CSV Spec, §9 API Inventory, §10 Security & Rate Limits, §12 Test Strategy & Targets, §14 Manual Tasks, TD-05 Tenant Isolation, TD-06 Auth, TD-11 Audit, TD-13 API Conventions)
  - `contracts/erd.md` (Skema Relasional PostgreSQL 21 Tabel, Foreign Keys Komposit, Snapshot Biaya)
  - `contracts/ui-hardening-security.md` (UI-M7-001: Global Error Boundary, Offline UX, Session Timeout Modal, Token Auto-Refresh, WCAG AA)
- **Base Prefix**: `/api/v1`

---

## 1. Filosofi & Prinsip Hardening Keamanan Produksi

Menuju kesiapan rilis produksi (Milestone M7), arsitektur backend Teman UMKM diperkuat dengan fondasi keamanan berlapis (*defense-in-depth*). Prinsip inti yang menjadi pegangan implementasi:

1. **Deny by Default & Tenant Boundary Absolut (TD-05)**:
   - Tidak ada rute internal yang dapat diakses tanpa otentikasi valid kecuali rute publik yang secara eksplisit didaftarkan dalam *Route Inventory*.
   - Setiap entitas data bisnis dilindungi secara ganda:
     - Di lapisan HTTP: Middleware `RequireTenant` memvalidasi keanggotaan pengguna pada `:business_id`.
     - Di lapisan Service/Repository: Setiap query SQL/GORM menyertakan filter `business_id = ?` dan memanfaatkan kunci asing komposit `(business_id, id)`.
2. **Kerahasiaan Diagnostik & Zero Information Leakage**:
   - Galat server (HTTP 500) **dilarang keras** mengembalikan jejak stack trace, potongan query SQL, nama tabel database, atau parameter internal ke pengguna/klien.
   - Seluruh kegagalan dipetakan ke format galat aman berstandar RFC/TRD dengan penyertaan `X-Request-ID` untuk audit internal.
3. **Pencegahan Eksploitasi Sumber Daya (Rate Limiting & Body Protection)**:
   - Batas permintaan ketat pada endpoint sensitif (login, refresh token, unggah berkas CSV, eksekusi analisis AI).
   - Batas ukuran payload (*MaxBytesReader*) diterapkan di muka sebelum body dibaca oleh parser JSON/multipart untuk mencegah serangan kehabisan memori (*Denial of Service*).
4. **Keterverifikasian Melalui Pengujian Otomatis (Automated Route Inventory Test)**:
   - Mencegah *route creep* (penambahan endpoint baru oleh pengembang tanpa proteksi otorisasi). Test suite wajib gagal secara otomatis jika ada rute di router Gin yang tidak terdaftar dalam matriks otorisasi.
5. **Kesiapan Bencana Teruji (Disaster Recovery & Restore Drill - TRD §10, §14)**:
   - Prosedur backup dan pemulihan data tidak hanya berupa dokumen rencana, melainkan prosedur teruji mandiri dengan skrip verifikasi integritas data dan checksum paritas baris.

---

## 2. Inventarisasi Otorisasi Rute (Route Inventory Contract)

Backend Teman UMKM memiliki **46 rute HTTP** terdaftar. Setiap rute diklasifikasikan ke dalam 4 tingkatan proteksi otorisasi:

### 2.1 Definisi Level Proteksi

| Level Proteksi | Nama Level | Penjelasan & Persyaratan Keamanan |
| :--- | :--- | :--- |
| **Level 0** | `Public` | Terbuka untuk umum tanpa header otorisasi. Terbatas pada health check dan pendaftaran/otentikasi awal. |
| **Level 1** | `Authenticated User` | Wajib menyertakan Bearer JWT access token yang valid (`RequireAuth`). Klaim `user_id` disuntikkan ke konteks. |
| **Level 2** | `Tenant Scoped (Member/Owner)` | Wajib lolos `RequireAuth` DAN `RequireTenant`. Pengguna harus terdaftar aktif sebagai `owner` atau `staff` pada `:business_id` di jalur URL. |
| **Level 3** | `Tenant Scoped (Owner Only)` | Wajib lolos `RequireAuth` DAN `RequireTenant`, dengan evaluasi peran `membership_role == 'owner'`. Staf operasional ditolak dengan `403 Forbidden`. |

---

### 2.2 Tabel Inventarisasi Rute Komprehensif (46 Endpoint)

| # | HTTP Method | Jalur Rute (Path Pattern) | Modul | Level Proteksi | Batas Ukuran Body | Rate Limiting | Kebijakan Audit Log |
| :-: | :--- | :--- | :--- | :--- | :-: | :--- | :--- |
| 1 | `GET` | `/health` | Ops | **Level 0 (Public)** | 0 KB | Tanpa batas | Tidak |
| 2 | `GET` | `/ready` | Ops | **Level 0 (Public)** | 0 KB | Tanpa batas | Tidak |
| 3 | `GET` | `/api/v1/health` | Ops | **Level 0 (Public)** | 0 KB | Tanpa batas | Tidak |
| 4 | `GET` | `/api/v1/ready` | Ops | **Level 0 (Public)** | 0 KB | Tanpa batas | Tidak |
| 5 | `POST` | `/api/v1/auth/register` | Auth | **Level 0 (Public)** | 64 KB | 10 req / 15 min (IP) | Tidak |
| 6 | `POST` | `/api/v1/auth/login` | Auth | **Level 0 (Public)** | 64 KB | 10 req / 15 min (IP + Email) | Ya (Login Success/Fail) |
| 7 | `POST` | `/api/v1/auth/refresh` | Auth | **Level 0 (Public)** | 64 KB | 10 req / 15 min (IP + Token) | Ya (Token Rotated/Revoked) |
| 8 | `POST` | `/api/v1/auth/logout` | Auth | **Level 0 (Public)** | 64 KB | 20 req / 15 min (IP) | Ya (Session Terminated) |
| 9 | `GET` | `/api/v1/auth/me` | Auth | **Level 1 (Auth User)** | 0 KB | 60 req / min | Tidak |
| 10 | `POST` | `/api/v1/businesses` | Business | **Level 1 (Auth User)** | 256 KB | 10 req / jam | Ya (`business.created`) |
| 11 | `GET` | `/api/v1/businesses/:business_id` | Business | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 12 | `PATCH` | `/api/v1/businesses/:business_id` | Business | **Level 3 (Owner Only)** | 256 KB | 30 req / min | Ya (`business.updated`) |
| 13 | `GET` | `/api/v1/businesses/:business_id/products` | Catalog | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 14 | `POST` | `/api/v1/businesses/:business_id/products` | Catalog | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`product.created`) |
| 15 | `GET` | `/api/v1/businesses/:business_id/products/:product_id` | Catalog | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 16 | `PATCH` | `/api/v1/businesses/:business_id/products/:product_id` | Catalog | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`product.updated`) |
| 17 | `GET` | `/api/v1/businesses/:business_id/products/:product_id/cost-versions` | Catalog | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 18 | `GET` | `/api/v1/businesses/:business_id/products/:product_id/price-history` | Catalog | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 19 | `POST` | `/api/v1/businesses/:business_id/pricing-calculator` | Catalog | **Level 2 (Tenant Scoped)** | 256 KB | 60 req / min | Tidak |
| 20 | `GET` | `/api/v1/businesses/:business_id/customers` | Customer | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 21 | `POST` | `/api/v1/businesses/:business_id/customers` | Customer | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`customer.created`) |
| 22 | `GET` | `/api/v1/businesses/:business_id/customers/:customer_id` | Customer | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 23 | `PUT` | `/api/v1/businesses/:business_id/customers/:customer_id` | Customer | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`customer.updated`) |
| 24 | `GET` | `/api/v1/businesses/:business_id/transactions` | Sales | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 25 | `POST` | `/api/v1/businesses/:business_id/transactions` | Sales | **Level 2 (Tenant Scoped)** | 1 MB | 60 req / min | Ya (`transaction.created`) |
| 26 | `GET` | `/api/v1/businesses/:business_id/transactions/:transaction_id` | Sales | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 27 | `POST` | `/api/v1/businesses/:business_id/transactions/:transaction_id/cancel` | Sales | **Level 3 (Owner Only)** | 256 KB | 30 req / min | Ya (`transaction.cancelled`) |
| 28 | `POST` | `/api/v1/businesses/:business_id/transactions/:transaction_id/returns` | Sales | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`transaction.returned`) |
| 29 | `GET` | `/api/v1/businesses/:business_id/transactions/:transaction_id/returns` | Sales | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 30 | `GET` | `/api/v1/businesses/:business_id/dashboard/summary` | Finance | **Level 2 (Tenant Scoped)** | 0 KB | 60 req / min | Tidak |
| 31 | `GET` | `/api/v1/businesses/:business_id/dashboard/trends` | Finance | **Level 2 (Tenant Scoped)** | 0 KB | 60 req / min | Tidak |
| 32 | `GET` | `/api/v1/businesses/:business_id/dashboard/data-health` | Finance | **Level 2 (Tenant Scoped)** | 0 KB | 60 req / min | Tidak |
| 33 | `GET` | `/api/v1/businesses/:business_id/imports/templates/:type` | Importer | **Level 2 (Tenant Scoped)** | 0 KB | 60 req / min | Tidak |
| 34 | `POST` | `/api/v1/businesses/:business_id/imports` | Importer | **Level 2 (Tenant Scoped)** | 10 MB (Multipart) | **5 uploads / jam (Business)** | Ya (`import.uploaded`) |
| 35 | `GET` | `/api/v1/businesses/:business_id/imports/:id` | Importer | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 36 | `PUT` | `/api/v1/businesses/:business_id/imports/:id/mapping` | Importer | **Level 2 (Tenant Scoped)** | 512 KB | 60 req / min | Ya (`import.mapping_updated`) |
| 37 | `GET` | `/api/v1/businesses/:business_id/imports/:id/rows` | Importer | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 38 | `POST` | `/api/v1/businesses/:business_id/imports/:id/confirm` | Importer | **Level 2 (Tenant Scoped)** | 256 KB | 20 req / jam | Ya (`import.confirmed`) |
| 39 | `POST` | `/api/v1/businesses/:business_id/imports/:id/cancel` | Importer | **Level 2 (Tenant Scoped)** | 256 KB | 20 req / jam | Ya (`import.cancelled`) |
| 40 | `GET` | `/api/v1/businesses/:business_id/imports/:id/errors/download` | Importer | **Level 2 (Tenant Scoped)** | 0 KB | 30 req / min | Tidak |
| 41 | `POST` | `/api/v1/businesses/:business_id/analysis/run` | Insight | **Level 2 (Tenant Scoped)** | 128 KB | **1 run / 10 min (Business)** | Ya (`analysis.triggered`) |
| 42 | `GET` | `/api/v1/businesses/:business_id/insights` | Insight | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 43 | `GET` | `/api/v1/businesses/:business_id/insights/:id` | Insight | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 44 | `GET` | `/api/v1/businesses/:business_id/recommendations` | Insight | **Level 2 (Tenant Scoped)** | 0 KB | 120 req / min | Tidak |
| 45 | `PATCH` | `/api/v1/businesses/:business_id/recommendations/:id` | Insight | **Level 2 (Tenant Scoped)** | 256 KB | 60 req / min | Ya (`recommendation.status_changed`) |
| 46 | `POST` | `/api/v1/businesses/:business_id/products/:product_id/deactivate` | Catalog | **Level 3 (Owner Only)** | 128 KB | 30 req / min | Ya (`product.deactivated`) |

---

### 2.3 Spesifikasi Pengujian Otorisasi Otomatis (*Automated Route Inventory Test Specification*)

Per TRD §10, pengujian keamanan wajib memiliki pengujian otomatis (*fail-closed*) yang menjamin tidak ada rute baru yang dibuat tanpa perlindungan otorisasi yang diuji:

1. **Prinsip Fail-Closed Route Registry**:
   - Pengujian membaca seluruh rute terdaftar langsung dari mesin Gin (`router.Routes()`).
   - Setiap rute `(METHOD, PATH)` dicocokkan dengan tabel *Declared Route Authorization Matrix*.
   - **Kondisi Kegagalan Test (Fatal)**:
     - Jika terdapat rute di dalam router Gin yang **tidak terdaftar** dalam matriks declared inventory: Test Suite **GAGAL SEKETIKA** (*Unregistered route detected*).
     - Jika rute bertingkat `Tenant Scoped` tidak memiliki pengujian *cross-tenant 403 Forbidden*: Test Suite **GAGAL**.
     - Jika rute bertingkat `Authenticated User` atau `Tenant Scoped` dapat diakses tanpa token Bearer dan tidak menghasilkan `401 Unauthorized`: Test Suite **GAGAL**.

2. **Spesifikasi Algoritma Test (`internal/platform/httpserver/route_inventory_test.go`)**:
   ```go
   // Pseudo-code arsitektur verifikasi otomatis rute
   func TestRouteInventoryAuthorizationIntegrity(t *testing.T) {
       router := httpserver.NewRouter(opts)
       routes := router.Routes()
       
       for _, r := range routes {
           inventoryEntry, exists := DeclaredRouteInventory[r.Method + " " + r.Path]
           if !exists {
               t.Fatalf("SECURITY VIOLATION: Route %s %s is mounted in Gin but not registered in Security Route Inventory Contract!", r.Method, r.Path)
           }
           
           // Verifikasi Pengujian Otorisasi
           switch inventoryEntry.ProtectionLevel {
           case LevelTenantScoped, LevelOwnerOnly:
               // 1. Uji tanpa token -> WAJIB 401 UNAUTHORIZED
               assertStatusWithoutToken(t, router, r.Method, r.Path, http.StatusUnauthorized)
               
               // 2. Uji token User B terhadap Tenant A -> WAJIB 403 FORBIDDEN
               assertStatusCrossTenant(t, router, r.Method, r.Path, tokenUserB, tenantA_ID, http.StatusForbidden)
           case LevelAuthenticated:
               // Uji tanpa token -> WAJIB 401 UNAUTHORIZED
               assertStatusWithoutToken(t, router, r.Method, r.Path, http.StatusUnauthorized)
           }
       }
   }
   ```

3. **Verifikasi Khusus Rute `Owner Only` (Level 3)**:
   - Rute bertanda Level 3 (`PATCH /businesses/:business_id`, `POST /transactions/:transaction_id/cancel`, `POST /products/:product_id/deactivate`):
   - Jika dipanggil oleh pengguna berstatus keanggotaan `staff` (meskipun berada di dalam bisnis yang sama), sistem wajib mengembalikan respons:
     - HTTP Status: `403 Forbidden`
     - Response Body:
       ```json
       {
         "error": {
           "code": "FORBIDDEN",
           "message": "Aksi ini memerlukan hak akses Pemilik Usaha (Owner)."
         }
       }
       ```

---

## 3. Spesifikasi Keamanan HTTP & Transport Hardening

Untuk memenuhi standar keamanan web modern (TRD §10 & OWASP API Top 10), seluruh pertukaran data dilindungi oleh middleware hardening terpusat.

### 3.1 Security Headers Middleware

Setiap respons HTTP dari server backend (baik status 2xx, 4xx, maupun 5xx) wajib menyertakan header keamanan berikut:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=(), payment=()
X-XSS-Protection: 0
Cache-Control: no-store, no-cache, must-revalidate, proxy-revalidate
Pragma: no-cache
```

#### Rincian Justifikasi Parameter:
1. `Content-Security-Policy`: Membatasi muatan sumber daya hanya dari origin aplikasi sendiri (`'self'`), melarang embedding ke dalam iframe penyerang (`frame-ancestors 'none'`), dan menonaktifkan evaluasi skrip pihak ketiga yang tidak tepercaya.
2. `X-Content-Type-Options: nosniff`: Mencegah browser melakukan inferensi MIME-type terhadap konten berbahaya (contoh: berkas ekspor CSV atau gambar yang disamarkan sebagai executable).
3. `X-Frame-Options: DENY`: Memitigasi serangan *Clickjacking* secara mutlak.
4. `Strict-Transport-Security` (HSTS): Memaksa seluruh koneksi menggunakan HTTPS selama minimal 1 tahun (`31536000` detik), berlaku untuk seluruh subdomain, dan siap didaftarkan ke HSTS preload list browser.
5. `Referrer-Policy: strict-origin-when-cross-origin`: Menjaga privasi path URL dan query parameter agar tidak bocor ke origin eksternal saat navigasi keluar.
6. `Permissions-Policy`: Menonaktifkan akses sensor perangkat keras browser (kamera, mikrofon, geolokasi) yang tidak dibutuhkan oleh sistem operasional UMKM.
7. `X-XSS-Protection: 0`: Standar modern OWASP yang menonaktifkan filter XSS browser lama yang memiliki celah kerentanan side-channel, digantikan sepenuhnya oleh CSP.
8. `Cache-Control: no-store`: Wajib pada seluruh rute API yang mengembalikan data bisnis, finansial, dan identitas pengguna guna mencegah proxy/browser menyimpan data sensitif di media lokal publik.

---

### 3.2 Penegakan Batas Frekuensi Permintaan (Rate Limiting Contract)

Sesuai TRD §10, diterapkan rate limiting ketat berbasis token bucket / sliding window in-memory:

| Ruang Lingkup Proteksi | Target Endpoint | Batas Maksimum | Kunci Pelacak (Rate Limit Key) | Perilaku Saat Terlampaui |
| :--- | :--- | :--- | :--- | :--- |
| **Autentikasi & Sesi** | `POST /auth/login`<br>`POST /auth/refresh` | 10 request / 15 menit | `client_ip + ":" + lower(email)` atau `client_ip + ":" + token_hash` | HTTP 429 (`RATE_LIMIT_EXCEEDED`) |
| **Pendaftaran Akun** | `POST /auth/register` | 10 request / 15 menit | `client_ip` | HTTP 429 (`RATE_LIMIT_EXCEEDED`) |
| **Unggah Batch CSV** | `POST /businesses/:business_id/imports` | 5 unggahan / 1 jam | `business_id` | HTTP 429 (`IMPORT_RATE_LIMIT_EXCEEDED`) |
| **Pemicuan Wawasan Manual** | `POST /businesses/:business_id/analysis/run` | 1 eksekusi / 10 menit | `business_id` | HTTP 429 (`ANALYSIS_COOLDOWN_ACTIVE`) |
| **Konkurensi Wawasan Ganda**| `POST /businesses/:business_id/analysis/run` | 1 analisis aktif per bisnis | Database Partial Unique Index | HTTP 409 (`ANALYSIS_ALREADY_RUNNING`) |

#### Format Respons Standar HTTP 429:
Header HTTP:
- `Retry-After: <detik_tersisa>` (Contoh: `Retry-After: 480`)
- `X-RateLimit-Limit: <total_kuota>`
- `X-RateLimit-Remaining: 0`
- `X-RateLimit-Reset: <unix_epoch_timestamp>`

Body Respons (JSON):
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Batas frekuensi permintaan terlampaui. Silakan coba kembali dalam beberapa saat.",
    "details": {
      "retry_after_seconds": 480,
      "limit": 10,
      "window": "15m"
    }
  }
}
```

---

### 3.3 Pembatasan Ukuran Body Permintaan (Request Body Hardening)

Untuk melindungi memori dan CPU server dari serangan DoS berbasis payload besar (*memory exhaustion*):

1. **Default Global Request Limit**: Maksimal **1 MB** (`1 << 20 bytes`) untuk seluruh endpoint JSON (`POST`, `PUT`, `PATCH`). Gin router membungkus body dengan `http.MaxBytesReader(c.Writer, c.Request.Body, 1<<20)`. Jika terlampaui, server langsung memutus streaming body dan mengembalikan `413 Payload Too Large` (`REQUEST_BODY_TOO_LARGE`).
2. **CSV Multipart Upload Limit**: Dikecualikan untuk endpoint `POST /api/v1/businesses/:business_id/imports`, batas ukuran berkas ditingkatkan hingga maksimal **10 MB** (`10 << 20 bytes`) sesuai TRD §8. Jika berkas CSV melebihi 10 MB atau melebihi 20.000 baris, parser streaming segera menghentikan pemrosesan dengan error `400 Bad Request` (`FILE_SIZE_EXCEEDED`).
3. **Konfigurasi HTTP Server Timeout (TRD §10)**:
   ```go
   srv := &http.Server{
       Addr:              fmt.Sprintf(":%d", cfg.Port),
       Handler:           router,
       ReadHeaderTimeout: 5 * time.Second,  // Melindungi dari Slowloris attacks
       ReadTimeout:       15 * time.Second, // Waktu maksimal membaca seluruh body request
       WriteTimeout:      30 * time.Second, // Waktu maksimal mengirimkan respons
       IdleTimeout:       60 * time.Second, // Waktu keep-alive koneksi idle
       MaxHeaderBytes:    1 << 20,          // Maksimal 1 MB ukuran header
   }
   ```

---

### 3.4 Kebijakan CORS Ketat (Cross-Origin Resource Sharing)

1. **Strict Origin Matching**:
   - Origin diperiksa secara eksplisit terhadap daftar konfigurasi lingkungan `CORS_ALLOWED_ORIGINS` (contoh: `http://localhost:5173,https://app.temanumkm.id`).
   - Dilarang keras menggunakan wildcard `*` bila `AllowCredentials: true` diaktifkan.
2. **Allowed Methods**: `GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD`.
3. **Allowed Headers**: `Origin, Content-Type, Content-Length, Accept, Authorization, X-Request-ID, X-Idempotency-Key`.
4. **Exposed Headers**: `X-Request-ID, Retry-After, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, Content-Disposition`.
5. **Preflight Max Age**: `86400` detik (24 jam) untuk mengurangi overhead preflight `OPTIONS` di browser.

---

## 4. Spesifikasi Tooling Seeding Mandiri (`cmd/seed`)

Tooling `cmd/seed` merupakan utilitas CLI mandiri Go untuk menginisialisasi dataset referensi realistis berskala produksi (TRD §12). Dataset ini digunakan oleh tim Backend-AI untuk verifikasi performa, tim Frontend untuk pengembangan visual, dan tim QA/Tester untuk mengaudit isolasi tenant dan kalkulasi finansial.

### 4.1 Antarmuka CLI & Perintah Eksekusi

```bash
# Eksekusi standar (Dataset Referensi Produksi)
go run ./cmd/seed

# Opsi Pembersihan Bersih (Wipe existing seed tenants before inserting)
go run ./cmd/seed --clean

# Opsi Skala Ringan (Untuk pengujian CI/CD cepat)
go run ./cmd/seed --scale=minimal

# Opsi Custom Database URL
go run ./cmd/seed --database-url="postgres://postgres:postgres@localhost:5432/teman_umkm?sslmode=disable"
```

#### Spesifikasi Parameter / Flags:
- `--clean`: (Boolean, default `false`). Jika `true`, skrip menghapus data lama milik Bisnis A dan Bisnis B secara aman berbasis cascading delete transaksional sebelum melakukan generasi baru.
- `--scale`: (String, pilihan: `standard` | `minimal`, default `standard`).
  - `standard`: ~10.000 transaksi realistis, ~200 produk, ~2.000 pelanggan (memenuhi TRD §12).
  - `minimal`: 100 transaksi, 20 produk, 50 pelanggan (untuk integrasi cepat).
- `--database-url`: (String, opsional). Jika tidak disertakan, membaca dari variabel lingkungan `DATABASE_URL` atau file `.env`.

---

### 4.2 Spesifikasi Bisnis A: Primary Realistic UMKM ("Warung Sembako & Kopi Lestari")

Bisnis utama yang mensimulasikan operasional UMKM retail & kedai kopi dinamis selama rentang waktu 180 hari ke belakang.

#### 1. Identitas Tenant & Akses Pengguna
- **ID Bisnis**: `0192e000-0000-7000-8000-000000000001` (UUIDv7 deterministik)
- **Nama Bisnis**: `Warung Sembako & Kopi Lestari`
- **Tipe Usaha**: `F&B & Retail`
- **Mata Uang**: `IDR`
- **Zona Waktu**: `Asia/Jakarta`
- **Kredensial Pengguna**:
  - **Owner**:
    - Email: `owner.lestari@temanumkm.id`
    - Kata Sandi: `Password123!` (Hash: argon2id)
    - Peran: `owner`
  - **Staff**:
    - Email: `staff.lestari@temanumkm.id`
    - Kata Sandi: `Password123!` (Hash: argon2id)
    - Peran: `staff`

#### 2. Katalog Produk (~200 Produk)
- **160 Produk Ber-HPP Aktif**:
  - Memiliki `product_cost_versions` aktif dengan tanggal efektif `effective_from` yang sesuai.
  - Memiliki rincian `product_cost_components` (contoh: "Bahan Baku Biji Kopi", "Gula Pasir", "Cup & Sedotan", "Tenaga Kerja Langsung").
  - Margin sehat terdistribusi antara 25% hingga 50%.
- **40 Produk Tanpa HPP (`NULL` Cost Version)**:
  - Dibuat sengaja tanpa komponen biaya untuk memicu aturan wawasan `DATA_MISSING_COST_V1` dan menguji penanganan null-margin engine keuangan.
- **5 Produk Anomali Negatif Margin**:
  - Harga jual berada di bawah HPP aktif (`selling_price < hpp_per_unit`) untuk memicu aturan wawasan `PROD_BELOW_COST_V1`.
- **10 Produk Low Margin**:
  - Margin produk tertinggal ≥ 10 poin persentase di bawah margin rata-rata usaha untuk memicu aturan `PROD_LOW_MARGIN_V1`.

#### 3. Database Pelanggan (~2.000 Pelanggan)
- **Distribusi Pareto (Top 20% Pareto)**:
  - ~400 pelanggan berbelanja frekuensi tinggi dan berkontribusi terhadap ~80% omzet kumulatif.
- **Kohort Anomali Aktivitas Pelanggan**:
  - **Overdue Customers**: 15 pelanggan reguler yang tidak berbelanja melebihi `max(1.5 × median, median + 7)` untuk memicu `CUST_OVERDUE_V1`.
  - **Frequency Drop Customers**: 10 pelanggan yang mengalami penurunan transaksi ≥ 50% pada 90 hari terakhir untuk memicu `CUST_FREQ_DROP_V1`.
  - **Value Drop Customers**: 10 pelanggan dengan penurunan nilai belanja rata-rata ≥ 40% untuk memicu `CUST_VALUE_DROP_V1`.
- **Komplain Pelanggan**:
  - 5 komplain berstatus `open` atau `in_progress` yang telah melewati 3 hari kalender untuk memicu `CUST_COMPLAINT_OPEN_V1`.

#### 4. Riwayat Transaksi (~10.000 Transaksi, ~50.000 Baris Item)
- **Rentang Waktu**: Tersebar merata dalam 180 hari terakhir dengan pola lonjakan akhir pekan realistis.
- **Tautan Pelanggan**:
  - 70% transaksi (~7.000 tx) tertaut dengan pelanggan terdaftar.
  - 30% transaksi (~3.000 tx) berupa transaksi anonim/tamu (*guest checkout*) untuk menguji ambang batas aturan `DATA_UNLINKED_TX_V1`.
- **Status Transaksi**:
  - 95% transaksi berstatus `paid` (lunas).
  - 3% transaksi berstatus `unpaid` (piutang/tempo).
  - 2% transaksi berstatus `cancelled` (batal dengan audit log terkait).
- **Integritas Snapshot Biaya (TD-08)**:
  - Seluruh item transaksi merekam `recorded_unit_cost` yang identik dengan HPP aktif pada tanggal transaksi, atau bernilai `NULL` jika produk tidak memiliki HPP saat itu.
- **Retur Penjualan (`transaction_returns`)**:
  - ~50 transaksi memiliki rekaman retur barang sebagian atau penuh dengan penyesuaian nominal dan stok yang valid.

#### 5. Wawasan & Rekomendasi Terhitung
- Menghasilkan 1 rekaman `analysis_runs` historis berstatus `completed`.
- Menghasilkan wawasan (`insights`) dari ke-8 aturan deterministik lengkap dengan rekaman pembuktian (`insight_sources`).
- Menghasilkan rekomendasi tindakan (`recommendations`) dengan status kombinasi (`new`, `reviewed`, `in_progress`, dan `dismissed` ber-jeda cooldown).

---

### 4.3 Spesifikasi Bisnis B: Isolation Benchmark ("Toko Elektronik Makmur Jaya")

Entitas pembanding independen mutlak yang digunakan untuk memverifikasi proteksi kebocoran data silang (*Zero Cross-Tenant Leakage*).

#### 1. Identitas Tenant & Akses Pengguna
- **ID Bisnis**: `0192e000-0000-7000-8000-000000000002` (UUIDv7 deterministik)
- **Nama Bisnis**: `Toko Elektronik Makmur Jaya`
- **Tipe Usaha**: `Retail Elektronik`
- **Mata Uang**: `IDR`
- **Zona Waktu**: `Asia/Jakarta`
- **Kredensial Pengguna**:
  - **Owner**:
    - Email: `owner.makmur@temanumkm.id`
    - Kata Sandi: `Password123!` (Hash: argon2id)
    - Peran: `owner`
  - **Staff**:
    - Email: `staff.makmur@temanumkm.id`
    - Kata Sandi: `Password123!` (Hash: argon2id)
    - Peran: `staff`

#### 2. Volume Data Terisolasi
- 50 Produk Elektronik (Komputer, Kabel, Adaptor, Aksesori).
- 200 Pelanggan independen (tidak memiliki irisan nomor telepon/email dengan Bisnis A).
- 500 Transaksi penjualan independen.

#### 3. Tujuan Uji Keamanan Matriks Isolasi (Security Isolation Benchmark)
Pengujian otomatis menggunakan token `owner.makmur@temanumkm.id` atau `staff.makmur@temanumkm.id` untuk:
1. Membaca data Bisnis A (`GET /businesses/0192e000-0000-7000-8000-000000000001/...`) -> **Wajib 403 Forbidden**.
2. Mengubah data produk Bisnis A (`PATCH /products/:id`) -> **Wajib 403 Forbidden**.
3. Menyisipkan transaksi pada Bisnis A -> **Wajib 403 Forbidden**.
4. Melakukan query langsung ke database dengan UUID Bisnis A -> Kunci asing komposit PostgreSQL wajib menolak referensi silang.

---

## 5. Prosedur Backup & Restore Drill Database (TRD §10, §14)

Untuk menjamin kelangsungan operasional data UMKM dari risiko kegagalan infrastruktur, korupsi disk, atau *human error*, prosedur backup dan pemulihan bencana (*Disaster Recovery*) diatur secara deterministik.

### 5.1 Target Metrik Pemulihan Bencana (DR SLA Targets)

| Metrik | Target SLA | Penjelasan |
| :--- | :--- | :--- |
| **RPO (Recovery Point Objective)** | **< 1 Jam** | Kehilangan data maksimal yang dapat ditoleransi dalam skenario bencana server (didukung oleh snapshot berkala & WAL archiving). |
| **RTO (Recovery Time Objective)** | **< 15 Menit** | Waktu maksimal yang dibutuhkan untuk memulihkan database dari arsip backup hingga sistem siap melayani trafik kembali pada dataset 10.000 transaksi. |
| **Data Parity Integrity** | **100% (Zero Loss)** | Seluruh baris, kunci asing komposit, dan totalitas saldo moneter wajib cocok presisi tanpa selisih 1 Rupiah pun. |

---

### 5.2 Format & Konfigurasi Ekspor Backup (`pg_dump`)

Prosedur backup menggunakan format biner terkompresi kustom PostgreSQL (`-Fc`). Format ini mendukung:
- Kompresi tinggi (menghemat ruang penyimpanan hingga 75%).
- Pemulihan paralel multi-utas (`pg_restore -j <threads>`).
- Pemulihan selektif skema atau tabel tertentu jika diperlukan.
- Pengecekan integritas internal checksum oleh utilitas PostgreSQL.

#### Perintah Ekspor Resmi:
```bash
# Direktori penyimpanan backup
mkdir -p ./backups

# Eksekusi pg_dump dengan format kustom terkompresi
pg_dump \
  --host="${DB_HOST:-localhost}" \
  --port="${DB_PORT:-5432}" \
  --username="${DB_USER:-postgres}" \
  --dbname="${DB_NAME:-teman_umkm}" \
  --format=custom \
  --compress=9 \
  --no-acl \
  --no-owner \
  --verbose \
  --file="./backups/teman_umkm_backup_$(date +%Y%m%d_%H%M%S).dump"
```

---

### 5.3 Prosedur Restore Drill Mandiri (Langkah Demi Langkah)

Prosedur drill wajib dijalankan secara berkala (minimal setiap sprint/rilis besar) pada lingkungan *sandbox* terisolasi tanpa mengganggu database produksi.

```
┌──────────────────────────┐
│   Database Sumber        │
│   (teman_umkm)           │
└────────────┬─────────────┘
             │ 1. Hitung Checksum & Snapshot Paritas Pra-Backup
             ▼
┌──────────────────────────┐
│   Arsip Dump Kustom      │ ──> Simpan Checksum SHA-256 Berkas
│   (teman_umkm_*.dump)    │
└────────────┬─────────────┘
             │ 2. pg_restore ke Database Uji Terisolasi
             ▼
┌──────────────────────────┐
│   Sandbox Database Drill │
│ (teman_umkm_drill_verify)│
└────────────┬─────────────┘
             │ 3. Skrip Audit Paritas Baris & Agregat Finansial (Zero Drift)
             │ 4. Sanity Check Koneksi Backend (/health, /ready, /dashboard)
             ▼
┌──────────────────────────┐
│   Laporan Hasil Drill    │ ──> Bersihkan Sandbox (Safe Teardown)
│   (PASS / FAIL)          │
└──────────────────────────┘
```

#### Langkah 1: Pengambilan Snapshot Paritas Data Sumber (Pre-Backup Checksum)
Jalankan kueri SQL untuk merekam jumlah baris dan agregat finansial pada database sumber:
```sql
SELECT 'businesses' AS tbl, count(*) AS cnt FROM businesses
UNION ALL
SELECT 'users', count(*) FROM users
UNION ALL
SELECT 'products', count(*) FROM products
UNION ALL
SELECT 'customers', count(*) FROM customers
UNION ALL
SELECT 'transactions', count(*) FROM transactions
UNION ALL
SELECT 'transaction_items', count(*) FROM transaction_items
UNION ALL
SELECT 'insights', count(*) FROM insights;

-- Verifikasi Agregat Moneter Sumber
SELECT 
    coalesce(sum(total_amount), 0) AS total_gross_sales,
    count(*) AS total_paid_tx
FROM transactions 
WHERE payment_status = 'paid';
```

#### Langkah 2: Pembuatan Database Sandbox Uji Terisolasi
```bash
# Pastikan database target bersih dan terisolasi
PGPASSWORD="${DB_PASSWORD:-postgres}" psql -h "${DB_HOST:-localhost}" -U "${DB_USER:-postgres}" -c "DROP DATABASE IF EXISTS teman_umkm_drill_verify;"
PGPASSWORD="${DB_PASSWORD:-postgres}" psql -h "${DB_HOST:-localhost}" -U "${DB_USER:-postgres}" -c "CREATE DATABASE teman_umkm_drill_verify;"
```

#### Langkah 3: Eksekusi Restorasi dengan `pg_restore`
```bash
BACKUP_FILE=$(ls -t ./backups/*.dump | head -n 1)

echo "Memulai restore drill dari: ${BACKUP_FILE}"
START_TIME=$(date +%s)

PGPASSWORD="${DB_PASSWORD:-postgres}" pg_restore \
  --host="${DB_HOST:-localhost}" \
  --port="${DB_PORT:-5432}" \
  --username="${DB_USER:-postgres}" \
  --dbname="teman_umkm_drill_verify" \
  --clean \
  --if-exists \
  --no-acl \
  --no-owner \
  --verbose \
  "${BACKUP_FILE}"

RESTORE_EXIT_CODE=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if [ $RESTORE_EXIT_CODE -ne 0 ]; then
    echo "RESTORE DRILL GAGAL: pg_restore menghasilkan kode keluar ${RESTORE_EXIT_CODE}"
    exit 1
fi
echo "Restorasi selesai dalam ${DURATION} detik. (Target RTO < 900 detik)"
```

#### Langkah 4: Verifikasi Integritas & Paritas Data (Post-Restore Check)
Bandingkan data pada `teman_umkm_drill_verify` dengan snapshot Langkah 1:
1. **Row Count Match**: Jumlah baris pada seluruh 21 tabel harus identik 100%.
2. **Financial Aggregation Match**:
   - Nilai `SUM(total_amount)` transaksi berstatus `paid` harus bernilai sama persis hingga digit desimal terkecil.
3. **Composite Foreign Key Integrity**:
   - Memastikan tidak ada baris transaksi atau produk yang kehilangan integritas relasi `(business_id, id)`.
4. **Application Probe**:
   - Menjalankan backend sementara dengan `DATABASE_URL` yang mengarah ke `teman_umkm_drill_verify`.
   - Menguji endpoint `/health` (harus `200 OK`) dan `/ready` (harus `200 OK`).
   - Melakukan panggilan `GET /api/v1/businesses/0192e000-0000-7000-8000-000000000001/dashboard/summary` menggunakan token Bisnis A dan memastikan ringkasan omzet sesuai.

#### Langkah 5: Pembersihan Lingkungan Drill (Safe Teardown)
Setelah hasil dicatat dan diverifikasi, database drill dihapus untuk membebaskan ruang penyimpanan:
```bash
PGPASSWORD="${DB_PASSWORD:-postgres}" psql -h "${DB_HOST:-localhost}" -U "${DB_USER:-postgres}" -c "DROP DATABASE teman_umkm_drill_verify;"
echo "Sandbox drill berhasil dibersihkan."
```

---

## 6. McCall's Quality Factors & Verification Matrix

| # | Faktor Kualitas McCall | Penerapan Nyata dalam Kontrak M7 | Tolok Ukur Keberhasilan |
| :-: | :--- | :--- | :--- |
| 1 | **Correctness** | Pengujian otorisasi rute menyeluruh memastikan seluruh 46 endpoint mematuhi level proteksi yang dideklarasikan. | Route Inventory Test lulus 100% tanpa celah rute tak teruji. |
| 2 | **Integrity** | Penegakan batas tenant (TD-05), rate limiting pada login/refresh/CSV/analisis, dan header HTTP standar industri. | Uji akses silang tenant menghasilkan mutlak 403 Forbidden. Tidak ada bypass rate limit. |
| 3 | **Reliability** | Timeouts server ketat (5s/15s/30s), batas ukuran payload 1MB/10MB, dan prosedur restore drill teruji mandiri. | RTO pemulihan bencana < 15 menit dengan integritas data 100%. Server kebal terhadap Slowloris. |
| 4 | **Efficiency** | Rate limiting sliding-window berbasis memori tanpa latensi overhead DB, kompresi custom pg_dump tingkat 9. | Latensi otorisasi middleware < 2ms; ukuran berkas backup tereduksi hingga 75%. |
| 5 | **Usability (API)** | Kode galat stabil (`RATE_LIMIT_EXCEEDED`, `FORBIDDEN`, `UNAUTHORIZED`), header `Retry-After`, pesan galat Indonesia ramah pengguna. | Klien frontend dapat menampilkan countdown modal dan toast tanpa parsing ambigu. |
| 6 | **Maintainability** | CLI seeding mandiri `go run ./cmd/seed` menyajikan data uji terisolasi dan deterministik tanpa konfigurasi manual rumit. | Satu perintah menghasilkan 2 tenant terisi penuh dengan 10.000 transaksi realistis. |
| 7 | **Testability** | Desain middleware dan test harness memungkinkan simulasi unit test dan integration test cepat berbasis mock dan sandbox DB. | Matriks pengujian otorisasi dapat diuji secara terisolasi tanpa intervensi manual. |
| 8 | **Flexibility** | Pengaturan rate limit, origins CORS, dan ukuran payload dikonfigurasikan melalui variabel lingkungan terpusat. | Konfigurasi dapat disesuaikan per environment (dev, test, staging, production). |
| 9 | **Portability** | Format backup PostgreSQL standar (`-Fc`) dapat dipulihkan ke RDS, Cloud SQL, Supabase, mau pun kontainer Docker lokal. | Prosedur restore drill independen dari penyedia cloud hosting tertentu. |
| 10 | **Reusability** | Middleware security headers dan error handler terpusat di package `internal/platform/httpserver`. | Mudah diadopsi untuk penambahan modul baru di masa depan tanpa duplikasi kode. |
| 11 | **Interoperability**| Headers HTTP standar (`CSP`, `HSTS`, `CORS`, `RFC 6585 429 Too Many Requests`) kompatibel dengan seluruh peramban modern. | Kompatibel penuh dengan SPA Vite/React, mobile browser, dan reverse proxy (Nginx/Traefik/Cloudflare). |

---

## 7. Manual Tasks & Catatan Operasional (TRD §14)

Sebelum peluncuran ke lingkungan produksi riil, pemilik sistem wajib menyelesaikan langkah-langkah manual berikut:

1. **Konfigurasi Penyedia Cloud & Backup Terkelola (TRD §14 Item 3)**:
   - Mengaktifkan fitur automated daily snapshot dan Point-in-Time Recovery (PITR) pada penyedia database PostgreSQL terkelola (AWS RDS, GCP Cloud SQL, Supabase, atau Neon).
   - Mengonfigurasi retensi backup minimal 30 hari kalender.
2. **Pelaksanaan Restore Drill Pertama Kali (TRD §14 Item 5)**:
   - Menjalankan prosedur restore drill (§5.3) pada lingkungan staging sebelum deployment produksi pertama untuk memvalidasi RTO dan RPO aktual.
3. **Pemberian Izin Domain & Pendaftaran HSTS Preload**:
   - Memastikan sertifikat TLS/HTTPS terpasang dengan nilai cipher modern di edge/load balancer.
   - Mendaftarkan domain produksi ke direktori `hstspreload.org` setelah pengujian stabil.
4. **Audit Kepatuhan Hukum (UU PDP No. 27/2022 - TRD §14 Item 6)**:
   - Mengonfirmasi kepatuhan prosedur retensi data pelanggan dan anonimisasi data dengan penasihat hukum perusahaan.
