---
description: Heads-up on the project roadmap — what's left, what needs the user's decision, repo / deploy state, and the suggested next item (read-only)
argument-hint: [optional focus, e.g. "gateway", "deploy", "fixes", or a phase number]
---

Give the user a short heads-up on the roadmap before any work starts. This
command is **read-only**: don't build, fix, migrate, seed, commit or push —
it ends with a recommendation and a question.

1. **Find the roadmap.** The workspace keeps it in `docs/` (the root
   `CLAUDE.md` "Current plan" names it; otherwise the `docs/*roadmap*.md`
   file). Read its "pick up here" / "left to do" section first, then the
   deploy-pending, "to fix" and "open questions" sections. If `$ARGUMENTS`
   names a focus, read that part fully and keep the rest to one line each.
   Treat the roadmap as a cache: verify any load-bearing claim (a commit
   pushed, a feature built, a file present) against the repositories before
   repeating it.

2. **Check the repositories' state** (read-only git only: `status`,
   `log -1`, `fetch` then `rev-list --count` ahead / behind, `stash list`)
   for this workspace and each application repository inside it, on the
   branch the roadmap says it should be on. Flag: wrong branch, behind or
   ahead of its remote, uncommitted changes (don't assume they're yours —
   never clean them up), stashes, and anything the roadmap's housekeeping
   notes mention.

3. **Report** in this shape, short and specific (names, numbers, commit
   ids, file paths):

   ```
   ## Where things stand        (last update date, latest pushed commits)
   ## Repositories              (branch, ahead/behind, uncommitted, stashes)
   ## Needs your decision       (open questions + items waiting on the user,
                                 e.g. a design to confirm before building)
   ## Left to do                (grouped as the roadmap groups them; one line
                                 per item, focus area in full)
   ## Before production         (deploy steps not yet done, checks pending)
   ## Suggested next            (one item, why it comes first, and which
                                 skill / command would run it)
   ```

   Keep confirmed-from-the-repo and taken-from-the-roadmap visibly apart
   when they disagree (e.g. "roadmap says pushed; branch is 2 ahead").

4. **End** by asking which item to start. Don't re-ask decisions the
   roadmap records as already made by the user.

When the user later finishes an item, the roadmap's own rule applies: mark
it done there (one line, details in the commit) and keep its "pick up here"
section current.
