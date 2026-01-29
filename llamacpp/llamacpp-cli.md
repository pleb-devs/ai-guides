# llama.cpp CLI — PlebDevs AI Guide

> Run LLM inference directly from the terminal with **llama-cli**. Maximum control, no dependencies, works on CPU and GPU. For power users who want to squeeze every drop of performance from their hardware.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Install pre-built binaries](#1-install-pre-built-binaries)
  - [Package managers](#2-package-managers)
  - [Build from source](#3-build-from-source)
  - [Get a model](#4-get-a-model)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Key flags](#key-flags)
  - [GPU acceleration](#gpu-acceleration)
  - [Interactive mode](#interactive-mode)
  - [Batch processing](#batch-processing)
  - [Download models from Hugging Face](#download-models-from-hugging-face)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

`llama-cli` is the primary command-line tool for running inference with llama.cpp. It loads GGUF model files and generates text from prompts. Supports CPU, CUDA (NVIDIA), Metal (Apple), ROCm (AMD), Vulkan, and more.

Key strengths:

- Zero external dependencies (pure C/C++)
- Fine-grained control over inference parameters
- CPU+GPU hybrid inference for models larger than VRAM
- Extensive quantization support (1.5-bit to 8-bit)

---

## Setup

### 1) Install pre-built binaries

Download from the [releases page](https://github.com/ggml-org/llama.cpp/releases):

```bash
# Example for Linux x64 with CUDA
wget https://github.com/ggml-org/llama.cpp/releases/download/b5000/llama-b5000-bin-ubuntu-x64-cuda.zip
unzip llama-b5000-bin-ubuntu-x64-cuda.zip
cd llama-b5000-bin-ubuntu-x64-cuda/bin
./llama-cli --version
```

Choose the right binary for your platform and GPU backend.

### 2) Package managers

```bash
# macOS / Linux (Homebrew)
brew install llama.cpp

# Arch Linux
pacman -S llama.cpp

# Nix
nix run nixpkgs#llama-cpp

# Windows (winget)
winget install llama.cpp
```

### 3) Build from source

For custom backends or latest features, build from source. See the [official build docs](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md) for detailed instructions.

Quick start:

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release -j
# Binaries in build/bin/
```

For GPU support, add flags like `-DGGML_CUDA=ON` (NVIDIA) or `-DGGML_METAL=ON` (Apple).

### 4) Get a model

Download a GGUF model from Hugging Face or use the built-in downloader:

```bash
# Direct download from Hugging Face
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF

# Or download manually and use local file
llama-cli -m ./my-model-Q4_K_M.gguf -p "Hello, world!"
```

Popular model sources:

- [TheBloke](https://huggingface.co/TheBloke) — extensive GGUF collection
- [ggml-org](https://huggingface.co/ggml-org) — official reference models
- [bartowski](https://huggingface.co/bartowski) — high-quality quantizations

---

## Beginner usage

1. Download a model (Q4_K_M is a good balance of size/quality)
2. Run with a prompt:

```bash
llama-cli -m model.gguf -p "Explain Bitcoin in one sentence"
```

1. For chat, use interactive mode:

```bash
llama-cli -m model.gguf -cnv
```

---

## Pro usage

### Key flags

```bash
llama-cli -m <model.gguf>   # Model file (required)
  -p "prompt"                # Input prompt
  -n 256                     # Max tokens to generate
  -c 4096                    # Context size (memory usage)
  -t 8                       # CPU threads
  -ngl 99                    # GPU layers (offload to VRAM)
  --temp 0.7                 # Temperature (creativity)
  --top-p 0.9                # Nucleus sampling
  --top-k 40                 # Top-k sampling
  --repeat-penalty 1.1       # Repetition penalty
  -cnv                       # Conversation/chat mode
  -i                         # Interactive mode
  --color                    # Colorized output
```

### GPU acceleration

```bash
# Offload all layers to GPU (if VRAM allows)
llama-cli -m model.gguf -ngl 99 -p "Hello"

# Partial offload (hybrid CPU+GPU)
llama-cli -m model.gguf -ngl 20 -p "Hello"

# Check which backend is active
llama-cli --version
```

For multi-GPU:

```bash
# Split across GPUs (CUDA)
llama-cli -m model.gguf -ngl 99 --tensor-split 0.5,0.5
```

### Interactive mode

```bash
# Chat with the model
llama-cli -m model.gguf -cnv --color

# With system prompt
llama-cli -m model.gguf -cnv \
  --system "You are a helpful coding assistant."
```

In interactive mode:

- Type your message and press Enter
- Use Ctrl+C to interrupt generation
- Type `/quit` or Ctrl+D to exit

### Batch processing

```bash
# Process a file
llama-cli -m model.gguf -f input.txt -n 500

# Pipe input
echo "Summarize: local AI" | llama-cli -m model.gguf -n 200
```

### Download models from Hugging Face

```bash
# Auto-download and run
llama-cli -hf Qwen/Qwen2.5-Coder-7B-Instruct-GGUF

# Specify quantization
llama-cli -hf TheBloke/Mistral-7B-Instruct-v0.2-GGUF:Q4_K_M

# Set custom endpoint (ModelScope, etc.)
export MODEL_ENDPOINT=https://modelscope.cn/api/v1
llama-cli -hf <model-id>
```

---

## Cost savings guide

- Use **smaller quantizations** (Q4_K_M, Q5_K_M) for faster inference and less memory
- **Offload to GPU** when possible—dramatically faster than CPU
- **Reduce context size** (`-c 2048`) if you don't need long conversations
- Run **one model at a time** on limited hardware
- Pre-download models to avoid repeated network transfers

---

## Privacy guide

- All inference is **100% local**—no data leaves your machine
- Models are static files; no telemetry or network calls during inference
- Store models on encrypted volumes for sensitive deployments
- Consider air-gapped setups for maximum security

---

## Security guide

- Download models only from **trusted sources** (official repos, known quantizers)
- Verify checksums when available
- Run llama-cli as a **non-root user**
- If exposing via scripts, sanitize inputs to prevent prompt injection

---

## Appendix

**Common commands**

```bash
llama-cli -m model.gguf -p "prompt"        # One-shot generation
llama-cli -m model.gguf -cnv               # Chat mode
llama-cli -m model.gguf -ngl 99            # Full GPU offload
llama-cli -hf <repo>/<model>               # Download and run from HF
llama-cli --help                           # Full flag reference
```

**Environment variables**

```bash
MODEL_ENDPOINT=https://...      # Custom model download endpoint
GGML_CUDA_VISIBLE_DEVICES=0,1   # Limit visible GPUs (CUDA)
```

**See also**

- [llamacpp-server.md](./llamacpp-server.md) — run as an API server
- [llamacpp-quantization.md](./llamacpp-quantization.md) — quantize your own models
- [Official docs](https://github.com/ggml-org/llama.cpp/tree/master/docs)
