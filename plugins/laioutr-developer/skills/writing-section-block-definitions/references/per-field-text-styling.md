# Per-Region Text Size and Color Live in the Region's Style Decorator

When a Section or Block lets editors restyle one text region (heading, subline, body, caption), put a `textSize` select and a `color` field inside that region's **style decorator**: the `object` field named `<region>Style`, with `as: 'style'` and `for: '<region>'`. Never add them as top-level fields, and never invent a per-component size enum or an ad-hoc color field.

```ts
{ type: 'text', name: 'heading', label: 'Heading' },
{
  type: 'object',
  name: 'headingStyle',
  label: 'Style',
  as: 'style',
  for: 'heading',
  schema: [{ fields: [textSizeField, textColorField] }],
},
```

## The schema side

- **`textSize`** is a `select` over the full ten-step scale, `xs`, `s`, `sm`, `m`, `ml`, `l`, `xl`, `2xl`, `3xl`, `4xl` (ui-kit's `TextSize`), with `'auto'` as the **first** option. `'auto'` means "no override, use the component's default". It is also the field's runtime fallback, because a picker falls back to its first option. Don't offer a trimmed S/M/L subset per region. A block-wide S/M/L `size` (the `size.ts` preset in [`section-config-standard.md`](./section-config-standard.md)) is a different concept.
- **`color`** is a `color` field with `allowCustom: true`, matching the canonical foreground-color field in the section config standard.

Laioutr UI's own `textSizeField`, `textColorField` and size resolvers are internal to `@laioutr-app/ui` and not exported. Declare the equivalents once in your `shared-fields/`:

```ts
// src/runtime/app/shared-fields/textStyle.ts
import type { TextSize } from '#ui-kit/types/text';
import { defineField } from './defineField'; // one-line identity helper, see shared-field-factories.md
import { defineSelectOptions } from './defineSelectOptions'; // see shared-field-options.md

export const textSizeOptions = defineSelectOptions([
  { label: 'Default', value: 'auto' },
  { label: 'XS', value: 'xs' },
  { label: 'S', value: 's' },
  { label: 'SM', value: 'sm' },
  { label: 'M', value: 'm' },
  { label: 'ML', value: 'ml' },
  { label: 'L', value: 'l' },
  { label: 'XL', value: 'xl' },
  { label: '2XL', value: '2xl' },
  { label: '3XL', value: '3xl' },
  { label: '4XL', value: '4xl' },
]);

export const textSizeField = defineField({
  type: 'select',
  name: 'textSize',
  label: 'Text Size',
  default: 'auto',
  options: textSizeOptions,
});

export const textColorField = defineField({
  type: 'color',
  name: 'color',
  label: 'Text Color',
  allowCustom: true,
});

/** `'auto'` is "no override": return `undefined` so the receiving component's own default wins. */
export const resolveTextSize = (textSize: TextSize | 'auto'): TextSize | undefined =>
  textSize === 'auto' ? undefined : textSize;
```

## The runtime side

Resolve `'auto'` to `undefined` and convert the color with `colorValueToCss` from `#ui/utils/colorValueToCss`, then bind the results to the receiving component's per-region props:

```ts
// in the Block's <script setup>, with the definition above
const headingSize = computed(() => resolveTextSize(props.headingStyle.textSize));
const headingColor = computed(() => (props.headingStyle.color ? colorValueToCss(props.headingStyle.color) : undefined));
```

```vue
<LTextGroup :heading="heading" :size="size" :heading-size="headingSize" :heading-color="headingColor" />
```

ui-kit's `LTextGroup` takes `caption-size` / `heading-size` / `subline-size` (a `TextSize`) and `caption-color` / `heading-color` / `subline-color` (a CSS color string). It resolves each region as "override, else its block-wide `size` mapping", so passing `undefined` for `'auto'` hands the decision back to `size`. Give your own presentational components the same per-region props and the same order. If one lacks a per-region size or color prop, add the prop to that component. Don't restyle it from the Block.

## Forbidden

```ts
// ✘ top-level fields and a per-component enum
{ type: 'select', name: 'headingSize', label: 'Heading Size', options: [{ label: 'Small', value: 'small' }, { label: 'Big', value: 'big' }] },
{ type: 'color', name: 'headingColor', label: 'Heading Color' },
```

```vue
<!-- ✘ 'auto' passed through as if it were a size -->
<LTextGroup :heading="heading" :heading-size="headingStyle.textSize" />

<!-- ✘ a ColorFieldValue ({ light, dark }) is an object, not a CSS color -->
<h2 :style="{ color: headingStyle.color }">{{ heading }}</h2>

<!-- ✘ colorValueToCss(undefined) is 'initial', which resets the inherited color on your own element -->
<h2 :style="{ color: colorValueToCss(headingStyle.color) }">{{ heading }}</h2>
```

## Why

1. **Editors learn one control.** Every region in every app shows the same "Style" panel under its field, with the same scale. A per-component `small` / `big` enum is one more vocabulary to learn and maps to nothing in the type scale.
2. **The decorator keeps the control next to the text it styles.** `as: 'style'` with `for` makes Studio render the panel inline under the target field. A top-level `headingSize` renders as a separate control, detached from the heading it changes.
3. **`'auto'` must mean "unset" at runtime too.** `textSize` always arrives with a value, its first option, so the only way to express "keep the default" is the sentinel. Passing it through gives the component a size it doesn't know.
4. **The full scale matches the components.** The ten steps are ui-kit's `TextSize`, the scale `Text` and `LTextGroup` already accept, so an override binds without mapping code.
5. **Colors need conversion.** A `color` field holds theme tokens and light/dark values. `colorValueToCss` turns that into `var(--token)` or `light-dark(…)`. Binding the raw object produces no color, and binding `'initial'` from an unset field overrides the inherited one.

## Related

- [`section-config-standard.md`](./section-config-standard.md): style decorators are named `<fieldName>Style` with `as: 'style'`.
- [`forbidden-field-names.md`](./forbidden-field-names.md): the bare `style` is forbidden; compound `headingStyle` decorators are fine.
- [`trust-the-schema-in-components.md`](./trust-the-schema-in-components.md): the decorator object and `textSize` always arrive filled, and `color` can be `undefined`.
- [`shared-field-options.md`](./shared-field-options.md) and [`shared-field-factories.md`](./shared-field-factories.md): `defineSelectOptions` and `defineField` keep the literal types.
