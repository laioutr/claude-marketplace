# Surface Tone

How a container's painted appearance (light / dark / bright) propagates to descendants so that text, icons, buttons, and other visual elements pick contrasting colors automatically. This rule covers any component in your Nuxt module that paints a background or that renders content whose contrast must adapt to its surroundings.

## What problem this solves

CMS authors compose pages by nesting containers. Each container can paint its own background — a themed preset, an image, or an arbitrary color from a picker. A `<Button>` placed inside a dark hero must render with light-on-dark chrome; the same button in a light section must render the opposite. Surface tone is the mechanism that lets a leaf component adapt automatically to whatever its surrounding container paints.

This is **not** the same as system dark mode (`prefers-color-scheme`) and **not** elevation/depth (Carbon's `<Layer>` tiers). It is a categorical classification of the current container's painted appearance: `light` (white-ish), `dark` (black-ish), `bright` (vibrant saturated). Three peer categories, declared by whichever container paints — never auto-derived from nesting depth.

The pattern has direct precedent in Bootstrap 5.3 (`data-bs-theme`), Squarespace 7.1 section themes (which use the same `light` / `dark` / `bright` vocabulary), Shopify section color schemes, and Radix Themes nested `<Theme>`.

## The model

Surface tone propagates via **two channels** that always travel together:

- **Channel A — Vue context.** `OnSurface` calls `provideSurfaceToneContext({ tone })`. Consumers can read it via `useSurfaceTone(props?)` which returns a `ComputedRef<SurfaceTone>`. Resolution order: explicit `surfaceTone` prop > injected ancestor context > `'light'`.
- **Channel B — CSS class + variable cascade.** `OnSurface` emits `class="on-{tone}"` and `data-surface-tone="{tone}"` on its root element. The `.on-{tone}` rules in `OnSurface.vue` redefine `--font-color-heading`, `--font-color-subline`, `--font-color-caption`, `--font-color-body-text`, and `--icon-color` to point at per-tone source variables (`--on-{tone}-font-color-heading`, etc.). Descendants that consume those variables resolve to the right values automatically.

The two channels exist because some atoms styling decisions can be expressed entirely in CSS (Channel B is enough), while others must branch in JS (Channel A is needed for picking between named variants, alternate icons, etc.).

Type: `SurfaceTone = 'light' | 'dark' | 'bright'`, exported from `@laioutr-core/ui-kit` (composable: `useSurfaceTone`).

## Setting tone

Only two components are allowed to establish a tone subtree:

- **`<OnSurface tone="…">`** — the canonical primitive provider. Sets both Channel A and Channel B. Use directly when you have a tone value and want to apply it to a subtree.
- **`<Backdrop background="…">`** — wraps `OnSurface` and derives the tone from a background prop. Use when the tone should be a consequence of a background the user picks (preset variant or custom color). Backdrop is the right choice for sections that paint themed backgrounds.

No other component is allowed to call `provideSurfaceToneContext` directly. Setting tone via `data-surface-tone` attributes or `.on-{tone}` classes manually is forbidden — only `OnSurface` is permitted to emit them.

### The "paint → declare" rule

If a component paints a visible background — themed preset, image, gradient, or custom color — it **must** declare a tone for its subtree, either by wrapping content in `<OnSurface>`, by going through `<Backdrop>`, or by composing a parent that does. A component that paints without declaring leaves descendants reading the wrong ancestor's tone.

If a component does **not** paint a background, it must **not** declare a tone. Inherit silently. Adding a redundant `<OnSurface>` wrapper on a transparent layout div breaks the inheritance chain and produces silent regressions when the ancestor's tone changes.

### Backdrop and custom colors

Backdrop's `background` prop accepts either a preset name (`'pale'` / `'solid'` / `'default'` / `'none'`) or any CSS color value. Pass the **resolved** color directly:

```vue
<Backdrop :background="resolvedBackground" />
```

Where `resolvedBackground` is the preset name when the author picked a preset, or the actual CSS color string (e.g. `'#ff0000'` or `'var(--accent-3)'`) when they picked custom. **Do not** pass the literal string `'custom'` to `background` — it produces invalid CSS in Backdrop's variant resolution.

`Backdrop` runs `colorToSurfaceTone(color)` on custom values to derive the tone automatically. This works for hex inputs; non-hex (`var(...)`, `light-dark(...)`, `rgb(...)`) currently short-circuit to `'light'` — if you need accurate classification for non-hex custom colors, resolve them to hex before passing.

## Consuming tone

Three patterns, in order of preference. Pick the lightest one that fits the job.

### Tier 1: Just consume the remapped tokens (preferred for most cases)

If the component only needs `color`, `fill`, `stroke`, etc. for its text and icons, consume the CSS variables `OnSurface` already remaps:

```css
@layer lui-components {
  .my-text { color: var(--font-color-body-text); }
}
```

**Nothing to opt into.** The cascade does the work. Most text/icon-rendering atoms (whether in upstream `@laioutr-core/ui-kit` or in your own module) should be Tier 1.

### Tier 2: Descendant selectors on `.on-{tone}` (preferred when Tier 1 isn't enough)

For tone-driven styling that goes beyond the remapped tokens — borders, opacity, modifier-shaped variants, conditional CSS — use ancestor selectors on `.on-dark` / `.on-light` / `.on-bright` from inside the component's own stylesheet:

```css
@layer lui-components {
  .swatch-chip          { border-color: var(--grey-7); }
  .on-dark   .swatch-chip { border-color: var(--text-always-white); }
  .on-bright .swatch-chip { border-color: var(--text-always-black); }
}
```

**No prop, no composable, no JS.** This is the canonical use of OnSurface's CSS contract — `.on-{tone}` is the public selector authors can hook into; the component reading it is using the system as designed.

This is **not** a `no-wrapper-css-overrides.md` violation. That rule forbids wrappers reaching INTO child component internals. A component reading its own ancestor's documented class is the inverse direction and is the cascade's intended use.

Pros over Tier 3: zero JS, no SSR/hydration concerns, naturally reactive, works through any nesting/portals (once Channel B is re-emitted at portal targets), no boilerplate per component.

### Tier 3: `surfaceTone?` prop + `useSurfaceTone()` (only when CSS can't do it)

Reach for the composable only when the tone affects **non-styling decisions**:

- Choosing between visually different icon SVGs
- Picking a different image source
- Selecting a named upstream component variant (`Button` choosing `'ghost-white'` vs `'ghost-black'` — class name differs, not just CSS within one class)
- Conditional sub-component rendering

When you do this, accept `surfaceTone?` as a prop AND use `useSurfaceTone(props)` — the composable already implements precedence (explicit prop > context > `'light'`).

```ts
const props = defineProps<{ surfaceTone?: SurfaceTone }>();
const tone = useSurfaceTone(props); // ComputedRef<SurfaceTone>
const buttonVariant = computed(() => tone.value === 'dark' ? 'ghost-white' : 'ghost-black');
```

**Always go through `useSurfaceTone(props)`.** Reading `props.surfaceTone` directly skips ancestor inheritance — the prop becomes the only source, and a parent's `<OnSurface>` won't reach the component.

Do **not** add `surfaceTone?` to atoms purely so a parent can override. The override pattern is to wrap in `<OnSurface tone="…">` at the call site — same pattern Radix Themes and Bootstrap document. Reserve the prop for atoms that genuinely need the JS branch.

## Components with multiple internal regions

A single component sometimes contains regions with different tones — e.g. an image card with text overlaid on the image (rendered against the image's tone) and a caption below the image (rendered against the ambient ancestor tone). **Use multiple `<OnSurface>` boundaries**, one per region with an independent tone. Don't try to make one tone serve both.

```vue
<template>
  <div class="image-card">
    <!-- Inner: independent tone subtree, driven by a region-specific prop -->
    <OnSurface :tone="imageTone" class="image-card__media">
      <img :src="image" alt="" />
      <h2 class="image-card__headline">{{ headline }}</h2>
    </OnSurface>

    <!-- Outer: inherits ambient tone, no wrapper -->
    <p class="image-card__caption">{{ caption }}</p>
  </div>
</template>
```

The headline reads `--font-color-heading` from the inner OnSurface's cascade. The caption reads the same variable but from the AMBIENT (ancestor) cascade. Each subtree resolves to the closest `OnSurface` ancestor — no manual coordination needed.

### Naming the region prop

When a component has multiple surface regions, **never** call the prop `surfaceTone` — it's ambiguous (which region?). Name it after the region:

- `imageTone` / `mediaTone` — when the inner is image-shaped
- `overlayTone` — when the inner is content overlaid on a media block
- `headerTone` / `headlineTone` — when the inner is a banded header region

This also leaves `surfaceTone` available as the canonical name for atoms that have a single tone region (Tier 3 above).

## Portal handling

Portaled content (Sheet, Popover, Tooltip, Dialog, DropdownMenu, ContextMenu, Lightbox, Select, InputAutocomplete, InputCombobox, HoverCard) inherits Channel A but may lose Channel B at the portal boundary depending on how the overlay is implemented upstream. If you observe a mixed-channel rendering bug (e.g. a swatch in a Popover triggered from a dark hero rendering against the wrong tone), opt into trigger-tone inheritance explicitly:

```vue
<!-- Inside a dark section -->
<Popover>
  <PopoverTrigger>...</PopoverTrigger>
  <PopoverContent>
    <OnSurface :tone="useSurfaceTone().value">
      <!-- Now this content honors the trigger's dark tone -->
      <slot />
    </OnSurface>
  </PopoverContent>
</Popover>
```

## Hard rules (forbidden patterns)

1. **No literal tone strings inside Section or Block templates.** A `tone="light|dark|bright"` literal in a Section or Block defeats the CMS author's choice. If a section or block needs a fixed tone, expose it as a schema field.

2. **No parallel naming axes.** `colorMode`, `textColor`, `contentColor`, `backgroundBrightness`, `nodesColorMode`, `isBackgroundDark` and similar synonyms are forbidden as new prop / schema names. Use `surfaceTone` (single-region) or a region-specific name (`imageTone`, etc.). Existing legacy axes are being migrated.

3. **No direct `props.surfaceTone` reads.** Always go through `useSurfaceTone(props)` so ancestor context flows when the prop is omitted. Reading `props.surfaceTone` directly silently breaks inheritance.

4. **Only `OnSurface` may set tone publicly.** `provideSurfaceToneContext` is internal to `OnSurface.vue`. Setting `data-surface-tone` or `.on-{tone}` manually on a `<div>` is forbidden — emit them via `<OnSurface>`.

5. **Paint → declare.** If a component paints a background, it must wrap content in `<OnSurface>` (or `<Backdrop>`). If it doesn't paint, it must not wrap.

6. **No CSS overrides of child component colors.** Per `no-wrapper-css-overrides.md`, a wrapper's `<style>` block must not target a child's BEM classes to hardcode colors. Such overrides defeat the OnSurface cascade. Express tone-conditional looks as variants on the child.

## Tones in detail

| Tone | Meaning | Foreground source |
|---|---|---|
| `light` | White or near-white surface (most pages, default) | Theme's natural text colors (light-mode tokens) |
| `dark` | Black or near-black surface (dark hero, footer, dark accent section) | `--text-always-white` family |
| `bright` | Vibrant, saturated color surface (brand accent, bold marketing block) | `--text-always-black` family |

`light` and `dark` flip between black and white text. `bright` is for surfaces with so much chroma that black text reads better than white regardless — pink, lime, royal blue, etc. Authors don't typically choose between dark and bright; the picker either uses the theme's preset map (`backgroundAwareBackdrop`) or runs `colorToSurfaceTone()` on a custom color.

When authoring a custom atom in your module, default to accepting the full `SurfaceTone` union. Narrow via `Exclude<SurfaceTone, 'bright'>` only when the design intent is "this component only ever sits in chrome / navigation / error-page contexts where bright surfaces are never used"; add a one-line comment naming the constraint when you do.

## Why this rule exists

Customer-facing CMS authors compose pages from sections and blocks, picking arbitrary backgrounds via Studio. Without an automatic foreground adaptation system, every block placed in a non-default section would need per-tone configuration, every theme change would require coordinated edits across hundreds of components, and accessibility (text contrast) would be the author's responsibility.

The Surface Tone system makes "leaf components adapt to their container" the platform's responsibility, not the author's. The two channels (Vue context + CSS variable cascade) cover both pure-CSS adaptation (most components, free) and JS-side branching (rare, when needed). The `OnSurface` / `Backdrop` boundary is the only place tone is set, so inheritance is always traceable. Paired tokens (`--on-{tone}-*`) make the foreground-per-tone vocabulary explicit and themeable.

The most common failure mode is a component that paints a background but doesn't declare a tone — descendants then read the wrong ancestor's cascade, producing dark-on-dark text or low-contrast icons. The "paint → declare" rule and the eventual lint check exist to catch this at authoring time rather than visual review.

## Related

- [`no-wrapper-css-overrides.md`](./no-wrapper-css-overrides.md) — the rule that paired tokens replace; wrappers must not hardcode colors targeting child BEM
- [`public-css-api.md`](./public-css-api.md) — broader public CSS contract; `data-surface-tone` and `.on-{tone}` are the documented selectors of OnSurface
