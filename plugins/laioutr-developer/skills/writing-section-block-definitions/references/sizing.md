# Size the Outer Box with One `sizing` Field and `<Sizer>`

When editors need to control how tall a Section's or Block's main box is (a fixed height, a viewport-relative height, an aspect ratio, or per-breakpoint values), give it **one** `object` field named `sizing` in Design → Layout and pass its value to ui-kit's `<Sizer>`. Never add bespoke outer-box fields (`height`, `aspectRatio`, `desktopHeight` / `mobileHeight`, `orientation`), and never give your presentational components outer-box height or aspect-ratio props.

## The model

- **`Sizer`** comes from `@laioutr-core/ui-kit`. It is auto-registered as `<LSizer>`, or you can import it with `import Sizer from '#ui-kit/components/Sizer/Sizer.vue'`. It turns a `sizing` config into `height` / `aspect-ratio` through CSS custom properties, with mobile, tablet and desktop fallbacks. It renders a `<div>` by default, `as` renders another element, and `as-child` merges onto the single child. The prop's type, `Sizing`, is exported from the same `.vue` file.
- **Your `sizing` field** is an `object` field shaped 1:1 like `Sizing`, so its value passes straight through. Laioutr UI's own `sizingField` lives inside `@laioutr-app/ui` and is not exported. Declare the equivalent in your `shared-fields/`:

```ts
// src/runtime/app/shared-fields/sizing.ts
import { defineField } from './defineField'; // the one-line identity helper, see shared-field-factories.md

export const sizingField = defineField({
  type: 'object',
  name: 'sizing',
  label: 'Sizing',
  schema: [
    {
      fields: [
        {
          type: 'select',
          name: 'mode',
          label: 'Mode',
          options: [
            { label: 'Auto', value: 'auto' },
            { label: 'Fill', value: 'fill' },
            { label: 'Fixed', value: 'fixed' },
            { label: 'Responsive Fixed', value: 'responsive-fixed' },
            { label: 'Aspect Ratio', value: 'aspect-ratio' },
            { label: 'Responsive Aspect Ratio', value: 'responsive-aspect-ratio' },
          ],
        },
        {
          type: 'select',
          name: 'unit',
          label: 'Unit',
          options: [
            { label: 'Pixels (px)', value: 'px' },
            { label: 'Viewport height (dvh)', value: 'dvh' },
          ],
          if: ['in', [['fixed', 'responsive-fixed']], ['get', 'mode']],
        },
        { type: 'number', name: 'value', label: 'Height', if: ['==', ['get', 'mode'], 'fixed'] },
        { type: 'number', name: 'valueMobile', label: 'Mobile', if: ['==', ['get', 'mode'], 'responsive-fixed'] },
        { type: 'number', name: 'valueDesktop', label: 'Desktop', if: ['==', ['get', 'mode'], 'responsive-fixed'] },
        { type: 'text', name: 'aspectRatio', label: 'Aspect Ratio', placeholder: 'e.g. 16/9', if: ['==', ['get', 'mode'], 'aspect-ratio'] },
        { type: 'text', name: 'aspectRatioMobile', label: 'Mobile', placeholder: 'e.g. 1/1', if: ['==', ['get', 'mode'], 'responsive-aspect-ratio'] },
        { type: 'text', name: 'aspectRatioTablet', label: 'Tablet', placeholder: 'empty = Mobile', if: ['==', ['get', 'mode'], 'responsive-aspect-ratio'] },
        { type: 'text', name: 'aspectRatioDesktop', label: 'Desktop', placeholder: 'e.g. 21/9', if: ['==', ['get', 'mode'], 'responsive-aspect-ratio'] },
      ],
    },
  ],
});
```

The field-type fallbacks make this safe without guards. `mode` falls back to its first option, `auto`. The numbers fall back to `undefined` and the texts to `''`, and `Sizer` treats both as "not set". Pass the prop straight through: `<Sizer :sizing="sizing">`.

## The six modes

| Mode | Reads | Result |
| --- | --- | --- |
| `auto` | nothing | Intrinsic content height, same as no `sizing` |
| `fill` | nothing | `height: 100%`. Needs a parent with a definite height |
| `fixed` | `value`, `unit` | `height: <value><unit>` |
| `responsive-fixed` | `valueMobile`, `valueDesktop`, `unit` | Mobile height below 1280px, desktop height from 1280px |
| `aspect-ratio` | `aspectRatio` | CSS `aspect-ratio`, e.g. `'16/9'` |
| `responsive-aspect-ratio` | `aspectRatioMobile`, `aspectRatioTablet`, `aspectRatioDesktop` | Below 600px, 600–1279px (falls back to mobile), from 1280px |

`unit` is `px` or `dvh`. `dvh` follows the dynamic viewport, so a viewport-relative hero stays correct as mobile browser chrome shows and hides.

`fill` only produces visible height when the parent has a definite height. Inside an auto-height parent, such as a slider's slide, it collapses to zero. That is a CSS fact, not a Sizer bug.

## When a component takes a `sizing` field

All four must hold:

1. It has a single primary content region whose outer dimensions an editor should control.
2. The presentational component inside is sizing-agnostic: its root sets `width: 100%` and no opinionated `height`.
3. No parent Section already dictates its dimensions. A uniform grid sizes its cards itself.
4. At least one realistic configuration exists where `auto` is the wrong answer.

Strong matches are banners, media blocks, embeds and hero slides. Text and control blocks, header and footer chrome, and slider sections are not matches, although a slider's individual slides can be.

## Outer box vs inner region

`Sizer` governs the **outer** box. A named inner region keeps its own prop.

To decide, ask whether the prop describes the component's bounding box or a region inside it with its own layout. A banner's overall aspect ratio is the outer box, so it becomes `sizing`. A card's image ratio or a product tile's image ratio is an inner region, so it keeps its own field (`mediaAspectRatio`, `imageAspectRatio`). When two regions have independent box shapes, use one region prop plus `sizing` for the outer box. Never put two Sizers around the same root.

When the editor's sizing should apply to one region rather than the whole component, pass `sizing` down instead of wrapping. ui-kit's `MediaPreview` takes a `sizing` prop and applies it to its media box only, so the caption stays below and outside the sized box. Your own presentational component can do the same: accept `sizing?: Sizing` and put the `<Sizer>` around that region.

## `as-child` vs the wrapping `<div>`

`<Sizer as-child>` merges Sizer's class and style onto the child's root element. Use it only when that root has `width: 100%` and no opinionated `height`.

Use the default wrapping `<div>` when the child's root sets `height: 100%`. With `as-child`, one element would carry two competing height rules, `.sizer { height: var(...) }` and your `height: 100%`, at equal specificity, and stylesheet load order would pick the winner. Also wrap raw HTML you size yourself, such as an `<iframe>`.

When in doubt, wrap. It always works.

```vue
<!-- src/runtime/app/blocks/BlockEmbed.vue -->
<script lang="ts">
import Sizer from '#ui-kit/components/Sizer/Sizer.vue';
import { defineBlock, definitionToProps } from '#imports';
import { sizingField } from '../shared-fields/sizing';

export const definition = defineBlock({
  component: 'BlockEmbed',
  studio: { label: 'Embed' },
  schema: [
    {
      label: 'Content',
      fields: [
        { type: 'text', name: 'src', label: 'URL' },
        { type: 'text', name: 'title', label: 'Title' },
      ],
    },
    { label: 'Design', fields: [sizingField] },
  ],
});
</script>

<script setup lang="ts">
defineProps(definitionToProps(definition));
</script>

<template>
  <!-- wrapping div, not as-child: the iframe fills it with height: 100% -->
  <Sizer :sizing="sizing">
    <iframe v-if="src" class="block-embed" :src="src" :title="title" />
  </Sizer>
</template>

<style scoped>
.block-embed {
  width: 100%;
  height: 100%;
  border: 0;
}
</style>
```

## Hard rules

1. **The Section or Block owns the sizing decision.** It reads `sizing` and either wraps in `<Sizer>` or passes `sizing` to a component that applies it to a named region.
2. **No outer-box sizing props on presentational components.** A prop that governs the component's bounding box moves into the consumer's `sizing` field. Region props stay.
3. **One sizing source per box.** Adding `sizing` means removing any bespoke height or aspect-ratio fields for the same box.
4. **The field is named `sizing`.** Don't rename it per component, and don't add a parallel `size`, `dimensions` or `height`.
5. **It lives in Design → Layout.** Laioutr UI places it last, after `alignment`.
6. **Editor labels are intent words**, such as "Auto", "Fill", "Fixed" and "Aspect Ratio". Never use CSS terms (`100%`, `vh`, `flex`).

## Why

1. **One sidebar shape for "make this taller".** Laioutr UI's banners, `BlockMedia` and `BlockIframe` all use this `sizing` shape (UI changelog 2.2.1, which removed their bespoke `aspectRatio`, `height` and `desktopHeight` / `mobileHeight` fields). Editors who learned it there expect it in your app too, the same argument the section config standard makes for field names.
2. **A definite height is what vertical alignment works against.** With `auto`, a banner's content has no spare height to distribute. Laioutr UI's banner vertical alignment only takes effect once Sizing gives the banner a height.
3. **No mapping code.** The field's shape is the prop's shape, and Sizer already handles every unset value the field types produce.
4. **The presentational component stays reusable.** A component that fills whatever box it is given works under any Section, in Storybook, and inside a parent that sets its own height. One that owns its height fights all three.

## Related

- [Sizer docs](https://docs.laioutr.com/laioutr-ui/ui-kit/utility-components/sizer): modes, breakpoints, API reference.
- [`section-config-standard.md`](./section-config-standard.md): Design → Layout and canonical field names.
- [`trust-the-schema-in-components.md`](./trust-the-schema-in-components.md): the field-type fallbacks that make `sizing` safe to pass straight through.
- [`shared-field-factories.md`](./shared-field-factories.md): the `defineField` identity helper that keeps the field's literal types.
- [`no-wrapper-css-overrides.md`](../../laioutr-platform/references/no-wrapper-css-overrides.md): use props and Sizer, not wrapper CSS, to size a child.
