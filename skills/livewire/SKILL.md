---
name: livewire
description: "Apply Livewire project rules: class-based components only, no Volt, and strict separation between app/Livewire PHP classes and resources/views/livewire Blade templates. Use when creating or editing Livewire components."
---

# Livewire Rules

> **IMPORTANT**
>
> - **Class-based components only:** Always use separate PHP class + Blade view, never single-file components
> - **No Volt:** Do not use `livewire/volt` or single-file components with `⚡` prefix
> - **Separation of concerns:** Logic in `app/Livewire/`, templates in `resources/views/livewire/`

## Examples

### Example: Component Structure (Correct)

```
app/Livewire/DocsViewer.php                     ← PHP logic
resources/views/livewire/docs-viewer.blade.php  ← Blade template
```

### Example: Component Class (Correct)

```php
<?php

declare(strict_types=1);

namespace App\Livewire;

use Illuminate\Contracts\View\View;
use Livewire\Component;

final class CreatePost extends Component
{
    public string $title = '';

    public function save(): void
    {
        // Logic here
    }

    public function render(): View
    {
        return view('livewire.create-post');
    }
}
```

### Example: Single-File Component (Wrong)

```php
// DO NOT USE - resources/views/pages/⚡create-post.blade.php
<?php
use Livewire\Component;

new class extends Component {
    public string $title = '';
};
?>

<div>{{ $title }}</div>
```

## Context

### Why Class-Based Components

- **Testability:** PHP classes are easier to unit test in isolation
- **IDE Support:** Full autocomplete, refactoring, and static analysis
- **Separation:** Clear boundary between business logic and presentation
- **Maintainability:** Follows Laravel's conventional file organization
- **Debugging:** Stack traces point to specific class methods

### Creating Components

Use Artisan with the `--class` flag:

```bash
php artisan make:livewire CreatePost --class --no-interaction
```

This creates:
- `app/Livewire/CreatePost.php`
- `resources/views/livewire/create-post.blade.php`

## Standards

- All Livewire components must be class-based in `app/Livewire/`
- Views must be in `resources/views/livewire/`
- Use `#[Layout]` attribute for full-page components
- Add `wire:navigate` to links for SPA-like navigation

## Constraints

- Never install `livewire/volt`
- Never create single-file components with `⚡` prefix
- Never mix PHP logic into Blade templates beyond simple expressions
