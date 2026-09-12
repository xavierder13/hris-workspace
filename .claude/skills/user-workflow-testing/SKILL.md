---
name: user-workflow-testing
description: Test a feature the way a real business user would use it end-to-end — login, navigate, fill a form, submit, verify the result actually shows up, edit it, reload, check it persisted, check permissions, check invalid input — rather than only checking that an individual function or endpoint executes without error. Use whenever asked to test a workflow, act as a user, verify a business process, or test a feature "end to end."
---

# User workflow testing

The question this skill answers is never "does the function execute?" It's:

**"Can a real user, following the application's actual real flow, successfully
complete the business process — and does the system behave correctly at every
step along the way, including the steps where things are supposed to go
wrong?"**

## Follow the real workflow, not an idealized one

Before testing, find out what the actual application flow is — read the
relevant frontend pages/routes and the module's own documentation
(`CLAUDE.md`, project skills) to learn the real sequence of screens and
actions, including any project-specific quirks (an extra confirmation step,
a two-stage save, a permission that only shows a button under certain
conditions). Do not invent a "clean" idealized workflow that skips steps the
real UI actually requires, and do not assume every module works the same way
as a similar module elsewhere in the app — verify per module.

## The shape of a full workflow test

Adapt this to the actual feature; not every step applies to every feature,
and some features need steps this list doesn't have. Treat it as a checklist
to consider, not a script to follow blindly:

1. Authenticate as a user with the relevant role/permissions.
2. Navigate to the module the way a real user would (menu, link, direct URL)
   — not by calling an API directly, if the point is to test the UI.
3. Start creating a record. Enter required fields.
4. Enter optional fields too, at least once per test pass — optional fields
   are where "accepted but never persisted" bugs hide.
5. Submit, and check what actually happens: validation messages for bad
   input, a success indicator for good input, whichever is expected for what
   you entered.
6. Verify the new record actually appears where a user would expect to find
   it (a list, a dashboard, a count) — not just that the API returned success.
7. Open the record and verify every field shows the value that was actually
   entered, not a default or a stale cache.
8. Edit it. Change some fields, leave others alone. Save.
9. Verify the changed fields show the new value and the untouched fields
   still show their original value.
10. Reload the page (a real refresh, not just re-rendering from local state)
    and verify everything still shows correctly — this is the single most
    common place a "works in the UI" bug turns out to be a persistence bug.
11. Check permissions: does a user without the relevant permission correctly
    lose access to the action/button/page? Does a user with the permission
    but who doesn't own the record behave correctly per that feature's
    ownership rules (if any)?
12. Test invalid input deliberately: empty required fields, wrong types,
    boundary values, a value that should trigger a specific validation
    message — confirm the message shown is accurate, not just that
    *something* was rejected.
13. Test the feature's own edge cases — the specific business rules that
    module implements (a status transition that should be blocked, an
    action that should only be available to the record's owner, a limit on
    quantity or level). Find these from the module's actual implementation
    and documentation, not from generic assumptions.
14. Where it's useful and safe, verify the underlying data too (via the
    application's own read endpoints, or a scoped read-only query) — a
    workflow can look correct in the UI while the database holds something
    different, and that gap is exactly the kind of defect this skill exists
    to catch. Use the least invasive method (see the workspace `CLAUDE.md`'s
    database safety rules).

## Browser testing vs. API-only testing

Prefer driving the actual application through the browser when browser
automation is available in the environment — click, type, select, submit,
navigate, reload, and verify what's actually visible on screen. A successful
HTTP response is not proof the user-visible workflow works: the UI might
never call that endpoint, might mishandle its response, or might show a
stale value despite the write succeeding.

When browser automation isn't available, direct API calls (matching exactly
what the frontend actually sends — verified via `cross-repository-review`,
not invented) are a legitimate fallback for exercising backend behavior, but
say explicitly that UI-level behavior (rendering, button visibility, client
validation messages, loading states) was **not** verified this way, so
nobody mistakes an API-level pass for a full workflow pass.

Also inspect technical evidence when it's available and useful: browser
console errors, the actual network request/response for the step that
failed, and server-side logs — these often explain *why* a step failed
faster than re-running it with variations would.

## Distinguish two kinds of failure

Report both when you find them, and don't conflate them:

- **User-visible failure**: the actual person using the app would notice
  something wrong — a missing success message, a field that doesn't save, a
  button that's there but shouldn't be, a workflow that dead-ends.
- **Technical implementation issue**: something is wrong in the code but a
  user wouldn't necessarily notice yet (a race condition that hasn't been
  hit, a field that's silently ignored but nobody's tried to set it, a
  missing server-side check that a well-behaved client never triggers).

A technical issue is still worth reporting — but don't inflate it to
"broken for users" if no user-visible symptom actually exists yet, and don't
undersell a user-visible failure just because the underlying cause is a
one-line technical fix.

## Recording results

Use `test-evidence` for the structure. At minimum, every step you actually
performed needs to be distinguishable from a step you skipped or couldn't
test — don't present an assumed-passing step as if it were verified.
