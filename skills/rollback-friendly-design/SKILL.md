---
name: rollback-friendly-design
description: Apply when writing any deployed code, DB schema changes, API changes, migrations, feature flags, config changes, or deployment plans — covers rollback safety, blue-green/canary deploys, backward compatibility, and one-way door decisions
---

# Rollback-Friendly Design

## Overview

Rollback-friendly design means new code and old code can coexist on the same infrastructure, against the same data, without either breaking. When this is true, rolling back a bad deploy is a single action that takes seconds. When it is not, a rollback is itself a risky operation that may cause data loss or downtime.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Mean Time to Restore (MTTR) | A bad deploy is restored by flipping a version, not by a hotfix-and-redeploy cycle that takes hours |

## When to Use

- Planning any database schema change alongside a code change
- Designing a deployment that will be released to production without a maintenance window
- Adding a new field, endpoint, or behavior that callers might not immediately support
- Evaluating whether a proposed change is a "one-way door"

## When NOT to Use

- Prototype code that is explicitly not going to production
- Batch migration jobs run with the system taken offline intentionally
- Internal tools with a single known caller that can be updated atomically

## Core Pattern

**Before — deploy that cannot be rolled back:**

```
Version 2 of the API changes the response format:
  GET /orders → { "items": [...] }         (v1)
  GET /orders → { "order_items": [...] }   (v2)

Deploy v2 code. Mobile clients still expect "items" field.
  → rollback to v1 code
  → but v1 config was overwritten by v2 deploy
  → rollback fails, both versions are broken
```

**After — deploy where old and new code coexist:**

```
Phase 1 (expand):
  Deploy code that returns BOTH fields:
    { "items": [...], "order_items": [...] }
  Old and new clients both work.

Phase 2 (migrate):
  Update clients to read "order_items"
  Monitor: are any clients still reading "items"?

Phase 3 (contract):
  Remove "items" field once no client reads it

Rollback is safe at any point in Phase 1 or 2 —
old code only added a field, never removed one.
```

## Expand-Contract Pattern

```
// Any schema or interface change follows three phases:

EXPAND   → add the new thing alongside the old thing
MIGRATE  → move traffic/data to the new thing gradually
CONTRACT → remove the old thing once nothing depends on it
```

This pattern applies to: columns, tables, API endpoints, queue topics, config keys.

## Rollback Strategies

| Strategy | How It Works | Rollback Speed |
|----------|-------------|----------------|
| Blue-green | Two identical environments; traffic switches between them | Instant (DNS/load balancer) |
| Canary | New version receives a small percentage of traffic | Instant for affected % |
| Feature flag | Logic change hidden behind a flag | Instant (toggle flag) |
| Versioned endpoints | Old endpoint kept alive during transition | Gradual (deprecate later) |

## Identifying One-Way Doors

A change is a one-way door if rolling back the code would break the running system. Ask:

```
If I deploy this and immediately roll back the code:
  - Does the old code still read the data correctly?
  - Does the old code still call the APIs correctly?
  - Can both old and new instances run simultaneously?

If any answer is NO → apply expand-contract before deploying.
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Never rename or drop in the same deploy as code | Schema change and code change must be independently rollback-safe |
| Test rollback before going to production | Run the rollback in staging; confirm data integrity |
| Use feature flags for behavioral changes | Separate the code deploy from the feature activation |
| Keep deploys small | A one-line change is trivially rollback-safe; a 2,000-line change is not |
| Version APIs | Old callers must continue working when the server is rolled back |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Rename column in same deploy as code change | Old code cannot read renamed column after rollback | Use expand-contract: add new column, migrate, drop old |
| New code writes format that old code cannot read | Rollback corrupts data or causes crashes | Ensure old code can safely ignore or parse new fields |
| No rollback plan documented | Incident pressure leads to improvised rollback that causes more damage | Write rollback steps before the deploy, not during the incident |
| Blue-green with shared database | Schema change makes old environment invalid | Make schema changes backward-compatible before switching traffic |
| Feature flag not tested in OFF state | Rollback (flag OFF) hits untested code paths | CI must test both flag states |

## Related Skills

- **backward-compatible-migrations** — schema changes must preserve rollback safety for the previous code version
- **feature-flags** — flags provide instant behavioral rollback without redeploying
- **api-versioning** — versioned API endpoints enable safe rollback by keeping old versions alive
