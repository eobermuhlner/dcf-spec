# Changelog

All notable changes to the DCF specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-12-20

### Added

#### Core Specification
- Initial release of Design Concept Format specification
- File versioning with semantic versioning (`dcf_version` required in all files)
- Validation profiles: `lite`, `standard`, `strict`
- Capability declarations for partial adoption

#### Design Tokens (Section 1)
- Color, spacing, radius, and typography tokens
- Motion tokens with duration and easing values
- Asset tokens for icons and theme-aware images
- RTL-aware icon definitions with `ltr`/`rtl` variants

#### Theming (Section 2)
- Theme set files with composable layers
- Theme merge contract with configurable order
- Responsive breakpoint tokens
- Density and shape variant support

#### Components (Section 3)
- Component definitions with props, variants, and states
- State-driven styling with precedence rules
- Variant matrix for valid combinations
- Primitive components (Text, Container, Icon, Image)
- Component categories for organization

#### Layouts (Section 4)
- Layout templates with region definitions
- Persistent and dynamic region roles
- Responsive region behavior

#### Navigation (Section 5)
- Route graph with path patterns
- Transition definitions with triggers
- Parent-child route relationships

#### Screens (Section 6)
- Screen definitions with intent declarations
- Standard intents: browse, view, edit, create, confirm, error, empty, loading
- Extended intents: authenticate, callback, search, settings, dashboard
- Data contracts with source types (api, context, local, derived, static)
- Data states (idle, loading, success, error, stale)

#### Flows (Section 7)
- Multi-step interaction flows
- Error context passing between steps
- Error types and recovery strategies

#### Rules (Section 8)
- Navigation constraints (max depth, destructive confirmation)
- Layout rules (primary action limits)
- Accessibility rules (touch targets, focus visibility)

#### Internationalization (Section 10)
- RTL/LTR layout direction support
- String externalization with fallbacks
- Plural forms following CLDR rules
- Locale-aware date, number, and list formatting

### JSON Schema
- Root schema (`dcf.schema.json`)
- Tokens schema (`tokens.schema.json`)
- Component schema (`component.schema.json`)
- Layout schema (`layout.schema.json`)
- Screen schema (`screen.schema.json`)
- Navigation schema (`navigation.schema.json`)
- Flow schema (`flow.schema.json`)
- Theme schema (`theme.schema.json`)
- Theming config schema (`theming.schema.json`)
- Rules schema (`rules.schema.json`)
- i18n schema (`i18n.schema.json`)

### Examples
- Minimal example (lite profile)
- Standard example (production usage)
- Strict example (full compliance)

### Documentation
- Architecture Decision Records (ADRs)
  - ADR 001: Semantic Versioning
  - ADR 002: Validation Profiles

---

## [Unreleased]

### Planned
- Additional primitive components
- Animation sequence definitions
- Conditional styling expressions
- Design-to-code mapping hints
