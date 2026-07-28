# Changelog

All notable changes to engineering-standards will be documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.6.0] - 2026-07-28

### Added

- `standards/ci-cd.md` section 6 gains a rule for macOS target architectures: release artifacts are universal binaries covering `arm64` and `x86_64`, the packaging target enforces this with `lipo -archs` instead of the README merely claiming it, and the binary path comes from `swift build --show-bin-path` rather than a hardcoded directory. Found in practice on `ServiceLLM`, which shipped four releases with an arm64-only DMG while no checklist anywhere caught it, because no rule on target architectures existed.

### Changed

- `standards/ci-cd.md` section 9 now specifies `actions/attest` instead of `actions/attest-build-provenance`. The latter is only a wrapper around the former as of its v4, and GitHub points new implementations at `actions/attest`. The section names the one migration trap: `actions/attest` requires a third permission, `artifact-metadata: write`, which otherwise only surfaces when the next tag push fails.

### Note

- The repository referred to as `CodeWhisper` in 0.5.3 was renamed to `ServiceLLM` on 2026-07-28.

## [0.5.3] - 2026-07-21

### Added

- `required_status_checks` promoted from a `WorkplaceAssessment`-only pilot to the portfolio-wide baseline: every public repository's `solo-main-protection` ruleset now technically blocks a merge with a failing check, not just the PR requirement. `ruleset-template.json` gets a minimal universal entry (`Analyze (actions)`) as the default for new repos; existing repos each have their own stack-specific required contexts applied directly via the API.
- `standards/ci-cd.md` section 7 documents a known caveat found during the rollout: the generic `CodeQL` check name reports `skipping` instead of `success` on some repos (observed on `SwiftAgent`, `NetSweep`, `CodeWhisper`), so it is deliberately left out of the required set there to avoid a required check that can never turn green.

## [0.5.2] - 2026-07-12

### Fixed

- Removed em-dashes from CLAUDE.md. Swiss German orthography rule.

## [0.5.1] - 2026-07-12

### Added

- `templates/new-repo-bootstrap-checklist.md` and `standards/ci-cd.md` section 9 now explicitly call out private vulnerability reporting and Dependabot security updates as pure repo-settings toggles (no workflow file, `gh api` only), alongside the already-documented CodeQL default setup. Discovered while enabling these across the portfolio: private vulnerability reporting was off everywhere, which meant the "report via GitHub Security Advisory" instruction in every repo's SECURITY.md did not actually work.
- Noted that `secret_scanning_non_provider_patterns` and `secret_scanning_validity_checks` require a paid GitHub Advanced Security license, unavailable on an individual Pro plan; documented as out of reach rather than silently omitted.

## [0.5.0] - 2026-07-12

### Added

- New `standards/ci-cd.md` section 2, "Action Pinning (Supply Chain Integrity)": every GitHub Action is pinned to a full commit SHA from the first commit, not a mutable tag or branch. Prompted by a real gap found in `github-actions-security-sandbox`, where an unpinned `dtolnay/rust-toolchain@stable` action contradicted the repository's own SECURITY.md.
- New `standards/ci-cd.md` section 9, "Automated Security Signals": CodeQL default setup, OpenSSF Scorecard, and build provenance/artifact attestation, as free, GitHub/Microsoft-native checks layered on top of the existing required pipeline stages. None require installing a third-party app or granting extra OAuth access.
- New `templates/security-policy-template.md`: a SECURITY.md template pointing to GitHub Security Advisories (not public issues) for vulnerability reporting, with an explicit usage note that every claim must be verified true before publishing, not copy-pasted aspirationally.
- New `templates/new-repo-bootstrap-checklist.md`: a Phase 0 through Phase 5 definition of done for a repository's first commit and first publish, converting every real security/CI gap this portfolio has hit into a fixed step at repo creation, instead of a correction discovered later.
- Updated `examples/ci-pipeline-example.yml` to pin every action to a commit SHA, matching the new section 2 requirement (the example previously used mutable tags itself).

### Fixed

- Section numbering in `standards/ci-cd.md` renumbered to accommodate the new sections 2 and 9 (previous sections 2-8 shifted to 3-8, 10).

## [0.4.0] - 2026-07-11

### Added

- New governance rule (`standards/governance.md` section 8): Dual-Licensing and Enterprise Feature Development. MIT releases are irrevocable, so newly developed enterprise-shaped features for Dual-Licensing candidate repositories are built privately, not in the public MIT-licensed main branch, until that repository's Community/Commercial split is actually implemented. Does not apply retroactively.

## [0.3.2] - 2026-07-11

### Fixed

- Replaced an eszett (ß) in README.de.md with "ss"; the project uses Swiss German orthography.

## [0.3.1] - 2026-07-11

### Fixed

- Replaced the LICENSE file's short-form CC BY 4.0 summary with the full canonical legal text, so GitHub's license detector correctly recognizes it as CC-BY-4.0 instead of showing "NOASSERTION".

## [0.3.0] - 2026-07-11

### Added

- Added the missing LICENSE file. This repository documents a personal working methodology and standards library, not a distributable software tool, so it is licensed under Creative Commons Attribution 4.0 International (CC BY 4.0) rather than MIT, which is used for the portfolio's actual software projects.

## [0.2.0] - 2026-07-11

### Added

- New standard: Personal and Third-Party Information (`standards/security.md` section 6, `CLAUDE.md`/`CLAUDE.en.md` Security-First section, `templates/security-checklist.md`). No repository may contain real employer/client/colleague names, hostnames, or IPs, and a project's metadata fields (Company/Publisher/Author) must be checked before publishing. Prompted by a real leftover employer reference found in a portfolio project's initial commit.

## [0.1.0] - 2026-07-10

### Fixed

- Changed the language-switch link from a blockquote to plain text
