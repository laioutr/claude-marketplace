# ui-kit

Atomic Vue 3 component library (Nuxt 3 module). Foundation of the design system.

## Universal Rules

### Structure

1. Components: `src/runtime/app/components/<Name>/<Name>.vue`
2. Stories required: `<Name>.stories.ts`
3. Types inline in `<script lang="ts">` block (not setup) for exports
4. Separate `types.ts` only if shared or >50 lines
5. Sub-components only used inside parent
6. Sub-component naming: `<Parent><Part>` (e.g., `DropdownMenuItem`)

### Code Style

7. Use `<script setup lang="ts">` for component logic
8. Use unscoped `<style>` blocks
9. Follow BEM: `.block`, `.block__element`, `.block--modifier`
10. Use `createContext` from reka-ui for parent-child state

### Tokens

11. Never hardcode colors, sizes, or spacing
12. Always use CSS variables from ui-tokens
13. Exception: animation/transition timings can be hardcoded
14. Cover all states: default, hovered, pressed, focus, disabled

### Dependencies

15. Prefer reka-ui components as a basis when available
16. Apply BEM classes to reka-ui components
17. Use reka-ui data attributes for state styling

### Quality

18. Follow WCAG 2.1 accessibility guidelines
19. Use reka-ui primitives for keyboard nav, ARIA, focus
20. Visual regression via Storybook + Chromatic
21. Unit tests only for complex logic

### Decisions

22. If unsure: new component vs variant → ask first
23. If unsure: extend existing vs build new → ask first

### Future Direction

- Moving from manual imports to auto-imports with `L` prefix
- `Text` component will be deprecated in favor of CSS classes
- Props naming revision for consistency across components
- Add index.ts file to each component directory so components and types can be imported as `import { DropdownMenu, DropdownMenuTrigger, DropdownProps } from '#ui-kit/DropdownMenu'`

## Reference

Detailed patterns in this folder (`rules/ui-kit/`):

- `package-role.md` - What ui-kit is for, how it relates to ui and ui-app, atom interface requirements
- `component-structure.md` - File organization, types, compound components
- `styling.md` - Tokens, BEM, layers, breakpoints
- `reka-ui.md` - Integration patterns, accessibility, context access
- `reka-ui-wrapper-patterns.md` - Concrete wrapper patterns: split vs flat, as-child, Portal, typed Props/emits
- `no-reka-ui-type-leakage.md` - reka-ui types must not appear on any ui-kit component's public surface (props, emits, slots, barrel re-exports)
- `no-outer-chrome-in-primitives.md` - ui-kit primitives must equal their visible footprint; chrome is allowed only on frame components (Backdrop, Card, Sheet, …) and molecules-with-built-in-frame (LoadMore, Accordion); when consumers override, ask Figma → variant → strip in that order
- `theming.md` - Themes, icons, images, background brightness, translations
- `vue-template-type-casts.md` - Avoiding `vue/no-deprecated-filter` from union casts in templates
- `todo-figma-marker.md` - Convention for marking provisional design decisions
- `storybook-atoms-categorization.md` - Atomic Design categories for ui-kit Storybook titles
- `css-first-responsive.md` - Prefer CSS media queries over `useBreakpoints` / `useIsMobile` for anything in the SSR tree
