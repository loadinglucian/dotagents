---
description: Cut a patch, minor, or major release from the default branch and publish it through the repository host
argument-hint: [patch|minor|major]
---

# Git Cut Release

Use `$git-ops` for provider-specific remote Git operations in this workflow.

## Protocol

1. Read the requested bump from `$1` and treat `$ARGUMENTS` as the raw argument string for validation. If no positional argument is provided, use `patch`.
2. If `$ARGUMENTS` contains more than one argument, or if the provided bump is not exactly `patch`, `minor`, or `major`, stop and show this usage:

   ```text
   Usage: /prompts:git-cut-release [patch|minor|major]

   Examples:
     /prompts:git-cut-release
     /prompts:git-cut-release patch
     /prompts:git-cut-release minor
     /prompts:git-cut-release major
   ```

3. Inspect the current branch, the working tree, and the configured remotes. Resolve a single canonical release remote from the default-branch context before you do any release work:
   - prefer a remote literally named `upstream`, because releases must be published from the canonical repository when that remote is configured
   - otherwise prefer the current branch's tracked remote
   - if no tracked remote is available and exactly one remote is authoritative for the default branch, use that remote
   - if more than one plausible canonical release remote exists, or none can be identified cleanly, stop and ask the user which remote should receive the release
4. Refuse to continue unless the current branch name matches the resolved release remote's default branch. Refuse to continue if the working tree is not clean. In this workflow, the release changelog commit is an explicit exception to repository rules that otherwise forbid direct commits or pushes to the default branch.
5. Discover the latest stable semantic-version release tag that is reachable from the resolved release remote's default branch tip. Consider only tags that match `vMAJOR.MINOR.PATCH` or `MAJOR.MINOR.PATCH`, compare them by semantic version order rather than lexicographic order, and ignore prerelease, build-metadata, deploy, and other non-release tags. If no matching tag exists there, use `1.0.0` as the next version and skip the release-delta validation step.
6. If a matching release tag exists, strip a leading `v`, require commits since that tag, and compute the next semantic version from the requested bump:
   - `patch`: increment patch
   - `minor`: increment minor and reset patch
   - `major`: increment major and reset minor and patch
7. Build a keepachangelog-style release entry dated for today:
   - Initial release with no prior tag: `## [1.0.0] - YYYY-MM-DD` followed by `First release.`
   - Otherwise, gather commit subjects since the latest tag, map `feat:` to `Added`, `fix:` to `Fixed`, `security:` to `Security`, `deprecated:` to `Deprecated`, and everything else to `Changed`.
8. Draft the changelog preview without editing files yet. If `CHANGELOG.md` exists, determine where the new entry will be inserted so the preview matches the eventual file update. If it does not exist, preview the simple `CHANGELOG.md` file that will be created.
9. Present the release summary, the resolved release remote, and the changelog preview. Stop and wait for explicit user approval before any mutating step. If the user asks for changes, revise the changelog preview first.
10. After approval:
    - update or create `CHANGELOG.md` with the previewed entry
    - commit the changelog update with `docs(changelog): add entry for v<version>`
    - create the tag `v<version>` on that commit
    - push the release commit to the resolved release remote
    - push the tag to the resolved release remote
    - verify the tag exists remotely on the resolved release remote before creating the provider release

11. Write the changelog entry to a temporary notes file and create the provider release:
    - use the resolved release remote as the target remote
    - detect whether the repository is hosted on GitHub or GitLab
    - confirm that a provider release for `v<version>` does not already exist
    - create the provider release from the already-pushed tag using the notes file

12. Finish with a report that includes the version, tag, commit count, provider, remote, release status, release URL, and the changelog entry that was published.

## Standards

- Use semantic versioning in `MAJOR.MINOR.PATCH` form.
- Keep tags in `vX.Y.Z` format.
- Publish releases only from the default branch.
- Require the tag to be pushed before the provider release is created.
- Treat the release changelog commit and push as an explicit exception to repository rules that normally forbid direct commits or pushes to the default branch.

## Constraints

- Never create prerelease tags or release names.
- Never proceed without explicit user approval.
- Never edit `CHANGELOG.md`, commit, tag, or push before presenting the release summary and receiving explicit user approval.
- Never let GitHub or GitLab auto-create the release tag from the default branch.
- Never update or overwrite an existing provider release for the same tag.
- Never assume `origin` is the canonical release remote.
- Never force push commits or tags.
