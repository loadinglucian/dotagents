# Release Report Template

```markdown
## Release Complete

| Metric | Value |
| --- | --- |
| Version | {version} |
| Tag | v{version} |
| Commits | {count} |
| Release URL | {url} |

### Changelog Entry

{changelog_entry}
```

## Approval Prompt Template

```markdown
## Ready to Release

| Field | Value |
| --- | --- |
| Current | v{old_version or N/A} |
| New | v{version} |
| Commits | {count} |

### Changelog Preview

{entry}

Proceed with release? (yes / request changes / cancel)
```
