# Architecture Principles

Scope: how systems in this portfolio are designed, structured, and evolved. This document defines the architectural baseline; individual repos may extend it but not contradict it without recording the deviation in an [ADR](../templates/architecture-decision-record.md).

## 1. Cloud-Native by Default

- Stateless application processes; all durable state lives in a managed store (Azure SQL, Cosmos DB, Storage, or an equivalent self-hosted service for non-Azure tools).
- Configuration is externalized (environment variables, Azure App Configuration, or a mounted secret). No environment-specific values baked into build artifacts.
- Horizontal scale is the default scaling strategy. Vertical scaling is a stopgap, not a design goal.
- Every service exposes a health endpoint (`/healthz` or platform equivalent) independent of business logic, so orchestrators can make liveness/readiness decisions without invoking domain code.

## 2. Resilience

- **Timeouts everywhere.** Every outbound call (HTTP, DB, queue) has an explicit timeout; there is no such thing as an unbounded wait in production code.
- **Retries with backoff.** Transient failures (network blips, 429/503 responses) are retried with exponential backoff and jitter, capped at a small, explicit maximum attempt count. Non-idempotent operations are never retried without an idempotency key.
- **Circuit breaking.** Any dependency that can fail repeatedly under load (external API, downstream service) sits behind a circuit breaker so a struggling dependency cannot cascade into a full outage.
- **Graceful degradation.** When a non-critical dependency is unavailable, the system serves a reduced feature set rather than failing the whole request. Critical-path dependencies fail fast and loud.
- **Bulkheads.** Resource pools (DB connections, thread pools, HTTP clients) are partitioned per dependency so one slow dependency cannot starve resources needed by an unrelated one.

## 3. Observability as a First-Class Concern

Architecture decisions are made with the assumption that the system will need to be debugged in production without a debugger attached. See [`observability.md`](observability.md) for the concrete logging, metrics, and tracing standard. At the architecture level this means:

- Every request/operation carries a correlation ID from ingress to the deepest downstream call.
- Business-significant state transitions emit a structured event, not just a log line.
- Dashboards and alerts are designed alongside the feature, not bolted on after an incident.

## 4. Security by Design

Security is an architectural property, not a testing-phase checklist. See [`security.md`](security.md) for the full model (STRIDE, Zero Trust, secrets, SBOM). At the architecture level:

- Trust boundaries are drawn explicitly in every design (client to edge, edge to service, service to service, service to data store) and each boundary has an explicit authentication and authorization mechanism.
- Least privilege is the default for every identity, managed or human: a service principal or managed identity is scoped to exactly the resources it needs, never to a subscription-wide role.
- No architecture is approved with a single point of compromise granting full system access.

## 5. Loose Coupling, Explicit Contracts

- Services communicate through versioned, explicitly documented contracts (REST/OpenAPI, gRPC/protobuf, or an event schema registry). Shared database access between independently deployed services is not an integration pattern.
- Breaking changes to a contract require a new version; consumers migrate on their own schedule within a published deprecation window.
- Asynchronous, event-driven integration (Azure Event Hub, Service Bus, or an equivalent queue) is preferred over synchronous chains for anything that is not a direct user-facing request/response.

## 6. Data Ownership

- Every piece of durable data has exactly one owning service. Other services read it only through that service's published interface or through an explicitly replicated read model, never through direct access to the owner's schema.
- Data classification (public, internal, confidential, restricted) is assigned at design time and drives encryption, retention, and access-control decisions downstream.

## 7. Evolutionary Architecture

- Architecture decisions with long-term consequences (data store choice, service boundaries, cross-cutting frameworks) are recorded as an [ADR](../templates/architecture-decision-record.md) at decision time, including the alternatives considered and the tradeoffs accepted.
- Reversibility is a design criterion: prefer decisions that are cheap to undo over decisions that are technically superior but lock the system in.
- Complexity is added only when a concrete, current requirement demands it (YAGNI applies at the architecture level exactly as it does at the code level).

## 8. Deployment Topology Baseline

| Concern | Standard |
|---|---|
| Compute | Containers on Azure Container Apps or AKS for services; native binaries for desktop/CLI tools |
| Networking | Private endpoints for data services; public ingress only through a managed gateway (Azure Front Door / App Gateway) |
| Identity | Microsoft Entra ID for both human and workload identities; no local user stores for anything Azure-hosted |
| Secrets | Azure Key Vault, referenced via Managed Identity; no secrets in environment files in any hosted environment |
| Multi-region | Only when the availability SLA of the specific project requires it; single-region with automated backups is the default |
