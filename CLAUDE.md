# hris-workspace

Guidance for Claude Code when working in this workspace.

## What this is

This is an **AI cross-repository development, integration, QA, and user-workflow
testing workspace**. It holds two or more independent application repositories
side by side (conventionally `frontend-repo/` and `backend-repo/`, though the
actual names may differ) plus a shared, reusable set of skills, agents, and
commands for working across them.

**This workspace is an orchestrator. It does not own the application code.**
Every repository inside it owns its own architecture, conventions, and rules.
The workspace's job is to understand how the repositories relate to each
other, verify that they agree with each other, exercise them the way a real
user would, and report back — not to reimplement or override what each
repository has already decided about itself.

This workspace is designed to be reusable: the same `.claude/` configuration
should work for any pair (or more) of frontend/backend repositories placed
inside it, for any module of the application, across different projects
entirely if the repositories are swapped out.

## Repository discovery — do this before anything else

Never assume which folder is the frontend and which is the backend, and never
assume a repository is correctly implemented just because it's present.
Inspect it. Use the `repository-discovery` skill for the full procedure; in
short:

- A `composer.json` + `artisan` file → almost certainly a Laravel backend.
- A `package.json` with `react`/`antd`/`vue`/`vuetify` and no server
  entrypoint → almost certainly a frontend SPA.
- A `package.json` with `express`/`nest`/`fastify`/`koa` and no build-only
  frontend tooling → likely a Node backend.
- Two backend-shaped repositories, or two frontend-shaped repositories, are
  both possible. Don't force a frontend/backend label onto something that
  doesn't fit — report what you actually find.

Do this fresh at the start of a task that touches unfamiliar repositories, and
re-check if a repository's structure looks different than the last time it
was mapped (someone may have restructured it). Persist what you find in
`docs/repository-map.md` so later sessions don't have to re-derive it —
but treat that file as a cache, not a source of truth: verify against the
actual repository before relying on a stale claim in it.

## Repository boundaries — respect them completely

Each repository inside this workspace is authoritative over itself. When
working inside a specific repository:

1. **Read that repository's own `CLAUDE.md` first**, if it has one. It defines
   framework rules, architecture, naming conventions, database rules,
   business terminology, and prior implementation decisions specific to that
   codebase. It always wins over this workspace's generic defaults for
   anything it covers.
2. **Read that repository's `.claude/skills/` and `.claude/agents/`**, if
   present, and use them for work inside that repository. A project-specific
   skill for (say) its approval-workflow module knows more than any generic
   workspace skill ever could.
3. **Never copy a repository's project-specific rules into this workspace's
   generic configuration**, and never copy this workspace's generic
   configuration into a repository. Generic cross-repo behavior lives here;
   project-specific behavior lives in the repository. Keep the two separate
   on purpose — that separation is what makes this workspace reusable for a
   different pair of repositories later.
4. **Never modify a repository's own `CLAUDE.md` or `.claude/` files** unless
   the user explicitly asks for that repository to be changed. Reading is
   always fine; writing is a repository-specific change like any other.
5. If a repository's own instructions conflict with this workspace's generic
   guidance, **follow the repository's instructions while working inside that
   repository** — unless doing so would conflict with an explicit instruction
   from the user in the current conversation, or with a safety rule below
   (destructive operations, secrets, database safety). Those always win.

## How the pieces fit together

- **Skills** (`.claude/skills/`) are reusable procedures: how to discover
  repositories, how to compare a frontend/backend contract, how to test a
  business workflow, how to review code, how to scope a regression check, how
  to write up evidence. Load the one that matches the task.
- **Agents** (`.claude/agents/`) are the reusable executors: an
  integration-tester, a user-workflow-tester, a code-reviewer, a
  regression-tester. Each one is built from the skills above, scoped to a
  specific job, and reports in the standard format below. Use an agent when a
  task is substantial enough to be worth delegating (a full feature test, a
  full review) — for a quick, narrow question, just use the relevant skill
  directly instead of spawning an agent.
- **Commands** (`.claude/commands/`) are short, named entry points
  (`/review-system`, `/test-feature`, etc.) that wire a specific request to
  the right skill or agent so the user doesn't have to re-explain the
  workflow every time.
- **`test-scenarios/`** holds reusable scenario templates and the actual
  scenarios written for this project's modules. **`test-results/`** holds the
  evidence produced when those scenarios are executed. **`docs/`** holds the
  workspace's own living documentation (architecture, testing strategy, the
  repository map).

## How frontend/backend relationships should be analyzed

Use the `cross-repository-review` skill for the full procedure. The
non-negotiable rule underneath it: **a mismatch is only real if the code
proves it.** Read the actual frontend service/API-client code and the actual
backend route/controller/validation code side by side. Do not report "the
frontend probably expects X" or "the backend likely validates Y" — open the
files and confirm. If something can't be confirmed from the code or from an
actual executed test, say so explicitly rather than presenting it as fact.

## How testing should be performed

Use the `user-workflow-testing` skill for the philosophy and the
`test-evidence` skill for how to capture results. In short: test the
business process a real user would follow, not just whether a function
returns 200. Prefer executing the real application (via browser automation
when it's available in the environment, or via direct API calls when a
browser isn't available) over reasoning about what "should" happen. A
successful HTTP response is not proof that the user-visible workflow works —
verify what the user would actually see, and where useful, verify the
underlying data too (database state, computed fields), using the least
invasive method available.

## How defects should be reported

Every finding goes through the `test-evidence` skill's structure and
severity scale (BLOCKER/CRITICAL/HIGH/MEDIUM/LOW/INFO). Do not inflate
severity to make a finding sound more urgent than the evidence supports, and
do not soften a finding that genuinely blocks the workflow. Every defect
needs enough detail (repository, file, endpoint, payload, actual vs.
expected, reproduction steps) that a developer could act on it without
re-deriving what you already found.

## When code changes are allowed vs. report-only

Default to **report-only** for anything framed as testing, review, or
investigation — "test this," "review this," "check the integration,"
"find defects." Produce findings, not fixes, unless the user's request
already includes an instruction to fix (e.g., "test this and fix what you
find," "apply the recommended fix"). When it's ambiguous, report first and
ask before fixing — the cost of asking is low, and unrequested changes across
multiple repositories are expensive to undo cleanly.

When a fix is explicitly requested, make the smallest change that fixes the
confirmed defect, inside the repository that actually owns the problem,
following *that repository's* conventions — not a workspace-generic
convention. See the `code-review` skill for what "small, targeted,
project-consistent" means in practice.

## Avoiding unrelated changes

Never make an edit inside `frontend-repo/` while investigating an issue that
turns out to live in `backend-repo/`, or vice versa — identify the owning
repository first, then act only there. Never touch a repository that isn't
part of the current task just because it happens to be present in the
workspace.

## Safety rules (apply everywhere in this workspace, no exceptions)

**Destructive operations.** Don't run anything that drops a database,
truncates a table, deletes production-looking data, force-pushes, resets a
branch, deletes a branch, or rewrites history, unless the user has explicitly
asked for that specific action. If a task seems to require one of these,
stop and confirm first rather than deciding it's implied.

**Database safety.** When verifying data touched by a test, prefer the least
invasive method — a scoped `SELECT`, a read through the application's own
API, a query against a dedicated test record. Never run a migration,
seed, or destructive query against a database without explicit instruction,
and never assume a database is a disposable test instance just because it's
reachable from this workspace. Clean up test records you create when it's
easy and safe to do so, and say clearly what you're leaving behind when it
isn't.

**Git safety.** Preserve every repository's working tree as you found it
while investigating. Never discard uncommitted changes, assume they're
yours, or clean them up without asking. When you do make a change the user
asked for, make it clear which repository and which files changed, and never
force-push, delete a branch, or rewrite history without an explicit request.

**Secrets.** Never commit `.env` files, credentials, tokens, database dumps,
private keys, or any other secret — in this workspace's own repo or in
either application repo. If a `.gitignore` in a repository you're working in
doesn't already exclude these, don't add them to git regardless.

## Avoid inventing contracts

If you cannot find where a frontend call is actually handled server-side, or
cannot find what fields a backend response actually contains, say exactly
that — "I could not locate a handler for this endpoint" or "I could not
confirm this field is returned" — rather than describing a plausible-sounding
contract as if it were confirmed. Confirmed-from-code and assumed-from-naming
are different confidence levels; keep them visibly different in every report.

## Avoiding redundant questions

Before asking the user something, check whether the repositories themselves
already answer it: their `CLAUDE.md`, their `.claude/skills/` and
`.claude/agents/`, their README, and the actual implementation. Only ask when
a genuine decision can't be resolved from what's available — a business
judgment call, an ambiguous product intent, or a choice between two equally
valid technical approaches with no repository precedent to follow.

## Reporting format

Use this shape for cross-repository testing and review output (agents and
commands below are built to produce it):

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

See the `test-evidence` skill for what belongs in each defect entry, and the
individual command files in `.claude/commands/` for how each command maps a
request onto this format.
