---
description: Push the current branch to its remote without opening a PR or MR
---

# Git Push

Use `$git-ops` for provider-specific remote Git operations in this workflow.

Push the current branch and nothing else.

## Protocol

1. Inspect the current branch, upstream, configured remotes, and working tree state with `git status --short --branch` or equivalent read-only Git commands.
2. Determine the target remote from the branch's upstream. If no upstream exists yet, prefer `origin` when it exists. Otherwise, if exactly one remote exists, use that remote. If no upstream exists and more than one plausible remote exists, stop and ask the user which remote should receive the push.
3. Resolve the target remote's actual default branch.
4. Refuse to continue if the current branch matches the resolved default branch.
5. If there are staged, unstaged, or untracked changes, stop and tell the user to commit or stash them first.
6. Present a push summary that includes the branch, target remote, resolved default branch, whether this is a first push or an upstream push, and the exact push command that will run.
7. Stop and wait for explicit user approval before pushing.
8. If the branch already tracks an upstream, run a normal `git push`.
9. If the branch has no upstream yet, run `git push -u <remote> <branch>`.
10. Do not open, inspect, create, or merge any PR or MR in this prompt.
11. Finish with a report that includes the branch, remote, resolved default branch, upstream status, pushed result, and a next step pointing to `/prompts:git-open-pr`.

## Constraints

- Never push the resolved default branch.
- Never force push.
- Never push before presenting the summary and receiving explicit user approval.
- Never open a browser or invoke provider-specific review commands here.
- Never use interactive Git flags.
