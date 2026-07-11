# Security Checklist

Per-release checklist implementing [`standards/security.md`](../standards/security.md) section 12. Copy this into a release PR or issue and check off each applicable item before tagging a release. Items that do not apply to a given tool (for example, MFA for a CLI tool with no user accounts) are marked N/A with a one-line reason, not silently omitted.

## Threat Modeling

- [ ] STRIDE threat model exists and is current for any attack surface introduced or changed since the last release
- [ ] Trust boundaries are documented for every external integration added since the last release

## Authentication and Authorization

- [ ] MFA enforced for all human access to production systems and secrets (or N/A: {{reason}})
- [ ] RBAC/ABAC definitions reviewed against actual usage; no unused elevated roles
- [ ] Managed Identity used for Azure-to-Azure auth wherever supported (or N/A: {{reason}})
- [ ] Session/token lifetimes reviewed and still appropriate

## Input Validation

- [ ] All new user-facing inputs validated at the boundary
- [ ] All new database queries use parameterized queries/ORM binding, no string-concatenated SQL
- [ ] All new output rendered into HTML/URL/shell contexts is properly encoded for that context

## Secrets Management

- [ ] No secret present in the diff, including test fixtures and example configuration
- [ ] Secret scanning clean on the release commit
- [ ] `.env.example` updated if new configuration keys were introduced
- [ ] Any new production secret is in Key Vault (or the project's equivalent secret store), not in a config file or CI variable where avoidable

## Personal and Third-Party Information

- [ ] No real employer, client, or colleague name, hostname, or IP address anywhere in the diff, including metadata fields (`Company`/`Publisher`/`Author` in `.csproj`, `Info.plist`, `package.json`, Cargo `authors`, installer scripts)
- [ ] Any example configuration, screenshot, or demo data added this release uses synthetic values, not real internal or production data
- [ ] If this tool originated in the context of employment, IP ownership has been clarified before this release

## Encryption

- [ ] TLS enforced for any new network endpoint
- [ ] New data classified as confidential/restricted is encrypted at rest
- [ ] No new use of a deprecated hash/cipher (MD5, SHA-1, DES) anywhere in the diff

## Error Handling

- [ ] Client-facing error messages reviewed: no stack traces, no internal paths, no raw database errors
- [ ] Detailed errors are still logged internally with a correlation ID for support/debugging

## Dependency Security

- [ ] `cargo audit`/`npm audit`/`pip-audit`/`dotnet list package --vulnerable` clean, or exceptions explicitly documented with a reason and a follow-up date
- [ ] Lock files committed and match the installed dependency tree
- [ ] SBOM generated for this release and attached to the GitHub Release

## Audit Logging

- [ ] New sensitive operations introduced since the last release emit an audit log entry (who, what, when, from where)
- [ ] Failed-authentication tracking/lockout still functions as expected (or N/A: {{reason}})

## Documentation

- [ ] README accurately reflects the current security posture (no claims about features, like a specific AI backend or auth method, that the code doesn't actually implement)
- [ ] Any known, accepted security limitation is documented with a `> **Note:**` callout and tracked in `ROADMAP.md`, not silently left undocumented

## Sign-off

- [ ] Reviewed by: {{name}}
- [ ] Date: {{YYYY-MM-DD}}
- [ ] Release version this checklist applies to: {{vX.Y.Z}}
