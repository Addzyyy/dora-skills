---
name: loose-coupling
description: Always apply when designing module or service boundaries — enforce failure isolation, explicit interfaces, and no shared state
---

# Loose Coupling

## Overview

Loose coupling means a service can fail, restart, or change without forcing the same outcome on its callers or dependents. Tightly coupled systems fail together; loosely coupled systems fail in isolation. The goal is to confine damage so one broken component degrades a feature, not the entire system.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Mean Time to Restore (MTTR) | Failures stay local; teams restore one service without a system-wide incident |

## When to Use

- Designing a new service that will be called by other services
- A downstream failure is causing upstream errors (cascade is happening now)
- Adding a new dependency that could be slow or unreliable
- A single deploy requires coordinating multiple teams or repositories simultaneously

## When NOT to Use

- A single-process application where splitting introduces more overhead than benefit
- Two services that truly share a transaction boundary (prefer keeping them together)
- Early-stage systems where the domain boundaries are not yet understood — premature splitting creates the wrong seams

## Core Pattern

**Before — tightly coupled, failures cascade:**

```
User Request
  → Order Service
      → Payment Service (times out)
          → entire Order Service thread pool exhausted
              → all order requests fail, including ones that don't need payment
```

**After — isolated with clear boundaries:**

```
User Request
  → Order Service
      → calls Payment Service with timeout + circuit breaker
          if payment unavailable:
              return degraded response ("payment temporarily unavailable")
              queue payment for async retry
          order service continues accepting other requests normally
```

## Interface Design

```
// Pseudocode — define contracts explicitly

// Good: service exposes a versioned, minimal interface
interface PaymentPort:
    charge(order_id, amount_cents, currency) -> Result<ChargeId, PaymentError>

// Bad: service exposes its internal model
interface PaymentService:
    processOrderWithLineItemsAndTaxAndShipping(order: OrderDomainObject) -> void
```

Keep interfaces narrow. Depend on behaviors, not on internal data structures.

## Failure Isolation Patterns

| Pattern | What It Does | When to Use |
|---------|-------------|-------------|
| Circuit breaker | Stops calling a failing dependency; fails fast and recovers | Any synchronous call to an external service |
| Bulkhead | Separate thread/connection pools per dependency | Prevent one slow dependency from exhausting shared resources |
| Timeout | Bound how long a call can block | Every outbound network call, no exceptions |
| Graceful degradation | Return a reduced but valid response when a dependency is down | Features with non-critical dependencies |
| Async messaging | Decouple sender from receiver via a queue | Work that does not need an immediate response |

## Dependency Direction

```
// Good: depend on abstractions, not concrete services
OrderService depends on PaymentPort (interface)
PaymentAdapter implements PaymentPort → calls Stripe API

// Bad: depend on the implementation directly
OrderService imports StripeClient and calls it inline
```

Dependencies should point inward (toward core domain logic), never outward toward infrastructure details.

## Quick Reference

| Rule | Guidance |
|------|----------|
| Set timeouts everywhere | Every outbound call must have a timeout; no exceptions |
| Use circuit breakers | Prevent retry storms from amplifying a dependency failure |
| Define explicit interfaces | Publish what you provide; hide how you do it |
| Avoid shared databases | Two services sharing a DB are coupled at the schema level |
| Prefer async for non-critical work | Queues absorb spikes and decouple availability |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| No timeouts on outbound calls | One slow dependency blocks all threads | Add timeouts to every HTTP, DB, and RPC call |
| Shared database between services | Schema change in one service breaks the other | Each service owns its own data store |
| Distributed monolith | Services are split but deploy together and fail together | Enforce independent deployability; break shared libraries |
| Calling services in critical path that are non-critical | Recommendation engine failure breaks checkout | Move non-critical calls off the critical path or degrade gracefully |
| No graceful degradation | All-or-nothing responses; partial failures become total failures | Define what a reduced response looks like for each dependency |
