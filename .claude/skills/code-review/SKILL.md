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
  without a browser (confirmed case: AntD `Divider`'s `type` prop is
  deprecated in favor of `orientation` in a version newer than commonly
  recalled — **correction 2026-09-23**: this bullet previously claimed
  `orientation` itself was "renamed to `titlePlacement`," which was wrong;
  re-verified directly against the installed `divider/index.d.ts` —
  `orientation` is current and non-deprecated, `titlePlacement` is an
  unrelated prop for text position, never a rename target for it; don't
  repeat that claim). Before relying on a specific prop name/shape for a UI
  library, check the installed version's own source or type defs
  (`node_modules/<package>/**/*.d.ts`, or the runtime source itself) when
  the prop touches placement, sizing, or anything that existed in an
  older major version of that library — don't assume recalled API shape
  is current. **A component's real prop types don't always live in its
  `index.d.ts`** (confirmed case: AntD `Alert`'s `message`/`title` are
  declared in `alert/Alert.d.ts`, only re-exported from `index.d.ts`) —
  grep every `.d.ts` file under that component's folder
  (`grep -rn "@deprecated" node_modules/<package>/es/<component>/`), not
  just the top-level one, or a real deprecated prop can be missed even
  when you did check. This check is easy to skip in practice because it
  only fires when someone remembers to run it manually on the specific
  component just touched — for a real regression check (e.g. after a
  batch of UI edits, or when a console warning is reported with no
  component named), a scripted sweep across every component actually
  imported in the app is more reliable than a per-file spot-check: collect
  every `import { X } from "antd"` name in `src/`, grep `@deprecated`
  across each one's whole `node_modules` folder (not just `index.d.ts`),
  then grep the app's own JSX for that component tag using that specific
  prop name (strip `//` comments from the matched attribute text first —
  a prop name mentioned only in a comment, e.g. explaining a prior fix, is
  a false positive, not a live finding). This found and fixed 3 real
  instances across the whole `reactjs-ant-design` app in one pass
  (2026-09-23), 2 of which a prior module-scoped, `index.d.ts`-only audit
  had missed — see that repo's own `CLAUDE.md` for the specific findings.
  **A whole component, not just a prop, can be deprecated too** — confirmed
  case: AntD's `List` itself logs a deprecation warning unconditionally on
  every render, with no `@deprecated` tag on any individual prop in its
  `.d.ts` (whole-component deprecations don't reliably show up that way);
  it's only visible by grepping the runtime `.js` for `'deprecated'`
  strings, not just the type defs. Same sweep method applies — collect
  every component actually imported, don't just check the one a report
  named, since (2026-09-24) a single reported instance on one page led to
  2 more of the exact same deprecated component elsewhere in the app.
- **Modal-hosted form timing (AntD, or any UI library with the same
  lazy-mount-on-open + destroy-on-close pattern)**: a `Form` connected via
  `Form.useForm()` that lives inside a `Modal` isn't in the render tree
  until the Modal has actually opened — a Modal that unmounts its content
  on close (AntD's `destroyOnHidden`, this workspace's standard for its
  list→modal CRUD pages) re-triggers this on every open, not just the
  first. A handler that populates or resets the form (`resetFields()`,
  `setFieldsValue()`) synchronously in the same function that also flips
  the "open" state — the common shape being `openCreate`/`openEdit`
  calling `form.setFieldsValue(...)` then `setModalOpen(true)` — runs
  before that render happens, producing a real runtime warning ("Instance
  created by `useForm` is not connected to any Form element") that, like
  the deprecated-prop class of bug above, won't surface without a browser.
  The fix is to move that population/reset into the Modal's own
  `afterOpenChange` callback instead, so it runs only once the Form is
  actually mounted. Confirmed real, 2026-09-23: a single reported instance
  (`reactjs-ant-design`'s new Work Schedule tab) led to a sweep that found
  the exact same copy-pasted bug in 6 other files across that repo — see
  its own `CLAUDE.md`'s Form and Validation Conventions section for the
  full list and the correct pattern to match.
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
