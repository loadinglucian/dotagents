# PHP Standards

## Required

- `declare(strict_types=1);` in every PHP file.
- PSR-12 formatting.
- Braces for every control structure.
- Yoda comparisons for literals/constants.

## Yoda Examples

Correct:

```php
if (null === $value) {
    // ...
}

if ('' === trim($name)) {
    // ...
}
```

Incorrect:

```php
if ($value === null) {
    // ...
}
```

## Static Analysis Guidance

Use `@var` annotations for narrowing when static analysis needs hints.
Avoid production `assert()` as a type-enforcement mechanism.

## Comments and Imports

- Use `use` imports for vendor classes.
- Keep comments minimal and explanatory (why, not what).
- Remove stale comments when code changes.
