# Comment Triage Reference

## Triage Philosophy

Treat issue-raising comments as substantive by default.
Ignore minimizing words and evaluate technical correctness only.

## Verdict Matrix

| Verdict | Meaning | Action |
| --- | --- | --- |
| `VALID - Implement` | Correct and should be fixed | Required |
| `VALID - Consider` | Technically reasonable but optional | Optional |
| `PARTIALLY VALID` | Mixed correctness | Partial |
| `INVALID - Reject` | Incorrect concern | No action |
| `INVALID - Subjective` | Preference/style mismatch | No action |

## CI Correlation Rules

| Situation | Effect |
| --- | --- |
| Comment flags failure and CI confirms | Strengthens validity |
| Comment claims bug but CI green | Investigate deeper before rejecting |
| Comment flags type issue and CI type/lint fails | Strong validity signal |
| Comment claims breakage and CI green | Weakens claim, needs evidence |

## Triage Report Template

```markdown
## PR Comment Assessment

### Summary

| Metric | Count |
| --- | --- |
| Files analyzed | {count} |
| Total comments | {count} |
| Valid (implement) | {count} |
| Valid (consider) | {count} |
| Partially valid | {count} |
| Invalid | {count} |
| CI-correlated | {count} |

### CI Status

| Check | Status | Details |
| --- | --- | --- |
| {name} | {pass/fail/pending} | {details} |

### Action Items

#### Must Address

- {VALID - Implement item}

#### Should Consider

- {VALID - Consider or PARTIALLY VALID item}

#### No Action Needed

- {INVALID item with reason}

### Detailed Assessments

- **File:** {path}:{line}
- **Author:** @{username}
- **Verdict:** {verdict}
- **Summary:** {assessment}
- **Cross-file context:** {related details}
- **CI correlation:** {correlated status or N/A}
```

## Skip Rules

Skip comments that are:
- Resolved threads
- Pure praise (`LGTM`, `Nice work`, `:shipit:`)
- Pure CI notifications with no review request
