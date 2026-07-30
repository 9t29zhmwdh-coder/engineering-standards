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

### The first screen

Somebody arriving from a search result or a link decides in seconds whether to
keep reading. They are not going to scroll past a feature list to find out
whether the tool is for them. Everything above the fold answers three questions,
in this order:

**What is it for, in a sentence anyone can read.** Not what it is, what it does
for the reader. "Lightweight modular Swift agent framework for local LLMs"
names the parts; "lets a local model do things, not just answer" names the
point. Category nouns like framework, toolkit and platform describe the shape of
the thing and carry no information about its purpose.

**Who it is not for.** The exclusion is worth more than the invitation, because
it is the sentence that stops the wrong reader from spending ten minutes finding
out. "If you only need a single answer, call the API directly" costs one line
and saves that reader the whole page.

**What using it looks like.** The shortest real example, immediately: a few
lines of code, a screenshot, one command. Not after the features, before them.
Whoever is still reading wants to see the thing, not a description of it.

A feature list belongs underneath all three. It is a reference for somebody who
has already decided, and it persuades nobody who has not.

The test: cover everything below the first screenful and ask whether a stranger
could say what the tool is for and whether they want it. If not, the opening is
describing the parts instead of the point, however complete the rest of the
document is.

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
