# Prompt Engineering Examples

Concrete examples for reusable instruction files, protocol structure, and portable skills.

Load this file when you need a concrete pattern to imitate rather than abstract guidance.

## Example: Good Reusable Instruction File

```markdown
# Security Review

Review application changes for concrete security risks.

## Context

Focus on authentication, authorization, input validation, secrets handling, and unsafe defaults.

## Instructions

1. Review changed files first.
2. Quote or reference the risky code before giving a conclusion.
3. Report findings ordered by severity.
4. Include mitigations only after the findings.

## Standards

- Findings are concrete and evidence-based
- Severity ordering is defensible

## Constraints

- Do not report style-only issues as security findings
- Do not invent missing code or config
```

## Example: Good Portable Skill Description

```yaml
---
name: csv-analysis
description: Analyze CSV and tabular data files, compute summary statistics, add derived columns, clean messy rows, and generate charts. Use when the user wants to explore, transform, or visualize spreadsheet-like data, even if they do not explicitly say CSV. Do not use for database ETL, spreadsheet formula editing, or general scripting tasks.
---
```

Why it works: It defines the job, the intent, implied triggers, and the nearby tasks it excludes.

## Example: Good Protocol Pattern

```markdown
## Protocol

### Step 1: Gather

Collect the required inputs and constraints.

### Step 2: Analyze

Choose the best approach using the constraints above.

### Step 3: Execute

Produce the requested output.

### Step 4: Verify

Check the output against the report template and constraints below.
```

## Example: Good Skill Structure

```markdown
---
name: security-audit
description: Review application code and dependencies for security risks. Use when auditing code for vulnerabilities, auth mistakes, unsafe data handling, or missing validation. Do not use for style-only reviews.
---

# Security Audit

## Instructions

1. Review the changed files for auth, input handling, secrets exposure, and unsafe defaults.
2. Read [references/owasp-top-10.md](references/owasp-top-10.md) when the code touches common web attack surfaces.
3. Read [references/dependency-audit.md](references/dependency-audit.md) when dependencies, lockfiles, or package manifests changed.
4. Report concrete risks first, then mitigations.
```
