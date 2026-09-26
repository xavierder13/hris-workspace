---
description: Analyze a change and perform focused regression testing
argument-hint: <description of the change, or leave empty to use the current diff>
---

Analyze the change in `$ARGUMENTS` (or, if empty, the current uncommitted
diff in the relevant repository) and scope regression testing.

Delegate to the `regression-tester` agent. Report in the workspace format
with `## Regression risk` (the prioritized checklist) as the centerpiece.
Report-only.
