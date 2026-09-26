# Repository map

This file is a **cache** of what `repository-discovery` found, written so
later sessions don't have to re-derive it from scratch. It is not a source
of truth — a specific claim in here should be re-verified against the
actual repository before being relied on for anything consequential,
especially if this file's "Last generated" date is old or a repository has
been worked on since.

---

## vueportal/

- **Detected role(s)**: Laravel API backend — evidence: `composer.json`
  (`laravel/framework`), `artisan`, `app/Http/Controllers/API/`. Not
  HRIS-only — a shared backend for several unrelated business domains
  (inventory, credit/collections, marketing, SAP integration, online
  banking, SMS blast, sales) bundled into one app. See
  `docs/hris-modules.md` for the "HRIS = the subset of vueportal that
  reactjs-ant-design consumes" framing.
- **Framework / version**: Laravel ^7.0, PHP ^7.2.5 (do not use newer
  syntax). Auth: `laravel/passport` ~9.0 (`auth:api` guard). Authorization:
  `spatie/laravel-permission` ^4.0.
- **CLAUDE.md**: `vueportal/CLAUDE.md` — repo-wide conventions (flat
  `app/` model layout, Service-layer pattern for newer modules, inline
  `Validator::make()` not Form Requests, per-module `<Module>Maintenance`
  middleware, response envelope `{success, message?, <resource_key>}`),
  plus a "Modules with their own skill" index. Module detail lives in the
  skills; dated history in `docs/<module>-history.md`.
- **Project-specific `.claude/skills/`**: `code-review`, `contract-review`,
  `feature-development`, `live-testing`, `test-evidence`, and `manpower-request` — the
  canonical, actively-maintained reference for the MRF module (file
  locations, models, routes, approval-workflow state machine, validation,
  permissions, and a "Module status" section tracking built/deferred/next
  — read it before touching MRF backend code). Dated change history lives
  in the repo's `docs/manpower-request-history.md`.
- **Project-specific `.claude/agents/`**: `code-reviewer`, `contract-reviewer`, `feature-tester`, `mrf-backend-tester`.
- **Project-specific `.claude/commands/`**: `add-feature`, `review-code`, `test-feature`, `test-mrf-backend`, `validate-contract`.
- **Key directories for integration work**:
  - Routes: `routes/api.php`, grouped by module prefix (e.g.
    `manpower_request`, ~line 2293).
  - Controllers: `app/Http/Controllers/API/*Controller.php`.
  - Service layer (newer modules only): `app/Services/*Service.php`.
  - Models: flat in `app/` (no subfolders).
  - Authorization middleware: `app/Http/Middleware/*Maintenance.php`.
  - Migrations: `database/migrations/`.
  - Permissions/roles seeders: `database/seeds/PermissionSeeder.php` +
    per-module role seeders (e.g. `ManpowerRequestRoleSeeder.php`).

## reactjs-ant-design/

- **Detected role(s)**: React SPA, specifically **the HRIS frontend** —
  evidence: `package.json` (`react`, `antd`), `src/pages/` containing
  only HR-shaped modules (auth, dashboard, employee_master_data, kpi,
  manpower_request, recruitment, permission, role, user).
- **Framework / version**: React 19 + Vite, Ant Design v6, react-router-dom
  v7, Zustand v5 (state), axios (HTTP), dayjs (dates). No test framework
  configured.
- **CLAUDE.md**: `reactjs-ant-design/CLAUDE.md` — tech stack, routing
  conventions (`AppRoutes.jsx` + `MainLayout.jsx` both required for a page
  to be reachable), Zustand store shape, API/service conventions (one
  `<name>Api.js` file per resource, all through the shared
  `axiosInstance`), Docker lint/build commands, a "Modules with their own
  skill" index, and a short Role & Permission entry. Dated history lives
  in `docs/<module>-history.md`.
- **Project-specific `.claude/skills/`**: `manpower-request` and
  `employee-master-data` (current-state module references), plus
  `code-review`, `feature-development`, `internal-contract-review`,
  `test-evidence`.
- **Project-specific `.claude/agents/`**: `code-reviewer`, `contract-checker`, `feature-tester`.
- **Project-specific `.claude/commands/`**: `add-feature`, `review-code`, `validate-contract`, `verify-feature`.
- **Key directories for integration work**:
  - API service layer: `src/services/<module>/<name>Api.js`.
  - State management: `src/store/*.js` (one Zustand store per
    resource/domain) + `src/hooks/use<Resource>.js`.
  - Routing: `src/routes/AppRoutes.jsx` (`permissionRoutes` array),
    `src/layouts/MainLayout.jsx` (`menuData` + `titleMap`/`getPageMeta`).
  - Shared axios instance: `src/api/axiosInstance.js`.
  - Auth/permission helpers: `src/hooks/useAuth.js`, `src/store/authStore.js`.

## Integration boundaries

- **API base URL**: frontend's `.env` sets
  `VITE_API_BASE_URL=http://localhost:8080/api` (read in
  `src/api/axiosInstance.js`); backend serves every route under Laravel's
  default `/api` prefix (`routes/api.php`) behind `nginx` on port 8080 in
  the local docker setup — **confirmed match**.
- **Auth mechanism**: frontend attaches `Authorization: Bearer <token>` on
  every request (`axiosInstance.js` request interceptor, token from
  `src/utils/tokenHelper.js` / `localStorage`); backend guards protected
  routes with `auth:api` (Laravel Passport) — **confirmed match**. Note:
  the frontend's 401 response interceptor exists but had its
  auto-redirect/clear logic commented out as of the last check (see
  `reactjs-ant-design/CLAUDE.md`) — re-verify if debugging a stale-session
  symptom.
- **Module correspondence**: `src/pages/<module>/` (+ matching
  `src/services/<module>/`) ↔ backend route group
  `Route::group(['prefix' => '<module>', ...])` in `routes/api.php`, each
  guarded by its own `<Module>Maintenance` middleware. Endpoint style is
  **not uniform across modules** — e.g. Manpower Request is POST-only for
  every action (including reads), while KPI uses proper REST verbs; match
  whichever convention the specific module's backend controller already
  uses, per both repos' own `CLAUDE.md`.

---

**Last generated**: 2026-09-14 by direct review (not the `repository-discovery`
skill's automated flow) — populated from this session's own extensive work
in both repos, then spot-verified (API base URL, route prefix, auth guard)
rather than taken purely from memory. Re-run `repository-discovery` or
re-verify manually if either repo's structure has changed materially since.
