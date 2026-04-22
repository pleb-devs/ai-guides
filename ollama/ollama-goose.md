# Use Ollama to Power Goose - PlebDevs Integration

> Run models with **Ollama** locally and point **Goose** at Ollama's local provider. Keep coding prompts on-device and avoid API spend.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Start Ollama locally](#1-start-ollama-locally)
  - [Configure Goose](#2-configure-goose)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Persistent CLI config](#persistent-cli-config)
  - [Tool shim for non-tool-calling models](#tool-shim-for-non-tool-calling-models)
  - [Operational tips](#operational-tips)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

Goose supports an Ollama provider for local models running on `http://localhost:11434`. That lets you keep the same Goose workflow, approvals, extensions, and repo context while running a local model instead of paying for a cloud API on every prompt.

**Why Ollama for Goose?**

- Simple local setup with one background server
- No per-token API cost after the model is downloaded
- Good fit for private repos or offline-ish workflows
- Easy model switching without rebuilding your workflow

---

## Setup

### 1) Start Ollama locally

```bash
ollama --version
ollama serve
ollama pull qwen2.5
```

Quick checks:

```bash
curl http://localhost:11434/api/version
ollama ls
```

If `ollama serve` is already managed by the desktop app or a service, you only need to make sure the server is listening on `localhost:11434`.

### 2) Configure Goose

**Option A: Goose Desktop**

Open Goose, go to provider setup, choose `Ollama`, leave the host as `http://localhost:11434`, and select an installed local model such as `qwen2.5`.

**Option B: Goose CLI**

Run the interactive setup once:

```bash
goose configure
```

Or launch a session with explicit local-provider environment variables:

```bash
export GOOSE_PROVIDER=ollama
export OLLAMA_HOST=http://localhost:11434
export GOOSE_MODEL=qwen2.5
export GOOSE_MODE=approve
goose session --name local-ollama
```

Goose stores shared Desktop and CLI configuration under `~/.config/goose/config.yaml`, so you only need to configure this once per machine.

---

## Beginner usage

1. Confirm Ollama is serving: `curl http://localhost:11434/api/version`
2. Confirm your model exists: `ollama ls`
3. Open your repo and launch Goose Desktop or start a CLI session
4. Start in `approve` mode if the repo has write access
5. Begin with a simple repo question or a small reversible edit request

**Quick first run**

```bash
# Terminal 1
ollama serve

# Terminal 2
cd your-project
goose session --name local-ollama
```

Inside Goose:

- Ask: `Summarize this codebase and point out the highest-risk area`
- Then ask: `Propose one small safe refactor and tell me how to verify it`

---

## Pro usage

### Persistent CLI config

If you want Ollama to stay your default Goose provider, keep it in `~/.config/goose/config.yaml`:

```yaml
GOOSE_PROVIDER: ollama
OLLAMA_HOST: http://localhost:11434
GOOSE_MODEL: qwen2.5
GOOSE_MODE: approve
```

This gives you a durable local default for future Desktop and CLI sessions on the same machine.

### Tool shim for non-tool-calling models

Goose works best with models that support tool calling. If you want to experiment with a model that does not, Goose has an experimental Ollama tool shim:

```bash
ollama pull mistral-nemo

export GOOSE_TOOLSHIM=true
export GOOSE_TOOLSHIM_OLLAMA_MODEL=mistral-nemo
goose session --name local-toolshim
```

If tool use feels unreliable with local models, increase the Ollama server context window before starting Goose:

```bash
OLLAMA_CONTEXT_LENGTH=32768 ollama serve
```

### Operational tips

- Keep Ollama and Goose on the same machine to minimize latency.
- Pull models before you need them so Goose does not stall on first use.
- Use smaller local models for repo exploration, summaries, and repetitive edits.
- Use `.gooseignore` to fence secrets, generated files, or infrastructure paths you do not want Goose to touch.
- If Goose misses project hints or tool context, raise the available input budget:

```bash
export GOOSE_INPUT_LIMIT=32000
```

---

## Cost savings guide

- After the initial model download, local usage has no per-token API bill.
- Use Ollama for repo exploration, codebase summaries, and draft edits.
- Reserve cloud providers for final review or harder architectural work.
- Keep a smaller local model as your default when you want cheap, fast iteration.

---

## Privacy guide

- Local Ollama keeps prompts and repo context on your machine by default.
- Goose config, transcripts, and exported sessions still live on your disk, so review them before sharing.
- Prefer local extensions and local providers when you want a stricter on-device workflow.
- On shared machines, avoid leaving unused cloud provider keys configured if your goal is local-only work.

---

## Security guide

- Keep Ollama bound to `127.0.0.1`; do not expose port `11434` to the public internet.
- In Goose, prefer `approve` or `smart_approve` when working in sensitive repos.
- Use `.gooseignore` to keep secrets, deployment files, and credentials out of the agent's working set.
- If you ever proxy Ollama beyond localhost, put authentication and network controls in front of it.

---

## Appendix

**Quick refs**

```bash
ollama ls
ollama show qwen2.5
curl http://localhost:11434/api/version
goose configure
goose session --name local-ollama
```

See also: `./ollama-cli.md`, `./ollama-desktop.md`, `../goose/goose-cli.md`, `../goose/goose-desktop.md`, and `../llamacpp/llamacpp-goose.md`.
