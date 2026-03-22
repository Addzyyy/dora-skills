---
name: trunk-based-development
description: Apply when creating branches, merging, choosing branching strategy, resolving merge conflicts, setting up CI/CD, or planning releases — branches must be short-lived and merge to main daily; never use long-lived feature branches
---

# Trunk-Based Development

## Overview

Trunk-based development (TBD) is a branching strategy where all developers integrate to a single shared branch (trunk/main) at least once per day. Long-lived branches are the leading cause of merge pain, delayed feedback, and slow deployment cycles. Short-lived branches eliminate that waste.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Deployment Frequency | Main is always releasable; any commit can ship without a merge event |
| Lead Time for Changes | Integration happens daily instead of at sprint-end; feedback is immediate |

## When to Use

- Deciding a branching strategy for a new team or project
- Branches routinely live longer than 2–3 days
- Merge conflicts consume significant time before releases
- CI only runs on pull request creation, not continuously

## When NOT to Use

- Regulated environments that mandate a separate release branch for audit purposes (use a short-lived release branch cut from main, not a long-lived dev branch)
- Open-source projects with untrusted contributors (use forks, not long-lived branches)

## Core Pattern

**Before — long-lived feature branch:**

```
main:    A---B-----------------------M   ← merge commit, often conflicted
              \                     /
feature:        C--D--E--F--G--H--I      ← 2 weeks of divergence
```

By the time the feature branch merges, main has moved far away. The merge is a mini-project. CI on the branch tested stale code.

**After — trunk-based with short-lived branch:**

```
main:    A---B---C---D---E---F---G   ← deployable at every commit
                  \       /
branch:            b1---b2           ← lives < 1 day, merges before diverging
```

Each branch stays close to main. Conflicts surface immediately and are trivial.

**Breaking a large feature into trunk-safe increments:**

```
Week 1: merge data model (behind feature flag, flag OFF)
Week 2: merge API layer (flag still OFF, but integration-tested on main)
Week 3: merge UI (flag still OFF)
Week 4: enable flag for internal users → validate → enable for all
Week 5: remove flag, clean up dead code
```

The feature ships continuously; users only see it when it is ready. See the `feature-flags` skill for flag mechanics.

## Quick Reference

| Rule | Guidance |
|------|----------|
| Branch lifespan | Under 1 day; delete after merge |
| PR size | Small enough to review in under 30 minutes |
| Integration frequency | Merge to main at least once per day |
| Main branch | Always green; never commit broken code directly |
| Release branches | Cut from main when needed; never merge back to main |

**How to handle a feature too large for one day — use stacked PRs:**
1. Decompose into independently mergeable slices
2. Create branch `feat/slice-1` from main, implement, commit, push, open PR
3. Create branch `feat/slice-2` from `feat/slice-1` (not main), keep working
4. Each PR targets the previous branch; they merge bottom-up
5. Do not stop between slices — keep the momentum going
6. Guard incomplete slices with a feature flag if needed

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| "We do TBD but our branches live 1–2 weeks" | That is not TBD; it is gitflow without the labels | Enforce a 1-day branch age limit in team agreements |
| Merging main into a feature branch repeatedly | Symptom of a branch that has lived too long | Shorten the branch, not the merge cadence |
| Skipping feature flags for large features | Forces big-bang merge at the end | Introduce flags to decouple deploy from release |
| Only one developer integrating daily | Others still accumulate divergence | Every developer merges to main daily |
| Disabling CI on short-lived branches | Fast merges with broken code | CI must run on every push, even small branches |

## Related Skills

- **feature-flags** — flags guard incomplete work merged to main so it is invisible to users
- **small-incremental-commits** — small commits enable frequent integration with minimal merge conflict
- **small-pull-requests** — short-lived branches naturally produce small, reviewable PRs
