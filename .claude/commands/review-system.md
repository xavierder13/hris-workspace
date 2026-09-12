---
description: High-level cross-repository architecture and integration review of the whole workspace
---

Perform a high-level cross-repository architecture and integration review.

1. Run `repository-discovery` if the repositories haven't been mapped yet
   this session, or if the map in `docs/repository-map.md` looks stale.
   Read each repository's own `CLAUDE.md` first.
2. Summarize each repository's role, framework, and the areas of it most
   relevant to how they integrate (API client / routes, auth, main modules).
3. Apply `cross-repository-review` at the architecture level — not every
   endpoint field by field (that's `/cross-check-api` for a specific
   feature), but the overall shape: does the frontend's auth mechanism match
   the backend's? Do the API base URL and route prefixes line up? Are there
   whole modules on one side with no counterpart on the other?
4. Note any high-level regression or maintainability concerns you notice
   along the way (via `code-review`'s categories) — this command is a
   survey, not a deep code review of every file, so keep this section to
   genuinely notable findings rather than an exhaustive pass.
5. If `$ARGUMENTS` names a specific module or concern, focus the review
   there instead of the whole system.

Report using the workspace's standard format from the root `CLAUDE.md`.
Keep findings evidence-based — cite the actual files/lines that support
each finding. If something can't be confirmed without deeper investigation,
say so and suggest which command (`/cross-check-api`, `/test-feature`,
`/review-code`) would confirm it.
