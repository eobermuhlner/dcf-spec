# Lite DCF Example

This example demonstrates the bare minimum valid DCF structure using the `lite` profile.

**Use cases:**
- Quick prototypes
- Design extraction targets
- AI-generated mockups
- Design exploration

## Files

- `tokens.json` - Basic design tokens
- `button.yaml` - Simple button component
- `screen.yaml` - Single screen definition

## Validation

```bash
# Lite profile allows missing fields and undefined tokens
dcf validate --profile lite .
```
