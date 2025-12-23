# OpenCode → SearXNG (Private Web Search)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Run SearXNG locally](#1-run-searxng-locally)
  - [Add the MCP server](#2-add-the-mcp-server)
  - [Wire OpenCode to SearXNG](#3-wire-opencode-to-searxng)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** This guide connects OpenCode to a **local SearXNG instance** via MCP so web search stays on your machine instead of going through hosted search APIs. ([Opencode][1])

**Why this matters**

- **Private search for local LLMs**: SearXNG is designed for private instances and reduces tracking. ([SearXNG][2])
- **OpenCode supports MCP servers** for custom tools, making SearXNG a simple drop‑in. ([Opencode][1])

---

## Setup

### 1) Run SearXNG locally

Use the local SearXNG guide for a minimal Docker setup (binds to `localhost:8888` and enables JSON search output). ([SearXNG Guide][3])

### 2) Add the MCP server

Run the SearXNG MCP server (Python) so OpenCode can call it. ([SearXNG MCP][4])

```bash
uvx mcp-searxng
```

### 3) Wire OpenCode to SearXNG

Add the MCP server to your `opencode.json` (global or per‑project). ([Opencode][6])

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "searxng": {
      "type": "local",
      "command": ["uvx", "mcp-searxng"],
      "enabled": true,
      "environment": {
        "SEARXNG_URL": "http://localhost:8888"
      }
    }
  },
  "tools": {
    "websearch": false,
    "codesearch": false
  }
}
```

---

## Beginner usage

- Ask OpenCode to search the web; when tool selection matters, pick the SearXNG tool.
- If you disable built‑in search tools, OpenCode will rely on the SearXNG MCP server by default.

---

## Pro usage

- **Pin SearXNG only** by disabling hosted search tools (`websearch`, `codesearch`) in config. ([Opencode][7])
- **Trim engines** in SearXNG for faster, cleaner results. ([SearXNG Guide][3])
- **Use local models** for end‑to‑end private workflows. ([Opencode][5])

---

## Cost savings guide

- **No per‑query fees** when self‑hosting search.

---

## Privacy guide

- **Hosted search is not private**: third‑party search APIs log queries. Use SearXNG locally for private workflows.
- **SearXNG minimizes tracking** and is intended for private instances. ([SearXNG][2])

---

## Security guide

- **Bind to localhost** and set a strong SearXNG `secret_key` if you ever expose the instance beyond your machine. ([SearXNG Guide][3])
- **Keep tools scoped** in OpenCode by disabling unneeded tools. ([Opencode][7])

---

## Appendix

**Quick checklist**

- SearXNG running locally (`http://localhost:8888`).
- MCP server running (`uvx mcp-searxng`).
- OpenCode config includes the SearXNG MCP server.

---

[1]: https://opencode.ai/docs/mcp-servers/ "MCP servers | opencode"
[2]: https://docs.searxng.org/own-instance.html "Why use a private instance?"
[3]: ../searxng/searxng-local.md "SearXNG local guide"
[4]: https://github.com/SecretiveShell/MCP-searxng "MCP SearXNG"
[5]: https://opencode.ai/docs/providers/ "Providers | opencode"
[6]: https://opencode.ai/docs/config/ "Config | opencode"
[7]: https://opencode.ai/docs/tools/ "Tools | opencode"
