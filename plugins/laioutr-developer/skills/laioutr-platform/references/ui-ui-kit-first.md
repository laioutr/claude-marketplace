# ui-kit First

Before writing new markup inside a custom organism in your module, search `@laioutr-core/ui-kit` for an existing primitive or molecule that already covers the pattern. The architectural principle "compose ui-kit primitives" (see [`three-layer-architecture.md`](./three-layer-architecture.md)) is a passive check — this rule defines the active workflow step.

## When to run the check

Any time you are about to author more than ~3 lines of structural markup inside your own organism-style component, or introduce a new repeating pattern.

## How

Three discovery surfaces, each for a different question:

| You need to know | Use |
| --- | --- |
| Props, emits, slots, usage, code samples | The official docs MCP server (`laioutr-docs`). Falls back to context7 if unreachable. |
| What visual states / variants a component supports | <https://storybook.laioutr.cloud/> |
| The actual implementation (e.g. before forking) | <https://github.com/laioutr/ui-source> |

Any exported component from `@laioutr-core/ui-kit` is in scope — import it directly when a match is plausible.

## Common miss-patterns (always check ui-kit first)

- **Caption + heading + subline trio** → `TextGroup`
- **Button groups / CTA rows** → look for an existing button-group composition before wrapping `<Button>` in a flex div
- **Input + label + helper text** → form-field primitives exist; don't handcraft the triple
- **Icon + text inline pairs** → check for icon-text primitives before composing inline
- **Card-shaped containers** (media + text + CTA) → check Card variants before building yet another wrapper
- **Backdrop / section chrome** (background, padding, container width) → `Backdrop` in ui-kit

## If nothing fits

Resolve in this order — see [`three-layer-architecture.md`](./three-layer-architecture.md) for the full ladder:

1. **Prop or minor CSS override** on the existing ui-kit / ui component when a single prop / style change suffices.
2. **Build a local custom component in your module** when the gap is specific to your app's domain.
3. **Fork the upstream component** from <https://github.com/laioutr/ui-source> when it is almost right and you need to adjust internals. Copy the file locally and modify in your module. You take on the maintenance.
4. **Open an upstream issue** when the gap is generic and would benefit other apps. Improving upstream beats a long-lived fork.

Only fall back to inline markup in your organism when the composition is genuinely one-off and domain-specific.

## Why this rule exists

It is easy to independently reinvent a primitive that already exists — caption/heading/subline trios, button rows, card chrome, backdrop containers. Each reinvention grows its own flex div, scoped color CSS, and alignment overrides. The passive "compose primitives" principle does not prevent this on its own — an active search step does.
