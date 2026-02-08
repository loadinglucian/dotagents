---
name: git-branch-pr-workflow
description: Execute structured git branch workflows for commit grouping, push/PR creation, repository sync, and full ship pipelines. Use when coordinating safe branch-based delivery with Conventional Commit naming and GitHub PR operations.
---

# Git Branch PR Workflow

## Overview

Run predictable git workflows for `commit`, `push`, `sync`, and `ship` style tasks.
Preserve branch safety, commit quality, and PR hygiene.

## Dispatch

Map user request to one workflow:
- `commit`: Group current changes into independent branchable sets and commit.
- `push`: Push current branch and create/update PR.
- `sync`: Rebase-sync local branches with remote and clean stale branches.
- `ship`: Run commit -> push -> merge -> sync end-to-end.

## Workflow: commit

### Step 1: Analyze working tree

Inspect staged, unstaged, untracked, and deleted files.
Read changed files to understand semantic relationships.

### Step 2: Propose groups

Create independent change sets by:
1. shared purpose
2. conventional commit type
3. dependency coupling

Test each group for merge independence.

### Step 3: Confirm grouping

Before creating branches, present proposed groups and wait for explicit user approval.
Allow user adjustments or combine-all behavior.

### Step 4: Execute commits

For each approved group:
1. create branch from base branch
2. stage only group files
3. commit with Conventional Commit formatting
4. return to base for next group

Do not push or rebase in this subworkflow.

### Step 5: Verify and report

Ensure clean working tree and report branches, commit hashes, and next actions.

## Workflow: push

### Step 1: Push safely

Push current branch with upstream tracking.
Never force push.

### Step 2: Resolve PR

Check for existing PR by head branch.
Create PR if missing, otherwise report existing/updated PR URL.

### Step 3: Report

Include branch, commit count, PR status, and PR URL.

## Workflow: sync

### Step 1: Capture status and stash if needed

If local changes exist, stash including untracked files.

### Step 2: Fetch and prune

Fetch remote refs with prune.

### Step 3: Update tracked branches

Fast-forward local tracking branches when possible.
Skip branches requiring merge.

### Step 4: Rebase current branch

Rebase on upstream.
If conflict occurs, abort rebase, restore stash if needed, and report failure.

### Step 5: Cleanup

Switch to `main` (or `master` fallback) and remove local branches whose upstream is gone.

### Step 6: Report

Provide sync summary including updated branches, deleted branches, and stash state.

## Workflow: ship

### Step 1: Commit phase

If uncommitted changes exist, run commit workflow or ask whether to combine groups.

### Step 2: Push phase

Run push workflow.

### Step 3: Merge phase

Merge PR with squash and delete remote branch unless user requests otherwise.

### Step 4: Sync phase

Run sync workflow to restore clean local state.

### Step 5: Report

Provide full phase-by-phase result.

## References

- `references/workflow-reference.md`
- `references/report-templates.md`

## Standards

- Use Conventional Commit semantics for branch and commit names.
- Keep commit titles imperative and concise.
- Keep each branch focused on one coherent change set.
- Prefer explicit user confirmation for destructive or high-impact actions.

## Constraints

- Never push directly to `main`/`master`.
- Never force push.
- Never use interactive git flags.
- Never skip hooks by default.
- Never commit sensitive files.
