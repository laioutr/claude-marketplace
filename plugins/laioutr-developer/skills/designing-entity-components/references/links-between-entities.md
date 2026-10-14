# A Link Exists Only When a Page Loads It

A link token (`defineLinkToken`) is a **frontend data-loading path**, not a model of the source system's relationships. Before you write one, answer a single question: **which page is the frontend on (its root query's entity, or an entity it reached through an earlier link), and what does it load and display from there?** If no real page loads the target from the source, there is no link, however real the relationship is in the backend. Copying the source system's graph one-to-one is the main over-linking failure.

Relationships are never fields on a component either: no `productId`, no `organizationId`, no list of ids. They resolve at request time through a link handler.

## Three relationships that look like links but aren't

- **"The current / my / the company's X" is a query, not a link.** The cart is fetched by the canonical `ecommerce/cart/get-current` query, never through a `Customer → Cart` link. "My quotes", "my lists" and "pending approvals" work the same way: they are session- or parent-scoped list *queries*.
- **A displayed name, owner, author or parent is denormalized onto a component, not linked.** Canonical `Review` carries its reviewer in the `reviewer` component (`ReviewReviewer`), and there is no `Review → Customer` link. Add a link into a person or organization **only when the UI navigates into that entity** (user management drills into an `Employee`). To render "raised by Jane" or "owned by Acme", denormalize the display value.
- **The reverse direction, unless a separate page roots at the other end.** The only bidirectional pair among the canonical links is `Product ↔ Category`: a product page shows its categories, and a category page lists its products. Never emit a reverse link speculatively.

## Direction is enforced, and the chain is shallow

- **`source` is the entity the frontend already has, and `target` is what it loads next.** You can only traverse a link from its `source`. Orchestr rejects the request with `WrongLinkSourceError` otherwise. A link whose `source` is an entity no page roots at, and that nothing links to, is dead.
- **Links run page-root → nested data.** Every canonical link has that shape: `Cart → CartItem → ProductVariant`, `Product → variants / reviews / breadcrumb`, `Organization → employees / cost-centers / budgets`. None of them runs child → parent.
- **At most two links from the root query's entity** (`root → link1 → link2`). A third throws `LinkRecursionError`. If a datum sits deeper, root the query closer, add a more direct link, or denormalize the value into reach.

## The idiom

A link is a pure string token. It imports nothing from the entity token files, because `source` and `target` are entity-type *names*, so it typechecks before either entity exists.

```ts
// src/runtime/shared/tokens/store-locator.ts
import { defineLinkToken } from '@laioutr-core/core-types/orchestr';

export const StoreEventsLink = defineLinkToken('store-locator/store-location/events', {
  label: 'Store Events',
  source: 'StoreLocation', // the page is on a store…
  target: 'StoreEvent',    // …and loads its events to render
  type: 'multi',
  defaultLimit: 10,
});
```

The handler lives in your `orchestr/` directory and returns `{ links: [{ sourceId, targetIds }] }` for a `multi` link, or `{ sourceId, targetId }` per entry for a `single` one. See [Queries & Links](https://docs.laioutr.com/frontend/orchestr/queries#links). To study canonical links, look at `CartItemsLink` + `CartItemProductVariantLink` (`Cart → CartItem → ProductVariant`), `CategoryProductsLink`, `ProductVariantsLink` and `ProductReviewsLink` under `node_modules/@laioutr-core/canonical-types/dist/server/lib/ecommerce/`.

## Metadata (`LinkTokenMetadata`)

| Field | Meaning |
|---|---|
| `source` / `target` | Entity-type names. |
| `type` | `'single'` (to-one) or `'multi'` (to-many). Match it to the cardinality. |
| `label` | Human label, shown in Studio and DevTools. |
| `defaultLimit?` | Page size for a `multi` link that can be large. Canonical `CategoryProductsLink` uses 48. |
| `nullable?` | The single target may be absent (canonical `CartItem → ProductVariant`). |
| `description?` | Optional prose. |

## Mechanics

- **Token name:** `<your-namespace>/<source-entity-kebab>/<relation>`, such as `store-locator/store-location/events` or `acme/recipe/author`. Namespace the first segment with your app, so two apps cannot define the same link name. Names are lowercase `a-z0-9-/`: for a scoped package drop the `@` and join with `-` (`@acme/events` → `acme-events/…`).
- **Placement:** in your shared token files (`src/runtime/shared/tokens/<feature>.ts`), so both the handler and your frontend can import it.
- **A line or container item is loaded *from* its container, then loads its own target.** `Cart → CartItem` (multi), then `CartItem → ProductVariant` (single), is depth 2. `Quote → QuoteItem → ProductVariant` and `ProductList → ProductListItem → ProductVariant` follow the same pattern. The line entity holds no `cartId`.
- **A link is orthogonal to a denormalized snapshot.** `CartItem` and `QuoteItem` snapshot title and price for order integrity *and* link the variant, while `ProductListItem` snapshots nothing and resolves live. The *frontend load* decides whether the link exists, not the snapshot.
- **Extending a canonical entity:** a link from a canonical source (`Product`) to your own entity is fine. Name it under your namespace, not under `ecommerce/`.

## In the Laioutr CMS

When the types are CMS collections ([`cms-form.md`](./cms-form.md)):

- A link is served by the collection of its **source** type, and only when it is exported from that collection's token module.
- **Both ends must be declared collections.** A link to a type that is not a collection is skipped, and Cockpit does not offer it for editing.
- **A source type owned by another app** (your events listed on another app's `StoreLocation`) follows from the rule above: the CMS serves the link only if that app's collection exports it. Ask the owning app to export it, or model the relation from a side you own.
- Component data holds no references. A list of items that each hold an entry *and* a caption is not a link. Model the item as its own type, with a link to it.

Source: [Content Collections → Links](https://docs.laioutr.com/apps/app-development/content-collections#links).

## Forbidden patterns

| Pattern | Instead |
|---|---|
| An id field (`productId`, `organizationId`) on a component | No foreign keys. If the frontend loads it, add a link. |
| Tokenizing every relationship the backend or its docs describe | Add a link only for a load a real page performs. |
| A link into a displayed person or organization (buyer, reviewer, requester) | Denormalize the display value. Link only when the UI navigates into it. |
| A "list my / the company's X" link | Use a `get-current`-style query. |
| A child → parent link (`CostCenter → Organization`) | Links run page-root → nested data. |
| A speculative reverse link | Add only a direction a separate page roots at. |
| A datum more than two links from the page root | Root closer, link more directly, or denormalize. |
| `single` for to-many, or `multi` for to-one | Match `type` to cardinality, and add `defaultLimit` to large `multi` links. |
