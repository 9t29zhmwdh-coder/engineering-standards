<div align="center">
  <img src="RayStudio.png" alt="RayStudio Logo" width="120"/>
  <h1>{{ProjectName}}</h1>
</div>

[🇩🇪 Deutsche Version](README.de.md)

**{{One or two sentence description: what does this do, in plain language.}}**

{{Optional second paragraph, expanding on the problem this solves.}}

[![CI](https://github.com/{{owner}}/{{repo}}/actions/workflows/ci.yml/badge.svg)](https://github.com/{{owner}}/{{repo}}/actions) ![Platform](https://img.shields.io/badge/Platform-{{platforms}}-lightgrey) {{additional badges: language, framework, AI tooling used}}

> **How it runs:** {{One sentence: native desktop app with no background service, or a local server at http://localhost:PORT, or a CLI tool, etc. Be explicit about what a user should expect after installing it.}}

![{{ProjectName}}](docs/screenshot.png)

---

{{Optional: a sentence noting UI language support, e.g. "This UI is available in English (default) and German; switch anytime with the language toggle."}}

**In practice:** {{Two to three plain-language sentences: what does the user actually end up with after using this tool, described concretely rather than as a feature list.}}

## Features

| Feature | Description |
|---|---|
| **{{Feature}}** | {{What it does}} |

{{Optional: a > **Note:** callout documenting any known limitation or non-functional setting, so the README never overstates what the tool currently does.}}

---

## Requirements

- {{Runtime/toolchain requirement 1}}
- {{Runtime/toolchain requirement 2}}
- {{Any external service credential required, and what it is needed for}}
- {{Supported operating systems}}

---

## Quick Start

```bash
git clone https://github.com/{{owner}}/{{repo}}
cd {{repo}}

{{exact copy-paste setup and run commands}}
```

### Usage

1. **{{Step}}**; {{what happens}}
2. **{{Step}}**; {{what happens}}

---

## Uninstall / Cleanup

- {{How to remove the app/binary}}
- {{Where local data is stored, and how to remove it}}
- {{Where any stored credential lives (Keychain, environment file), and how to remove it}}

{{A closing sentence confirming nothing else is left behind, if true.}}

---

## Architecture

{{Optional, for larger tools: a short paragraph or diagram reference. Link to an ADR for any decision with long-term consequences, per templates/architecture-decision-record.md.}}

---

**Author:** [{{Author}}](https://github.com/{{owner}}) · **Status:** Active · ![version](https://img.shields.io/github/v/release/{{owner}}/{{repo}}?color=6b7280&style=flat-square) · **License:** MIT
