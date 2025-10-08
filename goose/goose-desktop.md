# Goose Desktop — PlebDevs AI Guide

> Practical, privacy-first usage of **Goose Desktop** for adversarial, secure, and cost-efficient workflows. Goose is an on-machine AI agent that uses **MCP extensions** to edit files, run commands, and automate tasks. Sessions live locally. ([Block][1])

<img width="1789" height="1088" alt="image" src="https://github.com/user-attachments/assets/cc8d06ac-1c1d-44e8-9c78-611016ee916b" />

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install](#1-install)
  - [Configure a provider](#2-configure-a-provider)
  - [Harden defaults](#3-harden-defaults)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Lead/worker & model switching](#leadworker--model-switching)
  - [Extensions (MCP)](#extensions-mcp)
  - [Recipes](#recipes)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)
- [Troubleshooting quick hits](#troubleshooting-quick-hits)

---

## Overview

**What it is.** Goose Desktop is a local AI agent with an integrated chat UI. It is **extensible** via the Model Context Protocol (MCP), so the agent can use tools to edit files, run shell commands, call APIs, and browse. Sessions and config stay on disk. ([Block][2])

**When to use.**

- Iterative coding where you want tight control over edits and tool usage.
- Repeatable workflows that you can turn into **recipes**.
- Adversarial or locked-down setups where allowlists, strict permission modes, and **local models** are required. ([Block][3])

<img width="1333" height="626" alt="image" src="https://github.com/user-attachments/assets/033e52cb-95ba-4120-81f9-e2e89edc2f45" />

<img width="1766" height="809" alt="image" src="https://github.com/user-attachments/assets/4a3bfe83-05d8-4cde-8834-d0d5d4906aed" />


---

## Setup

### 1) Install

Download and install the Desktop app from the official install page. If you need the CLI later, add it separately—Desktop and CLI share config. ([Block][4])

<img width="1423" height="883" alt="image" src="https://github.com/user-attachments/assets/c4d92e3e-ccb0-4827-b2a6-3d12b20c8624" />

### 2) Configure a provider

On first launch, Goose prompts you to pick an **LLM provider** (OpenAI, Anthropic, OpenRouter, OpenAI-compatible proxies like vLLM/KServe, or **local** runners such as Docker Model Runner and Ollama). Models without **tool-calling** only support chat; Goose’s tool usage requires tool-calling, so OpenAI **o1-mini** and **o1-preview** are unsupported. ([Block][5])

<img width="1063" height="1037" alt="image" src="https://github.com/user-attachments/assets/d35d65f7-d86d-42b2-a026-8591f6d1a440" />

### 3) Harden defaults

- **Permission mode:** choose **Chat Only**, **Approve**, **Smart Approve**, or **Autonomous**. Toggle in Settings or with `/mode` inside a session. Start restrictive; loosen later. ([Block][6])
  <img width="1531" height="582" alt="image" src="https://github.com/user-attachments/assets/41ea918c-7d73-4296-b318-5283a7dba699" />
- **Extension allowlist:** restrict which MCP servers can load so Desktop and CLI stay in sync. ([Block][7])
  <img width="1765" height="636" alt="image" src="https://github.com/user-attachments/assets/31b4dea1-90db-49be-909c-ee6cd68ef8b8" />
- **Fence off files:** add `.gooseignore` (project or global) to block reads, writes, and shell on sensitive paths. ([Block][8])
  <img width="773" height="396" alt="image" src="https://github.com/user-attachments/assets/7251066d-d88b-44f3-809d-7955413bbe11" />
- **Know where config lives:** `~/.config/goose/config.yaml` (or find it in the ui from below img). Both Desktop and CLI share this file. ([Block][9])
  <img width="703" height="382" alt="image" src="https://github.com/user-attachments/assets/ff7377bd-1648-4e53-935e-8757957c8e71" />


---

## Beginner usage

### The mental model

Open a new session, describe the task, **review the plan**, then approve tool runs. Keep each session focused; export transcripts for audits or PR write-ups. ([Block][10])

<img width="1590" height="341" alt="image" src="https://github.com/user-attachments/assets/b25d9cfa-bb76-42c1-95d6-3198625cc0ae" />

### Core moves (Desktop)

- **Quick file reference** with `@`; fuzzy search pops in-line. ([Block][11])
  <img width="1568" height="672" alt="image" src="https://github.com/user-attachments/assets/31e068b5-0e32-401b-a07e-b3113da5685f" />
- **Smart context:** Goose auto-compacts older turns once usage hits ~80% of the model window; tune with `GOOSE_AUTO_COMPACT_THRESHOLD`. ([Block][12])
- **Export sessions** (currently via CLI ONLY) ([Block][10])
  ```bash
  goose session export   # interactive picker
  ```
  <img width="1466" height="891" alt="image" src="https://github.com/user-attachments/assets/5aac07bd-e9ec-4349-87ba-581635aa3d75" />

### A simple first flow

**Goal**

- Learn Goose’s plan → review → approve → test loop with one tiny, reversible change.

**Prereqs (safe start)**

- Set mode to Chat Only: `/mode chat` (or use Settings).
- Add a minimal `.gooseignore` to fence secrets (for example: `**/.env*`, `**/secrets/**`).

[SCREENSHOT OF "COMPOSER — /MODE DROPDOWN"]

**Phase 1 — Survey & plan (read‑only)**

- Ask: “Survey this repo. Propose one small, safe refactor with success criteria and tests to run.”
- Ensure you understand which file(s) will change and how you’ll verify.

**Phase 2 — Make one change (Approve mode)**

- Switch to Approve: `/mode approve`.
- Use `@` to reference the target file and request the smallest possible diff that meets the criteria.
- Review the diff; approve only that edit.

[SCREENSHOT OF "EDIT APPROVAL DIALOG — ONE‑FILE DIFF"]

[SCREENSHOT OF "APPROVE SHELL TOOL CALL — RUN TESTS"]

**Phase 3 — Verify locally**

- Run the test command Goose proposed (or approve it to run via shell).
- If tests fail, ask for the minimal fix; approve once; retest.

[SCREENSHOT OF "TEST RESULTS PANEL — PASSING TESTS"]

**Phase 4 — Summarize (optional)**

- Ask Goose to draft a short change summary and rationale; paste into your commit message.
- Exporting the session is optional; use it later when you want an audit trail or to help write a PR. ([Block][6])

---

## Pro usage

### Lead/worker & model switching

Use a strong **lead** model for planning and a cheaper **worker** for execution. Goose supports multi-model strategies and dynamic switching (AutoPilot/Planning Mode vary by setup). Configure in Settings or via environment variables. ([Block][13])

[SCREENSHOT OF “SETTINGS → MODELS (LEAD/WORKER)”]

### Extensions (MCP)

Install from **Settings → Extensions** or add remote servers (SSE/streaming HTTP). Example: **GitMCP** for deep repo context.

```bash
npx -y mcp-remote https://gitmcp.io/docs
```

Extensions power file edits, shell, browsers, and more. ([Block][2])

[SCREENSHOT OF “EXTENSIONS → ADD EXTENSION”]

### Recipes

Turn repeatable work into **recipes** (YAML/JSON). Capture from a session, then replay on Desktop or CLI. ([Block][3])

[SCREENSHOT OF “RECIPES → CREATE FROM SESSION”]

---

## Cost savings guide

- **Lead/worker split:** plan with a premium model, execute with a cheaper worker. ([Block][13])
- **Go local when quality allows:** Docker Model Runner or Ollama keeps spend at $0 API. ([Block][5])
- **Compact early:** rely on auto-compaction (~80% by default) or set a lower threshold for long sessions. ([Block][12])
- **Limit extensions:** enable only what you need; fewer tools reduce overhead. Consider **Tool Router (preview)** if it fits your stack. ([Block][14])

[SCREENSHOT OF “SELECT MODELS (CHEAPER WORKER)”]

---

## Privacy guide

- **Local by default:** config and logs remain on disk; Desktop and CLI share state. Secrets live in the system keychain unless you override it. ([Block][15])
- **Keyring control:** set `GOOSE_DISABLE_KEYRING=1` if OS keychains are unavailable; Goose falls back to a local secrets file. ([Block][16])
- **Avoid repeating sensitive context:** use **`.goosehints`** (project/global) for stable rules and combine with Memory as needed. ([Block][17])
- **Observability (opt-in):** if you enable **Langfuse**, point it to your own instance/region. ([Block][18])

[SCREENSHOT OF “HISTORY → EXPORT SESSION AS MARKDOWN”]

---

## Security guide

- **Pick the right mode:** use **Chat Only** for analysis, **Approve/Smart Approve** for normal coding, and reserve **Autonomous** for sandboxes. ([Block][6])
- **Allowlist extensions:** block unknown MCP servers across Desktop and CLI. ([Block][7])
- **Tool-level permissions:** override risky tools (e.g., destructive shell commands) on a per-tool basis. ([Block][19])
- **Isolate work:** run high-risk tasks in containers (see the **container-use** tutorial) or run models via Docker Model Runner. ([Block][20])

[SCREENSHOT OF “PERMISSIONS → TOOL PERMISSIONS TABLE”]

---

## Appendix

**Key paths**

- Config: `~/.config/goose/config.yaml`
- Sessions: `~/.local/share/goose/sessions/`
- Logs: `~/.local/state/goose/logs/` ([Block][9])

**Handy env (subset)**

```bash
# Models
GOOSE_PROVIDER=openai|anthropic|openrouter|...
GOOSE_MODEL=gpt-4o-mini
GOOSE_LEAD_MODEL=claude-3-5-sonnet

# Modes & safety
GOOSE_MODE=smart_approve       # auto|approve|chat|smart_approve
GOOSE_ALLOWLIST=https://example.com/allowlist.yaml
GOOSE_MAX_TURNS=25
GOOSE_AUTO_COMPACT_THRESHOLD=0.6
GOOSE_DISABLE_KEYRING=1
```

([Block][16])

---

## Troubleshooting quick hits

- **Ollama local models:** pull and run a model before using it; otherwise the provider fails. ([Block][15])
- **Context limits:** auto-compaction triggers around 80% of the context window; tune or disable via env. ([Block][12])

[SCREENSHOT OF “ABOUT/HELP → VERSION INFO”]

---

**Changelog note:** This guide reflects Goose docs as of **October 4, 2025**. Re-check provider support, permission defaults, and extension security before pinning workflows. ([Block][5])

---

[1]: https://block.github.io/goose/ "goose"
[2]: https://block.github.io/goose/docs/getting-started/using-extensions/ "Using Extensions | goose"
[3]: https://block.github.io/goose/docs/guides/recipes/ "Recipes | goose"
[4]: https://block.github.io/goose/docs/getting-started/installation/ "Install Goose"
[5]: https://block.github.io/goose/docs/getting-started/providers/ "Configure LLM Provider"
[6]: https://block.github.io/goose/docs/guides/goose-permissions/ "Goose Permission Modes"
[7]: https://block.github.io/goose/docs/guides/allowlist/ "Goose Extension Allowlist"
[8]: https://block.github.io/goose/docs/guides/using-gooseignore/ "Prevent Goose from Accessing Files"
[9]: https://block.github.io/goose/docs/guides/config-file/ "Configuration File"
[10]: https://block.github.io/goose/docs/guides/sessions/session-management/ "Session Management"
[11]: https://block.github.io/goose/docs/guides/file-management/ "File Access and Management"
[12]: https://block.github.io/goose/docs/guides/sessions/smart-context-management/ "Smart Context Management"
[13]: https://block.github.io/goose/docs/tutorials/lead-worker/ "Lead/Worker Multi-Model Setup"
[14]: https://block.github.io/goose/docs/guides/tool-router/ "Tool Router (preview)"
[15]: https://block.github.io/goose/docs/troubleshooting/ "Troubleshooting"
[16]: https://block.github.io/goose/docs/guides/environment-variables/ "Environment Variables"
[17]: https://block.github.io/goose/docs/guides/using-goosehints/ "Providing Hints to Goose"
[18]: https://block.github.io/goose/docs/tutorials/langfuse/ "Observability with Langfuse"
[19]: https://block.github.io/goose/docs/guides/managing-tools/tool-permissions/ "Managing Tool Permissions"
[20]: https://block.github.io/goose/docs/tutorials/isolated-development-environments/ "Isolated Development Environments"
