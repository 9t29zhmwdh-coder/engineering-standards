<div align="center"><img src="RayStudio.png" alt="RayStudio Logo" width="120"/><h1>engineering-standards</h1></div>

🇩🇪 [Deutsche Version](README.de.md)

**The single versioned source of truth for how software is designed, secured, built, and released across the RayStudio portfolio.** Microsoft-focused, security-first, written for senior engineers and architects.

## Purpose, Scope, Audience

This repository exists so that the same engineering bar applies everywhere, on every machine, in every language, across a solo-maintained portfolio of public repositories. It is written for the audience that actually uses it: senior software engineers and cloud architects making day-to-day design and delivery decisions, not a general audience needing the basics explained.

It is normative, not aspirational: every rule here is either technically enforced (branch protection, CI gates) or actively checked during code review. Where a repository cannot meet a bar for a stated reason, the exception is documented in that repository, not silently ignored (see [`standards/governance.md`](standards/governance.md) section 7).

## What is this, concretely?

- [`CLAUDE.md`](CLAUDE.md) is the operative instruction set: linked as global Claude Code instructions on every development machine, so an AI pair-programmer follows the exact same rules everywhere. [`CLAUDE.en.md`](CLAUDE.en.md) is its English reference translation.
- [`standards/`](standards/) is the detailed technical reference this README summarizes: architecture, security, Azure integration, CI/CD, observability, governance, and per-language coding standards.
- [`examples/`](examples/) contains complete, working reference implementations: a CI pipeline, an Azure Policy definition, and a worked STRIDE threat model.
- [`templates/`](templates/) contains the copy-paste starting points used across the portfolio: a README template, an Architecture Decision Record template, and a per-release security checklist.
- [`ruleset-template.json`](ruleset-template.json) is the GitHub ruleset (`solo-main-protection`) applied to every public repository's default branch.

## Microsoft-Style Principles

| Principle | What it means in practice |
|---|---|
| **Security-First** | Threat modeling, Zero Trust, and secrets management are designed in from the first commit, not retrofitted before a release. See [`standards/security.md`](standards/security.md). |
| **Cloud-Native** | Stateless services, externalized configuration, horizontal scale by default, resilience patterns (timeouts, retries, circuit breakers, bulkheads) built into every design. See [`standards/architecture.md`](standards/architecture.md). |
| **Azure-Ready** | Microsoft Entra ID for identity, Managed Identity over long-lived secrets, Bicep as code, Azure Monitor for observability, wherever a repository has an Azure footprint. See [`standards/azure-integration.md`](standards/azure-integration.md). |
| **Evidence-Based Governance** | Compliance is demonstrated through green CI, generated SBOMs, and documented threat models, not asserted. See [`standards/governance.md`](standards/governance.md). |

## Standards Overview

| Document | Covers |
|---|---|
| [`standards/architecture.md`](standards/architecture.md) | Cloud-native design, resilience, loose coupling, data ownership, evolutionary architecture |
| [`standards/security.md`](standards/security.md) | STRIDE threat modeling, Zero Trust, secrets, encryption, SBOM, audit logging |
| [`standards/azure-integration.md`](standards/azure-integration.md) | Entra ID, Azure Monitor, Event Hub, resource naming, Bicep/ARM, Azure Policy |
| [`standards/ci-cd.md`](standards/ci-cd.md) | Runner matrix, required pipeline stages, OIDC federation, SBOM generation, release automation |
| [`standards/observability.md`](standards/observability.md) | Structured logging, correlation, RED/USE metrics, dashboards, alerting |
| [`standards/governance.md`](standards/governance.md) | Ownership model, risk-based merge policy, policy lifecycle, living standards, ADRs |
| [`standards/coding-rust.md`](standards/coding-rust.md) | Clippy discipline, async runtime rules, sqlx, dead code, dependency declaration |
| [`standards/coding-typescript.md`](standards/coding-typescript.md) | Compiler strictness, state management correctness, i18n, styling, testing |
| [`standards/coding-python.md`](standards/coding-python.md) | Type hints, async/FastAPI discipline, Pydantic validation, testing, SQL safety |

## Architecture Philosophy

Systems in this portfolio are built cloud-native and resilient by default: stateless processes, externalized configuration, and explicit resilience patterns (timeouts, retries with backoff, circuit breaking, graceful degradation) at every dependency boundary. Complexity is added only when a concrete requirement demands it, and structural decisions with long-term consequences are recorded as [Architecture Decision Records](templates/architecture-decision-record.md), not made silently. Full detail in [`standards/architecture.md`](standards/architecture.md).

## Security-First Model

Every system with an external attack surface is threat-modeled using STRIDE before its first release (see the [worked example](examples/threat-model-example.md)), operates on Zero Trust principles (never trust network location alone, always verify identity and authorization), and treats secrets, encryption, and dependency vulnerabilities as release gates, not afterthoughts. Full detail in [`standards/security.md`](standards/security.md) and the [per-release checklist](templates/security-checklist.md).

## Azure Integration Matrix

| Need | Default Choice |
|---|---|
| Identity | Microsoft Entra ID + Managed Identity |
| Relational data | Azure SQL Database |
| Secrets | Azure Key Vault |
| Async messaging | Event Hub (streaming) / Service Bus (transactional) |
| Container hosting | Azure Container Apps (AKS for complex orchestration needs) |
| Infrastructure as Code | Bicep |
| Observability | Azure Monitor + Application Insights |
| CI/CD to Azure | GitHub Actions with OIDC federation, no stored client secrets |

Full detail, including resource naming conventions and the Azure Policy governance model, in [`standards/azure-integration.md`](standards/azure-integration.md).

## CI/CD Matrix

| Stage | Windows | Ubuntu | macOS |
|---|---|---|---|
| Lint | ✅ | ✅ | ✅ |
| Build | ✅ | ✅ | ✅ |
| Test | ✅ | ✅ | ✅ |
| Security scan | optional | ✅ | optional |
| SBOM generation | optional | ✅ | optional |
| Release packaging | if Windows target | if Linux target | if macOS target |

No pipeline stage is ever excluded via a scoping flag to work around a failing build; a failing stage is fixed, not hidden. Full detail, including the OIDC federation pattern for Azure deployment, in [`standards/ci-cd.md`](standards/ci-cd.md) and the [reference pipeline](examples/ci-pipeline-example.yml).

## Observability Guidelines

Every service emits structured JSON logs with a correlation ID threaded end to end, RED metrics (Rate, Errors, Duration) for anything request-handling and USE metrics (Utilization, Saturation, Errors) for anything resource-like, and every dashboarded signal that matters has a paired, actionable alert. Full detail in [`standards/observability.md`](standards/observability.md).

## Governance Model

A solo-maintained portfolio substitutes structural controls for peer review: technically enforced branch protection, a risk-based merge policy (low risk self-merged and reported, medium/high risk waits for explicit approval), and a monthly living-standards check against new security advisories and best practices, always proposed as a diff and never auto-applied. Full detail in [`standards/governance.md`](standards/governance.md).

## Example Flows

- **Shipping a new feature with an Azure backend**: design with [`standards/architecture.md`](standards/architecture.md) and [`standards/azure-integration.md`](standards/azure-integration.md) in mind, thread it through the [security checklist](templates/security-checklist.md) before release, and ship it through the [reference CI pipeline](examples/ci-pipeline-example.yml).
- **Starting a new public repository**: apply the [ruleset](ruleset-template.json), scaffold its README from the [template](templates/readme-template.md), and follow the per-language coding standard from day one.
- **Making a structural decision**: record it as an [ADR](templates/architecture-decision-record.md) before writing the implementation, not after.

## Quick Start (new machine)

```bash
git clone https://github.com/9t29zhmwdh-coder/engineering-standards.git ~/engineering-standards
ln -sf ~/engineering-standards/CLAUDE.md ~/.claude/CLAUDE.md
```

## Apply the ruleset to a new repo

```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset-template.json
```

## Release History

| Date | Change |
|---|---|
| 2026-07-03 | Initial commit; engineering standards and best practices established (Senior Engineer, Security-First, Microsoft-focused) |
| 2026-07-03 | Risk-based merge policy formalized |
| 2026-07-05 | English translation of standards added; bilingual READMEs |
| 2026-07-05 | AI transparency and positioning policy added (`CLAUDE.md` section 12) |
| 2026-07-08 | Expanded into a full enterprise standards library: `standards/`, `examples/`, `templates/` directories covering architecture, security, Azure integration, CI/CD, observability, governance, and Rust/TypeScript/Python coding standards |

---

**Author:** [Rafael Yilmaz](https://github.com/9t29zhmwdh-coder) · **Status:** Active
