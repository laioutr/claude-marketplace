# Block Naming

Naming convention for `Block*.vue` components in your module's ui-app-layer (typically `src/runtime/app/block/`, plus any section-colocated blocks).

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

If the block wraps a layout that doesn't yet have a ui component, build the ui component first. A block that names something that doesn't exist in `ui` is a code smell — see `package-role.md`.

## Non-standalone blocks

When `isStandalone: false`:

1. **Communicate** to the user that the block will be non-standalone and explain why.
2. **Ensure a hosting Section exists** that lists this block in a slot's `allow` array. A non-standalone block with no host is unreachable from Studio.
3. Both changes ship in the same PR — never land an orphan non-standalone block.

## Filename ↔ identifier ↔ studio label

These three are independent:

- **Filename + `component` string** — `BlockProductDetailCartButton` (developer-facing, must match)
- **`studio.label`** — `'Add to Cart'` (editor-facing, plain English/Deutsch, no `Block` prefix)

Editors never see the `Block` prefix; it's a developer-facing namespace marker.
