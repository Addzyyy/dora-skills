---
name: small-incremental-commits
description: Apply whenever running git commit, git add, staging changes, writing commit messages, or preparing code for review — each commit must be one logical change; split large diffs, never bundle unrelated changes
---

# Small, Incremental Commits

## Overview

A commit should represent one logical change — not one session of work. Small, focused commits make code review faster, history easier to read, and rollbacks surgical. When each commit compiles and passes tests independently, any commit can be reverted without side effects.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Deployment Frequency | Smaller units of work reach production faster and with less coordination overhead |
| Lead Time for Changes | Reviewers process focused commits in minutes, not hours; merges are unblocked |

## When to Use

- A commit message requires "and" to describe what changed
- A pull request touches unrelated files or concerns
- A reviewer struggles to understand what problem the commit solves
- Rollback of one change would unintentionally revert another

## When NOT to Use

- A true atomic change genuinely spans multiple layers (e.g., a database migration paired with the code that uses it) — keep those together
- Generated or vendored files that must change as a unit

## Core Pattern

**Before — one bloated commit:**

```
commit a3f92c1
"refactor auth module, add password reset feature, fix login timeout bug"

 src/auth/validator.js     (+120, -80)   ← refactor
 src/auth/reset.js         (+200, -0)    ← new feature
 src/auth/session.js       (+15, -3)     ← bug fix
 tests/reset.test.js       (+90, -0)
 tests/session.test.js     (+10, -2)
```

Rolling back the bug fix also removes the feature. The review touches everything at once.

**After — three focused commits:**

```
commit 1: "refactor: simplify auth validator to reduce cyclomatic complexity"
  src/auth/validator.js  (+45, -80)
  ← compiles, tests pass

commit 2: "fix: extend session timeout window to prevent spurious logouts"
  src/auth/session.js    (+15, -3)
  tests/session.test.js  (+10, -2)
  ← compiles, tests pass, can be cherry-picked to hotfix branch

commit 3: "feat: add self-service password reset flow"
  src/auth/reset.js      (+200, -0)
  tests/reset.test.js    (+90, -0)
  ← compiles, tests pass
```

Each commit can be reviewed, deployed, or reverted independently.

## Quick Reference

| Rule | Detail |
|------|--------|
| One logical change | If the message needs "and," split the commit |
| Must compile | Every commit should pass the build and tests in isolation |
| Message = "why" | Describe the intent, not the diff (`fix: prevent logout on slow networks`, not `update session.js`) |
| Typical size | Under 200 lines changed; no hard rule, but friction rises fast above that |

**Commit message format:**
```
<type>: <what and why in one line>

Optional body: context a reviewer needs that the diff doesn't show.
```
Types: `feat`, `fix`, `refactor`, `test`, `chore`, `docs`

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Committing "while I'm in there" cleanups with a feature | Obscures intent; bloats diff | Stage the cleanup separately first |
| Writing the message after the fact to cover everything | Message becomes a list, not a description | Decide the message before writing the code |
| Squashing all commits before merge | Loses the incremental story | Squash only true work-in-progress fixups (`fixup!` commits) |
| One commit per file | Splits a logical change artificially | Group by concept, not by file |
| Skipping tests "just for this commit" | Breaks the independent-compile rule | Keep the test alongside the code change |

## Related Skills

- **trunk-based-development** — small commits enable daily integration to main with minimal conflict
- **small-pull-requests** — focused commits are the building blocks of focused, reviewable PRs
- **test-driven-development** — each commit should include the tests that validate its logical change
