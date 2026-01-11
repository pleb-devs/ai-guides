# Code QA with Project Docs — Goose Recipe

A deep code review recipe that loads your project documentation before reviewing code. Designed for projects using the `llm/` directory structure from [new-project-boilerplate](https://github.com/pleb-devs/new-project-boilerplate).

## Overview

Standard code reviews catch syntax errors and obvious bugs. This recipe goes deeper by first loading your project's documentation—phase requirements, implementation notes, coding standards—then comparing the actual code against what was planned and documented.

**What it catches:**

- Low-hanging fruit (syntax, unused imports, naming issues)
- Discrepancies between docs and actual code
- Violations of your project's coding standards
- Implementation drift from phase requirements
- Security issues in sensitive code (crypto, auth, etc.)

## Prerequisites

Your project should have the `llm/` directory structure:

```text
your-project/
└── llm/
    ├── project/           # Requirements & standards
    │   ├── phases/        # Phase requirements (setup-phase.md, mvp-phase.md)
    │   └── project-rules.md
    ├── context/           # Reference docs (crypto specs, protocol specs)
    ├── implementation/    # What was actually built ([phase]-implementation.md)
    └── workflows/         # Operational runbooks
```

See [new-project-boilerplate](https://github.com/pleb-devs/new-project-boilerplate) for the full structure.

## How It Works

**Before reviewing code**, Goose:

1. Reads phase requirements (`llm/project/phases/[phase].md`) to understand what SHOULD be built
2. Reads implementation notes (`llm/implementation/[phase]-implementation.md`) to see what WAS documented
3. Skims coding standards (`llm/project/project-rules.md`) for conventions
4. For sensitive code, reads relevant context docs (`llm/context/`)

**During review**, Goose:

- Identifies easy wins (syntax, unused code, naming)
- Flags where code doesn't match implementation docs
- Checks compliance with project-rules.md
- Verifies sensitive implementations against context specs

## The Recipe

### JSON

```json
{
  "version": "1.0.0",
  "title": "Code QA with Docs",
  "description": "Deep code review that loads project documentation first, verifying implementation against requirements and standards",
  "instructions": "You are an AI assistant specialized in deep code reviews that verify implementation against project documentation.\n\n## Before Reviewing Code\n\nFirst, load context from the llm/ documentation:\n\n1. **Load phase requirements**: Read `@llm/project/phases/[current-phase].md` to understand what SHOULD have been built\n2. **Load implementation notes**: Read `@llm/implementation/[current-phase]-implementation.md` to see what WAS documented as built\n3. **Load standards**: Skim `@llm/project/project-rules.md` for code conventions\n4. **For sensitive code**: Read relevant docs from `@llm/context/` (crypto specs, protocol docs, etc.)\n\n### LLM Directory Quick Reference\n\n- `llm/project/` - Requirements, architecture, standards\n- `llm/context/` - Protocol specs, crypto model, library docs\n- `llm/implementation/` - What was actually built\n- `llm/workflows/` - Operational runbooks\n\n## During Review\n\nYour role is to identify issues, with special attention to doc-vs-code alignment:\n\n- Syntax errors, unused imports/variables\n- Naming problems, obvious bugs\n- Simple security or performance hints\n- **Discrepancies between docs and actual code**\n- **Violations of project-rules.md standards**\n- **Implementation errors in sensitive code** (verify against context/ specs)\n\n## Guidelines\n\n1. **Focus on easy wins** - Skip deep architectural reviews\n2. **Keep it short and actionable** - Bullet-point list with clear explanations\n3. **Provide line numbers** - Reference specific lines when useful\n4. **Show minimal code snippets** - Tiny diffs only, not whole files\n5. **Compare against docs** - Flag if code doesn't match implementation notes\n6. **Check sensitive code carefully** - Verify against llm/context/ specs\n7. **Tone** - Friendly, professional, helpful\n8. **If no issues found** - Respond with \"No issues found. Implementation matches documentation.\"\n",
  "extensions": [],
  "activities": [],
  "parameters": []
}
```

### Ready-to-Use URI

```text
goose://recipe?config=ewogICJ2ZXJzaW9uIjogIjEuMC4wIiwKICAidGl0bGUiOiAiQ29kZSBRQSB3aXRoIERvY3MiLAogICJkZXNjcmlwdGlvbiI6ICJEZWVwIGNvZGUgcmV2aWV3IHRoYXQgbG9hZHMgcHJvamVjdCBkb2N1bWVudGF0aW9uIGZpcnN0LCB2ZXJpZnlpbmcgaW1wbGVtZW50YXRpb24gYWdhaW5zdCByZXF1aXJlbWVudHMgYW5kIHN0YW5kYXJkcyIsCiAgImluc3RydWN0aW9ucyI6ICJZb3UgYXJlIGFuIEFJIGFzc2lzdGFudCBzcGVjaWFsaXplZCBpbiBkZWVwIGNvZGUgcmV2aWV3cyB0aGF0IHZlcmlmeSBpbXBsZW1lbnRhdGlvbiBhZ2FpbnN0IHByb2plY3QgZG9jdW1lbnRhdGlvbi5cblxuIyMgQmVmb3JlIFJldmlld2luZyBDb2RlXG5cbkZpcnN0LCBsb2FkIGNvbnRleHQgZnJvbSB0aGUgbGxtLyBkb2N1bWVudGF0aW9uOlxuXG4xLiAqKkxvYWQgcGhhc2UgcmVxdWlyZW1lbnRzKio6IFJlYWQgYEBsbG0vcHJvamVjdC9waGFzZXMvW2N1cnJlbnQtcGhhc2VdLm1kYCB0byB1bmRlcnN0YW5kIHdoYXQgU0hPVUxEIGhhdmUgYmVlbiBidWlsdFxuMi4gKipMb2FkIGltcGxlbWVudGF0aW9uIG5vdGVzKio6IFJlYWQgYEBsbG0vaW1wbGVtZW50YXRpb24vW2N1cnJlbnQtcGhhc2VdLWltcGxlbWVudGF0aW9uLm1kYCB0byBzZWUgd2hhdCBXQVMgZG9jdW1lbnRlZCBhcyBidWlsdFxuMy4gKipMb2FkIHN0YW5kYXJkcyoqOiBTa2ltIGBAbGxtL3Byb2plY3QvcHJvamVjdC1ydWxlcy5tZGAgZm9yIGNvZGUgY29udmVudGlvbnNcbjQuICoqRm9yIHNlbnNpdGl2ZSBjb2RlKio6IFJlYWQgcmVsZXZhbnQgZG9jcyBmcm9tIGBAbGxtL2NvbnRleHQvYCAoY3J5cHRvIHNwZWNzLCBwcm90b2NvbCBkb2NzLCBldGMuKVxuXG4jIyMgTExNIERpcmVjdG9yeSBRdWljayBSZWZlcmVuY2VcblxuLSBgbGxtL3Byb2plY3QvYCAtIFJlcXVpcmVtZW50cywgYXJjaGl0ZWN0dXJlLCBzdGFuZGFyZHNcbi0gYGxsbS9jb250ZXh0L2AgLSBQcm90b2NvbCBzcGVjcywgY3J5cHRvIG1vZGVsLCBsaWJyYXJ5IGRvY3Ncbi0gYGxsbS9pbXBsZW1lbnRhdGlvbi9gIC0gV2hhdCB3YXMgYWN0dWFsbHkgYnVpbHRcbi0gYGxsbS93b3JrZmxvd3MvYCAtIE9wZXJhdGlvbmFsIHJ1bmJvb2tzXG5cbiMjIER1cmluZyBSZXZpZXdcblxuWW91ciByb2xlIGlzIHRvIGlkZW50aWZ5IGlzc3Vlcywgd2l0aCBzcGVjaWFsIGF0dGVudGlvbiB0byBkb2MtdnMtY29kZSBhbGlnbm1lbnQ6XG5cbi0gU3ludGF4IGVycm9ycywgdW51c2VkIGltcG9ydHMvdmFyaWFibGVzXG4tIE5hbWluZyBwcm9ibGVtcywgb2J2aW91cyBidWdzXG4tIFNpbXBsZSBzZWN1cml0eSBvciBwZXJmb3JtYW5jZSBoaW50c1xuLSAqKkRpc2NyZXBhbmNpZXMgYmV0d2VlbiBkb2NzIGFuZCBhY3R1YWwgY29kZSoqXG4tICoqVmlvbGF0aW9ucyBvZiBwcm9qZWN0LXJ1bGVzLm1kIHN0YW5kYXJkcyoqXG4tICoqSW1wbGVtZW50YXRpb24gZXJyb3JzIGluIHNlbnNpdGl2ZSBjb2RlKiogKHZlcmlmeSBhZ2FpbnN0IGNvbnRleHQvIHNwZWNzKVxuXG4jIyBHdWlkZWxpbmVzXG5cbjEuICoqRm9jdXMgb24gZWFzeSB3aW5zKiogLSBTa2lwIGRlZXAgYXJjaGl0ZWN0dXJhbCByZXZpZXdzXG4yLiAqKktlZXAgaXQgc2hvcnQgYW5kIGFjdGlvbmFibGUqKiAtIEJ1bGxldC1wb2ludCBsaXN0IHdpdGggY2xlYXIgZXhwbGFuYXRpb25zXG4zLiAqKlByb3ZpZGUgbGluZSBudW1iZXJzKiogLSBSZWZlcmVuY2Ugc3BlY2lmaWMgbGluZXMgd2hlbiB1c2VmdWxcbjQuICoqU2hvdyBtaW5pbWFsIGNvZGUgc25pcHBldHMqKiAtIFRpbnkgZGlmZnMgb25seSwgbm90IHdob2xlIGZpbGVzXG41LiAqKkNvbXBhcmUgYWdhaW5zdCBkb2NzKiogLSBGbGFnIGlmIGNvZGUgZG9lc24ndCBtYXRjaCBpbXBsZW1lbnRhdGlvbiBub3Rlc1xuNi4gKipDaGVjayBzZW5zaXRpdmUgY29kZSBjYXJlZnVsbHkqKiAtIFZlcmlmeSBhZ2FpbnN0IGxsbS9jb250ZXh0LyBzcGVjc1xuNy4gKipUb25lKiogLSBGcmllbmRseSwgcHJvZmVzc2lvbmFsLCBoZWxwZnVsXG44LiAqKklmIG5vIGlzc3VlcyBmb3VuZCoqIC0gUmVzcG9uZCB3aXRoIFwiTm8gaXNzdWVzIGZvdW5kLiBJbXBsZW1lbnRhdGlvbiBtYXRjaGVzIGRvY3VtZW50YXRpb24uXCJcbiIsCiAgImV4dGVuc2lvbnMiOiBbXSwKICAiYWN0aXZpdGllcyI6IFtdLAogICJwYXJhbWV0ZXJzIjogW10KfQ==
```

## Usage Tips

1. **Specify the phase** — When invoking the recipe, tell Goose which phase you're reviewing (e.g., "Review the setup phase implementation").

2. **Point to specific files** — For targeted reviews, provide file paths: "Review `src/crypto/encrypt.ts` against the setup phase docs."

3. **Keep docs updated** — The recipe is only as good as your documentation. Update `llm/implementation/` notes after each phase.

4. **Use for PR reviews** — Run the recipe on changed files before merging to catch drift early.

## Customization

To adapt this recipe for a different doc structure:

1. Copy the JSON above
2. Update the paths in the instructions (e.g., change `llm/` to `docs/` or your custom path)
3. Adjust the "LLM Directory Quick Reference" section to match your folders
4. Re-encode to base64 for a new URI: `cat recipe.json | base64 | tr -d '\n'`
