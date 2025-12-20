# Standard DCF Example

This example demonstrates typical production usage of DCF using the `standard` profile.

**Use cases:**
- Production applications
- Incremental DCF adoption
- Real-world component libraries

## Structure

```
standard/
├── tokens.json          # Design tokens
├── themes/
│   ├── light.json       # Light theme
│   └── dark.json        # Dark theme
├── theming.yaml         # Theme configuration
├── components/
│   ├── button.yaml      # Button with variants
│   └── input.yaml       # Form input
├── layouts/
│   └── main.yaml        # Main layout template
├── navigation.yaml      # Route definitions
└── screens/
    └── users.yaml       # Users list screen
```

## Validation

```bash
# Standard profile enforces required fields
dcf validate --profile standard .
```
