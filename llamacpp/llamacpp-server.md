# llama.cpp Server — PlebDevs AI Guide

> Run **llama-server** to expose an OpenAI-compatible API for local inference. Connect any tool that speaks OpenAI's protocol—agents, editors, scripts—to your own hardware.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install llama-server](#1-install-llama-server)
  - [Start the server](#2-start-the-server)
  - [Verify with curl](#3-verify-with-curl)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Server flags](#server-flags)
  - [API endpoints](#api-endpoints)
  - [Multi-slot serving](#multi-slot-serving)
  - [GPU acceleration](#gpu-acceleration)
  - [Systemd service](#systemd-service)
  - [Docker deployment](#docker-deployment)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

`llama-server` is llama.cpp's built-in HTTP server. It provides:

- **OpenAI-compatible endpoints** (`/v1/chat/completions`, `/v1/completions`, `/v1/models`)
- **Multi-slot inference** (handle multiple requests concurrently)
- **Built-in WebUI** for testing
- **Streaming support** for real-time token generation

Any tool expecting an OpenAI API can connect by pointing `base_url` to your server.

---

## Setup

### 1) Install llama-server

Same installation as llama-cli—llama-server is included in all llama.cpp distributions.

```bash
# Pre-built binaries
# Download from: https://github.com/ggml-org/llama.cpp/releases

# Package managers
brew install llama.cpp    # Includes llama-server
pacman -S llama.cpp       # Arch

# From source
cmake -B build && cmake --build build --config Release -j
# Binary at: build/bin/llama-server
```

### 2) Start the server

```bash
# Basic start
llama-server -m model.gguf

# With GPU offload and custom port
llama-server -m model.gguf -ngl 99 --port 8080

# Download and serve from Hugging Face
llama-server -hf ggml-org/gemma-3-1b-it-GGUF
```

Default: listens on `http://127.0.0.1:8080`

### 3) Verify with curl

```bash
# Check server health
curl http://localhost:8080/health

# List models
curl http://localhost:8080/v1/models

# Test chat completion
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "any",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## Beginner usage

1. Start the server with your model
2. Point your client to `http://localhost:8080/v1`
3. Set API key to any non-empty string (server doesn't validate by default)

Example with Python OpenAI client:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="not-needed"
)

response = client.chat.completions.create(
    model="any",  # Model name is ignored; uses loaded model
    messages=[{"role": "user", "content": "Explain lightning network"}]
)
print(response.choices[0].message.content)
```

---

## Pro usage

### Server flags

```bash
llama-server -m <model.gguf>   # Model file (required)
  --host 0.0.0.0               # Bind address (LAN access)
  --port 8080                  # Port number
  -c 4096                      # Context size
  -ngl 99                      # GPU layers
  -t 8                         # CPU threads
  --parallel 4                 # Concurrent request slots
  --cont-batching              # Continuous batching (throughput)
  --api-key "secret"           # Require API key
  --embeddings                 # Enable embedding endpoint
  --metrics                    # Enable Prometheus metrics
```

### API endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Server health check |
| `GET /v1/models` | List loaded models |
| `POST /v1/chat/completions` | Chat completion (OpenAI format) |
| `POST /v1/completions` | Text completion |
| `POST /v1/embeddings` | Generate embeddings (if enabled) |
| `POST /completion` | Native llama.cpp completion |
| `POST /tokenize` | Tokenize text |
| `POST /detokenize` | Detokenize tokens |

**Chat completion request:**

```json
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "Hello"}
  ],
  "temperature": 0.7,
  "max_tokens": 500,
  "stream": true
}
```

**Native completion request:**

```json
{
  "prompt": "The capital of France is",
  "n_predict": 50,
  "temperature": 0.8,
  "stop": ["\n"]
}
```

### Multi-slot serving

Handle multiple concurrent requests:

```bash
# 4 parallel request slots
llama-server -m model.gguf --parallel 4 --cont-batching

# Monitor slot usage
curl http://localhost:8080/slots
```

Each slot maintains separate conversation state. More slots = more VRAM.

### GPU acceleration

```bash
# Full GPU offload
llama-server -m model.gguf -ngl 99

# Partial offload (limited VRAM)
llama-server -m model.gguf -ngl 30

# Multi-GPU
llama-server -m model.gguf -ngl 99 --tensor-split 0.5,0.5
```

### Systemd service

```ini
# /etc/systemd/system/llama-server.service
[Unit]
Description=llama.cpp Server
After=network.target

[Service]
Type=simple
User=llama
ExecStart=/usr/local/bin/llama-server \
  -m /models/mistral-7b-Q4_K_M.gguf \
  -ngl 99 \
  --host 127.0.0.1 \
  --port 8080 \
  --parallel 2
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable llama-server
sudo systemctl start llama-server
journalctl -u llama-server -f
```

### Docker deployment

```bash
# Official image (CPU)
docker run -p 8080:8080 \
  -v /path/to/models:/models \
  ghcr.io/ggml-org/llama.cpp:server \
  -m /models/model.gguf

# With CUDA
docker run --gpus all -p 8080:8080 \
  -v /path/to/models:/models \
  ghcr.io/ggml-org/llama.cpp:server-cuda \
  -m /models/model.gguf -ngl 99
```

---

## Cost savings guide

- Use **continuous batching** (`--cont-batching`) to improve throughput
- Start with **fewer parallel slots** and scale up based on demand
- Reduce context size (`-c`) for simpler use cases
- Use quantized models (Q4_K_M) to fit more in VRAM
- Run on consumer GPUs—llama.cpp is optimized for them

---

## Privacy guide

- All inference stays on your machine—no external API calls
- Bind to `127.0.0.1` to prevent network exposure
- Logs may contain prompts; configure log levels appropriately
- For multi-user deployments, consider per-user isolation

---

## Security guide

- **Never expose to the internet** without authentication
- Use `--api-key` flag to require bearer tokens
- For LAN access, put behind a reverse proxy with TLS and auth:

```nginx
# Nginx example
server {
    listen 443 ssl;
    server_name llm.local;

    location / {
        auth_basic "LLM Server";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://127.0.0.1:8080;
    }
}
```

- Rate limit requests to prevent abuse
- Run as a non-root user with minimal permissions

---

## Appendix

**Quick reference**

```bash
# Start server
llama-server -m model.gguf -ngl 99 --port 8080

# Test endpoints
curl http://localhost:8080/health
curl http://localhost:8080/v1/models
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"hi"}]}'
```

**Client configuration**

| Tool | Base URL | API Key |
|------|----------|---------|
| OpenAI Python | `http://localhost:8080/v1` | `"not-needed"` |
| Goose | `http://localhost:8080/v1` | Any string |
| OpenCode | `http://localhost:8080/v1` | Any string |
| LangChain | `http://localhost:8080/v1` | Any string |

**WebUI**

Open `http://localhost:8080` in a browser for the built-in chat interface.

**See also**

- [llamacpp-cli.md](./llamacpp-cli.md) — direct CLI usage
- [llamacpp-goose.md](./llamacpp-goose.md) — Goose integration
- [llamacpp-opencode.md](./llamacpp-opencode.md) — OpenCode integration
