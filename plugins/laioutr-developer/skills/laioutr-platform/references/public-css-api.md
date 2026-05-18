# Public CSS API

Every component your module publishes ships its CSS class names as part of its public surface. Downstream consumers of your Nuxt module write CSS against what they see. The rules below define what that surface must look like, regardless of which role the component plays (atom-style primitive, organism, section, or block).

Where these rules say "components in the ui-kit layer / ui layer / ui-app layer", they mean any component your module contributes that plays that role — typically the files under your module's `src/runtime/components/` or `src/runtime/app/` directory (or wherever your Nuxt module exposes its components).

## The four override surfaces

| Scope | Mechanism |
|---|---|
| Global theme | CSS custom properties (`--<token>`) exposed by `@laioutr-core/ui-tokens` |
| Per-component selector | BEM (`.<block>__<element>--<modifier>`) in `@layer lui-components` |
| Runtime state | HTML data-attributes (`[data-state="open"]`, `[data-disabled]`) |
| Per-instance slot | Typed slot-class props on multi-slot organisms where exposed |

Don't mix mechanisms within a scope. A given concern lives in exactly one of the four.

## Rules

### 1. Root class always present, matches the component file name

Every component must emit a single owned class on its root element, even when no styles are attached today. The class name is the kebab-case of the component file name.

```vue
<!-- ✓ -->
<!-- ProductTileBasic.vue -->
<template>
  <article class="product-tile-basic">…</article>
</template>

<!-- ✘ -->
<!-- Pagination.vue: first child has __list, no .pagination root -->
<!-- Avatar.vue: root carries .user-avatar instead of .avatar -->
<!-- HeroSlide.vue: emits two unrelated blocks .hero-slider-slide and .slide-card -->
```

The root class is the customer's only stable anchor to the component. Without it, every override path forces them through `:deep()` or DOM-position selectors that break on refactor.

### 2. BEM shape: `.<block>__<element>--<modifier>`

Inside the component, every styled element gets a class derived from the block name:

```vue
<template>
  <article class="product-tile-basic" :class="{ 'product-tile-basic--on-dark': onDark }">
    <div class="product-tile-basic__upper">
      <div class="product-tile-basic__flag">…</div>
    </div>
    <div class="product-tile-basic__info">…</div>
  </article>
</template>
```

- **Block** matches the file name.
- **Element** uses `__` separator.
- **Modifier** uses `--` separator and is for **render-time variants only** (`--ghost`, `--horizontal`, `--on-dark`).
- **Runtime state** (open/closed, disabled, loading) uses data-attributes (rule 7), not modifier classes.

### 3. `@layer lui-components` on every CSS block

Every `<style>` block in a component your module ships must wrap rules in the project layer:

```vue
<style>
@layer lui-components {
  .product-tile-basic { … }
  .product-tile-basic__info { … }
}
</style>
```

The layer order is declared upstream by `@laioutr-core/ui-kit`. Consumer CSS outside any layer wins by default — no `!important` needed.

### 4. No `<style scoped>`

Scoped styles inject runtime `data-v-*` hashes that leak into the customer's API surface. Use unscoped styles in `@layer lui-components` and rely on BEM block names for isolation.

### 5. No descendant selectors targeting child components

This is the existing rule from [`no-wrapper-css-overrides.md`](./no-wrapper-css-overrides.md), restated here because the public-CSS-API design depends on it.

```css
/* ✘ Reaching into a child's BEM */
.card .text-group__heading { color: var(--surface-fg); }
.footer-menu__mobile .accordion-item { width: 100%; }

/* ✘ :deep() into a child */
.popup__background :deep(img) { object-fit: cover; }
```

Express the difference as a **prop on the child** instead — a variant, a slot, a CSS-variable hook the child documents. If the prop doesn't exist yet, add it to the child first.

### 6. No `v-bind()` in CSS for overrideable values

`v-bind()` compiles to a per-component-instance hashed CSS custom property the customer cannot target. Use one of:

- A **CSS custom property on the root** the customer can override:
  ```vue
  <template><article class="card" :style="{ '--card-bg': background }">…</article></template>
  <style>@layer lui-components { .card { background: var(--card-bg, var(--default-bg)); } }</style>
  ```
- A **modifier class**:
  ```vue
  <template><article :class="['card', `card--align-${alignment}`]">…</article></template>
  <style>@layer lui-components { .card--align-start { text-align: start; } }</style>
  ```

`v-bind()` is acceptable for values that are genuinely instance-private and never customer-overrideable (e.g. computed positioning offsets driven by intrinsic layout). When in doubt, prefer a CSS custom property.

### 7. New runtime state uses data-attributes, not modifier classes

Runtime state styling targets data-attributes:

```vue
<!-- ✓ -->
<template>
  <button class="my-button" :data-disabled="disabled || undefined" :data-loading="loading || undefined">
    …
  </button>
</template>
<style>
@layer lui-components {
  .my-button[data-disabled] { opacity: 0.5; }
  .my-button[data-loading] .my-button__icon { animation: spin 1s linear infinite; }
}
</style>
```

```vue
<!-- ✘ Don't introduce state-modifier classes -->
<button :class="['my-button', { 'my-button--disabled': disabled, 'my-button--loading': loading }]">
```

reka-ui already emits `data-state`, `data-highlighted`, `data-disabled`, `data-orientation` on its primitives — match those when wrapping.

### 8. Block name = component file name (coordinated rename if it doesn't)

If you find a component whose root block doesn't match its file name, do not silently rename it in passing — that breaks consumers of the published class name. Treat it as a coordinated rename across exports, stories, registries, and consumer imports.

## Why this rule exists

Consumers of your module cannot reliably restyle its components if class names collide with their CSS, root classes are missing, scoped-style hashes leak in, `v-bind()` produces untargetable values, or your wrapper components reach into child internals. These rules close the gaps that make restyling unreliable.

## Related rules

- [`no-wrapper-css-overrides.md`](./no-wrapper-css-overrides.md) — the prop-not-selector mechanism for cross-component styling
- [`unique-component-names.md`](./unique-component-names.md) — component file names (which become block names) must be globally unique
- [`parent-prefix-naming.md`](./parent-prefix-naming.md) — when to use compound naming for sub-components
- [`storybook-stories.md`](./storybook-stories.md) — story conventions covering Public-Parts visibility
