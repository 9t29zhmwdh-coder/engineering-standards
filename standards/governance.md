# Governance

How decisions get made, reviewed, and kept current across this portfolio. This complements the day-to-day engineering rules in [`CLAUDE.md`](../CLAUDE.md); where the two overlap, `CLAUDE.md` is the operative, machine-enforced instruction set and this document is the rationale and process behind it.

## 1. Ownership Model

This is a solo-maintained portfolio. There is no second human reviewer, so governance substitutes structural controls for the peer review a larger team would provide:

- Technically enforced branch protection (see [`ci-cd.md`](ci-cd.md) section 6 and the [`ruleset-template.json`](../ruleset-template.json)) so no change, including by the owner, bypasses the PR flow.
- A risk-based merge policy (below) that requires an explicit pause for anything that isn't low risk, so a second look happens even without a second person.
- AI-assisted engineering is used deliberately and transparently rather than hidden; see [Section 12 of `CLAUDE.md`](../CLAUDE.md) for the AI transparency policy that governs this.

## 2. Risk-Based Merge Policy

| Risk level | Examples | Action |
|---|---|---|
| Low | Documentation, tests, CI configuration, dependency updates with green checks | Self-merge, then report what changed |
| Medium/High | Business logic, security-relevant changes, secrets/auth/permissions, breaking changes | Open the PR, show the diff, wait for explicit approval before merging |

This applies uniformly regardless of which machine or session performs the work, and it applies to changes to this governance document and to `CLAUDE.md` itself.

## 3. Policy Lifecycle

1. **Propose**: a new or changed standard is drafted as a PR against this repository, following the same PR flow as any other change.
2. **Review**: for anything beyond a low-risk documentation fix, the diff is reviewed explicitly before merge, per the risk-based policy above.
3. **Adopt**: once merged to `main`, the standard is considered active across the portfolio from that point forward. It is not applied retroactively to every existing repository as a mandatory backfill; it applies going forward and is checked against a given repository the next time that repository is meaningfully touched.
4. **Retire**: a standard that no longer reflects current practice (a deprecated tool, a superseded pattern) is removed or marked superseded in the same PR flow, with the reasoning recorded in the commit message.

## 4. Compliance Model

- **Self-attested, evidence-backed.** As a solo portfolio, compliance is not certified by a third party; it is demonstrated through visible artifacts: green CI, generated SBOMs, threat models, and a clean dependency audit trail.
- **OWASP Top 10** is the baseline security compliance reference for anything with a network-facing attack surface; see [`security.md`](security.md).
- **Licensing compliance**: every dependency's license is compatible with the repository's own license (MIT by default across the portfolio); a dependency audit flags license conflicts alongside vulnerabilities.

## 5. Living Standards {#living-standards}

This repository, and `CLAUDE.md` specifically, is a living document:

- **Cadence**: checked monthly against new Microsoft security advisories, GitHub security alerts, OWASP updates, and general best-practice shifts.
- **Process**: the monthly check produces a proposed diff, never a silent auto-update. The owner reviews and approves before anything in `CLAUDE.md` or the standards it links to changes.
- **Scope of the check**: `CLAUDE.md`, this `governance.md`, and the technical standards in [`standards/`](.) are all in scope for the monthly review; per-repository `README`/`CHANGELOG`/`ROADMAP` content is out of scope for this specific check (that content is reviewed when its repository is worked on).

## 6. Architecture Decision Records

Any decision with long-term structural consequences, a new data store, a cross-cutting framework choice, a service boundary, is recorded using [`../templates/architecture-decision-record.md`](../templates/architecture-decision-record.md). ADRs are immutable once accepted; a later change in direction is a new ADR that supersedes the old one, not an edit to history.

## 7. Exceptions

- An exception to any standard in this repository (a repo that cannot practically meet a given bar, for example a WPF app with no cross-platform CI runner available) is documented in that repository's own `README.md` or `ROADMAP.md`, stating what the exception is and why, rather than silently deviating.
- Security exceptions specifically (an accepted vulnerability, a deferred audit finding) are time-boxed: they carry a review date, not an open-ended acceptance.
