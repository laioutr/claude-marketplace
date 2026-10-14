# Shared Field Option Lists

Canonical reference: [Shared field factories — Laioutr docs](https://docs.laioutr.com/apps/app-development/guides/shared-field-factories) (Pattern 2).

Exported `*Options` arrays in `shared-fields/` must preserve literal types. Wrap them in a `defineSelectOptions` helper. No Laioutr package exports one — write it once in your own `shared-fields/`, as the upstream apps do:

```ts
// shared-fields/defineSelectOptions.ts
import type { FieldDefinitionSelectOption } from '@laioutr-core/core-types/fields';

export const defineSelectOptions = <
  const T extends readonly [FieldDefinitionSelectOption, ...FieldDefinitionSelectOption[]],
>(
  options: T,
): [...T] => [...options];
```

```ts
// shared-fields/buttonVariant.ts
import { defineSelectOptions } from './defineSelectOptions';

export const buttonVariantOptions = defineSelectOptions([
  { label: 'Primary', value: 'primary' },
  { label: 'Secondary', value: 'secondary' },
]);
```

This enforces the non-empty `FieldDefinitionSelectOption` tuple shape and preserves every `value` as a literal. It replaces the older `as const satisfies [FieldDefinitionSelectOption, ...FieldDefinitionSelectOption[]]` incantation — use the helper for any new list and migrate existing lists when you touch them. Returning a fresh array also means a field that mutates its `options` can't corrupt the shared export.

Plain `as const` (without `satisfies`) is only acceptable for private lists that don't need the `FieldDefinitionSelectOption` shape check.

**Never annotate with the type directly** — it widens every `value` to `string`:

```ts
// ❌ widens
export const myOptions: [FieldDefinitionSelectOption, ...FieldDefinitionSelectOption[]] = [...];
```

**Why:** `definitionToProps` derives the prop type from `options[i].value`. Widening turns it into `string`, killing narrow typing downstream and forcing unsafe casts.

Options written inline inside a field in `defineSection` / `defineBlock` don't need any of this — the define helpers already preserve literals.
