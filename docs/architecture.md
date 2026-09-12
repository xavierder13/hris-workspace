# Workspace architecture

## The core separation

This workspace deliberately separates two layers:

**Generic workspace intelligence** (`.claude/` at the workspace root,
`docs/`, `test-scenarios/templates/`) — reusable across any pair of
repositories, any module, any application. This is *how to work across
repositories in general*: how to discover them, how to compare their
contracts, how to test a workflow like a user would, how to review code
without assuming a framework, how to scope a regression check, how to write
up evidence.

**Project-specific intelligence** (each repository's own `CLAUDE.md` and
`.claude/`) — owned entirely by that repository. Framework rules,
architecture decisions, naming conventions, database rules, business
terminology, existing implementation decisions, and any project-specific
skills/agents/testing commands live there, not here.

The workspace-level configuration never assumes it knows a specific
repository's conventions. It reads them. See the root `CLAUDE.md`'s
"Repository boundaries" section for the exact rules this implies.

## Why this separation matters

If workspace-level skills hard-coded assumptions about a specific
repository (its module names, its status values, its permission strings),
the workspace would stop being reusable the moment either repository
changed or was swapped for a different one. The whole point of this
workspace is that it should work unchanged if:

- The frontend or backend repository is replaced with a different codebase
  entirely.
- A repository's internal conventions change over time.
- A third repository (another backend, a mobile client, a shared library)
  is added.

Anything that would break under those conditions belongs in a repository's
own `.claude/`, not here.

## Repository role detection

Repositories are placed in this workspace as plain directories (see the
root `README.md` for exactly how). Nothing here assumes a directory is a
frontend or backend based on its name — the `repository-discovery` skill
determines role from actual file signatures (framework markers, dependency
manifests, directory shapes). See that skill for the full detection
approach. This means a workspace with two backend repositories, or an
unconventional layout, still gets analyzed correctly instead of forced into
an assumed frontend/backend pair.

## How the pieces compose

```
CLAUDE.md (root)          -- the rules everything else operates under
  │
  ├── .claude/skills/      -- the reusable "how" (procedures)
  │     repository-discovery
  │     cross-repository-review
  │     user-workflow-testing
  │     code-review
  │     regression-testing
  │     test-evidence        (shared reporting structure, used by all)
  │
  ├── .claude/agents/       -- the reusable "who" (scoped executors,
  │     integration-tester    built from the skills above)
  │     user-workflow-tester
  │     code-reviewer
  │     regression-tester
  │
  └── .claude/commands/     -- the reusable "entry points"
        /review-system   -> repository-discovery + cross-repository-review
                             (system-wide, not agent-delegated — a survey)
        /test-feature    -> integration-tester agent
        /test-workflow   -> user-workflow-tester agent
        /cross-check-api -> cross-repository-review (direct, feature-scoped)
        /review-code     -> code-reviewer agent
        /regression-test -> regression-tester agent
```

`test-evidence` is intentionally used by every other skill and every agent,
rather than each one defining its own report shape — this is what makes
findings from a code review, an integration test, and a workflow test
directly comparable (same severity scale, same evidence structure).

`/review-system` is intentionally *not* wrapped in a dedicated agent. It's a
survey-level check meant to be quick and system-wide; delegating it to an
agent would add overhead without adding value. Feature-level and
change-level work (testing one feature, reviewing one change, scoping one
regression) is where delegating to a scoped agent is worth it — that's why
those commands each map to one.

## Extending this workspace

Adding a new generic capability: add a skill under `.claude/skills/`, and
either wire it into an existing agent/command or add a new one, following
the same "reusable procedure, not project-specific content" rule as
everything else here.

Adding project-specific capability: it goes inside the relevant
repository's own `.claude/`, never here — even if it would be convenient to
have it "close by." See the root `CLAUDE.md` for why.
