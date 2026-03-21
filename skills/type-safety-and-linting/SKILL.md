---
name: type-safety-and-linting
description: Use when setting up or improving static analysis, type checking, or linting rules
---

# Type Safety and Linting

## Overview

Static analysis catches bugs before a human runs the code. Type errors, null dereferences, unreachable branches, and violated domain invariants are all detectable at write-time or commit-time — not in production. Treating the type checker and linter as first-class gatekeepers eliminates an entire category of preventable failures.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Compile-time and lint-time catches prevent type errors, null dereferences, and invariant violations from reaching production |

## When to Use

- A production bug was caused by a wrong type, a null value, or an unhandled case
- New code is added without type annotations, leaving the checker with nothing to verify
- Linting is configured but not enforced in CI — warnings accumulate and are ignored
- The codebase is being migrated from a dynamically typed to a statically typed language or stricter mode
- A domain rule (e.g., "amount must be positive") is enforced only at runtime

## When NOT to Use

- A one-off script with no consumers and a lifespan under a week — strict typing adds friction without payoff
- Auto-generated code that is regenerated on every build — annotate the generator, not the output
- When adding strict mode to a large legacy codebase all at once; incremental adoption is safer (see Core Pattern)

## Core Pattern

**Before — runtime failure:**

```
function applyDiscount(order, discountCode) {
  return order.total * discountCode.rate   // crashes if discountCode is null
}

// Called elsewhere:
applyDiscount(order, lookupDiscount(code))  // lookupDiscount returns null on miss
// → TypeError in production on the first invalid code
```

**After — caught at write-time:**

```
// Type annotations make the null case explicit
function applyDiscount(order: Order, discountCode: DiscountCode | null): Money {
  if (discountCode === null) return order.total   // forced to handle null
  return order.total * discountCode.rate
}
// Type checker flags any caller that passes a possibly-null value without a guard
```

**Strict mode defaults:** Enable the strictest available mode from day one on new code. Strict mode catches nullability, implicit any/unknown, and unhandled union variants automatically.

**Incremental adoption on existing code:**

```
Phase 1: Enable lint rules with warnings; fix errors, leave warnings for now
Phase 2: Promote warnings to errors in CI on new files only (path-based config)
Phase 3: Fix warnings file-by-file as you touch existing code (boy scout rule)
Phase 4: Remove the exemption list once coverage reaches ~100%
```

**Custom rules for domain invariants:**

```
# Instead of runtime assertion:
  if amount <= 0: raise ValueError("amount must be positive")

# Encode in the type system:
  type PositiveAmount = private integer where value > 0
  constructor make(n: integer): PositiveAmount | Error

# The type checker now proves all amounts are positive at every call site
```

**Pre-commit hook enforcement:**

```
pre-commit hook:
  run type-check  → fail commit on type errors
  run lint        → fail commit on errors (warnings configurable)
  run formatter   → auto-fix formatting, re-stage
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Strict mode | Enable maximum strictness on all new code from the start |
| CI gate | Type check and lint must pass before merge; warnings are opt-in noise |
| Pre-commit | Run checks locally before push to catch issues early |
| Incremental | Add type coverage file-by-file as you touch existing code |
| Domain types | Encode invariants in types, not only in runtime assertions |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Disabling strict mode to silence errors | Hides exactly the bugs the checker was meant to find | Fix the underlying type error; use a narrow suppression with a comment if unavoidable |
| Lint warnings not enforced in CI | Warnings accumulate; no one fixes them | Promote errors to CI failures; keep warnings as opt-in local feedback only |
| Annotating only public APIs | Internal code still causes runtime failures | Apply type coverage uniformly; internal code is where most bugs originate |
| Using `any` / `unknown` to escape the type system | Removes the safety guarantee for the entire call chain downstream | Use a narrow type or a discriminated union; reserve escape hatches for genuinely dynamic boundaries |
| Adding type checks without pre-commit hooks | Engineers discover failures only after push, slowing feedback | Install hooks locally and in CI; fail fast at the earliest checkpoint |
