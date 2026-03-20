---
name: git-ops
description: Handle provider-aware Git review and release operations for repositories hosted on GitHub or GitLab, including canonical review remote selection, default branch identification, provider detection, base branch resolution, existing PR or MR lookup, PR or MR creation, squash merge flows, existing release lookup, and release creation from a pushed tag. Use when a workflow needs to resolve the default branch or canonical review remote, open or merge a review request, or publish a provider release for the current repository. Do not use for generic commits, branch grouping, plain pushes, or sync-only tasks beyond the remote and default-branch guidance they reuse.
---

# Git Provider Operations

Use this skill when a prompt needs canonical review remote selection, default-branch guidance, or provider-aware review or release operations for the current repository.

> **IMPORTANT**
>
> - Fail closed when the hosting provider cannot be identified cleanly.
> - Prefer non-interactive CLI commands and explicit arguments.
> - Support preview-first workflows by resolving remotes, base branches, and existing review or release artifacts before approval, and only performing create or merge actions after the caller confirms.
> - Never push the resolved default branch directly.
> - Never force push.
> - Never use `--web` or provider-specific autofill flows that change behavior unexpectedly.
> - Never use admin-style merge bypass flags unless the caller explicitly approved that merge mode.

## Context

This skill owns GitHub and GitLab differences so calling prompts can stay focused on their workflow.

The calling prompt should provide:

- The current branch
- The desired action: resolve default branch, open review, merge review, or publish release
- The target remote when the caller has already resolved the canonical review or release remote
- The source remote when the review branch lives on a different remote than the review target
- The merge mode for review merges: `standard` or `admin-bypass`
- A title and body when creating a new review request
- A tag, release title, and notes file when creating a release

This skill should return a normalized result containing:

- Provider: `github` or `gitlab`
- Remote: the selected target Git remote name
- Source Remote when the review branch comes from a different remote
- Artifact type: `PR`, `MR`, or `Release`
- Branch when the action is review-related
- Base branch when the action is review-related
- Tag when the action is release-related
- Merge mode for review merges: `standard` or `admin-bypass`
- Status: `Existing`, `Created`, `Merged`, or `Blocked`
- Blocker when `Status` is `Blocked`
- URL
- Identifier when available

## Protocol

### Step 1: Gather Repository Context

Collect the current branch, upstream branch, tracked remote, selected target remote, selected source remote, remote URLs, and repository slugs.

- For review actions, treat the selected target remote as the canonical review repository.
- If the caller provides a target remote, use it as the canonical remote for every later provider operation and report.
- Otherwise, for review actions, prefer a remote literally named `upstream`. If none exists, prefer the current branch's tracked remote.
- Otherwise, for release actions, prefer a remote literally named `upstream`, because releases must be published from the canonical repository when that remote is configured.
- Otherwise, for release actions, prefer the current branch's tracked remote.
- If no tracked remote exists and exactly one Git remote is configured, use that remote.
- If no tracked remote exists and more than one plausible remote is configured, stop and report that remote resolution is ambiguous.
- If the caller provides a source remote for a review action, use it as the branch source remote.
- Otherwise, for review actions, prefer the current branch's tracked remote as the source remote. If the current branch does not track a remote and the selected target remote is also the only plausible source remote, reuse it as the source remote.
- If a review action has more than one plausible source remote and none can be identified cleanly, stop and report that source-remote resolution is ambiguous.
- Derive the repository slug from the selected target remote URL when the provider CLI needs it.
- Derive the source repository slug from the selected source remote URL when a review action comes from a different remote than the target review repository.
- Use read-only Git commands to inspect context before deciding on provider operations.

### Step 2: Detect the Provider

Normalize the selected remote URL so SSH and HTTPS remotes can be compared consistently.

Classify by remote host first:

- GitHub when the host is `github.com` or clearly GitHub Enterprise
- GitLab when the host is `gitlab.com` or clearly GitLab-hosted

If the remote host is ambiguous, run non-mutating provider probes for the selected repository:

- GitHub probe: `gh repo view "<repo>"`
- GitLab probe: `glab repo view -R "<repo>"`

If exactly one probe clearly resolves the selected repository, use that provider. If both fail or both appear plausible, stop and report that provider detection is ambiguous.

### Step 3: Resolve the Default Branch

Determine the selected target remote's default branch before creating or merging a review request, publishing a release, or deciding whether a calling prompt is already on the protected default branch.

- Prefer the remote HEAD branch from Git for the selected remote.
- If that is unavailable, inspect the selected remote's provider metadata.
- If neither path resolves a single default branch, stop and report `Status: Blocked`.
- Calling prompts that only need default-branch identity may reuse this step without invoking the later provider-specific review or release steps.

### Step 4: Open Review Requests

Use this flow when the caller wants to open a review request for the current branch.

- Require a non-default branch.
- Require the branch to already exist on the selected source remote. If no source remote exists, stop and tell the caller to push first.
- Reuse an existing open review request for the current branch when one already exists, and return the actual base or target branch from provider metadata.
- When the selected source remote and selected target remote differ, treat the workflow as a fork review flow and match existing review requests by both source branch and source repository identity.
- Callers that need an approval gate may stop after remote, base-branch, and existing-review lookup, present a preview, and invoke creation only after approval.

#### GitHub Review Requests

- When the source and target repositories are the same, check for an existing open pull request with `gh pr list --repo "<target-repo>" --head "<branch>" --state open --json number,url,title,baseRefName`.
- When the source and target repositories differ, check for an existing open pull request with `gh pr list --repo "<target-repo>" --state open --json number,url,title,baseRefName,headRefName,headRepositoryOwner` and filter the results to the matching branch and source repository owner.
- When the source and target repositories are the same, create the pull request with `gh pr create --repo "<target-repo>" --base "<base>" --title "<title>" --body "<body>"`.
- When the source and target repositories differ, create the pull request with `gh pr create --repo "<target-repo>" --base "<base>" --head "<source-owner>:<branch>" --title "<title>" --body "<body>"`.

#### GitLab Review Requests

- Check for an existing open merge request with `glab mr list -R "<target-repo>" --source-branch "<branch>" --output json`. When the source and target repositories differ, filter the results to the matching source project path or namespace.
- When the source and target repositories are the same, create the merge request with `glab mr create -R "<target-repo>" --source-branch "<branch>" --target-branch "<base>" --title "<title>" --description "<body>" --yes`.
- When the source and target repositories differ, create the merge request with `glab mr create -R "<target-repo>" -H "<source-repo>" --source-branch "<branch>" --target-branch "<base>" --title "<title>" --description "<body>" --yes`.

Do not use `gh pr create --fill`, `glab mr create --fill`, `--web`, or flags that may push automatically or open editors.

### Step 5: Merge Review Requests

Use this flow when the caller wants to merge an existing review request for the current branch.

- Reuse the current branch's open review request when possible.
- Default to `standard` merge mode unless the caller explicitly approved `admin-bypass`.
- Use squash merges in both merge modes.
- Remove the source branch when the provider supports it.
- If checks, approvals, branch protections, or merge queue rules block a `standard` merge, stop and report `Status: Blocked` with a blocker that says admin-bypass approval is required.
- If the caller approved `admin-bypass`, only use a provider CLI's documented elevated merge flag for that provider. If the provider CLI does not support an elevated merge flag, stop and report `Status: Blocked` with a blocker that says the approved merge mode is unsupported for that provider.
- If the provider CLI reports that the current credentials do not have permission to use the approved merge mode, stop and report `Status: Blocked` with that permission blocker.

#### GitHub Review Merges

- Merge with `gh pr merge <number-or-branch> --repo "<repo>" --squash --delete-branch`.
- When the approved merge mode is `admin-bypass`, merge with `gh pr merge <number-or-branch> --repo "<repo>" --squash --delete-branch --admin`.

#### GitLab Review Merges

- Merge with `glab mr merge <id-or-branch> -R "<repo>" --squash --remove-source-branch --yes`.
- If the approved merge mode is `admin-bypass`, stop and report `Status: Blocked` because the provider CLI does not expose a documented elevated merge flag for this workflow.

If conflicts or permissions block the merge, stop and report the blocking condition. Never infer `admin-bypass` from the provider response alone.

### Step 6: Publish Releases

Use this flow when the caller wants to publish a provider release from an already-pushed Git tag.

- Require the tag to already exist on the selected remote before creating the provider release.
- Require a release title and a notes file path.
- Block the workflow if a release for the same tag already exists.
- Never rely on provider auto-tagging from the default branch.
- Use the selected remote consistently for release lookup, creation, and reporting.
- Callers that need an approval gate may stop after provider detection and existing-release lookup, present the planned notes preview, and invoke release creation only after approval.

#### GitHub Releases

- Check for an existing release with `gh release view "<tag>" --repo "<repo>" --json tagName,url,name`.
- If the release already exists, stop and report it as blocked.
- If it does not exist, create it with `gh release create "<tag>" --repo "<repo>" --title "<tag>" --notes-file "<notes-file>" --verify-tag`.

#### GitLab Releases

- Check for an existing release with `glab release view "<tag>" -R "<repo>" --output json`.
- If the release already exists, stop and report it as blocked.
- If it does not exist, create it with `glab release create "<tag>" -R "<repo>" --name "<tag>" --notes-file "<notes-file>" --no-update`.

If the provider CLI reports that the tag is missing remotely, stop and report that the tag must be pushed to the selected remote before release creation.

## Report

Return a concise provider-aware result that the calling prompt can reuse in its own report:

```text
Provider: github|gitlab
Remote: <remote>
Source Remote: <source-remote>
Artifact: PR|MR|Release
Branch: <branch>
Base: <base>
Tag: <tag>
Merge Mode: <standard|admin-bypass>
Status: Existing|Created|Merged|Blocked
Blocker: <reason>
Identifier: <number-or-id>
URL: <url>
```

## Constraints

- Never guess the provider when detection is ambiguous.
- Never guess the target remote or source remote when resolution is ambiguous.
- Never open both a PR and an MR for the same branch.
- Never update or overwrite an existing release.
- Never push code as part of review creation.
- Never let provider release commands create tags implicitly from the default branch.
- Never guess the base branch when the default branch cannot be resolved cleanly.
- Never infer `admin-bypass`; require explicit caller approval and block when the provider does not support the approved merge mode.
