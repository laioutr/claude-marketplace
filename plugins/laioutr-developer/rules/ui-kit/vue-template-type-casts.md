# Vue Template Type Casts

Vue's template parser interprets `|` in a binding expression as the **deprecated filter syntax** (`value | filter`). A TypeScript union cast like `as string | undefined` inside a `:prop="..."` binding triggers `vue/no-deprecated-filter` and blocks commits via pre-commit lint.

## The trap

```vue
<!-- ❌ Fails with vue/no-deprecated-filter -->
<PinInputRoot :model-value="props.modelValue as string[] | undefined" />
```

The parser sees `modelValue as string[]` | `undefined` and interprets the `|` as a pipe-to-filter operator.

## The fix

Move the cast into `<script setup>` as a `computed` or `ref`, then reference the bound name in the template:

```vue
<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<PinInputProps>();

const boundModelValue = computed(() => props.modelValue as string[] | undefined);
</script>

<template>
  <PinInputRoot :model-value="boundModelValue" />
</template>
```

## When it's safe

Single-type casts (no union) do not trigger the rule:

```vue
<!-- ✅ OK — no | character -->
<Foo :value="x as string" />
```

It only breaks when the cast includes a TypeScript union.

## Why not just suppress?

The deprecation is real — Vue 3 removed filters entirely. A passing suppression hides the fact that the parser is misreading the expression, which can produce subtle runtime issues (the part before `|` is evaluated, the part after is looked up as a method). Keep the rule on and move casts into the script block.

## General guidance

Templates are for rendering, not for type gymnastics. Any expression more complex than a property access or simple boolean should live in a `computed` — this rule is a specific instance of that broader principle.
