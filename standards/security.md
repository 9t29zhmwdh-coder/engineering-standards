# Security Standards

Security is built in from the first commit, not added before a release. This document is normative for every portfolio repository; see [`../templates/security-checklist.md`](../templates/security-checklist.md) for the per-release checklist and [`../examples/threat-model-example.md`](../examples/threat-model-example.md) for a worked example.

## 1. Threat Modeling (STRIDE)

Every service or tool with an external attack surface (network listener, file input, user-supplied data) gets a threat model before the first release, using STRIDE:

| Category | Question | Typical Mitigation |
|---|---|---|
| **S**poofing | Can an actor impersonate a user or service? | Strong authentication, mutual TLS for service to service, signed tokens |
| **T**ampering | Can data be modified in transit or at rest without detection? | TLS in transit, integrity checks (HMAC, signatures), append-only audit logs |
| **R**epudiation | Can an actor deny having performed an action? | Audit logging with immutable timestamps and identity, non-repudiable signing where needed |
| **I**nformation Disclosure | Can data be exposed to an unauthorized party? | Encryption at rest and in transit, least-privilege access, data classification |
| **D**enial of Service | Can the system be made unavailable? | Rate limiting, resource quotas, circuit breakers, autoscaling with caps |
| **E**levation of Privilege | Can an actor gain rights beyond what was granted? | Least privilege, RBAC/ABAC, input validation, sandboxing |

Threat models are revisited whenever a new trust boundary, external integration, or data flow is introduced, not only at initial design time.

## 2. Zero Trust Principles

- **Never trust, always verify.** Network location (VPN, internal subnet) is never treated as an implicit authorization signal. Every request is authenticated and authorized regardless of origin.
- **Explicit verification.** Authenticate and authorize based on identity, device health, and context signals available; do not rely on a single factor.
- **Least privilege access.** Every identity (human or workload) receives the minimum scope required for its task, time-boxed where possible (Privileged Identity Management for elevated roles).
- **Assume breach.** Design as if an attacker is already inside the perimeter: segment networks, encrypt data at rest, and log everything needed to detect and reconstruct an incident.

## 3. Secure Defaults

- Deny by default: new endpoints, roles, and permissions start with no access and are opened up explicitly, never the reverse.
- Fail closed: an authorization check that cannot complete (timeout, error) is treated as a denial, not an approval.
- TLS 1.2+ is the floor for every network connection; unencrypted internal traffic requires a documented exception.

## 4. Identity and Access

- Microsoft Entra ID is the identity provider for anything Azure-hosted or M365-integrated. No custom-built authentication for these contexts; use the Microsoft Graph SDK and MSAL libraries rather than hand-rolled OAuth flows.
- MFA is mandatory for every human identity with access to production systems or secrets, from day one.
- RBAC or ABAC governs authorization decisions; role and attribute definitions are versioned alongside the code that enforces them.
- Managed Identity is used for Azure-to-Azure authentication wherever the target resource supports it. Service principals with client secrets are a fallback, not a default, and their secrets are rotated on a defined schedule.

## 5. Secrets Management

- No secret (password, API key, connection string, certificate private key) is ever committed to a repository, including in commit history, test fixtures, or example configuration.
- Azure Key Vault holds all production secrets; applications read them via Managed Identity at runtime, not via secrets baked into configuration files or CI variables where avoidable.
- Local development uses `.env` files that are `.gitignore`d, paired with a committed `.env.example` documenting the required keys without values.
- Secret scanning (GitHub secret scanning / push protection, or an equivalent pre-commit hook) is enabled on every repository.

## 6. Input Validation and Sanitization

- Validate at every trust boundary: user input, API payloads, file uploads, and data read back from a database that another process could have written.
- Whitelist expected shapes and values rather than attempting to blacklist known-bad patterns.
- Use parameterized queries or an ORM's parameter binding for all database access; string-concatenated queries are not acceptable in any language.
- Encode output for the context it is rendered into (HTML, URL, shell argument) to prevent injection classes beyond SQL (XSS, command injection, path traversal).

## 7. Encryption

- **In transit:** TLS everywhere, including internal service-to-service traffic where the platform supports it cheaply (mutual TLS via a service mesh, or platform-managed TLS for PaaS-to-PaaS traffic).
- **At rest:** Platform-managed encryption at minimum (Azure Storage Service Encryption, transparent data encryption for databases); customer-managed keys in Key Vault for data classified as confidential or restricted.
- **Passwords:** Never stored in a recoverable form. Use a modern, salted, adaptive hash (Argon2id preferred, PBKDF2 or bcrypt acceptable) with parameters reviewed against current OWASP guidance.

## 8. Secure Error Handling

- Clients receive generic, non-identifying error messages and a correlation ID; stack traces, SQL fragments, and internal paths never reach a client response.
- The corresponding detailed error, including stack trace and context, is logged internally and retrievable via the correlation ID.

## 9. Dependency Governance and SBOM

- Every release generates a Software Bill of Materials in CycloneDX format (see [`../examples/`](../examples/) for pipeline integration); this is a CI gate, not an optional step.
- Dependency vulnerability scanning (`npm audit`, `cargo audit`, `pip-audit`, `dotnet list package --vulnerable`, GitHub Dependabot alerts) runs on every pull request and on a scheduled cadence against `main`.
- Lock files (`package-lock.json`, `Cargo.lock`, `requirements.txt`/`poetry.lock`) are committed and are the source of truth for reproducible builds.
- A vulnerability with an available fix is patched within a risk-proportional window: critical within days, high within two weeks, medium/low on the next regular dependency update cycle.

## 10. Audit Logging

- Every sensitive operation (authentication attempt, permission change, data export, secret access) is logged with who, what, when, and from where.
- Audit logs are append-only from the application's perspective; write access to modify or delete historical audit entries is not granted to application identities.
- Failed authentication attempts are tracked and trigger lockout/backoff after a defined threshold, with alerting on anomalous patterns (velocity, geography, impossible travel where identity telemetry is available).

## 11. Release Gate

No release ships without:

- [ ] A current threat model for any new attack surface introduced since the last release
- [ ] A clean dependency audit (or explicitly accepted, time-boxed exceptions with a stated reason)
- [ ] A generated SBOM attached to the release
- [ ] Secret scanning clean on the release commit
- [ ] The applicable items in [`../templates/security-checklist.md`](../templates/security-checklist.md) checked off
