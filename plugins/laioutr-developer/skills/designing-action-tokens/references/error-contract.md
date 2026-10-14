# Whole-Call Failures Are Thrown Typed Errors, Declared on the Token

When an action fails as a whole, the handler **throws a typed error class**, and the token's JSDoc declares it with `@throws {@link XError}`. Never encode a whole-call failure in the output (`error?: string`, `{ ok: false }`). Reuse a canonical error class before you write your own. A new class gets a semantic name, an HTTP base class, a `static readonly code` and typed `data`.

## The rule

```ts
// ✘ the failure is output data, so every caller must remember to check it
export const StoreAppointmentBookAction = defineActionToken('store-locator/appointment/book', {
  input: z.object({ storeId: z.string(), start: DateTime }),
  output: z.object({ id: z.string().optional(), error: z.string().optional() }),
});
```

```ts
// ✅ src/runtime/shared/tokens/errors/appointment-slot-taken.error.ts
import { ConflictError } from '@ebec/http';

/**
 * Thrown when the requested appointment slot was booked by someone else.
 *
 * @public
 */
export class AppointmentSlotTakenError extends ConflictError {
  declare data: { storeId: string; start: string };
  static readonly code = 'APPOINTMENT_SLOT_TAKEN';

  constructor(storeId: string, start: string) {
    super({
      message: `Slot ${start} at store ${storeId} is no longer available`,
      code: AppointmentSlotTakenError.code,
      data: { storeId, start },
    });
  }
}
```

```ts
// ✅ src/runtime/shared/tokens/store-locator.ts
/**
 * Book an appointment at a store.
 *
 * @public
 * @throws {@link UnauthenticatedError}
 * @throws {@link AppointmentSlotTakenError}
 */
export const StoreAppointmentBookAction = defineActionToken('store-locator/appointment/book', {
  input: z.object({ storeId: z.string(), start: DateTime }),
  output: z.object({ id: z.string(), status: z.enum(['confirmed', 'pending']) }),
});
```

## Why

1. **The status code comes from the class.** The action endpoint answers with the thrown error's `statusCode`, or 500 when it has none. An `@ebec/http` class (`NotFoundError` 404, `BadRequestError` 400, `ConflictError` 409, `PreconditionFailedError` 412, `UnauthorizedError` 401, …) carries that status. A plain `BaseError` from `ebec`, or any `Error`, ends up as a 500. A failure returned as output is a 200, so caches, logs and DevTools all record it as a success.
2. **The frontend branches on the class name.** Orchestr serializes the thrown error, and the client re-throws a generic `Error` whose `cause` carries `name` (prefixed `[SERVER] `), `message` and `data`. `data` arrives intact. A semantic name (`AppointmentSlotTakenError`, not `AcmeApiError`) is the stable handle a component matches on.
3. **The JSDoc `@throws` is the contract.** The output schema documents the success. `@throws` documents every failure a caller must handle, and Laioutr's own action tokens declare their errors the same way.
4. **Canonical classes come with frontend handling.** Laioutr's storefront templates already branch on the canonical error classes, so throwing `UnauthenticatedError` or `InvalidFieldError` works without any frontend change. A custom class needs its own UI handling.

## When to apply

- Any failure of the call **as a whole**: unauthenticated, not found, a precondition not met, a backend validation error, an unsupported operation.
- **Not** for a semantic non-failure outcome ("already subscribed"), which is an output enum value.
- **Not** for per-item rejections in a batch, which are output data. Throw only when the whole call fails. See [`input-output-schemas.md`](./input-output-schemas.md#output-rules).

## How to declare an error

1. **Reuse a canonical class first.** The ecommerce errors are imported from `@laioutr-core/canonical-types/ecommerce`: `UnauthenticatedError`, `InvalidCredentialsError`, `InvalidFieldError` (backend validation, with `data: { reason, field }`), `ProductNotFoundError`, and more. See the [Errors reference](https://docs.laioutr.com/frontend/api-reference/ecommerce/errors).
2. **Otherwise write a class** that extends the `@ebec/http` class matching the HTTP status, or `BaseError` from `ebec` for a 500. Give it a `static readonly code` (SCREAMING_SNAKE), a clear `message` and typed `data`, and mark it `@public`. See [Actions → Error Handling](https://docs.laioutr.com/frontend/orchestr/actions#error-handling).
3. **Place an error that is part of the token's contract next to the token**, in `src/runtime/shared/`, so your components can import the class to match its `name`. This is the same reason the token itself lives in `shared/`. Errors only your server code uses internally can follow the docs' `orchestr/<entity>/errors/*.error.ts` layout ([Coding Standards → Errors](https://docs.laioutr.com/apps/app-development/coding-standards#errors)).
4. **Add `@throws {@link XError}`** to the token's JSDoc for every semantic failure.
5. **Translate backend errors in the handler.** Map the vendor's error codes to your classes, and throw a plain `Error` for unknown codes so they surface as a loud 500, not a silent success ([Translating vendor errors](https://docs.laioutr.com/frontend/orchestr/recipes/translating-vendor-errors)).

## Matching an error in a component

```ts
const isServerError = (error: unknown, ErrorClass: { name: string }): boolean =>
  ((error as Error | undefined)?.cause as { name?: string } | undefined)?.name === `[SERVER] ${ErrorClass.name}`;

const { mutateAsync: book } = useMutationAction(StoreAppointmentBookAction);

try {
  await book({ storeId, start });
} catch (error) {
  if (isServerError(error, AppointmentSlotTakenError)) {
    // offer the next free slot
  } else {
    throw error;
  }
}
```

## Related

- [`input-output-schemas.md`](./input-output-schemas.md): outcomes that are output data, not errors.
- [`shape-conflicts-and-verb-splitting.md`](./shape-conflicts-and-verb-splitting.md): `InvalidFieldError` for inputs a narrow backend cannot represent.
