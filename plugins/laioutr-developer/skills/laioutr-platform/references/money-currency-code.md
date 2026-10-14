# Money Is an Integer in the Currency's Smallest Unit, With an ISO Code

Money objects are `{ amount, currency }` (type `Money` from `@laioutr-core/core-types/common`). `amount` is an **integer in the currency's smallest unit** — cents for EUR/USD, but yen for JPY (zero decimals) and fils for BHD (three). `currency` is an **ISO 4217 code** (`EUR`, `USD`, …): never a symbol (`€`, `$`), never a label (`"Euro"`), never lowercase.

Canonical reference: [Money type — Laioutr docs](https://docs.laioutr.com/frontend/api-reference/common-types/money).

Default to `EUR` in stories, fixtures, and any mock data unless a specific currency is material to the example.

```ts
// ✅
{ amount: 1999, currency: 'EUR' }

// ✘
{ amount: 1999, currency: '€' }
{ amount: 19.99, currency: 'EUR' }  // amount must be an integer in minor units
```

## Building a value

Convert a decimal with `Money.fromDecimal` from `@screeny05/ts-money` (add it to your app's dependencies, as the upstream apps do). It knows each currency's number of decimals; `decimal * 100` silently breaks JPY, KRW, BHD and KWD.

```ts
import { Money } from '@screeny05/ts-money';

// ✅ in a resolver
price: Money.fromDecimal(149.99, 'EUR'), // { amount: 14999, currency: 'EUR' }

// ✘
price: { amount: Math.round(149.99 * 100), currency: 'EUR' },
```

In the storefront, take the currency for a value you construct from the active market with `useCurrency()`; a `Money` you received already carries its own.

## Formatting

Formatters (`$money`, locale-aware price components) expect ISO codes — symbols render as opaque strings next to the amount. `$money` reads the value's own `currency`, not the market's.
