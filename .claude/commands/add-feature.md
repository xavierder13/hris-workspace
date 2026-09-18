---
description: Scope and implement a new feature/module in the repository that owns it, following established conventions
argument-hint: <feature/module name and a short description of what it should do>
---

Add the feature described in `$ARGUMENTS` to the repositories in this
workspace.

Use the `feature-development` skill for this — it checks for an existing
per-project feature/module registry (a `docs/*.md` living document, if this
project maintains one) and the owning repository's own `CLAUDE.md`/
conventions *before* writing anything, specifically so this doesn't need to
be re-explained from scratch every time a new feature comes up. Run
`repository-discovery` first if the repositories haven't been mapped yet
this session.

This command:
1. Identifies which repository (or repositories) actually own the new
   feature's pieces — doesn't touch one that isn't in scope.
2. Matches the nearest existing precedent (same domain, same layer) for
   structure, naming, and style, and cites it.
3. Implements the smallest complete version of what `$ARGUMENTS` describes,
   inside the owning repository's own conventions.
4. Updates the project's living feature/module registry doc, if one exists,
   with the new entry in the same shape as existing ones — asks first if no
   such registry exists yet, rather than inventing a new `docs/` convention
   unasked.

This command *does* modify code — unlike this workspace's report-only test
and review commands, adding a feature is inherently a build task. The root
`CLAUDE.md` safety rules still apply in full (no unrelated-repository edits,
no destructive operations, no secrets).

If `$ARGUMENTS` is empty, ask what the feature is, which module/domain it
belongs to, and which repositories it plausibly touches, before doing
anything — don't guess at a build target.

After implementing, recommend `/review-code` and `/test-feature` (or
`/test-workflow` for a full user-facing check) as next steps — this command
builds the feature, it doesn't certify it.
