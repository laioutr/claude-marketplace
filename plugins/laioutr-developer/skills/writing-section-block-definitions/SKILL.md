---
name: writing-section-block-definitions
description: Use when authoring or modifying a `defineSection` / `defineBlock` schema in a Laioutr module — choosing sidebar group placement (Section info / Content / Design → Styling / Design → Layout / Rules), picking canonical field names (`heading`, `media`, `cta`, `background`, `margin`, `alignment`, …), avoiding forbidden bare field names (`style`, `class`, `name`, `key`, `id`, `slot`, `is`, `default`, `ref`), adding conditional visibility with `if`, naming a `Block*.vue` component, writing a factory function in `shared-fields/*.ts` that returns a field, exporting a `*Options` array via `defineSelectOptions`, or consuming a schema-defaulted field in the matching Section/Block component (typing the prop, avoiding `??` fallbacks / `withDefaults` / optional `?` / runtime narrowing for impossible values).
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

### Trust the schema when consuming fields

A schema field with a `default` is guaranteed by frontend-core to reach the component already populated, picker-shaped fields (`select`, `toggle_button`, `content_alignment`, `select_radio`, `checkbox`, …) additionally guarantee the value is one of the declared options, and `object` / `array` fields are always materialized — `{}` with sub-fields recursively populated, or `[]` — never `null` or `undefined`, with or without a `default`. Type the prop as the exact literal union, the exact object shape, or `T[]`, and use it directly — **no `?? 'fallback'`, no `?? {}`, no `?? []`, no optional chaining on the prop, no `withDefaults`, no optional `?` on the prop, no `v-if` guarding the prop's existence, no `safeX` computed with `includes()`-narrowing**. Defensive code for impossible states breaks exhaustive checking and hides the schema contract. Forbidden patterns, the "value really can be absent" edge cases (text/media/link without `default`), and red flags in [`references/trust-the-schema-in-components.md`](./references/trust-the-schema-in-components.md).

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
| `props.size ?? 'm'` / `safeVariant` computed / `withDefaults` re-asserting a schema `default` | Drop the defensive code; type the prop as the exact union and use it directly (see `trust-the-schema-in-components.md`) |
| Optional `?` on a prop whose schema field has a `default` | Drop the `?` — frontend-core guarantees presence |
| `props.items ?? []` / `props.items?.map(...)` / `v-if="props.items"` on an `array` field | Drop it — arrays are always materialized as `[]`; use `.length` if you need an empty-state branch (see `trust-the-schema-in-components.md`) |
| `props.style ?? {}` / `props.style?.foo` / optional `?` on an `object` prop | Drop it — objects are always materialized with sub-fields recursively populated; guard the inner leaves, not the object prop |

## Reference index

- [`references/section-config-standard.md`](./references/section-config-standard.md) — complete sidebar group + canonical-name + preset specification
- [`references/forbidden-field-names.md`](./references/forbidden-field-names.md) — full forbidden-name list with failure modes
- [`references/schema-field-if.md`](./references/schema-field-if.md) — `if` operator reference + per-type fallback behavior
- [`references/block-naming.md`](./references/block-naming.md) — `Block*.vue` naming + component-resolution rules
- [`references/shared-field-factories.md`](./references/shared-field-factories.md) — factory function literal-type discipline
- [`references/shared-field-options.md`](./references/shared-field-options.md) — `defineSelectOptions` and `*Options` arrays
- [`references/trust-the-schema-in-components.md`](./references/trust-the-schema-in-components.md) — no defensive fallbacks for schema-defaulted picker fields, or for `object` / `array` fields (always materialized)

For the broader architectural context (when to create a custom section vs. fork upstream), see the [`three-layer-architecture.md`](../laioutr-platform/references/three-layer-architecture.md) in the `laioutr-platform` skill.
