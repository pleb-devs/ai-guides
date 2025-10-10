# Goose CLI — PlebDevs AI Guide

> Practical, privacy-first usage of the **Goose CLI** for headless tasks, CI/CD, and automation. Desktop and CLI share the same configuration and extensions. ([Block][4])

For helpful tutorials, check out the [Goose YouTube channel](https://www.youtube.com/@goose-oss) (@goose/).

[SCREENSHOT OF TERMINAL RUNNING `goose --help`]

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install](#1-install)
  - [Configure a provider](#2-configure-a-provider)
  - [Harden defaults](#3-harden-defaults)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Recipes & scheduling](#recipes--scheduling)
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

The CLI delivers **scriptable sessions** and **one-off runs** from the terminal. Use it for batch jobs, CI/CD steps, and scheduled recipes. Settings are shared with Desktop via `~/.config/goose/config.yaml`. ([Block][21])

---

## Setup

### 1) Install

**One-liner (script):**

```bash
# Interactive install
curl -fsSL https://github.com/block/goose/releases/download/stable/download_cli.sh | bash

# Skip interactive configure step
curl -fsSL https://github.com/block/goose/releases/download/stable/download_cli.sh | CONFIGURE=false bash
```

**Homebrew:**

```bash
brew install block-goose-cli
```

([Block][4])

[SCREENSHOT OF INSTALL SCRIPT OUTPUT]

### 2) Configure a provider

Run:

```bash
goose configure
```

Pick OpenAI, Anthropic, OpenRouter, an OpenAI-compatible proxy, or local runners (Docker Model Runner/Ollama). Models without **tool-calling** only support chat and cannot drive extensions (OpenAI **o1-mini** and **o1-preview** are unsupported). ([Block][5])

[SCREENSHOT OF `goose configure`]

### 3) Harden defaults

- **Permission mode:** choose **Chat Only**, **Approve**, **Smart Approve**, or **Autonomous**. Start restrictive, widen later. ([Block][6])
- **Extension allowlist:** block unknown MCP servers globally so scripts stay safe. ([Block][7])
- **`.gooseignore`:** fence off secrets or infra paths; applies to Developer tools. ([Block][8])
- **Config path:** `~/.config/goose/config.yaml` (Windows path documented). ([Block][9])

[SCREENSHOT OF CONFIG YAML EXCERPT]

---

## Beginner usage

### A simple first flow (interactive)

**Goal**

- Learn Goose’s plan → review → approve → test loop in the terminal with one tiny, reversible change.

**Prereqs (safe start)**

- Work in your project root. Add a minimal `.gooseignore` to fence secrets (e.g., `**/.env*`, `**/secrets/**`). ([Block][8])
- Start in **Approve** mode so you can read files but still gate edits/shell:

```bash
cd /your/project
GOOSE_MODE=approve goose session
```

**Phase 1 — Survey & plan (no edits yet)**

- Ask: “Survey this repo. Propose one small, safe refactor with success criteria and tests to run.”
- Approve safe file reads if prompted; do not approve any edits yet. ([Block][10])

**Phase 2 — Make one change (Approve)**

- Ask for the smallest possible diff to meet the criteria.
- When prompted, approve exactly one edit tool call; decline anything out of scope. ([Block][6])

**Phase 3 — Verify locally**

- Approve the shell call to run the proposed test command (or run it yourself in another terminal). ([Block][10])
- If tests fail, ask for the minimal fix; approve once; retest.

**Phase 4 — Summarize (optional)**

- Ask Goose to draft a short change summary and rationale; paste into your commit message.
- Exporting is optional for audits or later PR authoring:

```bash
goose session export   # interactive picker
```

[SCREENSHOT OF TERMINAL EXPORT PROMPT]

### A second flow (headless, optional)

Use `goose run` for non-interactive tasks (documentation checks, standard generators) where no approvals are needed.

```bash
# Inline text
GOOSE_MODE=chat goose run -t "Generate a changelog from last tag to HEAD"

# From an instruction file
GOOSE_MODE=chat goose run -i instructions.md
```

`goose run` starts a new session, executes, and exits—ideal for scripts or CI jobs. Keep it in **Chat Only** for beginners to avoid unintended edits; move to **Approve**/**Autonomous** later when you’ve locked down tools. ([Block][22])

---

## Pro usage

### Recipes & scheduling

Package stable workflows as **recipes** and run on demand or on a schedule.

```bash
# Run now (with params)
goose run --recipe ./recipes/daily-report.yaml env=prod

# Schedule daily at 09:00
goose schedule add --id daily-report --cron "0 0 9 * * *" \
  --recipe-source ./recipes/daily-report.yaml

goose schedule list
goose schedule run-now --id daily-report
```

Consult the Recipe Reference for YAML/JSON fields. ([Block][23])

[SCREENSHOT OF “SCHEDULES” LIST OUTPUT]

### Headless & CI/CD

Call `goose run` from GitHub Actions or any CI runner to automate code review, documentation checks, and more. ([Block][23])

[SCREENSHOT OF A CI JOB LOG SHOWING `goose run`]

### Projects

The CLI tracks working directories as **projects**, so you can resume context quickly. Metadata lives at `~/.local/share/goose/projects.json`. ([Block][24])

[SCREENSHOT OF `goose projects` PICKER]

### CLI providers

Reuse subscriptions (Claude Code, Cursor Agent, Gemini CLI) **through Goose** to centralize spend and session history. ([Block][25])

[SCREENSHOT OF “CLI PROVIDERS” GUIDE]

### ACP clients

Expose Goose as an **ACP** agent over stdio for editors like **Zed**:

```bash
goose acp
```

Usually the client launches this automatically. ([Block][21])

[SCREENSHOT OF ZED AGENT PANEL WITH GOOSE]

---

## Cost savings guide

- **Lead/worker strategy:** plan with a premium model, execute with a cheaper worker. ([Block][13])
- **Local models:** use Docker Model Runner or Ollama for $0 API fees. ([Block][5])
- **Compact early:** auto-compaction triggers near 80% of context; tune via env. ([Block][12])
- **CLI providers:** reuse Claude Code, Cursor, or Gemini CLI access if you already subscribe. ([Block][25])

---

## Privacy guide

- **On-disk by default:** logs/config under `~/.config/goose`; app data under platform paths. Secrets go to the system keychain unless disabled. ([Block][15])
- **Disable keyring if needed:** set `GOOSE_DISABLE_KEYRING=1` to store secrets in the documented local file instead. ([Block][16])
- **Persist preferences safely:** use **`.goosehints`** (global and project) to avoid re-sending sensitive prompts. ([Block][17])

---

## Security guide

- **Pick restrictive modes** for CI and production repos (**Chat Only** or **Approve**). ([Block][6])
- **Allowlist extensions** and configure per-tool ask/deny rules for destructive commands. ([Block][7])
- **Run in containers** for risky tasks or to isolate environments (see **container-use** tutorial). ([Block][20])

---

## Appendix

**Common env (subset)**

```bash
# Models
export GOOSE_PROVIDER=openai|anthropic|openrouter|...
export GOOSE_MODEL="gpt-4o-mini"
export GOOSE_LEAD_MODEL="claude-3-5-sonnet"

# Modes & safety
export GOOSE_MODE=smart_approve       # auto|approve|chat|smart_approve
export GOOSE_ALLOWLIST="https://example.com/allowlist.yaml"
export GOOSE_MAX_TURNS=25
export GOOSE_AUTO_COMPACT_THRESHOLD=0.6
export GOOSE_DISABLE_KEYRING=1
```

([Block][16])

**CLI essentials**

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

([Block][21])

---

## Troubleshooting quick hits

- **Local models:** with Ollama, pull and run a model before using it or requests will fail. ([Block][15])
- **Long-running dev servers:** customize the shell via `GOOSE_TERMINAL` to prevent hangs during `goose run`. ([Block][15])
- **Context limits:** auto-compaction engages around 80% of the window; tune via env. ([Block][12])

[SCREENSHOT OF `goose --version` OUTPUT]

---

**Changelog note:** This guide reflects Goose docs as of **October 4, 2025**. Re-check provider support, permission defaults, and extension security before pinning workflows. ([Block][5])

---

[4]: https://block.github.io/goose/docs/getting-started/installation/ "Install Goose"
[5]: https://block.github.io/goose/docs/getting-started/providers/ "Configure LLM Provider"
[6]: https://block.github.io/goose/docs/guides/goose-permissions/ "Goose Permission Modes"
[7]: https://block.github.io/goose/docs/guides/allowlist/ "Goose Extension Allowlist"
[8]: https://block.github.io/goose/docs/guides/using-gooseignore/ "Prevent Goose from Accessing Files"
[9]: https://block.github.io/goose/docs/guides/config-file/ "Configuration File"
[10]: https://block.github.io/goose/docs/guides/sessions/session-management/ "Session Management"
[12]: https://block.github.io/goose/docs/guides/sessions/smart-context-management/ "Smart Context Management"
[13]: https://block.github.io/goose/docs/tutorials/lead-worker/ "Lead/Worker Multi-Model Setup"
[15]: https://block.github.io/goose/docs/troubleshooting/ "Troubleshooting"
[16]: https://block.github.io/goose/docs/guides/environment-variables/ "Environment Variables"
[17]: https://block.github.io/goose/docs/guides/using-goosehints/ "Providing Hints to Goose"
[20]: https://block.github.io/goose/docs/tutorials/isolated-development-environments/ "Isolated Development Environments"
[21]: https://block.github.io/goose/docs/guides/goose-cli-commands/ "CLI Commands"
[22]: https://block.github.io/goose/docs/guides/running-tasks/ "Running Tasks"
[23]: https://block.github.io/goose/docs/tutorials/cicd/ "CI/CD Environments"
[24]: https://block.github.io/goose/docs/guides/managing-projects/ "Managing Projects"
[25]: https://block.github.io/goose/docs/guides/cli-providers/ "CLI Providers"
