---
description: Targeted code review of a specific change, file, or area
argument-hint: <file/diff/area to review>
---

Perform a targeted code review of `$ARGUMENTS`.

Use the `code-reviewer` agent for this — it reads the owning repository's
own conventions first (via that repository's `CLAUDE.md` and any
project-specific skills), applies the `code-review` skill's categories, adds
`cross-repository-review` if the change spans both a frontend and backend
repository, and uses `regression-testing` to note (not execute) what else
might need checking if the change is non-trivial.

If `$ARGUMENTS` is empty, ask what to review rather than guessing — default
to the current uncommitted diff in whichever repository the user most
recently mentioned, if that's a reasonable inference, but confirm rather
than assuming for anything broader.

Do not modify code as part of this command unless explicitly told this
review should also apply its fixes.
