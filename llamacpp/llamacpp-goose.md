# Configure Goose with llama.cpp Server — PlebDevs Integration

> Connect Goose (Desktop or CLI) to models hosted by your local **llama-server** via the OpenAI-compatible API. Full local inference with zero API costs.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Start llama-server](#1-start-llama-server)
  - [Configure Goose](#2-configure-goose)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Model selection](#model-selection)
  - [Performance tuning](#performance-tuning)
  - [Multiple providers](#multiple-providers)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

Goose can use any OpenAI-compatible backend. llama-server provides exactly that—point Goose at `http://localhost:8080/v1` and it works like any cloud provider, but runs entirely on your machine.

**Why llama.cpp over Ollama for Goose?**

- More control over inference parameters
- Better memory efficiency with manual tuning
- Access to cutting-edge features and backends
- Direct quantization control

---

## Setup

### 1) Start llama-server

```bash
# Basic start
llama-server -m /path/to/model.gguf

# With GPU offload (recommended)
llama-server -m model.gguf -ngl 99 --port 8080

# Or download and serve directly
llama-server -hf bartowski/Qwen2.5-Coder-7B-Instruct-GGUF -ngl 99
```

Verify it's running:

```bash
curl http://localhost:8080/v1/models
```

### 2) Configure Goose

**Desktop**

1. Open Settings → Providers
2. Add a new OpenAI-compatible provider
3. Set:
   - **Name**: `llama.cpp (local)`
   - **Base URL**: `http://localhost:8080/v1`
   - **API Key**: `not-needed` (or any string)
   - **Model**: The name shown in `/v1/models` response
4. Create a session using this provider

**CLI**

```bash
# Set environment variables
export GOOSE_PROVIDER=openai
export OPENAI_HOST=http://localhost:8080
export OPENAI_BASE_PATH=v1/chat/completions
export OPENAI_API_KEY=not-needed
export GOOSE_MODEL=any  # Model name (ignored by llama-server)

# Start a session
goose session --name local-llama
```

Or configure interactively:

```bash
goose configure
# Select: Configure Providers
# Choose: OpenAI
# Enter base URL: http://localhost:8080/v1
# Enter API key: not-needed
```

---

## Beginner usage

1. Start llama-server with your model
2. Open Goose and create a new session with the llama.cpp provider
3. Test with a simple prompt: "What can you help me with?"
4. If it responds, you're set!

**Quick test flow:**

```bash
# Terminal 1: Start server
llama-server -m qwen2.5-coder-7b-Q4_K_M.gguf -ngl 99

# Terminal 2: Run Goose
export OPENAI_HOST=http://localhost:8080
export OPENAI_API_KEY=x
goose session
```

---

## Pro usage

### Model selection

For Goose (coding agent), use capable instruction-tuned models:

**Recommended models (as of 2025):**

- **Qwen2.5-Coder** (7B/14B/32B) — excellent code understanding
- **DeepSeek-Coder-V2** — strong reasoning
- **CodeLlama** variants — code-specialized
- **Mistral/Mixtral Instruct** — good general + code

**Quantization for agent work:**

- Prefer Q5_K_M or Q6_K for complex reasoning
- Q4_K_M works but may lose nuance on tricky tasks
- Avoid Q3 and below for agent use

### Performance tuning

Optimize llama-server for agent workloads:

```bash
# Larger context for code files
llama-server -m model.gguf -ngl 99 -c 8192

# Enable continuous batching (if handling concurrent requests)
llama-server -m model.gguf -ngl 99 --parallel 2 --cont-batching

# More threads for prompt processing
llama-server -m model.gguf -ngl 99 -t 8
```

**Context size guidance:**

- 4096: Basic tasks, short files
- 8192: Most coding tasks
- 16384+: Large files, multiple file context

### Multiple providers

Configure Goose with both cloud and local:

```bash
# ~/.config/goose/profiles.yaml (example structure)
providers:
  - name: claude-cloud
    type: anthropic
    api_key: ${ANTHROPIC_API_KEY}
  - name: llama-local
    type: openai
    base_url: http://localhost:8080/v1
    api_key: not-needed
```

Switch based on task:

- **Cloud**: Complex reasoning, production code reviews
- **Local**: Iteration, experimentation, sensitive code

---

## Cost savings guide

- **Zero API costs** — all inference on your hardware
- Use local models for iteration; switch to cloud for final review
- Run lighter models (7B) for simple tasks, heavier (14B+) for complex
- Batch your prompts when possible to amortize model load time

**Cost comparison (rough monthly estimates):**

| Provider | Light Use | Heavy Use |
| -------- | --------- | --------- |
| Claude API | $20-50 | $200+ |
| OpenAI API | $20-50 | $200+ |
| llama.cpp | $0 (electricity) | $0 (electricity) |

---

## Privacy guide

- All prompts stay on your machine—nothing sent externally
- Code never leaves your network
- No telemetry, no logging to cloud services
- Ideal for proprietary codebases and client work

**Goose privacy tips:**

- Use `.gooseignore` to exclude sensitive files from context
- Review what Goose sends in prompts (visible in verbose mode)
- For maximum privacy, run on an air-gapped machine

---

## Security guide

- Keep llama-server bound to `127.0.0.1` (default)
- If exposing on LAN:
  - Use `--api-key` flag on llama-server
  - Put behind reverse proxy with TLS
  - Firewall to trusted IPs only
- In Goose, use Manual/Approve modes for destructive tools
- Don't run Goose as root

**Secure LAN setup:**

```bash
# llama-server with API key
llama-server -m model.gguf --host 0.0.0.0 --api-key "your-secret-key"

# Goose config
export OPENAI_API_KEY=your-secret-key
export OPENAI_HOST=http://192.168.1.100:8080
```

---

## Appendix

**Environment variables for Goose + llama.cpp**

```bash
export GOOSE_PROVIDER=openai
export OPENAI_HOST=http://localhost:8080
export OPENAI_BASE_PATH=v1/chat/completions
export OPENAI_API_KEY=not-needed
export GOOSE_MODEL=local-model
```

**Troubleshooting**

| Issue | Solution |
| ----- | -------- |
| Connection refused | Start llama-server first |
| Slow responses | Add `-ngl 99` for GPU offload |
| Out of memory | Reduce context (`-c 2048`) or use smaller quant |
| Goose can't find model | Model name doesn't matter; check server is responding |
| Timeouts | Increase context size or reduce prompt length |

**See also**

- [llamacpp-server.md](./llamacpp-server.md) — full server reference
- [goose-ollama.md](../goose/goose-ollama.md) — Ollama alternative
- [ollama-goose.md](../ollama/ollama-goose.md) — if you prefer Ollama's simplicity
