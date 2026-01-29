# llama.cpp Guides Appendix

This folder contains practical docs for running local LLM inference with **llama.cpp** and integrating it with agents and editors.

## What's Here

- [llamacpp-cli.md](./llamacpp-cli.md) — Terminal-based inference
  - Install methods, llama-cli usage, model downloads, flags, and common patterns.
- [llamacpp-server.md](./llamacpp-server.md) — OpenAI-compatible API server
  - Run llama-server, API endpoints, multi-slot serving, and client integration.
- [llamacpp-quantization.md](./llamacpp-quantization.md) — Model quantization guide
  - GGUF format, quantization types, convert and quantize workflows.
- [llamacpp-goose.md](./llamacpp-goose.md) — Integration with Goose
  - Point Goose (Desktop/CLI) at llama-server's OpenAI-compatible API.
- [llamacpp-opencode.md](./llamacpp-opencode.md) — Integration with OpenCode
  - Configure OpenCode to use llama.cpp as a local provider.

## Choose a Path

- Want maximum control over inference? Start with **llamacpp-cli.md**.
- Need an API for agents and tools? Use **llamacpp-server.md**.
- Optimizing model size/speed? Read **llamacpp-quantization.md**.
- Connecting to Goose? Jump to **llamacpp-goose.md**.
- Using OpenCode? See **llamacpp-opencode.md**.

## llama.cpp vs Ollama — When to Use Each

| Factor | llama.cpp | Ollama |
|--------|-----------|--------|
| **Ease of use** | More manual setup | One-command install and run |
| **Control** | Full control over inference params, quantization, GPU layers | Abstracted; fewer knobs |
| **Memory efficiency** | Fine-tune context, batch, GPU offload | Automatic but less tunable |
| **Cutting-edge features** | First to get new backends (Vulkan, SYCL, etc.) | Follows llama.cpp upstream |
| **Quantization** | Do it yourself; any format | Pre-quantized models from registry |
| **Multi-model** | Manual process management | Built-in model switching |
| **Best for** | Power users, custom deployments, edge cases | Quick local hosting, beginners |

**Rule of thumb:**

- Use **Ollama** if you want "it just works" local AI with minimal config.
- Use **llama.cpp** if you need maximum performance tuning, specific quantization, latest features, or are deploying in production/embedded scenarios.

> Ollama is built on llama.cpp internally—same inference engine, different UX layer.

## Related

- Ollama guides live in the `ollama/` folder:
  - [ollama-cli.md](../ollama/ollama-cli.md)
  - [ollama-desktop.md](../ollama/ollama-desktop.md)
  - [ollama-goose.md](../ollama/ollama-goose.md)
- Goose usage guides in `goose/`:
  - [goose-cli.md](../goose/goose-cli.md)
  - [goose-desktop.md](../goose/goose-desktop.md)
- OpenCode guides in `opencode/`:
  - [opencode.md](../opencode/opencode.md)
