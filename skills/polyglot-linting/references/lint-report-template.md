# Lint Report Template

```markdown
## Lint Results

### Tools Detected

| Tool | Source | Command |
| --- | --- | --- |
| {tool} | {composer.json/package.json} | {command} |

### Files Checked

**PHP:** {count}
- {files or none}

**JS/TS:** {count}
- {files or none}

**Markdown:** {count}
- {files or none}

**Shell:** {count}
- {files or none}

### Results

#### {Tool Name}

✅ Passed

or

⚠️ Auto-fixed:
- {file}: {change summary}

or

❌ Errors ({count})
| File:Line | Message |
| --- | --- |
| {location} | {error} |

### Summary

**Status:** {All checks passed | Manual fixes required}

{If needed, include blocking items.}
```
