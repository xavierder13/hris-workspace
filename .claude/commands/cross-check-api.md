---
description: Compare frontend API usage against backend API implementation for a specific feature or across the whole system
argument-hint: <feature/module name, or leave empty for full system>
---

Compare frontend API usage against the backend implementation for
`$ARGUMENTS`. If it's empty, the scope is the whole system — warn that it's
slow and confirm first.

Run `repository-discovery` first if needed, then apply the full
`cross-repository-review` checklist. Every finding cites the frontend and
backend code side by side; anything not confirmed on both sides is
reported as unconfirmed, with what would confirm it.

Report in the workspace format with `## Integration findings` as the
centerpiece. Report-only.
