# Repository Guidelines

This repository hosts practical guides for coding agents and AI clients. Follow these rules to keep contributions clear, consistent, and easy to review.

## Project Structure & Module Organization
- Guides live at the repo root and in topic folders (e.g., `goose/`). Examples: `opencode.md`, `goose/goose-cli.md`, `goose/goose-desktop.md`.
- Use kebab-case filenames and nest by vendor/tool. Update `README.md` to index new guides.

## Build, Test, and Development Commands
- No build step; Markdown only. Preview with your editor’s Markdown view (e.g., VS Code: `Cmd+Shift+V`).
- Helpful checks:
  - `rg --files` — list all docs.
  - `rg -n 'TODO|TBD'` — find placeholders before submitting.

## Coding Style & Naming Conventions
- Headings: ATX (`#`), Title Case for H1, sentence case for others.
- Sections: follow the README “Guide structure” (Overview, Setup, Beginner usage, Pro usage, Cost savings, Privacy, Security; Appendix if needed).
- Lists: `-` bullets; keep bullets short and action‑oriented.
- Code: fenced blocks with language hints; commands in bash blocks.
- Links: prefer relative links within the repo; use descriptive text.

## Testing Guidelines
- Self‑check before PR:
  - Required sections exist and are ordered per the template.
  - Internal anchors work (e.g., `[Setup](#setup)`).
  - Verify local links with: `rg -n "\[(.+)\]\((?!http|#)" -- **/*.md` and fix paths.
  - Proofread; ensure images (if any) live under `assets/` and use relative paths.

## Commit & Pull Request Guidelines
- Commits: concise, imperative mood (e.g., `add goose cli flow`). Prefix `docs:` if helpful; Conventional Commits not required.
- PRs must include: 1–2 sentence summary, list of changed guides, linked issues (if any), and screenshots/GIFs when UI output matters. Keep diffs focused.

## Security & Configuration Tips
- Never commit secrets, tokens, or private URLs. Use placeholders like `YOUR_API_KEY` and sample `.env` patterns. Redact IDs in screenshots.

## Agent‑Specific Instructions
- When adding a new tool guide, mirror existing examples (`goose/*`, `opencode.md`) and add it to the “Guides” list in `README.md`.

