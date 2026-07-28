# Claude Code Guidelines: Senior Software Engineer Standards
**Portfolio Edition** | Microsoft-focused | Security-First | Public Code Quality

> 🇩🇪 [Deutsche Version (normativ)](CLAUDE.md). This English file is a reference translation. The German `CLAUDE.md` is the normative source loaded by Claude Code.

These guidelines apply to all projects, especially public portfolio tools on GitHub.

---

## 1. SENIOR ENGINEER CORE BEHAVIOR

### Never
- **Delete files without comment**: If a file should be deleted, justify it first and ask (unless it is obviously redundant, like temp files)
- **Perform large refactorings without a plan**: Before major restructuring, create a plan that clarifies the impact
- **Present changes without a diff**: Before committing, always show and explain the diff
- **Commit secrets to code**: Never. Passwords, API keys and tokens belong in a secret manager (Azure Key Vault, 1Password)

### Always
- **Create a plan**: For non-trivial changes (>1 file or complex logic)
- **Analyze impact**: Affected files, dependencies, side effects
- **Review the git diff**: Show and explain before committing
- **Run tests**: If tests exist, run them before committing
- **Deliver a summary**: What, why, side effects
- **Atomic commits**: One commit = one logical change
- **Semantic commit messages**: `type(scope): description` (feat, fix, security, refactor, test, docs)

---

## 2. SECURITY-FIRST MINDSET (NOT OPTIONAL)

**This is the biggest difference from standard code.** For public portfolio tools, security MUST be built in from day 1.

### Security standard: canonical source

The full, binding security standard lives in this repository under `standards/security.md`, rather than being duplicated here as a second list that would drift out of sync:

- `standards/security.md`, 12 sections: STRIDE threat modeling, zero trust principles, secure defaults, identity and access (Entra ID, MFA, RBAC/ABAC, Managed Identity), secrets management, personal and third-party information (top priority, since 2026-07-11), input validation and sanitization, encryption, secure error handling, dependency governance and SBOM (CycloneDX), audit logging, release gate
- `templates/security-checklist.md`, the checklist to work through per release, including sign-off (reviewer, date, version)

Before every release: copy `templates/security-checklist.md` and work through it, rather than reconstructing it from memory.

---

## 3. VERSIONING & ROLLBACK STRATEGY

**This is critical for your portfolio.** If code gets messed up, you need a clean rollback.

### Git workflow
- **Main/Master:** ALWAYS production-ready, ALWAYS stable
- **feature/xxx:** Feature branches for development
- **Branch protection:**
  - No direct push to main (enforce it!)
  - No force push to main (ever!)
  - PR required before merge (solo maintainer: no required approval needed, but PR flow is mandatory)

### GitHub Ruleset (technically enforced, not just policy)

Every public repo gets the ruleset `solo-main-protection` on the default branch at creation. Setup commands, template path, monthly auto-check and the portfolio-wide baseline (including the known CodeQL limitation) are in `standards/release-process.md` section 1.

### Semantic Versioning (MAJOR.MINOR.PATCH)
- **MAJOR:** Breaking changes (e.g. API breaking change)
- **MINOR:** New features (backward compatible)
- **PATCH:** Bug fixes only

**Example:**
```
v1.0.0 -> v1.1.0 (new feature)
v1.1.0 -> v1.1.1 (bug fix)
v1.x.x -> v2.0.0 (breaking change)
```

### Release process & rollback capability
Step-by-step process, versioning discipline (every merged change is versioned, tag and release included) and rollback procedure are in `standards/release-process.md`.

---

## 4. TESTING STRATEGY

### Test pyramid
```
      E2E (critical flows only)
    Integration tests (APIs, DB)
  Unit tests (business logic) <- MOST
```

### Minimum standards
- **Unit tests:** Business logic, utilities (should run <1s, no I/O)
- **Integration tests:** APIs, database interactions (against a test DB)
- **E2E tests:** ONLY for critical user flows (too slow for everything)
- **Coverage target:** ~80% (not 100%, but also not <50%)

### Before every commit
```bash
npm test          # or pytest, dotnet test
npm run lint      # or black, eslint
npm run build     # or python setup.py build
```

### What NOT to test
- Trivial getters/setters
- Framework-provided code
- External APIs (mock them instead)

---

## 5. CODE QUALITY & ARCHITECTURE

### SOLID principles (Microsoft standard)
Standard SOLID principles, applied consistently.

### DRY, KISS, YAGNI
- **DRY (Don't Repeat Yourself):** 3x copy = abstract, 2x = consider, 1x = ok
- **KISS (Keep It Simple, Stupid):** simplicity > cleverness
- **YAGNI (You Aren't Gonna Need It):** no features for a hypothetical future

### Naming conventions
- **Functions:** verb + noun (`getUserById`, `validateEmail`, `processPayment`)
- **Variables:** clear intent (not `x`, `temp`, `data`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_RETRIES`, `DB_TIMEOUT`)
- **Classes:** PascalCase (`UserService`, `AuthController`)

### Function/method size
- **Max 20 lines** per function (ideally <10)
- If a class exceeds 300 lines, split it into several

### Comments
- **Only WHY, not WHAT**: `x++` is clear, but explain why it matters here
- No self-evident comments (`// increment x`)
- Inline docs for tricky logic

---

## 6. MICROSOFT STACK SPECIFIC

Since you primarily develop for M365/Azure/Windows: details (Graph API auth, Azure deployment, Windows/ARM targeting, package management) are in `standards/microsoft-stack.md`, with Azure resource design in `standards/azure-integration.md`.

---

## 7. CODE REVIEW PROCESS

### Author responsibility
- You own your code quality
- Before creating a PR, check yourself:
  - Tests green?
  - Security checklist done?
  - Lint/format OK?
  - Diff understandable?

### Reviewer expectations (for your portfolio)
- Review checklist:
  - Logic correct?
  - Tests present & meaningful?
  - Security OK (no credentials, input validation, etc.)?
  - Code style consistent?
  - Performance OK (no N+1 queries, etc.)?
  - Documentation updated?

### Merge criteria
- Tests passing
- Security review OK
- Code style OK
- PR flow completed (solo: self-merge OK, no required approval; see GitHub Ruleset in section 3)
- No conflicts

### Risk-based merge policy (solo maintainer)

Since PR requirement is technically enforced (section 3) but no second reviewer exists, this escalation rule applies to the merge decision itself, regardless of which machine or session the work happens on:

**Low risk: merge yourself, then briefly inform:**
- Tests, documentation (README, CHANGELOG, ADRs)
- CI/CD configuration
- Dependency updates with green checks

Even for these low-risk cases: do not merge immediately after diff review. Wait for `gh pr checks` to show CI and Dependabot green first, then merge. Diff review alone does not catch build failures, formatting checks, or security-scan hits that CI surfaces. If the repo has no CI workflows at all (pure documentation repos), this wait naturally does not apply.

**Medium/high risk: create PR, show diff, wait for explicit OK:**
- Business logic
- Security-relevant changes
- Secrets/auth/permissions
- Breaking changes

This rule also applies to changes to this CLAUDE.md itself and to `ruleset-template.json`.

---

## 8. DOCUMENTATION

For public portfolio code: README structure, API doc requirements, ADR format and CHANGELOG format are in `standards/documentation.md`.

---

## 9. DEPLOYMENT & RELEASE

Pre-release checklist, release process and rollback plan are in `standards/release-process.md` sections 3 to 5.

---

## 10. PORTFOLIO TOOL SPECIFIC CHECKLIST

Before publishing a tool on GitHub: the full checklist (core features, security, quality, documentation, GitHub setup) is in `templates/portfolio-publish-checklist.md`.

---

## 11. LIVING STANDARDS (AUTO-UPDATES)

This CLAUDE.md is **living** and is automatically checked monthly for:
- New Microsoft security advisories
- GitHub security alerts (dependency vulnerabilities)
- OWASP Top 10 updates
- Best practice changes

**Automated process:**
- A check runs monthly
- If updates are needed: you receive a proposal with a diff
- You approve or reject
- CLAUDE.md is updated (only with your OK)

---

## 12. AI TRANSPARENCY & POSITIONING

**Principle:** Rafael positions himself as a modern AI-native engineer. AI usage is not hidden but backed by visible governance (this standards repo, rulesets, PR flow, CI, tests).

### Allowed and intended
- `Co-Authored-By: Claude ...` trailers in commits
- The "claude" account in GitHub contributors
- "AI | Claude Code" badges in READMEs (existing badge standard)

### Still binding
- Quality ownership visibly stays with the human: merge decisions, reviews and standards are Rafael's
- README footer names Rafael as author (standard footer)
- No AI marketing in PR bodies or READMEs ("Generated with ..." promo links): transparency yes, advertising no
- No history rewrite to retroactively scrub AI references (the no-force-push rule from section 3 applies)

---

**Version:** 2026-07-28
**Last auto-check:** Never (checked monthly)
**Applies to:** All portfolio projects, especially GitHub public repos
**Microsoft focus:** M365, Azure, Windows (x86/ARM)
