# Shape Conflicts, Verb Splitting and the Default Subject

Four recurring design decisions surface once you have researched the backends an action targets. Each has a test. Apply the test and record the outcome in the proposal you confirm with the user.

## When source systems disagree on shape

Research routinely turns up **structural disagreements** between the systems an action could target: one backend's field is an array where another's is a scalar, a value set differs, or a whole operation is missing. Never resolve the disagreement by silently adopting one vendor's shape. Pick a resolution by the kind of conflict:

| Conflict | Resolution |
|---|---|
| **Cardinality or richness of a load-bearing field** (one role per employee vs many; a personal-only list vs a shareable one) | **Widen to the most general shape** (`roleIds: z.array(…)`, a `visibility` enum). A handler whose backend cannot represent an input **throws `InvalidFieldError`**, for example when given two roles on a single-role backend. The narrow shape silently drops data on richer backends, while the wide shape fails loudly on narrow ones. Loud beats lossy. |
| **An auxiliary or metadata field some backends lack** (a PO number on a quote request, a notification threshold) | Keep it **optional**, and document that handlers ignore what they cannot store. Canonical `ecommerce/cart/update-items` documents "unsupported features … will be ignored". |
| **The same concept, encoded differently** (decimal vs minor-unit amounts, vendor status enums) | Normalize by convention (`Money` minor units, ISO codes, `wellKnownString` code sets) and let the handler translate. No schema impact. |
| **A whole operation missing on one backend** (a backend can deactivate users but not delete them) | That backend's app **doesn't register a handler**, or throws a typed "not possible" error for per-item cases (canonical `EmployeeNotDeletableError`). Don't shrink the contract to the poorest backend. |

The test: *would the narrower choice silently lose data or intent on some backend?* If yes, widen it and use a typed error. If the field is just extra metadata, make it optional and document that it may be ignored.

## One token or two: merging and splitting verbs

Two operations on the same subject (approve/reject, accept/decline, publish/unpublish) can be one token with an enum or two tokens. Decide by whether they share a **contract**, not merely a subject:

- **Merge** when only the enum value differs: the same input fields, the same `@throws` set, the same permission, the same precondition, and symmetric effects. Canonical example: `b2b/approval-flow/decide` with `decision: z.enum(['approve', 'reject'])`. SAP's decision endpoint takes one decision field, and Shopware guards both outcomes with a single permission.
- **Split** when the outcomes differ in contract: a field meaningful for only one branch, an error thrown by only one branch, different follow-up flows, or (the strongest signal) **the source systems' own permission models split them**. Canonical example: `b2b/quote/accept` and `b2b/quote/decline` are separate. Only accept throws `QuoteExpiredError`, only decline carries a `comment`, and accept continues into checkout while decline is terminal. commercetools and Shopware both split the permissions too.

A merged token whose fields, errors or permissions vary by enum value is a discriminated union in disguise. When behavior forks on the field, the fork wants to be two tokens.

## Create + update collapse into a keyed `set`

The same contract test collapses a create/update pair into **one keyed `set`** when the resource has a **natural key with at most one instance per key**. Canonical example: `b2b/organization/product-number-set`. There is exactly one customer product number per (customer, variant), so the variant id is the key. A create/update pair plus a synthetic `id` is ceremony here: the caller already holds the key, a returned `{ id }` is a handle nobody needs, and already-exists conflicts cannot occur. Key the `set` (and its `delete`) by the natural key, and drop the synthetic id from the whole family.

## The implicit default subject

Some resources have a session-default instance: *the* cart, *the* wishlist. For these:

- **Item-level operations** take the subject id as **optional**. When it is absent, the handler targets the default instance, and resolves or creates it, so a one-click add-to-wishlist works without a prior create. Canonical example: `ecommerce/product-list/add-items` with `productListId?` ("Absent ⇒ the customer's default wishlist").
- **Subject-level operations** (rename, delete, duplicate, share) always require the **explicit id**. Nothing destructive or structural should target "whatever is the default" by omission.

The cart family is the degenerate case: there is no id at all, because there is a single session cart. The `ecommerce/product-list/*` family is the mixed case.

## Forbidden patterns

| Pattern | Instead |
|---|---|
| Adopting one vendor's narrower shape when backends disagree | Widen it. Narrow backends throw `InvalidFieldError`. |
| Shrinking the contract to what the poorest backend can do | That backend doesn't register the handler, or throws a typed error |
| Two tokens for a symmetric decision with identical contracts | One token with an enum |
| One token for asymmetric verbs whose fields, errors or permissions differ by branch | Two tokens |
| A create/update pair plus a synthetic id for a natural-key resource | One keyed `set` |
| Requiring the subject id on every item operation of a session-default resource | Optional id. Absent means the default instance. |
| Accepting an absent id on a destructive or structural operation | The explicit id is required |
