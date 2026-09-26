---
description: High-level cross-repository architecture and integration review of the whole workspace
---

High-level cross-repository architecture and integration review (focus on
`$ARGUMENTS` if it names a module or concern). Run directly — not
delegated to an agent.

1. Run `repository-discovery` if the repositories aren't mapped this
   session or `docs/repository-map.md` looks stale; read each repository's
   `CLAUDE.md`.
2. Summarize each repository's role, framework, and integration surface
   (API client/routes, auth, main modules).
3. Apply `cross-repository-review` at architecture level: auth mechanism
   match, API base URL/route prefixes, whole modules with no counterpart.
   Field-by-field checks are `/cross-check-api`'s job.
4. Note only genuinely notable maintainability/regression concerns
   (`code-review` categories) — this is a survey.

Report in the workspace format, citing files/lines. For anything needing
deeper confirmation, name the command that would confirm it
(`/cross-check-api`, `/test-feature`, `/review-code`).
