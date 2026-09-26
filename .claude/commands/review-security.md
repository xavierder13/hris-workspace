---
description: Focused security review of a repository, change, or specific concern
argument-hint: <file/diff/area/concern to review, or empty for the whole repo>
---

Perform a focused security review of `$ARGUMENTS`.

Use the `security-reviewer` agent for this — it reads the owning
repository's own conventions and constraints first (via that repository's
`CLAUDE.md`, since a legacy-pinned stack changes what a *fixable*
recommendation looks like), applies the `security-review` skill's category
list (broken access control, injection, secrets handling, auth/session/
token handling, file uploads, mass assignment, dependency/CVE awareness),
and adds `cross-repository-review`'s method for the specific case of a
permission/ownership check hidden client-side but not enforced
server-side if the concern spans both a frontend and backend repository.

If `$ARGUMENTS` is empty, review the current uncommitted diff in whichever
repository the user most recently mentioned if that's a reasonable
inference; otherwise ask what's in scope rather than guessing — a
security review of "everything" with no scope tends to produce a shallow
pass over a lot of surface area instead of a real one.

Do not modify code as part of this command unless explicitly told this
review should also apply its fixes. Never include a real secret, token,
key, or credential value in the report — reference it by name/location
only.
