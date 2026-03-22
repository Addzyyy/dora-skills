---
name: structured-logging-and-tracing
description: Apply when adding console.log, print, logger calls, error handling, debug output, or any observability — use structured JSON logs with trace/correlation IDs, never free-form strings; covers distributed tracing and log aggregation
---

# Structured Logging and Tracing

## Overview

Structured logging replaces freeform text strings with machine-parseable records. Distributed tracing links those records across services using a shared trace ID. Together they let engineers search, filter, and follow a request across an entire system — without guessing which log lines belong together.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Mean Time to Restore (MTTR) | Engineers find the exact failing request in seconds by filtering on trace ID; correlation cuts cross-service diagnosis from hours to minutes |

## When to Use

- Implementing or updating log output in any service
- Adding a new service to a distributed system that must be debuggable end-to-end
- Designing how requests will flow across service boundaries
- Investigating slow or error-prone incidents where the root cause is unclear

## When NOT to Use

- Simple single-process scripts where a human reads the output directly
- Batch jobs that run locally under direct supervision
- Replacing an existing structured logging library that already works — extend it instead

## Core Pattern

**Before — unstructured logs:**

```
[INFO] Processing order 12345
[ERROR] Payment failed!!! retry #2
[INFO] Done
```

These lines cannot be queried by order ID, cannot be correlated across services, and contain no timing information.

**After — structured logs with correlation:**

```
// Pseudocode log schema — emit as JSON

{
  "timestamp": "2026-03-21T14:02:33.412Z",  // ISO 8601 UTC
  "level": "info",                            // debug | info | warn | error
  "service": "order-service",
  "version": "1.4.2",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",  // W3C Trace Context
  "span_id": "00f067aa0ba902b7",
  "message": "payment attempted",
  "context": {
    "order_id": "12345",
    "amount_cents": 4999,
    "payment_provider": "stripe",
    "duration_ms": 143
  }
}
```

## Trace Context Propagation

```
// On every outbound HTTP call, inject W3C Trace Context headers:
traceparent: 00-{trace_id}-{span_id}-01

// On every inbound request, extract those headers before logging:
incoming_trace_id = request.header("traceparent").trace_id OR generate_new()
attach trace_id to all logs and outbound calls for this request
```

A single `trace_id` threads through every service. Search any log store for that ID and see every log line, in order, across all services.

## Log Levels Guidance

| Level | When to Use |
|-------|------------|
| debug | Internal state useful during development; off in production by default |
| info | Normal operations — request received, job started, significant state changes |
| warn | Unexpected but recoverable — retry succeeded, degraded mode active |
| error | Operation failed and action is required — alert on this level |

## Language Examples

### TypeScript (pino)

```typescript
import pino from 'pino'

const logger = pino({ level: 'info' })

// Structured log with context
function handleOrder(orderId: string, traceId: string) {
  const log = logger.child({ trace_id: traceId, order_id: orderId })

  log.info({ action: 'order_processing_started' }, 'Processing order')

  try {
    const result = chargePayment(orderId)
    log.info({ action: 'payment_charged', duration_ms: result.elapsed }, 'Payment successful')
  } catch (err) {
    log.error({ action: 'payment_failed', error: err.message }, 'Payment failed')
    throw err
  }
}
```

### Python (structlog)

```python
import structlog

logger = structlog.get_logger()

def handle_order(order_id: str, trace_id: str) -> None:
    log = logger.bind(trace_id=trace_id, order_id=order_id)

    log.info("order_processing_started")

    try:
        result = charge_payment(order_id)
        log.info("payment_charged", duration_ms=result.elapsed)
    except Exception as err:
        log.error("payment_failed", error=str(err))
        raise
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Always emit JSON | Use a logging library; never build log strings by hand |
| Include trace_id on every line | Without it, cross-service correlation is impossible |
| Use W3C Trace Context | Standard header: `traceparent` — compatible with all major tracing systems |
| Log at boundaries | Request in/out, external calls, significant decisions |
| Propagate context, don't re-create | Pass the trace ID through; never generate a new one mid-request |

## What NOT to Log

| Category | Risk | Alternative |
|----------|------|-------------|
| Passwords, API keys, tokens | Credential exposure in log stores | Log that auth was attempted, not the credential |
| PII (email, SSN, address) | Privacy / compliance violation | Log a user ID or hashed identifier |
| Full request/response bodies | Large volume, may contain secrets | Log size, content-type, and status — sample bodies only in debug |
| Credit card numbers | PCI violation | Log last 4 digits or a token reference |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| String concatenation for logs | Not machine-parseable; breaks search | Use a structured logging library that emits JSON |
| No trace_id in logs | Cannot correlate across services | Inject trace context on every inbound request and pass it forward |
| Generating a new trace_id per service | Breaks the trace chain | Extract the incoming `traceparent` header; only generate if absent |
| Logging everything at INFO | Signal buried in noise | Use DEBUG for detail; INFO for events that matter |
| Logging secrets or PII | Compliance and security exposure | Scrub sensitive fields before the log call |

## Related Skills

- **observability-aware-coding** — structured logging is one pillar of a complete observability strategy
- **loose-coupling** — trace context must propagate across service boundaries to maintain correlation
- **rollback-friendly-design** — logs and traces help detect regressions that signal when a rollback is needed
