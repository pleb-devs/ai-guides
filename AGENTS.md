# Repository Guidelines

## Project Structure & Module Organization
All published guides live at the repository root as Markdown files named after the tool, e.g., `opencode.md`. Keep filenames lowercase with hyphens and match the title case inside the document. `README.md` serves as the index and must list every guide you add or retire. Use the `goose/` directory for scratchpads or staging assets that have not been promoted; once guidance is ready, move it into a root-level Markdown file and remove the draft.

## Build, Test, and Development Commands
Run ad-hoc linting before pushing:

```bash
npx markdownlint-cli2 "**/*.md"
npx prettier --check "**/*.md"
```

`markdownlint-cli2` enforces heading order and list spacing, while `prettier --check` confirms consistent wrapping. When drafting new content, preview locally with any Markdown viewer (for example, `code opencode.md`) to confirm images render from their canonical URLs.

## Coding Style & Naming Conventions
Use ATX headings (`#`, `##`, `###`) and keep sections under roughly 120 words to stay scannable. Prefer second-person instructional voice and active verbs. Highlight callouts with bold text (e.g., **Tip**) instead of blockquotes. Indent nested lists by two spaces and leave a blank line before fenced code blocks. Store external links as inline Markdown; avoid raw URLs in the prose.

## Testing Guidelines
Treat linting as the minimum gate. When a guide references shell commands, execute them in a scratch environment to confirm they work or annotate prerequisites. Validate external links with `npx markdown-link-check README.md` (swap in the file you touched). Note the last validation date inside the affected section if the tool’s pricing or capabilities change frequently.

## Commit & Pull Request Guidelines
Follow the existing history by keeping commit subjects short Title Case statements (e.g., `Update README.md`). Group related edits into a single commit so reviewers can skim the diff per guide. For pull requests, include a bullet summary of the guide updates, a checklist of verification commands (`markdownlint`, `prettier`, link checks), and links to upstream docs you referenced. Add screenshots only when UI changes are central to the update; otherwise link to canonical assets. Request review when you introduce new workflows, pricing guidance, or security recommendations.
