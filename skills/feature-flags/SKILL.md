---
name: feature-flags
description: Always apply when shipping new features — wrap in feature flags to decouple deployment from release and enable instant rollback
---

# Feature Flags

## Overview

A feature flag is a conditional in code that enables or disables a feature at runtime without a deployment. Flags let teams deploy code continuously while controlling who sees what, and they make rollback instant — toggle off instead of rollback deploy.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Deployment Frequency | Incomplete features can ship to production hidden behind a flag; deploy is never blocked by feature readiness |
| Mean Time to Restore (MTTR) | Disable a problematic feature in seconds without a redeploy or incident bridge |

## When to Use

- A feature is not yet ready for all users but code needs to reach production
- A risky change needs an instant off-switch independent of the deploy pipeline
- Rolling out to a subset of users (percentage, region, cohort) before full release
- A/B testing or experiments where different users see different behavior
- Trunk-based development: guarding incomplete work merged to main

## When NOT to Use

- Security fixes — ship them immediately, do not hide behind a flag
- Simple low-risk changes that can go straight to production
- As a substitute for proper environment promotion (flags are not environments)

## Core Pattern

**Before — incomplete feature blocks deployment:**

```
feature branch lives for 3 weeks
  → painful merge to main
  → either delay release or ship broken UX
  → no safe rollback path
```

**After — feature behind a flag, shipped safely:**

```
// Pseudocode — any language/framework applies
if flag_enabled("new_checkout_flow", user):
    show_new_checkout(user)
else:
    show_existing_checkout(user)
```

Deploy this to production with the flag OFF. Enable it for internal users, validate, then roll out progressively. If something goes wrong, flip the flag — no redeploy required.

**Flag lifecycle:**

```
1. CREATE  — add flag, default OFF, merge to main
2. ENABLE  — turn ON for a test cohort (internal, beta, 5% of traffic)
3. VALIDATE — monitor metrics, error rates, user signals
4. RELEASE — enable for 100% of users
5. REMOVE  — delete the flag and dead code path (do not skip this step)
```

## Flag Types

| Type | Purpose | Example |
|------|---------|---------|
| Release | Hide incomplete work | `new_checkout_flow` |
| Experiment | A/B test a hypothesis | `checkout_single_page` |
| Ops | Kill switch for system behavior | `enable_rate_limiting` |
| Permission | Gate by user role or plan | `advanced_analytics` |

## Quick Reference

| Rule | Guidance |
|------|----------|
| Default state | New flags default OFF in production |
| Flag lifetime | Set a removal date when you create the flag |
| Cleanup | Remove flags within 1–2 sprints of full rollout |
| Nesting | Avoid flags inside flags; maximum 1 level deep |
| Testing | Test both flag ON and flag OFF paths in CI |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Never removing flags | Code accumulates dead branches; logic becomes unreadable | Add a cleanup task when the flag is created |
| Nesting flags 2–3 levels deep | Combinatorial explosion of states to test | Flatten: one flag per concern |
| Using flags as environment config | Flags are for features, not for DB URLs or secrets | Use `configuration-as-code` for environment config |
| No flag inventory | Unknown flags linger for years | Maintain a registry with owner and target removal date |
| Testing only the ON path | OFF path breaks silently | CI must validate both paths |
