---
name: contract-testing
description: Apply when changing API schemas, service-to-service integration, microservice boundaries, GraphQL schemas, or protobuf definitions. Verify consumer contracts before merging — the consumer owns the contract
---

# Contract Testing

## Overview

Integration tests verify that services work together in a shared environment. Contract tests verify that a provider still satisfies what each consumer actually needs — without requiring every service to be running. A breaking change that slips past integration tests is caught at the contract boundary before deployment.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Contract tests catch breaking API changes before they reach consumers in production |

## When to Use

- A service change broke a consumer that was not covered by the integration test suite
- Integration tests are slow, flaky, or require a full staging environment to run
- Multiple teams consume the same API and cannot coordinate deploys in lockstep
- A provider wants to know which consumers would break before merging a change
- Schema evolution is happening (adding fields, deprecating fields, changing types)

## When NOT to Use

- A single-consumer internal function with no versioning needs — a unit test is sufficient
- A truly monolithic system where all callers are in the same codebase and test suite
- When the consumer and provider are always deployed atomically and tested end-to-end before release

## Core Pattern

**Before — integration test that misses a breaking change:**

```
Integration test environment:
  Provider v1 running
  Consumer A tested against Provider v1 → passes

Provider ships v2:
  Renames field `user_id` → `userId` (camelCase migration)

Integration test:
  Consumer A tests are not re-run against Provider v2 before deploy
  → Consumer A crashes in production: field `user_id` is undefined
```

The integration test suite did not catch the break because it was not wired to verify the consumer's actual expectations against the new provider version.

**After — consumer-driven contract test:**

```
Consumer A defines its contract (what it actually reads from the response):
  {
    "user_id": integer,      ← Consumer expects this field name
    "email": string
  }

Contract is published to a shared broker.

Provider runs contract verification before merge:
  Loads Consumer A's contract
  Runs its own code against the contract expectations
  → FAIL: provider now returns `userId`, contract expects `user_id`
  → PR blocked before deployment
```

**Contract test vs. integration test:**

| | Contract Test | Integration Test |
|---|---|---|
| Environment | No running services needed | Full stack required |
| Speed | Seconds | Minutes to hours |
| Failure signal | "Consumer A expects X, provider returns Y" | "Something failed in staging" |
| Scope | One consumer's expectations | Emergent behavior across all services |

**Provider verification flow:**

```
1. Consumer writes expectations (the contract) and publishes to broker
2. On every provider PR:
   a. Pull all consumer contracts from broker
   b. Run provider code against each contract
   c. Fail merge if any contract is violated
3. On consumer change:
   a. Update the contract
   b. Re-verify against the current provider before deploying the consumer
```

**Schema evolution rules:**

```
Safe (non-breaking):      Adding an optional field
Safe (non-breaking):      Adding a new endpoint
Safe (non-breaking):      Widening a type (integer → number)

Breaking:                 Removing or renaming a field
Breaking:                 Narrowing a type (number → integer)
Breaking:                 Changing field semantics without a name change

For breaking changes: version the API or negotiate a migration window with consumers
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Contract ownership | Consumer owns and publishes the contract; provider verifies it |
| Verification gate | Provider contract verification must pass before any provider merge |
| Broker | Use a shared contract broker so provider can discover all consumers automatically |
| Schema changes | Safe = additive; breaking = requires versioning or migration window |
| Test scope | Contract tests complement integration tests; they do not replace them |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Provider writes the contract | Contract reflects what the provider does, not what consumers need | Consumer teams write and own their contracts |
| Contracts not updated when consumer changes | Stale contracts give false confidence | Update and republish the contract as part of every consumer PR that changes API usage |
| Verifying contracts only in staging | Breaking changes discovered too late | Run provider verification in CI on every PR, against the contract broker |
| Treating contract tests as a replacement for integration tests | Contract tests verify the interface; they do not test emergent behavior | Run both; use contracts for fast feedback and integration tests for end-to-end confidence |
| Skipping contract tests for "internal" APIs | Internal APIs break consumers just as often | Apply the same discipline to internal service boundaries |

## Related Skills

- **api-versioning** — contracts catch breaking changes before a new API version ships
- **loose-coupling** — contracts enforce discipline at service and module boundaries
- **code-review-discipline** — contract violations should block code review approval
