# HRIS roadmap — next phases up to payroll

The plan agreed with the user, so any session on any device can pick it
up. Keep it current: when a phase ships, mark it done here (one line, the
detail goes in the commit and the module's skill) and update
`hris-modules.md`. Decisions under a phase are the user's — don't
re-ask them. Open questions are listed separately; ask those before
building the phase they affect.

## Where things stand (2026-10-14)

Built and pushed. vueportal is on `F-HRIS-Staging`; reactjs-ant-design and this workspace are on `master`.
- **Leave:** types, applications with approval, balances and credits.
- **Shifts:** shifts, Work Schedule (each version picks a shift), shifting, and Group Shift Allocation.
- **Manual Time Entries:** time in/out plus break out/in.
- **Approvals:** Access Charts and Approving Officers pages, on the shared approval engine, approved the MRF way.
- **Payroll Cut-offs:** periods and the filing switch.
- **Audit Trail:** add, edit and delete on leave, attendance and payroll records.
- **Other:** Holiday Calendar, Hiring Officers, and mobile layout.
- **Payroll records (2026-10-08, second device):** Salary History, Payroll Settings + DOLE premium rates, Allowances, government Contribution tables + employee profiles, scheduled Deductions with a payment ledger, Payroll Adjustments (then called Retro adjustments), and Overtime filing — see phases 1–2, 4, 6–7 and "To fix" below.
- **UI:** app-wide menu search (Ctrl+K), clickable shift chip on Work Schedule, record tabs on My Profile.
- **2026-10-14 (pushed — vueportal `6ac05b4`…`333fdc1`, React `21f63fa`…`74eb297`):** Payroll Adjustments (renamed from Retro, + Holiday Pay (manual)); **multi-company payroll** — Payroll Settings → Employers (each company's TIN / RDO / SSS / PhilHealth / Pag-IBIG and its own payroll accounts with a Default, general accounts as fallback), Generate payroll / 13th month for All companies or one company, payslips keep their company (employer on payslips and reports; Remittances / Year-end Tax filter by company), each approved payslip saves the accounts it was paid with (Paid From on the payroll page, rounds, list, register), Add Employees blocked from the pay date, employer share hidden from employees' payslips (Payroll Settings → Payslip, default off); Banks → View employees; filings cancel rule (owner / creator, never once paid); hiring officers + Top Management / Branch Managers; Payroll Manager files overtime; KPI Computation Class dropdown and print period fix; breadcrumbs; wider employee pickers / solid toggles; Roles & Permissions table viewer; employee delete fix.
- **2026-10-15 (pushed — vueportal `554abca`…`4626d14`, React `ab71d97`…`e627dc9`):** Administrator-only **Delete Payroll** (any status / period, postings undone) and roll back / regenerate / add employees **after the pay date**; cut-off delete Administrator-only and refused while anything points at it. **BIR semi-monthly / monthly / annual tables** (semi-monthly on each cut-off's own pay with tax "Every cut-off"; annual for year-end and final pay). **Centavo rule per deduction** (PhilHealth Cut = old payroll's; others Round) and an exact monthly split (166.75 + 166.74). **Deductions:** on / off (hold) switch, Paid column, category filter, Specify / Item Bought; a partial cash payment tagged to a cut-off leaves only the rest due (warning on the payslip); new **Installment** type (local data). **Over-break counts as late** (paired break out / in, as the Attendance tab; setting, default on). **Other Allowance** with a Specify description. Local only: payroll cleared and restarted at 2026-09-A (21 / 6 cut-offs); Nancayasan paysheet 2026-09-B rebuilt in run 25 (salaries, NANC-S2…S5 shifts, schedules, one-time loans, Lapeña's punches imported — her BioBridge ID is 25191733, Briones assigned setting) — deductions tally to the centavo, net −991.99 (Templo / Ventura leaves, Manibog half-day rule, Cantorna rate rounding).
- **Payroll bank accounts (2026-10-11):** Banks, employee Bank Accounts by effective date (moved off the Contribution Profile), company Payroll Accounts with a Default / default periods / match-employee-bank rule, chosen per payout and kept on approval, one bank file per paying account. Also: employee picker table for Generate Payroll / Generate Selected, `/recruitment-dashboard`, the antd deprecated-props lint rule, BioBridge 5 s login timeout.

Module rules live in each repo's skills:
- **reactjs-ant-design** `.claude/skills/`: `leave-management`, `shift-management`, `manual-time-entry`, `payroll-cutoff`.
- **vueportal** `CLAUDE.md` for the backend.

### Production deploy still pending (vueportal)

1. **Migrations**, each by path (`php artisan migrate --path=database/migrations/new/<file>`), in this order, skipping any already run:
   - `2026_10_08_100000` hiring_officers
   - `2026_10_08_110000/110100/110200` leave
   - `2026_10_09_100000/100100` shifts and shifting
   - `2026_10_09_100200` rebuild work schedules (production table was empty)
   - `2026_10_09_110000` time entries
   - `2026_10_09_120000` payroll cut-offs
   - `2026_10_09_130000` activity_log.employee_id. This one must run **before** or together with the code, or audited saves fail.
2. **Seeders:**
   - `PermissionSeeder`
   - `LeaveTypeSeeder`
   - `LeaveApprovalProcedureSeeder`
   - `TimeEntryApprovalProcedureSeeder`
   - `ApproversFromMrfSeeder`

   Run `composer dump-autoload` first.
3. **Grant the new permissions to the HR roles:**
   - `leave-*`, `leave-type-*`, `leave-balance-list(-all)`, `leave-credits-edit`
   - `shift-*`, `shift-assignment-*`
   - `time-entry-*`
   - `payroll-cutoff-*`
   - `access-chart-*`
   - `hiring-officer-*`
   - `holiday-calendar-*`

   The Audit Trail uses the existing `activity-logs`.
4. **Create the Hiring Officers** before recruitment uses the picker.
5. **Payroll records** (after the above), migrations by path in this order:
   - `2026_10_09_140000` employee_compensations (Salary History)
   - `2026_10_10_100000/100100/100200` contribution tables, rows, profiles
   - `2026_10_10_110000/110100/110200` deduction types, deductions, payments
   - `2026_10_10_120000` payroll adjustments (`employee_payroll_adjustments`, the whole table in one migration)
   - `2026_10_11_100000/100100` payroll settings, premium rates
   - `2026_10_11_110000/110100` allowance types, allowances
   - `2026_10_11_120000` overtimes
   - `2026_10_19_100000` daily-rate factor to decimal (314.6 / 315 choices). Then pick the factor in Payroll Settings — 314.6 is DOLE's default, the old payroll used 315 (see the payroll manual's "How the daily rate is computed").
   - `2026_10_19_110000` payroll_setting_assignments (Payroll Settings → Assigned Settings).
   - `2026_10_19_120000` night_diff_enabled on payroll_settings (default On) and on assignments.

   Then `composer dump-autoload` and the seeders `PermissionSeeder`,
   `ContributionTableSeeder`, `DeductionTypeSeeder`, `PayrollSettingSeeder`,
   `OvertimeApprovalProcedureSeeder`. Every role / permission seeder now
   uses the guard the Administrator role already has (`web` when there is
   none) — check production's guard before seeding:
   `SELECT guard_name, COUNT(*) FROM roles GROUP BY 1`.
6. **Grant to the payroll / HR roles** — decided 2026-10-14: run
   `PayrollRoleSeeder` (after PermissionSeeder), then give the **Payroll
   Manager** role (compensation, payroll, allowance, deduction, payroll
   adjustment and overtime filing; approvals stay with the approver roles +
   Access Charts) to Grace Mae Hidalgo (`gchidalgo`), Lady Rose Lutrania
   (`lrlobra`) and Marilou Baltazar (`mqbaltazar`) on User Accounts. Done
   locally 2026-10-14. The payroll personnel set the levels and approvers
   of the Leave, Time Entry and new "Overtime" Access Charts.
   Deleting a payroll run (`payroll-run-delete`), rolling back / regenerating
   one after its pay date (`payroll-run-rollback-paid`) and deleting a
   cut-off (`payroll-cutoff-delete`) are the Administrator's alone:
   Migration `2026_10_22_100000` (sss_ / philhealth_ / pagibig_ /
   tax_centavos on payroll_settings — PhilHealth defaults to Cut, the old
   payroll's rule; the others to Round). Migration `2026_10_22_110000`
   (over_break_late on payroll_settings, default on — over-break counts as
   late, the old payroll's rule). Migration `2026_10_22_120000`
   (needs_description on allowance_types, description on
   employee_allowances), then `PayrollSettingSeeder` again (firstOrCreate —
   adds the OTHER "Other Allowance" type with Needs description).
   Payroll Assistant: migration `2026_10_23_100000` (assistant_topics,
   assistant_searches), PermissionSeeder (payroll-assistant, -manage),
   PayrollRoleSeeder, `composer dump-autoload`, then
   `PayrollAssistantTopicSeeder` (26 topics; existing codes kept).
   Tax tables: run `ContributionTableSeeder` again — it skips the existing
   tables and adds the TRAIN 2023 `BIR Semi-monthly` and `BIR Annual` ones
   (Contributions → Tables; check them against the current BIR table).
   PermissionSeeder adds the two new ones; PayrollRoleSeeder no longer gives
   Payroll Manager `payroll-cutoff-delete` (its sync removes it) — check no
   other role holds it.
7. **Daily scheduler** (vueportal `f7f39e8`: `app/Console/Kernel.php`,
   `app/Console/Commands/DeactivateResignedEmployees.php` +
   `RegularizePassedEmployees.php`; deploy with `dc55393`, the rehire /
   Date Resigned fix in `EmployeeMasterDataController` +
   `EmployeeOffboardingController`).
   Deactivate Resigned runs at 00:05 and Regularize Passed at 00:10 (Asia/Manila,
   from `config/app.php`). Both run the same rules as the Sync & Updates buttons.
   No migration or seeder is needed. Steps on the production server:
   - **Deploy the code**, then check that `cd /path/to/vueportal && php artisan list employees`
     shows `employees:deactivate-resigned` and `employees:regularize-passed`.
   - **Find PHP and the web user:** `which php` (with several PHP versions, use the
     site's full path, e.g. `/usr/bin/php7.4`), and `ls -ld /path/to/vueportal/storage`.
     The owner (usually `www-data` or `apache`) is the user the cron job runs as.
   - **Webmin → System → Scheduled Cron Jobs → Create a new scheduled cron job:**
     - Execute cron job as: the web user
     - Active: Yes
     - Command: `cd /path/to/vueportal && /usr/bin/php artisan schedule:run >> /path/to/vueportal/storage/logs/schedule.log 2>&1`
     - When to execute: **Times and dates selected below**, with Minutes, Hours, Days,
       Months and Weekdays all set to **All**. That runs it every minute, and Laravel
       picks the 00:05 / 00:10 times itself. Don't use Webmin's "Daily": it calls
       `schedule:run` once a day and misses those minutes.
   - **Test:** open the job and click **Run Now**. Outside 00:05 and 00:10 it prints
     "No scheduled commands are ready to run.", which is correct. "Permission denied"
     or "php: not found" means the user or the PHP path is wrong. Check the server
     clock with `date`.
   - **First run by hand** (optional), from Webmin → Others → Command Shell:
     `cd /path/to/vueportal && sudo -u www-data php artisan employees:regularize-passed`.
     Before running `employees:deactivate-resigned`, look at production's own
     candidates (the bell's "resigned to deactivate" count uses the same query). The
     local run was 14. Rehires whose only rehire record is a newer Date Employed are
     now kept active.
   - **Next morning:** check `storage/logs/schedule.log` and the Resigned list.
8. **Session Settings** (vueportal `5e32090` + login fix `3742574`; React
   `ff3ca03`, `7f191df`):
   migrate `database/migrations/new/2026_10_24_100000_create_session_settings_table.php`
   by path, deploy the code, `npm run prod` in vueportal (its Vue idle
   timer changed; `idle-vue` is no longer used). No seeder: no row = the
   defaults (60 min idle, 8 h duration). Tokens issued from now on carry a
   24 h expiry (the ceiling); the real limit is the setting. Administrator
   only: React Set Up & Authorizations → Session Settings, Vue Set Up &
   Authorizations → Settings → Session Settings.
9. **Company Asset + Offboarding Clearance** (built 2026-10-19; pushed —
   vueportal `8507291` on `F-HRIS-Staging`, React `39113db`; skills
   `company-asset` in vueportal, `clearance` in reactjs-ant-design). Company
   Asset is the `F-Asset-Management` commit `6edd681` brought onto
   `F-HRIS-Staging`. **Done locally 2026-10-19** (migrations, seeders,
   `npm run prod`). Migrate **by file path**, in this order:
   `2026_10_06_100000_create_asset_categories_table`,
   `2026_10_06_100001_create_company_assets_table`,
   `2026_10_06_100002_create_company_asset_assignment_histories_table`,
   `2026_10_07_100000_add_acquisition_and_disposal_fields_to_company_assets_table`,
   `2026_10_07_100001_create_company_asset_maintenance_logs_table`, then
   `2026_10_25_100000` … `2026_10_25_100015` (clearance tables, the Other
   separation's Specify, checklist categories / items, checkpoints,
   per-clearance in-charge, clearance templates / branches / components /
   component items, template steps). Deploy the code, `composer
   dump-autoload`, `PermissionSeeder` (8 `company-asset-*` + 9
   `clearance-*`: list, list-all, create, edit, cancel, manage, sign,
   approve, setup), `CompanyAssetRoleSeeder` (Asset Custodian role, on the
   Administrator's guard), **`ClearanceModuleSeeder`** (required — the
   "Employee Clearance" module the Access Chart picks, plus the Clearance
   Approver role), `ClearanceSetupSeeder` (checklist categories and two
   templates — Head Office Clearance for ADMINISTRATION, Branch Clearance
   the default — with their components and checklists; adds only what's
   missing), `npm run prod` in vueportal (Vue Asset Management menu), React
   build. Then: Clearance Setup → set the in-charges of the "User"
   components and the standard charges on each template (until then HR,
   `clearance-manage`, clears those components); Approvals → Access Charts →
   a "Clearance" chart (Module: Employee Clearance) with its levels and
   approvers for the final approval (Lady, Malou); grant the `clearance-*`
   permissions to roles (undecided — see open questions).

10. **My Workspace filings** (built 2026-10-19, uncommitted; vueportal
   `MyFilingController` + `my_filing` routes, own-exclusion in the leave /
   time-entry / overtime lists, `PayrollRoleSeeder`; React
   `src/pages/my_workspace/`). No migration. Deploy the code, then take
   `leave-list`, `time-entry-list`, `overtime-list` and `leave-balance-list`
   off the **Employee Self-Service** role (done locally 2026-10-19) (Set Up → Roles, or run `PayrollRoleSeeder` — it
   re-syncs every payroll job role to the seeder's lists, so check for
   hand-made changes first). Approvers keep their *-list /
   leave-balance-list through the approver roles. New view-only role
   **Time & Leave Viewer** (seeder; created locally): everyone's leave, time
   entries, overtime and leave balances, no actions — assign it on User
   Accounts.
11. **Audit trail: Employee Master Data + careers portal** (pushed 2026-10-20:
   vueportal `2a1ad6d`, recruitment-portal `ff0ed19`, React `15f804c`). **Careers portal
   first:** run `database/migrations/2026_10_20_120000_fix_activity_log_id_auto_increment.php`
   by path **before** deploying the portal code — the imported
   `activity_log` table has no AUTO_INCREMENT locally (check production
   with `SHOW CREATE TABLE activity_log`); without it every portal staff
   save fails once logging is on. Then deploy the portal, then vueportal and
   React. No vueportal migration or permission (uses `activity-logs`).
12. **Regularization interviews + computed regularization date + Overdue
   lists** (pushed 2026-10-20: vueportal `6729c03`, `ed83584`; React
   `c7fd2d8`, `25eba97`). Migration by
   path `database/migrations/new/2026_10_26_100000_create_employee_regularization_interviews_table.php`
   **with** the code (copies each employee's single interview into the new
   list). `PermissionSeeder` (new `employee-master-data-evaluation-regularization-delete`)
   and grant it with the -create / -edit holders. Rules: the regularization
   date is always Direct Hire Since + 180 (never typed, Import Data ignores
   its column); the final interview (latest date) decides Passed / Failed.
   New pages Employee → Overdue Deactivation / Overdue Regularization (the
   nightly jobs' lists, run by hand); the bell's To be deactivated / To be
   regularized open them. Open: 657 older regularization dates aren't
   Direct Hire Since + 180 (warning icon) — recalculate or keep (ask).
13. **KPI visibility by branch + HRIS role seeders** (pushed 2026-10-20:
   vueportal `5056e82`, `f6e9b43`; React `aeac8d1`). `composer dump-autoload`, then `PermissionSeeder` (new
   `kpi-evaluation-list-all`), `KpiRoleSeeder` (KPI Evaluator / Approver /
   Evaluation Viewer / Self Evaluation / Administrator — Evaluator and
   Approver get `kpi-evaluation-print`), `LeaveBalanceViewerRoleSeeder`
   (`leave-balance-list`), `EmployeeSelfServiceUserSeeder` (gives Employee
   Self-Service — file / edit / cancel own leave, time entries, overtime —
   and KPI Self Evaluation to every active account linked to an active
   employee; User Accounts assigns both when an employee is linked; rerun
   after linking accounts outside the form) and `HrisRoleSeeder` (the 13 other HRIS roles
   set up by hand: HRIS Administrator, Recruitment, Employees Relation,
   managers, Employee Deactivator, Branch / Department / Position
   Maintenance). The seeders create a
   missing role and only add permissions (never remove). Give KPI
   Evaluation Viewer to whoever sees every evaluation. Rule: an evaluator /
   approver sees and creates for their subordinates in their own branch
   (head office: every branch). Done locally 2026-10-20 (both seeders run
   except HrisRoleSeeder — no change needed locally, tested).
14. **My Workspace only for linked / self-service accounts** (React
   `MainLayout.jsx`, `ProtectedRoute.jsx`, `AppRoutes.jsx` — menu, search and
   the `/my-*`, `/my-evaluations`, `/approvals` routes need `employee_id` or
   the Employee Self-Service role; pushed 2026-10-20: React `15376be`, vueportal `af19896`). Deploy React, then
   run `PayrollAssistantTopicSeeder` again (adds `payslip-not-visible`;
   existing codes kept). No migration. Backend endpoints not gated by this
   rule.

## Pick up here — left to do (saved 2026-10-11)

Everything built so far is pushed (vueportal `333fdc1` on `F-HRIS-Staging`,
React `74eb297`, workspace `master`, 2026-10-14; then night-diff hours: vueportal
`fcc9c92`, React `c5807e8`; then the 2026-10-15 payroll rules: vueportal
`4626d14`, React `e627dc9`). The user manual (Claude Docs, link in
"Continue here") covers the bank accounts, picker and attendance import, and
lists "Not available yet".

### Next up, in order (from local commit `7860da2`, never pushed before this merge — re-check each item against the sections below; some are done)
1. **Remaining review findings (section C):** done 2026-10-20 (see C);
   `/review-code` after.
2. **Decisions from the user** (each unblocks a build): negative net pay —
   warn, block on submit, or carry over; which banks pay payroll (upload
   formats); payroll roles' grants; more than one
   employer company; restore level 2 on the Leave / Time Entry charts
   (open questions at the end).
3. **Browser test pass** on a device with Edge + puppeteer (vueportal
   CLAUDE.md): the 2026-10-11 pages, session expiry, persistent dialogs,
   the sidebar redesign check (section B).
4. **Verify seeded government tables and premium rates** (section B).
5. **Recruitment data gateway (A)** — agreed design; build once the user
   confirms.
6. **Production deploy** — the steps at the top, in order, after 1–4.
7. Then the LOW items and "Other" in section C, and section E.

In order of what the user raised:

### A. Recruitment data gateway — BUILT 2026-10-11 (items 1–4; item 5 still optional)

**Built; committed + pushed 2026-10-13** (recruitment-portal master `4f64588`, vueportal F-HRIS-Staging `5a81807`). Verified 2026-10-13 that the new portal keeps an older vueportal working: a request without `scope` gets the old rule (same data, same 403), and the lock is a no-op while `HRIS_GATEWAY_EMAIL` is unset.

**What it does:**
- **Portal `actAs`:**
  - Without portal permission checks when vueportal sends `scope`.
  - Acts as the matching portal user, else the service account for `scope` all.
  - Refuses a branch scope without a portal account, and refuses a role mismatch.
  - With no `scope` (an older vueportal), the old rule still applies.
- **Portal `RecruitmentMaintenance`:** locked to `HRIS_GATEWAY_EMAIL`. When it is unset, any portal Administrator may call the gateway, as before.
- **vueportal `RecruitmentController`:** sends `scope` from the HRIS role, and `audit()` logs accepted changes in the Audit Trail (new records in `ActivityLogController::RECORDS`).

**Tested locally:**
- Portal rules: 7 checks.
- End to end: Administrator 8,540 applicants; bm@urdaneta 70; bm@sanfernando refused with a message.
- Audit: create, update and delete logged; a failed save not logged.
- Lock: another portal Administrator gets 401.

**Deploy (either order works):**
1. Portal code.
2. Portal `.env`: `HRIS_GATEWAY_EMAIL=<vueportal's CAREERS_API_EMAIL>`, then `php artisan config:clear`. Copy the value from **production** vueportal's `.env`: any other email makes every HRIS recruitment screen return 401. Leaving it unset is safe (old any-Administrator rule) until it can be checked.
3. vueportal code.

HRIS Branch Managers still need a portal Branch Manager account with the same email (1 of 74 locally lacks one).

Original notes:
**Problem:** recruitment access is checked twice — vueportal `careers-*`
permission, then the careers portal again: `RecruitmentGatewayController::actAs()`
(recruitment-portal) looks up `user_email` among the portal's own users and
403s without one ("No careers portal account for …"). So an HRIS user with
no portal account (e.g. xadeguzman@admin.ac) is blocked — also against the
"Administrator can do everything" rule. The portal user only matters for
the second permission check and Branch Manager scoping (`hasRole('Branch
Manager')` + `branch_id` in `ApplicantController`); the portal has no audit
of who did what.

**Agreed recommendation (user asked to build later, after confirming):**
1. vueportal decides permissions; the gateway stops re-checking portal
   permissions. Gateway acts as: the matching portal user when one exists
   (keeps Branch Manager rules) → else the service account (full scope) for
   HRIS users who are not branch-scoped → refused for a branch-scoped HRIS
   user with no portal account. vueportal sends `scope: all | branch` from
   the HRIS role (`Branch Manager` role exists in vueportal).
2. Role mismatch closes, never widens: HRIS says Branch Manager but the
   portal account isn't → refuse with a message; portal account is a
   Branch Manager → branch rules always apply.
3. Lock the gateway to vueportal's service account (dedicated id / role)
   — today the portal's `RecruitmentMaintenance` lets ANY portal
   Administrator token call it with any `user_email` (impersonation).
4. Log recruitment changes in vueportal's activity log (HRIS user, action,
   applicant id) — the Audit Trail.
5. Later, optional: auto-create a portal account when an HRIS user gets a
   Branch Manager recruitment role, or send the branch from vueportal —
   first confirm vueportal and portal branch ids match (not verified).
Files: recruitment-portal `RecruitmentGatewayController::actAs`,
`app/Http/Middleware/RecruitmentMaintenance.php`; vueportal
`RecruitmentController::gatewayRequest` (+ logging). Workaround meanwhile:
create a portal account with the same email and `jobapplicants-*`.

### B. Before production
- Run the deploy steps above (migrations by path in order, then
  PermissionSeeder → PayrollRoleSeeder; payroll roles' grants — ask which
  roles; Access Chart approvers; Payroll Settings → Employer and Payroll
  Accounts; check Banks named by the bank-account move).
- Fix the 2026-10-11 code review findings (section C) before production —
  the HIGH one is fixed; the MEDIUM ones remain.
- Browser-test the forms never clicked through: Salary History add / edit /
  delete, payroll records add / edit forms, and the 2026-10-11 pages (Bank
  Accounts, Banks, Payroll Settings → Payroll Accounts, Bank File "Paid
  from", the Generate Payroll picker) — API-tested only (no headless browser
  on this device). 13th month paid-from: endpoint built, not exercised (no
  13th month run locally). Also check the persistent modals / drawers (React
  `2e2ffd3`: no outside-click / Esc close; the new X on the phone menu and
  phone bell).
- Verify seeded values against current rules: SSS / PhilHealth / Pag-IBIG /
  BIR tables, DOLE premium rates (e.g. Regular Holiday on Rest Day 260% /
  338%, ND 10%), de minimis limits.
- Sidebar / navbar redesign (in `1447c61`): user was to check it — if not
  kept, revert only the design (`src/layouts/sidebar.css`, `navbar.css`
  and the sidebar / header markup in `MainLayout.jsx`), keep the menu
  regrouping and My Workspace.
- Note: a DTR / My Attendance load / payroll generate takes ~20 s on a device
  where BioBridge (MSSQL) isn't reachable (was 60 s; `BIOBRIDGE_DB_LOGIN_TIMEOUT`
  5 s, retried once by Laravel); fine where it is. An unreachable BioBridge
  is only logged — the payroll is computed without its punches, no notice.

### C. Still to fix / build (details in "To fix" and "Continue here" below)
Code review of the 2026-10-09/10 work (2026-10-11):
- Done: HIGH preview data leak fixed (vueportal `5a54298`).
- Done 2026-10-20: the bell keeps each user's summary 60 s (its Refresh
  button rebuilds); Generate Template's Branch / Position / Employees
  options open to the payroll template permissions; the cancel-own race no
  longer applies (status and payroll lock are re-checked inside the locked
  transaction).
- Decided 2026-10-20: My Payslips defaults to the current year; when that
  hides older payslips the empty list names the range and how many are
  outside it.
- LOW — pending-filings bell count not user-scoped (count only); imports
  have no file type / size / row cap. Done 2026-10-20: OT night-diff
  over-credit, My Attendance stale response + hint, "My Evaluations" twice.
  Not a defect: a "Needs Description" type on an old deduction — the edit
  form asks for the description.
- INFO — migration `2026_10_15_110000` gives existing settings a 30-min OT
  minimum / 15-min rounding: changes OT pay after deploy (deliberate — tell
  payroll).

Other:
- Daily-rate employees: contribution preview and daily Retro Salary suggestion from the DTR
  (now typed earnings / scheduled days); daily-paid month projection.
- Bank file vs. the bank's own upload format; Final Pay posts nothing.
- Done 2026-10-20: salary import errors name the Excel row; Audit Trail
  filters Imported (and shows no "#null").
- Done 2026-10-20: a future full-day approved leave shows On Leave in the
  DTR. Remittance payment log (offered).
- A payroll whose net pay is negative can be generated and approved with no
  warning (local 2026-07-B: all 5 negative — no attendance in July, so the
  full month's contributions exceed the little gross). Decide: warn / block
  on submit, or carry the shortfall to the next cut-off.
- Bank-specific upload format: needs the payroll banks' names (user to give);
  the bank file is now one CSV per paying account, so each bank gets its own
  layout.
- Self-service request to change one's own bank account (HR enters it now).
- Bulk import of manual time entries does not exist — the attendance log
  import (Timekeeping → Attendance Template / Import Attendance) is the bulk
  route for a biometric outage; no approval there.
- Offered, not run: simulate "rest day + holiday + overtime to 10 pm" in a
  rolled-back payroll calc to show the payslip lines.

- Session expiry: browser-test the React and Vue idle logout (30 min — or
  lower `IDLE_TIMEOUT_MS` / `IDLE_TIMEOUT_MINUTES` locally), the cross-tab
  sync (one busy tab keeps the other alive) and the /login reason; tested
  over HTTP only (backend: unseeded / passive / extend / logout).
- Done: master merged into `F-HRIS-Staging` (vueportal `e189161`, HRIS
  unchanged — see the deploy note at the top); `F-Auth-Session` needs no
  merge (its session commit is in, the rest is master).

### D. Device housekeeping (the device with ~/projects)
The older clones `~/projects/vueportal` / `~/projects/reactjs-ant-design`
were wrongly synced on 2026-10-10 (the working repos are inside this
workspace). User's edits are safe in `stash@{0}` of each. To restore:
`cd ~/projects/vueportal && git checkout master && git branch -f F-HRIS-Staging a48ecda && git stash pop`;
`cd ~/projects/reactjs-ant-design && git reset --keep b26672f && git stash pop`.
Also (seen 2026-10-11, not touched): `recruitment-portal` (in this
workspace) is 5 commits ahead of its remote (merge commits + `ded9964`
"upload gitignore files"), has an uncommitted `docker-compose.yml` change
(+2 lines) and a `stash@{0}` — the user's; ask before pushing or cleaning.

### E. Later (agreed, not scheduled) and open questions
See "Later" (news posting, Company Asset port, KPI template versioning) and
"Open questions to ask the user" at the end — incl. (2026-10-11) more than
one employer company? (Payroll Settings and the payroll accounts are one
company now) and which banks pay payroll.

## Next phases, in order

Every phase follows the same pattern as Leave and Manual Time Entries:
- vueportal: Service plus thin controller and a `<Module>Maintenance` middleware, with an Administrator bypass; migrations go in `database/migrations/new`.
- React: a page, an api file and a skill.
- `AuditsActivity` on every new HR record, so the Audit Trail covers it.
- Bulk data updates go through Generate Template, then fill, then Import, not scripts.
- Commit one feature per repo, and test as the user would before committing.

### 1. Salary / compensation history — BUILT (skill `compensation`)

Pushed 2026-10-08 after lint, build, backend transaction tests and HTTP
import tests. The bulk import (Generate Template → fill by employee code →
Import Data, from Salary History or Employee Master Data) passed a browser
workflow test on 2026-10-09: added, unknown code and bad change type
rejected, all or nothing. **Not yet done:** a full code review and a
browser test of the Add / Edit / Delete forms — run them before
production. Production: migrate `2026_10_09_140000` by path,
run PermissionSeeder, grant `compensation-*` to the payroll / HR roles
(which roles: ask the user).

Record management of each employee's pay, kept as versions by effective date.

Decided:
- **No approval.** A change is saved directly by permission.
- **Effective dates.** Every change is a new version with an effective date, a reason and who made it. Past versions are never edited away; that is the history.
- **History by permission.** Salary and history are visible only by permission, e.g. `compensation-list`, `-create`, `-edit`.
- **Audited, with amounts hidden.** Amounts are hidden in the Audit Trail for users without the compensation permission. Leave reasons are not hidden today.
- **Bulk updates** (e.g. a salary adjustment) use template and import.

To design: pay basis (monthly or daily rate), and the rate on a given date for payroll.

### 2. Allowances — BUILT (allowance types + employee allowances)

Built: types (taxable / de minimis with limit per period, seeded with the
common PH ones), employee allowances per cut-off / per month / per day
worked, by effective dates (a Specific Period = a from–to range); no
approval, audited, hidden from the trail without `allowance-list`. Per
week (× 52 ÷ 12 a month, split like per month) and Template / Import
(2026-10-10: a new effective_from adds the allowance and ends the same type
running then; same employee + type + effective_from updates it) are built
too.

Allowance types, plus employee allowances with effective dates and history. Separated by frequency:
- **Daily:** paid per day worked, taken from DTR days.
- **Weekly / Monthly:** a recurring fixed amount.
- **Specific Period:** a from–to date range only, e.g. relieving or a project.

Decided: no approval, history kept, visible by permission, audited. Template and import for bulk.

### 3. Configurable cut-off generator — BUILT (skill `payroll-run`)

Payroll Settings → Cut-offs & Pay Days: the two start days (1 / 16, or
26 / 11 …), the pay day of each (0 = last day), and a pay day on a Sunday /
holiday moved to the previous / next working day. Generate Year reads them
and can re-apply pay dates to existing cut-offs. Local 2026 cut-offs were
re-dated (they paid 5 days after the end).

### 4. Overtime filing — BUILT

Built on the Manual Time Entry pattern (`OvertimeService` on
`ApprovalProcedure`, "Overtime" Access Chart, separate React pages that
reuse the time-entry helpers and `ApprovalSteps`); filed before or after
the day; hours with overnight and unpaid break; day type from
`PayrollSettingService::dayType()`; only Approved overtime is paid. User
decisions 2026-10-08: OT must be approved to be paid. **Still to design /
not built:** minimum OT, OT before the shift, rounding, and copying the MRF
approvers onto the Overtime chart (`ApproversFromMrfSeeder` does it for
Leave / Time Entry only).

An OT application. **User decision (2026-10-08): overtime filing is the
same process as Manual Time Entries — reuse the time-entry components
(index, form modal with the day's schedule + biometric punches, details
modal with ApprovalSteps, helpers) and its approval procedure and
approving officers (same Access Chart approvers), rather than building a
new flow.** Details:
- **Approval:** the shared approval engine with an Access Chart for "Overtime". Level 1 by subordinates (position_subs), the required approval count, the MRF managerial approvers added, and visibility by subordinates, approver and permission. One action per approver, and no self-approval.
- **Cut-off switch:** blocked by the cut-off filing switch (`PayrollCutoffService::assertFilingOpen`).
- **Fields:** date, OT start and end, hours, reason, and the type of day (regular, rest day, holiday), derived from `ScheduleService` and the Holiday Calendar.
- **Preview:** the day's schedule and the BioBridge punches, like the time-entry preview, so the approver sees the actual out time.
- **Rules:** permissions `overtime-list(-all)/-create/-edit/-approve/-cancel`, and audited.
- **To design:** minimum OT, whether OT before the shift counts, and rounding.

### 5. DTR / timekeeping per cut-off — BUILT (Payroll → Timekeeping, skill `payroll-run`)

Built 2026-10-09 as `DtrService` (computed live, copied onto each payroll
run): punches by shift window (night shifts), manual time entries,
leave, overtime, holidays by branch; a holiday is never an absence;
rest-day work pays through approved overtime; dates from today on are
Upcoming. The original brief:

User decision (2026-10-08): the attendance basis is the **Attendance tab
(biometric logs) combined with approved leave, overtime and manual time
in / out**. Use `PayrollSettingService::dayType()`, the premium rates and
the night-differential window from Payroll Settings, and
`OvertimeService::approvedFor()`.

Per employee per day in a cut-off, putting together:
- the schedule (Work Schedule shift, or shifting if one is active),
- the biometric punches,
- approved leave, manual time entries and OT,
- holidays.

Result: late, undertime, absences, OT hours and holiday or rest-day work, using the shift grace minutes. The user requires **a snapshot of the shift and schedule history used when payroll is generated** (shifting exists mainly for payroll). Lock it when filing is switched off.

### 6. Government contributions and tax tables — BUILT

SSS, PhilHealth, Pag-IBIG and BIR monthly tables versioned by effective
date (seeded: SSS 2025, PhilHealth 5% 2024, Pag-IBIG Feb 2024, BIR TRAIN
2023 — **verify**), employee profiles (Computed / Fixed / Exempt per
agency, Pag-IBIG voluntary top-up, minimum wage earner) and
`ContributionService::compute()` (monthly EE / ER / EC and withholding
tax). When each is deducted is a Payroll Settings choice — user decision
2026-10-08: **dynamic, every cut-off or once a month** (seeded: SSS /
PhilHealth / Pag-IBIG on the 2nd cut-off, tax every cut-off).

A renewed table can be imported (Contribution Tables → Import Version):
the agency's Excel template comes pre-filled with its latest version; the
upload goes with an effective date and is previewed before it is saved — a
new date adds a version, the date of a saved version replaces its brackets
(needs `contribution-table-edit` as well as `-create`). Each bracket is
compared with the version it follows or replaces (new / changed with the
old value / removed). No new permission or migration.

### 7. Deductions and loans — BUILT (scheduled deductions)

New tables, not the legacy `employee_loans` / `employee_premiums` (those
are imported reports keyed by name, not employee id). Deduction types,
scheduled deductions (total, per cut-off, start cut-off, every / 1st /
2nd cut-off of the month), a payment ledger (manual now, `Payroll` posted
by the run), hold / resume / cancel, Fully Paid when the balance is zero;
`DeductionService::dueOn()` for the run. Payroll adjustments (Payroll →
Payroll Adjustments; manual + Retro Salary suggested from back-dated Salary
History) are built too (`PayrollAdjustmentService`; renamed from Retro
Adjustments 2026-10-14 — table, model, routes `payroll_adjustment/*`,
permissions `payroll-adjustment-*`, React `/payroll-adjustments`). Built 2026-10-10: One-time deductions (the whole total
on the start cut-off, carried until taken) and Template / Import (one line
per deduction; same type + reference no. updates it; a line without one
that matches a saved one's type, total and start cut-off is refused).
An Other Adjustment names what it is for (Holiday Pay (manual), OT /
holiday / night diff underpayment, allowance under- / overpayment, refund
of over-deduction, Others (specify)), which fixes earning / deduction and
whether it is taxed. Holiday Pay (manual) covers a holiday the employee's
own branch doesn't observe (e.g. field work in a branch that does). A deduction type marked Needs Description
(OTHER by default) requires a description on each deduction, shown on the
payslip as "Other Deduction — …"; the import template has a description
column.

Backend controllers exist with no React pages: `EmployeeLoansController`, `EmployeePremiumsController`. Add recurring and one-time deductions and loan amortization per cut-off.

### 8. Payroll run — BUILT (Payroll → Payroll Runs, skill `payroll-run`)

Built 2026-10-09: Draft (regenerate) → Approved (deduction payments posted,
payroll adjustments Applied, filing off) | Cancelled; pay register and payslips with how
each line was computed; holiday pay / holiday OT rates from the premium
table. Tested on real BioBridge data for employee 23363 and with holiday
scenarios (monthly / daily-paid, worked / unworked, prior-day rule).
Sample left: 2026-08-B Approved, 2026-09-A Cancelled, 2026-10-A Draft.
The original brief:

Reads: `CompensationService::rateOn()`, Payroll Settings + premium rates
(snapshot them per run), the DTR (phase 5), `AllowanceService::forPeriod()`,
`ContributionService::compute()` split per the deduction schedule,
`DeductionService::dueOn()` (post `Payroll` payments), Open payroll
adjustments of the cut-off (mark Applied). Daily rate = monthly × 12 ÷ the daily-rate factor.

Per cut-off: gross pay (rate on date, DTR, OT, holiday pay, allowances), minus contributions, tax, deductions and loans, giving net pay. Then review, approve or lock, with no edits after the lock except by an adjustment.

### 9. Payslips and reports — BUILT (skill `payroll-run`)

Built 2026-10-09: printable payslip (Payroll Run → payslip → Print), My
Payslips (user menu, own approved payslips only), Payroll Register (Excel,
with a By Branch sheet) and Bank File (preview + CSV once approved) on the
payroll run page, payroll bank account on the Contribution Profile,
employer details in Payroll Settings.

### 10. Compliance and year-end — BUILT (Payroll → Reports & Compliance)

Built 2026-10-09: Remittances (month — SSS / PhilHealth / Pag-IBIG EE + ER
+ EC and BIR 1601-C per employee, Excel workbook), 13th Month Pay (PD 851,
Draft → Submit → the "13th Month Pay" Access Chart, same rules as the
payroll run → Approved | back to Draft; Cancel a Draft; register and bank
file), Year-end Tax (annualization from the BIR monthly table × 12,
₱90k benefits exemption, alphalist Excel, printable BIR 2316 data), Final
Pay (unpaid days per cut-off, pro-rated 13th month, SIL / VL conversion,
this month's contributions, loan balances, tax true-up; printable with the
2316 — a computation only, posts nothing). The 2316 / alphalist are data
layouts to transfer onto the BIR forms / eAlphalist, not the official forms.
Local: the 2026 13th month was approved from the page (before the chart existed).

7. **Payroll processing** (2026-10-09), migrations by path:
   `2026_10_12_100000` (cut-off rules on payroll_settings),
   `2026_10_12_110000/110100` (payroll runs). Then `composer dump-autoload`,
   PermissionSeeder (`dtr-list`, `payroll-run-list/-generate/-approve/-cancel`
   — grant to the payroll roles), set Payroll Settings → Cut-offs & Pay Days,
   enter the year's holidays in the Holiday Calendar (local 2026 holidays are
   samples), then Generate Year with "update pay dates".
   Payroll approval: migrate `2026_10_12_120000`, run
   `PayrollRunApprovalProcedureSeeder`, then on Access Charts set "Payroll
   Run" levels / required approvals / approvers and give them the "Payroll
   Approver" role (local: level 1, 2 approvals, Lady Rose Lutrania + Marilou
   Baltazar). Includes a fix to AccessChartController (adding a level to an
   existing chart crashed: the new row has no id).
8. **Payslips, reports and year-end** (2026-10-09), migrations by path:
   `2026_10_13_100000` (bank account on contribution profiles),
   `2026_10_13_110000` (13th-month runs / pays), `2026_10_13_120000`
   (employer details on payroll_settings), `2026_10_13_130000` (13th-month
   approval). Then `composer dump-autoload`, PermissionSeeder,
   `ThirteenthMonthApprovalProcedureSeeder` (the "13th Month Pay" Access
   Chart, copying the Payroll Run chart's levels / approvers when new; the
   "Payroll Approver" role gets thirteenth-month-list / -approve), and grant
   `payroll-report-view`, `final-pay-view`, `thirteenth-month-list /
   -generate / -approve / -cancel` to the payroll roles (My Payslips needs no
   permission — the account must be linked to its employee). Fill Payroll
   Settings → Employers (each company) and each employee's payroll bank
   account before the first bank file.
9. **Multi-company payroll** (2026-10-14), migrations by path, after 8:
   `2026_10_20_100000` (employer details on `companies`; copies the Payroll
   Settings employer to the company of the same name), `2026_10_20_110000`
   (`company_id` on payroll_runs / thirteenth_month_runs — null = all
   companies — and on payroll_run_employees / thirteenth_month_pays,
   backfilled from each employee's current branch's company),
   `2026_10_20_120000` (`company_id` on company_bank_accounts — existing
   accounts stay general), `2026_10_20_130000` (`bank_credit` on payslips /
   13th-month lines — saved on approval; payrolls approved before it show
   their kept accounts, worked out live; the local ones were backfilled),
   `2026_10_20_140000` (`payslip_show_employer_share` on payroll_settings,
   default off — the employer share hidden from employees' payslips). No new permission. Then fill Payroll Settings →
   Employers for every company that files on its own (TIN, RDO, SSS /
   PhilHealth / Pag-IBIG numbers) and add each company's payroll accounts
   there (expand the company; mark its Default) — a company without one is
   paid from the General tab's accounts.

## Payroll requests 2026-10-09 (phases 11–15)

These were requested on 2026-10-09. Some of the request already existed, so it is not rebuilt:
- **Allowance taxable / non-taxable:** the Allowance Type's Taxable switch and its de minimis limit.
- **Salary import with an effective date:** Salary History → Generate Template / Import.
- **Holiday and holiday OT rates:** Payroll Settings → Premium Rates, one row per day type.

Decisions (all taken from the user's answers):

### 11. Default shift / work schedule per company, branch and position — BUILT (Time & Leave → Schedule → Default Schedules)
Default rules, not a bulk copy:
- A company, branch or position gets a default shift (pattern) or work schedule.
- Every employee follows it automatically, including new hires and transfers, unless they have their own assignment.
- The most specific rule wins: the employee's own shift assignment, then the employee's work schedule, then the position default, then the branch default, then the company default, otherwise No Schedule.
- Both the DTR and the Shift Management views read the resolved schedule.

### 12. Payroll rollback / partial regenerate — BUILT (payroll run page: Generate Selected, Roll Back to Draft)
- **Draft:** regenerate only the employees you select, or all of them.
- **Approved:** roll it back to Draft. This needs a reason and its own permission, and it is refused while an approved 13th month for that year exists. After a rollback the payroll goes through approval again.
- Both actions are logged.

### 13. Attendance log import — BUILT (Timekeeping → Attendance Template / Import Attendance)
- Time-in/out rows are imported (Generate Template → Import) into an HRIS table.
- The DTR merges them with the BioBridge punches. A date that has imported punches uses them instead of that date's BioBridge punches; other dates keep BioBridge.
- Re-importing an employee's day replaces that day's imported rows.
- This covers any employee or branch without BioBridge.

### 14. Contribution history per employee — BUILT (Reports & Compliance → Contribution History; Contributions → History)
- For a date range: SSS, PhilHealth and Pag-IBIG (EE / ER / EC) and tax, per employee per cut-off, with totals.
- An Excel report.
- Approved payrolls only.

### 15. Pay sheet and payslips by date range — BUILT (Reports & Compliance → Pay Sheet, Print Payslips)
- **Pay sheet:** a date or cut-off range of approved payrolls, per employee, with subtotals by branch, company and position. Shown on screen and as Excel.
- **Batch payslip printing:** for a range and filter.

### Continue here (next session, 2026-10-09 hand-off)

Phases 11–15 are committed and pushed: vueportal `F-HRIS-Staging`, reactjs-ant-design `master`.

**Tested:**
- 11–13: backend in tinker (rolled-back transactions) and HTTP; UI in headless Edge.
- 14–15: backend over HTTP, plus both Excel downloads.

Test records were deleted. The test token was revoked.

**Done 2026-10-09 (this device):** per-employee rollback React screens
(lock tag, Roll Back Selected / All, approved rows out of Generate
Selected, All / Selected employees on Generate Payroll), the Contribution
History and Pay Sheet pages, the Contributions History modal, routes / menu,
and the payroll-run skill. Browser-tested in headless Chrome (15 checks) and
over HTTP (partial rollback, approved row refused for generate / cancel).
Deploy: migrate `2026_10_14_130000` after `110000`.

**Still to do:**
1. End-to-end review defects (2026-10-09):
   - Fixed: a leave / time entry / overtime paid by an approved payroll, or inside a payroll waiting for approval, can't be cancelled except by an Administrator (user decision 2026-10-09: Administrator can do everything); paid ones are corrected through Retro. Submitting a payroll turns its cut-off's filing off (user decision: the period is locked while waiting for approval); a disapproval turns it back on only if the submit did. Also fixed: Administrator approve text, loans numbering a month's cut-offs like contributions, and the approval error naming the employee when a loan changed. vueportal `2202a4a`, `c4ba0e8`; React `74f68e9`, `16d2c70`.
   - Not applicable on this device: BioBridge (MSSQL) isn't reachable here; payroll tests use manual time entries / imported attendance logs.
   - Done (user decision 2026-10-09): a month's payrolls are approved in order — submit / approve refused while an earlier cut-off of the month is Draft / Pending; generating refused while a later one is Pending / Approved (vueportal `22a7dbe`).
   - User decision 2026-10-09: salary / pay visibility stays permission-based (payroll-run-list / payroll-report-view show pay).
   - Payroll Run / 13th Month approvers: Lady Rose Lutrania + Marilou Baltazar (mapped at level 1, 2 required) now have the "Payroll Approver" role on this device — give it to them in production too.
   - Done 2026-10-09 (user decision): leave days without credit are filed and unpaid (deducted like an absence), not refused; a full-day leave and a time entry / overtime can't share a day (half-day leave can). vueportal `1d3426a`, React `f7d8bfa`.
   - Done: `PayrollRoleSeeder` — Payroll Officer, Payroll Manager, Timekeeper, Employee Self-Service (view only), plus the four approver roles.
   - Done 2026-10-09/10 (user decision): own vs. others permissions — `leave-/time-entry-/overtime-create-own / -edit-own / -cancel-own` (own employee record only, cancel while Pending) beside the existing any-employee ones; `*-list` = own + filed + to approve, `*-list-all` = everyone. Employee Self-Service gets list + the -own ones. vueportal `dbcd323`, React `f89e949`.
   - Done: Payroll Officer given to Lady Rose Lutrania, Marilou Baltazar, Grace Mae Hidalgo (this device). Note: Lady Rose and Marilou are also the only Payroll Run approvers (2 required) — if one of them submits, she can't approve her own, so let Grace generate and submit.
   - Decided by Claude (user delegated, 2026-10-10): daily-rate factor stays 313; SSS / PhilHealth / Pag-IBIG on the 2nd cut-off, tax every cut-off; overtime minimum 30 min and rounded down per 15 min (Payroll Settings → Overtime, migration `2026_10_15_110000`; vueportal `f5322a4`, React `dd10845`); pre-shift overtime counts when approved.
   - Done 2026-10-10 (vueportal `4b91fa0`/`506a31b`/`5f19b4d`, React `b9c1650`/`b0fa928`/`1447c61`): payroll record imports + Per week + One-time + deduction descriptions + retro Adjustment For; notification bell covers every approval (leave, time entry, overtime, payroll run, 13th month) and payroll deadlines (rule in both CLAUDE.md); self-service My Workspace (My Attendance = own DTR with Bio / Imported / Manual tags, My Payslips range print / Excel pay sheet), filing pages open on All for own-only filers, profile tab 401 fixed; menu regrouped by process, sidebar / navbar redesign.
   - User manual (simple steps, setup → reports; updated 2026-10-11 with bank accounts, the Generate Payroll picker, attendance import for a biometric outage, Other tools and Not available yet; then with Add Employees / approval rounds, rollback before the pay date, 13th month selected employees): Claude Docs "HRIS Payroll — User Manual", https://claude.ai/code/artifact/aa765f8d-62a9-427a-a30b-f53d7c1cbed0 (private until shared). Keep it current when a payroll feature ships.
   - Roles and permissions reference (every HRIS role: area, purpose, permission count, seeder; every HRIS permission by module with the screen it unlocks; how clearance signing works; seeders and their re-run behaviour; open grants; 2026-10-19): Claude Docs "HRIS Roles and Permissions", https://claude.ai/code/artifact/0086b4f9-e9ab-4308-9840-f31765b390af (private until shared). Keep it current when a role or permission is added.
   - Dashboards explanation (every section of the Workforce, Recruitment and KPI Dashboards — what it shows and how each number is computed; 2026-10-11): Claude Docs "HRIS Dashboards — Features and Computations", https://claude.ai/code/artifact/0fdf02e1-dc82-436d-8340-c5589eeeeeba (private until shared). Payroll column explanations: test-scenarios/payroll-column-reference.docx/.pdf. Keep both current when a dashboard section or payroll view changes.
2. **Known limits to tell the user:**
   - Groups, reports and pay sheet subtotals use the employee's **current** branch / position (payslips keep no branch).
   - Rollback leaves the cut-off's filing closed.
   - The Audit Trail now also lists Payroll Run, Default Schedule and Attendance Log Import records.
3. **Fixes / checks found:**
   - Local run 17 (2026-10-B, Draft) was made by someone else — leave it, or ask.
   - The Salary import's "same employee on line N" message numbers rows differently from the error list's Excel Row (pre-existing; the attendance import uses the Excel row).
   - The ActivityLogController description filter only allows created / updated / deleted, so the 'imported' entries can't be filtered by action.
4. **Older offers still open:** remittance payment log; future approved leave shows as Upcoming instead of On Leave in the DTR.

**Local test data (this device, 2026-10-09)** — for payroll testing, all marked TEST:
- Xavier De Guzman (2191): 41 approved manual time entries, Aug–Sep 2026 work days (reason "TEST data — …"); 5 approved overtimes (08/12, 08/26, 09/10, 09/19 rest day, 09/23 night diff).
- His setup: allowances Rice ₱2,000 / month, Transpo ₱1,500 / cut-off, Meal ₱100 / day worked (from 08/01); deductions TEST-SSS-SL-001 (₱1,000 every cut-off) and TEST-HDMF-MPL-001 (₱500 on the 2nd cut-off); contribution profile with a TEST bank account and ₱200 Pag-IBIG voluntary; Payroll Settings employer = "TEST Employer Corp.".
- Lock tests left: 2026-10-A run Cancelled, TEST-CA-LOCK deduction Cancelled, 10/05 time entry Cancelled, OT 09/10 cancelled by an Administrator (still paid in 09-A).
- Information System sample (2026-10-10): Calimlim (2410), Nagtalon (2588), Miranda (2176), Velasco (2681) — salary (2026 rate ₱30k / 28k / 22k / 21k), Xavier's schedule, his 3 allowances and 2 loans (refs `TEST-…-<code>`, from 10-A), contribution profile, bank account (TEST Bank `000000<employee id>`), approved manual time entries every October weekday (incl. future dates to 10/31) and 4 approved overtimes (10/07, 10/14, 10/17 rest day, 10/21 night diff). All marked TEST.
- Bank accounts (migrated here 2026-10-10): banks TESTBANK (moved from the profiles) and BDO (test); Calimlim → BDO from 2026-10-20, Nagtalon → BDO from 2026-11-01 (import test); Payroll Settings → Payroll Accounts: BDO 0011-2233-4455 (Default), BPI 1100-2200-3300 (default period 2026-11-01 – 11-30; bank BPI added for it), TESTBANK 7777-0000-1111 (for the matching rule) — all TEST. A test 2026-10-A run (#10) was approved and then rolled back and cancelled.
- Payroll runs: 2026-08-A, 08-B, 09-A, 09-B Approved (filing off; 09-B went through the end-to-end test — see `test-results/2026-10-09-payroll-end-to-end.md`, local to this device).
- The "Payroll Run" Access Chart has level 1 (2 required) with no approvers mapped, so only an Administrator can approve here.

**Deploy (production), after step 8 above** — migrations by path:
- `2026_10_14_100000` (group_schedules)
- `2026_10_14_110000` (payroll run rollback)
- `2026_10_14_120000` (attendance_logs)
- `2026_10_14_130000` (payslip posted_at; backfills approved runs)

Also migrate `2026_10_15_100000` (leave unpaid_days) and `2026_10_15_110000` (overtime rules), run PermissionSeeder (new `*-create-own / -edit-own / -cancel-own`) then `PayrollRoleSeeder`, and give "Payroll Approver" to the Payroll Run approvers.

Then run PermissionSeeder (new: `group-schedule-list/-create/-edit/-cancel`, `payroll-run-rollback`, `attendance-log-template-download`, `attendance-log-import`) and grant those permissions to the HR / payroll roles. Deploy vueportal `3bc1319` (Roles / Permissions guard fix) with it.

Payroll adjustment "Adjustment For" columns are part of
`2026_10_10_120000` now (the old `2026_10_16_100000` was merged into it and
deleted). Migrate `2026_10_16_110000`
(deduction_types.needs_description — turned on for code OTHER — and
employee_deductions.description).

master merged into F-HRIS-Staging (2026-10-11, vueportal `e189161`; HRIS
kept as on staging — 744 routes unchanged, 90 HRIS API responses
byte-identical before / after): at deploy also
- `composer install` (master adds `simplesoftwareio/simple-qrcode`, IT
  Asset) and `npm install` + `npm run prod` (master's Vue screens);
- production `.env`: `WEBPORTAL_VUE_ROOT=/media/webportal/backup1/webportal-vue`
  — the upload disk (MRF, employee files, NTE / disciplinary, offboarding,
  credit) is now env-driven; unset = public/, so without it production
  uploads would go to public/;
- master's pending migrations (credit remarks / invoice details,
  promodizer sales invoices, IT asset tables if not yet there) — by path,
  as master's own deploy does; not run locally (5 pending in `new/`).

Session expiry (2026-10-11, vueportal `18b5b94` cherry-picked from
`F-Auth-Session` + `72c5b6c` + `27e7395`; React `d122d41`): no migration.
Every token issued before the deploy has no idle window, so **everyone is
logged out once** on their next request (401 "expired due to inactivity") —
announce it. Run `npm run prod` for the Vue assets (Home.vue / app.js).
`php artisan cache:clear` also ends every session (the windows live in the
cache). Tokens now expire 8 h after login; idle 30 min.

Bank accounts (2026-10-10): migrate `2026_10_17_100000` (banks), `100100`
(employee_bank_accounts), `100200` (copies each Contribution Profile's bank
account into them, from 2000-01-01 — the old columns stay, unused), `100300`
(company_bank_accounts + payroll_settings.match_employee_bank), `100400`
(paid-from on payroll_runs / thirteenth_month_runs), in that order. Then PermissionSeeder
then PayrollRoleSeeder (new `bank-list/-create/-edit/-delete`,
`bank-account-list/-create/-edit/-delete/-template-download/-import`), add the company accounts on
Payroll Settings → Payroll Accounts (mark the Default; default periods for
planned switches; the match-employee-bank switch), and check Payroll → Setup → Banks (the
move names banks after the typed bank names). Optional `.env`
`BIOBRIDGE_DB_LOGIN_TIMEOUT` (default 5 s).

Payroll record imports (2026-10-10, no migration): run PermissionSeeder then
PayrollRoleSeeder — new `allowance-template-download / -import`,
`deduction-template-download / -import`,
`contribution-profile-template-download / -import` (Payroll Officer and
Manager get them).

Night-differential hours per assigned setting (2026-10-14, vueportal
`fcc9c92`, React `c5807e8`; user chose a
per-employee / group window with coverage, no rate override): migrate
`2026_10_21_100000` (night_diff_from / night_diff_to on
payroll_setting_assignments). No permission change. Payroll Settings →
Assigned Settings → "Night diff. from / to"; must cover 10 PM – 6 AM.

### 13th month / payroll: selected employees, rollback, paid lock, approval rounds (pushed 2026-10-11)

Commits: vueportal `435f7be` (ApproverRoleSeeder) + `a5b56db` on `F-HRIS-Staging`; React `6c23679` (badge padding) + `0a6fe5e`; workspace `f552359`. The Claude Docs user manual (step 4, after step 5, 13th month, Rules, Not available yet) and the testing manual 1.1 are updated.

Built in vueportal + reactjs-ant-design; backend tests (rolled-back
transactions) and a read-only headless Edge check passed.
- **Generate 13th Month:** All employees or Selected employees, like Payroll Runs (`thirteenth_month/candidates`). The Draft's Regenerate opens the same modal.
- **Rollback:**
  - Approved employees are posted (`thirteenth_month_pays.posted_at`) and locked.
  - Roll Back Selected / All (`thirteenth-month-rollback`, reason) returns the batch to Draft and through approval again.
  - Cancel is refused while any lines are still posted.
  - The year-end tax and final pay read posted lines only.
- **Paid lock:**
  - The 13th month can't be rolled back on or after its pay date (December 24 when no pay date is set).
  - Payroll run rollback is refused on or after the cut-off's pay date.
  - Payroll rollback is also refused while the year's 13th month is Pending or has posted lines.
- **ApproverRoleSeeder:** gives each approval chart's approvers their approver role and its permissions. Covers Leave, Time Entry, Overtime, Payroll Run + 13th Month, and the MRF charts. It only adds, so run it again whenever approvers change.
- **Column reference:** `test-scenarios/payroll-column-reference.docx/.pdf`, the companion to the testing manual.

- **Add employees to an approved payroll / 13th month (approval rounds):**
  - Generate on an approved cut-off or year, or the record's Add Employees button, lists only employees not in it.
  - The record goes back to Draft. The final approval releases the added employees as round 2, 3 …; earlier rounds stay locked and unchanged.
  - Pages: an Approval Rounds panel and a round filter, which the Register and Bank File follow (a round's CSV keeps its own paid-from account).
  - Remove Selected takes not-yet-approved lines out of a Draft; when only approved lines are left, the record is Approved again.
  - Tested: 40+ backend checks (rolled-back transactions, including the added employee's tax / basic and the 13th-month formula) and a browser round trip.
  - The local 13th-month batch's `generated_at` was touched by the browser test; nothing else changed.

**Deploy:** after the steps above:
1. Migrate by path, in order:
   - `2026_10_18_100000` (it backfills approved batches as posted).
   - `2026_10_18_110000` (approval rounds; it backfills approved lines as round 1).
2. Run `composer dump-autoload`.
3. Run PermissionSeeder (new: `thirteenth-month-rollback`; grant it to the payroll roles).
4. Run `ApproverRoleSeeder` after the approval procedure seeders.

## To fix / follow up on the new features (2026-10-08)

Found while building payroll records; none is applied yet.
- Done 2026-10-20 (setting off = no change): holiday prior-day rule — an
  unpaid holiday passes the absence on (successive holidays), and a holiday
  before the cut-off's first work day looks back into the previous cut-off;
  the night-diff snapshot compare (a run saved before the night-diff hours
  matches; `fcc9c92` can deploy); Default Schedules "Applies to" as solid
  buttons.
- **Verify seeded values** before payroll uses them: SSS / PhilHealth /
  Pag-IBIG / BIR tables (`ContributionTableSeeder`), DOLE premium rates
  (`PayrollSettingSeeder`) and the de minimis limits (RR 11-2018 values —
  check the latest BIR regulation).
- Done 2026-10-09: local 2026 cut-off pay dates re-applied (phase 3); the
  Overtime chart got Manual Time Entry's 16 officers (same levels) and the
  "Overtime Approver" role; the DTR reclassifies overtime day types; the
  payroll run snapshots Payroll Settings and premium rates.
- **Not tested in a browser:** menu search, shift chip, My Profile record
  tabs, and the payroll pages' add / edit forms. Run `/review-code` and
  `/test-workflow` on them before production. Browser-tested 2026-10-09:
  the viewers (Leave / Time Entry / Overtime / Payroll Adjustment details, Shifting and
  cut-off Filing History, Audit Trail), the Contributions compute modal and
  the contribution table import.
- **No module skills yet** for Contributions, Deductions, Payroll Adjustments,
  Allowances, Payroll Settings, Overtime (both repos) — the api files hold
  the contracts; write skills when the payroll run starts.
- Done 2026-10-10: Template / Import for allowances, deductions and
  contribution profiles; Per week allowances; One-time deductions. Not yet
  code-reviewed (`/review-code`).
- **Daily-rate employees:** the contribution preview needs the month's
  actual earnings typed in; daily Retro Salary suggestions count scheduled work days (not
  attendance) — switch both to the DTR.
- **Daily-paid contributions:** when the month's earlier cut-off has no
  payroll run, the month base uses only this cut-off's earnings (projected).
- **My Profile Attendance tab** stays hidden for employees without
  `employee-master-data-attendance` (its endpoint needs it); add self-access
  if employees should see their own logs.
- **Reports (phases 9–10):** check the bank file layout against the payroll
  bank's own upload format (generic CSV now: account no., name, amount,
  code, bank, credit date); an account number opened in Excel loses leading
  zeros (upload the CSV as-is). Daily-paid 13th month is earned-only (no
  projection). Final pay doesn't mark loans paid or post anything.

### Later (agreed, not scheduled)

- **News posting:** all users view, by permission.
- **Company Asset:** backend brought onto `F-HRIS-Staging` 2026-10-19 (deploy step 9; Clearance lists the assets still assigned). React page built 2026-10-19 (`/company-assets`, pushed `3701ba9`); Assets tab on the employee profile built 2026-10-20.
- **KPI template versioning:** see `kpi-template-versioning-plan.md`.

## Open questions to ask the user

- Clearance role grants: who gets `clearance-list` / `-list-all` / `-create` / `-edit` / `-cancel` / `-manage` (HR), `-sign` (in-charges who sign a component, and every manager who is an immediate head), `-approve` (final approvers; the seeded Clearance Approver role has list + approve), `-setup`?
- Built and pushed 2026-10-20: My Clearance in My Workspace (read-only own progress; vueportal `bdcdfa9`, React `9f5144b`) and the Approvals inbox (`/approvals`, a tab per module, counts match the bell; vueportal `81f2295`, React `82862a6`). No migrations.
- ~~Departing employee sees their own clearance~~ — built 2026-10-20: My Workspace → Clearance (`my_clearance`, read-only).
- Clearance: hard-block Final Pay until Cleared, or keep the warning (built)?
- ~~Approval levels~~ — decided 2026-10-14: the payroll personnel set the number of levels on the access charts. (Locally leave #34 and one time entry are still Pending at a removed level 2.)
- ~~Who cancels an approved filing~~ — decided and built 2026-10-14 (uncommitted): HR (`*-cancel`, e.g. the Timekeeper role), the owner or the creator (`*-cancel-own` + own record or `filed_by`) and Administrator, Pending or Approved, only while no payroll has paid it or holds it — Administrator included (the old Administrator bypass is gone). Applies to leave, time entry and overtime.
- Should manual time-entry filing be limited to the filer's subordinates?
- Should leave reasons and remarks stay visible to every `activity-logs` holder in the Audit Trail?
- ~~Hiring Officer eligibility~~ — decided and built 2026-10-14 (uncommitted): ADMINISTRATION Managerial, Top Management, or Branch Manager of any branch (`HiringOfficer::RULES`).
- ~~More than one employer company~~ — decided and built 2026-10-14 (uncommitted): Generate (payroll and 13th month) offers **All companies** or **one company**, one generation and one approval each; a cut-off / year has one All-companies run or runs per company, never both; an employee is in one run of a cut-off; each payslip keeps its company (the employer on payslips and reports); Payroll Settings → Employers per company; Remittances / Year-end Tax / range reports filter by the payslip's company. Next: the bank file CSV per bank (needs each bank's format).
- ~~Negative net pay~~ — decided 2026-10-14: the payroll user settles it with a payroll adjustment. Already in place: scheduled deductions that would go below zero are skipped (still due) with a warning; a negative net warns (wording now points to a payroll adjustment).
- Which banks pay payroll? The Bank File is a generic CSV today (name, account no., amount per company account); each bank's online-banking bulk-credit upload has its own layout — need a sample / spec per bank.
- ~~`employeeApi.delete` bug~~ — fixed 2026-10-14 (uncommitted): one `{ employee_id }` request per employee (an array cleared related records of the first id only).
