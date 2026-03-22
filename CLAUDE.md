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
4. **Open a PR.** Do not ask the user — just open it. Use `gh pr create` with a clear title and description.
5. **Keep going.** Do not stop between modules. Create the next feature branch off the current one and continue working. PRs are merged bottom-up as they get approved.

## Stacked PR Workflow (Default for Multi-Module Tasks)

When a task involves multiple modules, features, or logical units of work, use stacked PRs — do not stop between them:

1. Create `feat/first-module` branch from main
2. TDD, implement, commit, push, dora-review, open PR
3. Create `feat/second-module` branch **off the current branch** (not main)
4. TDD, implement, commit, push, dora-review, open PR targeting the previous branch
5. Repeat until all modules are complete — never pause to wait for review

Each PR is small, focused, and independently reviewable. They merge bottom-up.

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

## Parallel Agents

When a task involves independent pieces of work, use multiple agents to work in parallel. For example:
- Writing tests for module A while implementing module B (if the interface is already defined)
- Running `dora-review` on completed changes while starting TDD on the next module
- Loading and applying multiple skills simultaneously when they cover independent concerns (e.g., `structured-logging-and-tracing` for logging code while `contract-testing` for API boundaries)
- Running `dora-health-check` in the background while beginning work on the first module

The key constraint: each agent must work on independent files/concerns. Do not parallelize work that has dependencies between agents.

## Agent Checkpoints

| When | Run this |
|------|----------|
| Start of a new project or major task | `dora-health-check` agent |
| After completing a set of changes | `dora-review` agent |
| When a specific DORA metric needs improvement | `dora-improve` agent with the metric name |

## Why This Matters

Teams that follow these practices deploy more frequently, with shorter lead times, lower failure rates, and faster recovery. Speed and stability are not tradeoffs — the research shows elite teams achieve both. The process enables the speed, not the other way around.
