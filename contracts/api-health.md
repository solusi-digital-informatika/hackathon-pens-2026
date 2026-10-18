# API Contract: Ops & Health Endpoints

## 1. Overview
Liveness and readiness endpoints for orchestrators, load balancers, and monitoring.

- Base paths: `/` and `/api/v1`
- Auth: None (public ops endpoints)

---

## 2. Endpoints

### 2.1 Liveness Probe
Checks if the HTTP server process is running and able to handle incoming requests.

- **Endpoint**: `GET /health` (also aliased at `GET /api/v1/health`)
- **Auth**: None
- **Success Response (200 OK)**:
```json
{
  "status": "ok"
}
```

---

### 2.2 Readiness Probe
Checks if critical downstream dependencies (PostgreSQL database) are connected and responding to pings.

- **Endpoint**: `GET /ready` (also aliased at `GET /api/v1/ready`)
- **Auth**: None
- **Success Response (200 OK)** (dependencies ready):
```json
{
  "status": "ready",
  "database": "connected"
}
```
- **Error Response (503 Service Unavailable)** (dependency unhealthy or unreachable):
```json
{
  "status": "unready",
  "database": "disconnected",
  "error": {
    "code": "DATABASE_UNAVAILABLE",
    "message": "Layanan database tidak tersedia"
  }
}
```
