# New Repository Bootstrap Checklist

Definition of done for a repository's **first commit and first publish**,
before any feature code is written. Distinct from
[`security-checklist.md`](security-checklist.md), which governs every
*subsequent* release of an already-existing repository.

The reason this exists as its own document: every security/CI gap this
portfolio has actually hit (unpinned GitHub Actions, a gitignored lock file,
a SECURITY.md asserting a practice that wasn't real, a missing installer
build, an employer name leaking into project metadata) was a decision that
should have been made once, structurally, at repo creation, rather than
re-derived per project and occasionally forgotten. This checklist converts
each of those lessons into a fixed step, in the order they need to happen,
so a new repository cannot skip a step by never having thought about it.

Correcting an existing repository against this checklist after the fact is
still valid (see `security.md` section 6's retroactive-correction rule), but
the goal is that new repositories don't need the correction in the first
place.

## Phase 0 — Decisions made once, before any file exists

- [ ] License model decided: MIT (default for this portfolio) or, if the
      project is a plausible Dual-Licensing candidate (an enterprise-shaped
      problem space exists), draft the "Dual-Licensing Readiness" section
      of `ROADMAP.md` now, even if the honest answer is "not ready, no
      Enterprise features planned yet."
- [ ] If the project originated in the context of employment, IP ownership
      is clarified before the first commit, not after (`security.md`
      section 6). When in doubt, do not create the public repository yet.
- [ ] Repository name and description contain no employer, client, or
      colleague reference.

## Phase 1 — Repository scaffold (mechanical, identical every time)

- [ ] `LICENSE` (MIT, from Phase 0's decision)
- [ ] `.gitignore` for the target language, reviewed line by line: it does
      **not** ignore the dependency lock file for an application (Cargo.lock
      for a Rust binary, package-lock.json for a Node app). Libraries are
      the only case where an unlocked dependency file is normal; if this
      repository is an application, the lock file ships in the diff.
- [ ] `README.md` / `README.de.md` from `templates/readme-template.md`
- [ ] `CHANGELOG.md`, starting at `[0.1.0]`, Keep a Changelog format
- [ ] `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `PRIVACY.md`
- [ ] `ARCHITECTURE.md` skeleton (even a one-paragraph placeholder beats
      nothing; expand as the design solidifies)
- [ ] `ROADMAP.md`, including the Dual-Licensing Readiness section from
      Phase 0
- [ ] `SECURITY.md` from `templates/security-policy-template.md`, every
      placeholder filled in and every claim verified true against this
      repository's actual state (not copy-pasted aspirationally — see that
      template's usage note)
- [ ] `solo-main-protection` ruleset applied to the default branch via
      `gh api repos/<owner>/<repo>/rulesets` **at creation**, not deferred
      until after the first release (`governance.md`, `ci-cd.md` section 7)

## Phase 2 — CI baseline (mechanical, in place before the first feature PR)

- [ ] CI workflow adapted from `examples/ci-pipeline-example.yml`
- [ ] Every action pinned to a commit SHA from the first commit
      (`ci-cd.md` section 2) — there is no "pin it later" grace period
- [ ] Lint, build, and test stages present, even if tests are currently
      minimal; a zero-test repository is a tracked `ROADMAP.md` gap, not a
      silently skipped CI stage
- [ ] Security-audit job (`cargo audit`/`npm audit`/`pip-audit`/
      `dotnet list package --vulnerable`) present and green **before the
      first PR merges**, not added reactively after a scan finds something
- [ ] Dependabot enabled for the `github-actions` ecosystem and the
      project's package ecosystem, so SHA pins and dependencies get
      reviewable bump PRs instead of silently aging
- [ ] Release workflow present if this repository ships a packaged
      installer (Tauri bundler, PyInstaller, Inno Setup, `dotnet publish`);
      confirmed to actually attach build artifacts to a real tag push
      before relying on it (see Phase 4)
- [ ] CodeQL default setup enabled in the repository's Security settings
      (`ci-cd.md` section 9)
- [ ] OpenSSF Scorecard workflow added (`ci-cd.md` section 9); badge added
      to `README.md` once the first scan has run
- [ ] Build provenance attestation added to the release job if this
      repository ships a packaged installer (`ci-cd.md` section 9),
      especially relevant before any paid marketplace distribution

## Phase 3 — Feature development

Normal engineering standards apply (`coding-rust.md`, `coding-python.md`,
`coding-typescript.md`, `architecture.md`, `observability.md`). No special
bootstrap concern here beyond what Phases 1–2 already locked in.

## Phase 4 — Pre-first-publish verification

Run immediately before the repository's first public push / first tagged
release, since there is no "since the last release" scope to shrink this
to yet — everything is in scope:

- [ ] Full pass of `security-checklist.md`, every applicable item, not just
      the parts touched by the most recent commit
- [ ] Every claim in `SECURITY.md` re-verified against the actual CI
      config and repository state (grep the workflow file for the action
      pin, check `git ls-files` for the lock file, don't trust memory)
- [ ] No personal or third-party identifying information anywhere in code,
      metadata, or docs (`security.md` section 6): `Company`/`Publisher`/
      `Author` fields in `.csproj`, `Info.plist`, `package.json`, Cargo
      `authors`, installer scripts
- [ ] Portfolio orthography rules satisfied if applicable (Swiss German
      `ss` not `ß`; no em-dash/en-dash in any generated or hand-written
      text)
- [ ] Version fields in sync across every manifest that carries one
      (workspace `Cargo.toml` plus every member crate, `package.json`,
      `tauri.conf.json`, `.csproj`, `pyproject.toml`)
- [ ] If a Dual-Licensing skeleton applies (Phase 0), `LICENSE.COMMERCIAL`,
      `COMMERCIAL.md`, and `ENTERPRISE_FEATURES.md` are internally
      consistent with `ROADMAP.md`'s Dual-Licensing Readiness section

## Phase 5 — First release

- [ ] Tag pushed; local and `origin` `main`/`master` SHA compared and
      confirmed equal **before** tagging (an unpinned default-branch name
      or a premature tag on the wrong branch has caused a real
      mistagged release in this portfolio; verify, don't assume)
- [ ] If a release-build workflow exists, the created GitHub Release is
      checked for actually-attached installer assets, not just a green CI
      run (a release job can go green while silently failing to attach
      anything if it collides with a pre-existing release of the same tag)
- [ ] Release notes sourced from `CHANGELOG.md`, not hand-typed
