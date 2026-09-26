---
description: Scope and implement a new feature/module in the repository that owns it, following established conventions
argument-hint: <feature/module name and a short description of what it should do>
---

Add the feature described in `$ARGUMENTS`. If it's empty, ask what the
feature is, which module it belongs to and which repositories it touches.

Follow the `feature-development` skill (run `repository-discovery` first if
the repositories aren't mapped this session). This command **does** modify
code, but only in the repositories that own the feature; the root
`CLAUDE.md` safety rules still apply. Finish by recommending `/review-code`
and `/test-feature` (or `/test-workflow`) — this builds the feature, it
doesn't certify it.
