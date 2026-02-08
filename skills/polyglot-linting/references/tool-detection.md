# Tool Detection Reference

## PHP Tools (`composer.json`)

| Tool | Detect | Preferred Command | Fallback |
| --- | --- | --- | --- |
| Rector | `require-dev.rector/rector` or `scripts.rector` | `composer rector` | `vendor/bin/rector process` |
| Pint | `require-dev.laravel/pint` or `scripts.pint` | `composer pint` | `vendor/bin/pint` |
| PHPStan | `require-dev.phpstan/phpstan` or `scripts.phpstan` | `composer phpstan` | `vendor/bin/phpstan analyse --memory-limit=2G` |
| PHPMD | `require-dev.phpmd/phpmd` or `scripts.phpmd` | `composer phpmd` | `vendor/bin/phpmd` |
| PHP-CS-Fixer | `require-dev.friendsofphp/php-cs-fixer` | `vendor/bin/php-cs-fixer fix` | `php-cs-fixer fix` |

## JS/TS Tools (`package.json`)

| Tool | Detect | Preferred Command | Fallback |
| --- | --- | --- | --- |
| ESLint | `devDependencies.eslint` or `scripts.lint` | `{pm} run lint` | `{pm} eslint` |
| Prettier | `devDependencies.prettier` or `scripts.format` | `{pm} run format` | `{pm} prettier --write` |
| Biome | `devDependencies.@biomejs/biome` | `{pm} biome check --write` | `{pm} exec biome check --write` |

## Markdown Tools (`package.json`)

| Tool | Detect | Preferred Command |
| --- | --- | --- |
| markdownlint | `devDependencies.markdownlint-cli` or `scripts.lint:md` | `{pm} run lint:md:fix` |

## Shell Tools

| Tool | Detect | Command |
| --- | --- | --- |
| shfmt | `composer.json` `scripts.bash` | `composer bash` |

## Package Manager Selection

1. `bun.lockb`/`bun.lock`
2. `pnpm-lock.yaml`
3. `yarn.lock`
4. `package-lock.json`
5. fallback to `bun`
