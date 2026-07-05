<div align="center"><img src="RayStudio.png" alt="RayStudio Logo" width="120"/><h1>engineering-standards</h1></div>

> 🇩🇪 [Deutsche Version](README.de.md)

Engineering standards and best practices for all portfolio tools: Microsoft-focused, Security-First, Senior Engineer Standards.

## What is this?

This repo is the single versioned source of truth for how code is written, reviewed, merged and released across all portfolio repositories. Every machine (Windows, Mac) links its Claude Code global instructions to the `CLAUDE.md` in this repo, so the same rules apply everywhere.

## Contents

| File | Purpose |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | The standards (German, normative). Linked as global Claude Code instructions on every machine. |
| [`CLAUDE.en.md`](CLAUDE.en.md) | English reference translation of the standards. |
| [`ruleset-template.json`](ruleset-template.json) | GitHub ruleset `solo-main-protection`: blocks branch deletion and force push, enforces PR flow on the default branch. |

## Quick start (new machine)

```bash
git clone https://github.com/9t29zhmwdh-coder/engineering-standards.git ~/engineering-standards
ln -sf ~/engineering-standards/CLAUDE.md ~/.claude/CLAUDE.md
```

## Apply the ruleset to a new repo

```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset-template.json
```

## Key principles

- **Security-First:** MFA, input validation, secrets management, audit logging and dependency security are built in from day 1, not bolted on later.
- **Enforced branch protection:** No direct or force push to `main`, PR flow is mandatory, technically enforced via GitHub rulesets (also for the owner).
- **Risk-based merge policy:** Low-risk changes (docs, tests, CI) are self-merged and reported; medium/high-risk changes (business logic, security, auth) wait for an explicit OK.
- **Living standards:** Checked monthly against Microsoft security advisories, OWASP updates and best practice changes; updated only with explicit approval.

---

**Author:** [Rafael Yilmaz](https://github.com/9t29zhmwdh-coder) · **Status:** Active
