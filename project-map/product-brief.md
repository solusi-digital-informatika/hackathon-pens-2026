\# Product Brief — AI Production Workflow Assistant



Short version of `project-map/PRD.md` (v1.0, draft for hackathon MVP). Agents read this first.

If this brief and the PRD differ, the PRD wins. Report the difference to the Leader.



\## One line



A change-resilient workflow tool for small AI-assisted creative teams. It turns a brief into a structured shot plan. When creative direction changes, it shows which shots are affected and whether to retain, adapt, regenerate, or review them, with human approval and full version history.



\## Core problem (PRD §2)



Direction changes late, and the team cannot quickly tell which shots, prompts, references, and deliverables are affected. Result: rework, inconsistent output, schedule risk.



\## Principles (PRD §3)



Shared project memory · traceability · adaptation over regeneration · human-in-the-loop · evidence-aware AI · resilience over isolated features · public-data-only demo



\## Scope



\*\*P0 (must)\*\*

\- FR-01 project workspace

\- FR-02 brief and reference ingestion

\- FR-03 AI brief analysis (editable by human)

\- FR-04 shot list / storyboard planning

\- FR-05 prompt and asset tracking per shot

\- FR-06 direction versioning

\- FR-07 direction comparison

\- FR-08 change impact analysis

\- FR-09 human review and approval

\- FR-10 version history and traceability

\- FR-11 updated storyboard



\*\*P1 (should):\*\* FR-12 decision log · FR-13 quality review checklist

\*\*P2 (could):\*\* FR-14 deadline and rework overview



\*\*Non-goals (PRD §4.2):\*\* replacing creative decisions; full editor, compositing, or asset management; production multi-tenant platform; confidential footage; custom video model.



\## Rules that must hold in the implementation



\- A new version never overwrites an old one (FR-06, NFR-02).

\- LLM output is validated against a schema before it is saved. The LLM is not the source of truth (PRD §10).

\- Shot action is one of: `retain`, `adapt`, `regenerate`, `needs\_review`. Every recommendation has a reason (FR-08).

\- Missing dependency data lowers confidence and gives `needs\_review`. It never means "unaffected" (FR-08, PRD §12.3).

\- AI recommendation and human approval are stored as separate facts (FR-09).

\- Active direction version and each shot's current version are always visible (NFR-07).

\- API keys stay server-side (NFR-05).

\- Do not imply files are recoverable if only metadata or links were stored (FR-10).



\## Key entities (full schema: PRD §11)



Project · SourceDocument · Requirement · DirectionVersion · Shot · ShotVersion · Asset · Dependency · DecisionLog.

Stable IDs are required. A shot ID does not change between versions.



\## Milestones (PRD §13)



\- \*\*M1\*\* Structured project data: schema, Project and Shot CRUD, sample project with 4 to 6 shots.

\- \*\*M2\*\* Brief understanding: ingestion, structured extraction, human review and edit.

\- \*\*M3\*\* Versioned plan: direction and shot versions, prompts, asset metadata, shot board, version history.

\- \*\*M4\*\* Change analysis: direction comparison, requirement-to-shot dependencies, recommendations.

\- \*\*M5\*\* End-to-end demo: approval controls, updated storyboard, full scenario test, documented limitations.



\## Acceptance scenario (PRD §16)



Six-shot project. Original direction: realistic cinematic, cool blue lighting (Version 1). Proposed Version 2: stylized animation, warmer lighting, main character identity must stay consistent. The system must compare the versions, list affected shots with explanations and uncertainty, let a human override, create new shot versions without deleting old ones, and show the updated storyboard with outstanding reviews.



\## Metrics (PRD §14)



Time to understand brief · change impact identification · recommendation quality · asset reuse rate · traceability · direction compliance · workflow completion time.

No percentage may be reported before it is measured. Report setup, sample size, and limitations.



\## Decisions (PRD §17) — Confirmed & Automated

| # | Question | Decision (CONFIRMED) |
|---|---|---|
| 1 | Upload scope | Pasted text first; PDF/DOCX if time permits |
| 2 | LLM provider for the app | Configured router/provider endpoint with standard model alias |
| 3 | Asset storage | Local files plus URI metadata (D-001) |
| 4 | Image/video generation | Prepared public assets |
| 5 | Users | Single user (D-001 #10) |
| 6 | Interface language | English & Indonesian friendly UI |
| 7 | Evaluation scenario and ground-truth affected shots | Approved per contracts/test-scenario.md (T-002) |
| 8 | Stack | React + Vite + TypeScript (frontend) + FastAPI (backend) + PostgreSQL 17 (D-001) |

\## Repos (Confirmed for M1 Implementation)

| Repo | Path | Owner profile | Run command | Test command |
|---|---|---|---|---|
| backend | C:/project/ai-office/backend | backend-ai | `.venv/Scripts/python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000` | `.venv/Scripts/python.exe -m pytest -q` |
| frontend | C:/project/ai-office/frontend | frontend | `npm run dev -- --host 127.0.0.1 --port 5173` | `npm test` |

