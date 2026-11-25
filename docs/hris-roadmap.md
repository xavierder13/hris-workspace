# HRIS roadmap — current state

The plan agreed with the user, for any session on any device. **Live items
only**: what's being built, what's open, what waits on the user. Decisions
recorded here or in the files below are the user's — don't re-ask them.
When something ships, update its line here (one line; detail goes in the
commit and the files below) and delete what is no longer open.

| Read when … | File |
|---|---|
| deploying vueportal / checking what production still needs | `docs/vueportal-deploy-steps.md` |
| porting a module to hris-api, or at the cut-over contract check (each phase's contents and **deliberate differences**) | `hris-api/docs/port-history.md` |
| the *why* behind an older vueportal-era decision, its test data or hand-off notes | `docs/archive/hris-roadmap-2026-11-19-full.md` (snapshot, never edited) |
| a module's rules | the owning repo's module skill |

## Where things stand (2026-11-19)

- **vueportal** (`F-HRIS-Staging`) + **reactjs-ant-design** (`master`): the
  HRIS through payroll is built — leave, shifts / schedules, time entries,
  overtime, DTR, payroll runs, 13th month, compliance, clearance, company
  asset, biometric devices, My Workspace. Production still needs the
  deploy steps (`docs/vueportal-deploy-steps.md`); the careers-portal
  certificate fix (2026-11-19) is deployed and verified.
- **hris-api** (Laravel 13 port of vueportal's HRIS backend): phases 0–6c
  built (175 tests), each contract-checked against vueportal. **Next: 6d —
  DTR, My Attendance.** Then 7 Payroll, 8 Separation, 9 Cut-over (list below).
- Repos (check before relying on it): GitHub `xavierder13/hris-api`
  `master`; recruitment-portal `master` has an uncommitted
  `docker-compose.yml` change and vueportal an uncommitted `app/User.php`
  `guard_name` change, both the user's — ask before committing or cleaning.

## hris-api port

User decisions (2026-11-12): **same API as vueportal** (paths, verbs,
payloads, answers — React switches by `VITE_API_BASE_URL` alone), modern
inside (Form Requests, services, Sanctum, spatie `permission:` middleware);
**its own database**, filled by `php artisan hris:copy-legacy` (SELECT-only
on vueportal, upsert by id, re-runnable); **phased**, one module group per
session. Rules and the per-module checklist: `hris-api/CLAUDE.md`.

Phases (detail and deliberate differences: `hris-api/docs/port-history.md`):

| Phase | What | Built |
|---|---|---|
| 0 | Foundation: auth, session, users / roles / permissions, org, PSGC | 2026-11-12 |
| 1 | Audit Trail, approval engine, bell / inbox shells | 2026-11-13 |
| 2a–2d | Employee Master Data (record, tabs, cases, work schedule, reports, dashboard) | 2026-11-13 |
| 3 | Area, Holiday Calendar, Hiring Officers, Company Asset | 2026-11-14 |
| 4a / 4b | Recruitment gateway (+ New Hired sync) / Manpower Request | 2026-11-18 |
| 5 | KPI Management | 2026-11-18 |
| 6a / 6b / 6c | Shifts & default schedules / biometric devices & attendance / leave, time entries, overtime, My Filing | 2026-11-18 / 19 / 19 |
| **6d** | **DTR, My Attendance** — next | |
| 7 | Payroll — settings + assignments, employers, banks / accounts, compensation, allowances, contributions + tables, deductions, adjustments, cut-offs (+ the filing switch), payroll runs, 13th month, reports & compliance, payslips, Payroll Assistant (+ topics seeder). Already started in 6c: `Payroll\PayrollRunService` (paid / locked filings), `PayrollCutoffService::assertFilingOpen`, `PayrollSettingService::dayType`, `PayPremiumRate::DAY_TYPES` | |
| 8 | Separation — Clearance (+ setup), offboarding links, final pay | |
| 9 | Cut-over — full contract check, React `VITE_API_BASE_URL` → hris-api, browser test pass, production compose / deploy plan, vueportal HRIS routes retired (decision for the user then) | |
| 10 | Optional: Recruitment (ATS) inside hris-api — below | |

Open items:
- **Bell items of phases 2–3 not registered** (`open_nte`, `for_regularization`,
  `referral_codes`, the overdue lists …) — add their `NotificationSource`s;
  the Payroll group (`payroll_due`, `pending_filings_due`, `no_salary`,
  `next_year_*`) and `clearance_overdue` come with phases 7–8.
- **Docker image gained `pdo_sqlsrv`** (6b) — rebuild it on every device / server.
- **Production config:** `WEBPORTAL_VUE_ROOT` / `LEGACY_PUBLIC_ROOT`
  (uploads), `CNC_MSSQL_*` + `LEGACY_APP_KEY` (KPI compute), `BIOBRIDGE_DB_*`;
  at cut-over turn vueportal's biometric schedule / listener off and
  `BIOMETRIC_SCHEDULE_ENABLED` + the listener cron on in hris-api.
- **Not verified live** (unreachable from this device): SAP compute, ZKTeco
  devices, BioBridge — covered by tests only.
- Local data: the three `MRF - *` Access Charts have no levels (a new MRF
  can't be submitted until they're set up); check production has them.
- **Decisions for the user:** production deploy target; Training library
  (`training/*`, only vueportal's Vue pages use it) — port or drop.

### Recruitment (ATS) inside hris-api — option (user request 2026-11-19)

Goal: if the user decides on **one backend for everything** (HRIS + ATS) on
one server, port recruitment-portal's backend into hris-api instead of the
gateway (4a's `CareersGateway`). recruitment-portal (read 2026-11-19):
Laravel 7, its own database (51 migrations), ~400 routes; the core is
`ApplicantController` (3,534 lines — the stage flow), plus `job_applicant/*`,
`jobapplicant_file/*`, `job_vacancy/*`, `message/*` (SMS / email),
`public_api/*` (the public careers site, application-link tokens, referral
codes), `recruitment_gateway/*` (what hris-api calls today), its own users /
roles / org tables, an embedded Vue SPA (admin + public application form).
Plan: (1) a Recruitment module in hris-api — the process flow as stage
controllers + a `RecruitmentFlowService`, vacancies, files, messages; tables
migrated and copied from the careers database; (2) `recruitment/*` paths and
answers kept (React unchanged) — the local service replaces `CareersGateway`
behind the same interface, one binding to switch until cut-over; (3) the
public careers endpoints on hris-api, no sign-in, rate-limited.
**Decisions for the user then:** merge the careers site's branches /
positions / users into the HRIS ones or keep copies; who serves the public
application form; before or after payroll; applicant file storage.

## vueportal / React — still open

Before production:
- Run `docs/vueportal-deploy-steps.md` in order.
- **Browser-test** the forms never clicked through: Salary History add / edit
  / delete, the payroll records' add / edit forms, Bank Accounts, Banks,
  Payroll Accounts, Bank File "Paid from", the Generate Payroll picker,
  persistent modals / drawers, menu search, shift chip, My Profile tabs;
  session expiry (React + Vue idle logout, cross-tab sync, /login reason).
- **Verify seeded values** against current rules: SSS / PhilHealth /
  Pag-IBIG / BIR tables, DOLE premium rates, de minimis limits.
- User to confirm the sidebar / navbar redesign (`1447c61`) — if not kept,
  revert only the design (`sidebar.css`, `navbar.css`, the markup in
  `MainLayout.jsx`).
- Biometrics: go-live test with BioBridge running (coexistence unconfirmed).
- Tell payroll: migration `2026_10_15_110000` gives existing settings a
  30-min OT minimum / 15-min rounding (changes OT pay after deploy).

Table column drag / reorder + Columns chooser (`useTableColumns`, React only, localStorage; 2026-11-25,
React `a14f1d9` / `cb09e27`, pushed): on the main list pages and the employee lists — browser-test header drag + refresh.

My Attendance date range (vueportal `MyAttendanceController@show` `date_from`/`date_to`, ≤ 93 days, `c965708` + React
Per cut-off / Date range switch `b7c305d`; 2026-11-25): controller checked in tinker; browser-test it. Port to hris-api.

HRIS user pickers = employees (linked users) + Administrators (`User::employeeOrAdministrator()`; clearance in-charges,
Approving Officers via `access_chart/index?employees_only`; vueportal `e6cb7de`, React `7ca4c5e`; 2026-11-25): tinker-checked;
21 of 29 current approving officers aren't linked — HR to link them. Audit Trail user filter left on every user. Port to hris-api.

New recruitment process (2026-11-25, pushed; deploy steps in `docs/vueportal-deploy-steps.md` "New recruitment process"):
Failed remarks required per stage (portal `9e9abaf`, React `2552163`); referral lists Export (React `c38165d`);
Company Asset type (vueportal `301a0ec`, React `be768ad`); mass upload of approved (Open) MRFs (vueportal `bc42a96`,
React `edcd22d`); online examination — sample MPQ / MDQ, expiring links (hours / days setting), new repo
`recruitment-exam/` (GitHub `xavierder13/recruitment-exam`), portal `61fa334`, vueportal `ef454c6`, React `b3f7163`.
Local DB migrated and OnlineExamSeeder run (2026-11-25); not yet browser-tested end to end. Fixed after: Update Status
locked steps 2–5 for everyone (React `42a6159`), portal emails sent from "apikey" (portal `2bd14bb`; the local SendGrid
key is rejected — new key + verified sender needed before mail can be tested). TEST applicants 31903–31908 (one per
stage, user's email / mobile) are in the local careers DB — delete when done. Exam questions are
edited in `OnlineExamSeeder` only (no editor UI); SMS for exam links not built. Port to hris-api.

To fix / build (not scheduled):
- LOW: the pending-filings bell count isn't user-scoped; imports have no file
  type / size / row cap.
- Daily-rate employees: contribution preview and daily Retro Salary from the
  DTR (now typed earnings / scheduled days); daily-paid month projection
  when the month's earlier cut-off has no run.
- Bank file per bank's own upload format (needs the banks — see questions);
  an account number opened in Excel loses leading zeros. Final Pay posts
  nothing (computation only).
- My Profile Attendance tab hidden without `employee-master-data-attendance`
  — add self-access if employees should see their own logs.
- Salary import's "same employee on line N" numbers rows unlike the error
  list's Excel Row; the Audit Trail action filter can't pick 'imported'.
- Offered, not done: remittance payment log; self-service bank-account change
  request; "rest day + holiday + overtime to 10 pm" payslip simulation.
- Known limits to tell the user: groups / reports / pay sheet subtotals use
  the employee's **current** branch / position; rollback leaves the cut-off's
  filing closed; an unreachable BioBridge is only logged (payroll computed
  without its punches).

Later (agreed, not scheduled): news posting (all users view, by
permission); KPI template versioning (`docs/kpi-template-versioning-plan.md`).

## Open questions to ask the user

- Clearance role grants: who gets `clearance-list` / `-list-all` / `-create`
  / `-edit` / `-cancel` / `-manage`, `-sign`, `-approve`, `-setup`?
- Clearance: hard-block Final Pay until Cleared, or keep the warning (built)?
- Which banks pay payroll (a sample / spec of each bank's bulk-credit upload)?
- Should manual time-entry filing be limited to the filer's subordinates?
- Should leave reasons and remarks stay visible to every `activity-logs`
  holder in the Audit Trail?
- 657 older regularization dates aren't Direct Hire Since + 180 —
  recalculate or keep?
- Grants not given yet: `employee-acknowledgment-reports-all` (pick any
  branch on Submit Branch Report) and `employee-master-data-asset-assignment`
  (the Asset Assignment tab) — which roles?

## References

- User manual: Claude Docs "HRIS Payroll — User Manual",
  https://claude.ai/code/artifact/aa765f8d-62a9-427a-a30b-f53d7c1cbed0 — keep
  current when a payroll feature ships.
- Roles and permissions reference: https://claude.ai/code/artifact/0086b4f9-e9ab-4308-9840-f31765b390af
  — keep current when a role or permission is added.
- Dashboards explanation: https://claude.ai/code/artifact/0fdf02e1-dc82-436d-8340-c5589eeeeeba;
  payroll columns: `test-scenarios/payroll-column-reference.docx/.pdf`.
