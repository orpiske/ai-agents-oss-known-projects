# AI Agents OSS Helper — Known Projects

Centralized project configuration rules consumed by [ai-agents-oss-helper](https://github.com/Open-Harness-Engineering/ai-agents-oss-helper) commands.

## Purpose

This repository is an opt-in alternative to shipping `.oss-ai-helper-rules/` inside each project. Projects that prefer not to host AI-agent metadata in their own source tree (or that cannot ship dot-directories for policy reasons) can have their rules hosted here instead. Users install the rules locally with the `/oss-install-info` command from the helper.

## Layout

Each project has its own top-level directory containing the three rule files the helper expects, plus an optional fourth file for the security workflow:

```
ai-agents-oss-known-projects/
├── <project>/
│   ├── project-info.md          # Repository URLs, issue trackers, related repos
│   ├── project-standards.md     # Build tools, commands, code style restrictions
│   ├── project-guidelines.md    # Branch naming, commit formats, PR policies
│   └── project-security.md      # (optional) CVE-handling & publishing workflow
├── generic-github/              # Catch-all fallback for any GitHub project
│   ├── project-info.md
│   ├── project-standards.md
│   └── project-guidelines.md
└── workflows/                   # Cross-command process playbooks
    └── cve-workflow.md          # End-to-end CVE handling by chaining commands
```

The `<project>` directory name is the slug the helper uses internally (for example `camel-core` for `apache/camel`).

`project-security.md` is **optional** and consumed only by the security commands (`/oss-triage-security-report`, `/oss-create-security-advisory`, `/oss-draft-cve`, `/oss-analyze-third-party-cve`). It records the private reporting channel, the CVE Numbering Authority (CNA), the advisory format and publication location, the signing/disclosure steps, and the supported release lines a fix must be backported to. When the file is absent, those commands fall back to gathering the same information interactively.

## Workflows

The `workflows/` directory holds cross-command **process playbooks** — ordered recipes that chain several OSS Helper commands (with the manual gates between them) into one complete task.

- [`workflows/cve-workflow.md`](workflows/cve-workflow.md) — end-to-end CVE handling. It chains `/oss-triage-security-report` → `/oss-draft-cve` (passing the triage summary and fix PR through as `triage_ref=` / `fix_pr=`) plus the dependency-side `/oss-analyze-third-party-cve`, around the manual reserve / release / sign / publish / announce gates. It reads each project's `project-security.md` for the project-specific details.

## Adding a New Project

Open a pull request that adds a new `<project>/` directory with the three required files (and, optionally, a `project-security.md` describing the CVE-handling and publishing workflow). Use any existing project (for example `wanaku/`) as a template; for the optional security file, use any of the `camel-*` projects as a reference. The `generic-github/` directory should remain as the catch-all fallback.

Every rule file must include a `## Version` section at the bottom with the git SHA of the project being configured — `/oss-install-info` and `.oss-init.md` use this to detect when local copies are out of date.

## Installing Rules Locally

From any project directory:

```
/oss-install-info <project>
```

For example:

```
/oss-install-info camel-core
```

The command fetches the three rule files for `<project>` from this repository — plus `project-security.md` when that project provides it — and writes them under the helper's agent-specific rules directory (`~/.claude/rules/<project>/`, `~/.bob/rules/<project>/`, `~/.gemini/rules/<project>/`, `~/.config/opencode/rules/<project>/`, or `~/.codex/oss-helper/rules/<project>/`).

## Project-Local Rules Still Take Precedence

If a project repository already ships its own `.oss-ai-helper-rules/` directory, the helper uses those files first and ignores both this repository and any locally installed copies. See the helper's `.oss-init.md` for the full priority order.

## License

Apache License 2.0
