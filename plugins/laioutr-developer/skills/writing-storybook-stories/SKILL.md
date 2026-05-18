---
name: writing-storybook-stories
description: Use when authoring or modifying a Vue Storybook `.stories.ts` file in a Laioutr module — naming the meta title and exports, deciding the variant sequence, picking layout variants by Figma frame, attaching `parameters.design.url`, choosing args over render, handling theming under multiple themes, or removing legacy `ViewportXS` / `ViewportS` / `ViewportSM` / `ViewportLG` viewport-only exports during a refactor.
---

# Writing Storybook Stories

## Overview

Stories form a **state / variant matrix driven by props**, not a viewport matrix. Names follow Figma frame labels, the first export is always `Default`, and every story links back to its Figma source. The viewport toolbar covers responsive behavior — don't shadow it with `Viewport*` exports.

## Quick Reference

| Element | Rule |
| --- | --- |
| Meta title | `'Components/<ComponentName>'`; use Atomic-Design categories if the module groups by layer (`'Atoms/<Name>'`, `'Organisms/<Name>'`, `'Sections/<Name>'`) |
| First export | Always `Default` — the primary / standard Figma layout |
| Layout variants | PascalCase matching Figma frame labels: `Compact`, `Stacked`, `Centered`, `Split`, `Grid2Col`, `Grid3Col` |
| State variants on the Default layout | Bare state name, no `Default` prefix — `Loading`, `Disabled`, `WithIcon`, `OnSale` |
| Layout-specific modifier variants | `<LayoutName>With<Modifier>` — `CompactWithBadge`, `SideBySideWithCTA` |
| Figma link | Every story sets per-story `parameters.design` with `{ type: 'figma', url }` pointing at the specific Figma frame (see skeleton below) |
| Args vs render | Prefer declarative `args` over custom `render()`; only use `render()` when the component needs wrapping context |

## File skeleton

A minimal `.stories.ts` — import block, `Meta` / `StoryObj` generics, `type Story` alias, first export `Default` with `parameters.design`:

```ts
import type { Meta, StoryObj } from '@storybook/vue3';
import ProductCard from './ProductCard.vue';

const meta: Meta<typeof ProductCard> = {
  title: 'Components/ProductCard',
  component: ProductCard,
};

export default meta;
type Story = StoryObj<typeof ProductCard>;

export const Default: Story = {
  parameters: {
    design: {
      type: 'figma',
      url: 'https://figma.com/design/.../node-id=5-100',
    },
  },
};
```

`parameters.design` uses the `@storybook/addon-designs` shape — `type: 'figma'` is required alongside `url`. For stories that share a Figma frame with `Default` (e.g. state variants on the default layout), repeat the same `url` per story rather than hoisting onto `meta` — the addon resolves per-story values, and a shared meta-level URL silently shadows distinct frames on layout variants.

### When `render()` is needed

For wrapping context (sidebar container, narrow column, ambient backdrop), keep props in `args` so the Controls panel still works, and use `render()` only for the wrapping DOM:

```ts
export const Compact: Story = {
  args: { /* same prop shape as other stories */ },
  render: (args) => ({
    components: { ProductCard },
    setup: () => ({ args }),
    template: `
      <div style="max-width: 280px;">
        <ProductCard v-bind="args" />
      </div>
    `,
  }),
};
```

Hardcoding props inside the template (instead of `v-bind="args"`) breaks the Controls panel for that story.

## No per-viewport stories

```ts
// ❌ Don't do this — Storybook's viewport toolbar already covers this
export const ViewportXS: Story = {
  name: 'Viewport / xs (360)',
  parameters: { viewport: { defaultViewport: 'xs' } },
};
export const ViewportS: Story  = { /* ... */ };
export const ViewportSM: Story = { /* ... */ };
export const ViewportLG: Story = { /* ... */ };
```

```ts
// ✅ State and variant stories only
export const Default: Story = {};
export const Disabled: Story = { args: { disabled: true } };
export const WithIcon: Story = { args: { icon: 'essentials/search' } };
export const Loading: Story = { args: { isLoading: true } };
```

**Exception:** when a component renders a structurally different DOM per breakpoint that resizing can't reveal (drawer on mobile, inline menu on desktop), write **one** dedicated story named after the layout (`MobileDrawer`, `DesktopInline`) — never after the viewport width.

**On contact:** when you touch a stories file that still contains `ViewportXS` / `ViewportS` / `ViewportSM` / `ViewportLG` exports, remove them as part of the change. Don't leave them behind "for now".

## Theming

If the module supports multiple themes:

- Check your module's Storybook preview config (typically `.storybook/preview.ts` at the module root) to confirm which theme renders by default. The default may not be the same theme Figma was exported against — flag mismatches before authoring stories.
- For every component, verify that token-driven props (e.g. `colorScheme`) and CSS variable resolution work under every supported theme. Flag any token references that only work under one theme.

If the module supports only one theme, ignore this section.

## Refactor workflow

When you touch an existing `.stories.ts` as part of a component refactor:

1. **Viewport check** — compare Figma frames for mobile / tablet / desktop against the actual breakpoints used in the component (Tailwind `sm:` / `md:` / `lg:` / `xl:` or token-driven equivalents). Flag any mismatches.
2. **Rename, if requested** — touch all in one pass: folder name, file name (`.vue`), component name in `<script setup name="...">`, exports in `index.ts` and barrels, imports in any Sections and Blocks, and any string references (`defineSection({ component: '...' })`, registries, schemas). Use LSP `findReferences` before any delete or rename — grep is a fallback.
3. **Stories** — delete the old `*.stories.ts`, write new ones using the naming rules above.

If the module is migrating off `dark` / `light`-mode props in favor of the `surface-tone` mechanism, also strip: `dark:` Tailwind classes, conditionals on `colorMode` / `isDarkMode`, CSS variables scoped to `.dark`, imports from `nuxt-color-mode` or similar, and props that exist only to toggle theme.

## Releases

If the module uses changesets (or any other versioned release flow), gather all component renames and refactors from a session into a **single release entry** rather than one per component — they're conceptually one refactor.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| First export is `Primary`, `Base`, `Standard`, etc. | Always `Default` |
| Story name copied from Figma in Title Case (`"Compact With Badge"`) | PascalCase the export (`CompactWithBadge`); Storybook auto-formats the label |
| `Viewport*` exports re-introduced "to make it obvious" | Use the viewport toolbar; only keep a viewport-named story if the DOM differs |
| `render()` used to set basic args | Use `args`; reserve `render()` for wrapping context |
| Icon name typed from memory (`'search'`) | Always use the qualified path (`'essentials/search'`); see [`story-icons-must-exist.md`](../laioutr-platform/references/story-icons-must-exist.md) in the sibling `laioutr-platform` skill for verification |
| Missing `parameters.design.url` | Every story links to a Figma frame |
