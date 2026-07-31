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
4. Create the GitHub release with notes

```bash
gh release create vX.Y.Z --target main --notes "..."
```

The release is part of "bump the version", not a separate decision to be raised afterwards. Before tagging, check that the intended version does not already exist as a tag; if it does, move to the next free patch number rather than forcing the tag.

### Semantic Versioning, taken literally

- **Patch (0.0.X):** small adjustments, bug fixes, documentation corrections, CI configuration
- **Minor (0.X.0):** new features or larger changes, including ones that are a step in a bigger roadmap. Do not disguise a feature as a patch
- **Major (X.0.0):** reserved for a genuinely finished release that an end user can install

**What "finished" means for a major release:** not merely feature-complete in source, but concretely installable and runnable. An installer or package, a running Docker container, a launchable app, or a raw executable attached to a GitHub release all qualify; a `chmod +x` and run is enough to count as an installable distribution. A CLI tool with no release binary, installer or container path stays on 0.x.y even when it is functionally complete.

Classify before bumping: bug fix or documentation fix means patch, new functionality or a larger rework means minor. This applies across every repository in the portfolio without asking again each time.

## 3. Release Steps

1. Update the version in `package.json`/`setup.py`/`.csproj`/`Info.plist`/`pyproject.toml`
2. Update `CHANGELOG.md` (format in [`documentation.md`](documentation.md))
3. Create the release tag: `git tag v1.0.0`
4. Push the tag: `git push origin v1.0.0`
5. Create the GitHub release from the changelog entry
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

> **Why not "code review approved":** this portfolio has one maintainer, and
> `required_approving_review_count` is 0 by design, as section 1 explains. A box
> that can only be ticked by pretending somebody else looked trains the habit of
> ticking boxes. The two items above say what actually happens and can be
> checked afterwards: the pull request either carries the reasoning or it does
> not, and the checks are either green or they are not.

## 5. Rollback

- Every version has a git tag, which is what makes a rollback possible at all
- Roll back with `git checkout v1.0.0` or `git reset --hard <commit>`
- `CHANGELOG.md` documents breaking changes, so the cost of going back is visible before it is attempted
- Keep the previous version available, document the rollback steps, and test them in staging before relying on them in production
