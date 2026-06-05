# No Hardcoded UI Strings

User-visible text in a component must never be a hardcoded literal — not English (`'No results'`, `'Subscribe'`), not German (`'Spare {…}%'`, `'UVP'`, `'NEU'`). Applies to any presentational component your module ships, and to the upstream `@laioutr-core/ui-kit` and `@laioutr-core/ui` patterns you mirror.

## The rule

A text prop's default is `undefined`. The surface resolves the fallback via `$tl(...)` in the template (or `useLocale().t(...)` in script):

```vue
<script setup lang="ts">
defineProps<{ placeholder?: string }>();   // default undefined — no English default
</script>

<template>
  <!-- prop wins when provided, otherwise the locale key -->
  <input :placeholder="placeholder ?? $tl('select.placeholder')" />
</template>
```

```ts
// ✘ ships an untranslatable default
defineProps<{ emptyText?: string }>();      // = 'No results'
```

Use `$tl(...)` in templates and `useLocale().t(...)` in script. Add the key to your module's locale files (`locale/{en,de}.ts` or equivalent — and the consuming package's locale) when introducing a fallback.

## Related

- [`money-currency-code.md`](./money-currency-code.md) — currency is an ISO code, never a localized label
