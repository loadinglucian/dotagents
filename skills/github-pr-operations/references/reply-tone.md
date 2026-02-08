# Reply Tone Reference

## Transformation Rules

- Thank the reviewer first.
- Prefer collaborative `we` wording.
- Keep replies concise (usually 2-3 sentences).
- Use at most one emoji.
- Preserve meaning; improve tone.

## Terse Input Expansions

| Input | Suggested Transformation |
| --- | --- |
| `ok` | `Sounds good, thanks for the suggestion.` |
| `fixed` | `Great catch. We pushed a fix for this.` |
| `done` | `All set. Thanks for flagging this.` |
| `won't fix` | `Thanks for the suggestion. We reviewed it and will keep the current approach for now.` |
| `no` | `Thanks for raising this. We are going to pass on this one because {reason}.` |

## Anti-Patterns

Avoid:
- `No.`
- `Wrong.`
- `Already handled.`
- `See previous comment.`
- `That's not how it works.`
- `You're mistaken.`

Rewrite with context and respect.

## Comment Post Report Template

```markdown
## PR Comment Posted

### Details

| Field | Value |
| --- | --- |
| PR | #{pr_number} |
| Type | {reply/new comment} |
| Target | {comment_id or PR-level} |

### Original Input

> {original_message}

### Posted Comment

> {transformed_message}

### Status

{Success / Failed with error}
```
