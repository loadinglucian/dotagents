---
description: Rebase sync with remote, restore local work, and clean up gone branches
---

# Git Sync

Use `$git-ops` for provider-specific remote Git operations in this workflow.

Sync the local repository with its remotes while preserving uncommitted work safely.

## Protocol

1. Inspect the current branch, upstream status, default-branch identity when available, working tree state, and local branches with upstream tracking using read-only Git commands.
2. Record the current branch as the starting branch.
3. Prepare a sync summary that includes the starting branch, whether a stash will be created, whether the starting branch will be rebased or skipped, the non-current tracked branches that will be checked for safe fast-forward updates, the gone-upstream branches that are currently candidates for deletion, and a note that fetch with prune may change the final cleanup set.
4. Stop and wait for explicit user approval before stashing, fetching, rebasing, switching branches, or deleting branches.
5. If there are modified or untracked files, stash them with untracked files included and remember that the stash must be restored on the starting branch later.
6. Fetch with prune so remote-tracking references are current.
7. Inspect local branches with upstream tracking and fast-forward only non-current branches that can be updated safely. Skip branches that would require a merge or manual conflict resolution.
8. If the starting branch has an upstream, rebase the starting branch on that upstream. If rebase conflicts, abort the rebase, return to the starting branch, restore any stash if possible, and report the failure.
9. While the working tree is clean, delete local branches whose upstream is marked as gone, excluding the starting branch and the repository's default branch when it can be resolved. If the default branch cannot be resolved cleanly, do not delete branches named `main` or `master`.
10. If a stash was created earlier, return to the starting branch and restore it there. If stash application conflicts, stop and report that manual resolution is required.
11. Finish on the starting branch and report the starting branch, ending branch, branches updated, branches deleted, stash status, and remaining tracked branches.

## Constraints

- Never force any update.
- Never discard local changes silently.
- Never stash, fetch, rebase, switch branches, or delete branches before presenting the summary and receiving explicit user approval.
- Never continue past rebase or stash conflicts without reporting them.
