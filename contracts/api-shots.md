# Shots REST/JSON API

Base collection: `/projects/{project_id}/shots`. JSON media type is used for all bodies.
Entity fields are defined in `decisions/D-003.md` and `contracts/data-model.md`.

## Create Shot
`POST /projects/{project_id}/shots`

Request body (`title` required; `description` and `sequence_order` optional):
```json
{"title":"Establishing Aerial Alleyway","description":"Wide crane shot of cyberpunk city canyon","sequence_order":1}
```
If `sequence_order` is omitted, server assigns `(max(sequence_order) or 0) + 1`.
Server defaults `status` to `draft`.

Success: `201 Created`, body:
```json
{"id":"shot_01","project_id":"prj_01","sequence_order":1,"title":"Establishing Aerial Alleyway","description":"Wide crane shot of cyberpunk city canyon","status":"draft","created_at":"2026-10-09T10:00:00Z","updated_at":"2026-10-09T10:00:00Z"}
```

## List Shots
`GET /projects/{project_id}/shots`

Returns shots ordered by `sequence_order` ascending.
Success: `200 OK`, body:
```json
{"items":[{"id":"shot_01","project_id":"prj_01","sequence_order":1,"title":"Establishing Aerial Alleyway","description":"Wide crane shot of cyberpunk city canyon","status":"draft","created_at":"2026-10-09T10:00:00Z","updated_at":"2026-10-09T10:00:00Z"}],"total":1}
```

## Get Shot
`GET /projects/{project_id}/shots/{shot_id}`

Success: `200 OK`, body matches single shot representation above.
Unknown project or shot: `404 Not Found`.

## Update Shot
`PATCH /projects/{project_id}/shots/{shot_id}`

Request body (all fields optional):
```json
{"title":"Updated title","description":"Updated description","status":"in_progress","sequence_order":2}
```
Success: `200 OK`, returns updated shot representation.

## Delete Shot
`DELETE /projects/{project_id}/shots/{shot_id}`

Success: `204 No Content`.
Unknown project or shot: `404 Not Found`.

## Status codes & Error body
Uses standard error envelope defined in `contracts/api-projects.md`:
`{"error":{"code":"...","message":"...","fields":[...]}}`
- `200`: Success (get/list/patch)
- `201`: Shot created
- `204`: Shot deleted
- `400`: Validation error
- `404`: Project or Shot not found
- `500`: Internal error
