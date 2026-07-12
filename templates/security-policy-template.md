# Security Policy

## Reporting a Vulnerability

**Do NOT open a public GitHub issue for security vulnerabilities.**

Instead, report it via [GitHub Security Advisory](https://github.com/9t29zhmwdh-coder/{{repo}}/security/advisories/new) or contact the maintainer via the GitHub profile.

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

A response within **48 hours** is the target, and the issue will be worked on promptly.

## Supply Chain Security

- All GitHub Actions used in the CI pipeline are pinned to a specific commit SHA, not a mutable tag or branch (see `standards/ci-cd.md` section 2 in `engineering-standards`).
- Dependencies are managed via {{lock file, e.g. Cargo.lock / package-lock.json / poetry.lock}}, which is committed to the repository for reproducible builds.
- {{dependency audit tool, e.g. cargo audit / npm audit / pip-audit}} runs in CI on every pull request.

## Supported Versions

| Version | Supported |
|---------|-----------|
| Latest  | ✅ Yes    |
| Older   | ❌ No     |

Security fixes are only applied to the latest release.

---
Usage note (not part of the published document): every bullet above must be
verified true against the repository's actual state before publishing, not
assumed or copy-pasted aspirationally. A false claim in a security policy is
a worse outcome than a shorter, accurate one. See
`templates/new-repo-bootstrap-checklist.md` for when this template is applied
during a new repository's setup.
