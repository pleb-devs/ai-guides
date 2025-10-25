# Use Ollama to Power Goose — PlebDevs Integration

> Run models with **Ollama** locally and point **Goose** at your Ollama server. Keep prompts on-device and avoid API spend.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

Ollama serves models on `http://localhost:11434`. Goose can connect via Ollama’s OpenAI‑compatible interface at `/v1` using your local model name.

---

## Setup

Ollama side

```bash
ollama --version
ollama serve
ollama run llama3
```

Quick API check

```bash
curl http://localhost:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model":"llama3","prompt":"hello"}'
```

Goose side

- Desktop: Settings → Providers → Add OpenAI‑compatible provider → Base URL `http://localhost:11434/v1` → API key any string → Model `llama3`.
- CLI:

```bash
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_API_KEY=not-needed
export GOOSE_MODEL=llama3
goose session --name local-ollama
```

---

## Beginner usage

1) Confirm `ollama run llama3` is active.
2) Create a new Goose session using the OpenAI‑compatible provider.
3) Prompt once; verify latency and correctness.

---

## Pro usage

- Pin a custom Modelfile build (e.g., `my-code-fixer`) and select it in Goose.
- For automation, export the env vars above in CI and call `goose run`.
- Keep Ollama and Goose on the same machine to minimize latency.

---

## Cost savings guide

- Choose smaller/quantized models for common tasks.
- Reuse a single local Ollama host across multiple sessions.

---

## Privacy guide

- Leave Ollama bound to localhost; do not forward ports to the internet.
- Review transcripts and redact before sharing.

---

## Security guide

- If you must bind beyond localhost, add firewall rules and a reverse proxy with auth.
- In Goose, prefer Manual/Approve for write‑capable tools.

---

## Appendix

**Quick refs**

```bash
ollama list
ollama show llama3
goose session --name local-ollama
```

See also: `./ollama-cli.md`, `./ollama-desktop.md`, and `../goose/goose-ollama.md`.
