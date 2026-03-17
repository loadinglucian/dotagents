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

- **Minimalism:** Write minimum code necessary. Eliminate single-use methods. Cache computed values.
- **Organization:** Group related functions into comment-separated sections. Order alphabetically after grouping.
- **Consistency:** Same style throughout. Code should appear written by a single person.
- **Docstrings:** Each function, class, or module must include a relevant docstring comment.

### Consistency Patterns

Look for and match existing patterns such as command, service, action, trait, and playbook flows:

1. **Match execution order** - Operations, variable retrieval, and logic blocks stay in the same sequence
2. **Match variable placement** - Fetch values at the same relative point in the flow
3. **No undocumented deviations** - Don't introduce "improvements" that break structural alignment
4. **Same abstractions** - If a reference uses an array, use an array; if it uses an early return, use an early return

**Consistency means structure, not just features.** To match a pattern, replicate HOW it works, not just WHAT it does.

### Test Separately

- Don't run, create, or update tests UNLESS explicitly instructed
- Tests have enough complexity that they deserve dedicated attention and consideration

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
