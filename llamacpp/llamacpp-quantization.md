# llama.cpp Quantization — PlebDevs AI Guide

> Quantize models to reduce size and increase inference speed. Convert Hugging Face models to GGUF, choose quantization levels, and optimize for your hardware.

---

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Get the tools](#1-get-the-tools)
  - [Prepare a model](#2-prepare-a-model)
- [Beginner usage](#beginner-usage)
- [Pro usage](#pro-usage)
  - [Quantization types explained](#quantization-types-explained)
  - [Convert from Hugging Face](#convert-from-hugging-face)
  - [Quantize with llama-quantize](#quantize-with-llama-quantize)
  - [Importance matrix quantization](#importance-matrix-quantization)
  - [Verify and test](#verify-and-test)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Appendix](#appendix)

---

## Overview

**Quantization** reduces model precision (e.g., 16-bit → 4-bit) to:

- **Shrink file size** (7B model: ~14GB FP16 → ~4GB Q4)
- **Reduce VRAM usage** (fit larger models on consumer GPUs)
- **Speed up inference** (less data to move)

llama.cpp uses the **GGUF format**—a single-file format containing model weights, tokenizer, and metadata.

---

## Setup

### 1) Get the tools

Build llama.cpp from source to get quantization tools:

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release -j

# Tools available:
# build/bin/llama-quantize
# Also need Python for conversion
pip install -r requirements.txt
```

### 2) Prepare a model

You need either:

- A Hugging Face model (will be converted to GGUF)
- An existing GGUF file (can be re-quantized)

---

## Beginner usage

**Quick quantization from an existing GGUF:**

```bash
# Quantize to Q4_K_M (good balance)
./build/bin/llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M
```

**Common workflow:**

1. Download a model from Hugging Face
2. Convert to GGUF (if not already)
3. Quantize to desired level
4. Test with llama-cli

---

## Pro usage

### Quantization types explained

| Type | Bits | Size (7B) | Quality | Speed | Use Case |
|------|------|-----------|---------|-------|----------|
| F32 | 32 | ~28GB | Best | Slow | Reference only |
| F16 | 16 | ~14GB | Excellent | Slow | High-end GPUs |
| Q8_0 | 8 | ~7GB | Very good | Fast | When you have RAM |
| Q6_K | 6 | ~5.5GB | Great | Fast | Quality-focused |
| Q5_K_M | 5 | ~4.8GB | Very good | Fast | Good balance |
| **Q4_K_M** | 4 | ~4GB | Good | Fast | **Recommended default** |
| Q4_K_S | 4 | ~3.8GB | Good | Fast | Slightly smaller |
| Q3_K_M | 3 | ~3GB | Acceptable | Faster | Limited RAM |
| Q2_K | 2 | ~2.5GB | Degraded | Fastest | Extreme constraints |
| IQ4_XS | ~4 | ~3.6GB | Good | Fast | i-quants, smaller |
| IQ3_XXS | ~3 | ~2.6GB | Fair | Fast | Aggressive compression |

**Recommendations:**

- **Q4_K_M**: Best all-around choice for most users
- **Q5_K_M**: When quality matters more than size
- **Q6_K / Q8_0**: Maximum quality within quantization
- **Q3_K_M / IQ3_XXS**: When RAM is severely limited
- **Avoid Q2_K** unless absolutely necessary (significant quality loss)

### Convert from Hugging Face

```bash
# Download model (or use local path)
# Example: meta-llama/Llama-3.2-3B-Instruct

# Convert to GGUF (F16)
python convert_hf_to_gguf.py \
  /path/to/hf-model \
  --outfile model-f16.gguf \
  --outtype f16

# Or directly to a quantized format
python convert_hf_to_gguf.py \
  /path/to/hf-model \
  --outfile model-Q8_0.gguf \
  --outtype q8_0
```

### Quantize with llama-quantize

```bash
# Basic quantization
./build/bin/llama-quantize input.gguf output.gguf <type>

# Examples
./build/bin/llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M
./build/bin/llama-quantize model-f16.gguf model-Q5_K_M.gguf Q5_K_M
./build/bin/llama-quantize model-f16.gguf model-Q8_0.gguf Q8_0

# With specific thread count
./build/bin/llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M 8
```

### Importance matrix quantization

Use an importance matrix (imatrix) for better quality at low bit-widths:

```bash
# Generate importance matrix from calibration data
./build/bin/llama-imatrix \
  -m model-f16.gguf \
  -f calibration_data.txt \
  -o imatrix.dat

# Quantize with importance matrix
./build/bin/llama-quantize \
  --imatrix imatrix.dat \
  model-f16.gguf \
  model-IQ4_XS.gguf \
  IQ4_XS
```

Calibration data should be representative text (Wikipedia excerpts, code samples, etc.).

### Verify and test

```bash
# Check the quantized model
./build/bin/llama-cli -m model-Q4_K_M.gguf \
  -p "The meaning of life is" -n 50

# Compare outputs
./build/bin/llama-cli -m model-f16.gguf -p "Hello" -n 20
./build/bin/llama-cli -m model-Q4_K_M.gguf -p "Hello" -n 20

# Measure perplexity (quality metric)
./build/bin/llama-perplexity -m model-Q4_K_M.gguf -f test.txt
```

---

## Cost savings guide

- **Don't over-quantize**: Q4_K_M is usually the sweet spot
- **Batch quantize**: Create multiple quantization levels and test before deploying
- **Use imatrix** for Q3 and below—improves quality significantly
- **Share GGUF files**: Pre-quantized models save compute for everyone
- Many quantized models are already available on Hugging Face—check before quantizing yourself

---

## Privacy guide

- Quantization happens locally—no data sent anywhere
- The process only uses the model weights (no user data involved)
- Quantized models can be shared without privacy concerns (they're just compressed weights)

---

## Security guide

- Verify source models before conversion (check hashes, use trusted repos)
- GGUF files can contain arbitrary metadata—inspect before running untrusted files:

```bash
# Inspect GGUF metadata
python -c "
from gguf import GGUFReader
reader = GGUFReader('model.gguf')
for key in reader.fields:
    print(f'{key}: {reader.fields[key].data}')
"
```

- Store original FP16/F32 models if you need to re-quantize later

---

## Appendix

**Quantization decision tree:**

```
Have 24GB+ VRAM? → Q6_K or Q8_0
Have 8-16GB VRAM? → Q4_K_M or Q5_K_M
Have 4-8GB VRAM? → Q4_K_S or Q3_K_M
Have <4GB VRAM? → IQ3_XXS or Q2_K (with caveats)
```

**Quick commands**

```bash
# Convert HF → GGUF
python convert_hf_to_gguf.py ./hf-model --outfile model.gguf

# Quantize
./llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M

# Test
./llama-cli -m model-Q4_K_M.gguf -p "Hello" -n 50
```

**File size reference (7B model)**

| Quantization | Approximate Size |
|--------------|------------------|
| F16 | 14 GB |
| Q8_0 | 7 GB |
| Q6_K | 5.5 GB |
| Q5_K_M | 4.8 GB |
| Q4_K_M | 4.0 GB |
| Q3_K_M | 3.0 GB |
| IQ3_XXS | 2.6 GB |

**See also**

- [llamacpp-cli.md](./llamacpp-cli.md) — run quantized models
- [llamacpp-server.md](./llamacpp-server.md) — serve quantized models
- [llama.cpp quantize README](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md)
