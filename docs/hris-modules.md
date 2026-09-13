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
