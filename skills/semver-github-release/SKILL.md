---
name: semver-github-release
description: Automate semantic version releases with changelog generation, git tagging, and GitHub release publishing. Use when preparing patch/minor/major releases from commit history and requiring explicit approval before publishing.
---

# Semver GitHub Release

## Overview

Automate release preparation and publication with semver and keep-a-changelog style output.
Require explicit user approval before committing changelog updates or creating releases.

## Workflow

### Step 1: Resolve release bump

Interpret user intent:
- `patch` (default)
- `minor`
- `major`

If no tags exist, treat as initial release (`1.0.0`).

### Step 2: Discover current version

Get latest tag:
- `git describe --tags --abbrev=0 2>/dev/null`

Normalize by stripping leading `v` for calculations.

### Step 3: Validate releasability

If prior tag exists, check commits since last tag:
- `git log v{old_version}..HEAD --oneline`

If empty, stop and report `No Changes to Release`.

### Step 4: Calculate new version

Apply bump rules:
- major -> `MAJOR+1.0.0`
- minor -> `MAJOR.MINOR+1.0`
- patch -> `MAJOR.MINOR.PATCH+1`

Tag format: `v{version}`.

### Step 5: Build changelog entry

Gather commit subjects since last tag and categorize by prefix:
- `feat:` -> Added
- `fix:` -> Fixed
- `security:` -> Security
- `deprecated:` -> Deprecated
- other -> Changed

Update or create `CHANGELOG` with newest entry near the top.

### Step 6: Approval gate

Present release summary and changelog preview.
Require explicit user approval before any commit, tag, or release creation.

### Step 7: Publish release

After approval:
1. Commit changelog update.
2. Push branch.
3. Create GitHub release with notes from changelog entry.

Example:
- `gh release create v{version} --title v{version} --notes-file /tmp/release-notes.md`

### Step 8: Report

Output release summary including version, tag, release URL, and changelog excerpt.

## References

- `references/changelog-rules.md`
- `references/release-report-template.md`

## Standards

- Follow semantic versioning (`MAJOR.MINOR.PATCH`).
- Keep changelog entries concise and user-facing.
- Confirm release contents before publishing.
- Publish GitHub releases with meaningful release notes.

## Constraints

- Do not proceed without explicit user confirmation.
- Do not rewrite existing tags/releases.
- Do not force-push tags.
- Do not generate prerelease tags unless requested.
