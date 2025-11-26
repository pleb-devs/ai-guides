# Maple Document Upload — Private File Analysis

## Index
- [Overview](#overview)
- [Setup](#setup)
  - [Requirements](#requirements)
  - [Enable uploads](#enable-uploads)
- [Beginner usage](#beginner-usage)
  - [Attach a document](#attach-a-document)
  - [Example prompts](#example-prompts)
- [Pro usage](#pro-usage)
  - [Workflow tips](#workflow-tips)
  - [Model pairing](#model-pairing)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Troubleshooting](#troubleshooting)
- [Sources](#sources)

## Overview

Maple lets you send one document per message for encrypted, enclave-backed analysis (summaries, translations, code review, slide notes). Available on desktop, iOS, and Android; web support for uploads is coming soon.

## Setup

### Requirements
- Plan: document upload is available on Pro and Team plans.
- Apps: update Maple on desktop or mobile to the latest version.

### Enable uploads
No switches required—tap paperclip → **Document** inside a chat.

## Beginner usage

### Attach a document
- Supported types: PDF, DOC/DOCX, TXT, RTF, XLS/XLSX, PPT/PPTX, MD. Max size 5MB. One file per reply.
- Add a clear instruction in the same message (e.g., “summarize each section in 3 bullets”).

### Example prompts
- “Summarize this report and highlight risks.”  
- “Translate this contract to English and pull all dates.”  
- “Turn these slides into a meeting agenda.”

## Pro usage

### Workflow tips
- Chunk long documents into ≤5MB parts and send sequentially; keep a running thread for context.
- After the first pass, follow up with clarifying questions instead of re-uploading to save time and tokens. (Inference)

### Model pairing
- Start with a smaller model for extraction, then switch to a stronger reasoning model (e.g., GPT-OSS 120B or DeepSeek R1) for synthesis. (Inference)
- Vision models are not needed for documents unless you embed screenshots; stick to text models for lower cost. (Inference)

## Cost savings guide
- Compress or split large PDFs to stay under 5MB and reduce tokenization.
- Keep instructions concise; reuse the same thread to avoid re-uploading context. (Inference)
- Pro is required; only upgrade seats that need uploads—others can stay on Starter.

## Privacy guide
- Documents are encrypted end to end and processed inside secure enclaves; Maple states no clear-text logs.
- One-file and size limits help minimize accidental oversharing.
- Maple states that it does not train on your documents or share their contents with model providers; they are only used to answer your questions.
- Maple still tracks high-level usage and billing metadata (like counts and timestamps), but document content itself stays encrypted.

## Security guide
- Avoid uploading secrets you don’t need; redact or mask sensitive fields first. (General best practice)
- Use organization accounts for work data so access is scoped to your team. (Inference)

## Troubleshooting
- “Upload failed”: ensure file type is supported and ≤5MB; only one file per message.
- “Feature missing on web”: web uploads are rolling out later—use desktop or mobile.
- “Got partial summary”: for long docs, split into sections and send in order; reference earlier parts to maintain context.

- Maple has indicated plans to expand uploads over time (larger limits, multi-file workflows, and integrations like cloud storage); check product updates for capabilities beyond the current 1-file / 5MB per-message design.

## Sources

- [Introducing Confidential Document and Image Upload in Maple AI][1]

[1]: https://blog.trymaple.ai/introducing-confidential-document-and-image-upload-in-maple-ai/
