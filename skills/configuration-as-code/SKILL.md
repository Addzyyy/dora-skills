---
name: configuration-as-code
description: Apply when touching environment variables, .env files, feature toggle config, deployment params, infrastructure settings, or hardcoded values that should be config. All config version-controlled, secrets referenced by name never by value
---

# Configuration as Code

## Overview

Configuration as code means storing all environment configuration, deployment parameters, and infrastructure settings in version-controlled files — reviewed, tested, and deployed the same way as application code. It eliminates config drift, makes environment differences visible, and allows any environment to be reproduced from source.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Deployment Frequency | Consistent environments remove "works on my machine" blockers; deployments are repeatable |
| Mean Time to Restore (MTTR) | A broken environment can be rebuilt from source in minutes instead of reconstructed from memory |

## When to Use

- Any environment parameter that differs between dev, staging, and production
- Infrastructure that is provisioned manually and hard to reproduce
- Config changes that bypass review (applied directly to servers or dashboards)
- Incident post-mortems that cite config drift as a contributing factor

## When NOT to Use

- Secrets and credentials — never store these in version control; use a secrets manager and reference them by name
- Ephemeral local overrides a developer uses only on their own machine (`.env.local` files kept out of source control)

## Core Pattern

**Before — config lives in people's heads and server state:**

```
Production DB_POOL_SIZE: 20    ← set by hand 18 months ago, nobody remembers why
Staging    DB_POOL_SIZE: 5     ← different, set by a different person
Dev        DB_POOL_SIZE: 2     ← different again

Result: staging passes load test, production falls over at the same load
```

**After — config is versioned and reviewed:**

```
config/
  base.yaml          ← shared defaults
  environments/
    dev.yaml         ← overrides for dev
    staging.yaml     ← overrides for staging
    production.yaml  ← overrides for production

# base.yaml
database:
  pool_size: 10
  timeout_seconds: 30

# production.yaml (overrides base)
database:
  pool_size: 20
```

Every change goes through a pull request. The diff shows exactly what changed and who approved it.

**Secrets separation pattern:**

```
config/production.yaml:
  database:
    host: db.prod.example.com      ← safe to version
    pool_size: 20                  ← safe to version
    password: ${DB_PASSWORD}       ← reference only; value lives in secrets manager
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| All config in source | If it affects behavior, it belongs in a file |
| Secrets by reference | Store the secret name, never the value |
| Environment parity | Keep dev/staging/production as similar as possible; document every intentional difference |
| Validate on CI | Lint and schema-check config files in the pipeline before deployment |
| Single source of truth | Config is applied from the file, not patched on the server |

**Environment parity checklist:**
- Same OS and runtime version across environments
- Same dependency versions
- Same config structure; only scale/endpoint values differ
- Differences are documented as intentional exceptions

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Storing secrets in config files | Credentials leak via source control history | Reference secret names; resolve values at runtime from a secrets manager |
| One giant config file for all environments | Diff shows everything; reviewers miss the important change | Separate base and per-environment override files |
| Applying config manually after deploy | Drift accumulates; server state diverges from source | Make the deploy pipeline apply config from source; block manual changes |
| No validation in CI | Malformed config reaches production | Add schema validation and a dry-run apply step to the pipeline |
| Treating config as less important than code | Config changes cause outages as often as code changes | Require the same review process for config PRs as for code PRs |

## Related Skills

- **feature-flags** — feature flags are a form of runtime configuration that decouples deploy from release
- **dependency-management** — dependency versions are configuration that must be pinned and version-controlled
- **rollback-friendly-design** — versioned configuration enables instant rollback to a known-good state
