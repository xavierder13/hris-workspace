---
description: Analyze a change and perform focused regression testing
argument-hint: <description of the change, or leave empty to use the current diff>
---

Analyze the change described in `$ARGUMENTS` (or, if empty, the current
uncommitted diff in the relevant repository) and perform focused regression
testing.

Use the `regression-tester` agent for this — it traces the change's real
blast radius across frontend components, backend endpoints, database
tables, services, permissions, and workflows, then produces a prioritized
checklist rather than re-testing the whole application. If it can safely
execute some checklist items itself, it will and will say so; remaining
items are handed off for `/test-feature` or `/test-workflow` to execute.

Report using the workspace's standard format, with `## Regression risk`
(the checklist) as the centerpiece.
