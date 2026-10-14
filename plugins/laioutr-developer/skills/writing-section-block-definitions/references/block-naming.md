# Block Naming

Naming convention for `Block*.vue` components in your module (typically `src/runtime/app/blocks/`, plus any section-colocated blocks).

## The rule

Every block component is named **`Block<Name>`** — `Block` is always a **prefix**, never a suffix.

```
✅ BlockCard
✅ BlockProductDetailCartButton
✅ BlockHeroSlide

❌ CardBlock
❌ HeroSlideBlock
❌ PersonaQuoteBlock
```

The corresponding `defineBlock({ component: '...' })` string must match the filename verbatim:

```ts
// BlockCard.vue
export const definition = defineBlock({
  component: 'BlockCard',
  // ...
});
```

## Match the underlying ui component 1:1

`<Name>` is the name of the `ui` component the block wraps. Don't add suffixes, don't shorten:

| ui component              | Block file                         |
| ------------------------- | ---------------------------------- |
| `Card`                    | `BlockCard.vue`                    |
| `ProductDetailCartButton` | `BlockProductDetailCartButton.vue` |
| `LightboxGallery`         | `BlockLightboxGallery.vue`         |

If the block wraps a layout that no upstream `@laioutr-core/ui` component provides, follow the resolution ladder in [`three-layer-architecture.md`](../../laioutr-platform/references/three-layer-architecture.md) (prop override → local custom organism → fork from `laioutr-ui/ui-kit` or `laioutr-ui/ui` → upstream issue) and name the block after whichever component it ends up wrapping.

## Non-standalone blocks

When `isStandalone: false`:

1. **Communicate** to the user that the block will be non-standalone and explain why.
2. **Ensure a hosting Section exists** that lists this block in a slot's `allow` array. A non-standalone block with no host is unreachable from Studio.
3. Both changes ship in the same PR — never land an orphan non-standalone block.

## Filename ↔ identifier ↔ studio label

These three are independent:

- **Filename + `component` string** — `BlockProductDetailCartButton` (developer-facing, must match)
- **`studio.label`** — `'Add to Cart'` (editor-facing, plain English/Deutsch, no `Block` prefix)

Editors never see the `Block` prefix. It is a namespace marker: Sections and Blocks from every installed app share one global component registry, and a duplicate name silently overwrites the other app's component. Keep the prefix, and make the rest of the name specific enough to be unique across the project (see `unique-component-names.md` in the `laioutr-platform` skill).
