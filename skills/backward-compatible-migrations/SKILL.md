---
name: backward-compatible-migrations
description: Apply for any database change — ALTER TABLE, new columns, index additions, column renames, data backfills, storage-layer refactors. Uses expand-contract pattern, nullable columns, and schema-before-code deploys
---

# Backward-Compatible Migrations

## Overview

A backward-compatible migration changes the storage layer without breaking the currently-running version of the application. This means old code and new code can both operate against the same schema simultaneously, enabling zero-downtime deploys and safe rollback.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Schema changes that cannot break running code eliminate an entire class of deployment incidents |
| Mean Time to Restore (MTTR) | Rollback is safe because the previous code version still works against the new schema |

## When to Use

- Adding, modifying, or removing any database column, table, or index
- Changing data types, constraints, or default values
- Renaming any storage-layer entity
- Running a data backfill that affects records being actively read or written

## When NOT to Use

- Migrations run during a scheduled maintenance window with the application taken offline
- Single-developer projects with no concurrent users and no rollback requirement
- Pure additive changes (new table, new column with a nullable default) that have no risk of breaking existing queries

## Core Pattern

**Before — destructive migration:**

```
// All in one migration file, deployed with the new code:
RENAME COLUMN users.name TO users.full_name
DROP COLUMN users.legacy_score

// If the new code has a bug and must be rolled back:
// old code tries to SELECT users.name → column does not exist → outage
```

**After — expand-contract migration:**

```
// Phase 1 — EXPAND (deploy before new code)
ADD COLUMN users.full_name TEXT
// old code still reads users.name → no breakage
// new code writes to both users.name and users.full_name

// Phase 2 — BACKFILL (run as a background job or separate migration)
UPDATE users SET full_name = name WHERE full_name IS NULL
// both columns exist; both old and new code work

// Phase 3 — CONTRACT (deploy after confirming no code reads old column)
DROP COLUMN users.name
// only after all instances run code that no longer reads users.name
```

## Multi-Phase Migration Checklist

```
Before Phase 1 (Expand):
  [ ] New column is nullable OR has a safe default
  [ ] No NOT NULL constraint added without a default
  [ ] Index created CONCURRENTLY (non-blocking)
  [ ] Application code reviewed: does old code still work?

Before Phase 3 (Contract):
  [ ] Confirmed no running instance reads the old column (check logs, query stats)
  [ ] Backfill verified complete (row count, spot checks)
  [ ] Rollback plan documented if drop must be undone
```

## Dangerous Operations

These operations are not backward-compatible when paired with a simultaneous code deploy:

| Operation | Risk | Safe Alternative |
|-----------|------|-----------------|
| RENAME COLUMN | Old code references old name → crash | Add new column, migrate data, drop old |
| DROP COLUMN | Old code tries to read it → crash | Confirm zero reads first, then drop |
| DROP TABLE | Old code queries it → crash | Rename to archive table, verify, then drop |
| CHANGE COLUMN TYPE | Implicit cast may fail or truncate data | Add new column with new type, migrate, drop old |
| ADD NOT NULL without default | Breaks inserts from old code that doesn't supply the column | Add nullable first, backfill, add constraint after |
| Exclusive table lock | Blocks all reads and writes during migration | Use online schema change tools or CONCURRENTLY |

## Quick Reference

| Rule | Guidance |
|------|----------|
| Separate schema deploy from code deploy | Schema changes ship before the code that uses them |
| Always expand before you contract | Add first; remove only after old code is gone |
| New columns must be nullable or have a default | Old INSERT statements don't know about the new column |
| Never rename in production without a transition period | Always add-and-copy, never rename-in-place |
| Verify backfill before contracting | Count rows, check for NULLs, sample-check values |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Rename column in same deploy as code | Rolled-back code reads a column that no longer exists | Three-phase expand-contract across separate deploys |
| Add NOT NULL column without default | Old code inserts fail immediately | Add nullable, backfill, add constraint in a later migration |
| Drop column before removing code references | Any running old instance crashes on read | Remove code references first, deploy, then drop the column |
| Long-running migration that locks the table | All application traffic blocked during migration | Use batched updates; use CONCURRENTLY for index creation |
| Skipping the backfill verification step | Nulls or stale data in the new column cause silent bugs | Assert row counts match and sample-check values before dropping old column |

## Related Skills

- **rollback-friendly-design** — migrations must preserve the ability to roll back the previous code version safely
- **api-versioning** — apply the same expand-contract pattern to API endpoints and response schemas
- **configuration-as-code** — migration configuration and scheduling should be version-controlled alongside code
