# Tailwind Organization

## Class Ordering

Use consistent class order:
1. layout (`flex`, `grid`, `items-*`, `justify-*`)
2. spacing (`m*`, `p*`, `gap-*`)
3. sizing (`w-*`, `h-*`, `min-*`, `max-*`)
4. color/typography (`bg-*`, `text-*`, `border-*`)
5. effects/interactions (`shadow-*`, `hover:*`, `focus:*`, `transition-*`)

## Cleanup Rules

- Remove duplicate classes.
- Remove child classes that are already inherited from parent where possible.
- Keep variant classes (`hover:`, `focus:`) close to their base utility.

## Extraction Rule

When the same utility bundle appears repeatedly, extract a reusable component.

### Blade example

```blade
<button {{ $attributes->merge(['class' => 'px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700']) }}>
    {{ $slot }}
</button>
```

## Reporting Prompt

Document extraction decisions as:
- repeated pattern detected,
- target component created/updated,
- classes retained vs removed.
