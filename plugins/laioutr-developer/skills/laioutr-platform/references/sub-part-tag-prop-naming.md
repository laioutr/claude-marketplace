# Sub-Part Tag Prop Naming: `<part>As`

When a component, section, or block exposes a prop or schema field that controls the **HTML tag** used to render a named sub-part — e.g. which heading level the headline renders as, or which element wraps the subline — that prop must be named `<part>As`.

Applies to any Vue component, section, or block your module ships, and to upstream patterns in `@laioutr-core/ui-kit`, `@laioutr-core/ui`, and `@laioutr-core/ui-app`.

## The rule

```ts
// ✅
defineProps<{
  headingAs?: 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6';
  sublineAs?: 'p' | 'span' | 'div';
  captionAs?: 'p' | 'span';
}>();

// ✘
defineProps<{
  headingTag?: string;        // not "Tag"
  headingElement?: string;    // not "Element"
  headlineLevel?: number;     // not a numeric level
  as?: string;                // single `as` is ambiguous when there are multiple parts
}>();
```

The same applies to Studio schema fields on sections and blocks:

```ts
// ✅
defineSection({
  schema: {
    headingAs: { type: 'select', options: ['h1', 'h2', 'h3', 'h4'] },
    sublineAs: { type: 'select', options: ['p', 'span'] },
  },
});
```

## When to expose the prop at all

Only when the **tag choice is semantically meaningful** to consumers — usually for accessibility / document outline (e.g. `h1` on a hero versus `h2` on a section header in the same page) or for swapping a block-level wrapper for an inline one.

Don't expose `*As` for parts where the tag is fixed by design (a `<button>` stays a button, an image's `<img>` stays an `<img>`). One `*As` prop per sub-part that genuinely needs it — not a prop per element on principle.

## Why `<part>As` and not bare `as`

1. **Multiple parts in one component.** A typical Section or Block has more than one tagged text region (heading + subline + caption). A single `as` prop can only address one of them. Prefixing with the part name makes the target unambiguous and keeps the prop list parallel to the sub-part vocabulary already used in component APIs (`heading`, `subline`, `caption`).
2. **Industry precedent.** `<X>As` mirrors the `as` prop pattern from Chakra UI, Radix Themes, and styled-components, while scoping it per sub-part the way Shopify Polaris and Adobe Spectrum do for compound headings. Consumers reading the API recognize it on sight.
3. **Type narrowing.** Each sub-part has a different valid tag set (`heading` → `h1..h6`; `subline` → `p`/`span`/`div`). Per-part props let each one carry its own union literal type. A bare `as` collapses to the widest union and loses authoring-time safety.
4. **Avoids collision with reka-ui's `as` / `asChild`.** Ui-kit primitives that wrap reka-ui already forward a bare `as` to the underlying primitive (polymorphic element). Reserving `as` for that use and using `<part>As` for component-internal sub-parts keeps the two concerns separate.

## Naming the part

Use the same noun the rest of the component API uses for that region:

- `heading` / `headingAs` (not `headline` here, not `title` there — pick one per component and stay consistent)
- `subline` / `sublineAs`
- `caption` / `captionAs`
- `eyebrow` / `eyebrowAs`

If the component renames its sub-part (e.g. `heading` → `title`), rename the `*As` prop in the same commit, touching every consuming template, type import, and schema string in one coordinated pass.

## Defaults

Pick a sensible semantic default and document it. Heading defaults vary by context (`h2` for most section headings, `h1` for the page hero); never default to a non-semantic tag like `div` for a heading.

## Related

- [`public-css-api.md`](./public-css-api.md) — props, not selectors, are the override surface
- [`parent-prefix-naming.md`](./parent-prefix-naming.md) — part names should describe what the part is
- [`reka-ui-no-type-leakage.md`](./reka-ui-no-type-leakage.md) — when the bare `as` is the reka-ui polymorphic prop, not a sub-part name
