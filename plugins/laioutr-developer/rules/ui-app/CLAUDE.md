# CLAUDE.md - ui-app

This package contains section and block components for the laioutr platform. Sections are configurable page components used to build storefronts.

## Reference

Detailed rules in this folder (`rules/ui-app/`):

- `package-role.md` — What ui-app is for, how it relates to ui and ui-kit, Section/Block interface requirements, why there are no stories
- `section-config-standard.md` — Schema panel conventions, shared-field presets
- `block-naming.md` — `Block<Name>` prefix convention and non-standalone block requirements
- `shared-field-options.md` — Required `as const satisfies` pattern for exported `*Options` lists so select-field props retain literal types
- `shared-field-factories.md` — How to write generic factory functions in `shared-fields/` without their `name` literal getting widened to `string` by contextual typing inside `defineSection`
- `schema-field-if.md` — Conditional field/fieldset visibility via the `if?: SchemaCondition` property; path syntax, available operators, common patterns, fail-open semantics, relationship to visibility decorators

## External References

- [Section Definitions](https://docs.laioutr.io/apps/app-development/section-definitions)
- [Block Definitions](https://docs.laioutr.io/apps/app-development/block-definitions)
- [Schema Fields](https://docs.laioutr.io/apps/app-development/schema-fields)

## Location

- Sections: `src/runtime/app/section/`
- Blocks: `src/runtime/app/section/` (colocated with their parent section) or `src/runtime/app/block/`
- Shared field schemas: `src/runtime/app/shared-fields/`

## Section Definitions

**File naming:** `Section<Name>.vue`. The `component` property must match the filename.

**Required properties:** `component`, `studio` (at minimum `label`), `slots` (empty array if none).

**Structure:** Two `<script>` blocks. The first exports the definition, the second uses `setup`. All imports go in `<script setup>` — the SFC compiler hoists them to module scope so the first block can reference them.

```vue
<script lang="ts">
export const definition = defineSection({
  component: 'SectionHeroBanner',
  studio: {
    label: 'Hero Banner',
    description: 'A full-width banner.',
    tags: ['Heroes', 'Banner'],
  },
  slots: [
    {
      name: 'default',
      studio: { label: 'Content' },
    },
  ],
  schema: [],
});
</script>

<script setup lang="ts">
import { defineSection, definitionToProps } from '#imports';

const props = defineProps(definitionToProps(definition));
</script>

<template>
  <slot />
</template>
```

`definitionToProps` generates Vue props from the schema AND adds a typed `slots` prop for accessing block data in each named slot.

### Slots

Slots are named insertion points where editors place blocks. They map to Vue `<slot>` elements.

- `name` (required) — must match `<slot name="...">` in template. Use `'default'` for the main slot.
- `studio.label` — label in Studio sidebar.
- `restrictTo` — only these blocks can be placed (string names or imported definition objects).
- `allow` — non-standalone blocks must appear here to be usable.
- `prefer` — these blocks appear first in Studio's block picker.

### Studio object

- `label` (required) — display name in Studio component picker.
- `description` — short description below the label.
- `previewSrc` — path to a preview image in the app's `public/` directory. Only set this if the image file actually exists.
- `tags` — categorization tags for the component picker.
- `propsWizard` — multi-step wizard for pre-configuring variants.

**Well-known tags:** `Banner`, `Brands`, `Blog`, `Checkout & Cart`, `Content`, `Customer Relations`, `Featuring Products`, `Footer`, `Grids`, `Header`, `Heroes`, `Navigation`, `Products`, `Product Detail Page`, `Product Listing Page`, `Testimonials`, `Sliders`, `Blank Containers`. Custom strings are also accepted.

## Block Definitions

**File naming:** `Block<Name>.vue`. The `component` property must match the filename.

**Required properties:** `component`, `studio` (at minimum `label`). `schema` and `isStandalone` are optional.

Blocks use `defineBlock()` instead of `defineSection()`. Unlike sections, blocks do NOT receive an automatic `slots` prop from `definitionToProps`.

### Standalone vs non-standalone

Blocks default to `isStandalone: true` — they can be placed in any slot unless restricted by `restrictTo`.

Set `isStandalone: false` when a block only makes sense inside a specific section. Non-standalone blocks can only be used in slots that list them in `allow`.

**When setting `isStandalone: false`, you MUST:**

1. Communicate to the user that the block will be non-standalone and explain why.
2. Ensure at least one section exists that lists this block in a slot's `allow` array. A non-standalone block with no hosting section is unreachable.

## Schema Fields

`schema` is optional but present on almost every section/block. It is an array of **fieldsets** (collapsible panels in the Studio sidebar).

### Fieldset properties

- `label` — panel heading.
- `helpText` — help text below the label.
- `defaultOpen` (default: `false`) — whether the panel starts expanded.
- `fields` (required) — array of field definitions.

### Base field properties

Every field type shares these and ONLY these base properties:

- `type` (required) — the field type identifier.
- `name` (required) — property name on the component's props. Must be unique within the definition.
- `label` — display label in the sidebar.
- `default` — initial value applied once when an editor creates a new section/block in Studio.
- `description` — help text below the field.

There is **no `required` property**. Fields that have no value use a runtime fallback.

### Default values vs runtime fallbacks

`default` is applied once at creation time in Studio. **Runtime fallbacks** are type-appropriate zero values applied by Frontend Core whenever a prop has no configured value:

| Field type                                          | Fallback                                   |
| --------------------------------------------------- | ------------------------------------------ |
| `text`, `textarea`, `richtext`                      | `''` (empty string)                        |
| `checkbox`                                          | `false` (`true` for visibility decorators) |
| `select`, `radio`, `toggle_button`                  | First option's value                       |
| `object`                                            | Object with fallbacks applied recursively  |
| `array`                                             | `[]`                                       |
| `json`                                              | `null`                                     |
| `number`, `icon`, `media`, `link`, `query`, `color` | `undefined`                                |

String fields always resolve to a string (no null checks needed). Fields that fall back to `undefined` need a guard.

### Primitive fields

- **text** — single-line input. Props: `placeholder`, `maxLength`. Prop type: `string`.
- **textarea** — multi-line input. Props: `placeholder`, `maxLength`. Prop type: `string`.
- **number** — numeric input. Props: `min`, `max`, `step`, `prefix`, `suffix`. Prop type: `number | undefined`.
- **checkbox** — boolean toggle. Prop type: `boolean`.
- **select** — dropdown. Props: `options` (required). Prop type: `string`.
- **radio** — radio buttons. Props: `options` (required). Prop type: `string`.
- **toggle_button** — segmented buttons. Props: `options` (required, with optional `icon`). Prop type: `string`.
- **icon** — icon picker. Prop type: `string | undefined`.
- **info** — read-only heading/help text. Not a data field, produces no prop.

### Complex fields

- **richtext** — rich text editor. Render with `<RichContent :html="..." />`. Prop type: `string`.
- **color** — color picker. Props: `allowCustom`, `allowAlpha`. Prop type: `ColorFieldValue | undefined`.
- **media** — media picker. Props: `allowedTypes`, `allowResponsive`, `allowFocalPoint`. Prop type: `MediaImage | MediaVideo | undefined`.
- **link** — link picker (reference, url, anchor, page, pageType). Resolve with `linkResolver.resolve()`. Prop type: `Link | undefined`.
- **object** — nested fields. Props: `schema` (same fieldset structure). Prop type: shaped by nested schema.
- **array** — repeatable list. Props: `schema`, `labelSingular`, `max`, `itemLabelProperty`. Each item gets a stable `.id`. Prop type: array of items shaped by schema.
- **json** — raw JSON editor. Props: `placeholder`. Prop type: `JSONType | null`.

### Query fields

Connect a section/block to entity data from Orchestr.

```typescript
{
  type: 'query',
  name: 'product',
  label: 'Product',
  entityType: 'Product',
  singleEntity: true,
  components: [ProductRating],
  links: {
    [unwrapToken(ProductReviewsLink)]: {
      entityType: 'Review',
      components: [ReviewBase],
      limit: 10,
    },
  },
}
```

- `entityType` (required) — canonical entity type to query.
- `singleEntity` — when `true`, prop is a single `ClientEntity`. When `false` (default), prop is a `ClientEntitySet` with `.entities` array.
- `components` — entity component tokens declaring what data slices to fetch.
- `links` — related entities to fetch. Each key is a link token, value has `entityType` (required), `components`, `limit`, and nested `links` (max 2 levels deep).

Rules:

- When `singleEntity: true`, the prop IS the entity directly — access `props.product?.components.rating`, NOT `props.product?.entities[0]`.
- Always set `limit` on link configs explicitly when pagination is involved.
- Pagination uses `.pagination.current` for the current page, not `.page`.
- Every link increases data Orchestr resolves per request. Only fetch components and links the component actually uses.

### Field decorators

Decorators link fields together in the Studio sidebar. They do not affect the data model — both decorator and target are separate props.

**Visibility toggles:** A `checkbox` with `for` and `as: 'visibility'` controls whether another field is visible in the sidebar. Falls back to `true` (not `false` like regular checkboxes).

```typescript
{ type: 'text', name: 'subtitle', label: 'Subtitle' },
{ type: 'checkbox', name: 'subtitleVisible', for: 'subtitle', as: 'visibility', default: true },
```

**Style objects:** An `object` with `for` and `as: 'style'` attaches styling controls inline below the target field.

```typescript
{ type: 'text', name: 'heading', label: 'Heading' },
{
  type: 'object', name: 'headingStyle', label: 'Heading Style',
  for: 'heading', as: 'style',
  schema: [{ fields: [{ type: 'color', name: 'color', label: 'Text Color' }] }],
}
```

## Icons

Use `<LIcon>` component (same as ui package):

```vue
<LIcon name="shopping/cart" />
```
