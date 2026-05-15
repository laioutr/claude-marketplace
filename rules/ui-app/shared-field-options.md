# Shared Field Option Lists

Exported `*Options` arrays in `shared-fields/` must preserve literal types. Use the `defineSelectOptions` helper:

```ts
import { defineSelectOptions } from './defineFieldset';

export const myOptions = defineSelectOptions([
  { label: 'Foo', value: 'foo' },
  { label: 'Bar', value: 'bar' },
]);
```

This enforces the non-empty `FieldDefinitionSelectOption` tuple shape and preserves every `value` as a literal. It's the replacement for the older `as const satisfies [FieldDefinitionSelectOption, ...FieldDefinitionSelectOption[]]` incantation — use the helper for any new list and migrate existing lists when you touch them.

Plain `as const` (without `satisfies`) is only acceptable for private lists that don't need the `FieldDefinitionSelectOption` shape check.

**Never annotate with the type directly** — it widens every `value` to `string`:

```ts
// ❌ widens
export const myOptions: [FieldDefinitionSelectOption, ...FieldDefinitionSelectOption[]] = [...];
```

**Why:** `definitionToProps` derives the prop type from `options[i].value`. Widening turns it into `string`, killing narrow typing downstream and forcing unsafe casts.

Inline options inside `defineField(...)` / `defineFieldset(...)` don't need `as const` — the `<const T>` generic already preserves literals.
