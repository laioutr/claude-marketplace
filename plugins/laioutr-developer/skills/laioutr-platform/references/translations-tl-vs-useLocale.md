# Translate With `$tl` in Templates; Reach for `useLocale()` Only From Script

In the components, Sections and Blocks your module ships, translate in the template with `$tl(...)`. Import `useLocale()` only when script code calls the translator or reads locale metadata (`dir`, `code`, `lang`). Never mix both in one component.

## What each is

- **`$tl`** — Vue global property, injected by the `locale` plugin `@laioutr-core/ui-kit` installs. Available **only in templates**. Signature: `$tl(path, options?)`; `{name}` placeholders in the message are filled from `options`.
- **`useLocale()`** — ui-kit composable, auto-imported (explicit form: `import { useLocale } from '#imports'`). Returns `{ locale, lang, dir, code, t }`; `lang`, `dir` and `code` are refs. `t` is the same translator as `$tl`.

## Decision flow

```
Is t(...) called in <script setup>?          → useLocale()
Is locale.dir / code / lang / locale used?   → useLocale()
Only translating in <template>?              → $tl, no useLocale import
```

## Canonical patterns

### ✅ Template-only translation → `$tl`

```vue
<script setup lang="ts">
const emit = defineEmits<{ next: [] }>();
</script>

<template>
  <LIconButton icon="arrows/arrow-right" :label="$tl('sliderNavigation.next')" @click="emit('next')" />
</template>
```

No `useLocale` import, no `const { t } = …`.

### ✅ Translation in script (computed, watcher, handler) → `useLocale()`

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { useLocale } from '#imports';

const props = defineProps<{ color: string }>();
const locale = useLocale();

const tooltip = computed(() => locale.t('swatchChip.tooltipSingle', { color: props.color }));
</script>

<template>
  <LTooltip :label="tooltip">
    <span class="color-dot" :style="{ background: color }" />
  </LTooltip>
</template>
```

If this component also translates in its template, use `locale.t(...)` there too — not `$tl`.

### ✅ Locale metadata → `useLocale()`

```vue
<script setup lang="ts">
import { useLocale } from '#imports';

const locale = useLocale();
</script>

<template>
  <!-- locale is a plain object, so its refs are not auto-unwrapped in the template -->
  <div class="news-ticker" :dir="locale.dir.value">
    <slot />
  </div>
</template>
```

The same applies when a third-party component needs an RTL flag (`:rtl="locale.dir.value === 'rtl'"`).

## Forbidden

```ts
// ✘ importing useLocale only to expose t to the template — that is what $tl is for
const { t } = useLocale();

// ✘ faking the template shape — the plugin-backed $tl is already there
const { t: $tl } = useLocale();
```

- ✘ `$tl` and `locale.t(...)` in the same file. Pick one: if the script needs `t`, the template uses it too.
- ✘ `$tl` in a TypeScript helper or a composable. It lives on the component instance, not in module scope — pass the translated string in, or call `useLocale()` inside the composable.

## Why

1. **`$tl` covers the common case for free.** Most components only translate a label in the template. The plugin already provides `$tl` on every component, so there is nothing to import.
2. **Imports stay honest.** A `useLocale()` call in a file that never reads the locale from script is a dependency with no purpose. It also adds one line to dozens of simple components.
3. **Only one of them reaches script.** `$tl` is a template global, so `computed`s, watchers and handlers need `useLocale().t`. That is the line between the two, not taste.
4. **One translator, one spelling per file.** `locale.t` and `$tl` resolve the same keys against the same locale. Mixing them gives a reader two names for one call.

## Related

- [`no-hardcoded-ui-strings.md`](./no-hardcoded-ui-strings.md) — why the fallback text is a locale key in the first place
- [`reactive-value-args.md`](./reactive-value-args.md) — passing translated text to `openDialog` / `addToast` so it follows a locale switch
