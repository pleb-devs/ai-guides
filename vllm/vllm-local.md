# vLLM (Local Inference Server)

> Run fast, OpenAI-compatible LLM serving locally for private workflows and shared agents.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [1) Install vLLM](#1-install-vllm)
  - [2) Start the server](#2-start-the-server)
  - [3) Verify the API](#3-verify-the-api)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Serve GLM-4.7-Flash](#serve-glm-47-flash)
  - [OpenAI-compatible clients](#openai-compatible-clients)
  - [Offline batch inference (Python)](#offline-batch-inference-python)
  - [Multi-GPU serving](#multi-gpu-serving)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

vLLM is a fast library for LLM inference and serving, designed for high throughput and memory efficiency. It integrates with Hugging Face models and exposes an OpenAI-compatible API server. ([1])

Use it when you want local inference, a private API endpoint for tools, or a single server that can handle many concurrent requests. ([1])

---

## Setup

### 1) Install vLLM

- **Requirements**: Linux and Python 3.10-3.13. Windows is not supported natively; use WSL if needed. ([3])

**Recommended (uv, auto-selects the right PyTorch wheel):** ([3])

```bash
uv venv --python 3.12 --seed
source .venv/bin/activate
uv pip install -U vllm --torch-backend=auto
```

**pip fallback (choose the CUDA wheel that matches your system):** ([3])

```bash
# Example: CUDA 12.9
pip install vllm --extra-index-url https://download.pytorch.org/whl/cu129
```

**Cache location (Hugging Face Hub):** Model files are cached under `HF_HOME` (default `~/.cache/huggingface`) and `HF_HUB_CACHE` (default `$HF_HOME/hub`). Set these env vars if you want to change where models are stored. ([4])

### 2) Start the server

```bash
vllm serve Qwen/Qwen2.5-1.5B-Instruct
```

The server listens on `http://localhost:8000` by default. Use `--host` or `--port` to change the bind address or port. The server hosts one model at a time. ([2])

### 3) Verify the API

```bash
curl http://localhost:8000/v1/models
```

If you get a JSON response, the server is working. ([2])

---

## Beginner usage

1) Start the server with your model.
2) Send a completion request (OpenAI-compatible):

    ```bash
    curl http://localhost:8000/v1/completions \
      -H "Content-Type: application/json" \
      -d '{
        "model": "Qwen/Qwen2.5-1.5B-Instruct",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
      }'
    ```

3) For chat-style models, use `/v1/chat/completions` with a `messages` array. ([2])

---

## Pro usage

### Serve GLM-4.7-Flash

The GLM-4.7-Flash model card notes that vLLM support requires the main/nightly branch (and Transformers main). vLLM docs recommend `uv` for nightly installs because `pip` can miss pre-release wheels on extra indexes. ([3], [9])

```bash
uv pip install -U vllm --torch-backend=auto --extra-index-url https://wheels.vllm.ai/nightly
pip install -U git+https://github.com/huggingface/transformers.git
```

```bash
vllm serve zai-org/GLM-4.7-Flash
```

### OpenAI-compatible clients

Point OpenAI-compatible SDKs to your local base URL and, if desired, require an API key on the server. You can set the key via `--api-key` or `VLLM_API_KEY`. ([2])

```bash
vllm serve Qwen/Qwen2.5-1.5B-Instruct --api-key local-dev-key
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="local-dev-key",
)

response = client.chat.completions.create(
    model="Qwen/Qwen2.5-1.5B-Instruct",
    messages=[{"role": "user", "content": "Write a 1-line summary of vLLM."}],
    max_tokens=64,
)
print(response.choices[0].message.content)
```

### Offline batch inference (Python)

For scripts or batch jobs, use the Python API to generate outputs without running the HTTP server. `LLM.generate` automatically batches prompts. ([5])

```python
from vllm import LLM, SamplingParams

prompts = [
    "Summarize: continuous batching.",
    "Give three bullets on memory efficiency.",
]

sampling_params = SamplingParams(temperature=0.2, max_tokens=96)
llm = LLM(model="Qwen/Qwen2.5-1.5B-Instruct")
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(output.outputs[0].text)
```

### Multi-GPU serving

If a model does not fit on one GPU, use tensor parallelism to shard it across multiple GPUs. ([6])

```bash
vllm serve Qwen/Qwen2.5-1.5B-Instruct --tensor-parallel-size 4
```

vLLM supports continuous batching of incoming requests. ([1])

---

## Cost savings guide

- **Use quantized checkpoints** when available. Quantization trades precision for smaller memory footprint and broader hardware compatibility. ([7])
- **Pick the smallest model that meets the task** and switch up only when accuracy demands it.
- **Trim context and max tokens** to reduce compute cost per request.
- **Batch when possible** by sending multiple prompts in one call or letting `LLM.generate` auto-batch prompts. ([5])

---

## Privacy guide

- **Keep traffic local**: by default, the server binds to `localhost:8000`. ([2])
- **Model downloads are external**: vLLM uses Hugging Face model IDs (e.g., `Qwen/...`), so plan for an initial Hub download unless the files are already cached. ([1], [2], [4])
- **Disable usage stats** if required by policy: set `VLLM_NO_USAGE_STATS=1`, `DO_NOT_TRACK=1`, or create `~/.config/vllm/do_not_track`. ([8])

---

## Security guide

- **Do not expose the server publicly** unless you add authentication and firewall controls. vLLM can require an API key for requests. ([2])
- **Run as a non-root user** and avoid granting extra system permissions.
- **Treat prompts as sensitive data** if the host is shared or logs are enabled elsewhere.

---

## Appendix

**Common commands**

```bash
vllm serve <model>
vllm serve <model> --host 0.0.0.0 --port 8000
vllm serve <model> --api-key your-key
```

**Key environment variables**

```bash
VLLM_API_KEY=your-key
VLLM_NO_USAGE_STATS=1
DO_NOT_TRACK=1
```

---

## References

[1]: https://github.com/vllm-project/vllm "vLLM GitHub README"
[2]: https://docs.vllm.ai/en/latest/getting_started/quickstart/ "vLLM quickstart"
[3]: https://docs.vllm.ai/en/latest/getting_started/installation/gpu/ "vLLM GPU installation"
[4]: https://huggingface.co/docs/huggingface_hub/en/package_reference/environment_variables "Hugging Face Hub environment variables"
[5]: https://docs.vllm.ai/en/stable/api/vllm/entrypoints/llm/ "vLLM LLM API"
[6]: https://docs.vllm.ai/serving/parallelism_scaling.html "vLLM parallelism and scaling"
[7]: https://docs.vllm.ai/en/latest/features/quantization/ "vLLM quantization"
[8]: https://docs.vllm.ai/en/v0.9.1/usage/usage_stats.html "vLLM usage stats"
[9]: https://huggingface.co/zai-org/GLM-4.7-Flash/blob/main/README.md "GLM-4.7-Flash model card"
