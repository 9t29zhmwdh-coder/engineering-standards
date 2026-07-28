# Claude Code Guidelines: Senior Software Engineer Standards
**Portfolio Edition** | Microsoft-fokussiert | Security-First | Public Code Quality

Diese Richtlinien gelten für alle Projekte, insbesondere für öffentliche Portfolio-Tools auf GitHub.

---

## 1. SENIOR ENGINEER CORE BEHAVIOR

### Niemals
- **Dateien kommentarlos löschen**: Wenn eine Datei gelöscht werden soll, begründe es zuerst und frage nach (es sei denn, es ist offensichtlich überflüssig wie temp-files)
- **Grosse Refactorings ohne Plan durchführen**: Vor grösseren Umstrukturierungen wird zuerst ein Plan erstellt, der die Auswirkungen klärt
- **Änderungen ohne Diff präsentieren**: Vor dem Commit wird der Diff immer gezeigt und erklärt
- **Secrets in Code committen**: Nie. Passwords, API Keys, Tokens gehören in Secret Manager (Azure Key Vault, 1Password)

### Immer
- **Plan erstellen**: Bei nicht-trivialen Änderungen (>1 Datei oder komplexe Logik)
- **Auswirkungen analysieren**: Betroffene Dateien, Dependencies, Seiteneffekte
- **Git-Diff kontrollieren**: Vor dem Commit zeigen und erklären
- **Tests ausführen**: Falls Tests existieren, vor Commit laufen lassen
- **Zusammenfassung liefern**: Was, Warum, Nebenwirkungen
- **Atomic Commits**: Ein Commit = eine logische Änderung
- **Semantic Commit Messages**: `type(scope): description` (feat, fix, security, refactor, test, docs)

---

## 2. SECURITY-FIRST MINDSET (NICHT OPTIONAL)

**Dies ist der grösste Unterschied zu Standard-Code.** Für öffentliche Portfolio-Tools MUSS Security von Tag 1 eingebaut sein.

### Security-Standard: kanonische Quelle

Der vollständige, verbindliche Security-Standard lebt in diesem Repo unter `standards/security.md`, nicht als separate Liste hier dupliziert, um Drift zwischen zwei Kopien zu vermeiden:

- `standards/security.md`, 12 Abschnitte: STRIDE-Threat-Modeling, Zero Trust Principles, Secure Defaults, Identity & Access (Entra ID, MFA, RBAC/ABAC, Managed Identity), Secrets Management, Persönliche & Drittanbieter-Informationen (oberste Priorität, seit 2026-07-11), Input Validation & Sanitization, Encryption, Secure Error Handling, Dependency Governance & SBOM (CycloneDX), Audit Logging, Release Gate
- `templates/security-checklist.md`, die abzuhakende Checkliste pro Release, inklusive Sign-off (Reviewer, Datum, Version)

Vor jedem Release: `templates/security-checklist.md` kopieren und abarbeiten, nicht aus dem Gedächtnis rekonstruieren.

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

Jedes Public Repo bekommt bei Erstellung das Ruleset `solo-main-protection` auf dem Default-Branch. Setup-Befehle, Template-Pfad, monatlicher Auto-Check und die Portfolio-weite Baseline (inkl. bekannter CodeQL-Einschränkung) siehe `standards/release-process.md` Abschnitt 1.

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

### Release Process & Rollback Capability
Schritt-für-Schritt-Ablauf, Versionierungsdisziplin (jede gemergte Änderung wird versioniert, inklusive Tag und Release) und Rollback-Vorgehen siehe `standards/release-process.md`.

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
Standard SOLID-Prinzipien, konsequent durchsetzen.

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
- **Nur WHY, nicht WHAT**: `x++` ist klar, aber warum es hier wichtig ist, erklären
- Keine Self-Evident Comments (`// increment x`)
- Inline-Docs für Tricky Logic

---

## 6. MICROSOFT STACK SPECIFIC

Da du primär für M365/Azure/Windows entwickelst: Details (Graph API Auth, Azure Deployment, Windows/ARM Targeting, Package Management) siehe `standards/microsoft-stack.md`, für Azure-Ressourcen zusätzlich `standards/azure-integration.md`.

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
- PR-Flow durchlaufen (Solo: Self-Merge OK, kein Required-Approval, siehe GitHub Ruleset in Abschnitt 3)
- No Conflicts

### Risikobasierte Merge-Policy (Solo-Maintainer)

Da PR-Pflicht technisch erzwungen ist (Abschnitt 3), aber kein zweiter Reviewer existiert, gilt für die Merge-Entscheidung selbst diese Eskalationsregel, unabhängig davon, auf welcher Maschine oder in welcher Session gearbeitet wird:

**Niedriges Risiko → selbst mergen, danach kurz informieren:**
- Tests, Dokumentation (README, CHANGELOG, ADRs)
- CI/CD-Konfiguration
- Dependency-Updates mit grünen Checks

Auch bei diesen Low-Risk-Fällen: nicht sofort nach dem Diff-Review mergen. Erst `gh pr checks` abwarten, bis CI und Dependabot grün stehen, dann mergen. Diff-Review allein findet keine Build-Fehler, Formatierungs-Checks oder Security-Scan-Treffer, die CI aufdeckt. Hat das Repo gar keine CI-Workflows (reine Dokumentations-Repos), entfällt das Warten naturgemäss.

**Mittleres/hohes Risiko → PR erstellen, Diff zeigen, auf explizites OK warten:**
- Business-Logik
- Security-relevante Änderungen
- Secrets/Auth/Permissions
- Breaking Changes

Diese Regel gilt auch für Änderungen an dieser CLAUDE.md selbst sowie am `ruleset-template.json`.

---

## 8. DOCUMENTATION

Für öffentliches Portfolio Code: README-Struktur, API-Doc-Vorgaben, ADR-Format und CHANGELOG-Format siehe `standards/documentation.md`.

---

## 9. DEPLOYMENT & RELEASE

Pre-Release-Checkliste, Release-Prozess und Rollback-Plan siehe `standards/release-process.md` Abschnitte 3 bis 5.

---

## 10. PORTFOLIO TOOL SPECIFIC CHECKLIST

Bevor du ein Tool auf GitHub publishest: vollständige Checkliste (Core Features, Security, Quality, Documentation, GitHub-Setup) siehe `templates/portfolio-publish-checklist.md`.

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

## 12. AI-TRANSPARENZ & POSITIONIERUNG

**Grundsatz:** Rafael positioniert sich als moderner AI-nativer Engineer. KI-Einsatz wird nicht versteckt, sondern durch sichtbare Governance untermauert (dieses Standards-Repo, Rulesets, PR-Flow, CI, Tests).

### Erlaubt und gewollt
- `Co-Authored-By: Claude ...`-Trailer in Commits
- Der "claude"-Account in GitHub-Contributors
- "AI | Claude Code"-Badges in READMEs (bestehender Badge-Standard)

### Weiterhin verbindlich
- Qualitätsverantwortung liegt sichtbar beim Menschen: Merge-Entscheidungen, Reviews und Standards trägt Rafael
- README-Footer nennt Rafael als Author (Standard-Footer)
- Kein KI-Marketing in PR-Bodies oder READMEs ("Generated with ..."-Werbelinks): Transparenz ja, Werbung nein
- Kein History-Rewrite zur nachträglichen Bereinigung von KI-Referenzen (No-Force-Push-Regel aus Abschnitt 3 gilt)

---

**Version:** 2026-07-28  
**Last Auto-Check:** Nie (wird monatlich geprüft)  
**Gültig für:** Alle Portfolio-Projekte unter C:\Users\RafaelYilmaz, besonders für GitHub Public Repos  
**Microsoft Focus:** M365, Azure, Windows (x86/ARM)
