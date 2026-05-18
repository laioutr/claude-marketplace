# Vue Component File Structure

Conventions for any Vue component your module authors — atom-style primitives, organism-style compositions, or section/block templates.

## File organization

```
src/runtime/components/<ComponentName>/
├── <ComponentName>.vue          # Main component (required)
├── <ComponentName>.stories.ts   # Storybook stories (when your module ships stories)
└── types.ts                     # Only if types are shared or large (50+ lines)
```

The exact directory may differ (e.g. `src/runtime/app/section/` for Sections, `src/runtime/app/block/` for Blocks); the one-folder-per-component convention applies everywhere.

## Type exports

Prefer inline types in a separate non-setup `<script>` block:

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

This allows consumers to import the type via the file's auto-import alias.

See [`vue-script-imports-single-block.md`](./vue-script-imports-single-block.md) for the constraint on where imports live across the two script blocks.

## Compound components

For components with sub-components:

1. **Naming:** `<ParentComponent><SubComponentPart>` — `DropdownMenu` → `DropdownMenuItem`, `DropdownMenuSeparator`; `Dialog` → `DialogTitle`, `DialogContent`. Only use this pattern when the sub-component is genuinely owned by one parent (see [`parent-prefix-naming.md`](./parent-prefix-naming.md)).

2. **Usage:** Sub-components are only used inside their parent.

3. **Context:** Parent provides state and functions via `createContext` from reka-ui:

   ```typescript
   const [useMenuContext, provideMenuContext] = createContext<MenuContext>('Menu');
   ```

   reka-ui components also export `injectContext` functions (e.g. `injectAccordionRootContext`) that give access to internal state and methods. Use these in child components to access parent state for custom styling, extended functionality, or accessibility enhancements. Always check if the injected context exists before using it, as it may be `undefined` outside the component's scope. Prefer standard props and events when possible — use `injectContext` only for advanced composition scenarios.

## When to create a new component vs add a variant

- **Create new:** the component is distinct, with no similar existing one in your module or upstream.
- **Add a variant prop:** a similar component exists and only needs another `variant` value.
- **Ask first:** when the choice is unclear.

For upstream gaps specifically (no matching `@laioutr-core/ui-kit` or `@laioutr-core/ui` component), follow the resolution ladder in [`three-layer-architecture.md`](./three-layer-architecture.md).
