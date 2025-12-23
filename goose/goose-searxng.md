# Goose → SearXNG (Private Web Search)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Run SearXNG locally](#1-run-searxng-locally)
  - [Start the MCP server](#2-start-the-mcp-server)
  - [Add the extension in Goose](#3-add-the-extension-in-goose)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** This guide wires Goose to a **local SearXNG instance** using a custom MCP extension, so your agent can search the web without sending queries to hosted search APIs. ([Block][1])

**Why this matters**

- **Keep search local**: SearXNG is designed for private instances and avoids sending identifying data to upstream engines. ([SearXNG][2])
- **Goose supports custom extensions** over MCP, which makes a local search tool straightforward to add. ([Block][1])

---

## Setup

### 1) Run SearXNG locally

Use the local SearXNG guide for a minimal Docker setup (binds to `localhost:8888` and enables JSON search output). ([SearXNG Guide][3])

### 2) Start the MCP server

Use the lightweight SearXNG MCP server (Python) to expose a tool Goose can call. ([SearXNG MCP][4])

```bash
uvx mcp-searxng
```

### 3) Add the extension in Goose

Add a **Custom Extension** with the command and environment below, then save it. ([Block][1])

```bash
# Extension command
uvx mcp-searxng

# Extension env
SEARXNG_URL=http://localhost:8888
```

---

## Beginner usage

- Ask Goose to search, e.g. “Search the web for the latest Rust borrowing guidelines and summarize the results.”
- If Goose asks which tool to use, pick your SearXNG extension.

---

## Pro usage

- **Allowlist your extension** if you run with strict tool allowlists. ([Block][5])
- **Trim SearXNG engines** to reduce noise and outbound requests. See the SearXNG guide for settings. ([SearXNG Guide][3])
- **Combine with local models** to keep both inference and search local. ([Block][6])

---

## Cost savings guide

- **No search API fees** when self-hosting; you only pay local compute/bandwidth.

---

## Privacy guide

- **Hosted search is not private**: third‑party search APIs log queries. Use the local SearXNG extension for private workflows.
- **SearXNG minimizes tracking** and is designed for private instances. ([SearXNG][2])

---

## Security guide

- **Bind to localhost** and set a strong SearXNG `secret_key` if you ever expose the instance beyond your machine. ([SearXNG Guide][3])
- **Limit tool access** with Goose permission modes and allowlists. ([Block][5])

---

## Appendix

**Quick checklist**

- SearXNG running locally (`http://localhost:8888`).
- MCP server running (`uvx mcp-searxng`).
- Goose extension added with `SEARXNG_URL` set.

---

[1]: https://block.github.io/goose/docs/tutorials/custom-extensions/ "Custom Extensions"
[2]: https://docs.searxng.org/own-instance.html "Why use a private instance?"
[3]: ../searxng/searxng-local.md "SearXNG local guide"
[4]: https://github.com/SecretiveShell/MCP-searxng "MCP SearXNG"
[5]: https://block.github.io/goose/docs/guides/allowlist/ "Goose Extension Allowlist"
[6]: https://block.github.io/goose/docs/getting-started/providers/ "Configure LLM Provider"
