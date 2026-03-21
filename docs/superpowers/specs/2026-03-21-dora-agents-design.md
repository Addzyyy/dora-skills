# DORA Agents Design

## Summary

Add three autonomous agents to the dora-skills plugin that actively analyze and improve codebases against DORA practices. Agents ship alongside the existing 17 skills — when someone installs the plugin they get both.

## Decisions

- **Scope:** Working tree (staged + unstaged changes) for `dora-review`; full repo for `dora-health-check` and `dora-improve`
- **Behavior:** Agents make changes directly and output a report of what they did
- **Model:** No default — inherits from parent session
- **Tools:** Full access (no restrictions)
- **Structure:** Standalone agents (one markdown file per agent, no shared base)
- **Approach A** selected: each agent is self-contained in `agents/<name>.md`

## Agent File Format

Each agent is a markdown file with YAML frontmatter:

```markdown
---
name: agent-name
description: One-line description of what the agent does
---

System prompt content here — this is the full agent prompt.
```

No `tools` or `model` fields — agents inherit full tool access and the user's model. The markdown body *is* the system prompt. Each agent's prompt references DORA practice principles (drawn from the relevant skills) but does not inline full skill content. Skills provide the deep reference; agents provide the actionable workflow. Since agents contain their own condensed guidance, skill updates do not require agent changes unless the core principles change.

## Plugin Structure Change

```
dora-skills/
├── .claude-plugin/
│   └── plugin.json          # add "agents": "./agents/"
├── skills/                   # existing 17 skills (unchanged)
└── agents/                   # NEW
    ├── dora-review.md
    ├── dora-health-check.md
    └── dora-improve.md
```

`plugin.json` gains one field:

```json
"agents": "./agents/"
```

The plugin description should be updated to:
```
"description": "Claude Code skills and agents for engineering practices that improve DORA metrics (Deployment Frequency, Lead Time, Change Failure Rate, MTTR)"
```

## Scope Distinction: dora-health-check vs dora-improve

These two agents both scan the full repo but serve different purposes:

- **dora-health-check** is a broad audit — it scores all 17 practices and makes only low-risk additive fixes (adding configs, scaffolding test directories, etc.). Its primary output is the scorecard and recommendations.
- **dora-improve** is a focused remediation — given one metric, it makes substantive code changes targeting that metric's practices. It goes deeper on fewer practices.

Running `dora-improve` four times is not equivalent to `dora-health-check`. The health check is lighter-touch and report-oriented; improve is heavier and change-oriented.

## dora-improve Input

The metric is passed as a natural language argument when invoking the agent. Valid values: `frequency`, `lead-time`, `failure-rate`, `mttr`. If the user provides an ambiguous or missing metric, the agent should ask which of the four metrics to target.

## Agent 1: dora-review

**Purpose:** Reviews working tree changes against DORA practices, fixes issues, outputs a report.

**Workflow:**
1. Run `git diff` (unstaged) and `git diff --cached` (staged) to get current changes
2. If no changes found, inform the user and exit
3. Analyze changes against DORA practices: commit hygiene, change size, coupling, testability, rollback safety, observability, config management
4. Make fixes directly — add missing structured logging, improve test coverage, refactor tightly-coupled code
5. All changes are left unstaged so the user can review them via `git diff` before staging
6. Output a report: what was found, what was changed, what couldn't be fixed automatically (with recommendations)

**Relevant skills:** small-incremental-commits, small-pull-requests, code-review-discipline, loose-coupling, rollback-friendly-design, observability-aware-coding, test-driven-development

**Example report:**
```
## DORA Review Report

### Changes Made
- Added structured logging to new API handler
- Wrapped new feature in feature flag check
- Added unit test for new validation logic

### Recommendations (manual)
- Consider splitting this into 2 PRs: the refactor and the new behavior
- 3 unrelated concerns in one commit — stage and commit separately
```

## Agent 2: dora-health-check

**Purpose:** Audits the entire repo's engineering practices against DORA principles, makes improvements where safe, outputs a full report.

**Workflow:**
1. Scan repo structure — test coverage patterns, config management, branching strategy (git log, git branch), dependency setup, logging patterns, CI/CD config
2. Score each DORA practice area using heuristic LLM judgment: present / partial / missing (based on evidence found in the repo — e.g., test files exist, structured log calls present, short-lived branches)
3. Make low-risk additive fixes — add missing test directories, add linting configs, improve logging patterns. Does not delete or restructure existing code
4. Output a health report with scores, changes made, and prioritized recommendations

**Relevant skills:** All 17 — this is the comprehensive audit agent

**Example report:**
```
## DORA Health Check Report

### Practice Scores
| Practice | Status | Notes |
|----------|--------|-------|
| Small commits | present | Avg 45 lines/commit last 30 days |
| Trunk-based dev | partial | 3 branches older than 7 days |
| Feature flags | missing | No flag framework detected |
| Test coverage | partial | Tests exist but no coverage config |
| Structured logging | missing | Using console.log throughout |

### Changes Made
- Added structured logging helper based on existing patterns
- Created lint rule config for consistent error handling

### Top 3 Recommendations
1. Adopt feature flags — highest impact for deployment frequency
2. Add structured logging — critical for MTTR
3. Merge or delete 3 stale branches — blocking trunk-based flow
```

## Agent 3: dora-improve

**Purpose:** Given a specific DORA metric, analyzes the codebase and makes targeted changes to improve that metric.

**Workflow:**
1. Accept a metric as input: `frequency`, `lead-time`, `failure-rate`, or `mttr`
2. Use the practice-to-metric mapping from dora-overview to identify which practices to focus on
3. Scan the codebase for gaps in those specific practices
4. Make targeted changes — focus only on practices that improve the chosen metric
5. Output a report: what was found, what was changed, and next steps

**Relevant skills:** Depends on the metric — uses the same mapping as dora-overview:
- **frequency:** small-incremental-commits, trunk-based-development, feature-flags, configuration-as-code
- **lead-time:** small-pull-requests, test-driven-development, small-incremental-commits, trunk-based-development, dependency-management
- **failure-rate:** test-driven-development, code-review-discipline, type-safety-and-linting, contract-testing, api-versioning, observability-aware-coding, dependency-management, backward-compatible-migrations
- **mttr:** structured-logging-and-tracing, loose-coupling, rollback-friendly-design, feature-flags, configuration-as-code, observability-aware-coding, api-versioning, backward-compatible-migrations

**Example report (for mttr):**
```
## DORA Improve Report: MTTR

### Target Practices
structured-logging-and-tracing, loose-coupling, rollback-friendly-design,
feature-flags, configuration-as-code, observability-aware-coding

### Findings
- No structured logging — using ad-hoc print/log statements
- No correlation IDs in request handling
- Deployment has no rollback mechanism
- Feature flags not in use

### Changes Made
- Added structured log format utility matching existing patterns
- Added correlation ID propagation to request middleware
- Wrapped 2 recently-added features in feature flag checks

### Next Steps (manual)
- Set up a rollback runbook or script for deployments
- Evaluate a feature flag service (LaunchDarkly, Unleash, etc.)
- Add health check endpoints for faster incident detection
```
