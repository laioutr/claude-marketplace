# Token Mapping

Map Figma variable names to CSS custom properties. Used in Phase 3 of `figma-design-analysis`. Covers the conversion rules, the color-flattening pattern, hex validation against installed token files, verifying tokens against source code, and the direct-CSS-var-vs-UnoCSS-utility abstraction layer distinction.

## Conversion Rules

Figma uses `/` separators and mixed case. CSS variables use `-` separators and lowercase.

| Figma Variable | CSS Variable | Rule |
|---|---|---|
| `Spacing/SM` | `var(--spacing-sm)` | Category/Size -> lowercase, `/` -> `-` |
| `Spacing/2XL` | `var(--spacing-2-xl)` | Numbers get `-` separator |
| `font size/M` | `var(--font-size-m)` | Space in category -> `-` |
| `font size/4XL` | `var(--font-size-4-xl)` | Same number rule |
| `font/weight/600` | `var(--font-weight-600)` | Numeric values stay as-is |
| `font/family/heading` | `var(--font-family-heading)` | Direct mapping |
| `colors/primary/9` | `var(--primary-9)` | `colors/` prefix is DROPPED |
| `colors/grey/12` | `var(--grey-12)` | Same drop rule |
| `greys/stone_grey/5` | `var(--stone-grey-5)` | `greys/` dropped, `_` -> `-` |
| `overlays/white_alpha/7` | `var(--overlay-white-alpha-7)` | `overlays/` -> `overlay-`, `_` -> `-` |
| `buttons/primary/default/bg-color` | `var(--buttons-primary-default-bg-color)` | Component tokens: direct `/` -> `-` |
| `buttons/border-radius` | `var(--buttons-border-radius)` | Direct mapping |
| `forms-border-radius` | `var(--forms-border-radius)` | Already CSS-compatible |
| `border radius/SM` | `var(--border-radius-sm)` | Space -> `-`, lowercase |
| `text/primary` | `var(--text-primary)` | Semantic tokens: direct |
| `text/always-white` | `var(--text-always-white)` | Direct mapping |

## Key Pattern: Color Flattening

The token build pipeline flattens color group hierarchies:

```
Figma path          -> CSS variable
colors.primary.9    -> --primary-9         (colors/ prefix dropped)
greys.stone_grey.5  -> --stone-grey-5      (greys/ prefix dropped, _ -> -)
overlays.white_alpha.7 -> --overlay-white-alpha-7 (overlays/ -> overlay-)
```

## Validating Hex Colors

When Figma provides a hex color (e.g., `#743dcd`), check if it matches an existing token. Search the installed `@laioutr-core/ui-tokens` source:

```bash
# Search base tokens (case-insensitive)
grep -ri "743dcd" node_modules/@laioutr-core/ui-tokens/src/figma/

# If not in base tokens, check theme-specific tokens
grep -ri "743dcd" node_modules/@laioutr-core/ui-tokens/src/figma/theme.*.tokens.json
```

(Path layout assumes npm/yarn; pnpm users adjust to `node_modules/.pnpm/.../@laioutr-core/ui-tokens/...`.)

1. If found, trace the JSON path to derive the CSS variable name using the conversion rules above
2. If NOT found in any token file, flag it to the user -- hardcoded colors are almost always a mistake

## Verifying Tokens Against Source Code

When analyzing an **existing** component, verify your Figma-derived token mapping against the actual source.

```bash
# Extract all CSS variable references from a component
grep -oP 'var\(--[\w-]+\)' path/to/ComponentName.vue | sort -u

# Also check for v-bind() usage (Vue dynamic CSS bindings)
grep -oP 'v-bind\(\w+\)' path/to/ComponentName.vue
```

Compare the source token list against your Figma mapping. Discrepancies indicate either:
- **Tokens in Figma but not in code**: may be handled by parent/child components or global styles
- **Tokens in code but not in Figma**: may be hardcoded fallbacks or implementation-specific additions
- **Hardcoded pixel values in code**: should be flagged

## Distinguishing Token Abstraction Layers

Spacing tokens can be applied via **direct CSS variable references** (`var(--spacing-l)`) or via **UnoCSS utility classes** (`py-l`, `s:py-xl`, `sm:py-2xl`). When auditing, check both:

```bash
# Direct CSS vars
grep -oP 'var\(--[\w-]+\)' ComponentName.vue | sort -u
# UnoCSS utilities in CVA or template classes
grep -oP '(py|px|ps|pe|p|m|gap)-[\w-]+' componentClasses.ts | sort -u
```

Document which layer handles which tokens. If the same spacing concern is split across both layers, flag it as a maintenance risk.
