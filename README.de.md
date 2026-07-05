<div align="center"><img src="RayStudio.png" alt="RayStudio Logo" width="120"/><h1>engineering-standards</h1></div>

> 🇬🇧 [English Version](README.md)

Engineering-Standards und Best Practices für alle Portfolio-Tools: Microsoft-fokussiert, Security-First, Senior Engineer Standards.

## Was ist das?

Dieses Repo ist die einzige versionierte Quelle dafür, wie Code über alle Portfolio-Repositories hinweg geschrieben, reviewt, gemergt und released wird. Jede Maschine (Windows, Mac) verlinkt ihre globalen Claude-Code-Instruktionen auf die `CLAUDE.md` in diesem Repo, damit überall dieselben Regeln gelten.

## Inhalt

| Datei | Zweck |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | Die Standards (Deutsch, normativ). Auf jeder Maschine als globale Claude-Code-Instruktionen verlinkt. |
| [`CLAUDE.en.md`](CLAUDE.en.md) | Englische Referenz-Übersetzung der Standards. |
| [`ruleset-template.json`](ruleset-template.json) | GitHub-Ruleset `solo-main-protection`: blockiert Branch-Löschung und Force-Push, erzwingt PR-Flow auf dem Default-Branch. |

## Quick Start (neue Maschine)

```bash
git clone https://github.com/9t29zhmwdh-coder/engineering-standards.git ~/engineering-standards
ln -sf ~/engineering-standards/CLAUDE.md ~/.claude/CLAUDE.md
```

## Ruleset auf ein neues Repo anwenden

```bash
gh api repos/<owner>/<repo>/rulesets -X POST --input ruleset-template.json
```

## Kernprinzipien

- **Security-First:** MFA, Input-Validierung, Secrets-Management, Audit-Logging und Dependency-Security sind ab Tag 1 eingebaut, nicht nachträglich angeflanscht.
- **Erzwungene Branch-Protection:** Kein Direct- oder Force-Push auf `main`, PR-Flow ist Pflicht, technisch erzwungen via GitHub-Rulesets (auch für den Owner).
- **Risikobasierte Merge-Policy:** Änderungen mit niedrigem Risiko (Docs, Tests, CI) werden selbst gemergt und gemeldet; mittleres/hohes Risiko (Business-Logik, Security, Auth) wartet auf ein explizites OK.
- **Living Standards:** Monatlich gegen Microsoft Security Advisories, OWASP-Updates und Best-Practice-Änderungen geprüft; aktualisiert nur mit expliziter Freigabe.

---

**Autor:** [Rafael Yilmaz](https://github.com/9t29zhmwdh-coder) · **Status:** Active
