# Repository map

This file is a **cache** of what `repository-discovery` found, written so
later sessions don't have to re-derive it from scratch. It is not a source
of truth — a specific claim in here should be re-verified against the
actual repository before being relied on for anything consequential,
especially if this file's "Last generated" date is old or a repository has
been worked on since.

**Status: not yet generated.** Place your repositories in `frontend-repo/`
and `backend-repo/` (or whatever paths/names you're using), then run
`/review-system` or invoke the `repository-discovery` skill directly to
populate this file.

---

## How to fill this in

Run repository discovery, then replace this section with one block per
repository found, in this shape:

```
## <repository path, e.g. frontend-repo/>

- **Detected role(s)**: <e.g. "Frontend SPA (React + antd)"> — evidence:
  <the actual file(s)/pattern(s) that led to this conclusion>
- **Framework / version**: <e.g. "React 19, Vite">
- **CLAUDE.md**: <path, or "none found"> — <one-line summary of what it
  covers, if present>
- **Project-specific `.claude/skills/`**: <list, or "none found">
- **Project-specific `.claude/agents/`**: <list, or "none found">
- **Project-specific `.claude/commands/`**: <list, or "none found">
- **Key directories for integration work**:
  - <e.g. "API service layer: src/services/">
  - <e.g. "State management: src/store/">
  - <e.g. "Routing: src/routes/">

## <repository path, e.g. backend-repo/>

- **Detected role(s)**: <...>
- **Framework / version**: <...>
- **CLAUDE.md**: <...>
- **Project-specific `.claude/skills/`**: <...>
- **Project-specific `.claude/agents/`**: <...>
- **Project-specific `.claude/commands/`**: <...>
- **Key directories for integration work**:
  - <e.g. "Routes: routes/api.php">
  - <e.g. "Controllers: app/Http/Controllers/API/">
  - <e.g. "Service layer: app/Services/">
  - <e.g. "Migrations: database/migrations/">

## Integration boundaries

- **API base URL**: frontend configures it at `<path>`; backend serves
  routes under `<prefix>` — <confirmed match / mismatch, with evidence>
- **Auth mechanism**: frontend uses `<mechanism>`; backend guards with
  `<mechanism>` — <confirmed match / mismatch>
- **Module correspondence**: <e.g. "frontend src/pages/X/ ↔ backend routes
  group prefixed X">

---

**Last generated**: <date> by `repository-discovery`
```

Keep one such filled-in version of this file going forward — overwrite the
stale sections rather than appending a new copy each time discovery is
re-run, so this stays a map, not a history.
