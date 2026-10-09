# Projects REST/JSON API

Base collection: `/projects`. JSON media type is used for request and response bodies.
Project fields are defined in `data-model.md`.

## Create project
`POST /projects`

Request body (`name` required; `description` optional per D-001 #1):
```json
{"name":"Demo film","description":"Six-shot direction-change demo"}
```
`status` is not accepted on create; the server always defaults it to `draft` (D-001 #6).
Success: `201 Created`, `Location: /projects/prj_01`, body:
```json
{"id":"prj_01","name":"Demo film","description":"Six-shot direction-change demo","status":"draft","created_at":"2026-10-09T09:00:00Z","updated_at":"2026-10-09T09:00:00Z"}
```
Invalid JSON, missing required fields, wrong types, or invalid field values: `400 Bad Request`.

## List projects
`GET /projects`

Query parameters (D-001 #8): `limit` (default 20, max 100), `offset` (default 0).
Results are sorted by `created_at` descending. Response includes `total` (D-001 #8):
```json
{"items":[{"id":"prj_01","name":"Demo film","description":"Six-shot direction-change demo","status":"draft","created_at":"2026-10-09T09:00:00Z","updated_at":"2026-10-09T09:00:00Z"}],"total":1,"limit":20,"offset":0}
```

## Get (open) project
`GET /projects/{project_id}`

Success: `200 OK`, with the complete Project representation shown above.
Unknown ID: `404 Not Found`; supports FR-01 project isolation (D-001 #10, #11).

The opened workspace separately obtains project-scoped brief and shot data.
Those endpoint contracts are outside this document; every such request must be scoped
by the stable `project_id` (D-001 #4, #11).

## Error body (all non-2xx responses)
Validation errors (`400`):
```json
{"error":{"code":"validation_error","message":"Project input is invalid","fields":[{"field":"name","message":"Required"}]}}
```
All other errors (`404`, `500`, etc.):
```json
{"error":{"code":"not_found","message":"Project not found","fields":[]}}
```
`fields` contains per-field failures on `400`; it is empty for non-validation errors.
`message` is safe for users and never exposes implementation details.

## Status codes
| Code | Meaning |
|---:|---|
| `200` | List/get succeeded |
| `201` | Project created |
| `400` | Malformed or invalid request |
| `404` | Project is unknown or inaccessible |
| `500` | Unexpected server failure; no internal details returned |

No `401` or `403` in F1; authentication is out of scope (D-001 #10).

## CORS (D-001)
Allowed origins: `http://localhost:5173` and `http://127.0.0.1:5173` only.

## Assumptions
- The service generates `id`, `created_at`, and `updated_at`.
- Create is atomic: failure leaves no partially created Project.
- Duplicate project names are allowed; identity is the unique ID (D-001 #9).
- Brief and shot endpoint shapes are out of scope for F1 (D-001 #4).
- Error envelope fields are transport metadata, not Project entity fields.

## Changelog
| Date | Change |
|---|---|
| 2026-10-09 | `description` made optional in create request (D-001 #1). |
| 2026-10-09 | `status` removed from create request; server defaults to `draft` (D-001 #6). |
| 2026-10-09 | List: `limit`, `offset` query params and `total` response field added (D-001 #8). |
| 2026-10-09 | Non-validation error body shape added; 500 body defined (D-001 last bullet). |
| 2026-10-09 | No 401/403 in F1 noted (D-001 #10). CORS origins documented. |
| 2026-10-09 | Duplicate-name and brief/shot out-of-scope notes added (D-001 #4, #9). |
| 2026-10-09 | All open questions resolved; section removed. |
