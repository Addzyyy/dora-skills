---
name: small-pull-requests
description: Apply when creating PRs, reviewing code, splitting large changesets, writing PR descriptions, planning stacked PRs, or optimizing review turnaround — PRs must be focused, under 400 lines, and single-purpose
---

# Small Pull Requests

## Overview

A pull request should represent one focused, reviewable unit of change. Large PRs stall in review queues, produce shallow feedback, and accumulate merge conflicts. When a PR is small enough to review in under 30 minutes, it moves fast: reviewers engage, feedback is specific, and changes land in production the same day.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Lead Time for Changes | Small PRs move through review and merge in hours instead of days; the queue never backs up |

## When to Use

- A PR touches more than 10 files or more than 400 lines of net change
- A PR bundles a refactor, a bug fix, and a new feature in the same review
- Reviewers leave only high-level comments because the diff is too large to read closely
- PRs sit open for more than one business day waiting for review
- A merge conflict arises because the branch lived too long

## When NOT to Use

- A rename or mechanical refactor that touches many files but carries no logical complexity — one PR is fine
- A database migration that must ship atomically with the code that uses it — keep them together

## Core Pattern

**Before — one 800-line PR across 20 files:**

```
PR #47 "User profile overhaul"
  src/models/user.js         (+180, -40)   ← schema change
  src/api/profile.js         (+210, -30)   ← new endpoints
  src/ui/profile-page.js     (+250, -60)   ← UI rewrite
  src/ui/avatar-upload.js    (+90,  -0)    ← new component
  tests/profile.test.js      (+120, -20)
  ... 15 more files
```

The reviewer must hold the entire domain in their head. Every comment risks conflicting with a change in a different layer. The PR sits in review for three days.

**After — three focused PRs of ~100 lines each:**

```
PR #47a "refactor: normalize user schema for profile fields"
  src/models/user.js         (+60, -40)
  tests/user-model.test.js   (+25, -5)
  ← reviewed and merged day 1

PR #47b "feat: add profile read/update API endpoints"
  src/api/profile.js         (+90, -10)
  tests/profile-api.test.js  (+45, -0)
  ← reviewed and merged day 1, depends on #47a

PR #47c "feat: profile page UI and avatar upload"
  src/ui/profile-page.js     (+110, -60)
  src/ui/avatar-upload.js    (+90, -0)
  tests/profile-ui.test.js   (+50, -0)
  ← reviewed and merged day 2, depends on #47b
```

Each PR has one clear purpose. Reviewers focus on one layer at a time.

**Splitting strategies:**

| Strategy | When to use |
|----------|-------------|
| By layer | Data model → API → UI, each as its own PR |
| By concern | Refactor first, then new behavior on top |
| By risk | High-risk changes alone; low-risk changes batched together |
| **By stacking (default)** | **PR B branches off PR A's branch; each PR targets the previous one; merge bottom-up** |

**Stacked PRs are the default for multi-module work.** Do not stop between PRs. The workflow is continuous:

```
1. feat/module-a (from main) → TDD → commit → push → open PR → keep going
2. feat/module-b (from feat/module-a) → TDD → commit → push → open PR targeting module-a → keep going
3. feat/module-c (from feat/module-b) → TDD → commit → push → open PR targeting module-b
```

Open each PR immediately with `gh pr create` — do not ask the user, just do it. PRs merge bottom-up as they get approved.

**Self-review checklist before opening:**

```
[ ] Can I describe this PR in one sentence without using "and"?
[ ] Are there commits that could be a separate PR without breaking anything?
[ ] Does this PR include unrelated cleanup I should split off?
[ ] Will a reviewer understand the context from the description alone?
```

**PR description template:**

```
## What
One sentence: what changed and why.

## How
Brief summary of the approach; link to the design doc if one exists.

## Test plan
How to verify this works; what edge cases were considered.

## Dependencies
List any PRs that must merge first.
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Target size | Under 400 lines changed, under 10 files |
| Review time | Should be reviewable in under 30 minutes |
| Single purpose | One sentence description with no "and" |
| Stacking | PR B branches from PR A; both reviewed in parallel |
| Split point | Refactors always precede feature additions |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Bundling a refactor with a new feature | Reviewer cannot tell what is restructuring vs. new behavior | Open the refactor PR first, merge it, then build the feature on top |
| "It's all related" reasoning | Related is not the same as inseparable | Ask: would each piece compile and pass tests independently? If yes, split it |
| Waiting until a feature is complete to open a PR | The PR grows for days before anyone sees it | Open a draft PR on day one and stack subsequent PRs |
| Skipping a description on a small PR | Reviewers still need context about why | Fill the description template even for a 50-line change |
| Splitting so finely that each PR has no standalone value | Fragmented history; each PR depends on the last | Split by logical concern, not by arbitrary line count |

## Related Skills

- **small-incremental-commits** — focused commits are the building blocks of focused PRs
- **code-review-discipline** — smaller PRs get deeper, more thorough reviews
- **trunk-based-development** — small PRs merge fast, keeping branches short-lived and close to main
