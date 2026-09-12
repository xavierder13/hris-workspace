---
description: Test one feature across frontend and backend, integration-style
argument-hint: <feature or module name>
---

Test the feature named in `$ARGUMENTS` across the frontend and backend
repositories in this workspace.

Use the `integration-tester` agent for this — it traces the feature through
both repositories, applies `cross-repository-review`, executes existing
automated tests plus application-level tests (browser when available,
direct API calls matching the confirmed frontend contract otherwise), and
reports defects with evidence via `test-evidence`.

If `$ARGUMENTS` is empty, ask which feature to test rather than guessing —
this command needs a specific target to be useful.

Do not modify code as part of this command; it produces a report, per the
workspace `CLAUDE.md`'s report-only default.
