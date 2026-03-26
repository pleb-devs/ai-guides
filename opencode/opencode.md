# OpenCode (CLI Coding Agent)

![OpenCode terminal UI](https://github.com/anomalyco/opencode/raw/dev/packages/web/src/assets/lander/screenshot.png)

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
  - [CLI, IDE & headless server](#cli-ide--headless-server)
  - [GitHub & GitLab automation](#github--gitlab-automation)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** OpenCode is an **open-source AI coding agent built for the terminal**. It analyzes, edits, and refactors your codebase with LLMs, is **provider-agnostic** (75+ providers via the AI SDK and Models.dev), supports **local models**, integrates with **LSP servers** for live diagnostics, and offers **shareable sessions**, **GitHub/GitLab automation**, and a **headless server/API**. It also powers a **web UI** and IDE integrations, but this guide focuses on the CLI/TUI. ([Opencode][1], [Opencode][3])

**Key ideas**

- **Native terminal UI** with Plan/Build modes, file references (`@`), shell commands (`!`), and slash-commands (`/connect`, `/models`, `/undo`, `/redo`, `/share`, `/compact`, `/details`, `/themes`, etc.). ([Opencode][2])
- **Models & providers**: connect any major API (Anthropic, OpenAI, Bedrock, Groq, xAI, etc.) or local inference (e.g. LM Studio) via providers; **OpenCode Zen** and **OpenCode Go** are first-class paths, and OpenAI/GitHub Copilot subscriptions are supported too. ([Opencode][3])
- **Config-driven**: JSON/JSONC config with merged global and per-project files; explicit controls for **permissions**, **MCP servers**, **LSP**, **commands**, **formatters**, **autoupdate**, and **share** modes, plus separate `tui.json` files for **themes** and keybinds. ([Opencode][4], [Opencode][2])
- **Privacy posture**: project code is not stored by OpenCode; **sharing is opt-in** and can be disabled globally or per-project. ([Opencode][5])

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
brew install anomalyco/tap/opencode
brew install opencode

# Node toolchains
npm install -g opencode-ai@latest
bun install -g opencode-ai@latest
pnpm install -g opencode-ai@latest
yarn global add opencode-ai

# Arch
sudo pacman -S opencode
paru -S opencode-bin

# Windows
choco install opencode
scoop install opencode

# Other options
mise use -g opencode
nix run nixpkgs#opencode
```

See the official Install docs for the latest options and platforms. ([Opencode][1])

### 2) Pick a provider (fast path: **OpenCode Zen**)

OpenCode supports 75+ providers through the **AI SDK** and **Models.dev**; credentials are stored locally at `~/.local/share/opencode/auth.json`. For a zero-friction start:

1. Launch the TUI in any repo:

   ```bash
   opencode
   ```

2. Run `/connect` and choose **OpenCode Zen** to use the curated model list. Follow the browser flow at `opencode.ai/auth`, create/copy your API key, and then select a model with `/models`. If you want OpenCode’s lower-cost hosted subscription instead, **OpenCode Go** uses the same flow. ([Opencode][3])

You can also connect providers directly from the TUI (`/connect → Anthropic → Claude Pro/Max`, OpenAI, etc.) or via the CLI:

```bash
opencode auth login   # same auth flow as /connect, in the terminal
```

([Opencode][3], [Opencode][11])

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

- Start OpenCode in your repo, **ask questions**, and **iterate**.
- Use **Plan** mode first (reads, plans, and asks before edits), then **Build** to apply changes. Toggle with **Tab**. ([Opencode][6], [Opencode][7])

### Core moves (TUI)

- **Reference files** with `@` (fuzzy finds + inlines contents).
  *"How is auth handled in @packages/functions/src/api/index.ts?"* ([Opencode][2])
- **Run shell** with `!` (captured into context).
  *`!ls -la`* ([Opencode][2])
- **Connect providers** with `/connect`, pick models with `/models`, and toggle tool details with `/details`. ([Opencode][2], [Opencode][3])
- **Undo / redo** code & messages (`/undo`, `/redo`)—uses Git under the hood. ([Opencode][2])
- **Share / unshare** a session (`/share`, `/unshare`, manual by default). ([Opencode][13])
- **Compress long threads** with `/compact` (or `/summarize`) to keep context small. ([Opencode][2])

### A simple first flow

1. **Explain** the repo: “Give me a quick summary of the codebase.”
2. **Plan** a change: switch to Plan; describe the feature.
3. **Review plan**; add details (you can **drag-drop images** into the terminal).
4. **Build**: toggle back, ask it to implement.
5. **Undo/redo** if needed. ([Opencode][6])

---

## Pro usage

### Declarative config (global & per-project)

- Global runtime config: `~/.config/opencode/opencode.json`
- Global TUI config: `~/.config/opencode/tui.json`
- Project runtime config: `./opencode.json` (merged on top of global)
- Project TUI config: `./tui.json`
- Override runtime path with `OPENCODE_CONFIG=/path/to/config.json`
- Override TUI path with `OPENCODE_TUI_CONFIG=/path/to/tui.json`
- Override config directory with `OPENCODE_CONFIG_DIR=/path/to/dir`
- OpenCode also layers remote org defaults from `.well-known/opencode` plus `.opencode` directories for agents, commands, plugins, and related assets.
  Supports **JSON/JSONC** with schemas at `https://opencode.ai/config.json` and `https://opencode.ai/tui.json`. ([Opencode][4])

**Snippets you’ll reuse**

```jsonc
// Select models, set a cheaper small_model, and pin updates
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5",
  "autoupdate": false
}
```

([Opencode][4])

```jsonc
// Put theme and keybind customization in tui.json
{
  "$schema": "https://opencode.ai/tui.json",
  "theme": "opencode"
}
```

([Opencode][2], [Opencode][4])

```jsonc
// Lock down agent capabilities & bash
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "edit": "ask",
    "bash": {
      "*": "allow",
      "git push *": "ask",
      "terraform *": "deny",
      "kubectl *": "deny"
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

- Define **agents** in JSON or Markdown (eg. `~/.config/opencode/agents/*.md`) with custom prompts, models, and permissions/tool access.
- Generate or evolve `AGENTS.md` with `/init` and include **instructions** via `instructions: [...]` and glob patterns.
- OpenCode merges **project** and **global** rules and uses the Plan/Build modes plus permissions as the default safety baseline. ([Opencode][7], [Opencode][8])

### LSP, MCP, and tools

- OpenCode auto-starts LSP servers (TS, Go, Rust, Python, etc.) and can **disable LSP downloads** via `OPENCODE_DISABLE_LSP_DOWNLOAD=true`. You can add or override servers in config. ([Opencode][9])
- Add **MCP servers** (local or remote) to expose custom tools to the LLM; enable them globally or **per-agent** with wildcards. ([Opencode][10])

### CLI, IDE & headless server

- **CLI non-interactive**: `opencode run "…"` with flags (`--model`, `--agent`, `--session`, `--share`, `--attach`).
- **Headless HTTP**: `opencode serve` exposes an OpenAPI-backed HTTP server; `opencode web` runs the browser/mobile client against the same backend.
- **IDE/ACP**: the IDE extension auto-installs when you run `opencode` in supported integrated terminals, and `opencode acp` is available for ACP-compatible clients.
- **Attach & upgrade**: `opencode attach http://host:4096` connects a TUI to an existing backend, and `opencode upgrade` updates the CLI. ([Opencode][11], [Opencode][17], [Opencode][18])

### GitHub & GitLab automation

- Mention `/opencode` or `/oc` in GitHub issues/PRs; the agent runs **inside your GitHub Action runner**, creating branches/PRs as requested. Install via `opencode github install` or add the workflow manually. ([Opencode][12])
- On GitLab, mention `@opencode` in GitLab Duo comments or wire OpenCode into CI/CD with the published component and a stored `auth.json`. ([Opencode][15])

---

## Cost savings guide

**Pick cost-effective access**

- **OpenCode Zen** is the easiest supported fast path, and **OpenCode Go** is OpenCode’s lower-cost hosted subscription option. ([Opencode][3])
- If you already pay for **ChatGPT Plus/Pro** or **GitHub Copilot**, connecting those subscriptions can be cheaper than raw API spend. ([Opencode][3])
- **Anthropic sign-in works**, but OpenCode’s docs note that using **Claude Pro/Max** subscriptions this way is **not officially supported by Anthropic**. ([Opencode][3])

**Right-size the model**

- Set a **cheaper `small_model`** for light tasks (titles, summaries) and keep your main model for builds. ([Opencode][4], [Opencode][7])
- Prefer **local models** for refactors/tests on private code when quality allows—\$0 in API fees, only local compute. See the **local models** docs for setup patterns. ([Opencode][3])

**Reduce unnecessary tokens**

- Work in **Plan mode** first (asking before edits), get the spec tight, then **Build**—this reduces trial-and-error iterations. ([Opencode][6], [Opencode][7])
- Use `/compact` to summarize long sessions and keep context lean. ([Opencode][2])
- Prefer `@file` references over pasting large blobs manually. ([Opencode][2])

**Avoid accidental spend**

- Lock to allowed providers/models with `disabled_providers` and explicit `model` strings. ([Opencode][4])
- Use CLI `--model` to pin a cheaper model in batch jobs, or wire your config to `{env:OPENCODE_MODEL}` if you prefer environment-based overrides. ([Opencode][4], [Opencode][11])

---

## Privacy guide

**What OpenCode stores**

- **OpenCode does not store your code or context data**; processing happens locally or directly with your LLM provider. The exception is the **optional Share feature**. ([Opencode][5], [Opencode][13])
- **Sharing** creates a public link and syncs the conversation until you `/unshare`. Set `"share": "disabled"` globally (or per-project) if needed. ([Opencode][13])

**Practical privacy steps**

- Prefer **local models** (Ollama / LM Studio) for sensitive code so nothing leaves your machine. ([Opencode][3])
- Route APIs through a **custom `baseURL`** (egress proxy / self-hosted gateway) in the provider config. ([Opencode][3])
- Keep credentials out of history: store keys via `opencode auth login` (saved at `~/.local/share/opencode/auth.json`) or use **config variable substitution** to pull from environment/files. ([Opencode][3])
- Consider disabling **automatic LSP downloads** in restricted environments: `OPENCODE_DISABLE_LSP_DOWNLOAD=true`. ([Opencode][9])
- **Hosted web search is not private**: if you enable a hosted MCP search provider (for example Exa's remote MCP at `https://mcp.exa.ai/mcp`), your queries are sent to that third party. Treat it as non‑private and disable it when privacy matters. ([Exa][16])
- **Private web search (SearXNG)**: run SearXNG locally and connect it via MCP. See the [OpenCode → SearXNG guide](./opencode-searxng.md) for the minimal setup.

**Team/enterprise**

- For trials, **disable sharing**; enterprise deployments can centralize config, integrate SSO, and restrict share links to SSO users. If self-hosted share pages matter, confirm availability before planning around them, because the enterprise docs still describe that path as roadmap work. ([Opencode][5], [Opencode][13])

---

## Security guide

**Principle of least privilege**

- **Permissions**: require approval for edits and shell, and pattern-block risky commands (`git push`, `terraform *`, `kubectl *`, etc.). Use `"ask"`/`"deny"` and wildcard maps. Consider also setting `webfetch`, `doom_loop`, and `external_directory` to `"deny"` in sensitive environments. ([Opencode][7])
- **Per-agent tool scopes**: give heavy-duty tools only to agents that truly need them. (Enable/disable tools or MCP servers per agent.) ([Opencode][10])

**Supply-chain hygiene**

- Pin OpenCode behavior by setting `"autoupdate": false` and updating on your schedule via `opencode upgrade`. ([Opencode][4])
- Use a **private NPM registry** and ensure developers are authenticated before running OpenCode if your org requires it. ([Opencode][5])

**Data boundaries**

- Keep **share** mode on `"manual"` (default) or `"disabled"` for sensitive repos; `/unshare` to remove links and data. ([Opencode][13])
- Prefer **local models** for regulated code paths; otherwise, restrict providers with `disabled_providers`. ([Opencode][4])

**Network egress control**

- Point providers at a **proxy** via `provider.*.options.baseURL` and control outbound traffic centrally. ([Opencode][3])

---

## Appendix

**Useful commands**

- `opencode` (launch TUI) • `/connect` • `/init` • `/models` • `/details` • `/themes` • `/undo` • `/redo` • `/share` • `/unshare` • `/compact` • `!<cmd>` • `@path/to/file` ([Opencode][2], [Opencode][3])
- `opencode auth login` • `opencode models` • `opencode run "…" --model … --agent …` • `opencode serve` • `opencode web` • `opencode attach http://localhost:4096` • `opencode acp` • `opencode upgrade` ([Opencode][11], [Opencode][18])

**Version signal**

- Active, fast-moving repo (MIT) with frequent releases; this guide was rechecked against the public docs on **March 26, 2026**, and the latest tagged release at review time was **v1.3.3** published on **March 26, 2026**. Always consult releases before pinning. ([GitHub][14], [Opencode][1])

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
[14]: https://github.com/anomalyco/opencode/releases/latest "Latest release | opencode"
[15]: https://opencode.ai/docs/gitlab/ "GitLab | opencode"
[16]: https://docs.exa.ai/reference/exa-mcp "Exa MCP"
[17]: https://opencode.ai/docs/ide/ "IDE | opencode"
[18]: https://opencode.ai/docs/server/ "Server | opencode"
