# Trust the Field-Type Contract in Components

Type a Section's or Block's props with `defineProps(definitionToProps(definition))`, never by hand, and use each prop the way its **field type** guarantees. Frontend Core fills every unset prop with its field type's fallback before render. `text`, `textarea`, `richtext`, `checkbox`, the pickers (`select`, `radio`, `toggle_button`, `content_alignment`), `object` and `array` always arrive with a value, so write no `??`, no optional chaining, no `withDefaults` and no `includes()`-narrowing for them. `number`, `icon`, `color`, `media`, `link` and `query` can arrive as `undefined`, and `json` as `null` — guard those.

A field's `default` is **not** part of this contract. Studio writes the `default` once, when an editor adds the section to a page. A section placed before the field existed, or one whose value was cleared, gets the field type's fallback instead.

## The contract

| Field type | Value when unset | Prop type from `definitionToProps` |
| --- | --- | --- |
| `text`, `textarea`, `richtext` | `''` | `string`, required |
| `checkbox` | `false` — `true` for a visibility decorator (`as: 'visibility'`) | `boolean`, required |
| `select`, `radio`, `toggle_button` | the **first option's** value. A stored value outside the options is clamped to the first option too | union of the option values, required |
| `content_alignment` | `'center-center'` (`axis` unset or `'both'`); `'center'` for `axis: 'vertical'` or `'horizontal'` | `` `${'top' \| 'center' \| 'bottom'}-${'left' \| 'center' \| 'right'}` ``, or the single-axis union, required |
| `object` | an object whose sub-fields are each filled by these same rules | the object shape, required |
| `array` | `[]`. Each stored item carries an `id` and its sub-fields are filled by these same rules | `T[]`, required |
| `number`, `icon`, `color`, `media`, `link`, `query` | `undefined` | optional (`… \| undefined`) |
| `json` | `null` | optional |

`definitionToProps` already encodes this table: the always-filled types come out required, the rest optional. Declaring the props yourself throws that away and invites a `?` or a fallback the runtime never needs.

## Required

```vue
<script lang="ts">
import { defineSection, definitionToProps } from '#imports';

export const definition = defineSection({
  component: 'SectionPromo',
  studio: { label: 'Promo' },
  slots: [],
  schema: [
    {
      label: 'Content',
      fields: [
        { type: 'text', name: 'heading', label: 'Heading' },
        { type: 'media', name: 'media', label: 'Media' },
        { type: 'link', name: 'link', label: 'Link' },
        {
          type: 'array',
          name: 'items',
          label: 'Items',
          schema: [{ fields: [{ type: 'text', name: 'label', label: 'Label' }] }],
        },
      ],
    },
    {
      label: 'Design',
      fields: [
        { type: 'info', name: 'stylingDivider', label: 'Styling' },
        {
          type: 'select',
          name: 'background',
          label: 'Background',
          default: 'none',
          options: [
            { label: 'None', value: 'none' },
            { label: 'Pale', value: 'pale' },
            { label: 'Solid', value: 'solid' },
          ],
        },
        { type: 'info', name: 'layoutDivider', label: 'Layout' },
        { type: 'number', name: 'columns', label: 'Columns', default: 3 },
      ],
    },
  ],
});
</script>

<script setup lang="ts">
const props = defineProps(definitionToProps(definition));

// ✅ select: always one of the options — exhaustive lookup, no fallback
const bgClass = computed(() => ({ none: '', pale: 'promo--pale', solid: 'promo--solid' })[props.background]);
// ✅ number: really can be undefined, whatever `default` says — a fallback is correct here
const columns = computed(() => props.columns ?? 3);
</script>

<template>
  <section :class="['promo', bgClass]" :style="{ '--promo-columns': columns }">
    <!-- ✅ text is '' when unset: test the value, not the prop's existence -->
    <h2 v-if="heading">{{ heading }}</h2>
    <!-- ✅ media and link can be undefined: guard them -->
    <LMedia v-if="media" :media="media" />
    <NuxtLink v-if="link" :to="linkResolver.resolve(link)">More</NuxtLink>
    <!-- ✅ array is always an array: branch on .length -->
    <ul v-if="items.length">
      <li v-for="item in items" :key="item.id">{{ item.label }}</li>
    </ul>
  </section>
</template>
```

`LMedia` (and `LIcon`, which takes an icon value as its `name` prop) are ui-kit auto-imports; `linkResolver` and `useResolvedLink` are Frontend Core auto-imports. Outside a template or `computed()`, resolve a link with `useResolvedLink()` rather than a bare `linkResolver.resolve()`, so it reacts to locale changes and link-resolver hooks.

## Forbidden

```ts
// ✘ Hand-declared props — loses the derived optionality and invites the patterns below
interface Props { background?: 'none' | 'pale' | 'solid' }
const props = withDefaults(defineProps<Props>(), { background: 'none' });

// ✘ Fallback on a picker — the value is always one of the options
const bg = props.background ?? 'none';

// ✘ Runtime narrowing for an impossible off-list value
const safeBackground = computed(() =>
  (['none', 'pale', 'solid'] as const).includes(props.background) ? props.background : 'none',
);

// ✘ Fallback or optional chaining on an array or object prop
const labels = (props.items ?? []).map((item) => item.label);
props.items?.map((item) => item.label);

// ✘ Assuming `default` fills a number, icon, color, media or link field — it doesn't
const columns: number = props.columns; // can be undefined even with `default: 3`

// ✘ Guarding media or link on fields they don't have
// `Media` has no `url`, and `Link` is a union of reference / url / anchor / page / pageType with no `targetId`
v-if="media?.url"
v-if="link.targetId"
```

## Why

1. **Frontend Core fills the prop, not Studio.** Before render it walks the schema and replaces every missing value with `getFieldFallback(field)`, then clamps picker values onto their options. That is the only guarantee there is, and it is per field type. `default` only pre-fills Studio's form at creation time, so code that relies on it breaks for every section placed before the field was added.
2. **A picker's fallback is its first option, not its `default`.** The code is safe either way — the value is always a valid option. The visible difference appears when you add a picker to a Section or Block that is already placed on pages: those instances render the **first option**, new ones the `default`. Keep options in their natural order (`s`, `m`, `l`) as upstream does; when adding a picker to a component already in use, decide which option existing pages should get, and make that the first option or the `default` accordingly.
3. **Exact types keep exhaustiveness checks working.** A required `'none' | 'pale' | 'solid'` lets the compiler flag a missing branch when an option is added. A `?` or a fallback widens the type and turns that check off.
4. **Defensive code for impossible states hides the contract.** Each unreachable `??` tells the next reader the value might be missing, and the pattern gets copied into components where it hides real bugs.
5. **The types that can be absent really can be.** `number`, `icon`, `color`, `media`, `link` and `query` fall back to `undefined`, whatever their `default` says. Skipping their guard is the opposite mistake, and it crashes at render.

## Red flags — stop and check the table

- About to write `interface Props` or `withDefaults` for a Section or Block — use `definitionToProps`.
- About to write `?? 'x'`, `?? []`, `?? {}` or `?.` on a `text`, picker, `checkbox`, `object` or `array` prop.
- About to write a `safeX` computed or an `includes()` check on a picker value.
- About to drop the `v-if` on a `media`, `link`, `icon`, `color`, `number` or `query` prop because its field has a `default`.
- About to add a picker to a Section or Block already placed on pages without deciding what those pages get (the first option, not the `default`).

## Related

- [`schema-field-if.md`](./schema-field-if.md) — the same fallbacks decide what an `if` expression sees for an unset field.
- [Schema fields — default values and runtime fallbacks](https://docs.laioutr.com/apps/app-development/schema-fields) — the docs' version of the contract table.
