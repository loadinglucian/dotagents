---
description: Triage PR comments with CI and full diff context
argument-hint: "[<pr-number>|<owner/repo#pr-number>]"
---

# PR Triage Automation

## Context

Analyzes all changed files in a pull request, validates each
substantive comment with full cross-file context, correlates findings
with CI, and optionally posts friendly replies on GitHub.

## Examples

### Example: Current PR

**user:** `/pr-triage`

**assistant:**
> I'll resolve the current PR from this branch, analyze the changed
> files and comments, correlate them with CI, and report which comments
> are actionable.

### Example: Specific PR

**user:** `/pr-triage 123`

**assistant:**
> I'll analyze PR #123 in the current repository and sort the comments
> by validity, priority, and suggested action.

### Example: Explicit Repo

**user:** `/pr-triage owner/repo#456`

**assistant:**
> I'll analyze PR #456 in owner/repo with full file context and CI
> status, then summarize which comments should be implemented,
> considered, or rejected.

## Instructions

Parse `$ARGUMENTS` to resolve the target pull request.
Support only the argument forms below.
Treat empty input as "current PR for the current branch."
If the argument does not match a supported form, print the usage block
and stop.

## Argument Forms

| Form | Meaning |
| --- | --- |
| empty | Current PR in the current repository |
| `<number>` | PR number in the current repository |
| `<owner>/<repo>#<number>` | Explicit repository and PR number |
| anything else | Output usage help below |

```text
Usage: /pr-triage [<pr-number>|<owner/repo#pr-number>]

Examples:
  /pr-triage
  /pr-triage 123
  /pr-triage owner/repo#456
```

## Comment Philosophy

All comments expressing concerns, flagging issues, or suggesting changes
are substantive by default.
Evaluate technical merit, not reviewer tone.

Ignore:

- Minimizing language such as "minor thing" or "not a big deal"
- Hedging such as "maybe consider" or "might want to"
- Politeness framing such as "feel free to ignore"

## Protocol

### Step 1: Resolve Target

If the repository is not provided, resolve it with:

```bash
gh repo view --json nameWithOwner
```

If the PR number is not provided, resolve the PR for the current branch:

```bash
gh pr view --json number,title,url,headRefName,baseRefName
```

If a PR number is provided, resolve the target PR explicitly:

```bash
gh pr view <number> --repo <owner/repo> \
  --json number,title,url,author,headRefName,baseRefName
```

If no PR can be resolved, report the failure clearly and stop.

### Step 2: Fetch Review Data

Fetch the changed files:

```bash
gh pr diff <number> --repo <owner/repo> --name-only
```

Fetch PR-level comments:

```bash
gh api repos/<owner>/<repo>/issues/<number>/comments
```

Fetch review threads with resolution state and inline comment context:

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      reviewThreads(first: 100) {
        nodes {
          isResolved
          isOutdated
          path
          line
          originalLine
          comments(first: 20) {
            nodes {
              databaseId
              body
              diffHunk
              path
              line
              originalLine
              url
              author { login }
            }
          }
        }
      }
    }
  }
}' -F owner=<owner> -F repo=<repo> -F number=<number>
```

Fetch CI status:

```bash
gh pr checks <number> --repo <owner/repo> \
  --json name,state,conclusion,link,workflow,bucket
```

Capture these fields for each comment:

| Field | Source |
| --- | --- |
| `comment_id` | `id` for PR-level comments, `databaseId` for review comments |
| `body` | Comment text |
| `user.login` | Author username |
| `path` | File path for inline review comments |
| `line` or `originalLine` | Inline comment line number |
| `diffHunk` | Inline code context |
| `isResolved` | Review thread resolution status |
| CI checks | `gh pr checks` output |
| Failed checks | Entries with `bucket: fail` |

### Step 3: Analyze the PR

Analyze all changed files before judging comments.
Read every changed file fully.
Use parallel file reads when practical.

For each changed file, understand:

- Purpose and user-visible behavior
- Key functions, classes, and contracts
- Dependencies, data flow, and invariants
- Related changed files and shared assumptions

Inspect repository instructions and nearby patterns when present:

- `AGENTS.md`
- `CLAUDE.md`
- `README*`
- Adjacent files that implement the same workflow, command, service, or UI pattern

Use the diff hunk only as a pointer.
Base verdicts on whole-file and cross-file understanding.

### Step 4: Filter

Skip non-substantive comments:

| Skip If | Examples |
| --- | --- |
| Resolved or outdated thread | Already addressed in the PR |
| Pure praise | `LGTM`, `Nice work`, `:shipit:` |
| Automated status chatter | CI pass/fail notices with no requested action |
| Duplicated follow-up | Repeats an earlier comment with no new issue |

Treat remaining concern or suggestion comments as substantive.

### Step 5: Assess

For each substantive comment, assess validity using the accumulated
understanding.

Check each comment against:

| Check | Validation |
| --- | --- |
| Correctness | Is the reviewer technically accurate? |
| Cross-file impact | Does the concern still hold across related changes? |
| Project alignment | Does the suggestion match repository patterns and instructions? |
| CI correlation | Do checks strengthen or weaken the claim? |

Use these verdicts:

| Verdict | Meaning | Action |
| --- | --- | --- |
| `VALID - Implement` | Technically correct and should be fixed | Required |
| `VALID - Consider` | Technically reasonable but optional | Optional |
| `PARTIALLY VALID` | Some parts are correct, others are not | Partial |
| `INVALID - Reject` | Technically incorrect or not applicable | None |
| `INVALID - Subjective` | Preference or style disagreement only | None |

Apply CI context when available:

| Scenario | Effect |
| --- | --- |
| Comment flags failure and CI confirms | Strengthen validity |
| Comment claims bug but CI is green | Investigate deeper before rejecting |
| Comment flags type or lint issue and CI fails there | Strong validity signal |
| Comment claims breakage and CI is green | Weakens claim, but does not prove it false |

### Step 6: Report

Output findings using this format:

```text
## PR Comment Assessment

### Summary

| Metric | Count |
| --- | --- |
| Files analyzed | {count} |
| Total comments | {count} |
| Comments skipped | {count} |
| Valid (implement) | {count} |
| Valid (consider) | {count} |
| Partially valid | {count} |
| Invalid | {count} |
| CI-correlated | {count} |

### CI Status

| Check | Status | Details |
| --- | --- | --- |
| {name} | {pass/fail/pending} | {details or "—"} |

### Action Items

#### Must Address

- {VALID - Implement item with file:line}

#### Should Consider

- {VALID - Consider or PARTIALLY VALID item with file:line}

#### No Action Needed

- {INVALID item with brief reason}

### Detailed Assessments

- **Comment ID:** {id}
- **File:** {path}:{line or "N/A"}
- **Author:** @{username}
- **Verdict:** {verdict}
- **Summary:** {1-2 sentence assessment}
- **Cross-file context:** {related details or "N/A"}
- **CI correlation:** {relevant check status or "N/A"}

If no actionable comments remain after filtering, report:
"No actionable comments found."
```

### Step 7: Confirm Replies

After showing the report, ask:

```text
Would you like me to reply to these comments on GitHub?
(yes / yes, only <comment IDs> / no)
```

Wait for explicit user confirmation before posting anything.

If the user says `no`, stop.

### Step 8: Reply Post

For each selected comment:

1. Draft a concise, collaborative reply grounded in the verdict
2. Thank the reviewer first
3. Use warm `we` wording
4. Keep replies to 2-3 sentences
5. Use at most one emoji
6. Never mention AI, automation, or hidden reasoning

Post review comment replies with:

```bash
gh api repos/<owner>/<repo>/pulls/<number>/comments/<comment_id>/replies \
  -f body='<reply>'
```

Post PR-level follow-up comments with:

```bash
gh api repos/<owner>/<repo>/issues/<number>/comments \
  -f body='<reply>'
```

Track successes and failures for the final reply report.

### Step 9: Reply Report

Use this format after posting replies:

```text
## Reply Complete

| Metric | Value |
| --- | --- |
| PR | {owner/repo}#{number} |
| Replies requested | {count} |
| Replies posted | {count} |
| Failures | {count} |

### Posted Replies

- {comment_id}: {status} - {short summary}
```

## Standards

- Evaluate technical merit, not reviewer tone
- Read every changed file before assigning verdicts
- Use cross-file context before rejecting a concern
- Use CI as corroborating evidence, not as the sole justification
- Separate required fixes from optional suggestions
- Keep replies warm, concise, and specific
- Cite file and line context whenever available
- Never mention AI in reports or posted replies

## Constraints

- Do not modify code in this command
- Do not rerun or cancel CI unless the user explicitly asks
- Do not post replies without explicit confirmation
- Do not reject a comment without checking related files and patterns
- Keep any CI log excerpts short and relevant
- Skip non-substantive comments instead of over-analyzing them

## User Query

`$ARGUMENTS`
