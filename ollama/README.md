# Ollama Guides Appendix

This folder contains practical docs for hosting local models with **Ollama** and integrating them with agents.

## What’s Here

- [ollama-desktop.md](./ollama-desktop.md) — Desktop‑focused hosting
  - Install, run the local API, pull/manage models, Modelfiles, background service, LAN caveats, Docker hosting.
- [ollama-cli.md](./ollama-cli.md) — Terminal + API operations
  - Curl checks for `/api/generate` and `/api/chat`, model lifecycle commands, OpenAI‑compatible `/v1` usage, systemd/Docker patterns.
- [ollama-goose.md](./ollama-goose.md) — Integration with Goose
  - Point Goose (Desktop/CLI) at `http://localhost:11434` via Goose’s Ollama provider; model naming, security notes, and common errors.
- [ollama-opencode.md](./ollama-opencode.md) — Integration with OpenCode
  - Point OpenCode at `http://localhost:11434/v1` using an OpenAI-compatible provider config; includes global/project config and privacy controls.

## Choose a Path

- Want a quick local host for agents? Start with ollama-desktop.md → verify with curl.
- Automating or running headless? Read ollama-cli.md.
- Connecting to Goose? Jump to ollama-goose.md.
- Connecting to OpenCode? Jump to ollama-opencode.md.

## Related

- Goose usage guides live in the `goose/` folder:
  - [goose-desktop.md](../goose/goose-desktop.md)
  - [goose-cli.md](../goose/goose-cli.md)
  - [goose-ollama.md](../goose/goose-ollama.md)
- OpenCode usage guides live in the `opencode/` folder:
  - [opencode.md](../opencode/opencode.md)
