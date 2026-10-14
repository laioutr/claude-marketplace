# Tracking Never Degrades the Page

Tracking must never decide what a visitor sees, and must never be able to fail an operation it only observes. If the data an event needs is missing, skip the event and carry on. This applies to every analytics call in your app: in Sections, Blocks, presentational components, composables and plugins.

**Losing an event is acceptable. Losing merchandise is not.** An analytics gap costs a row in a report. A rendering gap costs an order. When the two conflict, the page wins.

```vue
<!-- ✘ a missing analytics entity hides the product -->
<ProductCard v-if="entityFor(tile.productId)" :tile="tile" :entity="entityFor(tile.productId)!" />

<!-- ✅ the card renders regardless; only the event degrades -->
<ProductCard :tile="tile" :entity="entityFor(tile.productId)" />
```

So the props a component needs only for analytics are **optional**, and the handler guards:

```ts
import { AddToCart, SelectItem } from '@laioutr-core/canonical-types/analytics';
import { CartAddItemsAction } from '@laioutr-core/canonical-types/ecommerce';
import type { Money } from '@laioutr-core/core-types/common';
import type { ClientEntity } from '@laioutr-core/core-types/orchestr';

const props = defineProps<{
  tile: { productId: string; variantId: string; name: string };
  /** Analytics only. The card renders without these four. */
  entity?: ClientEntity;
  price?: Money;
  list?: { id: string; name?: string };
  position?: number;
}>();

const { track } = useAnalytics();

const onSelect = () => {
  if (!props.entity || !props.list || props.position === undefined) return;
  track(SelectItem, {
    listId: props.list.id,
    listName: props.list.name,
    product: { entity: props.entity, position: props.position },
  });
};
```

## Never build a payload inside someone else's `try`

This is the subtler form, and the one that actually bites. A payload read that throws inside a `try` around a mutation is caught by that mutation's `catch`. A **successful** add to cart then reports a failure to the visitor.

```ts
const addToCartMutation = useMutationAction(CartAddItemsAction);
const toaster = useToasterStore();
const { t } = useLocale();

// ✘ no `price` on this placement: `props.price!.amount` throws, and a successful add reports "could not add"
const addToCart = async (quantity: number) => {
  try {
    await addToCartMutation.mutateAsync([{ type: 'product', productId: props.tile.productId, variantId: props.tile.variantId, quantity }]);
    track(AddToCart, {
      products: [{ entity: props.entity!, quantity }],
      value: { amount: props.price!.amount * quantity, currency: props.price!.currency },
    });
    toaster.addToast({ title: t('cart.added'), variant: 'success' });
  } catch {
    toaster.addToast({ title: t('cart.addFailed'), variant: 'error' });
  }
};

// ✅ the mutation's catch covers the mutation and nothing else; the report guards its own inputs
const reportAddToCart = (quantity: number) => {
  if (!props.entity || !props.price) return;
  track(AddToCart, {
    products: [{ entity: props.entity, quantity }],
    value: { amount: props.price.amount * quantity, currency: props.price.currency },
  });
};

const addToCart = async (quantity: number) => {
  try {
    await addToCartMutation.mutateAsync([{ type: 'product', productId: props.tile.productId, variantId: props.tile.variantId, quantity }]);
  } catch {
    toaster.addToast({ title: t('cart.addFailed'), variant: 'error' });
    return;
  }

  reportAddToCart(quantity);
  toaster.addToast({ title: t('cart.added'), variant: 'success' });
};
```

Emit **after** the effect it reports, outside the `try` that guards that effect.

`useAnalytics`, `useMutationAction`, `useToasterStore` and `useLocale` are auto-imports. For an event of your own, define a token with `defineAnalyticsEventToken` from `@laioutr-core/core-types/analytics`, under your app's namespace (`my-laioutr-app/quote_requested`).

## Why

1. **The asymmetry is real.** A missed event is a gap in a dashboard. A hidden tile, a blocked redirect or a false "could not add to cart" costs a sale and teaches the visitor the shop is broken.
2. **Analytics data is the least reliable input on the page.** It depends on entities, list context and positions that one placement may not supply. A render or mutation path that depends on it inherits all of those failure modes.
3. **`track()` is built to be fire-and-forget.** It returns nothing, swallows a failed emission with a warning, and is ignored during SSR. Code that waits on it, or branches on it, is working against its contract.
4. **A shared `try` mixes two outcomes.** A `catch` cannot tell a failed mutation from a failed payload read. Keeping the emission outside it is the only way to keep the visitor's message truthful.

## Also covered

- A tracking failure never blocks a navigation, a mutation or a redirect.
- Never `await` an emission on a user-interaction path.
- An observer, listener or directive added for tracking never changes layout, focus or event propagation. Never call `preventDefault` from a tracking handler.

## Not covered

**Consent gating is not this.** A consent gate deliberately withholds delivery, and that is required behaviour. Laioutr's analytics bus applies it per destination, so you never branch on consent at the call site (see [How consent gates delivery](https://docs.laioutr.com/frontend/features/tracking#how-consent-gates-delivery)). This rule is about tracking breaking the page, never about tracking being correctly suppressed.

**Nor is it a reason to swallow errors.** Skipping an event on missing data is the intended behaviour, so it needs no log. A genuine defect in your tracking code should still surface in development.

## Related

- [Tracking docs](https://docs.laioutr.com/frontend/features/tracking): the event vocabulary, entity projection, custom tokens and destinations.
- [`mutations-use-action-tokens.md`](./mutations-use-action-tokens.md): the mutation whose `try` the event must stay out of.
- [`components-render-without-blocking.md`](./components-render-without-blocking.md): the same "the page wins" stance for data loading.
- [`fail-soft-resilience.md`](../../laioutr-platform/references/fail-soft-resilience.md): the server-side counterpart — a bad unit of connector data costs only that unit.
