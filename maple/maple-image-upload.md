# Maple Image Upload — Private Vision Features

## Index
- [Overview](#overview)
- [Setup](#setup)
  - [Requirements](#requirements)
  - [Enable uploads](#enable-uploads)
- [Beginner usage](#beginner-usage)
  - [Attach an image](#attach-an-image)
  - [Example prompts](#example-prompts)
- [Pro usage](#pro-usage)
  - [Pick the right model](#pick-the-right-model)
  - [Cross-device tips](#cross-device-tips)
- [Cost savings guide](#cost-savings-guide)
- [Privacy guide](#privacy-guide)
- [Security guide](#security-guide)
- [Troubleshooting](#troubleshooting)
- [Sources](#sources)

## Overview

Maple lets you send a single image (up to 5MB) to get private, on-device-encrypted vision analysis, processed inside hardware enclaves. Available on desktop, iOS, and Android; web support for uploads is coming soon.

## Setup

### Requirements
- Plan: image uploads are available on Pro and Team plans; Free and Starter accounts need to upgrade to use uploads.
- Apps: update Maple on desktop or mobile to the latest release.

### Enable uploads
No extra toggles—just sign in and tap the paperclip/camera icon in any chat.

## Beginner usage

### Attach an image
- Tap paperclip/camera → choose **Image** → select JPG/PNG/WEBP → send (one file, ≤5MB).
- Add context in the same message for better answers (e.g., “extract text,” “generate alt text,” “describe this chart”).

### Example prompts
- “Summarize the slide in two bullet points.”  
- “Translate the sign to English and list the numbers you see.”  
- “Find UI issues in this mock.”

## Pro usage

### Pick the right model
- Vision-capable options include Gemma and Mistral variants; select them from the model picker before sending.
- For code-in-image or math-heavy content, switch to a larger reasoning model after the initial extraction to save tokens. (Inference)

### Cross-device tips
- Capture on mobile, open the same thread on desktop to copy outputs into your IDE—history sync stays encrypted.

## Cost savings guide
- Keep files small (crop/resize) to stay under 5MB and reduce tokenization cost.
- Use vision models only for the image step; follow-up text Q&A can use a smaller model. (Inference)
- Image uploads themselves require a Pro or Team plan; Starter and Free users can still run non-vision chats without upgrading but must move to Pro/Team to attach images.

## Privacy guide
- Images are encrypted on-device and processed inside secure enclaves; Maple states there are no clear-text logs.
- One-file-per-message and size caps reduce inadvertent data spill.
- Maple states that it does not use your images to train models or share them with model providers; they are only used to fulfill your requests.
- Maple still tracks high-level usage and billing metadata (like counts and timestamps), but image content itself stays encrypted.

## Security guide
- Avoid sensitive PII unless required; redact visible secrets before upload. (General best practice)
- Stay on trusted networks when sending large media; uploads are still encrypted but this reduces metadata leakage. (General best practice)

## Troubleshooting
- “Upload failed”: ensure ≤5MB and JPG/PNG/WEBP, one file only.
- “Model unavailable”: pick a vision-capable model; some are gated to Starter+ or Pro.
- “Web can’t upload”: web image upload isn’t live yet—use desktop or mobile.

- Multi-file uploads, additional file types, and deeper integrations (such as cloud storage) are on Maple’s roadmap; watch release notes for expanded capabilities beyond the current 1-file / 5MB limit.

## Sources

- [Introducing Confidential Document and Image Upload in Maple AI][1]
- [Maple AI Model Guide with Example Prompts][2]

[1]: https://blog.trymaple.ai/introducing-confidential-document-and-image-upload-in-maple-ai/
[2]: https://blog.trymaple.ai/maple-ai-model-guide-with-example-prompts/
