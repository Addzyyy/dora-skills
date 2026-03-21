---
name: dora-overview
description: Use when discussing software delivery performance, DORA metrics, deployment pipeline improvements, or engineering team effectiveness
---

# DORA Metrics Overview

DORA (DevOps Research and Assessment) research identified four key metrics that distinguish high-performing engineering teams. These metrics measure both throughput (how fast you deliver) and stability (how reliably you deliver).

## The 4 DORA Metrics

### Deployment Frequency
How often does your team deploy to production?

| Level | Benchmark |
|-------|-----------|
| Elite | On-demand / multiple times per day |
| High | Weekly |
| Medium | Monthly |
| Low | Less than monthly |

### Lead Time for Changes
How long from code commit to running in production?

| Level | Benchmark |
|-------|-----------|
| Elite | Less than 1 hour |
| High | Less than 1 day |
| Medium | Less than 1 week |
| Low | More than 1 month |

### Change Failure Rate
What percentage of deployments cause a degraded service or require remediation?

| Level | Benchmark |
|-------|-----------|
| Elite | Less than 5% |
| High | Less than 10% |
| Medium | Less than 15% |
| Low | Greater than 15% |

### Mean Time to Restore (MTTR)
How long to recover when a deployment causes an incident?

| Level | Benchmark |
|-------|-----------|
| Elite | Less than 1 hour |
| High | Less than 1 day |
| Medium | Less than 1 week |
| Low | More than 1 week |

## Practice-to-Metric Mapping

Each engineering practice improves specific metrics. Use this table to target your investment:

| Practice | Frequency | Lead Time | Failure Rate | MTTR |
|----------|:---------:|:---------:|:------------:|:----:|
| small-incremental-commits | X | X | | |
| trunk-based-development | X | X | | |
| feature-flags | X | | | X |
| configuration-as-code | X | | | X |
| small-pull-requests | | X | | |
| test-driven-development | | X | X | |
| code-review-discipline | | | X | |
| type-safety-and-linting | | | X | |
| contract-testing | | | X | |
| dependency-management | | X | X | |
| api-versioning | | | X | X |
| observability-aware-coding | | | X | X |
| structured-logging-and-tracing | | | | X |
| loose-coupling | | | | X |
| rollback-friendly-design | | | | X |
| backward-compatible-migrations | | | X | X |

## Self-Assessment: Which Metric Should You Focus On?

Answer these questions to identify where to invest first:

- **"Are deployments painful or infrequent?"** — Focus on **Deployment Frequency** skills: `small-incremental-commits`, `trunk-based-development`, `feature-flags`, `configuration-as-code`

- **"Does it take too long for code to reach production?"** — Focus on **Lead Time** skills: `small-pull-requests`, `test-driven-development`, `small-incremental-commits`, `trunk-based-development`, `dependency-management`

- **"Do deployments often cause incidents?"** — Focus on **Change Failure Rate** skills: `test-driven-development`, `code-review-discipline`, `type-safety-and-linting`, `contract-testing`, `observability-aware-coding`, `api-versioning`, `dependency-management`, `backward-compatible-migrations`

- **"Does it take too long to recover from incidents?"** — Focus on **MTTR** skills: `structured-logging-and-tracing`, `loose-coupling`, `rollback-friendly-design`, `feature-flags`, `configuration-as-code`, `observability-aware-coding`, `api-versioning`, `backward-compatible-migrations`

## Router Guidance

When a user's question maps to a specific metric, load the corresponding practice skills:

**Deployment Frequency**: Load `small-incremental-commits`, `trunk-based-development`, `feature-flags`, `configuration-as-code`

**Lead Time for Changes**: Load `small-pull-requests`, `small-incremental-commits`, `trunk-based-development`, `test-driven-development`, `dependency-management`

**Change Failure Rate**: Load `test-driven-development`, `code-review-discipline`, `type-safety-and-linting`, `contract-testing`, `api-versioning`, `observability-aware-coding`, `dependency-management`, `backward-compatible-migrations`

**MTTR**: Load `structured-logging-and-tracing`, `loose-coupling`, `rollback-friendly-design`, `feature-flags`, `configuration-as-code`, `observability-aware-coding`, `api-versioning`, `backward-compatible-migrations`

## How Skills Compose

Skills in this collection are **complementary, not conflicting**. You can load multiple practice skills simultaneously — they address different aspects of the same delivery pipeline.

**Specificity wins**: When a user's context is specific (e.g., "our canary deployments keep failing"), load the more-specific skill (`rollback-friendly-design`, `observability-aware-coding`) rather than staying at the overview level. This skill serves as the entry point; the practice skills provide the actionable depth.

**Each skill is self-contained**: Every practice skill works independently. Users don't need to read this overview to benefit from a specific practice skill. Load whichever skills match the problem at hand.
