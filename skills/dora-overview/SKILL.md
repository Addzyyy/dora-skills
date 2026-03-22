---
name: dora-overview
description: Load at the START of any coding session, project planning, architecture decision, code review, or deployment planning — this is the master skill that routes to all DORA practices and agents; when in doubt, load this first
---

# DORA Practices — Active Enforcement

**This is not reference material. These are standing instructions.**

When this skill is loaded, you MUST actively apply DORA practices throughout the session. Do not wait to be asked. Do not treat these as suggestions. Load the specific practice skills listed below based on what you are doing, and run the agents at the checkpoints described.

## Standing Rules

1. **Write tests first.** Load `test-driven-development` before writing any implementation code.
2. **Commit after every passing change.** Do not batch multiple modules into one commit.
3. **Push immediately after committing.** Keep the feedback loop tight.
4. **Load relevant skills before starting work.** See the router table below.
5. **Run `dora-review` agent after completing any set of changes.** Do not skip this.
6. **Run `dora-health-check` agent at the start of a new project or major task.**
7. **Never optimize for speed over process.**

## The 4 DORA Metrics

DORA (DevOps Research and Assessment) research identified four key metrics that distinguish high-performing engineering teams. These metrics measure both throughput (how fast you deliver) and stability (how reliably you deliver).

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
| stop-and-clarify | | X | X | |

## Self-Assessment: Which Metric Should You Focus On?

Answer these questions to identify where to invest first:

- **"Are deployments painful or infrequent?"** — Focus on **Deployment Frequency** skills: `small-incremental-commits`, `trunk-based-development`, `feature-flags`, `configuration-as-code`

- **"Does it take too long for code to reach production?"** — Focus on **Lead Time** skills: `small-pull-requests`, `test-driven-development`, `small-incremental-commits`, `trunk-based-development`, `dependency-management`

- **"Do deployments often cause incidents?"** — Focus on **Change Failure Rate** skills: `test-driven-development`, `code-review-discipline`, `type-safety-and-linting`, `contract-testing`, `observability-aware-coding`, `api-versioning`, `dependency-management`, `backward-compatible-migrations`

- **"Does it take too long to recover from incidents?"** — Focus on **MTTR** skills: `structured-logging-and-tracing`, `loose-coupling`, `rollback-friendly-design`, `feature-flags`, `configuration-as-code`, `observability-aware-coding`, `api-versioning`, `backward-compatible-migrations`

## Skill Router — Load These Automatically

Do not wait for the user to ask. When you detect any of these activities, load the corresponding skills immediately:

| Activity | Load these skills |
|----------|-------------------|
| Writing any new code | `test-driven-development` |
| Making commits | `small-incremental-commits` |
| Creating or merging branches | `trunk-based-development` |
| Opening or reviewing a PR | `small-pull-requests`, `code-review-discipline` |
| Designing module or service boundaries | `loose-coupling` |
| Adding or changing API endpoints | `api-versioning`, `contract-testing` |
| Writing database migrations | `backward-compatible-migrations` |
| Adding logging or error handling | `structured-logging-and-tracing`, `observability-aware-coding` |
| Managing config or environment values | `configuration-as-code` |
| Planning a deployment or release | `rollback-friendly-design`, `feature-flags` |
| Adding or updating dependencies | `dependency-management` |
| Setting up linting or type checking | `type-safety-and-linting` |
| Encountering ambiguity, spec gaps, or unexpected state | `stop-and-clarify` |

## Agent Checkpoints — Run These Automatically

| Checkpoint | Run this agent |
|------------|----------------|
| After completing a set of changes | `dora-review` |
| Start of a new project or major task | `dora-health-check` |
| When a specific DORA metric needs improvement | `dora-improve` with the metric name |

## How Skills Compose

Skills are **complementary, not conflicting**. Load multiple skills simultaneously — they address different aspects of the same delivery pipeline. When the user's context is specific, load the most specific skill rather than staying at this overview level.
