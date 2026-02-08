# CI Analysis Reference

## Error Pattern Mapping

| Pattern | Category | Example Tool |
| --- | --- | --- |
| `FAILURES!` or `Tests:.*Failures:` | Test failure | PHPUnit |
| `Test Suites:.*failed` | Test failure | Jest |
| `Found [0-9]+ errors` | Static analysis | PHPStan |
| `[0-9]+ problems? \([0-9]+ errors?` | Lint failure | ESLint |
| `error:.*cannot find` | Build/type failure | TypeScript |
| `ENOENT|EACCES|EPERM` | File/permission issue | Node.js |
| `exit code [1-9]` | Generic command failure | Any |
| `timed out` | Timeout | Any |

## CI Status Report Template

```markdown
## CI Status Report

### Summary

| Metric | Value |
| --- | --- |
| Total checks | {count} |
| Passed | {count} |
| Failed | {count} |
| Pending | {count} |
| Overall status | {pass/fail/pending} |

### Checks

| Check | Status | Workflow | Details |
| --- | --- | --- | --- |
| {name} | {pass/fail/pending} | {workflow} | {short details} |

### Failed Checks

#### {Check Name}

**Workflow:** {workflow}
**Run URL:** {url}

**Error excerpt:**
```text
{relevant log lines}
```

**Likely cause:** {cause}

### Recommendations

- {next action}
```

## Edge Cases

- No checks configured: report this explicitly and stop.
- All checks pending: report pending state and no failure root cause.
- API/auth failure: report command error and suggest `gh auth status`.
