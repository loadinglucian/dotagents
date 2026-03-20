# AGENTS.md

- **All rules ARE MANDATORY**: Never ignore or skip any rules, context, instructions, or examples.
- **No AI Attribution**: No "Generated with", "Co-Authored-By", or any AI references in code, comments, documentation, or commits.
- **Git Safety**: Never commit or push directly to main or master. Always use a dedicated branch.
- **Git Standards**: Use the Conventional Commits Conventions when working with commits and branches.
- **Use Agent SKILLS**: The `$...` notation references AI agent SKILLS to use.

## Constraints

- Never commit directly to main/master branches
- Never include AI attribution in any output
- Never run tests unless explicitly instructed
- Never deviate from established patterns without documentation
- Never wrap lines in this file

## Restricted Files — DO NOT ACCESS

You MUST NOT read, write, edit, search, grep, or cat the following files and directories in any way:

- `**/auth.json` — Authentication credentials
- `**/.env` and `**/.env.*` — Environment variable files (`.env`, `.env.local`, `.env.production`, etc.)
- `~/.ssh/**` — SSH keys and configuration
- `~/.aws/**` — AWS credentials and configuration

This applies to all tools including shell commands.

These files contain sensitive credentials and secrets.

Never take ANY action that could reveal credentials or secrets.

### Rules

- Never try to read or look for environment variables being set
- Never read or display contents of these files
- Never write to or modify these files
- Never include these files in grep, find, rg, cat, head, tail, or any shell command
- Never reference these files in code suggestions or outputs
- If a task requires accessing these files, stop and ask the user to handle it manually
- Do not attempt to work around these restrictions

## Code Philosophy

- **Minimalism:** Write minimum code necessary. Eliminate single-use methods. Reuse already-computed values within a flow.
- **Organization:** Group related functions into comment-separated sections. Order alphabetically after grouping.
- **Consistency:** Same style throughout. Code should appear written by a single person.
- **Docstrings:** Each function, class, or module must include a brief relevant docstring comment.

### Consistency Patterns

Look for and match existing patterns such as command, service, action, trait, and playbook flows:

1. **Match execution order** - Operations, variable retrieval, and logic blocks stay in the same sequence
2. **Match variable placement** - Fetch values at the same relative point in the flow
3. **No undocumented deviations** - Don't introduce "improvements" that break structural alignment
4. **Same abstractions** - If a reference uses an array, use an array; if it uses an early return, use an early return

**Consistency means structure, not just features.** To match a pattern, replicate HOW it works, not just WHAT it does.

### Engineering Defaults

- Read at least 2 existing files of the same type before creating a new file, module, or structure
- Write the smallest correct change and avoid speculative refactors
- Never over-abstract: three similar lines are better than a premature abstraction
- Never create separate files for single-use types, helpers, or tiny wrappers
- Prefer established project abstractions or official integrations before inventing custom infrastructure
- Keep boundary layers thin: handlers, commands, adapters, hooks, and components validate inputs and delegate business logic
- Treat external input as untrusted and validate it at the boundary before it reaches shell commands, file paths, SQL, HTML, or persistence sinks
- Never swallow errors silently; every caught error must be re-thrown, returned, or logged with context
- Define shared contracts and types once in a canonical location to avoid near-duplicate definitions and drift
- Keep dependencies explicit and avoid hidden global state when a local dependency can be passed directly
- Measure before optimizing; do not add caching, batching, memoization, or concurrency complexity without evidence

### Comments and Documentation

- Docstrings should explain intent, constraints, invariants, or non-obvious tradeoffs
- Never write comments that only narrate obvious code; refactor unclear code instead
- Remove or update comments whenever the related code changes or is deleted

### Testing Principles

- Use risk-based testing: choose the smallest set of tests that covers the user-facing, data, security, or release risks introduced by the change
- Optimize for defect detection, not test count
- Tests should cover meaningful failure paths, not only happy paths
- Test observable behavior and named risks instead of implementation trivia or framework internals
- Keep tests deterministic: avoid sleeps, real-time waiting, flaky retries, and uncontrolled polling
- Prefer data-driven tests when one behavior has multiple meaningful input permutations
- Eliminate overlapping tests that cover the same behavior without adding signal
- Mock only external boundaries and verify only the interactions that matter

### Before Running JavaScript/TypeScript

Detect the package manager from lock files before running any JS/TS commands.

Detection order:

1. `bun.lockb` or `bun.lock` → `bun`
2. `pnpm-lock.yaml` → `pnpm`
3. `yarn.lock` → `yarn`
4. `package-lock.json` → `npm`
5. No lock file → `bun` (default)

Use detected package manager for:

- Installing packages
- Running scripts
- Adding/removing dependencies
- Building and dev servers
- Executing one-off packages (dlx/npx/bunx)

### After Modifying Code

Run linters if available in the project:

- `*.php`: PHP via Rector, Pint, PHPStan, PHPMD, and PHP-CS-Fixer
- `*.js`, `*.ts`, `*.jsx`, `*.tsx`: JavaScript or TypeScript via ESLint, Prettier, and Biome
- `*.md`: Markdown via `markdownlint`
- `playbooks/*.sh`: Shell via `shfmt` through Composer bash

Update and maintain an `Architecture` section in `AGENTS.md` with repository-specific architecture:

- Use `$ai-prompting` for AI prompting rules and structure when updating `AGENTS.md`
- Prefer concrete file ownership, scripts, and tests over abstract architecture language
- Keep examples tied to current code paths and current verification coverage
- Update or remove stale references when implementation moves
- Link to shared skills instead of restating generic stack doctrine
- The `dotagents/AGENTS.md` is exempts from the `Architecture` section rule
