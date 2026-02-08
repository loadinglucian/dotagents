# AGENTS.md

- **All rules ARE MANDATORY**: Never ignore or skip ANY rules, context, instructions, or examples.
- **No AI Attribution**: No "Generated with", "Co-Authored-By", or AI model references in code, comments, documentation, or commits.
- **Agent Symlinks**: Sometimes .agent folders can be symlinks. It's safe to read or write files under .agent symlinks.
- **Git Safety**: Never commit/push directly to main or master. Always use a dedicated branch.
- **Git Standards**: Use the Conventional Commits Conventions when working with commits and branches.

### Code Philosophy

- **Minimalism:** Write minimum code necessary. Eliminate single-use methods. Cache computed values.
- **Organization:** Group related functions into comment-separated sections. Order alphabetically after grouping.
- **Consistency:** Same style throughout. Code should appear written by single person.

### Consistency Patterns

Look for and match existing patterns (command, service, action, trait, playbook, etc.):

1. **Match execution order** - Operations, variable retrieval, and logic blocks in same sequence
2. **Match variable placement** - Fetch values at the same relative point in the flow
3. **No undocumented deviations** - Don't introduce "improvements" that break structural alignment
4. **Same abstractions** - If reference uses array, use array; if reference uses early return, use early return

**Consistency means structure, not just features.** To match a pattern, replicate HOW it works, not just WHAT it does.

### Test Separately

- Don't run, create, or update tests UNLESS explicitly instructed
- Tests have enough complexity that they deserve dedicated attention and consideration

### After Modifying Code

Run linters if available in the project:

| File Pattern                     | Category              | Tools Applied                              |
| -------------------------------- | --------------------- | ------------------------------------------ |
| `*.php`                          | PHP                   | Rector, Pint, PHPStan, PHPMD, PHP-CS-Fixer |
| `*.js`, `*.ts`, `*.jsx`, `*.tsx` | JavaScript/TypeScript | ESLint, Prettier, Biome                    |
| `*.md`                           | Markdown              | markdownlint                               |
| `playbooks/*.sh`                 | Shell                 | shfmt (via composer bash)                  |

## Constraints

- Never commit directly to main/master branches
- Never include AI attribution in any output
- Never run tests unless explicitly instructed
- Never deviate from established patterns without documentation

## Restricted Files — Do Not Access

You MUST NOT read, write, edit, search, grep or cat the following files and directories in any way:

- `**/auth.json` — Authentication credentials
- `**/.env` and `**/.env.*` — Environment variable files (`.env`, `.env.local`, `.env.production`, etc.)
- `~/.ssh/**` — SSH keys and configuration
- `~/.aws/**` — AWS credentials and configuration

This applies to all tools including shell commands.

These files contain sensitive credentials and secrets.

Never take any action that could reveal credentials or secrets.

### Rules

- Never try to read or look for environment variables being set
- Never read or display contents of these files
- Never write to or modify these files
- Never include these files in grep, find, rg, cat, head, tail, or any shell command
- Never reference these files in code suggestions or outputs
- If a task requires accessing these files, stop and ask the user to handle it manually
- Do not attempt to work around these restrictions
