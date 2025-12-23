# Maple App Quickstart (Web, Desktop, iOS)

## Index

- [Overview](#overview)
- [Setup](#setup)
  - [Create and sign in](#create-and-sign-in)
  - [Install apps](#install-apps)
  - [Pick a plan](#pick-a-plan)
- [Beginner usage](#beginner-usage)
  - [Start a chat](#start-a-chat)
  - [Switch models](#switch-models)
  - [Use voice](#use-voice)
  - [Upload an image or document](#upload-an-image-or-document)
- [Pro usage](#pro-usage)
  - [Team accounts](#team-accounts)
  - [Cross-device flow](#cross-device-flow)
  - [Model selection tips](#model-selection-tips)
  - [Live Data](#live-data)
  - [Image workflows](#image-workflows)
  - [Document workflows](#document-workflows)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Troubleshooting](#troubleshooting)
- [Sources](#sources)

## Overview

Maple is a privacy-first AI chat app with end-to-end encryption, secure enclaves for processing, and encrypted sync across devices. It now supports web, desktop, and mobile (iOS/Android) with shared chat history and model access.

## Setup

### Create and sign in

- Sign up at `https://trymaple.ai` with a free account, then log in on each device; chats sync automatically.

### Install apps

- Desktop: download for macOS or Linux from `trymaple.ai/downloads`.
- Mobile: install the iOS or Android app (document/image upload is available there; web support is coming soon for uploads).
- Web: open `https://trymaple.ai` and sign in; voice works on web, desktop, and iOS for Pro+.

### Pick a plan

- As of Nov 26, 2025: Free (10 chats/week), Starter ($5.99/mo), Pro ($20/mo), Team ($30/mo/seat).
- File uploads and the best experience for vision/doc tasks require Pro or Team.
- For the latest plan details and per-plan feature matrix (models, uploads, voice, Live Data), always refer to Maple's current pricing and model guide pages.

## Beginner usage

### Start a chat

Open Maple, select or create a conversation, and type your prompt. Chats stay encrypted and sync between devices.

### Switch models

Tap the model name in the chat header and choose a different model (e.g., GPT-OSS 120B, DeepSeek R1, Qwen, Gemma, Mistral).

### Use voice

Tap the microphone icon, speak, then send; Maple transcribes and replies with text and optional audio. Requires Pro, Team, or Max. Works on web, desktop, and iOS today.

### Upload an image or document

- Tap the paperclip/camera, choose **Image** or **Document**, and send one file (≤5MB).
- Supported images: JPG, PNG, WEBP. Supported documents: PDF, DOC/DOCX, TXT, RTF, XLS/XLSX, PPT/PPTX, MD.
- Add clear instructions in the same message (for example, "summarize each section in 3 bullets," "extract all dates," or "describe this slide in two bullet points").
- Great for translations, summaries, code reviews, slide notes, and quick data extraction.

## Pro usage

### Team accounts

- Buy seats (minimum 2), invite members, and chat with encrypted AI. Use the standard payment methods shown in Maple's billing settings; alternative methods such as crypto may not be supported.
- Team owners and admins manage seats and billing in a shared workspace, while each member keeps their own encrypted account and device keys.

### Cross-device flow

- Start on desktop, continue on mobile, finish on web—history and keys stay encrypted during sync.

### Model selection tips

- Free users see limited models; Starter+ unlocks vision models; Pro/Team unlock document and image uploads plus all models.

### Live Data

- Turn on Live Data in a chat when you need up-to-date answers; Maple will privately fetch results from the web and combine them with the selected model.
- Live Data uses the same encrypted stack as normal chats, and Maple does not use your browsing queries or results to train models.

### Image workflows

- Use image uploads for things like slide summaries, UI reviews, and text-in-image extraction; keep files under 5MB and crop or resize to what you actually need.
- Attach only one image per message and include context (for example, "translate the sign and list all numbers you see") so the model knows what to focus on.
- Start with a vision-capable model (such as a Gemma or Mistral vision variant) to interpret the image, then switch to a cheaper text model in follow-up turns for additional Q&A.

### Document workflows

- For long PDFs and decks, split into ≤5MB chunks and send them sequentially in the same thread; ask for sectioned summaries to keep things organized.
- After an initial summary, refine your understanding with follow-up questions instead of re-uploading the file (for example, "compare sections 2 and 3 on risk").
- Start with a smaller, faster model for extraction, then switch to a stronger reasoning model (like GPT-OSS 120B or DeepSeek R1) when you need deep analysis or synthesis.

## Cost savings guide

- Stay on Free for light use (10 chats/week); upgrade only when you need uploads or higher limits.
- Pick smaller models (e.g., Mistral Small) for quick replies; reserve heavier models (DeepSeek R1, GPT-OSS 120B) for tough tasks.
- Keep uploads under 5MB; trim or crop images and split long PDFs to avoid retries.
- For uploads, compress or split large PDFs and images to reduce tokenization and avoid hitting per-message limits.
- Pro/Team plans are required for document and image uploads; only upgrade seats that actually need uploads—others can stay on Starter.

## Privacy guide

- Chats, uploads, and voice are encrypted on-device and processed inside secure enclaves; no clear-text logging.
- Sync uses encrypted storage so history moves between devices without exposing content.
- Maple states that it does not train on your conversations or share your data with model providers; your content is only used to serve your requests.
- Maple still tracks basic usage and billing metadata (like request counts and timestamps) to keep the service running, but message content remains encrypted end to end.

## Security guide

- Upload limits (one file, 5MB) reduce blast radius and keep parsing deterministic.
- Use strong device auth (passcode/biometrics) because encryption keys live on your devices; avoid sharing accounts across users. (General best practice; no external citation needed.)
- For team billing, use the payment methods supported in Maple's billing UI and prefer traceable rails for accounting.
- Maple's backend is built from open-source components with reproducible builds, so organizations can verify that the code running in secure enclaves matches the published source when checking attestation.

## Troubleshooting

- Upload fails: confirm size ≤5MB and one file per message; check supported extensions.
- Model not available: feature may require Starter+ (vision) or Pro/Team (uploads); upgrade or switch plans.
- Voice icon missing: ensure Pro/Team/Max and use web, desktop, or iOS.
- Upload feature missing on web: uploads are rolling out on the web client; if you do not see the option, use desktop or mobile.
- Got a partial summary from a document: split long files into smaller sections and send them in order, referring back to earlier parts as needed.

## Sources

- [Getting Started with Maple AI][1]
- [Introducing Confidential Document and Image Upload in Maple AI][2]
- [Speak Freely – Encrypted Voice Arrives in Maple][3]
- [Get Started with a Team in Maple][4]
- [Maple AI Model Guide with Example Prompts][5]

[1]: https://blog.trymaple.ai/getting-started-with-maple-ai-a-step-by-step-guide/
[2]: https://blog.trymaple.ai/introducing-confidential-document-and-image-upload-in-maple-ai/
[3]: https://blog.trymaple.ai/speak-freely-encrypted-voice-arrives-in-maple/
[4]: https://blog.trymaple.ai/get-started-with-a-team-in-maple/
[5]: https://blog.trymaple.ai/maple-ai-model-guide-with-example-prompts/
