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

```bash
ollama serve   # starts the API at http://localhost:11434
```

### 3) Pull a starter model

```bash
ollama run llama3   # downloads (first run) and starts serving the model
```

Confirm the API is live:

```bash
curl http://localhost:11434/api/generate \
  -d '{"model":"llama3","prompt":"Say hello"}'
```

<img width="962" height="710" alt="image" src="https://github.com/user-attachments/assets/67127c7e-e82c-4934-bf65-b4887ca9167a" />

To point an agent (e.g., Goose) at Ollama, use `http://localhost:11434` and the exact model name (e.g., `llama3`). Many tools also support an OpenAI‑compatible base URL at `http://localhost:11434/v1`.

---

## Beginner usage

1) Start the server: `ollama serve`
2) Pull & run: `ollama run llama3`
3) Sanity check with curl (see above)
4) In your agent, select the local provider and set model=`llama3` (or use OpenAI‑compatible mode with base URL `http://localhost:11434/v1`).

---

## Pro usage

### Manage models

```bash
ollama list                 # installed models
ollama pull mistral         # download but don’t run
ollama show llama3          # details (size, families)
ollama rm llama3:latest     # remove when space is tight
```

### Custom models (Modelfile)

Create a `Modelfile` to pin a base model and parameters:

```text
FROM llama3
PARAMETER temperature 0.2
SYSTEM You are a concise assistant for code reviews.
```

Build and run:

```bash
ollama create -f Modelfile my-local-code
ollama run my-local-code
```

### Run as a background service

- macOS/Windows: use the standard OS facilities to start `ollama serve` at login.
- Linux (systemd):

```bash
sudo systemctl enable ollama
sudo systemctl start ollama
journalctl -u ollama -f
```

### LAN access (use sparingly)

Bind beyond localhost only on trusted networks:

```bash
export OLLAMA_HOST=0.0.0.0:11434
ollama serve
```

Prefer localhost for security; if exposing, place behind a reverse proxy + auth and restrict by firewall.

### Docker hosting

```bash
docker run -d --name ollama -p 11434:11434 ollama/ollama
docker exec -it ollama ollama run llama3
```

---

## Cost savings guide

- Choose smaller/quantized variants that match your hardware.
- Pre‑pull frequently used models; avoid repeated downloads.
- Run one model at a time on low‑RAM machines to prevent swapping.

---

## Privacy guide

- After download, inference stays local; no prompts leave your device.
- Keep transcripts in your agent; Ollama does not store chat history.
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
ollama list
ollama pull <model>
ollama show <model>
ollama rm <model>
ollama create -f Modelfile <name>
```

**Key paths**

- Models/cache live under your user data directory (commonly `~/.ollama/`).

---

## Troubleshooting quick hits

- Port 11434 in use → stop other services or change `OLLAMA_HOST`.
- Slow first token → first request warms the model; subsequent calls speed up.
- Out of memory → choose a smaller/quantized model or close other GPU tasks.
- Client can't connect → confirm server is running and the base URL (native vs `/v1`).
