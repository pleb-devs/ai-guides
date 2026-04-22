# Configure Goose with a Local Ollama Provider - PlebDevs Integration

> Connect Goose to models hosted by your local **Ollama** server at `http://localhost:11434`. Keep repo context on-device and run fully local when you want privacy, lower cost, or offline-ish workflows.

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

Goose includes an Ollama provider for local models served from `http://localhost:11434`. That means you can keep the same Goose Desktop or CLI workflow, approval modes, extensions, and project context while running a model on your own machine.

**Why connect Goose to Ollama?**

- Simple local setup with one background server
- No per-token API cost after the model is downloaded
- Good fit for private repos or offline-ish workflows
- Easy model switching as your hardware or task changes

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

### 2) Configure Goose

**Desktop**

Open Goose, configure the `Ollama` provider, keep the host as `http://localhost:11434`, and choose an installed local model such as `qwen2.5`.

**CLI**

Run the interactive setup:

```bash
goose configure
```

Or launch Goose with explicit local-provider environment variables:

```bash
export GOOSE_PROVIDER=ollama
export OLLAMA_HOST=http://localhost:11434
export GOOSE_MODEL=qwen2.5
export GOOSE_MODE=approve
goose session --name local-ollama
```

Goose stores shared Desktop and CLI configuration in `~/.config/goose/config.yaml`.

---

## Beginner usage

1. Verify Ollama is serving: `curl http://localhost:11434/api/version`
2. Confirm the model exists locally: `ollama ls`
3. Start Goose with the Ollama provider selected
4. Begin in `approve` mode if the repo has write access
5. Start with a simple repo question or a small reversible task

**Quick first run**

```bash
# Terminal 1
ollama serve

# Terminal 2
cd your-project
goose session --name local-ollama
```

First prompts to try:

- `Summarize this codebase and point out the highest-risk area`
- `Propose one small safe refactor and tell me how to verify it`

---

## Pro usage

### Persistent CLI config

If you want Ollama to remain your default provider:

```yaml
GOOSE_PROVIDER: ollama
OLLAMA_HOST: http://localhost:11434
GOOSE_MODEL: qwen2.5
GOOSE_MODE: approve
```

Keep that in `~/.config/goose/config.yaml` so future Desktop and CLI sessions default to local inference.

### Tool shim for non-tool-calling models

Goose is strongest with models that support tool calling. If you want to test a model that does not, Goose includes an experimental Ollama tool shim:

```bash
ollama pull mistral-nemo

export GOOSE_TOOLSHIM=true
export GOOSE_TOOLSHIM_OLLAMA_MODEL=mistral-nemo
goose session --name local-toolshim
```

If local tool use becomes unreliable, start Ollama with a larger context window:

```bash
OLLAMA_CONTEXT_LENGTH=32768 ollama serve
```

### Operational tips

- Keep Ollama and Goose on the same machine to minimize latency.
- Pull models before you need them so Goose does not pause on first use.
- Use smaller local models for exploration, summaries, and repetitive edits.
- Add `.gooseignore` to fence secrets, generated files, and infrastructure paths.
- If Goose misses project instructions or tool context, raise the available input budget:

```bash
export GOOSE_INPUT_LIMIT=32000
```

---

## Cost savings guide

- After the initial model download, local usage has no per-token API bill.
- Use Ollama for repo exploration, summaries, and draft edits.
- Reserve cloud providers for final review or harder architectural work.
- Keep a smaller local model as your default for cheap, fast iteration.

---

## Privacy guide

- With Ollama on localhost, prompts and repo context stay on-device by default.
- Goose config, transcripts, and exported sessions still live on your disk, so review them before sharing.
- Prefer local providers and local extensions when you want a stricter on-device workflow.
- On shared machines, avoid leaving unused cloud provider keys configured if your goal is local-only use.

---

## Security guide

- Keep Ollama bound to `127.0.0.1`; do not expose port `11434` to the public internet.
- In Goose, prefer `approve` or `smart_approve` for sensitive repos.
- Use `.gooseignore` to keep secrets, credentials, and deployment files out of the agent working set.
- If you ever proxy Ollama beyond localhost, add authentication and network controls in front of it.

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

See also: `./goose-cli.md`, `./goose-desktop.md`, `../ollama/ollama-cli.md`, `../ollama/ollama-desktop.md`, and `../llamacpp/llamacpp-goose.md`.
