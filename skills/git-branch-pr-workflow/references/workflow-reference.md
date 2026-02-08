# Workflow Reference

## Branch Type Prefixes

| Prefix | Use Case |
| --- | --- |
| `feat/` | New feature |
| `fix/` | Bug fix |
| `docs/` | Documentation |
| `style/` | Formatting only |
| `refactor/` | Internal restructuring |
| `perf/` | Performance work |
| `test/` | Test updates |
| `build/` | Build/dependency updates |
| `ci/` | CI/CD updates |
| `chore/` | Maintenance |
| `revert/` | Revert work |

## Key Commands

### Push workflow

```bash
git push -u origin <branch>
gh pr list --head <branch> --json number,url
gh pr create --title "<title>" --body "<body>"
```

### Sync workflow

```bash
git stash push --include-untracked -m "Auto-stash for sync"
git fetch --prune
git branch -vv
git rebase
git checkout main || git checkout master
git for-each-ref --format '%(refname:short) %(upstream:track)' refs/heads
```

### Ship merge

```bash
gh pr merge <number> --squash --admin --delete-branch
```

## Safety Gates

- Require explicit user confirmation before creating multiple branches.
- Abort and report on rebase conflicts.
- Restore stashed changes when safe to do so.
