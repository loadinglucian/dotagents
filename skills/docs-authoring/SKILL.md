---
name: docs-authoring
description: Write and edit user-facing documentation with Laravel-style structure, reader-first tone, progressive examples, and consistent markdown formatting. Use when updating docs markdown files (including README files), especially when creating tutorials, feature docs, and conceptual guides.
---

# Docs Authoring

## Overview

Produce clear documentation that explains context first, then implementation.
Use direct reader language, progressive examples, and predictable section structure.

## Workflow

### Step 1: Identify document type

Classify the target document:
- Feature documentation
- Concept/reference documentation
- Quickstart/tutorial documentation
- README overview documentation

### Step 2: Build structure

For substantial docs, use:
1. H1 title
2. table of contents
3. introduction with context/problem
4. quickstart example (when feature-oriented)
5. H2/H3 sections in simple-to-complex order

Use semantic anchors for sections when needed.

### Step 3: Apply writing style

- Address the reader as `you` and `your`.
- Prefer active voice.
- Explain why before how.
- Keep paragraphs to 2-4 sentences.
- Use contractions for conversational tone.
- Avoid dismissive words like `just` and `simply`.

### Step 4: Add examples and references

- Start with a minimal working example.
- Add complexity incrementally.
- Use realistic names (`User`, `Post`, `Order`, etc.).
- Add cross-links to related documentation when it helps execution.

### Step 5: Add callouts sparingly

Use only where high-signal:
- `NOTE` for helpful context
- `WARNING` for hazards/security concerns

### Step 6: Format and quality gate

Run Prettier for docs using the detected JS package manager.
Use the quality report template from references.

## References

- `references/structure-and-style.md`
- `references/phrase-patterns.md`
- `references/code-and-callout-patterns.md`
- `references/quality-gate.md`

## Output Requirements

When authoring docs with this skill:
1. Deliver updated markdown content.
2. Include a short quality gate report (PASS/FAIL checks).
3. Mention formatting command used (or why it was skipped).

## Constraints

- Never open with code before context.
- Never introduce advanced usage before basic usage.
- Never use generic placeholder names (`Foo`, `Bar`, `Thing`, `Item`).
- Never use em-dash characters; use commas, colons, or parentheses.
