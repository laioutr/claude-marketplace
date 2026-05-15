# Component Structure Rules

## File Organization

```
src/runtime/app/components/<ComponentName>/
├── <ComponentName>.vue          # Main component (required)
├── <ComponentName>.stories.ts   # Storybook stories (required)
└── types.ts                     # Only if types are shared or large (50+ lines)
```

## Type Exports

**Preferred:** Inline types in a separate non-setup script block:

```vue
<script lang="ts">
export interface ButtonProps {
  variant?: ButtonVariant;
  size?: ButtonSize;
}

export type ButtonVariant = 'primary' | 'secondary';
export type ButtonSize = 'small' | 'medium' | 'large';
</script>

<script setup lang="ts">
const props = withDefaults(defineProps<ButtonProps>(), {
  variant: 'primary',
  size: 'medium',
});
</script>
```

This allows: `import type { ButtonVariant } from '#ui-kit/Button/Button.vue'`

## Compound Components

For components with sub-components:

1. **Naming:** `<ParentComponent><SubComponentPart>`

   - `DropdownMenu` → `DropdownMenuItem`, `DropdownMenuSeparator`
   - `Dialog` → `DialogTitle`, `DialogContent`

2. **Usage:** Sub-components are only used inside their parent

3. **Context:** Parent can provide state and functions via `createContext` from reka-ui:

   ```typescript
   const [useMenuContext, provideMenuContext] = createContext<MenuContext>('Menu');
   ```

   reka-ui components also export `injectContext` functions (e.g., `injectAccordionRootContext`) that give access to internal state and methods. Use these in child components to access parent state for custom styling, extended functionality, or accessibility enhancements. Always check if the injected context exists before using it, as it may be `undefined` outside the component's scope. Prefer standard props and events when possible—use `injectContext` only for advanced composition scenarios.

## When to Create New Components

- **Create new:** Component is distinct with no similar existing component
- **Add variant:** Similar component exists, just needs a new variant prop value
- **Ask first:** If unclear whether to extend or create new
