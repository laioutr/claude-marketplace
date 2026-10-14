# Read a Query Prop the Way Its Field Declares It

Read a `query` prop in the shape its field declares. With `singleEntity: true` the prop **is** the entity. Without it, the prop is an entity set and the entities are in `.entities`. Either way it can be `undefined` while loading or after an error, so guard it. Read the current page from `.pagination.current`. Request only the `components` and `links` your template uses, and give each link a `limit` that matches what you display.

```ts
// field: { type: 'query', name: 'product', entityType: 'Product', singleEntity: true, components: [ProductBase] }

// ✘ singleEntity: true, read as if it were a set
const name = computed(() => props.product?.entities[0]?.components.base.name);

// ✅ the prop is the entity
const name = computed(() => props.product?.components.base.name);
```

```ts
// field: { type: 'query', name: 'products', entityType: 'Product', components: [ProductBase] }

// ✘ there is no `.page`
const page = computed(() => props.products?.page ?? 1);

// ✅ `pagination.current` is the 1-based page; `offset`, `limit`, `total`, `pages`, `next`, `previous` sit beside it
const page = computed(() => props.products?.pagination.current ?? 1);
```

```ts
// ✅ only what the template renders, with a limit that matches it
// ProductBase, ProductPrices: '@laioutr-core/canonical-types/entity/product'
// ProductVariantBase: '@laioutr-core/canonical-types/entity/product-variant'
{
  type: 'query',
  name: 'product',
  label: 'Product',
  entityType: 'Product',
  singleEntity: true,
  components: [ProductBase, ProductPrices],
  links: {
    'ecommerce/product/variants': {
      entityType: 'ProductVariant',
      components: [ProductVariantBase],
      limit: 5,
    },
  },
}
```

## Why

1. **The two shapes are different types.** `definitionToProps` types a `singleEntity` prop as one entity and any other query prop as a set. Reading one as the other is either a type error or, with loose typing, a silent `undefined`.
2. **The prop really is absent at times.** Frontend Core resolves the query through Orchestr and hands the result over when it lands. Until then, or after an error, the prop is `undefined`, whatever the field's configuration.
3. **Every component and link is work per request.** Orchestr resolves everything you declare, whether the template reads it or not. Unused components, extra links, large `limit` values and deep link chains all slow page rendering.
4. **An unset `limit` is a page size you didn't choose.** Without it, a link's result size and its `pagination` come from a default rather than from what your template shows.

## Limits to know

- Links nest at most two levels. Orchestr rejects deeper chains with a `LinkRecursionError`.
- Filters, sorting and paging change through the URL: build it with `buildQueryUrl(entitySet, { … })` and push it with the router. Never mutate the prop. See [Consuming Query Fields](https://docs.laioutr.com/apps/app-development/consuming-query-fields).

## Related

- [Schema fields: query](https://docs.laioutr.com/apps/app-development/schema-fields#query): `entityType`, `components`, `links` and their options.
- [`trust-the-schema-in-components.md`](./trust-the-schema-in-components.md): `query` is one of the field types that can arrive `undefined`.
