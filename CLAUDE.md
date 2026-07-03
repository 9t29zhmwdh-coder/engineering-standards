# Claude Code Guidelines — Senior Software Engineer Standards
**Portfolio Edition** | Microsoft-fokussiert | Security-First | Public Code Quality

Diese Richtlinien gelten für alle Projekte, insbesondere für öffentliche Portfolio-Tools auf GitHub.

---

## 1. SENIOR ENGINEER CORE BEHAVIOR

### Niemals
- **Dateien kommentarlos löschen** — Wenn eine Datei gelöscht werden soll, begründe es zuerst und frage nach (es sei denn, es ist offensichtlich überflüssig wie temp-files)
- **Grosse Refactorings ohne Plan durchführen** — Vor grösseren Umstrukturierungen wird zuerst ein Plan erstellt, der die Auswirkungen klärt
- **Änderungen ohne Diff präsentieren** — Vor dem Commit wird der Diff immer gezeigt und erklärt
- **Secrets in Code committen** — Nie. Passwords, API Keys, Tokens gehören in Secret Manager (Azure Key Vault, 1Password)

### Immer
- **Plan erstellen** — Bei nicht-trivialen Änderungen (>1 Datei oder komplexe Logik)
- **Auswirkungen analysieren** — Betroffene Dateien, Dependencies, Seiteneffekte
- **Git-Diff kontrollieren** — Vor dem Commit zeigen und erklären
- **Tests ausführen** — Falls Tests existieren, vor Commit laufen lassen
- **Zusammenfassung liefern** — Was, Warum, Nebenwirkungen
- **Atomic Commits** — Ein Commit = eine logische Änderung
- **Semantic Commit Messages** — `type(scope): description` (feat, fix, security, refactor, test, docs)

---

## 2. SECURITY-FIRST MINDSET (NICHT OPTIONAL)

**Dies ist der grösste Unterschied zu Standard-Code.** Für öffentliche Portfolio-Tools MUSS Security von Tag 1 eingebaut sein.

### Security Features — Vor der ersten Zeile Code
- ✅ **Authentication & Authorization**
  - MFA (Multi-Factor Authentication) — nicht optional, ab Tag 1
  - RBAC oder ABAC (Role/Attribute-Based Access Control)
  - Least Privilege Principle
  - Session Management mit Timeout & Rotation

- ✅ **Input Validation & Sanitization**
  - Whitelist > Blacklist
  - Validate at Boundaries (User Input, APIs, DB)
  - Never Trust Untrusted Input

- ✅ **Secrets Management**
  - Keine hardcoded Passwords/Keys in Code
  - Azure Key Vault für Production Secrets
  - Environment Variables für Development (mit .env.example, niemals .env committen)

- ✅ **Intrusion Detection & Audit Logging**
  - Rate Limiting (Login, API Endpoints)
  - Failed Login Tracking & Lockout
  - Audit Logs für sensitive Operations (Who, What, When, Why)
  - Anomaly Detection (Unusual Access Patterns)

- ✅ **Encryption**
  - TLS/HTTPS in Transit (Standard)
  - AES-256 für sensitive Data at Rest
  - Use Industry-Standard Algorithms (SHA-256, PBKDF2 für Passwords)

- ✅ **Secure Error Handling**
  - Keine Stack Traces in Client Responses
  - Keine DB Errors zu Users leaken
  - Generic Error Messages zu Clients, Detailed Logs Internally

- ✅ **Dependency Security**
  - npm audit, pip check, dotnet outdated vor jedem Release
  - Lock Files (package-lock.json, requirements.txt)
  - Kein Arbitrary Upgrades in Production

### Security Checklist (vor jedem Release)
- [ ] MFA implementiert?
- [ ] Alle User Inputs validiert?
- [ ] Secrets nicht in Code?
- [ ] Audit Logging vorhanden?
- [ ] Error Messages sicher (keine Leaks)?
- [ ] Dependencies aktuell & geprüft?
- [ ] OWASP Top 10 gecheckt? (Injection, Auth, Sensitive Data, XML, Broken Auth, etc.)
- [ ] Penetration Test durchgeführt? (zumindest manual Security Review)

---

## 3. VERSIONING & ROLLBACK STRATEGY

**Das ist kritisch für dein Portfolio.** Wenn Code verhunzt wird, braucht du einen sauberen Rollback.

### Git Workflow
- **Main/Master:** IMMER Production-Ready, IMMER Stable
- **feature/xxx:** Feature Branches für Development
- **Branch Protection:** 
  - No Direct Push to Main (erzwingen!)
  - No Force Push to Main (ever!)
  - PR-Pflicht vor Merge (Solo-Maintainer: kein Required-Approval nötig, aber PR-Flow ist Pflicht)

### GitHub Ruleset (technisch erzwungen, nicht nur Richtlinie)

Jedes Public Repo bekommt bei Erstellung das Ruleset `solo-main-protection` auf dem Default-Branch:
- `deletion` — Branch-Löschung blockiert
- `non_fast_forward` — Force-Push blockiert
- `pull_request` — PR vor Merge erforderlich, `required_approving_review_count: 0` (Solo-Workflow, kein Self-Approval-Deadlock)
- Kein `bypass_actor` gesetzt → gilt auch für den Owner selbst

**Setup für neue Repos:**
```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset.json
```
Ruleset-Template liegt in diesem Repo: `ruleset-template.json`

**Monatlicher Auto-Check:** Prüft, ob alle Public Repos dieses Ruleset aktiv haben (`gh api repos/<owner>/<repo>/rulesets`). Fehlt es, wird es im Audit-PR nachgetragen.

### Semantic Versioning (MAJOR.MINOR.PATCH)
- **MAJOR:** Breaking Changes (z.B. API Breaking Change)
- **MINOR:** New Features (backward compatible)
- **PATCH:** Bug Fixes nur

**Example:**
```
v1.0.0 → v1.1.0 (new feature)
v1.1.0 → v1.1.1 (bug fix)
v1.x.x → v2.0.0 (breaking change)
```

### Release Process
1. Update Version in package.json/setup.py/csproj
2. Update CHANGELOG.md (Was hat sich geändert für Users?)
3. Create Release Tag: `git tag v1.0.0`
4. Push to GitHub with Release Notes
5. Never Amend/Rebase published Commits

### Rollback Capability
- Jede Version hat einen Git Tag
- Rollback = `git checkout v1.0.0` (alt) oder `git reset --hard <commit-hash>`
- CHANGELOG dokumentiert Breaking Changes

---

## 4. TESTING STRATEGY

### Test Pyramid
```
      E2E (Critical Flows Only)
    Integration Tests (APIs, DB)
  Unit Tests (Business Logic) ← MOST
```

### Minimum Standards
- **Unit Tests:** Business Logic, Utilities (sollten <1s laufen, kein I/O)
- **Integration Tests:** APIs, Database Interactions (gegen Test DB)
- **E2E Tests:** NUR für kritische User Flows (zu langsam für alles)
- **Coverage Target:** ~80% (nicht 100%, aber auch nicht <50%)

### Before Every Commit
```bash
npm test          # oder pytest, dotnet test
npm run lint      # oder black, eslint
npm run build     # oder python setup.py build
```

### What NOT to Test
- Triviale Getter/Setter
- Framework-provided Code
- Externe APIs (mock stattdessen)

---

## 5. CODE QUALITY & ARCHITECTURE

### SOLID Principles (Microsoft Standard)
- **S**ingle Responsibility: Eine Klasse = eine Reason to Change
- **O**pen/Closed: Offen für Extension, Geschlossen für Modification
- **L**iskov Substitution: Subclasses sollten Superclasses ersetzen können
- **I**nterface Segregation: Clients sollten nicht Methoden implementieren, die sie nicht brauchen
- **D**ependency Inversion: Depend on Abstractions, not Concrete Implementations

### DRY, KISS, YAGNI
- **DRY (Don't Repeat Yourself):** 3x Copy = abstrahieren, 2x = consider, 1x = ok
- **KISS (Keep It Simple, Stupid):** Simplicity > Cleverness
- **YAGNI (You Aren't Gonna Need It):** Keine Features für hypothetische Zukunft

### Naming Conventions
- **Functions:** Verb + Noun (`getUserById`, `validateEmail`, `processPayment`)
- **Variables:** Clear Intent (nicht `x`, `temp`, `data`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_RETRIES`, `DB_TIMEOUT`)
- **Classes:** PascalCase (`UserService`, `AuthController`)

### Function/Method Size
- **Max 20 Zeilen** pro Funktion (ideal <10)
- Wenn >300 Zeilen Klasse → split in mehrere

### Comments
- **Nur WHY, nicht WHAT** — `x++` ist klar, aber warum es hier wichtig ist, erklären
- Keine Self-Evident Comments (`// increment x`)
- Inline-Docs für Tricky Logic

---

## 6. MICROSOFT STACK SPECIFIC

Da du primär für M365/Azure/Windows entwickelst:

### M365 Authentication (Graph API)
- Verwende Microsoft Graph SDK (nicht eigene Auth bauen)
- OAuth 2.0 mit Azure AD (nicht Basic Auth!)
- MFA via Azure AD
- Token Refresh Strategy

### Azure Deployment
- Infrastructure as Code (ARM Templates, Bicep, oder Terraform)
- Use Azure Key Vault für Secrets
- Use Managed Identity wenn möglich (nicht Service Principals mit Secrets)
- Staging vor Production Deployment

### Windows/ARM Targeting
- Build & Test für beide x86 und ARM
- Visual Studio für Desktop Apps, VS Code für Cloud/Web
- Use .NET 6+ (LTS) für Long-Term Support
- PowerShell 7+ (cross-platform)

### Package Management
- NuGet für .NET
- npm für Node.js
- pip für Python
- GitHub Packages für Private Packages

---

## 7. CODE REVIEW PROCESS

### Author Responsibility
- Du ownest dein Code Quality
- Bevor du einen PR erstellen, selbst überprüfen:
  - Tests grün?
  - Security Checklist abgehakt?
  - Lint/Format OK?
  - Diff verständlich?

### Reviewer Expectations (für dein Portfolio)
- Review Checklist:
  - Logic korrekt?
  - Tests vorhanden & aussagekräftig?
  - Security OK (keine Credentials, Input Validation, etc.)?
  - Code Style Consistent?
  - Performance OK (keine N+1 Queries, etc.)?
  - Documentation updated?

### Merge Criteria
- Tests passing
- Security Review OK
- Code Style OK
- PR-Flow durchlaufen (Solo: Self-Merge OK, kein Required-Approval — siehe GitHub Ruleset in Abschnitt 3)
- No Conflicts

### Risikobasierte Merge-Policy (Solo-Maintainer)

Da PR-Pflicht technisch erzwungen ist (Abschnitt 3), aber kein zweiter Reviewer existiert, gilt für die Merge-Entscheidung selbst diese Eskalationsregel — unabhängig davon, auf welcher Maschine oder in welcher Session gearbeitet wird:

**Niedriges Risiko → selbst mergen, danach kurz informieren:**
- Tests, Dokumentation (README, CHANGELOG, ADRs)
- CI/CD-Konfiguration
- Dependency-Updates mit grünen Checks

**Mittleres/hohes Risiko → PR erstellen, Diff zeigen, auf explizites OK warten:**
- Business-Logik
- Security-relevante Änderungen
- Secrets/Auth/Permissions
- Breaking Changes

Diese Regel gilt auch für Änderungen an dieser CLAUDE.md selbst sowie am `ruleset-template.json`.

---

## 8. DOCUMENTATION

Für öffentliches Portfolio Code:

### README.md (Entry Point)
- What does this do? (1-2 Sätze)
- Quick Start (Copy-Paste Setup)
- Features & Limitations
- Architecture Overview (kurz)
- How to Contribute
- License

### API Documentation
- Docstrings/Comments für jede Public Function
- Parameter Types & Return Types
- Example Usage
- Exceptions/Error Cases

### Architecture Decision Record (ADR)
Für grössere Tools:
- Warum diese Architektur?
- Welche Alternativen wurden betrachtet?
- Tradeoffs?
- Decision Date

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

### Pre-Release Checklist
- [ ] Version updated (MAJOR.MINOR.PATCH)
- [ ] CHANGELOG updated
- [ ] All Tests Passing
- [ ] Security Checklist completed
- [ ] Dependencies audited
- [ ] Documentation updated
- [ ] Code Review approved
- [ ] Build artifacts tested

### Release Process
1. Create Release Tag: `git tag v1.0.0`
2. Push Tag: `git push origin v1.0.0`
3. Create GitHub Release with CHANGELOG
4. Deploy to Production (if applicable)
5. Monitor for Errors

### Rollback Plan
- Keep Previous Version Available
- Document Rollback Steps (script or manual)
- Test Rollback in Staging First

---

## 10. PORTFOLIO TOOL SPECIFIC CHECKLIST

Bevor du ein Tool auf GitHub publishest:

### Core Features
- [ ] Main Feature funktioniert & getestet
- [ ] Error Handling robust
- [ ] Logging für Debugging
- [ ] Help/Usage dokumentiert

### Security
- [ ] Security Checklist abgehakt
- [ ] No Secrets in Repo
- [ ] No Credentials in .env
- [ ] Input Validation implemented
- [ ] OWASP Top 10 gecheckt

### Quality
- [ ] Tests grün (Unit + Integration)
- [ ] Linting & Format OK
- [ ] No Dead Code
- [ ] Meaningful Commit Messages

### Documentation
- [ ] README.md vollständig
- [ ] API Docs vorhanden
- [ ] CHANGELOG.md aktuell
- [ ] License included (MIT, Apache, etc.)

### GitHub
- [ ] README rendering OK
- [ ] .gitignore vorhanden (node_modules, .env, etc.)
- [ ] LICENSE file vorhanden
- [ ] GitHub Actions für CI/CD (optional aber empfohlen)

---

## 11. LIVING STANDARDS (AUTO-UPDATES)

Diese CLAUDE.md ist **lebendig** und wird monatlich automatisch geprüft auf:
- Neue Microsoft Security Advisories
- GitHub Security Alerts (Dependency Vulnerabilities)
- OWASP Top 10 Updates
- Best Practices Changes

**Automatischer Prozess:**
- Monatlich läuft ein Check
- Falls Updates nötig: Du erhältst einen Vorschlag mit Diff
- Du genehmigst oder lehnst ab
- CLAUDE.md wird aktualisiert (nur mit deinem OK)

---

**Version:** 2026-07-03  
**Last Auto-Check:** Nie (wird monatlich geprüft)  
**Gültig für:** Alle Portfolio-Projekte unter C:\Users\RafaelYilmaz, besonders für GitHub Public Repos  
**Microsoft Focus:** M365, Azure, Windows (x86/ARM)
