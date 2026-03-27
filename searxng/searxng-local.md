# SearXNG (Local Private Search for Goose, Ollama, and OpenCode)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Create config directory and settings.yml](#1-create-config-directory-and-settingsyml)
  - [Run a local container](#2-run-a-local-container)
  - [Verify the search API](#3-verify-the-search-api)
  - [Optional: start Ollama locally](#4-optional-start-ollama-locally)
  - [Connect Goose to SearXNG](#5-connect-goose-to-searxng)
  - [Connect OpenCode to SearXNG](#6-connect-opencode-to-searxng)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** SearXNG is a **metasearch engine** that aggregates results from other search engines **without storing user information** or building profiles. It is a strong fit for privacy-conscious AI workflows because you can run it yourself and expose JSON search results to local tools. ([SearXNG][1], [SearXNG][3])

**How this fits the stack**

- **SearXNG** handles private web search.
- **Ollama** hosts local models on your machine.
- **Goose** and **OpenCode** are the agent shells that call both of them.

That means there is no special "SearXNG-to-Ollama" integration by itself. The common pattern is: **agent uses Ollama for inference and SearXNG for web search**. ([SearXNG][2], [Goose Custom Extensions][8], [OpenCode MCP][11])

**Why this setup works**

- **Local-first web search**: run your own search layer instead of sending prompts to hosted search APIs. ([SearXNG][2])
- **Agent-friendly API**: the built-in Search API supports JSON output when enabled, which makes it easy to wire into MCP servers and local tools. ([SearXNG][3])
- **Clean separation of concerns**: Ollama handles models, while SearXNG handles retrieval. This keeps the stack simple to reason about.

---

## Setup

### 1) Create config directory and settings.yml

Create the config directory and a minimal `settings.yml` **before** starting the container. This keeps defaults, enables JSON output for MCP integrations, and binds to all container interfaces so Docker port-forwarding works. The settings docs show `use_default_settings` plus server `secret_key`, and the search settings list default formats (HTML only by default). ([SearXNG][5], [SearXNG][6], [SearXNG][7])

```bash
mkdir -p ./searxng/config/ ./searxng/data/
cat > ./searxng/config/settings.yml << 'EOF'
use_default_settings: true

search:
  # Enable JSON output for /search?format=json
  formats:
    - html
    - json

server:
  # Bind to all interfaces so Docker port-forwarding works
  # (host-side restriction is handled by -p 127.0.0.1:8080:8080)
  bind_address: "0.0.0.0"
  # Change this before exposing beyond localhost
  secret_key: "change-me-please"
EOF
```

### 2) Run a local container

The official container guide documents Docker-based setup; here is a minimal local example that binds to `localhost:8080` and persists config/cache to local volumes. ([SearXNG][4])

```bash
cd ./searxng/

docker run --name searxng -d \
  -p 127.0.0.1:8080:8080 \
  -v "./config/:/etc/searxng/" \
  -v "./data/:/var/cache/searxng/" \
  docker.io/searxng/searxng:latest
```

### 3) Verify the search API

The Search API supports `format=json` when `json` is present in `search.formats`. ([SearXNG][3], [SearXNG][6])

```bash
curl "http://localhost:8080/search?q=searxng&format=json"
```

### 4) Optional: start Ollama locally

If you want a full local stack, start Ollama so Goose or OpenCode can use local models while SearXNG handles search.

```bash
ollama serve
ollama pull qwen3.5:4b
```

Quick checks:

```bash
curl http://localhost:11434/api/version
ollama ls
```

### 5) Connect Goose to SearXNG

Goose supports custom MCP extensions, which makes SearXNG straightforward to add as a local search tool. ([Goose Custom Extensions][8])

Start the MCP server:

```bash
npx mcp-searxng
```

Add a **Custom Extension** in Goose with:

```bash
# Extension command
npx mcp-searxng

# Extension env
SEARXNG_URL=http://localhost:8080
```

If you also want Goose to use Ollama for inference:

- Desktop: configure the **Ollama** provider and set API Host to `http://localhost:11434`.
- CLI:

```bash
export GOOSE_PROVIDER=ollama
export OLLAMA_HOST=http://localhost:11434
export GOOSE_MODEL=qwen3.5:4b
goose session --name private-search
```

### 6) Connect OpenCode to SearXNG

OpenCode supports MCP servers for tools and OpenAI-compatible providers for local inference. That makes `SearXNG + Ollama + OpenCode` a clean single-machine workflow. ([OpenCode MCP][11], [OpenCode Config][12], [OpenCode Providers][13])

Add the SearXNG MCP server and optional Ollama provider to `~/.config/opencode/opencode.json` or a project-local `./opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3.5:4b": {
          "name": "Qwen 3.5 4B (local)"
        }
      }
    }
  },
  "model": "ollama/qwen3.5:4b",
  "small_model": "ollama/qwen3.5:4b",
  "mcp": {
    "searxng": {
      "type": "local",
      "command": ["uvx", "mcp-searxng"],
      "enabled": true,
      "environment": {
        "SEARXNG_URL": "http://localhost:8080"
      }
    }
  },
  "tools": {
    "websearch": false
  }
}
```

This tells OpenCode to use Ollama for model calls, SearXNG for web search, and the built-in hosted `websearch` tool stays off.

---

## Beginner usage

- **Use SearXNG directly**: open `http://localhost:8080` and search in the browser.
- **Use JSON for tooling**: call `/search?format=json` and pass `q`, `categories`, or `engines`. ([SearXNG][3])
- **Goose flow**: start a Goose session and ask it to search the web; if it asks which tool to use, pick your SearXNG extension.
- **OpenCode flow**: launch `opencode`, select your Ollama-backed model with `/models`, and ask for a search-based summary or research task.
- **Simple mental model**: SearXNG retrieves, Ollama reasons, Goose/OpenCode orchestrate.

---

## Pro usage

- **Trim engines** to reduce noise and outbound requests using `engines.remove` or `engines.keep_only`. The settings docs show how to override and filter engines with `use_default_settings`. ([SearXNG][5])
- **Pin SearXNG as the only search path** in OpenCode by disabling the built-in `websearch` tool. ([OpenCode Tools][14])
- **Use local models with Goose** so both inference and search stay on your machine as much as possible. ([Goose Providers][9])
- **Use smaller Ollama models for search-heavy tasks** and keep larger hosted models only for difficult refactors.
- **Keep the instance local** by binding Docker to `127.0.0.1`, or place it behind a VPN/reverse proxy only if you truly need remote access. ([SearXNG][2])
- **Use proxies or Tor** if you want extra anonymity when SearXNG queries upstream search engines. ([SearXNG][2])

---

## Cost savings guide

- **No per-query API fees** when you self-host SearXNG; you only pay local compute and bandwidth.
- **Avoid hosted search add-ons** by routing agent search through your own SearXNG instance.
- **Pair SearXNG with Ollama** to cut both search spend and inference spend for everyday work.
- **Limit engines** to reduce rate limits, latency, and background resource usage.

---

## Privacy guide

- **Run your own instance**: private instances keep source code, logging settings, and private data under your control. ([SearXNG][2])
- **SearXNG minimizes tracking** by not sending cookies to external search engines and by generating a random browser profile per request. ([SearXNG][2])
- **Local Ollama keeps prompts on-device** when it is bound to localhost.
- **Disable OpenCode sharing** if you want stricter local-only operation. ([OpenCode Share][15])
- **Use Goose allowlists and `.gooseignore`** when working in sensitive repos. ([Goose Allowlists][10], [Goose Ignore][16])

---

## Security guide

- **Bind SearXNG to localhost** for single-user setups using Docker's host-side restriction (`-p 127.0.0.1:8080:8080`). ([SearXNG][7])
- **Set a strong `secret_key`** before exposing the instance beyond your machine. ([SearXNG][5], [SearXNG][7])
- **Use `use_default_settings: true`** and override only what you need to reduce misconfiguration risk. ([SearXNG][5])
- **Keep Ollama on `127.0.0.1`** unless you add firewall rules and authentication in front of it.
- **Scope tool access** in Goose and OpenCode so search and shell tools are not broader than they need to be. ([Goose Allowlists][10], [OpenCode Tools][14])

---

## Appendix

**Minimal settings.yml**

```yaml
use_default_settings: true

search:
  formats:
    - html
    - json

server:
  bind_address: "0.0.0.0"
  secret_key: "change-me-please"
```

**Quick checklist**

- SearXNG running locally at `http://localhost:8080`
- SearXNG `settings.yml` has `json` in `search.formats`
- Ollama running locally at `http://localhost:11434` if you want local inference
- Goose extension added with `SEARXNG_URL=http://localhost:8080`
- OpenCode config includes the SearXNG MCP server

**Two common local stacks**

```bash
# Goose + Ollama + SearXNG
ollama serve
npx mcp-searxng
goose session --name private-search
```

```bash
# OpenCode + Ollama + SearXNG
ollama serve
opencode
```

---

[1]: https://docs.searxng.org/user/about.html "About SearXNG"
[2]: https://docs.searxng.org/own-instance.html "Why use a private instance?"
[3]: https://docs.searxng.org/dev/search_api.html "Search API"
[4]: https://docs.searxng.org/admin/installation-docker "Installation container"
[5]: https://docs.searxng.org/admin/settings/settings "settings.yml"
[6]: https://docs.searxng.org/admin/settings/settings_search.html "search settings"
[7]: https://docs.searxng.org/admin/settings/settings_server.html "server settings"
[8]: https://block.github.io/goose/docs/tutorials/custom-extensions/ "Goose Custom Extensions"
[9]: https://block.github.io/goose/docs/getting-started/providers/ "Goose Providers"
[10]: https://block.github.io/goose/docs/guides/allowlist/ "Goose Extension Allowlist"
[11]: https://opencode.ai/docs/mcp-servers/ "OpenCode MCP"
[12]: https://opencode.ai/docs/config/ "OpenCode Config"
[13]: https://opencode.ai/docs/providers/ "OpenCode Providers"
[14]: https://opencode.ai/docs/tools/ "OpenCode Tools"
[15]: https://opencode.ai/docs/share/ "OpenCode Share"
[16]: https://block.github.io/goose/docs/guides/using-gooseignore/ "Prevent Goose from Accessing Files"
