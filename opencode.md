# OpenCode (CLI Coding Agent)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install](#1-install)
  - [Pick a provider](#2-pick-a-provider-fast-path-opencode-zen)
  - [Initialize a project](#3-initialize-a-project)
- [Beginner usage](#beginner-usage)
  - [The mental model](#the-mental-model)
  - [Core moves (TUI)](#core-moves-tui)
  - [A simple first flow](#a-simple-first-flow)
- [Pro usage](#pro-usage)
  - [Declarative config](#declarative-config-global--per-project)
  - [Agents, rules & instructions](#agents-rules--instructions)
  - [LSP, MCP, and tools](#lsp-mcp-and-tools)
  - [CLI & headless server](#cli--headless-server)
  - [GitHub automation](#github-automation)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)


## Overview

**What it is.** OpenCode is a **terminal UI (TUI) coding agent** that can analyze, edit, and refactor your codebase with LLMs. It’s **provider-agnostic** (supports 75+ providers via Models.dev), can run **local models** (Ollama / LM Studio), integrates with **LSP servers** for code feedback, and offers **shareable sessions**, **GitHub automation**, and a **headless server** mode. ([Opencode][1])

**Key ideas**

* **Native terminal UI** with Plan/Build modes, file references (`@`), shell commands (`!`), and slash-commands (`/undo`, `/redo`, `/share`, etc.). ([Opencode][2])
* **Models & providers**: use any major API (Anthropic, OpenAI, Bedrock, Groq, xAI, etc.) or local inference via Ollama/LM Studio. **“Claude Pro/Max”** login is supported; **OpenCode Zen** is a curated model list. ([Opencode][3])
* **Config-driven**: JSON/JSONC config with per-project overrides; explicit controls for **permissions**, **MCP servers**, **LSP**, **commands**, **formatters**, **themes**, **autoupdate**, and **share** modes. ([Opencode][4])
* **Privacy posture**: project code is not stored by OpenCode; **sharing is opt-in** and can be disabled globally. ([Opencode][5])

---

## Setup

### 1) Install

**One-liner (recommended)**

```bash
curl -fsSL https://opencode.ai/install | bash
```

**Package managers**

```bash
# macOS/Linux
brew install sst/tap/opencode

# Node toolchains
npm install -g opencode-ai
bun install -g opencode-ai
pnpm install -g opencode-ai
yarn global add opencode-ai

# Arch (paru)
paru -S opencode-bin

# Windows
choco install opencode
winget install opencode
scoop bucket add extras && scoop install extras/opencode
```

(Windows Bun install coming soon; binary releases available.) ([Opencode][1])

### 2) Pick a provider (fast path: **OpenCode Zen**)

OpenCode supports 75+ providers through **AI SDK** and **Models.dev**; credentials are stored locally at `~/.local/share/opencode/auth.json`. For a zero-friction start:

```bash
opencode auth login   # choose "opencode" to use the curated Zen models
```

Then run `/models` in the TUI to select. ([Opencode][3])

> Tip: You can also log in with **Anthropic Claude Pro/Max** from the TUI auth flow. ([Opencode][3])

### 3) Initialize a project

```bash
cd /your/project
opencode
# in TUI:
#   /init   (generates AGENTS.md tailored to your repo)
```

Commit `AGENTS.md` so the agent understands your structure and conventions. ([Opencode][6])

---

## Beginner usage

### The mental model

* Start OpenCode in your repo, **ask questions**, and **iterate**.
* Use **Plan** mode first (no write access), then **Build** to apply changes. Toggle with **Tab**. ([Opencode][6])

### Core moves (TUI)

* **Reference files** with `@` (fuzzy finds + inlines contents).
  *“How is auth handled in @packages/functions/src/api/index.ts?”* ([Opencode][2])
* **Run shell** with `!` (captured into context).
  *`!ls -la`* ([Opencode][2])
* **Undo / redo** code & messages (`/undo`, `/redo`)—uses Git under the hood. ([Opencode][2])
* **List models** (`/models`) and **share a session** (`/share`, manual by default). ([Opencode][2])

### A simple first flow

1. **Explain** the repo: “Give me a quick summary of the codebase.”
2. **Plan** a change: switch to Plan; describe the feature.
3. **Review plan**; add details (you can **drag-drop images** into the terminal).
4. **Build**: toggle back, ask it to implement.
5. **Undo/redo** if needed. ([Opencode][6])

---

## Pro usage

### Declarative config (global & per-project)

* Global: `~/.config/opencode/opencode.json`
* Project: `./opencode.json` (takes precedence)
* Override path with `OPENCODE_CONFIG=/path/to/config.json`
  Supports **JSON/JSONC** + schema at `https://opencode.ai/config.json`. ([Opencode][4])

**Snippets you’ll reuse**

```jsonc
// Select models, set a cheaper small_model, pin theme & updates
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-20250514",
  "small_model": "anthropic/claude-3-5-haiku-20241022",
  "theme": "opencode",
  "autoupdate": false
}
```

([Opencode][4])

```jsonc
// Lock down agent capabilities & bash
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "edit": "ask",
    "bash": {
      "git push": "ask",
      "terraform *": "deny",
      "*": "allow"
    }
  }
}
```

([Opencode][7])

```jsonc
// Add custom commands you can run as "/command <args>"
{
  "$schema": "https://opencode.ai/config.json",
  "command": {
    "component": {
      "template": "Create a new React component named $ARGUMENTS with TypeScript."
    }
  }
}
```

([Opencode][4])

```jsonc
// Load local models (LM Studio or Ollama)
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": { "baseURL": "http://127.0.0.1:1234/v1" },
      "models": { "google/gemma-3n-e4b": { "name": "Gemma 3n-e4b (local)" } }
    },
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": { "llama2": { "name": "Llama 2" } }
    }
  }
}
```

([Opencode][3])

```jsonc
// Disable risky/expensive providers outright (belt-and-suspenders)
{ "disabled_providers": ["openai", "gemini"] }
```

([Opencode][4])

### Agents, rules & instructions

* Define **agents** in JSON or Markdown (eg. `~/.config/opencode/agent/*.md`) with custom prompts, models, and allowed tools.
* Generate or evolve `AGENTS.md` with `/init` and include **instructions** via `instructions: [...]` and glob patterns.
* OpenCode merges **project** and **global** rules. ([Opencode][8])

### LSP, MCP, and tools

* OpenCode auto-starts LSP servers (TS, Go, Rust, Python, etc.) and can **disable LSP downloads** via `OPENCODE_DISABLE_LSP_DOWNLOAD=true`. You can add or override servers in config. ([Opencode][9])
* Add **MCP servers** (local or remote) to expose custom tools to the LLM; enable them globally or **per-agent** with wildcards. ([Opencode][10])

### CLI & headless server

* **CLI non-interactive**: `opencode run "…"` with flags (`--model`, `--agent`, `--session`, `--share`).
* **Headless HTTP**: `opencode serve` exposes an API (see server docs).
* **Upgrade** from CLI (`opencode upgrade`). ([Opencode][11])

### GitHub automation

* Mention `/opencode` or `/oc` in issues/PRs; the agent runs **inside your GitHub Action runner**, creating branches/PRs as requested. Install via `opencode github install` or add the workflow manually. ([Opencode][12])

---

## Cost savings guide

**Pick cost-effective access**

* If available to you, **Claude Pro/Max** is called out as a **cost-effective** way to use OpenCode. Use `opencode auth login → Anthropic → Claude Pro/Max`. ([Opencode][3])

**Right-size the model**

* Set a **cheaper `small_model`** for light tasks (titles, summaries) and keep your main model for builds. ([Opencode][4])
* Prefer **local models (Ollama / LM Studio)** for refactors/tests on private code when quality allows—\$0 in API fees, only local compute. ([Opencode][3])

**Reduce unnecessary tokens**

* Work in **Plan mode** first (no edits), get the spec tight, then **Build**—this reduces trial-and-error iterations. ([Opencode][6])
* Use `/compact` to summarize long sessions and keep context lean. ([Opencode][2])
* Prefer `@file` references over pasting large blobs manually. ([Opencode][2])

**Avoid accidental spend**

* Lock to allowed providers/models with `disabled_providers` and explicit `model` strings. ([Opencode][4])
* Use CLI `--model` to pin a cheaper model in batch jobs. ([Opencode][11])

---

## Privacy guide

**What OpenCode stores**

* **OpenCode does not store your code or context data**; processing happens locally or directly with your LLM provider. The exception is the **optional Share feature**. ([Opencode][5])
* **Sharing** creates a public link and syncs the conversation until you `/unshare`. Set `"share": "disabled"` globally (or per-project) if needed. ([Opencode][13])

**Practical privacy steps**

* Prefer **local models** (Ollama / LM Studio) for sensitive code so nothing leaves your machine. ([Opencode][3])
* Route APIs through a **custom `baseURL`** (egress proxy / self-hosted gateway) in the provider config. ([Opencode][3])
* Keep credentials out of history: store keys via `opencode auth login` (saved at `~/.local/share/opencode/auth.json`) or use **config variable substitution** to pull from environment/files. ([Opencode][3])
* Consider disabling **automatic LSP downloads** in restricted environments: `OPENCODE_DISABLE_LSP_DOWNLOAD=true`. ([Opencode][9])

**Team/enterprise**

* For trials, **disable sharing**; enterprise deployments can restrict share links to SSO users or self-host the share service. ([Opencode][5])

---

## Security guide

**Principle of least privilege**

* **Permissions**: require approval for edits and shell, and pattern-block risky commands (`git push`, `terraform *`, `kubectl *`, etc.). Use `"ask"`/`"deny"` and wildcard maps. ([Opencode][7])
* **Per-agent tool scopes**: give heavy-duty tools only to agents that truly need them. (Enable/disable tools or MCP servers per agent.) ([Opencode][10])

**Supply-chain hygiene**

* Pin OpenCode behavior by setting `"autoupdate": false` and updating on your schedule via `opencode upgrade`. ([Opencode][4])
* Use a **private NPM registry** and ensure developers are authenticated before running OpenCode if your org requires it. ([Opencode][5])

**Data boundaries**

* Keep **share** mode on `"manual"` (default) or `"disabled"` for sensitive repos; `/unshare` to remove links and data. ([Opencode][13])
* Prefer **local models** for regulated code paths; otherwise, restrict providers with `disabled_providers`. ([Opencode][4])

**Network egress control**

* Point providers at a **proxy** via `provider.*.options.baseURL` and control outbound traffic centrally. ([Opencode][3])

---

## Appendix

**Useful commands**

* `opencode` (launch TUI) • `/init` • `/models` • `/undo` • `/redo` • `/share` • `/unshare` • `/compact` • `!<cmd>` • `@path/to/file` ([Opencode][2])
* `opencode auth login` • `opencode models` • `opencode run "…" --model … --agent …` • `opencode serve` • `opencode upgrade` ([Opencode][11])

**Version signal**

* Active, fast-moving repo (MIT) with frequent releases; latest as of **Sep 24, 2025** shows `v0.11.3`. Always consult releases before pinning. ([GitHub][14])

---

If you want, I can generate a **starter `opencode.json`** tuned for your stacks (TS/Node, Rust, Go, Python), plus a couple of **ready-made agents** (e.g., “Security Auditor,” “DB Migration Planner”) and a **sane permissions baseline**.

[1]: https://opencode.ai/ "opencode | AI coding agent built for the terminal"
[2]: https://opencode.ai/docs/tui/ "TUI | opencode"
[3]: https://opencode.ai/docs/providers/ "Providers | opencode"
[4]: https://opencode.ai/docs/config/ "Config | opencode"
[5]: https://opencode.ai/docs/enterprise/ "Enterprise | opencode"
[6]: https://opencode.ai/docs "Intro | opencode"
[7]: https://opencode.ai/docs/permissions/ "Permissions | opencode"
[8]: https://opencode.ai/docs/agents/ "Agents | opencode"
[9]: https://opencode.ai/docs/lsp/ "LSP Servers | opencode"
[10]: https://opencode.ai/docs/mcp-servers/ "MCP servers | opencode"
[11]: https://opencode.ai/docs/cli/ "CLI | opencode"
[12]: https://opencode.ai/docs/github/ "GitHub | opencode"
[13]: https://opencode.ai/docs/share/ "Share | opencode"
[14]: https://github.com/sst/opencode "GitHub - sst/opencode: AI coding agent, built for the terminal."
