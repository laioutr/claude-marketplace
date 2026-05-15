# ui-kit First

Before writing new markup inside a `ui` component, search `ui-kit` for an existing primitive or molecule that already covers the pattern. The architectural rule "compose ui-kit primitives" (see `package-role.md`) is a passive check — this rule defines the active workflow step.

## When to run the check

Any time you are about to author more than ~3 lines of structural markup inside a `ui` component, or introduce a new repeating pattern.

## How

```
Glob node_modules/@laioutr-core/ui-kit/**/components/*/
```

Or import directly: any exported component from `@laioutr-core/ui-kit` is in scope. Scan folder names first. If a match is plausible, read the component's `types.ts` / `.vue` to confirm the prop surface fits.

## Common miss-patterns (always check ui-kit first)

- **Caption + heading + subline trio** → `TextGroup`
- **Button groups / CTA rows** → look for existing button-group composition before wrapping `<Button>` in a flex div
- **Input + label + helper text** → form-field primitives exist; don't handcraft the triple
- **Icon + text inline pairs** → check for icon-text primitives before composing inline
- **Card-shaped containers** (media + text + CTA) → check Card variants before building yet another wrapper
- **Backdrop / section chrome** (background, padding, container width) → `Backdrop` in ui-kit

## If nothing fits

Two options, in order:

1. **Extend the existing ui-kit component** if the gap is a missing prop/variant on something that clearly belongs there.
2. **Add a new ui-kit primitive/molecule** if the pattern is reusable across more than one `ui` component.

Only fall back to inline markup in the `ui` component when the composition is genuinely one-off and domain-specific.

## Why this rule exists

The TextGroup migration (April 2026) touched ~10 `ui` components that had each independently reinvented the caption/heading/subline trio. The molecule could have existed from day one; instead every component grew its own flex div + scoped color CSS + alignment overrides. The passive "compose primitives" rule did not prevent this — an active search step does.
