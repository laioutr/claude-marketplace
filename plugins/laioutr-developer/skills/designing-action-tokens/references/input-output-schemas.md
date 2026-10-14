# Action Input and Output Schemas

How to shape the `input` and `output` of a `defineActionToken`. An action's input/output *is* the contract that the frontend, your handler and every backend system the handler calls must line up with. Reuse a sibling's idiom and the shared value types before inventing a shape. Canonical actions to copy from are documented under [Canonical Actions](https://docs.laioutr.com/frontend/api-reference/ecommerce/actions) and installed under `node_modules/@laioutr-core/canonical-types/dist/server/lib/<domain>/<entity>/<verb>.action.d.ts`.

## Design for the frontend's hand

Actions are called from the frontend with the data the frontend already has. Frontend ↔ backend-for-frontend round-trips are the expensive, UX-critical resource. The handler owns any lookup or join, because a hop from your handler to the backend is cheap.

**Never design an input that forces a read before the write.** Key inputs by what the render tree already delivered: the product reference on a listing card, the id of the rendered row. Do not key them by internal handles the frontend would have to fetch first, such as a list-entry id, a share id or a mapping id. This principle drives the natural-key `set`, the optional default subject, and per-item batch outputs (one call in, the complete outcome back). See [`shape-conflicts-and-verb-splitting.md`](./shape-conflicts-and-verb-splitting.md).

## Input rules

- **Omit `input` for a no-argument action.** It defaults to `z.undefined()`. Don't write `z.void()` or an empty object.
- **Ids are plain string inputs.** An action is an RPC, so `productId`, `variantId` and `id: z.string()` are legitimate parameters. Foreign keys stay out of *stored* entity data, but action arguments are not stored.
- **Reuse shared value types** (`Credentials`, `MailingAddress`, `PersonName`, `PhoneNumber`, `Money`) and compose them, for example `z.object({ ...Credentials.shape, ...PersonName.shape, address: MailingAddress })`.
- **A naturally plural payload is a bare `z.array`.** Canonical `ecommerce/cart/remove-items` takes `z.array(z.string())`, and `ecommerce/cart/add-items` takes `z.array(CartAddItemInput)`. Wrap the array in an object only when it travels with something else. Canonical `ecommerce/product-list/add-items` takes `{ productListId?, items }`.
- **Heterogeneous items: `z.discriminatedUnion('type', [...])`** with named member schemas. Canonical `cart/add-items` uses the `product`, `discount-code`, `sku` and `custom` members.
- **Raw passthrough rows are typed loosely.** A spreadsheet or bulk-import row the user may have filled in wrong is `z.string().optional()`, not `Money` or a required field. Canonical `CartAddItemSkuInput.quantity` is a raw string. Inputs are validated against the zod schema before your handler runs, so one malformed row with a strict type fails the **whole array**. Type raw rows loosely, validate in the handler, and report per row.
- **Server-stamped facts are not input.** Locale, an audit `created_at` and the session identity belong to the handler (from `clientEnv` and the request). A client-authored **domain** time *is* legitimate input: a scheduled publish instant, a chosen delivery or pickup date. The test is who owns the value.
- **Inputs travel as plain JSON.** The client POSTs `{ input, clientEnv }` as JSON, while outputs come back as turbo-stream (which revives `Date`). A `z.date()` input therefore receives a string and rejects its own serialized form. Use the temporal types below.

## Output rules

- **Omit `output` for fire-and-forget mutations.** It defaults to `z.void()` (canonical `cart/remove-items`, `customer/address-create`).
- **Object-wrap every non-void output** (`{ customer }`, `{ addresses }`, `{ checkoutUrl }`, `{ downloadUrl }`), so a field can be added later without a breaking reshape. A bare scalar or array output paints you into a corner.
- **Choose the output by what the caller needs afterward:**
  - `{ id, status }` when the action creates something addressed individually later. Canonical `b2b/quote/request` returns `{ id, status, statusLabel? }`, and `ecommerce/product-list/create` returns `{ id }`.
  - void when the resulting state is re-read wholesale by a companion query or action (`customer/address-*`, `b2b/quote/accept`).
  - `FeedbackResponse` (`{ success, message? }`, from `@laioutr-core/canonical-types`) only for a **pure acknowledgement** with nothing else to hand back (`ecommerce/auth/login`, `ecommerce/legal/withdrawal`, `ecommerce/product/reviews/create`). Never hand-roll `{ ok, error }`.
- **Semantic non-failure outcomes are output values, not throws.** Canonical `newsletter/subscribe` returns `status: 'success' | 'error' | 'already-subscribed'`, because "already subscribed" is a normal result — even when the backend signals it with a 4xx such as 409. When you copy that idiom, drop its legacy `'error'` member: a failure is a thrown error, never an output value.
- **Partial-batch outcomes are output data, not throws.** When one call processes many items that can fail independently, return a **per-item result list** and throw only for whole-call failures (auth, unsupported operation, malformed request). Canonical precedent: `cart/add-items` returns `{ items: CartBatchResultItem[] }`, where each item is `{ status: 'added', … }` or `{ status: 'rejected', reason }` with `reason` from an open code set.
- **No per-action `error?: string` field.** A whole-call failure is a thrown typed error. See [`error-contract.md`](./error-contract.md).

## Open coded sets: `wellKnownString`

A field with a known core of values that backends extend (a return reason, a payment-method key, a status) is an open coded set. Do not freeze it to a closed `z.enum`, and do not store a display string. Store the **code**, and let the frontend map it to a label and icon. Build it with `wellKnownString` from `@laioutr-core/core-types/common`, as canonical `DocumentType` and `BatchRejectionReason` do:

```ts
import { wellKnownString, type WellKnownString } from '@laioutr-core/core-types/common';

export const WELL_KNOWN_RETURN_REASONS = ['damaged', 'wrong-item', 'wrong-size'] as const;
export type WellKnownReturnReason = WellKnownString<typeof WELL_KNOWN_RETURN_REASONS>;

// in the schema
reason: wellKnownString(WELL_KNOWN_RETURN_REASONS),
```

Use a **closed `z.enum`** only when the set is fixed and owned by your app or the platform: a lifecycle status like `newsletter/subscribe`'s, or a refund method your own storefront offers.

**Whether a display label rides along depends on direction:**

- **Output:** the backend resolves a key the frontend may not recognize. Pair the key with a connector-resolved label (`{ status, statusLabel? }`, as `b2b/quote/request` does), so an unmapped key still shows something.
- **Input:** the frontend presents the picklist and already owns the labels. Carry the **key only**. If the shopper elaborates, add a free-text `note`, never a paired display label.

**Entity-side vocabulary wins.** If an entity component already defines the concept, import its type instead of minting a sibling set: `WellKnownQuoteStatus` (canonical `Quote`), `WellKnownBudgetPeriod` (canonical `Budget`), or your own entity's set.

## Temporal values: pick by precision

All of these come from `@laioutr-core/core-types/common`:

| Concept | Type |
|---|---|
| Whole day (delivery date, check-in) | `CalendarDate` (`YYYY-MM-DD`) |
| Time of day (a slot's wall-clock time) | `Time` (`{ time }`) |
| An instant already fixed (an appointment `start` the store offered) | `DateTime` (`z.iso.datetime({ offset: true })`). Never bare `z.date()`, which fails the JSON input transport and normalizes away the source system's UTC offset. |
| Elapsed or configured length (a subscription interval, a watch time) | `Duration` (`{ duration }`) |
| A date or instant **range** | A closed domain pair (`{ checkIn, checkOut }` of `CalendarDate` / `DateTime`), **not** `Timespan`, which is open-ended and `z.date()`-based |

**Two traps for a *future* time the user schedules** (a publish-at, a reminder):

1. **Don't accept a client-resolved offset instant.** The UTC offset of a future date can be wrong or disputed between client and server time-zone data. Accept a **naive wall-clock time** (`z.iso.datetime({ local: true })`) **plus the IANA zone**, and let the handler resolve the instant. The handler is the authority on whether the time is still in the future.
2. **Keep the zone the user chose.** A resolved instant does not preserve which named zone the user picked, so carry `timeZone: z.string()` alongside it. The UI can then redisplay and re-edit in the original zone, not the viewer's.

## Files are not `Media`, and bytes never travel in a payload

`Media` / `MediaImage` cover image, video and audio for on-page rendering. A PDF or an ID scan is not one of them. A file's bytes never travel in an action payload. For downloads, return a URL from the action and let the client fetch the file. Canonical `ecommerce/document/get-download-url` returns `{ downloadUrl }`, which may be short-lived, so fetch it on demand and never persist it. For uploads, use a **staged upload**: a `request-upload` action returns a presigned URL, the client PUTs the file directly, and a `finalize` action registers it by reference.

## When the type you need is missing

Create it. Don't fall back to `z.string()` / `z.date()`, and don't copy an inline shape across sibling actions. Define it once in your app (for example in `src/runtime/shared/`, beside your tokens), and reuse it across the action and any entity component that carries the same concept. If it is a generic primitive every app would want, also propose it upstream. "Reuse before invent" does not mean "fall back before create".

## Forbidden patterns

| Pattern | Instead |
|---|---|
| `input: z.void()` / `z.object({})` for a no-argument action | Omit `input` |
| An input keyed by a handle the frontend must fetch first | Key by what the frontend already holds. The handler resolves internal handles. |
| Strict value types on raw import rows | `z.string().optional()`, validated per row in the handler |
| Locale, audit timestamps or session identity as input | Read them in the handler |
| `z.date()` in an input | `CalendarDate` / `DateTime` / naive wall-clock + `timeZone` |
| A bare scalar or array output | Object-wrap it |
| `FeedbackResponse` for a create, dropping the new id | `{ id, status }` |
| A whole-call failure as an `error?` output field | A thrown typed error |
| Throwing for 30 bad rows out of 1000 | A per-item result list |
| A closed `z.enum` for a backend-extensible set | `wellKnownString` |
| A display label paired onto a WellKnown **input** | Key only, plus an optional free-text `note` |
| File bytes in a payload, or a document typed as `Media` | A download URL or a staged upload |
