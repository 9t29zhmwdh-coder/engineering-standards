# Azure Integration Standards

How portfolio services integrate with Azure. Applies to any repository that deploys to, or authenticates against, Azure.

## 1. Identity: Microsoft Entra ID

- Human access to Azure resources (portal, CLI, `az` commands in CI) authenticates via Entra ID with MFA enforced at the tenant level.
- Workload identity uses **Managed Identity** wherever the calling service runs on an Azure compute resource (App Service, Container Apps, AKS with workload identity federation, Functions). Service principals with client secrets are used only when Managed Identity is not supported by the target integration, and their secrets live in Key Vault with a rotation schedule.
- Application-level user authentication (for apps with their own sign-in) uses the Microsoft Identity Platform (MSAL) against an Entra ID app registration; scopes are requested incrementally, not as a single broad consent.
- Microsoft Graph API access uses the official Graph SDK for the target language, never hand-rolled REST calls against `graph.microsoft.com` where an SDK exists.

## 2. Observability: Azure Monitor

| Signal | Service | Standard |
|---|---|---|
| Metrics | Azure Monitor Metrics | Platform metrics enabled by default; custom business metrics emitted via Application Insights SDK |
| Logs | Log Analytics Workspace | Structured JSON logs shipped via the Application Insights SDK or diagnostic settings; one workspace per environment (dev/staging/prod), not per service |
| Traces | Application Insights (distributed tracing) | W3C Trace Context propagated across every service-to-service call; sampling configured explicitly, never left at an undocumented default |
| Alerts | Azure Monitor Alerts | Defined as code (Bicep/Terraform) alongside the resource they monitor, not created ad hoc in the portal |

See [`observability.md`](observability.md) for the language-level logging/tracing conventions that feed into this pipeline.

## 3. Event Hub Standards

- Event Hub is the default backbone for high-throughput telemetry and event streaming; Service Bus is used instead when ordered, transactional, or session-based messaging semantics are required.
- Every event carries: a schema version field, a correlation ID, an event type, and an ISO 8601 UTC timestamp.
- Consumer groups are provisioned per logical consumer, never shared across unrelated services, so one slow consumer cannot stall another's checkpoint.
- Partition keys are chosen for even distribution (typically a tenant or entity ID), never a low-cardinality value that would create a hot partition.
- Schema evolution is additive only within a major version; breaking changes bump the schema version field and are consumed behind an explicit compatibility shim during migration.

## 4. Azure Resource Naming Conventions

Pattern: `<resource-type-abbr>-<workload>-<environment>-<region>-<instance>`

| Example | Meaning |
|---|---|
| `rg-stateforge-prod-weu-01` | Resource group, StateForge, production, West Europe, instance 01 |
| `app-stateforge-api-prod-weu` | App Service, StateForge API, production, West Europe |
| `kv-stateforge-prod-weu` | Key Vault, StateForge, production, West Europe |
| `evh-stateforge-events-prod-weu` | Event Hub, StateForge events, production, West Europe |
| `st-stateforge-prod-weu-01` | Storage account, StateForge, production, West Europe (storage account names have no hyphens; concatenate: `ststateforgeprodweu01`) |

Standard abbreviations follow the [Microsoft Cloud Adoption Framework resource abbreviation reference](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations). Environments are always one of `dev`, `staging`, `prod`; regions use the short Azure region code (`weu`, `neu`, `eus2`, ...).

Tags applied to every resource: `workload`, `environment`, `owner`, `costCenter` (where applicable), `managedBy: bicep|terraform|manual`.

## 5. Infrastructure as Code: Bicep/ARM

- Bicep is the default IaC language for Azure-native infrastructure; Terraform is acceptable for multi-cloud or hybrid workloads where a single tool across providers is valuable.
- Every deployable environment (dev, staging, prod) is defined from the same Bicep modules with environment-specific parameter files; no hand-edited drift between environments.
- Modules are versioned and published to a private Bicep registry (Azure Container Registry) for reuse across repositories rather than copy-pasted.
- `what-if` deployments run in CI on every pull request touching infrastructure, so the reviewer sees the actual planned change, not just the diff of the template.
- State for Terraform-managed infrastructure lives in an Azure Storage backend with soft delete and versioning enabled; Bicep is stateless by design and does not require this.

## 6. Application Insights Logging Standard

- Structured logging only: every log entry is a JSON object with at minimum `timestamp`, `level`, `message`, `correlationId`, and `service`.
- Log levels follow: `Error` (needs attention), `Warning` (degraded but functioning), `Information` (business-significant events), `Debug` (verbose, disabled in production by default, enabled per-request via a sampling flag when diagnosing an issue).
- PII and secrets are never logged; logging code is reviewed for this exactly as rigorously as the feature code it instruments.
- Custom telemetry (business events, not just technical logs) is emitted via `TrackEvent`/`TrackMetric` on the Application Insights SDK so it is queryable independent of the raw log stream.

## 7. Azure Policy Governance

- Subscriptions are governed by Azure Policy assignments that enforce, at minimum: allowed regions, required tags, encryption at rest, and denial of public network access for data services unless explicitly exempted.
- New policy definitions are authored as code (see [`../examples/azure-policy-example.json`](../examples/azure-policy-example.json)) and go through the same PR review as application code.
- Policy compliance is reviewed on the same monthly cadence as the [living standards check](governance.md#living-standards).

## 8. Integration Decision Matrix

| Need | Default Choice | Notes |
|---|---|---|
| Relational data | Azure SQL Database | Cosmos DB for globally distributed / flexible-schema workloads |
| Blob/file storage | Azure Blob Storage | Lifecycle management rules for tiering/expiry configured from day one |
| Secrets | Azure Key Vault | Referenced via Managed Identity |
| Async messaging | Event Hub (streaming) or Service Bus (transactional) | See section 3 for the decision criteria |
| Container hosting | Azure Container Apps | AKS only when workload complexity (custom schedulers, sidecars, service mesh) justifies the operational overhead |
| CI/CD to Azure | GitHub Actions with OIDC federation | See [`ci-cd.md`](ci-cd.md); no long-lived service principal secrets in GitHub Actions secrets |
