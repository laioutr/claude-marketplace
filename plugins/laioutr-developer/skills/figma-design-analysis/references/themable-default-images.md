# Themable Default Images

Components sometimes provide default decorative images (backgrounds, illustrations, empty states, placeholders) that vary by theme and optionally by breakpoint or color mode. These are theme-provided assets rendered via the `Media` component, not content from APIs.

This reference covers the **themable** image lane only. For one-off images that ship as files in `runtime/public/` and don't belong in any `ImageSet` (component-specific photos/illustrations, partner logos, custom markers, raster CTA backgrounds), use the `figma-export-assets` skill — it owns format/scale/destination/filename decisions and the export spec.

## Image Classification

| Category | Source | Themable? | Example |
|---|---|---|---|
| Product content | API data layer, arrives as `Media` prop | No | Product photo in a card |
| Product placeholder | Theme `ImageSet`, shown when API data is missing | Yes | `placeholder1x1Product` |
| Empty state illustration | Theme `ImageSet`, shown when a list/section is empty | Yes | `emptyStateBox` |
| Decorative background | Theme `ImageSet`, component visual design | Yes | `bannerBasicDefault` |
| Brand/partner assets | External URLs or icon set (`logos/*` in shared icons) | No | PayPal logo, Visa badge |
| Inline decorative SVGs | Hardcoded in component template or CSS | No | Wave divider, border pattern |

## How to Identify in Figma

Look for images that:
- Serve as backgrounds, illustrations, placeholders, or empty states
- Would logically change per theme (even if the Figma file only shows one theme)
- Change depending on background brightness (dark vs light versions) — if the component uses `OnBackground`
- Have different aspect ratios or crops for mobile vs desktop

Even if the Figma file shows only one theme variant, any non-product, non-brand decorative image should be assumed themable. The codebase convention is that ALL such images are theme-provided.

**NOT themable:** inline SVGs embedded directly in the template, CSS-generated decorative elements (gradients, shapes via `::before`/`::after`), and external brand logos.

## Registration Helpers

| Helper | When | Result |
|---|---|---|
| `themeMedia(path, w, h, color, ext, scales)` | Same image for all breakpoints | Single `MediaImage` |
| `themeResponsiveMedia(path, mobileW, mobileH, desktopW, desktopH, color, ext, scales)` | Separate `{path}-mobile.{ext}` and `{path}-desktop.{ext}` | Single `MediaImage` with responsive sources |

Images are registered per theme in `ImageSet` (defined by `@laioutr-core/ui-kit`). Each entry is a `MediaImage` or a `[MediaImage, MediaImage]` tuple for `[light, dark]` color modes. Upstream ships canonical keys (`placeholder1x1Product`, `emptyStateBox`, …). Your module registers additional keys (and overrides existing ones) by calling `extendTheme()` in its own theme module.

## Brightness-Based Naming Convention

Components using `OnBackground` typically require **three** `ImageName` entries:

| Key | Type | When used |
|---|---|---|
| `{prefix}Default` | `[MediaImage, MediaImage]` tuple | `backgroundBrightness` is `'light'` — element 0 for light color mode, element 1 for dark |
| `{prefix}Dark` | `MediaImage` | `backgroundBrightness` is `'dark'` |
| `{prefix}Bright` | `MediaImage` | `backgroundBrightness` is `'bright'` |

Example: `bannerBasicDefault`, `bannerBasicDark`, `bannerBasicBright`.

## Consumption in Components

```typescript
const theme = useTheme();
const bg = theme.image('bannerBasicDefault'); // auto-resolves [light,dark] tuple
```

Pass the result to `<Media :media="bg" />`.

## Theme Inheritance

Upstream themes form a chain: `@laioutr-core/ui-kit`'s base theme defines all canonical images, and child themes (`classic`, `sunny`, `tech`) override only what differs via `extendTheme()`. Your module typically defines its own theme on top of one of these via `extendTheme()` as well.

When proposing new themable images:

- **If the key is app-specific** (only your storefront needs it), register it in your module's own theme. `ImageName` includes `| (string & {})` for extensibility, so `theme.image('myCustomKey')` works without upstream changes. Document the contract in your theme's source.
- **If the key is generic** (multiple Laioutr storefronts would want it), the canonical key needs to land in `@laioutr-core/ui-kit`'s `ImageName` type and base theme. You can't edit those directly — file an issue against `laioutr/ui-source` or open a PR there. In the meantime, register it locally as an app-specific key and revisit when upstream lands.

## In the Analysis Plan

Note themable images as behavioral notes in the Component Hierarchy and map them to `ImageName` keys. If no existing `ImageName` matches, flag whether the new key is app-specific (register in your module's theme) or generic (needs an upstream change — short-term workaround: register locally via the string-extension escape).

```markdown
- **BannerHero** -- NEW
  - _Themable default image: background illustration varies by theme, breakpoint, and color mode (3 keys: Default/Dark/Bright). Uses themeResponsiveMedia()._
```
