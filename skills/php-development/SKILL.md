---
name: php-development
description: "Author and review PHP production code and Pest tests with strict_types, PSR-12 structure, braces on all control structures, Yoda conditions for literal comparisons, PHPStan-friendly annotations, Symfony-first utilities, and behavior-first Pest conventions. Use when working in `**/*.php`, `tests/**/*.php`, or when implementation and Pest coverage move together. Do not use for Livewire-specific component structure, docs-only work, or non-PHP stacks."
---

# PHP Development Rules

Rules for writing PHP production code and Pest tests as one coherent workflow.

> **IMPORTANT**
>
> - **Single source of truth:** Apply this skill for PHP source files and Pest tests
> - **Supersedes legacy skills:** Do not recreate separate `php` or `pest` skills
> - **Strict types:** Declare `declare(strict_types=1);` in every PHP file, including tests
> - **Always use braces:** All control structures use `{ }`
> - **Yoda conditions:** Put constants and literals on the left side of comparisons
> - **PHPStan level 10:** Maintain maximum strictness for production code
> - **Behavior over ceremony:** Production code must stay strict, while tests must prove observable behavior

## Context

### Scope

Use this skill when the work involves:

- PHP source files such as services, commands, actions, traits, and helpers
- Pest tests in `tests/**/*.php`
- Refactors where implementation and Pest coverage should stay aligned

Do not use this skill when a more specific skill owns the structure:

- Livewire component authoring and file layout
- Documentation-only edits
- Frontend-only or non-PHP changes

### PHP Standards

- PSR-12 coding standard
- Strict types in every file
- PHP 8.x features: unions, match, attributes, readonly, constructor promotion
- Explicit return types with generics where applicable, such as `Collection<int, User>`
- Dependency injection via Symfony patterns
- Use Symfony classes over native PHP functions, such as `Filesystem` and `Process`, for testability

### Pest Testing Context

Tests verify behavior using Pest PHP `it()` syntax. AAA pattern. Business logic over type checking.

Philosophy: "A test that never fails is not a test, it's a lie."

This unified skill intentionally does not enforce a global minimum coverage percentage or a test-file-size ratio.

### Production vs Test Expectations

Production code and tests share the same language and formatting baseline, but they optimize for different outcomes:

- Production code favors explicit types, predictable control flow, and Symfony-first utilities
- Pest tests favor behavior assertions, concise setup, and minimal mocking of external boundaries
- PHPStan strictness applies to production code; tests should still be clean, but value assertions matter more than type-only compliance
- Coverage targets stay meaningful only when the tests prove behavior, not when they pad numbers with shallow assertions

### Verification Commands

Use these only when the task or local instructions explicitly allow test execution:

```bash
composer pest                 # Full suite with coverage (parallel)
vendor/bin/pest $TEST_FILE    # Specific file
```

## Documents

Read [references/rules-and-examples.md](references/rules-and-examples.md) before editing when the task needs any exact compatibility detail from the legacy `php` or `pest` skills, especially:

- Yoda condition examples, PHPStan annotation examples, or comment section formatting
- Pest file structure, naming examples, DI setup patterns, datasets, or assertion chaining examples
- Arch tests, exact mock API patterns, or the canonical forbidden and required test pattern snippets

## Examples

### Example: Production File Skeleton

```php
<?php

declare(strict_types=1);

namespace App\Service;

use Symfony\Component\Filesystem\Filesystem;

final class ConfigWriter
{
    /**
     * Persist configuration contents.
     */
    public function __construct(
        private readonly Filesystem $filesystem,
    ) {
    }

    /**
     * Write the config file when contents are present.
     */
    public function write(string $path, string $contents): void
    {
        if ('' === trim($contents)) {
            return;
        }

        $this->filesystem->dumpFile($path, $contents);
    }
}
```

### Example: Pest AAA Pattern

```php
it('calculates total with tax', function () {
    // ARRANGE
    $calculator = new PriceCalculator(taxRate: 0.1);
    $items = [['price' => 100], ['price' => 50]];

    // ACT
    $total = $calculator->calculateTotal($items);

    // ASSERT
    expect($total)->toBe(165.0);
});
```

### Example: Mock Interaction and Assertion Chaining

```php
it('hydrates the server from API data', function () {
    // ARRANGE
    $client = mock(ServerClient::class);
    $client->shouldReceive('fetch')
        ->once()
        ->with('web-1')
        ->andReturn([
            'name' => 'web-1',
            'host' => '192.168.1.10',
            'port' => 22,
        ]);

    $service = new ServerService($client);

    // ACT
    $server = $service->load('web-1');

    // ASSERT
    expect($server)
        ->name->toBe('web-1')
        ->and($server)
        ->host->toBe('192.168.1.10')
        ->and($server)
        ->port->toBe(22);
});
```

## Instructions

### Production Code

1. Match the existing file structure and execution order before introducing changes.
2. Add `declare(strict_types=1);` to every PHP file.
3. Use PSR-12 formatting, explicit return types, and `use` statements for vendor imports.
4. Use Yoda conditions for literal and constant comparisons, but keep normal comparisons for two variables.
5. Use PHP 8.x features when they match the surrounding codebase and improve clarity.
6. Add relevant docblocks for classes, functions, and modules; keep them brief and non-obvious.
7. Parameters and return tags belong in docblocks only when they are not inferable from the signature.
8. Use `@var` annotations for PHPStan when inference is insufficient. Do not use `assert()` in production code.
9. Prefer Symfony utilities such as `Filesystem` and `Process` when they replace native PHP with more testable abstractions.
10. Separate sections visually using the documented comment structure when the local pattern uses it.
11. Remove comments that only narrate obvious code.
12. Remove comments when removing code.

### Imports

- Always add `use` statements for vendor packages.
- Root namespace FQDNs are acceptable for exceptions, such as `\RuntimeException`.

### DocBlocks

- Keep descriptions minimalist.
- Add parameters and return types only when they are not inferable from the signature.
- Use `@var` annotations for PHPStan, not `assert()` in production code.

### Comments

- Separate sections visually using the structure shown in the examples.
- No obvious comments. If code needs explanation, refactor it.
- Remove comments when removing code.

### Pest Tests

1. Use Pest `it()` syntax with names that read like behavior.
2. Describe behavior, not implementation.
3. Follow Arrange-Act-Assert with `// ARRANGE`, `// ACT`, `// ASSERT`, or `// ACT & ASSERT` comments.
4. Dependency injection rules apply to production code, not tests.
5. Prefer manual instantiation with mocks for unit tests.
6. Use `mockCommandContainer()` for command tests that need container bindings.
7. Cover core business behavior, not implementation trivia or type-only facts.
8. Use datasets when one behavior has multiple input permutations.
9. Eliminate overlap so no two tests cover the same functionality.
10. Consolidate related assertions with chaining when they describe one outcome.
11. Do not consolidate distinct public methods, exception versus normal flow, or distinct business logic into one test.
12. Mock external dependencies only.
13. Verify the interaction that matters.
14. Use zero polling intervals in timeout-based tests to keep them deterministic.
15. Mirror the source structure where practical, using paths such as `tests/Unit/ServiceNameTest.php` and `tests/Integration/FeatureTest.php`.

### Forbidden Patterns

Do not write production code that:

- Omits braces from control structures
- Places variables on the left side of literal comparisons
- Uses `assert()` instead of PHPStan-friendly annotations
- Reaches for native PHP helpers when an established Symfony abstraction is the project default
- Writes obvious comments

Do not write Pest tests that:

- Assert only types or nullability without proving behavior
- Use meaningless assertions such as `expect(true)->toBeTrue()`
- Sleep or wait on real time progression
- Duplicate an existing test's scenario without adding new behavioral coverage

Load [references/rules-and-examples.md](references/rules-and-examples.md) for the exact forbidden-pattern, required-pattern, and mock-pattern snippets before writing or reviewing tests that depend on those details.

### Layer Defaults

| Layer | Default test type |
| ----- | ----------------- |
| CLI commands | Integration |
| Business services | Unit |
| Utilities and helpers | Unit |

### Organization

- Use section comments: `// {Section} tests // ----`
- File naming: `tests/Unit/ServiceNameTest.php` or `tests/Integration/FeatureTest.php`
- Mirror source structure where practical

### Static Analysis

PHPStan applies to production code, not tests. Focus on functionality over type compliance.

## Protocol

### Step 1: Classify the Change

Decide whether the work touches:

- Production PHP only
- Pest tests only
- Both implementation and Pest coverage

Apply only the sections that fit, but keep this file as the governing skill.

### Step 2: Match Local Structure

Before editing, mirror the surrounding file's execution order, naming, and abstractions.

### Step 3: Write the Smallest Correct Change

- Keep production code minimal and explicit
- Cache computed values instead of recomputing them
- Avoid single-use helper methods unless they remove real complexity
- Keep tests focused on the public behavior that changed

### Step 4: Verify the Result

Use the report below to confirm the edited paths still satisfy the skill.

Run Pest only when the task or repo instructions explicitly allow it.

## Report

After writing or editing PHP or Pest files, evaluate the relevant rows:

```text
**Production Standards:** PASS | FAIL | N/A
**Strict Types:** PASS | FAIL | N/A
**Braces and Yoda Conditions:** PASS | FAIL | N/A
**AAA Pattern:** PASS | FAIL | N/A
**Descriptive Names:** PASS | FAIL | N/A
**No Forbidden Assertions:** PASS | FAIL | N/A
**Datasets for Scenarios:** PASS | FAIL | N/A
**Value Assertions (not type-only):** PASS | FAIL | N/A
**Behavior Assertions:** PASS | FAIL | N/A
**Mock Interactions Verified:** PASS | FAIL | N/A
**No Test Overlap:** PASS | FAIL | N/A

**Proceeding with:** [next action]
**Blocked by:** [issue or none]
```

## Standards

- PSR-12 compliance on all PHP files
- Strict types declared in every file
- Explicit return types with generics where applicable
- Yoda conditions for all literal and constant comparisons
- Braces on all control structures
- Keep one coherent standard for PHP source and Pest tests
- Prefer procedures and concrete rules over abstract advice
- Use examples only when they teach a reusable pattern
- Keep the skill portable and readable without client-specific extensions
- Default to concise edits that match the repository's existing abstractions

## Constraints

- Never split PHP and Pest rules across multiple source-of-truth skills
- Never recreate legacy `php` or `pest` skills beside this one
- Never run tests by default when local instructions forbid it
- Never use `assert()` in production code
- Never omit braces from control structures
- Never place variables on the left side of literal comparisons
- Never use native PHP functions when Symfony equivalents exist
- Never write obvious comments
- Never use vague guidance such as "follow best practices"
- Never bury the trigger boundary only in the body; keep it clear in `description`
