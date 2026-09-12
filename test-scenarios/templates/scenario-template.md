<!--
Generic test scenario template — reusable for any module in any application.
Copy this file into test-scenarios/scenarios/, rename it for the feature and
case it covers, and fill in every section. Delete this comment block once
copied.
-->

# Scenario: <short, specific title>

## Feature

<the business feature/module this scenario belongs to>

## Objective

<what this scenario is actually trying to verify — one or two sentences,
specific enough that "pass" and "fail" are unambiguous>

## Prerequisites

<what must already be true before this scenario can run — test accounts,
existing records, required permissions, required configuration>

## User role

<the role/permission set the test should be performed as; if the scenario
compares two roles (e.g. owner vs. non-owner), say so explicitly>

## Starting state

<the exact state of the relevant record(s)/data before the first test step
— e.g. "a Manpower Request in Draft status, owned by the test user">

## Test steps

<numbered, concrete, one action per step — see the user-workflow-testing
skill for the shape of a full workflow if this scenario covers one>

1. ...
2. ...
3. ...

## Test data

<the actual values used — field values, payloads, IDs. Be specific; "a
valid branch" is not test data, "branch_id: 1 (ADMINISTRATION)" is>

## Expected result

<concrete, checkable — what should be true after the steps above, for both
the UI (if applicable) and the underlying data (if verified)>

## Actual result

<filled in when the scenario is executed — what actually happened>

## Status

<PASS / FAIL / BLOCKED / NOT YET RUN>

## Evidence

<filled in when executed — link or inline reference to the relevant
file/endpoint/payload/response/error, per the test-evidence skill's
structure. For a FAIL, this should be enough for a developer to reproduce
the problem without re-running this scenario from scratch.>

## Regression impact

<what else could plausibly be affected if this scenario's feature changes —
see the regression-testing skill; fill in when known, not necessarily at
authoring time>
