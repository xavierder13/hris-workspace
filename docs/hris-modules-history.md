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
