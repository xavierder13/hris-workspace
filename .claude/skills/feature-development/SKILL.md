---
name: feature-development
description: Scope and implement a new feature or module in whichever repository actually owns it, by reusing the conventions and precedent already established by similar features — instead of re-deriving context or inventing a new pattern each time. Use whenever asked to add, build, or scaffold a new feature/module across the repositories in this workspace.
---

# Feature development

This is the one skill in this workspace that *builds* rather than reports.
Everything else here defaults to report-only; adding a feature is
inherently a build task. The goal of this skill specifically is to stop
each new feature from needing the same context re-explained by hand — the
conventions for "how we build things here" should already be sitting in the
repositories themselves, not in the user's head.

## Orient before writing any code

1. Run `repository-discovery` if the repositories haven't been mapped yet
   this session, or if a repository's structure looks different than last
   time.
2. Check whether the project maintains a living feature/module registry doc
   under `docs/` (a project may keep one — e.g. an index of existing
   modules, what they cover, and what conventions a new one should follow).
   If one exists, read it fully before anything else. It exists specifically
   so a new feature can reuse existing patterns/endpoints instead of
   duplicating them. Treat it like `repository-map.md`: a cache, not ground
   truth — re-verify anything load-bearing against the actual code if it
   looks stale.
3. If no such registry exists, find the nearest existing precedent yourself:
   the most similar feature/module already implemented in the owning
   repository. Read its actual files (controller/service/routes on the
   backend, page/service/store/hook on the frontend, or whatever that
   repository's own shape is) rather than assuming a generic structure.
4. Read the owning repository's own `CLAUDE.md` and any project-specific
   `.claude/skills/` relevant to the feature's domain. Its conventions
   always win over a workspace-generic idea of "clean" structure.

## Scope the change

Identify which repository (or repositories) actually need code for this
feature — a pure backend report needs no frontend change; a full CRUD
module needs both. Per the root `CLAUDE.md`'s "Avoiding unrelated changes,"
never touch a repository that isn't actually in scope for this feature.

## Build to the precedent, not to taste

Match the structure, naming, and style of the nearest existing precedent
found above — folder layout, thin-controller-plus-service split (or
whatever the pattern actually is there), validation style, permission
naming, state-management pattern. Cite the precedent you're matching so the
choice is traceable, not just asserted.

Build the smallest complete version of what was actually asked. Don't add
speculative extensibility, extra endpoints, extra fields, or extra UI the
request didn't call for — the same restraint the root `CLAUDE.md` asks for
everywhere else in this workspace applies here too.

If the request is ambiguous about which module/domain it belongs to, or
about which repositories it should touch, ask before writing code — a build
task needs a concrete target even more than a review does, and guessing
wrong here is more expensive to undo than guessing wrong on a report.

## After implementing

Update the project's living feature/module registry doc, if one exists,
by editing the affected row(s) in place — same shape/tier as existing
entries, current state only, no dated narrative (that goes in commit
messages). Update the owning repository's module skill the same way, per
that repository's own `feature-development` rule. If no such registry exists yet, ask the user before creating one;
don't silently invent a new `docs/` convention on a project that doesn't
already have it.

This skill builds the feature; it does not certify it. Point whoever's
reading the result at `code-review` (or the `/review-code` command) and at
`user-workflow-testing`/`cross-repository-review` (or `/test-feature` /
`/test-workflow`) as the next steps, rather than treating "it compiles and
runs" as proof the feature actually works.

## Safety

The root `CLAUDE.md` safety rules apply unchanged: no destructive database
or git operations without an explicit ask, no secrets committed, and no
discarding uncommitted work found in a repository while you're in there
building something else.
