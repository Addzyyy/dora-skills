---
name: code-review-discipline
description: Apply when reviewing PRs, giving code feedback, preparing code for review, or self-reviewing before opening a PR. Checks correctness, security, maintainability, and test coverage systematically
---

# Code Review Discipline

## Overview

A code review is the last human checkpoint before code reaches production. When reviews collapse into rubber-stamp "LGTM" approvals, defects slip through that tests never catch: logic errors, security gaps, missing edge cases, and unmaintainable designs. Structured reviews catch these before they become incidents.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Disciplined reviews catch defects before deployment; fewer changes cause production incidents |

## When to Use

- Reviews are approved within minutes with no comments
- Reviewers only comment on style and formatting, never on logic or correctness
- The same class of bug appears in production repeatedly
- New team members are onboarding and need calibration on quality standards
- A pull request arrives with no description and no test changes

## When NOT to Use

- Trivial changes: fixing a typo in a comment or updating a version string — a sanity check is enough
- Auto-generated code that no human maintains directly
- Time-critical hotfixes where a senior engineer has already pair-programmed the fix — apply discipline after, in the follow-up cleanup PR

## Core Pattern

**Before — rubber-stamp review:**

```
PR #88 "add payment retry logic"
  Reviewer A: "LGTM 👍" (approved after 4 minutes)
  Reviewer B: "Looks good!" (approved after 2 minutes)
  → Merged. Deployed. Retries fire on non-retryable errors.
  → Production incident: duplicate charges.
```

The reviewers never asked: what happens on a network timeout vs. a card decline? Are retries idempotent?

**After — structured review:**

```
PR #88 "add payment retry logic"

Reviewer checklist:
  [x] Correctness   — Does retry logic distinguish retryable from non-retryable errors?
  [x] Security      — Are payment amounts validated before each retry attempt?
  [x] Maintainability — Is the retry policy configurable without a code change?
  [x] Test coverage — Are duplicate-charge and partial-failure scenarios tested?

Reviewer comment: "What stops this from retrying a card-declined error?
  That would generate duplicate charges. The retryable error list needs
  to be explicit, not opt-out."

Author addresses feedback → re-review → merge.
```

**Review checklist:**

```
[ ] Correctness: Does the code do what the description says, including edge cases?
[ ] Security: Are inputs validated? Are secrets handled safely? Any injection surface?
[ ] Maintainability: Will the next engineer understand this in six months?
[ ] Test coverage: Are the happy path, error paths, and edge cases covered?
[ ] Scope: Does this PR do exactly one thing, or is something unrelated hiding here?
```

**Actionable feedback format:**

```
Avoid:  "This looks wrong."
Prefer: "If `user` is null here, this throws before the null check on line 42.
         Add a guard at the top of the function, or handle it in the caller."

Avoid:  "Nit: rename this."
Prefer: "Nit: `d` → `deploymentDate` — the abbreviation is ambiguous in this context."
```

**Time-boxing:** Block 20–30 minutes per review session. If the PR is too large to review in that window, request a split rather than rushing.

## Quick Reference

| Rule | Guidance |
|------|----------|
| Review dimensions | Correctness, security, maintainability, test coverage — check all four |
| Feedback format | State the problem, explain the risk, suggest the fix |
| Approval threshold | No open correctness or security comments; style nits may be left to author |
| Time box | 20–30 minutes; request a split if the PR exceeds that |
| Turnaround | Same business day; reviews that sit 24 hours block deployment flow |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Approving to unblock rather than because it is ready | Defers risk to production instead of eliminating it | Block the merge; leave a specific comment on what must change |
| Only reviewing style and formatting | Logic bugs and security issues go uncaught | Work through the checklist; style is last, not first |
| Leaving vague feedback ("this is confusing") | Author cannot act on it | Name the specific line, explain the risk, suggest a concrete alternative |
| Reviewing a 1000-line PR in one pass | Attention degrades; the second half gets a rubber stamp | Request the PR be split; review each piece with fresh eyes |
| Never approving without every nit resolved | Slows delivery without improving reliability | Distinguish blocking issues (correctness, security) from non-blocking nits |

## Related Skills

- **small-pull-requests** — smaller PRs enable deeper, more focused reviews
- **test-driven-development** — reviewers verify that tests cover the intended behavior, not just the implementation
- **type-safety-and-linting** — automated static checks free reviewers to focus on logic and correctness
