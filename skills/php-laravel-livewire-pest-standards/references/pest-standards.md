# Pest Standards

## Test Philosophy

Test behavior, not superficial types.
Use concise, expressive tests that fail for meaningful regressions.

## Required Patterns

### AAA structure

- `// ARRANGE`
- `// ACT`
- `// ASSERT`

For exception flows, use `// ACT & ASSERT`.

### Naming

Use descriptive behavior names:
- `it('returns empty array when no servers configured')`
- `it('throws when SSH connection fails')`

Avoid vague names:
- `it('test1')`
- `it('works')`

### Datasets

Use `->with([...])` for similar validation scenarios.

### Mocking

Verify interactions where integration boundaries matter:

```php
$mock->shouldReceive('method')->once()->with('param')->andReturn('result');
```

## Forbidden Patterns

- `expect(true)->toBeTrue()`
- type-only assertions that do not verify behavior
- sleep-based time assertions for deterministic logic

## Test Layering

- CLI commands: integration tests
- business services: unit tests
- helpers/utilities: unit tests
