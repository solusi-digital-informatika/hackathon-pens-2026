# Brief REST/JSON API

Base collection: `/projects/{project_id}/brief`. JSON media type is used for all bodies.
Entity fields are defined in `decisions/D-005.md` and `contracts/data-model.md`.

## 1. Ingest Brief
`POST /projects/{project_id}/brief/ingest`

Request body:
```json
{"source_name":"CyberPulse Creative Brief v1","content":"Full creative brief text..."}
```
Success: `201 Created`, body contains created SourceDocument metadata.

## 2. Extract Structured Brief
`POST /projects/{project_id}/brief/extract`

Extracts structured requirements from the active source document.
Success: `200 OK`, body contains structured extraction:
```json
{
  "id": "brief_01",
  "project_id": "prj_01",
  "objective": "Courier transporting encrypted data core",
  "visual_style": "Realistic Cinematic Cyberpunk",
  "lighting_mood": "Moody cool blue lighting, wet asphalt",
  "characters": [{"name": "Aria", "details": "athletic build, dark undercut, cybernetic left eye"}],
  "key_props": [{"name": "Data Core", "details": "hexagonal encrypted slate"}],
  "constraints": ["No confidential footage", "Cool blue color palette"],
  "unresolved_questions": ["Is rain present in all shots?"],
  "review_status": "pending_review"
}
```

## 3. Get Active Brief
`GET /projects/{project_id}/brief`

Success: `200 OK`, returns source document and current structured brief representation.
When no brief exists: `404 Not Found`.

## 4. Update / Approve Structured Brief
`PUT /projects/{project_id}/brief`

Allows human editor to adjust extracted fields or mark review_status as `approved`.
Success: `200 OK`, returns updated ProjectBrief representation.

## Error Codes
Standard error envelope: `400` validation error, `404` project not found, `500` internal error.
