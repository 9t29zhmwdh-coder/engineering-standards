# Release Process

How a repository in this portfolio goes from a merged change to a published release, and how it gets rolled back. The branch protection that makes this enforceable lives in [`ci-cd.md`](ci-cd.md) section 7; this document covers the release itself.

## 1. Ruleset Setup for a New Repository

Every public repository gets the `solo-main-protection` ruleset on its default branch at creation time:

- `deletion`: branch deletion blocked
- `non_fast_forward`: force push blocked
- `pull_request`: PR required before merge, with `required_approving_review_count: 0`, since a solo workflow would otherwise deadlock on self-approval
- No `bypass_actor` is set, so the rules apply to the owner as well

```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset.json
```

The template lives in this repository as [`ruleset-template.json`](../ruleset-template.json).

**Monthly check:** verify every public repository still has the ruleset active (`gh api repos/<owner>/<repo>/rulesets`). A missing ruleset is restored in the audit PR.

**Portfolio-wide baseline (as of 2026-07-21):** `required_status_checks` is set on every public repository, so GitHub refuses the merge outright while a check is failing, rather than relying on discipline. `ruleset-template.json` carries a minimal universal entry (`Analyze (actions)`) as the default for new repositories; the checks actually enforced on an existing repository are stack-specific and were added individually via the API. One known limitation: the generic `CodeQL` check reports `skipping` instead of `success` on some repositories, so it is deliberately left out of the required set there. See [`ci-cd.md`](ci-cd.md) section 7.

## 2. Versioning Discipline

**Every merged change to a public repository is versioned in full**, including documentation-only fixes such as a README correction. That means all of the following, not a subset:

1. Bump the patch version in every file that carries one
2. Add the `CHANGELOG.md` entry
3. Create the git tag
4. Make sure the GitHub release exists

The release is part of "bump the version", not a separate decision to be raised afterwards. Before tagging, check that the intended version does not already exist as a tag; if it does, move to the next free patch number rather than forcing the tag.

### Step 4 depends on whether the repository has a release workflow

```bash
ls .github/workflows/release.yml
```

**If it exists, push the tag and stop.** The workflow builds the artifacts and creates the release itself. Do not run `gh release create` as well.

**If it does not exist**, create the release by hand:

```bash
gh release create vX.Y.Z --target main --notes "..."
```

**Why the distinction is written down rather than left to judgement.** On 2026-08-04 a release round called `gh release create` by hand in five repositories that already had a workflow doing the same thing on tag push. Whichever finished first won; the other aborted with `a release with the same tag name already exists`. The build had succeeded in every case, so the failure looked like a red run for no reason.

The real damage was elsewhere. In three of those repositories the workflow is what attaches the binaries, and it never got that far:

```
azure-policy-drift-detector v1.0.9        0 assets instead of 3
entra-least-privilege-analyzer v1.0.10    0 assets instead of 3
private-model-orchestrator v1.0.9         0 assets instead of 1
```

A release page that looks finished and offers nothing to download is worse than a visibly failed one, because nothing draws attention to it.

### Repairing a release that went out without its files

Without deleting anything:

1. `gh run download <run-id>` pulls the artifacts from the failed run, as long as they were uploaded as artifacts and have not expired.
2. `gh release upload vX.Y.Z <files>` attaches them to the release that already exists.

If the artifacts were built inside the release job itself and never uploaded separately, they are gone with the runner. Use the workflow's manual start instead: the release workflows take a `workflow_dispatch` with a tag input for exactly this case, and checkout and the version string follow that tag rather than the default branch.

### Release workflows create or update, they do not fail on an existing release

Since 2026-08-04 every release workflow in the portfolio checks first:

```bash
if gh release view "$TAG" >/dev/null 2>&1; then
  gh release upload "$TAG" <files> --clobber
else
  gh release create "$TAG" <files> --title "..." --notes "..."
fi
```

That removes the failure mode, but it does not remove the rule above: two processes creating the same release is still a race, and the one that loses produces a red run.

### Semantic Versioning, taken literally

- **Patch (0.0.X):** small adjustments, bug fixes, documentation corrections, CI configuration
- **Minor (0.X.0):** new features or larger changes, including ones that are a step in a bigger roadmap. Do not disguise a feature as a patch
- **Major (X.0.0):** reserved for a genuinely finished release that an end user can install

**What "finished" means for a major release:** not merely feature-complete in source, but concretely installable and runnable. An installer or package, a running Docker container, a launchable app, or a raw executable attached to a GitHub release all qualify; a `chmod +x` and run is enough to count as an installable distribution. A CLI tool with no release binary, installer or container path stays on 0.x.y even when it is functionally complete.

Classify before bumping: bug fix or documentation fix means patch, new functionality or a larger rework means minor. This applies across every repository in the portfolio without asking again each time.

## 3. Release Steps

1. Update the version in `package.json`/`setup.py`/`.csproj`/`Info.plist`/`pyproject.toml`
2. Update `CHANGELOG.md` (format in [`documentation.md`](documentation.md))
3. Create the release tag on the merged commit: fetch, check out the default branch as `gh repo view --json defaultBranchRef` reports it, and tag there. Before pushing, confirm the tag is the tip of the default branch (`git merge-base --is-ancestor <tag> origin/<default>` and the same SHA). Never derive the branch name from human-readable `git` output: on a German system `git remote show origin` says "Hauptbranch", a parser looking for "HEAD branch" gets nothing, and on 2026-09-25 four tags landed on feature-branch commits that way and started their release workflows before anyone noticed.
4. Push the tag: `git push origin v1.0.0`
5. Make sure the GitHub release exists, carrying the changelog entry as its notes. If the repository has a `release.yml` workflow, the tag push already did this and running `gh release create` on top of it is a race, see section 2
6. Deploy to production where applicable
7. Watch for errors
8. Never amend or rebase published commits

## 4. Pre-Release Checklist

- [ ] Version updated (MAJOR.MINOR.PATCH)
- [ ] `CHANGELOG.md` updated
- [ ] All tests passing
- [ ] Security checklist completed ([`../templates/security-checklist.md`](../templates/security-checklist.md))
- [ ] Dependencies audited
- [ ] Documentation updated
- [ ] The full diff read end to end by its author, with the reasoning for each change written into the pull request
- [ ] Every required status check green on the pull request, not merely locally
- [ ] Build artifacts tested
- [ ] Every promise in the README checked against the running build (section 4.1)

> **Why not "code review approved":** this portfolio has one maintainer, and
> `required_approving_review_count` is 0 by design, as section 1 explains. A box
> that can only be ticked by pretending somebody else looked trains the habit of
> ticking boxes. The two items above say what actually happens and can be
> checked afterwards: the pull request either carries the reasoning or it does
> not, and the checks are either green or they are not.

### 4.1 Live test against the README

Green CI proves that the code compiles and the tests pass. It does not prove the app does what its README says. The review round of 2026-09-24 to 2026-09-26 found, in repositories with green CI and published releases: an Execute button that never executed anything (CleanFlow), a log source that could never be added (BugRadar), a drift detector that never detected drift (NetFathom), a live mode that read every cost as zero (azure-cost-forecasting-engine), and a pipeline that had never run to the end (AdapterForge).

Before a release that touches behaviour:

1. Build the artifact the user gets, not only the test binary. For a Tauri app a debug bundle is enough (`npx @tauri-apps/cli build --debug --bundles app`), started with a separate `HOME`.
2. Walk through every feature the README names, with invented data (see [`security.md`](security.md) section 6). Check the outcome where it lands, in the file system, the database or the other service, not only on screen.
3. Watch for errors that go nowhere: an unhandled promise rejection, a panic in a command, a form that silently keeps its input. In a debug web view, "Inspect Element" opens the console.
4. What cannot be checked here (a tenant, hardware, a paid API) is named in the pull request as untested, not left out.

## 5. Rollback

- Every version has a git tag, which is what makes a rollback possible at all
- Roll back with `git checkout v1.0.0` or `git reset --hard <commit>`
- `CHANGELOG.md` documents breaking changes, so the cost of going back is visible before it is attempted
- Keep the previous version available, document the rollback steps, and test them in staging before relying on them in production
