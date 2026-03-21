# dora-skills

A collection of 17 Claude Code skills focused on engineering practices that improve [DORA metrics](https://dora.dev/). Use these skills to get actionable, context-aware guidance on shipping software faster and more reliably.

---

## What Are DORA Metrics?

DORA (DevOps Research and Assessment) defines four key metrics that measure software delivery performance:

| Metric | What It Measures |
|---|---|
| **Deployment Frequency** | How often code ships to production |
| **Lead Time for Changes** | Time from commit to production |
| **Change Failure Rate** | Percentage of deploys that cause failures |
| **Mean Time to Restore (MTTR)** | Time to recover from a production failure |

High-performing teams deploy frequently, ship fast, break things rarely, and recover quickly. The skills in this repo target the specific practices that move those numbers.

---

## Skill Index

| # | Skill | Description |
|---|---|---|
| 1 | `dora-overview` | Router skill that maps DORA metrics to the practices most likely to improve them |
| 2 | `small-incremental-commits` | One logical change per commit — keeps history readable and rollbacks clean |
| 3 | `trunk-based-development` | Short-lived branches merged to main frequently, avoiding long-running divergence |
| 4 | `feature-flags` | Decouple deployment from release to ship safely and enable gradual rollouts |
| 5 | `configuration-as-code` | Keep config versioned and reviewable alongside the code that uses it |
| 6 | `small-pull-requests` | Focused, reviewable PRs that reduce review lag and merge risk |
| 7 | `test-driven-development` | RED-GREEN-REFACTOR cycle to drive design and catch regressions early |
| 8 | `code-review-discipline` | Structured, meaningful reviews that improve quality without slowing delivery |
| 9 | `type-safety-and-linting` | Catch bugs at compile/lint time before they reach production |
| 10 | `contract-testing` | Consumer-driven API validation to prevent breaking changes across services |
| 11 | `dependency-management` | Managed dependency hygiene to reduce supply-chain risk and upgrade pain |
| 12 | `api-versioning` | Evolve APIs without breaking consumers — additive changes and deprecation strategies |
| 13 | `observability-aware-coding` | Instrument code for production debuggability from day one |
| 14 | `structured-logging-and-tracing` | Structured logs, correlation IDs, and traces for fast incident diagnosis |
| 15 | `loose-coupling` | Isolated services and failure boundaries that contain blast radius |
| 16 | `rollback-friendly-design` | Design deployments so rollback is safe, fast, and a real option |
| 17 | `backward-compatible-migrations` | Expand-contract DB migrations that support zero-downtime deploys |

---

## Installation

### Option 1: Symlink skills into Claude's skill directory

```bash
git clone <repo-url> ~/dora-skills
ln -s ~/dora-skills/skills/* ~/.claude/skills/
```

### Option 2: Reference via CLAUDE.md

Add the following line to your project's `CLAUDE.md` to make the skills available in that project:

```
@path/to/dora-skills/skills/
```

Replace `path/to/dora-skills` with the actual path where you cloned this repo (e.g., `~/dora-skills`).

### Option 3: Copy skills directly

Copy individual skill files from `skills/` into `~/.claude/skills/` to pick only the ones you want.

---

## Usage

Once installed, invoke skills by name in Claude Code:

```
/dora-overview
/small-pull-requests
/test-driven-development
```

Start with `/dora-overview` if you're unsure which practice to focus on — it will ask about your current pain points and route you to the most relevant skill.

---

## Project Purpose

This repo exists to make DORA-aligned engineering practices accessible and actionable through Claude Code. Rather than reading a doc and hoping it sticks, each skill guides you through applying a practice in the context of real work you're already doing.

The skills are designed for personal use but written to be shareable. Contributions and forks welcome.
