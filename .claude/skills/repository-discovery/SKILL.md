---
name: repository-discovery
description: Discover and map the repositories inside this workspace — detect each one's actual role (frontend/backend/other) from its real contents rather than its folder name, locate its documentation and project-specific Claude configuration, and build a repository map. Use this first, before any cross-repository review, integration test, or workflow test, whenever the repositories haven't been mapped yet this session or the map might be stale.
---

# Repository discovery

Never assume a folder's name tells you its role, and never assume a
repository is correctly implemented just because it exists. Inspect it.

## 1. Enumerate what's actually in the workspace

List the top-level directories in the workspace root. Anything that isn't
`.claude/`, `docs/`, `test-scenarios/`, `test-results/`, or a dotfile is a
candidate repository — regardless of what it's named. A workspace may
contain exactly two (`frontend-repo/`, `backend-repo/`), more than two
(multiple backends, a shared library, a mobile client), or repositories with
entirely different names. Treat the names as labels a human chose, not as
facts about role.

## 2. Determine each repository's actual role from its contents

Check for these signals, in the repository's root and one level deep. A
repository can show more than one signal (a Laravel app that also serves a
Blade-rendered SPA, for instance) — report what you find rather than forcing
a single label.

**Backend / API signals:**
- `composer.json` + `artisan` → Laravel.
- `composer.json` alone, no frontend build tooling → likely another PHP
  framework or plain PHP — check `composer.json`'s `require` block for the
  actual framework before assuming Laravel.
- `package.json` with `express`/`fastify`/`koa`/`@nestjs/core` as a dependency
  and no frontend framework alongside it → Node backend.
- A `routes/` or `src/routes/` directory containing route *definitions*
  (not React Router routes — look at what's actually inside), `app/Http` or
  `app/Controllers`, `migrations/`, `models/` or `entities/`.
- `manage.py` + a Django-shaped tree, `go.mod` + a Go web framework,
  `pom.xml`/`build.gradle` + a Spring-shaped tree — same principle, look for
  the actual framework markers rather than guessing from the language alone.

**Frontend signals:**
- `package.json` with `react` + `react-dom` → React app. Check further for
  `antd`, `@mui/material`, `bootstrap`, etc. to know the UI library in play.
- `package.json` with `vue` → Vue app; `vuetify` alongside it narrows the UI
  library.
- `vite.config.js`/`webpack.config.js`/`angular.json`/`next.config.js` as
  further confirmation of a frontend build.
- A `src/pages/` or `src/views/` or `src/components/` tree with route
  definitions that reference an API base URL (grep for `axios`, `fetch`,
  `VITE_API_BASE_URL`, `NEXT_PUBLIC_API_URL`, or similar).

**Ambiguous or mixed:**
- A Laravel repo can *also* be a legacy frontend if it ships server-rendered
  Vue/Blade views (check `resources/views/`, `resources/js/`) alongside its
  API routes. Report both roles if both are genuinely present — don't
  collapse a repository that does two things into a single label.
- If nothing matches confidently, say so. "I could not confidently classify
  this repository from its contents" is a correct, useful finding — better
  than a wrong guess.

## 3. Locate each repository's own documentation and configuration

For every repository found, check for and note whether each of these exists
(don't assume; look):

- `CLAUDE.md` (root, and any nested ones for monorepo-style projects)
- `README.md`
- `.claude/skills/` — list what's in it
- `.claude/agents/` — list what's in it
- `.claude/commands/` — list what's in it
- Package/dependency manifest (`package.json`, `composer.json`, `go.mod`,
  etc.) — note the framework and major version if stated
- For likely backends: routes directory, controllers directory, service
  layer directory (if the project has one), models/entities directory,
  migrations directory, existing test directory
- For likely frontends: API client/service layer directory, state
  management directory (store/context/redux), routing configuration, an
  env file defining the API base URL (note that it exists; never read or
  echo its actual secret values)

## 4. Build the repository map

Summarize what you found per repository:

- Path (relative to workspace root)
- Detected role(s) and the evidence for each (don't just assert "backend" —
  say what file/pattern led to that conclusion)
- Framework + version if determinable
- Location of its `CLAUDE.md`, and a one-line summary of what it covers
- Location of its project-specific skills/agents/commands, if any
- Key directories relevant to cross-repository work (routes, API client,
  etc.)

If asked to persist this (or if it will clearly be reused across sessions),
write it into `docs/repository-map.md` in the format that file already
uses. Mark it with the date it was generated, since repositories change —
a stale map is worse than no map if it's trusted blindly. Whoever reads it
later should re-verify a specific claim against the actual repository before
relying on it for anything consequential.

## 5. Identify likely integration boundaries

Once roles are known, identify where the repositories actually talk to each
other:

- The frontend's API base URL configuration and the backend's route prefix —
  do they line up?
- The frontend's auth mechanism (bearer token, cookie/session, OAuth) and the
  backend's auth guard/middleware — are they the same mechanism?
- Any shared module/feature naming that suggests which frontend area maps to
  which backend module (e.g., a frontend `src/pages/manpower_request/` next
  to a backend `routes` group prefixed `manpower_request`).

This becomes the starting point for `cross-repository-review` — don't
re-derive it there if it's already established here.

## What NOT to do

- Don't assume a two-repository workspace is always one frontend + one
  backend. Verify both.
- Don't skip discovery because "it's obviously the same project as last
  time" — a repository can be restructured, and a stale assumption produces
  wrong findings downstream.
- Don't read or copy a repository's `CLAUDE.md` content into this workspace's
  own configuration. Note where it is and what it covers; leave its content
  where it lives.
