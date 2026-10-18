# Hub Conventions

Environment: Windows + PowerShell. Use PowerShell-compatible commands.
Do not chain dependent commands with `;` (it continues after a failure). Run them one at a time and check each result.

## Folders

- `board/board.md`: task board. One row per task: ID, title, owner, size, status (Todo/Doing/Review/Testing/Done), repo, commit hash. Leader owns it. Each agent updates its own row.
- `contracts/`: `design-direction.md`, `design-tokens.md`, `ui-<feature>.md` (Designer); `api-<feature>.md` (Backend-AI).
- `decisions/`: `NNN-title.md`, Large decisions only (Leader).
- `inbox/<agent>/`: messages TO that agent. File name `<task-id>-<slug>.md`.
  - Leader → agent: delegation, in `inbox/<agent>/`.
  - Agent → Leader: report or question, in `inbox/leader/`.
  - Tester → responsible agent: bug report, in `inbox/<agent>/` (plus a short summary to `inbox/leader/`).
  - When a message is handled, move it to `inbox/<agent>/done/`. Read only unhandled files.
- `project-map/`: this file and `git-workflow.md`.

## Repos

| Name     | Remote                                                                | Local path                               | Writes                          |
| -------- | --------------------------------------------------------------------- | ---------------------------------------- | ------------------------------- |
| backend  | https://github.com/solusi-digital-informatika/teman-umkm-bakcend.git  | C:/Project/ai-office/office-hub/backend  | backend-ai; tester (tests only) |
| frontend | https://github.com/solusi-digital-informatika/teman-umkm-frontend.git | C:/Project/ai-office/office-hub/frontend | frontend; tester (tests only)   |

If a Local path is still `(fill in)`, STOP and ask Leader.
Hub files are internal. Never commit anything from `office-hub/` into these repos.

## Stack

Backend: Go + Gin + GORM + PostgreSQL. Frontend: React + TypeScript + Vite.
AI: hosted LLM API called from Go. Custom ML = separate Python service (Large task).

## Concurrency

One agent writes to a repo at a time. Parallel work is allowed only across different repos or between a repo and the hub.
Tester starts a task only after the coder has pushed, and tests the commit hash from the coder's report.
