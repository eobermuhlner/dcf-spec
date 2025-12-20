# ADR 001: Semantic Versioning for DCF Specification

**Status:** Accepted
**Date:** 2024-12-20
**Authors:** DCF Team

## Context

The Design Concept Format (DCF) specification needs a versioning strategy that:

1. Communicates the impact of changes to implementers
2. Enables tooling to validate compatibility
3. Supports graceful migration between versions
4. Is widely understood by the developer community

## Decision

We adopt [Semantic Versioning 2.0.0](https://semver.org/) for the DCF specification.

### Version Format

```
MAJOR.MINOR.PATCH
```

- **MAJOR**: Breaking changes that require migration
- **MINOR**: Backward-compatible additions (new optional fields, sections)
- **PATCH**: Clarifications, typo fixes, non-functional changes

### File Version Declaration

All DCF files must declare their specification version as the first key:

```yaml
dcf_version: "1.0.0"
```

### Compatibility Rules

1. **Same MAJOR**: Files are compatible
2. **Different MINOR**: Tools should warn but proceed
3. **Different PATCH**: Always compatible

### Tool Behavior

| Scenario | Tool Behavior |
|----------|---------------|
| File MAJOR > Tool MAJOR | Reject with error |
| File MAJOR < Tool MAJOR | Process with deprecation warnings |
| File MINOR > Tool MINOR | Process with unknown field warnings |
| File PATCH differs | Process normally |

## Consequences

### Positive

- **Predictability**: Implementers know what to expect from version bumps
- **Tooling**: Validators can make informed decisions about compatibility
- **Communication**: Version numbers convey change impact at a glance
- **Industry Standard**: SemVer is familiar to most developers

### Negative

- **Discipline Required**: Maintainers must correctly categorize changes
- **Migration Burden**: MAJOR bumps require migration guides and tooling updates

### Mitigations

- MAJOR version bumps must include:
  - Migration guide in `/spec/vX.0.0/MIGRATION.md`
  - Codemods or migration tooling where feasible
  - Deprecation period in the previous MINOR release

## Alternatives Considered

### CalVer (Calendar Versioning)

**Rejected** because it doesn't communicate change impact. A release in January could be a breaking change or a typo fix.

### Single Incrementing Version

**Rejected** because it provides no information about compatibility or change scope.

### Git Commit Hashes

**Rejected** because they're not human-readable and don't indicate compatibility relationships.

## References

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
