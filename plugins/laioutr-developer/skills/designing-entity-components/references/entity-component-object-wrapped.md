# Entity Component Schemas Are Always Object-Wrapped

Every `defineEntityComponentToken(...)` schema is a `z.object({ ... })` with named fields. Never pass a bare value type, a bare array, a bare string or a union as the `schema`, and that includes a value type that is itself a `z.object` (`HtmlFragment`, `GeoCoordinates`, `SeoMeta`). This applies to every component you define: on your app's own entity types, and on a canonical entity you extend (`Product`, `Location`, …).

## The rule

```ts
// src/runtime/shared/tokens/store-location.ts
import { z } from 'zod/v4';
import { HtmlFragment } from '@laioutr-core/core-types/common';
import { defineEntityComponentToken } from '@laioutr-core/core-types/orchestr';
import { GeoCoordinates } from '@laioutr-core/canonical-types';

// ✅ the component owns an object and names its keys
export const StoreLocationCoordinates = defineEntityComponentToken('coordinates', {
  entityType: 'StoreLocation',
  schema: z.object({
    geo: GeoCoordinates,
  }),
});

export const StoreLocationContent = defineEntityComponentToken('content', {
  entityType: 'StoreLocation',
  schema: z.object({
    description: HtmlFragment,
  }),
});
```

```ts
// ✘ a shared value type adopted as the component's top level
export const StoreLocationContent = defineEntityComponentToken('content', {
  entityType: 'StoreLocation',
  schema: HtmlFragment, // the value is { html }, and there is no key of the component's own
});

// ✘ a bare array or string
export const StoreLocationTags = defineEntityComponentToken('tags', {
  entityType: 'StoreLocation',
  schema: z.array(z.string()), // write z.object({ tags: z.array(z.string()) })
});
```

The type of `schema` accepts any zod schema, so nothing stops a bare value at compile time. This is a convention, and the Laioutr CMS docs state it for every component: ["Each component schema is a `z.object` with named fields, even when the value is a shared type such as `HtmlFragment`"](https://docs.laioutr.com/apps/app-development/content-collections#components).

## Why

1. **One predictable shape.** Every component value is an object with named keys. Sections read `entity.components.coordinates.geo.latitude`. They never have to know whether the author wrapped the value.
2. **Extensible without a breaking change.** A sibling field added later (a `precision` next to `geo`) is a non-breaking addition to the component. If the component *were* the value type, you would have to widen the shared type, which you do not own, or reshape the component and break every consumer and every stored CMS entry.
3. **Reuse without coupling.** The shared value type stays single-purpose and is referenced under a key. The component's top-level shape remains its own contract.
4. **Consistent keys across carriers.** The same concept gets the same key everywhere. `geo` holds a `GeoCoordinates` on canonical `LocationCoordinates` and on `MailingAddress` alike.

## How to fix

1. Name the value: pick the key a consumer would naturally read (`description`, `geo`, `seo`, `tags`).
2. Wrap it in `z.object({ <key>: <ValueType> })`.
3. Update the component resolver to return `{ <key>: value }`, and update every section that reads the component.
4. If the component has already shipped, treat the reshape as a breaking change of your app. In the CMS, entries stored under the old shape fail the new schema, and the resolver skips them.

## Legacy shapes you will see upstream (do not copy)

Some canonical components predate this rule. They are kept for backwards compatibility, because consumers and stored configuration read their current shape. When you study canonical analogues in `node_modules/@laioutr-core/canonical-types/dist/server/entity/`, do not take these as precedent:

- `ProductDescription` uses `HtmlFragment` directly, so its value is `{ html }`.
- `ProductSeo`, `CategorySeo`, `BlogPostSeo` and `BlogCollectionSeo` use `SeoMeta` directly. The newer `LocationSeo` wraps it as `{ seo: SeoMeta }`, which is the shape to follow.
- `ProductVariantQuantityRule` uses `QuantityRule` directly.
- `ReviewContent` is a bare `z.string()`.
- `BlogPostTags`, `ProductFlags`, `ProductVariantQuantityPrices` and `BlogCollectionAuthors` are bare arrays.
- `MenuItemBase` is a bare union.

If you extend one of these entities, your new components still follow the rule.

## Related

- [`value-types-and-code-systems.md`](./value-types-and-code-systems.md): which shared value type to put under the key.
- [`cms-form.md`](./cms-form.md): how the wrapped object becomes a CMS form.
