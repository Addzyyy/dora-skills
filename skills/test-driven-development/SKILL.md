---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development

## Overview

Test-driven development (TDD) is a discipline where you write a failing test before you write any implementation. The test describes the desired behavior; the implementation is written only to make that test pass. This inverts the usual order: instead of tests verifying code that already exists, tests define what code must do before it exists.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Lead Time for Changes | Fast, trustworthy tests let developers merge with confidence and skip slow manual verification |
| Change Failure Rate | Defects are caught at the unit level, before they reach review, staging, or production |

## When to Use

- Implementing any new feature, however small
- Fixing a bug — write a test that reproduces the bug before touching the implementation
- Refactoring — existing tests must already cover the code being restructured
- Integrating with an external dependency — write tests against the expected contract first

## When NOT to Use

- Pure UI layout and styling where behavior is visual and subjective
- Exploratory spikes written to learn an API or architecture — delete the spike, then TDD the real implementation
- Auto-generated code (migrations, serializers) where the generator provides the test

## Core Pattern

**Before — write code, then retrofit tests:**

```
1. Write implementation until it "feels right"
2. Open a test file
3. Try to test the implementation
4. Discover the implementation is hard to isolate → patch it
5. Write tests that match what the code already does (not what it should do)
6. Ship code and tests together

Result: tests verify the implementation, not the requirement.
         Coverage exists, but wrong behavior is also covered.
```

**After — RED-GREEN-REFACTOR cycle:**

```
RED:     Write a failing test that describes one behavior
         Run it → confirm it fails for the right reason (not a syntax error)

GREEN:   Write the minimum code that makes the test pass
         No extra logic, no "while I'm here" additions
         Run the test → it passes

REFACTOR: Clean up the implementation without changing behavior
          Tests stay green throughout
          Then repeat for the next behavior
```

**Concrete example — a price calculation function:**

```
Cycle 1 — basic case
  RED:    test("returns item price when no discount applies")
  GREEN:  function totalPrice(item) { return item.price }
  REFACTOR: nothing to clean yet

Cycle 2 — discount case
  RED:    test("applies percentage discount when item is on sale")
  GREEN:  add discount branch to totalPrice
  REFACTOR: extract discount logic to named helper

Cycle 3 — edge case
  RED:    test("returns zero when item price is negative")
  GREEN:  add guard clause
  REFACTOR: consolidate guard clauses
```

Each cycle takes 2–5 minutes. The function grows only as far as the tests demand.

**Anatomy of a good failing test:**

```
GIVEN:  a specific, controlled starting state
WHEN:   one action or call is made
THEN:   one observable outcome is asserted

test("rejects transfer when balance is insufficient") {
  account = Account(balance: 50)           // GIVEN
  result  = account.transfer(amount: 100)  // WHEN
  assert result.error == "insufficient_funds"  // THEN
}
```

One assertion per test. If a test can fail for two different reasons, split it.

## Quick Reference

| Rule | Guidance |
|------|----------|
| Test first | No implementation line exists before its corresponding failing test |
| Minimum green | Write exactly enough code to pass the test; no more |
| One cycle at a time | Finish RED-GREEN-REFACTOR before adding the next test |
| Test the behavior | Assert outcomes visible to callers, not internal state |
| Bug fix entry point | Reproduce the bug in a failing test before touching the fix |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Writing multiple tests before any implementation | Skips the feedback of watching each one fail and pass | One test, one cycle |
| Writing tests that always pass | Tests that cannot fail give no information | Run the test before writing implementation; confirm it is red |
| Testing internal implementation details | Tests break on every refactor even when behavior is unchanged | Assert only on public outputs and side effects |
| Skipping the refactor step | Code passes tests but accumulates complexity | Treat refactor as mandatory; tests are the safety net for it |
| Writing the implementation "temporarily" before the test | The test is retrofitted; intent is lost | Commit to test-first on every cycle, including small ones |
| Large test setups shared across many tests | A change to setup breaks unrelated tests | Keep each test's setup local and minimal |
