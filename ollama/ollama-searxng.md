# Ollama → SearXNG (Force Local Web Search)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Run SearXNG locally](#1-run-searxng-locally)
  - [Disable Ollama cloud web search](#2-disable-ollama-cloud-web-search)
  - [Start Ollama with enough context](#3-start-ollama-with-enough-context)
  - [Install the Python dependencies](#4-install-the-python-dependencies)
  - [Run a local SearXNG tool loop](#5-run-a-local-searxng-tool-loop)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

## Overview

**What it is.** This guide shows how to make an Ollama-hosted local model do web search through **your own SearXNG instance** instead of Ollama's hosted web search API. The pattern is: **disable Ollama cloud search, then expose SearXNG as the model's only search tool using Ollama tool calling**. ([Ollama Web Search][1], [Ollama Tool Calling][2], [Ollama FAQ][3])

**Important distinction**

- **Ollama does have an official hosted web search API**, and it requires an Ollama API key. ([Ollama Web Search][1])
- **Ollama's docs do not describe a setting to redirect that hosted API to SearXNG.**
- So if you want to "force Ollama to use SearXNG", the practical local-only approach is:
  - disable Ollama cloud features
  - register a local `searxng_search` tool
  - instruct the model to use that tool for any current-events or web lookup task

**Why this setup works**

- **Tool calling is built into Ollama**, so the model can call your local search function directly. ([Ollama Tool Calling][2])
- **SearXNG gives you a JSON search API** on your own machine. ([SearXNG Search API][4])
- **Local-only mode in Ollama disables cloud models and hosted web search**, which helps enforce the SearXNG path. ([Ollama FAQ][3])

---

## Setup

### 1) Run SearXNG locally

Use the local SearXNG guide for the Docker setup and required `json` format setting. ([SearXNG Guide][5])

At minimum, verify this works:

```bash
curl "http://localhost:8080/search?q=ollama&format=json"
```

### 2) Disable Ollama cloud web search

Ollama documents that disabling cloud features also disables its hosted web search. Use either `~/.ollama/server.json` or the environment variable below, then restart Ollama. ([Ollama FAQ][3])

```json
{
  "disable_ollama_cloud": true
}
```

```bash
export OLLAMA_NO_CLOUD=1
```

### 3) Start Ollama with enough context

Ollama recommends at least `64000` tokens for web search, agents, and coding tools. ([Ollama Context Length][6])

```bash
OLLAMA_NO_CLOUD=1 OLLAMA_CONTEXT_LENGTH=64000 ollama serve
ollama pull qwen3:4b
```

Quick checks:

```bash
curl http://localhost:11434/api/version
ollama ps
```

### 4) Install the Python dependencies

Use the Ollama Python SDK plus `requests` for the local SearXNG HTTP call.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install ollama requests
```

### 5) Run a local SearXNG tool loop

Save this as `search_with_searxng.py`. It gives the model one search tool, backed by your local SearXNG instance.

```python
import json
import requests
from ollama import chat

SEARXNG_URL = "http://localhost:8080"
MODEL = "qwen3:4b"


def searxng_search(query: str, max_results: int = 5) -> str:
    """Search the web with a local SearXNG instance.

    Args:
      query: The search query.
      max_results: Maximum number of results to keep.

    Returns:
      JSON string of top search results.
    """
    response = requests.get(
        f"{SEARXNG_URL}/search",
        params={"q": query, "format": "json"},
        timeout=20,
    )
    response.raise_for_status()
    payload = response.json()

    results = []
    for item in payload.get("results", [])[:max_results]:
        results.append(
            {
                "title": item.get("title"),
                "url": item.get("url"),
                "content": item.get("content", ""),
            }
        )

    return json.dumps(results, ensure_ascii=True)


available_functions = {
    "searxng_search": searxng_search,
}

messages = [
    {
        "role": "system",
        "content": (
            "You are a local research assistant. "
            "For any request that needs current, external, or web-based information, "
            "you must call the searxng_search tool first. "
            "Do not use any hosted web search."
        ),
    },
    {
        "role": "user",
        "content": "Search the web for the latest Ollama tool-calling guidance and summarize it.",
    },
]

while True:
    response = chat(
        model=MODEL,
        messages=messages,
        tools=[searxng_search],
        think=True,
    )

    messages.append(response.message)

    if response.message.tool_calls:
        for call in response.message.tool_calls:
            result = available_functions[call.function.name](**call.function.arguments)
            messages.append(
                {
                    "role": "tool",
                    "tool_name": call.function.name,
                    "content": result,
                }
            )
    else:
        print(response.message.content)
        break
```

Run it:

```bash
source .venv/bin/activate
python search_with_searxng.py
```

---

## Beginner usage

- Start SearXNG on `http://localhost:8080`.
- Start Ollama with `OLLAMA_NO_CLOUD=1`.
- Run the Python script above.
- Ask a prompt that clearly requires fresh web info.
- Confirm the model responds after calling `searxng_search` instead of any hosted search path.

---

## Pro usage

- **Keep SearXNG as the only search tool** in your loop if you want hard local-only behavior.
- **Trim SearXNG engines** to reduce noise and outbound requests. See the SearXNG guide for settings. ([SearXNG Guide][5])
- **Add a second local tool for URL fetches** if you want the model to read full pages after search; SearXNG search snippets alone are often enough for short summaries but not deep research.
- **Use a stronger system instruction** if a smaller model skips tool calling too often.
- **Raise context length** when search results are long or multi-step. Ollama recommends `64000` tokens for web search and agent workflows. ([Ollama Context Length][6])
- **Swap in a different local model** as long as it supports tool calling.

---

## Cost savings guide

- **Disable Ollama cloud features** so you do not accidentally use hosted web search.
- **Use SearXNG instead of paid search APIs** for routine lookups.
- **Keep a smaller local model** for search-heavy tasks and reserve larger models for synthesis or final editing.
- **Limit SearXNG engines** to reduce latency and bandwidth.

---

## Privacy guide

- **Ollama local inference stays on-device** when you run against `http://localhost:11434`.
- **Ollama's hosted web search is cloud-backed**, so disable cloud features for stricter privacy. ([Ollama Web Search][1], [Ollama FAQ][3])
- **SearXNG is designed for private instances** and minimizes tracking toward upstream engines. ([SearXNG Private Instance][7])
- **This pattern keeps search routing under your control** because your script only exposes the local SearXNG tool.

---

## Security guide

- **Bind both services to localhost** unless you explicitly need LAN access.
- **Set a strong SearXNG `secret_key`** before exposing it beyond your machine. ([SearXNG Guide][5])
- **Do not expose Ollama on the public internet** without a reverse proxy, authentication, and firewall rules. ([Ollama FAQ][3])
- **Treat search results as untrusted input** and avoid automatically executing commands or code generated from the web.

---

## Appendix

**Quick checklist**

- SearXNG is running at `http://localhost:8080`
- `search.formats` includes `json`
- Ollama is running at `http://localhost:11434`
- `OLLAMA_NO_CLOUD=1` is set, or `disable_ollama_cloud` is enabled
- The script passes `tools=[searxng_search]`

**Why this is the right mental model**

- Ollama is the **model host**
- SearXNG is the **search backend**
- Your script is the **agent loop** that forces the model to use SearXNG

**See also**

- [../searxng/searxng-local.md](../searxng/searxng-local.md) — unified SearXNG guide
- [./ollama-cli.md](./ollama-cli.md) — base Ollama API usage
- [../goose/goose-searxng.md](../goose/goose-searxng.md) — Goose using SearXNG as a web search tool

---

[1]: https://docs.ollama.com/capabilities/web-search "Ollama web search"
[2]: https://docs.ollama.com/capabilities/tool-calling "Ollama tool calling"
[3]: https://docs.ollama.com/faq "Ollama FAQ"
[4]: https://docs.searxng.org/dev/search_api.html "SearXNG Search API"
[5]: ../searxng/searxng-local.md "SearXNG local guide"
[6]: https://docs.ollama.com/context-length "Ollama context length"
[7]: https://docs.searxng.org/own-instance.html "Why use a private instance?"
