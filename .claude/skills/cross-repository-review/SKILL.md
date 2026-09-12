---
name: cross-repository-review
description: Compare what a frontend repository actually sends/expects against what a backend repository actually accepts/returns, field by field and endpoint by endpoint, to find real mismatches — nonexistent endpoints, wrong HTTP methods, naming mismatches, missing/unused fields, validation/auth/pagination mismatches. Use whenever asked to cross-check an API, verify a contract, or find integration defects between two repositories. Requires repository-discovery to have identified which repo is which first.
---

# Cross-repository review

The one rule that overrides everything else in this skill: **a mismatch is
only real if the code proves it.** Every finding below must trace to an
actual line of code on both sides — the frontend call site and the backend
handler — not to a guess about what a reasonable API "probably" does.

## Before starting

Confirm (from `repository-discovery`, or by doing it now if it hasn't run):
which repository is the frontend, which is the backend, where each one's
`CLAUDE.md` lives, and read both. A repository's own documented conventions
(e.g. "this module intentionally uses POST for reads, don't unify it with
REST verbs") change what counts as a defect versus intentional design. Don't
flag something as wrong that the project has explicitly documented as
deliberate.

## What to extract from the frontend, per feature/module

- Every API service/client method relevant to the feature: its name, HTTP
  method, URL, request payload shape, query/route parameters.
- How auth is attached (header, cookie, token source).
- How the response is consumed: which fields are read, whether it expects a
  flat object, a nested object, or an array; how pagination/filtering/sorting
  params are sent if applicable.
- How errors and validation failures are handled (what status code/shape is
  expected, and where that's parsed).
- Whether the field names sent match the repository's own stated convention
  (camelCase vs snake_case, singular vs plural) — inconsistency *within* the
  frontend is itself worth noting even before comparing to the backend.

## What to extract from the backend, per feature/module

- The actual route definition: method, URL, middleware/guards applied.
- The controller action: what it reads from the request, what validation
  rules apply (and whether they're required/nullable/typed how you'd expect).
- What it delegates to (service layer, direct model access) and what it
  ultimately does — don't stop at the controller if the real logic (and
  real field usage) lives one layer deeper.
- What the response actually contains — read the exact keys returned, not a
  docblock or comment claiming what's returned.
- Authorization: what permission/role/ownership check actually gates this
  action, enforced *server-side* (a frontend-only check is not enforcement).

## Comparison checklist

Work through these systematically rather than spot-checking:

- **Existence**: does every frontend call have a real, reachable backend
  handler? Does every backend endpoint have a frontend caller, or is it
  dead/unused?
- **Method + URL**: do they match exactly, including trailing structure
  (`/resource/{id}` vs `/resource?id=`)?
- **Request fields**: for every field the frontend sends, does the backend
  read it? For every field the backend requires or reads, does the frontend
  send it? Flag both directions — a frontend field the backend silently
  ignores is as real a defect as a backend requirement the frontend never
  satisfies.
- **Field naming**: same field, different name/case on each side (a classic:
  frontend sends `department_id`, backend reads `department`).
- **Shape**: array vs. scalar, nested object vs. flat, ID vs. embedded object
  (frontend sends `{employee: {id, name}}`, backend expects `employee_id`).
- **Types/nullability**: does the frontend ever send `""` where the backend
  expects `null`, or omit a field the backend expects present-but-nullable?
- **Response contract**: does the frontend read a field the backend response
  doesn't actually contain? Does the backend return data the frontend never
  uses (not necessarily a defect, but worth noting as dead payload)?
- **Create vs. update asymmetry**: does a field work on create but silently
  get dropped on update (or vice versa)? This is a common, high-impact class
  of bug — check both operations explicitly, don't assume update mirrors
  create.
- **Validation**: does frontend client-side validation match what the
  backend actually enforces? A field marked required in the UI that the
  backend accepts as optional (or vice versa) is worth flagging even if it
  doesn't break anything today.
- **Auth/authorization**: does the frontend hide an action behind a
  permission check that the backend doesn't actually enforce (or enforces
  differently)? This is a security-relevant class of finding — treat it
  with the severity it deserves in `test-evidence`.
- **Pagination/filtering/sorting**: do the parameter names and semantics
  (page-based vs. cursor-based, 0- vs. 1-indexed) actually match?
- **Error responses**: does the frontend's error-handling code correctly
  interpret the shape the backend actually returns on validation failure vs.
  business-rule failure vs. not-found vs. unauthorized? These are often
  different shapes from the same API — check each path.

## Producing findings

For each confirmed mismatch, capture (see `test-evidence` for the full
structure): which repository is the actual root cause, the exact
file/line/function on each side, the frontend code and backend code side by
side as evidence, and what a fix would need to touch. Don't report a
mismatch without both sides' evidence — "the frontend calls X" is not a
finding until you've also shown what the backend does (or doesn't do) with
it.

If something looks wrong but you can't fully confirm it (e.g. you can see the
frontend call but the backend logic is generated/dynamic and hard to trace
statically), report it as **unconfirmed** rather than as a defect, and say
exactly what would confirm it (e.g. "would need to execute this call against
a running backend to see the actual response shape").

## What NOT to flag

- A difference the owning repository's `CLAUDE.md` documents as intentional.
- A backend endpoint that's unused by *this* frontend but is clearly used by
  another consumer (check for other frontends/mobile clients before calling
  something dead).
- A naming difference that's consistent and intentional across the whole
  API (e.g. the whole backend uses snake_case and the whole frontend
  translates it at one boundary layer) — that's a design choice, not a bug,
  unless the translation layer itself has a bug.
