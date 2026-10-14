# Bad Data Costs Only the Unit That Carries It

Your app does not control some of the data it handles: backend API responses, webhook payloads, request data (host, cookies, query, bodies), the options a customer saved for your app, and the environment. None of it may fail a build or take down a whole request. Guard each unit where you map it, substitute or skip that unit, and warn with its id.

```ts
// src/runtime/server/orchestr/product/prices.resolver.ts
import { Money as TsMoney } from '@screeny05/ts-money';
import { ProductPrices } from '@laioutr-core/canonical-types/entity/product';
import { defineAcmeComponentResolver } from '../../middleware/defineAcme';

// ✘ one product priced 7.140001 throws, and every product in the call loses its prices
export default defineAcmeComponentResolver({
  label: 'Acme product prices',
  entityType: 'Product',
  provides: [ProductPrices],
  resolve: async ({ entityIds, context, $entity }) => {
    const products = await context.acme.getProducts(entityIds);
    return {
      entities: products.map((product) =>
        $entity({
          id: product.id,
          prices: {
            price: TsMoney.fromDecimal(product.price.value, product.price.currency),
            isOnSale: false,
            isStartingFrom: false,
          },
        }),
      ),
    };
  },
});
```

```ts
// ✅ same imports; each product is its own unit: a bad one is skipped and named, the rest render
export default defineAcmeComponentResolver({
  label: 'Acme product prices',
  entityType: 'Product',
  provides: [ProductPrices],
  resolve: async ({ entityIds, context, $entity }) => {
    const products = await context.acme.getProducts(entityIds);
    return {
      entities: products.flatMap((product) => {
        try {
          const price = TsMoney.fromDecimal(product.price.value, product.price.currency, 'round');
          return [$entity({ id: product.id, prices: { price, isOnSale: false, isStartingFrom: false } })];
        } catch (error) {
          console.warn(`[acme] product ${product.id}: price ${product.price.value} ${product.price.currency} is unusable; prices omitted`, error);
          return [];
        }
      }),
    };
  },
});
```

`defineAcmeComponentResolver` is your app's re-export of `defineOrchestr.meta(…).extendRequest(…).componentResolver` (see the docs' Orchestr coding standards). In real code the per-item mapping belongs in `orchestr-helper/` or `mappers/`. The guard goes inside the loop either way.

## The rule

1. **The blast radius is the unit of bad data.** Guard inside the loop, not above it. One bad product, variant, price, category or webhook line item costs exactly that unit. It never costs the query, the page, the server function or the build. The archetype: a backend returned one price at float precision (`7.140001`), `Money.fromDecimal` threw, the throw failed the whole query, and every product slider on the page rendered empty. Orchestr's own containment is coarser than you want. A throwing query handler fails that query, so its list gets no ids. A throwing component resolver loses the components it provides for **every** entity in the call.
2. **Degrade loudly.** Pair every fallback with a `console.warn` that names the offending unit's id (product id, SKU, option name, `app@version`) and what was substituted. A silent fallback is as wrong as a crash, because it moves the debugging to someone with less context.
3. **Fail soft in the data path, not in components.** During SSR, Vue's server renderer runs a component's render inside `try/finally` with no `catch`, and `<NuxtErrorBoundary>` registers its error hook on the client only. A throw while server-rendering one of your sections fails the whole page. Normalize data in the handler or mapper, so a section only ever receives data it can render.
4. **Module scope is boot scope.** Top-level code in a server route, Nitro plugin, middleware or handler file runs once when the server starts. Code in your `module.ts` `setup` runs during `nuxt build`. Neither has per-request error handling around it, so a throw there fails every request or the customer's build. Guard per item there too, for example when parsing an option map or building a lookup table.
5. **Wrap dependency calls that throw on data** whenever the input is untrusted. `Money.fromDecimal` from `@screeny05/ts-money` throws on excess decimals unless you pass the `'round'` rounder, and on an unknown currency code even with it. `Intl.*` constructors throw `RangeError` on a bad locale or currency. `new URL()` and `JSON.parse` throw. Zod's `.parse` throws, so prefer `.safeParse`. **Guard the inverse trap too.** `Number()` and `parseInt` don't throw: `Number('12,50')` is `NaN` and `Number('')` is `0`. Either one flowing into money or quantity math is corruption, not resilience. Check `Number.isFinite` before you use the result.

```ts
// src/module.ts, inside setup(options): runs during `nuxt build`

// ✘ one malformed domain fails the customer's build
const storeOrigins = Object.entries(options.storeDomains ?? {}).map(([market, domain]) => [market, new URL(`https://${domain}`).origin]);

// ✅ one malformed domain costs its market, by name
const storeOrigins = Object.entries(options.storeDomains ?? {}).flatMap(([market, domain]) => {
  try {
    return [[market, new URL(`https://${domain}`).origin]];
  } catch {
    console.warn(`[${name}] storeDomains.${market}: "${domain}" is not a host name; market skipped`);
    return [];
  }
});
```

## When to fail hard, deliberately

Fail hard when serving would be wrong or dangerous. Then attribute the failure precisely instead of softening it:

- **Security boundaries get no fallback.** A webhook whose signature does not verify is rejected; it is never processed "just in case". No platform gate protects `/api/app-<name>/webhooks/…`, so the signature is the whole authorization. Shopify's webhook route shows both sides. A bad signature gets a non-success status. An unknown topic or an unreadable body is answered with success and logged, because repeated failures cost the vendor subscription, while one skipped delivery costs only a stale cache window.
- **A connector that cannot do its job at all.** A storefront that silently serves an empty catalogue is worse than a visible error. When a credential your module cannot start without is missing, fail with an error that names the option and your package. Don't let it surface later as the backend's `401` three layers down.
- **A hard dependency with no substitute.** When there is nothing correct to serve instead, fail, and name the dependency.

On these paths the resilience work is diagnostics, not softening. The error must name the failing `app@version`, credential or option, so the customer never has to decode someone else's stack trace.

## Boundaries

- **Fail-soft is not error-swallowing.** A bare `catch {}` with no fallback value and no warning is a bug.
- **Where a fallback would corrupt, refuse instead of guessing.** Wrong prices, wrong redirects and wrong legal or consent state are corruption. Never substitute `0` for a missing price: `0,00 €` makes a different claim from "this store sent no price". Omit the value, or fail with a clear, attributed error.

## Why

1. **One bad record is normal; a blank page is an outage.** Backends return malformed rows all the time. Guarding the unit turns an incident into a warning.
2. **SSR gives you no safety net.** There is no server-side error boundary to catch a throw from a section, so the data path is the only place resilience can live.
3. **Boot-time throws multiply.** A throw at module scope or in `setup` is paid by every visitor, or by a deployment that never ships.
4. **An unattributed failure lands on the wrong person.** A warning that names the product id or option lets the customer fix their data. A bare stack trace makes them file a ticket against your app.

## Related

- [`app-path-prefixing.md`](./app-path-prefixing.md): webhooks live under `/api/app-<name>/webhooks/…`, unguarded by the platform.
- [`app-settings-manifest.md`](./app-settings-manifest.md): mark an option `required` only when the module cannot start without it, the same line this rule draws for failing hard.
- [`money-currency-code.md`](./money-currency-code.md): what a valid Money value looks like.
