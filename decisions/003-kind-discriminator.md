# ADR 003: Kind Field as File Type Discriminator

**Status:** Accepted
**Date:** 2024-12-23
**Authors:** DCF Team

## Context

DCF defines 10 distinct file types:
- `tokens` - Design token definitions
- `theme` - Theme overrides
- `theming` - Theme configuration
- `component` - UI component definition
- `layout` - Layout template
- `screen` - Screen/page definition
- `navigation` - App navigation
- `flow` - Multi-step flow
- `rules` - Validation/business rules
- `i18n` - Internationalization strings

Before this decision, file type identification was inconsistent:

1. **5 file types** had implicit identification through dual-purpose fields:
   - `component: Button` (both identifies type AND names the component)
   - `screen: HomePage`
   - `layout: Main`
   - `flow: CreateItem`
   - `app: MyApp` (for navigation)

2. **5 file types** had no explicit identification:
   - `tokens`, `theme`, `theming`, `rules`, `i18n`

This created several problems:
- Tools couldn't reliably determine file type without parsing context
- Schema selection required heuristics or file path conventions
- Files weren't self-documenting about their purpose

## Decision

Add a required `kind` field as the universal file type discriminator.

### File Structure

**For entity types** (component, screen, layout, flow, navigation):

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard
name: Button
# ... rest of definition
```

**For non-entity types** (tokens, theme, theming, rules, i18n):

```yaml
dcf_version: "1.0.0"
kind: tokens
# ... rest of definition
```

### Field Order Convention

```yaml
dcf_version: "1.0.0"   # 1. Always first
kind: component        # 2. File type discriminator
profile: standard      # 3. Validation profile (optional)
name: Button           # 4. Entity name (if applicable)
# ... rest of content
```

### Valid `kind` Values

| kind | Has `name`? | Description |
|------|-------------|-------------|
| `tokens` | No | Design token definitions |
| `theme` | No | Theme overrides |
| `theming` | No | Theme configuration |
| `component` | Yes | UI component definition |
| `layout` | Yes | Layout template |
| `screen` | Yes | Screen/page definition |
| `navigation` | Yes | App navigation (name = app name) |
| `flow` | Yes | Multi-step flow |
| `rules` | No | Validation/business rules |
| `i18n` | No | Internationalization strings |

### Field Renames

To eliminate redundancy, the following fields were renamed to `name`:
- `component: Button` → `name: Button`
- `screen: HomePage` → `name: HomePage`
- `layout: Main` → `name: Main`
- `flow: CreateItem` → `name: CreateItem`
- `app: MyApp` → `name: MyApp`

## Consequences

### Positive

- **Unambiguous Type Identification**: `kind` instantly reveals file purpose
- **Consistent Schema Selection**: Tools can use `kind` value to select the correct JSON schema
- **Self-Documenting Files**: Files declare their type explicitly
- **Tooling Simplification**: No heuristics needed for file type detection
- **No Redundancy**: `kind` + `name` is cleaner than `kind` + `component:`

### Negative

- **Breaking Change**: All existing DCF files need updating
- **Additional Field**: Every file now requires the `kind` field

### Mitigations

- v1.0.0 is not yet published, so no backwards compatibility concern
- Simple migration: add `kind` field, rename entity fields to `name`
- Tools can provide automated migration

## Alternatives Considered

### Use `type` Instead of `kind`

**Rejected** because `type` conflicts with common YAML/JSON usage and property names within DCF schemas.

### Keep Dual-Purpose Fields

**Rejected** because it maintains the inconsistency between entity and non-entity types.

### Infer Type from File Extension or Path

**Rejected** because files should be self-documenting and not depend on external context.

## References

- Kubernetes resource manifests (`kind: Deployment`)
- GitHub Actions (`on:`, `jobs:` patterns)
- AWS CloudFormation (`Type: AWS::EC2::Instance`)
