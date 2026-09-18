# HRIS modules registry

This is a **living document**, not a one-time snapshot. It exists so that
when a new HRIS feature/module comes up, there's a single place that says
what already exists (so new work reuses existing patterns/endpoints instead
of duplicating them) and what counts as "HRIS" at all in this workspace.

**Update this file whenever a new HRIS module is added** (new backend
controller/routes + service, new frontend page/service/store) — add it to
the registry below in the same shape as the existing entries. If something
in here turns out to be wrong or incomplete, that's expected — the user
will call it out and it should be corrected here, not just fixed in
conversation and forgotten.

Everything below was verified directly against the code on
2026-09-13 (controller list, `routes/api.php` group prefixes, frontend
`src/pages`/`src/services`/`src/store` contents) — not assumed. Re-verify
anything load-bearing if it's been a while since this was last touched.

## The core fact this registry is built on

`vueportal` (the Laravel backend) is **not** an HRIS-only backend — it's a
shared backend for several unrelated business domains bundled into one app
(inventory, credit/collections, marketing, SAP integration, online banking,
SMS blast, sales). `reactjs-ant-design` (the frontend in this workspace) is
specifically **the HRIS frontend** — its `src/pages` only contains HR-shaped
pages (auth, dashboard, employee_master_data, kpi, manpower_request,
recruitment, permission, role, user). So in practice:

**"HRIS" = the subset of `vueportal` that `reactjs-ant-design` consumes or
plausibly should consume, not the whole `vueportal` backend.**

A large chunk of `vueportal`'s HR-shaped backend capability (see "Backend HR
capability with no frontend yet" below) has no UI in this frontend at all
yet — that may be deliberate scoping or it may be a real gap; don't assume
either way, ask if it matters for a task.

## What HRIS covers here

### Tier 1 — Core HRIS modules

| Module | Backend | Frontend |
|---|---|---|
| Employee Master Data | `EmployeeMasterDataController`, prefix `employee_master_data` | `src/pages/employee_master_data/`, `src/services/employee/employeeOptionApi.js`, hooks |
| — Key Performance | `EmployeeKeyPerformanceController`, prefix `employee_master_data/key_performance` | none yet |
| — Classroom Performance Rating | `EmployeeClassroomPerformanceRatingController`, prefix `employee_master_data/classroom_performance_rating` | none yet |
| — OJT Performance Rating | `EmployeeOjtPerformanceRatingController`, prefix `employee_master_data/ojt_performance_rating` | none yet |
| — Branch Assignment Position | `EmployeeBranchAssignmentPositionController`, prefix `employee_master_data/branch_assignment_position` | none yet |
| — Merit History | `EmployeeMeritHistoryController`, prefix `employee_master_data/merit_history` | none yet |
| — Training (employee's own history) | `EmployeeTrainingController`, prefix `employee_master_data/training` | none yet |
| — NTE (Notice to Explain) | `EmployeeNTEController`, prefix `employee_master_data/nte` | none yet |
| — Disciplinary | `EmployeeDisciplinaryController`, prefix `employee_master_data/disciplinary` | none yet |
| — Offboarding | `EmployeeOffboardingController`, prefix `employee_master_data/offboarding` | none yet |
| Recruitment | `RecruitmentController`, prefix `recruitment` — **external "careers" portal integrated over HTTP** (token auth via `getOrCreateToken()`), not a locally-owned dataset. `new_hired()` pulls newly-hired applicants from that external API; hires land in `EmployeeMasterData` (tracked via `EmployeeNewHiredSyncLog` to avoid re-syncing). `vacancies()` already computes Required Plantilla vs. Existing Headcount (reused by MRF's headcount snapshot — see below) | `src/pages/recruitment/JobApplicantList.jsx` |
| Manpower Request (MRF) | `ManpowerRequestController` + `ManpowerRequestService`, prefix `manpower_request` | `src/pages/manpower_request/`, `src/services/manpower_request/manpowerRequestApi.js`, `manpowerRequestStore.js`, `useManpowerRequests.js` — see `.claude/skills/manpower-request/SKILL.md` in `vueportal` for full detail |
| KPI Management | `KpiAuthController`, `KpiTemplateController`, `KpiTemplateItemController`, `KpiEvaluationController`, `KpiMyEvaluationController`, `KpiBehaviorCriteriaController`, `KpiSettingController`, `KpiReportController`, `KpiEmployeeController`, prefix `kpi` | `src/pages/kpi/{templates,evaluations,my-evaluations}`, `src/services/kpi/{kpiTemplateApi,kpiEvaluationApi}.js`, `kpiTemplateStore.js`, `kpiEvaluationStore.js` |
| Employee Loans | `EmployeeLoansController`, prefix `employee_loans` | none yet |
| Employee Premiums | `EmployeePremiumsController`, prefix `employee_premiums` | none yet |
| Employee Attendance Log | `EmployeeAttlogController`, prefix `employee_attlog` | none yet |
| Holiday Calendar | `HolidayCalendarController`, prefix `holiday_calendar` | none yet |
| Training File Library (by position, not per-employee) | `TrainingController`, prefix `training` (uploads/permissions, position-scoped) | none yet |
| Legacy Employee module | `EmployeeController` (model `App\Employee`, import/export) — **appears superseded by Employee Master Data**; confirm before building anything new on it | none |

### Tier 2 — HR-owned org/reference structure (used broadly, not HR-exclusive)

Branch (`BranchController`, `branch`), Company (`CompanyController`,
`company`), Department (`DepartmentController`, `department`), Division
(`DivisionController`, `division`), Rank (`RankController`, `rank`),
Position (`PositionController`, `position`), Address
(`AddressController`, `address`). Frontend: `branchStore.js`,
`departmentStore.js`, `positionStore.js` + matching hooks — consumed as
dropdown/reference data by MRF and other forms, no dedicated CRUD pages in
this frontend today.

### Tier 3 — Cross-cutting platform infrastructure (not HR-specific at all)

Auth (`AuthController`, `auth`), User/Role/Permission
(`UserController`/`RoleController`/`PermissionController`, prefixes
`user`/`role`/`permission`), the shared multi-level approval engine
(`AccessModuleController`/`AccessChartController`/`AccessChartUserMapController`,
prefixes `access_module`/`access_chart`/`access_chart_user_map` — used by
MRF and also by non-HR modules like Marketing Event), Activity Log
(`ActivityLogController`, `activity_logs`), Email
(`EmailController`, `email`). Frontend: `authStore.js`, `useAuth.js`,
`src/pages/{user,role,permission}/`.

### Explicitly NOT HRIS (other business domains sharing this backend)

Inventory Reconciliation, Product/ProductModel/ProductCategory/Brand,
Credit Investigation, Hot Area, Tactical Requisition, Marketing Event (+
User Map), SAP Database / SAP Business One, Online Banking, Flagship Sale
(+ List + Text), Expense Particular, Sales Main / Sales Reports,
Promodizer Brand. Don't reference these as precedent for HRIS work, and
don't touch them while working on an HRIS task.

## Conventions a new HRIS module should follow

Verified from the existing modules above, especially MRF (the newest,
most deliberately-built one — see its `CLAUDE.md`/skill for the fullest
detail):

- **Backend**: thin controller (`Http\Controllers\API\*Controller`) +
  business logic in `app\Services\*Service.php`, inline
  `Validator::make()` (no Form Requests), a per-module `*Maintenance`
  middleware checking Spatie permissions per action (not per-route
  `can:` middleware), routes grouped by prefix in `routes/api.php` — all
  POST-only if the module needs to match MRF's convention, or REST verbs
  if it matches KPI's — **match whichever style neighboring modules in the
  same domain already use**, per `vueportal`'s own `CLAUDE.md`.
- **Approval workflows**: reuse `AccessModule`/`AccessChart`/
  `AccessChartUserMap`/`ApproverPerLevel`/`ApprovedLog` (see MRF), not a
  module-specific parallel mechanism (Tactical Requisition's
  `MarketingApproverPerLevel` is the negative example — don't copy it).
- **Permissions**: hyphenated `<module>-<action>`, seeded in
  `database/seeds/PermissionSeeder.php`; a role/permission seeder specific
  to the module (see `ManpowerRequestRoleSeeder.php`) if the module
  introduces its own dedicated roles.
- **Frontend**: one page folder per module under `src/pages/<module>/`,
  one API-wrapper file per resource under `src/services/<module>/`, one
  Zustand store per resource under `src/store/`, one `use<Resource>.js`
  hook wrapping it — see `reactjs-ant-design`'s own `CLAUDE.md` for the
  exact shape.
- Both repos' own `CLAUDE.md` always win over anything summarized here for
  their own conventions — this file is an index across both, not a
  replacement for either.

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
