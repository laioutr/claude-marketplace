# Forbidden Field Names

A schema field's **top-level** `name` becomes a prop on the underlying Vue
component. A small set of names collides with Vue's reactive attribute
handling — the prop you think you defined gets intercepted, merged, or
silently dropped onto the component root via `inheritAttrs`. None of these
names may be used as a bare `name` value in `defineSection` /
`defineBlock` schemas.

The rule applies at the **top level only**. Nested fields inside an
`object`-typed field's schema become keys on the parent's value
(`props.<parent>.<key>`), not direct props, so the same names there don't
collide. They're still discouraged for clarity, but mechanically safe.

## The forbidden list

| `name`             | Why                                                                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `style`            | Vue style-binding. Merges into the root element's inline style via `inheritAttrs`; the schema field never reaches the component as a prop. Also misleads readers into expecting CSS.          |
| `class`            | Vue class-binding. Same merge behaviour as `style`.                                                                                                                                           |
| `key`              | Vue's list-rendering key. Consumed by the v-DOM diff algorithm; never reaches `props`.                                                                                                        |
| `ref`              | Vue's template ref. Consumed by the renderer to register a template ref.                                                                                                                      |
| `is`               | Vue's dynamic-component prop. Consumed by `<component :is>` and customized built-in elements.                                                                                                 |
| `slot`             | Not special in normal SFC usage, but reserved for named-slot routing into Vue Custom Elements (`<my-element><div slot="…">`). Avoid for forward-compatibility with Web-Component compilation. |
| `refFor`, `refKey` | Vue 3 compiler internals emitted when refs appear inside `v-for`. Reserved even though undocumented as public API.                                                                            |

## Field names are camelCase only

Per `section-config-standard.md` §5, schema `name` values must be
`camelCase`. Dash-case (`my-field`, `ref-for`, `data-foo`) is not
supported: Vue resolves dash-case attributes in templates back to the
`camelCase` prop name, so a literally dash-cased prop never receives a
value from a parent.

This rule lists the **camelCase equivalents** of any reserved attribute
Vue would otherwise emit in dash-case (e.g. Vue's internal `ref-for` is
reserved here as `refFor`).

## What is _not_ on this list

- **`type`** — not a Vue-special attribute. The `type` key has meaning
  _inside_ a prop's options definition (`{ type: String }`), but a prop
  literally named `type` is fine.
- Names starting with `$` or `_` — reserved by Vue for instance APIs and
  internals, but moot here because `camelCase` doesn't allow them as
  leading characters in our naming convention.
- `data-*`, `aria-*` — fine; not Vue-special.
- `modelValue` — not reserved, but couples the component to `v-model`.
  Use deliberately.

## Compound names are fine

The rule applies to **bare** names only. Compound names that happen to
end (or start) with one of the reserved tokens are allowed and frequently
correct:

- `headingStyle`, `captionStyle`, `cardStyle` — style decorators (`as: 'style'`)
- `cardClass`, `wrapperClass` — explicit class-passthrough props on a child
- `productKey`, `searchKey` — domain identifiers
- `customRef`, `pageSlot` — when the domain word actually applies

## What to do instead

For the most common case — a top-level "look" selector on a section/block
component — use `variant` (per `section-config-standard.md` §3, Design →
Styling). `variant` is the canonical name; legacy `containerStyle`,
`accordionStyle`, and similar `<component>Style` selectors should be
renamed to `variant` when the component is touched.

If the field genuinely needs to control a style attachment of a _different_
field, model it as a style decorator: `<fieldName>Style` with
`as: 'style'`, attached `for: '<fieldName>'`.

## How this surfaces if you ignore it

- The component renders without the prop you wired through Studio. Studio
  shows the control; the component never receives the value.
- For `style` and `class`, the value lands on the root element's inline
  `style="…"` or `class="…"` attribute — visible in DevTools, but not in
  the prop-driven render path.
- For `key` / `ref` / `is`, the value is consumed by the renderer and
  the `props` object simply lacks the key.
- For `slot`, the bug only surfaces if the component is ever compiled
  to a Custom Element — silent in normal SFC use, which makes it harder
  to catch later.
- For `refFor` / `refKey`, the value collides with compiler-emitted
  attributes when the component is rendered inside a `v-for` with a
  template ref; observed as a runtime warning or as the prop being
  shadowed by the compiler's own value.

None of these are caught by typecheck or lint — the field definition is
structurally valid TypeScript. They surface only when an editor configures
the field in Studio and notices nothing changed.
