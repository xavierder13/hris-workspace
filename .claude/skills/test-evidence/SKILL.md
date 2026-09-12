---
name: test-evidence
description: Capture test results and defects with enough concrete detail (feature, workflow step, repository, file, endpoint, payload, actual vs expected, reproduction steps, severity) that a developer could act on the finding without re-deriving it. Use whenever recording the result of a test, review, or investigation — this is the shared reporting structure every other skill and agent in this workspace writes findings into.
---

# Test evidence

A finding is only useful if someone who wasn't in the room can act on it.
Every failed test, and every defect found during review, should carry enough
of the fields below that reproducing and fixing it doesn't require asking
"what exactly did you mean?"

## Fields to capture for a failure or defect

Not every field applies to every finding — use what's relevant, but check
each one rather than skipping straight to the ones that are easy to fill in:

- **Feature / workflow** — what business capability was being tested.
- **Step** — which specific step in the workflow failed (see
  `user-workflow-testing`'s step list for the shape).
- **Expected result** — stated concretely, not "it should work."
- **Actual result** — what actually happened, concretely.
- **Repository** — which one owns the problem (this requires having actually
  identified the root cause, not just where the symptom appeared — a
  frontend symptom can have a backend root cause).
- **File** — the actual file path.
- **Function / method / endpoint** — the actual name, not a paraphrase.
- **Relevant payload** — the actual request body/params involved, trimmed
  to what's relevant (redact anything that looks like a secret or real
  personal data even in a test payload).
- **Relevant response** — the actual response body, same redaction rule.
- **Error message** — verbatim, not summarized, when one exists.
- **Browser console error** — verbatim, if the finding came from browser
  testing and the console showed something relevant.
- **Server/log error** — verbatim, if available and relevant.
- **Likely root cause** — your best determination of *why*, distinguished
  clearly from the symptom.
- **Confidence level** — CONFIRMED (traced in code and/or reproduced) or
  SUSPECTED (plausible but not fully verified) — never present a suspected
  cause as if it were confirmed.
- **Reproduction steps** — concrete enough that someone else gets the same
  result following them, in order, starting from a known state.

## Severity scale

Use exactly these levels, and calibrate honestly — don't inflate to get
attention, don't downplay to avoid sounding alarming:

- **BLOCKER** — the core workflow cannot be completed at all by any user. No
  workaround exists.
- **CRITICAL** — a security gap (e.g. a permission enforced only client-side),
  data loss, or data corruption. May have a narrow trigger condition, but the
  consequence is severe.
- **HIGH** — a real, reachable defect that breaks a workflow or produces
  wrong data/behavior for a normal user, without needing an unusual trigger.
- **MEDIUM** — a real defect with a narrow trigger, a workaround exists, or
  the impact is limited to a secondary/less-common path.
- **LOW** — cosmetic, or affects a rare edge case with no real-world
  consequence.
- **INFO** — not a defect: an observation, a style note, a suggestion, or an
  explicitly-unconfirmed suspicion worth flagging for someone else to check.

When genuinely unsure between two adjacent levels, say so and give the
reasoning — "HIGH, not CRITICAL, because X" is more useful than silently
picking one.

## What a PASS looks like

Don't only write up failures. Record what was actually verified and how, so
a passing result is distinguishable from something nobody checked. A short
"verified: created a Draft, submitted, approved through all 3 configured
levels via direct API calls with tokens for the actual mapped approvers,
confirmed final status and full approval history" is worth recording — it
tells the next person exactly what confidence already exists and what
still doesn't.

## Where results go

Write substantial results into `test-results/` (one file per test run or
per feature, whichever keeps it navigable), using the report structure in
the workspace `CLAUDE.md`. Quick, narrow checks that don't warrant a
standalone file can be reported inline in the conversation instead — use
judgment; the point is that a *significant* finding survives past the
conversation, not that every single check produces a file.
