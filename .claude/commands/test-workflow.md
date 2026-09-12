---
description: Act as an end user and execute a complete business workflow
argument-hint: <workflow description, e.g. "create and submit a manpower request">
---

Act as an end user and execute the complete business workflow described in
`$ARGUMENTS`.

Use the `user-workflow-tester` agent for this — it follows the application's
actual real flow (not an idealized version), prioritizes business-workflow
correctness over implementation details, prefers browser automation when
available, and distinguishes user-visible failures from technical
implementation issues in its report via `test-evidence`.

If `$ARGUMENTS` is empty, ask which workflow to run rather than guessing.

Do not modify code as part of this command; it produces a report, per the
workspace `CLAUDE.md`'s report-only default.
