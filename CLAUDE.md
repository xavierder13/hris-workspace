# hris-workspace

An **AI cross-repository development, integration, QA and user-workflow
testing workspace**. It holds two or more independent application
repositories side by side plus a reusable set of skills, agents and
commands for working across them. **It orchestrates; it does not own the
application code.** Its job is to understand how the repositories relate,
verify they agree with each other, exercise them the way a real user would,
and report back. The `.claude/` here must stay generic so it works for any
repositories placed in this workspace.

## Repository discovery — before anything else

Never assume which folder is the frontend or backend, or that a repository
is correctly implemented. Inspect it with the `repository-discovery` skill
(signals: `composer.json` + `artisan` → Laravel; `package.json` with
react/antd/vue/vuetify and no server entrypoint → SPA; express/nest/
fastify/koa → Node backend; two of the same kind is possible — report what
you find). `docs/repository-map.md` is a cache of the last discovery:
verify a claim against the repository before relying on it.

## Repository boundaries

Each repository is authoritative over itself.
1. Read its own `CLAUDE.md` first; it wins over this file for anything it
   covers.
2. Use its own `.claude/skills/` and `.claude/agents/` for work inside it
   (they aren't listed as skills when Claude runs from this workspace —
   read their `SKILL.md` files directly).
3. Never copy project-specific rules into this workspace's config, or this
   workspace's generic config into a repository.
4. Never modify a repository's `CLAUDE.md` or `.claude/` unless the user
   asks for that repository to be changed.
5. If a repository's instructions conflict with this file, follow the
   repository — unless that conflicts with an explicit user instruction or
   the safety rules below, which always win.

## Skills, agents, commands

Skills (`.claude/skills/`) are the procedures; agents (`.claude/agents/`)
are scoped executors built from them; commands (`.claude/commands/`) wire a
request to the right skill or agent. Load the skill that matches the task;
use an agent only for substantial delegated work (a full feature test, a
full review). Agents have no Skill tool — they read
`.claude/skills/<name>/SKILL.md` directly. `test-scenarios/` holds scenario
templates, `test-results/` holds evidence from executed runs, `docs/` holds
this workspace's own documentation.

## Evidence rules

- **A mismatch or defect is only real if the code or an executed test
  proves it.** Read the actual frontend call and the actual backend
  route/controller/validation side by side (`cross-repository-review`).
- Keep **confirmed-from-code** and **assumed-from-naming** visibly
  different. If you can't find a handler or can't confirm a field is
  returned, say exactly that — never describe a plausible contract as fact.
- Test the business process a real user follows, not just a 200
  (`user-workflow-testing`). Prefer executing the real app (browser when
  available, otherwise direct API calls matching the real frontend
  payload). Verify what the user would see, and underlying data where
  useful, using the least invasive method.
- Record findings with `test-evidence`'s structure and severity scale
  (BLOCKER/CRITICAL/HIGH/MEDIUM/LOW/INFO) — don't inflate or soften, and
  give enough detail (repository, file, endpoint, payload, actual vs.
  expected, repro) that a developer can act without re-deriving it.

## Report-only vs. fixes vs. new features

- **Testing, review, investigation** ("test this", "review this", "check
  the integration") → report-only. Don't fix unless the request says to;
  when ambiguous, report first and ask.
- **Requested fix** → the smallest change, in the repository that owns the
  problem, following that repository's conventions (`code-review`).
- **New feature** → `/add-feature` / `feature-development`: implement only
  in the owning repository(ies), following their conventions, then hand
  off to `/review-code` / `/test-feature` rather than self-certifying.
- Identify the owning repository first and act only there. Never touch a
  repository that isn't part of the current task.

## Safety rules (everywhere, no exceptions)

**Destructive operations.** Don't drop a database, truncate a table, delete
production-looking data, force-push, reset or delete a branch, or rewrite
history unless the user explicitly asked for that specific action. If a
task seems to require one, stop and confirm.

**Database safety.** Prefer the least invasive verification (a scoped
`SELECT`, the application's own API, a dedicated test record). Never run a
migration, seed or destructive query without explicit instruction, and
never assume a reachable database is disposable. Clean up test records when
easy and safe; say clearly what you leave behind.

**Git safety.** Preserve every repository's working tree as you found it.
Never discard uncommitted changes, assume they're yours, or clean them up
without asking. Say which repository and files you changed. Never
force-push, delete a branch, or rewrite history without an explicit
request.

**Secrets.** Never commit `.env` files, credentials, tokens, database
dumps, private keys or other secrets — here or in any application repo —
even if a `.gitignore` doesn't exclude them.

## Before asking the user

Check whether the repositories already answer it (their `CLAUDE.md`,
`.claude/`, README, and code). Ask only for genuine decisions: business
judgment, ambiguous product intent, or a choice between equally valid
approaches with no precedent.

## Reporting format

Cross-repository testing and review output uses:

```
## Summary
## Feature
## Repositories
## Workflow
## Integration findings
## Test results
## Defects
## Regression risk
## Recommendation
```
