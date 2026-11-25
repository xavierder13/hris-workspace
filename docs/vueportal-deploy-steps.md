# vueportal — production deploy steps still pending

Read this only when deploying vueportal (or checking what production still
needs). Moved out of `hris-roadmap.md` on 2026-11-19, word for word; the
full old roadmap is in `archive/hris-roadmap-2026-11-19-full.md`. When a step
has run in production, mark it DEPLOYED (or delete it).

## Main list

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
15. **Biometric attendance — devices first (live listener + scheduled pull),
   BioBridge only as fallback, one table** (pushed 2026-10-21: vueportal
   `b1aeddf`, React `a7fca0a`).
   User decisions: `biometric_device_logs` is the one basis for the DTR,
   payroll and attendance screens (from 2026-01-01); the devices are the main
   feed — real-time listener, a scheduled pull of the devices not live
   (interval 3/5/10/15/30/60 min on the page, default 15, Administrator only),
   Pull all devices (background round with progress) and Pull now; BioBridge is a fallback, automatic (every 5 min, only devices
   whose feed has a problem: failed, no OK pull for one interval + 5 min,
   never pulled) and
   manual (Fetch from BioBridge per device), saving a punch only when that
   employee code + second isn't recorded; turning the fallback off is
   Administrator only; role Biometric Device In-Charge (list + pull). Menu:
   Set Up & Authorizations → Biometric Devices. Deploy:
   migrate by path `2026_10_27_100000`, `2026_10_27_100001`,
   `2026_10_27_100002`; `PermissionSeeder`; `composer dump-autoload` +
   `BiometricDeviceInChargeRoleSeeder`; `php artisan biometric:copy-biobridge`
   once (~1.5 min; until it finishes readers use BioBridge directly); the
   `schedule:run` cron (step 7 — the same entry runs the BioBridge copy every
   5 min and the device pull every minute, nothing else to add for auto sync);
   `php artisan biometric:pull-devices` once;
   the listener kept alive by a second Webmin cron job (no Supervisor, nothing
   to install), after a one-device go-live test at a busy hour with BioBridge
   running:
   - Check first in Webmin → Others → Command Shell: `which flock` prints a
     path (util-linux, part of the OS).
   - Webmin → System → Scheduled Cron Jobs → new job, as the web user, every
     minute (all fields **All**):
     `cd /path/to/vueportal && flock -n storage/biometric-listen.lock /usr/bin/php artisan biometric:listen --minutes=60 >> storage/logs/listener.log 2>&1`
   - `flock -n`: while the listener runs the lock is held and the new run
     exits; after a crash the next minute starts it again. `--minutes=60`
     makes it exit hourly so it comes back on the latest code (deploys need
     no restart; to apply a deploy at once, kill the `biometric:listen`
     process and cron restarts it within a minute).
   - A gap while it is down loses nothing: punches stay on the device, it
     stops being live after 3 min and the scheduled pull (then BioBridge)
     catches up.
   The server must reach the devices on TCP 4370 24/7. Steps and config in
   the Claude Doc (its Supervisor section is replaced by this cron job). Live delivery confirmed on Baguio 2026-10-21
   12:48 (a device may send a punch twice — saved once). Open: go-live test
   with BioBridge running (coexistence unconfirmed).
   BioBridge stopped collecting ~09:00 on 2026-10-21 (all branches).
16. **Shift late make-up** (pushed 2026-10-21: vueportal `bacd3a4`, React
   `fb30500`; vueportal
   `ShiftController`, `Shift`, `ScheduleService`, `DtrService`, migration
   `2026_10_27_100003` — run by path; React `ShiftFormModal`, `ShiftIndex`,
   `shiftHelpers`). A shift setting "Required work (break excluded)": a late
   day counts no late / over-break when out − in − break taken reaches it.
   Locked once a shift is assigned → create new shifts with it (e.g. a FLEX
   copy of REG-0800-1700). Found while checking Xavier De Guzman's
   2026-10-A: hand-typed Work Schedules (no shift) carry break 0, so every
   break is over-break — fix by giving employees Work Schedules with a shift.
17. **Withholding tax whole-peso rounding** (pushed 2026-10-21: vueportal
   `95e6549`, React `ba56a93`; no migration). Payroll Settings → General → Statutory deductions → Withholding
   Tax centavos → "Rounded to the whole peso" (`Peso`, tax only). Local
   setting left at Round — choose Peso to match the old payroll, then
   regenerate draft runs (all employees — settings changed).
18. **Edit Employee → Asset Assignment tab** (pushed 2026-10-21: vueportal
   `67d3fc4`, React `2a66522`; React `AssetAssignmentTab.jsx`, `employeeTabItems.jsx`, `EmployeeTabs.jsx`;
   vueportal `CompanyAssetMaintenance`, `CompanyAssetController` index
   `Unassigned` filter, `PermissionSeeder`). Deploy: `PermissionSeeder`
   (new `employee-master-data-asset-assignment`), then grant it to the HR
   roles that should see the tab (none granted yet; Administrator has it).
   No migration.
19. **Shift flexible hours** (pushed 2026-10-22: vueportal `2a88915`, React `7a4b118`; vueportal
   `ShiftController`, `Shift`, `ShiftDay`, `ScheduleService`, `DtrService`,
   migration `2026_10_27_100004` — run by path, migrated locally; React
   `ShiftFormModal`, `ShiftIndex`, `shiftHelpers`, `ShiftChip`,
   `DtrDaysTable`, `timeEntryHelpers`). A shift's "Flexible hours" switch:
   each work day has no time in / out, only a required work (H:mm, break
   excluded, per day — e.g. 8:00 Mon–Fri, 4:00 Sat) from any time in. No late
   (grace 0, no late make-up); short of it = undertime. Locked once assigned,
   like grace. Limit: a flexible day's punches run 04:00 → next day's start,
   so overnight flexible work past 04:00 isn't matched.
20. **Submit Branch Report — own branch only** (pushed 2026-10-22: vueportal
   `bd2db8e`, React `edb474a`; also React `86a331e`, the employee-list
   selection bar). Only Administrator / `employee-acknowledgment-reports-all`
   picks the branch; others get their account's branch read-only, and
   `store()` refuses another branch (422). No migration or seed. Open: the
   permission is held only by Administrator, Employee Master Data
   Administrator and HRIS Administrator — grant it to any other role
   (e.g. Recruitment & Hiring, Payroll Admin) that should pick a branch.
21. **Careers portal certificate fix — DEPLOYED and verified on production
   2026-11-19 (Recruitment pages load again)** (pushed 2026-11-19: vueportal `master` `1c6581f`, same fix on `F-HRIS-Staging` `fde10cb`). Fixes "Connection
   refused for URI https://recruitment.addessa.com/…": the portal's new
   certificate (2026-11-15) isn't trusted by the server's old CA store, so
   vueportal now ships `resources/certs/cacert.pem`. Deploy: pull, then
   `php artisan config:clear` (or `config:cache` if production caches
   config) and `php artisan cache:forget careers_api_token`. **Not**
   `cache:clear`: that logs everyone out. No migration, seed, composer or npm
   step. Check: Recruitment → Applicant List loads. Why: vueportal
   `docs/recruitment-history.md`.

## Payroll processing, payslips, multi-company (from the old phase 10 section)
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

## Phases 11–15, bank accounts, imports, night diff, session expiry (from the old "Continue here" hand-off)
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

## 13th month selected employees / rollback / approval rounds
**Deploy:** after the steps above:
1. Migrate by path, in order:
   - `2026_10_18_100000` (it backfills approved batches as posted).
   - `2026_10_18_110000` (approval rounds; it backfills approved lines as round 1).
2. Run `composer dump-autoload`.
3. Run PermissionSeeder (new: `thirteenth-month-rollback`; grant it to the payroll roles).
4. Run `ApproverRoleSeeder` after the approval procedure seeders.

## Recruitment data gateway (recruitment-portal + vueportal)
**Deploy (either order works):**
1. Portal code.
2. Portal `.env`: `HRIS_GATEWAY_EMAIL=<vueportal's CAREERS_API_EMAIL>`, then `php artisan config:clear`. Copy the value from **production** vueportal's `.env`: any other email makes every HRIS recruitment screen return 401. Leaving it unset is safe (old any-Administrator rule) until it can be checked.
3. vueportal code.

HRIS Branch Managers still need a portal Branch Manager account with the same email (1 of 74 locally lacks one).

## New recruitment process (2026-11-25)

Order: careers portal first, then vueportal, then React, then the exam app.

1. **recruitment-portal**: deploy `9e9abaf` + `61fa334`; `php artisan migrate --path=database/migrations/2026_11_25_100000_add_failed_remarks_to_applicants_table.php`
   and `--path=database/migrations/2026_11_25_110000_create_online_exam_tables.php`; `composer dump-autoload`;
   `php artisan db:seed --class=OnlineExamSeeder`; `.env` `EXAM_APP_URL=https://exam.<domain>`; `php artisan config:clear`.
   Also `2bd14bb` (mail sender): if `.env` has `MAIL_USERNAME=apikey` (SendGrid), set `MAIL_FROM_ADDRESS` to a SendGrid
   verified sender (+ `MAIL_FROM_NAME`) before `config:clear` — else every portal email fails ("[apikey] does not comply
   with RFC 2822"). The SendGrid key committed in `.env.example` is public: replace it and use only the new one.
2. **vueportal**: deploy `301a0ec`, `bc42a96`, `ef454c6`; `php artisan migrate --path=database/migrations/new/2026_11_25_100000_add_asset_type_to_company_assets_table.php`;
   `php artisan db:seed --class=PermissionSeeder`; `php artisan db:seed --class=ManpowerRequestRoleSeeder`.
   Grant `careers-online-exam-send` / `-results` / `-settings` to the recruitment roles (Administrator has them).
3. **React**: deploy (`2552163` … `4db06c3`).
4. **recruitment-exam** → SiteGround subdomain: build with the production `VITE_API_BASE_URL`, upload `dist/` (with `.htaccess`), SSL — its README.
