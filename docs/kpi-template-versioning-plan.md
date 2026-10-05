# KPI template versioning — plan (pending)

Status: **not built**, raised 2026-10-05. Repositories: vueportal (backend)
and reactjs-ant-design (frontend). Read both repos' `kpi-management` skill
("Known issues") for the current behaviour.

## The concern

One position may need several KPI templates over time. For example,
Account Analyst evaluated under template v1, then under a changed v2.
How should data and the KPI Consolidated Report handle that?

## Verified current behaviour (2026-10-05)

- `kpi_templates.position_id` is **UNIQUE**, and deactivated rows count, so
  a position has exactly one template.
- Removing a template item that an evaluation already uses returns 422.
  The message says "deactivate this template and create a new one", but
  creating one then fails with "A KPI template already exists for this
  position." It is a dead end.
- `kpi_evaluation_items` freeze each component's **weight**. The component
  and demerit **code and name** are read live from the template item, so
  renaming a component relabels old evaluations.
- The Consolidated Report (`kpiReportLayout.js`, Excel writer) keys
  columns by **position + component code**:
  - Summary: one sheet. Per-template components and demerits go to the
    "Job Component Averages" / "Demerit Averages" sheets.
  - Detailed: one sheet per position, or per branch + position.

  If a position had two templates, components with the same code would
  merge into one column.

## Decision needed first

For a position with evaluations under two templates, should the report
be laid out:

1. **Separate per template** (recommended):
   - Summary: one row per position + template ("Account Analyst — v1",
     "— v2").
   - Detailed: one sheet per position + template ("Account Analyst (v2)").
   - Every column means one thing, and final grades stay comparable
     across rows.
2. **Merged per position**: one row or sheet per position, with the union
   of components; same codes are disambiguated as "A (v1)" / "A (v2)".
   This is wider and harder to read.

Also confirm that versioning should be built at all.

## Phases (after the decision)

1. **vueportal — data model** (migrations in
   `database/migrations/new/`, run with `--path`, only with the user's
   OK):
   - Drop the UNIQUE index on `kpi_templates.position_id`; enforce "one
     *active* template per position" in code.
   - Add `kpi_evaluations.kpi_template_id`, backfilled from the
     evaluation's items.
   - Snapshot `component_code` and `component_name` on
     `kpi_evaluation_items` and on the demerit ratings, backfilled from
     the current template items, so renames no longer rewrite history.
   - Evaluation create uses the active template.
   - Fix the 422 message.
2. **vueportal — template API:**
   - "New version" = copy the active template, then deactivate the old
     one (replaces the dead end).
   - The template list returns versions per position.
   - `KpiReportController@consolidated` reads the snapshot code and name,
     and returns the template on each row and position group.
3. **React — templates UI:**
   - The list shows versions and status.
   - A "New version" action.
   - Versions already used by evaluations are read-only, or show a
     warning.
4. **React — Consolidated Report:**
   - Group by position + template as decided, in `kpiReportLayout.js`
     (Summary and Detailed), the Excel writer (sheet names) and the print
     headers.
   - Screen, print and Excel must stay identical.
5. **Verify:**
   - Create v2 for a test position with overlapping component codes, and
     evaluate under both versions.
   - Export Summary, Detailed and Detailed grouped by branch. Columns
     must never merge across versions.
   - Rename a component in v2: v1 evaluations must keep their old name.
   - Print to PDF (headless Edge `Page.printToPDF`; see reactjs
     `CLAUDE.md` "Printing").

## Test data (only on the original dev machine's local database)

- Templates 2 "Cashier KPI (test)" (position 20, 3 components, 2 demerit
  items) and 3 "Sales Specialist KPI (test)" (position 14, 4 components,
  no demerits).
- Approved evaluations #19–22, September 2026, Supervisor type:
  - Cashier: Oribello (AGOO), Ducusin (ILAGAN).
  - Sales Specialist: Lopez (AGOO), Dulay (BAGUIO).
- Created through the KPI API. On another machine, recreate them the same
  way: POST `/api/kpi/templates`, then for each evaluation POST
  `evaluations` → PUT grades → `submit` → `approver-ratings` → `approve`.
- Approved evaluations can't be deleted through the API.
