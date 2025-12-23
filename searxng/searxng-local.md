# SearXNG (Local Private Search)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Create config directory and settings.yml](#1-create-config-directory-and-settingsyml)
  - [Run a local container](#2-run-a-local-container)
  - [Verify the search API](#3-verify-the-search-api)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** SearXNG is a **metasearch engine** that aggregates results from other search engines **without storing user information** or building profiles. It never shares your searches with third parties and is intended for privacy-conscious use. ([SearXNG][1])

**Why it fits local AI workflows**

- **Local-first web search**: run your own instance so your search traffic stays under your control. Private instances can be single-user and run locally. ([SearXNG][2])
- **Agent-friendly API**: the built-in Search API supports JSON output when enabled, which makes it easy to wire into MCP or other local tools. ([SearXNG][3])

---

## Setup

### 1) Create config directory and settings.yml

Create the config directory and a minimal `settings.yml` **before** starting the container. This override keeps defaults, enables JSON output for tool integrations, and binds to all container interfaces so Docker port-forwarding works. The settings docs show `use_default_settings` plus server `secret_key`, and the search settings list default formats (HTML only by default). ([SearXNG][5], [SearXNG][6], [SearXNG][7])

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
  # (host-side restriction is handled by -p 127.0.0.1:8888:8080)
  bind_address: "0.0.0.0"
  # Change this before exposing beyond localhost
  secret_key: "change-me-please"
EOF
```

### 2) Run a local container

The official container guide documents Docker-based setup; here's a minimal local example that maps `8888 -> 8080` and persists config/cache to local volumes. ([SearXNG][4])

```bash
cd ./searxng/

docker run --name searxng -d \
  -p 127.0.0.1:8888:8080 \
  -v "./config/:/etc/searxng/" \
  -v "./data/:/var/cache/searxng/" \
  docker.io/searxng/searxng:latest
```

### 3) Verify the search API

The Search API supports `format=json` (when enabled in `search.formats`). ([SearXNG][3], [SearXNG][6])

```bash
curl "http://localhost:8888/search?q=searxng&format=json"
```

---

## Beginner usage

- **Use the web UI**: open `http://localhost:8888` and search normally.
- **Use JSON for tooling**: when wiring a local agent tool, call `/search?format=json` and pass query params like `q`, `categories`, or `engines`. ([SearXNG][3])

---

## Pro usage

- **Trim engines** to reduce noise and outbound requests using `engines.remove` or `engines.keep_only`. The settings docs show how to override and filter engines with `use_default_settings`. ([SearXNG][5])
- **Keep the instance local** for private use by binding the Docker port to `127.0.0.1` (e.g., `-p 127.0.0.1:8888:8080`), or front it with a VPN/reverse proxy only if you need remote access. Private instances can be single-user and run locally. ([SearXNG][2])
- **Use proxies or Tor** if you want extra anonymity when SearXNG queries upstream search engines. ([SearXNG][2])

---

## Cost savings guide

- **No per-query API fees** when you self-host; you only pay local compute/bandwidth.
- **Limit engines** to reduce rate limits and resource usage.

---

## Privacy guide

- **Run your own instance**: private instances keep source code, logging settings, and private data under your control. ([SearXNG][2])
- **SearXNG minimizes tracking** by not sending cookies to external search engines and by generating a random browser profile per request. ([SearXNG][2])
- **No ads / trackers**: SearXNG does not serve ads or tracking content, so data is not forwarded to third parties. ([SearXNG][2])

---

## Security guide

- **Bind to localhost** for single-user setups using Docker's host-side restriction (`-p 127.0.0.1:8888:8080`). ([SearXNG][7])
- **Set a strong secret key** before exposing the instance. The settings docs call out `secret_key` as a required override. ([SearXNG][5], [SearXNG][7])
- **Use `use_default_settings: true`** and override only what you need to reduce misconfiguration risk. ([SearXNG][5])

---

## Appendix

**Minimal settings.yml (copy/paste)**

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

---

[1]: https://docs.searxng.org/user/about.html "About SearXNG"
[2]: https://docs.searxng.org/own-instance.html "Why use a private instance?"
[3]: https://docs.searxng.org/dev/search_api.html "Search API"
[4]: https://docs.searxng.org/admin/installation-docker "Installation container"
[5]: https://docs.searxng.org/admin/settings/settings "settings.yml"
[6]: https://docs.searxng.org/admin/settings/settings_search.html "search settings"
[7]: https://docs.searxng.org/admin/settings/settings_server.html "server settings"
