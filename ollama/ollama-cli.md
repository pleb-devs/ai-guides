# Ollama CLI — PlebDevs AI Guide

> Operate **Ollama** from the terminal to install, serve, and manage local models for other agents (Goose, editors, scripts). Emphasis: reliability, API checks, and model lifecycle.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Verify install](#1-verify-install)
  - [Start the server](#2-start-the-server)
  - [Pull and run a model](#3-pull-and-run-a-model)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [API checks (curl)](#api-checks-curl)
  - [Model lifecycle](#model-lifecycle)
  - [Custom models with Modelfile](#custom-models-with-modelfile)
  - [OpenAI‑compatible clients](#openai-compatible-clients)
  - [Headless service patterns](#headless-service-patterns)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)
- [Troubleshooting quick hits](#troubleshooting-quick-hits)

---

## Overview

Ollama’s CLI is the control surface for running models locally and exposing a simple HTTP API on `localhost:11434`. Use it to power agents without sending data to external APIs.

---

## Setup

### 1) Verify install

```bash
ollama --version
```

### 2) Start the server

```bash
ollama serve   # runs the HTTP API
```

### 3) Pull and run a model

```bash
ollama run llama3   # downloads (if needed) and serves the model
```

---

## Beginner usage

1) `ollama serve`
2) `ollama run llama3`
3) Point your agent to `http://localhost:11434` and set model=`llama3`.

---

## Pro usage

### API checks (curl)

```bash
# Generate (single prompt)
curl http://localhost:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model":"llama3","prompt":"Hello from Ollama"}'

# Chat (multi‑turn)
curl http://localhost:11434/api/chat \
  -H 'Content-Type: application/json' \
  -d '{"model":"llama3","messages":[{"role":"user","content":"Summarize: local model hosting"}]}'
```

### Model lifecycle

```bash
ollama list                 # installed models
ollama pull <model>         # download without running
ollama show <model>         # metadata, families, size
ollama rm <model[:tag]>     # free disk space
```

### Custom models with Modelfile

```text
# Modelfile
FROM llama3
PARAMETER temperature 0.1
SYSTEM You are a terse code fixer.
```

```bash
ollama create -f Modelfile my-code-fixer
ollama run my-code-fixer
```

### OpenAI‑compatible clients

Many tools can talk to Ollama via OpenAI‑style endpoints. Point them at `http://localhost:11434/v1` and select your local model name.

```bash
# Example: Node client using OPENAI_BASE_URL
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_API_KEY=not-needed
node your_client.js
```

### Headless service patterns

- Linux systemd:

```bash
sudo systemctl enable ollama
sudo systemctl start ollama
journalctl -u ollama -f
```

- Docker (isolated):

```bash
docker run -d --name ollama -p 11434:11434 ollama/ollama
docker exec -it ollama ollama run llama3
```

- Bind to a different interface/port if needed:

```bash
export OLLAMA_HOST=0.0.0.0:11434
ollama serve
```

---

## Cost savings guide

- Prefer smaller/quantized models for routine tasks.
- Pre‑pull models in CI images to eliminate cold starts.
- Run one heavy model at a time on limited hardware.

---

## Privacy guide

- After download, inference runs locally; keep agents pointed at `localhost`.
- Avoid exposing the API to the internet; disable telemetry in dependent tools.

---

## Security guide

- Keep the API bound to `127.0.0.1` by default.
- If you must expose on LAN, use a reverse proxy with auth and strict firewall rules.

---

## Appendix

**Common commands**

```bash
ollama serve
ollama run <model>
ollama list
ollama pull <model>
ollama show <model>
ollama rm <model>
ollama create -f Modelfile <name>
```

**Key env**

```bash
OLLAMA_HOST=127.0.0.1:11434   # bind/port
```

---

## Troubleshooting quick hits

- Connection refused → start `ollama serve` and retry.
- Model won’t load → not enough RAM/VRAM; choose a smaller variant.
- High latency → first run compiles/warms; subsequent calls improve.
- Client schema mismatch → try the OpenAI‑compatible base URL at `/v1`.

