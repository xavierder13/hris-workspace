---
description: Compare frontend API usage against backend API implementation for a specific feature or across the whole system
argument-hint: <feature/module name, or leave empty for full system>
---

Compare frontend API usage against backend API implementation for
`$ARGUMENTS` (or, if empty, across the whole system — warn that this will
take longer and confirm before doing a full sweep on a large codebase).

Run `repository-discovery` first if the repositories haven't been mapped
yet this session. Then apply the `cross-repository-review` skill's full
checklist: existence, HTTP method + URL, request fields (both directions),
field naming, shape, types/nullability, response contract, create/update
asymmetry, validation, auth/authorization enforcement, pagination/filtering/
sorting, and error response shapes.

Every finding must cite the actual frontend code and the actual backend code
side by side — do not report a mismatch you haven't confirmed on both sides.
Mark anything you couldn't fully confirm as unconfirmed rather than as a
defect, and say what would confirm it.

Report using the workspace's standard format, with `## Integration findings`
as the centerpiece and each item in `## Defects` carrying full evidence per
`test-evidence`.
