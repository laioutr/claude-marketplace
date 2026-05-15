# No Outer Chrome on ui-kit Primitives

A `ui-kit` primitive's **bounding box must equal its visible footprint**. Spacing inside the control's own border is anatomy and stays. Spacing around the control is context and belongs to whoever places it.

Inverse of `no-wrapper-css-overrides.md`: that rule forbids the override; this one forbids the chrome that forces it.

## The rule

A `ui-kit` component must not bake outer padding, margin, container width, or surrounding background — anything that extends its bounding box past the visible control — unless the component is _chrome by design_ (see Exceptions).

**Smell test:** if two or more consumers override the same property the same way, the default was wrong. Strip the chrome from the primitive in the same commit you spot the second override.

## Forbidden

```vue
<!-- ❌ SwiperNavigation.vue (ui-kit) -->
<style>
.swiper-navigation {
  display: flex;
  gap: var(--spacing-s);
  padding: var(--spacing-m); /* outer chrome */
  margin-block: var(--spacing-l); /* outer chrome */
}
</style>
```

Also forbidden: a wrapper `<div>` whose only job is layout context, a `min-height` larger than the natural control "for layout", or a background/border on a wrapping container instead of on the control itself.

## Allowed: control anatomy

Spacing _inside_ the control's own boundary that defines its visible identity. Test: does removing it change what the control fundamentally **looks like**, or only **where it sits**?

- Button interior padding between border and label — anatomy.
- Token-driven sizing (`--sizing-button-height-l`, `--sizing-icon-size-m`) — anatomy.
- **Natural control dimensions** — an Input's `height`, a tile's intrinsic `width`. The control IS that size.
- **Hit-target padding** around an interactive surface (Scrollbar's track padding that enlarges the grab area, Switch padding around its thumb track) — anatomy.
- **Outline / focus-ring buffer** — when a control's outline (or focus ring) renders outside its border via `outline-offset`, the visible footprint extends past the border by `outline-offset + outline-width`. Margin equal to that extent is anatomy — without it, adjacent controls' outlines collapse or overlap.
- Margin between the control and its neighbour — context. Not yours.

## Components that are frames

`Backdrop`, `Card`, `OnSurface`, `Sheet`, `Dialog`, `Popover`, `EmptyState`, `LoadMore`, `Accordion`. The first seven are pure frame primitives whose only job is to host content; the last two are molecules with built-in frame — they bundle controls into a single Figma-specified surface. Both kinds ship chrome by Figma design and are immune to this rule.

The Figma spec is the contract. If a consumer wants any of these without the frame, that's a missing variant prop (`variant: 'plain' | 'boxed'`) on the primitive — not an excuse to override its CSS. A `SwiperNavigation` / `Pagination` / `IconButton` is **not** a frame.

## When you find an override

The smell test ("two consumers override the same property") is a **signal**, not a verdict. Decide which side is wrong before stripping anything:

1. **Does Figma show the frame on this primitive?** Yes → the override is a `no-wrapper-css-overrides` violation. Delete it, or add a variant prop on the primitive if the consumer represents a real design state.
2. **Is the consumer asking for a different size or dimension?** Add a `size` variant. The primitive's natural dimensions are anatomy — consumers shouldn't override them.
3. **Otherwise** the primitive is leaking chrome. Strip it:
   - LSP `findReferences` to count consumers and how many neutralize each property.
   - Strip the chrome.
   - **If the chrome lives on a wrapper element you're deleting, move the component's root block-class onto the new root.** Per [`../public-css-api.md`](../public-css-api.md) rule #1, every component must emit a kebab-case root class matching its file name. Removing the wrapper without re-anchoring the class breaks the public CSS surface.
   - Delete every now-redundant override (`padding: 0`, `:deep()` reset, etc.) in the same commit.
   - Fix Storybook: if a story needs spacing to look right, the _decorator_ gets the spacing, not the primitive.
   - If your module ships versioned releases, mark the change breaking for any consumer whose CSS changed — the bounding box just shrank.

## Why

1. **Composition fails when primitives have opinions about placement.** Two contexts, two margins, one component — the only escape is overrides, which `no-wrapper-css-overrides.md` forbids.
2. **Overrides are invisible to refactor tools.** A `.swiper-navigation { padding: 0 }` reset doesn't show up in `findReferences` of `SwiperNavigation`. Rename the BEM class and every override silently breaks.
3. **Studio can't reach hardcoded chrome.** Section spacing is editorial; outer chrome on a primitive bypasses the `Backdrop` / surface-tone pipeline that owns it.

If every plausible call site genuinely wants the same spacing, write a chrome-named wrapper (`NavigationFrame`) — don't bake it into the control.

## Related

- [`../no-wrapper-css-overrides.md`](../no-wrapper-css-overrides.md) — wrapper-side counterpart
- [`package-role.md`](package-role.md) — "visually authoritative" applies to shape, not placement
- [`../ui/ui-kit-first.md`](../ui/ui-kit-first.md) — `ui`-layer sections compose `Backdrop`; controls inside must not bring competing chrome
