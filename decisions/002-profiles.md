# ADR 002: Validation Profiles for Progressive Adoption

**Status:** Accepted
**Date:** 2024-12-20
**Authors:** DCF Team

## Context

DCF serves multiple use cases with different requirements:

1. **Quick Prototypes**: Designers extracting concepts from mockups
2. **Production Apps**: Teams adopting DCF incrementally
3. **Enterprise Systems**: Organizations requiring strict compliance

A single validation strictness level would either:
- Be too strict for prototyping (friction in early adoption)
- Be too loose for enterprise use (insufficient guardrails)

## Decision

We introduce three validation profiles: `lite`, `standard`, and `strict`.

### Profile Definitions

| Profile | Missing Required | Undefined Tokens | Incomplete Variants | Accessibility |
|---------|------------------|------------------|---------------------|---------------|
| `lite` | warn | allow | allow | skip |
| `standard` | error | warn | warn | warn |
| `strict` | error | error | error | error |

### Declaration

```yaml
dcf_version: "1.0.0"
profile: standard  # lite | standard | strict
```

### Default Behavior

If `profile` is omitted, tools assume `standard`.

### Profile Scope

- Profile applies to the entire file
- Cannot be mixed within a single file
- CI/CD can override with stricter enforcement

## Consequences

### Positive

- **Progressive Adoption**: Teams start with `lite` and tighten over time
- **Appropriate Strictness**: Each use case gets fitting validation
- **Clear Expectations**: Profile name communicates intent

### Negative

- **Complexity**: Three validation modes to implement and test
- **Potential Confusion**: Users may not understand profile differences

### Mitigations

- Clear documentation of profile behaviors
- Tooling shows which profile is active
- Validators explain why a check was skipped/warned/errored

## Alternatives Considered

### Single Strictness Level

**Rejected** because it forces a one-size-fits-all approach that serves no use case well.

### Per-Field Strictness

**Rejected** because it creates complexity and makes validation behavior unpredictable.

### Numeric Strictness Levels (1-10)

**Rejected** because named profiles are more intuitive and self-documenting.

## References

- ESLint configuration levels
- TypeScript `strict` mode
