# CLAUDE.md - ui

This package contains commerce-specific organism components (Headers, Hero sections, Product components, Error pages) built on top of ui-kit primitives.

## Component Location

Components live in `src/runtime/components/`. These are higher-level compositions like `Error404Page.vue`, `Header.vue`, product grids, etc.

## Key Rules

1. **Compose ui-kit components** - Don't recreate primitives. Use Button, Icon, Input, etc. from ui-kit.

2. **Use LIcon for icons** (not Icon):

   ```vue
   <LIcon name="essentials/search" />
   ```

3. **Follow ui-kit token rules** - All styling must use CSS variables from `@laioutr-core/ui-tokens`. See [`../ui-kit/CLAUDE.md`](../ui-kit/CLAUDE.md) for details.

4. **Assets** live in `src/runtime/public/`. Reference via theme abstractions, not raw paths.

## Reference

Detailed rules in this folder (`rules/ui/`):

- `package-role.md` — what `ui` is for, how it relates to ui-kit and ui-app, interface requirements for Studio-configurable Sections/Blocks
- `z-ordering.md` — z-index token scale, section isolation behavior, when to opt out via `rendering: { isolate: false }`

For CSS-first responsive guidance, see [`../ui-kit/css-first-responsive.md`](../ui-kit/css-first-responsive.md) — the rule applies identically to the ui layer.

## Documentation

Component documentation lives at <https://docs.laioutr.io/laioutr-ui/>. The plugin also registers an MCP server (`laioutr-docs`) that exposes the same documentation to Claude at runtime.
