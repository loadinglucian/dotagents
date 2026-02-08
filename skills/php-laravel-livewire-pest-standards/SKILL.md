---
name: php-laravel-livewire-pest-standards
description: Enforce PHP coding standards, class-based Livewire architecture, and Pest testing conventions for Laravel projects. Use when editing PHP source, Livewire components, or Pest test files and when verifying rule compliance before commit.
---

# PHP Laravel Livewire Pest Standards

## Overview

Apply one consistent standards profile across Laravel PHP code, Livewire components, and Pest tests.
Favor strictness, clarity, and behavior-focused testing.

## Workflow Decision

### Step 1: Classify edited files

- PHP source (`**/*.php` outside tests)
- Pest tests (`tests/**/*.php`)
- Livewire components (`app/Livewire/*`, `resources/views/livewire/*`)

### Step 2: Apply relevant standard set

- Source files: PHP coding rules
- Livewire files: class-based architecture rules
- Test files: Pest behavior-testing rules

### Step 3: Validate and report

Provide a compliance summary listing PASS/FAIL by standard category.

## Core Rules

### PHP source rules

- Declare `strict_types=1` in every PHP file.
- Use braces on all control structures.
- Use Yoda conditions for literal/constant comparisons.
- Prefer explicit types and clear signatures.
- Use `@var` annotations for static-analysis narrowing when needed.
- Avoid obvious comments; refactor instead.

### Livewire rules

- Use class-based components only.
- Keep logic in `app/Livewire/` and templates in `resources/views/livewire/`.
- Do not use Volt or single-file `⚡` components.
- Prefer clear separation between business logic and presentation.

### Pest rules

- Use descriptive `it()` names that state behavior.
- Follow Arrange-Act-Assert structure.
- Prefer value/behavior assertions over type-only assertions.
- Use datasets for scenario matrices.
- Verify mock interactions for integration boundaries.
- Keep tests focused and non-overlapping.

## References

- `references/php-standards.md`
- `references/livewire-standards.md`
- `references/pest-standards.md`
- `references/quality-gate.md`

## Output Requirements

Always provide:
1. category-level compliance status,
2. concrete violations with file references,
3. recommended fixes aligned to these standards.

## Constraints

- Never omit `strict_types=1`.
- Never use single-line control statements without braces.
- Never create Livewire Volt/single-file components.
- Never rely on meaningless test assertions (`expect(true)->toBeTrue()`, pure type checks without behavior).
