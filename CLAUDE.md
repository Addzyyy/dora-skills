# DORA Engineering Practices

This plugin enforces engineering practices that improve DORA metrics. These are standing instructions — not suggestions.

## Before Writing Code

1. **Load the relevant skill first.** Check the router below and load the skill that matches your activity before touching any implementation file.
2. **Write the test first.** No implementation code exists before a failing test. This is non-negotiable — load `test-driven-development` and follow the RED-GREEN-REFACTOR cycle.
3. **Create a feature branch.** Never commit directly to main. Create a short-lived branch (e.g., `feat/add-shipping-calc`) before the first commit.
4. **Stop and ask if something is unclear.** If the requirements are ambiguous, the codebase contradicts what you expected, or the task is bigger than it seemed — stop implementing and ask for clarification before proceeding. A 30-second question saves hours of rework. Load `stop-and-clarify` when in doubt.

## After Each Passing Change

1. **Commit immediately.** One logical change per commit. If the message needs "and", split it.
2. **Push to the branch.** Keep the feedback loop tight.
3. **Run dora-review agent.** It checks your changes against DORA practices and fixes issues.
4. **Only then start the next piece of work.**

## Skill Router — Load Based on Activity

| You are about to... | Load this skill |
|---------------------|----------------|
| Write any code | `test-driven-development` |
| Commit changes | `small-incremental-commits` |
| Create/merge branches | `trunk-based-development` |
| Open or review a PR | `small-pull-requests`, `code-review-discipline` |
| Design module/service boundaries | `loose-coupling` |
| Add or change API endpoints | `api-versioning`, `contract-testing` |
| Write database migrations | `backward-compatible-migrations` |
| Add logging or error handling | `structured-logging-and-tracing`, `observability-aware-coding` |
| Manage config or env values | `configuration-as-code` |
| Plan a deployment or release | `rollback-friendly-design`, `feature-flags` |
| Add or update dependencies | `dependency-management` |
| Set up linting or type checking | `type-safety-and-linting` |
| Encounter ambiguity, spec gaps, or unexpected state | `stop-and-clarify` |

## Agent Checkpoints

| When | Run this |
|------|----------|
| Start of a new project or major task | `dora-health-check` agent |
| After completing a set of changes | `dora-review` agent |
| When a specific DORA metric needs improvement | `dora-improve` agent with the metric name |

## Why This Matters

Teams that follow these practices deploy more frequently, with shorter lead times, lower failure rates, and faster recovery. Speed and stability are not tradeoffs — the research shows elite teams achieve both. The process enables the speed, not the other way around.
