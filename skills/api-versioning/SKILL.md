---
name: api-versioning
description: Apply when adding, removing, or modifying any API endpoint, response field, or request parameter — even "simple" additions may need versioning. Covers REST, GraphQL, and RPC changes. Sets sunset dates and ships new alongside old
---

# API Versioning

## Overview

Every public API change is a promise to existing consumers. Breaking that promise without a versioning strategy causes immediate production failures across every consumer that deployed without knowing. Versioning gives consumers a migration window while the provider moves forward.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Versioned APIs prevent breaking changes from crashing consumers on deployment |
| MTTR | A clear deprecation lifecycle means rollback is a version switch, not an emergency patch |

## When to Use

- Removing or renaming a field in a response
- Changing the type or semantics of an existing field
- Removing an endpoint or changing its required parameters
- Introducing a behavior change that existing consumers have not opted into
- Multiple consumers at different deployment cadences consume the same API

## When NOT to Use

- Purely additive changes (new optional fields, new endpoints) — these are backward-compatible and do not require a new version
- An internal API consumed only by code in the same repository, deployed atomically — coordinate the change in one PR instead
- A pre-1.0 API with no external consumers, where breaking changes are expected and documented as such

## Core Pattern

**Before — breaking change without versioning:**

```
API v1 response:
  { "user_id": 123, "full_name": "Ada Lovelace" }

Provider migrates to a new schema and deploys:
  { "id": 123, "firstName": "Ada", "lastName": "Lovelace" }

Consumer A reads `user_id` and `full_name` → both are now undefined
→ Consumer A crashes in production immediately after the provider deploys
→ Incident: provider and consumer must roll back and coordinate
```

**After — additive change with deprecation path:**

```
Step 1: Deploy v2 alongside v1 (both served simultaneously)
  GET /v1/users/123 → { "user_id": 123, "full_name": "Ada Lovelace" }
  GET /v2/users/123 → { "id": 123, "firstName": "Ada", "lastName": "Lovelace" }

Step 2: Announce deprecation with a sunset date
  Response header on v1: Deprecation: true
  Response header on v1: Sunset: 2026-06-01
  Response header on v1: Link: </v2/users>; rel="successor-version"

Step 3: Consumers migrate to v2 on their own schedule before the sunset date

Step 4: Remove v1 after the sunset date; all consumers are already on v2
```

**Versioning strategies:**

| Strategy | Example | Tradeoffs |
|----------|---------|-----------|
| URL path | `/v1/users`, `/v2/users` | Explicit and cacheable; URLs change with each version |
| Query parameter | `/users?version=2` | Easy to add without routing changes; harder to cache correctly |
| Header | `API-Version: 2` | Clean URLs; harder to test in a browser; requires documentation |
| Content negotiation | `Accept: application/vnd.api+json;version=2` | Standards-based; highest implementation complexity |

URL path versioning is the most discoverable and is the safest default for public APIs.

**Backward compatibility rules:**

```
Safe (non-breaking):    Adding an optional response field
Safe (non-breaking):    Adding a new endpoint
Safe (non-breaking):    Adding an optional request parameter with a sensible default
Safe (non-breaking):    Relaxing a validation rule

Breaking:               Removing or renaming a field
Breaking:               Changing a field's type
Breaking:               Making an optional field required
Breaking:               Changing the semantics of an existing field
Breaking:               Removing an endpoint
```

**Deprecation lifecycle:**

```
1. Ship the new version alongside the old
2. Set a sunset date (minimum: one full consumer release cycle, typically 3–6 months)
3. Add Deprecation and Sunset headers to every response on the old version
4. Monitor traffic to the old version; reach out to consumers still using it before the sunset date
5. Remove the old version on or after the sunset date
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Default strategy | URL path versioning (`/v1/`, `/v2/`) for public APIs |
| Non-breaking | Additive changes only; no new version needed |
| Breaking changes | Always ship a new version; never modify the existing version in place |
| Sunset headers | Include `Deprecation` and `Sunset` headers on every deprecated response |
| Minimum sunset window | One full consumer release cycle (typically 3–6 months) |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Renaming a field "in place" on the same version | All consumers break on deploy | Introduce the new field alongside the old; deprecate the old field; remove it in the next major version |
| Setting a sunset date too short | Consumers cannot migrate in time | Give at least one full release cycle; coordinate with consumer teams before setting the date |
| Running too many live versions simultaneously | Maintenance burden grows; bugs must be fixed in N versions | Limit to two live versions at a time (current + previous); accelerate sunset |
| Not monitoring traffic to deprecated versions | Sunset arrives with consumers still using the old version | Track request counts per version; alert when a deprecated version still has active callers near the sunset date |
| Versioning every minor change | Version proliferation; consumers cannot track the latest stable target | Only create a new version for breaking changes; additive changes land on the current version |

## Related Skills

- **contract-testing** — verify consumer contracts before publishing a new API version
- **backward-compatible-migrations** — apply the same expand-contract pattern to the data layer
- **rollback-friendly-design** — versioned APIs enable safe rollback by keeping old endpoints alive
