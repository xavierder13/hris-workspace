---
name: regression-testing
description: Given a change (a diff, a feature, a fix), determine the actual blast radius — which frontend components, backend endpoints, database tables, services, permissions, and workflows could plausibly be affected — and produce a focused checklist to verify, instead of re-testing the whole application. Use whenever asked to check for regressions, assess the impact of a change, or scope what needs re-testing after a fix.
---

# Regression testing

The goal is a **focused** checklist, not exhaustive re-testing of the whole
application. Testing everything after every change isn't rigor — it's a way
to avoid actually thinking about what the change could break. Scope it.

## 1. Identify what actually changed

Read the actual diff (or the actual change described), not just its
headline description. A change described as "add a delete button" might
also have touched a shared status-gating function used by three other
actions — the diff shows that; the description might not.

## 2. Trace outward from the change, one hop at a time

For each changed piece, ask what else touches it:

- **Directly affected**: the function/endpoint/component actually modified.
- **Frontend components**: what else renders using the same store, the same
  API service file, the same shared component (a status-color map, a
  permission-check helper, a shared form) that was touched?
- **Backend endpoints**: what else calls the same service method, the same
  model, the same shared validation rule?
- **Database tables**: does the change alter a column's meaning, a status
  enum's set of valid values, or a relationship? Everything that reads that
  column/relationship is in scope.
- **Services**: a shared service method used by more than one controller
  action — changing its behavior for one caller can silently change it for
  all of them.
- **Permissions**: did the change alter what a permission gates, or which
  permission gates an action? Every place that permission is checked is in
  scope, not just the one you were asked about.
- **Workflows**: does the change sit inside a multi-step business process
  (an approval chain, a multi-page form, a state machine)? Check the steps
  before and after it in that process, not just the step itself.

Stop tracing outward once you reach something that clearly doesn't share
state, code, or data with the change — don't keep going "just in case."

## 3. Weigh actual impact, not just reachability

Something being technically reachable from the change isn't the same as it
being at real risk. A shared utility function that was extended with a new
optional parameter (existing callers unaffected) has a smaller real blast
radius than a shared function whose existing behavior was altered for every
caller. Say which you're dealing with — it changes how much the checklist
needs to cover.

## 4. Produce the checklist

For each item identified as genuinely at risk, write one concrete,
checkable item: what to do, what result confirms it's fine. Order by risk —
highest-risk items first. Keep it a checklist, not a narrative — someone
else should be able to run through it without re-deriving your reasoning.

Example shape for one item:

```
[ ] MRF Index list — status filter dropdown still matches every real status
    value after the status-string rename (create a Draft, Disapprove it,
    confirm it's both visible and filterable as "Disapproved")
```

## 5. Escalate scope only when the evidence says to

If tracing the change reveals it touches something broad by nature (a shared
auth middleware, a core status enum used across many modules, a database
migration that alters a column read by several features), say so plainly and
widen the checklist accordingly — don't stay artificially narrow just to
keep the list short. The point is matching effort to actual risk in both
directions.

## Handoff

Once the checklist exists, executing it is `user-workflow-testing` (for
user-facing items) or `cross-repository-review` (for contract-shaped items)
territory — this skill's job ends at producing the accurate, focused list.
