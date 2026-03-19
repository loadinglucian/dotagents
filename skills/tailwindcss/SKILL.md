---
name: tailwindcss
description: "Apply Tailwind CSS class organization and component extraction rules, including logical class ordering and extracting repeated patterns into components. Use when editing `resources/views/**/*.blade.php`, `resources/**/*.css`, `**/*.vue`, `**/*.jsx`, or `**/*.tsx`."
---

# Tailwind CSS Rules

Supplementary rules for class organization and component extraction (core Tailwind v4 syntax covered by Laravel Boost).

## Examples

### Example: Class Organization

```html
<!-- Organized class order: layout → spacing → sizing → colors → effects -->
<button class="flex items-center gap-2 px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700 transition-colors">
  Submit
</button>
```

### Example: Component Extraction

```blade
{{-- resources/views/components/button.blade.php --}}
<button {{ $attributes->merge(['class' => 'px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700']) }}>
    {{ $slot }}
</button>
```

## Instructions

### Class Organization

Think through class placement carefully:

1. **Remove redundant classes** - Don't repeat what's inherited
2. **Parent vs child** - Add shared styles to parent, unique to children
3. **Logical grouping** - Layout → spacing → sizing → colors → effects

### Component Extraction

When patterns repeat, extract into Blade/Vue/React components matching project conventions.

## Standards

- Organize classes logically (layout → spacing → sizing → colors → effects)
- Extract repeated patterns into components

## Constraints

- Never repeat classes that can be inherited from parent
- Never leave duplicated class patterns unextracted
