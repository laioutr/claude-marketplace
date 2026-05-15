# ui — Package Role & Boundaries

This rule defines what `@laioutr-core/ui` is for and how it relates to `ui-kit` and `ui-app`. Every component change in this package must respect these boundaries.

## The three-layer architecture

```
ui-kit  →  ui  →  ui-app
  │        │        │
  │        │        └─ Customer-facing Studio config. Sections & Blocks.
  │        │           Dynamic configurability for Laioutr Studio. No stories.
  │        │
  │        └─ Composed, domain-aware components. The building blocks that
  │           ui-app wraps into Sections and Blocks. Must have stable props,
  │           clean interfaces, high variability. Stories required.
  │
  └─ Design system primitives (atoms). Buttons, inputs, layout helpers.
     Pure presentation, no domain knowledge. Stories required.
```

## ui lives in the middle

**Purpose:** `ui` contains the components that serve as the **foundation for Sections and Blocks** that live in `ui-app`.

- ui-kit primitives compose into ui components.
- ui components are wrapped by ui-app Sections/Blocks.
- ui-app is never imported back into ui.

## Interface requirements

Because ui-app is the **customer-facing Studio layer**, every ui component must be designed so a Section/Block in ui-app can expose its configurability through Studio without internal refactors.

Non-negotiable:

1. **Clean, typed Props interface** — exported from `<script lang="ts">`, named `<ComponentName>Props`. No implicit shapes.
2. **Variability over assumption** — slots, configurable alignment, sizes, counts, layout modifiers. Bake no layout decisions that a customer might want to change.
3. **No hard-coded domain content** — headings, labels, copy come in via props. A component that hardcodes a title cannot be wrapped as a configurable Section.
4. **No internal-only branching** — every meaningful branch must be a prop. If a Section needs to toggle something, it must already be a prop on the ui component.
5. **Composition over inheritance** — expose sub-slots (`#heading`, `#footer`, `#media`) so ui-app can inject Block-level content where needed.

## Why ui-app has no stories

`ui-app` adds **no new visuals**. Its job is purely to:

- Bind Studio configuration schemas (`defineSection`, `definitionToProps`) to props on ui components.
- Wire up dynamic data (CMS, commerce backends) into the static structure ui provides.

Because ui-app never introduces new visual layouts, there is nothing to show in Storybook beyond what ui already covers. **Visual regression and design review happen in ui's Storybook.** ui-app's correctness is verified through Studio itself.

## Decision checklist before adding or changing a ui component

- [ ] Is this component reusable as the basis for at least one Section or Block?
- [ ] Does the Props interface expose every layout/content decision that Studio might need to configure?
- [ ] Are all texts, images, and links injected via props or slots?
- [ ] Is the component composed from ui-kit primitives — not reimplementing atoms?
- [ ] Does it have a Storybook story covering the primary Figma layout and relevant variants?
