# reka-ui Wrapper Patterns

Concrete patterns for wrapping reka-ui primitives into ui-kit atoms. Complements `reka-ui.md` (which covers accessibility + styling basics).

## Split vs. flat composition

Decide early whether consumers need to arrange the sub-parts themselves.

**Split into sub-components** when the consumer places parts at different positions in their template:

```
Tabs / TabsList / TabsTrigger / TabsContent
Accordion / AccordionItem / AccordionTrigger / AccordionContent
Popover / PopoverTrigger / PopoverContent
```

Each sub-component is its own `.vue` file, exported with matching filename, prop interface named `<SubComponent>Props`.

**Keep flat** when the internal arrangement is fixed and the consumer only configures via props:

```
Slider            — composes SliderRoot + SliderTrack + SliderRange + SliderThumb internally
PinInput          — v-for of length prop renders N PinInputInput children
BadgePromotion    — single visual, no consumer-level composition
```

Rule of thumb: if you'd always render the same sub-tree in the same order, flatten. If the consumer legitimately needs to interleave content or reorder parts, split.

## as-child for triggers

reka-ui triggers (`PopoverTrigger`, `CollapsibleTrigger`, `HoverCardTrigger`, etc.) should accept an `as-child` slot so consumers render their own Button, Link, or Icon without a wrapping `<button>`:

```vue
<template>
  <RekaPopoverTrigger as-child>
    <slot />
  </RekaPopoverTrigger>
</template>
```

This keeps the trigger chrome under the consumer's control and avoids double-button nesting.

## Portal + Content separation

Overlay primitives (Popover, HoverCard, ContextMenu, Combobox) require a Portal around the floating content. Wrap it inside the `<Content>` component, not the consumer's template:

```vue
<template>
  <RekaPopoverPortal>
    <RekaPopoverContent class="popover__content" :side="side" :align="align">
      <slot />
    </RekaPopoverContent>
  </RekaPopoverPortal>
</template>
```

Never ask the consumer to remember the portal — it is part of the primitive's contract.

## Typed Props export

Export the Props interface from a separate non-setup `<script>` block so consumers can import the type:

```vue
<script lang="ts">
export interface PopoverContentProps {
  side?: 'top' | 'right' | 'bottom' | 'left';
  align?: 'start' | 'center' | 'end';
  sideOffset?: number;
  alignOffset?: number;
  width?: string;
}
</script>

<script setup lang="ts">
withDefaults(defineProps<PopoverContentProps>(), {
  side: 'bottom',
  align: 'center',
  sideOffset: 8,
  alignOffset: 0,
  width: 'auto',
});
</script>
```

## Typed emits and AcceptableValue

reka-ui's `AcceptableValue` unions include `null` for "no selection". When forwarding `@update:value`, cast explicitly:

```vue
@update:model-value="$emit('update:modelValue', ($event ?? []) as (string | number)[])"
```

Don't rely on `AcceptableValue` leaking into consumer code. Narrow at the wrapper boundary.

## Styling via data attributes

Use reka-ui's `data-*` attributes for state styling — never style reka-ui's internal class names:

```css
.popover__content[data-state='open'] {
  /* open styles */
}
.toggle[data-state='on'] {
  /* pressed */
}
.menubar__item[data-highlighted] {
  /* hover/keyboard focus */
}
.tabs__trigger[data-disabled] {
  /* disabled */
}
```

## Import order inside wrappers

ESLint sort-imports reorders these; write them in the expected order up-front:

1. External libs (`reka-ui`, `vue`)
2. Aliased Reka imports (`Toggle as RekaToggle`) **after** the named imports from the same module

If lint autofixes this on commit, don't revert — the autofix is correct.
