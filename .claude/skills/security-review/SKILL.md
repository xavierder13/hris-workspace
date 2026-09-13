---
name: security-review
description: Perform a focused security review of a repository or a specific change — broken access control, injection, secrets handling, authentication/session/token weaknesses, unsafe file uploads, mass assignment, and known-vulnerable dependencies — grounded in what the code actually does, not a generic checklist recited without evidence. Use whenever asked to check for vulnerabilities, do a security review, or assess whether a change introduces a security risk, in any repository inside this workspace.
---

# Security review

The same rule `cross-repository-review` uses applies here: **a vulnerability
is only real if the code proves it.** Point at the actual reachable line
that has the flaw, not a generic "this class of framework has had CVEs
before." A theoretical weakness with no reachable path in this app is worth
noting as INFO, not reported as if it were exploitable today.

## Before starting

Read the owning repository's own `CLAUDE.md` first. It changes what a
*fixable* recommendation looks like — a repository pinned to a legacy stack
on purpose (e.g. `vueportal`: "do not upgrade Laravel, PHP, or major
dependencies") turns "upgrade the vulnerable package" from a real fix into a
non-starter; the useful finding there is the narrowest mitigation that
doesn't require the upgrade, stated explicitly, including "no such
mitigation exists without violating that constraint" if that's the truth.

Check for already-surfaced signal before re-discovering it from scratch —
GitHub's own Dependabot alert count (shown on every push to a repo that has
it enabled) is a starting point, not a report to reproduce verbatim. Cross-
reference specific alerts against what this review actually needs to cover,
and prioritize by whether the vulnerable code path is reachable/exercised by
this app, not by raw alert count.

## What to check, by category

1. **Broken access control** — the single most common and most severe class
   of finding in a workspace like this one. Server-side enforcement is the
   only kind that counts; a check that exists only in a frontend
   `hasPermission`/`hasRole` call or only in client-side JS is not a control
   at all. Look specifically for: a permission checked in a route's
   middleware but not in the service method it calls (or vice versa); an
   ownership check (`record.user_id === user.id`) present client-side with
   no matching server-side check; an "Administrator bypasses ownership"
   pattern applied asymmetrically between read and write actions; a
   resource-level check that verifies the acting user *can approve/edit
   something* but not that the *specific record ID* belongs to them or their
   scope (IDOR — incrementing an ID in the URL/payload to reach someone
   else's record).
2. **Injection** — SQL (raw queries or string concatenation instead of
   parameter binding/an ORM's query builder), command injection
   (`shell_exec`/`exec`/`passthru`/backticks with any user-controlled input),
   XSS (unescaped output in Blade/JSX, `dangerouslySetInnerHTML`, `v-html`,
   or any `{!! !!}`-style unescaped Blade output), path traversal
   (user-controlled input used to build a filesystem path without
   normalization or an allowlist).
3. **Secrets & credential handling** — matches this workspace's own
   `CLAUDE.md` rule (never print, paste, commit, or log a real secret).
   Additionally check for: hardcoded API keys/tokens/passwords in code
   (not just missing from `.env`); secrets echoed into application logs or
   error responses (a stack trace that includes a DB password, a token
   printed in a debug log); a `.gitignore` that doesn't actually exclude
   `.env`/credential files/DB dumps — verify against the actual file, don't
   assume it's covered.
4. **Authentication & session/token handling** — does logout actually
   revoke the token server-side, or only clear client-side storage? Is
   password hashing done via the framework's standard mechanism (bcrypt/
   Argon2), not something weaker — checked in the actual code, not assumed?
   Is there any rate limiting on login or other sensitive endpoints, or is
   its absence worth flagging explicitly? Key material file permissions and
   ownership matter here too — a signing key owned by the wrong OS user is
   a real, previously-seen failure mode in this workspace (a root-owned
   private key silently broke every login for the actual request-handling
   process, which ran as a different user) — check who owns and who can
   read any private key/credential file, not just whether it exists.
5. **File uploads** — is file type/size/extension validated server-side (an
   `accept=""` attribute on a frontend input is not validation)? Is the
   upload stored outside the public webroot, or served only through an
   access-controlled route rather than a directly guessable public path? Is
   any part of the stored filename or path built from user-controlled input
   without sanitization?
6. **Mass assignment** — for any model touched by the review, check its
   fillable/guarded list for a sensitive column (a role/permission flag, an
   `is_admin`-shaped field, a `user_id`/owner column, a price/amount field)
   that's mass-assignable through a create/update endpoint that doesn't
   explicitly strip it first.
7. **Dependency / known-vulnerability awareness** — check whether a
   reported vulnerability (Dependabot alert, a CVE you already know about
   for a pinned version) actually sits on a code path this app exercises.
   For a legacy-pinned stack, the deliverable is "here's the specific
   exploitable path and the smallest compatible mitigation," not "run
   `composer update`."

## Severity

Use `test-evidence`'s scale. Security-specific calibration:

- Broken access control reachable by an authenticated-but-unauthorized user
  → **CRITICAL** minimum; **BLOCKER** if it lets that user read or mutate
  another user's/tenant's data with no ownership check at all anywhere in
  the request path.
- A known-vulnerable dependency → severity follows *actual reachability* in
  this specific app, not the CVE's own CVSS score in isolation. A critical
  CVE in a code path this app never calls is not automatically CRITICAL
  here — say so explicitly, in either direction.
- A secret already committed to git history → **CRITICAL** regardless of
  whether it still works. The fix is rotate-and-confirm, not just deleting
  the file going forward (history still has it).

## What NOT to do

- Don't recommend upgrading a pinned legacy dependency as *the* fix in a
  repository whose own `CLAUDE.md` says not to — find the narrowest
  mitigation that doesn't require it, and say plainly if none exists.
- Don't report a theoretical vulnerability class without pointing at the
  actual reachable line in *this* app that has the flaw.
- Don't paste, log, write, or otherwise include a real discovered secret
  anywhere — chat, a file, a commit, or your own findings report. Reference
  it by name/location only (e.g. "the private key at
  `storage/oauth-private.key`"), never by value, not even a partial one.

## Reporting

Use `test-evidence`'s structure. Every finding needs the exact file/line, a
concrete exploit scenario ("a user with only `X` permission can reach `Y`
by doing `Z`" — not "this could be a problem"), and the smallest fix
consistent with that repository's own conventions and constraints — not a
rewrite, and not an upgrade the repo has explicitly ruled out.
