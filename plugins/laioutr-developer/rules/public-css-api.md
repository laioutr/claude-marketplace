# Public CSS API

Every component you publish across the three Laioutr UI layers — `ui-kit` (atoms), `ui` (commerce organisms), `ui-app` (sections/blocks) — ships its CSS class names as part of its public surface. Downstream consumers of your Nuxt module write CSS against what they see. The rules below define what that surface looks like today (ship-now hygiene) and what it will look like after the vNext migration (forward-looking conventions).

Where these rules say "components in the ui-kit layer", they mean any component your module contributes that plays the ui-kit role — typically the files under your module's `src/runtime/components/` directory (or wherever your Nuxt module exposes its components).

## The four override surfaces

| Scope | Mechanism | Status |
|---|---|---|
| Global theme | CSS custom properties (`--l-color-primary`, `--l-button-bg`) | vNext |
| Per-component selector | Prefixed BEM (`.l-<block>__<element>--<modifier>`) in `@layer l-components` | vNext |
| Runtime state | HTML data-attributes (`[data-state="open"]`, `[data-disabled]`) | **Required for new code now**; migration of existing modifiers in vNext |
| Per-instance slot | Typed `:ui` slot-class prop on multi-slot organisms | vNext |

Don't mix mechanisms within a scope. A given concern lives in exactly one of the four.

## Rules that apply now

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

Every `<style>` block in ui-kit/ui/ui-app must wrap rules in the project layer:

```vue
<style>
@layer lui-components {
  .product-tile-basic { … }
  .product-tile-basic__info { … }
}
</style>
```

The layer order is declared in ui-kit's module setup. Customer CSS outside any layer wins by default — no `!important` needed.

(Layer rename to `l-components` happens with the vNext prefix migration.)

### 4. No `<style scoped>`

Scoped styles inject runtime `data-v-*` hashes that leak into the customer's API surface. Use unscoped styles in `@layer lui-components` and rely on BEM block names for isolation.

### 5. No descendant selectors targeting child components

This is the existing rule from [`no-wrapper-css-overrides.md`](no-wrapper-css-overrides.md), restated here because the public-CSS-API design depends on it.

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

For new components or new state on existing components, runtime state styling targets data-attributes:

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
<!-- ✘ Don't introduce new state-modifier classes -->
<button :class="['my-button', { 'my-button--disabled': disabled, 'my-button--loading': loading }]">
```

Reka-ui already emits `data-state`, `data-highlighted`, `data-disabled`, `data-orientation` on its primitives — match those when wrapping. **Don't migrate existing class-based modifiers to data-attributes ad-hoc** — that is a vNext task with a coordinated codemod. The rule here is forward-looking: don't add new ones.

### 8. Block name = component file name (eight-touchpoint rename if it doesn't)

If you find a component whose root block doesn't match its file name, do not silently rename it in passing — that breaks consumers of the published class name. Treat it as a coordinated rename across exports, stories, registries, and consumer imports.

## Rules that apply after vNext migration (do not pre-empt)

The following are part of the planned Laioutr CSS API v2 migration but **not enforceable until the coordinated breaking release**:

- `l-` class prefix and `--l-` CSS variable prefix
- `@layer l-components` (renamed from `lui-components`)
- `:ui` slot-class prop on ui/ui-app organisms
- Storybook "Public Parts" table per component

Do not add `l-`-prefixed classes or `--l-` variables to individual components today — that creates a half-migrated state worse than either end. The coordinated codemod runs once, atomically, across all three component layers.

## Why this rule exists

Customers cannot reliably restyle our components today: class names collide with their CSS, root classes are missing, scoped-style hashes leak in, `v-bind()` produces untargetable values, and our own wrapper components reach into child internals — proving that even we don't trust the public surface. The ship-now rules close the worst gaps without breaking consumers; the vNext migration locks in the contract.

Each rule above corresponds to a specific failure mode catalogued in the Laioutr CSS architecture audit.

## Related rules

- [`no-wrapper-css-overrides.md`](no-wrapper-css-overrides.md) — the prop-not-selector mechanism for cross-component styling
- [`unique-component-names.md`](unique-component-names.md) — component file names (which become block names) must be globally unique
- [`parent-prefix-naming.md`](parent-prefix-naming.md) — when to use compound naming for sub-components
- [`storybook-stories.md`](storybook-stories.md) — story conventions referenced from the future Public Parts tables
- [`lint-after-ui-changes.md`](lint-after-ui-changes.md) — lint must pass after every CSS edit
