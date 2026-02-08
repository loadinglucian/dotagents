# Report Templates

## Commit Report

```markdown
## Commit Complete

| Metric | Value |
| --- | --- |
| Branches | {count} |
| Total commits | {count} |
| Total files | {count} |
| Current branch | {branch} |

### Branches Created

#### {branch_name}
- `{hash}`: {message}
- Files: {files}

### Working Tree

{Clean | Issues remaining}
```

## Push Report

```markdown
## Push Complete

| Metric | Value |
| --- | --- |
| Branch | {branch} |
| Commits pushed | {count} |
| PR Status | {Created/Updated/Existing} |
| PR URL | {url} |
```

## Sync Report

```markdown
## Sync Complete

| Metric | Value |
| --- | --- |
| Current branch | {branch} |
| Branches updated | {count} |
| Branches deleted | {count} |
| Stash | {Applied/N/A} |

### Deleted Branches

- {branch}
```

## Ship Report

```markdown
## Ship Complete

| Phase | Status |
| --- | --- |
| Commit | {status} |
| Push | {status} |
| PR | {url} |
| Merge | {status} |
| Sync | {status} |
```
