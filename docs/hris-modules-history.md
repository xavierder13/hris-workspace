# HRIS modules registry — history archive

Moved here verbatim on 2026-09-26 from `docs/hris-modules.md` so the
registry (read in full by `feature-development` on every new feature) stays
current-state only. Superseded in places — the MRF print layout's current
behavior is documented in `reactjs-ant-design/.claude/skills/manpower-request/SKILL.md`.

---

## Cross-repo implementation notes

Detail that spans, or belongs to, more than one repository lives here —
not inside a single repository's own `.claude/skills/`, even when that
repository's skill doc is the natural place someone would first look.
Moved here 2026-09-18 from `vueportal`'s own `manpower-request` skill,
where it had accumulated detail that was actually about `reactjs-ant-design`
alone (see the root workspace `CLAUDE.md`'s repository-boundary rules for
why that's the wrong home for it).

### MRF print layout (2026-09-14)

Frontend-only feature, implemented entirely in `reactjs-ant-design` — no
backend endpoint or PDF library was added in `vueportal`; the print view
reuses the existing `edit()` + `approvalHistory()` MRF endpoints, so there
was no new backend surface to test.

- **Files**: `reactjs-ant-design`'s
  `src/pages/manpower_request/request/ManpowerRequestPrint.jsx` + `.css`,
  route `/manpower-requests/:id/print`, following the same pattern already
  established by that repo's KPI Evaluation print feature
  (`KpiEvaluationPrint.jsx`): a real page route (still wrapped in
  `MainLayout`/`ProtectedRoute` like every other page) whose `@media
  print` CSS hides everything except the printable content
  (`body * { visibility: hidden }`, then reveals only `.mrf-print`), plus a
  `window.print()` button.
- **Permission**: `manpower-request-print` (seeded in `vueportal`'s
  `PermissionSeeder.php`, granted to both `Manpower Requestor` and
  `Manpower Request Approver` in `ManpowerRequestRoleSeeder.php`) gates the
  Print button and route.
- **Content differs from the blank paper form on purpose**: it's a filled
  record of a real request, so the Requested/Reviewed/Approved-by
  signature lines are replaced with the actual `approval_history` (real
  approver names, actions, remarks, timestamps) rather than blank lines,
  and "C. NEW POSITION" is shown as a static N/A block (that request type
  remains explicitly deferred). Multiple position lines per MRF (which the
  app supports but the paper form doesn't) are each rendered as their own
  Nature-of-Request row.
- **Reason for Request prints once for the whole request** (not once per
  line item, as an earlier version did), with Column A itemizing every
  `Replacement` line (numbered when there's more than one, each with its
  own employee-to-be-replaced/reason/last-working-day) and Column B
  itemizing every `Additional` line the same way — fixed 2026-09-14 after
  review found the original per-line-item version repeated the entire
  A/B/C table once per detail row. Job Specifications remain per-line-item
  blocks below (labeled "Replacement Item N" / "Additional Item N" to
  correlate with the Reason section above). Verified live against a real
  2-Replacement + 2-Additional test record (4 total lines) — confirmed the
  API returns exactly that grouping and each replacement line carries its
  own distinct `replacement_employee`.
- **2026-09-14 — reason display simplified per feedback**: Column A no
  longer renders the full fixed checkbox list of every possible
  `REPLACEMENT_REASONS` value with checked/unchecked marks — it prints
  only the actual recorded reason as plain text (the `Others` free-text
  value when that's what was picked). The unused `REPLACEMENT_CHECKBOXES`
  constant and its `.checkbox-row`/`.checked` CSS were removed from
  `ManpowerRequestPrint.jsx`/`.css` accordingly.
- **2026-09-14 — "For HR Use Only" itemized per position line**: the "Name
  of Hired Applicant" / "Date Hired" fields were a single fixed row
  regardless of how many positions the MRF covers — now one blank hiring
  row per line item (`details.map`, columns: Position, Type, Name of Hired
  Applicant, Date Hired). "MRF Received by" stays a single row — that's
  about receiving the document as a whole, not per-position. Refined
  further same day: the itemized hiring table only replaces the single
  row when `details.length > 1` — a single-position MRF keeps the
  original plain single-row layout matching the paper form.
- **Logo asset**: the header includes the actual Addessa Corporation logo,
  extracted directly from the embedded JPEG XObject in `vueportal`'s
  `public/pdf/MANPOWER-REQUISITION-FORM-MRF-REVISED.pdf` (no logo asset
  existed anywhere in either repo before this) and saved as
  `reactjs-ant-design/src/assets/addessa-logo.jpg` — reuse this file for
  any other Addessa-branded output rather than re-extracting it.
- **Verification**: compiled cleanly through the Vite dev server with no
  new ESLint errors; every field the component reads
  (`record.branch.name`, `record.user.name`, `d.position.name`,
  `d.replacement_employee.full_name`, `entry.approver.name`, etc.) was
  checked against real `edit()`/`approval_history()` responses for an
  actual test record, not assumed from the model. **Not yet done**: no
  automated/browser-level check that the print output actually paginates
  and looks right on real paper/PDF export — no browser automation was
  available at the time, so only the CSS pattern (reused from the working
  KPI precedent) and the underlying data were verified, not the rendered
  visual result. If a print-layout defect is ever reported, check that
  first.

### Employee Master Data — Work Schedule tab (2026-09-23)

New module, both repos — not a port of any vueportal Vue reference (none
exists for this; confirmed by grep before building). Records an employee's
work-schedule **history**: Rest Day + Time In/Time Out, versioned by
Effective Date (a new row per schedule change, not an overwrite). Built
following the Offboarding/NTE sub-module precedent (thin controller +
per-module `*Maintenance` middleware + `employee_master_data/<module>`
route prefix + one tab component reading `initialData.<relation>` off the
already-eager-loaded employee record), minus file attachments and
import/export, which weren't asked for **at the time** — template
download + bulk import were added same day, once asked for; see
`reactjs-ant-design/CLAUDE.md`'s "Generate Template / Import Data
dialogs" for the full detail (a new reusable, document-type-selectable
dialog pair replacing the old single-purpose Template button, ported from
vueportal's `TemplateDownloadDialog.vue`/`ImportDialog.vue`, plus a real
pre-existing bug fixed along the way: the old Import modal never checked
for the `error_column`/`error_row_data`/`error_empty` response shape
every import endpoint in this codebase actually uses).

- **Naming, deliberately not generic**: "Work Schedule" (not "Employee
  Schedule"), `rest_day` (not `day_off` — the actual PH Labor Code Art.
  91-93 term, matching this app's existing PH-statutory vocabulary: COE,
  TIN, Pag-IBIG, PhilHealth, SSS), `time_in`/`time_out` (the DTR/biometric
  vocabulary this app's BioBridge punches already imply, not a generic
  "shift_start/end"). Applied consistently across the DB table
  (`employee_work_schedules`), model (`EmployeeWorkSchedule`), controller,
  routes, permission strings, and the frontend tab/API/labels — user-
  requested explicitly, not a default assumption.
- **Backend** (`vueportal`): migration
  `2026_09_23_140000_create_employee_work_schedules_table.php` (columns:
  `employee_id`, `effective_date`, `rest_day`, `time_in`, `time_out`,
  `remarks`), model `App\EmployeeWorkSchedule` (`belongsTo` back to
  `EmployeeMasterData`, matching `EmployeeOffboarding`'s exact relation
  style — this repo has both `belongsTo` and 3-arg-`hasOne` conventions in
  use depending on the file, and this is the one the nearest precedent
  uses), `EmployeeWorkScheduleController` (`index`/`store`/`update`/
  `delete`, inline `Validator::make()`, `rest_day` validated against a
  fixed `REST_DAYS` list on the controller), `EmployeeWorkScheduleMaintenance`
  middleware (registered in `Kernel.php` as
  `employee.work_schedule.maintenance`), routes under
  `employee_master_data/work_schedule` (`auth:api` + that middleware).
  `EmployeeMasterData::work_schedules()` (`hasMany`) added and eager-loaded
  in `EmployeeMasterDataController@index()` alongside `offboardings`/
  `disciplinaries`/etc., so the frontend tab needs no separate fetch —
  same pattern Offboarding already established.
- **Permissions** (added to `PermissionSeeder.php`, auto-granted to
  Administrator only when the seeder is re-run — no other role has them
  yet, same as any newly-added permission): `employee-master-data-work-schedule`
  (tab-visibility gate, matching Offboarding's base-string pattern, not
  `-list`), `employee-master-data-work-schedule-list` (the dedicated
  `index()` route's own gate — unused by this tab, kept only for
  structural parity with sibling modules, see the NTE/Disciplinary
  precedent), `-create`, `-edit`, `-delete`. Deliberately did **not**
  replicate Offboarding's real, already-documented seeder-vs-middleware
  permission-string mismatch (`-list`/`-add` in the seeder vs. `-list`/
  `-create` in the middleware, meaning literally 0 roles including
  Administrator hold the string the middleware checks for its `/index`
  route) — this module's middleware, seeder, and frontend all use the
  identical four/five strings, matching NTE's clean, mismatch-free
  precedent instead.
- **Frontend** (`reactjs-ant-design`): `WorkScheduleTab.jsx` (new tab in
  `EmployeeTabs.jsx`, 7th tab now, gated on the base permission per the
  `TAB_PERMISSIONS` pattern documented in that file), `workScheduleApi.js`
  (plain JSON, no multipart — this record has no file attachments, unlike
  Offboarding/NTE/Disciplinary).
- **Intended future use, not built yet**: the user's stated purpose is to
  eventually compare this schedule against the read-only Attendance tab's
  BioBridge punch logs to compute late/absence, and to feed the KPI
  Management "Attendance" component — currently
  `App\Services\KpiComputation\AccountAnalyst\AttendanceService` is a stub
  returning a hardcoded `90.00` with a "Future SAP query example" comment,
  confirmed by reading it directly. Neither that computation nor any KPI
  wiring was touched in this pass — this was record-management only, per
  the user's own scoping.
- **Permissions seeded and granted to Administrator (2026-09-23,
  user-requested)**: `php artisan db:seed --class=PermissionSeeder` was
  run against the live `vueportal` database — confirmed via `tinker` that
  all 5 `employee-master-data-work-schedule*` permissions now exist and
  Administrator holds all 5. No other role has them yet — that's a
  product decision (which of Branch Manager, Department Manager,
  Employees Relation, Payroll Admin, etc. should see this tab) not made
  here; grant via the Role UI when decided, matching how Offboarding's
  non-Administrator roles were granted (manually, outside of any
  committed seeder).
- **Migration run (2026-09-23, user-requested)**: `employee_work_schedules`
  now exists on the live `vueportal` database — confirmed via `tinker`
  (`Schema::hasTable`/`getColumnListing`). Run scoped to just this one
  migration file (`php artisan migrate --path=database/migrations/2026_09_23_140000_create_employee_work_schedules_table.php`)
  rather than a plain `php artisan migrate`, because the latter failed
  first on an unrelated, pre-existing problem: `create_employee_files_table`
  (a 2023 migration, nothing to do with this feature) tried to re-create a
  table that already exists — this database's migration history is out of
  sync with its actual schema from before this session. Not touched/fixed
  here (would mean guessing at dropping/faking migration-ledger rows on a
  problem outside this feature's scope) — worth a separate look if
  `php artisan migrate` is ever needed for something else.
- The tab is now fully live for Administrator (table exists, permissions
  granted). `php -l` confirms PHP syntax only; `npm run lint`/`npm run
  build` are real and clean; nothing has been live-tested end-to-end yet
  (no browser automation available in this environment) — the table and
  permissions are real and ready for that now, this just hasn't been
  driven through the actual UI/API by a real request.

### MRF Excel reports (2026-09-28)

Both repos. Two `.xlsx` exports from an Export Report dropdown on the MRF
list page, both filtered by a new **Date Filter** option (Date Created /
Approved Date) + the list's date range, which now drive the on-screen list
too. Built on the `EmployeeAcknowledgmentReport` export precedent
(`FromCollection` + `WithHeadings` export class, `Excel::download` from a
POST action, frontend `responseType: 'blob'` + `downloadBlobResponse()`).

- **MRF Report** — Approved MRFs only, Open / Closed / Overall. One row per
  recorded hire (Closed) or one Open row per line with no hires (same rule
  as the existing per-line Status tag). Name columns = the replaced
  employee on Replacement lines, `N/A` otherwise. Effectivity Date =
  `date_approved`; Last Day of Work = Replacement line's
  `last_working_day` (fallback: replaced employee's `date_resigned`),
  `N/A` otherwise; "DateReplaced/Date Filled" = hire's `date_hired`; Aging
  = Effectivity Date → Date Filled (or today if Open).
- **Status Report** — any status or All, one row per MRF detail line
  (MRF number, dates, requestor, branch, position, employment type,
  allocation, quantity, no. of hires, replaced employee/reason/last day,
  status, current level).
- **Backend** (`vueportal`): `ManpowerRequestController@export` /
  `@exportStatus` → `ManpowerRequestService::reportRows()` /
  `statusReportRows()` (shared `reportQuery()`) → `App\Exports\ManpowerRequestReport`
  / `ManpowerRequestStatusReport`. Routes `POST manpower_request/export`
  and `/export_status`, one `ManpowerRequestMaintenance` branch, permission
  `manpower-request-export` (`PermissionSeeder.php`, `Manpower Request
  Administrator` role in `ManpowerRequestRoleSeeder.php`). Visibility
  scoped like `index()` unless `-list-all`.
- **Frontend** (`reactjs-ant-design`): `exportReport()` /
  `exportStatusReport()` in `manpowerRequestApi.js`; Date Filter `Select`
  + grouped Export Report `Dropdown` in `ManpowerRequestIndex.jsx`, gated
  `hasRole('Administrator') || hasPermission('manpower-request-export')`.
- **Verification**: PHP syntax check, routes registered, both row builders
  dry-run read-only against the live DB with both date fields and ranges
  (status-report line counts match a direct SQL count per status), `.xlsx`
  generated in memory; frontend lint clean on touched files + `npm run
  build` OK; full HTTP matrix (both endpoints, filters, 422s, 401s) passed
  — see `test-results/2026-09-28-mrf-excel-reports.md`. Exports implement
  `WithStrictNullComparison` (otherwise `0` is written as an empty cell).
  **Not done**: browser click-through.

### Area Assignment (2026-09-28)

New module, both repos. Record management for **areas** (a named group of
branches) and the **HR head personnel** (EmployeeMasterData) assigned to
each. Record only — nothing else is scoped by it yet. Built on the MRF
pattern (service + thin controller + `*Maintenance` middleware, POST-only
routes, `{success, message, <key>}` envelope, 422 on validation/business
errors) rather than the older Work Schedule/EMD style (HTTP-200 errors).

- **Rules** (user-specified): a branch belongs to at most one area
  (`area_branches.branch_id` unique + a `lockForUpdate` check in
  `AreaService::syncBranches()` with a named-branch error); an area can
  have several HR heads and one HR head can cover several areas
  (`area_hr_heads`, unique `area_id`+`employee_id`); only **active**
  employees can be newly assigned (already-assigned ones who later went
  inactive can keep or lose areas but not gain new ones, shown
  "Inactive"); at least one branch required.
- **Backend** (`vueportal`): migrations `2026_09_28_1000..1002_*` in
  `database/migrations/new/` (`areas`: `code` unique, `name`, `description`;
  `area_branches`; `area_hr_heads`), models `App\Area`/`AreaBranch`/
  `AreaHrHead` (3-arg `hasOne`/`hasMany`), `App\Services\AreaService`,
  `AreaController` (`index`, `create` = branch options with their current
  area, `store`, `edit/{id}`, `update/{id}`, `delete/{id}`,
  `assign_employee` = replace-all areas for one employee — HR heads are
  assigned per employee, never via the area store/update),
  `AreaMaintenance` (`area.maintenance` in `Kernel.php`), permissions
  `area-list/-create/-edit/-delete` in `PermissionSeeder.php` (no
  dedicated role — granted per role in the Roles UI).
  `EmployeeMasterDataMaintenance`'s `option_list` branch now also allows
  `area-create`/`area-edit` so the HR head picker works. Branch options come
  from `area/create`, not `/branch/index` (which needs `branch-list`).
  Employee eager loads are column-limited and `setAppends(['full_name'])`
  to skip EmployeeMasterData's per-row manager-lookup accessors.
- **Frontend** (`reactjs-ant-design`): route `/areas` (`area-list`), menu
  "Human Resource → Area Assignment". `AreaIndex.jsx` has two tabs over the
  same `area/index` data — **Areas** (CRUD, expandable branch list) and
  **HR Heads** (one row per employee: areas + branches covered,
  expandable per-area branch list, Edit Areas / Unassign — the "which
  areas is this employee assigned to" view). **Assign Areas** button →
  `AssignAreasModal.jsx` (employee + areas multi-select — user-preferred
  over picking HR heads inside each area; picking an already-assigned
  employee pre-fills their areas). `AreaFormModal.jsx` (branch `Transfer`
  — Available ⇄ In this Area, searchable, fixed height — with taken
  branches disabled + labelled with their area; no HR heads).
  `EmployeeSelect` reused unchanged (single, `activeOnly`). Buttons gated
  `hasRole('Administrator') || hasPermission('area-*')`.
- **Reusable recipe**: both repos now have a `record-management` skill
  (backend / frontend halves) with Area as the reference implementation —
  start there for the next admin CRUD/master-data module.
- **Verification**: migrations run (only the 3 area files — two unrelated
  Credit Customer migrations in `new/` were left pending on purpose),
  `PermissionSeeder` + `ManpowerRequestRoleSeeder` run; 33/33 HTTP checks
  passed (CRUD, every 422 incl. partial-write rollback, taken-branch and
  inactive-employee rules, inactive-kept-on-edit, 404s, 401s,
  delete-then-recreate), then 20/20 after assignment moved to
  `assign_employee`; test rows/tokens removed. Frontend lint clean on
  new files + build OK. **Not done**: browser click-through; a non-admin
  role holding only `area-*` (picker allow-list) not exercised live.
