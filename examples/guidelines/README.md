# Guidelines DCF Example

This example demonstrates DCF used for design system documentation without a concrete application. It defines tokens, themes, and reusable components but no screens, layouts, or navigation.

**Use cases:**
- Design system documentation
- Component library specifications
- Brand guidelines
- Cross-application design standards
- Style guide foundations

## Structure

```
guidelines/
├── tokens.json          # Comprehensive design tokens
├── themes/
│   ├── light.json       # Light theme values
│   └── dark.json        # Dark theme values
├── theming.yaml         # Theme configuration
└── components/
    ├── button.yaml      # Button component with variants
    ├── card.yaml        # Card container component
    └── typography.yaml  # Typography components
```

## What's Included

- **Design Tokens**: Colors, spacing, typography, radii, motion, and breakpoints
- **Theming**: Light and dark theme support with configurable token groups
- **Components**: Reusable UI building blocks with variants, states, and accessibility

## What's NOT Included

- Screens (no concrete application)
- Navigation (no routes or flows)
- Layouts (no page templates)
- Data contracts (no application-specific data)

## Validation

```bash
# Guidelines profile validates design system completeness
dcf validate --profile standard .
```
