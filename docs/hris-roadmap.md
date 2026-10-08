# HRIS roadmap — next phases up to payroll

The plan agreed with the user, so any session on any device can pick it
up. Keep it current: when a phase ships, mark it done here (one line, the
detail goes in the commit and the module's skill) and update
`hris-modules.md`. Decisions under a phase are the user's — don't
re-ask them. Open questions are listed separately; ask those before
building the phase they affect.

## Where things stand (2026-10-08)

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
import tests. **Not yet done:** a full code review and a browser workflow
test (`/review-code`, `/test-workflow` on the compensation files) — run
them before production. Production: migrate `2026_10_09_140000` by path,
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

### 3. Configurable cut-off generator

Payroll is semi-monthly, paid on the **15th and the end of the month**. The user wants it **dynamic**: configurable period start and end days and pay days, rather than the fixed 1–15 / 16–end that "Generate Year" makes today. Probably a stored setting the generator reads. Keep the filing switch as it is.

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

### 5. DTR / timekeeping per cut-off — NEXT

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

### 8. Payroll run

Reads: `CompensationService::rateOn()`, Payroll Settings + premium rates
(snapshot them per run), the DTR (phase 5), `AllowanceService::forPeriod()`,
`ContributionService::compute()` split per the deduction schedule,
`DeductionService::dueOn()` (post `Payroll` payments), Open retros of the
cut-off (mark Applied). Daily rate = monthly × 12 ÷ the daily-rate factor.

Per cut-off: gross pay (rate on date, DTR, OT, holiday pay, allowances), minus contributions, tax, deductions and loans, giving net pay. Then review, approve or lock, with no edits after the lock except by an adjustment.

### 9. Payslips and reports

Payslips that employees can view, a payroll register, bank or remittance files, and government reports.

### 10. Compliance and year-end (agreed 2026-10-08, after 9)

Monthly remittances (SSS R-3 / contribution list, PhilHealth RF-1,
Pag-IBIG MCRF, BIR 1601-C), 13th-month pay (by Dec 24), tax annualization,
BIR 2316 and alphalist, final pay with leave conversion (within 30 days).

## To fix / follow up on the new features (2026-10-08)

Found while building payroll records; none is applied yet.
- **Verify seeded values** before payroll uses them: SSS / PhilHealth /
  Pag-IBIG / BIR tables (`ContributionTableSeeder`), DOLE premium rates
  (`PayrollSettingSeeder`) and the de minimis limits (RR 11-2018 values —
  check the latest BIR regulation).
- **Sample cut-offs pay dates:** the 2026 cut-offs generated locally pay 5
  days after the period end; the user pays on the 15th and the end of the
  month — fix with the configurable generator (phase 3).
- **Overtime chart has no approvers** locally (and `ApproversFromMrfSeeder`
  found no "MRF - Additional" chart, so Leave / Time Entry charts have no
  levels either) — anyone with `*-approve` decides in one step until set.
- **Not tested in a browser:** menu search, shift chip, My Profile record
  tabs, every payroll page, Overtime. Run `/review-code` and
  `/test-workflow` on them before production.
- **No module skills yet** for Contributions, Deductions, Retro,
  Allowances, Payroll Settings, Overtime (both repos) — the api files hold
  the contracts; write skills when the payroll run starts.
- **Template / Import** not built for allowances, deductions,
  contributions (Salary History has one); **Weekly** allowance basis and
  **one-time deductions** not built.
- **Daily-rate employees:** the contribution preview needs the month's
  actual earnings typed in; daily retro counts scheduled work days (not
  attendance) — the DTR should replace both.
- **Overtime `day_type`** is stored when filed; the DTR must reclassify
  (schedule / holidays can change).
- **Payroll Settings and premium rates are not effective-dated** — the
  payroll run must snapshot them per run.
- **My Profile Attendance tab** stays hidden for employees without
  `employee-master-data-attendance` (its endpoint needs it); add self-access
  if employees should see their own logs.
- **Pre-existing lint errors** in `MainLayout.jsx` (unused `Divider`,
  `Title`).

### Later (agreed, not scheduled)

- **News posting:** all users view, by permission.
- **Company Asset:** port from vueportal branch `F-Asset-Management` (cherry-pick `6edd681`).
- **KPI template versioning:** see `kpi-template-versioning-plan.md`.

## Open questions to ask the user

- Restore level 2 approval on the Leave and Manual Time Entry access charts? Someone removed it locally and added user "Bhem" at level 1.
- Should only HR cancel an already-approved leave or time entry?
- Should manual time-entry filing be limited to the filer's subordinates?
- Should leave reasons and remarks stay visible to every `activity-logs` holder in the Audit Trail?
- Hiring Officer eligibility: also Branch Managers / Top Management, or only ADMINISTRATION + Managerial?
- Known bug, fix offered but not applied: `employeeApi.delete` sends `{ids}` while the backend reads `employee_id`.
- Which roles get the payroll permissions (`compensation-*`, `allowance-*`, `deduction-*`, `retro-*`, `contribution-*`, `payroll-setting-*`, `overtime-*`)?
- Production's permission guard (`web` or `api`)? Local is `api` (matches `User::$guard_name`).
- Daily-rate factor: default 313 (6-day week) — confirm 261 / 313 / 365.
- Statutory deduction schedule defaults OK (SSS / PhilHealth / Pag-IBIG on the 2nd cut-off, tax every cut-off)?
- Overtime: minimum minutes, OT before the shift, rounding?
