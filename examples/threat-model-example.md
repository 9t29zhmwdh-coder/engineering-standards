# Threat Model Example

Worked example applying [`standards/security.md`](../standards/security.md) section 1 (STRIDE) to a representative service: a multi-tenant web API with Entra ID authentication, an Azure SQL database, and an Event Hub-based audit stream. Use this as a template shape, not a literal checklist; adapt the components and threats to the system actually being modeled.

## System Overview

```
Browser/Client
     |  (HTTPS, Entra ID OAuth2 token)
     v
Azure Front Door (edge, WAF)
     |
     v
API (Container Apps, Managed Identity)
     |                     |
     v                     v
Azure SQL (tenant data)   Event Hub (audit events)
     |
     v
Key Vault (connection secrets, via Managed Identity)
```

## Trust Boundaries

| Boundary | Between | Authentication |
|---|---|---|
| B1 | Client and edge | Entra ID OAuth2 bearer token, validated at Front Door/WAF and again by the API |
| B2 | Edge and API | Network restriction (private endpoint) plus token re-validation |
| B3 | API and Azure SQL | Managed Identity, Azure AD authentication for SQL |
| B4 | API and Event Hub | Managed Identity, SAS disabled |
| B5 | API and Key Vault | Managed Identity, access policy scoped to specific secrets |

## STRIDE Analysis

### Spoofing

| Threat | Component | Mitigation |
|---|---|---|
| Attacker presents a forged or stolen token to the API | B1/B2 | Token signature and issuer validated against Entra ID metadata on every request; short token lifetime with refresh; audience claim checked to prevent token reuse across services |
| A compromised container image impersonates the legitimate API when calling SQL/Event Hub | B3/B4 | Managed Identity tied to the specific Container App resource; no portable credential exists to steal |

### Tampering

| Threat | Component | Mitigation |
|---|---|---|
| Request payload modified in transit | B1, B2 | TLS 1.2+ enforced end to end, including internal hops |
| Audit event tampered with after being written | Event Hub | Event Hub capture to an append-only, immutable-policy storage container; API identities granted send-only, not manage, rights |

### Repudiation

| Threat | Component | Mitigation |
|---|---|---|
| A tenant admin denies having changed a permission | API, audit log | Every permission change emits an audit event with the authenticated identity, timestamp, and before/after state, written to the immutable Event Hub capture store |

### Information Disclosure

| Threat | Component | Mitigation |
|---|---|---|
| Cross-tenant data leakage through a missing tenant filter | API, Azure SQL | Every query scoped by tenant ID at the data-access layer, enforced by a repository pattern that makes an unscoped query a compile-time impossibility, not a runtime discipline |
| Secrets exposed via error messages or logs | API | Generic error responses to clients; secrets never logged; secret scanning enabled on the repository |
| Data at rest read by an unauthorized party with storage-level access | Azure SQL, Storage | Transparent data encryption plus customer-managed keys in Key Vault for tenant data classified as confidential |

### Denial of Service

| Threat | Component | Mitigation |
|---|---|---|
| Flood of requests against a public endpoint | Front Door/WAF | Rate limiting and WAF rules at the edge, before traffic reaches the API |
| A single tenant's workload starves shared resources | API, Azure SQL | Per-tenant rate limiting; Azure SQL elastic pool with resource governance rather than a single shared unmanaged database |

### Elevation of Privilege

| Threat | Component | Mitigation |
|---|---|---|
| A standard user calls an admin-only endpoint directly | API | Authorization checked server-side on every endpoint via RBAC claims from the validated token, never inferred from client-supplied role hints |
| A compromised API identity is used to escalate to subscription-level Azure access | Managed Identity | Role assignment scoped to the specific resource group only; no Owner/Contributor at the subscription level for any workload identity |

## Residual Risk and Review

Every mitigation above is paired with an owner and a review date in the actual project's tracked risk register (not reproduced here). This threat model is revisited whenever a new external integration, data flow, or trust boundary is added, per [`standards/security.md`](../standards/security.md) section 1.
