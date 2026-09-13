---
name: code-review
description: Review code for correctness, maintainability, security, and consistency with the owning repository's existing conventions — after actually understanding why the code is structured the way it is, not by pattern-matching against a generic "best practice." Use whenever asked to review code, review a change, or assess code quality in any repository inside this workspace.
---

# Code review

## Understand before you critique

Read the owning repository's `CLAUDE.md` and any relevant project-specific
skills before forming an opinion. A pattern that looks like a shortcut or an
anti-pattern in isolation is very often a deliberate, documented decision
(a flat model structure instead of DDD folders, POST-only endpoints instead
of REST verbs, business logic in a specific existing style for a legacy
module). Cite the actual convention you're checking against, from the actual
repository, not a generic textbook rule.

**Do not recommend rewriting working code just because a different pattern is
theoretically cleaner.** If the code works, matches the codebase's existing
conventions, and isn't the thing the user asked you to change, leave it
alone — even if you'd have written it differently. Prefer small, targeted,
project-consistent changes over broad refactors, and say so explicitly when
you're intentionally not suggesting a "nicer" alternative because it isn't
what was asked or isn't consistent with the codebase.

## What to review

Work through whichever of these are relevant to the actual change or file —
not every review needs every category:

- **Correctness**: does the code do what it's supposed to, including edge
  cases (empty input, concurrent access, boundary values)? This is the
  highest-priority category — a maintainability nitpick doesn't outrank a
  real bug.
- **Security**: injection (SQL, command, XSS), broken access control
  (a permission checked client-side but not server-side — this is common
  enough to check explicitly every time), secrets in code, unsafe
  deserialization, missing authorization on a newly added endpoint.
- **Validation & authorization**: is user input validated where it enters
  the system? Is authorization enforced at the layer that actually matters
  (server-side), not just hidden in the UI?
- **Error handling**: are failures handled in a way consistent with the rest
  of the codebase (exceptions vs. return codes, logging, user-facing
  messages)? Is a failure case silently swallowed where it shouldn't be?
- **Database queries**: N+1 queries, missing indexes for a new query
  pattern, transactions used (or missing) where multi-step writes need
  atomicity, locking used (or missing) where concurrent writes could race.
- **API design**: consistent with the rest of that API's conventions
  (status codes, envelope shape, naming) — see `cross-repository-review` if
  the concern spans frontend and backend.
- **Frontend state management**: does a new piece of state duplicate
  something already tracked elsewhere? Does it follow the repository's
  existing store/hook conventions?
- **UI-library API currency**: a prop/API used from memory can be stale —
  training data skews toward older major versions of a UI library, and a
  newer one (or one released/updated after the knowledge cutoff) can have
  renamed or deprecated the exact prop being used, producing a runtime
  console warning that won't surface as a build error and won't be visible
  without a browser (confirmed case: AntD `Divider`'s `orientation` prop
  was renamed to `titlePlacement` in a version newer than commonly
  recalled). Before relying on a specific prop name/shape for a UI
  library, check the installed version's own source or type defs
  (`node_modules/<package>/**/*.d.ts`, or the runtime source itself) when
  the prop touches placement, sizing, or anything that existed in an
  older major version of that library — don't assume recalled API shape
  is current.
- **Duplication**: real, harmful duplication (the same business rule
  encoded twice, likely to drift) vs. superficial similarity that doesn't
  need a shared abstraction. Don't flag the second kind.
- **Naming & consistency**: does new code match the naming conventions
  already established in the file/module it's added to?
- **Performance**: only flag this where it's plausibly significant (a query
  in a hot path, an O(n²) over a collection that can be large) — don't
  micro-optimize code that runs once per request over a handful of rows.
- **Regression risk**: does this change touch something else relies on?
  Hand off to `regression-testing` for the full blast-radius analysis if the
  change is non-trivial.

## Severity and framing

Use the same severity scale as `test-evidence`
(BLOCKER/CRITICAL/HIGH/MEDIUM/LOW/INFO) so review findings and test findings
are comparable. A style preference is INFO at most. A missing server-side
authorization check on a destructive action is CRITICAL or higher. Don't let
volume of minor comments bury the one finding that actually matters — lead
with the highest-severity findings.

## Reporting

State the finding, the exact file/line, why it matters (a concrete failure
scenario, not just "this could be a problem"), and the smallest change that
would fix it — in the style and convention of the repository it's in. If you
genuinely don't have a good targeted fix, say that too rather than proposing
a disruptive rewrite to avoid looking incomplete.
