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

### Security features: before the first line of code
- ✅ **Authentication & Authorization**
  - MFA (Multi-Factor Authentication): not optional, from day 1
  - RBAC or ABAC (Role/Attribute-Based Access Control)
  - Least Privilege Principle
  - Session management with timeout & rotation

- ✅ **Input Validation & Sanitization**
  - Whitelist > Blacklist
  - Validate at boundaries (user input, APIs, DB)
  - Never trust untrusted input

- ✅ **Secrets Management**
  - No hardcoded passwords/keys in code
  - Azure Key Vault for production secrets
  - Environment variables for development (with .env.example, never commit .env)

- ✅ **Personal & Third-Party Information** (top priority, since 2026-07-11)
  - No repository (public or private) contains real names, hostnames, IP addresses, or other identifying details of a third party (employer, client, colleague), unless that party has explicitly agreed to the reference
  - Metadata fields that commonly carry this unnoticed (`Company`/`Publisher`/`Author` in `.csproj`, `Info.plist`, `package.json`, Cargo `authors`, installer scripts) are checked before first publish and on every release
  - A tool originally built in the context of employment is reviewed for IP ownership before being published as a personal project; when in doubt, the employer's moonlighting/IP policy governs, not this document
  - Example configuration, screenshots, and demo data use synthetic values, never real internal hostnames, real customer names, or real production data

- ✅ **Intrusion Detection & Audit Logging**
  - Rate limiting (login, API endpoints)
  - Failed login tracking & lockout
  - Audit logs for sensitive operations (who, what, when, why)
  - Anomaly detection (unusual access patterns)

- ✅ **Encryption**
  - TLS/HTTPS in transit (standard)
  - AES-256 for sensitive data at rest
  - Use industry-standard algorithms (SHA-256, PBKDF2 for passwords)

- ✅ **Secure Error Handling**
  - No stack traces in client responses
  - No DB errors leaked to users
  - Generic error messages to clients, detailed logs internally

- ✅ **Dependency Security**
  - npm audit, pip check, dotnet outdated before every release
  - Lock files (package-lock.json, requirements.txt)
  - No arbitrary upgrades in production

### Security checklist (before every release)
- [ ] MFA implemented?
- [ ] All user inputs validated?
- [ ] No secrets in code?
- [ ] No employer/client references in code, metadata, or docs?
- [ ] Audit logging in place?
- [ ] Error messages safe (no leaks)?
- [ ] Dependencies up to date & audited?
- [ ] OWASP Top 10 checked? (Injection, Auth, Sensitive Data, XML, Broken Auth, etc.)
- [ ] Penetration test performed? (at least a manual security review)

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

Every public repo gets the ruleset `solo-main-protection` on the default branch at creation:
- `deletion`: branch deletion blocked
- `non_fast_forward`: force push blocked
- `pull_request`: PR required before merge, `required_approving_review_count: 0` (solo workflow, no self-approval deadlock)
- No `bypass_actor` set, so it also applies to the owner

**Setup for new repos:**
```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset.json
```
The ruleset template lives in this repo: `ruleset-template.json`

**Monthly auto-check:** Verifies that all public repos have this ruleset active (`gh api repos/<owner>/<repo>/rulesets`). If missing, it is added via the audit PR.

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

### Release process
1. Update version in package.json/setup.py/csproj
2. Update CHANGELOG.md (what changed for users?)
3. Create release tag: `git tag v1.0.0`
4. Push to GitHub with release notes
5. Never amend/rebase published commits

### Rollback capability
- Every version has a git tag
- Rollback = `git checkout v1.0.0` (old) or `git reset --hard <commit-hash>`
- CHANGELOG documents breaking changes

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
- **S**ingle Responsibility: one class = one reason to change
- **O**pen/Closed: open for extension, closed for modification
- **L**iskov Substitution: subclasses should be able to replace superclasses
- **I**nterface Segregation: clients should not implement methods they do not need
- **D**ependency Inversion: depend on abstractions, not concrete implementations

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

Since you primarily develop for M365/Azure/Windows:

### M365 authentication (Graph API)
- Use the Microsoft Graph SDK (do not build your own auth)
- OAuth 2.0 with Azure AD (not Basic Auth!)
- MFA via Azure AD
- Token refresh strategy

### Azure deployment
- Infrastructure as Code (ARM templates, Bicep, or Terraform)
- Use Azure Key Vault for secrets
- Use Managed Identity when possible (not service principals with secrets)
- Staging before production deployment

### Windows/ARM targeting
- Build & test for both x86 and ARM
- Visual Studio for desktop apps, VS Code for cloud/web
- Use .NET 6+ (LTS) for long-term support
- PowerShell 7+ (cross-platform)

### Package management
- NuGet for .NET
- npm for Node.js
- pip for Python
- GitHub Packages for private packages

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

For public portfolio code:

### README.md (entry point)
- What does this do? (1-2 sentences)
- Quick start (copy-paste setup)
- Features & limitations
- Architecture overview (brief)
- How to contribute
- License

### API documentation
- Docstrings/comments for every public function
- Parameter types & return types
- Example usage
- Exceptions/error cases

### Architecture Decision Record (ADR)
For larger tools:
- Why this architecture?
- Which alternatives were considered?
- Tradeoffs?
- Decision date

### CHANGELOG.md
```
## [1.1.0] - 2026-07-03
### Added
- New MFA feature via Azure AD

### Fixed
- Security: Fixed XSS vulnerability in form inputs

### Changed
- Improved performance of user list query

### Security
- Updated dependencies (npm audit fix)
```

---

## 9. DEPLOYMENT & RELEASE

### Pre-release checklist
- [ ] Version updated (MAJOR.MINOR.PATCH)
- [ ] CHANGELOG updated
- [ ] All tests passing
- [ ] Security checklist completed
- [ ] Dependencies audited
- [ ] Documentation updated
- [ ] Code review approved
- [ ] Build artifacts tested

### Release process
1. Create release tag: `git tag v1.0.0`
2. Push tag: `git push origin v1.0.0`
3. Create GitHub release with CHANGELOG
4. Deploy to production (if applicable)
5. Monitor for errors

### Rollback plan
- Keep the previous version available
- Document rollback steps (script or manual)
- Test rollback in staging first

---

## 10. PORTFOLIO TOOL SPECIFIC CHECKLIST

Before publishing a tool on GitHub:

### Core features
- [ ] Main feature works & is tested
- [ ] Error handling robust
- [ ] Logging for debugging
- [ ] Help/usage documented

### Security
- [ ] Security checklist done
- [ ] No secrets in the repo
- [ ] No credentials in .env
- [ ] Input validation implemented
- [ ] OWASP Top 10 checked

### Quality
- [ ] Tests green (unit + integration)
- [ ] Linting & format OK
- [ ] No dead code
- [ ] Meaningful commit messages

### Documentation
- [ ] README.md complete
- [ ] API docs present
- [ ] CHANGELOG.md up to date
- [ ] License included (MIT, Apache, etc.)

### GitHub
- [ ] README rendering OK
- [ ] .gitignore present (node_modules, .env, etc.)
- [ ] LICENSE file present
- [ ] GitHub Actions for CI/CD (optional but recommended)

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

**Version:** 2026-07-05
**Last auto-check:** Never (checked monthly)
**Applies to:** All portfolio projects, especially GitHub public repos
**Microsoft focus:** M365, Azure, Windows (x86/ARM)
