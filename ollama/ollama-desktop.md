# Ollama Desktop — PlebDevs AI Guide

> Use **Ollama** on your workstation to install and host local LLMs that other agents (e.g., Goose) can call over `http://localhost:11434`. This guide focuses on dependable hosting, not chat UX.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install](#1-install)
  - [Start the local server](#2-start-the-local-server)
  - [Pull a starter model](#3-pull-a-starter-model)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Manage models](#manage-models)
  - [Custom models (Modelfile)](#custom-models-modelfile)
  - [Run as a background service](#run-as-a-background-service)
  - [LAN access (use sparingly)](#lan-access-use-sparingly)
  - [Docker hosting](#docker-hosting)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)
- [Troubleshooting quick hits](#troubleshooting-quick-hits)

---

## Overview

**What it is.** Ollama is a lightweight local model runner with a simple HTTP API. It’s ideal as a private, zero‑API‑cost backend for desktop agents and IDE tools.

**Why use it.**

- Keep prompts and outputs on your machine.
- Reuse the same local models across apps (Goose Desktop/CLI, editors, scripts).
- Control versions and performance per device.

<img width="808" height="610" alt="Screenshot 2025-10-27 at 6 57 34 AM" src="https://github.com/user-attachments/assets/f88f7e31-e1ca-4c39-bcab-8cc984ceea9e" />

---

## Setup

### 1) Install

- Install from the official site or your OS package manager.
- Verify: `ollama --version`

### 2) Start the local server

If you installed the Ollama desktop app, it runs the local server while the app is open. Use `ollama serve` mainly for headless/CLI setups.

```bash
ollama serve   # starts the API at http://localhost:11434 (native API base: /api)
```

### 3) Pull a starter model

```bash
ollama pull qwen3.5:3b  # download only
ollama run qwen3.5:3b   # downloads (first run) and opens an interactive chat
```

Confirm the API is live:

```bash
curl http://localhost:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen3.5:3b","prompt":"Say hello","stream":false}'
```

<img width="962" height="710" alt="image" src="https://github.com/user-attachments/assets/67127c7e-e82c-4934-bf65-b4887ca9167a" />

To point an agent (e.g., Goose) at Ollama, use `http://localhost:11434` and the exact model name (e.g., `qwen3.5:3b`). Many tools also support OpenAI‑compatible endpoints at `http://localhost:11434/v1/` (API key required by most clients, but ignored by Ollama).

---

## Beginner usage

1) Ensure the server is running (Ollama app open, or `ollama serve`)
2) Pull a model: `ollama pull qwen3.5:3b` (or `ollama run qwen3.5:3b` to chat)
3) Sanity check with curl (see above)
4) In your agent, select the local provider and set model=`qwen3.5:3b` (or use OpenAI‑compatible mode with base URL `http://localhost:11434/v1/`).

---

## Pro usage

### Manage models

```bash
ollama ls                   # installed models
ollama ps                   # running models
ollama pull qwen3.5:1.7b   # download another small variant
ollama show qwen3.5:3b     # details (size, families)
ollama stop qwen3.5:3b     # unload from memory
ollama rm qwen3.5:3b       # remove when space is tight
```

### Custom models (Modelfile)

Create a `Modelfile` to pin a base model and parameters:

```text
FROM qwen3.5:3b
PARAMETER temperature 0.2
SYSTEM You are a concise assistant for code reviews.
```

Build and run:

```bash
ollama create my-local-code -f Modelfile
ollama run my-local-code
```

### Run as a background service

- macOS/Windows: the Ollama desktop app runs the background server; enable “start at login” if you want it always on.
- Linux (systemd):

```bash
sudo systemctl enable ollama
sudo systemctl start ollama
journalctl -u ollama -f
```

### LAN access (use sparingly)

By default Ollama binds to `127.0.0.1:11434`. Only listen on LAN on trusted networks.

```bash
# If you're running `ollama serve` directly:
OLLAMA_HOST=0.0.0.0:11434 ollama serve
```

If you're running Ollama as an app/service (common), set `OLLAMA_HOST` for that service (macOS `launchctl setenv ...`, Linux `systemctl edit ollama.service`, Windows system env) and restart Ollama.

### Docker hosting

```bash
docker run -d --name ollama -p 11434:11434 -v ollama:/root/.ollama ollama/ollama
docker exec -it ollama ollama run qwen3.5:3b
```

---

## Cost savings guide

- Choose smaller/quantized variants that match your hardware.
- Pre‑pull frequently used models; avoid repeated downloads.
- Run one model at a time on low‑RAM machines to prevent swapping.

---

## Privacy guide

- For **local** models (`http://localhost:11434`), prompts stay on-device.
- If you use **Ollama cloud models** or **web search**, prompts are sent to `ollama.com` (content is processed but not stored per Ollama).
- For strict local-only, disable Ollama cloud features (`OLLAMA_NO_CLOUD=1` or `~/.ollama/server.json` with `"disable_ollama_cloud": true`) and restart Ollama.
- Ollama’s API is stateless; your client (Goose, editor, etc.) may store transcripts on disk.
- Avoid third‑party proxies unless you control them.

---

## Security guide

- Default to `127.0.0.1:11434`.
- If remote access is required, use TLS/auth at the proxy and restrict IPs.
- Keep your OS updated; treat model files as untrusted inputs only from trusted sources.

---

## Appendix

**Useful commands**

```bash
ollama serve
ollama run <model>
ollama ls
ollama ps
ollama pull <model>
ollama show <model>
ollama stop <model>
ollama rm <model>
ollama create <name> -f Modelfile
```

**Key paths**

- macOS: `~/.ollama/models/` (logs: `~/.ollama/logs/server.log`)
- Linux (systemd installer): `/usr/share/ollama/.ollama/models/` (logs: `journalctl -u ollama`)
- Windows: `C:\Users\%USERNAME%\.ollama\models` (logs: `%LOCALAPPDATA%\Ollama\server.log`)
- Docker: `/root/.ollama/` inside the container (mount a volume to persist)

---

## Troubleshooting quick hits

- Port 11434 in use → stop other services or change `OLLAMA_HOST`.
- Slow first token → first request warms the model; subsequent calls speed up.
- Out of memory → choose a smaller/quantized model or close other GPU tasks.
- Client can't connect → confirm server is running and the base URL (native vs `/v1`).
