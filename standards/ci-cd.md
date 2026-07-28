# CI/CD Standards

Baseline for every GitHub Actions pipeline across the portfolio. See [`../examples/ci-pipeline-example.yml`](../examples/ci-pipeline-example.yml) for a complete reference implementation.

## 1. Runner Matrix

- Cross-platform tools (anything shipping a Windows and a macOS/Linux build) run their check/test matrix on both `windows-latest` and `ubuntu-latest` at minimum; `macos-latest` is added when the tool targets macOS specifically (for example, Tauri desktop apps).
- A pipeline is not considered green until every matrix leg passes; there is no "informational only" leg for a supported platform.

## 2. Action Pinning (Supply Chain Integrity)

- Every third-party GitHub Action used in a workflow is pinned to a full commit SHA, not a mutable version tag (`@v2`) or branch/alias (`@stable`, `@main`). A tag or branch can be force-moved to point at different, potentially malicious code after the fact without changing the workflow file; a commit SHA cannot.
- Pin with a trailing comment noting the human-readable version for maintainability: `uses: actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7.0.0`.
- This applies to first-party (`actions/*`) and third-party actions alike; there is no exception for "well-known" publishers.
- Dependabot (or Renovate) is configured with `github-actions` ecosystem updates enabled, so pinned SHAs are bumped via a normal, reviewable PR on a schedule rather than silently freezing a repository on a stale or later-disclosed-vulnerable action version.
- This applies from the first commit of a new repository. A repository found running an unpinned action is corrected as soon as discovered, with the fix treated as a normal patch release, the same retroactive-correction rule as Section 6 of `security.md`.

## 3. Required Pipeline Stages

Every repository's CI runs, at minimum, in this order:

1. **Lint**: language-appropriate static analysis (`clippy -D warnings`, `eslint`, `ruff`/`black --check`, `dotnet format --verify-no-changes`). Warnings are build failures, not advisories.
2. **Build**: a full compile/bundle for every target platform in the matrix.
3. **Test**: unit and integration tests; a repository with zero tests is a documented gap, not a silent skip (tracked in that repo's `ROADMAP.md`).
4. **Security scan**: dependency audit (`cargo audit`, `npm audit`, `pip-audit`, `dotnet list package --vulnerable`) plus secret scanning.
5. **SBOM generation**: CycloneDX SBOM produced and uploaded as a build artifact on every `main` build and every release.
6. **Package/Release** (on tag push): build the release artifact, attach the SBOM, publish the GitHub Release with generated notes from `CHANGELOG.md`.

No pipeline skips a stage by excluding a crate/package/project from the check command to work around a failing build; a failing stage is fixed, not scoped out. (This portfolio has hit this exact anti-pattern before, hiding real bugs behind a `--exclude` flag; it is treated as a defect in the pipeline itself when found.)

## 4. OIDC Federation for Azure Deployment

- GitHub Actions authenticates to Azure via **OIDC federated credentials** on an Entra ID app registration, never via a long-lived `AZURE_CREDENTIALS` client-secret JSON blob stored in repository secrets.
- The federated credential subject is scoped to the specific repository and branch (or environment) that is allowed to deploy; a wildcard subject is not acceptable for anything beyond a throwaway sandbox.
- Deployment permissions follow least privilege: the app registration's role assignment is scoped to the target resource group, not the subscription, unless the pipeline genuinely manages subscription-level resources.

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@a457da9ea143d694b1b9c7c869ebb04ebe844ef5 # v2
    with:
      client-id: ${{ vars.AZURE_CLIENT_ID }}
      tenant-id: ${{ vars.AZURE_TENANT_ID }}
      subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

## 5. SBOM Generation

- CycloneDX format, generated with the ecosystem-appropriate tool (`cyclonedx-npm`, `cargo cyclonedx`, `cyclonedx-py`, `dotnet CycloneDX`).
- Attached as a build artifact on every `main` build; attached to the GitHub Release on every tagged release.
- Retained for the lifetime of the release it documents, so a future vulnerability disclosure can be checked against historical releases without rebuilding them.

## 6. Release Automation

- A push of a `vMAJOR.MINOR.PATCH` tag triggers the release pipeline: build the release artifacts for every supported platform, attach the SBOM, and create the GitHub Release with notes sourced from the corresponding `CHANGELOG.md` entry.
- Release notes are never hand-typed into the GitHub UI as the source of truth; `CHANGELOG.md` is the source of truth and the release description is generated from it.
- Pre-release versions (`-alpha`, `-beta`, `-rc`) are marked as a GitHub pre-release and are not promoted to "latest" automatically.

### Target architectures for macOS releases

- A macOS release artifact is a universal binary covering `arm64` and
  `x86_64`, for as long as Apple's toolchain still supports the latter.
  Building on an Apple Silicon machine defaults to `arm64` only, so an
  Intel Mac cannot start the result. Nothing in the pipeline notices this
  by itself, which is exactly how it stays unnoticed until a user reports
  it.
- The build enforces this rather than the documentation asserting it: the
  packaging target runs `lipo -archs` on the binary it is about to bundle
  and fails when both architectures are not present. Documenting the
  requirement in the README is not sufficient, because the README is not
  what produces the artifact.
- Multi-architecture output does not land in the single-architecture build
  directory, and SwiftPM has moved that path between releases. Ask for it
  with `swift build --show-bin-path` and the same architecture flags used
  for the build instead of hardcoding a path. A hardcoded path silently
  bundles the previous single-architecture binary, which produces a
  release that contradicts its own release notes.
- Apple is phasing `x86_64` out; recent SDKs emit a deprecation warning for
  it. Treat continued Intel support as a decision to revisit per release,
  not as a permanent guarantee. Once dropped, say so in the README's
  requirements and in the download line, not only in a badge.

## 7. Branch and Merge Gating

- The default branch is protected by the `solo-main-protection` ruleset (see the repository root [`ruleset-template.json`](../ruleset-template.json)): no force push, no branch deletion, PR required before merge.
- CI must be green on the PR's head commit before merge; a merge that bypasses a failing required check is not permitted by the ruleset and is not manually worked around.
- See [`governance.md`](governance.md) for the risk-based merge policy that governs *who* clicks merge, as distinct from the *technical* gate described here.

### Status: `required_status_checks` is now the portfolio-wide baseline

As of 2026-07-21, every public repository's `solo-main-protection` ruleset carries a `required_status_checks` rule (`strict_required_status_checks_policy: false`), so the line above ("not permitted by the ruleset") is now the enforced technical baseline, not just discipline. The pilot on `WorkplaceAssessment` (2026-07-21, earlier the same day) validated the approach before the portfolio-wide rollout.

`ruleset-template.json` at the repository root now includes a minimal, universal `required_status_checks` entry (`Analyze (actions)`, present in every repo via the shared CodeQL/security workflow). That single context is a safe default for brand-new repos; it is **not** the full list actually enforced on existing repos. Each existing repository's ruleset additionally requires that repo's own primary lint/test job(s) by name (e.g. `Check`, `Lint & Check`, `Coverage`, `Security audit`, per-OS `Check (ubuntu-latest)`/`Check (windows-latest)`/`Check (macos-latest)` for matrix builds), applied directly via the API per repo, not through this template (context names differ per stack and can't be expressed generically here).

**Known caveat, checked during the rollout:** the generic `CodeQL` check name is unreliable on some repos, it reports `skipping` instead of `success` on a normal PR (observed on `SwiftAgent`, `NetSweep`, `CodeWhisper`), likely a GitHub default-setup vs. committed-workflow interaction. `CodeQL` is required only on repos where it was directly confirmed to report `success` on a real PR; it is deliberately left out of the required set for the affected repos to avoid a required check that can never turn green. Revisit if GitHub's CodeQL default-setup behavior changes.

## 8. Caching and Performance

- Dependency caches (`actions/cache`, `Swatinem/rust-cache`, npm/pip caches) are used on every pipeline to keep CI feedback fast; a slow CI pipeline is treated as a productivity defect worth fixing.
- Caches are keyed on the lock file hash so a dependency bump correctly invalidates the cache rather than silently reusing stale dependencies.

## 9. Automated Security Signals

Beyond the dependency audit already required in Section 3, several free,
GitHub/Microsoft-native signals catch classes of problem a manual audit
misses or forgets to re-check on every change. The first three below are
pure repository-settings toggles, enabled via the GitHub API with no
workflow file and no third-party app install; the next two need an actual
workflow file:

- **Private vulnerability reporting**: enabled via
  `gh api -X PUT repos/<owner>/<repo>/private-vulnerability-reporting`.
  Without this, the "report via GitHub Security Advisory" instruction in
  `SECURITY.md` (from `templates/security-policy-template.md`) does not
  actually accept a submission — verify it is on, do not assume it defaults
  to on.
- **Dependabot security updates**: enabled via
  `security_and_analysis.dependabot_security_updates.status` in a
  `gh api -X PATCH repos/<owner>/<repo>` call. Distinct from the
  version-update `dependabot.yml` config in Section 2, which opens routine
  bump PRs; this toggle specifically reacts to disclosed vulnerabilities.
- **Secret scanning and push protection**: on by default for public
  repositories, but verify with
  `gh api repos/<owner>/<repo> --jq .security_and_analysis` rather than
  assume, the same way every other claim in this document is verified, not
  asserted. Two further sub-features
  (`secret_scanning_non_provider_patterns`, `secret_scanning_validity_checks`)
  require a paid GitHub Advanced Security license and are out of reach on
  an individual Pro plan; do not claim them as enabled without checking.
- **CodeQL** (GitHub's static analysis security scanner, Microsoft-backed
  research): enabled via
  `gh api -X PATCH repos/<owner>/<repo>/code-scanning/default-setup
  -f state=configured -f query_suite=default` for every supported
  language. Finds real code-level vulnerabilities (injection, unsafe
  deserialization, path traversal) that a dependency audit does not,
  because a dependency audit only checks known-vulnerable *packages*, not
  vulnerable code written in this repository.
- **OpenSSF Scorecard**: a scheduled workflow (`ossf/scorecard-action`)
  that scores the repository against supply-chain hygiene checks,
  including the exact thing Section 2 requires by hand (pinned actions),
  branch protection, and dependency update tooling. This turns Section 2's
  rule from something that has to be remembered into something checked on
  a schedule automatically, and the resulting badge is a public, verifiable
  signal of the repository's posture (relevant to this portfolio's
  visible-governance positioning, `CLAUDE.md` section 12).
- **Build provenance / artifact attestation** (`actions/attest`): for any
  repository that ships a packaged installer, the release job generates a
  signed attestation proving the artifact was built by this repository's
  actual CI from this actual source commit, not substituted or tampered
  with afterward. This matters once a packaged build is distributed for
  payment (see `COMMERCIAL.md`/`TERMS_OF_SALE.md` where applicable): a
  purchaser, including a corporate one, can verify the binary they
  received is genuine. Verify from the consumer's side with
  `gh attestation verify <artifact> -R <owner>/<repo>`, which is also the
  cheapest way to confirm the release job actually signed anything.

  Use `actions/attest`, not `actions/attest-build-provenance`. As of its
  v4, the latter is only a wrapper around the former, and GitHub points
  new implementations at `actions/attest` directly. The default mode
  produces the same SLSA build provenance, so migrating means changing the
  `uses:` line and nothing else, with one exception that will otherwise
  only surface at the next tag push: `actions/attest` needs a third
  permission, `artifact-metadata: write`, alongside `id-token: write` and
  `attestations: write`.

None of these three replace Section 3's required stages; they are
additional, low-effort signals layered on top, enabled once at repository
creation (Phase 2 of
[`../templates/new-repo-bootstrap-checklist.md`](../templates/new-repo-bootstrap-checklist.md)).

### README badge order

The badge row is three lines, not one, grouped by what each badge actually
certifies, so the certification signals stay scannable instead of buried
among platform/tech badges:

- **Line 1, "Certs.":** CI, CodeQL, OpenSSF Scorecard, OpenSSF Best
  Practices (once a repo is registered on
  bestpractices.coreinfrastructure.org). This is the "does it work, is it
  scanned, is it hygienic" row, ordered from most to least fundamental.
  Every future professionalism/compliance badge (a new scanner, a new
  certification signal) is added here, not to Line 2 or 3.
- **Line 2, "Tech.":** Microsoft/Apple badge (if applicable), Platform,
  language/framework badges, AI-tooling badges. The stack, not a
  certification.
- **Line 3, "Div.", only if the repo actually has one of these:** the
  dynamic GitHub Release badge, and anything else that fits neither Line 1
  nor Line 2. Do not add this line for a repo that has nothing to put in
  it. No separate License badge belongs here or anywhere, GitHub already
  surfaces MIT via the repo's own "license" tab (unchanged rule); if an
  existing repo still carries one, that is a standing violation to clean
  up, not a reason to add it back.

Never let a single badge end up alone on its own line by itself when a
line otherwise would have had more than one: markdown treats a lone image
link on its own line as visually isolated (reads as disconnected rather
than part of a row). Add new badges to whichever of the three lines
matches what they certify, never as an ad-hoc fourth line.

## 10. Minimal Reference Matrix

| Stage | Windows | Ubuntu | macOS (if applicable) |
|---|---|---|---|
| Lint | ✅ | ✅ | ✅ |
| Build | ✅ | ✅ | ✅ |
| Test | ✅ | ✅ | ✅ |
| Security scan | optional (dependency audit is platform-independent; run once on Ubuntu) | ✅ | optional |
| SBOM | optional | ✅ | optional |
| Release packaging | ✅ (if Windows target) | ✅ (if Linux target) | ✅ (if macOS target) |
