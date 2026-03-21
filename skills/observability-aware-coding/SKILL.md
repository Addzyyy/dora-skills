---
name: observability-aware-coding
description: Always apply when writing code that handles requests, calls external services, or makes business decisions — instrument every boundary
---

# Observability-Aware Coding

## Overview

Observability-aware coding means writing code that exposes its own internal state so that engineers can understand what is happening in production without attaching a debugger. The goal is to answer "what is the system doing right now, and why?" from dashboards and logs alone.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Change Failure Rate | Instrumented code catches regressions at the boundary before they become incidents |
| Mean Time to Restore (MTTR) | Rich signals cut diagnosis time from hours to minutes; teams locate the failing component immediately |

## When to Use

- Designing a new service or module that will run in production
- Adding a new integration with an external dependency (database, API, queue)
- Reviewing code that has previously been hard to debug in production
- Building background jobs, async workers, or scheduled tasks with no user-facing output

## When NOT to Use

- Prototype or throwaway code that will never reach production
- Unit test helpers or test fixtures
- One-off migration scripts run by hand under supervision

## Core Pattern

**Before — opaque service:**

```
request arrives
  → some logic runs
  → response returned (or not)
  → engineer has no idea what happened inside
```

**After — instrumented service:**

```
// Pseudocode — apply the same pattern in any language

// 1. Instrument every external boundary
result = call_database(query)
record_histogram("db.query.duration_ms", elapsed, tags=["query:user_lookup"])
increment_counter("db.query.total", tags=["status:success"])

// 2. Enrich errors with context
if error:
    increment_counter("db.query.total", tags=["status:error", "error:timeout"])
    raise EnrichedError(
        message="user lookup failed",
        context={user_id: id, query: query, elapsed_ms: elapsed}
    )

// 3. Expose health and readiness
GET /health   → {status: "ok"}
GET /ready    → {status: "ok", checks: {db: "ok", cache: "ok"}}
GET /metrics  → Prometheus-format counters, gauges, histograms
```

## What to Instrument

| Boundary | What to Measure |
|----------|----------------|
| Inbound requests | Count, duration, status code |
| Outbound calls (DB, API, queue) | Count, duration, error rate |
| Business decisions | Event count per outcome (e.g., `payment.result: success/decline`) |
| Background jobs | Start, finish, duration, items processed, errors |
| Queue consumers | Queue depth, processing lag, DLQ size |

## Metric Types

| Type | Use For | Example |
|------|---------|---------|
| Counter | Things that happen | `requests.total`, `errors.total` |
| Gauge | Current state | `queue.depth`, `connections.active` |
| Histogram | Distributions | `request.duration_ms` (enables p50/p95/p99) |

## Quick Reference

| Rule | Guidance |
|------|----------|
| Instrument boundaries | Measure every call that crosses a process or network boundary |
| Use histograms for latency | Averages hide tail latency; use p95/p99 |
| Enrich errors with context | Log what the code was trying to do, not just the exception class |
| Health vs. readiness | `/health` = process alive; `/ready` = safe to receive traffic |
| Tag consistently | Establish a team convention for tag names before shipping |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Logging only on error | No baseline; can't distinguish degradation from silence | Log success and failure at every boundary |
| Using averages for latency | p99 slowdowns are invisible in the average | Use histograms; alert on p95/p99 |
| Missing context in errors | "database error" is not actionable | Include query, input IDs, elapsed time, and error code |
| No readiness endpoint | Load balancer routes to a service that isn't ready | Add `/ready` that checks all dependencies |
| Instrumenting inside loops | Counter increments per item, not per operation | Measure at the boundary of the operation, not each iteration |
