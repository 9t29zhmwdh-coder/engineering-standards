# Observability Guidelines

If it cannot be observed in production, it is not considered done. This document defines the minimum logging, metrics, and tracing bar for every service.

## 1. The Three Pillars

| Pillar | Purpose | Portfolio Standard |
|---|---|---|
| Logs | What happened, in detail, at a point in time | Structured JSON, correlation ID on every entry |
| Metrics | Aggregate numeric signal over time | RED (Rate, Errors, Duration) for every request-handling component; USE (Utilization, Saturation, Errors) for every resource |
| Traces | The path of a single request across components | W3C Trace Context propagated end to end; one trace per logical operation |

## 2. Structured Logging

- Every log line is a JSON object, never free-form interpolated text, so it is queryable without regex archaeology.
- Minimum fields: `timestamp` (ISO 8601, UTC), `level`, `message`, `service`, `correlationId`. Add `userId`/`tenantId` where relevant to the operation and permitted by data classification.
- Log levels are used consistently:
  - `Error`: an operation failed and requires investigation.
  - `Warning`: something unexpected happened but the system recovered or degraded gracefully.
  - `Information`: a business-significant event occurred (request completed, state transitioned, job finished).
  - `Debug`: verbose detail, disabled by default in production, enabled selectively when diagnosing a specific issue.
- Never log secrets, full authentication tokens, or unmasked PII. Log the presence and shape of sensitive data ("API key configured: yes"), not the value.

## 3. Correlation and Tracing

- A correlation ID is generated (or propagated from an inbound `traceparent` header) at the edge of the system and threaded through every log line, outbound call, and background job triggered by that request.
- Distributed tracing (Application Insights, OpenTelemetry) captures the full call graph for a request, including downstream service calls, database queries, and external API calls, so a slow request can be attributed to its actual bottleneck rather than guessed at.
- Background/async work (queue consumers, scheduled jobs) generates its own correlation ID at the start of the unit of work and logs it consistently, since there is no inbound request to inherit one from.

## 4. Metrics

- **RED** for anything that serves requests: Rate (requests/sec), Errors (error rate), Duration (latency distribution, not just an average; track p50/p95/p99).
- **USE** for anything that is a resource: Utilization, Saturation, Errors (CPU, memory, connection pools, queue depth).
- Business metrics (machines extracted, emails processed, jobs completed) are emitted as custom metrics alongside technical metrics; a system can be technically healthy while failing its actual purpose, and only business metrics catch that.
- Metrics are cardinality-aware: a label that can take unbounded values (user ID, request ID) is never used as a Metric dimension; use it in logs and traces instead.

## 5. Dashboards

- Every service with production traffic has a dashboard covering, at minimum: request rate, error rate, latency percentiles, and the resource-level USE metrics for its dependencies (database, cache, downstream APIs).
- Dashboards are defined alongside the feature that needs them, not created reactively after the first incident that lacked one.
- A dashboard without a corresponding alert threshold is a monitoring gap; every dashboarded signal that indicates a real problem has a paired alert.

## 6. Alerting

- Alerts fire on symptoms that matter to users (error rate, latency, availability), not purely on internal implementation details (a specific log line appearing), unless that log line is a direct proxy for a user-facing failure.
- Every alert has a documented, actionable response; an alert nobody knows how to act on is noise and is either fixed or removed.
- Alert thresholds are defined as code alongside the infrastructure they monitor (see [`azure-integration.md`](azure-integration.md) section 7), not created ad hoc in a portal and left undocumented.

## 7. Local and CLI Tools

Desktop and CLI tools without a hosted backend do not need Application Insights, but the same discipline applies at a smaller scale:

- Structured logs (even if written to a local rotating file) rather than unstructured `print`/`console.log` calls scattered through the codebase.
- A clear, documented way for a user to retrieve diagnostic logs when reporting a bug (log file location documented in the README, per the portfolio's [Uninstall/Cleanup README convention](../README.md)).
- Crash reporting, if implemented, is opt-in and never transmits data without explicit user consent, consistent with the [security standards](security.md) on data handling.
