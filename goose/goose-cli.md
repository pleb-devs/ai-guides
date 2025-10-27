# Goose CLI — PlebDevs AI Guide

> Practical, privacy-first usage of the Goose CLI for
> headless tasks, CI/CD, and automation. Desktop and CLI share
> config, providers, and extensions via
> `~/.config/goose/config.yaml`. ([Block][15])

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Verify install](#1-verify-install)
  - [Configure a provider](#2-configure-a-provider)
  - [Harden defaults](#3-harden-defaults)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Recipes, Tasks, and Plans](#recipes-tasks-and-plans)
  - [Headless & CI/CD](#headless--cicd)
  - [Projects](#projects)
  - [CLI providers](#cli-providers)
  - [ACP clients](#acp-clients)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)
- [Troubleshooting quick hits](#troubleshooting-quick-hits)

---

## Overview

The CLI provides scriptable sessions and one‑off runs from the
terminal. Use it for batch jobs, CI steps, and scheduled recipes.
See Quickstart and CLI Commands for options. ([Block][1],
[Block][15])

---

## Setup

### 1) Verify install

Verify the binary is available and inspect configuration:

```bash
goose --version
goose info --verbose
```

Refer to Quickstart/Installation for platform‑specific install
methods. ([Block][1], [Block][2])

### 2) Configure a provider

Run interactive configuration:

```bash
goose configure
```

Pick a provider (Anthropic, OpenAI, OpenRouter, or a CLI provider).
Models without tool‑calling can chat but won’t use extensions.
([Block][3])

### 3) Harden defaults

- Permission mode: choose Chat Only, Approve Mode, Smart Approval,
  or Completely Autonomous. In the CLI, Approve Mode shows
  Allow/Deny prompts for write-classified tools. ([Block][4]) Watch the
  [Goose Permission Modes Explained](https://www.youtube.com/watch?v=bMVFFnPS_Uk)
  video for a walkthrough of each option.
- Extension allowlist: restrict which MCP servers can load. ([Block][5])
- `.gooseignore`: fence secrets and infra paths. ([Block][6])
- Shared config path: `~/.config/goose/config.yaml`. ([Block][7])

---

## Beginner usage

### A simple first flow (interactive)

Goal

- Learn Goose’s plan → review → approve → test loop with one tiny,
  reversible change.

Prereqs (safe start)

- Work in your project root. Add a minimal `.gooseignore` (e.g.,
  `**/.env*`, `**/secrets/**`). ([Block][6])
- Start in Approve Mode so you can read files but still gate
  edits/shell:

```bash
cd /your/project
GOOSE_MODE=approve goose session --name demo
```

Phase 1 — Survey & plan (no edits yet)

- Ask: “Survey this repo. Propose one small, safe refactor with
  success criteria and tests to run.”
- Approve safe reads if prompted; do not approve edits yet.
  ([Block][8])

Phase 2 — Make one change (Approve Mode)

- Ask for the smallest possible diff.
- When prompted, approve exactly one edit tool call; decline anything
  out of scope. ([Block][4])

Phase 3 — Verify locally

- Approve a shell run for the test command (or run it yourself).
  ([Block][8])

Phase 4 — Summarize (optional)

- Ask Goose to draft a short change summary; paste into your commit
  message.
- Export the session for audits or PRs:

```bash
goose session export   # interactive picker
```

### A second flow (headless)

Use `goose run` for non‑interactive tasks where no approvals are
needed.

```bash
# Inline text
GOOSE_MODE=chat goose run -t "Generate a changelog from last tag to HEAD"

# From an instruction file
GOOSE_MODE=chat goose run -i instructions.md
```

`goose run` starts a new session, executes, and exits—ideal for CI.
See Running Tasks. ([Block][16])

---

## Pro usage

### Recipes, Tasks, and Plans

Package repeatable work as recipes and run on demand or on a
schedule.

```bash
# Validate and preview
goose recipe validate ./recipes/review.yaml
goose run --recipe ./recipes/review.yaml --explain

# Run now (with params)
goose run --recipe ./recipes/review.yaml repo=my/repo --interactive

# Schedule daily at 09:00
goose schedule add --id daily-review \
  --cron "0 0 9 * * *" \
  --recipe-source ./recipes/review.yaml
goose schedule list
goose schedule run-now --id daily-review
```

Minimal `recipe.yaml` with a parameter:

```yaml
name: code-review-assistant
description: Helps review pull requests for a repository

parameters:
  repo:
    type: string
    required: true
    description: GitHub repository in owner/name form

instructions: |
  You are a code review assistant.
  Focus on security issues and actionable comments.
  The target repository will be provided via the repo parameter.

# Optional: list MCP extensions needed by the workflow
extensions:
  - id: github

# Optional: common prompts shown at the top of the session
activities:
  - "Scan latest PR for major issues"
```

Run with a parameter:

```bash
goose run --recipe ./recipes/review.yaml repo=my/repo --interactive
```

Consult the Recipe Reference for YAML/JSON fields. ([Block][9])

### Headless & CI/CD

Call `goose run` from GitHub Actions or any CI runner to automate
checks and generators. ([Block][9])

### Projects

Quickly resume context across codebases. Project metadata is stored
under your user data directory.

```bash
goose project     # resume most recent project
goose projects    # browse all projects
```

See Managing Projects. ([Block][18])

### CLI providers

Use existing subscriptions (Claude Code, Cursor Agent, Gemini CLI) via
Goose for persistent sessions, recipes, and scheduling; extension
support may be limited. ([Block][19])

### ACP clients

Expose Goose over stdio to ACP‑compatible clients (e.g., Zed):

```bash
goose acp
```

Usually launched by the client. See CLI Commands. ([Block][15])

---

## Cost savings guide

- Lead/worker strategy: plan with a premium model, execute with a
  cheaper worker. ([Block][1])
- Local models where quality allows. ([Block][3])
- Compact early: auto‑compaction triggers near 80% of context; tune via env. ([Block][10])
- Consider CLI providers if you already subscribe. ([Block][19])

---

## Privacy guide

- On‑disk by default: configs in `~/.config/goose`; sessions/logs in
  platform paths. ([Block][7])
- Disable keyring if needed: `GOOSE_DISABLE_KEYRING=1`. ([Block][12])
- Persist preferences safely: use `.goosehints` (global or project). ([Block][13])

---

## Security guide

- Pick restrictive modes for CI and production repos (Chat Only or
  Approve Mode). ([Block][4])
- Allowlist extensions and configure per‑tool ask/deny rules for
  destructive commands. ([Block][5])
- Isolate risky tasks in containers. ([Block][11])

---

## Appendix

Common env (subset)

```bash
# Models
export GOOSE_PROVIDER=openai|anthropic|openrouter|...
export GOOSE_MODEL="gpt-4o-mini"

# Modes & safety
export GOOSE_MODE=smart_approve       # auto|approve|chat|smart_approve
export GOOSE_ALLOWLIST="https://example.com/allowlist.yaml"
export GOOSE_MAX_TURNS=25
export GOOSE_AUTO_COMPACT_THRESHOLD=0.6
export GOOSE_DISABLE_KEYRING=1
```

CLI essentials

```bash
goose --help
goose info --verbose
goose configure
goose session
goose run -t "..."
goose session export
goose schedule add --id demo --cron "0 0 9 * * *" --recipe-source ./recipes/demo.yaml
goose acp
```

Reference: CLI Commands. ([Block][15])

---

## Troubleshooting quick hits

- Local models: ensure your runner is started; see Troubleshooting. ([Block][14])
- Long‑running dev servers: customize shell via `GOOSE_TERMINAL` for
  `goose run`. ([Block][14])
- Context limits: auto‑compaction engages around 80%; tune via env. ([Block][10])

---

[1]: https://block.github.io/goose/docs/quickstart "Quickstart"
[2]: https://block.github.io/goose/docs/getting-started/installation/ "Install Goose"
[3]: https://block.github.io/goose/docs/getting-started/providers/ "Configure LLM Provider"
[4]: https://block.github.io/goose/docs/guides/goose-permissions/ "Goose Permission Modes"
[5]: https://block.github.io/goose/docs/guides/allowlist/ "Goose Extension Allowlist"
[6]: https://block.github.io/goose/docs/guides/using-gooseignore/ "Prevent Goose from Accessing Files"
[7]: https://block.github.io/goose/docs/guides/config-file/ "Configuration File"
[8]: https://block.github.io/goose/docs/guides/sessions/session-management/ "Session Management"
[9]: https://block.github.io/goose/docs/guides/recipes/ "Recipes"
[10]: https://block.github.io/goose/docs/guides/sessions/smart-context-management/ "Smart Context Management"
[11]: https://block.github.io/goose/docs/tutorials/isolated-development-environments/ "Isolated Development Environments"
[12]: https://block.github.io/goose/docs/guides/environment-variables/ "Environment Variables"
[13]: https://block.github.io/goose/docs/guides/using-goosehints/ "Providing Hints to Goose"
[14]: https://block.github.io/goose/docs/troubleshooting/ "Troubleshooting"
[15]: https://block.github.io/goose/docs/guides/goose-cli-commands/ "CLI Commands"
[16]: https://block.github.io/goose/docs/guides/running-tasks/ "Running Tasks"
[18]: https://block.github.io/goose/docs/guides/managing-projects/ "Managing Projects"
[19]: https://block.github.io/goose/docs/guides/cli-providers/ "CLI Providers"
