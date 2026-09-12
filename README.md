# hris-workspace

A reusable **AI cross-repository development, integration-testing, QA,
code-review, and user-workflow-testing workspace** for Claude Code.

This is not an application. It doesn't contain frontend or backend code of
its own — it contains the reusable skills, agents, and commands that let
Claude Code work *across* two or more separate application repositories at
once: understanding how they integrate, testing them the way a real user
would, cross-checking their API contracts, and reviewing code quality —
while fully respecting each repository's own conventions.

See [`CLAUDE.md`](./CLAUDE.md) for the full behavioral contract this
workspace establishes for Claude Code. This README is the human-facing
quick start.

## Structure

```
hris-workspace/
├── CLAUDE.md               # the workspace's behavioral contract for Claude Code
├── README.md               # this file
├── .gitignore
├── .claude/
│   ├── agents/              # integration-tester, user-workflow-tester,
│   │                        # code-reviewer, regression-tester
│   ├── skills/              # repository-discovery, cross-repository-review,
│   │                        # user-workflow-testing, code-review,
│   │                        # regression-testing, test-evidence
│   └── commands/            # /review-system /test-feature /test-workflow
│                            # /cross-check-api /review-code /regression-test
├── test-scenarios/
│   ├── templates/           # generic scenario template — reusable for any module
│   └── scenarios/           # actual scenarios written for this project
├── test-results/            # evidence from executed tests (gitignored contents)
├── docs/
│   ├── architecture.md      # how the pieces fit together and why
│   ├── testing-strategy.md  # the testing philosophy in depth
│   └── repository-map.md    # cache of what repository-discovery found
├── frontend-repo/           # <- put your frontend application here
└── backend-repo/            # <- put your backend application here
```

`frontend-repo/` and `backend-repo/` are placeholders (just a `.gitkeep`
each). Their contents are gitignored by this workspace on purpose — each
one is its own separate git repository with its own history and remote.
This workspace's own git repo only ever tracks the workspace shell.

## Placing your repositories

You have two reasonable options; pick whichever fits how you work:

**Clone directly into the placeholder folders:**

```bash
git clone git@github.com:<you>/<your-frontend-repo>.git frontend-repo
git clone git@github.com:<you>/<your-backend-repo>.git backend-repo
```

**Or use a symlink** if you already have the repositories checked out
elsewhere on this machine and don't want a second copy:

```bash
ln -s /path/to/existing/frontend-checkout frontend-repo
ln -s /path/to/existing/backend-checkout backend-repo
```

Either way, the repository names don't matter — nothing in this
workspace's configuration hard-codes `frontend-repo`/`backend-repo` as
literal names it depends on; it detects each repository's actual role from
its contents (see `.claude/skills/repository-discovery/`). You can rename
the folders, add a third repository alongside them, or replace either one
entirely with a different codebase later, and the workspace configuration
keeps working unchanged.

## Recommended first workflow after cloning

1. Place your two (or more) repositories as above.
2. Open Claude Code in this workspace root.
3. Run `/review-system` first. This maps the repositories (detecting each
   one's actual role, reading its `CLAUDE.md` and any project-specific
   `.claude/` configuration), writes a cache of that into
   `docs/repository-map.md`, and gives you a high-level integration/
   architecture survey — the fastest way to get Claude Code oriented before
   asking it to test or review anything specific.
4. From there, use whichever command matches what you actually need:
   - `/cross-check-api <feature>` — verify a specific feature's frontend↔
     backend contract.
   - `/test-feature <feature>` — full integration test of one feature.
   - `/test-workflow <workflow description>` — act as an end user and run
     a complete business process.
   - `/review-code <target>` — targeted code review.
   - `/regression-test <change>` — scope what a change could affect.

## Ground rules (see `CLAUDE.md` for the full version)

- This workspace **reports findings by default**; it doesn't fix things
  unless you ask it to.
- It **never copies** a repository's project-specific rules into this
  workspace's generic config, or vice versa — read the architecture note in
  `docs/architecture.md` if you're curious why that separation matters.
- It **never** runs destructive database/git operations without you
  explicitly asking for that specific action.
- It reports mismatches and defects only when the actual code (or an actual
  executed test) proves them — not from assumptions.
