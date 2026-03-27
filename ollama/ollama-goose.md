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

Ollama serves models on `http://localhost:11434` (native API under `/api`, OpenAI‑compatible endpoints under `/v1/`). Goose can connect via Goose’s **Ollama** provider (recommended) using your local model name.

<img width="1524" height="1086" alt="image" src="https://github.com/user-attachments/assets/a4eb4c23-409e-47c5-aa46-0eb8bb7fc0fe" />

---

## Setup

Ollama side

```bash
ollama --version
ollama serve
ollama pull qwen3.5:4b
```

Quick API check

```bash
curl http://localhost:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen3.5:4b","prompt":"hello","stream":false}'
```

Goose side

- Desktop: configure the **Ollama** provider and set API Host to `http://localhost:11434`, then select a tool-calling-capable model (e.g., `qwen3.5:4b`).
- CLI (one-off via env vars):

```bash
export GOOSE_PROVIDER=ollama
export OLLAMA_HOST=http://localhost:11434
export GOOSE_MODEL=qwen3.5:4b
goose session --name local-ollama
```

---

## Beginner usage

1) Confirm Ollama is serving: `curl http://localhost:11434/api/version`
2) Ensure your model is pulled: `ollama pull qwen3.5:4b`
3) Create a new Goose session using the **Ollama** provider.
4) Prompt once; verify latency and correctness.

<img width="1400" height="972" alt="image" src="https://github.com/user-attachments/assets/58be1972-44cf-4c7b-b3a1-cfab2d8c8ea6" />

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
ollama ls
ollama show qwen3.5:4b
goose session --name local-ollama
```

See also: `./ollama-cli.md`, `./ollama-desktop.md`, and `../goose/goose-ollama.md`.
