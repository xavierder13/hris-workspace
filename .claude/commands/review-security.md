---
description: Focused security review of a repository, change, or specific concern
argument-hint: <file/diff/area/concern to review, or empty for the whole repo>
---

Security-review `$ARGUMENTS`. If it's empty, review the current uncommitted
diff in the repository the user most recently mentioned when that's a
reasonable inference; otherwise ask for a scope (an unscoped review is
shallow).

Delegate to the `security-reviewer` agent. Don't modify code unless told to
apply fixes. Never include a real secret, token, key or credential value in
the report — name/location only.
