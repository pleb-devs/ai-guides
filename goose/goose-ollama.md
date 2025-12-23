# Configure Goose with a Local Ollama Provider — PlebDevs Integration

> Connect Goose (Desktop or CLI) to models hosted by your local Ollama server at `http://localhost:11434`. Run fully local with your preferred models.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

Point Goose to Ollama via the OpenAI‑compatible interface at `/v1` (works broadly) or a native Ollama provider if available in your build.

<img width="1144" height="709" alt="image" src="https://github.com/user-attachments/assets/ddf3a50c-d530-489e-a36c-bb90bb0399e5" />

---

## Setup

1) Start Ollama locally

```bash
ollama serve
ollama run llama3   # downloads on first use
```

1) Configure Goose

- Base URL: `http://localhost:11434/v1`
- API key: any non‑empty string (unused by Ollama)
- Model: exact Ollama model name (e.g., `llama3`)

Desktop

- Settings → Providers → Add OpenAI‑compatible provider → set Base URL and Model → create a session using that provider.

CLI

```bash
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_API_KEY=not-needed
export GOOSE_MODEL=llama3
goose session --name local-model
```

---

## Beginner usage

1) Verify Ollama is serving: `curl http://localhost:11434/api/generate -d '{"model":"llama3","prompt":"hi"}'`
2) Create a new Goose session selecting your Ollama-backed provider.
3) Send a short prompt; confirm responses are fast and consistent.

<img width="1155" height="713" alt="image" src="https://github.com/user-attachments/assets/0a95c820-4dab-4bf0-b2dd-bc9cd7b6a9a7" />

---

## Pro usage

- Use smaller local models for routine tasks; switch up per session.
- For CLI automation, set env vars in scripts/CI and run `goose run`.
- Keep model names pinned (e.g., custom Modelfile builds) to avoid surprises.
- Combine with `.gooseignore` to fence sensitive paths during agent runs.

---

## Cost savings guide

- Prefer smaller/quantized models; keep heavier models off by default.
- Reuse a single shared Ollama host for multiple Goose sessions.
- Limit context sizes in prompts to reduce CPU/GPU load.

---

## Privacy guide

- With Ollama on localhost, prompts stay on-device.
- Avoid third‑party proxies; if you must, review their privacy posture.

---

## Security guide

- Keep Ollama bound to `127.0.0.1`. If exposing on LAN, add firewall and proxy auth.
- In Goose, prefer Manual/Approve modes for write‑classified tools.

---

## Appendix

**Handy commands**

```bash
ollama list
ollama show llama3
goose session --name local-model
```

See also: `../ollama/ollama-desktop.md`, `../ollama/ollama-cli.md` for hosting details.
