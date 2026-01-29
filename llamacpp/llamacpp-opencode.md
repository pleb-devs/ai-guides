# Configure OpenCode with llama.cpp Server — PlebDevs Integration

> Connect OpenCode to models hosted by your local **llama-server** via the OpenAI-compatible API. Full local inference for AI-assisted coding with zero API costs.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Start llama-server](#1-start-llama-server)
  - [Configure OpenCode](#2-configure-opencode)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Model selection for coding](#model-selection-for-coding)
  - [Declarative config](#declarative-config)
  - [Performance tuning](#performance-tuning)
  - [Multiple providers](#multiple-providers)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

OpenCode supports any OpenAI-compatible API through its flexible provider system. Point it at llama-server and you get the full OpenCode experience—Plan/Build modes, file references, shell integration—powered by your own hardware.

**Why llama.cpp for OpenCode?**

- Maximum control over inference parameters
- No API costs, no rate limits
- Run larger context windows on your hardware
- Complete privacy for sensitive codebases

---

## Setup

### 1) Start llama-server

```bash
# Basic start (CPU)
llama-server -m /path/to/model.gguf

# With GPU offload and larger context (recommended for code)
llama-server -m model.gguf -ngl 99 -c 8192 --port 8080

# Or download and serve directly
llama-server -hf bartowski/Qwen2.5-Coder-14B-Instruct-GGUF:Q4_K_M -ngl 99 -c 8192
```

Verify:

```bash
curl http://localhost:8080/v1/models
```

### 2) Configure OpenCode

**Option A: Declarative config (recommended)**

Create or edit `~/.config/opencode/opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llamacpp": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp (local)",
      "options": {
        "baseURL": "http://localhost:8080/v1"
      },
      "models": {
        "local": {
          "name": "Local Model"
        }
      }
    }
  },
  "model": "llamacpp/local"
}
```

**Option B: TUI configuration**

1. Launch OpenCode: `opencode`
2. Run `/connect`
3. Choose "Custom OpenAI-compatible"
4. Enter:
   - **Base URL**: `http://localhost:8080/v1`
   - **API Key**: `not-needed`
5. Select model with `/models`

---

## Beginner usage

1. Start llama-server with a coding model
2. Add the provider config to `~/.config/opencode/opencode.json`
3. Launch `opencode` in your project directory
4. Run `/init` to generate AGENTS.md
5. Start coding!

**Quick test:**

```bash
# Terminal 1
llama-server -hf Qwen/Qwen2.5-Coder-7B-Instruct-GGUF -ngl 99

# Terminal 2
cd your-project
opencode
# In TUI: "Explain the structure of this codebase"
```

---

## Pro usage

### Model selection for coding

**Recommended models (2025):**

| Model | Size | Best For |
| ----- | ---- | -------- |
| Qwen2.5-Coder-32B | 32B | Complex refactoring, architecture |
| Qwen2.5-Coder-14B | 14B | General coding, good balance |
| Qwen2.5-Coder-7B | 7B | Quick tasks, limited VRAM |
| DeepSeek-Coder-V2 | 16B | Strong reasoning |
| CodeLlama-34B | 34B | Code generation focus |

**Quantization for OpenCode:**

- **Q5_K_M or Q6_K**: Best for complex reasoning tasks
- **Q4_K_M**: Good balance for most coding
- **Avoid Q3 and below**: Reasoning quality degrades

### Declarative config

Full config with multiple local providers:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llamacpp-coder": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp Coder",
      "options": {
        "baseURL": "http://localhost:8080/v1"
      },
      "models": {
        "qwen-coder": {
          "name": "Qwen2.5-Coder-14B"
        }
      }
    },
    "llamacpp-reasoning": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp Reasoning",
      "options": {
        "baseURL": "http://localhost:8081/v1"
      },
      "models": {
        "deepseek": {
          "name": "DeepSeek-V2"
        }
      }
    }
  },
  "model": "llamacpp-coder/qwen-coder",
  "small_model": "llamacpp-coder/qwen-coder"
}
```

### Performance tuning

**llama-server flags for coding:**

```bash
# Large context for multi-file projects
llama-server -m model.gguf -ngl 99 -c 16384

# More parallel slots for multiple operations
llama-server -m model.gguf -ngl 99 -c 8192 --parallel 2

# Continuous batching for throughput
llama-server -m model.gguf -ngl 99 --cont-batching
```

**Context size recommendations:**

- 4096: Small scripts, single-file edits
- 8192: Standard projects, multi-file context
- 16384: Large codebases, extensive context
- 32768+: Mono-repos, documentation generation

### Multiple providers

Mix local and cloud providers:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "local": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Local (llama.cpp)",
      "options": { "baseURL": "http://localhost:8080/v1" },
      "models": { "local": { "name": "Local Model" } }
    }
  },
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "local/local"
}
```

Use `/models` in TUI to switch between providers during a session.

---

## Cost savings guide

- **$0 API costs** — only electricity
- Use local for iteration, cloud for final review
- Smaller models (7B) for simple edits, larger (14B+) for complex tasks
- Set `small_model` to local for summaries and titles

**Token cost comparison:**

| Provider | Input/1M | Output/1M |
| -------- | -------- | --------- |
| Claude Sonnet | $3 | $15 |
| GPT-4 | $10 | $30 |
| llama.cpp | $0 | $0 |

---

## Privacy guide

- **All code stays local** — nothing sent to external servers
- No telemetry from llama.cpp
- Perfect for:
  - Client codebases under NDA
  - Proprietary algorithms
  - Security-sensitive code
  - Pre-release features

**OpenCode privacy settings:**

```jsonc
{
  "share": "disabled",
  "disabled_providers": ["openai", "anthropic"]
}
```

---

## Security guide

- Bind llama-server to `127.0.0.1` (default)
- For team use with LAN access:
  - Use `--api-key` on llama-server
  - Put behind reverse proxy with TLS
  - Restrict by IP/firewall

**OpenCode permission lockdown:**

```jsonc
{
  "permission": {
    "edit": "ask",
    "bash": {
      "rm *": "deny",
      "git push": "ask",
      "*": "allow"
    }
  }
}
```

**Secure LAN deployment:**

```bash
# Server with auth
llama-server -m model.gguf --host 0.0.0.0 --api-key "team-secret"
```

```jsonc
// opencode.json
{
  "provider": {
    "team-llm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Team LLM",
      "options": {
        "baseURL": "http://llm-server.local:8080/v1",
        "apiKey": "team-secret"
      }
    }
  }
}
```

---

## Appendix

**Minimal config for llama.cpp**

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "local": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp",
      "options": { "baseURL": "http://localhost:8080/v1" },
      "models": { "m": { "name": "Local" } }
    }
  },
  "model": "local/m"
}
```

**Troubleshooting**

| Issue | Solution |
| ----- | -------- |
| "Provider not found" | Check `npm` field matches `@ai-sdk/openai-compatible` |
| Connection refused | Ensure llama-server is running on correct port |
| Slow generation | Add `-ngl 99` for GPU, increase threads |
| Context too small | Increase `-c` flag on server |
| Model not responding | Check server logs, verify model loaded |

**Zed Editor Integration**

Zed can also use llama-server's OpenAI-compatible API. In `~/.config/zed/settings.json`:

```json
{
  "language_models": {
    "openai": {
      "api_url": "http://localhost:8080/v1",
      "available_models": [
        { "name": "local-model", "display_name": "Local LLM" }
      ]
    }
  }
}
```

Note: Zed's agentic features with local models are experimental. Test with simpler completions first.

**See also**

- [llamacpp-server.md](./llamacpp-server.md) — full server reference
- [opencode.md](../opencode/opencode.md) — OpenCode guide
- [llamacpp-goose.md](./llamacpp-goose.md) — Goose integration
