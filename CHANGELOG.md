# Changelog

All notable changes to engineering-standards will be documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.14.1] - 2026-07-31

### Changed

- The pre-release checklist asked for "code review done". There is one maintainer and `required_approving_review_count` is 0 by design, so that box could only ever be ticked by pretending somebody else had looked, which trains the habit of ticking boxes rather than the habit of checking. It is replaced by two items that describe what actually happens and can be verified afterwards: the diff read end to end by its author with the reasoning recorded in the pull request, and every required status check green on the pull request rather than only locally. A short note states the reasoning, so the item does not quietly drift back.

---

## [0.14.0] - 2026-07-31

### Added

- `security.md` section 10 now covers the case it was missing: a vulnerability with no available fix. The existing rule only said what to do when a patch exists, which left six Tauri repositories carrying a permanently open Dependabot alert with nothing written down about it. The rule now requires establishing which direct dependency pulls the advisory in, whether an upgrade is possible at all, and whether the vulnerable path is reachable, then recording that in the repository's `SECURITY.md` with the condition that would end it.

---

## [0.13.0] - 2026-07-30

### Added

- `documentation.md` gains a section on the first screen of a README. Somebody arriving from a search result decides in seconds whether to keep reading, and a feature list does not help them decide. The section requires three things above the fold, in order: what the tool does for the reader in one plain sentence, who it is not for, and the shortest real example of using it. A feature list belongs underneath, because it is a reference for somebody who has already decided.
- The section carries a test rather than a preference: cover everything below the first screenful and ask whether a stranger could say what the tool is for and whether they want it. Without that, the rule would be a matter of taste and would be argued about instead of applied.

### Changed

- This repository's own READMEs now follow the rule they introduce. They opened by calling themselves the single versioned source of truth, which describes the document's standing rather than what a reader gets from it. They now say what is here, that standards kept in someone's head drift, and who should not adopt these wholesale: they are specific to one portfolio, one maintainer and a Microsoft-centred stack.

---

## [0.12.1] - 2026-07-30

### Changed

- Three tools appear under their new names: EmissaryKit was SwiftAgent, NetFathom was NetScanX, MailLoom was MailPilot. Each was renamed because another product carried the same name in the same category.
- The CodeQL caveat in `ci-cd.md` named the wrong cause. It blamed a default-setup versus committed-workflow interaction, but all three affected repositories now run a committed `codeql.yml` and still see `skipping`, while three others require the same check and see it turn green on every pull request. The cause is unidentified, and the note now says so instead of offering an explanation the evidence contradicts.

---

## [0.12.0] - 2026-07-29

### Added

- `standards/security-score.md`, recording what the OpenSSF Scorecard number can reach in this portfolio and what it cannot. Three checks (Code-Review, the gold tier of CII-Best-Practices, and the top tier of Branch-Protection) require a second person, so ten out of ten is not reachable by a single maintainer. The reachable ceiling is around eight, and the document lists the remaining steps in order of payoff, with the per-repository status as measured on 2026-07-29.
- The largest single loss was Token-Permissions, which drops to 0 the moment any workflow carries a top-level write permission. Two repositories already granted it per job and were the two highest scores in the portfolio, which is what turned this from an estimate into a measurement.

---

## [0.11.2] - 2026-07-29

### Changed

- The dash check in `writing-style.md` also looks for a hyphen used as a dash. The rule always covered that form, but the command next to it searched only for `—` and `–`, so the one variant a person actually types by hand passed every scan. Two sentences in a portfolio changelog broke the rule while the file reported clean, and that text is what release notes are generated from verbatim, so it was one step away from being published rather than merely stored.

---

## [0.11.1] - 2026-07-28

### Fixed

- `standards/writing-style.md` gave a `grep` over files as the mechanical check for the dash rule, but repository descriptions, topics and the profile bio live in API fields that no file grep can reach. Three descriptions had carried em-dashes through every previous scan of repositories whose contents were checked repeatedly. The section now includes a command that walks the descriptions of all repositories and the bio, and names release notes, issue titles and pull request titles as text that is equally out of reach of a file scan.

## [0.11.0] - 2026-07-28

### Changed

- `templates/dependabot.yml` and `standards/ci-cd.md` section 2 now require minor and patch updates to be grouped per ecosystem while majors arrive individually. The template shipped in 0.10.0 grouped everything, which produced a pull request carrying React 18 to 19, Tailwind 3 to 4, recharts 2 to 3 and zustand 4 to 5 in one diff, with an urgently needed security patch buried inside it. Actions stay grouped wholesale, majors included: the diff is a handful of SHA lines and a stale pin is a supply-chain concern rather than a compatibility one.

### Note

- Also learned on 2026-07-28, and worth recording because it nearly caused a wrong decision: the generic `CodeQL` status check is produced by the `github-advanced-security` app, not by CodeQL's default setup. It reports `skipping` when no analysis runs at all, which is what made dependency pull requests permanently unmergeable. Moving a repository to an advanced setup with a committed workflow makes it report `success` normally. Removing it from a ruleset's required checks is therefore not necessary and would weaken the gate for nothing.

## [0.10.0] - 2026-07-28

### Added

- `templates/dependabot.yml`: the configuration to copy into every repository, with grouped updates so one PR per ecosystem per week arrives instead of one per dependency.

### Changed

- `standards/ci-cd.md` section 2 now states that the Dependabot rule requires a committed `.github/dependabot.yml`, and that repository-level security alerts are not a substitute. Alerts only fire for disclosed vulnerabilities, so a repository can have them enabled, look configured, and still never receive a version update.
- `templates/new-repo-bootstrap-checklist.md` phase 2 asks for the file, not for a setting.

### Note

- A portfolio scan on 2026-07-28 found the file missing in 31 of 36 repositories. The consequence is visible in the `actions/checkout` pins, which are spread across v4, v6, v7.0.0 and v7.0.1; the three current ones are the repositories touched by hand today. Rollout follows in stages.

## [0.9.0] - 2026-07-28

### Added

- `standards/writing-style.md`: the dash rule with its rationale and the grep that checks it, Swiss orthography, German/English compound rules, first person in solo-maintainer documents, and the bilingual document rule. These constraints have governed every text in this portfolio for a while and have repeatedly caused rework, but they existed only in a locally installed Claude Code skill and were never published, so the repository enforced rules it did not state. Linked from both READMEs.

### Note

- The `writing-style-check` skill now points at this file as its canonical source, in the same way the other skills were realigned in 0.8.0. Its own description used a hyphen as a dash, which the rule it enforces forbids; that is fixed.

## [0.8.0] - 2026-07-28

### Added

- `standards/release-process.md`: ruleset setup for new repositories, versioning discipline, semantic versioning taken literally including what "finished" means for a major release, the release steps, the pre-release checklist and the rollback procedure.
- `standards/documentation.md`: README structure, API documentation, ADR, CHANGELOG format and the bilingual/first-person language conventions.
- `standards/microsoft-stack.md`: Graph API authentication, Azure deployment, Windows and ARM targeting, package management. Azure resource design stays in `azure-integration.md`; the new file cross-references it rather than duplicating it.
- `templates/portfolio-publish-checklist.md`: the pre-publish gate for a new tool, distinct from the per-release and security gates.
- All four are listed in both READMEs, so they are reachable without knowing they exist.

### Changed

- `CLAUDE.md` no longer carries the long-form content of sections 3, 6, 8, 9 and 10 and points at the files above instead. This commits a change that had been sitting uncommitted in the working tree: since `~/.claude/CLAUDE.md` is a symlink to this file, the shortened version was already in force in every session while the repository still showed the long one.
- The shortened version previously referenced Claude Code skills by name (`portfolio-release`, `portfolio-docs`, `microsoft-stack-guidelines`, `portfolio-publish-checklist`). Those live outside this repository, so a reader of this public repo, or any session on a machine without them, had no way to follow the reference. All pointers now resolve to versioned files here.
- `CLAUDE.en.md` follows the same structure again. It had kept the full long form, including a second copy of the security rules that section 2 has referenced out to `standards/security.md` for a while.

### Fixed

- Removed the last two em-dashes in the repository, both in `CLAUDE.md` section 2.

## [0.7.1] - 2026-07-28

### Fixed

- The tag verification command in `standards/ci-cd.md` section 2 did not work on the actions it was written for. It searched `git/refs/tags` for the pinned SHA, which fails twice over: the listing is paginated, so an action with many tags does not return the entry at all, and an annotated tag's `object.sha` is the tag object rather than the commit, so the comparison finds nothing even when the pin is correct. Both failure modes hit `github/codeql-action` and `ossf/scorecard-action` in practice. Replaced with `gh api repos/<owner>/<action>/commits/<tag> --jq '.sha'`, which dereferences either tag kind and returns the commit directly. Verified against all five actions pinned across the portfolio today.

## [0.7.0] - 2026-07-28

### Added

- `standards/ci-cd.md` section 2 gains "Tooling inside the job": linters, formatters, build and audit tools are pinned to an exact version, not an open range. Section 2 covered `uses:` lines only, so nothing stopped `AdapterForge` from declaring `ruff>=0.6`, picking up 0.16.0 at run time and going red on unchanged source because that release reordered imports. The same workflow already pinned `build` and `pip-audit` exactly, which is why the outlier went unnoticed.
- `templates/new-repo-bootstrap-checklist.md` phase 2 gains the matching checklist item.

### Changed

- Section 2 is now titled "Pinning (Supply Chain Integrity)" instead of "Action Pinning", with actions and job tooling as sub-sections. The section number is unchanged, so existing cross-references still resolve.

## [0.6.0] - 2026-07-28

### Added

- `standards/ci-cd.md` section 6 gains a rule for macOS target architectures: release artifacts are universal binaries covering `arm64` and `x86_64`, the packaging target enforces this with `lipo -archs` instead of the README merely claiming it, and the binary path comes from `swift build --show-bin-path` rather than a hardcoded directory. Found in practice on `ServiceLLM`, which shipped four releases with an arm64-only DMG while no checklist anywhere caught it, because no rule on target architectures existed.

- `standards/ci-cd.md` section 2 gains two rules that follow from the same release: verify the version comment on every Dependabot PR, because Dependabot bumps the SHA and leaves the comment on the old major, and pin every occurrence of an action to the same SHA, because separate workflow files drift apart.

### Changed

- `standards/ci-cd.md` section 9 now specifies `actions/attest` instead of `actions/attest-build-provenance`. The latter is only a wrapper around the former as of its v4, and GitHub points new implementations at `actions/attest`. The section names the one migration trap: `actions/attest` requires a third permission, `artifact-metadata: write`, which otherwise only surfaces when the next tag push fails.

### Fixed

- Removed 12 em-dashes from `templates/new-repo-bootstrap-checklist.md` and `standards/ci-cd.md`. Swiss orthography rule, and the standards repository should not violate the rule it defines.

### Note

- The repository referred to as `CodeWhisper` in 0.5.3 was renamed to `ServiceLLM` on 2026-07-28.
- `CLAUDE.md` still carries two em-dashes. It has uncommitted local changes from an earlier session, so it was deliberately left untouched here.

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
