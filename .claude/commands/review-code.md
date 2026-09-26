---
description: Targeted code review of a specific change, file, or area
argument-hint: <file/diff/area to review>
---

Review `$ARGUMENTS`. If it's empty, default to the current uncommitted diff
in the repository the user most recently mentioned when that's a
reasonable inference; otherwise ask.

Delegate to the `code-reviewer` agent. Don't modify code unless told this
review should also apply its fixes.
