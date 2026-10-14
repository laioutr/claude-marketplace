# A Section or Block Never Awaits in Setup

Never put a top-level `await` in the `<script setup>` of a Section, a Block, or any component of yours that loads its own data. Start the request, render the empty state immediately, and fill in when the data lands.

A top-level `await` compiles to an async `setup()`. Vue suspends the component, and Nuxt's root Suspense stays unresolved until that promise settles, so nothing on the page becomes interactive until your component's data arrives. A Section in a header or footer pays that cost on every route.

```ts
// ✘ the whole page waits for the visitor's addresses
if (import.meta.client) {
  const { addresses: loaded } = await fetchAction(AddressGetAllAction);
  addresses.value = loaded;
}

// ✅ the request starts now, the section renders now
const addresses = ref<ActionTokenOutputOf<typeof AddressGetAllAction>['addresses']>([]);

if (import.meta.client) {
  fetchAction(AddressGetAllAction)
    .then((result) => {
      addresses.value = result.addresses;
    })
    .catch((error) => {
      console.warn('[my-laioutr-app] The saved addresses could not load.', error);
    });
}
```

`fetchAction` and `ref` are auto-imports from `#imports`; `AddressGetAllAction` comes from `@laioutr-core/canonical-types/ecommerce`, `ActionTokenOutputOf` from `@laioutr-core/core-types/orchestr`. Prefix the warning with your app's name.

Three obligations come with it:

- **Hold the result in a `ref`** that starts `undefined` or empty, and derive everything else from it.
- **Keep the template safe for that starting value.** Computeds return `[]` / `undefined`, and the props you pass down are optional. A component that needs a `v-if` around its whole template to survive the empty state never had its empty state designed.
- **Catch the rejection.** The `await` used to carry it to the Suspense boundary. Without a `.catch`, it becomes an unhandled rejection, and the component keeps stale contents with nothing in the console.

## Composables that return refs

`useFetchAction(Token, input)` returns `data` and `error` refs that fill in. The [Actions docs](https://docs.laioutr.com/frontend/orchestr/actions#usefetchaction) show it with `await`. In a Section or Block, call it without the `await` and render from the refs.

When the request follows a reactive input instead of running once, use VueUse's `computedAsync` (auto-imported, because Frontend Core installs `@vueuse/nuxt`). It returns a ref that updates when each promise resolves, and never suspends the component:

```ts
// OrderGetStatisticsAction from '@laioutr-core/canonical-types/ecommerce'
const year = ref(new Date().getFullYear());

const months = computedAsync(
  async () => (await fetchAction(OrderGetStatisticsAction, { year: year.value })).months,
  [],
  { onError: (error) => console.warn('[my-laioutr-app] The order statistics could not load.', error) },
);
```

## Loading a heavy component

Split it with `defineAsyncComponent(() => import('../components/HeavyMap.vue'))`, or use the `Lazy` prefix on a component Nuxt registers. Either suspends only that subtree. Never `await import(...)` at the top level of setup, because that suspends the importer.

## Why

1. **One slow component holds the whole page.** The root Suspense waits for every async `setup()` beneath it. Your Section shares that boundary with every other Section on the page, so its round trip becomes the page's time to interactive.
2. **Chrome multiplies the cost.** A header, footer or flyout Section renders on every route, so an `await` there is paid on every navigation, not once.
3. **The empty state is part of the contract.** Data can be slow, missing, or fail. A component that renders without it degrades to empty instead of blocking the page.
4. **A swallowed rejection is invisible.** Without `.catch` (or `onError`), a failed load leaves old data on screen and no trace in the console.

## When this doesn't apply

An `await` is fine wherever it does not run during setup: an event handler, a `watch` callback, `onMounted`, a `computedAsync` body, or a mutation's `mutateAsync`. Those are the normal case, and nothing here restricts them.

## How to check

The Vue SFC compiler decides. A component is async when its compiled output contains `_withAsyncContext`. In practice, grepping `<script setup>` for a top-level `await` finds the same thing.

## Related

- [`mutations-use-action-tokens.md`](./mutations-use-action-tokens.md): the `await` inside a mutation handler belongs there, not in setup.
- [`tracking-never-degrades-the-page.md`](./tracking-never-degrades-the-page.md): the same "the page wins" stance, applied to analytics.
