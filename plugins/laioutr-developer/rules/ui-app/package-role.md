# ui-app — Package Role & Boundaries

This rule defines what `@laioutr-app/ui` is for and how it relates to `ui` and `ui-kit`. Every Section/Block change in this package must respect these boundaries.

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

## ui-app is the customer-facing layer

**Purpose:** `ui-app` is where **Laioutr Studio meets the design system**. Every Section and Block defined here is a configurable unit an editor can place, wire up, and parameterize in Studio.

- ui-app composes components from `ui` (and, indirectly via ui, from `ui-kit`).
- ui-app adds **no new visuals**. Its only job is Studio configurability and data binding.
- ui-app is never imported back into `ui` or `ui-kit`.

## Section & Block requirements

Because ui-app is the surface customers interact with through Studio, Sections and Blocks must be **thin configuration adapters** over ui components.

Non-negotiable:

1. **No new visual layouts** — if a Section/Block needs a look that ui doesn't yet provide, add or extend the component in `ui` first. Never introduce layout or styling here.
2. **Schema is the contract** — the `defineSection` / `defineBlock` schema is the Studio surface. Every configurable aspect must be a schema field, wired via `definitionToProps` to a prop on the ui component.
3. **Wrap, don't reimplement** — the template should render the underlying ui component and forward props/slots. If template logic grows beyond binding + data resolution, push it down into `ui`.
4. **Runtime data binding** — query fields, link resolution, entity components, and CMS wiring live here. Keep ui components pure and push domain resolution into ui-app.
5. **Respect Studio fallbacks** — rely on the runtime fallback table (see `section-config-standard.md`). Don't add ad-hoc `v-if` guards for fields that already have type-appropriate fallbacks.
6. **Follow section-config-standard** — top-level panels are Content + Design only, with shared-field presets for background/margin/padding/visibility/button/etc.

## Why ui-app has no stories

Storybook documents **visual primitives and their variants**. ui-app introduces neither:

- All visuals come from `ui` (and its Storybook covers them).
- All configurability is expressed through Studio schemas, not visual variants.
- There is nothing to show in Storybook that isn't already covered one layer down.

Correctness in ui-app is verified in **Studio itself** — does the schema render the right controls, does the configuration flow to the right prop, does the data resolve. Visual regression belongs in `ui`'s Storybook.

If you feel the urge to add a story here, the work almost certainly belongs in `ui`.

## Decision checklist before adding or changing a Section or Block

- [ ] Does the corresponding ui component already exist? If not, build it in `ui` first.
- [ ] Is every configurable aspect expressed as a schema field?
- [ ] Does the template only bind props/slots — no layout, no styling?
- [ ] Are runtime fallbacks relied upon instead of ad-hoc guards?
- [ ] Does the schema follow section-config-standard (Content + Design panels, shared-field presets)?
- [ ] For non-standalone blocks: is there a hosting Section that lists the block in a slot's `allow`?
