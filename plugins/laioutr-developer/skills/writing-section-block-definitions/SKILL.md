---
name: writing-section-block-definitions
description: Use when authoring or modifying a `defineSection` / `defineBlock` schema in a Laioutr module — choosing sidebar group placement (Section info / Content / Design → Styling / Design → Layout / Rules), picking canonical field names (`heading`, `media`, `cta`, `background`, `margin`, `alignment`, …), avoiding reserved bare field names (`style`, `class`, `key`, `ref`, `is`, `slot`, `slots`, `refFor`, `refKey`), adding conditional visibility with `if`, naming a `Block*.vue` component, writing a shared field or a factory function in `shared-fields/*.ts`, exporting a `*Options` array via `defineSelectOptions`, or consuming schema fields in the matching Section/Block component (`definitionToProps`, which field types can arrive `undefined`, avoiding `??` fallbacks / `withDefaults` / runtime narrowing for impossible values).
---

# Writing Section & Block Definitions

## Overview

Sections and blocks are configured via `defineSection()` / `defineBlock()` schemas. Both helpers, and `definitionToProps()`, are `@laioutr-core/frontend-core` auto-imports: import them from `#imports`. Each `.vue` file exports its `definition` from a plain `<script lang="ts">` block and types its props with `defineProps(definitionToProps(definition))` in `<script setup>`. `defineSection` requires `slots` (`slots: []` when the section takes no blocks); blocks have none. Sections live in `src/runtime/app/sections/` and blocks in `src/runtime/app/blocks/`, both registered through `registerLaioutrApp`. Conventions cover four orthogonal concerns:

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

Omit a group if the component has no options in it; never reorder. In the schema these are two fieldsets, `Content` and `Design`; inside `Design`, `info` fields act as the Styling / Layout sub-headings:

```ts
schema: [
  { label: 'Content', fields: [/* … */] },
  {
    label: 'Design',
    fields: [
      { type: 'info', name: 'stylingDivider', label: 'Styling' },
      /* variant, background, color, overlay … */
      { type: 'info', name: 'layoutDivider', label: 'Layout' },
      /* margin, padding, alignment, sizing … */
    ],
  },
],
```

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

`style`, `class`, `key`, `ref`, `is`, `slot`, `slots` (sections), `refFor`, `refKey` — Vue or Frontend Core consumes the value, so it never reaches `props`, and nothing errors. Compound names like `accordionStyle` (with `as: 'style'`) are allowed; **bare** names are not. Per-name failure modes in [`references/forbidden-field-names.md`](./references/forbidden-field-names.md).

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

### Trust the field-type contract when consuming fields

Type props with `defineProps(definitionToProps(definition))` — never by hand. Frontend Core fills every unset prop with its **field type's** fallback, not with the field's `default` (that only pre-fills Studio when an editor adds the section):

- **Always present** — `text` / `textarea` / `richtext` (`''`), `checkbox` (`false`; `true` for a visibility decorator), `select` / `radio` / `toggle_button` (first option; off-list values are clamped to it), `content_alignment` (`'center-center'`), `object` (sub-fields filled recursively), `array` (`[]`). Use them directly: **no `??`, no optional chaining, no `withDefaults`, no `safeX` computed with `includes()`-narrowing**.
- **Can be absent** — `number`, `icon`, `color`, `media`, `link`, `query` arrive `undefined` and `json` arrives `null`, even with a `default`. Guard them in templates (`v-if="media"`, `v-if="link"`), and give a value that drives layout a fallback (`props.columns ?? 3`).

A picker's runtime fallback is its first option, not its `default`: a picker you add to a Section already placed on pages gives those pages the first option. Full table, examples and red flags in [`references/trust-the-schema-in-components.md`](./references/trust-the-schema-in-components.md).

### Shared-fields

Live in `src/runtime/app/shared-fields/*.ts`. Two shapes:

**Static field objects** — wrap them in a local identity helper, `defineField = <const T extends StudioFieldDefinition>(d: T): T => d` with `StudioFieldDefinition` from `@laioutr-core/core-types/fields` (or write `as const satisfies StudioFieldDefinition`). `satisfies` alone, or a `: StudioFieldDefinition` annotation, widens `name` and option values to `string`, and a widened `name` collapses the whole component's props into a `string` index signature.

**Factory functions** — return a field. Inside `defineSection`'s `fields: [...]` array, contextual typing widens the field's `name` back to `string` unless the factory uses a **single `const Opts extends {...}` generic** with the resolved name derived from `Opts` at the return type. The older multi-generic-with-template-literal-default pattern (`<const ForKey extends string, const Name extends string = \`${ForKey}Visible\`>`) is broken under contextual typing — see [`references/shared-field-factories.md`](./references/shared-field-factories.md).

For exported `*Options` arrays, use `defineSelectOptions(...)` — it preserves literal types AND enforces the non-empty tuple shape. No Laioutr package exports it: write the small helper in your own `shared-fields/`. See [`references/shared-field-options.md`](./references/shared-field-options.md).

## Common mistakes

| Mistake | Fix |
| --- | --- |
| Field named `style` to hold the variant selector | Rename to `variant`; the bare `style` is forbidden |
| `containerStyle`, `accordionStyle` (or any `<component>Style`) as the top-level variant selector | Rename to `variant`; compound `*Style` names are reserved for the `as: 'style'` decorator pattern |
| `backgroundColor`, `customBackground` (used as variant) | Rename to `background` |
| `blockMargin` / `innerBlockPadding` | Rename to `margin` / `padding` |
| `required: true` on a schema field | Schema fields have no `required` property — the field type's runtime fallback fills an unset value |
| Static shared field written `satisfies StudioFieldDefinition` or annotated `: StudioFieldDefinition` | Wrap it in your local `defineField(...)` (or `as const satisfies`); otherwise `name` widens and the props type collapses |
| `import { defineSelectOptions } from …` a Laioutr package | It isn't exported anywhere — define it in your `shared-fields/` |
| Generic factory in `defineSection.fields` and `name` widens to `string` | Collapse to a single `const Opts extends {...}` generic and derive the resolved `name` from `Opts` at the return type (see `shared-field-factories.md`); or extract the call outside the array context |
| `*Options` typed via `as const satisfies [Option, ...]` | Replace with `defineSelectOptions(...)` |
| Block component named `ProductCard` (missing prefix) | Rename to `BlockProductCard` |
| Hand-written `interface Props` / `withDefaults` for a Section or Block | `defineProps(definitionToProps(definition))` |
| `props.size ?? 'm'` / `safeVariant` computed on a picker | Drop it — a picker is always one of its options (see `trust-the-schema-in-components.md`) |
| `props.items ?? []` / `props.items?.map(...)` / `v-if="props.items"` on an `array` field | Drop it — arrays are always `[]` at least; use `.length` for an empty-state branch |
| `props.style ?? {}` / `props.style?.foo` on an `object` prop | Drop it — objects are always filled, sub-fields recursively; guard the inner leaves that can be `undefined` |
| No `v-if` on a `media` / `link` / `number` / `icon` / `color` prop because its field has a `default` | Guard it — these fall back to `undefined`; `default` only pre-fills Studio |
| `media?.url` / `link.targetId` | `Media` has no `url`, `Link` has no `targetId`: guard with `v-if="media"` / `v-if="link"`, resolve links with `linkResolver.resolve()` or `useResolvedLink()` |
| Adding a picker to a Section already on pages and expecting those pages to get its `default` | They get the first option — decide which option existing pages should show and put it first, or make it the `default` |

## Reference index

- [`references/section-config-standard.md`](./references/section-config-standard.md) — complete sidebar group + canonical-name + preset specification
- [`references/forbidden-field-names.md`](./references/forbidden-field-names.md) — full forbidden-name list with failure modes
- [`references/schema-field-if.md`](./references/schema-field-if.md) — `if` operator reference + per-type fallback behavior
- [`references/block-naming.md`](./references/block-naming.md) — `Block*.vue` naming + component-resolution rules
- [`references/shared-field-factories.md`](./references/shared-field-factories.md) — factory function literal-type discipline
- [`references/shared-field-options.md`](./references/shared-field-options.md) — `defineSelectOptions` and `*Options` arrays
- [`references/trust-the-schema-in-components.md`](./references/trust-the-schema-in-components.md) — per-field-type runtime fallbacks: which props are always present, which can be `undefined`, and why `default` doesn't count
- [`references/consuming-query-fields.md`](./references/consuming-query-fields.md) — reading a `query` prop in a component: one entity vs a set, pagination, choosing components / links / limit
- [`references/components-render-without-blocking.md`](./references/components-render-without-blocking.md) — before writing `await` (including `await useFetchAction` / `await fetchAction`) in the `<script setup>` of a Section, Block or a component that loads its own data
- [`references/mutations-use-action-tokens.md`](./references/mutations-use-action-tokens.md) — a Section, Block or component needs to submit, add to cart, save, delete or otherwise call a backend, before reaching for `$fetch` or an SDK
- [`references/tracking-never-degrades-the-page.md`](./references/tracking-never-degrades-the-page.md) — adding a `track(...)` call or analytics-only props, especially next to a mutation's try/catch
- [`references/ai-metadata.md`](./references/ai-metadata.md) — before adding an `ai` key to a definition, or when writing `studio.description`
- [`references/sizing.md`](./references/sizing.md) — editors need to control a Section's or Block's height or aspect ratio, or before adding a `height` / `aspectRatio` field
- [`references/per-field-text-styling.md`](./references/per-field-text-styling.md) — letting editors change the size or color of one text region (heading, subline, body, caption)

For the broader architectural context (when to create a custom section vs. fork upstream), see the [`three-layer-architecture.md`](../laioutr-platform/references/three-layer-architecture.md) in the `laioutr-platform` skill.
