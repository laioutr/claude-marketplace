# Boolean Prop Naming — the `is*` Rule

Decide a boolean prop's name by asking what its **subject** is. Applies to any Vue component your module ships, and to the upstream `@laioutr-core/ui-kit`, `@laioutr-core/ui`, and `@laioutr-core/ui-app` patterns you mirror.

## The test: what is the subject of the boolean?

- **Subject = the component itself → drop `is`.** The bare token is unambiguous because every reader knows the component is the subject.
  - Visual / interactive states: `loading`, `open`, `selected`, `active`, `disabled`, `required`, `checked`, `invalid`, `rounded`, `centered`, `sticky`.
- **Subject = some entity the component is *given* (user, product, cart, order, viewport) → keep `is`.** The prefix anchors the reader to "this prop testifies to a state, it does not command a behavior."
  - World-state facts: `isLoggedIn`, `isSoldOut`, `isFreeDelivery`, `isShippingFree`, `isVerified`, `isAboveTheFold`, `isApplied`.

```ts
// ✅ component is the subject — bare token
defineProps<{ loading?: boolean; open?: boolean; selected?: boolean; disabled?: boolean }>();

// ✅ prop testifies to an external fact — keep is*
defineProps<{ isSoldOut?: boolean; isLoggedIn?: boolean }>();

// ✘ bare token for a world-state fact reads as a command
defineProps<{ soldOut?: boolean }>();   // is the cart sold out, or the product?
// ✘ is* on a state the component is in
defineProps<{ isLoading?: boolean }>();
```

## Why the rule isn't "always drop `is`"

1. **Imperative collision.** Vue templates use bare tokens for actions (`disabled`, `selected`, `checked`). For a fact-about-the-world the same syntax becomes ambiguous: `<ReviewItem verified />` reads as a command. `is` flips the mood from imperative to declarative.
2. **Subject confusion.** When the component renders information *about something else*, dropping `is` makes the reader think the component is the subject.

## Borderline / better-solved-by-rename

- `isStandalone` on a block reads fine as `standalone` if every block converts together; `isHighlight` / `isSaleDesign` lean toward dropping `is` (visual-modifier reading).
- `isError` → `invalid` (disambiguates from `error: string` — see [`component-state-contract.md`](./component-state-contract.md)).
- `isNotAvailable` (double negative) → re-express as `disabled` or `unavailable`.
- Scoped sub-states like `isAddToCartLoading` → consider a single `loadingState` enum (don't invent one without a clear decision).

## Related

- [`component-state-contract.md`](./component-state-contract.md) — `invalid`, v-model, field context
- [`prop-naming-vocabulary.md`](./prop-naming-vocabulary.md) — sizes, icons, links, `variant`
