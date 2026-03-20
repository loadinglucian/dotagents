---
name: prompt-engineering
description: "Write prompts, reusable instruction files, and portable SKILL.md files with agent-agnostic guidance, clear trigger boundaries, semantic markdown structure, strong information ordering, concise examples, protocols, standards, constraints, progressive disclosure, and unwrapped Markdown prose. Use when creating or editing prompts, agent instructions, command docs, rules, skills, AGENTS.md, or similar instruction artifacts."
---

# Prompt Engineering Rules

Rules for writing prompts, instruction files, and portable skills that work well across agent products and model families.

This skill replaces the old `prompting` and `skill` skills. Use it as the single source of truth for prompt and instruction authoring.

> **IMPORTANT**
>
> - **Treat prompts and skills as the same discipline**: Both are instruction artifacts. Skills add trigger logic, reuse, and optional supporting files.
> - **Optimize for portability first**: Write the core guidance so it still makes sense without vendor-specific features.
> - **Keep the core AI agent agnostic**: Write prompts so they carry across Codex, Claude, Gemini, and future agents without rewriting the core instructions.
> - **Do not hard-wrap Markdown prompt prose**: Keep paragraphs and list items on single lines unless Markdown syntax requires a structural line break.
> - **The trigger contract matters**: Reusable instructions should make their scope obvious. Skills must state what they do, when they should trigger, and when they should not.
> - **Keep the core small and high-value**: Put only always-needed guidance in the main file. Move deep references, large examples, and deterministic helpers to supporting files.

## Context

### One Discipline, Multiple Artifact Types

Prompts, system instructions, agent files, command docs, rule files, and skills all solve the same problem: giving a model the right job, context, procedure, boundaries, and output shape.

Use the same fundamentals everywhere:

- Define the job clearly
- Give the right context before the task
- Show examples when pattern accuracy matters
- State the procedure when sequence matters
- Define output expectations
- Add constraints and verification

### Choose the Right Artifact

Use a one-off prompt when:

- The task is specific to one conversation
- Reuse is unlikely
- The context can live inline

Use a reusable instruction file when:

- The same structure or quality bar should apply repeatedly
- The artifact describes a stable workflow, style, or policy
- You want a shared format for agents, commands, rules, or playbooks

Use a skill when:

- The capability should activate based on user intent
- The workflow benefits from progressive disclosure
- The task may need bundled references, templates, or scripts
- You want a reusable capability rather than a one-off prompt

### What Makes a Skill Portable

Portable skills assume only the shared Agent Skills model:

- A skill is a directory with a required `SKILL.md`
- Agents load `name` and `description` first, then load the full file when the skill is chosen
- Supporting files are loaded only when needed

Write the main skill so it still makes sense if a client does not support slash commands, hooks, subagents, model selectors, or runtime variables.

Portable prompts and skills should also read naturally if they are moved between agent products. The core instructions should address the `agent`, `model`, or `client` generically rather than assuming a branded assistant identity.

### Portable Skill Structure

```text
skill-name/
├── SKILL.md
├── references/
│   └── REFERENCE.md
├── assets/
│   └── report-template.md
└── scripts/
    └── validate.sh
```

### Portable Skill Frontmatter

These fields come from the shared Agent Skills specification:

| Field           | Required | Notes                                                                                 |
| --------------- | -------- | ------------------------------------------------------------------------------------- |
| `name`          | Yes      | Lowercase letters, numbers, and hyphens only; max 64 chars; must match directory name |
| `description`   | Yes      | Max 1024 chars; explain what the skill does and when to use it                        |
| `license`       | No       | License name or bundled license file                                                  |
| `compatibility` | No       | Environment requirements such as tools, packages, or network access                   |
| `metadata`      | No       | Additional string key-value data                                                      |
| `allowed-tools` | No       | Experimental; support varies by client                                                |

## Instructions

### Structure the Document Semantically

Markdown structure should make the artifact easy for both humans and models to follow.

Prefer semantic headers:

- `## Context`
- `## Documents`
- `## Examples`
- `## Instructions`
- `## Protocol`
- `## Report`
- `## Standards`
- `## Constraints`

Use them consistently:

- Use the same section names across related files
- Reference sections explicitly when needed
- Use H2 for major sections and H3 for named subsections
- Keep critical callouts in a short `> **IMPORTANT**` block

### Order Information Deliberately

Position affects performance. Put supporting information before the task that depends on it.

Preferred order for prompt-like artifacts:

1. Role or job definition, if needed
2. Context and background
3. Long source material or documents
4. Examples
5. Instructions
6. Output format or report template
7. The live task or query last

For reusable instruction files and skills, the eventual user task may arrive later. In those files, keep the reusable guidance first and avoid burying the core procedure under examples or theory.

### Write Strong Trigger Boundaries

For skills, the description is the trigger contract.

A strong description should answer:

1. What does this skill do?
2. When should it trigger?
3. When should it stay out of the way?

Include:

- The main job the skill does
- The user intent that should trigger it
- Natural phrasing and likely synonyms
- Implicit trigger contexts where the user may not name the domain directly
- Nearby tasks the skill should not handle

Avoid:

- Vague phrases like "helps with"
- Internal implementation details instead of user intent
- Product names unless the skill genuinely depends on a specific product
- Keyword stuffing without clear boundaries

Good pattern:

```text
Use when the user wants to [goal], especially for [common cases]. Do not use for [nearby but different tasks].
```

### Write Agent-Agnostic Core Instructions

Portable prompts should describe capabilities and workflow in a way that survives copy-paste across agent products.

Prefer:

- `agent`, `model`, or `client` over branded assistant names
- Capability-based language like `tool access`, `terminal access`, `file editing`, or `web access`
- Optional compatibility notes only when a workflow truly depends on one client

Avoid:

- Writing the core prompt as if it belongs to Codex, Claude, Gemini, or any other named agent
- Depending on product-specific UI terms when a capability-based phrase would work
- Forcing users to rewrite the prompt just to move it into another agent product

Weak:

```text
Codex should inspect the repo and then Claude should rewrite the prompt for Gemini.
```

Better:

```text
Have the agent inspect the repository, then revise the prompt so it remains effective across agent products. If a client-specific adaptation is required, isolate it in a clearly labeled compatibility note.
```

Weak:

```text
Use Codex slash commands to gather context before you answer.
```

Better:

```text
Gather the required context using the client's available tools before answering. If the target client exposes special commands, document them in an optional client-specific addendum rather than the portable core.
```

### Do Not Hard-Wrap Markdown Prompt Files

Markdown prompt artifacts should keep prose unwrapped.

Prefer:

- One line per paragraph
- One line per list item unless the list item contains a nested Markdown structure
- Line breaks for structure, such as headings, lists, blockquotes, tables, and fenced code blocks

Avoid:

- Manually wrapping prose to satisfy line-length rules
- Reflowing prompt text into narrow columns
- Introducing noisy diffs by rewrapping unchanged Markdown prose

Weak:

```markdown
This prompt should be portable across agent products and should remain easy to
edit when a collaborator updates one sentence in the middle of the paragraph.
```

Better:

```markdown
This prompt should be portable across agent products and should remain easy to edit when a collaborator updates one sentence in the middle of the paragraph.
```

### Use Examples Only When They Add Real Value

Examples are one of the highest-leverage tools for prompt quality, but only when they teach a pattern the model might otherwise miss.

Use examples when:

- Format must be followed closely
- Edge cases matter
- Good and bad outputs are easy to confuse
- You need to anchor tone, structure, or decision criteria

Good example design:

- Relevant to the real task
- Diverse enough to avoid accidental overfitting
- Short enough to scan quickly
- Clearly labeled

Prefer 2 to 5 strong examples over a large pile of similar ones.

If you need concrete models for reusable instruction files, protocol structure, or portable skills, read [references/examples.md](references/examples.md).

### Prefer Procedures Over Declarations

Do not just state principles. Tell the model how to apply them.

Weak:

```text
Write clean code and follow best practices.
```

Better:

```text
1. Match the existing file structure and naming patterns.
2. Prefer the project's established abstractions over new ones.
3. Add comments only where the control flow would otherwise be hard to follow.
4. Verify the edited path still satisfies the documented constraints.
```

### Provide Defaults, Not Menus

When several tools or approaches could work, pick a default path and mention alternatives briefly.

Weak:

```text
You can use tool A, tool B, tool C, or tool D depending on what feels right.
```

Better:

```text
Use tool A by default. If the input is scanned or image-based, switch to tool B.
```

### Match Specificity to Fragility

Be highly prescriptive when:

- The workflow is fragile
- Order matters
- A command or format must be exact
- The task is safety-critical

Stay more flexible when:

- Several approaches are valid
- The model needs judgment
- The work is exploratory or analytical

Most good instruction artifacts mix rigid steps and flexible judgment. Calibrate each part independently.

### Use Reasoning Scaffolds Carefully

For simple tasks, direct instructions are enough.

For complex tasks, add light reasoning scaffolds such as:

- Compare two or three approaches before choosing one
- Check edge cases before finalizing
- Verify the answer against explicit criteria
- Produce a short rationale section when the user benefits from seeing the reasoning

Prefer guidance that improves accuracy without forcing verbose thought dumps. Use visible reasoning sections only when the deliverable genuinely benefits from them.

### Ground on Source Material

When accuracy depends on documents, code, or evidence:

- Put the source material before the task
- Quote or extract the relevant evidence first when helpful
- Base conclusions on the cited material rather than memory
- Separate "evidence" from "analysis" when hallucination risk is high

### Use Prompt Chaining for Complex Work

Break complicated workflows into stages when one large prompt would blur together incompatible jobs.

Common chains:

- Research -> Outline -> Draft -> Review -> Revise
- Analyze -> Decide -> Execute -> Verify
- Generate -> Grade -> Refine

Use chaining when you need:

- Better traceability
- Stronger verification
- Clearer intermediate artifacts
- Different quality bars at different stages

### Define Output Shape Explicitly

If format matters, give the model a template instead of vague prose.

Short templates can live inline. Longer templates belong in `assets/`.

Example:

````markdown
## Verification Report

```text
**Artifact Type:** [prompt | instruction file | skill]
**Trigger Boundary:** PASS | FAIL | N/A
**Structure:** PASS | FAIL
**Examples:** PASS | FAIL | N/A
**Progressive Disclosure:** PASS | FAIL | N/A
**Client Extensions Isolated:** PASS | FAIL | N/A

**Proceeding with:** [next action]
**Blocked by:** [issue or none]
```
````

### Write Reusable Skills with Progressive Disclosure

Keep the main skill focused on the instructions that should load every time.

- Keep `SKILL.md` under 500 lines
- Put detailed reference material in `references/`
- Put reusable output templates or static resources in `assets/`
- Put deterministic helpers and validators in `scripts/`
- Tell the agent exactly when to load or run each supporting file
- Use relative paths from the skill root
- Avoid deep reference chains

Weak reference:

```markdown
See `references/` for more details.
```

Better reference:

```markdown
If the API returns a non-200 status, read [references/api-errors.md](references/api-errors.md) before retrying.
```

### Use Scripts and Assets Deliberately

Add a script only when prose is not enough.

Good reasons to add a script:

- The task needs deterministic validation
- A helper encapsulates repeated mechanics
- The workflow depends on a reliable transformation or check

Scripts should:

- Be self-contained or clearly document dependencies
- Fail with useful error messages
- Handle expected edge cases
- Be referenced from `SKILL.md` with a clear trigger for when to run them

Assets should contain templates, schemas, examples, or static resources that improve consistency.

### Separate Portable Core from Client Extensions

Keep the main guidance portable across agent products.

- Put shared guidance in standard frontmatter and Markdown content
- Write the portable core so it still works across Codex, Claude, Gemini, and future agent products
- Describe capabilities generically before naming any client-specific feature
- Treat client-specific fields and behaviors as optional extensions
- Only use runtime variables documented by the target client
- Do not assume one client's slash commands, hooks, subagents, or model selectors exist everywhere

If you need client-specific behavior, keep it in a clearly labeled addendum, companion file, or client-owned metadata layer rather than making it the default shape of the skill.

### Write Tightly

Prefer:

- Code examples over long prose
- Imperative bullets over abstract essays
- Short declarative sentences over padded explanations
- Tables for compact reference data only

Cut anything the model is likely to know already unless the local convention overrides general knowledge.

## Protocol

### Step 1: Define the Artifact

Decide whether the work is:

- A one-off prompt
- A reusable instruction file
- A portable skill with supporting files

Choose the smallest artifact that still captures the workflow reliably.

### Step 2: Define the Job and Boundary

Write the job in one or two sentences.

If the artifact is a skill, write the trigger boundary:

1. What does it do?
2. When should it trigger?
3. When should it not trigger?

### Step 3: Draft the Smallest Useful Core

Write only the content the model should see on every run:

- Job definition
- Essential context
- Main procedure
- Critical gotchas
- Output shape, if needed
- Standards and constraints

### Step 4: Add High-Leverage Support

Only add:

- Examples that teach a pattern
- Templates that reduce formatting errors
- References that hold deeper material
- Scripts that add deterministic capability or validation

### Step 5: Evaluate Triggering

For skills, create realistic prompts for both sides of the boundary:

- `should-trigger`: 8 to 10 prompts the skill should activate for
- `should-not-trigger`: 8 to 10 near-miss prompts that mention related concepts but need a different skill

Vary phrasing, explicitness, complexity, and realism. Prefer real user language with concrete details.

### Step 6: Evaluate Output Quality

For representative tasks, define:

- The expected output or outcome
- A few objective assertions
- A simple grading method with concrete evidence

Useful assertions are observable and checkable, such as:

- "The output file is valid JSON"
- "The report includes at least 3 recommendations"
- "The chart has labeled axes"

### Step 7: Iterate Without Overfitting

If the artifact fails:

- Broaden wording when relevant prompts are missed
- Narrow wording when near-miss prompts trigger incorrectly
- Fix category boundaries, not single keywords
- Remove stale examples or instructions that no longer add value

## Standards

- Use semantic Markdown structure with consistent section names
- Put supporting context before the task that depends on it
- Keep reusable artifacts concise and procedural
- Write the portable core in AI agent agnostic language
- Keep Markdown prompt prose unwrapped unless syntax requires a structural line break
- Use examples only when they materially improve accuracy
- Define output shape explicitly when format matters
- Keep one skill focused on one coherent capability or workflow
- Follow the shared Agent Skills specification first
- Keep the portable core usable across different agent products
- Use progressive disclosure for larger skills
- Reference supporting files with explicit load conditions
- Add scripts only when prose is not enough
- Test both triggering quality and output quality

## Constraints

- Never write vague instructions like "follow best practices"
- Never write vague skill descriptions like "helps with tasks"
- Never bury core instructions under long theory sections
- Never add many equal options when one default path is better
- Never write the default prompt as though it belongs to Codex, Claude, Gemini, or any other branded agent
- Never assume one product's extensions are universal
- Never make client-specific features the default shape of a portable skill
- Never require product-specific wording in the portable core when capability-based language would work
- Never hard-wrap Markdown prompt prose to satisfy line-length rules
- Never rely on undocumented runtime substitutions
- Never bury trigger criteria only in the body; keep them in `description`
- Never add supporting files that are not referenced from the main artifact
- Never bloat `SKILL.md` with material that should live in `references/`, `assets/`, or `scripts/`
