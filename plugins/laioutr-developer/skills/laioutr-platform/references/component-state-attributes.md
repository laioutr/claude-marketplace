# Expose Runtime State as reka-ui-Named `data-*` Attributes

Expose a component's runtime state as a `data-*` attribute on the element the state belongs to — never as a modifier class. Use reka-ui's attribute name and values wherever reka has one. **Booleans are presence-only, enums carry a value**: `[data-disabled]`, `[data-state="open"]`.

Applies to every component, Section and Block your module ships. Projects install your module from npm and cannot edit its components, so the selectors they write against it are an API. State attribute names are part of that API and must not be renamed later.

```vue
<!-- ✅ -->
<template>
  <article class="plan-card" :data-selected="selected || undefined" :data-disabled="disabled || undefined">…</article>
</template>
<style>
@layer lui-components {
  .plan-card[data-selected] { outline: 2px solid currentColor; }
  .plan-card[data-disabled] { opacity: 0.5; }
}
</style>
```

```vue
<!-- ✘ state as modifier classes -->
<article :class="['plan-card', { 'plan-card--selected': selected, 'plan-card--disabled': disabled }]">…</article>

<!-- ✘ Vue renders `false` as a live attribute: data-selected="false" still matches [data-selected] -->
<article class="plan-card" :data-selected="selected">…</article>
```

A presence-only attribute must be `undefined` when off (`:data-selected="selected || undefined"`) — the same trap as `inert`.

## Vocabulary

Where reka has a name for a state, use it verbatim. Don't invent a synonym, or a new attribute where one below fits. A component wrapping a reka primitive usually inherits the right attribute already: check before adding one, and never shadow it with a differently-spelled equivalent.

| Attribute | Values |
| --- | --- |
| `data-state` | see below |
| `data-disabled`, `data-readonly`, `data-invalid` | presence |
| `data-selected`, `data-focused`, `data-active`, `data-pressed` | presence |
| `data-highlighted` | presence — virtual focus in a collection, not `:hover` / `:focus` |
| `data-placeholder`, `data-placeholder-shown` | presence — no value chosen yet |
| `data-orientation` | `vertical` \| `horizontal` |
| `data-side` | `top` \| `right` \| `bottom` \| `left` — resolved after collision handling |
| `data-align` | `start` \| `center` \| `end` |
| `data-motion` | `from-start` \| `from-end` \| `to-start` \| `to-end` |

`data-state`, by archetype:

| Archetype | Values |
| --- | --- |
| Disclosure / overlay | `open` \| `closed` |
| Selection within a set | `active` \| `inactive` |
| Binary control | `checked` \| `unchecked` |
| Tri-state control | `checked` \| `unchecked` \| `indeterminate` |
| Pressed control | `on` \| `off` |
| Affordance visibility | `visible` \| `hidden` |
| Async progress | `loading` \| `complete` \| `indeterminate` |

## Variants are not state

A variant is what the component *is*, chosen when the markup is written (`variant="ghost"`, `size="l"`, `--on-dark`). Variants stay BEM modifier classes. State is what the component is *doing now*. Never emit both for the same fact — they will drift.

## Style off `data-*`, even when an ARIA attribute or `inert` carries the same state

When a state has an ARIA equivalent (`aria-expanded`, `aria-disabled`, `aria-hidden`, `aria-pressed`, …) or is expressed with `inert`, set **both**. Each attribute serves its own audience:

- the **`aria-*` / `inert` attribute** is the accessibility contract — what assistive technology is told;
- the **`data-*` attribute** is the styling hook — what your `@layer lui-components` rules and the project's CSS target.

```vue
<!-- ✅ both, each for its own audience — the same pair reka's CollapsibleTrigger emits -->
<button class="faq-item__trigger" :aria-expanded="open" :data-state="open ? 'open' : 'closed'">…</button>
```

```css
.faq-item__trigger[data-state='open'] .faq-item__icon { rotate: 180deg; }     /* ✅ */
.faq-item__trigger[aria-expanded='true'] .faq-item__icon { rotate: 180deg; }  /* ✘ couples styling to the a11y tree */
```

This is not redundancy. Drop the `data-*` and the styling contract has moved onto ARIA. Drop the `aria-*` and accessibility breaks.

## Why

1. **The selectors are a contract.** Projects style your components from outside and cannot patch them. A renamed or removed state attribute silently breaks their stylesheet.
2. **One spelling per idea.** Anyone who has styled a reka-based component already knows `data-highlighted` and `data-state="open"`. A second spelling of the same state is pure cost.
3. **Accessibility attributes move for accessibility reasons.** `inert` and `aria-*` change when a11y requirements change — "keep this tabbable", "use `inert` instead of `aria-hidden`". Styles keyed to them break silently when that happens. A `data-*` attribute can stay put while the behavioural attributes move.
4. **Modifier classes conflate two axes.** Expressing both variant and state as classes is how a component ends up with a long list of BEM strings where a handful of attribute names would do.

## Related

- [`public-css-api.md`](./public-css-api.md) — the other override surfaces: root class, BEM, `@layer`, CSS custom properties
- [`ui-kit-reka-ui.md`](./ui-kit-reka-ui.md) — wrapping a reka-ui primitive, which usually brings these attributes with it
- [`component-state-contract.md`](./component-state-contract.md) — the props (`disabled`, `invalid`, `open`) that usually drive these attributes
