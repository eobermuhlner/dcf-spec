# Design Concept Format (DCF)

**Version:** 1.0.0

**Purpose**
A text-first, deterministic format for describing a complete UI design system and application structure, optimized for AI-assisted UI development, code generation, validation, and theming.

**Principles**
- Declarative, not imperative
- Platform-agnostic
- Diffable and composable
- Explicit intent and constraints
- No visual ambiguity
- Theming is token-driven and deterministic

---

## 0. File Versioning

**Scope:** Version declaration for all DCF artifacts
**Goal:** Enable compatibility checking, migration tooling, and deterministic parsing

All DCF files must declare the specification version they conform to:

**YAML Files**

```yaml
dcf_version: "1.0.0"
kind: component
name: Button
# ... rest of component definition
```

**JSON Files**

```json
{
  "dcf_version": "1.0.0",
  "color": {
    "bg": { "value": "#FFFFFF" }
  }
}
```

**Version Format**

DCF uses [Semantic Versioning](https://semver.org/):

- `MAJOR.MINOR.PATCH` (e.g., `1.0.0`, `1.2.3`)
- **MAJOR**: Breaking changes to the specification
- **MINOR**: Backward-compatible additions (new optional fields, sections)
- **PATCH**: Clarifications, typo fixes, non-functional changes

**Compatibility Rules**

```yaml
compatibility:
  # Files with same MAJOR version are compatible
  # Tools should warn on MINOR version mismatch
  # PATCH differences are always compatible

  check: major_match
  warn_on: minor_mismatch
  ignore: patch_difference
```

**Rules**
- Every DCF file must include `dcf_version` as a required field (first key recommended)
- Version must be a valid semver string matching pattern `^[0-9]+\.[0-9]+\.[0-9]+$`
- Tools must validate version before parsing
- Older tools should reject files with higher MAJOR versions
- Migration guides must accompany MAJOR version bumps

---

### 0.1 Profiles

**Scope:** Declare completeness and validation strictness
**Goal:** Avoid one-size-fits-all validation; support progressive adoption

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard  # lite | standard | strict
name: Button
# ...
```

**Profile Levels**

| Profile | Purpose | Validation |
|---------|---------|------------|
| `lite` | Mockups, prototypes, design extraction targets | Warnings only, missing fields allowed |
| `standard` | Real applications, partial DCF adoption | Required fields enforced, optional fields warned |
| `strict` | Large systems, full DCF compliance | All fields validated, no implicit defaults |

**Profile Behaviors**

```yaml
profiles:
  lite:
    missing_required_fields: warn
    undefined_tokens: allow
    incomplete_variants: allow
    accessibility_rules: skip
    use_case: "Quick prototypes, AI extraction, design exploration"

  standard:
    missing_required_fields: error
    undefined_tokens: warn
    incomplete_variants: warn
    accessibility_rules: warn
    use_case: "Production apps, incremental adoption"

  strict:
    missing_required_fields: error
    undefined_tokens: error
    incomplete_variants: error
    accessibility_rules: error
    use_case: "Enterprise systems, design system libraries, CI validation"
```

**Default Profile**

If `profile` is omitted, tools should assume `standard`.

**Rules**
- Profile applies to the entire file (not individual sections)
- Tools must respect profile when validating
- Stricter profiles are supersets of looser ones
- CI/CD pipelines can override profile (e.g., enforce `strict` in production)

---

### 0.2 Capabilities

**Scope:** Declare which DCF layers are actually used
**Goal:** Enable partial adoption, focused validation, and accurate AI generation

```yaml
dcf_version: "1.0.0"
profile: standard

capabilities:
  tokens: true
  themes: false
  components: true
  layouts: true
  navigation: true
  screens: true
  flows: false
  rules: false
  i18n: false
```

**Why Capabilities Matter**

| Benefit | Description |
|---------|-------------|
| AI accuracy | Generators won't invent unused layers |
| Faster validation | Validators skip irrelevant checks |
| Partial adoption | Use only what you need without warnings |
| Clear intent | Explicit about what's in scope |

**Capability Definitions**

```yaml
capability_definitions:
  tokens:
    description: Design tokens (colors, spacing, typography, motion, assets)
    files: [tokens.base.json, tokens/*.json]

  themes:
    description: Theme sets and merge configuration
    files: [theming.yaml, themes/*.json]
    requires: [tokens]

  components:
    description: Reusable UI component definitions
    files: [components/*.yaml]
    requires: [tokens]

  layouts:
    description: Page templates and region definitions
    files: [layouts/*.yaml]

  navigation:
    description: Route graph and transitions
    files: [navigation.yaml]
    requires: [layouts, screens]

  screens:
    description: Screen definitions with data contracts
    files: [screens/*.yaml]
    requires: [components]

  flows:
    description: Multi-step interaction flows
    files: [flows/*.yaml]
    requires: [screens]

  rules:
    description: Global constraints and guardrails
    files: [rules.yaml]

  i18n:
    description: Internationalization and localization
    files: [i18n/*.yaml]
```

**Implied Dependencies**

When a capability is enabled, its dependencies are implicitly enabled:

```yaml
# If you declare:
capabilities:
  screens: true

# Tools should infer:
capabilities:
  tokens: true      # implied by components
  components: true  # required by screens
  screens: true     # declared
```

**Default Capabilities**

If `capabilities` is omitted, tools should assume all layers are in use.

**Rules**
- Capabilities are optional but recommended for partial adoption
- Disabled capabilities should not generate validation errors for missing files
- AI tools must respect capabilities when generating new content
- Dependencies are soft requirements (warn, don't error if missing)

---

### 0.3 File Type Identification

**Scope:** Explicit file type discriminator for all DCF files
**Goal:** Unambiguous schema selection, consistent structure, self-documenting files

All DCF files must declare their type using the `kind` field:

```yaml
dcf_version: "1.0.0"
kind: screen
profile: standard
name: UsersPage
# ... rest of content
```

**Valid `kind` Values**

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

**Field Order Convention**

The recommended field order for DCF files:

```yaml
dcf_version: "1.0.0"   # 1. Always first
kind: component        # 2. File type discriminator
profile: standard      # 3. Validation profile (optional)
name: Button           # 4. Entity name (if applicable)
# ... rest of content
```

**Examples**

Token file (no `name` field):
```json
{
  "dcf_version": "1.0.0",
  "kind": "tokens",
  "color": {
    "primary": { "value": "#007AFF" }
  }
}
```

Component file (with `name` field):
```yaml
dcf_version: "1.0.0"
kind: component
name: Button
category: control
props:
  label: string
```

**Rules**
- Every DCF file must include `kind` as a required field
- The `kind` value determines which schema validates the file
- Files with named entities (component, screen, layout, flow, navigation) must include `name`
- The `name` field uses PascalCase for components, screens, layouts, and flows

---

## 1. Design Tokens

**Scope:** Visual constants only (colors, spacing, typography, radii, shadows, etc.)
**No semantics, no behavior, no routing**

Token files support the optional `profile` field for validation level (lite/standard/strict).

```json
{
  "color": {
    "bg": { "value": "{theme.color.bg}" },
    "fg": { "value": "{theme.color.fg}" },
    "accent": { "value": "{theme.color.accent}" }
  },
  "space": {
    "xs": { "value": "{theme.space.xs}" },
    "sm": { "value": "{theme.space.sm}" },
    "md": { "value": "{theme.space.md}" }
  },
  "radius": {
    "sm": { "value": "{theme.radius.sm}" },
    "md": { "value": "{theme.radius.md}" },
    "lg": { "value": "{theme.radius.lg}" }
  },
  "font": {
    "body": {
      "family": "Inter",
      "size": "16px",
      "lineHeight": "1.5"
    }
  }
}
```

**Font Token Properties**
- `family`: Font family name (string)
- `size`: Font size with units, e.g., `"16px"`, `"1rem"` (string)
- `lineHeight`: Line height, unitless or with units (string or number)
- `weight`: Font weight, e.g., `400`, `"bold"` (string or number)
- `letterSpacing`: Letter spacing with units (string)

Font tokens must define at least one property.

**Rules**
- Tokens must be atomic and reusable
- Components must reference tokens (no hardcoded values)
- Token values must be concrete (e.g., `#FFFFFF`) **or** theme references (e.g., `{theme.color.bg}`)
- Tokens must not reference components, screens, or layouts

---

### 1.1 Motion Tokens

**Scope:** Animation timing, easing, and motion preferences
**Goal:** Consistent, accessible animations across all components

```json
{
  "duration": {
    "instant": { "value": "0ms" },
    "fast": { "value": "150ms" },
    "normal": { "value": "250ms" },
    "slow": { "value": "400ms" }
  },
  "easing": {
    "linear": { "value": "linear" },
    "ease-in": { "value": "cubic-bezier(0.4, 0, 1, 1)" },
    "ease-out": { "value": "cubic-bezier(0, 0, 0.2, 1)" },
    "ease-in-out": { "value": "cubic-bezier(0.4, 0, 0.2, 1)" },
    "spring": { "value": "cubic-bezier(0.34, 1.56, 0.64, 1)" }
  },
  "motion": {
    "reduce": { "value": "prefers-reduced-motion" }
  }
}
```

**Motion in Components**

```yaml
kind: component
name: Button
states:
  hover:
    transition:
      property: background
      duration: duration.fast
      easing: easing.ease-out

accessibility:
  reduced_motion:
    transition: none
```

**Rules**
- All animations must use duration/easing tokens
- Components must define `reduced_motion` fallback
- No animation exceeds `duration.slow` without justification
- Transitions should target specific properties, not `all`

---

### 1.2 Asset Tokens

**Scope:** Icons, images, and other visual assets
**Goal:** Centralized, themeable asset references

**Icon Set Definition**

```json
{
  "icon": {
    "set": "lucide",
    "prefix": "icon",
    "fallback": "icon.placeholder"
  },
  "icons": {
    "arrow-left": { "ltr": "lucide:arrow-left", "rtl": "lucide:arrow-right" },
    "arrow-right": { "ltr": "lucide:arrow-right", "rtl": "lucide:arrow-left" },
    "check": { "value": "lucide:check" },
    "close": { "value": "lucide:x" },
    "menu": { "value": "lucide:menu" },
    "user": { "value": "lucide:user" },
    "placeholder": { "value": "lucide:help-circle" }
  }
}
```

**Icon Format**
- Directional icons use `{ "ltr": "...", "rtl": "..." }` format
- Non-directional icons use `{ "value": "..." }` format
- See Section 10.4 for full RTL-aware icon handling

**Icon Usage in Components**

```yaml
kind: component
name: Button
props:
  label: string
  icon:
    type: icon_ref
    optional: true
    position: [leading, trailing]
  iconOnly:
    type: boolean
    requires: icon

layout:
  icon:
    size:
      sm: 16px
      md: 20px
      lg: 24px
    color: inherit
    margin:
      leading: space.none
      trailing: space.xs
```

**Theme-Aware Assets**

```json
{
  "asset": {
    "logo": {
      "light": "/assets/logo-dark.svg",
      "dark": "/assets/logo-light.svg"
    },
    "empty-state": {
      "value": "/assets/empty-{theme.mode}.svg"
    }
  }
}
```

**Rules**
- Icons referenced by token name, not raw path
- Icon sets must be declared (lucide, heroicons, custom)
- Theme-variant assets use `{theme.*}` interpolation
- `iconOnly` buttons must have accessible label via `aria-label`
- Icons with directional meaning (arrows) need RTL variants

---

## 2. Theming

**Scope:** User- or product-selectable theme sets that supply token values
**Goal:** Deterministic composition (e.g., *dark + compact + rounded*)

### 2.1 Theme Set Files

Recommended: store themes as separate JSON files that can be merged.

```text
design/
 ├─ tokens.base.json
 └─ themes/
     ├─ light.json
     ├─ dark.json
     ├─ compact.json
     └─ rounded.json
```

Example `themes/light.json`:

```json
{
  "theme": {
    "color": {
      "bg": "#FFFFFF",
      "fg": "#111827",
      "accent": "#2563EB"
    },
    "space": {
      "xs": "4px",
      "sm": "8px",
      "md": "16px"
    },
    "radius": {
      "sm": "6px",
      "md": "12px",
      "lg": "16px"
    }
  }
}
```

Example `themes/compact.json` (density):

```json
{
  "theme": {
    "space": {
      "xs": "3px",
      "sm": "6px",
      "md": "12px"
    }
  }
}
```

Example `themes/rounded.json` (shape):

```json
{
  "theme": {
    "radius": {
      "sm": "10px",
      "md": "16px",
      "lg": "24px"
    }
  }
}
```

### 2.2 Theme Merge Contract

To keep AI/codegen deterministic, define merge order and requirements:

```yaml
theming:
  customizable_token_groups: [color, space, radius]
  required_themes: [light, dark]
  supports_density: true
  supports_shape: true
  merge_order: [base, brand, mode, density, shape]
```

**Rules**
- Merging is shallow-by-default but supports deep-merge for nested keys (e.g. `theme.color.*`)
- Later theme layers override earlier layers according to `merge_order`
- A theme layer may partially specify values (e.g. compact only changes spacing)
- Components must remain theme-agnostic (only token references)

---

### 2.3 Responsive Behavior

**Scope:** Breakpoint definitions and adaptive layout rules
**Goal:** Deterministic, token-driven responsiveness

**Breakpoint Tokens**

```json
{
  "breakpoint": {
    "sm": { "value": "640px" },
    "md": { "value": "768px" },
    "lg": { "value": "1024px" },
    "xl": { "value": "1280px" }
  }
}
```

**Layout Responsiveness**

```yaml
kind: layout
name: Main
regions:
  sidebar:
    role: persistent
    component: SideNav
    responsive:
      hidden_below: md
      collapsed_at: [md, lg]
      expanded_at: xl

  content:
    role: dynamic
    responsive:
      padding:
        default: space.md
        below_md: space.sm
```

**Component Responsiveness**

```yaml
kind: component
name: Card
layout:
  direction:
    default: row
    below_sm: column
  padding:
    default: space.lg
    below_md: space.md
  gap:
    default: space.md
    below_sm: space.sm
```

**Rules**
- Breakpoints must be defined as tokens
- Components specify responsive overrides per breakpoint
- `below_*` and `above_*` modifiers reference breakpoint tokens
- Mobile-first: `default` applies to smallest, overrides scale up
- Avoid content hiding; prefer reflow and adaptation

---

## 3. Components

**Scope:** Reusable UI building blocks
**No routing, no application state**

**Naming:** Component names must be PascalCase (pattern: `^[A-Z][a-zA-Z0-9]*$`)

```yaml
kind: component
name: Button
category: control
description: Triggers an action

props:
  label: string
  icon: optional
  disabled: boolean

variants:
  intent: [primary, secondary, danger]
  size: [sm, md, lg]

states:
  - default
  - hover
  - disabled

layout:
  minHeight: token.size.touch-target
  padding: space.md
  borderRadius: radius.md
  background: color.accent

accessibility:
  role: button
  keyboard: true
```

**Component Categories**

Components should be organized by category for discoverability:

| Category | Purpose | Examples |
|----------|---------|----------|
| `primitive` | Basic building blocks | Text, Container, Grid, Image, Icon |
| `control` | Interactive elements | Button, Input, Select, Checkbox, Toggle |
| `form` | Form-specific components | FormField, FieldError, FormSection |
| `navigation` | Navigation elements | Link, Breadcrumb, NavItem, Dropdown |
| `layout` | Structural containers | Card, List, Divider, Accordion |
| `feedback` | User feedback | Alert, Toast, LoadingSpinner, EmptyState |
| `display` | Data presentation | Badge, Avatar, Tag, Stat |
| `media` | Rich content | VideoPlayer, ImageGallery, Carousel |
| `content` | Domain-specific | CourseCard, LessonItem, UserProfile |
| `auth` | Authentication | LoginForm, GoogleLoginButton |

**Rules**
- Components must be context-free
- Variants are finite and explicit
- Accessibility is mandatory
- Components must reference tokens (including `space.*` and `radius.*`)
- Use semantic token names (e.g., `token.size.touch-target`) for accessibility values

---

### 3.1 State-Driven Styling

**Scope:** How visual properties change per state
**Goal:** Deterministic mapping from state → token overrides

```yaml
kind: component
name: Button
variants:
  intent: [primary, secondary, danger]

tokens:
  primary:
    background: color.accent
    foreground: color.on-accent
  secondary:
    background: color.bg-subtle
    foreground: color.fg
  danger:
    background: color.danger
    foreground: color.on-danger

states:
  default:
    # inherits from variant tokens

  hover:
    transform:
      background: darken(10%)    # relative transform
    # OR explicit token:
    # background: color.accent-hover

  focus:
    outline:
      width: 2px
      color: color.focus-ring
      offset: 2px

  active:
    transform:
      background: darken(15%)
    scale: 0.98

  disabled:
    opacity: 0.5
    cursor: not-allowed
    # prevents hover/focus states

  loading:
    content: spinner
    pointer_events: none

state_precedence: [disabled, loading, focus, active, hover, default]
```

**Relative Transforms**

```yaml
transforms:
  darken:
    description: Reduce lightness by percentage
    bounds: [0, 100]
  lighten:
    description: Increase lightness by percentage
    bounds: [0, 100]
  alpha:
    description: Set opacity
    bounds: [0, 1]
  saturate:
    description: Adjust saturation by percentage
    bounds: [-100, 100]
```

**Rules**
- States override variant tokens, not replace entirely
- `state_precedence` defines which state wins on conflict
- `disabled` and `loading` are blocking states (suppress others)
- Transforms (darken, lighten, alpha) must be relative and bounded
- Focus state must always be visible for keyboard accessibility

---

### 3.2 Variant Matrix

**Scope:** Valid variant combinations and exclusions
**Goal:** Prevent invalid or untested states

```yaml
kind: component
name: Button
variants:
  intent: [primary, secondary, danger]
  size: [sm, md, lg]
  style: [solid, outline, ghost]

variant_matrix:
  mode: allowlist  # or 'blocklist' or 'all' (default)

  # Explicit valid combinations (when mode: allowlist)
  allow:
    - intent: danger
      style: [solid, outline]  # no ghost danger buttons
    - intent: primary
      style: solid
      size: lg                 # primary+lg only as solid

  # Explicit invalid combinations (when mode: blocklist)
  deny:
    - intent: secondary
      style: ghost             # too low contrast
    - size: sm
      style: outline           # border too prominent at small size

  # Default values when combination not specified
  fallback:
    style: solid

variant_defaults:
  intent: primary
  size: md
  style: solid
```

**Validation Output**

```yaml
# Generated validation report
variant_coverage:
  total_combinations: 27
  valid_combinations: 21
  invalid_combinations: 6
  coverage: 78%

  invalid:
    - { intent: danger, style: ghost, size: sm }
    - { intent: danger, style: ghost, size: md }
    - { intent: danger, style: ghost, size: lg }
    - { intent: secondary, style: ghost, size: sm }
    - { intent: secondary, style: ghost, size: md }
    - { intent: secondary, style: ghost, size: lg }
```

**Rules**
- Undefined combinations fall back to `fallback` values
- `deny` takes precedence over `allow`
- Code generators must validate against matrix
- Design tools should only offer valid combinations
- Each valid combination should have visual documentation

---

### 3.3 Primitive Components

**Scope:** Foundational components that other components build upon
**Goal:** Ensure consistent base elements across all implementations

Every DCF implementation should include these primitive components:

**Text** - Typography rendering with semantic variants

```yaml
kind: component
name: Text
category: primitive

props:
  value: string
  element: [p, span, h1, h2, h3, h4, h5, h6, label]
  variant: [body, body-sm, caption, heading-xs, heading-sm, heading-md, heading-lg, heading-xl]
  color: [default, muted, primary, error, success, warning]
  align: [start, center, end]
  truncate: boolean
  lineClamp: number
```

**Container** - Flexible layout wrapper

```yaml
kind: component
name: Container
category: primitive

props:
  direction: [row, column]
  align: [start, center, end, stretch, baseline]
  justify: [start, center, end, between, around, evenly]
  gap: space_token
  padding: space_token
  wrap: boolean
  condition: string  # Conditional rendering
```

**Icon** - Icon rendering with size variants

```yaml
kind: component
name: Icon
category: primitive

props:
  name: icon_ref
  size: [sm, md, lg]
  color: color_token

layout:
  size:
    sm: 16px
    md: 20px
    lg: 24px
```

**Image** - Responsive image with loading states

```yaml
kind: component
name: Image
category: primitive

props:
  src: string
  alt: string  # Required for accessibility
  object_fit: [contain, cover, fill]
  loading: [lazy, eager]
  fallback: string

states:
  loading:
    background: color.muted
  error:
    display: fallback
```

**Rules**
- Primitive components must be stateless
- Primitive components accept token references, not hardcoded values
- Screen definitions can use primitives directly in `content` arrays
- Higher-level components should compose primitives where possible

---

## 4. Layout Templates

**Scope:** Page grammar and region composition
**Defines where things may exist**

**Naming:** Layout names must be PascalCase (pattern: `^[A-Z][a-zA-Z0-9]*$`)

```yaml
kind: layout
name: Main
description: Standard application shell

regions:
  header:
    role: persistent
    component: AppHeader
  sidebar:
    role: persistent
    component: SideNav
  content:
    role: dynamic
```

```yaml
kind: layout
name: Detail
description: Focused content with back navigation

regions:
  header:
    component: BackHeader
  content:
    scroll: true
```

**Rules**
- Screens must choose exactly one layout
- Regions define allowed component placement
- Layouts are reusable and finite

---

## 5. Navigation Graph

**Scope:** Where users can go
**Modeled as a graph, not a tree**

```yaml
kind: navigation
name: AdminConsole

routes:
  dashboard:
    path: /
    layout: Main
    screen: DashboardPage

  users:
    path: /users
    layout: Main
    screen: UsersPage

  user_detail:
    path: /users/:id
    layout: Detail
    screen: UserDetailPage
    parent: users
```

```yaml
transitions:
  - from: users
    to: user_detail
    trigger: click
    source: UsersTable.row
```

**Rules**
- Every route maps to exactly one screen
- Transitions must be explicit
- Deep links are first-class

---

## 6. Screens

**Scope:** Intent + component composition
**This is where UI meaning lives**

**Naming:** Screen names must be PascalCase (pattern: `^[A-Z][a-zA-Z0-9]*$`)

```yaml
kind: screen
name: UsersPage
intent: browse
description: View and manage users

primary_actions:
  - label: Create User
    action: navigate(users_create)

content:
  - component: UsersTable
    props:
      sortable: true
      filterable: true
```

**Standard intents**
- `browse` - View a collection of items
- `view` - View a single item's details
- `edit` - Modify an existing item
- `create` - Create a new item
- `confirm` - Confirm a destructive or important action
- `error` - Display an error state
- `empty` - Display an empty state
- `loading` - Display a loading state

**Extended intents** (common patterns not covered by standard intents)
- `authenticate` - Login, signup, or identity verification screens
- `callback` - OAuth callbacks, webhooks, or redirect handlers
- `search` - Search interface with results
- `settings` - Configuration or preferences screens
- `dashboard` - Overview/summary screens with multiple data sources

Custom intents may be defined in `rules.yaml` under an `intents.custom` key.

**Rules**
- One dominant intent per screen
- Prefer composition over custom logic
- Screens may not define tokens or components

---

### 6.1 Data Contracts

**Scope:** Where screen data originates and how it flows
**Goal:** Decouple UI from data fetching implementation

**Data Sources**

```yaml
kind: screen
name: UsersPage
intent: browse

data:
  users:
    source: api
    endpoint: /api/users
    method: GET
    params:
      page: query.page
      limit: 20
    refresh: on_focus

  currentUser:
    source: context
    key: auth.user

content:
  - component: UsersTable
    props:
      data: $users.items
      loading: $users.loading
      error: $users.error
      onRetry: $users.refetch
```

**Source Types**

```yaml
sources:
  api:
    description: Remote HTTP endpoint
    properties: [endpoint, method, params, headers, refresh, cache]

  context:
    description: Global application state
    properties: [key]

  local:
    description: Screen-local state
    properties: [initial, persist]

  derived:
    description: Computed from other sources
    properties: [from, transform]

  static:
    description: Hardcoded values
    properties: [value]
```

**Data States**

```yaml
data_states:
  idle:
    description: Not yet fetched
    available_props: []

  loading:
    description: Fetch in progress
    available_props: [loading: true]

  success:
    description: Data available
    available_props: [data, loading: false]

  error:
    description: Fetch failed
    available_props: [error, loading: false, retry]

  stale:
    description: Cached but outdated
    available_props: [data, stale: true, refresh]
```

**Handling Stale Data in Screens**

```yaml
kind: screen
name: UsersPage
intent: browse

data:
  users:
    source: api
    endpoint: /api/users
    cache: 5m  # Cache for 5 minutes
    stale_while_revalidate: true

content:
  # Show stale indicator when data is outdated
  - component: Banner
    condition: $users.stale
    props:
      intent: info
      message: "Showing cached data"
      action:
        label: "Refresh"
        onClick: $users.refresh

  # Show data (works for both fresh and stale states)
  - component: UsersTable
    condition: $users.data
    props:
      data: $users.data
      # Visual hint for stale data
      stale_style:
        condition: $users.stale
        opacity: 0.7
```

**Derived Data Example**

```yaml
data:
  users:
    source: api
    endpoint: /api/users

  activeUsers:
    source: derived
    from: users
    transform: filter(item => item.status === 'active')

  userCount:
    source: derived
    from: users
    transform: count()
```

**Rules**
- Screens declare data dependencies explicitly
- Components receive data via props (no direct fetching)
- Loading/error states must be handled at screen level
- `$` prefix denotes data binding reference
- Transforms are pure functions, no side effects

---

## 7. Interaction Flows

**Scope:** Multi-step user journeys
**Declarative success/error paths**

**Naming:** Flow names must be PascalCase (pattern: `^[A-Z][a-zA-Z0-9]*$`)

```yaml
kind: flow
name: CreateUser
start: UsersPage

steps:
  - screen: UserFormPage
    on_submit:
      success: UserDetailPage
      error: UserFormPage
```

**Rules**
- Flows are linear unless branching is explicit
- No UI layout definitions here
- Used for generation, validation, and testing

---

### 7.1 Error Context in Flows

**Scope:** Passing error state through flow transitions
**Goal:** Enable contextual error recovery

```yaml
kind: flow
name: CreateUser
start: UsersPage

steps:
  - screen: UserFormPage
    on_submit:
      success:
        target: UserDetailPage
        params:
          id: $result.user.id
          toast: "User created successfully"

      error:
        target: UserFormPage
        context:
          error_type: $error.type
          error_message: $error.message
          error_fields: $error.fields
          submitted_data: $input

        field_errors:
          email: $error.fields.email
          name: $error.fields.name
```

**Error Types**

```yaml
error_types:
  validation:
    description: Client-side validation failed
    recoverable: true
    user_action: Fix highlighted fields

  conflict:
    description: Duplicate entry, version mismatch
    recoverable: true
    user_action: Resolve conflict or retry

  unauthorized:
    description: Auth required or insufficient permissions
    recoverable: false
    user_action: Log in or request access

  not_found:
    description: Resource does not exist
    recoverable: false
    user_action: Navigate elsewhere

  server:
    description: 5xx server errors
    recoverable: true
    user_action: Retry later

  network:
    description: Connection failed
    recoverable: true
    user_action: Check connection and retry
```

**Error Recovery Strategies**

```yaml
error_recovery:
  validation:
    action: highlight_fields
    persist_input: true
    focus: first_error_field

  conflict:
    action: show_modal
    modal: ConflictResolutionModal
    params:
      existing: $error.existing
      submitted: $input

  unauthorized:
    action: redirect
    target: LoginPage
    return_to: $current_screen

  not_found:
    action: redirect
    target: $parent_screen
    toast: $error.message

  server:
    action: show_toast
    toast:
      type: error
      message: "Something went wrong. Please try again."
      retry: true

  network:
    action: show_toast
    toast:
      type: error
      message: "Connection lost. Check your network."
      retry: true
```

**Screen Error Display**

```yaml
kind: screen
name: UserFormPage
intent: create

error_display:
  inline:
    component: FieldError
    position: below_field

  summary:
    component: ErrorBanner
    position: top
    show_when: $errors.length > 3

  toast:
    component: Toast
    types: [network, server]

data:
  form_errors:
    source: context
    key: flow.error_context.error_fields
```

**Rules**
- Errors must carry type, message, and optional field mapping
- Flows must define recovery strategy per error type
- Submitted data should persist through validation errors
- Network/server errors surface differently than validation
- `$error.*` bindings only available in error branch

---

## 8. Design Rules & Constraints

**Scope:** Global heuristics and guardrails
**Critical for AI correctness**

```yaml
rules:
  navigation:
    max_depth: 3
    confirm_on_destructive: true

  layout:
    one_primary_action_per_screen: true

  accessibility:
    min_touch_target: 44px
    focus_visible: true
```

**Rules**
- Rules are enforceable constraints
- Violations should be detectable
- Rules override defaults everywhere

---

## 9. File Structure (Recommended)

```text
design/
 ├─ tokens.base.json          # Design tokens (colors, spacing, typography, etc.)
 ├─ theming.yaml              # Theme merge contract and configuration
 ├─ themes/
 │   ├─ light.json            # Light mode colors
 │   ├─ dark.json             # Dark mode colors
 │   ├─ compact.json          # Compact density spacing
 │   └─ rounded.json          # Rounded shape variant
 ├─ components/
 │   ├─ primitives/           # Basic building blocks
 │   │   ├─ Text.yaml
 │   │   ├─ Container.yaml
 │   │   └─ Icon.yaml
 │   ├─ Button.yaml
 │   └─ Input.yaml
 ├─ layouts/
 │   ├─ Main.yaml
 │   └─ Detail.yaml
 ├─ navigation.yaml
 ├─ screens/
 │   ├─ UsersPage.yaml
 │   └─ UserDetailPage.yaml
 ├─ flows/
 │   └─ CreateUser.yaml
 ├─ i18n/
 │   ├─ en.yaml
 │   └─ strings.yaml
 └─ rules.yaml
```

**theming.yaml** should contain the theme merge contract from Section 2.2:

```yaml
theming:
  customizable_token_groups: [color, space, radius]
  required_themes: [light]
  optional_themes: [dark]
  supports_density: true
  supports_shape: true
  merge_order: [base, brand, mode, density, shape]
```

---

## 10. Internationalization

**Scope:** Multi-language, RTL, and locale-aware formatting
**Goal:** Design system must be locale-agnostic

**Required fields:** `dcf_version`, `locale`, `strings`

**Locale format:** Pattern `^[a-z]{2,3}(-[A-Za-z]{4})?(-[A-Z]{2})?$` supports:
- Two or three letter language codes: `en`, `de`, `yue`
- Optional script subtags: `zh-Hans`, `sr-Latn`
- Optional region codes: `en-US`, `zh-Hans-CN`, `sr-Latn-RS`

### 10.1 Layout Direction

```yaml
kind: layout
name: Main
direction: auto  # reads from document/locale

regions:
  sidebar:
    position: inline-start  # logical property (left in LTR, right in RTL)

  content:
    role: dynamic
```

**Physical vs Logical Properties**

```yaml
# Use logical, not physical properties
spacing:
  # Correct - adapts to RTL
  padding_inline_start: space.md
  padding_inline_end: space.sm
  margin_block_start: space.lg

  # Incorrect - breaks in RTL
  padding_left: space.md
  padding_right: space.sm
  margin_top: space.lg

alignment:
  text_align: start    # ✓ Correct - adapts to RTL
  # text_align: left   # ✗ Incorrect - hardcoded direction
```

### 10.2 String Externalization

```yaml
kind: screen
name: UsersPage

strings:
  title:
    key: users.page.title
    fallback: "Users"

  empty:
    key: users.empty_state
    fallback: "No users found"

  count:
    key: users.count
    fallback:
      one: "{count} user"
      other: "{count} users"

content:
  - component: PageHeader
    props:
      title: $strings.title

  - component: UserCount
    props:
      label: $strings.count
      count: $data.users.length
```

**String File Format**

```yaml
# i18n/en.yaml
users:
  page:
    title: "Users"
  empty_state: "No users found"
  count:
    one: "{count} user"
    other: "{count} users"
  actions:
    create: "Create User"
    delete: "Delete User"
    confirm_delete: "Are you sure you want to delete {name}?"
```

### 10.3 Locale-Aware Formatting

```yaml
format:
  date:
    short: { style: short }           # 12/19/2025 or 19/12/2025
    long: { style: long }             # December 19, 2025
    relative: { style: relative }     # 2 days ago

  number:
    decimal: { style: decimal }       # 1,234.56 or 1.234,56
    currency: { style: currency, currency: auto }
    percent: { style: percent }

  list:
    conjunction: { style: conjunction }  # A, B, and C
    disjunction: { style: disjunction }  # A, B, or C
```

**Usage in Screens**

```yaml
content:
  - component: Text
    props:
      value: $data.createdAt
      format: date.short

  - component: Price
    props:
      value: $data.price
      format: number.currency
```

### 10.4 RTL-Aware Icons

```yaml
icons:
  arrow-left:
    ltr: "lucide:arrow-left"
    rtl: "lucide:arrow-right"

  arrow-right:
    ltr: "lucide:arrow-right"
    rtl: "lucide:arrow-left"

  # Non-directional icons don't need variants
  check: "lucide:check"
  close: "lucide:x"
```

**Rules**
- All user-visible strings must be externalized
- Use logical properties (start/end) not physical (left/right)
- Layouts must declare `direction: auto` or explicit handling
- Number/date formatting must use locale-aware formatters
- Icons with directional meaning need RTL variants
- Pluralization must follow CLDR plural rules
- String interpolation uses `{variable}` syntax

---

## Summary

DCF separates UI concerns into orthogonal layers that are easy for both humans and AI systems to reason about, validate, generate from, and theme reliably.

**Core layers**
0. **Versioning** → compatibility (semver, profiles, capabilities)
1. **Tokens** → visual values (colors, spacing, motion, assets)
2. **Themes** → token overrides (composable, responsive)
3. **Components** → structure (states, variants, accessibility)
4. **Layouts** → grammar (regions, responsiveness)
5. **Navigation** → movement (routes, transitions)
6. **Screens** → intent (data contracts, composition)
7. **Flows** → behavior (error handling, recovery)
8. **Rules** → constraints (guardrails, validation)
9. **i18n** → localization (strings, formatting, RTL)

**Key additions in v1.0.0**
- File versioning with semantic versioning (all artifacts must declare `dcf_version`)
- Validation profiles (`lite`, `standard`, `strict`) for progressive adoption
- Capability declarations for partial adoption and focused validation
- Motion tokens with reduced-motion support
- Asset/icon token system with theme variants
- Responsive breakpoints and adaptive layouts
- State-driven styling with precedence rules
- Variant matrix for valid combinations
- Data contracts with source types and states
- Error context and recovery strategies in flows
- Full internationalization support (RTL, plurals, formatting)
- PascalCase naming enforcement for components, screens, flows, and layouts
- Expanded locale pattern supporting script subtags (e.g., `zh-Hans`, `sr-Latn-RS`)

## Complete Example

Here's a simple but complete DCF example that demonstrates all the core concepts working together. This example can serve as a reference implementation for AI agents and developers learning the DCF specification.

### Tokens (`tokens.json`)

```json
{
  "dcf_version": "1.0.0",
  "kind": "tokens",
  "profile": "standard",
  "color": {
    "primary": {
      "value": "#0066cc",
      "description": "Primary brand color"
    },
    "on-primary": {
      "value": "#ffffff",
      "description": "Text on primary backgrounds"
    },
    "background": {
      "value": "#ffffff",
      "description": "Default background color"
    },
    "text": {
      "value": "#333333",
      "description": "Default text color"
    }
  },
  "space": {
    "xs": {
      "value": "4px"
    },
    "sm": {
      "value": "8px"
    },
    "md": {
      "value": "16px"
    },
    "lg": {
      "value": "24px"
    }
  },
  "radius": {
    "md": {
      "value": "6px"
    }
  },
  "font": {
    "body": {
      "family": "Inter, system-ui, sans-serif",
      "size": "16px",
      "lineHeight": "1.5",
      "weight": "400"
    }
  }
}
```

### Theme (`theme.json`)

```json
{
  "dcf_version": "1.0.0",
  "kind": "theme",
  "theme": {
    "color": {
      "primary": "#0055aa",
      "background": "#f8f9fa",
      "text": "#212529"
    }
  }
}
```

### Button Component (`button.yaml`)

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard
name: Button
category: control
description: Interactive button for triggering actions

props:
  label: string
  variant: string
  size: string
  disabled: string

variants:
  variant: [primary, secondary]
  size: [sm, md, lg]

tokens:
  primary:
    background: color.primary
    foreground: color.on-primary
  secondary:
    background: transparent
    foreground: color.primary
    border: color.primary

states: []
state_precedence: [disabled, hover, default]

layout:
  padding: space.md
  borderRadius: radius.md
  fontFamily: font.body.family
  fontSize: font.body.size
  lineHeight: font.body.lineHeight

accessibility:
  role: button
  keyboard: true
```

### Input Component (`input.yaml`)

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard
name: Input
category: form
description: Text input field

props:
  label: string
  name: string
  type: string

layout:
  padding: space.sm
  borderRadius: radius.md
  borderWidth: 1px
  borderColor: color.text
  fontFamily: font.body.family
  fontSize: font.body.size

accessibility:
  role: textbox
  keyboard: true
```

### Text Component (`text.yaml`)

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard
name: Text
category: primitive
description: Text display component

props:
  value: string
  variant: string

variants:
  variant: [body, heading]

tokens:
  body:
    fontSize: font.body.size
    lineHeight: font.body.lineHeight
  heading:
    fontSize: "24px"
    fontWeight: "600"
    lineHeight: "1.2"

layout:
  fontFamily: font.body.family
  color: color.text

accessibility:
  role: none
  keyboard: false
```

### Header Component (`header.yaml`)

```yaml
dcf_version: "1.0.0"
kind: component
profile: standard
name: Header
category: layout
description: Application header component

props:
  title: string

layout:
  padding: space.md
  backgroundColor: color.primary
  color: color.on-primary

accessibility:
  role: banner
  keyboard: false
```

### Main Layout (`main-layout.yaml`)

```yaml
dcf_version: "1.0.0"
kind: layout
profile: standard
name: MainLayout
description: Main application layout with header, content and footer

regions:
  header:
    role: persistent
    component: Header
    position: block-start
  sidebar:
    role: persistent
    component: Sidebar
    position: inline-start
  content:
    role: dynamic
    component: Content
    position: inline-end
  footer:
    role: persistent
    component: Footer
    position: block-end
```

### Login Screen (`login-screen.yaml`)

```yaml
dcf_version: "1.0.0"
kind: screen
profile: standard
name: LoginPage
intent: authenticate
description: User authentication screen

data:
  credentials:
    source: static
    params:
      username: ""
      password: ""

strings:
  title:
    key: login.title
    fallback: "Login"
  username_label:
    key: login.username
    fallback: "Username"
  password_label:
    key: login.password
    fallback: "Password"
  submit_button:
    key: login.submit
    fallback: "Sign In"

primary_actions:
  - label: $strings.submit_button
    action: submit(credentials)

content:
  - component: Text
    props:
      value: $strings.title
  - component: Input
    props:
      label: $strings.username_label
      name: username
  - component: Input
    props:
      label: $strings.password_label
      type: password
      name: password
  - component: Button
    props:
      label: $strings.submit_button
      variant: primary
```

### Navigation (`navigation.yaml`)

```yaml
dcf_version: "1.0.0"
kind: navigation
profile: standard
name: AppNavigation

routes:
  login:
    path: /login
    screen: LoginPage
  dashboard:
    path: /dashboard
    screen: DashboardPage
  profile:
    path: /profile
    screen: ProfilePage

transitions:
  - from: login
    to: dashboard
    trigger: submit
  - from: dashboard
    to: profile
    trigger: click
```

This complete example demonstrates how all the core DCF concepts work together to create a cohesive design system. Each file represents a different layer of the system, and they all work together to define a complete UI application structure that can be validated, processed by AI tools, and used to generate code for various platforms.
