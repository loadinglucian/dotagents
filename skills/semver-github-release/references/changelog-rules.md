# Changelog Rules

## Commit Prefix Mapping

| Prefix | Section |
| --- | --- |
| `feat:` | Added |
| `fix:` | Fixed |
| `security:` | Security |
| `deprecated:` | Deprecated |
| other | Changed |

## Entry Template

```markdown
## [{version}] - {YYYY-MM-DD}

### Added
- {feat commits}

### Fixed
- {fix commits}

### Security
- {security commits}

### Deprecated
- {deprecated commits}

### Changed
- {other commits}
```

Include only non-empty sections.

## Initial Release Template

```markdown
# Changelog

## [1.0.0] - {YYYY-MM-DD}

First release.
```

## No-Change Exit Template

```markdown
## No Changes to Release

There are no commits since v{old_version}. Nothing to release.

To inspect:
- git log --oneline -5
- git describe --tags --abbrev=0
```
