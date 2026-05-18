# No Wrapper CSS Overrides of Child Components

A wrapping component must **never** target a child component's internal class names from its own `<style>` block to change how the child renders. Child component class names are an implementation detail, not a public API.

This is one face of the broader [Public CSS API](./public-css-api.md) — the prop-not-selector mechanism for cross-component styling.

## The rule

Inside a wrapper's CSS, selectors must not reach into a child component's BEM classes, data attributes, or structural elements. If the wrapper needs the child to look or behave differently, that difference must be expressed as a **prop** on the child — not as a style override from the outside.

Forbidden pattern:

```vue
<!-- PopUpNewsletter.vue -->
<template>
  <div class="popup-newsletter__form">
    <EmailInputForm />
  </div>
</template>

<style>
/* ❌ Reaching into the child's internals */
.popup-newsletter__form .newsletter-form__box--input {
  padding: 0;
  background: transparent;
}
.popup-newsletter__form .newsletter-form__input-wrapper {
  flex-direction: column;
}
</style>
```

Required pattern:

```vue
<!-- EmailInputForm.vue (child) adds a prop -->
<script setup lang="ts">
defineProps<{
  variant?: 'boxed' | 'plain';
  // ...
}>();
</script>
<template>
  <form :class="['newsletter-form', { 'newsletter-form--plain': variant === 'plain' }]">
    <!-- ... -->
  </form>
</template>
<style>
@layer lui-components {
  .newsletter-form--plain {
    /* variant styles live with the component that owns them */
  }
}
</style>
```

```vue
<!-- PopUpNewsletter.vue (wrapper) uses the prop -->
<template>
  <EmailInputForm variant="plain" />
</template>
```

## Why

1. **Encapsulation.** A child component's class names are its internals. When the wrapper depends on them, any refactor inside the child — a BEM rename, a DOM restructure, a prop that changes which classes are emitted — silently breaks the wrapper. The compiler cannot catch it; the lint cannot catch it; only a visual regression will.
2. **Action-at-a-distance CSS.** `findReferences` on a component name finds its callers. It does not find `.newsletter-form__box--input` selectors in unrelated files across packages. These overrides are invisible to every refactoring tool.
3. **Cross-package coupling.** When your module overrides a class name from `@laioutr-core/ui-kit` or `@laioutr-core/ui`, an upstream release changes the class shape without any type-level signal — your override silently breaks. The relationship between your module and upstream packages only works if each side's surface is its props and CSS variables, not its DOM.
4. **Lost reusability.** A styling need expressed as an override is invisible to the next consumer who has the same need — they re-derive it from scratch. A styling need expressed as a prop is discoverable, typed, documented in Storybook, and reused.
5. **Theme and token drift.** Wrappers tend to hardcode overrides that bypass the child's token pipeline (e.g. `background: transparent` instead of letting the child's theme tokens resolve). Variant props keep everything on the token scale.

## When to apply

- Any time a wrapper's `<style>` contains a selector descending into a child's BEM class (`.wrapper .child__part`, `.wrapper :deep(.child__part)`, etc.).
- Any time a wrapper needs the child to change padding, background, layout direction, min-height, or any other visual property beyond what the child's props expose.

## What to do when you hit this

1. **Name the difference.** What is the wrapper asking the child to be? (`plain`, `inline`, `compact`, `frameless`, etc.) That name is your variant.
2. **Promote the override to a prop.** Add `variant?: 'boxed' | 'plain'` (or similar) on the child. Emit a BEM modifier class (`block--plain`) on the child's root.
3. **Move the CSS into the child.** The override rules now live under the modifier selector in the child's own `<style>` block, using the same tokens and layer as the rest of the child's styles.
4. **Update the wrapper to pass the prop.** Delete the override CSS and — if the wrapper div existed only as a CSS scope anchor — delete the wrapper div too.
5. **Add a Storybook story** for the new variant in the child's stories file so the variant is visible and visually regressed.
6. **Add changesets** for every package whose exports change as a result.

## Exceptions

- **Portaled content you own.** If the child is a tiny leaf primitive (Icon, Text) and the parent needs to apply layout context to it (width, alignment) in a way that is purely a parent-layout concern, a direct class on the parent's wrapper div that sets flex/grid rules on `> *` is fine. The line is: **parent-layout rules on the parent's own wrapper** = OK, **changing the child's internals** = not OK.
- **`:deep()` with a tracked TODO.** If you genuinely cannot avoid a `:deep()` override (e.g. waiting for a prop to be added and the deadline is today), include a `// TODO:` comment naming the child prop that should replace it and a link to the ticket. Clean it up in the same sprint.

## Why the wrapper is tempting

Wrappers often feel like the "right" place because the override is visually scoped to one context. But "visually scoped" is the problem, not the solution — the child doesn't know the wrapper exists, and the next time someone touches the child they have no way to know the wrapper depends on the exact DOM and class shape they see today. Variants are the mechanism we already have for context-specific looks. Use them.

## Related

- [`public-css-api.md`](./public-css-api.md) — the four override surfaces and the rules that govern the public CSS contract
