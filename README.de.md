<div align="center"><img src="RayStudio.png" alt="RayStudio Logo" width="120"/><h1>engineering-standards</h1></div>

[🇬🇧 English Version](README.md)

**Die einzige versionierte Quelle dafür, wie Software im RayStudio-Portfolio designt, abgesichert, gebaut und released wird.** Microsoft-fokussiert, Security-First, geschrieben für Senior Engineers und Architects.

## Zweck, Umfang, Zielgruppe

Dieses Repository sorgt dafür, dass über ein solo-betreutes Portfolio hinweg, auf jeder Maschine, in jeder Sprache, derselbe Engineering-Standard gilt. Es richtet sich an genau die Zielgruppe, die es tatsächlich nutzt: Senior Software Engineers und Cloud-Architects, die tägliche Design- und Delivery-Entscheidungen treffen, nicht ein allgemeines Publikum, dem Grundlagen erklärt werden müssen.

Es ist normativ, nicht ambitioniert formuliert: Jede Regel hier ist entweder technisch erzwungen (Branch-Protection, CI-Gates) oder wird aktiv im Code-Review geprüft. Wo ein Repository eine Vorgabe aus einem konkreten Grund nicht erfüllen kann, wird die Ausnahme dokumentiert, statt sie stillschweigend zu ignorieren (siehe [`standards/governance.md`](standards/governance.md), Abschnitt 7).

## Was ist das konkret?

- [`CLAUDE.md`](CLAUDE.md) ist das operative Regelwerk: auf jeder Entwicklungsmaschine als globale Claude-Code-Instruktion verlinkt, damit ein KI-Pair-Programmer überall exakt dieselben Regeln befolgt. [`CLAUDE.en.md`](CLAUDE.en.md) ist die englische Referenzübersetzung.
- [`standards/`](standards/) ist die technische Detailreferenz, die dieses README zusammenfasst: Architecture, Security, Azure-Integration, CI/CD, Observability, Governance sowie sprachspezifische Coding-Standards.
- [`examples/`](examples/) enthält vollständige, funktionierende Referenzimplementierungen: eine CI-Pipeline, eine Azure-Policy-Definition und ein durchgearbeitetes STRIDE-Threat-Model.
- [`templates/`](templates/) enthält die Copy-Paste-Ausgangspunkte, die im gesamten Portfolio verwendet werden: eine README-Vorlage, eine Architecture-Decision-Record-Vorlage, eine Security-Checklist pro Release, eine Publish-Checkliste für neue Tools und eine Dependabot-Konfiguration.
- [`ruleset-template.json`](ruleset-template.json) ist das GitHub-Ruleset (`solo-main-protection`), das auf den Default-Branch jedes öffentlichen Repos angewendet wird.

## Microsoft-Style-Prinzipien

| Prinzip | Was das in der Praxis bedeutet |
|---|---|
| **Security-First** | Threat Modeling, Zero Trust und Secrets-Management werden ab dem ersten Commit mitgedacht, nicht kurz vor einem Release nachgerüstet. Siehe [`standards/security.md`](standards/security.md). |
| **Cloud-Native** | Zustandslose Services, externalisierte Konfiguration, horizontale Skalierung als Standard, Resilience-Patterns (Timeouts, Retries, Circuit Breaker, Bulkheads) fest in jedes Design eingebaut. Siehe [`standards/architecture.md`](standards/architecture.md). |
| **Azure-Ready** | Microsoft Entra ID für Identität, Managed Identity statt langlebiger Secrets, Bicep als Code, Azure Monitor für Observability, überall wo ein Repository einen Azure-Bezug hat. Siehe [`standards/azure-integration.md`](standards/azure-integration.md). |
| **Evidenzbasierte Governance** | Compliance wird durch grüne CI, generierte SBOMs und dokumentierte Threat Models nachgewiesen, nicht behauptet. Siehe [`standards/governance.md`](standards/governance.md). |

## Übersicht der Standards

| Dokument | Inhalt |
|---|---|
| [`standards/architecture.md`](standards/architecture.md) | Cloud-natives Design, Resilience, lose Kopplung, Data Ownership, evolutionäre Architektur |
| [`standards/security.md`](standards/security.md) | STRIDE-Threat-Modeling, Zero Trust, Secrets, Verschlüsselung, SBOM, Audit-Logging |
| [`standards/azure-integration.md`](standards/azure-integration.md) | Entra ID, Azure Monitor, Event Hub, Ressourcen-Namenskonventionen, Bicep/ARM, Azure Policy |
| [`standards/ci-cd.md`](standards/ci-cd.md) | Runner-Matrix, Pflicht-Pipeline-Stages, OIDC-Federation, SBOM-Generierung, Release-Automation |
| [`standards/release-process.md`](standards/release-process.md) | Ruleset-Setup, Versionierungsdisziplin, Release-Schritte, Pre-Release-Checkliste, Rollback |
| [`standards/documentation.md`](standards/documentation.md) | README-Struktur, API-Docs, ADR, CHANGELOG-Format, Sprachkonventionen |
| [`standards/writing-style.md`](standards/writing-style.md) | Gedankenstrich-Regel, Schweizer Rechtschreibung, DE/EN-Komposita, Ich-Form in Solo-Maintainer-Dokumenten |
| [`standards/microsoft-stack.md`](standards/microsoft-stack.md) | Graph-API-Auth, Azure-Deployment, Windows/ARM-Targeting, Package-Management |
| [`standards/observability.md`](standards/observability.md) | Strukturiertes Logging, Correlation, RED/USE-Metriken, Dashboards, Alerting |
| [`standards/governance.md`](standards/governance.md) | Ownership-Modell, risikobasierte Merge-Policy, Policy-Lifecycle, Living Standards, ADRs |
| [`standards/coding-rust.md`](standards/coding-rust.md) | Clippy-Disziplin, Async-Runtime-Regeln, sqlx, Dead Code, Dependency-Deklaration |
| [`standards/coding-typescript.md`](standards/coding-typescript.md) | Compiler-Strictness, korrektes State-Management, i18n, Styling, Testing |
| [`standards/coding-python.md`](standards/coding-python.md) | Type Hints, Async/FastAPI-Disziplin, Pydantic-Validierung, Testing, SQL-Sicherheit |

## Architektur-Philosophie

Systeme in diesem Portfolio werden standardmässig cloud-nativ und resilient gebaut: zustandslose Prozesse, externalisierte Konfiguration und explizite Resilience-Patterns (Timeouts, Retries mit Backoff, Circuit Breaking, Graceful Degradation) an jeder Dependency-Grenze. Komplexität wird nur hinzugefügt, wenn eine konkrete Anforderung sie verlangt, und strukturelle Entscheidungen mit langfristigen Folgen werden als [Architecture Decision Record](templates/architecture-decision-record.md) festgehalten, nicht stillschweigend getroffen. Details in [`standards/architecture.md`](standards/architecture.md).

## Security-First-Modell

Jedes System mit externer Angriffsfläche wird vor dem ersten Release mit STRIDE threat-modeled (siehe das [durchgearbeitete Beispiel](examples/threat-model-example.md)), arbeitet nach Zero-Trust-Prinzipien (niemals allein aufgrund des Netzwerkstandorts vertrauen, immer Identität und Berechtigung prüfen) und behandelt Secrets, Verschlüsselung und Dependency-Schwachstellen als Release-Gate, nicht als Nachgedanken. Details in [`standards/security.md`](standards/security.md) und der [Security-Checklist pro Release](templates/security-checklist.md).

## Azure-Integrationsmatrix

| Bedarf | Standardwahl |
|---|---|
| Identität | Microsoft Entra ID + Managed Identity |
| Relationale Daten | Azure SQL Database |
| Secrets | Azure Key Vault |
| Async Messaging | Event Hub (Streaming) / Service Bus (transaktional) |
| Container-Hosting | Azure Container Apps (AKS bei komplexem Orchestrierungsbedarf) |
| Infrastructure as Code | Bicep |
| Observability | Azure Monitor + Application Insights |
| CI/CD nach Azure | GitHub Actions mit OIDC-Federation, keine gespeicherten Client-Secrets |

Details, inklusive Ressourcen-Namenskonventionen und Azure-Policy-Governance-Modell, in [`standards/azure-integration.md`](standards/azure-integration.md).

## CI/CD-Matrix

| Stage | Windows | Ubuntu | macOS |
|---|---|---|---|
| Lint | ✅ | ✅ | ✅ |
| Build | ✅ | ✅ | ✅ |
| Test | ✅ | ✅ | ✅ |
| Security Scan | optional | ✅ | optional |
| SBOM-Generierung | optional | ✅ | optional |
| Release-Packaging | falls Windows-Target | falls Linux-Target | falls macOS-Target |

Keine Pipeline-Stage wird jemals über ein Scoping-Flag ausgeschlossen, um einen fehlschlagenden Build zu umgehen, ein fehlschlagender Schritt wird behoben, nicht versteckt. Details, inklusive des OIDC-Federation-Patterns für Azure-Deployment, in [`standards/ci-cd.md`](standards/ci-cd.md) und der [Referenz-Pipeline](examples/ci-pipeline-example.yml).

## Observability-Guidelines

Jeder Service liefert strukturierte JSON-Logs mit einer durchgängigen Correlation-ID, RED-Metriken (Rate, Errors, Duration) für alles Request-verarbeitende und USE-Metriken (Utilization, Saturation, Errors) für alles Ressourcenartige, und jedes relevante Dashboard-Signal hat einen zugehörigen, umsetzbaren Alert. Details in [`standards/observability.md`](standards/observability.md).

## Governance-Modell

Ein solo-betreutes Portfolio ersetzt Peer-Review durch strukturelle Kontrollen: technisch erzwungene Branch-Protection, eine risikobasierte Merge-Policy (niedriges Risiko selbst gemergt und gemeldet, mittleres/hohes Risiko wartet auf explizite Freigabe) und ein monatlicher Living-Standards-Check gegen neue Security-Advisories und Best Practices, immer als Diff vorgeschlagen, nie automatisch übernommen. Details in [`standards/governance.md`](standards/governance.md).

## Beispiel-Flows

- **Ein neues Feature mit Azure-Backend ausliefern**: Design anhand [`standards/architecture.md`](standards/architecture.md) und [`standards/azure-integration.md`](standards/azure-integration.md), vor dem Release durch die [Security-Checklist](templates/security-checklist.md), Auslieferung über die [Referenz-CI-Pipeline](examples/ci-pipeline-example.yml).
- **Ein neues öffentliches Repository starten**: [Ruleset](ruleset-template.json) anwenden, README aus der [Vorlage](templates/readme-template.md) aufsetzen, sprachspezifischen Coding-Standard ab Tag 1 befolgen.
- **Eine strukturelle Entscheidung treffen**: als [ADR](templates/architecture-decision-record.md) festhalten, bevor die Umsetzung geschrieben wird, nicht danach.

## Quick Start (neue Maschine)

```bash
git clone https://github.com/9t29zhmwdh-coder/engineering-standards.git ~/engineering-standards
ln -sf ~/engineering-standards/CLAUDE.md ~/.claude/CLAUDE.md
```

## Ruleset auf ein neues Repo anwenden

```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset-template.json
```

## Release-Historie

| Datum | Änderung |
|---|---|
| 2026-07-03 | Initial-Commit; Engineering Standards und Best Practices etabliert (Senior Engineer, Security-First, Microsoft-fokussiert) |
| 2026-07-03 | Risikobasierte Merge-Policy formalisiert |
| 2026-07-05 | Englische Übersetzung der Standards ergänzt; bilinguale READMEs |
| 2026-07-05 | KI-Transparenz- und Positionierungs-Policy ergänzt (`CLAUDE.md`, Abschnitt 12) |
| 2026-07-08 | Erweitert zu einer vollständigen Enterprise-Standards-Library: `standards/`, `examples/`, `templates/`-Verzeichnisse zu Architecture, Security, Azure-Integration, CI/CD, Observability, Governance sowie Rust/TypeScript/Python-Coding-Standards |
| 2026-07-21 | `required_status_checks` von Einzel-Repo-Pilot auf portfolio-weite Baseline gehoben (v0.5.3): GitHub blockiert jetzt technisch einen Merge bei rotem Check auf jedem Public Repo, nicht nur die PR-Pflicht |

---

**Autor:** [Rafael Yilmaz](https://github.com/9t29zhmwdh-coder) · **Status:** Active · **Lizenz:** [CC BY 4.0](LICENSE)
