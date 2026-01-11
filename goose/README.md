# Goose Guides Appendix

This folder indexes the Goose guides and how to use them together. Use this as a quick map to pick the right doc for your task.

## What’s Here

- [goose-desktop.md](./goose-desktop.md) — Desktop app guide
  - Setup, providers, permission modes, extensions (MCP), recipes, privacy/security, and troubleshooting for GUI workflows.
- [goose-cli.md](./goose-cli.md) — CLI guide for headless/CI
  - Sessions and one‑off runs, recipes and scheduling, projects, CI/CD usage, environment variables, and audit/export tips.
- [goose-ollama.md](./goose-ollama.md) — Goose + Ollama integration
  - Connect Goose (Desktop/CLI) to a local Ollama server. Includes OpenAI‑compatible base URL (`/v1`) configuration, model naming, and common errors.
- [goose-searxng.md](./goose-searxng.md) — Goose + SearXNG private search
  - Run SearXNG locally and add it as a custom MCP extension for private web search.
- [recipes/](./recipes/) — Ready-to-use Goose recipes
  - Deep code review with project docs, automation templates. Designed for projects using the [new-project-boilerplate](https://github.com/pleb-devs/new-project-boilerplate) `llm/` structure.

## Choose a Path

- New to Goose? Start with Desktop → goose-desktop.md, then skim goose-cli.md if you need automation.
- Already hosting local models? Jump straight to goose-ollama.md to wire Goose to your Ollama instance.
- Want private web search? Use goose-searxng.md after a quick read of goose-desktop.md or goose-cli.md.
- Using the PlebDevs boilerplate? Check out recipes/ for deep code review workflows that leverage your `llm/` docs.

## Related

- Local model hosting guides live in the `ollama/` folder:
  - [ollama-desktop.md](../ollama/ollama-desktop.md) — host and manage models on your machine
  - [ollama-cli.md](../ollama/ollama-cli.md) — API checks, lifecycle, service patterns
  - [ollama-goose.md](../ollama/ollama-goose.md) — use Ollama to power Goose
