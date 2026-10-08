# HRIS modules registry

This is a **living document**, not a one-time snapshot. It exists so that
when a new HRIS feature/module comes up, there's a single place that says
what already exists (so new work reuses existing patterns/endpoints instead
of duplicating them) and what counts as "HRIS" at all in this workspace.

**Keep it current-state:** when an HRIS module is added or gains a
frontend, edit its row in place (same shape as existing rows). Module
detail belongs in the owning repository's skill, not here. Re-verify
anything load-bearing against the code before relying on it.

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
| Employee Master Data | `EmployeeMasterDataController`, prefix `employee_master_data` (incl. `my_profile` — the signed-in user's record via `users.employee_id`) | `src/pages/employee_master_data/`, `src/services/employee/`, `employeeStore.js`; Employee Profile (`profile/EmployeeProfile.jsx`, ports `EmployeeProfile2.vue`) at `/employees/:id` and, for linked accounts, `/user/profile` — see the `employee-master-data` skill in `reactjs-ant-design` |
| — Key Performance | `EmployeeKeyPerformanceController`, prefix `employee_master_data/key_performance` | Performance Management tab |
| — Classroom Performance Rating | `EmployeeClassroomPerformanceRatingController`, prefix `employee_master_data/classroom_performance_rating` | Performance Management tab |
| — OJT Performance Rating | `EmployeeOjtPerformanceRatingController`, prefix `employee_master_data/ojt_performance_rating` | Performance Management tab |
| — Branch Assignment Position | `EmployeeBranchAssignmentPositionController`, prefix `employee_master_data/branch_assignment_position` | Performance Management tab |
| — Merit History | `EmployeeMeritHistoryController`, prefix `employee_master_data/merit_history` | Performance Management tab |
| — Training (employee's own history) | `EmployeeTrainingController`, prefix `employee_master_data/training` | Performance Management tab |
| — NTE (Notice to Explain) | `EmployeeNTEController`, prefix `employee_master_data/nte` | Disciplinary tab |
| — Disciplinary | `EmployeeDisciplinaryController`, prefix `employee_master_data/disciplinary` | Disciplinary tab |
| — Offboarding | `EmployeeOffboardingController`, prefix `employee_master_data/offboarding` | Offboarding tab (authoritative offboarding source) |
| — Work Schedule | `EmployeeWorkScheduleController`, prefix `employee_master_data/work_schedule` (model `EmployeeWorkSchedule`) | Work Schedule tab |
| — Acknowledgment Report | `EmployeeAcknowledgmentReportController`, prefix `employee_master_data/acknowledgment_reports` | `src/pages/employee_master_data/acknowledgment_report/` |
| Recruitment | `RecruitmentController`, prefix `recruitment` — **external "careers" portal integrated over HTTP** (token auth via `getOrCreateToken()`), not a locally-owned dataset. `new_hired()` pulls newly-hired applicants from that external API; hires land in `EmployeeMasterData` (tracked via `EmployeeNewHiredSyncLog` to avoid re-syncing). `vacancies()` already computes Required Plantilla vs. Existing Headcount (reused by MRF's headcount snapshot — see below) | ATS applicant lists per stage: `src/pages/recruitment/JobApplicantList.jsx` + `applicants/`; Recruitment Setup (careers portal Positions / Ranks / Branches / Job Vacancies, via `RecruitmentController@setup` → gateway `setup`; plus Hiring Officers — HRIS-own `hiring_officers` table of Employee Master Data records (active ADMINISTRATION, Managerial rank), `HiringOfficerController`, the Hiring Officer Name options of the status/hiring-details form): `src/pages/recruitment/setup/` — see the `recruitment-ats` skill in `reactjs-ant-design` |
| Manpower Request (MRF) | `ManpowerRequestController` + `ManpowerRequestService`, prefix `manpower_request` | `src/pages/manpower_request/`, `src/services/manpower_request/manpowerRequestApi.js`, `manpowerRequestStore.js`, `useManpowerRequests.js` — incl. Excel reports (`export` hiring report, Approved only; `export_status` per-line status report) and per-type approval procedures — one Access Chart per MRF type (`MRF - Replacement` / `MRF - Additional` / `MRF - New Position`, `access_for` = Manpower Request; maintained on vueportal's existing Access Chart screen; required approvals now keyed by `approver_per_levels.access_chart_id`; one request type per MRF; Additional/New Position level 1 filtered by the `position_subs` hierarchy, walked through other level-1 approvers' positions); see the `manpower-request` skill in each repository |
| KPI Management | `KpiAuthController`, `KpiTemplateController`, `KpiTemplateItemController`, `KpiEvaluationController`, `KpiMyEvaluationController`, `KpiBehaviorCriteriaController`, `KpiSettingController`, `KpiReportController`, `KpiEmployeeController`, prefix `kpi` | `src/pages/kpi/{templates,evaluations,my-evaluations}`, `src/services/kpi/{kpiTemplateApi,kpiEvaluationApi}.js`, `kpiTemplateStore.js`, `kpiEvaluationStore.js` |
| Area Assignment | `AreaController` + `AreaService`, prefix `area` (models `Area`/`AreaBranch`/`AreaHrHead`) — areas = groups of branches (a branch in ≤ 1 area), HR heads assigned per employee | `src/pages/area/`, `src/services/area/areaApi.js`, `areaStore.js`, `useAreas.js` — see the `record-management` skill in each repository |
| Employee Loans | `EmployeeLoansController`, prefix `employee_loans` | none yet |
| Employee Premiums | `EmployeePremiumsController`, prefix `employee_premiums` | none yet |
| Employee Attendance Log | `EmployeeAttlogController`, prefix `employee_attlog` | none yet |
| Leave Management | `LeaveTypeController` (prefix `leave_type`), `EmployeeLeaveController` + `LeaveService` (prefix `leave`) — leave types, applications (HR files, HR approves), balances and credits; days counted from Work Schedule rest days and the Holiday Calendar | `src/pages/leave/` (menu Time & Leave) — see the `leave-management` skill in reactjs-ant-design |
| Shifts & Shifting | `ShiftController` (prefix `shift`), `ShiftAssignmentController` + `ShiftAssignmentService` (prefix `shift_assignment`), `ScheduleService` (schedule in force on a date: temporary shifting, else the EMD Work Schedule) — no approval, revision history per change | `src/pages/shift/` (Time & Leave → Shifting, Setup → Shifts) — see the `shift-management` skill in reactjs-ant-design |
| Manual Time Entries | `EmployeeTimeEntryController` + `TimeEntryService` (prefix `time_entry`) — time-in / out for days the biometric device missed; day's schedule + BioBridge punches shown; MRF-style approval ("Manual Time Entry" Access Chart via `ApprovalProcedure`) | `src/pages/time_entry/` (Time & Leave → Attendance) — see the `manual-time-entry` skill in reactjs-ant-design |
| Approvals (Access Charts) | `AccessChartController`, `AccessChartUserMapController` (prefixes `access_chart`, `access_chart_user_map`) — the shared approval procedures (levels, approvals needed, approving officers) used by MRF, Leave, Manual Time Entries and older modules | `src/pages/approval/` (Set Up → Approvals: Access Charts, Approving Officers) — see the `record-management` skill in reactjs-ant-design |
| Payroll Cut-offs | `PayrollCutoffController` + `PayrollCutoffService` (prefix `payroll_cutoff`) — payroll periods and the filing switch (OFF blocks leave / manual time entry filing, editing and approving inside the period; logged) | `src/pages/payroll_cutoff/` (Time & Leave → Setup → Payroll Cut-offs) — see the `payroll-cutoff` skill in reactjs-ant-design |
| Holiday Calendar | `HolidayCalendarController`, prefix `holiday_calendar` (`holiday_calendars` + `holiday_calendar_branches`; `import`/`template/download` routes have no controller methods) | `src/pages/record_management/holiday_calendar/` (`/holiday-calendar`, calendar + list views) — see the `record-management` skill in reactjs-ant-design |
| Training File Library (by position, not per-employee) | `TrainingController`, prefix `training` (uploads/permissions, position-scoped) | none yet |
| Legacy Employee module | `EmployeeController` (model `App\Employee`, import/export) — **appears superseded by Employee Master Data**; confirm before building anything new on it | none |

### Tier 2 — HR-owned org/reference structure (used broadly, not HR-exclusive)

Branch (`BranchController`, `branch`), Company (`CompanyController`,
`company`), Department (`DepartmentController`, `department`), Division
(`DivisionController`, `division`), Rank (`RankController`, `rank`),
Position (`PositionController`, `position`), Address
(`AddressController`, `address`). Frontend: `branchStore.js`,
`departmentStore.js`, `positionStore.js` + matching hooks — consumed as
dropdown/reference data by MRF and other forms. CRUD pages for Company,
Branch, Department, Position, Rank and Promodizer Brand live in
`src/pages/record_management/` (menu Set Up → Organization); no Division or
Address page yet — see the `record-management` skill in reactjs-ant-design.

### Tier 3 — Cross-cutting platform infrastructure (not HR-specific at all)

Auth (`AuthController`, `auth`), User/Role/Permission
(`UserController`/`RoleController`/`PermissionController`, prefixes
`user`/`role`/`permission`), the shared multi-level approval engine
(`AccessModuleController`/`AccessChartController`/`AccessChartUserMapController`,
prefixes `access_module`/`access_chart`/`access_chart_user_map` — used by
MRF and also by non-HR modules like Marketing Event), Activity Log
(`ActivityLogController`, `activity_logs`), Email
(`EmailController`, `email`). Frontend: `authStore.js`, `useAuth.js`,
`src/pages/{user,role,permission}/` (Role & Permission pages are built; see
`reactjs-ant-design/CLAUDE.md`). **The platform `Administrator` role can do
every action in every module** — every gate needs an Administrator bypass
on both sides.

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
- **Admin CRUD / master-data modules**: start from each repository's
  `record-management` skill (Area Assignment is the reference
  implementation).
- Both repos' own `CLAUDE.md` always win over anything summarized here for
  their own conventions — this file is an index across both, not a
  replacement for either.
