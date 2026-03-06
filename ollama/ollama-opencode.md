# Use Ollama to Power OpenCode — PlebDevs Integration

> Run models with **Ollama** locally and point **OpenCode** at Ollama's OpenAI-compatible API. Keep coding prompts on-device and avoid API spend.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Start Ollama locally](#1-start-ollama-locally)
  - [Configure OpenCode](#2-configure-opencode)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Project-local config](#project-local-config)
  - [Hybrid local-and-cloud setup](#hybrid-local-and-cloud-setup)
  - [Operational tips](#operational-tips)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

OpenCode can connect to any OpenAI-compatible provider. Ollama exposes one at `http://localhost:11434/v1`, so you can run OpenCode against a fully local model while keeping the same TUI workflow, project rules, and file-editing flow.

**Why Ollama for OpenCode?**

- Simple local setup with one background server
- No per-token API cost after the model is downloaded
- Good fit for private repos or offline-ish workflows
- Easy model switching without rebuilding config from scratch

---

## Setup

### 1) Start Ollama locally

```bash
ollama --version
ollama serve
ollama pull qwen3.5:3b
```

Quick API checks:

```bash
curl http://localhost:11434/api/version
curl http://localhost:11434/v1/models
```

If `ollama serve` is already managed by the desktop app or a service, you only need to make sure the server is listening on `localhost:11434`.

### 2) Configure OpenCode

**Option A: Global config**

Create or edit `~/.config/opencode/opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3.5:3b": {
          "name": "Qwen 3.5 3B (local)"
        }
      }
    }
  },
  "model": "ollama/qwen3.5:3b",
  "small_model": "ollama/qwen3.5:3b"
}
```

**Option B: Project-local config**

Create `./opencode.json` in the repo you want OpenCode to use. OpenCode merges it on top of your global config, which is useful when only some projects should default to Ollama.

**TUI flow**

1. Launch OpenCode: `opencode`
2. Run `/models`
3. Select `ollama/qwen3.5:3b`

---

## Beginner usage

1. Confirm Ollama is serving: `curl http://localhost:11434/api/version`
2. Confirm your model exists: `ollama ls`
3. Open your repo and launch OpenCode: `opencode`
4. Run `/init` if the project does not already have `AGENTS.md`
5. Use `/models` to select your Ollama-backed model
6. Start with a simple repo question or small edit request

**Quick first run**

```bash
# Terminal 1
ollama serve

# Terminal 2
cd your-project
opencode
```

Inside OpenCode:

- `/models` → choose `ollama/qwen3.5:3b`
- Ask: `Summarize this codebase and point out the highest-risk area`

---

## Pro usage

### Project-local config

Use a per-project `opencode.json` when you want one repo pinned to local models without changing your global default:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3.5:3b": { "name": "Qwen 3.5 3B (local)" },
        "qwen3.5:7b": { "name": "Qwen 3.5 7B (local)" }
      }
    }
  },
  "model": "ollama/qwen3.5:7b",
  "small_model": "ollama/qwen3.5:3b",
  "share": "disabled"
}
```

This gives you a larger default local model for coding and a smaller one for lightweight tasks.

### Hybrid local-and-cloud setup

Keep a cloud model available for difficult refactors while making Ollama your cheap default:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3.5:3b": { "name": "Qwen 3.5 3B (local)" }
      }
    }
  },
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "ollama/qwen3.5:3b"
}
```

Use `/models` to switch when a task needs more reasoning depth.

### Operational tips

- Keep Ollama and OpenCode on the same machine to minimize latency.
- Pull models before you need them so OpenCode does not stall on first use.
- Use smaller local models for repo exploration and repetitive edits.
- If responses suddenly fail, verify both `http://localhost:11434/api/version` and `http://localhost:11434/v1/models`.

---

## Cost savings guide

- After the initial model download, local usage has no per-token API bill.
- Use Ollama for exploration, codebase summaries, and draft edits.
- Reserve cloud models for final review or harder architectural work.
- Set `small_model` to your Ollama model so cheap background tasks stay local.

---

## Privacy guide

- Local Ollama keeps prompts and code on your machine by default.
- In OpenCode, set `"share": "disabled"` if you do not want session sharing available.
- Disable unused remote providers for stricter local-only workflows:

```jsonc
{
  "share": "disabled",
  "disabled_providers": ["openai", "anthropic", "gemini"]
}
```

- Remember that OpenCode project rules, transcripts, and repo files still live on your own disk.

---

## Security guide

- Keep Ollama bound to `127.0.0.1`; do not expose port `11434` to the public internet.
- If you ever proxy Ollama beyond localhost, add authentication and network controls in front of it.
- Treat local models as code-execution-adjacent infrastructure: review agent permissions and keep destructive actions on approval when working in sensitive repos.
- Re-check your active model with `/models` before making broad repo changes.

---

## Appendix

**Quick refs**

```bash
ollama ls
ollama show qwen3.5:3b
curl http://localhost:11434/v1/models
opencode
```

See also: `./ollama-cli.md`, `./ollama-desktop.md`, `../opencode/opencode.md`, and `../llamacpp/llamacpp-opencode.md`.
