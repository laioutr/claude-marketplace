# Component State Contract — v-model, Validation, Field Context

Use one shape for v-model, validation, and field context across the components your module ships. Applies to any Vue component you build, and to the upstream `@laioutr-core/ui-kit`, `@laioutr-core/ui`, and `@laioutr-core/ui-app` patterns you mirror. For the reka-ui *mechanics* of typed re-emits, see [`reka-ui-wrapper-patterns.md`](./reka-ui-wrapper-patterns.md).

## v-model

- **Form inputs**: always `modelValue` / `update:modelValue` via `defineModel<T>()`. No `update:checked` (Checkbox / Switch / InputCheckbox / Filter*), no `update:searchTerm` (InputSearch).
- **Overlays**: `open` / `update:open` (Reka-aligned). No `update:isOpen` (AlertDialog, VariantOffCanvas, MediaGallery, MenuMegaMenu, …).
- **A component whose sole purpose is the value**: also `modelValue`.
- **A component with several state channels**: decide the secondary channel per component (e.g. `FilterOffCanvas` = `open` + `selected: SelectedFilters`). The primary channel still follows the rules above.

```ts
// ✅ form input
const model = defineModel<string>();          // modelValue / update:modelValue
// ✅ overlay
const open = defineModel<boolean>('open');    // open / update:open
```

## Validation

Replace every overloaded `error: boolean | string` with two props:

```ts
defineProps<{
  invalid?: boolean;      // drives aria-invalid
  errorMessage?: string;  // the message to show
}>();
```

`is*` does not apply here — `invalid` is a state the component is in (see [`boolean-prop-naming.md`](./boolean-prop-naming.md)).

## Field context — one shape

`Field` provides, and every form input injects via `injectFieldContext` and reads the keys it cares about:

```ts
{ id, disabled, readonly, required, invalid, errorMessage }
```

No partial-honor models: an input that wraps in `<Field disabled required>` must actually consume `disabled` and `required` (and put `aria-required` on the control) rather than silently dropping them.

## Related

- [`boolean-prop-naming.md`](./boolean-prop-naming.md) · [`prop-naming-vocabulary.md`](./prop-naming-vocabulary.md)
- [`reka-ui-wrapper-patterns.md`](./reka-ui-wrapper-patterns.md) — typed emits / props export mechanics
