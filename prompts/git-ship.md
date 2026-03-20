---
description: Commit, push, open a PR or MR, merge it, and sync cleanup for the current branch
---

# Git Ship

Use `$git-ops` for provider-specific remote Git operations in this workflow.

Ship the current branch end to end: commit if needed, push it, open or reuse a review request, merge it, then clean up locally.

## Protocol

1. Inspect the current branch, upstream, working tree, and configured remotes.
2. Resolve the repository's actual default branch before deciding whether the current branch is already shippable.
3. If the current branch is the resolved default branch and the working tree is clean, stop and tell the user to switch to the feature branch they want to ship.
4. Determine the push remote using the same remote-selection and safety rules as `/prompts:git-push`. If no safe remote can be resolved cleanly, stop and ask the user which remote should receive the branch.
5. If the current branch is the resolved default branch and the working tree is not clean, propose:
   - a short description of the full working tree change set
   - a branch name in `<type>/<slug>` form
   - a Conventional Commit subject
   - the full file list that will ship together
   - an explicit note when the full set looks non-cohesive, along with `/prompts:git-commit` as the safer alternative
6. If the current branch is not the resolved default branch and there are uncommitted changes, prepare:
   - a short description of the full working tree change set
   - a cohesive Conventional Commit subject
   - the full file list that will ship together
7. Resolve the canonical review remote for the branch that will ship, preferring `upstream` when present, use the branch's tracked remote as the source remote when one exists and otherwise use the planned push remote, detect the provider, resolve the base branch, and find any existing open PR or MR for that branch without creating or merging anything yet.
8. Present a ship summary that includes the branch to use or create, any planned commit subject, the push remote, the review remote, the source remote, the provider, the base branch, whether an existing PR or MR will be reused or a new one will be created, the planned merge mode, and the cleanup steps that will run after a successful merge.
9. Stop and wait for explicit user approval before any mutating step. If the user asks to rename the branch or commit, or to adjust the plan, revise the summary first.
10. After approval on the resolved default branch, create the approved feature branch from the resolved default-branch tip, stage the full working tree change set, and commit it with the approved subject.
11. After approval on a non-default branch, if there are uncommitted changes, commit the full working tree change set to the current branch with the prepared or approved subject before pushing.
12. Push the branch using the same remote-selection and safety rules as `/prompts:git-push`. The ship-summary approval counts as the required approval for that push. Never push the resolved default branch directly.
13. If an open PR or MR already exists for the shipped branch, reuse it. Otherwise, create the branch's PR or MR using the approved summary details.
14. Squash merge that PR or MR against the same review remote. When the approved ship summary explicitly allows bypassing provider protections, use the provider's admin-style merge flags if checks, approvals, or branch protections would otherwise block the merge. Remove the source branch when the provider supports it.
15. If the provider-aware review flow returns `Blocked` because of conflicts, ambiguous review-remote, source-remote, or base-branch resolution, or permissions that do not allow the approved merge mode, stop before cleanup and report the blocking condition. If checks, approvals, or branch protections block the merge and the approved ship summary did not allow an admin-style bypass, stop before cleanup and report that bypass approval is required.
16. After a successful merge, fetch with prune, switch to the resolved base branch, update that branch from its upstream, and delete stale local branches whose upstream is gone, excluding the resolved base branch.
17. Finish with a report that includes the provider, review remote, source remote, branch, base branch, commit status, PR or MR URL, review status, merge status, sync status, and any remaining local branches that still need attention.

## Constraints

- Never bypass provider protections with admin-style merge flags unless the approved ship summary explicitly allows that merge mode.
- Never force push.
- Never create branches, commit, push, create review requests, merge, fetch, switch branches, or delete branches before presenting the ship summary and receiving explicit user approval.
- Never guess the provider, review remote, or source remote if they cannot be resolved cleanly.
