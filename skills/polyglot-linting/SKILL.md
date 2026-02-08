---
name: polyglot-linting
description: Detect and run project-configured linting and formatting tools across PHP, JavaScript/TypeScript, Markdown, and shell files. Use when validating changed files with existing repo tooling and producing a structured lint report.
---

# Polyglot Linting

## Overview

Run repo-native lint and formatting workflows without hardcoding one stack.
Auto-detect available tools from project manifests and run them in safe order.

## Workflow

### Step 1: Detect configured tools

Read project config files and build a tool inventory:
- `composer.json` for PHP and shell workflows
- `package.json` for JS/TS and markdown tooling

Resolve package manager in this order:
1. `bun.lockb` or `bun.lock` -> `bun`
2. `pnpm-lock.yaml` -> `pnpm`
3. `yarn.lock` -> `yarn`
4. `package-lock.json` -> `npm`
5. fallback -> `bun`

If no tools are detected, report `No linting tools found` and stop cleanly.

### Step 2: Select target files

Choose files from user input if provided.
Otherwise derive from current git changes:
- `git status --porcelain`
- optional comparison range: `git diff --name-only main...HEAD`

Group files by category:
- PHP (`*.php`)
- JS/TS (`*.js`, `*.ts`, `*.jsx`, `*.tsx`)
- Markdown (`*.md`)
- Shell (`playbooks/*.sh`)

### Step 3: Execute tools by category

Run only tools that are both detected and relevant to changed files.

Execution order:
1. PHP: Rector -> Pint/PHP-CS-Fixer -> PHPMD -> PHPStan
2. JS/TS: ESLint -> Prettier -> Biome
3. Markdown: markdownlint
4. Shell: shfmt workflow

Use fix/write modes where supported.
Capture command output and exit codes for report quality.

### Step 4: Apply constraints

- Exclude tests from PHPStan (`tests/`, `*Test.php`).
- Do not install missing tools.
- Do not change lint configuration files.
- Do not manually patch code in this skill; only run tools and report.

### Step 5: Report

Output a unified lint report containing:
- detected tools
- files checked
- pass/fail/autofix status per tool
- blocking issues requiring manual intervention

## References

- `references/tool-detection.md`
- `references/lint-report-template.md`

## Output Requirements

Always include:
1. Exact commands run (or selected script command).
2. File counts by category.
3. Tool-by-tool outcome with actionable failures.
4. Clear final status (`all clear` or `manual fixes required`).
