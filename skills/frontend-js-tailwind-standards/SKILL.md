---
name: frontend-js-tailwind-standards
description: Apply JavaScript/TypeScript package-manager-aware workflows and Tailwind CSS class organization standards. Use when running JS tooling commands, editing package-managed frontend projects, or cleaning/restructuring Tailwind class usage in Blade, Vue, JSX, or TSX files.
---

# Frontend JS Tailwind Standards

## Overview

Standardize JS/TS command execution and Tailwind class composition.
Detect package manager from lock files before running any package command.

## Workflow

### Step 1: Detect package manager

Use lock-file precedence:
1. `bun.lockb` or `bun.lock` -> `bun`
2. `pnpm-lock.yaml` -> `pnpm`
3. `yarn.lock` -> `yarn`
4. `package-lock.json` -> `npm`
5. none -> `bun`

### Step 2: Map command semantics

Apply the detected manager for:
- dependency install/add/remove
- script execution
- one-off package execution (`dlx`/`npx`/`bunx`)

Never mix package-manager commands within one task unless user requests it.

### Step 3: Apply Tailwind class standards

When editing component/view classes:
- Remove redundant classes that can be inherited.
- Prefer parent-level shared styling and child-level overrides only.
- Order class groups consistently:
  1. layout
  2. spacing
  3. sizing
  4. colors
  5. effects/interaction

### Step 4: Extract repeated patterns

If class bundles repeat, extract into framework-native component patterns:
- Blade components
- Vue components
- React components

### Step 5: Report

Summarize:
- detected package manager
- commands executed
- class cleanup/extraction performed

## References

- `references/package-manager-detection.md`
- `references/tailwind-organization.md`

## Output Requirements

Always provide:
1. lock-file evidence for package manager selection,
2. command(s) used with selected manager,
3. class-ordering and extraction decisions.

## Constraints

- Never assume npm/yarn/pnpm/bun without lock-file check.
- Never leave duplicated class bundles when component extraction is appropriate.
- Never keep redundant inherited classes in child elements.
