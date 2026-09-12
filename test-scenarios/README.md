# Test scenarios

This directory holds written test scenarios for features across the
repositories in this workspace, plus the reusable template they're written
from.

- `templates/` — the generic scenario template. Reusable across any module
  (Employee Master Data, MRF, KPI, Recruitment, Offboarding, Reports, Auth,
  RBAC, or any future feature) and across a completely different
  application if the repositories in this workspace are ever swapped out.
- `scenarios/` — actual scenarios written against this workspace's current
  repositories. Name each file after the feature and case it covers, e.g.
  `employee-master-data-create.md`, `manpower-request-multi-level-approval.md`,
  `kpi-evaluation-submit-and-approve.md`.

## Writing a new scenario

Copy `templates/scenario-template.md`, fill it in for the specific feature
and case, and save it under `scenarios/`. Keep one scenario file focused on
one coherent case — a full happy-path workflow, or one specific edge case —
rather than combining many unrelated cases into one file.

## Running a scenario

Use `/test-feature` or `/test-workflow` (or the `integration-tester` /
`user-workflow-tester` agents directly) and point them at the scenario file.
Record the executed result back into the scenario's own `Actual Result` /
`Status` / `Evidence` fields, or into a matching file in `test-results/` if
the result is substantial enough to warrant its own write-up — see the
`test-evidence` skill for what belongs in each.

## Reusing scenarios across modules

These templates are deliberately generic. When testing a new module for the
first time, don't invent a new template — reuse the existing one and let the
*content* (steps, data, expected results) be specific to that module.
