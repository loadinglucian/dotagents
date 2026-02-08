# Prompting Reference

## Table of Contents

1. Document Structure
2. Information Architecture
3. Examples (Multishot)
4. Reasoning Guidance
5. Role Prompting
6. Response Prefill
7. Prompt Chaining
8. Grounding in Quotes
9. Extended Reasoning
10. Writing Style
11. Verification Reports
12. Agent and Command Templates
13. Model Selection Guidance
14. Quality Gate

## Document Structure

Use markdown headers as semantic control signals.

Core rules:
- Use section names that describe function (`## Instructions`, not `## Section 1`).
- Keep section naming consistent inside each document.
- Refer to sections explicitly ("Follow the instructions above").
- Use header hierarchy for nesting: H2 -> H3 -> H4.

Common sections:

| Section | Purpose |
| --- | --- |
| `## Context` | Background information |
| `## Instructions` | Direct task directives |
| `## Examples` | Container for concrete examples |
| `### Example: {name}` | A single named example |
| `## Protocol` | Ordered execution workflow |
| `### Step N: {name}` | A required protocol step |
| `## {Name} Report` | Structured output template |
| `## Standards` | Quality requirements |
| `## Constraints` | Guardrails and limits |
| `> **IMPORTANT**` | Critical non-optional callout |

## Information Architecture

Prompt quality improves when information appears in execution order.

Use this order:
1. Role/persona (if used)
2. Context and background
3. Long documents/data
4. Examples
5. Instructions
6. Query/task (always last)

Example:

```markdown
## Context

{{BACKGROUND}}

## Documents

### Document 1: report.pdf

{{CONTENT}}

## Examples

### Example: Ticket Classification

Input: The dashboard loads slowly
Category: Performance
Sentiment: Negative

## Instructions

1. Analyze the documents above
2. Focus on X, Y, Z

What are the key findings?
```

## Examples (Multishot)

Use 3-5 diverse examples for non-trivial tasks.

Requirements:
- Relevant: match real use cases.
- Diverse: cover edge cases and avoid pattern overfitting.
- Clear: each example has its own header.

Example:

```markdown
## Examples

### Example: Performance Issue

Input: The dashboard loads slowly
Category: Performance
Sentiment: Negative
Priority: High

### Example: Positive Feedback

Input: Love the new dark mode!
Category: UI/UX
Sentiment: Positive
Priority: Low

### Example: Feature Request

Input: Please add Slack integration
Category: Feature Request
Sentiment: Neutral
Priority: Medium
```

For code examples, include correctness labels (`Correct`, `Wrong`) when relevant.

## Reasoning Guidance

Use explicit reasoning prompts only when needed.

Levels:
- Basic: "Think step-by-step."
- Guided: provide explicit reasoning steps.
- Structured: require named output sections (`## Thinking`, `## Answer`) if policy allows.

Use for:
- Math and logic tasks
- Multi-step analysis
- Complex tradeoff decisions

Skip for:
- Simple lookups
- Straight formatting tasks

## Role Prompting

Use specific roles for domain-sensitive tasks.

Pattern:
- System: role definition only
- User: task, constraints, and data

Example:

```text
System: You are a senior security engineer specializing in application security for financial services.

User: Review this authentication code for vulnerabilities:

## Code

{{CODE}}
```

Guidance:
- Prefer specific roles over generic expert labels.
- Add domain context (industry, constraints) when available.

## Response Prefill

If the target platform supports assistant prefills, use them to constrain format.

Patterns:
- JSON output: prefill with `{`
- Character voice: prefill with role marker
- No preamble: prefill first content line

If prefills are not supported, enforce output shape with templates and constraints.

## Prompt Chaining

Decompose complex tasks into separate prompts.

Use chaining for:
- Multi-step analysis
- Research -> Outline -> Draft -> Review flows
- Verification-heavy tasks
- Complex transformations

Example chain:
1. Analyze source material.
2. Draft output using analysis.
3. Critique the draft.
4. Revise using critique.

Self-correction pattern:
1. Generate
2. Grade
3. Refine

## Grounding in Quotes

For long sources, require quote extraction before analysis.

Pattern:
1. Extract relevant quotes in `## Relevant Quotes`.
2. Produce interpretation in `## Analysis`.

This reduces hallucination and improves traceability.

## Extended Reasoning

For difficult STEM, optimization, or deep-analysis tasks:
- Start with broad guidance.
- Increase reasoning specificity only if needed.
- Require verification against explicit test cases.
- Do not feed hidden reasoning back into the model as source content.

Example:

```text
Think deeply about this problem. Consider multiple approaches.
If your first method fails, try alternatives.
Before finalizing, verify against:
- Edge case 1
- Edge case 2
```

## Writing Style

Prefer compact, direct phrasing.

Prefer:

```text
Commands handle I/O. Services contain logic. No circular dependencies.
```

Over long prose.

Hierarchy:
1. Code examples
2. Imperative bullets
3. Short declarative sentences
4. Tables (reference data only)
5. Paragraph prose

## Verification Reports

Use report templates in fenced blocks for explicit PASS/FAIL gating.

Template:

```
**[Constraint]:** PASS | FAIL (value)
**[Details]:** findings

**Proceeding with:** [next action] | **Blocked by:** [issue]
```

## Agent and Command Templates

Use this section when documenting agent or command specs.

Agent frontmatter:

```yaml
---
name: {agent-name}
description: "{purpose}\n\nExamples:\nuser: \"{example input}\"\nassistant: \"{expected response}\""
model: inherit
---
```

Command frontmatter:

```yaml
---
description: {one-line purpose and trigger}
allowed-tools: {tool list}
model: inherit
---
```

Body template:

````markdown
# {Role Title}

{One-line role description}

## Examples

### Example: {Use Case}

**user:** "{invocation}"
**assistant:** "{expected response}"

## Protocol

### Step 1: Gather
...

### Step 2: Analyze
...

### Step 3: Report
Output using report format below.

## {Report Name}

```text
{structured output template}
```

## Standards

- {quality requirement}

## Constraints

- {guardrail}
```
````

Callouts:

```markdown
> **IMPORTANT**
>
> - Critical rule 1
> - Critical rule 2
```

## Model Selection Guidance

Pick lower-cost/faster models for deterministic procedural tasks.
Pick stronger reasoning models for analysis, judgment, and synthesis.

Use orchestrator/worker patterns when splitting complex workflows:
- Orchestrator handles decomposition and integration.
- Worker agents handle bounded subtasks in parallel.

## Quality Gate

After drafting prompt docs, run:

```bash
wc -l <file>
wc -c <file> | awk '{print int($1/4)}'
```

General prompt gate:

```
**Document Structure:** PASS | FAIL
**Information Order:** PASS | FAIL
**Examples:** PASS | FAIL | N/A
**Clarity:** PASS | FAIL

**Proceeding with:** [action] | **Blocked by:** [issue]
```

Agent/command gate:

```
**Frontmatter Examples:** PASS | FAIL
**Protocol Structure:** PASS | FAIL
**Report Template:** PASS | FAIL
**Standards Section:** PASS | FAIL
**Constraints Section:** PASS | FAIL

**Proceeding with:** [action] | **Blocked by:** [issue]
```
