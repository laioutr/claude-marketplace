---
name: writing-section-block-definitions
description: Use when authoring or modifying a `defineSection` / `defineBlock` schema in a Laioutr module — choosing sidebar group placement (Section info / Content / Design → Styling / Design → Layout / Rules), picking canonical field names (`heading`, `media`, `cta`, `background`, `margin`, `alignment`, …), avoiding forbidden bare field names (`style`, `class`, `name`, `key`, `id`, `slot`, `is`, `default`, `ref`), adding conditional visibility with `if`, naming a `Block*.vue` component, writing a factory function in `shared-fields/*.ts` that returns a field, or exporting a `*Options` array via `defineSelectOptions`.
---

# Writing Section & Block Definitions

## Overview

Sections and blocks are configured via `defineSection()` / `defineBlock()` schemas (helpers exported from `@laioutr-core/ui-app`). Conventions cover four orthogonal concerns:

1. **Structural placement** — which sidebar group a field lives in (fixed order, top to bottom)
2. **Canonical names** — one concept → one field `name` across the module
3. **Conditional fields** — `if` for visibility, with a typed operator set
4. **Shared-field reuse** — factories and option lists in `shared-fields/*.ts`, with literal-type discipline

The deep-detail references live in [`references/`](./references/). The body below is the orientation — open the right reference when its trigger matches your task.

## Quick reference

### Sidebar group order (top to bottom, never reordered)

1. **Section info & settings** — Studio-managed (`studio.label`, `studio.description`, `studio.previewSrc`, tags). Not part of `schema`.
2. **Content** — editorial content authored per instance.
3. **Design → Styling** — visual appearance (variant, background, color, overlay).
4. **Design → Layout** — spacing, columns, breakpoints, alignment.
5. **Rules** — data binding + visibility conditions.

Omit a group if the component has no options in it; never reorder.

### Canonical field names (selected)

| Concept | `name` | `type` | Group |
| --- | --- | --- | --- |
| Caption | `caption` | `text` | Content |
| Heading | `heading` | `text` | Content |
| Subline | `subline` | `text` | Content |
| Description | `description` | `textarea` / `richtext` | Content |
| Media | `media` | `media` | Content |
| CTA | `cta` | `object` (use shared `button.ts`) | Content |
| Query | `query` | `query` | Content |
| Style variant | `variant` | `toggle_button` / `select` | Styling |
| Background preset | `background` | `select` (`none` / `pale` / `solid` / `default`) | Styling |
| Foreground color | `color` | `color` | Styling |
| Overlay | `overlay` | `object` (use shared `overlay.ts`) | Styling |
| Margin / Padding | `margin` / `padding` | `select` (`none` / `s` / `m` / `l`) | Layout |
| Columns / Rows | `columns` / `rows` | `number` / `select` | Layout |
| Alignment (1D / 2D) | `alignment` | `toggle_button` / `content_alignment` | Layout |

Full table + Rules group + Design → Layout breakpoints in [`references/section-config-standard.md`](./references/section-config-standard.md).

### Forbidden bare field names

`style`, `class`, `name`, `key`, `ref`, `id`, `slot`, `is`, `default` — most break Vue-template binding or shadow framework primitives. Compound names like `accordionStyle` (with `as: 'style'`) are allowed; **bare** names are not. Per-name failure modes in [`references/forbidden-field-names.md`](./references/forbidden-field-names.md).

### Conditional visibility (`if`)

Hide a field based on another field's value. Expressions are lisp-style JSON arrays (`['op', ...args]`), with `['get', 'fieldName']` reading a sibling schema value:

```ts
fields: [
  { name: 'showHeading', type: 'checkbox', default: true },
  {
    name: 'heading',
    type: 'text',
    if: ['get', 'showHeading'], // bare get on a checkbox returns the boolean directly
  },
],
```

Full operator list (`get`, `and` / `or` / `not`, `eq` / `ne` / comparisons, `in` / `arr-of`, …), argument-order quirks (haystack-first for `in`), the `[[…]]` literal-array escape, and per-field-type fallback behavior in [`references/schema-field-if.md`](./references/schema-field-if.md). The object-shape `{ field, eq }` form does **not** exist — every `if` clause is a JSON expression array.

### Block component naming

Components rendered as blocks must be named `Block*.vue` (e.g. `BlockProductCard.vue`, `BlockHero.vue`). Resolution rules in [`references/block-naming.md`](./references/block-naming.md).

### Shared-fields

Live in `src/runtime/app/shared-fields/*.ts`. Two shapes:

**Static field objects** — just export the field.

**Factory functions** — return a field. Inside `defineSection`'s `fields: [...]` array, contextual typing widens the field's `name` back to `string` unless the factory uses a **single `const Opts extends {...}` generic** with the resolved name derived from `Opts` at the return type. The older multi-generic-with-template-literal-default pattern (`<const ForKey extends string, const Name extends string = \`${ForKey}Visible\`>`) is broken under contextual typing — see [`references/shared-field-factories.md`](./references/shared-field-factories.md).

For exported `*Options` arrays, use `defineSelectOptions(...)` — it preserves literal types AND enforces the non-empty tuple shape. See [`references/shared-field-options.md`](./references/shared-field-options.md).

## Common mistakes

| Mistake | Fix |
| --- | --- |
| Field named `style` to hold the variant selector | Rename to `variant`; the bare `style` is forbidden |
| `containerStyle`, `accordionStyle` (or any `<component>Style`) as the top-level variant selector | Rename to `variant`; compound `*Style` names are reserved for the `as: 'style'` decorator pattern |
| `backgroundColor`, `customBackground` (used as variant) | Rename to `background` |
| `blockMargin` / `innerBlockPadding` | Rename to `margin` / `padding` |
| `required: true` on a schema field | Schema fields have no `required` property — defaults provide the fallback |
| Generic factory in `defineSection.fields` and `name` widens to `string` | Collapse to a single `const Opts extends {...}` generic and derive the resolved `name` from `Opts` at the return type (see `shared-field-factories.md`); or extract the call outside the array context |
| `*Options` typed via `as const satisfies [Option, ...]` | Replace with `defineSelectOptions(...)` |
| Block component named `ProductCard` (missing prefix) | Rename to `BlockProductCard` |

## Reference index

- [`references/section-config-standard.md`](./references/section-config-standard.md) — complete sidebar group + canonical-name + preset specification
- [`references/forbidden-field-names.md`](./references/forbidden-field-names.md) — full forbidden-name list with failure modes
- [`references/schema-field-if.md`](./references/schema-field-if.md) — `if` operator reference + per-type fallback behavior
- [`references/block-naming.md`](./references/block-naming.md) — `Block*.vue` naming + component-resolution rules
- [`references/shared-field-factories.md`](./references/shared-field-factories.md) — factory function literal-type discipline
- [`references/shared-field-options.md`](./references/shared-field-options.md) — `defineSelectOptions` and `*Options` arrays

For the broader architectural context (when to create a custom section vs. fork upstream), see the [`three-layer-architecture.md`](../laioutr-platform/references/three-layer-architecture.md) in the `laioutr-platform` skill.
