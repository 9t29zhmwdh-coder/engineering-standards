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

## 8. Dual-Licensing and Enterprise Feature Development

- Repositories assessed as Dual-Licensing candidates (Community MIT plus a Commercial/Enterprise tier) keep their existing MIT-licensed code exactly as it is. An MIT release, once published, cannot be retroactively restricted; this is a legal fact about how MIT works, not a policy choice.
- Because of that, any newly developed feature that is enterprise-shaped (multi-tenant or multi-site aggregation, credentialed fleet enrichment, SIEM/compliance export, a centralized dashboard aggregating many local instances, and similar) is developed privately (a private branch or a separate private repository) rather than committed to the repository's public, MIT-licensed main branch, until that repository's actual Community/Commercial split is implemented.
- This rule is not retroactive: it does not apply to features already released under MIT, only to new work going forward, starting from the date a repository is identified as a Dual-Licensing candidate (see that repository's own `ROADMAP.md`, "Dual-Licensing Readiness" section).
- The portfolio's public "open source, MIT" positioning is unchanged by this rule: the eventual Community edition of a Dual-Licensing repository remains genuinely MIT and genuinely open. This rule only governs which new features get built where, before a repository's Commercial edition exists.
