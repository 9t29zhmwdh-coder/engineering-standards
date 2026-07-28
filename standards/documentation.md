# Documentation Standards

What every public repository in this portfolio documents, and in what form. The README structure itself is templated in [`../templates/readme-template.md`](../templates/readme-template.md); this document covers what belongs in each artifact.

## 1. README.md (Entry Point)

- What does this do, in one or two sentences
- Quick start that can be copy-pasted
- Features and, explicitly, limitations
- Short architecture overview
- How to contribute
- License

A README that lists features without limitations is a sales page, not documentation. State what the tool does not do, including platform and architecture constraints, in the requirements section rather than leaving it to a badge.

## 2. API Documentation

- Docstrings or comments on every public function
- Parameter types and return types
- Example usage
- Exceptions and error cases

## 3. Architecture Decision Record (ADR)

For larger tools, recorded from [`../templates/architecture-decision-record.md`](../templates/architecture-decision-record.md):

- Why this architecture
- Which alternatives were considered
- What the tradeoffs are
- Decision date

## 4. CHANGELOG.md

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), one section per release, newest first:

```
## [1.1.0] - 2026-07-03

### Added
- New MFA feature via Azure AD

### Fixed
- Security: fixed XSS vulnerability in form inputs

### Changed
- Improved performance of the user list query

### Security
- Updated dependencies (npm audit fix)
```

Write entries for the person reading them later, which usually means stating what changed for a user rather than which files moved. When an entry corrects an earlier release's claim, say so explicitly; a changelog that quietly supersedes itself is harder to trust than one that admits the correction.

## 5. Language

READMEs are bilingual: `README.md` in English as the default, `README.de.md` in German, cross-linked at the top. Everything else in `standards/` and `templates/` is English.

Solo-maintainer documents (`SECURITY.md`, `CODE_OF_CONDUCT.md`, `PRIVACY.md`) use the first person singular, never an institutional "we" that implies a team behind a single maintainer.
