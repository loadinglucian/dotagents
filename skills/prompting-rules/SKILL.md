---
name: prompting-rules
description: Write and review AI-related documentation with proven prompt-engineering patterns for structure, information order, examples, protocols, standards, and constraints. Use when creating or editing prompt docs, AGENTS.md guidance, SKILL.md files, command docs, or agent playbooks.
---

# Prompting Rules Skill

## Overview

Apply consistent prompt-engineering rules when writing AI-facing documentation.
Prioritize clear structure, explicit execution protocols, and testable quality gates.

For complete patterns and templates, read `references/prompting-reference.md`.

## Workflow

### Step 1: Classify the document type

Determine which artifact is being written:
- Generic prompt documentation
- Agent instructions
- Command instructions
- Skill instructions
- Review or verification report

### Step 2: Build a semantic structure

Use semantic headers and stable section names.
Prefer:
- `## Context`
- `## Instructions`
- `## Examples`
- `## Protocol`
- `## Standards`
- `## Constraints`
- Structured report template sections when required

### Step 3: Order information for execution quality

Place content in this order:
1. Context and background
2. Source documents/data
3. Examples
4. Instructions
5. User query or exact task (last)

### Step 4: Add examples and protocol

For complex tasks, include:
- 3-5 diverse examples
- Numbered protocol steps (`### Step N: ...`)
- A required output format or report template in a fenced block

### Step 5: Add quality bars and guardrails

End docs with:
- `## Standards` for quality expectations
- `## Constraints` for hard limits and disallowed behavior

### Step 6: Run the quality gate

Verify:
- Section semantics and consistency
- Information ordering
- Example diversity/relevance
- Protocol completeness
- Clear, direct language with minimal prose

Use the report templates in `references/prompting-reference.md` to capture PASS/FAIL checks.

## Rules

- Use imperative phrasing and short declarative sentences.
- Prefer examples and templates over long explanation.
- Use tables only for compact reference data.
- Ground analysis in quoted source text for long-context tasks.
- Break high-complexity tasks into chained prompts.

## References

- Read `references/prompting-reference.md` for full rules, templates, and examples.

## Output Requirements

When producing documentation with this skill:
1. Deliver the updated document content.
2. Include a short verification report with PASS/FAIL checks.
3. Call out any deliberate deviations from the rules.
