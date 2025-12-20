# DCF Specification

The canonical source of truth for the **Design Concept Format (DCF)** — a text-first, deterministic format for describing complete UI design systems and application structures.

## What is DCF?

DCF is a specification for describing UI design systems in a way that is:

- **Declarative** — describe what, not how
- **Platform-agnostic** — generate for any target framework
- **Diffable and composable** — version control friendly
- **AI-optimized** — deterministic enough for code generation
- **Themeable** — token-driven with composable theme layers

DCF separates UI concerns into orthogonal layers: tokens, themes, components, layouts, navigation, screens, flows, and rules.

## Repository Structure

```
dcf-spec/
├── spec/                    # Versioned specification documents
│   └── v1.0.0/
│       └── design-concept-format.md
├── schema/                  # JSON Schema definitions
│   └── v1.0.0/
│       ├── dcf.schema.json
│       ├── tokens.schema.json
│       ├── component.schema.json
│       └── ...
├── examples/                # Reference implementations
│   ├── minimal/             # Bare minimum valid DCF
│   ├── standard/            # Typical production usage
│   └── strict/              # Full compliance examples
├── decisions/               # Architecture Decision Records
│   └── 001-versioning.md
├── CHANGELOG.md             # Human-friendly version history
└── README.md
```

## Quick Links

| Resource | Description |
|----------|-------------|
| [Specification](spec/v1.0.0/design-concept-format.md) | Full DCF v1.0.0 specification |
| [JSON Schema](schema/v1.0.0/) | Machine-readable validation schemas |
| [Examples](examples/) | Reference implementations by profile |
| [Changelog](CHANGELOG.md) | Version history and migration notes |

## Core Concepts

### Layers

DCF organizes UI definitions into distinct layers:

| Layer | Purpose |
|-------|---------|
| **Tokens** | Visual constants (colors, spacing, typography, motion) |
| **Themes** | Composable token overrides (dark mode, compact density) |
| **Components** | Reusable UI building blocks with variants and states |
| **Layouts** | Page templates and region definitions |
| **Navigation** | Route graph and transitions |
| **Screens** | Intent-driven compositions with data contracts |
| **Flows** | Multi-step user journeys with error handling |
| **Rules** | Global constraints and guardrails |

### Profiles

DCF supports progressive adoption through validation profiles:

| Profile | Use Case |
|---------|----------|
| `lite` | Prototypes, design extraction, quick mockups |
| `standard` | Production apps, incremental adoption |
| `strict` | Enterprise systems, full compliance |

### Versioning

All DCF files declare their specification version:

```yaml
dcf_version: "1.0.0"
profile: standard

component: Button
# ...
```

## Using DCF

### Validation

Validate DCF files against the JSON Schema:

```bash
# Using ajv-cli
ajv validate -s schema/v1.0.0/dcf.schema.json -d your-file.yaml
```

### Code Generation

DCF files can be used as input for code generators targeting:

- React / React Native
- Vue
- SwiftUI
- Jetpack Compose
- Flutter
- And more...

## Versioning Policy

This specification follows [Semantic Versioning](https://semver.org/):

- **MAJOR** — Breaking changes to the specification
- **MINOR** — Backward-compatible additions (new optional fields)
- **PATCH** — Clarifications, typo fixes, non-functional changes

### Compatibility

- Files with the same MAJOR version are compatible
- Tools should warn on MINOR version mismatch
- PATCH differences are always compatible

## Stable URLs

Published documentation is available at predictable URLs:

```
/spec/v1.0.0/                    # Specification
/schema/v1.0.0/dcf.schema.json   # JSON Schema
/latest/                         # Redirect to newest stable
```

## Related Projects

| Project | Description |
|---------|-------------|
| `dcf-validator` | CLI tool for validating DCF files |
| `dcf-codegen` | Code generators for various platforms |
| `dcf-figma` | Figma plugin for DCF import/export |
| `dcf-vscode` | VS Code extension with IntelliSense |

## Contributing

We welcome contributions! Please see our contributing guidelines for:

- Reporting issues
- Proposing specification changes
- Submitting examples
- Improving documentation

### Architecture Decision Records

Major design decisions are documented as ADRs in the `decisions/` directory. When proposing significant changes, please include a draft ADR.

## License

[MIT](LICENSE)

---

**Current Version:** 1.0.0

For questions or discussions, please open an issue or start a discussion.
