---
name: stop-and-clarify
description: Apply whenever you encounter ambiguity, spec gaps, conflicting requirements, unexpected codebase state, or anything that deviates from what was asked. Stop implementing and ask before proceeding. This prevents building the wrong thing, which is the biggest source of wasted work and failed changes. Use this when something feels off, when you're making assumptions, or when the task has grown beyond what was originally requested.
---

# Stop and Clarify

## Overview

The most expensive line of code is the one that solves the wrong problem. When you encounter ambiguity, a gap in requirements, or something in the codebase that contradicts what you expected, the instinct is to make an assumption and keep going. That instinct is wrong. A 30-second clarification question saves hours of rework — and rework is the #1 killer of lead time and the #1 source of failed changes.

This practice is borrowed from Toyota's production system, where any worker can pull the Andon cord to stop the assembly line when they spot a problem. It feels slower. It is faster — because defects caught at the source cost 10x less to fix than defects caught downstream.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Building the right thing the first time eliminates an entire class of "failed changes" caused by misunderstood requirements |
| Lead Time for Changes | A 30-second question prevents hours or days of rework; the fastest code is code you don't have to rewrite |

## When to Stop

Stop implementing and ask the user when you encounter any of these situations:

### Ambiguity in Requirements
- The task could be interpreted two or more reasonable ways
- A key detail was left unspecified (e.g., "validate the input" but which validations?)
- Business rules are implied but not stated (e.g., "calculate the total" — before or after tax?)

### Spec Gaps
- The task doesn't cover an edge case that will definitely occur (e.g., what happens with empty input? null? concurrent access?)
- The happy path is clear but error handling is unspecified
- The task references a concept or entity that doesn't exist yet in the codebase

### Deviations from What Was Asked
- The codebase is structured differently than the task assumes
- An existing function already does part of what was asked (should you extend it or create a new one?)
- The task asks for a change that would break existing tests or contracts
- What seemed like a small change is actually touching a critical path

### Scope Creep Detection
- While implementing, you discover the task is larger than it appeared
- You need to refactor existing code to make the change fit, but that wasn't part of the ask
- You're about to touch files or modules outside the scope of the original request

### Conflicting Signals
- The task description conflicts with existing code comments, documentation, or test expectations
- Two parts of the codebase handle the same concern differently, and it's unclear which pattern to follow
- A dependency or API behaves differently than the task assumes

## How to Ask

When you stop, be specific about what you found and why it matters. Don't just say "I have a question." Frame it so the user can make a quick decision.

**Good — specific, actionable, shows what you found:**
> I found that `calculateTotal()` in `src/billing.ts` already handles tax calculation, but the task asks me to add tax calculation to the new `checkout` module. Should I:
> (a) reuse the existing `calculateTotal()`, or
> (b) create a new one in the checkout module?
> Option (a) avoids duplication but couples checkout to billing.

**Bad — vague, forces the user to investigate:**
> I'm not sure how to handle the tax calculation. What should I do?

**Structure your clarification as:**
1. **What you found** — the specific ambiguity, gap, or deviation
2. **Why it matters** — what could go wrong if you guess
3. **Options** — 2-3 concrete choices, with tradeoffs if relevant

## When NOT to Stop

Not every uncertainty requires a question. Use judgment:

- **Trivial decisions** with no meaningful tradeoff (variable naming, minor formatting) — just pick one
- **Standard patterns** the codebase already establishes (if all services use the same error format, use it)
- **Things you can verify yourself** by reading existing code, tests, or documentation
- **Implementation details** that don't affect behavior (which algorithm to use when both are correct)

The test: **if you guess wrong, would the user need to redo significant work?** If yes, ask. If no, proceed.

## Quick Reference

| Situation | Action |
|-----------|--------|
| Two reasonable interpretations | Stop and ask which one |
| Edge case not covered in requirements | Stop and ask how to handle it |
| Codebase contradicts the task | Stop and flag the contradiction |
| Task is bigger than it looked | Stop and confirm scope |
| Existing code already does something similar | Stop and ask: extend or create new? |
| You're about to touch unrelated code | Stop and confirm it's in scope |
| Trivial choice with no real impact | Just pick one and move on |
| Codebase has an established pattern | Follow the pattern, don't ask |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Asking about every tiny decision | Slows the user down with decision fatigue | Reserve questions for choices that affect correctness or architecture |
| Making assumptions to avoid "bothering" the user | Builds the wrong thing; rework costs 10x more than the interruption | A quick clarification is always cheaper than a rewrite |
| Asking vague questions | User can't answer without investigating themselves | Include what you found, why it matters, and concrete options |
| Powering through ambiguity to "stay productive" | Productivity is measured by outcomes, not activity — wrong code is negative productivity | Stop the line; the 30-second pause saves hours |
| Not flagging scope creep | Small task silently becomes large; commit discipline breaks down | When scope grows, pause and confirm before expanding |

## Related Skills

- **small-incremental-commits** — stopping to clarify naturally creates commit boundaries
- **small-pull-requests** — catching scope creep early keeps PRs focused
- **code-review-discipline** — the same mindset applies in review: flag ambiguity, don't rubber-stamp
- **test-driven-development** — writing the test first often reveals spec gaps before implementation begins
