# Translations: `$tl` vs `useLocale()`

Two ways to translate in ui-kit-, ui-, and ui-app-layer components. Pick the one that fits the call site — mixing both in the same component is a smell.

## What each is

- **`$tl`** — Vue global property, injected by the `locale` plugin shipped with `@laioutr-core/ui-kit`. Available **only in templates**. Signature: `$tl(path, options?)`.
- **`useLocale()`** — composable from `@laioutr-core/ui-kit` returning `{ locale, lang, dir, code, t }`. Available in both `<script setup>` and templates. `t` is the same translator as `$tl`.

## The rule

**Prefer `$tl` in templates.** Reach for `useLocale()` only when you need one of the things `$tl` cannot give you:

1. You call `t(...)` from `<script setup>` (computed, watcher, event handler, function body) — `$tl` is not reachable there.
2. You need `locale.lang`, `locale.dir`, `locale.code`, or the `locale` ref itself (e.g. for `@vueuse/useMasonry` RTL handling, `ConfigProvider`, language-aware formatting).

If neither applies, drop `useLocale()` entirely and use `$tl` in the template.

## Decision flow

```
Is `t(...)` called in <script setup>?          → useLocale()
Is locale.lang / dir / code / locale used?     → useLocale()
Only t('...') used in <template>?              → $tl, no useLocale import
```

## Canonical patterns

### Template-only translation → `$tl`

```vue
<script setup lang="ts">
import Button from '../Button/Button.vue';
</script>

<template>
  <Button :aria-label="$tl('sliderNavigation.next')">
    <Icon name="arrows/arrow-right" />
  </Button>
</template>
```

No `useLocale` import, no `const { t } = ...`.

### Translation in script (computed / handler) → `useLocale()`

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { useLocale } from '../../composables/useLocale';

const locale = useLocale();

const tooltip = computed(() => locale.t('colorSwatch.tooltipSingle', { color }));
</script>

<template>
  <Tooltip :content="tooltip" />
</template>
```

If the same component also translates in the template, keep using `locale.t(...)` (or the destructured `t`) there for consistency — **do not mix `$tl` and `locale.t` in the same file**.

### Locale metadata (lang / dir / code) → `useLocale()`

```vue
<script setup lang="ts">
import { useLocale } from '#ui-kit/composables/useLocale';

const locale = useLocale();
const lang = locale.lang;
</script>

<template>
  <MasonryWall :rtl="locale.dir.value === 'rtl'">
    <slot />
  </MasonryWall>
</template>
```

## What not to do

- **Don't import `useLocale` just to expose `t` in a template.** That's exactly what `$tl` is for.
- **Don't mix `$tl` and `locale.t(...)` in the same component.** Pick one. If the script needs `t`, the template uses `locale.t` / the destructured `t` too.
- **Don't reach for `$tl` from a TypeScript helper file or a composable.** It lives on the component instance, not in module scope.
- **Don't destructure as `const { t: $tl } = useLocale()`** to fake the template shape — that defeats the point (the plugin-backed `$tl` is already there for free).

## Why this split exists

`$tl` is a zero-boilerplate shortcut for the 80% case (translate a label in the template). Keeping `useLocale()` for the script-side cases and for locale metadata avoids pulling a composable into files that don't reactively depend on anything from it, which in turn keeps imports honest and removes one unnecessary line at the top of dozens of simple components.
