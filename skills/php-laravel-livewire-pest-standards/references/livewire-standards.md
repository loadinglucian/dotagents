# Livewire Standards

## Architecture

Use class-based Livewire components only.

Correct file split:
- `app/Livewire/ComponentName.php`
- `resources/views/livewire/component-name.blade.php`

Avoid:
- Volt package usage
- single-file components
- inline PHP component logic in Blade files

## Creation Command

```bash
php artisan make:livewire CreatePost --class --no-interaction
```

## Class Guidance

- Keep component state/properties typed.
- Keep actions in class methods.
- Keep view rendering in `render()`.
- Use `#[Layout]` for full-page components where applicable.
- Use `wire:navigate` when SPA-like transitions are desired.
