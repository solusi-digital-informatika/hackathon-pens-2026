# Git Workflow (PowerShell)

## Start of task (in the repo's local path)

```powershell
git status -sb
git config user.name
git config user.email
git remote get-url origin
```

- Empty name or email: STOP, report a Manual Task.
- Remote missing: `git remote add origin <URL from hub.md>`. Remote different: STOP and report.
- Clean working tree: `git pull --rebase`. Dirty tree that isn't yours: STOP and report.

## Commit

```powershell
git add <specific files>
git diff --staged
git commit -m "feat(products): add paginated product list" -m "Optional body explaining why."
```

Use multiple `-m` flags for multi-line messages. Avoid `git add .` unless `git status` was reviewed.

## Push (after the task's last commit)

```powershell
git push
git status -sb
git log -1 --format="%h %s"
```

First push of a new branch: `git push -u origin <branch>`. Success means `status` shows no "ahead". Report the hash(es) and repo URL.

## Push rejected

```powershell
git pull --rebase
```

Check the result. Only if it succeeded cleanly, run `git push` once more. Conflict or second failure: STOP and report to Leader. Never force-push.

## New folder that isn't a repo yet

```powershell
git init -b main
git remote add origin <URL from hub.md>
```

Before the first commit, create `.gitignore` and `.gitattributes` (`* text=auto eol=lf`, avoids Windows line-ending noise and gofmt diffs).

- Backend ignore: `.env`, `*.exe`, `/bin`, `/tmp`, `coverage.out`.
- Frontend ignore: `node_modules`, `dist`, `.env.local`, `.env.*.local`, `coverage`.

## Secrets

Committed but not pushed: remove it and amend or reset your own local commit, then confirm with the diff. Already pushed: treat it as leaked. Report immediately and add a Manual Task for the owner to rotate it. Rewriting remote history is the owner's decision.

## Commit types

feat, fix, refactor, perf, test, docs, style, build, ci, chore, revert.
Scope = module name (auth, products, orders...). Breaking change: `feat(auth)!: ...` plus a `BREAKING CHANGE:` footer.
