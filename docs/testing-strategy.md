# Testing strategy

## Philosophy

Test the business process, not the function call. A feature is "working"
when a real user, following the application's actual real flow, can
complete it correctly — including the points where it's supposed to reject
bad input, deny an unauthorized action, or block an invalid state
transition. See `.claude/skills/user-workflow-testing/SKILL.md` for the full
approach; this document is the shorter strategy-level summary.

## Layers of testing this workspace performs

1. **Contract verification** (`cross-repository-review`) — does the frontend
   actually call what the backend actually implements, with matching
   fields, types, auth, and error handling? This catches integration bugs
   that neither side's own tests would catch in isolation.
2. **Workflow verification** (`user-workflow-testing`) — does the complete
   business process work end to end, the way a user would actually
   experience it? This catches bugs that only show up across multiple
   steps (a value that saves but doesn't survive a reload; a status
   transition that's allowed by one action but silently blocked by
   another).
3. **Code review** (`code-review`) — is the implementation itself sound,
   secure, and consistent with the repository's own conventions,
   independent of whether a specific test happens to exercise the risky
   path today?
4. **Regression scoping** (`regression-testing`) — given a change, what else
   could plausibly be affected, so re-testing after a fix is targeted
   instead of either "test nothing" or "test everything."

These layers are complementary, not redundant: a feature can pass contract
verification (frontend and backend agree) while still failing workflow
verification (the agreed-upon behavior itself doesn't match what a real
process needs), and vice versa.

## Evidence over assumption

Every finding traces to something actually observed: a line of code read on
both sides of a contract, an actual executed request/response, an actual
screen state after a real action. A plausible-sounding claim that wasn't
actually checked is reported as unconfirmed, not as a finding. See
`.claude/skills/test-evidence/SKILL.md` for the exact structure this
takes.

## Preferring real execution over reasoning

When it's available, drive the actual running application — browser
automation for user-facing workflows, direct API calls (matching a
confirmed real payload shape, never an invented one) when a browser isn't
available or when testing purely backend-level contract details. A
successful-looking response is not proof a user-visible workflow works;
verify what would actually be visible, and where useful and safe, the
underlying data too.

## Database and git safety while testing

Testing against real repositories means real risk if done carelessly. The
root `CLAUDE.md` has the authoritative rules; the short version: don't run
anything destructive without explicit instruction, prefer the least
invasive verification method, don't touch a repository's working tree
beyond what the current task requires, and never assume uncommitted changes
found in a repository are yours to discard.

## Severity discipline

Findings use one shared scale (BLOCKER/CRITICAL/HIGH/MEDIUM/LOW/INFO, see
`test-evidence`) across code review and test execution alike, so a security
gap found during review and a broken workflow found during testing are
comparably prioritized rather than living in separate, incompatible
taxonomies.

## This strategy is module-agnostic

Nothing above assumes a specific module, a specific framework, or a
specific pair of repositories. The same strategy applies whether the
feature under test is Employee Master Data, an approval workflow, a report,
or authentication itself — and applies unchanged if the repositories in
this workspace are ever replaced with a different application entirely.
