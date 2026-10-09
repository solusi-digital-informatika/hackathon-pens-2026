# Projects REST/JSON API

Base collection: `/projects`. JSON media type is used for request and response bodies.
Project fields are defined in `data-model.md`.

## Create project
`POST /projects`

Request (all fields required):
```json
{"name":"Demo film","description":"Six-shot direction-change demo","status":"draft"}
```
Success: `201 Created`, `Location: /projects/prj_01`, body:
```json
{"id":"prj_01","name":"Demo film","description":"Six-shot direction-change demo","status":"draft","created_at":"2026-10-09T09:00:00Z","updated_at":"2026-10-09T09:00:00Z"}
```
Invalid JSON, missing fields, wrong types, or invalid field values: `400 Bad Request`.

## List projects
`GET /projects`

Success: `200 OK`. Each item has the complete Project representation:
```json
{"items":[{"id":"prj_01","name":"Demo film","description":"Six-shot direction-change demo","status":"draft","created_at":"2026-10-09T09:00:00Z","updated_at":"2026-10-09T09:00:00Z"}]}
```

## Get (open) project
`GET /projects/{project_id}`

Success: `200 OK`, with the complete Project representation shown above.
Unknown or inaccessible ID: `404 Not Found`; this avoids exposing another project's
data and supports FR-01 project isolation.

The opened workspace separately obtains project-scoped active-brief and shot-list data.
Those resource contracts are outside this document; every such request must be scoped
by the stable `project_id` and must never return another project's records (FR-01).

## Validation error
All `400` responses use:
```json
{"error":{"code":"validation_error","message":"Project input is invalid","fields":[{"field":"name","message":"Required"}]}}
```
`fields` may contain multiple failures. Each `field` names the rejected request field;
`message` is safe for users and does not expose implementation details.

## Status codes
| Code | Meaning |
|---:|---|
| `200` | List/get succeeded |
| `201` | Project created |
| `400` | Malformed or invalid request |
| `404` | Project is unknown or inaccessible |
| `500` | Unexpected server failure; no internal details are returned |

## Assumptions
- The service generates `id`, `created_at`, and `updated_at`.
- Create is atomic: failure leaves no partially created Project.
- Error envelope fields are transport metadata, not Project entity fields.

## Open questions
- Valid `status` values and field length/blank-value rules are not specified.
- Authentication, authorization, and whether `401`/`403` are exposed are unspecified.
- List ordering, filtering, pagination, and empty-list limits are unspecified.
- The active-brief and shot-list endpoint shapes and empty states are unspecified.
- Conflict and duplicate-name behavior are unspecified.
