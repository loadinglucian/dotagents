---
name: github-pr-operations
description: "Analyze GitHub pull requests end-to-end: CI status and failures, technical triage of review comments, and friendly reviewer replies. Use when working on PR checks, PR comment validity assessment, or posting PR review responses with gh CLI."
---

# GitHub PR Operations

## Overview

Run PR-focused GitHub workflows with consistent analysis and reporting.
Handle CI diagnosis, PR comment triage, and reviewer replies in one reusable skill.

## Workflow Decision

### Step 1: Classify the requested operation

Choose one mode:
- `ci-analysis`: User asks why CI is failing, blocked, or pending.
- `comment-triage`: User asks which PR comments are valid/actionable.
- `reply-posting`: User asks to post or draft replies to review comments.
- `full-pr-ops`: Perform triage with CI context, then optionally post replies.

### Step 2: Resolve target scope

Determine target repository and PR:
- If `owner/repo` not given, resolve with `gh repo view --json nameWithOwner -q '.nameWithOwner'`.
- If PR number is not provided, infer from current branch with `gh pr view --json number,headRefName`.
- If a branch is provided for CI-only analysis, use branch run listing mode.

### Step 3: Execute selected mode

#### Mode: CI analysis

1. Fetch checks or runs:
- PR checks: `gh pr checks {pr} --json name,state,conclusion,link,workflow,bucket`
- Branch runs: `gh run list --branch {branch} --limit 5 --json databaseId,name,conclusion,status,workflowName,url`
2. For failed checks, inspect failed logs:
- `gh run view {run_id} --log-failed | tail -100`
3. Classify failures by error pattern and infer likely cause.
4. Output CI status report.

#### Mode: Comment triage

1. Fetch changed files and comments:
- `gh pr diff --name-only`
- `gh api /repos/{owner}/{repo}/issues/{pr}/comments`
- `gh api /repos/{owner}/{repo}/pulls/{pr}/comments`
2. Fetch CI status for corroboration:
- `gh pr checks {pr} --json name,state,conclusion,link,workflow,bucket`
3. Analyze changed files before assessing comments.
4. Skip non-substantive comments (pure praise, resolved threads, automated status chatter).
5. Score each substantive comment with verdict:
- `VALID - Implement`
- `VALID - Consider`
- `PARTIALLY VALID`
- `INVALID - Reject`
- `INVALID - Subjective`
6. Output triage report with file/line context.

#### Mode: Reply posting

1. Parse message, target type, and IDs.
2. Transform tone to collaborative and concise.
3. Post via the correct endpoint:
- Review comment reply: `gh api repos/{owner}/{repo}/pulls/{pr}/comments/{comment_id}/replies -f body=...`
- PR-level comment: `gh api repos/{owner}/{repo}/issues/{pr}/comments -f body=...`
4. Output comment-post report.

#### Mode: Full PR ops

1. Run comment triage with CI context.
2. Ask whether to post replies.
3. If yes, generate replies from verdicts and post each requested reply.
4. Output combined summary (triage + reply status).

## Operating Rules

- Treat concern/suggestion comments as substantive by default.
- Evaluate technical merit, not reviewer tone.
- Include CI correlation when it strengthens or weakens a verdict.
- Keep replies warm, short, and direct.
- Use no more than one emoji in a posted reply.

## References

- `references/ci-analysis.md`
- `references/comment-triage.md`
- `references/reply-tone.md`

## Output Requirements

Always provide:
1. A structured report matching the selected mode.
2. Explicit verdicts and rationale for each assessed comment.
3. Run URLs for failed CI checks when available.
4. Exact posting status when replies are sent.

## Constraints

- Do not implement code fixes in this skill.
- Do not rerun CI jobs unless explicitly requested.
- Do not post raw terse messages without tone transformation.
- Limit failed-log excerpts to relevant lines (default: last 100).
