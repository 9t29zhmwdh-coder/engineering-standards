# CI/CD Standards

Baseline for every GitHub Actions pipeline across the portfolio. See [`../examples/ci-pipeline-example.yml`](../examples/ci-pipeline-example.yml) for a complete reference implementation.

## 1. Runner Matrix

- Cross-platform tools (anything shipping a Windows and a macOS/Linux build) run their check/test matrix on both `windows-latest` and `ubuntu-latest` at minimum; `macos-latest` is added when the tool targets macOS specifically (for example, Tauri desktop apps).
- A pipeline is not considered green until every matrix leg passes; there is no "informational only" leg for a supported platform.

## 2. Required Pipeline Stages

Every repository's CI runs, at minimum, in this order:

1. **Lint**: language-appropriate static analysis (`clippy -D warnings`, `eslint`, `ruff`/`black --check`, `dotnet format --verify-no-changes`). Warnings are build failures, not advisories.
2. **Build**: a full compile/bundle for every target platform in the matrix.
3. **Test**: unit and integration tests; a repository with zero tests is a documented gap, not a silent skip (tracked in that repo's `ROADMAP.md`).
4. **Security scan**: dependency audit (`cargo audit`, `npm audit`, `pip-audit`, `dotnet list package --vulnerable`) plus secret scanning.
5. **SBOM generation**: CycloneDX SBOM produced and uploaded as a build artifact on every `main` build and every release.
6. **Package/Release** (on tag push): build the release artifact, attach the SBOM, publish the GitHub Release with generated notes from `CHANGELOG.md`.

No pipeline skips a stage by excluding a crate/package/project from the check command to work around a failing build; a failing stage is fixed, not scoped out. (This portfolio has hit this exact anti-pattern before, hiding real bugs behind a `--exclude` flag; it is treated as a defect in the pipeline itself when found.)

## 3. OIDC Federation for Azure Deployment

- GitHub Actions authenticates to Azure via **OIDC federated credentials** on an Entra ID app registration, never via a long-lived `AZURE_CREDENTIALS` client-secret JSON blob stored in repository secrets.
- The federated credential subject is scoped to the specific repository and branch (or environment) that is allowed to deploy; a wildcard subject is not acceptable for anything beyond a throwaway sandbox.
- Deployment permissions follow least privilege: the app registration's role assignment is scoped to the target resource group, not the subscription, unless the pipeline genuinely manages subscription-level resources.

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ vars.AZURE_CLIENT_ID }}
      tenant-id: ${{ vars.AZURE_TENANT_ID }}
      subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

## 4. SBOM Generation

- CycloneDX format, generated with the ecosystem-appropriate tool (`cyclonedx-npm`, `cargo cyclonedx`, `cyclonedx-py`, `dotnet CycloneDX`).
- Attached as a build artifact on every `main` build; attached to the GitHub Release on every tagged release.
- Retained for the lifetime of the release it documents, so a future vulnerability disclosure can be checked against historical releases without rebuilding them.

## 5. Release Automation

- A push of a `vMAJOR.MINOR.PATCH` tag triggers the release pipeline: build the release artifacts for every supported platform, attach the SBOM, and create the GitHub Release with notes sourced from the corresponding `CHANGELOG.md` entry.
- Release notes are never hand-typed into the GitHub UI as the source of truth; `CHANGELOG.md` is the source of truth and the release description is generated from it.
- Pre-release versions (`-alpha`, `-beta`, `-rc`) are marked as a GitHub pre-release and are not promoted to "latest" automatically.

## 6. Branch and Merge Gating

- The default branch is protected by the `solo-main-protection` ruleset (see the repository root [`ruleset-template.json`](../ruleset-template.json)): no force push, no branch deletion, PR required before merge.
- CI must be green on the PR's head commit before merge; a merge that bypasses a failing required check is not permitted by the ruleset and is not manually worked around.
- See [`governance.md`](governance.md) for the risk-based merge policy that governs *who* clicks merge, as distinct from the *technical* gate described here.

## 7. Caching and Performance

- Dependency caches (`actions/cache`, `Swatinem/rust-cache`, npm/pip caches) are used on every pipeline to keep CI feedback fast; a slow CI pipeline is treated as a productivity defect worth fixing.
- Caches are keyed on the lock file hash so a dependency bump correctly invalidates the cache rather than silently reusing stale dependencies.

## 8. Minimal Reference Matrix

| Stage | Windows | Ubuntu | macOS (if applicable) |
|---|---|---|---|
| Lint | ✅ | ✅ | ✅ |
| Build | ✅ | ✅ | ✅ |
| Test | ✅ | ✅ | ✅ |
| Security scan | optional (dependency audit is platform-independent; run once on Ubuntu) | ✅ | optional |
| SBOM | optional | ✅ | optional |
| Release packaging | ✅ (if Windows target) | ✅ (if Linux target) | ✅ (if macOS target) |
