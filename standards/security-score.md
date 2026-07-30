# OpenSSF Scorecard: what this portfolio can reach

Scorecard grades a repository from 0 to 10 across roughly eighteen checks.
The portfolio publishes the badge, so the number is read by anyone
evaluating the work. This document records what the number can become
here, what it cannot, and why.

## 1. Ten out of ten is not reachable by a single maintainer

Three checks require a second person. No amount of configuration
substitutes for one.

**Code-Review** measures whether changes were approved by somebody other
than their author. A solo maintainer opening a pull request and merging it
is unreviewed by that definition, however carefully it was written. The
check stays at 0.

**CII-Best-Practices** awards its full ten points only for the gold badge,
and gold states: "The project MUST have at least two unassociated
significant contributors" and "The project MUST have a bus factor of 2 or
more." Both are counts of people.

**Branch-Protection** reaches its top tier only when the branch requires
approving reviewers. That setting is available, but turning it on with
nobody to approve blocks every merge, including the security fix that is
in a hurry. The honest position is to leave it off and accept the tier.

These are not defects to fix. They are the check set measuring a property
this portfolio does not have and does not claim: a team.

## 2. The reachable ceiling is about eight

Everything below is achievable by one person, in rough order of effort
against payoff.

| Check | Now | Reachable | What it takes |
|---|---|---|---|
| Token-Permissions | 0 | 10 | Grant `contents: write` in the publishing job, not at workflow level |
| Maintained | 0 | 10 | Nothing. The 0 reads "repository created within the last 90 days" and expires on its own |
| Signed-Releases | 0 | 10 | Attach a SLSA provenance file (`*.intoto.jsonl`) to each release |
| Fuzzing | 0 | 10 | `cargo-fuzz` targets for the Rust tools, or ClusterFuzzLite |
| CII-Best-Practices | 2 | 5 | The passing badge, which a solo project can earn; gold cannot be |
| Branch-Protection | 3 | 6 | Everything short of required approvers |

Token-Permissions is the cheapest and was the largest single loss: the
check drops to 0 the moment **any** workflow carries a top-level write
permission, no matter how small the part of the run that needs it. One
line in `release.yml` held twenty-six repositories at zero while every
other setting around it was correct.

## 3. Evidence rather than estimate

Before the fix, exactly two repositories scored 10 on Token-Permissions:
`agent-governance-console` and `ServiceLLM`. Both already granted the
permission per job. They were also the two highest overall scores in the
portfolio, at 6.2 and 6.3 against a median near 5.2.

That is what makes this measurable rather than hopeful: the same change
had already been made twice by accident, and both times the check moved.

## 4. Per-repository status

Recorded 2026-07-29, before the Token-Permissions rollout. Three tools were
renamed on 2026-07-30 and appear under their new names: EmissaryKit was
SwiftAgent, NetFathom was NetScanX, MailLoom was MailPilot. The scores belong
to the same repositories. `Sig` is
Signed-Releases, `CII` is CII-Best-Practices. A dash means the check did
not apply.

| Repository | Score | Token | Sig | Fuzz | CII |
|---|---|---|---|---|---|
| ServiceLLM | 6.3 | 10 | 0 | 0 | 0 |
| agent-governance-console | 6.2 | 10 | 0 | 0 | 2 |
| EmissaryKit | 6.1 | 0 | - | 0 | 2 |
| WorkplaceAssessment | 5.8 | 0 | 0 | 0 | 2 |
| BugRadar | 5.5 | 0 | 0 | 0 | 2 |
| ClarityDesk | 5.5 | 0 | 0 | 0 | 2 |
| CleanFlow | 5.5 | 0 | 0 | 0 | 2 |
| github-actions-security-sandbox | 5.5 | 0 | 0 | 0 | 2 |
| LogLens | 5.5 | 0 | 0 | 0 | 2 |
| private-model-orchestrator | 5.5 | 0 | 0 | 0 | 2 |
| StateForge | 5.5 | 0 | 0 | 0 | 2 |
| RepoLedger | 5.4 | 0 | - | 0 | 2 |
| GardenFlow | 5.3 | 0 | - | 0 | 2 |
| HomePortal | 5.3 | 0 | - | 0 | 2 |
| AdapterForge | 5.2 | 0 | 0 | 0 | 2 |
| entra-access-graph-engine | 5.2 | 0 | 0 | 0 | 2 |
| LifePlanner | 5.2 | 0 | 0 | 0 | 2 |
| NetSweep | 5.2 | 0 | 0 | 0 | 2 |
| azure-policy-drift-detector | 5.1 | 0 | 0 | 0 | 2 |
| entra-least-privilege-analyzer | 5.1 | 0 | 0 | 0 | 2 |
| NetFathom | 5.1 | 0 | 0 | 0 | 2 |
| azure-cost-forecasting-engine | 5.0 | 0 | 0 | 0 | 2 |
| eventhub-otlp-mapper | 5.0 | 0 | 0 | 0 | 2 |
| DeviceHealth | 4.8 | 0 | 0 | 0 | 2 |
| LifeSort | 4.8 | 0 | 0 | 0 | 2 |
| MailLoom | 4.8 | 0 | 0 | 0 | 2 |
| SiliconMark | 4.8 | 0 | 0 | 0 | 2 |
| NetDashboard | 4.5 | 0 | 0 | 0 | 2 |

## 5. Order of work

1. **Token-Permissions**, twenty-six repositories, one line each. Done
   2026-07-29.
2. **Signed-Releases**, via `actions/attest`. The release workflows are
   already uniform enough to take the same step in each.
3. **Fuzzing**, Rust repositories first, since `cargo-fuzz` needs a target
   function rather than a new pipeline.
4. **CII-Best-Practices passing badge**, one questionnaire per repository,
   worth doing once the three above have settled.

Scorecard runs weekly, so a change shows in the badge on the next
scheduled run rather than at merge.

## 6. What not to do

Do not chase the number past what the project is. Turning on required
approvers with no reviewer, or filing a gold badge questionnaire that
overstates the contributor count, buys points by describing a project that
does not exist. The badge is only worth publishing while it is accurate.
