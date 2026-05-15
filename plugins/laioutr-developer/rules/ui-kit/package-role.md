# ui-kit — Package Role & Boundaries

This rule defines what `@laioutr-core/ui-kit` is for and how it relates to `ui` and `ui-app`. Every component change in this package must respect these boundaries.

## The three-layer architecture

```
ui-kit  →  ui  →  ui-app
  │        │        │
  │        │        └─ Customer-facing Studio config. Sections & Blocks.
  │        │           Dynamic configurability for Laioutr Studio. No stories.
  │        │
  │        └─ Composed, domain-aware components. The building blocks that
  │           ui-app wraps into Sections and Blocks. Stories required.
  │
  └─ Design system primitives (atoms). Buttons, inputs, layout helpers.
     Pure presentation, no domain knowledge. Stories required.
```

## ui-kit is the foundation

**Purpose:** `ui-kit` **is the Laioutr design system**. Everything visual in the platform is composed from the primitives that live here.

- ui-kit is consumed by `ui`. Components in `ui` are assembled from ui-kit atoms.
- ui-kit never imports from `ui` or `ui-app`. It is the bottom layer.
- ui-kit has no awareness of commerce domain, CMS, or Studio. It is pure design.

## Atom requirements

Because ui-kit is the base every layer above depends on, primitives must be **generic, stable, and visually authoritative**.

Non-negotiable:

1. **No domain knowledge** — no products, no articles, no CMS, no Studio. An atom that knows about "price" or "cart" belongs in `ui`, not here.
2. **Clean, typed Props interface** — exported from `<script lang="ts">`, named `<ComponentName>Props`. No implicit shapes.
3. **Token-only styling** — never hardcode colors, spacing, sizing, or radii. Everything goes through `@laioutr-core/ui-tokens` CSS variables. Animation timings are the only allowed literals.
4. **Every state styled** — default, hovered, pressed, focus, disabled. Interactive atoms must cover all of them.
5. **Prefer reka-ui as basis** — wrap reka-ui primitives for accessibility, keyboard nav, and ARIA. Style via data attributes (`[data-state]`, `[data-highlighted]`, `[data-disabled]`).
6. **Theme-agnostic** — must work across all five themes (classic, laioutr, tech, sunny, strawberry-field). Dark mode is handled automatically via tokens, no explicit dark styles.
7. **Stories required** — `<ComponentName>.stories.ts` under `UI Kit/<Category>/<Name>`. Every primary layout and variant from Figma must have a story.

## Why ui-kit has stories

`ui-kit` is the **canonical visual reference** of the design system. Storybook is where:

- Figma designs are validated against implementation.
- All variants and states are visible in isolation.
- Consumers (ui, ui-app, Studio) discover what primitives exist.

A primitive without a story is invisible to the rest of the organization. Visual regression and design review happen here.

## Decision checklist before adding or changing a ui-kit component

- [ ] Is this generic enough to be reused in many domain contexts (commerce, editorial, Studio)?
- [ ] Does the Props interface expose every meaningful variant and state?
- [ ] Is all styling token-based, with no hardcoded values?
- [ ] Are hover/focus/pressed/disabled states covered?
- [ ] Does it wrap a reka-ui primitive where one exists?
- [ ] Does it work across all five themes without theme-specific branches?
- [ ] Does it have a Storybook story under the correct Atomic Design folder?
