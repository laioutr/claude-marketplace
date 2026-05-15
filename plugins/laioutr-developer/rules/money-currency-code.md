# Money Object Currency

Money objects are `{ amount, currency }` where `amount` is **minor units (cents)** and `currency` is an **ISO 4217 code** (`EUR`, `USD`, …). Never a symbol (`€`, `$`), never a label (`"Euro"`), never lowercase.

Default to `EUR` in stories, fixtures, and any mock data unless a specific currency is material to the example.

```ts
// ✅
{ amount: 1999, currency: 'EUR' }

// ✘
{ amount: 1999, currency: '€' }
{ amount: 19.99, currency: 'EUR' }  // amount must be cents
```

Formatters (`$money`, locale-aware price components) expect ISO codes — symbols render as opaque strings next to the amount.
