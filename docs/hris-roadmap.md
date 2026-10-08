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

### 2. Allowances

Allowance types, plus employee allowances with effective dates and history. Separated by frequency:
- **Daily:** paid per day worked, taken from DTR days.
- **Weekly / Monthly:** a recurring fixed amount.
- **Specific Period:** a from–to date range only, e.g. relieving or a project.

Decided: no approval, history kept, visible by permission, audited. Template and import for bulk.

### 3. Configurable cut-off generator

Payroll is semi-monthly, paid on the **15th and the end of the month**. The user wants it **dynamic**: configurable period start and end days and pay days, rather than the fixed 1–15 / 16–end that "Generate Year" makes today. Probably a stored setting the generator reads. Keep the filing switch as it is.

### 4. Overtime filing

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

### 5. DTR / timekeeping per cut-off

Per employee per day in a cut-off, putting together:
- the schedule (Work Schedule shift, or shifting if one is active),
- the biometric punches,
- approved leave, manual time entries and OT,
- holidays.

Result: late, undertime, absences, OT hours and holiday or rest-day work, using the shift grace minutes. The user requires **a snapshot of the shift and schedule history used when payroll is generated** (shifting exists mainly for payroll). Lock it when filing is switched off.

### 6. Government contributions and tax tables

SSS, PhilHealth, Pag-IBIG and BIR withholding tables, versioned by effective date. Philippines is assumed from the timezone and business; confirm before building.

### 7. Deductions and loans

Backend controllers exist with no React pages: `EmployeeLoansController`, `EmployeePremiumsController`. Add recurring and one-time deductions and loan amortization per cut-off.

### 8. Payroll run

Per cut-off: gross pay (rate on date, DTR, OT, holiday pay, allowances), minus contributions, tax, deductions and loans, giving net pay. Then review, approve or lock, with no edits after the lock except by an adjustment.

### 9. Payslips and reports

Payslips that employees can view, a payroll register, bank or remittance files, and government reports.

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
