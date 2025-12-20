# Strict DCF Example

This example demonstrates full DCF compliance using the `strict` profile.

**Use cases:**
- Enterprise design systems
- Design system libraries
- CI/CD validation pipelines
- Full specification compliance

## Structure

```
strict/
├── tokens.json              # Complete design tokens
├── themes/
│   ├── light.json
│   ├── dark.json
│   └── compact.json
├── theming.yaml             # Full theme configuration
├── components/
│   ├── primitives/
│   │   ├── text.yaml
│   │   └── container.yaml
│   ├── button.yaml          # Full variant matrix
│   └── card.yaml
├── layouts/
│   └── main.yaml
├── navigation.yaml
├── screens/
│   └── dashboard.yaml
├── flows/
│   └── create-item.yaml
├── i18n/
│   └── en.yaml
└── rules.yaml               # Design constraints
```

## Validation

```bash
# Strict profile validates all fields and constraints
dcf validate --profile strict .
```

## Features Demonstrated

- Complete variant matrices with explicit allow/deny rules
- Full accessibility specifications
- Motion tokens with reduced-motion fallbacks
- RTL-aware icons and logical properties
- Complete i18n with plural forms
- Error recovery strategies in flows
- Design rules and constraints
