---
description: Create cohesive branches and commits from current working tree changes
---

# Git Commit

Use `$git-ops` for provider-specific remote Git operations in this workflow.

Create branches and commits from the current working tree, grouping independent changes so they can be reviewed separately.

## Protocol

1. Run `git status --short --branch` and inspect the configured remotes. Resolve the repository's actual default branch.
2. Read the changed files and group them into independent change sets. Group by semantic purpose first, conventional commit type second, and dependency third.
3. Before creating any branch or commit, present the resolved default branch, the proposed groups with a short description, a proposed branch name, a conventional commit type, the file list for each group, and an explicit note when any group will require hunk-level splitting inside a shared file.
4. Stop and wait for explicit user approval before you create branches or commits. If the user asks to regroup or combine changes, update the proposal first.
5. After approval, create one branch per group from the resolved default branch.
6. When groups share a file, split the approved changes by patch so each branch receives only its own hunks.
7. Stage only the approved files and hunks that belong to each group and create concise Conventional Commit messages.
8. Prefer non-interactive Git commands, but allow patch-mode staging when hunk-level separation is required to ship approved mixed-file changes safely.
9. Do not push branches, open review requests, or merge anything in this workflow.
10. Finish with a report that includes the resolved default branch, created branches, commit hashes and subjects, files per branch, whether the working tree is clean, and next steps that point to `/prompts:git-push` and `/prompts:git-open-pr`.

## Standards

- Keep branch names short, specific, and aligned with Conventional Commit prefixes.
- Keep commit subjects imperative, under 72 characters, and without trailing periods.
- If all changes belong together, propose a single group instead of forcing extra branches.

## Constraints

- Never create branches or commits before presenting the summary and receiving explicit user approval.
- Never push, pull, or rebase in this prompt.
- Prefer non-interactive Git commands. Interactive patch staging is allowed only when it is required to separate approved mixed hunks in the same file.
- Never skip hooks.
- Never commit sensitive files.
