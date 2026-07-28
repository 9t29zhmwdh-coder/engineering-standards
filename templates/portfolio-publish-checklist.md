# Portfolio Publish Checklist

Worked through once, immediately before a tool is first made public on GitHub. This is the pre-publish gate; the per-release gate is the pre-release checklist in [`../standards/release-process.md`](../standards/release-process.md) section 4, and the security gate is [`security-checklist.md`](security-checklist.md).

Copy this file into the repository being published and tick it there, rather than reconstructing it from memory.

## Core Features

- [ ] Main feature works and is tested
- [ ] Error handling is robust
- [ ] Logging sufficient to debug a user's report without reproducing locally
- [ ] Help and usage documented

## Security

- [ ] [`security-checklist.md`](security-checklist.md) completed and signed off
- [ ] No secrets in the repository, including in history
- [ ] No credentials in `.env` or committed configuration
- [ ] Input validation implemented
- [ ] OWASP Top 10 reviewed against this tool's actual attack surface
- [ ] No personal or third-party identifying information in code, metadata or documentation

## Quality

- [ ] Tests green (unit and integration)
- [ ] Linting and formatting clean, with the tooling pinned to exact versions
- [ ] No dead code
- [ ] Commit messages meaningful

## Documentation

- [ ] `README.md` complete, per [`../standards/documentation.md`](../standards/documentation.md)
- [ ] `README.de.md` present and consistent with the English version
- [ ] Requirements state platform and architecture constraints explicitly
- [ ] API documentation present
- [ ] `CHANGELOG.md` current
- [ ] `LICENSE` included

## GitHub

- [ ] README renders correctly on GitHub, badges included
- [ ] `.gitignore` present and covering the build output of this stack
- [ ] `LICENSE` file present
- [ ] CI workflows in place and green
- [ ] `solo-main-protection` ruleset applied, per [`../standards/release-process.md`](../standards/release-process.md) section 1
- [ ] Repository description and topics set, so the tool is findable and its purpose readable without opening the README
