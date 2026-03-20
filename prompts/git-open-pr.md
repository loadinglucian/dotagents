---
description: Open or reuse a GitHub PR or GitLab MR for the current branch
---

# Git Open PR

Use `$git-ops` for provider-specific remote Git operations in this workflow.

## Protocol

1. Inspect the current branch, upstream, recent commits, and configured remotes.
2. Resolve the canonical review remote and source remote for the current branch, preferring `upstream` when present, detect whether the repository is hosted on GitHub or GitLab, resolve that review remote's default branch and base branch, and find any existing open PR or MR for the current branch without creating anything yet.
3. Refuse to continue if the current branch matches the resolved default branch.
4. Require the branch to already be pushed to a source remote. If no upstream exists, stop and tell the user to run `/prompts:git-push` first.
5. Generate a concise title and body from the branch's commits. Prefer Conventional Commit-aware wording and summarize the user-visible intent rather than listing raw hashes.
6. If the provider-aware review flow returns `Blocked` because the review remote, source remote, or base branch cannot be resolved cleanly, stop and report the blocking condition.
7. If an open PR or MR already exists for the current branch, finish with a provider-aware report that includes the provider, review remote, source remote, artifact type, branch, base branch, status, and URL. Use `PR` for GitHub and `MR` for GitLab.
8. Otherwise, present a creation summary that includes the provider, review remote, source remote, branch, base branch, proposed title, and body preview.
9. Stop and wait for explicit user approval before creating a new PR or MR.
10. After approval, create the new PR or MR using the resolved review remote as the target remote, the branch's tracked remote as the source remote, the resolved base branch, and the approved title and body.
11. Do not push the branch in this prompt.
12. Finish with a provider-aware report that includes the provider, review remote, source remote, artifact type, branch, base branch, status, and URL. Use `PR` for GitHub and `MR` for GitLab.

## Constraints

- Never guess the provider, review remote, or source remote if they cannot be resolved cleanly.
- Never create both a PR and an MR.
- Never create a new PR or MR before presenting the summary and receiving explicit user approval.
- Never use provider autofill or browser flows that may push or prompt interactively.
