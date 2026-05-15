# Storybook Stories — Global Rules

Conventions for `.stories.ts` files accompanying Vue components in any UI layer of your Nuxt module (ui-kit-, ui-, or ui-app-layer).

## Story naming

- **Meta title:** `'Components/<ComponentName>'` (use Atomic-Design categories if your module groups by layer — e.g. `'Atoms/<Name>'`, `'Organisms/<Name>'`, `'Sections/<Name>'`).
- **First export:** always `Default` — the primary / standard Figma layout.
- **Layout variants:** PascalCase names matching the Figma frame labels (e.g. `Compact`, `Stacked`, `Centered`, `Split`, `Grid2Col`, `Grid3Col`).
- **Modifier variants:** `<LayoutName>With<Modifier>` (e.g. `CompactWithBadge`).
- **Viewports:** do **not** create per-viewport stories. Stories form a state/variant matrix driven by props; use Storybook's viewport toolbar to resize. Only add a dedicated story if Figma shows a structurally different DOM per breakpoint that isn't CSS-solvable, and name it after the layout (not the viewport width).
- **Figma link:** every story should set `parameters.design.url` to the specific Figma layout.
- **Args over render:** prefer declarative `args` over custom `render()` functions unless the component requires wrapping context.

## Theming

If your module supports multiple themes:

- Check your Storybook config (typically `playground/.storybook/preview.ts`) to confirm which theme renders by default. The default may not be the same theme Figma was exported against — flag mismatches before authoring stories.
- For every component, verify that token-driven props (e.g. `colorScheme`) and CSS variable resolution work correctly under every supported theme. Flag any token references that only work under one theme.

If your module supports only one theme, ignore this section.

## When refactoring an existing component

When you touch an existing `.stories.ts` as part of a component refactor, the following workflow keeps the change reviewable:

1. **Viewport check** — compare the Figma frames for mobile / tablet / desktop against the actual breakpoints used in the component (Tailwind `sm:` / `md:` / `lg:` / `xl:` or your token-driven equivalents). Flag any mismatches.
2. **Rename, if requested** — touch all in one pass: folder name, file name (`.vue`), component name in `<script setup name="...">`, exports in `index.ts` and barrels, imports in any ui-app-layer Sections and Blocks, and any string references (`defineSection({ component: '...' })`, registries, schemas). Use LSP `findReferences` before any delete or rename — grep is a fallback.
3. **Stories** — delete the old `*.stories.ts`, write new ones using the naming rules above.

If your module is migrating off of dark/light-mode props in favor of the `surface-tone` mechanism, also strip: `dark:` Tailwind classes, conditionals on `colorMode` / `isDarkMode`, CSS variables scoped to `.dark`, imports from `nuxt-color-mode` or similar, and props that exist only to toggle theme.

## Releases

If your module uses changesets or any other versioned release flow, gather all component renames and refactors from a session into a **single release entry** rather than one per component — they're conceptually one refactor.
