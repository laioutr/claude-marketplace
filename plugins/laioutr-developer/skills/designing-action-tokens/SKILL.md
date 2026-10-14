---
name: designing-action-tokens
description: Use when adding or reshaping an action token (`defineActionToken`, a `.action.ts` handler or a shared token file) in a Laioutr app module. An action is an imperative call the frontend makes to the backend, either a mutation (add to cart, subscribe, register, book, submit a form, save, delete) or an on-demand read with no entity behind it (a checkout URL, the current customer, a balance, a download URL). Also use when a section, block or component in your app needs `useMutationAction`, `useQueryAction` or `fetchAction` for a side effect and no fitting action token exists yet, or before asking Laioutr for a new canonical action. Not for loading entity data (a query token) or for a relationship between entities (a link token).
---

# Designing Action Tokens

## Overview

An action token is the **contract for one imperative call**. It is the seam between `useMutationAction` / `fetchAction` in the frontend and an action handler on the server. `defineActionToken(name, { input, output })` returns a tagged string. The token file declares **only** the name, the input schema, the output schema and, in JSDoc, the errors the call throws. The handler lives elsewhere.

An action is **researched and its shape proposed before it is written**. Look at internal prior art first (canonical siblings and your app's own tokens), then at the backend systems the handler will call. Their operations *are* the contract your input and output must line up with, and the research routinely changes the design. You may discover that a backend has no such operation, or keys a field differently.

**Design for the frontend's hand.** The frontend calls an action with data it already has, and frontend ↔ server round-trips are the expensive, UX-critical resource. The handler owns any lookup or join. Never force a read before the write.

**The three failures this skill prevents:**

1. picking the wrong token kind, when an entity read is a query and a relationship is a link;
2. reinventing an input or output shape that a sibling, or the backend's own vocabulary, already defines;
3. baking failure states into the output instead of throwing a typed error.

## Decide the token kind first

| You want to… | Token |
|---|---|
| Cause a mutation or side effect | `defineActionToken` |
| Read something on demand that is **not an entity** (a scalar, a URL, a value type such as `Customer`) | `defineActionToken` |
| Load **entities** the page renders, i.e. ids that component resolvers fill | `defineQueryToken` |
| Traverse from one loaded entity to related ones | `defineLinkToken` |

The line between action and query is **not** "does the page render it". Canonical `ecommerce/cart/get-current` is a query because `Cart` is an entity. `ecommerce/customer/get-current` is an action because `Customer` is only a value type. Depth, including request-scoped facts and companion reads, is in [`token-kind-decision.md`](./references/token-kind-decision.md).

## Canonical first, custom second

If a canonical action covers the operation (see [Canonical Actions](https://docs.laioutr.com/frontend/api-reference/ecommerce/actions), plus the b2b and newsletter references), **implement its handler** instead of defining a token. Storefront sections already call canonical tokens. Define a custom token only when no canonical one fits. If you think the canonical set *should* have it, or a canonical token needs a field, propose that upstream to Laioutr and ship your custom token meanwhile. Never redefine a canonical token name.

## Where this lives in your app

```ts
// src/runtime/shared/tokens/store-locator.ts (shared, because server and frontend both import it)
import { z } from 'zod/v4';
import { DateTime } from '@laioutr-core/core-types/common';
import { defineActionToken } from '@laioutr-core/core-types/orchestr';
import { PersonName } from '@laioutr-core/canonical-types';

/**
 * Book an appointment at a store.
 *
 * @public
 * @throws {@link UnauthenticatedError}
 * @throws {@link AppointmentSlotTakenError}
 */
export const StoreAppointmentBookAction = defineActionToken('store-locator/appointment/book', {
  input: z.object({
    storeId: z.string(),
    start: DateTime,
    person: PersonName,
  }),
  output: z.object({
    id: z.string(),
    status: z.enum(['confirmed', 'pending']),
  }),
});
```

- **Handler:** `src/runtime/server/orchestr/<feature>/<verb>.action.ts`, with `export default defineMyAppAction(StoreAppointmentBookAction, async ({ input, context, clientEnv }) => { … })`. Here `defineMyAppAction` is your base orchestr's `.actionHandler` ([Actions](https://docs.laioutr.com/frontend/orchestr/actions#action-handler), [Coding Standards → Orchestr](https://docs.laioutr.com/apps/app-development/coding-standards#orchestr)).
- **Frontend:** `const { mutateAsync, isLoading } = useMutationAction(StoreAppointmentBookAction)`. It is auto-imported, like `useQueryAction`, `fetchAction` and `useFetchAction`.
- **Endpoint:** `POST /api/orchestr/action/store-locator/appointment/book`. Input travels as JSON and is validated against `input` before the handler runs. Output returns as turbo-stream.

## Naming

- **Token name:** `<your-namespace>/<entity-or-feature>/<verb>`, in kebab-case. The namespace is your app (`store-locator/…`, `acme/…`), so names cannot collide with other packages or with canonical `ecommerce/…`. It **must match `^[a-z0-9-/]+$`**, because the name becomes a URL path and `defineActionToken` throws at load time otherwise. So no `@`, `_` or uppercase, even if your package name has them: for a scoped package drop the `@` and join with `-` (`@acme/stock-alerts` → `acme-stock-alerts/stock-alert/subscribe`), keeping three segments.
- **Const:** PascalCase with an `Action` suffix (`StoreAppointmentBookAction`).
- **Defaults:** omitted `input` is `z.undefined()`, and omitted `output` is `z.void()`.

## Workflow

Do these in order. Don't jump to writing the schema.

1. **Confirm it's an action** (table above). If it is a query, link or component, stop and use that token kind.
2. **Check for a canonical token** that already covers the operation. If one exists, implement its handler and stop.
3. **Grep for the shape, not just the entity.** Read the closest siblings: canonical actions under `node_modules/@laioutr-core/canonical-types/dist/server/lib/<domain>/` (the `.d.ts` files carry the JSDoc), and your app's own tokens. Copy the fitting idiom verbatim: `{ id, status }` for a create, `FeedbackResponse` for a pure acknowledgement, no `output` for fire-and-forget, a bare `z.array` for bulk input, `z.discriminatedUnion('type', …)` for mixed items.
4. **Research the backend operations the handler couples to.** For each system, find which fields the operation requires, which values are **coded** (ISO 3166 / ISO 4217 / E.164 / BCP-47, so store the code), and which keys and enums it uses. Dispatch subagents for web research if you can. Look in priority order:
   - **the coupled API first.** commercetools "update actions" are the best public catalogue of mutation shapes. Cross-check the Shopify Storefront API, the Shopware 6 Store API and Adobe Commerce GraphQL to tell common shape from vendor quirk.
   - **the domain standard:** OAuth 2.0 / OpenID Connect and WebAuthn for auth, SCIM and WHATWG autofill tokens for customer and address input, the W3C Payment Request API for payments, the Segment / GA4 e-commerce specs for tracking, RFC 9457 for error envelopes.
   - **schema.org Actions** (`BuyAction`, `RegisterAction`, …) as a conceptual cross-check.
5. **Propose the design and confirm it before writing.** Present the token kind, input, output and error set, the alternatives the research surfaced and why you rejected them, and the companion query the UI needs to render the result.
6. **Write the input and output schemas** ([`input-output-schemas.md`](./references/input-output-schemas.md)). Where backends disagree, apply [`shape-conflicts-and-verb-splitting.md`](./references/shape-conflicts-and-verb-splitting.md).
7. **Declare the error contract.** Add `@throws {@link XError}` for every semantic failure, reusing canonical classes first ([`error-contract.md`](./references/error-contract.md)).

## Common mistakes

| Mistake | Fix |
|---|---|
| Deciding query vs action by *where* the read is triggered | A query needs an entity. A scalar or value type is an action, even when a page renders it. |
| Inventing an entity just to make a scalar read a query | An action with an object-wrapped output (`{ balance: Money }`) |
| A custom token for an operation a canonical token covers | Implement the canonical handler |
| A token name with `@`, `_`, uppercase or no app namespace | `<your-namespace>/<feature>/<verb>`, matching `^[a-z0-9-/]+$` |
| Inventing field names or enums the backend already defines | Adopt the backend's vocabulary and codes |
| An input keyed by a handle the frontend must fetch first | Key by what the frontend holds. The handler resolves the rest. |
| A whole-call failure as an `error?` output field | A thrown typed error, declared with `@throws` |
| Throwing for per-item failures in a batch | A per-item result list in the output |
| `FeedbackResponse` for a create, dropping the id | `{ id, status }` |
| A closed `z.enum` for a backend-extensible set | `wellKnownString(WELL_KNOWN_X)` |
| A display label paired onto a WellKnown **input** | Key only, plus an optional free-text `note`. Labels ride on outputs. |
| `z.date()` in an input | `CalendarDate` / `DateTime`, or a naive wall-clock time + `timeZone` for a future time |
| Shipping the mutation without its read side | Scope the companion query and entity alongside it |

## References

- [`references/token-kind-decision.md`](./references/token-kind-decision.md): unsure whether it is an action, query or link. Covers request-scoped facts, companion reads, and which composable calls what.
- [`references/input-output-schemas.md`](./references/input-output-schemas.md): writing `input` / `output`. Covers frontend-first keys, ids, arrays and unions, raw rows, server-stamped facts, output shapes, batch outcomes, `wellKnownString` direction rules, temporal types, files.
- [`references/error-contract.md`](./references/error-contract.md): the action can fail. Covers class shape, `@ebec/http` status codes, `@throws`, canonical errors, matching an error in a component.
- [`references/shape-conflicts-and-verb-splitting.md`](./references/shape-conflicts-and-verb-splitting.md): backends disagree on shape, two verbs share a subject, a create/update pair, or a session-default resource.

Related skill: `designing-entity-components`, for the entity and query that render an action's result.
