---
description: Git commit, push, sync, and ship workflows
argument-hint: <commit|push|sync|ship>
---

# Git Workflow Automation

## Context

Handles branch management, commits, PRs, and repository sync with
Conventional Commits format.

## Examples

### Example: Commit

**user:** `/git commit`

**assistant:**
> I'll analyze your changes and group them into independent branches that
> can be reviewed separately. Let me propose the groupings for your
> confirmation.

### Example: Push

**user:** `/git push`

**assistant:** `I'll push the branch and create a PR on GitHub.`

### Example: Sync

**user:** `/git sync`

**assistant:**
> I'll rebase on upstream, update tracked branches, and delete stale
> branches.

### Example: Ship

**user:** `/git ship`

**assistant:**
> I'll commit, push, create or merge the PR, remove the remote branch,
> and sync for the full
> end-to-end workflow.

## Instructions

Parse `$ARGUMENTS` to determine which subcommand to run.
Use the first positional argument in `$1` as the subcommand.
Treat empty or unknown subcommands as a help request and stop after
printing the usage block.

## Dispatch

| Argument | Action |
| --- | --- |
| `commit` | Jump to `Subcommand: commit` |
| `push` | Jump to `Subcommand: push` |
| `sync` | Jump to `Subcommand: sync` |
| `ship` | Jump to `Subcommand: ship` |
| empty or unknown | Output usage help below |

```text
Usage: /git <subcommand>

Subcommands:
  commit  Group changes into independent branches for isolated review
  push    Push branch and open PR on GitHub
  sync    Rebase sync with remote, update branches, delete gone branches
  ship    Full pipeline: commit -> push -> merge -> sync
```

## Subcommand: commit

Create branches and commits based on working tree changes, grouping
independent changes into separate branches for isolated review.

### Step 1: Analyze

Run `git status` to identify:

- Modified files, staged and unstaged
- Untracked files
- Deleted files

Read changed files to understand their purpose and relationships.

### Step 2: Group

Categorize changes into independent change sets that can be reviewed and
merged separately.

Grouping criteria, in order:

1. Semantic purpose
2. Conventional Commit type
3. Dependency coupling

Use this independence test: if the group could be merged without the
others, it is a valid group.
If all changes serve one purpose, create a single group.

### Step 3: Confirm

Do not create any branches until the user explicitly approves the grouping.

Present proposed groupings with branch names:

```text
## Proposed Change Sets

### Group 1: {description}
Branch: {proposed_branch_name}
Type: {conventional_type}
Files:
- {file_path}
- {file_path}

### Group 2: {description}
Branch: {proposed_branch_name}
Type: {conventional_type}
Files:
- {file_path}

---

Ready to create {N} branches. Proceed? (yes / adjust / combine all)
```

Wait for explicit user response:

- `yes`, `proceed`, or `looks good`: continue to Step 4
- Adjustment request: re-group and present the proposal again
- `combine all` or `single branch`: merge all groups into one branch
- No response or unclear response: ask for clarification and do not proceed

### Step 4: Execute

For each approved group, starting from `main`:

1. Create a branch using a Conventional Commit prefix
2. Stage only the files in that group
3. Commit with Conventional Commits format
4. Return to `main` before processing the next group

If `main` does not exist, use `master`.

Use these branch prefixes:

| Type | Use case |
| --- | --- |
| `feat/` | New feature |
| `fix/` | Bug fix |
| `docs/` | Documentation only |
| `style/` | Formatting with no logic change |
| `refactor/` | Code restructure with no behavior change |
| `perf/` | Performance improvement |
| `test/` | Adding or updating tests |
| `build/` | Build system or dependencies |
| `ci/` | CI/CD configuration |
| `chore/` | Maintenance tasks |
| `revert/` | Reverting previous commit |

Keep branch names short, informative, and at most 50 characters.

Commit messages must:

- Use Conventional Commits format
- Keep the title imperative, 72 characters or fewer, with no trailing period
- Add a body only when it improves context
- Use `BREAKING CHANGE:` when required

Do not push, pull, or rebase in this subcommand.

### Step 5: Verify

Run `git status` and confirm:

- The working tree is clean
- No untracked files remain
- No modified files remain

If files are left behind, determine the correct group and add additional
commits until the working tree is clean.

### Step 6: Report

Use this format:

```text
## Commit Complete

| Metric | Value |
| --- | --- |
| Branches | {count} |
| Total commits | {count} |
| Total files | {count} |
| Current branch | main |

### Branches Created

#### {branch_name}
- `{hash}`: {message}
- Files: {file_list}

#### {branch_name}
- `{hash}`: {message}
- Files: {file_list}

### Working Tree

{Clean | Issues remaining}

### Next Steps

Run `/git push` on each branch to create PRs:
- `git checkout {branch_1} && /git push`
- `git checkout {branch_2} && /git push`
```

## Subcommand: push

Push the current branch and open a PR on GitHub.

### Step 1: Push

Push the current branch to `origin` with upstream tracking via `-u`.
Do not force push.

### Step 2: Check PR

Check for an existing pull request:

```bash
gh pr list --head <current-branch> --json number,url
```

### Step 3: Create or Report

If a PR exists, report the existing URL and confirm the push updated it.

If no PR exists, create one:

```bash
gh pr create --title "<title>" --body "<body>"
```

Generate the PR title in Conventional Commit format that matches the
branch prefix. Keep the title imperative, concise, and no longer than 72
characters, with no trailing period.

Generate the PR body from the commits:

- Brief description of what changed
- Key implementation details when relevant

Target `main` unless the branch naming clearly indicates a different base branch.

### Step 4: Report

Use this format:

```text
## Push Complete

| Metric | Value |
| --- | --- |
| Branch | {branch_name} |
| Commits pushed | {count} |
| PR Status | {Created | Updated | Existing} |
| PR URL | {url} |
```

## Subcommand: sync

Sync with remote using rebase, update tracked branches, and delete
branches with deleted upstreams.

### Step 1: Status

Run:

```bash
git status
```

Determine:

- Current branch name
- Whether the current branch has upstream tracking
- Modified files, staged and unstaged
- Untracked files

If there is no upstream tracking branch, skip directly to Step 3.

### Step 2: Stash

If there are modified or untracked files, run:

```bash
git stash push --include-untracked -m "Auto-stash for sync"
```

Track whether a stash must be restored later.

### Step 3: Fetch

Run:

```bash
git fetch --prune
```

### Step 4: Update Branches

Find all local branches with upstream tracking:

```bash
git branch -vv
```

For each tracked branch other than the current branch, fast-forward it when possible:

```bash
git fetch origin remote_branch:local_branch
```

Skip any branch that would require a merge commit.

### Step 5: Rebase

Rebase the current branch on its upstream:

```bash
git rebase
```

If conflicts occur:

1. Abort with `git rebase --abort`
2. Restore the stash if one was created
3. Inform the user and exit with failure

### Step 6: Restore

If a stash was created, run:

```bash
git stash pop
```

If stash application conflicts, inform the user that manual resolution is
required and stop.

### Step 7: Switch Main

If not already on `main`, switch to `main`.
If `main` does not exist, switch to `master`.

### Step 8: Cleanup

Delete local branches whose upstream is `[gone]`:

```bash
git for-each-ref \
  --format '%(refname:short) %(upstream:track)' refs/heads |
while read branch track; do
  if [ "$track" = "[gone]" ]; then
    git branch -D "$branch"
  fi
done
```

Track deleted branches for the final report.

### Step 9: Report

Use this format:

```text
## Sync Complete

| Metric | Value |
| --- | --- |
| Current branch | {branch_name} |
| Branches updated | {count} |
| Branches deleted | {count} |
| Stash | {Applied | N/A} |

### Deleted Branches

- {branch_name} (upstream gone)

### Remaining Branches

{Output of `git branch -vv`}
```

## Subcommand: ship

Complete the current branch workflow end to end: commit -> push -> merge -> sync.

Ship only the current branch. For multi-branch workflows created by
`/git commit`, ship each branch separately or use `/git push` for manual
review.

### Step 1: Commit Phase

If on `main` with uncommitted changes:

1. Analyze the changes for grouping
2. If multiple independent groups exist, present them and wait for
   explicit user choice

Do not proceed until the user chooses one of these options:

```text
Multiple independent change sets detected:
- Group 1: {description} ({N} files)
- Group 2: {description} ({N} files)

How would you like to proceed?
1. Ship all as one branch (faster, less granular)
2. Ship first group only, leave others uncommitted
3. Abort - run `/git commit` first to review groupings
```

If already on a feature branch, commit any uncommitted changes to the current branch.

### Step 2: Push Phase

Run the `push` subcommand workflow:

1. Push the branch with upstream tracking
2. Check for an existing PR
3. Create the PR when needed

### Step 3: Merge Phase

Merge the PR and remove the remote branch:

```bash
gh pr merge <number> --squash --admin --delete-branch
```

After the merge completes, verify that `origin/<branch>` no longer
exists. If the remote branch still exists, delete it explicitly:

```bash
git push origin --delete <branch>
```

### Step 4: Sync Phase

Run the `sync` subcommand workflow with this outcome:

1. Fetch with prune
2. Switch to `main`
3. Update `main` with `git pull --rebase`
4. Delete merged local branches

### Step 5: Report

Use this format:

```text
## Ship Complete

| Phase | Status |
| --- | --- |
| Commit | {count} commits created |
| Push | Branch pushed |
| PR | {url} |
| Merge | Squashed to main |
| Remote Branch | Removed |
| Sync | Cleaned up |

### Summary

- **Branch:** {branch_name}
- **Commits:** {count}
- **PR:** {url}
- **Merged to:** main

### Remaining Branches

{List any unshipped branches from `/git commit` grouping, or "None"}
```

## Standards

- Use Conventional Commit semantics for branch names, commit messages, and PR titles
- Create cohesive, independently meaningful commits
- Never leave uncommitted changes after `/git commit`
- Generate concise PR descriptions from commit history
- Use squash merge for clean history
- Delete remote branches after merge so merged branches do not linger

## Constraints

- Never push directly to `main` or `master`
- Never force push
- Never use interactive flags such as `-i`
- Do not commit sensitive files such as `.env` or credentials files
- Do not modify git config
- Do not skip hooks such as `--no-verify` or `--no-gpg-sign`

## User Query

`$ARGUMENTS`
