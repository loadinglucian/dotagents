# Quality Gate

## Docs Report Template

```text
**Reader Directness:** PASS | FAIL
**Active Voice:** PASS | FAIL
**Why Before How:** PASS | FAIL
**Simple to Complex:** PASS | FAIL
**Examples Realistic:** PASS | FAIL
**Structure Complete:** PASS | FAIL

**Proceeding with:** [next action] | **Blocked by:** [issue]
```

## Formatting Workflow

Use detected package manager to run Prettier on docs:

- `bunx prettier --write "docs/**/*.md"`
- `pnpm dlx prettier --write "docs/**/*.md"`
- `yarn dlx prettier --write "docs/**/*.md"`
- `npx prettier --write "docs/**/*.md"`
