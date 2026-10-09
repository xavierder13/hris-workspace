# HRIS roadmap — next phases up to payroll

The plan agreed with the user, so any session on any device can pick it
up. Keep it current: when a phase ships, mark it done here (one line, the
detail goes in the commit and the module's skill) and update
`hris-modules.md`. Decisions under a phase are the user's — don't
re-ask them. Open questions are listed separately; ask those before
building the phase they affect.

## Where things stand (2026-10-09)

Built and pushed. vueportal is on `F-HRIS-Staging`; reactjs-ant-design and this workspace are on `master`.
- **Leave:** types, applications with approval, balances and credits.
- **Shifts:** shifts, Work Schedule (each version picks a shift), shifting, and Group Shift Allocation.
- **Manual Time Entries:** time in/out plus break out/in.
- **Approvals:** Access Charts and Approving Officers pages, on the shared approval engine, approved the MRF way.
- **Payroll Cut-offs:** periods and the filing switch.
- **Audit Trail:** add, edit and delete on leave, attendance and payroll records.
- **Other:** Holiday Calendar, Hiring Officers, and mobile layout.
- **Payroll records (2026-10-08, second device):** Salary History, Payroll Settings + DOLE premium rates, Allowances, government Contribution tables + employee profiles, scheduled Deductions with a payment ledger, Retro adjustments, and Overtime filing — see phases 1–2, 4, 6–7 and "To fix" below.
- **UI:** app-wide menu search (Ctrl+K), clickable shift chip on Work Schedule, record tabs on My Profile.

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
   - `2026_10_10_120000` retro adjustments
   - `2026_10_11_100000/100100` payroll settings, premium rates
   - `2026_10_11_110000/110100` allowance types, allowances
   - `2026_10_11_120000` overtimes

   Then `composer dump-autoload` and the seeders `PermissionSeeder`,
   `ContributionTableSeeder`, `DeductionTypeSeeder`, `PayrollSettingSeeder`,
   `OvertimeApprovalProcedureSeeder`. Every role / permission seeder now
   uses the guard the Administrator role already has (`web` when there is
   none) — check production's guard before seeding:
   `SELECT guard_name, COUNT(*) FROM roles GROUP BY 1`.
6. **Grant to the payroll / HR roles** (which roles: ask the user):
   `compensation-*`, `payroll-setting-view/-edit`, `allowance-type-*`,
   `allowance-*`, `contribution-table-*`, `contribution-profile-list/-edit`,
   `deduction-type-*`, `deduction-*` (incl. `-cancel`, `-payment`),
   `retro-*`, `overtime-*`. Set the levels and approvers of the new
   "Overtime" Access Chart.

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
approval, audited, hidden from the trail without `allowance-list`. **Not
built from the decision below:** a Weekly basis and Template / Import for
bulk (see "To fix").

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
`DeductionService::dueOn()` for the run. Retro adjustments (manual +
suggested from back-dated Salary History) are built too
(`RetroService`). **Not built:** one-time deductions, Template / Import.

Backend controllers exist with no React pages: `EmployeeLoansController`, `EmployeePremiumsController`. Add recurring and one-time deductions and loan amortization per cut-off.

### 8. Payroll run — BUILT (Payroll → Payroll Runs, skill `payroll-run`)

Built 2026-10-09: Draft (regenerate) → Approved (deduction payments posted,
retros Applied, filing off) | Cancelled; pay register and payslips with how
each line was computed; holiday pay / holiday OT rates from the premium
table. Tested on real BioBridge data for employee 23363 and with holiday
scenarios (monthly / daily-paid, worked / unworked, prior-day rule).
Sample left: 2026-08-B Approved, 2026-09-A Cancelled, 2026-10-A Draft.
The original brief:

Reads: `CompensationService::rateOn()`, Payroll Settings + premium rates
(snapshot them per run), the DTR (phase 5), `AllowanceService::forPeriod()`,
`ContributionService::compute()` split per the deduction schedule,
`DeductionService::dueOn()` (post `Payroll` payments), Open retros of the
cut-off (mark Applied). Daily rate = monthly × 12 ÷ the daily-rate factor.

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
   Settings → Employer and each employee's payroll bank account before the
   first bank file.

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
   - Fixed: a paid leave / time entry / overtime can't be cancelled (vueportal `2202a4a`, React `74f68e9`) — corrections go through Retro.
   - Not applicable on this device: BioBridge (MSSQL) isn't reachable here; payroll tests use manual time entries / imported attendance logs instead.
   - Waiting for the user: freeze a Pending payroll (close filing on submit vs. a "changed since generated" warning); refuse approving a cut-off while an earlier one of the same month is Draft / Pending.
   - Small fixes not yet applied: Administrator approve text ("still needs 1 more"), loans numbering a month's cut-offs by start date vs. contributions by end date, approval error that names no employee when a loan changed after generating.
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
- Payroll runs: 2026-08-A, 08-B, 09-A, 09-B Approved (filing off; 09-B went through the end-to-end test — see `test-results/2026-10-09-payroll-end-to-end.md`, local to this device).
- The "Payroll Run" Access Chart has level 1 (2 required) with no approvers mapped, so only an Administrator can approve here.

**Deploy (production), after step 8 above** — migrations by path:
- `2026_10_14_100000` (group_schedules)
- `2026_10_14_110000` (payroll run rollback)
- `2026_10_14_120000` (attendance_logs)
- `2026_10_14_130000` (payslip posted_at; backfills approved runs)

Then run PermissionSeeder (new: `group-schedule-list/-create/-edit/-cancel`, `payroll-run-rollback`, `attendance-log-template-download`, `attendance-log-import`) and grant those permissions to the HR / payroll roles. Deploy vueportal `3bc1319` (Roles / Permissions guard fix) with it.

## To fix / follow up on the new features (2026-10-08)

Found while building payroll records; none is applied yet.
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
  the viewers (Leave / Time Entry / Overtime / Retro details, Shifting and
  cut-off Filing History, Audit Trail), the Contributions compute modal and
  the contribution table import.
- **No module skills yet** for Contributions, Deductions, Retro,
  Allowances, Payroll Settings, Overtime (both repos) — the api files hold
  the contracts; write skills when the payroll run starts.
- **Template / Import** not built for allowances, deductions and
  contribution profiles (Salary History and contribution tables have one);
  **Weekly** allowance basis and **one-time deductions** not built.
- **Daily-rate employees:** the contribution preview needs the month's
  actual earnings typed in; daily retro counts scheduled work days (not
  attendance) — switch both to the DTR.
- **Daily-paid contributions:** when the month's earlier cut-off has no
  payroll run, the month base uses only this cut-off's earnings (projected).
- **My Profile Attendance tab** stays hidden for employees without
  `employee-master-data-attendance` (its endpoint needs it); add self-access
  if employees should see their own logs.
- **Pre-existing lint errors** in `MainLayout.jsx` (unused `Divider`,
  `Title`).
- **Reports (phases 9–10):** check the bank file layout against the payroll
  bank's own upload format (generic CSV now: account no., name, amount,
  code, bank, credit date); an account number opened in Excel loses leading
  zeros (upload the CSV as-is). Daily-paid 13th month is earned-only (no
  projection). Final pay doesn't mark loans paid or post anything.

### Later (agreed, not scheduled)

- **News posting:** all users view, by permission.
- **Company Asset:** port from vueportal branch `F-Asset-Management` (cherry-pick `6edd681`).
- **KPI template versioning:** see `kpi-template-versioning-plan.md`.

## Open questions to ask the user

- Restore level 2 approval on the Leave and Manual Time Entry access charts? Someone removed it locally and added user "Bhem" at level 1. The level-2 approver mappings are still there, and one leave (#34) and one time entry are still Pending at level 2 — their Approval Route no longer shows that level.
- Should only HR cancel an already-approved leave or time entry?
- Should manual time-entry filing be limited to the filer's subordinates?
- Should leave reasons and remarks stay visible to every `activity-logs` holder in the Audit Trail?
- Hiring Officer eligibility: also Branch Managers / Top Management, or only ADMINISTRATION + Managerial?
- Known bug, fix offered but not applied: `employeeApi.delete` sends `{ids}` while the backend reads `employee_id`.
- Which roles get the payroll permissions (`compensation-*`, `allowance-*`, `deduction-*`, `retro-*`, `contribution-*`, `payroll-setting-*`, `overtime-*`)?
- Production's permission guard: the committed `User::$guard_name` is `"api"`, so roles and permissions must be `api` (`SELECT guard_name, COUNT(*) FROM roles GROUP BY 1` before seeding). This device's DB is all `api`; the other device's local DB was `web` with an uncommitted User edit — convert that DB rather than the code. Fixed 2026-10-09 (vueportal `3bc1319`): the Roles / Permissions pages hard-coded `'web'` on create, so a role made there couldn't get permissions or be assigned; they now use the User model's guard.
- Daily-rate factor: default 313 (6-day week) — confirm 261 / 313 / 365.
- Statutory deduction schedule defaults OK (SSS / PhilHealth / Pag-IBIG on the 2nd cut-off, tax every cut-off)?
- Overtime: minimum minutes, OT before the shift, rounding?
